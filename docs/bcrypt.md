---
id: bcrypt
title: OpenSSH's bcrypt implementation
---

OpenSSH encrypted private keys use a modified form of bcrypt for [password hashing](https://en.wikipedia.org/wiki/Key_derivation_function#Password_hashing). Because of these modifications neither PHP's [password_hash()](https://www.php.net/manual/en/function.password-hash.php) or [crypt()](https://www.php.net/manual/en/function.crypt.php) can be used. [OpenSSH's bcrypt_pbkdf.c](https://github.com/openssh/openssh-portable/blob/master/openbsd-compat/bcrypt_pbkdf.c) enumerates on some of these changes but not all of them.

To illustrate the differences we'll consider a number of different scenarios.

## Making phpseclib match `crypt()`

The following changes (notated using the [phpBB MOD Text Template](phpbb.md#actions)) are for [phpseclib 3.0.38's version of phpseclib/Crypt/Blowfish.php](https://github.com/phpseclib/phpseclib/blob/3.0.38/phpseclib/Crypt/Blowfish.php). They may or may not work on other versions.

```
#
#-----[ FIND ]------------------------------------------
#
private static function bcrypt_hash($sha2pass, $sha2salt)
#
#-----[ REPLACE WITH ]----------------------------------
# this change is so that we can verify the changes
#
public static function bcrypt_hash($sha2pass, $sha2salt)
#
#-----[ FIND ]------------------------------------------
#
$cdata = array_values(unpack('N*', 'OxychromaticBlowfishSwatDynamite'));
#
#-----[ REPLACE WITH ]----------------------------------
# OpenSSH's bcrypt_pbkdf.c documents this change
#
$cdata = array_values(unpack('N*', 'OrpheanBeholderScryDoubt'));
#
#-----[ FIND ]------------------------------------------
#
    self::expand0state($sha2salt, $sbox, $p);
    self::expand0state($sha2pass, $sbox, $p);
#
#-----[ REPLACE WITH ]----------------------------------
# this is an undocumented change; basically, the function calls are swapped
#
    self::expand0state($sha2pass, $sbox, $p);
    self::expand0state($sha2salt, $sbox, $p);
#
#-----[ FIND ]------------------------------------------
#
    for ($j = 0; $j < 8; $j += 2) { // count($cdata) == 8
#
#-----[ REPLACE WITH ]----------------------------------
# this change follows as a natural consequence of the OrpheanBeholderScryDoubt change
#
    for ($j = 0; $j < 6; $j += 2) { // count($cdata) == 6
#
#-----[ FIND ]------------------------------------------
# in 3.0.39+ this'll be V* - not L*
#
return pack('L*', ...$cdata);
#
#-----[ REPLACE WITH ]----------------------------------
# this is an undocumented change
#
return pack('N*', ...$cdata);
```
Even with these changes you'll still need to call `Blowfish::bcrypt_hash()` differently than you would `crypt()`. `password_hash()` won't do since "_as of PHP 8.0.0, an explicitly given salt is ignored_".

```
$salt = '1234567812345678';

// quoting OpenSSH's bcrypt_pbkdf.c, "input password and salt are preprocessed with SHA512"
// this means that we need to make them both 64 bytes long altho we'll expand the salt later
$pass = 'aaaaaaaabbbbbbbbccccccccddddddddaaaaaaaabbbbbbbbccccccccdddddddd';

// we concatenate $salt to itself so that it's 64 bytes long
// as wikipedia notes, the output is "the "base-64 encoding of the first 23 bytes of the computed 24 byte hash",
// hence the substr(..., 0, -1) bit
$hash = substr(Blowfish::bcrypt_hash($pass, $salt . $salt . $salt . $salt), 0, -1);

echo '$2a$06$' . encodeBase64($salt) . encodeBase64($hash) . "\n";

// we concatenate substr($pass, 0, 8) to $pass because that's what OpenSSH's bcrypt_pbkdf.c does
//
// the $2a$ part of the salt tells bcrypt to add a null byte to the end. as https://en.wikipedia.org/wiki/Bcrypt
// notes you're supposed to "treat the password as cyclic" but if you're adding a null byte to the end you're
// essentially doing "\0" . substr($pass, 0, 7) vs substr($pass, 0, 8)
//
// some bcrypt implementations support $2$, which doesn't specify whether or not a null byte ought to be tacked
// onto the end or not, so of those that do support $2$ it's a tossup as to how it'd behave.
//
// $2x$ and $2y$ behave as $2a$ does w.r.t. the null byte. PHP doesn't support $2$
//
// the $06$ bit means that OpenSSH's implementation has a "cost" of 6
echo crypt($pass . substr($pass, 0, 8), '$2a$06$' . encodeBase64($salt));

// encodeBase64() is used vs base64_encode() because crypt() uses it's own custom alphabet.
function encodeBase64($input)
{
    $old = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/';
    $new = './ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    $input = base64_encode($input);
    $output = '';
    for ($i = 0; $i < strlen($input); $i++) {
        if ($input[$i] == '=') {
            break;
        }
        $pos = strpos($old, $input[$i]);
        $output.= $new[$pos];
    }

    return $output;
}
```
The two key takeaways from this are: (1) `Blowfish::bcrypt_hash()` expects 64 byte inputs whereas the normal bcrypt uses 16 byte salts and variable length passwords and (2) the last byte of `Blowfish::bcrypt_hash()` need to be removed.

If one wanted to make `Blowfish::bcrypt_hash()` support variable length passwords and 16 byte salts the following additional changes would need to be made:

```
#
#-----[ FIND ]------------------------------------------
# in bcrypt_hash()
#
        $sha2pass = array_values(unpack('N*', $sha2pass));
#
#-----[ BEFORE, ADD ]-----------------------------------
#
        // per the $2a$ bit we append a null byte and then "treat the password as cyclic"
        $sha2pass.= "\0";
        while (strlen($sha2pass) < 72) {
            $sha2pass.= $sha2pass;
        }
        $sha2pass = substr($sha2pass, 0, 72);
#
#-----[ FIND ]------------------------------------------
# in expand0state()
#
        $p = [
            $p[0] ^ $key[0],
            $p[1] ^ $key[1],
            $p[2] ^ $key[2],
            $p[3] ^ $key[3],
            $p[4] ^ $key[4],
            $p[5] ^ $key[5],
            $p[6] ^ $key[6],
            $p[7] ^ $key[7],
            $p[8] ^ $key[8],
            $p[9] ^ $key[9],
            $p[10] ^ $key[10],
            $p[11] ^ $key[11],
            $p[12] ^ $key[12],
            $p[13] ^ $key[13],
            $p[14] ^ $key[14],
            $p[15] ^ $key[15],
            $p[16] ^ $key[0],
            $p[17] ^ $key[1]
        ];
#
#-----[ REPLACE WITH ]----------------------------------
# this is a documented change; we can't keep the loop unrolled version
# unless we expand the salt out to 72 bytes as well to match the password
# size
#
        for ($i = 0; $i < 18; $i++) {
            $p[$i]^= $key[$i % count($key)];
        }
#
#-----[ FIND ]------------------------------------------
# in expandstate()
#
            $p[16] ^ $key[0],
            $p[17] ^ $key[1]
#
#-----[ REPLACE WITH ]----------------------------------
# this is a documented change
#
            $p[16] ^ $key[16],
            $p[17] ^ $key[17]
#
#-----[ FIND ]------------------------------------------
# in expandstate()
#
        // @codingStandardsIgnoreStart
        list( $p[0],  $p[1]) = self::encryptBlockHelperFast($data[ 0]         , $data[ 1]         , $sbox, $p);
        list( $p[2],  $p[3]) = self::encryptBlockHelperFast($data[ 2] ^ $p[ 0], $data[ 3] ^ $p[ 1], $sbox, $p);
        list( $p[4],  $p[5]) = self::encryptBlockHelperFast($data[ 4] ^ $p[ 2], $data[ 5] ^ $p[ 3], $sbox, $p);
        list( $p[6],  $p[7]) = self::encryptBlockHelperFast($data[ 6] ^ $p[ 4], $data[ 7] ^ $p[ 5], $sbox, $p);
        list( $p[8],  $p[9]) = self::encryptBlockHelperFast($data[ 8] ^ $p[ 6], $data[ 9] ^ $p[ 7], $sbox, $p);
        list($p[10], $p[11]) = self::encryptBlockHelperFast($data[10] ^ $p[ 8], $data[11] ^ $p[ 9], $sbox, $p);
        list($p[12], $p[13]) = self::encryptBlockHelperFast($data[12] ^ $p[10], $data[13] ^ $p[11], $sbox, $p);
        list($p[14], $p[15]) = self::encryptBlockHelperFast($data[14] ^ $p[12], $data[15] ^ $p[13], $sbox, $p);
        list($p[16], $p[17]) = self::encryptBlockHelperFast($data[ 0] ^ $p[14], $data[ 1] ^ $p[15], $sbox, $p);
        // @codingStandardsIgnoreEnd

        list($sbox[0], $sbox[1]) = self::encryptBlockHelperFast($data[2] ^ $p[16], $data[3] ^ $p[17], $sbox, $p);
        for ($i = 2, $j = 4; $i < 1024; $i += 2, $j = ($j + 2) % 16) { // instead of 16 maybe count($data) would be better?
            list($sbox[$i], $sbox[$i + 1]) = self::encryptBlockHelperFast($data[$j] ^ $sbox[$i - 2], $data[$j + 1] ^ $sbox[$i - 1], $sbox, $p);
        }
#
#-----[ REPLACE WITH ]----------------------------------
# this is a documented change; instead of doing $data[0]..$data[15] we're doing $data[0]..$data[3] cyclically
#
        // @codingStandardsIgnoreStart
        list( $p[0],  $p[1]) = self::encryptBlockHelperFast($data[ 0]         , $data[ 1]         , $sbox, $p);
        list( $p[2],  $p[3]) = self::encryptBlockHelperFast($data[ 2] ^ $p[ 0], $data[ 3] ^ $p[ 1], $sbox, $p);
        list( $p[4],  $p[5]) = self::encryptBlockHelperFast($data[ 0] ^ $p[ 2], $data[ 1] ^ $p[ 3], $sbox, $p);
        list( $p[6],  $p[7]) = self::encryptBlockHelperFast($data[ 2] ^ $p[ 4], $data[ 3] ^ $p[ 5], $sbox, $p);
        list( $p[8],  $p[9]) = self::encryptBlockHelperFast($data[ 0] ^ $p[ 6], $data[ 1] ^ $p[ 7], $sbox, $p);
        list($p[10], $p[11]) = self::encryptBlockHelperFast($data[ 2] ^ $p[ 8], $data[ 3] ^ $p[ 9], $sbox, $p);
        list($p[12], $p[13]) = self::encryptBlockHelperFast($data[ 0] ^ $p[10], $data[ 1] ^ $p[11], $sbox, $p);
        list($p[14], $p[15]) = self::encryptBlockHelperFast($data[ 2] ^ $p[12], $data[ 3] ^ $p[13], $sbox, $p);
        list($p[16], $p[17]) = self::encryptBlockHelperFast($data[ 0] ^ $p[14], $data[ 1] ^ $p[15], $sbox, $p);
        // @codingStandardsIgnoreEnd

        list($sbox[0], $sbox[1]) = self::encryptBlockHelperFast($data[2] ^ $p[16], $data[3] ^ $p[17], $sbox, $p);
        for ($i = 2, $j = 0; $i < 1024; $i += 2, $j%= 4) { // instead of 16 maybe count($data) would be better?
            list($sbox[$i], $sbox[$i + 1]) = self::encryptBlockHelperFast($data[$j++] ^ $sbox[$i - 2], $data[$j++] ^ $sbox[$i - 1], $sbox, $p);
        }
```

## Making Wikipedia match OpenSSH

The following changes (notated using the [phpBB MOD Text Template](phpbb.md#actions)) are for [revision 08:07, 14 May 2024 of wikipedia's bcrypt article](https://en.wikipedia.org/w/index.php?title=Bcrypt&oldid=1223800920#Algorithm). They may or may not work on other revisions.

```
#
#-----[ FIND ]------------------------------------------
# in Function bcrypt
#
   Input:
      password: array of Bytes (1..72 bytes)   UTF-8 encoded password
      salt:     array of Bytes (16 bytes)      random salt
      cost:     Number (4..31)                 log2(Iterations). e.g. 12 ==> 2**12 = 4,096 iterations
#
#-----[ REPLACE WITH ]----------------------------------
# this is a documented change; well, the cost isn't, but everything else is
#
   Input:
      password: array of Bytes (64 bytes)   UTF-8 encoded password
      salt:     array of Bytes (64 bytes)   random salt
      cost:     Number (6)                  64 iterations
#
#-----[ FIND ]------------------------------------------
# in Function bcrypt
#
   ctext ← "OrpheanBeholderScryDoubt"  //24 bytes ==> three 64-bit blocks
#
#-----[ REPLACE WITH ]----------------------------------
# this is a documented change
#
   ctext ← "OxychromaticBlowfishSwatDynamite"  //32 bytes ==> four 64-bit blocks
#
#-----[ FIND ]------------------------------------------
# in Function EksBlowfishSetup
#
      P, S ← ExpandKey(P, S, password, 0)
      P, S ← ExpandKey(P, S, salt, 0)
#
#-----[ REPLACE WITH ]----------------------------------
# this is an undocumented change; we're just swapping the order
#
      P, S ← ExpandKey(P, S, salt, 0)
      P, S ← ExpandKey(P, S, password, 0)
#
#-----[ FIND ]------------------------------------------
# in Function ExpandKey
#
   //Treat the 128-bit salt as two 64-bit halves (the Blowfish block size).
   saltHalf[0] ← salt[0..63]  //Lower 64-bits of salt
   saltHalf[1] ← salt[64..127]  //Upper 64-bits of salt
#
#-----[ REPLACE WITH ]----------------------------------
# this is a documented change
#
   saltChunk[0] ← salt[0..63]
   saltChunk[1] ← salt[64..127]
   saltChunk[2] ← salt[128..191]
   saltChunk[3] ← salt[192..255]
   saltChunk[4] ← salt[256..319]
   saltChunk[5] ← salt[320..383]
   saltChunk[6] ← salt[384..447]
   saltChunk[7] ← salt[448..511]
#
#-----[ FIND ]------------------------------------------
# in Function ExpandKey
#
      block ← block xor saltHalf[(n-1) mod 2] //each iteration alternating between saltHalf[0], and saltHalf[1]
#
#-----[ REPLACE WITH ]----------------------------------
# this is a documented change
#
      block ← block xor saltChunk[n-1]
#
#-----[ FIND ]------------------------------------------
# in Function ExpandKey
#
         block ← Encrypt(state, block xor saltHalf[(n-1) mod 2]) //as above
#
#-----[ REPLACE WITH ]----------------------------------
# this is a documented change
#
         block ← Encrypt(state, block xor saltChunk[(n-1) mod 8])
```
Being pseudo code this doesn't quite capture everything. For example it doesn't capture the change in endianness.

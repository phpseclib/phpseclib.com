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
# this change is to that we can verify the changes
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
The two key takeaways from this are: (1) `Blowfish::bcrypt_hash()` expects 64 byte inputs whereas the normal bcrypt uses 8 byte salts and variable length passwords and (2) the last byte of `Blowfish::bcrypt_hash()` need to be removed.
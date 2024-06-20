---
id: publickeys
title: Overview
---

## Best Practices

Elliptic Curves are the best practices solution for both signature creation / verification and for diffie-hellman key exchange. The specific curves that you ought to be using for signing are Ed25519 (for 128-bit comparable security) or Ed448 (for 256-bit comparable security). For diffie-hellman key exchange the curves that you ought to be using are Curve25519 (for 128-bit comparable security) or Curve448 (for 256-bit comparable security).

Also, doing anything with public / private keys from randos on the internet is ill advisable. For example, loading keys and performing cryptographic operations on a [16384-bit RSA key](https://stackoverflow.com/a/39069182/569976) may be suitably performant on a system with the latest Intel i9 processor and the [GMP extension](https://www.php.net/gmp) installed but what about a Raspberry Pi without either OpenSSL or GMP? phpseclib is designed to be able to operate on both, after all. Maybe dynamic guardrails could be put in place to scale the maximum supported key based on your system specs but, then again, what if you're using phpseclib to connect to an SSH server with an RSA key from behind a password protected admin control panel? Maybe authentication takes 20s because of your system specs and the key you're using and maybe you're okay with waiting that long. That's a perfectly valid use case. On the flip side, if a rando on the internet can send an HTTP request that'll eat up 20 seconds of compute time then that makes it a lot easier for an attacker to perform a [DOS attack](https://en.wikipedia.org/wiki/Denial-of-service_attack) if they were to flood your server with a bunch of said HTTP requests over the span of, say, 30 seconds. Ultimately, phpseclib has no control over how it's used and phpseclib isn't going to neuter valid use cases to safeguard against invalid use cases.

## Loading Keys

The recommended way to load public keys is to do so thusly:

```php
use phpseclib3\Crypt\PublicKeyLoader;

$key = PublicKeyLoader::load(file_get_contents('/path/to/key.pem'), $password = false);
```
The password is optional.

So let's say you have a key that looks like `-----BEGIN PRIVATE KEY-----`. Is that an RSA key, a DSA key, an EC key, or what? With this approach you don't need to know what type the key is - it'll be automatically determined for you. Even passing in an X.509 certificate will work - you'll get back the public key portion of that certificate <sup style="color: red"><strong>[1]</strong></sup>

If a key can't be load a `phpseclib3\Exception\NoKeyLoadedException` exception will be thrown.

If you know whether or not a key is a public or private but just don't know the type (eg. RSA, DSA, EC, etc) you can do `PublicKeyLoader::loadPublicKey()` or `PublicKeyLoader::loadPrivateKey()`.

If you have a key that you believe should have been loadable but wasn't you can get more detailed information on why it didn't load if you know the key type (RSA, DSA, EC, etc) and the format. eg.

```php
use phpseclib3\Crypt\RSA;

$key = RSA::loadFormat('PKCS1', file_get_contents('/path/to/key.pem'), 'password');
```
(again, the password is optional)

All successfully loaded keys are an instance of `phpseclib3\Crypt\Common\AsymmetricKey`.

The format of a successfully loaded key can be determined by looking at `$key->getLoadedFormat()`. This will return a string. Note that keys embedded within X.509 certificates will not identify themselves as X.509 - rather, they'll identify themselves as PKCS8, due to various technical reasons.

<div style="font-size: 11px">

<sup style="color: red"><strong>[1]</strong></sup> When an X.509 key is loaded _no_ validation is performed. A public key will be returned even if the certificate is expired and even if the signature is invalid. The _best_ way to get the public key in an X.509 cert is to use the explicit [X.509 functionality](x509.md) that phpseclib provides but PublicKeyLoader does work, as well, albeit suboptimally.
</div>

## PublicKey vs. PrivateKey

`PublicKeyLoader::load` will return either PublicKey objects, PrivateKey objects or Parameters objects (discussed in another section).

Public and Private Keys can be distinguished as follows:

```php
echo $key instanceof \phpseclib3\Crypt\Common\PublicKey ? 'public key' : 'private key';
echo "\n";
echo $key instanceof \phpseclib3\Crypt\Common\PrivateKey ? 'private key' : 'public key';
```
`PrivateKey` is an interface implementing the following methods:

- `sign($message)`
- `getPublicKey()`
- `toString($type, array $options = [])`
- `withPassword($string)`

RSA PrivateKey objects also implement `decrypt($ciphertext)` but because EC and DSA PrivateKey objects do not support encryption / decryption the interface itself does not define it.

`PublicKey` is an interface implementing the following methods:

- `verify($message, $signature)`
- `toString($type, array $options = [])`
- `getFingerprint($algorithm)`

RSA PublicKey objects also implement `encrypt($plaintext)` but because EC and DSA PrivateKey objects do not support encryption / decryption the interface itself does not define it.

All of these methods are discussed elsewhere, either on this page or on the algorithm specific pages.

## Immutability

All AsymmetricKey object are [immutable](https://en.wikipedia.org/wiki/Immutable_object). Consequently, if you want to set a parameter (eg. the hash) you do `$key = $key->withHash('sha256')` (instead of `$key->setHash('sha256')`).

## Supported Key Formats

Supported key formats for a given algorithm can be determined by doing `\phpseclib3\Crypt\RSA::getSupportedKeyFormats()` (replace `RSA` with `DSA` or `EC` or whatever, as appropriate). What's returned is an associative array wherein the key is the name of the format and the value is the namespaced classname of the plugin that enables phpseclib to read the format.

## Common Key Formats

**PKCS1** keys supports _most_ public key algorithms but not Ed25519 / Curve25519 <sup style="color: red"><strong>[1]</strong></sup>. Both private and public keys are supported as are [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) (base64) encoded keys and binary encoded keys.

**<span style="color: green">PKCS8</span>** keys are the only keys that support all public key algorithms. Both private and public keys are supported <sup style="color: red"><strong>[2]</strong></sup> as are PEM encoded keys and binary encoded keys. <span style="color: green">_This is the default format that phpseclib uses_</span>.

**PuTTY** keys support DSA but only keys with an N (length of group order Q) of 160 are supported because that's all SSH2 supports. Similarily, PuTTY keys do not support the full gamut of curves that phpseclib supports (most notably, secp256k1, the Bitcoin curve). Both private and public keys are supported <sup style="color: red"><strong>[3]</strong></sup> as are v2 and, as of phpseclib v3.0.14, v3 formatted keys (with v2 being the default).

**OpenSSH** keys have the same limitations as PuTTY keys. Both public and private keys are supported <sup style="color: red"><strong>[4]</strong></sup>. Notably, from OpenSSH 6.5 to 9.6 (2014-2023), this was the only format that OpenSSH supported for Ed25519 <sup style="color: red"><strong>[5]</strong></sup>. Encrypted keys are supported as of phpseclib v3.0.15 <sup style="color: red"><strong>[6]</strong></sup>.

**JWK** keys are supported as of phpseclib v3.0.15. DSA and encrypted private keys are not supported.

**XML** keys only support private keys for RSA. Public keys are supported for all other algorithms, including RSA, but not Ed25519 / Curve25519 <sup style="color: red"><strong>[7]</strong></sup>.

<div style="font-size: 11px">

<sup style="color: red"><strong>[1]</strong></sup> [PKCS1](https://en.wikipedia.org/wiki/PKCS_1) only discusses the storage of RSA keys but there's not a formal specification I'm aware of for the `-----BEGIN DSA PRIVATE KEY-----` format. For Elliptic Curves there's [RFC5915](https://tools.ietf.org/html/rfc5915), which is implemented in Go as [ECPrivateKey](https://pkg.go.dev/crypto/x509@go1.21.5#ParseECPrivateKey), but I believe it's better to minimize the number of distinct format names then it is to maximize it. In all cases, the "[pre-encapsulation boundaries](https://tools.ietf.org/html/rfc1421#section-4.4)" for public and private keys supported by the format phpseclib calls PKCS1 is (by convention) `-----BEGIN [algorithm] PRIVATE KEY-----` and `-----BEGIN [algorithm] PUBLIC KEY-----`, so it make sense to lump them all together.

<sup style="color: red"><strong>[2]</strong></sup> [PKCS8](https://en.wikipedia.org/wiki/PKCS_8) only discusses the storage of private keys but the concept can be extended more generally to public keys as well. The public key format conforms to `SubjectPublicKeyInfo` as defined in [RFC5280](https://tools.ietf.org/html/rfc5280#page-117) and as used in JavaScript's [SubtleCrypto API](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/importKey#SubjectPublicKeyInfo), Java's [X509EncodedKeySpec](https://docs.oracle.com/javase/7/docs/api/java/security/spec/X509EncodedKeySpec.html), Go's [PKIX](https://pkg.go.dev/crypto/x509@go1.21.5#ParsePKIXPublicKey), etc. In my opinion SubjectPublicKeyInfo, X509EncodedKeySpec and PKIX are pretty terrible names and since there's not a naming conflict with PKCS8 for public keys I've opted to use that name. They're spiritually very similar as is.

In PKCS8 the pre-encapsulation boundary for a private key is (by convention) `-----BEGIN PRIVATE KEY-----`. If PKCS8 did discuss how to encode public keys it stands to reason that the convention would be to use `-----BEGIN PUBLIC KEY-----` for the format that it described.

<sup style="color: red"><strong>[3]</strong></sup> The public key format conforms to [RFC4716](https://tools.ietf.org/html/rfc4716). phpseclib calls it PuTTY because puttygen is the only package I've seen that supports this format and because PuTTY doesn't otherwise have a public key format.

<sup style="color: red"><strong>[4]</strong></sup> Private keys conform to the format described in [PROTOCOL.key](https://cvsweb.openbsd.org/cgi-bin/cvsweb/~checkout~/src/usr.bin/ssh/PROTOCOL.key?rev=1.1) and were made the default format for OpenSSH in [the 7.8 release (2018)](https://www.openssh.com/txt/release-7.8). Public keys are of the same format utilized by `~/.ssh/authorized_keys` and conform to [RFC4253](https://tools.ietf.org/html/rfc4253#page-15) / [RFC5656](https://tools.ietf.org/html/rfc5656#section-3.1).

<sup style="color: red"><strong>[5]</strong></sup> Quoting the [OpenSSH 6.5/6.5p1 (2014-01-30) changelog](https://www.openssh.com/txt/release-6.5), "_this format is used unconditionally for Ed25519 keys_". Quoting the [OpenSSH 9.6/9.6p1 (2023-12-18) changelog](https://www.openssh.com/txt/release-9.6), "_add support for reading ED25519 private keys in PEM PKCS8 format. Previously only the OpenSSH private key format was supported._".

<sup style="color: red"><strong>[6]</strong></sup> Under the hood these encrypted keys use a [modified bcrypt implementation](bcrypt.md), which slows things down considerably. Furthermore, due to these modifications, neither [`crypt()`](https://www.php.net/manual/en/function.crypt.php) or [`password_hash()`](https://www.php.net/manual/en/function.password-hash.php) can be used to speed things up.

<sup style="color: red"><strong>[7]</strong></sup> RSA Private Keys conform to the format described in the [XML Key Management Specification (XKMS)](https://en.wikipedia.org/wiki/XKMS). Public keys (for all algorithms, save for Ed25519 / Curve25519) conform to the format described in the [XML Signature](https://en.wikipedia.org/wiki/XML_Signature) standard.
</div>

## Saving Keys

Keys can be saved ([serialized](https://en.wikipedia.org/wiki/Serialization)) by casting the key objects to strings. eg. `file_put_contents('key.pem', $key)` or `echo $key`. When this approach is used the format utilized is PKCS8.

To use a custom format one must call `$key->toString('PKCS1')` (changing that up as appropriate for the specific format you desire).

Private keys can be encrypted (in formats that support encryption) by doing `$key->withPassword('demo')`. Calling `$key->withPassword()` on a public key will throw an exception since that method is not implemented for public keys (since no public key format supports encryption).

Note that loading a key and then saving it may not yield the exact same sequence of bytes. In the case of encrypted keys, even though the password is preserved, the encryption parameters are not. So a PKCS8 key using pbeWithSHAAnd3-KeyTripleDES-CBC will be saved using id-PBES2 / aes128-CBC-PAD, unless you changed it. And even if you did load a id-PBES2 / aes128-CBC-PAD key the salt is randomly generated. OpenSSH keys have two random "checkint"'s that are supposed to match, ASN.1 encoded keys may have optional defaults that can either be explicitly or implicitly encoded, etc.

If you want to save an encrypted private key _without_ encryption then you'll need to do `$key->withPassword()`.

### PKCS1

By default, PKCS1 private keys uses AES-128-CBC as the encryption algorithm / mode. This can be configured thusly:

```php
use phpseclib3\Crypt\Common\Formats\Keys\PKCS1;

PKCS1::setEncryptionAlgorithm('DES-EDE3-CBC');

echo $key->withPassword('demo')->toString('PKCS1');
```

Supported encryption algorithms for PKCS1 are as follows:

- AES-128
- AES-192
- AES-256
- DES-EDE3
- DES

What you pass to `setEncryptionAlgorithm` needs to include the algorithm and the mode, separated by a dash. Supported modes for PKCS1 are as follows:

- CBC
- ECB
- CFB
- OFB
- CTR

If you want to set the mode on a per-key basis instead of globally you can do this:

```php
echo $key
    ->withPassword('demo')
    ->toString('PKCS1', ['encryptionAlgorithm' => 'DES-EDE3-CBC']);
```

### PKCS8

By default, PKCS8 private keys uses id-PBES2 as the encryption algorithm, aes128-CBC-PAD as the encryption scheme, id-hmacWithSHA256 as the pseudo-random function and 2048 as the iteration count.

Valid options for `PKCS8::setEncryptionAlgorithm` are as follows:

[PBES1 encryption scheme](https://tools.ietf.org/html/rfc2898#appendix-A.3):
- pbeWithMD2AndDES-CBC
- pbeWithMD2AndRC2-CBC
- pbeWithMD5AndDES-CBC
- pbeWithMD5AndRC2-CBC
- pbeWithSHA1AndDES-CBC
- pbeWithSHA1AndRC2-CBC
- pbeWithSHAAnd3-KeyTripleDES-CBC
- pbeWithSHAAnd2-KeyTripleDES-CBC
- pbeWithSHAAnd128BitRC2-CBC
- pbeWithSHAAnd40BitRC2-CBC
- pbeWithSHAAnd128BitRC4
- pbeWithSHAAnd40BitRC4

[PBES2 encryption scheme](https://tools.ietf.org/html/rfc2898#appendix-A.4):
- id-PBES2

PBES1 "combines the PBKDF1 function" (or a key deriviation function defined in [PKCS#12](https://tools.ietf.org/html/rfc7292#appendix-B)) and "is recommended only for compatibility with existing applications".

PBES2 "combines a password-based key derivation function, which shall be [PBKDF2](https://en.wikipedia.org/wiki/PBKDF2)" and "is recommended for new applications".

`PKCS8::setIterationCount` sets the iteration count.

The rest of the methods discussed are only used for PBES2.

Valid options for `PKCS8::setEncryptionScheme` are as follows:

- desCBC
- des-EDE3-CBC
- rc2CBC
- aes128-CBC-PAD
- aes192-CBC-PAD
- aes256-CBC-PAD

Valid options for `PKCS8::setPRF` are as follows:

- id-hmacWithSHA1
- id-hmacWithSHA224
- id-hmacWithSHA256
- id-hmacWithSHA384
- id-hmacWithSHA512
- id-hmacWithSHA512-224
- id-hmacWithSHA512-256

Putting everthing together here's a complete example of how you'd change these parameters up:

```php
use phpseclib3\Crypt\Common\Formats\Keys\PKCS8;

PKCS8::setEncryptionAlgorithm('id-PBES2');
PKCS8::setEncryptionScheme('aes256-CBC-PAD');
PKCS8::setPRF('id-hmacWithSHA512-256');
PKCS8::setIterationCount(4096);

echo $key->withPassword('demo');
```

If you want to set the mode on a per-key basis instead of globally you can do this:

```php
echo $key
    ->withPassword('demo')
    ->toString('PKCS8', [
        'encryptionAlgorithm' => 'id-PBES2',
        'encryptionScheme' => 'aes256-CBC-PAD',
        'PRF' => 'id-hmacWithSHA512-256',
        'iterationCount' => 4096
    ]);
```

Uniquely, the PKCS8 class has a `extractEncryptionAlgorithm` that'll let you see the encryption algorithm the key is using. This method does not exist for other key formats because, with other key formats, the format is in plain ASCII. In the case of PKCS8 it's embedded into the keys ASN.1 structure with OIDs. Here's an example of that:

```php
use phpseclib3\Crypt\Common\Formats\Keys\PKCS8;

print_r(PKCS8::extractEncryptionAlgorithm($key));
```

### PuTTY

PuTTY private keys only support one encryption algorithm (aes256-cbc) so there are no methods to change it.

Both PuTTY public and private keys support comments, which can be changed thusly:

```php
use phpseclib3\Crypt\Common\Formats\Keys\PuTTY;

PuTTY::setComment('whatever');
echo $key->toString('PuTTY');
```
...or...
```php
echo $key->toString('PuTTY', ['comment' => 'whatever']);
```

### OpenSSH

Although OpenSSH private keys support encryption phpseclib does not support them, for various technical reasons.

Both OpenSSH public and private keys support comments, which can be changed thusly:

```php
use phpseclib3\Crypt\Common\Formats\Keys\OpenSSH;

OpenSSH::setComment('whatever');
echo $key->toString('OpenSSH');
```
...or...
```php
echo $key->toString('OpenSSH', ['comment' => 'whatever']);
```

## Custom Key Formats

Custom key formats can be added by doing `\phpseclib3\Crypt\RSA::addFileFormat()`. So let's say you wanted to be able to load keys created by [PEAR's Crypt_RSA](https://pear.php.net/package/Crypt_RSA). The following would be sufficient:

```php
use phpseclib3\Crypt\PublicKeyLoader;
use phpseclib3\Crypt\RSA;
use phpseclib3\Math\BigInteger;

class PEAR
{
    public static function load($key, $password = '')
    {
        if (!is_string($key)) {
            throw new \UnexpectedValueException('Key should be a string - not a ' . gettype($key));
        }

        // should use ParagonIE\ConstantTime\Base64 but for brevity we won't
        $key = base64_decode($key);

        if (!is_string($key)) {
            throw new \UnexpectedValueException('Base64 decoding produced an error');
        }

        $key = unserialize($key);

        if (!is_array($key)) {
            throw new \UnexpectedValueException('Unserializing produced an error');
        }

        return [
            'modulus' => new BigInteger($key[0], 256),
            'publicExponent' => new BigInteger($key[1], 256),
            'isPublicKey' => $key[2] == 'public'
        ];
    }
}

$key = 'YTozOntpOjA7czoxMjg6Ij3ZbmQEZuLuvDVxVo3FaoNI0gaOiDKguZqWKX+DTjW19umiCMf43onkCdsGp/wZ5Qyn+xTgBZ/1Dt4FM9z40HFq05fJCKtZFDxtBa5be5khtqqzvS6riXjCF3g7DDlvr1MCDytHxk42KnjWM/C32Qk5xl1PTUqkB8WMVICqLryhIjtpOjE7czoxMjg6IoXrTkH1481uyr/CNVnlLn0HwxE4HIIBJHzPiitvvVynqPwq8bUV3hfHrJ1W+4khgGbVhJjqtGtZGmsRnlLCK349aEb05occqCpNR//IcBvfLPZ0Db6jJ2+JdOTcVhjIw33/wPhPReAuIxmMBmItUot+68vxifq2fj46NdXQJ5ZNIjtpOjI7czo3OiJwcml2YXRlIjt9';

RSA::addFileFormat(PEAR::class);

$key = PublicKeyLoader::load($key);
```
Keys of this format can be generated with PEAR's Crypt_RSA thusly:

```php
$key_pair = new Crypt_RSA_KeyPair(1024);
echo $key_pair->getPrivateKey()->toString() . "\n\n\n";
echo $key_pair->getPublicKey()->toString() . "\n\n\n";
```
<sup>(note that public keys won't correctly generate due to [Bug # 26925](https://pear.php.net/bugs/bug.php?id=26925))</sup>

To make it so that keys can be saved using these custom format plugins you'll need to add `savePublicKey` and `savePrivateKey` methods.

The exact parameters and return values depend on the algorithm being used. Check the files in the `Crypt/*/Formats/Keys` directories for specific examples.

## Public Key Fingerprints
[Public key fingerprints](https://en.wikipedia.org/wiki/Public_key_fingerprint) can be obtained by doing `$publicKey->getFingerprint($algorithm)`. The only supported values for `$algorithm` are `'sha256'` and `'md5'`. What phpseclib returns is identical to what you'd get by running `ssh-keygen -lf key.pub` on the command line.

Due to the SSH2 tie-in a public key algorithm with parameters not supported by SSH2 will not generate a fingerprint.

See [RFC4716](https://tools.ietf.org/html/rfc4716#section-4) for more information.

## Comments
Only PuTTY and OpenSSH formatted keys support comments. See [Saving Keys](publickeys.md#saving-keys) for information on saving keys.

To read keys doing `$key->getComment()` is sufficient. If the loaded key is in a format that does not support keys then `NULL` will be returned.
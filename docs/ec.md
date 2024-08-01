---
id: ec
title: Elliptic Curves
---

Loading and saving keys is discussed in [Public Keys: Overview](publickeys.md).

## Supported Formats

- **PKCS1** <sup style="color: red"><strong>[1]</strong></sup>
  - Keys start with `-----BEGIN EC PRIVATE KEY-----` or `-----BEGIN EC PARAMETERS-----`
- **PKCS8** <sup style="color: red"><strong>[1]</strong></sup>
  - Keys start with `-----BEGIN PRIVATE KEY-----` or `-----BEGIN ENCRYPTED PRIVATE KEY-----` or `-----BEGIN PUBLIC KEY-----`
- **PuTTY** <sup style="color: red"><strong>[2]</strong></sup>
  - Public keys start off with `---- BEGIN SSH2 PUBLIC KEY ----`
- **OpenSSH** <sup style="color: red"><strong>[2]</strong></sup><sup style="color: red"><strong>[3]</strong></sup>
  - Private keys start with `-----BEGIN OPENSSH PRIVATE KEY-----`
- **JWK** <sup style="color: red"><strong>[4]</strong></sup>
- **XML** <sup style="color: red"><strong>[5]</strong></sup>

A more in-depth discussion of the common formats can be found in [Common Key Formats](publickeys.md#common-key-formats). See [Sample EC Keys](ec-keys.md) for actual samples.

<div style="font-size: 11px">

<sup style="color: red"><strong>[1]</strong></sup> Supports both named and specified curves.

<sup style="color: red"><strong>[2]</strong></sup> The only curves supported by these formats are as follows:

- nistp256 (alias: secp256r1, prime256v1)

- nistp384 (alias: secp384r1)

- nistp521 (alias: secp521r1)

- Ed25519

The first three are the required curves of [RFC5656](https://tools.ietf.org/html/rfc5656#section-10.1) and the last one is specified in [draft-ietf-curdle-ssh-ed25519-02](https://tools.ietf.org/html/draft-ietf-curdle-ssh-ed25519-02).

Specified curves are not supported - only named curves.

<sup style="color: red"><strong>[3]</strong></sup> Quoting the [OpenSSH 6.5/6.5p1 (2014-01-30) changelog](https://www.openssh.com/txt/release-6.5), "_this format is used unconditionally for Ed25519 keys_". [No newer version of OpenSSH](https://www.openssh.com/releasenotes.html), as of this writing, seems to change this.

<sup style="color: red"><strong>[4]</strong></sup> The only curves supported by this format are as follows:

- nistp256 (alias: secp256r1, prime256v1)

- nistp384 (alias: secp384r1)

- nistp521 (alias: secp521r1)

- secp256k1

- Ed25519 (as an OKP key)

- Ed448 (as an OKP key)

<sup style="color: red"><strong>[5]</strong></sup> Supports [regular ECKeyValues](https://www.w3.org/TR/xmldsig-core/#sec-ECKeyValue) and [RFC4050 formatted ECKeyValues](https://www.w3.org/TR/xmldsig-core/#sec-RFC4050Compat). Supports named curves and has expiremental specified prime curve support. Specified binary curves are not supported at all.
</div>

### Parameters

Parameters consist of either the curve name or the curve parameters. They correspond to [ECParameters in RFC5480](https://tools.ietf.org/html/rfc5480#section-2.1.1).

PKCS1 is the only format that supports Parameters. They can be extracted from a private or public key by doing `$key->getParameters()`.

Whereas public and private keys are instances of `\phpseclib3\Crypt\Common\PublicKey` and `\phpseclib3\Crypt\Common\PrivateKey`, respectively, parameters are an instance of `phpseclib3\Crypt\EC\Parameters` (which is what `$key->getParameters()` returns).

## Supported Curves

The following curves are supported:

- [Curve448](https://tools.ietf.org/html/rfc7748)
- [Curve25519](https://tools.ietf.org/html/rfc7748)
- [Ed448](https://tools.ietf.org/html/rfc8032)
- [Ed25519](https://tools.ietf.org/html/rfc8032)
- [Brainpool curves](https://tools.ietf.org/html/rfc5639)
- [SECG prime field curves](http://www.secg.org/SEC2-Ver-1.0.pdf), which includes, most notably:
  - NIST P-256 (the [most popular curve](https://malware.news/t/everyone-loves-curves-but-which-elliptic-curve-is-the-most-popular/17657))
  - secp256k1 (used by BitCoin)
- [SECG binary field curves](http://www.secg.org/SEC2-Ver-1.0.pdf)
  - these curves are rarely used and as such have not been as highly optimized as the prime field curves

<span style="font-size: 11px">(_SECG stands for Standards for Efficient Cryptography_)</span>

The full list of curves can be found at https://github.com/phpseclib/phpseclib/tree/3.0/phpseclib/Crypt/EC/Curves

## Named vs Specified Curves

To understand the difference between named and specified curves it first helps to understand that there are multiple "classes" of curves.

||Montgomery|Twisted Edwards|Weierstrass|
|---|---|---|---|
|**Individual Curves**|[Curve25519](https://en.wikipedia.org/wiki/Curve25519), [Curve448](https://en.wikipedia.org/wiki/Curve448)|[Ed25519, Ed448](https://en.wikipedia.org/wiki/EdDSA)|_[everything](https://www.secg.org/sec2-v2.pdf) [else](https://tools.ietf.org/html/rfc5639)_|

All curves correspond to an equation. For Weierstrass Curves that equation has the following form:

<span class="equation">
**_Y <sup>2</sup> + a<sub>1</sub> X Y + a<sub>3</sub> Y = X <sup>3</sup> + a<sub>2</sub> X <sup>2</sup> + a<sub>4</sub> X + a<sub>6</sub>_**
</span>

For prime field Weierstrass Curves **_a<sub>1</sub>_**, **_a<sub>3</sub>_** and **_a<sub>2</sub>_** are always 0. For binary field Weierstrass Curves **_a<sub>3</sub>_** and **_a<sub>2</sub>_** are always 0 and **_a<sub>1</sub>_** is always 1.

The remaining variables can be specified either explicitly, using a **Specified Curve** or implictly by invoking a **Named Curve**. phpseclib supports both.

Montgomery and Twisted Edwards Curves do not have user-definable coefficients and therefore the concept of a "specified curve" is inapplicable.

Named curves are the default. Specified curves can be enabled by doing `PKCS1::useSpecifiedCurve()` or `PKCS8::useSpecifiedCurve()`. Named curves can be specified by doing `PKCS1::useNamedCurve()` or `PKCS8::useSpecifiedCurve()`.

Specified curves can also be enabled by doing `$key->toString('PKCS8', ['namedCurve' => false])` or `$key->toString('PKCS1', ['namedCurve' => false])`.

## Creating Keys

Keys can be created thusly:

```php
use phpseclib3\Crypt\EC;

$private = EC::createKey('Ed25519');
$public = $private->getPublicKey();
```

## Creating / Verifying Signatures

Signatures can be created / verified thusly:

```php
//$private = $private->withSignatureFormat('ASN1');
$signature = $private->sign($message);
echo $private->getPublicKey()->verify($message, $signature) ?
    'valid signature' :
    'invalid signature';
```
The signatures generated are _not_ deterministic, as discussed in [RFC6979](https://tools.ietf.org/html/rfc6979). Such determinism is chiefly of benefit when a [CSPRNG](https://en.wikipedia.org/wiki/Cryptographically_secure_pseudorandom_number_generator) is _not_ available and with PHP there is one that's available.

Signatures have two components - **r** and **s**. How these two components are combined to a single string depends on the signature format being employed.

### ASN1

<span style="color: green">This is the default format</span>. ASN1-formatted signatures employee the format discussed in [RFC3279](https://tools.ietf.org/html/rfc3279#section-2.2.3). This is the format used by X.509 certificates.

### SSH2

SSH2-formatted signatures employee the format discussed in [RFC4253](https://tools.ietf.org/html/rfc4253#page-15) or (for Ed25519) [draft-ietf-curdle-ssh-ed25519-02](https://tools.ietf.org/html/draft-ietf-curdle-ssh-ed25519-02#section-6).

### IEEE

This format used with [JSON Web Signatures](https://en.wikipedia.org/wiki/JSON_Web_Signature) (JWS), JavaScript's [Web Cryptography API](https://en.wikipedia.org/wiki/Web_Cryptography_API), et al. [SubtleCrypto: sign() method - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/sign#ecdsa) explains why IEEE is used as the name:

> This encoding was also proposed by the [IEEE 1363-2000](https://standards.ieee.org/ieee/1363/2049/) standard, and is sometimes referred to as the IEEE P1363 format. It differs from the [X.509](https://www.itu.int/rec/T-REC-X.509) signature structure, which is the default format produced by some tools and libraries such as [OpenSSL](https://www.openssl.org/).

In the case of JSON Web Signatures the signature should also be passed through `\phpseclib3\Common\Functions\Strings::base64url_encode()`.

<!-- see also https://datatracker.ietf.org/doc/html/rfc7515#page-45 -->

### Raw

Returns an array with **r** and **s** as keys.

## Key Attributes

`$key->getCurve()` returns either a string with the name of the curve or an array containing the keys specified parameters.

`$key->getLength()` returns the length of the modulo, in bits.

All the `with` methods have corresponding `get` methods as follows:

<!--
getCurve, getLength, withContext (ed25519, ed448), 
-->

| Setter | Getter |
|---|---|
| `withContext` | `getContext` |
| `withHash` | `getHash` |
| `withSignatureFormat` | `getSignatureFormat` |

While `withHash` accepts strings, `getHash` returns a Hash object (that can be cast to a string via [__toString](https://www.php.net/manual/en/language.oop5.magic.php#object.tostring)).

`withContext` is only usable with Ed25519 and Ed448. `withHash` does not work for Curve25519, Curve448, Ed25519 or Ed448.
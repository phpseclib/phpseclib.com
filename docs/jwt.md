---
id: jwt
title: Example: JWT
---

A [JSON Web Token](https://en.wikipedia.org/wiki/JSON_Web_Token) consists of three parts - a header a payload and a signature - each encoded separately using [Base64url](https://en.wikipedia.org/wiki/Base64#URL_applications) (`\phpseclib3\Common\Functions\Strings::base64url_encode()`) and concatenated together using periods. eg.

```
const token = base64urlEncoding(header) + '.' +
              base64urlEncoding(payload) + '.' +
              base64urlEncoding(signature)
```
The signature is created from the concatenation of the Base64url encoded header and payload. The algorithm used for the signature is specified in the header.

A list of all the algorithms and how to implement them with phpseclib is discussed below. In these examples `$header` and `$payload` are assumed to already be Base64url encoded and `Strings` is assumed to be namespaced to `\phpseclib3\Common\Functions\Strings`.

## ES256

```php
assert($private instanceof \phpseclib3\Crypt\EC);
assert($private->getCurve() == 'secp256r1');

$private = $private->withHash('sha256')->withSignatureFormat('IEEE');
$sig = $private->sign("$header.$payload");
$sig = Strings::base64url_encode($sig);
```

## ES384

```php
assert($private instanceof \phpseclib3\Crypt\EC);
assert($private->getCurve() == 'secp384r1');

$private = $private->withHash('sha384')->withSignatureFormat('IEEE');
$sig = $private->sign("$header.$payload");
$sig = Strings::base64url_encode($sig);
```

## ES512

```php
assert($private instanceof \phpseclib3\Crypt\EC);
assert($private->getCurve() == 'secp521r1');

$private = $private->withHash('sha512')->withSignatureFormat('IEEE');
$sig = $private->sign("$header.$payload");
$sig = Strings::base64url_encode($sig);
```

## RS256

```php
assert($private instanceof \phpseclib3\Crypt\RSA);

$private = $private->withHash('sha256')->withPadding(RSA::SIGNATURE_PKCS1);
$sig = $private->sign("$header.$payload");
$sig = Strings::base64url_encode($sig);
```

## RS384

```php
assert($private instanceof \phpseclib3\Crypt\RSA);

$private = $private->withHash('sha384')->withPadding(RSA::SIGNATURE_PKCS1);
$sig = $private->sign("$header.$payload");
$sig = Strings::base64url_encode($sig);
```

## RS512

```php
assert($private instanceof \phpseclib3\Crypt\RSA);

$private = $private->withHash('sha512')->withPadding(RSA::SIGNATURE_PKCS1);
$sig = $private->sign("$header.$payload");
$sig = Strings::base64url_encode($sig);
```

## PS256

```php
assert($private instanceof \phpseclib3\Crypt\RSA);

$private = $private->withHash('sha256')->withPadding(RSA::SIGNATURE_PSS);
$sig = $private->sign("$header.$payload");
$sig = Strings::base64url_encode($sig);
```

## PS384

```php
assert($private instanceof \phpseclib3\Crypt\RSA);

$private = $private->withHash('sha384')->withPadding(RSA::SIGNATURE_PSS);
$sig = $private->sign("$header.$payload");
$sig = Strings::base64url_encode($sig);
```

## PS512

```php
assert($private instanceof \phpseclib3\Crypt\RSA);

$private = $private->withHash('sha512')->withPadding(RSA::SIGNATURE_PSS);
$sig = $private->sign("$header.$payload");
$sig = Strings::base64url_encode($sig);
```

## EdDSA

```php
assert($private instanceof \phpseclib3\Crypt\EC);
assert($private->getCurve() == 'Ed25519');

$sig = $private->sign("$header.$payload");
$sig = Strings::base64url_encode($sig);
```
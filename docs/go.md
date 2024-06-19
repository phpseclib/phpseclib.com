---
id: go
title: Go
---

<!-- go run test.go -->

All the Go code samples can be previewed on https://go.dev/play/

## RSA Decryption

Encryption with PHP:

```php
use phpseclib3\Crypt\PublicKeyLoader;

$key = PublicKeyLoader::load('-----BEGIN RSA PUBLIC KEY-----
MEgCQQCo9+BpMRYQ/dL3DS2CyJxRF+j6ctbT3/Qp84+KeFhnii7NT7fELilKUSnx
S30WAvQCCo2yU1orfgqr41mM70MBAgMBAAE=
-----END RSA PUBLIC KEY-----');
$key = $key->withPadding(RSA::ENCRYPTION_PKCS1);
echo base64_encode($key->encrypt('test'));
```

Decryption with Go:

```go
package main

import (
    "encoding/pem"
    "encoding/base64"
    "crypto/x509"
    "crypto/rsa"
    "fmt"
)

func main() {
    key := `-----BEGIN RSA PRIVATE KEY-----
MIIBOgIBAAJBAKj34GkxFhD90vcNLYLInFEX6Ppy1tPf9Cnzj4p4WGeKLs1Pt8Qu
KUpRKfFLfRYC9AIKjbJTWit+CqvjWYzvQwECAwEAAQJAIJLixBy2qpFoS4DSmoEm
o3qGy0t6z09AIJtH+5OeRV1be+N4cDYJKffGzDa88vQENZiRm0GRq6a+HPGQMd2k
TQIhAKMSvzIBnni7ot/OSie2TmJLY4SwTQAevXysE2RbFDYdAiEBCUEaRQnMnbp7
9mxDXDf6AU0cN/RPBjb9qSHDcWZHGzUCIG2Es59z8ugGrDY+pxLQnwfotadxd+Uy
v/Ow5T0q5gIJAiEAyS4RaI9YG8EWx/2w0T67ZUVAw8eOMB6BIUg0Xcu+3okCIBOs
/5OiPgoTdSy7bcF9IGpSE8ZgGKzgYQVZeN97YE00
-----END RSA PRIVATE KEY-----`
    ciphertext := "L812/9Y8TSpwErlLR6Bz4J3uR/T5YaqtTtB5jxtD1qazGPI5t15V9drWi58colGOZFeCnGKpCrtQWKk4HWRocQ==";

    keyBytes := []byte(key)
    // the second return value are the extra bytes after the PEM
    // has been decoded
    decodedKey, _ := pem.Decode(keyBytes)
    privateKey, err := x509.ParsePKCS1PrivateKey(decodedKey.Bytes)
    if err != nil {
        panic(err)
    }

    // other encoding schemes are:
    // RawStdEncoding, RawURLEncoding, URLEncoding
    ciphertextBytes, err := base64.StdEncoding.DecodeString(ciphertext)
    if err != nil {
        panic(err)
    }

    // the first parameter is a customizable random number generator
    plaintextBytes, err := privateKey.Decrypt(nil, ciphertextBytes, &rsa.PKCS1v15DecryptOptions{})
    if err != nil {
        panic(err)
    }

    plaintext := string(plaintextBytes[:])

    fmt.Println(plaintext)
}
```
(alternatively, one could call [`rsa.DecryptPKCS1v15`](https://pkg.go.dev/crypto/rsa@go1.21.5#DecryptPKCS1v15) instead of [`privateKey.Decrypt`](https://pkg.go.dev/crypto/rsa@go1.21.5#PrivateKey.Decrypt))

The above code only works with PKCS1 keys. If you want to load a PKCS8 key you'll need to make the following changes (using the [phpBB MOD Text Template](phpbb.md#actions)):

```go
#
#-----[ FIND ]------------------------------------------
#
    key := `-----BEGIN RSA PRIVATE KEY-----
MIIBOgIBAAJBAKj34GkxFhD90vcNLYLInFEX6Ppy1tPf9Cnzj4p4WGeKLs1Pt8Qu
KUpRKfFLfRYC9AIKjbJTWit+CqvjWYzvQwECAwEAAQJAIJLixBy2qpFoS4DSmoEm
o3qGy0t6z09AIJtH+5OeRV1be+N4cDYJKffGzDa88vQENZiRm0GRq6a+HPGQMd2k
TQIhAKMSvzIBnni7ot/OSie2TmJLY4SwTQAevXysE2RbFDYdAiEBCUEaRQnMnbp7
9mxDXDf6AU0cN/RPBjb9qSHDcWZHGzUCIG2Es59z8ugGrDY+pxLQnwfotadxd+Uy
v/Ow5T0q5gIJAiEAyS4RaI9YG8EWx/2w0T67ZUVAw8eOMB6BIUg0Xcu+3okCIBOs
/5OiPgoTdSy7bcF9IGpSE8ZgGKzgYQVZeN97YE00
-----END RSA PRIVATE KEY-----`
#
#-----[ REPLACE WITH ]----------------------------------
#
    key := `-----BEGIN PRIVATE KEY-----
MIIBVAIBADANBgkqhkiG9w0BAQEFAASCAT4wggE6AgEAAkEAqPfgaTEWEP3S9w0t
gsicURfo+nLW09/0KfOPinhYZ4ouzU+3xC4pSlEp8Ut9FgL0AgqNslNaK34Kq+NZ
jO9DAQIDAQABAkAgkuLEHLaqkWhLgNKagSajeobLS3rPT0Agm0f7k55FXVt743hw
Ngkp98bMNrzy9AQ1mJGbQZGrpr4c8ZAx3aRNAiEAoxK/MgGeeLui385KJ7ZOYktj
hLBNAB69fKwTZFsUNh0CIQEJQRpFCcydunv2bENcN/oBTRw39E8GNv2pIcNxZkcb
NQIgbYSzn3Py6AasNj6nEtCfB+i1p3F35TK/87DlPSrmAgkCIQDJLhFoj1gbwRbH
/bDRPrtlRUDDx44wHoEhSDRdy77eiQIgE6z/k6I+ChN1LLttwX0galITxmAYrOBh
BVl433tgTTQ=
-----END PRIVATE KEY-----`
#
#-----[ FIND ]------------------------------------------
#
    privateKey, err := x509.ParsePKCS1PrivateKey(decodedKey.Bytes)
    if err != nil {
        panic(err)
    }
#
#-----[ REPLACE WITH ]----------------------------------
#
    privateAny, err := x509.ParsePKCS8PrivateKey(decodedKey.Bytes)
    if err != nil {
        panic(err)
    }
    privateKey := privateAny.(*rsa.PrivateKey)
```

OAEP encryption is supported as well. Here's the phpseclib code to encrypt something with OAEP:

```php
use phpseclib3\Crypt\PublicKeyLoader;

$key = PublicKeyLoader::load('-----BEGIN RSA PUBLIC KEY-----
MEgCQQCo9+BpMRYQ/dL3DS2CyJxRF+j6ctbT3/Qp84+KeFhnii7NT7fELilKUSnx
S30WAvQCCo2yU1orfgqr41mM70MBAgMBAAE=
-----END RSA PUBLIC KEY-----');
$key = $key
    //->withPadding(RSA::ENCRYPTION_OAEP)
    ->withHash('md5')
    ->withMGFHash('sha1');
echo base64_encode($key->encrypt('test'));
```
<sup>_(md5 is being used because the key is a 512-bit key from [Sample RSA Keys](/docs/rsa-keys); 512-bits is used for brevity but because it's 512-bits sha256 can't be used per the max size formulas discussed at [RSA::ENCRYPTION_OAEP](/docs/rsa#rsaencryption_oaep); sha1 would work but for the purposes of this demonstration it's useful to have them be different)_</sup>

To decrypt with Go one need only make the following changes to the original script:

```go
#
#-----[ FIND ]------------------------------------------
#
    "crypto/rsa"
#
#-----[ AFTER, ADD ]------------------------------------
#
    "crypto"
#
#-----[ FIND ]------------------------------------------
#
    ciphertext := "L812/9Y8TSpwErlLR6Bz4J3uR/T5YaqtTtB5jxtD1qazGPI5t15V9drWi58colGOZFeCnGKpCrtQWKk4HWRocQ==";
#
#-----[ REPLACE WITH ]----------------------------------
#
    ciphertext := "h3j3zLT2jXCaZuwF7cgUE/Zmc/5IsIfKbaTiBhpCJo86AiyuoA3Yvni+Lrm5wu2OGv2h5R7Zu3voFcHugiystw=="
#
#-----[ FIND ]------------------------------------------
#
    plaintextBytes, err := privateKey.Decrypt(nil, ciphertextBytes, &rsa.PKCS1v15DecryptOptions{})
#
#-----[ REPLACE WITH ]----------------------------------
#
    plaintextBytes, err := privateKey.Decrypt(nil, ciphertextBytes, &rsa.OAEPOptions{
        Hash: crypto.MD5,
        MGFHash: crypto.SHA1,
        //Label: []byte(""),
    })
```
([`rsa.DecryptOAEP`](https://pkg.go.dev/crypto/rsa@go1.21.5#DecryptOAEP) only works when `Hash` and `MGFHash` are the same)

## RSA Signature Verification

Signature creation with PHP:

```php
use phpseclib3\Crypt\PublicKeyLoader;

$key = PublicKeyLoader::load('-----BEGIN RSA PRIVATE KEY-----
MIIBOgIBAAJBAKj34GkxFhD90vcNLYLInFEX6Ppy1tPf9Cnzj4p4WGeKLs1Pt8Qu
KUpRKfFLfRYC9AIKjbJTWit+CqvjWYzvQwECAwEAAQJAIJLixBy2qpFoS4DSmoEm
o3qGy0t6z09AIJtH+5OeRV1be+N4cDYJKffGzDa88vQENZiRm0GRq6a+HPGQMd2k
TQIhAKMSvzIBnni7ot/OSie2TmJLY4SwTQAevXysE2RbFDYdAiEBCUEaRQnMnbp7
9mxDXDf6AU0cN/RPBjb9qSHDcWZHGzUCIG2Es59z8ugGrDY+pxLQnwfotadxd+Uy
v/Ow5T0q5gIJAiEAyS4RaI9YG8EWx/2w0T67ZUVAw8eOMB6BIUg0Xcu+3okCIBOs
/5OiPgoTdSy7bcF9IGpSE8ZgGKzgYQVZeN97YE00
-----END RSA PRIVATE KEY-----')
    //->withHash('sha256')
    ->withPadding(RSA::SIGNATURE_PKCS1);

echo base64_encode($key->sign('zzz'));
```

Signature verification with Go:

```go
package main

import (
    "encoding/pem"
    "encoding/base64"
    "crypto/sha256"
    "crypto/x509"
    "crypto/rsa"
    "crypto"
    "fmt"
)

func main() {
    key := `-----BEGIN RSA PUBLIC KEY-----
MEgCQQCo9+BpMRYQ/dL3DS2CyJxRF+j6ctbT3/Qp84+KeFhnii7NT7fELilKUSnx
S30WAvQCCo2yU1orfgqr41mM70MBAgMBAAE=
-----END RSA PUBLIC KEY-----`
    message := "zzz"
    signature := "MUE536c4UJSAmycs7V6qFaLMATrKMQA8TYj5xX1+fwHINz3/BafgaRt0ycoD5IxTxaclLWavrGSza4xSBHraEw==";

    keyBytes := []byte(key)
    // the second return value are the extra bytes after the PEM
    // has been decded
    decodedKey, _ := pem.Decode(keyBytes)
    publicKey, err := x509.ParsePKCS1PublicKey(decodedKey.Bytes)
    if err != nil {
        panic(err)
    }

    // other encoding schemes are:
    // RawStdEncoding, RawURLEncoding, URLEncoding
    signatureBytes, err := base64.StdEncoding.DecodeString(signature)
    if err != nil {
        panic(err)
    }
    messageBytes := []byte(message)
    hashed := sha256.Sum256(messageBytes)

    err = rsa.VerifyPKCS1v15(publicKey, crypto.SHA256, hashed[:], signatureBytes)
    if err != nil {
        fmt.Println("bad")
    } else {
        fmt.Println("good")
    }
}
```
PSS signatures are supported as well. Here's the phpseclib code to sign something with PSS:

```php
use phpseclib3\Crypt\PublicKeyLoader;

$key = PublicKeyLoader::load('...');
    // the following are implied:
    //->withPadding(RSA::SIGNATURE_PSS)
    //->withHash('sha256')
    //->withMGFHash('sha256')

echo base64_encode($key->sign('zzz'));
```
<sup>_(the actual key is omitted because, for this example, a larger key than the 512-bit key we've been using, is needed)_</sup>

To perform PSS signature verification one need only make the following changes to the original script:

```go
#
#-----[ FIND ]------------------------------------------
#
    signature := "MUE536c4UJSAmycs7V6qFaLMATrKMQA8TYj5xX1+fwHINz3/BafgaRt0ycoD5IxTxaclLWavrGSza4xSBHraEw==";
#
#-----[ REPLACE WITH ]----------------------------------
#
    signature := "oa7eJv3Ocl4Uh+6M2UBalglijFtCLiYOxRSFafzRt3mp6eNnxsS5GMqvs3nXzRT2KhDlMelssjDJE2wULsnDySld64Wm7+0SYTAQNU1tFVO4JUMpROodT9We24MuLlOssgssr71scolg4NPc+ltCDGu5Y+NRHEwG0vtA7lwLM3c=";
#
#-----[ FIND ]------------------------------------------
#
    err = rsa.VerifyPKCS1v15(publicKey, crypto.SHA256, hashed[:], signatureBytes)
#
#-----[ REPLACE WITH ]----------------------------------
#
    opts := &rsa.PSSOptions{
        Hash: crypto.SHA256,
        SaltLength: rsa.PSSSaltLengthEqualsHash, // == crypto.Hash.Size(crypto.SHA256)
    }

    err = rsa.VerifyPSS(publicKey, crypto.SHA256, hashed[:], signatureBytes, opts)
```
Note that, at this time, Go does not let you set the Hash and MGFHash to different values, per https://github.com/golang/go/issues/46233

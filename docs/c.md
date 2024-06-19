---
id: c
title: C
---

All C code samples use [OpenSSL](https://en.wikipedia.org/wiki/OpenSSL).

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

Decryption with C:
<!-- gcc -x c test.c -lcrypto && ./a.out -->

```c
#include <openssl/evp.h>
#include <openssl/pem.h>
#include <string.h>

int main (void)
{
    char privateKey[] = "-----BEGIN RSA PRIVATE KEY-----\n"\
"MIIBOgIBAAJBAKj34GkxFhD90vcNLYLInFEX6Ppy1tPf9Cnzj4p4WGeKLs1Pt8Qu\n"\
"KUpRKfFLfRYC9AIKjbJTWit+CqvjWYzvQwECAwEAAQJAIJLixBy2qpFoS4DSmoEm\n"\
"o3qGy0t6z09AIJtH+5OeRV1be+N4cDYJKffGzDa88vQENZiRm0GRq6a+HPGQMd2k\n"\
"TQIhAKMSvzIBnni7ot/OSie2TmJLY4SwTQAevXysE2RbFDYdAiEBCUEaRQnMnbp7\n"\
"9mxDXDf6AU0cN/RPBjb9qSHDcWZHGzUCIG2Es59z8ugGrDY+pxLQnwfotadxd+Uy\n"\
"v/Ow5T0q5gIJAiEAyS4RaI9YG8EWx/2w0T67ZUVAw8eOMB6BIUg0Xcu+3okCIBOs\n"\
"/5OiPgoTdSy7bcF9IGpSE8ZgGKzgYQVZeN97YE00\n"\
"-----END RSA PRIVATE KEY-----\n";
    char ciphertext[] = "L812/9Y8TSpwErlLR6Bz4J3uR/T5YaqtTtB5jxtD1qazGPI5t15V9drWi58colGOZFeCnGKpCrtQWKk4HWRocQ==";

    // base64 decode ciphertext
    EVP_DecodeBlock(ciphertext, ciphertext, strlen(ciphertext));

    BIO* bo = BIO_new(BIO_s_mem());
    BIO_write(bo, privateKey, strlen(privateKey));
    EVP_PKEY* pkey = 0;
    PEM_read_bio_PrivateKey(bo, &pkey, 0, 0);
    BIO_free(bo);

    EVP_PKEY_CTX *ctx = EVP_PKEY_CTX_new(pkey, NULL);
    EVP_PKEY_decrypt_init(ctx);
    EVP_PKEY_CTX_ctrl_str(ctx, "rsa_padding_mode", "pkcs1");

    size_t plaintext_len;
    unsigned char* plaintext;

    // determine output size
    EVP_PKEY_decrypt(ctx, NULL, &plaintext_len, ciphertext, strlen(ciphertext));

    // allocate memory (+1 for terminating null-character)
    plaintext = OPENSSL_malloc(plaintext_len + 1);
    EVP_PKEY_decrypt(ctx, plaintext, &plaintext_len, ciphertext, strlen(ciphertext));
    plaintext[plaintext_len] = '\0'; // required for %s

    printf("%s\n", plaintext);
}
```
PKCS8 private keys can be loaded using this technique as well. OAEP is also supported.

Here's the phpseclib code to encrypt something with OAEP:

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
<sup>_(sha1 is being used because the key is a 512-bit key from [Sample RSA Keys](/docs/rsa-keys); 512-bits is used for brevity but because it's 512-bits sha256 can't be used per the max size formulas discussed at [RSA::ENCRYPTION_OAEP](/docs/rsa#rsaencryption_oaep))_</sup>

To decrypt that with C / OpenSSL you'll need to make the following changes to the above C code (using the [phpBB MOD Text Template](phpbb.md#actions)):

```c
#
#-----[ FIND ]------------------------------------------
#
    char ciphertext[] = "L812/9Y8TSpwErlLR6Bz4J3uR/T5YaqtTtB5jxtD1qazGPI5t15V9drWi58colGOZFeCnGKpCrtQWKk4HWRocQ==";
#
#-----[ REPLACE WITH ]----------------------------------
#
    char ciphertext[] = "h3j3zLT2jXCaZuwF7cgUE/Zmc/5IsIfKbaTiBhpCJo86AiyuoA3Yvni+Lrm5wu2OGv2h5R7Zu3voFcHugiystw==";
#
#-----[ FIND ]------------------------------------------
#
    EVP_PKEY_CTX_ctrl_str(ctx, "rsa_padding_mode", "pkcs1");
#
#-----[ REPLACE WITH ]----------------------------------
#
    EVP_PKEY_CTX_ctrl_str(ctx, "rsa_padding_mode", "oaep");
    EVP_PKEY_CTX_ctrl_str(ctx, "rsa_oaep_md", "md5");
    EVP_PKEY_CTX_ctrl_str(ctx, "rsa_mgf1_md", "sha1");
    //EVP_PKEY_CTX_ctrl_str(ctx, "rsa_oaep_label", "");
```
The parameters passed to `EVP_PKEY_CTX_ctrl_str` correspond to the [pkeyopt values in pkeyutil](https://www.openssl.org/docs/man1.1.1/man1/openssl-pkeyutl.html).

An older API for decrypting RSA does exist but the hash and the MGF hash are hard-coded as sha1 for OAEP (which are the [default values in the ASN.1 definition of RSAES-OAEP-params in PKCS1](https://datatracker.ietf.org/doc/html/rfc8017#appendix-A.2.1)).

Here's an example of PKCS1 decryption with that older API:

```c
#include <openssl/evp.h>
#include <openssl/pem.h>
#include <string.h>

int main (void)
{
    char privateKey[] = "-----BEGIN RSA PRIVATE KEY-----\n"\
"MIIBOgIBAAJBAKj34GkxFhD90vcNLYLInFEX6Ppy1tPf9Cnzj4p4WGeKLs1Pt8Qu\n"\
"KUpRKfFLfRYC9AIKjbJTWit+CqvjWYzvQwECAwEAAQJAIJLixBy2qpFoS4DSmoEm\n"\
"o3qGy0t6z09AIJtH+5OeRV1be+N4cDYJKffGzDa88vQENZiRm0GRq6a+HPGQMd2k\n"\
"TQIhAKMSvzIBnni7ot/OSie2TmJLY4SwTQAevXysE2RbFDYdAiEBCUEaRQnMnbp7\n"\
"9mxDXDf6AU0cN/RPBjb9qSHDcWZHGzUCIG2Es59z8ugGrDY+pxLQnwfotadxd+Uy\n"\
"v/Ow5T0q5gIJAiEAyS4RaI9YG8EWx/2w0T67ZUVAw8eOMB6BIUg0Xcu+3okCIBOs\n"\
"/5OiPgoTdSy7bcF9IGpSE8ZgGKzgYQVZeN97YE00\n"\
"-----END RSA PRIVATE KEY-----\n";
    char ciphertext[] = "L812/9Y8TSpwErlLR6Bz4J3uR/T5YaqtTtB5jxtD1qazGPI5t15V9drWi58colGOZFeCnGKpCrtQWKk4HWRocQ==";

    // base64 decode ciphertext
    EVP_DecodeBlock(ciphertext, ciphertext, strlen(ciphertext));

    BIO* bo = BIO_new(BIO_s_mem());
    BIO_write(bo, privateKey, strlen(privateKey));
    EVP_PKEY* pkey = 0;
    PEM_read_bio_PrivateKey(bo, &pkey, 0, 0);
    BIO_free(bo);
    RSA* rsa = EVP_PKEY_get1_RSA(pkey);
    char plaintext[RSA_size(rsa)];

    RSA_private_decrypt(strlen(ciphertext), ciphertext, plaintext, rsa, RSA_PKCS1_PADDING);
    printf("%s\n", plaintext);
}
```
To do OAEP with that API you'd swap out `RSA_PKCS1_PADDING` with `RSA_PKCS1_OAEP_PADDING`.
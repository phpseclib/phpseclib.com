---
id: csharp
title: C#
---

## AES-128-CBC Decryption

Encrypting a string using AES-128-CBC with phpseclib:

```php
use phpseclib3\Crypt\AES;

$cipher = new AES('cbc');
$cipher->setKey(str_repeat('a', 16));
$cipher->setIV(str_repeat('b', 16));

echo bin2hex($cipher->encrypt('test'));
```

Decryption with C#:

```c#
using System;
using System.Security.Cryptography;
using System.Text;
using System.IO;

class Test
{
    // from https://stackoverflow.com/a/311179/569976
    public static byte[] StringToByteArray(String hex)
    {
        int NumberChars = hex.Length;
        byte[] bytes = new byte[NumberChars / 2];
        for (int i = 0; i < NumberChars; i += 2)
            bytes[i / 2] = Convert.ToByte(hex.Substring(i, 2), 16);
          return bytes;
    }

    static void Main()
    {
        var ciphertext = StringToByteArray("10f42fd95857ed2775cfbc4b471bc213");

        var aes = new AesManaged();
        //aes.Mode = CipherMode.CBC;
        var key = Encoding.ASCII.GetBytes("aaaaaaaaaaaaaaaa");
        var iv = Encoding.ASCII.GetBytes("bbbbbbbbbbbbbbbb");

        ICryptoTransform decryptor = aes.CreateDecryptor(key, iv);
        MemoryStream msDecrypt = new MemoryStream(ciphertext);
        CryptoStream csDecrypt = new CryptoStream(msDecrypt, decryptor, CryptoStreamMode.Read);
        StreamReader srDecrypt = new StreamReader(csDecrypt);
        String plaintext = srDecrypt.ReadToEnd();

        Console.WriteLine(plaintext);
    }
}
```

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

Decryption with C#
<!-- csc test.cs && mono test.exe -->

```c#
using System;
using System.Security.Cryptography;
using System.Text;

class Test
{
    static void Main()
    {
        var key = @"<RSAKeyPair>
  <Modulus>qPfgaTEWEP3S9w0tgsicURfo+nLW09/0KfOPinhYZ4ouzU+3xC4pSlEp8Ut9FgL0AgqNslNaK34Kq+NZjO9DAQ==</Modulus>
  <Exponent>AQAB</Exponent>
  <P>oxK/MgGeeLui385KJ7ZOYktjhLBNAB69fKwTZFsUNh0=</P>
  <Q>AQlBGkUJzJ26e/ZsQ1w3+gFNHDf0TwY2/akhw3FmRxs1</Q>
  <DP>bYSzn3Py6AasNj6nEtCfB+i1p3F35TK/87DlPSrmAgk=</DP>
  <DQ>yS4RaI9YG8EWx/2w0T67ZUVAw8eOMB6BIUg0Xcu+3ok=</DQ>
  <InverseQ>E6z/k6I+ChN1LLttwX0galITxmAYrOBhBVl433tgTTQ=</InverseQ>
  <D>IJLixBy2qpFoS4DSmoEmo3qGy0t6z09AIJtH+5OeRV1be+N4cDYJKffGzDa88vQENZiRm0GRq6a+HPGQMd2kTQ==</D>
</RSAKeyPair>";

        var ciphertext = "L812/9Y8TSpwErlLR6Bz4J3uR/T5YaqtTtB5jxtD1qazGPI5t15V9drWi58colGOZFeCnGKpCrtQWKk4HWRocQ==";

        var ciphertextBytes = Convert.FromBase64String(ciphertext);

        var rsa = RSA.Create();
        rsa.FromXmlString(key);

        var plaintextBytes = rsa.Decrypt(ciphertextBytes, RSAEncryptionPadding.Pkcs1);

        var plaintext = Encoding.Default.GetString(plaintextBytes);

        Console.WriteLine(plaintext);
    }
}
```
Note that `rsa.FromXmlString(key)` only supports XML formatted keys (as the name implies lol).

If you're using .NET Core 3.0+ / are _not_ using [Mono](https://en.wikipedia.org/wiki/Mono_(software)), [ImportRSAPrivateKey](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.rsa.importrsaprivatekey?view=netcore-3.0) can be used to load PKCS1 formatted keys, using the following (untested) changes to the above (using the [phpBB MOD Text Template](phpbb.md#actions)):

```c#
#
#-----[ FIND ]------------------------------------------
#
using System.Text;
#
#-----[ AFTER, ADD ]------------------------------------
#
using System.Text.RegularExpressions;
#
#-----[ FIND ]------------------------------------------
#
        var key = @"<RSAKeyPair>
  <Modulus>qPfgaTEWEP3S9w0tgsicURfo+nLW09/0KfOPinhYZ4ouzU+3xC4pSlEp8Ut9FgL0AgqNslNaK34Kq+NZjO9DAQ==</Modulus>
  <Exponent>AQAB</Exponent>
  <P>oxK/MgGeeLui385KJ7ZOYktjhLBNAB69fKwTZFsUNh0=</P>
  <Q>AQlBGkUJzJ26e/ZsQ1w3+gFNHDf0TwY2/akhw3FmRxs1</Q>
  <DP>bYSzn3Py6AasNj6nEtCfB+i1p3F35TK/87DlPSrmAgk=</DP>
  <DQ>yS4RaI9YG8EWx/2w0T67ZUVAw8eOMB6BIUg0Xcu+3ok=</DQ>
  <InverseQ>E6z/k6I+ChN1LLttwX0galITxmAYrOBhBVl433tgTTQ=</InverseQ>
  <D>IJLixBy2qpFoS4DSmoEmo3qGy0t6z09AIJtH+5OeRV1be+N4cDYJKffGzDa88vQENZiRm0GRq6a+HPGQMd2kTQ==</D>
</RSAKeyPair>";
#
#-----[ REPLACE WITH ]----------------------------------
#
        var key = @"-----BEGIN RSA PRIVATE KEY-----
MIIBOgIBAAJBAKj34GkxFhD90vcNLYLInFEX6Ppy1tPf9Cnzj4p4WGeKLs1Pt8Qu
KUpRKfFLfRYC9AIKjbJTWit+CqvjWYzvQwECAwEAAQJAIJLixBy2qpFoS4DSmoEm
o3qGy0t6z09AIJtH+5OeRV1be+N4cDYJKffGzDa88vQENZiRm0GRq6a+HPGQMd2k
TQIhAKMSvzIBnni7ot/OSie2TmJLY4SwTQAevXysE2RbFDYdAiEBCUEaRQnMnbp7
9mxDXDf6AU0cN/RPBjb9qSHDcWZHGzUCIG2Es59z8ugGrDY+pxLQnwfotadxd+Uy
v/Ow5T0q5gIJAiEAyS4RaI9YG8EWx/2w0T67ZUVAw8eOMB6BIUg0Xcu+3okCIBOs
/5OiPgoTdSy7bcF9IGpSE8ZgGKzgYQVZeN97YE00
-----END RSA PRIVATE KEY-----";
#
#-----[ FIND ]------------------------------------------
#
        rsa.FromXmlString(key);
#
#-----[ REPLACE WITH ]----------------------------------
#
        var rx = new Regex("-+[^-]+-+");
        key = rx.Replace(key, "")
                .Replace("\r", "")
                .Replace("\n", "");
        var keyBytes = Convert.FromBase64String(key);

        rsa.ImportRSAPrivateKey(keyBytes, out _);
```
[ImportPkcs8PrivateKey](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.rsa.importpkcs8privatekey) can be used to load PKCS8 formatted keys.

Now let us consider OAEP decryption. Here's the phpseclib to generate OAEP encrypted strings:

```php
use phpseclib3\Crypt\PublicKeyLoader;

$key = PublicKeyLoader::load('-----BEGIN RSA PUBLIC KEY-----
MEgCQQCo9+BpMRYQ/dL3DS2CyJxRF+j6ctbT3/Qp84+KeFhnii7NT7fELilKUSnx
S30WAvQCCo2yU1orfgqr41mM70MBAgMBAAE=
-----END RSA PUBLIC KEY-----');
$key = $key
    //->withPadding(RSA::ENCRYPTION_OAEP)
    ->withHash('sha1')
    ->withMGFHash('sha1');
echo base64_encode($key->encrypt('test'));
```
<sup>_(sha1 is being used because the key is a 512-bit key from [Sample RSA Keys](/docs/rsa-keys); 512-bits is used for brevity but because it's 512-bits sha256 can't be used per the max size formulas discussed at [RSA::ENCRYPTION_OAEP](/docs/rsa#rsaencryption_oaep))_</sup>

Here are the modifications we need to do to the above C# program to perform OAEP decryption:

```c#
#
#-----[ FIND ]------------------------------------------
#
        var ciphertext = "L812/9Y8TSpwErlLR6Bz4J3uR/T5YaqtTtB5jxtD1qazGPI5t15V9drWi58colGOZFeCnGKpCrtQWKk4HWRocQ==";
#
#-----[ REPLACE WITH ]----------------------------------
#
        var ciphertext = "XZ/zwGTyLwm3S1A2N7vg87qlpqtqB4sLo5t6BlayLIfW3kICEiol46ryz/oSsOypAH6GKxcn5TvQF/gJvMOFBg==";
#
#-----[ FIND ]------------------------------------------
#
        var plaintextBytes = rsa.Decrypt(ciphertextBytes, RSAEncryptionPadding.Pkcs1);
#
#-----[ REPLACE WITH ]----------------------------------
#
        var plaintextBytes = rsa.Decrypt(ciphertextBytes, RSAEncryptionPadding.OaepSHA1);
```
If you want or need to use non-matching MGFHash and Hash algorithms in OAEP then you'll need to use [BouncyCastle](https://www.bouncycastle.org/csharp/index.html).

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

Signature verification with C#:

```c#
using System;
using System.Security.Cryptography;
using System.Text;
using System.Text.RegularExpressions;

class Test
{
    static void Main()
    {
        var key = @"<RSAKeyValue>
  <Modulus>qPfgaTEWEP3S9w0tgsicURfo+nLW09/0KfOPinhYZ4ouzU+3xC4pSlEp8Ut9FgL0AgqNslNaK34Kq+NZjO9DAQ==</Modulus>
  <Exponent>AQAB</Exponent>
</RSAKeyValue>";

        var signature = "MUE536c4UJSAmycs7V6qFaLMATrKMQA8TYj5xX1+fwHINz3/BafgaRt0ycoD5IxTxaclLWavrGSza4xSBHraEw==";
        var data = "zzz";

        var signatureBytes = Convert.FromBase64String(signature);
        var dataBytes = Encoding.ASCII.GetBytes(data);

        var rsa = RSA.Create();
        rsa.FromXmlString(key);

        var result = rsa.VerifyData(dataBytes, signatureBytes, HashAlgorithmName.SHA256, RSASignaturePadding.Pkcs1);

        Console.WriteLine(result ? "valid" : "invalid");
    }
}
```
Using [RSAPKCS1SignatureDeformatter](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.rsapkcs1signaturedeformatter?view=netcore-3.0) is not recommended because that class requires you hash the data and then compare that to the hash in the signature. ie. it's more steps and more opportunities to screw up.
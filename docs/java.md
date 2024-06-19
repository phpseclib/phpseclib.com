---
id: java
title: Java
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

Decrypting that same string with Java:

```java
import javax.xml.bind.DatatypeConverter;
import javax.crypto.spec.SecretKeySpec;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.Cipher;

public class Test
{
    public static void main(String[] args) throws Exception
    {
        String key = "aaaaaaaaaaaaaaaa";
        String iv = "bbbbbbbbbbbbbbbb";
        String ciphertext = "10f42fd95857ed2775cfbc4b471bc213";

        byte[] ciphertextBytes = DatatypeConverter.parseHexBinary(ciphertext);

        // perform the actual decryption
        Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
        SecretKeySpec keySpec = new SecretKeySpec(key.getBytes(), "AES");
        IvParameterSpec ivSpec = new IvParameterSpec(iv.getBytes());
        cipher.init(Cipher.DECRYPT_MODE, keySpec, ivSpec);
        byte[] plaintextBytes = cipher.doFinal(ciphertextBytes);
        String plaintext = new String(plaintextBytes);

        System.out.println(plaintext);
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

Decryption with Java:

```java
import org.bouncycastle.openssl.PEMParser;
import org.bouncycastle.openssl.jcajce.JcaPEMKeyConverter;
import org.bouncycastle.openssl.PEMKeyPair;
import java.security.KeyPair;
import java.io.StringReader;
import javax.crypto.Cipher;
import java.util.Base64;
import java.security.interfaces.RSAPrivateKey;

public class Test
{
    public static void main(String[] args) throws Exception
    {
        String ciphertext = "L812/9Y8TSpwErlLR6Bz4J3uR/T5YaqtTtB5jxtD1qazGPI5t15V9drWi58colGOZFeCnGKpCrtQWKk4HWRocQ==";

        String key = "-----BEGIN RSA PRIVATE KEY-----\n" +
"MIIBOgIBAAJBAKj34GkxFhD90vcNLYLInFEX6Ppy1tPf9Cnzj4p4WGeKLs1Pt8Qu\n" +
"KUpRKfFLfRYC9AIKjbJTWit+CqvjWYzvQwECAwEAAQJAIJLixBy2qpFoS4DSmoEm\n" +
"o3qGy0t6z09AIJtH+5OeRV1be+N4cDYJKffGzDa88vQENZiRm0GRq6a+HPGQMd2k\n" +
"TQIhAKMSvzIBnni7ot/OSie2TmJLY4SwTQAevXysE2RbFDYdAiEBCUEaRQnMnbp7\n" +
"9mxDXDf6AU0cN/RPBjb9qSHDcWZHGzUCIG2Es59z8ugGrDY+pxLQnwfotadxd+Uy\n" +
"v/Ow5T0q5gIJAiEAyS4RaI9YG8EWx/2w0T67ZUVAw8eOMB6BIUg0Xcu+3okCIBOs\n" +
"/5OiPgoTdSy7bcF9IGpSE8ZgGKzgYQVZeN97YE00\n" +
"-----END RSA PRIVATE KEY-----\n";

        // load the private key
        PEMParser pemParser = new PEMParser(new StringReader(key));
        JcaPEMKeyConverter converter = new JcaPEMKeyConverter();
        KeyPair keyPair = converter.getKeyPair((PEMKeyPair) pemParser.readObject());
        RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();

        // load the ciphertext
        byte[] cipherBytes = Base64.getDecoder().decode(ciphertext);

        // perform the actual decryption
        Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");
        cipher.init(Cipher.DECRYPT_MODE, privateKey);
        byte[] plaintextBytes = cipher.doFinal(cipherBytes);
        String plaintext = new String(plaintextBytes);

        System.out.println(plaintext);
    }
}
```

The above code only works with PKCS1 keys. If you want to load a PKCS8 key you'll need to make the following changes (using the [phpBB MOD Text Template](phpbb.md#actions)):

```java
#
#-----[ FIND ]------------------------------------------
#
import org.bouncycastle.openssl.PEMParser;
import org.bouncycastle.openssl.jcajce.JcaPEMKeyConverter;
import org.bouncycastle.openssl.PEMKeyPair;
import java.security.KeyPair;
import java.io.StringReader;
#
#-----[ REPLACE WITH ]----------------------------------
#
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.KeyFactory;
#
#-----[ FIND ]------------------------------------------
#
        String key = "-----BEGIN RSA PRIVATE KEY-----\n" +
"MIIBOgIBAAJBAKj34GkxFhD90vcNLYLInFEX6Ppy1tPf9Cnzj4p4WGeKLs1Pt8Qu\n" +
"KUpRKfFLfRYC9AIKjbJTWit+CqvjWYzvQwECAwEAAQJAIJLixBy2qpFoS4DSmoEm\n" +
"o3qGy0t6z09AIJtH+5OeRV1be+N4cDYJKffGzDa88vQENZiRm0GRq6a+HPGQMd2k\n" +
"TQIhAKMSvzIBnni7ot/OSie2TmJLY4SwTQAevXysE2RbFDYdAiEBCUEaRQnMnbp7\n" +
"9mxDXDf6AU0cN/RPBjb9qSHDcWZHGzUCIG2Es59z8ugGrDY+pxLQnwfotadxd+Uy\n" +
"v/Ow5T0q5gIJAiEAyS4RaI9YG8EWx/2w0T67ZUVAw8eOMB6BIUg0Xcu+3okCIBOs\n" +
"/5OiPgoTdSy7bcF9IGpSE8ZgGKzgYQVZeN97YE00\n" +
"-----END RSA PRIVATE KEY-----\n";

        // load the private key
        PEMParser pemParser = new PEMParser(new StringReader(key));
        JcaPEMKeyConverter converter = new JcaPEMKeyConverter();
        KeyPair keyPair = converter.getKeyPair((PEMKeyPair) pemParser.readObject());
        RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();
#
#-----[ REPLACE WITH ]----------------------------------
#
        String key = "-----BEGIN PRIVATE KEY-----\n" +
"MIIBVAIBADANBgkqhkiG9w0BAQEFAASCAT4wggE6AgEAAkEAqPfgaTEWEP3S9w0t\n" +
"gsicURfo+nLW09/0KfOPinhYZ4ouzU+3xC4pSlEp8Ut9FgL0AgqNslNaK34Kq+NZ\n" +
"jO9DAQIDAQABAkAgkuLEHLaqkWhLgNKagSajeobLS3rPT0Agm0f7k55FXVt743hw\n" +
"Ngkp98bMNrzy9AQ1mJGbQZGrpr4c8ZAx3aRNAiEAoxK/MgGeeLui385KJ7ZOYktj\n" +
"hLBNAB69fKwTZFsUNh0CIQEJQRpFCcydunv2bENcN/oBTRw39E8GNv2pIcNxZkcb\n" +
"NQIgbYSzn3Py6AasNj6nEtCfB+i1p3F35TK/87DlPSrmAgkCIQDJLhFoj1gbwRbH\n" +
"/bDRPrtlRUDDx44wHoEhSDRdy77eiQIgE6z/k6I+ChN1LLttwX0galITxmAYrOBh\n" +
"BVl433tgTTQ=\n" +
"-----END PRIVATE KEY-----";

        // extract BER from PEM
        key = key
                  .replaceAll("-+[^-]+-+", "")
                  .replace("\n", "")
                  .replace("\r", "");
        byte[] keyBytes = Base64.getDecoder().decode(key);

        // load decoded private key
        PKCS8EncodedKeySpec spec = new PKCS8EncodedKeySpec(keyBytes);
        KeyFactory kf = KeyFactory.getInstance("RSA");
        RSAPrivateKey privateKey = (RSAPrivateKey) kf.generatePrivate(spec);
```

In the PKCS1 example BouncyCastle is being used to read the key and the actual decryption is being performed by the com.sun.crypto.provider.SunJCE provider.

Normally, com.sun.crypto.provider.SunJCE and org.bouncycastle.jce.provider.BouncyCastleProvider will yield the same result but that is not always the case. Consider OAEP.

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
<sup>_(md5 is being used because the key is a 512-bit key from [Sample RSA Keys](/docs/rsa-keys); 512-bits is used for brevity but because it's 512-bits sha256 can't be used per the max size formulas discussed at [RSA::ENCRYPTION_OAEP](/docs/rsa#rsaencryption_oaep); sha1 would work but for the purposes of this demonstration it can't be sha1)_</sup>

Now let's modify the Java code again:

```java
#
#-----[ FIND ]------------------------------------------
#
        String ciphertext = "L812/9Y8TSpwErlLR6Bz4J3uR/T5YaqtTtB5jxtD1qazGPI5t15V9drWi58colGOZFeCnGKpCrtQWKk4HWRocQ==";
#
#-----[ REPLACE WITH ]----------------------------------
#
        String ciphertext = "h3j3zLT2jXCaZuwF7cgUE/Zmc/5IsIfKbaTiBhpCJo86AiyuoA3Yvni+Lrm5wu2OGv2h5R7Zu3voFcHugiystw==";
#
#-----[ FIND ]------------------------------------------
#
        Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");
#
#-----[ REPLACE WITH ]----------------------------------
#
        Cipher cipher = Cipher.getInstance("RSA/ECB/OAEPWithMD5AndMGF1Padding");
```
That works just fine. BUT what if use `Cipher.getInstance("RSA/ECB/OAEPWithMD5AndMGF1Padding", "BC")` instead? eg. what if we force org.bouncycastle.jce.provider.BouncyCastleProvider to be the crypto provider instead of com.sun.crypto.provider.SunJCE? Then it doesn't work.

org.bouncycastle.jce.provider.BouncyCastleProvider assumes the MGFHash and the Hash are using the same algorithm whereas com.sun.crypto.provider.SunJCE has the MGFHash hard-coded as sha1. To work around this we can do the following:

```java
#
#-----[ FIND ]------------------------------------------
#
import java.security.interfaces.RSAPrivateKey;
#
#-----[ AFTER, ADD ]------------------------------------
#
import javax.crypto.spec.OAEPParameterSpec;
import java.security.spec.MGF1ParameterSpec;
import javax.crypto.spec.PSource.PSpecified;
#
#-----[ FIND ]------------------------------------------
#
        Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");
        cipher.init(Cipher.DECRYPT_MODE, privateKey);
#
#-----[ REPLACE WITH ]----------------------------------
#
        Cipher cipher = Cipher.getInstance("RSA/ECB/OAEPPadding");
        OAEPParameterSpec oaepParams = new OAEPParameterSpec(
            "MD5",
            "MGF1",
            new MGF1ParameterSpec("SHA-1"),
            PSpecified.DEFAULT
        );
        cipher.init(Cipher.DECRYPT_MODE, privateKey, oaepParams);
```

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

Signature verification with Java:

```java
import org.bouncycastle.openssl.PEMParser;
import org.bouncycastle.openssl.jcajce.JcaPEMKeyConverter;
import org.bouncycastle.asn1.x509.SubjectPublicKeyInfo;
import java.io.StringReader;
import java.util.Base64;
import java.security.interfaces.RSAPublicKey;
import java.security.Signature;

public class Test
{
    public static void main(String[] args) throws Exception
    {
        String plaintext = "zzz";
        String signature = "MUE536c4UJSAmycs7V6qFaLMATrKMQA8TYj5xX1+fwHINz3/BafgaRt0ycoD5IxTxaclLWavrGSza4xSBHraEw==";

        String key = "-----BEGIN RSA PUBLIC KEY-----\n" +
"MEgCQQCo9+BpMRYQ/dL3DS2CyJxRF+j6ctbT3/Qp84+KeFhnii7NT7fELilKUSnx\n" +
"S30WAvQCCo2yU1orfgqr41mM70MBAgMBAAE=\n" +
"-----END RSA PUBLIC KEY-----\n";

        // load the public key
        PEMParser pemParser = new PEMParser(new StringReader(key));
        JcaPEMKeyConverter converter = new JcaPEMKeyConverter();
        SubjectPublicKeyInfo pub = SubjectPublicKeyInfo.getInstance(pemParser.readObject());
        RSAPublicKey publicKey = (RSAPublicKey) converter.getPublicKey(pub);

        // load the plaintext
        byte[] signatureBytes = Base64.getDecoder().decode(signature);

        // perform the actual signature verification
        Signature sig = Signature.getInstance("SHA256withRSA");
        sig.initVerify(publicKey);
        sig.update(plaintext.getBytes());
        System.out.println(sig.verify(signatureBytes) ? "good" : "bad");
    }
}
```

The above code only works with PKCS1 keys. If you want to load a PKCS8 key you'll need to make the following changes (using the [phpBB MOD Text Template](phpbb.md#actions)):

```java
#
#-----[ FIND ]------------------------------------------
#
import org.bouncycastle.openssl.PEMParser;
import org.bouncycastle.openssl.jcajce.JcaPEMKeyConverter;
import org.bouncycastle.asn1.x509.SubjectPublicKeyInfo;
import java.io.StringReader;
#
#-----[ REPLACE WITH ]----------------------------------
#
import java.security.spec.X509EncodedKeySpec;
import java.security.KeyFactory;
#
#-----[ FIND ]------------------------------------------
#
        String key = "-----BEGIN RSA PUBLIC KEY-----\n" +
"MEgCQQCo9+BpMRYQ/dL3DS2CyJxRF+j6ctbT3/Qp84+KeFhnii7NT7fELilKUSnx\n" +
"S30WAvQCCo2yU1orfgqr41mM70MBAgMBAAE=\n" +
"-----END RSA PUBLIC KEY-----\n";

        // load the public key
        PEMParser pemParser = new PEMParser(new StringReader(key));
        JcaPEMKeyConverter converter = new JcaPEMKeyConverter();
        SubjectPublicKeyInfo pub = SubjectPublicKeyInfo.getInstance(pemParser.readObject());
        RSAPublicKey publicKey = (RSAPublicKey) converter.getPublicKey(pub);
#
#-----[ REPLACE WITH ]----------------------------------
#
        String key = "-----BEGIN PUBLIC KEY-----\n" +
"MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAKj34GkxFhD90vcNLYLInFEX6Ppy1tPf\n" +
"9Cnzj4p4WGeKLs1Pt8QuKUpRKfFLfRYC9AIKjbJTWit+CqvjWYzvQwECAwEAAQ==\n" +
"-----END PUBLIC KEY-----";

        // extract BER from PEM
        key = key
                  .replaceAll("-+[^-]+-+", "")
                  .replace("\n", "")
                  .replace("\r", "");
        byte[] keyBytes = Base64.getDecoder().decode(key);

        // load decoded public key
        X509EncodedKeySpec spec = new X509EncodedKeySpec(keyBytes);
        KeyFactory kf = KeyFactory.getInstance("RSA");
        RSAPublicKey publicKey = (RSAPublicKey) kf.generatePublic(spec);
```

If you want to do PSS signature verification and you have org.bouncycastle.jce.provider.BouncyCastleProvider as an available crypto provider (as of Java 8, sun.security.rsa.SunRsaSign does not support PSS), you can do `Signature.getInstance("SHA1withRSA/PSS")` instead of `Signature.getInstance("SHA1withRSA")`. If you do `SHA256withRSA/PSS` (which you're not going to be able to do with 512-bit RSA keys) you'll need to keep in mind that the MGFHash is assumed to be the same as the regular Hash. If that's not the case (or if you're using a salt length other than the length of the Hash) then you'll need to do something like this:

```java
#
#-----[ FIND ]------------------------------------------
#
import java.security.Signature;
#
#-----[ AFTER, ADD ]------------------------------------
#
import java.security.spec.PSSParameterSpec;
import java.security.spec.MGF1ParameterSpec;
import java.security.MessageDigest;
#
#-----[ FIND ]------------------------------------------
#
        Signature sig = Signature.getInstance("SHA256withRSA");
#
#-----[ REPLACE WITH ]----------------------------------
#
        Signature sig = Signature.getInstance("RSASSA-PSS");
        PSSParameterSpec pssParams = new PSSParameterSpec(
            "SHA-256",
            "MGF1",
            new MGF1ParameterSpec("SHA-1"),
            MessageDigest.getInstance("SHA-256").getDigestLength(),
            PSSParameterSpec.TRAILER_FIELD_BC
        );
        sig.setParameter(pssParams);
```
In this example SHA-256 is the Hash, SHA-1 is the MGFHash and the salt length is equal to the length of the hash in bytes _(this example requires a minimum 528-bit RSA private key instead of the 512-bit RSA private keys we've been using so a complete working example is not provided; in theory we could use MD5 as the Hash as we did with the OAEP example but Java doesn't support MD5 for PSS)_
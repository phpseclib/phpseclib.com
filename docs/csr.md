---
id: csr
title: CSR
---

<script src="//ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
<script src="/js/jquery.treeview.js"></script>
<link rel="stylesheet" href="/css/jquery.treeview.css">
<script>
var showPath = function() {
  path = '';
  parents = $(this).parents('li:visible');
  for (var i = parents.length - 1; i >= 0; i--) {
    name = $('.name', parents[i])[0].textContent;
    delimiter = isNaN(name) ? '\'' : '';
    path+= '[' + delimiter + name + delimiter + ']';
  }
  $('#path').html('<code id="path">$csr' + path + '</code>:');
};
$(document).ready(function() {
  $('.printr').treeview({
     persist: "location",
     collapsed: true,
     unique: true
   });
   $('.name,.hitarea').click(showPath);
   $('#path').html('<code id="path">$csr</code>');
});
</script>

## Reading CSRs

```php
use phpseclib3\File\X509;

$x509 = new X509();
$csr = $x509->loadCSR(file_get_contents('csr.csr'));

var_dump($csr);
```
<span style="font-size: 11px">([download csr.csr](/x509/csr.csr))</span>

Running the above will produce an array that looks something like this:

<code id="path">$csr</code>
<ul class="printr"><li><span class="name">certificationRequestInfo</span><ul><li><span class="name">version</span><ul><li>v1</li></ul></li><li><span class="name">subject</span><ul><li><span class="name">rdnSequence</span><ul><li><span class="name">0</span><ul><li><span class="name">0</span><ul><li><span class="name">type</span><ul><li>id-at-organizationName</li></ul></li><li><span class="name">value</span><ul><li><span class="name">utf8String</span><ul><li>phpseclib demo cert</li></ul></li></ul></li></ul></li></ul></li></ul></li></ul></li><li><span class="name">subjectPKInfo</span><ul><li><span class="name">algorithm</span><ul><li><span class="name">algorithm</span><ul><li>rsaEncryption</li></ul></li></ul></li><li><span class="name">subjectPublicKey</span><ul><li>

```
-----BEGIN PUBLIC KEY-----
MIGdMAsGCSqGSIb3DQEBAQOBjQAwgYkCgYEApQBlwwoORvSZqUmpL5poADLWsRBx
55tLs3Yq8ZfbkSo8nHetbPnOukIP/Nlxn3n/MGOvsXQa9NLisLQ6UmyLyJXXVW8F
n6pCQIpnygjNbocgs6uj47UEXDwMwlfLKPZ80M7hmMphZZ4Ub/IBe0S5KN77YxYv
FKGKfMNXKZWx1/8CAwEAAQ==
-----END PUBLIC KEY-----
```

</li></ul></li></ul></li></ul></li><li><span class="name">signatureAlgorithm</span><ul><li><span class="name">algorithm</span><ul><li>sha1WithRSAEncryption</li></ul></li></ul></li><li><span class="name">signature</span><ul><li>...</li></ul></li></ul>

### getDNProp()

```php
print_r($x509->getDNProp('O'));
```
That will produce the following:

<ul class="printr"><li><span class="name">0</span><ul><li>phpseclib demo cert</li></ul></li></ul>

An array is returned because each distinguished name property can (in theory) have multiple values

Valid property names are enumerated upon at [Distinguished Property Names](dnprops.md).

### getDN()

```php
print_r($x509->getDN());
```

`getDN()` accepts several different parameters:

- `X509::DN_ARRAY` (the default value) returns an array who's keys are based on the ASN.1 syntax for X.509:
  <ul class="printr"><li><span class="name">rdnSequence</span><ul><li><span class="name">0</span><ul><li><span class="name">0</span><ul><li><span class="name">type</span><ul><li>id-at-organizationName</li></ul></li><li><span class="name">value</span><ul><li><span class="name">utf8String</span><ul><li>phpseclib demo cert</li></ul></li></ul></li></ul></li></ul></li></ul></li></ul>

- `X509::DN_STRING` returns an OpenSSL-style string:
  ```
  O=phpseclib demo cert
  ```

- `X509::DN_OPENSSL` returns an OpenSSL-style array:
  <ul class="printr"><li><span class="name">O</span><ul><li>phpseclib demo cert</li></ul></li></ul>

- `X509::DN_ASN1` returns a DER encoded binary string
- `X509::DN_CANON` returns a "canonicalized" DER encoded binary string wherein SEQUENCE around RDNs and all string values normalized as trimmed lowercase UTF-8 with all spacing as one blank. Constructed RDNs are not canonicalized.

### getPublicKey()

```php
echo $x509->getPublicKey();
```
Returns a `\phpseclib3\Crypt\Common\PublicKey` object that, by default, gets cast to a PKCS8-encoded public key:

```
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQClAGXDCg5G9JmpSakvmmgAMtax
EHHnm0uzdirxl9uRKjycd61s+c66Qg/82XGfef8wY6+xdBr00uKwtDpSbIvIlddV
bwWfqkJAimfKCM1uhyCzq6PjtQRcPAzCV8so9nzQzuGYymFlnhRv8gF7RLko3vtj
Fi8UoYp8w1cplbHX/wIDAQAB
-----END PUBLIC KEY-----
```

## Validating Signatures

CSR's are always self-signed.

```php
$x509 = new X509();
$csr = $x509->loadCSR(file_get_contents('csr.csr'));
echo $x509->validateSignature() ? 'valid' : 'invalid';
```

## Creating CSRs: An Example

```php
use phpseclib3\File\X509;
use phpseclib3\Crypt\RSA;

$privKey = RSA::createKey();

$x509 = new X509();
$x509->setPrivateKey($privKey);
$x509->setDNProp('id-at-organizationName', 'phpseclib demo cert');

$csr = $x509->signCSR();

echo $x509->saveCSR($csr);
```

Domains can be added to a CSR by doing `$x509->setDomain('www.domain.tld')`. Note that whereas `$x509->setDomain()` can be used to add multiple domains for X.509 certificates only one domain can be added, with this approach, to CSRs. This is a limitation of phpseclib - not of the CSR format.

Multiple domains may be added by doing the following (using the [phpBB MOD Text Template](phpbb.md#actions)):

```php
#
#-----[ FIND ]------------------------------------------
#
$csr = $x509->signCSR();
#
#-----[ BEFORE, ADD ]-----------------------------------
#
$x509->loadCSR($x509->saveCSR($x509->signCSR()));
$x509->setExtension('id-ce-subjectAltName', [
    ['dNSName' => 'www.domain.tld'],
    ['dNSName' => 'domain.tld']
]);
```
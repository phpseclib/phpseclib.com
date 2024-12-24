---
id: connect
title: Connecting
---

## Basic Example with Signature Verification

As the "secure" part of Secure Shell (SSH) implies, SSH is designed to work over hostile networks. SSH encrypts your data so that eavesdroppers cannot read the data being sent back and forth and it provides a method that can be used to verify that the server you're connecting to hasn't been replaced with a hostile server. To facilitate the latter SSH servers have a host public key. Data that's unique to the SSH session is signed by the server and should be verified by the client with the host public key. Of course, simply verifying the signature is insufficient - you need to verify that the host public key is correct. X.509 / SSL / TLS does this with [certificate authorities](https://en.wikipedia.org/wiki/Certificate_authority) but in SSH, in theory, you'd get the host public key through some [out-of-band method](https://en.wikipedia.org/wiki/Out-of-band_agreement). In practice, however, people usually just cache the key the first time they connect to a server and assume all subsequent connections should be using that same key. How the expected host key is saved is up to the application designer (OpenSSH saves them in `~/.ssh/known_hosts`) but here is an example of how the host key would be retrieved from the SSH server (prior to authentication) and checked against the expected value (`$expected`):

```php
use phpseclib3\Net\SSH2;

$ssh = new SSH2('localhost', 22);
if ($expected != $ssh->getServerPublicHostKey()) {
    throw new \Exception('Host key verification failed');
}
```

All subsequent code samples omit this part for brevity but if you're concerned about eavesdroppers (which isn't always a legit concern; eg. if you're connecting to localhost) it should not be skipped.

The port number, incidentally, is optional. If not specified it will be assumed to be 22.

## Using an HTTP Proxy

```php
use phpseclib3\Net\SSH2;

$fsock = fsockopen('127.0.0.1', 80, $errno, $errstr, 1);
if (!$fsock) {
    throw new \Exception($errstr);
}
fputs($fsock, "CONNECT localhost:22 HTTP/1.0\r\n");
//fputs($fsock, "Proxy-Authorization: Basic " . base64_encode('user:pass') . "\r\n");
fputs($fsock, "\r\n");
while ($line = fgets($fsock, 1024)) {
    if ($line == "\r\n") {
        break;
    }
    //echo $line;
}

$ssh = new SSH2($fsock);
$ssh->login('username', 'password');
echo $ssh->exec('ls -latr');
```

## Using a SOCKS5 Proxy

```php
use phpseclib3\Net\SSH2;

// SSH connection info
$port = 22;
$address = 'localhost';

// SOCKS5 connection info
$fsock = fsockopen('127.0.0.1', 1080, $errno, $errstr, 1);
if (!$fsock) {
    throw new \Exception($errstr);
}

$port = pack('n', $port);
$address = chr(strlen($address)) . $address;

$request = "\5\1\0";
if (fwrite($fsock, $request) != strlen($request)) {
    throw new \Exception('Premature termination');
}

$response = fread($fsock, 2);
if ($response != "\5\0") {
    throw new \Exception('Unsupported protocol or unsupported method');
}

$request = "\5\1\0\3$address$port";
if (fwrite($fsock, $request) != strlen($request)) {
    throw new \Exception('Premature termination');
}

$response = fread($fsock, strlen($address) + 6);
if (substr($response, 0, 2) != "\5\0") {
echo bin2hex($response) . "\n";
    throw new \Exception("Unsupported protocol or connection refused");
}

$ssh = new SSH2($fsock);
$ssh->login('username', 'password');
echo $ssh->exec('ls -latr');
```

## Connecting to an IPv6 address

When specifying a numerical IPv6 address (e.g. `fe80::1`), you must enclose the IP in square brackets—for example, `tcp://[fe80::1]:22`.

## Binding to a Specific IP Address

```php
use phpseclib3\Net\SSH2;

// http://php.net/manual/en/context.socket.php
$opts = [
    'socket' => [
        'bindto' => '127.255.255.255:0',
    ],
];
$context = stream_context_create($opts);
$socket = stream_socket_client('tcp://localhost:22', $errno, $errstr, ini_get('default_socket_timeout'), STREAM_CLIENT_CONNECT, $context);

$ssh = new SSH2($socket);
$ssh->login('username', 'password');
echo $ssh->exec('ls -latr');
```

## Using a Custom Cipher Suite

You can tell phpseclib which algorithms you'd like to use by doing `$ssh->setPreferredAlgorithms($methods)`. `$methods` should be an associative array with any or all of the following parameters (inspired by [ssh2_connect](https://www.php.net/ssh2-connect)):

| Index | Meaning | Supported Values |
|---|---|---|
| kex | List of key exchange methods to advertise, comma separated in order of preference. | _curve25519-sha256_, _curve25519-sha256[]()@libssh.org_, _ecdh-sha2-nistp256_, _ecdh-sha2-nistp384_, _ecdh-sha2-nistp521_, _diffie-hellman-group-exchange-sha256_, _diffie-hellman-group-exchange-sha1_, _diffie-hellman-group14-sha256_, _diffie-hellman-group14-sha1_, _diffie-hellman-group15-sha512_, _diffie-hellman-group16-sha512_, _diffie-hellman_group17-sha512_, _diffie-hellman-group18-sha512_, _diffie-hellman-group1-sha1_. Pretty much anything returned by `$ssh->getSupportedKEXAlgorithms()` |
| hostkey | List of hostkey methods to advertise, comma separated in order of preference. | _ssh-ed25519_, _ecdsa-sha2-nistp256_, _ecdsa-sha2-nistp384_, _ecdsa-sha2-nistp521_, _rsa-sha2-256_, _rsa-sha2-512_, _ssh-rsa_, _ssh-dss_. Pretty much anything returned by `$ssh->getSupportedHostKeyAlgorithms()` |
| client_to_server | Associative array containing crypt, compression, and message authentication code (MAC) method preferences for messages sent from client to server. ||
| server_to_client | Associative array containing crypt, compression, and message authentication code (MAC) method preferences for messages sent from server to client. ||

`client_to_server` and `server_to_client` should be an associative array with any or all of the following parameters.

| Index | Meaning | Supported Values |
|---|---|---|
| crypt | List of crypto methods to advertise, comma separated in order of preference. | _aes128-gcm[]()@openssh.com_, _aes256-gcm[]()@openssh.com_, _arcfour256_, _arcfour128_, _aes128-ctr_, _aes192-ctr_, _aes256-ctr_, _chacha20-poly1305[]()@openssh.com_, _twofish128-ctr_, _twofish192-ctr_, _twofish256-ctr_, _aes128-cbc_, _aes192-cbc_, _aes256-cbc_, _twofish128-cbc_, _twofish192-cbc_, _twofish256-cbc_, _twofish-cbc_, _blowfish-ctr_, _blowfish-cbc_, _3des-ctr_, _3des-cbc_. Pretty much anything returned by `$ssh->getSupportedEncryptionAlgorithms()` |
| comp | List of compression methods to advertise, comma separated in order of preference. | _none_, _zlib[]()@openssh.com_, _zlib_ <sup style="color: red"><strong>[1]</strong></sup>. Pretty much anything returned by `$ssh->getSupportedCompressionAlgorithms()` |
| mac | List of MAC methods to advertise, comma separated in order of preference. | _hmac-sha2-256-etm[]()@openssh.com_, _hmac-sha2-512-etm[]()@openssh.com_, _umac-64-etm[]()@openssh.com_, _umac-128-etm[]()@openssh.com_, _hmac-sha1-etm[]()@openssh.com_, _hmac-sha2-256_, _hmac-sha2-512_, _umac-64[]()@openssh.com_, _umac-128[]()@openssh.com_, _hmac-sha1-96_, _hmac-sha1_, _hmac-md5-96_, _hmac-md5_. Pretty much anything returned by `$ssh->getSupportedMACAlgorithms()` |

Note that a given algorithm will only be used if it's supported by both phpseclib and the server. The algorithms that the server supports can be determined by doing `$ssh->getServerAlgorithms()`. The algorithms that ultimately wind up being used can be determined by doing `$ssh->getAlgorithmsNegotiated()`.

Using a custom cipher suite is not recommended. phpseclib's prioritization of algorithms is intended to maximize speed and security. For example, if OpenSSL is installed and you're using PHP >= 7.1.0 then _aes128-gcm[]()@openssh.com_ will be the preferred algorithm. If (1) OpenSSL is not installed or you're using PHP < 7.1.0 BUT (2) libsodium is installed, then _aes256-gcm[]()@openssh.com_ will be the preferred algorithm. You can make either of those the preferred algorithms even if neither OpenSSL or libsodium are installed but your connection will be slowed down because the pure-PHP implemenation of both of those is not nearly as fast as OpenSSL / libsodium.

_chacha20-poly1305[]()@openssh.com_ is the latest hotness in the cryptographic community but it is not prioritized higher because (1) while OpenSSL supports ChaCha20, it doens't support Poly1305 and (2) libsodium doesn't use Poly1305 in the same way that SSH uses it. Despite that, _chacha20-poly1305[]()@openssh.com_ is still pretty fast but not as fast as some of the other available algorithms.

<div style="font-size: 11px">

<sup style="color: red"><strong>[1]</strong></sup> _zlib[]()@openssh.com_ and _zlib_ support were introduced in phpseclib v3.0.11, require PHP 7.0+ and that the [zlib extension be installed](https://www.php.net/manual/en/zlib.installation.php) (until such time that a shim can be written).
</div>
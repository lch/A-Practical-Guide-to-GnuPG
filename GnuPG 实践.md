# GnuPG 实践

## 什么是 GnuPG ？

### GnuPG 简介

GnuPG 是一个用于加密、数字签名及产生非对称匙对的软件。

[With Logo]



### 从 PGP（Pretty Good Privacy）说起

* PGP —— Pretty Good Privacy
* 由 Phil R. Zimmermann 在 1991 年开发并免费于网络上发放。
* 由于美国的出口限制，PGP 二进制程序、源代码在法律上不允许被传播到美国本土以外。
* PGP 的作者将源代码由 MIT Press 出版并印刷成了一本书《PGP Source and Internals》，由于书籍的出口受到美国宪法第一修正案的保护，任何人都可以以 $60 购买这本书，并将源代码通过 OCR 扫描并录入电脑，并使用编译器编译为二进制文件。
* PGP 仍然是商业软件，属于 PGP Inc. （Zimmermann 作为公司 CTO 与 Chairman），后并入 Network Associate （NAI，McAfee）现在已经被 Symantec 收购。
* 在 1997 年 Zimmermann 与 IETF 共同制订 OpenPGP 标准（RFC1991 等）。




### OpenPGP 与 GnuPG

* GnuPG （GNU Privacy Guard） 基于 OpenPGP 标准开发，是 OpenPGP 标准的一个实现。
* GnuPG 最初由德国人 Werner Koch 开发。
* GnuPG 以 GNU Public License 发布，是自由软件。




### 对称加密与非对称加密

#### 对称加密

使用同一个密钥加密和解密（DES、AES）

#### 非对称加密

使用不同的密钥加密解密（RSA、ECC）



### 获取 GnuPG

#### GNU/Linux

大多数 GNU/Linux 发行版自带 GnuPG。



#### Windows

Gpg4win [https://www.gpg4win.org/download.html](https://www.gpg4win.org/download.html)



#### macOS

GPG Suite [https://gpgtools.org/](https://gpgtools.org/)



#### Android

OpenKeychain [https://www.openkeychain.org/](https://www.openkeychain.org/)



## GnuPG 最佳实践

### 使用 GnuPG 2

* 请使用 GnuPG 2。

* 查看版本

  ```bash
  gpg --verion
  ```

* GnuPG 1 与 2 的区别

  * GnuPG 2 提供新的加密算法。

    ```
    gpg (GnuPG) 2.2.1
    libgcrypt 1.7.9
    Copyright (C) 2017 Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <https://gnu.org/licenses/gpl.html>
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.

    Home: /home/lch/.gnupg
    支持的算法：
    公钥：RSA, ELG, DSA, ECDH, ECDSA, EDDSA
    对称加密：IDEA, 3DES, CAST5, BLOWFISH, AES, AES192, AES256,
         TWOFISH, CAMELLIA128, CAMELLIA192, CAMELLIA256
    散列：SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224
    压缩：不压缩, ZIP, ZLIB, BZIP2
    ```

    ​

  * GnuPG 1 作为独立版拥有最小依赖，提供给嵌入式系统或服务器。

    ```
    This is the standalone verion of gpg. For desktop use you should consider using gpg2 from GnuPG-2 Package.

    ... GnuPG 1.x, which is might be better suited for server and embedded platforms...
    ```

  * GnuPG 1 可能还提供一部分对 PGP 2.6 的支持。

  * ……



### 产生密钥对

最新版本的 GnuPG 在使用 `gpg2 --gen-key ` 命令的时候会默认产生 RSA-2048bit 的一个主密钥和一个子密钥，并且主密钥默认开启签名和认证功能，子密钥默认开启加密功能，并且采用 SHA1 作为摘要算法。这一默认设置存在问题，故要尽量避免使用简单的 `--gen-key` 参数生成密钥。

#### Do it in right way

#### 编辑配置文件

在开始之前，你需要编辑`~/.gnupg/gpg.conf`，以提高安全性

```
# Long keyids are more collision-resistant than short keyids
keyid-format 0xlong

# List all keys along with their fingerprints
with-fingerprint

# Do not merge primary user ID and primary key in --with-colon
# listing mode and print all timestamps as seconds since
# 1970-01-01
fixed-list-mode

# This allows the user to safely override the algorithm chosen by the recipient
# key preferences, as GPG will only select an algorithm that is usable by all recipients
personal-cipher-preferences   AES256 CAMELLIA256 AES192 CAMELLIA192
personal-digest-preferences   SHA512 SHA384 SHA256 SHA224
personal-compress-preferences ZLIB BZIP2 Uncompressed


# This preference list is used for new keys and becomes the default for "setpref" in the edit menu
default-preference-list AES256 CAMELLIA256 AES192 CAMELLIA192 SHA512 SHA384 SHA256 SHA224 ZLIB BZIP2 Uncompressed

# Message digest algorithm used when signing a key
cert-digest-algo SHA512

# Command line that should be run to view a photo ID
photo-viewer feh --quiet --borderless --title 'GnupG KeyID 0x%K' -
```

编辑 `~/.gnupg/gpg-agent.conf` 

```
# Set the minimal length of a passphrase
min-passphrase-len 10

# Set the minimal number of digits or special characters required in a passphrase
min-passphrase-nonalpha 3

# Ask the user to change the passphrase since the last change
max-passphrase-days 90

# Enable the OpenSSH Agent protocol
enable-ssh-support
```





Full Ver of gpg.conf :

```
# Suppress the initial copyright message
no-greeting

# Disable inclusion of the version string in ASCII armored output
no-emit-version

# Disable comment string in clear text signatures and ASCII armored messages
no-comments

# Refuse to run if GnuPG cannot get secure memory
require-secmem

# Long keyids are more collision-resistant than short keyids
keyid-format 0xlong

# List all keys along with their fingerprints
with-fingerprint

# Do not merge primary user ID and primary key in --with-colon
# listing mode and print all timestamps as seconds since
# 1970-01-01
fixed-list-mode

# Show usage information for keys and subkeys in the standard key listing
list-options show-usage

# Show policy URLs in the --list-sigs or --check-sigs listings
list-options show-policy-urls

# Show all signature notations in the -list-sigs or --check-sigs listings
list-options show-notations

# Show any preferred keyserver URL in the --list-sigs or --check-sigs listings
list-options show-keyserver-urls

# Display the calculated validity of user IDs during key listings
list-options show-uid-validity

# Show revoked and expired user IDs in key listings
list-options show-unusable-uids

# Show revoked and expired subkeys in key listings
list-options show-unusable-subkeys

# Show signature expiration dates (if any) during --list-sigs or --check-sigs listings
list-options show-sig-expire

# Display any photo IDs present on the key that issued the signature
verify-options show-photos

# Show policy URLs in the signature being verified
verify-options show-policy-urls

# Show all signature notations in the signature being verified
verify-options show-notations

# Show any preferred keyserver URL in the signature being verified
verify-options show-keyserver-urls

# Display the calculated validity of the user IDs on the key that issued the signature
verify-options show-uid-validity

# Show revoked and expired user IDs during signature verification
verify-options show-unusable-uids

# Enable PKA lookups to verify sender addresses
verify-options pka-lookups

# Locate a key using DNS CERT, as specified in RFC4398
auto-key-locate cert

# Locate a key using DNS PKA
auto-key-locate pka

# Locate  a  key  using whatever keyserver is defined using the --keyserver option
auto-key-locate keyserver

# Use name as your keyserver
keyserver http://keys.gnupg.net
keyserver http://subset.pool.sks-keyservers.net

# Automatically fetch keys as needed from the keyserver when verifying
# signatures or when importing keys that have been revoked by a revocation
# key that is not present on the keyring
keyserver-options auto-key-retrieve

# When searching, include keys marked as "revoked" on the keyserver
keyserver-options include-revoked

# If the key in question has a preferred keyserver URL, then use that preferred
# keyserver to refresh the key from
keyserver-options honor-keyserver-url

# If auto-key-retrieve is set, and the signature being verified has a PKA
# record, then use the PKA information to fetch the key
keyserver-options honor-pka-record

# Tell the keyserver helper program how long (in seconds) to try and perform
# a keyserver action before giving up
keyserver-options timeout=10

# To make use of the agent, you have to run an agent as daemon and use the option
use-agent

# This allows the user to safely override the algorithm chosen by the recipient
# key preferences, as GPG will only select an algorithm that is usable by all recipients
personal-cipher-preferences   AES256 CAMELLIA256 AES192 CAMELLIA192
personal-digest-preferences   SHA512 SHA384 SHA256 SHA224
personal-compress-preferences ZLIB BZIP2 Uncompressed

# This preference list is used for new keys and becomes the default for "setpref" in the edit menu
default-preference-list AES256 CAMELLIA256 AES192 CAMELLIA192 SHA512 SHA384 SHA256 SHA224 ZLIB BZIP2 Uncompressed

# Message digest algorithm used when signing a key
cert-digest-algo SHA512

# Command line that should be run to view a photo ID
photo-viewer feh --quiet --borderless --title 'GnupG KeyID 0x%K' -
```



#### 生成认证用主 Key

在命令行中执行：

```bash
gpg2 --full-gen-key --expert
```



```
gpg (GnuPG) 2.2.1; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
   (9) ECC and ECC
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (13) Existing key
Your selection? 11
```



```
Possible actions for a ECDSA/EdDSA key: Sign Certify Authenticate
Current allowed actions: Sign Certify

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? S
```



```
Possible actions for a ECDSA key: Sign Certify Authenticate 
Current allowed actions: Certify 

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? Q
```



```
Please select which elliptic curve you want:
   (1) Curve 25519
   (2) NIST P-256
   (3) NIST P-384
   (4) NIST P-521
   (5) Brainpool P-256
   (6) Brainpool P-384
   (7) Brainpool P-512
Your selection? 1
```



```
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 5y
```



```
Key expires at Tue 15 Nov 2022 03:13:48 PM UTC
Is this correct? (y/N) y
```



```
GnuPG needs to construct a user ID to identify your key.

Real name: John Smith
Email address: john@example.com
Comment:
You selected this USER-ID:
    "John Smith <john@example.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
```

确保 Comment 处**没有** 任何内容。



```
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key 09F33B41FEDB1EE9 marked as ultimately trusted
gpg: revocation certificate stored as '/home/lch/.gnupg/openpgp-revocs.d/211E5AF21332341490088FF309F33B41FEDB1EE9.rev'
public and secret key created and signed.

gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   2  signed:   2  trust: 0-, 0q, 0n, 0m, 0f, 2u
gpg: depth: 1  valid:   2  signed:   0  trust: 1-, 0q, 0n, 0m, 1f, 0u
gpg: next trustdb check due at 2016-10-01


pub   ed25519 2017-11-16 [C] [expires: 2022-11-15]
      211E5AF21332341490088FF309F33B41FEDB1EE9
uid                      John Smith <john@example.com>
```



#### 增加签名用子 Key

确保 Current allowed actions: Sign

```
gpg2 --expert --edit-key 211E5AF21332341490088FF309F33B41FEDB1EE9
```

```
gpg> addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
Your selection? 11

Possible actions for a ECDSA/EdDSA key: Sign Authenticate
Current allowed actions: Sign

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? Q
Please select which elliptic curve you want:
   (1) Curve 25519
   (3) NIST P-256
   (4) NIST P-384
   (5) NIST P-521
   (9) secp256k1
Your selection? 1
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 2y
Key expires at Sat 16 Nov 2019 03:26:17 PM UTC
Is this correct? (y/N) y
Really create? (y/N) y
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

sec  ed25519/09F33B41FEDB1EE9
     created: 2017-11-16  expires: 2022-11-15  usage: C
     trust: ultimate      validity: ultimate
ssb  ed25519/59DA6BD32790C898
     created: 2017-11-16  expires: 2019-11-16  usage: S
[ultimate] (1). John Smith <john@example.com>

gpg> save
```



#### 增加加密用子 Key

确保 Current allowed actions: Encrypt

```
gpg> addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
Your selection? 8

Possible actions for a RSA key: Sign Encrypt Authenticate
Current allowed actions: Sign Encrypt

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? S

Possible actions for a RSA key: Sign Encrypt Authenticate
Current allowed actions: Encrypt

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? Q
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 2y
Key expires at Sat 16 Nov 2019 03:30:55 PM UTC
Is this correct? (y/N) y
Really create? (y/N) y
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

sec  ed25519/09F33B41FEDB1EE9
     created: 2017-11-16  expires: 2022-11-15  usage: C
     trust: ultimate      validity: ultimate
ssb  ed25519/59DA6BD32790C898
     created: 2017-11-16  expires: 2019-11-16  usage: S
ssb  rsa4096/9A9F21B268173541
     created: 2017-11-16  expires: 2019-11-16  usage: E
[ultimate] (1). John Smith <john@example.com>

gpg> save
```



#### 增加认证用子 Key

```
gpg> addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
Your selection? 11

Possible actions for a ECDSA/EdDSA key: Sign Authenticate
Current allowed actions: Sign

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? S

Possible actions for a ECDSA/EdDSA key: Sign Authenticate
Current allowed actions:

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? A

Possible actions for a ECDSA/EdDSA key: Sign Authenticate
Current allowed actions: Authenticate

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? Q
Please select which elliptic curve you want:
   (1) Curve 25519
   (3) NIST P-256
   (4) NIST P-384
   (5) NIST P-521
   (9) secp256k1
Your selection? 1
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 2y
Key expires at Sat 16 Nov 2019 03:39:00 PM UTC
Is this correct? (y/N) y
Really create? (y/N) y
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

sec  ed25519/09F33B41FEDB1EE9
     created: 2017-11-16  expires: 2022-11-15  usage: C
     trust: ultimate      validity: ultimate
ssb  ed25519/59DA6BD32790C898
     created: 2017-11-16  expires: 2019-11-16  usage: S
ssb  rsa4096/9A9F21B268173541
     created: 2017-11-16  expires: 2019-11-16  usage: E
ssb  ed25519/2FE99B61DB34CDAF
     created: 2017-11-16  expires: 2019-11-16  usage: A
[ultimate] (1). John Smith <john@example.com>

gpg> save
```

至此密钥生成完成。



#### 我该使用多少位的 RSA 密钥？

根据[官方文档](https://www.gnupg.org/faq/gnupg-faq.html#default_rsa2048)，2048bit 的密钥理论上可以安全使用至 2020 年，并且在默认创建的密钥中使用了 2048bit 的密钥，也没有推荐或者反对使用更长的密钥。

尽管如此，我们仍然强烈建议在使用 RSA 密钥的时候选择更长的密钥，强烈推荐使用 4096bit 的密钥。



### 加密／解密

#### 加密

```bash
gpg2 --encrypt --sign --recipient john@example.com --armor --output /tmp/very-secret-message.gpg /tmp/clear-text.txt
```



#### 解密

```bash
gpg2 --decrypt --output /tmp/clear-text.txt /tmp/very-secret-message.gpg
```





### 发布与签名

#### 发布你的公钥

```bash
gpg2 --send-keys 0x09F33B41FEDB1EE9
```



#### 签名一个 Key

验证签名信息

签名方

```bash
gpg2 --list-public-keys --with-icao-spelling 0x09F33B41FEDB1EE9
```

被签名方

```bash
gpg2 --list-secret-keys --with-icao-spelling 0x09F33B41FEDB1EE9
```

签名

```bash
gpg2 --sign-key john@example.com
```

发送签名信息

```bash
gpg2 --send-keys john@example.com
```

验证用签名

```bash
gpg2 --lsign-key john@example.com
```





### 密钥管理

#### 生成吊销证书

```bash
gpg2 --output gpg-0x09F33B41FEDB1EE9.asc --gen-revoke 0x09F33B41FEDB1EE9
```

务必保证这一吊销证书保存在一个安全的地方，这一文件将能直接吊销你的密钥。如果可能的话，将其手写在纸上（一些打印系统甚至会记录你打印的东西），并保存在一个私密的地方。

####  密钥存储

##### 在线存储

主密钥保存在本地，要求你必须信任你在本地安装的所有软件。

非常不安全

##### 离线存储

* 智能卡（YubiKey、GnuK 等等）
* USB 闪存、Tails
* 一台独立的、离线的电脑

##### 删除主密钥的私钥

首先备份整个 `~/.gnupg` 文件夹到 U 盘上。

如果你的 GnuPG 版本 > 2.1，那么只需要删除 `$HOME/.gnupg/private-keys-v1.d/KEYGRIP.key`

如果你的 GnuPG 版本 < 2.1，你需要导出所有子密钥

```bash
gpg2 --output secret_subkeys --export-secret-subkeys YOURMASTERKEYID
```

删除主密钥私钥

```bash
gpg --delete-secret-keys YOURMASTERKEYID
```

导入子密钥私钥

```bash
gpg --import secret-subkeys
```

删除导出的文件 `secret-subkeys`

此时可以检查 `gpg2 --list-keys` ，私钥前的 `sec` 已经变成 `sec#`。表明主密钥私钥并不真正保存在此处。

此时还可以使用 `gpg --edit-key YOURMASTERKEYID passwd` 修改密码，防止日常使用中密码泄漏影响主密钥的安全。



## 总结

* 再小心都不为过
* 务必保证私钥安全
* Take responsibility to your action.




## 参考文献与延伸阅读

[https://phab.enlightenment.org/w/gnupg/](https://phab.enlightenment.org/w/gnupg/)

[Debian Wiki: Subkeys](https://wiki.debian.org/Subkeys)

[GnuPG manpage](https://www.gnupg.org/gph/de/manual/r1023.html)

[The GNU Privacy Handbook](https://www.gnupg.org/gph/en/manual/book1.html)

[An Advanced Intro to GnuPG](https://begriffs.com/posts/2016-11-05-advanced-intro-gnupg.html)

[Public-key cryptography - Wikipedia](https://en.wikipedia.org/wiki/Public-key_cryptography)

[Symmetric-key algorithm - Wikipedia](https://en.wikipedia.org/wiki/Symmetric-key_algorithm)

[Pretty Good Privacy - Wikipedia](https://en.wikipedia.org/wiki/Pretty_Good_Privacy)

[GNU Privacy Guard - Wikipedia](https://en.wikipedia.org/wiki/GNU_Privacy_Guard)

Applied Cryptography: Protocols, Algorithms, and Source Code in C, Bruce Schneier, Wiley
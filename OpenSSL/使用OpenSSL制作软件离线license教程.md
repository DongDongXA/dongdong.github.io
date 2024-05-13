# 使用OpenSSL制作软件离线license教程

[TOC]

## 软件License简介

### 什么是License？

在软件公司向客户销售商业软件的时候，常常需要对所发布的软件实行一系列管控措施，诸如验证使用者身份、软件是否到期，以及保存版权信息和开发商详情等。考虑到诸多应用场景可能处于离线环境，无法依赖网络进行实时认证，所以还需要考虑单机认证时的防破解问题。

总之，License许可证利用Https网站的证书和签名技术，一方面证明当前使用者是申请License的本人，另一方面要防止恶意破解，并伪造篡改License达到白嫖的目的。

### 为什么使用License授权？

License的作用是什么呢？收费软件的License其目的肯定是防止用户白嫖啦，所以License还应该具有以下一些功能：

- 授权使用

明确用户需要满足的使用条件，如单用户、多用户、企业内部使用、全球使用等，并且通常会限定可安装和激活的设备数量。

- 限制功能

根据不同等级的License，软件可以提供不同等级的功能，例如基础版、专业版、企业版等，License可以解锁相应版本的功能。

- 期限控制

规定软件的使用期限，可能是永久授权，也可能是订阅式授权，到期后用户需要续费才能继续使用。

- 版权保护

重申软件的知识产权归属，禁止未经授权的复制、分发、反编译、篡改或逆向工程等侵犯版权的行为。

- 法律保障

License作为法律合同，确立了软件提供商和用户之间的法律关系，明确了双方的权利和责任，如果发生违反协议的情况，软件提供商有权采取法律手段追究责任。

### License分类

基于使用的期限，License可以划分为固定期限License和永久License两类。

固定期限License在激活后的指定时间内有效，过了预设的使用期限，用户必须更新许可期限并通过重新激活才能继续使用；

而永久License则是在激活后赋予用户无时间限制的使用权，一旦激活，无需担忧许可失效的问题，可以无限期地持续使用软件。

### 如何实现License授权？

要想生成一个安全性高的License，必须让其满足以下几个特征：

- 保密性

- 防篡改
- 时效性
- 可找回

保密性是指License里携带的data信息具有一定的隐蔽性，这样可以防止想要破解License的人寻找到生成License的规律，进而伪造自己的License。

防篡改是指防止License里携带的重要信息被篡改，例如License有效时间如果被篡改，那么License就起不到限制用户使用期限的作用了。

时效性是指License会记录软件可以使用的有效期，并在验证License的时候判断其是否过期。

可找回是指用户申请的License一旦丢失或者要续期，基于第一次申请License时创建的源文件，再一次生成新的License，新的License会携带用户当初申请时的信息。

### 军工产品软件License的设计思路

军工产品的软件与自身运行的硬件设备是高度绑定的，由于军工装备的保密性设备无法访问外部互联网，属于离线场景；由于军工装备的可靠性方面要求，License没有时效性的要求，属于永久License；由于军工设备硬件环境的高度定制性，所以软件License的设计在兼容性方面要求并不高；军工设备硬件很多设备都是嵌入式芯片，算力较低，但由于保密性要求严格，应采用信息密度高，解密速度快，破解难度大的算法。

综上所述，最佳实践方案是一台设备绑定一个License，这样一台设备的License文件如果被复制到另外一台设备上，由于并非同一台设备，相同的程序依然无法被正常使用和破解。设备和License之间产生的一一映射关系应该是设备的UUID信息或者与UUID相关的校验信息被加密包含在License文件里，这样设备可以通过验证比较UUID信息本身或者校验信息来确认License是否合法有效。

## OpenSSL简介

### 背景技术

**SSL**是Secure Sockets Layer（安全套接字层）的缩写，可以在Internet上提供秘密性传输。Netscape公司在推出第一个Web浏览器的同时，提出了SSL协议标准：该标准的目标是保证两个应用间通信的保密性和可靠性，可在服务器端和用户端同时实现支持，已经成为Internet上保密通讯的工业标准。

SSL能使用户/服务器应用之间的通信不被攻击者窃听，并且始终对服务器进行认证，还可选择对用户进行认证。SSL协议要求建立在可靠的传输层协议TCP之上。SSL协议的优势在于它是与应用层协议独立无关的，高层的应用层协议(例如：HTTP，FTP，TELNET等)能透明地建立于SSL协议之上。SSL协议在应用层协议通信之前就已经完成加密算法、通信密钥的协商及服务器认证工作。在此之后应用层协议所传送的数据都会被加密，从而保证通信的私密性。

### 基本功能

**OpenSSL**是一个实现SSL协议的开放源代码的软件库，应用程序可以使用这个包来进行安全通信，避免窃听，同时确认另一端连接者的身份，这个包广泛被应用在互联网的网页服务器上。

作为一个基于密码学的安全开发包，OpenSSL提供的功能相当强大和全面，囊括了主要的密码算法、常用的密钥和证书封装管理功能以及SSL协议，并提供了丰富的应用程序供测试或其它目的使用。

密码算法部分是一个强大完整的密码算法库，它是OpenSSL的基础部分，也是很值得一般密码安全技术人员研究的部分，它实现了目前大部分主流的密码算法和标准。主要包括对称算法、非对称算法、散列算法、数字签名和认证、X509数字证书标准、PKCS12、PKCS7等标准。其他两个功能部分SSL协议和应用程序都是基于这个库开发的。

SSL协议部分完全实现和封装了SSL协议的三个版本和TLS协议。使用协议库，你完全可以建立一个SSL服务器和SSL客户端。

应用程序是基于密码算法库和SSL协议库实现的命令，应用程序覆盖了密码技术的应用，主要包括了各种算法的加密程序和各种类型密钥的产生程序(如RSA、Md5、Enc等等)、证书签发和验证程序（如Ca、X509、Crl等）、SSL连接测试程序（如S_client和S_server等）以及其它的标准应用程序（如Pkcs12和Smime等）。

### 为什么使用OpenSSL来制作软件License

1. OpenSSL是一个强大的安全套接字层密码库，Apache使用它加密HTTPS，OpenSSH使用它加密SSH，但是，你不应该只将其作为一个库来使用，它还是一个多用途的、跨平台的密码工具。
2. OpenSSL包含一个命令行工具来完成OpenSSL库中的所有功能，而且它在大部分主流Linux内核系统中是默认内置安装好的，这使得我们可以通过执行调用OpenSSL工具的脚本对目标用户机快速生成符合要求的License。
3. OpenSSL采用C语言作为开发语言，这使得OpenSSL具有优秀的跨平台性能，它支持Linux、Windows、BSD、Mac等平台，这使得OpenSSL具有广泛的适用性，这使得我们可以在客户端使用OpenSSL开发License的验证接口供需要License保护的软件包调用。

## 离线License设计

### 背景理论

License本质是一组具有保密性的数据，能够验证License拥有者的身份合法性，但同时对外隐藏自己的数据加密方法和组成逻辑防止被他人得到该数据后破解生成同类数据伪装成License合法拥有者以通过License审核。因此，License设计的核心问题在于数据加密算法和身份验证算法是否足够安全能抵御破解攻击。这就涉及到一门古老的学科——密码学。

密码，最初的目的是用于对信息加密，计算机领域的密码技术种类繁多。但随着密码学的运用，密码还被用于身份认证、防止否认等功能上。密码是通信双方按约定的法则进行信息特殊变换的一种重要保密手段。依照这些法则，变明文为密文，称为加密变换；变密文为明文，称为脱密变换。密码在早期仅对文字或数码进行加、脱密变换，随着通信技术的发展，对语音、图像、数据等都可实施加、脱密变换。

密码学是一门古老而深奥的学科，只在很小的范围内使用，如军事、外交、情报等部门。计算机密码学是研究计算机信息加密、解密及其变换的科学，是数学和计算机的交叉学科 ，也是一门新兴的学科。密码学是网络空间安全主要研究方向之一，也是许多安全机制的基础。

密码学术语

> - 消息(Message)：消息是指用语言、文字、数字、符号、图像、声音或其组合等方式记载或传递的有意义的内容。在密码学里，消息也称为信息 。
> - 明文(Plaintext)：未经过任何伪装或隐藏技术处理的消息称为明文。
> - 加密(Encryption)：利用某些方法或技术对明文进行伪装或隐藏的过程称为加密。
> - 密文(Cipher Text)：被加密的消息称为密文。
> - 解密(Decryption)：将密文恢复成原明文的过程或操作称为解密，解密也可称为脱密。
> - 加密算法(Encryption Algorithm)：将明文消息加密成密文所采用的一组规则或数学函数。
> - 解密算法(Decryption Algorithm)：将密文消息解密成明文所采用的一组规则或数学函数。
> - 密钥(Key)：进行加密或解密操作所需要的秘密参数或关键信息 。在密码系统中，密钥分为私钥(Private-key)与公钥(Public-key)两种。私钥指必须保密的密钥，公钥指可以向外界公开的密钥。
> - 密码系统 (Cryptosystem)：一个密码体制或密码系统是指由明文空间、密文空间、密钥空间、加密算法以及解密算法 组成的一个多元素集合体。
> - 对称加密(Symmetric-key algorithm)：数据通信双方共同定义一把密钥，一方用密钥加密，另一方用密钥解密。这种数据加密操作方式又被称为：共享密钥加密、单一密钥加密。总体来说这种加密方式速度快、效率高、计算小也是目前比较常用的数据加密方式。但是一旦出现一方的密钥被泄密，就凉凉了加密的数据信息也就不安全了。
> - 非对称加密(Public-key cryptography)：这种加密算法需要公钥和私钥两个密钥对数据进行加密解密处理。这种数据加密操作方式又被称为：公开密钥加密。一方用公钥或者私钥加密后，另外一方必须用对应的私钥或者公钥进行解密，如果是用公钥加密公钥解密或者私钥加密私钥解密这是完全行不通的，只能是公钥加密私钥解密或者私钥加密公钥解密，正因为是这种数据公私加解的方式才被称之为非对称加密，在现实应用场景中，秘密信息收取方给发送方提供一个公钥，发送方利用公钥对自己的秘密信息进行加密，收取方收到密文后用私钥对密文进行解密获得原始信息。
> - 数字签名(Digital Signature)：附加在某一电子文档中的一组特定的符号或代码。它利用非对称加密技术对该电子文档进行关信息提取并进行认证形成，用于标识签发者的身份以及签发者对电子文档的认可，并能被接收者用来验证该电子文档在传输过程中是否被篡改或伪造。在这种使用非对称加密技术的场景中，使用的是私钥加密，公钥解密。

### 典型License范例参考与分析

[keygen.sh](https://keygen.sh/)是一个经典的license功能开发SDK，但由于它使用Go语言开发，属于互联网服务器颁发License的使用场景，我们无法直接使用它来进行License功能的开发，但这并不妨碍我们可以对它的官网文档里关于离线License的例子进行参考分析：

> [keygen.sh/Offline Lisensing](https://keygen.sh/docs/api/cryptography/)
>
> > **Each Keygen account is equipped with multiple unique public/private keypairs**, which are used for example, in signing response payloads using RSA-SHA256, and which can also be used to sign or encrypt license files and license keys, providing powerful options for offline license verification as well as activation.
> >
> > You can find your account's **public keys** within your [dashboard settings page](https://app.keygen.sh/settings), which you can use to `verify` response payloads, webhooks and license keys, and license files. Private keys are kept securely encrypted on our servers and never shared.
> >
> > The algorithm used for license file/key signing and encryption will depend on the [cryptographic scheme](https://keygen.sh/docs/api/policies/#policies-object-attrs-scheme) that a particular license's policy implements.

从这里可以分析出它介绍他们自己官网SDK的License使用了非对称加密和数字签名机制，使用的算法是RSA-SHA256。

> [keygen.sh/Cryptographic license files](https://keygen.sh/docs/api/cryptography/#cryptographic-lic)
>
> > Keygen's API supports a couple different license file types, namely ["license" files](https://keygen.sh/docs/api/licenses/#licenses-actions-check-out) and ["machine" files](https://keygen.sh/docs/api/machines/#machines-actions-check-out), representing a snapshot of a license or a machine, respectively, at the time of checkout. For brevity, we'll refer to both of these as license files, as they are largely the same. Each license file consists of an encoded certificate.
> >
> > All license files have a time-to-live (TTL) that must be respected. Once a license file's TTL has been met, i.e. it has expired, a new one should be checked out. We recommend starting out with a TTL of 30 days (which is the default), and going up or down from there based on your use case. A license file's expiry is separate from the license's expiry.
> >
> > > **Why even set a time-to-live?** In short: *eventual consistency*. During checkout, a license file can be set to expire after a certain amount of time (default 30 days) via a TTL. This gives room for any changes made to the license to propagate into subsequent license files. A good way to think of these would be in terms of a Let's Encrypt TLS certificate, having a short expiry and then re-upping after e.g. 90 days.
> >
> > For most applications, we recommend setting an expiry, so that license files are *eventually* consistent when it comes to changes in its license's state, e.g. an update, renewal, expiration, or suspension event, since each time a new license file is checked out, a new snapshot of the license is taken.
> >
> > Below, you will find information on each certificate's format, and how to decode a license file certificate and cryptographically verify its contents.
> >
> > Want to see an example? Check out these examples on our GitHub:
> >
> > - [Verifying and decrypting machine files in Python](https://github.com/keygen-sh/example-python-cryptographic-machine-files)
> >
> > - [Verifying and decrypting license files in Rust](https://github.com/keygen-sh/example-rust-cryptographic-license-files)
> >
> > - [Verifying and decrypting license files in Java](https://github.com/keygen-sh/example-java-cryptographic-license-files)
> >
> > - [Verifying and decrypting license files in C++](https://github.com/keygen-sh/example-cpp-cryptographic-license-files)
> >
> > - [Verifying and decrypting license files in C#](https://github.com/keygen-sh/example-csharp-cryptographic-license-files)
> >
> > - [Verifying and decrypting license files in Dart](https://github.com/keygen-sh/example-dart-cryptographic-license-files)
> >
> > - [Implementing air-gapped activation in Node](https://github.com/keygen-sh/air-gapped-activation-example)
> >
> > - [Utilizing license files with our Go SDK](https://github.com/keygen-sh/keygen-go)
> >
>

Keygen SDK的API支持两类License文件，一种是license文件，一种是machine文件，这两类文件差别很小，都支持过期检测更新（我们场景不需要的功能），并提供了一堆代码范例。

> [License file features](https://keygen.sh/docs/api/cryptography/#cryptographic-lic-format)
>
> > ###  [License file disposition](https://keygen.sh/docs/api/cryptography/#cryptographic-lic-format)
> >
> > We recommend saving license files with the `.lic` file extension. This is a widely used file extension for licensing purposes, and it's perfect for our license files. These files can be distributed in offline or air-gap environments using email or USB drives.
> >
> > When downloading a license file, the `Content-Disposition` header will contain our recommended filename. But feel free to choose your own.
> >
> > ### [License file format](https://keygen.sh/docs/api/cryptography/#cryptographic-lic-format)
> >
> > License file certificates consist of a few parts:
> >
> > - The certificate header.
> > - The certificate body, an encoded payload.
> > - The certificate footer.
> >
> > #### Certificate header
> >
> > The header of the certificate will be as below, where `<type>` is either `LICENSE` or `MACHINE`, depending on the checked out resource.
> >
> >```text
> >-----BEGIN <type> FILE-----
> >```
> >
> > The header will end in a newline.
> >
> >#### Certificate body
> >
> > Between the header and footer, you will find the base64 encoded certificate payload.
> >
> >```text
> >-----BEGIN MACHINE FILE-----
> >eyJlbmMiOiJ0S3JYUmw4NCtDSTJ6WDVpN1NSUWJqdkt1d1BpRWFhZDdudlF2
> >bWNVVnd2L3B2ajEvcDdLb0NzWVVvbWJzT0tQdkwyS0xUcTBhNUtKeGpLRmNP
> >RG1oc3JUR2NSNUF4bktoSVRJWTA0THFJelJzNW9tZzRpbWRDSVpZZWdzbzdK
> >Rk9ISDVZaFBnaEFYK1FkK0U4aDUzZWlkNXM4KzVXL0xwN1h2ZUJVamhHeEFZ
> >SjhTb3VwOFZpOXQwTmlDSHlaL1hIMDRNM0YwSm83MmI0b1VXSWlQT3NNcm9n
> >VVhudllTckk1TU9abWNhZWlIN1NtNTlDWWozeUVhOE1BcW0xOTdCWktEU0k3
> >cDdUV2l2VW5FeHprRUhFN2xaUHVleDdGbCtHaURQRHhGbjlkamRRQ3hyamp2
> >OWRVUG0rdXkzS0Rzb0RwRjZWMWp4OU8wVWNpNFdZZzlUaWNoVm5KK0Yxd3g4
> >aFJmQ25lU3QvSmo3SHVXSDZ4MC9DS0ZHczFZU0NicUU5VmMxaHFDWTdYNCsw
> >KzdYQmNhZWpRSituUU5TemVkdXNya0JTa0NEcm1PUHpzUW9JNzJFUWUxS20x
> >dkY0MlRxbHBXUzB3aFVFSml2a1VlcmZqVFQ4aGpPZTlJZEFzTEphQytqS3F3
> >S2hmQ0QxVERqZVQ4QnMwT0JLOXpZV0JUbmlDY2Q1bUlUSFVySWZDanhmOGNh
> >MDdmaW1TdnV0UTdCN3FNNlMvWEJ6VWhqNXAvNlJ2OU9TQS9pcDFkT0RuWHhm
> >K2FOempTM1planB1Q2M3UGkzM0ltejZabCtZUWl0cklnMUpaaXhmZ0l0aGpr
> >VS9BakVqaW1PekdxcS9NTjlSdjIwTUk3RWRrWE40Z3VhNGFjMGJNQzNQQnhs
> >a3BZK0hMQW1HSWlxckJZSVAzVDRMNzBFMk5JRkl5bldtKzk2aHdKazBVR1RQ
> >cUlVdCt1QmIzdWRueWhuS3h6bnR5eXR1eTBuNzVOaVpndlFONE14MUc5dWlF
> >a2xIOWVQN2Jjbk5PWGEvRW9Mdm9NKy9NZDNZc01KdlkzZ09Sd2w0R0FFZ1Bv
> >Z3VMZTc4L3R3T1dsU3NSS2ZrWlM1QW1GdWd2cU13dmdwOVhzT3NwOFRPa1F0
> >MEtCdHVZejFFSjh5TmthSzBReXMwMUNjWDUzelJHZDJYc0dKMEo0TTBnR2FX
> >QVBJZndVTm5iSThnRTV6eEZ0T2lGdXZXMTRML1JHRXdJZUxVaXhhbkVpa28z
> >TjMrMHdhdGhjTzlVcGJreXR1VlpFOGprR0t3OHRZdGdyaFRDalc0bllEQW5O
> >b2Z2Tm9hUkt2ZHp2RzEvODg0SDZuVWhYbEw5ZVBvaE1pZXJVOE9iYWhIQTR4
> >T0dnQjhGQXp6QVhndGtaK3ZFRUpBOUl5Y2x5QWdzWjR3elVHMEpZcDF6bEhL
> >RmdjMjA4eldxeUcxdW9VR3czTEtMZjRjZklNRlBkS2ozOWpCNms1TVFSOXc4
> >UnBmSkJvU2pnemt3eS9GSnhVcnk0Ni9Pdlk4VDNSMGFxb0VOZHVrRHVvOUF2
> >a1N3VmY2WkRadjR4Qk1FVmFJUFRxbUxIc1pyZmkwcTdRa3RjdklzNHpmSEZK
> >SW15MnZHQndMb0ZzOGhteXNOOVQzbnB2MzFiM0ZaVmQ3VUR4SmxTR21qa281
> >enNaNEVFeDFEUjhHRzMzYjg5Q1Yxc09GUytBWWxXem9iM0sza3NnREt5NzhY
> >Qk9ucEttY3ZKZnNvWm83ZSt2dTZtOHAzQzVzak53UVJycUJJb1Z1K0czbWIv
> >U2dvTHRkWExZcGlEaE5pTzVFS3hDVSsvUCtNcEFPbERRU211cy85N0VadlRP
> >R05RVDZGWkVjWEpocUl0Zm9DemFYZGQxenJYamhXbGpCNng3Q0F0Y3Fudkl5
> >MlZ0UHplSlB6Z2hFSzZlZ1VyL2MybDBHU3ZVNjNmTnFxUzZoQzJpZUZaWTg0
> >T2ptZ1hKb1BZU2YxODAwTlRlMUkyRzdrZi9LaTh0S2FqMVVONndDQ1Y0RDk0
> >d3B1WElJdVZiaXl0U1ArYktudGtiQ1NUZ1ZpZEwyZHdZTnE0Y2o1bHdTM0xn
> >R1BpVnpQZDNPMWs3V2tlemlpRVJ0K0xKNTFNbHZIVS9kVkI2RTZIREY2dFVt
> >UHpHaDhaSTFUTGhOUU1GV3pSMWJUaFhzdFRuZ2FrR3ZKc0I4WFd1Y1R5RGg5
> >cXdnYlhWYytqNmFQZ0FBb3M0enh3VVowdTJTV2lGWHpVTWxlUUtqWTM4VGk4
> >dFp1ZTlac1ZBUEJVUFo2TTVSV0dEQVZ1OGttVno3VjFVTkhEQS9OTTA5dGZS
> >aFVkUjVJNlhlRXZ0ZWJFdmM1YktNNnFXOTBId3V0QWx2b0dXcmFpak9qbGti
> >VmZZV2d3RlloYmg0eEdJL2NoYkR0OWtBcVFIMDhPUCsyZVhTbmxZWU1GSVdI
> >Qk8rK1g5WGd0RktmK2dJb3N5K2hKZjVIekhZL3hGWHFiVzJzWnE4U0lZaUhW
> >RTVWMXA1cGVKbUNHaVZnTEV0YUZSU0o1bWxWeUxWdldLSmdIRG9SWUlPaFR4
> >WFUvVnkxM3hXVmF3c0dTa2s0NDU5bHZoVEt1bUhIK3o4aTkyOG9EY2dsWnk0
> >bENtbTFGTUhEQ3dHMUkzUjJNbzR3ZkQ1MXhVNHVCTW1iR0dBckowSlBJZHNV
> >ZUZUeEFpS2lLV25KRzBrbGF5cWNUR0huTGR0QUY0TWJ0R0N5YytsMldzZ0xh
> >cFZlOE9ablJ1MWZUY1M3TGE0RjZudUdrMVpxeVhJeEhlU3RVUlFWOVZ0ZFI3
> >ZGwzcHFsaXh1eDZHTTRBNVhMbVp1NG9DeWtqU1c5SGp2MFhQdjFwdnFsemtq
> >b2xoTjF6RzlHbmJVQ2c2cExmMGJ4VFcwdlJSV0kwWU9NZm1kNkZMeURRZ1ZE
> >YkY3L2UwVS9jRXllRHZTWkM0KzZCcTFjdFhTZlZ4NDFqaHBMV3E2eG9WT3hl
> >NHg3UXZreEEzM3JkdFVYeWNEa0RneFNDQ1lSZzFxckFWOUx5SjdRMlIwOVZs
> >cnd5YitMWm1RQkVRMGRxdlVmNVB0UU1LQVRnWXlvclo2cEFTWmI4QUdMeis3
> >QzNXTjJHbnNoR1dwZUlCV2RzSGJFTXZnR0ZTUGt1NlpCaDR3MVlTRm44NUwr
> >a3FGNnJTTUVscVRtdjcyVVJRdTRyNE85L0FSa3VOMHFYRmZNMldVNlpTUTkz
> >TzR4STdpbW92SWMyWktaUHFKOG5nRUR4UEIyMFhpaUFFSmM2Y0tCUUNmYTA1
> >SnlRYWt2Z0ZnQ0gvTDl3YkQxcUJMVDRRSTJ3d0M0TnlYcHlHNm9mMWpaVElN
> >SFprVVBhMDlwYVNnVXZvVDRTaUtYZXBObG41dzdIWHdVY1JMQjBRUzVvUHZL
> >OXBoYW0zcTBMbHlFMHVkUFhINEkvdEFWZThIcDJwQWRiUkl5c0E0dHJ1bUFv
> >VS9EN3E5czZVc0JiMW02azBwQ0w1Z3ZLV3Z2M096YnBGOXRNbEFhYW12ZnFG
> >a1dIanNQK2pwZnVoeGhsL3llTEl3ZU5lZjJmTmxIOVNtWjExbzVsL3lpVTJ1
> >QWVrd0dabGVQUmx2QzgrS3ovd3o4ejgxUllLVnVWSGNvcVc2dkY3OEJadkFa
> >VDgvbVZyQzR5TElpcXp3NkhvYmpQWnhsNW5rRXhVSHdLalRsRXdPWExWNFJu
> >dlY1ZitjZlk3Qnh1RGhCdlpGU2dEb0NiUmpOK2JIOTRxM3A3SDQzUlV4SG9N
> >azY4M1pOMmZCM09nNW1CSUZxOFA0SU94c0dSWGltcDZvR1FzQTcxMGdxV1Z1
> >YWdVbWJ5YWhjWWRsQW1majVlWC8zV2RCN3VOdm5idDViZmlTL05GS21LTWxy
> >T2NoSUNLUU56bU1IZ2xPZW95SzA3ZlAwUVZLc2hsZEJhZzNicXF0MSsxU3hi
> >akFwVTN3UHNjakVuSVBhUjBUemhrcFVuMm5rR25iWDhxSVVOcURsMlkyc3RI
> >S3V5VmEybUpZaUVLVm1sdE9GakYyVm92ZFZPY3lqT3lTOWFlUTduY0tlZTdp
> >c1E1RC9NdURlK091aWhLY3RzLzRobnRCMHV3a0x5SVZKdE5xSHgwQzlxVE9D
> >NEtNbjdlcGlDT0ltdjBHRVdaQkZyS2kvOXJEbE1FOXBnWTZPTU5ySXZhK1Br
> >UWVRQllmRXo1bnAzTzJNZm5zS1B2dHVjamF2OWg5TlQyUjkwcVBDSXkzcjl5
> >YUFZV1hTR1cyTlZtNFdXcENnM05jdjFDTDhZTEMwR1I5OFI3LzgwdUxEQ1A0
> >a04wRG1PLzFZN2wxb2VXVlVVY0ZvTnFBSmo2NCtGS1BsL21CUDNYY0FDMWhE
> >cmlENTlDYVFNWVhkOGdXVkJiRzVYeEpIdU5mNmdvZjRidEhFVUM3UHdaUjVp
> >cDROai9OdUh4Vk4rVVhselVFUmh1UTR2a2daWkFpVE9IbE5Pc3ZlLzJmNTlt
> >VkhMSnpDRTF3Wlc1c09TZlVuN2Zwb2hzUUMrRVROdHhRemdURkU3N1NFUDVi
> >dFdSZytFWDNmU1Fibjh6LzBiektUYTc3VFpmdzYrL2g3SXN6UmNCTDVZOFN5
> >UzZnbGxjbmtoVTVLZjVwc0dJc0JXaklDMjhwQnhMMkhqaWF4ZlowVkpKTVNF
> >MWkzSFVQU29RbmdUaFdxckQvd3dzN1Fjb280VVdrbWF4VlF5Z3k4MWRaZlhr
> >amdMcmxvQVhodSthb0lxa0I2ZkxYbTUxT2JUK3dEVHQ3SHpwVkRTb0VDamEy
> >VUR3MDRPNXNOcitDR3p6aGc0LzdYZTQvZWhQbW85MmhyeVFPRUFaK3pveVRp
> >TGx1bWhtbE1zMUhTeHRxSGw2TU93VnZNV3lEVW9BWXNCc3MzTWpGazk4b0hM
> >RTFIajVCMUNxYTBKbk9kQXF2UE0wOUZ0UjZxdlVRSFVLbE9mRnhuY2xlSlpm
> >WnJreVBlV0tCeld6RHhYL1hGR1BpT3JldUVLeE9hZWRyaUQ5STNHalg4clpk
> >Y08xUUVmWlFCN1NQdkc2Q25XZUVzWDFWV2RNclVTQlJLaldUQkVlRlVjdmdH
> >QzhRamVFS1RlcXpuTFpkZFh5MmdpVnl5ejdMSW1kOWVNUkJpOHNmbzZYSTgr
> >TWsyOXJUZksvNGZJd0tub0ZtNVdCS1hSWENvalJMVTV1MGw1MUwyVmtkZHNR
> >VUlzZjc1UlFIZkxKUVZEODY3elJJM2JaZTJOUkRMOEtQQU95bUVMd3hKMmRq
> >OURJa1N2WXJSR3kxOGE4VXdzRVNrS1RtZFRyYlhRQ0ZCYUMxeXlhTng1dUJo
> >dU85eTJSMlEzRWE0NGJlQmFXaHRZVm9uQ0ZQM01WSTQ5bXI2MzNyM3F3QlAz
> >RW1yMHBrck9aczA0NUNDa3diTktLdmlYREtBbm1FcUduRmxBMmNtQnl2TURP
> >NktadEpyR2wvUWFqZ05wck1PQnNycE5wWFZjdVNhRVIrT0JFQXR1WnNXYUZT
> >T1RUaFFVVWRBbFN3NWN4N3BnL2xTRlV5aWdudFBBWVVjVllzM3N1NEZpU0Zl
> >L0UwT3Qzek5CL2wzUHIyb1g1bHFYNG9mVEZEZ2NySm5FVDBmU1ZhaW5zR3Zy
> >TXNtTXV5UHhMcjJiSTJWWEIxMXNWNDhKQmR5aXhDNGxWSTZEUCtWNWs2MW01
> >Qzd4bEFPYUFtSXVLWHBkdjROV3dkRXdrZmk2d3ZnSjluMGh4K09ORk1GT20y
> >S0ZoR1NFeEpxQ1JNOXREa255elNtZWN4YnRqRllkdGp1bm4vL2kvTFNZdVhh
> >bzFWQUFXcUVPNEloVXcwQUxJM0Z2ZUt3TDhUdWxIL05OWEpkVXJUOGh3Rllu
> >ak1RMC9yYWhMTmpqNkEzeVB5YU1aWnFpYjJLbVlQcnRBOWxSc1pDN1h6SEJx
> >cVJrTUMxVjlFRnJEcjBsYjNaeUNWUzlVYzNWZmUvWlhwSjlmZXBhQUlpb096
> >OEt3N1JaUlZERVNmS3VFN2dYY2RsYkF6V2x2UjhWR3k5OWswMUo1Vms0Q2wy
> >NS9mMmR1UERHUmgvMzR2V2hNbGhpc3A3bjE2VjM0a3NrcFgra2JkTjVwcC94
> >UGc1M2VWcHVIVWNheHlhQ3pLWHdTZUlFY2xQQTM2SFJ5SGwrU2o5aUEvS0Fn
> >NWJ2enVIQU9RVm1nNkVxVml3aE1Ub1pJRUlaZE85K1pGZ2pnd0U0LzVlOTMv
> >bXN6RC8vazdSU2RkdkZKaTJTdjMvV2xwMHhkTUFIbkQwM0dWNTdNaFEzelRw
> >V1ZnRGlscmo0RzA0R2dCVjBGUkd6dUxqMDNQNy9iZk84Z08zOUFScEh1Tlo1
> >cjJmT2dVcTllOWZYTGNGS0tDdFltbk5iLy85NmljQzJ5QUhqUGR6NHlac1gz
> >T085TEFYek5wOVJWK3RrU0p0S1hZRjh6Wk1rWTg5WmZEbEh5QkFleDByVFlZ
> >ZlVVUS84OXFCR1Uwd2g3WVhGdWtPMUVqMkhCN0ZIb1lxSlNyMG5SeVFLMlJw
> >SDBTeEhHR1NwK2lnaWpMbHp5VkJTVFZwL2EvdUpuZHRvOHhhOGJXZ3BiTkhn
> >ZE9XaHNWMDRLQ0hFNWlXTS9BQ1RwVW9EOVU5QzdPZzRBPS5sTWhWTEpOV1VF
> >S3lHcGt4LmMyUmhDbEJZWGY0SVRmcEV0WHRUdmc9PSIsInNpZyI6ImlHMGh2
> >Y1VaeGUwU3BDZWlNWi95aldFcEFnV2t6Ly9vV2FNRmFkUjRmWGd1UkozTGZO
> >K3FIOCt4eFhhRmRoSS8rbVFvdTdnZzRSRzdxSVUwdjRoUERRPT0iLCJhbGci
> >OiJhZXMtMjU2LWdjbStlZDI1NTE5In0=
> >-----END MACHINE FILE-----
> >```
> >
> >The encoded certificate payload has a newline every 80 characters. These may need to be stripped from the string before base64 decoding, depending on programming language support. Once base64 decoded, you now have a [license file payload](https://keygen.sh/docs/api/cryptography/#cryptographic-lic-payload).
> >
> >#### Certificate footer
> >
> >The footer of the certificate will be as below, where `<type>` is either `LICENSE` or `MACHINE`, depending on the checked out resource.
> >
> >```text
> >-----END <type> FILE-----
> >```
> >
> >The footer will end in a newline.

提炼一下要点，离线License文件的业界设计惯例：

1. 文件都是lic扩展名
2. 文件都分为三个部分：文件头，文件数据体，文件尾
3. 文件数据体为了保证数据文本可打印性都进行了base64编码

> **base64编码**
>
> 针对ASCII字符部分的不可打印性问题，互联网信息传输中常用的Base64是一种基于65个可打印字符来表示二进制八位数据的方法，3n个八进制字节可以被4n个base64字符表示，3n+1个八进制字节可以被4n+4个base64字符（结尾两个补位符）表示，3n+2个八进制字节可以被4n+4个base64字符（结尾一个补位符）表示。
>
> | **索引** | 二进制 | **对应字符** | **索引** | 二进制 | **对应字符** | **索引** | 二进制 | **对应字符** | **索引** | 二进制 | **对应字符** |
> | -------- | ------ | ------------ | -------- | ------ | ------------ | -------- | ------ | ------------ | -------- | ------ | ------------ |
> | 0        | 000000 | **A**        | 17       | 010001 | **R**        | 34       | 100010 | **i**        | 51       | 110011 | **z**        |
> | 1        | 000001 | **B**        | 18       | 010010 | **S**        | 35       | 100011 | **j**        | 52       | 110100 | **0**        |
> | 2        | 000010 | **C**        | 19       | 010011 | **T**        | 36       | 100100 | **k**        | 53       | 110101 | **1**        |
> | 3        | 000011 | **D**        | 20       | 010100 | **U**        | 37       | 100101 | **l**        | 54       | 110110 | **2**        |
> | 4        | 000100 | **E**        | 21       | 010101 | **V**        | 38       | 100110 | **m**        | 55       | 110111 | **3**        |
> | 5        | 000101 | **F**        | 22       | 010110 | **W**        | 39       | 100111 | **n**        | 56       | 111000 | **4**        |
> | 6        | 000110 | **G**        | 23       | 010111 | **X**        | 40       | 101000 | **o**        | 57       | 111001 | **5**        |
> | 7        | 000111 | **H**        | 24       | 011000 | **Y**        | 41       | 101001 | **p**        | 58       | 111010 | **6**        |
> | 8        | 001000 | **I**        | 25       | 011001 | **Z**        | 42       | 101010 | **q**        | 59       | 111011 | **7**        |
> | 9        | 001001 | **J**        | 26       | 011010 | **a**        | 43       | 101011 | **r**        | 60       | 111100 | **8**        |
> | 10       | 001010 | **K**        | 27       | 011011 | **b**        | 44       | 101100 | **s**        | 61       | 111101 | **9**        |
> | 11       | 001011 | **L**        | 28       | 011100 | **c**        | 45       | 101101 | **t**        | 62       | 111110 | **+**        |
> | 12       | 001100 | **M**        | 29       | 011101 | **d**        | 46       | 101110 | **u**        | 63       | 111111 | **/**        |
> | 13       | 001101 | **N**        | 30       | 011110 | **e**        | 47       | 101111 | **v**        | 补位     |        |              |
> | 14       | 001110 | **O**        | 31       | 011111 | **f**        | 48       | 110000 | **w**        |          | 缺两位 | =            |
> | 15       | 001111 | **P**        | 32       | 100000 | **g**        | 49       | 110001 | **x**        |          | 缺四位 | ==           |
> | 16       | 010000 | **Q**        | 33       | 100001 | **h**        | 50       | 110010 | **y**        |          |        |              |

我们将示例license保存为文件machine.lic，使用sed工具将license第一行license头和最后一行license尾去除后得到的内容也就是License的数据主体，使用openssl命令行工具进行base64解码，我们发现解码结果是一个json结构的数据，于是我们使用jq工具对其进行json结构化显示得到如下结果。

```shell
$ sed "1d; $d" examples/machine.lic | openssl base64 -d | jq
#解码结果
{
  "enc": "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.lMhVLJNWUEKyGpkx.c2RhClBYXf4ITfpEtXtTvg==",
  "sig": "iG0hvcUZxe0SpCeiMZ/yjWEpAgWkz//oWaMFadR4fXguRJ3LfN+qH8+xxXaFdhI/+mQou7gg4RG7qIU0v4hPDQ==",
  "alg": "aes-256-gcm+ed25519"
}
```

从以上结果可以看出所得json结构中enc元素和sig元素字段依然存储的是类似于base64格式字符串数据，其中enc元素存储的是三个base64格式字符串用**.**符号分隔，sig元素存储的是一个数字签名数据，alg元素中指出license的验证算法是aes-256-gcm加ed25519，aes-256-gcm是一个对称加密并可以验证数据完整性的对称加密算法，ed25519是一个轻量级的椭圆曲线curve25519被主要作为数字签名验证使用的算法。由此我们可以推断出，sig元素存储的应该是enc元素的数据经过ed25519算法处理的数字签名数据。

为了验证我们的推断，我们来对Keygen SDK离线License制作文档中一个关于machine文件验证的python项目进行分析：

> [keygen-sh/example-python-cryptographic-machine-files: Example of verifying cryptographically signed and encrypted machine files using Python, Ed25519 and AES-256-GCM.](https://github.com/keygen-sh/example-python-cryptographic-machine-files)
>
> >## Example Python Cryptographic Machine Files
> >
> >
> >
> >This is an example of how to verify and decrypt [cryptographic machine files](https://keygen.sh/docs/api/cryptography/#cryptographic-lic) in Python, using Ed25519 signing and AES-256-GCM encryption. This example verifies the `aes-256-gcm+ed25519` algorithm.
> >
> >### Running the example
> >
> >
> >
> >First up, add an environment variable containing your Ed25519 public key:
> >
> >```bash
> >export KEYGEN_PUBLIC_KEY='799efc7752286e6c3815b13358d98fc0f0b566764458adcb48f1be2c10a55906'
> >```
> >
> >
> >
> >You can either run each line above within your terminal session before starting the app, or you can add the above contents to your `~/.bashrc` file and then run `source ~/.bashrc` after saving the file.
> >
> >Next, install dependencies with [`pip`](https://packaging.python.org/):
> >
> >```bash
> >python3 -m pip install -r requirements.txt
> >```
> >
> >
> >
> >Then run the script, passing in a `path` to a machine file, and a `license` key as arguments:
> >
> >```bash
> >python3 main.py --license 'A_LICENSE_KEY' \
> >--path /etc/keygen/machine.lic
> >```
> >
> >
> >
> >Or run one of the pre-defined examples:
> >
> >```bash
> >KEYGEN_PUBLIC_KEY='e8601e48b69383ba520245fd07971e983d06d22c4257cfd82304601479cee788' python3 main.py \
> >--fingerprint '198e9fe586114844f6a4eaca5069b41a7ed43fb5a2df84892b69826d64573e39' \
> >--license 'B10760-1B177D-656D1F-C03298-9AF89E-V3' \
> >--path examples/machine.lic
> >```
> >
> >
> >
> >The following will happen:
> >
> >1. The current device will be fingerprinted, using a SHA256-HMAC digest of the device's [machineid](https://github.com/keygen-sh/py-machineid). You may want to look into alternative fingerprinting, depending on your expected run environment.
> >2. The machine file's authenticity will be verified using Ed25519, by verifing its signature using the public key.
> >3. The machine file will be decrypted using the license key and fingerprint as the decryption key.
> >
> >If everything checks out, the script will print the decrypted contents of the machine file — the machine object, with any included data. If it fails, check your inputs.
> >
> >### Questions?
> >
> >
> >
> >Reach out at [support@keygen.sh](mailto:support@keygen.sh) if you have any questions or concerns!
>
> [main.py](https://github.com/keygen-sh/example-python-cryptographic-machine-files/blob/master/main.py)
>
> ```python
> from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
> from cryptography.hazmat.primitives import hashes
> from cryptography.hazmat.backends import default_backend
> from cryptography.exceptions import InvalidKey, InvalidTag
> from ed25519 import BadSignatureError
> import machineid
> import argparse
> import ed25519
> import base64
> import json
> import sys
> import os
> 
> parser = argparse.ArgumentParser()
> 
> parser.add_argument('-p', '--path', dest='path', required=True, help='Path to machine file (required)')
> parser.add_argument('-l', '--license', dest='license', required=True, help='License key (required)')
> parser.add_argument('-f', '--fingerprint', dest='fingerprint', default=machineid.hashed_id('example-app'), help='Machine fingerprint')
> 
> args = parser.parse_args()
> 
> # Read the machine file
> machine_file = None
> 
> try:
>     with open(args.path) as f:
>        machine_file = f.read()
> except (FileNotFoundError, PermissionError):
>     print('[error] path does not exist! (or permission was denied)')
> 
>     sys.exit(1)
> 
> # Strip the header and footer from the machine file certificate
> payload = machine_file.removeprefix('-----BEGIN MACHINE FILE-----\n') \
>                          .removesuffix('-----END MACHINE FILE-----\n')
> 
> # Decode the payload and parse the JSON object
> data = json.loads(base64.b64decode(payload))
> 
> # Retrieve the enc and sig properties
> enc = data['enc']
> sig = data['sig']
> alg = data['alg']
> 
> if alg != 'aes-256-gcm+ed25519':
>     print('[error] algorithm is not supported!')
> 
>     sys.exit(1)
> 
> # Verify using Ed25519
> try:
>     verify_key = ed25519.VerifyingKey(
>        os.environ['KEYGEN_PUBLIC_KEY'].encode(),
>        encoding='hex',
>     )
> 
>     verify_key.verify(
>        base64.b64decode(sig),
>        ('machine/%s' % enc).encode(),
>     )
> except (AssertionError, BadSignatureError):
>     print('[error] verification failed!')
> 
>     sys.exit(1)
> 
> print('[info] verification successful!')
> 
> # Hash the license key and fingerprint using SHA256
> digest = hashes.Hash(hashes.SHA256(), default_backend())
> digest.update(args.license.encode())
> digest.update(args.fingerprint.encode())
> key = digest.finalize()
> 
> # Split and decode the enc value
> ciphertext, iv, tag = map(
>     lambda p: base64.b64decode(p),
>     enc.split('.'),
> )
> 
> # Decrypt ciphertext
> try:
>     aes = Cipher(
>        algorithms.AES(key),
>        modes.GCM(iv, None, len(tag)),
>        default_backend(),
>     )
>     dec = aes.decryptor()
> 
>     plaintext = dec.update(ciphertext) + \
>                  dec.finalize_with_tag(tag)
> except (InvalidKey, InvalidTag):
>     print('[error] decryption failed!')
> 
>     sys.exit(1)
> 
> print('[info] decryption successful!')
> print(
>     json.dumps(json.loads(plaintext.decode()), indent=2)
> )
> ```

该python程序所展示的完整流程图如下：

```mermaid
graph TD;

macFile(Machine文件)
fileHead(文件头)
fileTail(文件尾)
fileBody(文件数据体)
jsonMachine(Json数据)
jsonEnc(enc)
jsonSig(sig)
jsonAlg("alg<br>(aes-256-gcm+ed25519)")
signature(License数字签名)
ed25519Key("ed25519公钥<br>(硬编码于设备程序中)")
machineStr("字符串&quotmachine/&quot")
machineInfo(设备信息)
sampleKeyWords(采样关键字)
machineFingerPrint(机器数字指纹)
licenseOK(License验证成功)
licenseFail(License验证失败)
gcmCipherText(aes-gcm<br>密文)
gcmPlainText(aes-gcm<br>明文)
gcmIV(gcm初始化变量)
gcmTag(gcm验证数据)
licenseKey(license序列号)
aesgcmKey(aes-gcm-256秘钥)
machineInfo -->|HMAC加密| machineFingerPrint;
sampleKeyWords -->|HMAC加密| machineFingerPrint;

macFile---fileHead;
macFile---fileBody;
macFile---fileTail;
jsonEnc-->|debase64|gcmCipherText;
jsonEnc-->|debase64|gcmIV;
jsonEnc-->|debase64|gcmTag;
fileBody-->|debase64|jsonMachine;
jsonMachine---jsonEnc;
jsonMachine---jsonSig;
jsonMachine---jsonAlg;
subgraph ed25519Str[ed25519数字签名文本]
machineStr ---|+| jsonEnc
end
subgraph hash256Str[SHA256加密数据]
licenseKey ---|+| machineFingerPrint
end 
subgraph aesgcmDecryption[aes-gcm解密]
gcmCipherText;
gcmIV;
gcmTag;
aesgcmKey;
end

hash256Str-->|SHA256|aesgcmKey;
jsonSig-->|debase64|signature;
ed25519Key -->|验证|signature;
ed25519Str -->|验证|signature;
signature -->|验证成功|aesgcmDecryption;
signature -->|验证失败|licenseFail;
aesgcmDecryption -->|解密失败|licenseFail;
aesgcmDecryption -->|解密成功|gcmPlainText;
gcmPlainText -->|信息非法|licenseFail;
gcmPlainText -->|信息合法|licenseOK;
```

从该程序中我们可以看出，暴露给用户的离线License数据主要包含两个数据——Machine文件和License序列号，至于设备信息，采样关键字的提取策略以及ed25519公钥必须保密，否则License的生成策略就会有破解的风险。

### Keygen.sh项目的精简逆向工程

我们将Keygen.sh中的所有步骤进行逆处理就是一套设备的完整软件License生成过程，为了避免不必要的加密策略泄露，我们删除掉Json数据结构中关于加密算法的说明也就是alg部分。

```mermaid
graph TD;

macFile(Machine文件)
fileHead(文件头)
fileTail(文件尾)
fileBody(文件数据体)
jsonMachine(Json数据)
jsonEnc(enc)
jsonSig(sig)
signature(License数字签名)
ed25519Key("ed25519私钥<br>(只能用于License颁发程序)")
machineStr("字符串&quotmachine/&quot")
machineInfo(设备信息)
sampleKeyWords(采样关键字)
machineFingerPrint(机器数字指纹)
uuidTool(uuidgen<br>工具)
gcmCipherText(aes-gcm<br>密文)
gcmPlainText(aes-gcm<br>明文)
gcmIV(gcm初始化变量)
gcmTag(gcm验证数据)
licenseKey(license序列号)
aesgcmKey(aes-gcm-256秘钥)

subgraph ed25519Str[ed25519数字签名文本]
machineStr ---|+| jsonEnc
end
subgraph aesgcmStr[aes-gcm加密数据]
gcmPlainText;
gcmIV;
aesgcmKey;
end
subgraph hash256Str[SHA256加密数据]
licenseKey ---|+| machineFingerPrint
end 
ed25519Key -->|生成|signature;
ed25519Str -->|生成|signature;
aesgcmStr -->|加密生成|gcmCipherText;
aesgcmStr -->|加密生成|gcmTag;

machineInfo -->|HMAC加密| machineFingerPrint;
sampleKeyWords -->|HMAC加密| machineFingerPrint;

fileHead---macFile;
fileBody---macFile;
fileTail---macFile;
gcmCipherText-->|base64|jsonEnc;
gcmIV-->|base64|jsonEnc;
gcmTag-->|base64|jsonEnc;
jsonMachine-->|base64|fileBody;
jsonEnc---jsonMachine;
jsonSig---jsonMachine;
%%jsonAlg---jsonMachine;
hash256Str-->|SHA256|aesgcmKey;
signature-->|base64|jsonSig;
uuidTool-->|生成|licenseKey;
uuidTool-->|生成|gcmIV;
```

uuidgen工具可以随机生成license序列号和gcm的初始化变量，openssl的命令行工具可以生成ed25519的数字签名、base64数据转化、hmac加密、sha256算法的加密，aes-gcm的加密模块无法通过openssl的命令行工具直接调用，但是可以通过编写程序来实现完成，以下是openssl的aes-gcm范例demo。

> [openssl/demos/cipher/aesgcm.c](https://github.com/openssl/openssl/blob/openssl-3.0/demos/cipher/aesgcm.c)
>
> ```c
> /*
>  * Copyright 2012-2021 The OpenSSL Project Authors. All Rights Reserved.
>  *
>  * Licensed under the Apache License 2.0 (the "License").  You may not use
>  * this file except in compliance with the License.  You can obtain a copy
>  * in the file LICENSE in the source distribution or at
>  * https://www.openssl.org/source/license.html
>  */
> 
> /*
>  * Simple AES GCM authenticated encryption with additional data (AEAD)
>  * demonstration program.
>  */
> 
> #include <stdio.h>
> #include <openssl/err.h>
> #include <openssl/bio.h>
> #include <openssl/evp.h>
> #include <openssl/core_names.h>
> 
> /* AES-GCM test data obtained from NIST public test vectors */
> 
> /* AES key */
> static const unsigned char gcm_key[] = {
>     0xee, 0xbc, 0x1f, 0x57, 0x48, 0x7f, 0x51, 0x92, 0x1c, 0x04, 0x65, 0x66,
>     0x5f, 0x8a, 0xe6, 0xd1, 0x65, 0x8b, 0xb2, 0x6d, 0xe6, 0xf8, 0xa0, 0x69,
>     0xa3, 0x52, 0x02, 0x93, 0xa5, 0x72, 0x07, 0x8f
> };
> 
> /* Unique initialisation vector */
> static const unsigned char gcm_iv[] = {
>     0x99, 0xaa, 0x3e, 0x68, 0xed, 0x81, 0x73, 0xa0, 0xee, 0xd0, 0x66, 0x84
> };
> 
> /* Example plaintext to encrypt */
> static const unsigned char gcm_pt[] = {
>     0xf5, 0x6e, 0x87, 0x05, 0x5b, 0xc3, 0x2d, 0x0e, 0xeb, 0x31, 0xb2, 0xea,
>     0xcc, 0x2b, 0xf2, 0xa5
> };
> 
> /*
>  * Example of Additional Authenticated Data (AAD), i.e. unencrypted data
>  * which can be authenticated using the generated Tag value.
>  */
> static const unsigned char gcm_aad[] = {
>     0x4d, 0x23, 0xc3, 0xce, 0xc3, 0x34, 0xb4, 0x9b, 0xdb, 0x37, 0x0c, 0x43,
>     0x7f, 0xec, 0x78, 0xde
> };
> 
> /* Expected ciphertext value */
> static const unsigned char gcm_ct[] = {
>     0xf7, 0x26, 0x44, 0x13, 0xa8, 0x4c, 0x0e, 0x7c, 0xd5, 0x36, 0x86, 0x7e,
>     0xb9, 0xf2, 0x17, 0x36
> };
> 
> /* Expected AEAD Tag value */
> static const unsigned char gcm_tag[] = {
>     0x67, 0xba, 0x05, 0x10, 0x26, 0x2a, 0xe4, 0x87, 0xd7, 0x37, 0xee, 0x62,
>     0x98, 0xf7, 0x7e, 0x0c
> };
> 
> /*
>  * A library context and property query can be used to select & filter
>  * algorithm implementations. If they are NULL then the default library
>  * context and properties are used.
>  */
> OSSL_LIB_CTX *libctx = NULL;
> const char *propq = NULL;
> 
> int aes_gcm_encrypt(void)
> {
>     int ret = 0;
>     EVP_CIPHER_CTX *ctx;
>     EVP_CIPHER *cipher = NULL;
>     int outlen, tmplen;
>     size_t gcm_ivlen = sizeof(gcm_iv);
>     unsigned char outbuf[1024];
>     unsigned char outtag[16];
>     OSSL_PARAM params[2] = {
>         OSSL_PARAM_END, OSSL_PARAM_END
>     };
> 
>     printf("AES GCM Encrypt:\n");
>     printf("Plaintext:\n");
>     BIO_dump_fp(stdout, gcm_pt, sizeof(gcm_pt));
> 
>     /* Create a context for the encrypt operation */
>     if ((ctx = EVP_CIPHER_CTX_new()) == NULL)
>         goto err;
> 
>     /* Fetch the cipher implementation */
>     if ((cipher = EVP_CIPHER_fetch(libctx, "AES-256-GCM", propq)) == NULL)
>         goto err;
> 
>     /* Set IV length if default 96 bits is not appropriate */
>     params[0] = OSSL_PARAM_construct_size_t(OSSL_CIPHER_PARAM_AEAD_IVLEN,
>                                             &gcm_ivlen);
> 
>     /*
>      * Initialise an encrypt operation with the cipher/mode, key, IV and
>      * IV length parameter.
>      * For demonstration purposes the IV is being set here. In a compliant
>      * application the IV would be generated internally so the iv passed in
>      * would be NULL. 
>      */
>     if (!EVP_EncryptInit_ex2(ctx, cipher, gcm_key, gcm_iv, params))
>         goto err;
> 
>     /* Zero or more calls to specify any AAD */
>     if (!EVP_EncryptUpdate(ctx, NULL, &outlen, gcm_aad, sizeof(gcm_aad)))
>         goto err;
> 
>     /* Encrypt plaintext */
>     if (!EVP_EncryptUpdate(ctx, outbuf, &outlen, gcm_pt, sizeof(gcm_pt)))
>         goto err;
> 
>     /* Output encrypted block */
>     printf("Ciphertext:\n");
>     BIO_dump_fp(stdout, outbuf, outlen);
> 
>     /* Finalise: note get no output for GCM */
>     if (!EVP_EncryptFinal_ex(ctx, outbuf, &tmplen))
>         goto err;
> 
>     /* Get tag */
>     params[0] = OSSL_PARAM_construct_octet_string(OSSL_CIPHER_PARAM_AEAD_TAG,
>                                                   outtag, 16);
> 
>     if (!EVP_CIPHER_CTX_get_params(ctx, params))
>         goto err;
> 
>     /* Output tag */
>     printf("Tag:\n");
>     BIO_dump_fp(stdout, outtag, 16);
> 
>     ret = 1;
> err:
>     if (!ret)
>         ERR_print_errors_fp(stderr);
> 
>     EVP_CIPHER_free(cipher);
>     EVP_CIPHER_CTX_free(ctx);
> 
>     return ret;
> }
> 
> int aes_gcm_decrypt(void)
> {
>     int ret = 0;
>     EVP_CIPHER_CTX *ctx;
>     EVP_CIPHER *cipher = NULL;
>     int outlen, rv;
>     size_t gcm_ivlen = sizeof(gcm_iv);
>     unsigned char outbuf[1024];
>     OSSL_PARAM params[2] = {
>         OSSL_PARAM_END, OSSL_PARAM_END
>     };
> 
>     printf("AES GCM Decrypt:\n");
>     printf("Ciphertext:\n");
>     BIO_dump_fp(stdout, gcm_ct, sizeof(gcm_ct));
> 
>     if ((ctx = EVP_CIPHER_CTX_new()) == NULL)
>         goto err;
> 
>     /* Fetch the cipher implementation */
>     if ((cipher = EVP_CIPHER_fetch(libctx, "AES-256-GCM", propq)) == NULL)
>         goto err;
> 
>     /* Set IV length if default 96 bits is not appropriate */
>     params[0] = OSSL_PARAM_construct_size_t(OSSL_CIPHER_PARAM_AEAD_IVLEN,
>                                             &gcm_ivlen);
> 
>     /*
>      * Initialise an encrypt operation with the cipher/mode, key, IV and
>      * IV length parameter.
>      */
>     if (!EVP_DecryptInit_ex2(ctx, cipher, gcm_key, gcm_iv, params))
>         goto err;
> 
>     /* Zero or more calls to specify any AAD */
>     if (!EVP_DecryptUpdate(ctx, NULL, &outlen, gcm_aad, sizeof(gcm_aad)))
>         goto err;
> 
>     /* Decrypt plaintext */
>     if (!EVP_DecryptUpdate(ctx, outbuf, &outlen, gcm_ct, sizeof(gcm_ct)))
>         goto err;
> 
>     /* Output decrypted block */
>     printf("Plaintext:\n");
>     BIO_dump_fp(stdout, outbuf, outlen);
> 
>     /* Set expected tag value. */
>     params[0] = OSSL_PARAM_construct_octet_string(OSSL_CIPHER_PARAM_AEAD_TAG,
>                                                   (void*)gcm_tag, sizeof(gcm_tag));
> 
>     if (!EVP_CIPHER_CTX_set_params(ctx, params))
>         goto err;
> 
>     /* Finalise: note get no output for GCM */
>     rv = EVP_DecryptFinal_ex(ctx, outbuf, &outlen);
>     /*
>      * Print out return value. If this is not successful authentication
>      * failed and plaintext is not trustworthy.
>      */
>     printf("Tag Verify %s\n", rv > 0 ? "Successful!" : "Failed!");
> 
>     ret = 1;
> err:
>     if (!ret)
>         ERR_print_errors_fp(stderr);
> 
>     EVP_CIPHER_free(cipher);
>     EVP_CIPHER_CTX_free(ctx);
> 
>     return ret;
> }
> 
> int main(int argc, char **argv)
> {
>     if (!aes_gcm_encrypt())
>         return 1;
> 
>     if (!aes_gcm_decrypt())
>         return 1;
> 
>     return 0;
> }
> ```
>
> [openssl/demos/signature/EVP_ED_Signature_demo.c](https://github.com/openssl/openssl/blob/openssl-3.0/demos/signature/EVP_ED_Signature_demo.c)
>
> ```c
> /*-
>  * Copyright 2023 The OpenSSL Project Authors. All Rights Reserved.
>  *
>  * Licensed under the Apache License 2.0 (the "License").  You may not use
>  * this file except in compliance with the License.  You can obtain a copy
>  * in the file LICENSE in the source distribution or at
>  * https://www.openssl.org/source/license.html
>  */
> 
> /*
>  * This demonstration will calculate and verify an ED25519 signature of
>  * a message using  EVP_DigestSign() and EVP_DigestVerify().
>  */
> 
> #include <string.h>
> #include <stdio.h>
> #include <openssl/err.h>
> #include <openssl/evp.h>
> #include <openssl/core_names.h>
> 
> /* A test message to be signed (TBS) */
> static const unsigned char hamlet[] =
>     "To be, or not to be, that is the question,\n"
>     "Whether tis nobler in the minde to suffer\n"
>     "The slings and arrowes of outragious fortune,\n"
>     "Or to take Armes again in a sea of troubles,\n";
> 
> static int demo_sign(EVP_PKEY *priv,
>                      const unsigned char *tbs, size_t tbs_len,
>                      OSSL_LIB_CTX *libctx,
>                      unsigned char **sig_out_value,
>                      size_t *sig_out_len)
> {
>     int ret = 0;
>     size_t sig_len;
>     unsigned char *sig_value = NULL;
>     EVP_MD_CTX *sign_context = NULL;
> 
>     /* Create a signature context */
>     sign_context = EVP_MD_CTX_new();
>     if (sign_context == NULL) {
>         fprintf(stderr, "EVP_MD_CTX_new failed.\n");
>         goto cleanup;
>     }
> 
>     /*
>      * Initialize the sign context using an ED25519 private key
>      * Notice that the digest name must NOT be used.
>      * In this demo we don't specify any additional parameters via
>      * OSSL_PARAM, which means it will use default values.
>      * For more information, refer to doc/man7/EVP_SIGNATURE-ED25519.pod
>      * "ED25519 and ED448 Signature Parameters"
>      */
>     if (!EVP_DigestSignInit_ex(sign_context, NULL, NULL, libctx, NULL, priv, NULL)) {
>         fprintf(stderr, "EVP_DigestSignInit_ex failed.\n");
>         goto cleanup;
>     }
> 
>     /* Calculate the required size for the signature by passing a NULL buffer. */
>     if (!EVP_DigestSign(sign_context, NULL, &sig_len, tbs, tbs_len)) {
>         fprintf(stderr, "EVP_DigestSign using NULL buffer failed.\n");
>         goto cleanup;
>     }
>     sig_value = OPENSSL_malloc(sig_len);
>     if (sig_value == NULL) {
>         fprintf(stderr, "OPENSSL_malloc failed.\n");
>         goto cleanup;
>     }
>     fprintf(stdout, "Generating signature:\n");
>     if (!EVP_DigestSign(sign_context, sig_value, &sig_len, tbs, tbs_len)) {
>         fprintf(stderr, "EVP_DigestSign failed.\n");
>         goto cleanup;
>     }
>     *sig_out_len = sig_len;
>     *sig_out_value = sig_value;
>     BIO_dump_indent_fp(stdout, sig_value, sig_len, 2);
>     fprintf(stdout, "\n");
>     ret = 1;
> 
> cleanup:
>     if (!ret)
>         OPENSSL_free(sig_value);
>     EVP_MD_CTX_free(sign_context);
>     return ret;
> }
> 
> static int demo_verify(EVP_PKEY *pub,
>                        const unsigned char *tbs, size_t tbs_len,
>                        const unsigned char *sig_value, size_t sig_len,
>                        OSSL_LIB_CTX *libctx)
> {
>     int ret = 0;
>     EVP_MD_CTX *verify_context = NULL;
> 
>     /*
>      * Make a verify signature context to hold temporary state
>      * during signature verification
>      */
>     verify_context = EVP_MD_CTX_new();
>     if (verify_context == NULL) {
>         fprintf(stderr, "EVP_MD_CTX_new failed.\n");
>         goto cleanup;
>     }
>     /* Initialize the verify context with a ED25519 public key */
>     if (!EVP_DigestVerifyInit_ex(verify_context, NULL, NULL,
>                                  libctx, NULL, pub, NULL)) {
>         fprintf(stderr, "EVP_DigestVerifyInit_ex failed.\n");
>         goto cleanup;
>     }
>     /*
>      * ED25519 only supports the one shot interface using EVP_DigestVerify()
>      * The streaming EVP_DigestVerifyUpdate() API is not supported.
>      */
>     if (!EVP_DigestVerify(verify_context, sig_value, sig_len,
>                           tbs, tbs_len)) {
>         fprintf(stderr, "EVP_DigestVerify() failed.\n");
>         goto cleanup;
>     }
>     fprintf(stdout, "Signature verified.\n");
>     ret = 1;
> 
> cleanup:
>     EVP_MD_CTX_free(verify_context);
>     return ret;
> }
> 
> static int create_key(OSSL_LIB_CTX *libctx,
>                       EVP_PKEY **privout, EVP_PKEY **pubout)
> {
>     int ret = 0;
>     EVP_PKEY *priv = NULL, *pub = NULL;
>     unsigned char pubdata[32];
>     size_t pubdata_len = 0;
> 
>     /*
>      * In this demo we just create a keypair, and extract the
>      * public key. We could also use EVP_PKEY_new_raw_private_key_ex()
>      * to create a key from raw data.
>      */
>     priv = EVP_PKEY_Q_keygen(libctx, NULL, "ED25519");
>     if (priv == NULL) {
>         fprintf(stderr, "EVP_PKEY_Q_keygen() failed\n");
>         goto end;
>     }
> 
>     if (!EVP_PKEY_get_octet_string_param(priv,
>                                          OSSL_PKEY_PARAM_PUB_KEY,
>                                          pubdata,
>                                          sizeof(pubdata),
>                                          &pubdata_len)) {
>         fprintf(stderr, "EVP_PKEY_get_octet_string_param() failed\n");
>         goto end;
>     }
>     pub = EVP_PKEY_new_raw_public_key_ex(libctx, "ED25519", NULL, pubdata, pubdata_len);
>     if (pub == NULL) {
>         fprintf(stderr, "EVP_PKEY_new_raw_public_key_ex() failed\n");
>         goto end;
>     }
>     ret = 1;
> end:
>     if (ret) {
>         *pubout = pub;
>         *privout = priv;
>     } else {
>         EVP_PKEY_free(priv);
>     }
>     return ret;
> }
> 
> int main(void)
> {
>     OSSL_LIB_CTX *libctx = NULL;
>     size_t sig_len = 0;
>     unsigned char *sig_value = NULL;
>     int ret = EXIT_FAILURE;
>     EVP_PKEY *priv = NULL, *pub = NULL;
> 
>     libctx = OSSL_LIB_CTX_new();
>     if (libctx == NULL) {
>         fprintf(stderr, "OSSL_LIB_CTX_new() returned NULL\n");
>         goto cleanup;
>     }
>     if (!create_key(libctx, &priv, &pub)) {
>         fprintf(stderr, "Failed to create key.\n");
>         goto cleanup;
>     }
> 
>     if (!demo_sign(priv, hamlet, sizeof(hamlet), libctx,
>                    &sig_value, &sig_len)) {
>         fprintf(stderr, "demo_sign failed.\n");
>         goto cleanup;
>     }
>     if (!demo_verify(pub, hamlet, sizeof(hamlet),
>                      sig_value, sig_len, libctx)) {
>         fprintf(stderr, "demo_verify failed.\n");
>         goto cleanup;
>     }
>     ret = EXIT_SUCCESS;
> 
> cleanup:
>     if (ret != EXIT_SUCCESS)
>         ERR_print_errors_fp(stderr);
>     EVP_PKEY_free(pub);
>     EVP_PKEY_free(priv);
>     OSSL_LIB_CTX_free(libctx);
>     OPENSSL_free(sig_value);
>     return ret;
> }
> ```
>
> [openssl/demos/mac/hmac-sha512.c](https://github.com/openssl/openssl/blob/openssl-3.0/demos/mac/hmac-sha512.c)
>
> ```c
> /*-
>  * Copyright 2022-2023 The OpenSSL Project Authors. All Rights Reserved.
>  *
>  * Licensed under the Apache License 2.0 (the "License").  You may not use
>  * this file except in compliance with the License.  You can obtain a copy
>  * in the file LICENSE in the source distribution or at
>  * https://www.openssl.org/source/license.html
>  */
> 
> /*
>  * Example of using EVP_MAC_ methods to calculate
>  * a HMAC of static buffers
>  */
> 
> #include <string.h>
> #include <stdio.h>
> #include <openssl/crypto.h>
> #include <openssl/core_names.h>
> #include <openssl/err.h>
> #include <openssl/evp.h>
> #include <openssl/hmac.h>
> #include <openssl/params.h>
> 
> /*
>  * Hard coding the key into an application is very bad.
>  * It is done here solely for educational purposes.
>  */
> static unsigned char key[] = {
>     0x25, 0xfd, 0x12, 0x99, 0xdf, 0xad, 0x1a, 0x03,
>     0x0a, 0x81, 0x3c, 0x2d, 0xcc, 0x05, 0xd1, 0x5c,
>     0x17, 0x7a, 0x36, 0x73, 0x17, 0xef, 0x41, 0x75,
>     0x71, 0x18, 0xe0, 0x1a, 0xda, 0x99, 0xc3, 0x61,
>     0x38, 0xb5, 0xb1, 0xe0, 0x82, 0x2c, 0x70, 0xa4,
>     0xc0, 0x8e, 0x5e, 0xf9, 0x93, 0x9f, 0xcf, 0xf7,
>     0x32, 0x4d, 0x0c, 0xbd, 0x31, 0x12, 0x0f, 0x9a,
>     0x15, 0xee, 0x82, 0xdb, 0x8d, 0x29, 0x54, 0x14,
> };
> 
> static const unsigned char data[] =
>     "To be, or not to be, that is the question,\n"
>     "Whether tis nobler in the minde to suffer\n"
>     "The ſlings and arrowes of outragious fortune,\n"
>     "Or to take Armes again in a sea of troubles,\n"
>     "And by opposing, end them, to die to sleep;\n"
>     "No more, and by a sleep, to say we end\n"
>     "The heart-ache, and the thousand natural shocks\n"
>     "That flesh is heir to? tis a consumation\n"
>     "Devoutly to be wished. To die to sleep,\n"
>     "To sleepe, perchance to dreame, Aye, there's the rub,\n"
>     "For in that sleep of death what dreams may come\n"
>     "When we haue shuffled off this mortal coil\n"
>     "Must give us pause. There's the respect\n"
>     "That makes calamity of so long life:\n"
>     "For who would bear the Ships and Scorns of time,\n"
>     "The oppressor's wrong, the proud man's Contumely,\n"
>     "The pangs of dispised love, the Law's delay,\n"
> ;
> 
> /* The known value of the HMAC/SHA3-512 MAC of the above soliloqy */
> static const unsigned char expected_output[] = {
>     0x3b, 0x77, 0x5f, 0xf1, 0x4f, 0x9e, 0xb9, 0x23,
>     0x8f, 0xdc, 0xa0, 0x68, 0x15, 0x7b, 0x8a, 0xf1,
>     0x96, 0x23, 0xaa, 0x3c, 0x1f, 0xe9, 0xdc, 0x89,
>     0x11, 0x7d, 0x58, 0x07, 0xe7, 0x96, 0x17, 0xe3,
>     0x44, 0x8b, 0x03, 0x37, 0x91, 0xc0, 0x6e, 0x06,
>     0x7c, 0x54, 0xe4, 0xa4, 0xcc, 0xd5, 0x16, 0xbb,
>     0x5e, 0x4d, 0x64, 0x7d, 0x88, 0x23, 0xc9, 0xb7,
>     0x25, 0xda, 0xbe, 0x4b, 0xe4, 0xd5, 0x34, 0x30,
> };
> 
> /*
>  * A property query used for selecting the MAC implementation.
>  */
> static const char *propq = NULL;
> 
> int main(void)
> {
>     int ret = EXIT_FAILURE;
>     OSSL_LIB_CTX *library_context = NULL;
>     EVP_MAC *mac = NULL;
>     EVP_MAC_CTX *mctx = NULL;
>     EVP_MD_CTX *digest_context = NULL;
>     unsigned char *out = NULL;
>     size_t out_len = 0;
>     OSSL_PARAM params[4], *p = params;
>     char digest_name[] = "SHA3-512";
> 
>     library_context = OSSL_LIB_CTX_new();
>     if (library_context == NULL) {
>         fprintf(stderr, "OSSL_LIB_CTX_new() returned NULL\n");
>         goto end;
>     }
> 
>     /* Fetch the HMAC implementation */
>     mac = EVP_MAC_fetch(library_context, "HMAC", propq);
>     if (mac == NULL) {
>         fprintf(stderr, "EVP_MAC_fetch() returned NULL\n");
>         goto end;
>     }
> 
>     /* Create a context for the HMAC operation */
>     mctx = EVP_MAC_CTX_new(mac);
>     if (mctx == NULL) {
>         fprintf(stderr, "EVP_MAC_CTX_new() returned NULL\n");
>         goto end;
>     }
> 
>     /* The underlying digest to be used */
>     *p++ = OSSL_PARAM_construct_utf8_string(OSSL_MAC_PARAM_DIGEST, digest_name,
>                                             sizeof(digest_name));
>     *p = OSSL_PARAM_construct_end();
> 
>     /* Initialise the HMAC operation */
>     if (!EVP_MAC_init(mctx, key, sizeof(key), params)) {
>         fprintf(stderr, "EVP_MAC_init() failed\n");
>         goto end;
>     }
> 
>     /* Make one or more calls to process the data to be authenticated */
>     if (!EVP_MAC_update(mctx, data, sizeof(data))) {
>         fprintf(stderr, "EVP_MAC_update() failed\n");
>         goto end;
>     }
> 
>     /* Make a call to the final with a NULL buffer to get the length of the MAC */
>     if (!EVP_MAC_final(mctx, NULL, &out_len, 0)) {
>         fprintf(stderr, "EVP_MAC_final() failed\n");
>         goto end;
>     }
>     out = OPENSSL_malloc(out_len);
>     if (out == NULL) {
>         fprintf(stderr, "malloc failed\n");
>         goto end;
>     }
>     /* Make one call to the final to get the MAC */
>     if (!EVP_MAC_final(mctx, out, &out_len, out_len)) {
>         fprintf(stderr, "EVP_MAC_final() failed\n");
>         goto end;
>     }
> 
>     printf("Generated MAC:\n");
>     BIO_dump_indent_fp(stdout, out, out_len, 2);
>     putchar('\n');
> 
>     if (out_len != sizeof(expected_output)) {
>         fprintf(stderr, "Generated MAC has an unexpected length\n");
>         goto end;
>     }
> 
>     if (CRYPTO_memcmp(expected_output, out, sizeof(expected_output)) != 0) {
>         fprintf(stderr, "Generated MAC does not match expected value\n");
>         goto end;
>     }
> 
>     ret = EXIT_SUCCESS;
> end:
>     if (ret != EXIT_SUCCESS)
>         ERR_print_errors_fp(stderr);
>     /* OpenSSL free functions will ignore NULL arguments */
>     OPENSSL_free(out);
>     EVP_MD_CTX_free(digest_context);
>     EVP_MAC_CTX_free(mctx);
>     EVP_MAC_free(mac);
>     OSSL_LIB_CTX_free(library_context);
>     return ret;
> }
> ```

我们首先来编写shell脚本可以完成的功能：

```bash
$ uuidgen #产生128bit随机序列
a8cf9d9a-d884-4234-a553-9b8b5c4afe67
$ uuidgen | tr -d ' \t\n-' #去掉其中的空格制表符回车符-字符
f0a0d7b5929b4d8ebcfed631982ca70b$ uuidgen | tr -d ' \t\n-' | tr a-z A-Z #把小写字母全部换成大写
74475C6954B84EED95CA510C8A24F0A2$

VERSION=V1
getUUID(){
 uuidgen | tr -d ' \t\n-'| tr a-z A-Z
#cat /proc/sys/kernel/random/uuid | tr -d ' \t\n-'| tr a-z A-Z
}

UUID=$(getUUID)
SERIALNO=${UUID:0:6}-${UUID:6:6}-${UUID:12:6}-${UUID:18:6}-${UUID:24:6}-${VERSION}
echo Generated serialnumber is $SERIALNO.

#arch=$(uname -m)
#case $arch in
#    "x86_64")
#        MACHINEID=$(cat /etc/machine-id | tr -d ' \t\n');;
#    "aarch64" | "arm64")
#        MACHINEID=$(cat /proc/cpuinfo | grep Serial | sed 's/Serial//g' | tr -d ' :\t\n')
#        if[ -z $MACHINEID ]; then
#        	MACHINEID=$(cat /etc/machine-id | tr -d ' \t\n')
#        fi;;
#esac

MACHINEID=$(cat /proc/cpuinfo | grep Serial | sed 's/Serial//g' | tr -d ' :\t\n')
if [ -z $MACHINEID ];then
 MACHINEID=$(cat /etc/machine-id | tr -d ' \t\n')
fi

HMACKEY=example-app
FINGERPRINT=$(echo -n ${MACHINEID} | openssl dgst -sha256 -hmac $HMACKEY | sed 's/(stdin)= //g' | sed 's/SHA2-256//g' | tr -d ' \t\n')
TOBEHASHED=$SERIALNO$FINGERPRINT
AESKEYB64=$(echo -n $TOBEHASHED | openssl dgst -sha256 -hex | sed 's/(stdin)= //g' | sed 's/SHA2-256//g' | xxd -r -p | openssl base64 -A | tr -d ' \t\n')
GCMIVB64=$(echo -n $(getUUID) | xxd -r -p | openssl base64 -A | tr -d ' \t\n' )
PLAINTXTB64=$(jq -n --arg IDVAL "$(uuidgen | tr -d ' \t\n')" --arg FPVAL "$FINGERPRINT" --arg CTVAL "$(date -u '+%Y-%m-%dT%H:%M:%S.%3NZ')" '{id:$IDVAL}+{type:"machines"}+{fingerprint:$FPVAL}+{created:$CTVAL}+{expiry:null}' | openssl base64 -A )

JSONENC=$(./gcmb64enc "$PLAINTXTB64" "$AESKEYB64" "$GCMIVB64" | grep "enc:" | sed 's/enc://g' | tr -d ' \t\n')
#待编写aesgcm加密程序，输入为明文，aeskey，gcm初始化向量，以上参数皆为base64文本格式，输出为JSON数据结构enc部分数据。
TOBEED25519=machine\\$JSONENC

tempGoal(){
if [ -z $1 ];then
 mktemp -p $(mktemp -d -t dir.XXXXXXXXXXX) -t tmpXXXXXXXXXXXXX
else
 mktemp -p $(mktemp -d -t dir.XXXXXXXXXXX) -t tmpXXXXXXXXXXXXX.$1
fi
}
temptobeed25519=$(tempGoal bin)
temped25519sig=$(tempGoal bin)
temped25519privatekey=$(tempGoal)
temped25519pubkey=$(tempGoal)
echo '-----BEGIN PRIVATE KEY-----
MC4CAQAwBQYDK2VwBCIEIH51B2DJpl9p2Unz58hlbHxDGfCnTeBqRAOhVXznRVHU
-----END PRIVATE KEY-----' > $temped25519privatekey
echo -n $TOBEED25519 > $temptobeed25519
openssl pkeyutl -sign -inkey $temped25519privatekey -out $temped25519sig -rawin -in $temptobeed25519
JSONSIG=$(cat $temped25519sig | openssl base64 -A)
JSONSIG=$(./edsign $TOBEED25519 | grep "sig:" | sed 's/sig://g' | tr -d ' \t\n')
JSONB64=$(jq -n --arg ENCVAL "$JSONENC" --arg SIGVAL "$JSONSIG" '{enc:$ENCVAL}+{sig:$SIGVAL}' | openssl base64 )
openssl pkey -in $temped25519privatekey -pubout -out $temped25519pubkey
openssl pkeyutl -verify -pubin -inkey $temped25519pubkey -rawin $temptobeed25519 -sigfile $temped25519sig
LICENSEFILE=machine.lic
echo -e '-----BEGIN MACHINE FILE-----\n' > $LICENSEFILE
echo $(JSONB64) >> $LICENSEFILE
echo '-----END MACHINE FILE-----\n' >> $LICENSEFILE
if [ -e ./machine.lic ]; then
 echo "machine license generated!"
else
 echo "machine license unable to generate!" 
fi
```

先写一个base64的encode和decode接口

```c
#include <openssl/bio.h>
#include <string.h>

size_t Base64Encode(const unsigned char* in, int len, char* out_base64, int no_nl)
{
    if (!in || len <= 0 || !out_base64)//做校验，检验参数是否合法
        return 0;
    //内存源source，从内存中读数据
    BIO* mem_bio = BIO_new(BIO_s_mem());//生成了一个内存的Bio，通过内存源来获取数据，结果要写在内存之中，做一下判断
    if (!mem_bio)return 0;

    //base64 filter过滤器,过滤器内写数据
    BIO* b64_bio = BIO_new(BIO_f_base64());
    if (!b64_bio)
    {
        BIO_free(mem_bio);
        return 0;
    }

    //形成BIO链表
    //b64-mem
    BIO_push(b64_bio, mem_bio);
    if(no_nl)
        BIO_set_flags(b64_bio, BIO_FLAGS_BASE64_NO_NL);//超过64字节不添加换行，编码的数据在一行中
     //默认结尾有换行符\n，超过64字节会再添加换行符

    //写入到base64 filter进行编码，结果会传递到链表的下一个节点
    //到mem中读取结果（从链表头部代表了整个链表,即从链表头部获取数据）
    int re = BIO_write(b64_bio, in, len);
    if (re <= 0)
    {
        //情况整个链表节点
        BIO_free_all(b64_bio);
        return 0;
    }

    //刷新缓存，写入链表的mem
    BIO_flush(b64_bio);
    
     
    size_t outsize = 0;
    //从链表源内存读取
    BUF_MEM* p_data = 0;
    BIO_get_mem_ptr(b64_bio, &p_data);
    if ((p_data)&&(p_data->data)&&(p_data->length>0))
    {
        memcpy(out_base64, p_data->data,p_data->length);
        outsize = p_data->length;
        *(out_base64+p_data->length)='\0';
    }
    BIO_free_all(b64_bio);
    return outsize;
}

size_t Base64Decode(const char* in, int len, unsigned char* out_data, int no_nl)
{
    if (!in || len <= 0 || !out_data)
        return 0;
    //内存源密文
    BIO* mem_bio = BIO_new_mem_buf(in, len);
    if (!mem_bio)return 0;
    //base64过滤器
    BIO* b64_bio = BIO_new(BIO_f_base64());
    if (!b64_bio){
        BIO_free(mem_bio);
        return 0;
    }
    //形成BIO链
    BIO_push(b64_bio, mem_bio);
    //默认换行符作为结束，否则会解码失败，因为除了结尾的换行符还有其他间隔换行符，设置后编码中有\n会失败
    if(no_nl)
        BIO_set_flags(b64_bio, BIO_FLAGS_BASE64_NO_NL);
    //读取解码4字节转3字节
    size_t size = 0;
    BIO_read_ex(b64_bio,out_data,len, & size);
    BIO_free_all(b64_bio);
    return size;
}

```

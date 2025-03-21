DICE
########

doc
==============

- `DICE: Device Identifier Composition Engine <https://www.microsoft.com/en-us/research/project/dice-device-identifier-composition-engine/>`_
- `RIoT <https://www.microsoft.com/en-us/research/wp-content/uploads/2016/06/RIoT20Paper-1.1-1.pdf>`_
- `TCG: DICE <https://trustedcomputinggroup.org/resources/?workgroups=DICE&>`_
- `DICE <https://trustedcomputinggroup.org/work-groups/dice-architectures/>`_
- `Dominance as a New Trusted Computing Primitivefor the Internet of Things <https://www.microsoft.com/en-us/research/uploads/prod/2019/03/cider-sp19.pdf>`_

glossary
==========================================================

DICE: Device Identifier Composition Engine

CDI: Compound Device Identifier

DeviceID: 从CDI派生的asymmetric key

DevID, IDevID, LDevID: follow IEEE802.1AR 标准的定义
- IDevID: Initial DevID，是manufacturing阶段的id，主要与制造商相关
- LDevID: Local DevID，是post manufacturing阶段的id，主要与oem商相关

ECA: Embedded Certificate Authority

TCB: Trusted Computing Base，指启动过程中载入的component

FSD: firmware security descriptor(一般是firmware的hash)

TCI: TCB Component Identity，指TCB的标识（例如firmware的FSD）

UDS: Unique Device Secret, 256 bits is better

overview
==========================================================

    power on -> DICE ( UDS ) ->  Layer 0 (Secret 0) -> Layer 1 (Secret 1) -> ...

以UDS为基础，为每个layer自动化链式派生Secret

layer如果发生变化，secret也会变化——也就是说，每次firmware升级，secret都会变

每个layer对应的secret，对于该layer本身，也是保密的——也就是说，layer自己无法获得自己的secret原始值

后台无需维护UDS的database

Policy OID
==========================================================

注意同样有Initial/Local的区分

Identity Policy OID: 设备标识

Attestation Policy OID: 使用Attestation Key签发device的特征信息（例如sw hash， product name等）, describe a device

Assertion Policy OID: 使用Assertion Key签发device的相关信息，reference measurements(一般指该对象的hash)

Certificate
==========================================================

IDevID Certificate的Issuer如果是ECA，那么ECA的issuer应该往上链到manufacturer CA

LDevID Certificate的Issuer如果是ECA，那么ECA的issuer应该往上链到owner CA（一般是OEM的CA）

Alias Key Certificate的Issuer是DevID Certificate，Alias Key Pair由UDS & 下一级Firmware的FSD派生

ECA Certificate的Issuer如果是ECA自身，那么必须标识当前layer的TCB

Attestation Certificate的Issuer可能是ECA，或者外部Issuer

first mutable code
==========================================================

first mutable code，要经过manufacturer的签名校验才能更新，或者出厂后就不再更新。。。

CDI可能由硬件直接保护（例如hsm）、也可能由first mutable code保护，避免泄漏。参考ISO/IEC 27034。

在重置后，会将first mutable code的measurement(即hash)，与UDS（重置后只允许读一次）结合，派生出firmware mutable code对应的CDI。在下次重置之前，禁止再次读取UDS。

first mutable code可能从DICE提供的专区使用CDI(read, write, modify)，但不能读取CDI原始值本身。

Basic Attestation Protocol
==========================================================

基于symmetric 

首先，基于layer 0的measurement、UDS派生symmetric alias key。

verifier发challenge C。

layer 0生成随机Nonce N

    R = HMAC(symmetric alias key, C | N) 

向verifier返回N、R

TLS-PSK Based Attestation Protocol
==========================================================

与Basic Attestation Protocol大体类似，只是派生的是PSK，而非symmetric alias key；verifier要从device提供的PSK ID hints中选择一个PSK ID hint，用于与CDI派生PSK。

再使用TLS-PSK与verifier安全通信。

DeviceID Key Pair
==========================================================

DICE根据UDS + First Mutable Code的measurement派生CDI，根据CDI派生DeviceID Key Pair

无需每次boot重复派生。

device可以从CDI派生一个symmetric key，用于加解密 DeviceID Key Pair。

Alias Key Pair
==========================================================

Device Firmware可获取DeviceID Public Key内容，并使用之。注意，private key内容是保密的。

Device Firmware可获取Alias Key Pair内容，并使用之。

Alias Key Pair的合法性由DeviceID Key保证。


RIoT
==========================================================

    FDS (Fuse Derived Secret) = KDF( Device Secret, Hash(L1) )

基于FDS派生对称密钥，可以进一步保护敏感数据。



DICE (Device Identity Composition Engine)
#################################################


背景
======

随着IoT生态的快速发展，全球IoT设备数量已达到百亿量级。

与此同时，利用IoT设备发动网络安全攻击的事件逐年上升，IoT安全形势非常严峻。

然而，由于IoT设备品种多样、计算能力差异较大，难以完全复用PC、手机已有的安全策略。

因此，TCG（Trusted Computing Group）推出了DICE（Device Identity Composition Engine）标准，旨在为多种类型的终端设备提供统一的设备标识机制，增强终端安全能力。

目前，DICE已获得ARM、Microsoft、Google、NXP等公司的支持，未来将在IoT、手机等终端设备进一步推广。

名词
======

DICE engine：开机时从ROM载入的程序，无条件信任执行。

TCB（Trusted Computing Base）：在系统启动过程中执行的各层组件。

TCI（TCB Component Identifier）：TCB的度量值，例如TCB firmware的hash可以作为TCI。

UDS（Unique Device Secret）：设备唯一机密值，仅允许DICE层访问，禁止下层TCB直接访问。

CDI（Compound Device Identifier）：每一层TCB的机密值。UDS与TCBL0的TCIL0派生出TCBL0的CDIL0。TCBLn的CDILn与TCBLn+1的TCILn+1派生出TCBLn+1的CDILn+1。

OWF（One Way Function）：单向函数，一般根据输入的参数生成定长的输出，例如hmac-sha256等。

ECA(Embedded Certificate Authority) Key：当前层级的TCB为了自身、或者下一层TCB签发证书的非对称密钥。

IDevID:受设备制造方控制的DeviceID。

DevID：受设备拥有方控制的DeviceID。

基础信息
============

DICE机制基于UDS链式创建安全启动中每个层独有的机密信息，UDS通过硬件安全机制保障。各层从上级接收自身的机密信息，并安全存储，各自保密。当安全启动中各层的代码、或者配置信息发生变动时，各层的机密信息会随之改变，各不相同。如果某层出现漏洞、导致机密信息泄露，可以通过补丁升级，系统自动为该层生成新的机密信息，重新保护。

链式派生CDI
**************

    DICE机制要求为每一层TCB链式派生CDI，再基于CDI进一步派生密钥、生成证书。

    DICE以UDS（Unique Device Secret）为起点，后续启动过程中自动化链式派生每一层TCBLn的CDILn。

基于CDI派生密钥
******************
    每一层TCBLn可以基于自身的CDILn派生非对称密钥对AKeyLn。

    每一层TCBLn可以基于自身的CDILn、TCILn派生对称密钥SKeyLn。

生成证书
******************

    每一层TCBLn可以基于自身的非对称密钥对AKeyLn，生成自身的ECALn证书；

    每一层TCBLn可以使用自身的ECALn证书，为下一层TCBLn+1的AKeyLn+1签发CertLn+1证书；

    每一层TCBLn可以由其他外部CA，为自身的AKeyLn签发CertLn证书。

注意：证书中不会包含任何私钥信息。

业务场景
===========

DICE支持设备标识、设备证明、安全启动、数据保护等业务安全场景。

设备标识
**********

    CAMFGCertL0作为TCBL0的设备标识(IDevIDL0)

    ECAL0CertL1作为TCBL1的设备标识(IDevIDL1)

    CAOWNCertL2作为TCBL2的设备标识(LDevIDL2)

外部相关方可以基于层次化的DevID标识每一个设备、设备上的每一层TCB。

设备证明
**********

    TCB基于自身的CDI、结合当前Firmware的信息派生Alias Key。

    TCB的DeviceID Key对应的ECA证书为Alias Key签发证书Alias Cert。

    Alias Cert中包含当前TCB的关键信息，例如配置信息、firmware版本号、firmware的hash值等。

设备可以基于Alias Cert，向外部相关方证明自身的状态。

安全启动
**********

DICE设备可以基于自动派生的ECA证书信任链实施安全启动。

Quacomm的安全启动信任链与DICE机制的底层校验方式相似。

二者区别主要在于：

    DICE支持设备级唯一的、自动派生的安全启动信任链。

    Quacomm信任链一般由多设备共享，通过统一镜像签名保证。

快速启动
**********

DICE设备可以启用快速启动模式：

    设备仅在每次firmware更新后、首次重新启动时，进行安全启动信任链的校验；并且使用由CDI派生的对称密钥SKeyLn，计算与当前firmware信息关联的hmac值。

    设备后续启动，基于hmac值进行校验，快速启动。

数据保护
**********

DICE设备可以基于UDS、结合ayer0的信息（可选），派生对称密钥，对设备敏感信息进行加密保护。

关联信息
==========

安全保护
**********

DICE设备应具备硬件安全保护能力，确保UDS信息无法被下层TCB直接读取。

DICE设备应具备安全存储各层TCB相关敏感数据的能力，包含CDI、以及由CDI派生的非对称私钥、对称密钥等。

各层TCB的敏感信息，仅能由自身、或上一层TCB读取。
Firmware升级

DICE设备各层TCB的firmware升级，可以基于设备证明触发，确保安全更新。

由于DICE机制基于UDS派生的设备证明，无法借用其他设备的证明信息、或者自行伪造新版本firmware的设备证明，绕过OTA版本升级要求。


Firmware升级过程示例
*************************

    设备通过AliasCert向OTA后台上报设备证明，包含当前firmware的版本信息、配置信息等。

    OTA后台检查设备证明，根据版本安全策略（例如低版本、漏洞版本等），下发升级指令。

    设备对升级指令进行签名校验，确保指令来源的合法性。

    设备向OTA后台获取升级信息，并对升级信息进行签名校验，确保升级信息来源的合法性。

    设备根据升级信息，下载升级包，并对升级包进行签名校验，确保升级包下载来源的合法性。

    设备根据升级信息设定的顺序，启动指定TCB的firmware升级。

    设备对firmware升级包进行签名校验、检查版本信息，确定firmware升级包内容的合法性、防止回滚。

    设备对firmware进行升级。

    设备确认firmware升级成功，基于新版本的firmware信息，重新派生DICE体系的密钥、证书。

小结
======

DICE是一种新型的终端安全机制，优势明显：

    为终端设备提供了统一的设备标识机制，支持设备证明、安全启动、数据保护等业务安全场景，具有良好的安全性、鲁棒性、可扩展性。

    兼容现有的安全启动机制、OTA升级机制，灵活度较高。

    TCB的firmware支持独立升级部署，不强制依赖于上级私钥签发安全启动证书。

    支持基于CDI派生对称密钥实现快速启动，适用于能力有限的IoT终端设备。

参考资料
=========

1. `What is DICE architecture <https://www.microcontrollertips.com/what-is-dice-architecture-faq/>`_
#. `TCG. DICE Layering Architecture. <https://trustedcomputinggroup.org/wp-content/uploads/DICE-Layering-Architecture-r19_3june2020.pdf>`_
#. `Microsoft. DICE: Device Identifier Composition Engine. <https://www.microsoft.com/en-us/research/project/dice-device-identifier-composition-engine/>`_
#. `Qualcomm. Secure Boot and Image Authentication. <https://www.qualcomm.com/media/documents/files/secure-boot-and-image-authentication-technical-overview-v2-0.pdf>`_
#. `Microsoft. RIoT. <https://www.microsoft.com/en-us/research/wp-content/uploads/2016/06/RIoT20Paper-1.1-1.pdf>`_
#. `DICE resource <https://trustedcomputinggroup.org/resources/?workgroups=DICE&>`_
#. `Open Profile for DICE <https://pigweed.googlesource.com/open-dice/+/refs/heads/main/docs/specification.md#Deriving-Asymmetric-Key-Pairs>`_
#. `Measured Boot and Remote Attestation <https://lf-edge.atlassian.net/wiki/spaces/EVE/pages/14584444/Measured+Boot+and+Remote+Attestation>`_


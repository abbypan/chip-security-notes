Intel SGX
#############

概要
=======

Intel在桌面、IoT终端废弃SGX，在Xeon保留SGX用于机密计算。

server通过attestation证明可信身份，与data owner computer建立可信communication channel，进行机密计算数据交互。


key provision
=================

GWK(global wrapping logic key)在芯片电路中硬编码，用于加密efuse content：EPID key (256-bit),  pre-seed key0 (128-bit)。

另有 pre-seed key1 (128-bit)、EPID group ID (32-bit) 明文烧入efuse。

PUF key增强方案：gwk加密puf key后提交key generation server。key generation server解密后使用puf key加密fuse key。

Provisioning Secret 由 intel基于SGX Master Derivation Key派生（EGETKEY），intel知晓并烧入efuse。

Seal Secret 由芯片内部生成并烧入efuse，理论上intel不知晓。

Provisioning Secret 用于provisioning service对Provisioning enclave的authentication，通过认证后，生成Attestation key并返回。Provisioning enclave使用Provisioning Seal key加密安全存储Attestation key。

Attestation Key 用于 Intel Enhanced Privacy ID (EPID) 系统。


local attestation
===================

cmac

remote attestation
======================

local attestation 转交后调ak签名。

敏感PPID通过PPIDEK加密保护。


Quoting Enclave
===================

quoting enclave(QE) report 内容含 Attestation Key。

Provisioning Certification Key (PCK) 负责对quoting enclave report签名。

sealing key
=================

sgx_get_key的sealing key是CPU-bound，本地data sealing。

sealing key派生key policy有两种：MRENCLAVE，MRSIGNER分别锁定enclave版本与code signer。


参考资料
=============

- `Intel SGX deprecation review  <https://hardenedvault.net/blog/2022-01-15-sgx-deprecated/>`_
- `I cannot play back my 4k Blu-Ray even though I have an SGX-capable CPU in my computer <https://answers.microsoft.com/en-us/windows/forum/all/i-cannot-play-back-my-4k-blu-ray-even-though-i/255cc241-4dd3-4ebc-9d8f-1cadfe5e2173>`_
- `Intel SGX Explained <https://eprint.iacr.org/2016/086.pdf>`_
- `Intel® Trust Domain Extensions Data Center Attestation Primitives (Intel® TDX DCAP): Quote Generation Library and Quote Verification Library <https://download.01.org/intel-sgx/latest/dcap-latest/linux/docs/Intel_TDX_DCAP_Quoting_Library_API.pdf
>`_
- `Intel® Trust Domain Extensions (Intel® TDX) Module Base Architecture Specification <https://cdrdv2-public.intel.com/733575/intel-tdx-module-1.5-base-spec-348549002.pdf>`_
- `Overview of Intel SGX - Part 1, SGX Internals <https://blog.quarkslab.com/overview-of-intel-sgx-part-1-sgx-internals.html>`_
- `Overview of Intel SGX - Part 2, SGX Externals <https://blog.quarkslab.com/overview-of-intel-sgx-part-2-sgx-externals.html>`_
- `Intel SGX <https://www.intel.com/content/dam/develop/external/us/en/documents/overview-of-intel-sgx-enclave-637284.pdf>`_
- `Another Intel SGX Security Flaw? Our Analysis of the SGX Fuse Key Extraction Claim <https://blog.mithrilsecurity.io/another-intel-sgx-security-flaw-our-analysis-of-the-sgx-fuse-key-extraction-claim/>`_

AMD SEV-SNP Attestation
#########################

AMD Secure Encrypted Virtualization (SEV) - Secure Nested Paging (SNP)

链式信任:

    AMD Root Key (ARK)  -> AMD Signing Key (ASK) -> Versioned Chip Endorsement Key (VCEK)

VCEK 用于签发 attestation report，内容包含 Bootloader, TEE, SNP, Microcode, Chip ID。

支持基于TCB参数派生 DerivedKey 。


参考资料
============

- `SEV-SNP Platform Attestation Using VirTEE/SEV <https://www.amd.com/content/dam/amd/en/documents/developer/58217-epyc-9004-ug-platform-attestation-using-virtee-snp.pdf>`_
- `AMD Secure Encrypted Virtualization (SEV) <https://www.amd.com/en/developer/sev.html>`_

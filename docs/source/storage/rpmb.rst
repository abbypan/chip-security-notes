RPMB (Replay Protected Memory Block)
========================================


emmc
-------------

eMMC security feature: password lock/unlock, write protect, RPMB

::

    AP (Application Processor)
    CP (Communication Processor)，也称为modem
    AP <--- IPC ---> CP
    eMMC (Embedded Multi Media Card) 存储软件、用户数据


RPMB是从eMMC划出一块特殊的区域，128KB的整数倍，进行访问控制，确保外部无法直接读取，而是需要通过hmac-sha256校验后由RPMB返回相关内容

modem 通过 IPC，请 AP 帮忙把某些数据（例如device id、subscriber id、call number）写入eMMC，这些信息一般是需要保密、且防篡改。

emmc partition
-----------------

Boot Area Partitions: boot1, boot2

RPMB Partition

General Purpose Partitions (GPP): 1~4，GPP分区一般OTP单次写入。

User Data Area: 可基于GPT重分区。


partition size
-------------------

boot1, boot2, rpmb  三个partition的默认size

::

    Partition Size = 128KB × BOOT_SIZE_MULT

默认 BOOT_SIZE_MULT = 32，Partition = 4MB。

BOOT_SIZE_MULT最高 128，Partition 最高16MB。

overview
----------

RPMB Data frame的关键内容：Authentication Key，Data，Nonce，Write Counter, Address

产线阶段，Host生成HUK，基于HUK派生Authentication Key，并将Authentication Key写入RPMB。

read
-------

Host生成nonce，向eMMC请求读取某个Address的内容

eMMC基于预置的Authentication Key，结合nonce、Data等信息，计算MAC

Host校验收到的MAC，确认Data内容未经篡改

write
-------

Host生成nonce，并基于Authentication Key，结合nonce，计算MAC；向eMMC请求谋取某个Address的Write Counter

eMMC校验MAC，确认读取Write Counter请求的合法性

eMMC返回Write Counter，且返回的内容也要带MAC

Host校验收到的Write Counter未经篡改

Host基于预置的Authentication Key，结合Write Counter、新的Data等信息，计算MAC；发给eMMC

eMMC校验MAC，确认写入新的Data，并将Write Counter加1

eMMC基于预置的Authentication Key，结合新的Write Counter 以及Result，计算MAC；发给Host

Host校验MAC，确认写入成功

use case
---------

anti-rollback, 存version number

fail unlock attempts，防止暴力破解

secure write protect


trusty
---------

`hwkey_srv_fake_provider.c <https://android.googlesource.com/trusty/app/sample/+/refs/heads/main/hwcrypto/hwkey_srv_fake_provider.c>`_

::

    get_rpmb_ss_auth_key
        EVP_EncryptInit_ex(&evp, EVP_aes_256_cbc(), NULL, fake_device_key, NULL);
        rc = EVP_EncryptUpdate(&evp, kbuf, &out_len, rpmb_salt, sizeof(rpmb_salt));


optee
------

optee的处理更好一点。

- `RPMB Secure Storage <https://optee.readthedocs.io/en/latest/architecture/secure_storage.html#rpmb>`_


`rpmb.c <https://github.com/OP-TEE/optee_client/blob/master/tee-supplicant/src/rpmb.c>`_

::

    read_cid
        static const uint8_t test_cid[] = {
            /* MID (Manufacturer ID): Micron */
            0xfe,
            /* CBX (Device/BGA): BGA */
            0x01,
            /* OID (OEM/Application ID) */
            0x4e,
            /* PNM (Product name) "MMC04G" */
            0x4d, 0x4d, 0x43, 0x30, 0x34, 0x47,
            /* PRV (Product revision): 4.2 */
            0x42,
            /* PSN (Product serial number) */
            0xc8, 0xf6, 0x55, 0x2a,
            /*
             * MDT (Manufacturing date):
             * June, 2014
             */
            0x61,
            /* (CRC7 (0xA) << 1) | 0x1 */
            0x15
        };



`derive_rpmb_key.py <https://github.com/OP-TEE/optee_os/blob/master/scripts/derive_rpmb_key.py>`_


::

    derive_key
        # Prepare the CID and Clear the PRV (Product revision) and CRC (CRC7
        # checksum) fields as OP-TEE does.
        data = bytearray(cid)
        data[9] = 0
        data[15] = 0

        # This is how __huk_subkey_derive() is implemented, if huk_subkey_derive()
        # is overridden the key derived here may not match what OP-TEE is using
        #
        # HUK is as tee_otp_get_hw_unique_key() in OP-TEE returns it
        okm = hmac-sha256(huk, data)




参考资料
-----------

1. `e.MMC Security Methods <https://documents.westerndigital.com/content/dam/doc-library/en_us/assets/public/western-digital/collateral/white-paper/white-paper-emmc-security.pdf>`_
#. `Replay Protected Memory Block (RPMB) <https://www.sdcard.org/developers/boot-and-new-security-features/replay-protected-memory-block/>`_
#. `Hardware-Backed Mobile Secure Storage <https://www.qualcomm.com/media/documents/files/guard-your-data-with-the-qualcomm-snapdragon-mobile-platform.pdf>`_
#. `Mobile Secure Data protection using eMMC RPMB Partition <https://ieeexplore.ieee.org/document/7411305>`_
#. `Exploiting RPMB authentication in a closed source TEE implementation <https://www.sciencedirect.com/science/article/pii/S2666281723002019>`_
#. `RPMB, a secret place inside the eMMC <https://sergioprado.blog/rpmb-a-secret-place-inside-the-emmc/>`_
#. `Keyless Entry: Breaking and Entering eMMC RPMB with EMFI <https://dl.acm.org/doi/pdf/10.1145/3643833.3656114>`_  
#. `i.MX RT eMMC RPMB Enablement <https://www.nxp.com/docs/en/application-note/AN13975.pdf>`_
#. `RPMB, a secret place inside the eMMC <https://sergioprado.blog/rpmb-a-secret-place-inside-the-emmc/>`_
#. `emmc 分区管理 <https://blog.csdn.net/weixin_43982460/article/details/136429640>`_
#. `Android tamper-resistant anti-replay secure storage solution and virtualization <https://itdks.su.bcebos.com/1a9c67a8c5d64f568925cdebb401f491.pdf>`_
#. `Implement Android Tamper-Resistant Secure Storage and Secure it in Virtualization <https://events19.linuxfoundation.org/wp-content/uploads/2017/12/Implement-Android-Tamper-Resistant-Secure-Storage-Bing-Zhu_and-Secure-it-in-Virtualization-Bing-Zhu-Intel-Corporation.pdf>`_

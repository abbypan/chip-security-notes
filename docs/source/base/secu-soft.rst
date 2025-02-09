Secure software guidelines
############################

`Arm: Secure software guidelines <https://developer.arm.com/documentation/100720/0200/>`_

secure & non-secure 状态切换
===============================

SG: secure gateway. 从non-secure到secure

BXNS: branch with exchange to non-secure state. secure software 可以 branch/return non-secure program

BLXNS: branch with link and exchange to non-secure state. secure software 调用 non-secure function

Data validation and prevention of information leakage
==========================================================

Memory Protection Unit (MPU) is banked between the security states



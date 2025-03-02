Generic Interrupt Controller (GIC)
#####################################

`Generic Interrupt Controllers <https://developer.arm.com/ip-products/system-ip/system-controllers/interrupt-controllers>`_

Universal Asynchronous Receiver/Transmitter (UART)

Interrupt types
==================

SPI: Shared Peripheral Interrupt

PPI: Private Peripheral Interrupt

SGI: Software Generated Interrupt

LPI: Locality-specific Peripheral Interrupt

Peripheral -> { Interrupt Signal } -> Interrupt Controller -> { IRQ / FIQ } -> PE

Security model
==================

Secure Group 0: Interrupt for EL3 (Secure Firmware)

Secure Group 1: Interrupt for EL1 (Trusted OS)

Non-secure Group 1: Interrupt for the Non-secure state (OS or Hypervisor)

FIQ & IRQ
==============

FIQ: Firmware/Secure manager level (EL3)

IRQ: OS level



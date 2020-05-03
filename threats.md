---
layout: page
title: Ledger Nano Threat model
permalink: /threat-model/
---

# Threat Model


----
## Security Objectives
- Prevent attackers from extracting user seeds or private keys
- Bypass PIN protection / user's consent 
- Prevent attackers from misleading the end user (eg. by displaying arbitrary data on the device screen)
- Prevent attackers from manufacturing fake devices
- Protect the confidentiality of the firmware and Secure Element API / intellectual property (under NDA)
- Protect Ledger secret keys (attestation)
- Protect user's privacy - Prevent users to be uniquely identified

Software and hardware attacks are considered.


## Definitions

### Roles:

- **End user**: The end user is the happy owner of a Ledger Nano S/X. He has physical access to the device
- **Firmware developer**: Only some Ledger employees can develop the Firmware of the Ledger Nano devices. They are in charge of developping the OS and its crypto-lib
- **App Developer**: The Ledger Nano S OS is open. Anyone can develop an app running on top of it. Developping on Ledger Nano X requires Ledger authorization.
- **HSM**: Hardware Security Modules are basically remote computers able to check the device genuineness and perform privileged operations (install/remove apps, update firmware) on the devices

### Key usage scenarios:

- User install apps thanks to the Ledger Live
- User makes crypto-currency transaction thanks to the Ledger Live. Critical pieces of information are displayed and confirmed on the device.
- User updates its device thanks to the Ledger Live

### Technologies:
The Ledger Nano S and Nano X are composed of:

- A Secure Element (ST31 for Nano S, ST33 for Nano X)
- A general purpose MCU (STM32Fxxx for Nano S, STM32WB55 for Nano S)
- External peripherals: screen, buttons


# Security Mechanisms 

## Devices
### Device Genuineness

### Physical Protection



## OS

## Applications








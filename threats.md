---
layout: page
title: Ledger Nano Threat model
permalink: /threat-model/
---

# Threat Model

This page is intended to describe the threat model of Ledger Nano S and Nano X devices. It first lists the main security objectives the devices intend to fullfil. Then it describes the security mechanisms implemented in order to actually reach these objectives. The associated threats to these security mechanisms are also mentionned.

## Security Objectives
The main security objective of the Ledger Nano devices is to provide a **physical and logical** security to users' funds. This objective can be divided in the following sub-objectives:
1. Guarantee the **confidentiality of user seeds and private keys**
2. Ensure the use of digital assets is performed **under user consent**. In particular, the device shall prevent attackers from misleading the end user (eg. by displaying arbitrary data on the device screen)
3. Provide a mechanism allowing the user to verify he has a **genuine** device.
4. Protect users' **privacy** - In particular, the device shall prevent users to be uniquely identified
5. Protect the **confidentiality of the firmware and Secure Element API** / intellectual property (under NDA)


## Definitions

For the sake of clarity, some basic definition are recalled. In particular, the roles, the key usage and the components of the devices are recalled.

### Roles:

- **End user**: The end user is the happy owner of a Ledger Nano S/X. He has physical access to the device
- **Firmware developer**: Only some Ledger employees can develop the Firmware of the Ledger Nano devices. They are in charge of developping the OS and its crypto-lib
- **App Developer**: The Ledger Nano S OS is open. Anyone can develop an app running on top of it. Developping on Ledger Nano X requires Ledger authorization.
- **HSM**: Hardware Security Modules are basically remote computers able to check the device genuineness and perform privileged operations (install/remove apps, update firmware) on the devices.

### Key usage scenarios:

- User install apps thanks to the Ledger Live
- User makes crypto-currency transaction thanks to the Ledger Live. Critical pieces of information are displayed and confirmed on the device.
- User updates its device thanks to the Ledger Live

### High Level architecture:
The Ledger Nano S and Nano X are composed of:

- A Secure Element (ST31 for Nano S, ST33 for Nano X)
- A general purpose MCU (STM32Fxxx for Nano S, STM32WB55 for Nano S)
- External peripherals: screen, buttons

----

# Security Mechanisms 

Several security mechanisms are implemented at different levels. In the following we'll distinguish device security mechanisms, OS security mechanisms and app security mechanisms.

## Devices

### Device Genuineness
The ability to prove genuineness of the device is one of the main security features, from both hardware and firmware points of view. The hardware wallet must have a secure mechanism for this, and this is at the utmost importance. An attacker could otherwise have replaced a genuine device by a fake and backdoored one (through supply chain or evil maid attack for instance). In this case, he would be able to access to the crypto assets afterwards. For the record, [non-genuine Trezor One devices](https://blog.trezor.io/psa-non-genuine-trezor-devices-979b64e359a7) were sold on the Internet in 2018.
Anti-tampering seals (or holographic seals) can give a false sense of security: not only are they trivial to clone, but it is also easy to open and close a package without damaging the seal.

#### Ledger Genuine Check

To prove the genuineness of Ledger devices, the following steps take place during the manufacturing (in secure environment):
- Each Ledger device generates a unique pair of keys: a public key and a private key. The private key is kept secret to the device only and cannot be exported nor retrieved.
- The device sends its public key to Ledger’s HSM. Our HSM signs the public key with the Ledger Root of Trust and sends it back to the device. This signed public key is the device’s attestation. It is stored inside the device and cannot be extracted either.

After manufacturing, this attestation allows the user (through Ledger Live) to verify if the device is genuine. The HSM sends a challenge which must be signed by the device and sent back along the attestation. This allows the HSM to verify the attestation and the challenge signature and eventually tell whether the device is genuine or not. More details can be found in [this blogpost]([https://www.ledger.com/a-closer-look-into-ledger-security-the-root-of-trust/).

<U>Associated Threat</U>
> An attack allowing to extract a device is a major threat to device genuineness security mechanism. Generally speaking, any attack allowing a non genuine device to pass the genuine check is a valid attack.

#### End User Physical Verification

We have designed the Ledger Nano S to be easily openable, so users can [check the integrity](https://support.ledger.com/hc/en-us/articles/360019352834-Check-hardware-integrity) of their device by themselves as detailed here. Being aware that this solution might not be suitable for all users, the architecture of the Ledger Nano X is different: the buttons and the screen are directly connected to the Secure Element to prevent this kind of chip-in-the-middle attack.

<U>Associated Threat</U>
> An attack allowing to extract a device is a major threat to device genuineness security mechanism. Generally speaking, any attack allowing a non genuine device to pass the genuine check is a valid attack.


### Secure Display

The wallet should ensure secure display and secure inputs, for verifying and granting transactions. These properties can only be granted if the wallet has a genuineness and integrity mechanism.

### Physical Protection



## OS

## Applications







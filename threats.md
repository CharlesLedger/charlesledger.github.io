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

## Device: Genuineness

The ability to prove genuineness of the device is one of the main security features, from both hardware and firmware points of view. The hardware wallet must have a secure mechanism for this, and this is at the utmost importance. An attacker could otherwise have replaced a genuine device by a fake and backdoored one (through supply chain or evil maid attack for instance). In this case, he would be able to access to the crypto assets afterwards. For the record, [non-genuine Trezor One devices](https://blog.trezor.io/psa-non-genuine-trezor-devices-979b64e359a7) were sold on the Internet in 2018.
Anti-tampering seals (or holographic seals) can give a false sense of security: not only are they trivial to clone, but it is also easy to open and close a package without damaging the seal.

### Ledger Genuine Check

To prove the genuineness of Ledger devices, the following steps take place during the manufacturing (in secure environment):
- Each Ledger device generates a unique pair of keys: a public key and a private key. The private key is kept secret to the device only and cannot be exported nor retrieved.
- The device sends its public key to Ledger’s HSM. Our HSM signs the public key with the Ledger Root of Trust and sends it back to the device. This signed public key is the device’s attestation. It is stored inside the device and cannot be extracted either.

After manufacturing, this attestation allows the user (through Ledger Live) to verify if the device is genuine. The HSM sends a challenge which must be signed by the device and sent back along the attestation. This allows the HSM to verify the attestation and the challenge signature and eventually tell whether the device is genuine or not. More details can be found in [this blogpost]([https://www.ledger.com/a-closer-look-into-ledger-security-the-root-of-trust/).

<U>Associated Threat</U>
> An attack allowing to extract a device is a major threat to device genuineness security mechanism. Generally speaking, any attack allowing a non genuine device to pass the genuine check is a valid attack.

### End User Physical Verification

We have designed the Ledger Nano S to be easily openable, so users can [check the integrity](https://support.ledger.com/hc/en-us/articles/360019352834-Check-hardware-integrity) of their device by themselves as detailed here. Being aware that this solution might not be suitable for all users, the architecture of the Ledger Nano X is different: the buttons and the screen are directly connected to the Secure Element to prevent this kind of chip-in-the-middle attack.

<U>Associated Threat</U>
> An attack allowing to extract a device is a major threat to device genuineness security mechanism. Generally speaking, any attack allowing a non genuine device to pass the genuine check is a valid attack.


## Device: Secure Display and inputs

The wallet should ensure secure display and secure inputs, for verifying and granting transactions. It allows to ensure the "What you see is what you sign" property. It's at the utmost importance for the user to have a trusted display for verifying his transactions, getting his receiving address... It prevents an attacker from tricking the user in various ways to leverage his rights. It's equally important to have trusted inputs. It avoids an attacker to bypass user consent for approving a transaction for instance. This security mechanism is implemented through the genuineness property, the firmware integrity firmware mechanism and the device architecture itself.

<U>Associated Threat</U>
> An attack allowing to run a non legit firmware on the secure element, execute arbitrary code controlling the display or bypassing user's input, an hardware modification are some examples of threat against this property.


## Device: Physical Resistance

Once an attacker gains physical access to a device (for instance by stealing it), a wide range of attacks become possible. It is thus important to build protections to prevent access to the device secrets. For instance, the number of PIN tries should be limited, otherwise an attacker could try every combination possible. That’s why hardware wallets usually wipe the device memory after a low threshold is reached.

Hardware wallets can also be targeted by more sophisticated attacks such as fault injection or side-channel attacks. Side-channel attacks are a wide range of attacks that consist of exploiting physical leakages of a device handling sensitive information. These attacks focus on measurable information obtained from the implementation of an algorithm, rather than weaknesses in the algorithm itself. For instance, an attacker with physical access to a security device could measure the power consumption or electromagnetic emanations of the circuit to extract information that could lead to the secret manipulated by the device.

Ledger devices use Secure Elements along with software especially developped to prevent these kind of attacks.

<U>Associated Threat</U>
> An attacker with a physical access to the device with a high potential (expertise, time, equipement), is the typical threats the Ledger devices shall counter. This comprises Laser, EM, Glitch Fault injection, Side Channel Attacks, Hardware reverse... Any physical attack allowing to extract seeds, PIN, firmware is a valid attack.


## OS - PIN Security Mechanism
An attacker with a physical access to a device (eg. device stolen) might get a full control over the device, meaning that sensitive operations can be processed.

To prevent this, a PIN security mechanism is implemented. During the boot of Ledger devices, the end-user must prove that he is the owner of its device thanks to its PIN (Personal Indentification Number). This security function is the first interaction between the end-user and the device and is critical because it gives access to all services. For instance, all cryptocurrency apps are available meaning cryptocurrency transfer is available. Note that all other Apps (for instance Password Manager, FIDO) are also available as soon as the PIN verification is successfully performed.

The length of the PIN, defined by the end-user during the on-boarding stage, must be in the following range: minimum 4 digits, maximum 8 digits. The PIN Try Counter (PTC), whose default value is set to 3, counteracts brute-force attacks revealing the value of the PIN. It shall be noticed that a PIN is not ultimate protection. It leaves 3 tries to find the user's PIN value. With a 4-digit PIN, if PIN values were equally ditributed, it would leave 0.03% of chance to find the user' PIN value. Unfortunately, 10% of PIN values are '1234'. 

As soon as the PTC exceeds its limit, the device wipes the following sensitive assets:

- The PIN
- The seed
- Secret Data

Thanks to this security action of wiping, the device cannot be used because because the current state is not operational anymore. An initialization (either normal mode or restore mode) is then required.

<U>Associated Threat</U>
> Any attack allowing to bypass the PIN verification or to guess/extract the correct value of the user's PIN is a valid attack.

## OS - Random Number Generation
One of the main security feature of Ledger devices is the capability to generate high quality randomness. The master seed, which is generated from random numbers during the setup of a wallet, is used to derive almost every secrets of a wallet. Having a low quality randomness has terrible consequences because it allows attackers, in the worst case, to recreate the seed without any specific knowledge. For instance, some [Android wallets were cleared out](https://bitcoinmagazine.com/articles/critical-vulnerability-found-in-android-wallets-1376273924) in 2013 because of a bug in the random generator of Android itself. It is thus especially important to guarantee a high quality randomness.

However, ensuring the true randomness of a Random Number Generator is especially difficult because pseudo-randomness is statistically indistinguishable from true randomness, but can still be predictable for attackers. 

Hardware wallets are built with Integrated Circuit (IC) and inside the circuit there is often a specific part of electronics, the TRNG, which stands for True Random Number Generator. Different kinds of design can be found, they often rely on free oscillators which run in parallel and are sampled at a very specific timing. This very tiny source of entropy is amplified with different means and the result is used to feed the Random Number Generator. This kind of circuit can be found in Secure Elements. The high quality of the randomness is verified during mandatory security evaluations. These evaluations are performed by a 3rd party laboratory to obtain highest level certifications EAL5+, AIS-31. This methodology includes a mathematical proof of randomness and very large number of tests. The RNG is tested under various conditions of temperatures, frequency, voltage and must pass all the statistical tests. It also includes randomness defects and attacks detection mechanisms.

On Ledger devices Random Number generation is used for seed generation, Ephemeral keys generation, and countermeasures.

<U>Associated Threat</U>
> Any mean allowing an attacker to reduce the entropy, predict seed/keys value without detection is a valid attack.

## OS - Confidentiality of Seed/Private keys

Even if the device is genuine and the random generator of high quality, a hardware wallet which stores its seed unencrypted on a SD card cannot be considered as secure because the seed can be retrieved trivially.

On Ledger devices, the seed is stored in the non-volatile memory of the Secure Element. The seed can be either generated by the Secure Element itself thanks to its True Random Number Generator, or imported when the device is booted in Recovery mode.

Once the device is initialized, there is absolutely no way to retrieve the seed. Even apps installed on the device cannot read it (cf Isolation <!TODO> ).

<U>Associated Threat</U>
> Seed Extraction attacks are classical threat vectors already demonstrated by the security community ([Glitch attack](https://colinoflynn.com/2019/03/glitching-trezor-using-emfi-through-the-enclosure/), [SRAM seed extraction](https://saleemrashid.com/2017/08/17/extracting-trezor-secrets-sram/), [EMFI attack](https://www.offensivecon.org/speakers/2019/sergei-volokitin.html))
 
## OS - Integrity


## OS - Confidentiality


# OS - Transport Security

Without communication the 

USB is the only way to communicate with the Nano S while the Ledger Nano X also features Bluetooth Low Energy (BLE) connectivity.

As these protocols expose a broad attack surface, there is a dedicated and untrusted piece of hardware, the MCU, whose main role is to implement these communication protocols. Once the packets are decoded by the MCU, their content is forwarded to the Secure Element which has little to no knowledge about the original communication protocol.


# App - Isolation

One of the main feature of Ledger devices is that anyone can load its own app on the Secure Element. Each app is isolated from each other thanks to BOLOS, the Operating System. That essentially means that:

- An app cannot access to the OS memory;
- An app cannot read or write the volatile and non-volatile memory from another app;
- An app can derive keys on its own HD path only, which ensures that cryptocurrency apps cannot steal keys from each other. For instance, the Zcoin app cannot derive keys on the Dogecoin derivation path (`m/44'/3'/`), since its own derivation path is `m/44'/128'/`.

The OS relies on dedicated hardware (the MPU or MMU) to isolate the apps between them and also to isolate the OS itself from the apps.



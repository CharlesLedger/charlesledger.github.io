---
layout: post
title: Is MPC truly ready for digital assets cusody 
author: yhql
summary: Trying to bring a modern language and memory safety into our applications
featured-img: mpc 
categories: Vault
---

Securing critical digital assets is Ledger’s mission and we have been observing with great interest the latest developments in multi-party computation (MPC). Within this post, we wanted to provide Ledger’s current stance on the use of MPC to secure digital assets and what we envision for the future.

# The challenge of managing the key lifecycle

Unlike traditional financial assets, critical digital assets such as cryptocurrencies bear high operational risks stemming from key management, transactions being irrevocable and the openness of the network. Cryptocurrencies empower end users with full control over the asset and thereby grants them complete responsibility over it. Since a security breach can result in an instantaneous and irrevocable loss of funds, the security of keys both at rest and in usage is paramount.

* Securing the key at rest means securing the generation and storage of the key.
* Securing the usage of the key encompasses transaction signature itself but also user management, wallet management,

<center>
<img src="/assets/mpc/key_mgmt_lifecycle.png" >
governance rule enforcement and transaction verification.
</center>
_<center>governance rule enforcement and transaction verification</center>_
<br/>


## Key management lifecycle

Securing the full key management lifecycle is not an easy task and MPC is the latest addition to the existing security toolbox available to institutions and investors aiming at protecting their digital assets.

# What is MPC?

Multi-party computation refers to specific cryptographic algorithms enabling multiple parties to perform some computation from each party’s individual data without disclosing it to the other parties. For example: using MPC it could be possible to compute the average salary among several employees without any employee knowing the salary of the other coworkers[1](#References).

Using the same properties to secure digital assets, MPC is used to create setups where the signature process is split among several users and the signature process does not require each signer to disclose their key to the other signers. The benefits are straightforward. Rather than compromising a single endpoint an attacker would then need to target several instances at once through a coordinated attack. Each instance would ideally rely on a different security setup, thereby increasing significantly the complexity of the potential attack.

When applying a MPC scheme, the first step is wallet creation. Several users each generate a part of the key used to sign transaction from a wallet without the key being generated in full at any point. This can be done by implementing an MPC-version of the key derivation algorithms for cryptocurrencies (BIP-32).

Once the wallet is created, MPC can be applied for transaction signature. One category of MPC algorithms is Threshold Signature Schemes[2](#References) (TSS) . It allows a group of n signers to define a threshold of m required approvals to perform a signature. The process of transaction signature does not require any single approver to assemble or hold the full key at any point. Most MPC-based solutions currently available on the market implement TSS.

<center>
<img src="/assets/mpc/quorum.png" >
</center>
_<center> Simplified example of a 3 approvers MPC quorum using TSS </center>_
<br/>


In the above example the transaction is not complete and fully formed until all the approvers have computed their share of the transaction. It should also be noted that some MPC schemes do not require any specific sequentiality of approvals: Alice, Bob and Charlie could have signed the data in any order. The three approvers can also use different devices to perform the computation, such as a smartphone, a laptop or a regular computer.

# What are pitfalls of MPC?
## MPC still is a very young technology with very little security testing

Current MPC schemes are quite new and still not battle-tested. The seminal papers on the topic date back to the 80s but the first paper outlining a usable case was only published in 2016. Security setups based on MPC are yet to pass extensive penetration testing and security certifications such as Common Criteria, FIPS or CSPN certifications. Standardisation initiatives[3](#References). will give security researchers a common ground to assess solutions using MPC but are yet to be widely accepted.

During the 2019 IACR Crypto conference Jonathan Katz presented a keynote talk where he outlined vulnerabilities in MPC implementations of fixed-key AES[4](#Reference). While this vulnerability does not mean that MPC setups are insecure per se, it does raise the question of the security of their implementations. As Trail of Bits stated on the talk: “This attack highlights the recklessness of rushing to deploy cutting-edge cryptography. These protocols are often extremely slow and complex, and few people understand the subtle details of the security proof. More work must be done to quantify the concrete security of these protocols as they are actually instantiated, not just asymptotically using idealized functionalities.”[5](#References)

Because the security of digital assets is a “one error lose it all” game, introducing new technologies must be done carefully. Implementing new technology as the linchpin of a security infrastructure can have catastrophic consequences as proved by the recent security issues of Zcash implementing custom zk-SNARK setups in production. The gap between a theoretical implementation and an actual production[6](#References) setup requires the highest level of caution. Even though security proofs provide higher security guarantees they can pass the filter of peer review while further research might find they’re false, as highlighted recently by Koblitz and Menezes[7](#References).

We believe that MPC-only systems are not mature enough from a security track-record perspective to be put to use extensively in a production environment.

# Implementing complex governance schemes remains out of reach for solutions relying solely on MPC

The main technical limitation of MPC is the exponential complexity of existing algorithms. The more complex the calculation, the more difficult and slower it will be to compute. Even simple “m of n” quorums will become practically impossible to implement if the number of signers increases, more so if the computation performed is already computational-intensive without MPC. Recent papers on the topic have improved performance but the research is very new (improvements applied to transaction signature started roughly in 2016[8](#References)) and has yet to be tested thoroughly.

This computational limitation means that some operations are extremely hard to implement using only MPC or unusable due to high computational requirements. Specifically:

* Key derivation algorithms to create public/private key pairs: Implementing an MPC version of BIP-32 becomes unworkable with more than 2 signers because of the computational complexity.
* TSS-based approval quorums with many approvers.

Although these limitations might be overcome by using more powerful hardware, attacks on the system running the MPC computation must withstand extensive software and hardware-based attacks.

It is important to point out that the security of digital assets goes beyond defining approval quorums and a good solution must encompass a way to implement governance rules over the signature process such as specific approval flows defined by amount thresholds and address whitelisting. These schemes are currently out of reach for MPC-based solutions.

To circumvent the issue, governance rules can be implemented in another environment. Those can be hardened servers, HSMs, or secure enclaves. While this approach is pragmatic it can significantly undermine the security enhancement of MPC setups if the environment implementing the governance is not highly secured.

The lack of secure execution environment for key lifecycle activities not encompassed by the MPC schemes can hinder the management of wallets at scale in fast growing institutions. Indeed, besides the limitation of governance schemes, other essential processes cannot be performed using MPC alone. For instance, using MPC protocols there are methods to update a quorum and revoke keys but the dissemination of new keys, must still be performed outside of the scope of MPC. These processes must be enforced via regular business processes.

Another point of concern is trusted display. MPC does not inherently allow signers to securely review what they are signing. Should an attacker push a rogue transaction to the MPC quorum by targeting one of the endpoints, the other signers will have no way to securely verify what is being signed. Signers rely on regular hardware running the MPC software which means they can be targeted to display inacurate information from a malicious source.
# What is the way forward?

MPC schemes open up new possibilities to secure parts of the key management lifecycle but current schemes are not satisfactory from the standpoint of performance, flexibility and security track-record. As such, we believe MPC-only solutions are not ready for production. MPC should be used cautiously until it has passed the test of time and never as the unique basis of a security infrastructure.

We can imagine solutions where the pitfalls of MPC are addressed through the complementary implementation of a secure execution environments and secure hardware providing tampering protection and trusted display.

At Ledger, we are convinced that the key generation process must always be performed on secure hardware otherwise the entropy of the keys will be too low and subject to attacks. To ensure keys are securely generated, certified secure hardware using true random number generators is still the best solution to avoid this attack vector. Keys must also be stored on secure hardware: the interest of MPC lies in distributing the keys on several endpoints which distributes risks but attacking several unsecured endpoints in synchronization is a realistic attack vector, as recent hacks have proved[9](#References).

Furthermore, a secure execution framework within secure hardware as we currently offer within our solutions such as the Vault would enable to protect much-needed governance schemes and user management frameworks. MPC would then be a solution to reduce the operational risk related to the concentration of keys on a single technical platform.

The implementation of MPC in secure hardware remains a missing piece of the puzzle in order to address the limitations of the technology. We thereby warn that the limited functional scope for governance schemes and the incomplete key lifecycle management capabilities of current MPC-based systems should be carefully taken into consideration as institutions implement MPC in its current form will remain exposed to significant operational risks.

MPC will be a great addition to the arsenal of security solutions made available with secure hardware, and Ledger will investigate integrating MPC to our existing portfolio of security solutions to create a more robust end-to-end security solution to protect critical digital assets.

# References
<a name="References"></a>
* 1:https://www.inpher.io/technology/what-is-secure-multiparty-computation
* 2:https://www.binance.vision/security/threshold-signatures-explained
* 3:https://csrc.nist.gov/Projects/Threshold-Cryptography
* 4:https://eprint.iacr.org/2019/074.pdf
* 5:https://blog.trailofbits.com/2019/09/11/crypto-2019-takeaways/
* 6:https://electriccoin.co/blog/zcash-counterfeiting-vulnerability-successfully-remediated/#counterfeiting-vulnerability-details
* 7:https://eprint.iacr.org/2019/1336
* 8:https://eprint.iacr.org/2016/013
* 9:https://www.wired.com/story/ios-attack-watering-hole-project-zero/






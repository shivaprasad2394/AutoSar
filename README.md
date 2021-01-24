# AutoSar

https://blog.csdn.net/wujianing_110117/article/details/107186425

|Crypto |The Crypto Drivers allow defining of different Crypto Driver Objects (i.e. AES accelerator, SW component, etc), which shall be used for concurrent requests in different buffers.
------- |-------------
| CryIf |	The Crypto Interface module provides a unique interface to manage different Crypto HW and SW solutions like HSM, SHE or SW-based CDD.
| Csm |	The Crypto Service Manager (Csm) provides services to enable access to basic cryptographic functionalities for all software modules and SWCs.


How to enable AUTOSAR in the hardware security module
The hardware security module (HSM) has appropriate firmware, which can guarantee the encryption of the system even in the case of insufficient resources.



for example, to verify communication partners and communication content and prevent interception. Proper implementation must allow real-time requirements. At first glance, this contradicts the computational time requirements of cryptographic methods. The hardware security module (HSM) is used to solve this problem. Allows to calculate the password on a separate processor. However, an optimized implementation is needed to realize its full potential.

In order to meet and realize the real-time requirements of the software, the AUTOSAR operating system divides the relevant software functions into sub-steps, so-called tasks. The task is activated by an external event, that is, marked as runnable, and then processed according to priority. For most applications, a timer with a cyclic activation table is used to activate the task, because the related control function will continue to process new signal generation. 

This will result in a default time response, which repeats in a period of tens of milliseconds. Several steps are required to ensure that real-time requirements are met. On the one hand, when implementing signal processing steps, appropriate measures must of course ensure that the required time response is provided. On the other hand, the activation of time-related tasks in the ECU must be able to meet the time requirements. For this reason, it is necessary to analyze the runtime behavior of all tasks to find an appropriate priority ordering and a chronological activation sequence with guaranteed time.

AUTOSAR basic software

The function of the ECU no longer only involves signal processing. The basic software that handles communication and management is an important part of these functions.  In short, 
there are two different sets of functions: 
A part of the basic software, for example for transmitting signal values, is necessary for real-time signal processing. This part must be given high priority appropriately. 
The other part has significantly lower real-time requirements: this part must run in the background regularly; however, it may be temporarily preempted by functions with higher real-time requirements.

Therefore, a common implementation pattern is to transfer these non-critical functions to low-priority background tasks. In this case, the problem is the order of priority between functions: Generally speaking, there are multiple background activities, none of which can fail for a long time. 

The solution to this sort of priority is round-robin scheduling, where each low-priority function is calculated for a short time, and then the next function. In AUTOSAR terminology, each module involved provides a so-called "MainFunction" with a limited running time. In background tasks, these functions are called in sequence.

Security

Safety

An example of this classic "background function" is a cryptographic algorithm. Since version 4.0, AUTOSAR has included the specifications of the encryption basic software-in subsequent versions, the corresponding specifications have been revised and fine-tuned. In this case, the so-called cryptographic service manager (CSM) provides cryptographic services to the application: 

 ![basic arcitecture](https://github.com/shivaprasad2394/AutoSar/blob/main/autosar_csm.png)
 
Figure 1 Security module in AUTOSAR 4.3 system

For example, it is used by Secure Onboard Communication (SecOC) to verify the signal value in the transmitted data packet in an encrypted manner. As the name suggests, CSM manages encryption services. The actual implementation is in a manufacturer-specific encryption library. In order to support integration into AUTOSAR systems with real-time requirements, CSM processes requests asynchronously: initially, they are only saved, and then processed one by one in the call of CSM MainFunction. To this end, the CSM MainFunction will call the mainformations of all the underlying encryption primitives, and each primitive will then have to calculate several steps.

The calculation time of a typical cryptographic primitive is an order of magnitude higher than the calculation time of a signal processing function. This brings up a dilemma when dividing MainFunction calls: What is the exact range allowed for calculation? Too many calculation steps will make the real-time performance of the entire system questioned, making encryption useless. Too few steps will delay the calculation of longer encryption operations, so that the benefits are limited.

In addition, another effect must be considered: a specific main function is responsible for managing its internal calculation state in order to allow the calculation to be performed gradually. On its own, this state management means overhead. The fewer the actual calculations for a single call, the greater the overhead.

Software vendors should find a standard solution to this problem. In classic scenarios, for example, runtime encryption is used to authenticate smaller data blocks through a symmetric encryption method. One solution might be to calculate a symmetric block for each MainFunction call. However, when using more complex methods, it is difficult to find a reasonable compromise.

Examples include verifying/encrypting large amounts of data or generating asymmetric signatures. For example, assuming that 100µs is calculated every millisecond is acceptable, this is a very optimistic assumption and it is difficult to maintain in most real-time scenarios.

an obvious solution is to use specialized hardware that can compute the appropriate algorithms-or most of them-in parallel with the main processor. Then AUTOSAR CSM and related encryption libraries only pass the request to this hardware, and in the main function, periodically check whether the result is available. The first of these hardware coprocessors has been designated as a "safe hardware extension" by the manufacturer's software program (HIS) in the past decade. Regarding cryptographic algorithms, this specification is still limited to the implementation of AES-128 in different modes. Therefore, recent developments are often due to a large number of hardware developments or not ideal, so more pure hardware is possible.


13.
The interface layer module of the CRYIF cryptographic service unifies the driver interface of the hardware encryption module and the software encryption module, which can be called downwards, crypto (SW) and crypto (HW).
14.
Crypto (SW) software-implemented encryption algorithm , Such as AES-128 and other algorithms.
15.
The driver of crypto (HW) hardware encryption module. This program usually contains a module in the driver of the chip MCAL, which is used to control HSM (hardware confidential module) or SHE, which is related to the specific chip, such as Freescale's 574x Series chips have HSM module.
16. HSM
is not a software module, but a hardware acceleration engine of a hardware chip to implement hardware encryption algorithms. The advantage is that it does not consume cpu resources, and the calculation speed is fast, which may be hundreds of times faster than software calculations.


https://translate.googleusercontent.com/translate_c?depth=1&hl=en&prev=search&pto=aue&rurl=translate.google.com&sl=zh-CN&sp=nmt4&u=https://blog.csdn.net/DJAction/article/details/105508687&usg=ALkJrhiO1CnmiBLJHsMmZkIKv51ZpqZnbw&utm_medium=distribute.pc_relevant.none-task-blog-searchFromBaidu-13.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-searchFromBaidu-13.control
I. Overview

 

    The AUTOSAR adaptive platform supports APIs for general encryption operations and security key management. The API supports dynamic generation of keys and encryption jobs at runtime, as well as operations on data streams. In order to reduce storage requirements, the key can be stored internally in the encrypted backend, or it can be stored externally and imported on demand.

    The API is designed to support the encapsulation of security-sensitive operations and decisions in separate components, such as hardware security modules (HSM). Additional protection of keys and key usage can be provided by restricting the key to a specific purpose (for example, decryption only), or restricting the key to the use of a single application as reported by IAM.

    Depending on application support, APIs can also be used to protect session keys and intermediate secrets when dealing with encryption protocols such as TLS and SecOC.

 
Two, security architecture

 

    Although AUTOSAR AP only defines the high-level Crypto Stack API that is exposed to the application, the definition of this API must take into account the security architecture designed to meet the above-mentioned security and functional requirements. The general architecture is shown in Figure 1.

    At the highest level, AUTOSAR AP and native and hybrid applications are linked to the AUTOSAR AP encryption stack API. API implementation can reference a central unit (cryptographic service manager) to implement platform-level tasks, such as consistent access control and certificate storage across applications.

    The implementation can also use a cryptographic service manager to coordinate the offloading of functions to cryptographic drivers, such as a hardware security module (HSM). Indeed, this way of uninstalling the Crypto Stack API is expected to become a typical implementation strategy: Crypto Driver can implement a complete set of key management and encryption functions to speed up encryption operations and protect escrow keys from malicious applications.
    Figure 1 Encryption stack-reference architecture

    In order to achieve this layered security architecture, the Crypto Stack API not only performs bulk encryption operations, but also provides native support for the following aspects:

    (1) Use encrypted keys or key handles to operate

    (2) Manage keys securely although it may harm the application

    (3) Restrict application access to keys and allowed operations

 
Three, key management architecture

 

    In order to support secure remote management of keys in the event of potential application damage, Crypto Stack integrates a key management architecture, in which keys and related data are managed in an end-to-end protected form. The key can be introduced into the system in a trusted manner based on an existing supply key, or it can be introduced into the system in an untrusted manner through local key generation.

    Assuming that the encryption backend/driver is properly secured, the application will not be able to modify the key unless through a well-defined authorization request (such as key update or revocation).
    Figure 2 CKI key management interaction

     

 
Four, API extension description

 

    Important new usages and interactions that require the introduction of new or modified permission/policy verification logic should be associated with the corresponding new key usage policy flags. For example, alternative supply keys with different ownership/authority checks can be introduced by adding corresponding new key usage policies and enforcing new logic in all key management operations involving those new keys.

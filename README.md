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
 

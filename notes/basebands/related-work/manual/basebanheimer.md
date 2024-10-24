# Basebanheimer

- Reverse engineering the Exynos baseband (Shannon)

## Found Vulnerabilities in Shannon

### LTE bearer context setup:

- The number of Information Elements for a Traffic Flow Template Message was specified implicitly in the documentation
- The implementation does allocate a number of elements that is too small compared to the specification
- Result: Buffer overflow on the heap

#### Debugging the Heap Layout and Creating a Vulnerability

- The UE provides RAM dumps for understanding the Heap layout
    - It also records all events of the heap allocaitons in a ring-buffer for convenient debugging
- The heap can be shaped with TFT (Traffic Flow Template) messages
- The overflow from the TFT IE bug was not detailed unfortunately

### Baseeband to Application Processor Bugs

- Radio-Interface Layer (RIL) is the largest attack surface from CP to AP
- Previously, the direction was from AP to BB to perform vendor unlocking
- AT commands exist and are still supported, but many vendors have their own implementation/extension (Qualcomm QMI; Samsung SIPC)

#### Debugging Android Kernel Code

- standard ELF; shared libraries etc. make reversing easier
- Bugs were found (stack and heap buffer overflows) though they could not be exploited due to strong mitigations of Android Userspace


## Found Vulnerability in MediaTek

### Layer 2: Segmentation implementation

- The implementation does not account for a maximum number of segments - heap buffer overflow
- Could be exploited, but no rich debugging capabilities as in Shannon

### BB to AP bugs

- Try to exploit the kernel CCCI ring-buffer implementation
    - The BB can control the parameters and data written into the shared memory
    - The implementation in the kernel does not handle improper lengths for writing in the ring-buffer
- Enables arbitrary write in the kernel
    - 32 Bit kernels without KASLR are straightforward to exploit (hard-coded addresses)
    - 64 Bit kernels with KASLR are harder to exploit but was possible

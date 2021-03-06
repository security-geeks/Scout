Compilation Modes
=================
Scout is a configurable debugger, that could be deployed in several different environments:

* Linux User-Mode Process - "User Mode"
* Linux Kernel Driver - "Kernel Mode"
* Linux In-Process Debugging - "User Mode" + "PIC Mode"
* Embedded "In-Process" Debugging (low privileges) - "PIC Mode" + "User Mode"
* Embedded "In-Process" Debugging (high privileges) - "PIC Mode" + "Kernel Mode"

To decide what will be the suitable compilation mode / architecture flags, one should check the following parameters.
Each of the defined parameters is a C MACRO (define) that controls the behavior (and compilation) of the resulting binary.

**Important Note:**
When using ``scoutCompiler``, it will automatically generate most of the needed flags by deducing the right values based on the architecture and configuration flags that are supplied to it. Please see the ``embedded_scout`` example for more info.

Target Endianness
-----------------
* SCOUT_BIG_ENDIAN - Scout is executed on a Big Endian architecture
* SCOUT_LITTLE_ENDIAN - Scout is executed on a Little Endian architecture

Only one of above flags can be defined.
If none are defined the base library will define "SCOUT_LITTLE_ENDIAN" on it's own.

Target Bitness
--------------
* SCOUT_BITS_32 - Scout is executed on a 32 bit machine
* SCOUT_BITS_64 - Scout is executed on a 64 bit machine

Only one of above flags can be defined.
If none are defined the base library will define "SCOUT_BITS_32" on it's own.

Target CPU Architecture
-----------------------
* SCOUT_ARCH_INTEL - Scout is executed on an Intel (x86 \ x64) CPU
* SCOUT_ARCH_ARM   - Scout is executed on an ARM (maybe thumb mode) CPU
* SCOUT_ARCH_MIPS  - Scout is executed on a  MIPS (*not* mips16 mode) CPU

Only one of above flags can be defined.
If none are defined the base library will define "SCOUT_ARCH_INTEL" on it's own.

**Additional Flags:**
SCOUT_ARM_THUMB - Scout will be executed on an ARM cpu in Thumb mode. Can only be used together with the "SCOUT_ARCH_ARM" flag.

The flags are needed only in PIC mode, in which we use inline assembly.

Target Permission Level
-----------------------
* SCOUT_MODE_USER - Scout is executed in User-Mode (& low CPU privileges)
* SCOUT_MODE_KERNEL - Scout is executed in Kernel-Mode (& high CPU privileges)

Only one of above flags can be defined.
If none are defined the base library will define "SCOUT_MODE_USER" on it's own.

"SCOUT_MODE_KERNEL" will also be the right choice for an RTOS (Real-time OS) in which every task / our task has high privileges. The flag will lead to the definition of "SCOUT_HIGH_PRIVILEGES" by the compilation environment.

**Important Note:** As flushing the CPU cache usually requires high privileges, the per-architecture implementation will only be available if compiling using the "SCOUT_MODE_KERNEL" flag.

Position Independent Mode - SCOUT_PIC_CODE
------------------------------------------
Scout will be compiled for full Position Independent Code (PIC) mode. Any access to an external function / global variable will pass through a unique "Context" object. Read the section about "PIC Compilation" for more information.

"SCOUT_PIC_CODE" will lead to the definition of "SCOUT_ISOLATED_ENV" by the compilation environment, because a PIC blob will always be isolated from the environment, and won't have the luxory of a proper executable loader such as "ld.so".

Host LibC Implementation
------------------------
When injecting our (PIC) code into a host binary, we should make sure to use the proper constants for the matching standard library implementation that is used by the respective binary:
* SCOUT_HOST_GLIBC - The used library is Glibc
* SCOUT_HOST_UCLIBC - The used library is uClibc (or uClibc-NG)

If none are defined the base library will define "SCOUT_HOST_GLIBC" on it's own.

Loader Flags
------------
* SCOUT_LOADER - We are now compiling a loader (that might be using it's own pic plt / globals).
* SCOUT_LOADING_THUMB_CODE - The loader will load a Scout that was compiled to be executed on an ARM cpu in Thumb mode.
* SCOUT_RESTORE_FLOW - The default loaders (``tcp_client_server.c``, ``tcp_loader_server.c``) will clean-up after themselves if the loaded scout will finish the endless loop.

If the loader will be compiled to be Position Independent (PIC), which is probably the most common use case, it will also define a new flag of "SCOUT_SLIM_SIZE", to help shrink the size of the binary (to serve as an effective shellcode).
Under this definition the TCP server would expect the following flags (if needed):
* SCOUT_TCP_CLIENT - There is a need to include the feature of a TCP client
* SCOUT_TCP_SERVER - There is a need to include the feature of a TCP server
* SCOUT_TCP_SEND - There is a need to include the ability to reliably send TCP messages

Additional Flags:
-----------------
* SCOUT_INSTRUCTIONS - Scout is going to use the instructions api (using the TCP server for instance)
* SCOUT_DYNAMIC_BUFFERS - Scout will dynamically ``malloc()`` buffers to be used by the tcp server. Otherwise static buffers will be used.
* SCOUT_PROXY - Scout is going to act as a proxy (user scout passing instructions to a kernel driver for instance)
* SCOUT_MMAP - Should scout's loaders use ``mmap()`` and ``mprotect()`` when loading (if defined) or should they simply use ``malloc()`` (if undefined)

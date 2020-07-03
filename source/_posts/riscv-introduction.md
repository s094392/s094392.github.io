---
title: RISC-V Introduction
date: 2020-07-04 00:01:38
tags: [Architecture]
---
## Introduction
RISC-V is an open standard Instruction Set Architecture. Unlike most other ISA designs, the RISC-V ISA is provided under open source licenses that do not require fees to use. The RISC-V was first introduced at 2010 by David Patterson at Berkeley. RISC-V is widely adopted by a lot of companies because the features like modular design and open source licenses fits current environment of IoT and Machine learning edge devices.
{% asset_img riscv.png [riscv-introduction] %}

## Key Features
The RISC-V is a free, open and lightweight ISA supervised by a nonprofit foundation, which means you don’t need to spend plenty of money to buy a license and are free to modified and adjust the ISA to meet your requirement. Furthermore, The RISC-V is modular designed and provides several standard ISA set for user to adopt. Also, the processor designers are free to create their own ISA sets.
A processor designer should choose a Base ISA and multiple Extensions ISA.
- Base ISAs
    - RV32I
    - RV64I
    - RV32E
    - RV128I
- Extensions
    - M Standard (Integer Multiplication and Divison)
    - A Standard (Atomic Instructions)
    - F Standard (Single-Precision Floating-Point)
    - D Standard (Double-Precision Floating-Point)
    - C Standard (Compressed Instruction)

### 
For example, FU540, a processor produced by SiFive, is labeled “RV64IMAC”. Which means this 64-Bit processor is adopted M Standard, A Standard and C Standard.

## Privileged Architecture
The privileged architecture in RISC-V contains three privileged levels: Machine, Supervisor and User. Privilege levels are used to provide protection between different components of the software stack, and attempts to perform operations not permitted by the current privilege mode will cause an exception raised.
    
| Level    | Encoding | Name             | Abbreviation |
| -------- | -------- | ---------------- | ------------ |
| 0        | 00       | User/Application | U            |
| 1        | 01       | Supervisor       | S            |
| 2        | 10       | Reserved         |              |
| 3        | 11       | Machine          | M            |
    
The Machine level has the highest privileges and is the only mandatory privilege level for a RISC-V hardware platform. The supervisor level can’t access crucial configuration of the processor but has the privilege to control the virtual memory addressing. User level has least privilege and usually runs untrusted program.
    
Each level has corresponding core set of ISA and optional extensions. Privileged instructions and CSRs (control and status registers) are executable and accessible only from appropriate mode (or higher). For example, the register mstatus is an important CSR that contains machine level configurations and information. So, this register is accessible only from Machine level.
    
Processor designers could determine the adaption of these three levels based on their requirements (But Machine level is mandatory). That is, the simplest RISC-V implementations may provide only M-mode, though this will provide no protection against incorrect or malicious application code.

| Number of levels | Supported Modes | Intended Usage                              |
| ---------------- | --------------- | ------------------------------------------- |
| 1                | M               | Simple embedded systems                     |
| 2                | M, U            | Secure embedded systems                     |
| 3                | M, S, U         | Systems running Unix-like operating systems |

## Next..
The topic of next post would be how exception and interrupt work in RISC-V.

## References
1. [The RISC-V Instruction Set Manual, Volume I: Unprivileged Architecture](https://content.riscv.org/wp-content/uploads/2019/06/riscv-spec.pdf)
2. [The RISC-V Instruction Set Manual, Volume II: Privileged Architecture](https://content.riscv.org/wp-content/uploads/2019/12/riscv-spec-20191213.pdf)

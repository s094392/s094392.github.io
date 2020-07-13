---
title: RISC-V Exception and Interrupt implementation
date: 2020-07-09 23:36:09
tags:
categories: [Architecture]
---
## Exception and Interrupt
When the core enters a trap, the core will store current state, the cause and address of current instruction to corresponding register and Jump to the handler table stored in register mtvec.
{% asset_img mtvec.png [riscv-exception-interrupt] %}
The register *mcause* stores the cause of current trap. The Interrupt bit in the *mcause* register is set if the trap was caused by an interrupt. The Exception Code field contains a code identifying the last exception. For example, Exception Code 8 means there is a environment call from User mode (like svc in arm) and Exception Code 2 means the processor meet an illegal instruction.

### Exception
We can use the ECALL instruction to raise an exception for higher privilege request. For example, the implementation of system call. The ECALL instruction will update following registers with current status:
1. update *mcause* with is_interuupt and exception_code
2.	update *mtval* with exception-specific information
3.	update *mstatus* with current mode
4.	update *mstatus* to disable interrupt
5.	update *mepc* with current instruction address

###	Interrupt
We can enable interrupts in every mode by setting corresponding interrupt-enable register (MIE/SIE/UIE). For example, we can enable machine timer interrupt by raising the MTIE bit in MIE register.
{% asset_img mie.png [riscv-exception-interrupt] %}

Once an interrupt raised, the corresponding bit in the interrupt-pending register will also be raised and waiting the interrupt handler to reset it.

## Exception and Interrupt – implementation in C
I've implemented a exception and interrupt example in RISC-V on qemu emulator. The example works as following steps:
1.	Switch to U-mode before the main function
2.	Raise an exception to enable timer interrupt
3.	Handle exception and interrupt
First, in the startup script, we add some code to load trap handle vector to the register mtvec and switch to User mode. 
```as
   la     t0, trap_entry
   csrw   mtvec, t0
   lla t0, 1f
   csrw mepc, t0
   mret
1:
   call   main
```
In the trap handle vector, we should manually backup and restore current context with the stack and use MRET to return to User mode.
```as
.align 2
trap_entry:
    addi sp, sp, -17*4
  
    sw ra, 0*REGBYTES(sp)
    sw a0, 1*REGBYTES(sp)
    sw a1, 2*REGBYTES(sp)
    sw a2, 3*REGBYTES(sp)
    sw a3, 4*REGBYTES(sp)
    sw a4, 5*REGBYTES(sp)
    sw a5, 6*REGBYTES(sp)
    sw a6, 7*REGBYTES(sp)
    sw a7, 8*REGBYTES(sp)
    sw t0, 9*REGBYTES(sp)
    sw t1, 10*REGBYTES(sp)
    sw t2, 11*REGBYTES(sp)
    sw t3, 12*REGBYTES(sp)
    sw t4, 13*REGBYTES(sp)
    sw t5, 14*REGBYTES(sp)
    sw t6, 15*REGBYTES(sp)

    jal handle_trap
  
    lw ra, 0*REGBYTES(sp)
    lw a0, 1*REGBYTES(sp)
    lw a1, 2*REGBYTES(sp)
    lw a2, 3*REGBYTES(sp)
    lw a3, 4*REGBYTES(sp)
    lw a4, 5*REGBYTES(sp)
    lw a5, 6*REGBYTES(sp)
    lw a6, 7*REGBYTES(sp)
    lw a7, 8*REGBYTES(sp)
    lw t0, 9*REGBYTES(sp)
    lw t1, 10*REGBYTES(sp)
    lw t2, 11*REGBYTES(sp)
    lw t3, 12*REGBYTES(sp)
    lw t4, 13*REGBYTES(sp)
    lw t5, 14*REGBYTES(sp)
    lw t6, 15*REGBYTES(sp)

    addi sp, sp, 17*4

    mret
```
In the main function we just initialize the Uart for printing messages and use ECALL instruction to raise an exception for enable timer interrupt.
```c
int main() {
    uart_init();
    print_s("Hello world!\n");
    print_s("Raise exception to enable timer...\n");
    asm volatile("ecall");
    print_s("Back to user mode\n");
    while (1)
        ;
    return 0;
}
```
The only thing trap handler does is to determine whether the incoming trap is an interrupt or not and call appropriate handler. To be noticed, the register *mepc* stores the address of instruction before entering a trap. In our case, the address would be the address of the instruction ECALL. So, we need to modified the *mepc* to the address of next instruction after the exception handler.
```c
void handle_trap() {
    uint64_t mcause, mepc;
    asm volatile("csrr %0, mcause" : "=r"(mcause));
    asm volatile("csrr %0, mepc" : "=r"(mepc));

    if (mcause >> 63) {
        handle_interrupt(mcause);
    } else {
        handle_exception(mcause);
        asm volatile("csrr t0, mepc");
        asm volatile("addi t0, t0, 0x4");
        asm volatile("csrw mepc, t0");
    }
}
```
In the exception handler, we need to enable the timer interrupt by set the MTIE bit in the MIE (Machine interrupt-enable register) to 1. The timer interrupt when the machine time counter *mtime* >= register *mtimecmp*. And once we modified the register *mtimecmp*, the MTIP bit in the MIP (Machine interrupt-pending register) would be cleared automatically.
```c
void handle_exception(uint64_t mcause) {
    unsigned long long int mie;

    if (mcause == 0x8) {
        *MTIMECMP = *MTIME + 0xfffff * 5;

        asm volatile("csrr %0, mie" : "=r"(mie));
        mie |= (1 << 7);
        asm volatile("csrw mie, %0" : "=r"(mie));
    } else {
        print_s("Unknown exception: ");
        print_i(mcause << 1 >> 1);
        print_s("\n");
        while (1)
            ;
    }
}
```
In the interrupt handler, we only need to reset the register mtimecmp.
```c
void handle_interrupt(uint64_t mcause) {
    if ((mcause << 1 >> 1) == 0x7) {
        print_s("Timer Interrupt: ");
        print_i(++count);
        print_s("\n");

        *MTIMECMP = *MTIME + 0xfffff * 5;
        if (count == 10) {
            unsigned long long int mie;
            asm volatile("csrr %0, mie" : "=r"(mie));
            mie &= ~(1 << 7);
            asm volatile("csrw mie, %0" : "=r"(mie));
        }
    } else {
        print_s("Unknown interrupt: ");
        print_i(mcause << 1 >> 1);
        print_s("\n");
        while (1)
            ;
    }
}
```
The result:
{% asset_img result.png [riscv-exception-interrupt] %}

## How to run my code
The source code is available on my github:
[https://github.com/s094392/riscv-bare-metal](https://github.com/s094392/riscv-bare-metal)
This bare metal example runs on qemu and you should install required toolchain of riscv64.
### Requirement
- qemu
- riscv64-linux-gnu-*

### Run
```bash
make
make run
```

### Debug
```bash
make
make debug
riscv64-linux-gnu-gdb -x debug.txt
```
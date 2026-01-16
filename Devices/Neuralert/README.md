# Compile the Firmware

* Follow the instructions in the [neuralert_firmware](https://github.com/IoMT-Lab/neuralert_firmware.git) repository.

* The output elf firmware locates under `build/`:

    * neuralert.elf

# Run the Firmware on Renode

* Install Renode

    * Download the [release version](https://github.com/renode/renode/releases/tag/v1.16.0) and add it to your `$PATH`.

* The file `neuralert.repl` specifies the memory mapping of the board.

* The file `neuralert.resc` specifies how the firmware is loaded and executed. Please modify the path in `sysbus LoadELF @../build/neuralert.elf true` to your `neuralert.elf` path.

* Why renode?

    * Qemu does not support the DA16200 board. It may be easier to create the memory mapping and peripheral in the renode.

* Some information about DA16200 Board

    * Here is the board manual: https://datasheet.octopart.com/DA16600MOD-DEVKT-Renesas-datasheet-179201150.pdf

    * In source code, you can find the memory mapping in `./da16200_sdk/core/bsp/ldscripts/mem.ld`

# How to run

* Run the following command:
    ```
    renode neuralert.resc
    ```

* **Current issue**:
    ```shell
    # stop at pc 10155a (inside UsageFault_Handler)
    22:37:59.9711 [ERROR] cpu: write access to unsupported AArch32 64 bit system register cp:0 opc1:0 crm:8 (privilege)
    ```

    GDB outputs:
    ```shell
    info stack
    0x0010155a in UsageFault_Handler_C ()
    (gdb) info stack
    #0  0x0010155a in UsageFault_Handler_C ()
    #1  <signal handler called>
    #2  0x0010142c in __isr_vectors ()
    #3  0x00134e3e in init_system_step0pre ()
    #4  0x0010158a in __start ()
    #5  0x0010165a in Reset_Handler ()
    ```
    The system stops at `UsageFault_Handler_C()`, but it is not the root cause. The root cause is unlikely in `__isr_vectors()`, because it is a vector table and the value inside it should be a jump target; therefore, it shouldn't be executed directly. The root cause may stay in `init_system_step0pre()`. The `0x00134e3e` is the return address, so the last executed instruction is `bl      0x8f970 <_sys_nvic_create>`. However, the `_sys_nvic_create` is all `0` (i.e., disassemble as `movs    r0, r0` in GDB) in memory. That is, some code sections are not loaded correctly. 
    ```shell
    disassemble 
    Dump of assembler code for function init_system_step0pre:
    0x00134e38 <+0>:     push    {r3, lr}
    0x00134e3a <+2>:     bl      0x8f970 <_sys_nvic_create>
    => 0x00134e3e <+6>:     ldr     r0, [pc, #12]   ; (0x134e4c <init_system_step0pre+20>)
    0x00134e40 <+8>:     bl      0x8f984 <_sys_nvic_init>
    0x00134e44 <+12>:    ldmia.w sp!, {r3, lr}
    0x00134e48 <+16>:    b.w     0x107760 <da16x_zeroing_init>
    0x00134e4c <+20>:    asrs    r0, r0, #16
    0x00134e4e <+22>:    movs    r0, r2

    Dump of assembler code for function _sys_nvic_create:
    0x0008f970 <+0>:     movs    r0, r0
    0x0008f972 <+2>:     movs    r0, r0
    0x0008f974 <+4>:     movs    r0, r0
    0x0008f976 <+6>:     movs    r0, r0
    0x0008f978 <+8>:     movs    r0, r0
    0x0008f97a <+10>:    movs    r0, r0
    0x0008f97c <+12>:    movs    r0, r0
    0x0008f97e <+14>:    movs    r0, r0
    0x0008f980 <+16>:    movs    r0, r0
    0x0008f982 <+18>:    movs    r0, r0
    ```
    **Next step**: check the binary load process (`sysbus LoadELF`) and memory mapping in `repl` to ensure the code section is loaded correctly into the memory. 
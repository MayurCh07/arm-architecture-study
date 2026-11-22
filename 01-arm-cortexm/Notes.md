# Notes

## CMSIS (Cortex Microcontroller Software Interface Standard)

What is CMSIS?

CMSIS is a standard software layer created by ARM that provides a common interface to Cortex-M processors.

```
Application
    ↓
HAL / LL (Optional)
    ↓
CMSIS
    ↓
Cortex-M Core
```

CMSIS allows developers to access core processor features in a vendor-independent way.

Examples:
- NVIC_EnableIRQ();
- SysTick_Config();

These APIs work similarly across STM32, NXP, Kinetis, EFM32, and other Cortex-M devices.

Typical Project Structure

```
Drivers/
├── CMSIS/
├── STM32F4xx_HAL_Driver/
└── ...
```

## Main CMSIS Components

### Core

Contains Cortex-M core definitions and functionality:

- NVIC
- SysTick
- SCB
- MPU
- Core Registers
- Exception Handling

Common files:
- core_cm4.h
- core_cm7.h

### Device

Contains device-specific definitions.

Example:
- stm32f401xe.h

Provides access to:
- GPIOA
- GPIOB
- RCC
- USART1

This enables code such as:

```c
GPIOC->ODR |= (1 << 13);
```

instead of using raw memory addresses.

## Why CMSIS Matters

CMSIS helps in understanding:

- Cortex-M execution model
- Vector table concepts
- Interrupt architecture
- NVIC
- SysTick
- Exception handling
- MPU
- Core registers

## Startup File

"startup_stm32f401xc.s" is assembly file that runs before C "main()".

It has three main jobs:
- Set up the processor’s initial execution state.
- Initialize the parts of RAM that C code expects initialized.
- Tell the processor where to go for reset, faults, and interrupts.

Process is:
```
Power-on/reset
    → CPU reads the first two vector-table entries
    → Reset_Handler
    → SystemInit()
    → initialize .data and .bss
    → C runtime initialization
    → main()
```

### Assembler directives and symbols

directives tell the assembler what instruction syntax and target architecture to use:
- .syntax unified selects the unified ARM/Thumb assembly syntax.
- .cpu cortex-m4 identifies the processor.
- .fpu softvfp is an assembler setting; it does not set the C compiler’s floating-point ABI.
- .thumb selects Thumb instructions, which Cortex-M processors execute.

### Reset_Handler: getting ready for C

The linker script marks Reset_Handler as the entry point:
- Set the stack pointer (it loads sp from _estack.)

The stack is used for function calls, local variables, saved registers and other temporary data. _estack is defined by the linker script as the top of RAM. The processor also initially loads the stack pointer from the vector table’s first entry, so this instruction sets it again explicitly.

#### Call SystemInit()

The startup code calls SystemInit().

In this project, that function is in system_stm32f4xx.c. It handles early system-level setup, such as FPU access and optional vector-table relocation. The application’s SystemClock_Config(), in main.c configures the clocks later.

#### Copy initialized variables into RAM
The .data section contains global or static variables that have initial values.

The initial value is stored in FLASH, but the variable must live in RAM while the program runs. The startup loop copies the initial bytes from FLASH to RAM.

The linker script supplies the addresses:

_sidata: where the initial values are stored in FLASH
_sdata: start of the destination section in RAM
_edata: end of the destination section

#### Zero uninitialized variables

The .bss section contains global or static variables without explicit initial values. C requires these variables to start at zero. 

The loop writes zero through the .bss range:

```
FillZerobss:
  str  r3, [r2]
  adds r2, r2, #4

LoopFillZerobss:
  cmp r2, r4
  bcc FillZerobss
```
Without the .data copy and .bss clearing, C variables may begin with incorrect values after reset.

#### Initialize the C runtime and call main()
```
/* Call static constructors */
    bl __libc_init_array
```

__libc_init_array runs C-library initialization routines. It is also important for C++ static constructors. It comes from the embedded C runtime linked into the firmware.
```
/* Call the application's entry point.*/
  bl  main   
```
The routine branches to main().

Project’s main() then configures the application and runs its loop.

### Default_Handler:

what happens for an unimplemented interrupt
```
    .section  .text.Default_Handler,"ax",%progbits
Default_Handler:
Infinite_Loop:
  b  Infinite_Loop
  .size  Default_Handler, .-Default_Handler
```

The default handler loops forever.

That behavior is useful for debugging: if an interrupt fires unexpectedly, the processor stops making normal progress instead of continuing with an unknown state. We can halt the debugger and inspect where it stopped.

### The interrupt vector table

```
   .section  .isr_vector,"a",%progbits
  .type  g_pfnVectors, %object
    
g_pfnVectors:
  .word  _estack
  .word  Reset_Handler
  .word  NMI_Handler
  .word  HardFault_Handler
  .word  MemManage_Handler
  .word  BusFault_Handler
  .word  UsageFault_Handler
  ...
  ...
  ...
.size  g_pfnVectors, .-g_pfnVectors
```

In a section named .isr_vector. Each .word stores one 32-bit entry.

The linker script places that section in FLASH and uses KEEP to ensure it is not discarded (line 76). On reset, the processor reads:

- Entry 0: initial stack pointer (_estack)
- Entry 1: reset-handler address (Reset_Handler)

The following entries correspond to processor exceptions, such as NMI and HardFault, followed by peripheral interrupts.

The order matters: the hardware uses the interrupt number to select a position in the table. 

### Weak handlers and overrides

```
.weak      NMI_Handler
   .thumb_set NMI_Handler,Default_Handler
  
   .weak      HardFault_Handler
   .thumb_set HardFault_Handler,Default_Handler
```
The file declares interrupt handler names as weak aliases of Default_Handler.

An alias is a second name for the same handler. “Weak” means the linker will use this default mapping unless another object file provides a normal, strong function with the same name.

That lets application code implement only the interrupts it needs. For instance, if you define a strong SysTick_Handler() in a C file, it replaces the weak SysTick_Handler alias in the startup assembly.

## Linker File

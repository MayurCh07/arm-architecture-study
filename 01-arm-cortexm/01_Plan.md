Cortex-M Architecture

Setup Used:
1. STM32F401 Black Pill
2. ST-Link V2
3. VS Code

Study Topics: 
1. Cortex-M Overview 
2. Registers 
3. Program Counter (PC) 
4. Stack Pointer (SP) 
5. Link Register (LR) 
6. xPSR 
7. Vector Table 
8. Reset Handler 
9. Startup Code 
10. Interrupt Architecture 
11. NVIC 
12. Memory Map 
13. Flash 
14. SRAM 
15. Stack 
16. Heap 

Questions To Answer 
1. What occurs after power-up? 
2. How does MCU start execution? 
3. What is vector table? 
4. Why startup code is required? 
5. What is MSP? 
6. What is PSP? 
7. What happens during interrupt entry? 
8. What happens during interrupt exit? 
9. Why are interrupts faster than polling? 
10. How does RTOS use SysTick and PendSV? 

Blog:

ARM Cortex-M Execution Model for System Engineers 

Include: 
1. Boot sequence 
2. Interrupt flow 
3. Memory layout 
4. Real automotive implications 

Project: 

ARM Architecture Explorer 

Implement: 
1. SysTick timer 
2. Timer interrupt 
3. HardFault handler 
4. Memory inspection 

Deliverables: 
1. ARM_Architecture.md 
2. Boot_Process.png 
3. Interrupt_Flow.png
# Report of Computer Architecture

* Stage: PA1.1
* Date: 2019.3.27

* All contents in PA1.1 is finished.

## Category

[TOC]

## Quiz

1. What's in PC?

   As always, PC was implemented as int_32 register in IA-32, meaning that it can only store variables of 32-bits. For executable codes, length of particular instruction might exceed 32 bits, as well as varies from each other. This means computer cannot store them consistently. Also, storing reference to instructions makes it easier to calculate what's next to do and track the entire execution.

2. Architecture while running "HelloWorld" on NEMU

Actual Hardware |->| MacOS (Host) |->| VMware |->| GNU/Linux |->| NEMU |->| "Hello world"

3. Emulator? Virtual Machine?

For emulator, hardware will usually be implemented virtually and thus support software with different ISA, Android on Windows for example. 
For virtual machine, auxiliary hardware like network adapter are implemented virtually. But to boot efficiency, key components like CPU will be shared with actual hardware under protection. This means it can run particular software with sophisticated ISA.

4. Where programs start and end? <Combination of 2 Quiz>

First we need to setup the boardline: when a program gets its control to hardware? If we scout purly on source code files(.c), it's definitely the case that `main()` will be the entry point of application. However, it's called by supported fundation (OS), which means OS will perform security preparation like stack randomization before transfering control, check status (ret value of main) and clean up before it terminate (after return from `main()`). So application actually starts way before `main()` is called, and ends way after `main()` returns, thus defines the lifecycle of an app as **whether a program gets control to hardware**. Function `main()` only serves as an boundary between application and system behaviors. Explicitly, asm instruction `hlt` will guard forced termination of an application, an illustration why ret of `main()` is not the end of an app.

5. How far will cpu_exec() in NEMU execute?

`cpu_exec()` takes `uint32_t` as para0 indicating how much steps it should execute at most before suspending. Literal '-1' (bit pattern) indicates `0xFFFFFFFF`, or `UINT_MAX`. So in fact cpu_exec won't run infinitely, but cycles it runs is sufficient to most programs.

6. Block Sequence vs Hex

A matter of little-endian storage method in memory. To better demo the theory, check my implementation of `cmd_x()`. If it runs on big endian machines, byte order of each 4-byte value will be reversed. (Can I omit explaining that 'cause it's simple)

## Contents

### Config structure of register

Reg structure was simulated via the following structure.

```c
typedef struct {
union {

union {
uint32_t _32;
uint16_t _16;
uint8_t _8[2];
} gpr[8];

struct {
rtlreg_t eax, ecx, edx, ebx, esp, ebp, esi, edi;
};

};

/* Do NOT change the order of the GPRs' definitions. */

/* In NEMU, rtlreg_t is exactly uint32_t. This makes RTL instructions
* in PA2 able to directly access these registers.
*/
vaddr_t eip;

} CPU_state;
```

run `make run` under `ics2017/nemu`, we get test passed.

![executed nemu](./img1.jpg)

### Implement cmd_info

First Implement `reg_print()` in `reg.h`:

```c
static inline void reg_print() {
for(int index = R_EAX; index < R_EDI; index += 1) {
printf( 
"%4s\t  %2.2hx  %2.2hx  %2.2hx  %2.2hx\t  0x%8.8x\n",
regsl[index], (cpu.gpr[index]._32 >> 0) & 0xFF, (cpu.gpr[index]._32 >> 8) & 0xFF,
(cpu.gpr[index]._32 >> 16) & 0xFF, (cpu.gpr[index]._32 >> 24) & 0xFF, cpu.gpr[index]._32
);
}
printf(
"%4s\t  %2.2hx  %2.2hx  %2.2hx  %2.2hx\t  0x%8.8x\n",
"eip", (cpu.eip >> 0) & 0xFF, (cpu.eip >> 8) & 0xFF,
(cpu.eip >> 16) & 0xFF, (cpu.eip >> 24) & 0xFF, cpu.eip
);
return;
}
```

Then Implement `cmd_info` in `ui.c`, with cmd_table updated.

```c
static int cmd_info(char* args) {
char *arg = strtok(NULL, " ");
//bool stage_execed[2] = {false};

if(arg == NULL) {
/* Missing argument */
printf("Missing Argument\n");
return 0;
}
/*FIXME: segmentation fault */
switch(*arg) {
case 'r':
reg_print();
break;
case 'w':
/* TODO: Implement watchpoint */
break;
default:
printf("Invalid arg: \"%c\"\n", *arg);
return 0;
}
return 0;
}
```

execution:

![cmd_info](./img7.jpg)

### Implement cmd_si

Implement of `cmd_si()`:

```c
static int cmd_si(char* args) {
char *arg = strtok(NULL, " ");
int step = 1;

if(arg != NULL) {
/* Argument given, override default step interval */
step = atoi(arg);
}
cpu_exec(step);
return 0;
}
```

To distinguish execution from `si -1` or `c`, we update wrapper setting in `cpu-exec.c`:

```c
void cpu_exec(uint64_t n) {
if (nemu_state == NEMU_END) {
printf("Program execution has ended. To restart the program, exit NEMU and run again.\n");
return;
}
nemu_state = NEMU_RUNNING;

//bool print_flag = n < MAX_INSTR_TO_PRINT;
bool print_flag = n != -1;

for (; n > 0; n --) {
/* Execute one instruction, including instruction fetch,
* instruction decode, and the actual execution. */
exec_wrapper(print_flag);

#ifdef DEBUG
/* TODO: check watchpoints here. */

#endif

#ifdef HAS_IOE
extern void device_update();
device_update();
#endif

if (nemu_state != NEMU_RUNNING) { return; }
}

if (nemu_state == NEMU_RUNNING) { nemu_state = NEMU_STOP; }
}                                                                                                                                                                                                                                    
```
execution:

![cmd_si_0](./img2.jpg)
![cmd_si_1](./img5.jpg)
![cmd_si_2](./img6.jpg)

### Implement cmd_x

Same principle as `info -r` :

```c
static int cmd_x(char* args) {
char* arg0 = strtok(NULL, " ");
char* arg1 = strtok(NULL, " ");

if(!(arg0 && arg1)) {
printf("Missing para: len srtAddr\n");
return 0;
}

uint32_t step = atoi(arg0);
uint32_t startAddr = 0;
sscanf(arg1, "%x", &startAddr);
printf("srt: 0x%x, step: 0x%x\n", startAddr, step);

for(unsigned int i = 0; i < step; i += 4) {
uint32_t addr = startAddr + i;
uint32_t result = vaddr_read(addr, 4); // TODO: Implement auto type check
printf(
"0x%8.8x\t  %2.2hx  %2.2hx  %2.2hx  %2.2hx\t  0x%8.8x\n",
addr, (result >> 0) & 0xFF, (result >> 8) & 0xFF,
(result >> 16) & 0xFF, (result >> 24) & 0xFF, result
);
}
return 0;
}
```
We altered the rule so endAddr = startAddr + Rnd_down(step)

execution:

![cmd_x](./img3.jpg)

## Problems Encountered

1. Q: Anonymous `struct` / `union`.

A: When encounter anonymous `struct` / `union` without generating named object in C, compiler will regard struct as data alignment and generate an object base on that definiton. All elements in anonymous `struct` / `union` can be accessed directly outside the host structure.

2. Q: use of `strtok()`

A: Function `strtok()` devides string by div_char. It will replace original str with result and store rest of the string in a buffer. This buffer will be cleaned up if new target was specified. **Use with caution from reference chaos.**

## Summary

Enhanced my understanding to reg structure and solution on command arguments. Gained deeper recognition on emulator implementation.

## Git Log

`git log --oneline`

![git_log](./img8.jpg)

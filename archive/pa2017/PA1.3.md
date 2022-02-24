# Report of Computer Architecture

* Stage: PA1.3
* Date: 2019.4.17

* All contents in PA1.3 is finished.

## Category

[TOC]

## Quiz

1. Symbol conflict

Yes, you can name a variant `free`, it won't cause any issue... Unless you include `stdlib.h`. That'll override the func `free(void *)` by re-binding its symbol. 
But who could be that handycapped.

2. `static` in public scale

`static` in private scale (ie. in a func / struct) identifies a variant global and specified to parent. But `static` in struct-public scale will identifies a global variant / func as file-private.

3. A longer instruction coding of `INT 3`

Of course you can implement `INT 3` with longer bytes as long as you preserves origional data. But one shall be very careful dealing with instruction less than one of assumed `INT 3` against buffer overflow or instruction overlaps.

4. Misplaced `INT 3`

Well, if misplaced, `INT 3` instruction will be considered data other than instruction. This might cause runtime error. Ex:
Origion:
```
0x100000: b8 34 21 00 00  mov $0x1234, %eax
```
Break at `0x100001`:
```
0x100000: b8 cc 21 00 00  mov $0x12cc, %eax
```

GG.

5. Emulator vs Debugger

Emulator works as an virtual layout of simulated hardware, while debugger intercept with target process / thread. Overall, both methods of debugging gains control of "debugee" by different ways.

6. How to refer to manual

Well, table of content is crucial. So if we'd like to refer to new concepts, opting into the TOC is first priority. Like we want to look concept `selection` up, we just serach in the menu.

![img_man](./img1.jpg)

7. What's `CF`, `mov`, `ModR/M`

> 0   CF     Carry Flag -- Set on high-order bit carry or borrow; cleared otherwise.

> The ModR/M and SIB bytes follow the opcode byte(s) in many of the 80386 instructions. They contain the following information:
* The indexing type or register number to be used in the instruction
* The register to be used, or more information to select the instruction
* The base, index, and scale information
The ModR/M byte contains three fields of information:
* The mod field, which occupies the two most significant bits of the byte, combines with the r/m field to form 32 possible values: eight registers and 24 indexing modes
* The reg field, which occupies the next three bits following the mod field, specifies either a register number or three more bits of opcode information. The meaning of the reg field is determined by the first (opcode) byte of the instruction.
* The r/m field, which occupies the three least significant bits of the byte, can specify a register as the location of an operand, or can form part of the addressing-mode encoding in combination with the field as described above
The based indexed and scaled indexed forms of 32-bit addressing require the SIB byte. The presence of the SIB byte is indicated by certain encodings of the ModR/M byte. The SIB byte then includes the following fields:
* The ss field, which occupies the two most significant bits of the byte, specifies the scale factor
* The index field, which occupies the next three bits following the ss field and specifies the register number of the index register
* The base field, which occupies the three least significant bits of the byte, specifies the register number of the base register

> Opcode   Instruction       Clocks        Description
* 88  /r   MOV r/m8,r8       2/2           Move byte register to r/m byte
* 89  /r   MOV r/m16,r16     2/2           Move word register to r/m word
* 89  /r   MOV r/m32,r32     2/2           Move dword register to r/m dword
* 8A  /r   MOV r8,r/m8       2/4           Move r/m byte to byte register
* 8B  /r   MOV r16,r/m16     2/4           Move r/m word to word register
* 8B  /r   MOV r32,r/m32     2/4           Move r/m dword to dword register
* 8C  /r   MOV r/m16,Sreg    2/2           Move segment register to r/m word
* 8D  /r   MOV Sreg,r/m16    2/5,pm=18/19  Move r/m word to segment register
* A0       MOV AL,moffs8     4             Move byte at (seg:offset) to AL
* A1       MOV AX,moffs16    4             Move word at (seg:offset) to AX
* A1       MOV EAX,moffs32   4             Move dword at (seg:offset) to EAX
* A2       MOV moffs8,AL     2             Move AL to (seg:offset)
* A3       MOV moffs16,AX    2             Move AX to (seg:offset)
* A3       MOV moffs32,EAX   2             Move EAX to (seg:offset)
* B0 + rb  MOV reg8,imm8     2             Move immediate byte to register
* B8 + rw  MOV reg16,imm16   2             Move immediate word to register
* B8 + rd  MOV reg32,imm32   2             Move immediate dword to register
* C6       MOV r/m8,imm8     2/2           Move immediate byte to r/m byte
* C7       MOV r/m16,imm16   2/2           Move immediate word to r/m word
* C7       MOV r/m32,imm32   2/2           Move immediate dword to r/m dword

8. Workload summary

`wc -l*` for summarizing lines from each files.
To utilize the result, we use pipelined command`find .*|xargs wc -l | grep total`. As it's implemented into `Makefile`.

9. `-Wall` and `-Werror`

>-Wall
This enables all the warnings about constructions that some users
consider questionable, and that are easy to avoid (or modify to
prevent the warning), even in conjunction with macros.  This also
enables some language-specific warnings described in C++ Dialect
Options and Objective-C and Objective-C++ Dialect Options.

>-Werror
Make all warnings into hard errors.  Source code which triggers
warnings will be rejected.

These will enhance type security and avoid unintended misbehaviors.

10. Greater efficiency

Well, this is the calculation: Assuming analyzing info take 30 sec per in `GDB`, 10 sec per in simple debugger.

$$
{{30} \over {10} }= 3
$$

This means simple debugger saves time twice against `GDB`. That's a lot if it's scaled up.
if we need to compile 450 times for debugging, and takes 20 info per bug.

we got $450\space bug \times 20\space info/bug \times 20\space sec/info = 50\space hour$, nearly a day.

11. `GDB` : In "Content" section.

## Contents

### Try watchpoints in `GDB`

Watchpoint acts differently to breakpoint in `GDB`: First it's not static for it's connected to symbols in exection, rather than actual memory location as breakpoint does. So it can only be defined within its lifecycle.

We did such experiment.

![img_gdb_1](./img7.jpg)
![img_gdb_2](./img8.jpg)

> As we can see, global var `glo` is long-lived and can be watched statically, while local var `a` is only valid in its scope `main()`.

### Implementing Watchpoint / Breakpoint structure

Info structure of WP / BP

```c
typedef struct breakpoint_data {
vaddr_t tar_eip;
uint8_t reserved_instr; // replace 1 byte with 0xcc
} BreakpointData;

typedef struct watchpoint_data {
char* expr;
uint32_t old_val;
} WatchpointData;
```

We use dual direction chain table as carrier of WP / BP information.

```c
typedef struct watchpoint {
int index;
WatchpointData data;
bool is_active;
struct watchpoint *priv;
struct watchpoint *next;
} WP;
```
> `struct breakpoint` shares simular structure.


### Implementing WP / BP pool administration

Because both pool shares simular administration method, we only demo the WP one.

```c
static WP* new_wp() {
WP *new_node = (WP *) calloc(sizeof(WP), 1);
Assert(new_node != NULL, "Block allocation failed\n");

new_node -> data.expr = NULL;
new_node -> is_active = true;
new_node -> next = NULL;
new_node -> priv = tail;

Assert(tail != NULL, "Structure Failure: tail_ptr\n");
new_node -> index = new_node -> priv -> index + 1;

tail -> next = new_node;
tail = new_node;
return new_node;
}

static void free_wp(WP *wp) {
Assert(wp != NULL && wp != &wp_pool, "Reference failed: wp\n");
WP * priv = wp -> priv, *next = wp -> next;
if(priv != NULL) priv -> next = next;
if(next != NULL) next -> priv = priv;

if(wp -> data.expr != NULL) free(wp -> data.expr);
free(wp);
}
```

### Binding WP / BP func to debugger

In this part, we made the following instruction possible:

`info b` for BP list, `info w` WP list
`w`, `b` for WP / BP pool management, in which arg `a`, `t`, `d` serves adding / toggling / deleting nodes

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
list_watchpoint();
break;
case 'b':
list_breakpoint();
break;
default:
printf("Invalid arg: \"%c\"\n", *arg);
return 0;
}
return 0;
}
```

```c
static int cmd_w(char* args) {

char *arg = strtok(NULL, " ");
if (arg == NULL) {
printf("Missing Parameter\n");
return 0;
}

bool is_success = false;
int res = 0;
char *arg2 = NULL;
switch(*arg) {
case 'a':
arg2 = strtok(NULL, "");
if(arg2 != NULL) {
set_watchpoint(arg2);
}else{
printf("Missing Expression\n");
return 0;
}
break;
case 't':
arg2 = strtok(NULL, "");
if(arg2 != NULL) {
res = expr(arg2, &is_success);
if(is_success) toggle_watchpoint(res);
}else{
printf("Missing index\n");
return 0;
}
break;
case 'd':
arg2 = strtok(NULL, "");
if(arg2 != NULL) {
res = expr(arg2, &is_success);
if(is_success) delete_watchpoint(res);
}else{
printf("Missing index\n");
return 0;
}
break;
default:
printf("Invalid arg: \"%c\"\n", *arg);
return 0;
}
return 0;
}

static int cmd_b(char* args) {

char *arg = strtok(NULL, " ");
if (arg == NULL) {
printf("Missing Parameter\n");
return 0;
}

bool is_success = false;
int res = 0;
char *arg2 = NULL;
switch(*arg) {
case 'a':
arg2 = strtok(NULL, "");
if(arg2 != NULL) {
res = expr(arg2, &is_success);
if(is_success) set_breakpoint(res);
}else{
printf("Missing expression\n");
return 0;
}
break;
case 't':
arg2 = strtok(NULL, "");
if(arg2 != NULL) {
res = expr(arg2, &is_success);
if(is_success) toggle_breakpoint(res);
}else{
printf("Missing index\n");
return 0;
}
break;
case 'd':
arg2 = strtok(NULL, "");
if(arg2 != NULL) {
res = expr(arg2, &is_success);
if(is_success) delete_breakpoint(res);
}else{
printf("Missing index\n");
return 0;
}
break;
default:
printf("Invalid arg: \"%c\"\n", *arg);
return 0;
}
return 0;
}
```

### Implementing WP functionality

For `int set_watchpoint(char *e); bool delete_watchpoint(int NO); void list_watchpoint(void); WP* scan_watchpoint(void);`, those are basic chain iteration with auxiliary info processing.

```c

int set_watchpoint(char *e) {
WP* new_node = new_wp();
new_node -> data.expr = (char *) calloc(sizeof(char), strlen(e) + 1);
memcpy(new_node -> data.expr, e, strlen(e));
bool success = false;
new_node -> data.old_val = expr(e, &success);
if(success == false) {printf("Eval  Failed\n"); free_wp(new_node); return 0;}

new_node -> is_active = true;

return new_node -> index;
}

void delete_watchpoint(int index) {
bool flag = false;

WP* current = &wp_pool;
while(current -> next != NULL) {
current = current -> next;
if(current -> index == index) {
Assert(flag == false, "Structure Failure: index\n");
flag = true;
free_wp(current);
continue;
}
}
if(!flag) printf("Watchpoint[%d] not found\n", index);
return;
}

void list_watchpoint(void) {
WP* current = &wp_pool;
while(current -> next != NULL) {
current = current -> next;
printf(
"# %d:\t %s\nexpr = %s\nold_var = \t  0x%8.8x\t  %2.2hx  %2.2hx  %2.2hx  %2.2hx\n",
current -> index, (current -> is_active ? "<Active>" : ""), current -> data.expr,
current -> data.old_val, (current -> data.old_val >> 0) & 0xFF, (current -> data.old_val >> 8) & 0xFF,
(current -> data.old_val >> 16) & 0xFF, (current -> data.old_val >> 24) & 0xFF
);
}
}

void toggle_watchpoint(int index) {
bool flag = false;

WP* current = &wp_pool;
while(current -> next != NULL) {
current = current -> next;
if(current -> index == index) {
Assert(flag == false, "Structure Failure: index\n");
flag = true;
current -> is_active = !(current -> is_active);
continue;
}
}
if(!flag) printf("Watchpoint[%d] not found\n", index);
return;
}

WatchpointData* scan_watchpoint(void) {
WatchpointData* result = NULL;
WP* current = &wp_pool;
while(current -> next != NULL) {
current = current -> next;
if(!current->is_active) continue;
bool success = false;
uint32_t new_val = expr(current -> data.expr, &success);
if(success == false) {printf("Eval  Failed\n"); return NULL;}
if(new_val != current -> data.old_val) {
printf(
"# %d \33[1;31m<TRIG>\33[0m:\texpr = %s\nold_var = \t  0x%8.8x\t  %2.2hx  %2.2hx  %2.2hx  %2.2hx\nnew_var = \t  0x%8.8x\t  %2.2hx  %2.2hx  %2.2hx  %2.2hx\n",
current -> index, current -> data.expr,
current -> data.old_val, (current -> data.old_val >> 0) & 0xFF, (current -> data.old_val >> 8) & 0xFF,
(current -> data.old_val >> 16) & 0xFF, (current -> data.old_val >> 24) & 0xFF,
new_val, (new_val >> 0) & 0xFF, (new_val >> 8) & 0xFF,
(new_val >> 16) & 0xFF, (new_val >> 24) & 0xFF
);
current -> data.old_val = new_val;
result = &(current -> data);
}
}
return result;
}
```

For watchpoint, we eval `expr` of all active watchpoints and compare with each of their `old_val`, setting `neum_state` to `NEMU_STOP` when spotted differences.

```diff
@@ -41,7 +41,7 @@ void cpu_exec(uint64_t n) {

+    if(scan_watchpoint())nemu_state = NEMU_STOP; // simulated stop   

#endif
```

![img_WP](./img3.jpg)

### Implementing BP functionality

For basic BP info management, we just omit that for it's simular to WP, except everytime info was updated, `void refresh_breakpoint_status(void)` will be called to engage memory updating.

Now Implement `INT 3` instruction.

On refering to i386 manual, we know that `INT 3` is a single byte instruction with code `0xCC`.
> Opcode    Instruction    Clocks          Description
CC             INT 3              33                  Interrupt 3--trap to debugger

So it refers to no decoding function and one execution function, which means using `EX(ex)` macro for the reference table in `exec.c` indexing `0xCC` without escaping.

```diff
@@ -124,7 +124,7 @@ opcode_entry opcode_table [512] = {
/* 0xc0 */   IDEXW(gp2_Ib2E, gp2, 1), IDEX(gp2_Ib2E, gp2), EMPTY, EMPTY,
/* 0xc4 */   EMPTY, EMPTY, IDEXW(mov_I2E, mov, 1), IDEX(mov_I2E, mov),
/* 0xc8 */   EMPTY, EMPTY, EMPTY, EMPTY,
-  /* 0xcc */   EMPTY, EMPTY, EMPTY, EMPTY,
+  /* 0xcc */   EX(int), EMPTY, EMPTY, EMPTY,
/* 0xd0 */   IDEXW(gp2_1_E, gp2, 1), IDEX(gp2_1_E, gp2), IDEXW(gp2_cl2E, gp2, 1), IDEX(gp2_cl2E, gp2),
/* 0xd4 */   EMPTY, EMPTY, EX(nemu_trap), EMPTY,
/* 0xd8 */   EMPTY, EMPTY, EMPTY, EMPTY,

```

Then generate `exec_` function using macro `#define make_EHelper(name) void concat(exec_, name) (vaddr_t *eip)`, guiding `nemu_state` to `NEMU_BREAK` if trapped.

```c
make_EHelper(int) {

nemu_state = NEMU_BREAK;

print_asm("int %s", id_dest->str);

#ifdef DIFF_TEST
diff_test_skip_nemu();
#endif
}
```

> `NEMU_BREAK` differs from `NEMU_STOP` for additional steps to resume are required.

Then we engage following functions for instruction replacing.

```c
static void allocate_breakpoint (BP *bp) {
Assert(vaddr_read(bp -> data.tar_eip, 1) == 0xCC || vaddr_read(bp -> data.tar_eip, 1) == bp -> data.reserved_instr, "Value Error\n");
vaddr_write(bp -> data.tar_eip, 1, (bp -> is_active ? 0xCC : bp -> data.reserved_instr));
return;
}

void refresh_breakpoint_status(void) {
BP* current = &bp_pool;
while(current -> next != NULL) {
current = current -> next;
allocate_breakpoint(current);
}
return;
}

void resolve_breakpoint(vaddr_t eip) {
BP* current = &bp_pool;
bool is_found = false;
while(current -> next != NULL) {
current = current -> next;
if(current -> data.tar_eip == eip) {
is_found = true;
Assert(vaddr_read(eip, 1) == 0xCC, "Value Error\n");
// temporary resume to origion instr
vaddr_write(current -> data.tar_eip, 1, current -> data.reserved_instr);
printf(
"# %d \33[1;31m<TRIG>\33[0m:\teip = 0x%x\n",
current -> index, current -> data.tar_eip
);
break;
}
}
Assert(is_found, "Uncaptured BP\n");
return;
}

```

These functions replace certain bytes at specific location during guided execution.

Then interferes with normal execution.

```diff
@@ -41,7 +41,9 @@ void cpu_exec(uint64_t n) {
exec_wrapper(print_flag);

#ifdef DEBUG
-
+    
+      if(should_restore_debug)refresh_breakpoint_status(); // restore bp
+      
if(scan_watchpoint())nemu_state = NEMU_STOP; // simulated stop

#endif
@@ -51,6 +53,19 @@ void cpu_exec(uint64_t n) {
device_update();
#endif

+    switch (nemu_state) {
+      case NEMU_RUNNING:
+        break;
+      case NEMU_BREAK:
+        cpu.eip -= 1;
+        resolve_breakpoint(cpu.eip);
+        return;
+        break;
+      default:
+        return;
+        break;
+    }
+  }

if (nemu_state == NEMU_RUNNING) { nemu_state = NEMU_STOP; }
}
```

![img_WP_1](./img4.jpg)
![img_WP_2](./img6.jpg)

> Note that instruction pool got changed


## Problems Encountered

1. Q: Macro abuse

A: A great block on understanding the code is the extraction of macros. As an example, func `exec_real()` cannot be refered directly, for it's extracted from `make_EHelper(real)`. Understanding definition of macro and range it affects really procastinated my procedure.

2. 
## Summary

Practice on vector manipulation. Both ways to implement breakpoint. Navigation through sophisticated macro. Ability to generate code with simular structure via macro. Basic skill on manual reference.

## Git Log

`git log --oneline`

![git_log](./img2.jpg)

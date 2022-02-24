# Report of Computer Architecture

* Stage: PA1.2
* Date: 2019.4.13

* All contents in PA1.2 is finished.

## Category

[TOC]

## Quiz

1. Regex Expression of specific pattern:

32-bits Hex Number starting with `0x` : `0x0*[0-9A-Fa-f]{1,8}` or simply `0x[0-9A-Fa-f]+`
String mixed with Num & Alpha:  `[0-9A-Za-z]+` 
Symbol in C: `[_A-Za-z][_A-Za-z0-9]{0,254}` excluding keyword matching
Index(9) - Name(1~6) - PA1.1.pdf: `[0-9]{9} - [\u4e00-\u9fa5]{1,6} - PA1\.1\.pdf` 

2. Regex stored in C literal

Sometimes we need double escaping layer while implementing regex matching: first one is for regex rule matching such as rule `\+` to match `+` in target, second one is for storage of `\` in C literal, as we need `\\+` to store rule `\+`.

3. Avoid buffer overflow

Two strategies for both problems: One: Introduce Boundary check on static array. Two: Dynamic Array Allocation. View **Content** section for example.  

4. How can we eval expressions? What's BNF?

According to  definition, expression is a recursive structure consisting of atoms and sub-expressions. Each atom is bound to one data or function, and each sub-expression consists of at least 1 functional atom and serveral data atoms (**atomic** with 1 func atom). The calculation result, a data atom, works as the equative replacement of **atomic** sub-expression. The procedure ends when expression is replaced with single data atom.

To illustrate this, we can use BNF to demo a typical expression.

```BNF
<expr> ::= <expr> | <atomic_expr> | data_atom
<atom> ::= func_atom | data_atom
<atomic_expr> ::= func_atom [{data_atom}]
```

BNF defines grammar on what we can (but not what we should) substitude one expression with.

Here's an example: <a Spam use of BNF defines the grammar of function fac>
```BNF
  <fac>(<int>) ::= 1 | <int> * <fac>(<int>)
  <int> ::= 1|2|3|...
```

According BNF expression, we could append logical implementation to function fac.
```CommonLisp
  (defun fac (n)
    (if (= n 1)
      1
      (* n (fac (- n 1)))
```

And here is what happens when we call `(fac 5)`:
```CommonLisp
;?   (fac 5)
;<=> (* (fac 4) 5)
;<=> (* (* (fac 3) 4) 5)
;<=> (* (* (* (fac 2) 3) 4) 5)
;<=> (* (* (* (* (fac 1) 2) 3) 4) 5)
;<=> (* (* (* (* 1 2) 3) 4) 5)
;<=> (* (* (* 2 3) 4) 5)
;<=> (* (* 6 4) 5)
;<=> (* 24 5)
;->  120
```



## Contents

### Implementing Regex rules

First we add token types necessary to the following job.
```c
enum {
TK_DEF = 256, // start from 256 to avoid ascii instruction, CLEVER

TK_NUM_HEX, TK_NUM_DEC, TK_NUM_OCT, TK_NUM_BIN, // num

TK_REG_EAX, TK_REG_ECX, TK_REG_EDX, TK_REG_EBX, TK_REG_ESP, TK_REG_EBP, TK_REG_ESI, TK_REG_EDI, // long reg
TK_REG_AX, TK_REG_CX, TK_REG_DX, TK_REG_BX, TK_REG_SP, TK_REG_BP, TK_REG_SI, TK_REG_DI, // word reg
TK_REG_AH, TK_REG_CH, TK_REG_DH, TK_REG_BH, TK_REG_AL, TK_REG_CL, TK_REG_DL, TK_REG_BL, // byte reg
TK_REG_EIP, // eip reg

TK_OP_POS, TK_OP_NEG, TK_OP_PLUS, TK_OP_MINUS, TK_OP_MUL, TK_OP_DIVIDE, // Archi OP
TK_OP_EQUAL, TK_OP_NE, //TK_OP_LESS, TK_OP_GREATER, TK_OP_LE, TK_OP_GE, // Comp OP
TK_OP_LAND, TK_OP_LOR, TK_OP_LNOT, TK_OP_XOR, // Logic OP

TK_OP_DEREF, // De-ref OP

};
```

Then, according to pairing demands, we implement initial phrasing rules to the system. For some elements, token type is context related and thus under-determined.
```c

static struct rule { // Diction
char *regex;
int token_type;
} rules[] = {

// const char *regsl[] = {"eax", "ecx", "edx", "ebx", "esp", "ebp", "esi", "edi"};
{"0x0*[0-9A-Fa-f]{1,8}" , TK_NUM_HEX},   // hex num 32-bit restri.
{"0b0*[01]{1,32}"       , TK_NUM_BIN},   // bin num 32-bit restri.
{"0+[1-3]?[0-7]{0,10}"  , TK_NUM_OCT},   // oct num 32-bit restri.
{"([1-9][0-9]*)"        , TK_NUM_DEC},   // dec num non restricted

{"%eax"   , TK_REG_EAX},   // reg eax
{"%ebx"   , TK_REG_EBX},   // reg ebx
{"%ecx"   , TK_REG_ECX},   // reg ecx
{"%edx"   , TK_REG_EDX},   // reg edx
{"%esi"   , TK_REG_ESI},   // reg esi
{"%edi"   , TK_REG_EDI},   // reg edi
{"%ebp"   , TK_REG_EBP},   // reg ebp
{"%esp"   , TK_REG_ESP},   // reg esp

{"%eip"   , TK_REG_EIP},   // reg eip

{"%ax"   , TK_REG_AX},   // reg ax
{"%bx"   , TK_REG_BX},   // reg bx
{"%cx"   , TK_REG_CX},   // reg cx
{"%dx"   , TK_REG_DX},   // reg dx
{"%si"   , TK_REG_SI},   // reg si
{"%di"   , TK_REG_DI},   // reg di
{"%bp"   , TK_REG_BP},   // reg bp
{"%sp"   , TK_REG_SP},   // reg sp

{"%ah"   , TK_REG_AH},   // reg ah
{"%bh"   , TK_REG_BH},   // reg bh
{"%ch"   , TK_REG_CH},   // reg ch
{"%dh"   , TK_REG_DH},   // reg dh

{"%al"   , TK_REG_AL},   // reg al
{"%bl"   , TK_REG_BL},   // reg bl
{"%cl"   , TK_REG_CL},   // reg cl
{"%dl"   , TK_REG_DL},   // reg dl

{"\\(", '('},          // left quote
{"\\)", ')'},          // right quote

{"\\{", '{'},          // left {
{"\\}", '}'},          // right }
{"\\[", '['},          // left [
{"\\]", ']'},          // right ]

{"\\*", '*'},          // multiple / deref
{"\\/", TK_OP_DIVIDE}, // divide

{"\\+", '+'},          // plus / pos
{"-"  , '-'},          // minus / neg

{"==" , TK_OP_EQUAL},  // equal
{"!=" , TK_OP_NE   },  // not equal

{"!"      , TK_OP_LNOT},   // logic not
{"&&"     , TK_OP_LAND},   // logic and
{"\\|\\|" , TK_OP_LOR},    // logic or
{"\\^"    , TK_OP_XOR},    // logic xor

{" +" , TK_DEF}        // spaces

};
```

### Add `p` instruction to debugger

In ui.c, implement the following code:

```diff
@@ -44,7 +44,7 @@ static int cmd_info(char* args);

static int cmd_x(char* args);

+static int cmd_p(char* args);

//static int cmd_w(char* args);

@@ -61,7 +61,7 @@ static struct {
{ "si", "Step in execution for N steps, 1 as default", cmd_si},
{ "info", "Print register/watchpoint info", cmd_info},
{ "x", "Scan sections of memory", cmd_x},
+  { "p", "Evaluate expression", cmd_p}
/* TODO: Add more commands */

};
```

### Token stroage

We introduce dynamic array allocation to enhance capacity flexibility. 
`#include <stdlib.h>` is added to common.h

```c
Token *tokens = NULL; // read-only outside
uint32_t nr_token = 0;

static bool generate_token(char *e) {
int position = 0;
int tmp_nr_token = 0;
int i;
regmatch_t pmatch;

struct tempToken{
Token token;
struct tempToken *next;
};
struct tempToken *head = (struct tempToken *) calloc(sizeof(struct tempToken), 1);
struct tempToken *current = head;
head -> next = NULL;

while (e[position] != '\0') {
/* Try all rules one by one. */
for (i = 0; i < NR_REGEX; i ++) {
if (regexec(&re[i], e + position, 1, &pmatch, 0) == 0 && pmatch.rm_so == 0) {
/*
int regexec(const regex_t *preg, const char *string,
size_t nmatch, regmatch_t pmatch[], int eflags);
The regexec() function compares the null-terminated string specified by string with the compiled regular expression preg initialised by a previous call to regcomp().
*/
char *substr_start = e + position;
int substr_len = (int)pmatch.rm_eo;

Log("match rules[%d] = \"%s\" at position %d with len %d: %.*s",
i, rules[i].regex, position, substr_len, substr_len, substr_start);
position += substr_len;

struct tempToken *newToken = (struct tempToken *) calloc(sizeof(struct tempToken), 1);
newToken -> next = NULL;

bool should_discard = false;

switch (rules[i].token_type) {
case TK_DEF:
should_discard = true;
break;
default:
should_discard = false;
break;
}

if (should_discard) {
free(newToken);
}else{
newToken -> token.type = rules[i].token_type;
newToken -> token.token_length = substr_len;
newToken -> token.str = (char *) calloc(sizeof(char), substr_len + 1); // just for sure
memcpy(newToken -> token.str, substr_start, substr_len * sizeof(char));

tmp_nr_token += 1;
current -> next = newToken;
current = newToken;
}

break;
}
}

if (i == NR_REGEX) {
printf("no match at position %d\n%s\n%*.s^\n", position, e, position, "");
return false;
}
}

// proceed
nr_token = 0;
if(tokens != NULL) free(tokens);

current = head -> next;
tokens = (Token *) calloc(sizeof(Token), tmp_nr_token);
for(i = 0; i < tmp_nr_token; i += 1) {
Assert(current != NULL, "Structure Failure");
tokens[i].type = current -> token.type;
tokens[i].token_length = current -> token.token_length;
tokens[i].str = current -> token.str; // Assertion: Ctrl transfer

current = current -> next;
}

while(head) {
current = head;
head = head -> next;
free(current);
}

nr_token = tmp_nr_token;

return true;
}
```

### Context Re-phrasing

For some tokens whose type's not determined, we should phrase them according to context. For context complection, we procrastinate the procedure until tokens are stored.

For PA1.2, we need to phrase the following operand: `*`, `+`, `-`.

```c
static void context_chk(uint32_t p, uint32_t q) { // TODO: Reconstruction
for(int i = p; i <= q; i += 1) {
switch(tokens[i].type) {
case '*':
if(i != 0 && ((tokens[i - 1].type >= TK_NUM_HEX && tokens[i - 1].type <= TK_REG_EIP) || tokens[i - 1].type == '(' || tokens[i - 1].type == ')'))
tokens[i].type = TK_OP_MUL;
else tokens[i].type = TK_OP_DEREF;
break;
case '+':
if(i != 0 && ((tokens[i - 1].type >= TK_NUM_HEX && tokens[i - 1].type <= TK_REG_EIP) || tokens[i - 1].type == '(' || tokens[i - 1].type == ')'))
tokens[i].type = TK_OP_PLUS;
else tokens[i].type = TK_OP_POS;
break;
case '-':
if(i != 0 && ((tokens[i - 1].type >= TK_NUM_HEX && tokens[i - 1].type <= TK_REG_EIP) || tokens[i - 1].type == '(' || tokens[i - 1].type == ')'))
tokens[i].type = TK_OP_MINUS;
else tokens[i].type = TK_OP_NEG;
break;
default:
break;
}
}
}
```

### Parentheses Matching

Match parentheses by seperating evaluation layout.
```c
static bool check_parentheses(uint32_t p, uint32_t q) {
uint32_t *pair = (uint32_t *) calloc(sizeof(uint32_t), q - p + 2);
uint32_t layout = 1, last_paired_ind = 0;
bool flag = true;
for(uint32_t i = 1; i <= q - p + 1; i += 1) {
pair[i] = ~0x0;
switch (tokens[p + i - 1].type) {
case '(':
pair[i] = last_paired_ind;
last_paired_ind = i;
layout += 1;
break;
case ')':
if(last_paired_ind == 0) flag = false;
pair[i] = last_paired_ind;
int tmp = pair[last_paired_ind];
pair[last_paired_ind] = i;
last_paired_ind = tmp;
layout -= 1;
break;
default:
break;
}
//printf("Current: %c, pair[%d] = %d, l_p_l = %d\n", tokens[p + i - 1].type, i, pair[i], last_paired_ind);
eval_stat |= (layout > 0)? 0: ERR_PAREN_R;
}
eval_stat |= (layout == 1)? 0: ERR_PAREN_L;

flag = !eval_stat? flag: false;
if(pair[q - p + 1] == 1) {
flag = true && flag;
}else{
flag = false;
}
free(pair);
//printf("pa:%d, st: %d \n", flag, eval_stat);
return flag;
}
```

### Dominant Operand

For expression segregation, least prioritized operand of current sub-expression should be identified.

We first introduce operand priority level and associated "fake" anomynous functions in C:
```c
static struct Operation {
int token_type;
int arg_distro;
int level;
enum {OP_CAL_L = 0, OP_CAL_R} comb_order;
union {
int (*monadic)(int x);
int (*binary)(int x, int y);
//int (*ternary)(int x, int y, int z); // Not used
};
} operations[] = {
//{'('         , 0, 1, 1, NULL},
//{')'         , 0, 1, 1, NULL},
{TK_OP_NEG   , 0x1, 2, 1, .monadic = FUNC_NEG},
{TK_OP_POS   , 0x1, 2, 1, .monadic = FUNC_POS},
{TK_OP_DEREF , 0x1, 2, 1, .monadic = FUNC_DEREF},
{TK_OP_LNOT  , 0x1, 2, 1, .monadic = FUNC_LNOT},
{TK_OP_MUL   , 0x3, 3, 1, .binary = FUNC_MUL},
{TK_OP_DIVIDE, 0x3, 3, 1, .binary = FUNC_DIVIDE},
{TK_OP_PLUS  , 0x3, 4, 0, .binary = FUNC_PLUS},
{TK_OP_MINUS , 0x3, 4, 1, .binary = FUNC_MINUS},
{TK_OP_EQUAL , 0x3, 7, 1, .binary = FUNC_EQUAL},
{TK_OP_NE    , 0x3, 7, 1, .binary = FUNC_NE},
{TK_OP_XOR   , 0x3, 9, 0, .binary = FUNC_XOR},
{TK_OP_LAND  , 0x3, 11, 0, .binary = FUNC_LAND},
{TK_OP_LOR   , 0x3, 12, 0, .binary = FUNC_LOR},
};
```
> For language like C++ which supports anomynous functions, we can use Lambda expression Here. Unfortunately, this is C, in which every function should be symbolized.

Then we implement the searching function:
```c
static uint32_t check_domainantOp(uint32_t p, uint32_t q) {
struct Operation res = {0, 0, 0, 0, NULL};
uint32_t layout = 0;
uint32_t pos = ~0x0;
bool flag = false;
for(int i = p; i <= q; i += 1) {
switch(tokens[i].type) {
case '(':
layout += 1;
break;
case ')':
layout -= 1;
break;
default:
break;
}
if(layout != 0) continue;
const struct Operation* const cur = find_op(tokens[i]);
if(cur == NULL) continue;
else if(cur -> level > res.level || (cur -> level == res.level && cur -> comb_order == 0)) { // lower pri or right comb
pos = i;
res = *cur;
flag = true;
}
}
eval_stat |= flag ? 0 : ERR_STRUCT;
return flag ? pos : 0;
}
```
> Actually `()` serves as a monadic operation. But we just keep it simple.

Execution running `(nemu) p (1+   2)  * (3+4 *(5 +6))`:

![pa_domiOp](./img5.jpg)

### Evaluation

First we need to extract values from data atoms: There're two types: number literal and register reference.

Reg refer:
```c
static inline int ref_reg(uint32_t p) {
int ref_ind = tokens[p].type;
int res = 0;
uint32_t mask = 0xFFFFFFFF, shiftCount = 0;

if(ref_ind >= TK_REG_AL && ref_ind <= TK_REG_BL) { // XL
mask = 0x000000FF;
ref_ind -= TK_REG_AL - TK_REG_EAX;
shiftCount = 0;
}else if(ref_ind >= TK_REG_AH && ref_ind <= TK_REG_BH) { // XH
mask = 0x0000FF00;
ref_ind -= TK_REG_AH - TK_REG_EAX;
shiftCount = 8;
}else if(ref_ind >= TK_REG_AX && ref_ind <= TK_REG_BX) { // XX
mask = 0x0000FFFF;
ref_ind -= TK_REG_AX - TK_REG_EAX;
shiftCount = 0;
}

if(ref_ind >= TK_REG_EAX && ref_ind <= TK_REG_EBX) { // EXX
int ind = ref_ind - TK_REG_EAX;
res = (cpu.gpr[ind]._32 & mask) >> shiftCount;
}else if(ref_ind == TK_REG_EIP) {
res = cpu.eip;
}else{
Assert(0, "Type Error\n");
}
printf("%d\n", res);
return res;
}
```

Number:
```c
static inline int cus_atoi(uint32_t p) {
Assert(p >= 0 && p < nr_token, "Structure Failure\n");
int exp = 0;
char *c = NULL;
switch (tokens[p].type) {
case TK_NUM_BIN:
exp = 2; c = tokens[p].str + 2; break;
case TK_NUM_DEC:
exp = 10; c = tokens[p].str; break;
case TK_NUM_HEX:
exp = 16; c = tokens[p].str + 2; break;
case TK_NUM_OCT:
exp = 8; c = tokens[p].str; break;
default:
Assert(0,"Bad Expression: NAN\n");
}
int res = 0;
uint32_t len = (uint32_t) strlen(c);
for (uint32_t j = 0; j < len; j += 1)
{
res *= exp;
switch(c[j]) {
case 'F':
case 'f':
res += 1;
case 'E':
case 'e':
res += 1;
case 'D':
case 'd':
res += 1;
case 'C':
case 'c':
res += 1;
case 'B':
case 'b':
res += 1;
case 'A':
case 'a':
res += 1;
case '9':
res += 1;
case '8':
res += 1;
case '7':
res += 1;
case '6':
res += 1;
case '5':
res += 1;
case '4':
res += 1;
case '3':
res += 1;
case '2':
res += 1;
case '1':
res += 1;
case '0':
break;
default:
Assert(0,"Invalid Symbol\n");
}
}
printf("cov:%d\n", res);
return res;
}
```

EXEC: `(nemu) p  0x100 - (073 + 0b1011) * *(%ecx) / ----144` with step output disabled.

> Includes number convertion `cus_atoi`, register reference `ref_reg`, context re-phrasing `context_chk`

![pa_all](./img4.jpg) 


Main procedure <we solves `()` as an exception>:
```c
static int eval(uint32_t p, uint32_t q) {
if (eval_stat != 0) return 0;

printf("%d, %d \n", p, q);

if (p > q) {
eval_stat |= ERR_STRUCT;
printf("Bad Expression\n");
return 0;
}
else if (p == q) {
if(tokens[p].type >= TK_NUM_HEX && tokens[p].type <= TK_NUM_BIN) {
return cus_atoi(p);
}else if(tokens[p].type >= TK_REG_EAX && tokens[p].type <= TK_REG_EIP){
return ref_reg(p);
}else{
printf("%d, at %d ", tokens[p].type, p);
Assert(0,"Bad Expression: Type Invalid\n");
}
}
else if (check_parentheses(p, q) == true) {
return eval(p + 1, q - 1);
}
else {
if(eval_stat) {
printf("Bad Expression\n");
return 0;
}

uint32_t op_ind = check_domainantOp(p, q);
struct Operation *op = find_op(tokens[op_ind]);
if(op == NULL) {
eval_stat |= ERR_STRUCT;
printf("Bad Expression\n");
return 0;
}

printf("Domin OP: %d, pos: %d\n", op->token_type, op_ind);

int result = 0;

switch(op->arg_distro) {
case 0x1: // left monadic
Assert(op_ind + 1 <= q, "Bad Expression: DT_MA_L\n");
result = op->monadic(eval(op_ind + 1, q));
break;
case 0x2: // right monadic
Assert(p <= op_ind - 1, "Bad Expression: DT_MA_R\n");
result = op->monadic(eval(p, op_ind - 1));
break;
case 0x3:
Assert(op_ind + 1 <= q, "Bad Expression: DT_MA_L\n");
Assert(p <= op_ind - 1, "Bad Expression: DT_MA_R\n");
result = op->binary(eval(p, op_ind - 1), eval(op_ind + 1, q));
break;
default:
Assert(0, "Structure Failure: OP\n");
break;
}

if(eval_stat) {
printf("Bad Expression\n");
return 0;
}

return result;
}
return 0;
}
```

Test exec:

![pa_eval_test_req](./img3.jpg) 

ERROR TEST EXEC: `(nemu) p 0x100 (073 + 0b1011) * *(%ecx) / ----144` with step output disabled.

> Delebrate bad expr at character 8.

![pa_all](./img6.jpg) 

### Implement x eval

at `cmd_x()`:
```diff
- sscanf(arg1, "%x", &startAddr);
+ bool success = false;
+ startAddr = expr(arg1, &success);
+ if(success == false) {printf("Eval  Failed\n");return 0;}
```

We gets:

![pa_x](./img1.jpg) 

## Problems Encountered

1. Q: regex pattern issue

A: For fallback support, escape words `\d` might not be recognized as matching number. To ensure stability, we ues standard ASCII  format of regex to  proceed.

2. Q: enum start from 256

A: That's for ASCII matching flexibility that allows direct match to  ASCII characters without overlapping.

3. Q: use of  macro

A: For const arrays like read-only rules, we can count them with macro. That's because relevant operand  `sizeof()` are fixed and computable at  preprocessing / compiling stage.

4. Q:  why use `%` instead of `$` at the front of reg ref?

A: This is the AT&T format in ASM, as `$` refers to an immediate. Of course, It comes true when `$` actually requires `\\$` as matching rule as `$` matches the end of line. I bet that's your trick.

## Summary

Another great practice on optimizable coding. New knowledge on **BNF** and **Regex**. Some out-smart use of macro. Basic word phrasing procedure.

## Git Log

`git log --oneline`

![git_log](./img2.jpg)

# nasm 知识点

## 指令

需要先学习一下 `nasm` 的一些指令

NASM supports two special tokens in expressions, allowing calculations to involve the current assembly position: the `$` and `$$` tokens.

$ evaluates to the **assembly position at the beginning of the line** containing the expression; 

so you can code an infinite loop using JMP $. $$ evaluates to the beginning of the current section; so you can tell how far into the section you are by using (`$-$$`).


- `db` : A pseudo-instruction that declares bytes that will be in memory when the program runs
- `$$` ：is the address of the beginning of the current section.
- `$` : evaluates to the **assembly position at the beginning of the line** containing the expression

下面的代码就是将 `Code Segment` 赋值给 `Data Segment` 和 `Extra Segment`

```asm
	org	07c00h			; 告诉编译器程序加载到7c00处
	mov	ax, cs			; ax = cs
	mov	ds, ax			; ds = ax
	mov	es, ax			; es = ax
	call	DispStr		; 调用显示字符串例程
	jmp	$				; 无限循环
```

## 方括号 `[]` 的使用

在 nasm 中，任何不被 `[]` 括起来的标签或者变量名都被认为是地址：

```asm
mov ax, BootMessage		; 将 "Hello, OS world!" 这个字符串的首地址传给寄存器 ax。

BootMessage: db "Hello, OS World!"
```

## 关于 `$` 和 `$$`

$ 表示 **当前行被汇编之后的地址**

```bash
# 将我们汇编得到的可执行文件反汇编
ndisasm -o 0x7c00 boot.bin >> disboot.asm
```

可以看到 `jmp $` 这一行的指令被汇编器生成了 `jmp short 0x7c0f`，而这个地址就是这一条指令的地址。

```asm
00007C0F  EBFE              jmp short 0x7c0f
```

$$ 表示一个 `section` (代码段) 的开始处被汇编后的地址，在这里我们的程序只有一个段，所以 $$ 实际地址就是表示程序被编译后的开始地址。

```asm
jmp $$
```

得到的反汇编文件

```asm
00007C09  EBF5              jmp short 0x7c00
```

### times 510 - ($ - $$)	db	0

`times 510 - ($ - $$)	db	0` 表示将 0 这个字节重复一定次数，就是简单的将整个程序的实际长度填充到 510 字节为止， **这样加上结束标志 0xAA55 占用的两个字节正好是 512 个字节**

## macro

[4.3 Multi-Line Macros: %macro](https://www.nasm.us/doc/nasmdoc4.html#section-4.3)

Multi-line macros are much more like the type of macro seen in MASM and TASM: a multi-line macro definition in NASM looks something like this.

```asm
%macro  prologue 1 

        push    ebp 
        mov     ebp,esp 
        sub     esp,%1 

%endmacro
```

This defines a C-like function prologue as a macro: so you would invoke the macro with a call such as:

```asm
myfunc:   prologue 12
```

which would expand to the three lines of code

```asm
myfunc: push    ebp 
        mov     ebp,esp 
		sub     esp,12
```

## DB and Friends: Declaring Initialized Data

```asm
      db    0x55                ; just the byte 0x55 
      db    0x55,0x56,0x57      ; three bytes in succession 
      db    'a',0x55            ; character constants are OK 
      db    'hello',13,10,'$'   ; so are string constants 
      dw    0x1234              ; 0x34 0x12 
      dw    'a'                 ; 0x61 0x00 (it's just a number) 
      dw    'ab'                ; 0x61 0x62 (character constant) 
      dw    'abc'               ; 0x61 0x62 0x63 0x00 (string) 
      dd    0x12345678          ; 0x78 0x56 0x34 0x12 
      dd    1.234567e20         ; floating-point constant 
      dq    0x123456789abcdef0  ; eight byte constant 
      dq    1.234567e20         ; double-precision float 
      dt    1.234567e20         ; extended-precision float
```

## 3.2.4 EQU: Defining Constants

EQU defines a symbol to a given constant value: when EQU is used, the source line must contain a label. The action of EQU is to define the given label name to the value of its (only) operand. This definition is absolute, and cannot change later. So, for example,

```asm
message         db      'hello, world' 
msglen          equ     $-message
```

defines `msglen` to be the `constant` 12. msglen **may not then be redefined later**. 

This is not a preprocessor definition either: the value of msglen is evaluated once, using the value of $ (see section 3.5 for an explanation of $) at the point of definition, rather than being evaluated wherever it is referenced and using the value of $ at the point of reference.

---
layout: jrasm-widenavi
title: jrasm User's Manual
tracking_id: UA-38761061-3
---
# jrasm User's Manual


<h2>Command Line<a name="command-line"></a></h2>

The execution format of `jrasm` is:

```
jrasm [options] src
```

Available options are:

|Long Format         |Short format|Function                                                           |
|--------------------|------------|-------------------------------------------------------------------|
|`--output=file`     |`-o file`   |Specifies the filename to output.                                  |
|`--print-disasm-l`  |`-d`        |Prints disassembler dump of the product in lower case.             |
|`--print-disasm-u`  |`-D`        |Prints disassembler dump of the product in upper case.             |
|`--print-hexdump-l` |`-x`        |Prints hexadecimal dump of the product in lower case.              |
|`--print-hexdump-u` |`-X`        |Prints hexadecimal dump of the product in upper case.              |
|`--print-list-l`    |`-l`        |Prints list of labels in lower case.                               |
|`--print-list-u`    |`-L`        |Prints list of labels in upper case.                               |
|`--print-resource-l`|`-r`        |Prints resource usage in lower case.                               |
|`--print-resource-u`|`-R`        |Prints resource usage in upper case.                               |
|`--verbose`         |`-v`        |Reports various things.                                            |
|`--help`            |`-h`        |Prints help message.                                               |


<h2>Comment<a name="comment"></a></h2>

When a semicolon `;` or a pair of slash characters `//` appears, the following text until the end of the line
is parsed as a line comment.

You can also use a block comment by surrounding texts with `/*` and `*/`, which can contain multiple lines.

```
        ; line comment

        // another line comment
        
        ldaa    32    ; line comment

        /* block comment
           that
           contains
           multiple lines. */

        .db     0x01,0x02,/* a comment inserted in a code */0x03,0x04
```


<h2>Literal<a name="literal"></a></h2>

### String

A string literal consists of a series of characters surrounded by a pair of double quotations (e.g. `"Hello World"`).

A string literal can contain escape characters listed below:

- `\0`
- `\r`
- `\n`


### Number

The format of number literal is as follows:

- Decimal number ... Begins with `1` to `9` and cosists of digit characters (e.g. `123`).
- Hexadecimal number ... Begins with `0x` and consists of digit and `A` to `F` characters (e.g. `0x3a22`).
- Octal number ... Begins with `0` and consts of `0` to `7` characters (e.g. `0327`).


### Symbol

A sybmol literal consists of a series of characters and is used for following purposes:

- Label
- Instruction's operation code
- Directive's name
- Register name

A symbol is case insensitive.
This means that you can describe an instruction `LDAA` with a symbol `LDAA`, `ldaa`, `Ldaa`, `LdAA` and so on.
Also, when you define a label named `Label1`, it can be referred to as `label1`, `LABEL1`, `LaBel1`
and anything like that.

There are some pre-defined symbols listed below:

|Symbol  |Assigned Value|Usage                                                     |
|--------|--------------|----------------------------------------------------------|
|`@BYTE` |`1`           |Specifies the size of one-byte data.                      |
|`@WORD` |`2`           |Specifies the size of one-word data.                      |


### Bit-Pattern

A string preceded by "`b`" or "`B`" is a bit-pattern literal that consists of a series of ascii characters
that correspond to binary data `0` and `1`.
The space, period '`.`', comma '`,`', under score '`_`' and hyphen '`-`' are recognized as `0` and
others are `1`.
The bit-pattern literal generates a sequence of byte-sized data like follows:


```
        .db     b"#.#.#.#."                                 ; 0xaa
        .db     b"#.#......##...#."                         ; 0xa0, 0x62
        .db     b"#.#......##...#.#.#.#.#.########..#....." ; 0xa0, 0x62, 0xaa, 0xff, 0x20
```


### MML

A string preceded by "`m`" or "`M`" is an MML literal that contain a sequence of MML data.
It will be parsed and converted to JR-200 music data.

- `A`-`G` ... A note. Appending a character `#` or `+` makes a sharped note and `-` makes a flatted one.

- `R` ... A rest.

- `O` ... Specifies octave with a number from `1` to `5`. 

- `>` ... Raises octave.

- `<` ... Lowers octave.

- `;` ... Specifies an end mark of the MML sequence.

- `L` ... Specifies the default length.

A note and rest can be followed by numbers `1`, `2`, `4`, `8` and `16` that mean the length of the duration
where `1` is a whole, `2` a half, `4` a quarter and so on.
A number followed by a period specifies a dotted note.
If no number specified, a default length specified by `L` command is applied.

MML sequences are joined together until an end mark appears even if they are described in separate lines.

Example:
```
        .db     m"CDEFGAB >CDEFGAB"
        .db     m"C4"   ; a quarter note of C
        .db     m"C8"   ; an eighth note of C
        .db     m"R4"   ; a quarter rest
        .db     m"D+4"  ; a quarter note of D sharp
        .db     m"D-4"  ; a quarter note of D flat
        .db     m"CDE;" ; all the MML sequences are joined until a semicolon
```


<h2>Operator<a name="operator"></a></h2>

You can use the following operators in operands:

|Operator|Function                                                           |
|--------|-------------------------------------------------------------------|
|`+`     |Addiction.                                                         |
|`-`     |Subraction.                                                        |
|`*`     |Multiplication.                                                    |
|`/`     |Division.                                                          |
|`&`     |Bitwise AND.                                                       |
|`|`     |Bitwise OR.                                                        |
|`^`     |Bitwise XOR.                                                       |
|`<+>`   |Adds value in a bracket or brace. e.g. `[0x1320] <+> 8` makes `[0x1328]`, `[x+0x23] <+> 5` makes `[x+0x28]`.|
|`<<`    |Bit shift to left.                                                 |
|`>>`    |Bit shift to right.                                                |


<h2>Directive<a name="directive"></a></h2>

The jrasm assembler supports following directives:


### .CSEG, .DSEG and .WSEG

The directives `.CSEG` and `.DSEG` declare the beginning of code and data segment respectively.
They don't put any restriction on what items are place in: you can write data sequence using directive `.DB`
in the code segment and can put instructions in the data segment as well.

Since they hold different address counters each other,
you can put some data sequence in the middle of program code like follows:

```
        .CSEG
        LDX     Hello
        .DSEG
Hello:  .DB     "Hello"
        .CSEG
        LDAA    [X]
        ; ... any jobs ...
```

The directive `.WSEG` specifies the start of work segment that is used
to allocate uninitialized memory are. It can not contain any instruction codes and directives that generate data
including `.DB` and `.DW`. It is basically supposed that you declare labels and `.DS` directives in it.

The code segment is selected before any of `.CSEG`, `.DSEG` or `.WSEG` directive appears.

You have to specify at least one `.ORG` directive in the first code segment.
The address of the data and work segment starts right after the code segment in ths order
unless their start adddress is explicitly specified by `.ORG` directive.


### .DB, .DW and .DS

These directive are used to store binary data. The directive `.DB` contains 8bit data while `.DW` does 16bit.

Directive `.DB` accepts string literal as well.

Example:
```
        .DB     0x00, 0x01, "ABC"
        .DW     0x1234
```

When a string literal appears in the directive, a null terminate character is NOT appended.
So, for examle, `"ABC"` in the direcive is equivalent to a sequence of `0x41`, `0x42`, `0x43`

Directive `.DS` reserves memory the size specified by the given parameter in bytes.

```
        .DS     32           ; 32bytes
        .DS     @byte * 32   ; also 32bytes, but more readable
        .DS     @word        ; 2bytes
        .DS     @word * 32   ; 64bytes
```


### .FILENAME.JR

Specifies a filename up to 16 charactesrs that is to be stored in CJR file.
This name is displayed when you run `LOAD` or `MLOAD` command in JR BASIC environment.
If this directive is omitted, the name of the assembler source file will be stored.

Example:
```
        .FILENAME.JR "Hello"
```


### .INCLUDE

Directive `.INCLUDE` takes in the content of another source file whose name is specified as its parameter.

```
        .INCLUDE "something.inc"
        .INCLUDE "src/another.inc"
```

If the specified file is not found in the current directory, it is searched in the following paths:

- A directory `inc` right under where `jrasm.exe` exists.
- Directories specified in the environment variable `JRASMPATH`, which are joined together with a semicolon character.

If the same file is already included, the second attempt to include it will be ignored. If you want to include the same file more than once, append an exclamation mark at the beginning of the file name.

```
        .INCLUDE "something.inc"
        .INCLUDE "!something.inc"
        .INCLUDE "!something.inc"
```

### .MACRO

Directive `.MACRO` defines a macro, a sequence of instructions and directives that is bound to a specific symbol.
You can expand the sequence anywhere in the code by specifying the symbol.

A macro consists of a label that represents the macro name
and a code sequence surrounded by `.MACRO` and `.END` directives.

```
mul4_a: .MACRO  ; A macro to multiply A-accumulator's value by four
        ASLA
        ASLA
        .END

        mul4_a  ; Expands the macro
```

When directive `.MACRO` is called with some arguments, the created macro can take values and expressions as its arguments. When the macro is expanded, a symbol that matches argument name will be replaced with the value passed to that argument. You can pass not only immediate value to an argument but other addressing expression like index and external memory.
 There's no limit on the number of arguments.

```
mul4:   .MACRO target ; A macro to multiply a value in memory by four
        LDAA    target
        ASLA
        ASLA
        STAA    target
        .END

        mul4    {0x09}    ; DIRECT addressing
        mul4    [X+0x03]  ; INDEX addressing
        mul4    [foo]     ; EXTERNAL addressing
```

Since the code in a macro is implicitly surrounded by `.SCOPE`, any labels that are defined within it are hidden from outside.

A macro can set a default value for each parameter by specifying the parameter followed by `=` and the value.
If the number of parameters passed to the macro is less than required
or parameters are declared blank by being specified by a series of commas, the default values are used instead.
```
macro1  .MACRO arg1=0x11, arg2=0x22, arg3=0x33, arg4=0x44
        .DB     arg1, arg2, arg3, arg4
        .END

        macro1  0xaa, 0xbb, 0xcc, 0xdd   ; 0xaa, 0xbb, 0xcc, 0xdd
        macro1  0xaa                     ; 0xaa, 0x22, 0x33, 0x44
        macro1  0xaa, 0xbb               ; 0xaa, 0xbb, 0x33, 0x44
        macro1  0xaa, , 0xcc             ; 0xaa, 0x22, 0xcc, 0x44
        macro1  , , 0xcc, 0xdd           ; 0x11, 0x22, 0xcc, 0xdd

```

### .ORG

It specifies the current address where program or data is to be stored.
You must specify at least one `.ORG` directive before any source code that generates binary data appears.

Example:

```
        .ORG    0x2000
```

You can specify more than one `.ORG` directive in a program.


### .PCGPAGE and .PCG

Directives `.PCGPAGE` and `.PCG` are used to create pattern data for JR-200's character generator.

A PCG is a pattern data and a PCG page is a set of PCGs.
You can create multiple PCG pages, whic may be useful to flip data in some operations.

Below is a sample:

```
        .PCGPAGE page1,USER:32

        .PCG    circle1x1,1,1
        .DB     b"..####.."
        .DB     b".#....#."
        .DB     b"#......#"
        .DB     b"#......#"
        .DB     b"#......#"
        .DB     b"#......#"
        .DB     b".#....#."
        .DB     b"..####.."
        .END

        .PCG    circle2x2,2,2
        .DB     b".....######....."
        .DB     b"...##......##..."
        .DB     b"..#..........#.."
        .DB     b".#............#."
        .DB     b".#............#."
        .DB     b"#..............#"
        .DB     b"#..............#"
        .DB     b"#..............#"
        .DB     b"#..............#"
        .DB     b"#..............#"
        .DB     b"#..............#"
        .DB     b".#............#."
        .DB     b".#............#."
        .DB     b"..#..........#.."
        .DB     b"...##......##..."
        .DB     b".....######....."

        .END

        .END
```

The format of `.PCGPAGE` is as follows:

```
.PCGPAGE symbol,range,...
```

The `symbol` parametet specifies a name associated with the PCG page. This name is used to create a macro.

The `range` parameter has a format of `[USER|CRAM]:start` or `[USER|CRAM]:star:end'
where the symbol `USER` and `CRAM` represents the user-defined character and the character RAM respectively.

The `start` and `end` parameter specify the starting and ending value of character codes that are to be assigned to each PCG. If `end` is omitted, the largest possible value is specified: `95` for the user-defined and `255` for the character RAM.

The example above, which specifies `32` for the parameter, assigns code `32` for `circle1x1`, and `33`, `34`, `35` and `36` for `circle2x2`.

You can specify more than one ranges.

The assignment of character code follows the rules below:

- A pattern filled with `zero` will be assigned with code `0x00` instead of a new one.

- A pattern that matches one that already exists will be assigned with a code of the existing.

The for mat of `.PCG` is as follows:

```
.PCG symbol,width,height
```

The `symbol` parameter specifies a name associated with the PCG. This name is used to create macros.

The `width` and `height` parameters specify the PCG's size in 8x8 characters.

There must be only `.DB` directives in `.PCG`.

You can describe pattern data with bit-pattern literal as well as numeric literals.
The code below is a sample that uses numeric literals to describe the same information as above:

```
        .PCG    circle1x1,1,1
        .DB     0x3c,0x42,,0x81,0x81,0x81,0x81,0x42,0x3c
        .END
```

Directive `.PCGPAGE` creates the following macro where `symbol` is replaced with a symbol specified in the direcive's parameter:

- `PCGPAGE.symbol.STORE` ... Executing it copies PCG data to appropriate address of user-defined or CRAM area.

Directive `.PCG` created the following macro where `symbol` is replaced with a symbol specified in the directive's paramter:

- `PCG.symbol.PUT offset=0` ... Writes character codes in memory indicated by X register.

- `PCG.symbol.PUTATTR offset=0` ... Writes attributes that are specified in `.PCG` directive in memory indicated by X register. For user-defined characters, `b6` bit is set to one.

- `PCG.symbol.PUTATTRFG offset=0` ... Writes foreground attributes that are specified in `.PCG` directive in memory indicated by X register. The background attribute remains intact. For user-defined characters, `b6` bit is set to one.

- `PCG.symbol.SETATTR fg=7,bg=0,offset=0` ... Writes the foreground color `fg` and background color `bg` in memory indicated by X register. Attributes that are specified in `.PCG` directive are just ignored. For user-defined characters, `b6` bit is set to one.

- `PCG.symbol.ERASE offset=0` ... Writes zeros in memory indicated by X register.

- `PCG.symbol.ERASEATTR fg=7,bg=0,offset=0` ... Writes the foreground color `fg` and background color `bg` in memory indicated by X register. Even for user-defined characters, `b6` bit is not set to one.

- `PCG.symbol.FILL data,offset=0` ... Writes specified value `data` in memory indicated by X register.

The parameter `offset` indicates an address offset for each writing.

The macros above don't render characters with code `zero`. If you want to render all the characters including `zero`, use the following macros that are named after corresponding macros but have suffix `X`:

- `PCG.symbol.PUTX offset=0`
- `PCG.symbol.PUTATTRX offset=0`
- `PCG.symbol.PUTATTRFGX offset=0`
- `PCG.symbol.SETATTRX fg=7,bg=0,offset=0`
- `PCG.symbol.ERASEX offset=0`
- `PCG.symbol.ERASEATTRX fg=7,bg=0,offset=0`
- `PCG.symbol.FILLX data,offset=0`


### .SAVE and .RESTORE

Directive `.SAVE` will save and restore value of accumulators `A` and `B`, and index register `X`.
Specify the register names as operands the directive like follows:

```
        LDX     0x1234
        .SAVE   X
        ;
        ; any process
        ;
        .END
        ; The value of X is restored after exiting .SAVE, to 0x1234 in this case.

        LDX     0x1234
        LDAA    0x56
        LDAB    0x78
        .SAVE   X,A,B
        ;
        ; any process
        ;
        .END
        ; The values of X, A and B are restored after exiting .SAVE.
```

Within the `.SAVE` region, you can restore the values at any point using `.RESTORE` directive that
takes accumulator or register name as its operands.

```
        .SAVE   A,B,X
        ;
        ; any process
        ;
        .RESTORE X   ; X is restored here
        ;
        ; any process
        ;
        .RESTORE A   ; A is restored here
        ;
        ; any process
        ;
        .RESTORE A,X ; A and X are restored here
        ;
        ; any process
        ;
        .END
        ; A, B and X are restored here
```


### .SCOPE

You can localize labels by surrounding codes with `.SCOPE` and `.END` directives.
Any labels that appear between these directives are hidden from outside.
It's possible to specify the same symbol for the global and the local labels,
which are dealt with as different ones.

```
label1: .EQU    0x1111
label2: .EQU    0x2222

        .SCOPE
label1: .EQU    0x1234   ; not visible from outside
        .DW     label1   ; 0x1234
        .DW     label2   ; 0x2222
        .END

        .SCOPE
label1: .EQU    0x5678   ; not visible from outside
        .DW     label1   ; 0x5678
        .DW     label2   ; 0x2222
        .END

        .DW     label    ; 0x1111
        .DW     label2   ; 0x2222
```

Even in the localized region, labels declared with double-colon `::` will be defined as global ones.

```
         .SCOPE
label1:  .EQU    0x1234   ; not visible from outside
label2:: .EQU    0x5678   ; visible from outside
         .DW     label1   ; 0x1234
         .DW     label2   ; 0x5678
         .END

         .DW     label2   ; 0x5678
```


### .STRUCT

Directive `.STRUCT` provides a method to declare data structure.
The directive must be preceded by a label assigned to it as the structure name and consist of labels and `.DS` directives.

It creates symbols that are composed with the structure name and labels inside and assigns them with offset values from the beginning.

Below is an example:

```
struct1:
        .STRUCT
posx:   .DS     @byte
posy:   .DS     @byte
score:  .DS     @word
attr:   .DS     @byte
        .END
```

In this case, symbols `struct1.posx`, `struct1.posy`, `struct1.score` and `struct1.attr` are created and assigned to `0`, `1`, `2` and `4` respectively.

It also creates a symbol that joins a character `@` and the structure name and assigns the structure's size to it. In the example above, a symbol `@struct1` assigned with `5` is created.


<h2>Instruction<a name="instruction"></a></h2>

Here is a list of the assembler's syntax for M6800 instructions.

|Syntax              |Another Syntax      |Operation                                                 |
|--------------------|--------------------|----------------------------------------------------------|
|`ABA`               |                    |`A<-A+B`                                                  |
|`ADCA data8`        |`ADC A,data8`       |`A<-A+data8+C`                                            |
|`ADCA {addr8}`      |`ADC A,{addr8}`     |`A<-A+{addr8}+C`                                          |
|`ADCA [X+data8]`    |`ADC A,[X+data8]`   |`A<-A+[X+data8]+C`                                        |
|`ADCA [addr16]`     |`ADC A,[addr16]`    |`A<-A+[addr16]+C`                                         |
|`ADCB data8`        |`ADC B,data8`       |`B<-B+data8+C`                                            |
|`ADCB {addr8}`      |`ADC B,{addr8}`     |`B<-B+{addr8}+C`                                          |
|`ADCB [X+data8]`    |`ADC B,[X+data8]`   |`B<-B+[X+data8]+C`                                        |
|`ADCB [addr16]`     |`ADC B,[addr16]`    |`B<-B+[addr16]+C`                                         |
|`ADDA data8`        |`ADD A,data8`       |`A<-A+data8`                                              |
|`ADDA {addr8}`      |`ADD A,{addr8}`     |`A<-A+{addr8}`                                            |
|`ADDA [X+data8]`    |`ADD A,[X+data8]`   |`A<-A+[X+data8]`                                          |
|`ADDA [addr16]`     |`ADD A,[addr16]`    |`A<-A+[addr16]`                                           |
|`ADDB data8`        |`ADD B,data8`       |`B<-B+data8`                                              |
|`ADDB {addr8}`      |`ADD B,{addr8}`     |`B<-B+{addr8}`                                            |
|`ADDB [X+data8]`    |`ADD B,[X+data8]`   |`B<-B+[X+data8]`                                          |
|`ADDB [addr16]`     |`ADD B,[addr16]`    |`B<-B+[addr16]`                                           |
|`ANDA data8`        |`AND A,data8`       |`A<-A&data8`                                              |
|`ANDA {addr8}`      |`AND A,{addr8}`     |`A<-A&{addr8}`                                            |
|`ANDA [X+data8]`    |`AND A,[X+data8]`   |`A<-A&[X+data8]`                                          |
|`ANDA [addr16]`     |`AND A,[addr16]`    |`A<-A&[addr16]`                                           |
|`ANDB data8`        |`AND B,data8`       |`B<-B&data8`                                              |
|`ANDB {addr8}`      |`AND B,{addr8}`     |`B<-B&{addr8}`                                            |
|`ANDB [X+data8]`    |`AND B,[X+data8]`   |`B<-B&[X+data8]`                                          |
|`ANDB [addr16]`     |`AND B,[addr16]`    |`B<-B&[addr16]`                                           |
|`ASLA`              |`ASL A`             |Arithmetic shift left on `A`, bit 0 is set to `0`         |
|`ASLB`              |`ASL B`             |Arighmatic shift Left on `B`, bit 0 is set to `0`         |
|`ASL [X+data8]`     |`ASL [X+data8]`     |Arithmetic shift left on `[X+data8]`, bit 0 is set to `0` |
|`ASL [addr16]`      |`ASL [addr16]`      |Arithmetic shift left on `[addr16]`, bit 0 is set to `0`  |
|`ASRA`              |`ASR A`             |Arithmetic shift right on `A`, bit 0 is set to `0`        |
|`ASRB`              |`ASR B`             |Arighmatic shift right on `B`, bit 0 is set to `0`        |
|`ASR [X+data8]`     |`ASR [X+data8]`     |Arithmetic shift right on `[X+data8]`, bit 0 is set to `0`|
|`ASR [addr16]`      |`ASR [addr16]`      |Arithmetic shift right on `[addr16]`, bit 0 is set to `0` |
|`BCC disp`          |                    |if `C=0` then `PC<-PC+disp+2`                             |
|`BCS disp`          |                    |if `C=1` then `PC<-PC+disp+2`                             |
|`BEQ disp`          |                    |if `Z=1` then `PC<-PC+disp+2`                             |
|`BGE disp`          |                    |if `N^V=0` then `PC<-PC+disp+2`                           |
|`BGT disp`          |                    |if `(N^V)|Z=0` then `PC<-PC+disp+2`                       |
|`BHI disp`          |                    |if `C|Z=0` then `PC<-PC+disp+2`                           |
|`BITA data8`        |`BIT A,data8`       |`A&data8`                                                 |
|`BITA {addr8}`      |`BIT A,{addr8}`     |`A&{addr8}`                                               |
|`BITA [X+data8]`    |`BIT A,[X+data8]`   |`A&[X+data8]`                                             |
|`BITA [addr16]`     |`BIT A,[addr16]`    |`A&[addr16]`                                              |
|`BITB data8`        |`BIT B,data8`       |`B&data8`                                                 |
|`BITB {addr8}`      |`BIT B,{addr8}`     |`B&{addr8}`                                               |
|`BITB [X+data8]`    |`BIT B,[X+data8]`   |`B&[X+data8]`                                             |
|`BITB [addr16]`     |`BIT B,[addr16]`    |`B&[addr16]`                                              |
|`BLE disp`          |                    |if `(N^V)|Z=1` then `PC<-PC+disp+2`                       |
|`BLS disp`          |                    |if `C|Z=1` then `PC<-PC+disp+2`                           |
|`BLT disp`          |                    |if `N^V=1` then `PC<-PC+disp+2`                           |
|`BMI disp`          |                    |if `N=1` then `PC<-PC+disp+2`                             |
|`BNE disp`          |                    |if `Z=0` then `PC<-PC+disp+2`                             |
|`BPL disp`          |                    |if `N=0` then `PC<-PC+disp+2`                             |
|`BRA disp`          |                    |`PC<-PC+disp+2`                                           |
|`BVC disp`          |                    |if `V=0` then `PC<-PC+disp+2`                             |
|`BVS disp`          |                    |if `V=1` then `PC<-PC+disp+2`                             |
|`CBA`               |                    |A-B                                                       |
|`CLC`               |                    |C<-0                                                      |
|`CLI`               |                    |I<-0                                                      |
|`CLRA`              |`CLR A`             |`A<-0`                                                    |
|`CLRB`              |`CLR B`             |`B<-0`                                                    |
|`CLR [X+data8]`     |`CLR [X+data8]`     |`[X+data8]<-0`                                            |
|`CLR [addr16]`      |`CLR [addr16]`      |`[addr16]<-0`                                             |
|`CLV`               |                    |`V<-0`                                                    |
|`CMPA data8`        |`CMP A,data8`       |`A-data8`                                                 |
|`CMPA {addr8}`      |`CMP A,{addr8}`     |`A-{addr8}`                                               |
|`CMPA [X+data8]`    |`CMP A,[X+data8]`   |`A-[X+data8]`                                             |
|`CMPA [addr16]`     |`CMP A,[addr16]`    |`A-[addr16]`                                              |
|`CMPB data8`        |`CMP B,data8`       |`B-data8`                                                 |
|`CMPB {addr8}`      |`CMP B,{addr8}`     |`B-{addr8}`                                               |
|`CMPB [X+data8]`    |`CMP B,[X+data8]`   |`B-[X+data8]`                                             |
|`CMPB [addr16]`     |`CMP B,[addr16]`    |`B-[addr16]`                                              |
|`COMA`              |`COM A`             |`A<-0xff-A`                                               |
|`COMB`              |`COM B`             |`B<-0xff-B`                                               |
|`COM [X+data8]`     |`COM [X+data8]`     |`[X+data8]<-0xff-[X+data8]`                               |
|`COM [addr16]`      |`COM [addr16]`      |`[addr16]<-0xff-[addr16]`                                 |
|`CPX {addr8}`       |                    |`X(hi)-{addr8}, X(lo)-{addr8+1}`                          |
|`CPX [X+data8]`     |                    |`X(hi)-[X+data8], X(lo)-[X+data8+1]`                      |
|`CPX data16`        |                    |`X(hi)-data16(hi), X(lo)-data16(lo)`                      |
|`CPX [addr16]`      |                    |`X(hi)-[addr16], X(lo)-[addr16+1]`                        |
|`DAA`               |                    |Decimal adjust on `A`                                     |
|`DECA`              |`DEC A`             |`A<-A-1`                                                  |
|`DECB`              |`DEC B`             |`B<-B-1`                                                  |
|`DEC [X+data8]`     |`DEC [X+data8]`     |`[X+data8]<-[X+data8]-1`                                  |
|`DEC [addr16]`      |`DEC [addr16]`      |`[addr16]<-[addr16]-1`                                    |
|`DES`               |                    |`SP<-SP-1`                                                |
|`DEX`               |                    |`X<-X-1`                                                  |
|`EORA data8`        |`EOR A,data8`       |`A<-A^data8`                                              |
|`EORA {addr8}`      |`EOR A,{addr8}`     |`A<-A^{addr8}`                                            |
|`EORA [X+data8]`    |`EOR A,[X+data8]`   |`A<-A^[X+data8]`                                          |
|`EORA [addr16]`     |`EOR A,[addr16]`    |`A<-A^[addr16]`                                           |
|`EORB data8`        |`EOR B,data8`       |`B<-B^data8`                                              |
|`EORB {addr8}`      |`EOR B,{addr8}`     |`B<-B^{addr8}`                                            |
|`EORB [X+data8]`    |`EOR B,[X+data8]`   |`B<-B^[X+data8]`                                          |
|`EORB [addr16]`     |`EOR B,[addr16]`    |`B<-B^[addr16]`                                           |
|`INCA`              |`INC A`             |`A<-A+1`                                                  |
|`INCB`              |`INC B`             |`B<-B+1`                                                  |
|`INC [X+data8]`     |`INC [X+data8]`     |`[X+data8]<-[X+data8]+1`                                  |
|`INC [addr16]`      |`INC [addr16]`      |`[addr16]<-[addr16]+1`                                    |
|`INS`               |                    |`SP<-SP+1`                                                |
|`INX`               |                    |`X<-X+1`                                                  |
|`JMP X+data8`       |                    |`PC<-X+data8`                                             |
|`JMP addr16`        |                    |`PC<-addr16`                                              |
|`JSR X+data8`       |                    |`[SP]<-PC(lo), [SP+1]<-PC(hi), SP<-SP-2, PC<-X+data8`     |
|`JSR addr16`        |                    |`[SP]<-PC(lo), [SP+1]<-PC(hi), SP<-SP-2, PC<-addr16`      |
|`LDAA data8`        |`LDA A,data8`       |`A<-data8`                                                |
|`LDAA {addr8}`      |`LDA A,{addr8}`     |`A<-{addr8}`                                              |
|`LDAA [X+data8]`    |`LDA A,[X+data8]`   |`A<-[X+data8]`                                            |
|`LDAA [addr16]`     |`LDA A,[addr16]`    |`A<-[addr16]`                                             |
|`LDAB data8`        |`LDA B,data8`       |`B<-data8`                                                |
|`LDAB {addr8}`      |`LDA B,{addr8}`     |`B<-{addr8}`                                              |
|`LDAB [X+data8]`    |`LDA B,[X+data8]`   |`B<-[X+data8]`                                            |
|`LDAB [addr16]`     |`LDA B,[addr16]`    |`B<-[addr16]`                                             |
|`LDS {addr8}`       |                    |`SP(hi)<-{addr8}, SP(lo)<-{addr8+1}`                      |
|`LDS [X+data8]`     |                    |`SP(hi)<-[X+data8], SP(lo)<-[X+data8+1]`                  |
|`LDS data16`        |                    |`SP(hi)<-data16(hi), SP(lo)<-data16(lo)`                  |
|`LDS [addr16]`      |                    |`SP(hi)<-[addr16], SP(lo)<-[addr16+1]`                    |
|`LDX {addr8}`       |                    |`X(hi)<-{addr8}, X(lo)<-{addr8+1}`                        |
|`LDX [X+data8]`     |                    |`X(hi)<-[X+data8], X(lo)<-[X+data8+1]`                    |
|`LDX data16`        |                    |`X(hi)<-data16(hi), X(lo)<-data16(lo)`                    |
|`LDX [addr16]`      |                    |`X(hi)<-[addr16], X(lo)<-[addr16+1]`                      |
|`LSRA`              |`LSR A`             |Logical shift right on `A`, bit 7 is set to `0`           |
|`LSRB`              |`LSR B`             |Logical shift right on `B`, bit 7 is set to `0`           |
|`LSR [X+data8]`     |`LSR [X+data8]`     |Logical shift right on `[X+data8]`, bit 7 is set to `0`   |
|`LSR [addr16]`      |`LSR [addr16]`      |Logical shift right on `[addr16]`, bit 7 is set to `0`    |
|`NEGA`              |`NEG A`             |`A<-0-A`                                                  |
|`NEGB`              |`NEG B`             |`B<-0-B`                                                  |
|`NEG [X+data8]`     |`NEG [X+data8]`     |`[X+data8]<-0-[X+data8]`                                  |
|`NEG [addr16]`      |`NEG [addr16]`      |`[addr16]<-0-[addr16]`                                    |
|`NOP`               |                    |No operation                                              |
|`ORAA data8`        |`ORA A,data8`       |`A<-A|data8`                                              |
|`ORAA {addr8}`      |`ORA A,{addr8}`     |`A<-A|{addr8}`                                            |
|`ORAA [X+data8]`    |`ORA A,[X+data8]`   |`A<-A|[X+data8]`                                          |
|`ORAA [addr16]`     |`ORA A,[addr16]`    |`A<-A|[addr16]`                                           |
|`ORAB data8`        |`ORA B,data8`       |`B<-B|data8`                                              |
|`ORAB {addr8}`      |`ORA B,{addr8}`     |`B<-B|{addr8}`                                            |
|`ORAB [X+data8]`    |`ORA B,[X+data8]`   |`B<-B|[X+data8]`                                          |
|`ORAB [addr16]`     |`ORA B,[addr16]`    |`B<-B|[addr16]`                                           |
|`PSHA`              |`PSH A`             |`[SP]<-A, SP<-SP-1`                                       |
|`PSHB`              |`PSH B`             |`[SP]<-B, SP<-SP-1`                                       |
|`PULA`              |`PUL A`             |`SP<-SP+1, A<-[SP]`                                       |
|`PULB`              |`PUL B`             |`SP<-SP+1, B<-[SP]`                                       |
|`ROLA`              |`ROL A`             |Rotate left through carry flag on `A`                     |
|`ROLB`              |`ROL B`             |Rotate left through carry flag on `B`                     |
|`ROL [X+data8]`     |`ROL [X+data8]`     |Rotate left through carry flag on `[X+data8]`             |
|`ROL [addr16]`      |`ROL [addr16]`      |Rotate left through carry flag on `[addr16]`              |
|`RORA`              |`ROR A`             |Rotate right through carry flag on `A`                    |
|`RORB`              |`ROR B`             |Rotate right through carry flag on `B`                    |
|`ROR [X+data8]`     |`ROR [X+data8]`     |Rotate right through carry flag on `[X+data8]`            |
|`ROR [addr16]`      |`ROR [addr16]`      |Rotate right through carry flag on `[addr16]`             |
|`RTI`               |                    |Return from interrupt.                                    |
|`RTS`               |                    |`PC(hi)<-[SP+1], PC(lo)<-[SP+2], SP<-SP+2`                |
|`SBA`               |                    |`A<-A-B`                                                  |
|`SBCA data8`        |`SBC A,data8`       |`A<-A-data8-C`                                            |
|`SBCA {addr8}`      |`SBC A,{addr8}`     |`A<-A-{addr8}-C`                                          |
|`SBCA [X+data8]`    |`SBC A,[X+data8]`   |`A<-A-[X+data8]-C`                                        |
|`SBCA [addr16]`     |`SBC A,[addr16]`    |`A<-A-[addr16]-C`                                         |
|`SBCB data8`        |`SBC B,data8`       |`B<-B-data8-C`                                            |
|`SBCB {addr8}`      |`SBC B,{addr8}`     |`B<-B-{addr8}-C`                                          |
|`SBCB [X+data8]`    |`SBC B,[X+data8]`   |`B<-B-[X+data8]-C`                                        |
|`SBCB [addr16]`     |`SBC B,[addr16]`    |`B<-B-[addr16]-C`                                         |
|`SEC`               |                    |`C<-1`                                                    |
|`SEI`               |                    |`I<-1`                                                    |
|`SEV`               |                    |`V<-1`                                                    |
|`STAA {addr8}`      |`STA A,{addr8}`     |`{addr8}<-A`                                              |
|`STAA [X+data8]`    |`STA A,[X+data8]`   |`[X+data8]<-A`                                            |
|`STAA [addr16]`     |`STA A,[addr16]`    |`[addr16]<-A`                                             |
|`STAB {addr8}`      |`STA B,{addr8}`     |`{addr8}<-B`                                              |
|`STAB [X+data8]`    |`STA B,[X+data8]`   |`[X+data8]<-B`                                            |
|`STAB [addr16]`     |`STA B,[addr16]`    |`[addr16]<-B`                                             |
|`STS {addr8}`       |                    |`{addr8}<-SP(hi), {add8+1}<-SP(lo)`                       |
|`STS [X+data8]`     |                    |`[X+data8]<-SP(hi), [X+data8+1]<-SP(lo)`                  |
|`STS [addr16]`      |                    |`[addr16]<-SP(hi), [addr16+1]<-SP(lo)`                    |
|`STX {addr8}`       |                    |`{addr8}<-X(hi), {addr8+1}<-X(lo)`                        |
|`STX [X+data8]`     |                    |`[X+data8]<-X(hi), [X+data8+1]<-X(lo)`                    |
|`STX [addr16]`      |                    |`[addr16]<-X(hi), [addr16+1]<-X(lo)`                      |
|`SUBA data8`        |`SUB A,data8`       |`A<-A-data8`                                              |
|`SUBA {addr8}`      |`SUB A,{addr8}`     |`A<-A-{addr8}`                                            |
|`SUBA [X+data8]`    |`SUB A,[X+data8]`   |`A<-A-[X+data8]`                                          |
|`SUBA [addr16]`     |`SUB A,[addr16]`    |`A<-A-[addr16]`                                           |
|`SUBB data8`        |`SUB B,data8`       |`B<-B-data8`                                              |
|`SUBB {addr8}`      |`SUB B,{addr8}`     |`B<-B-{addr8}`                                            |
|`SUBB [X+data8]`    |`SUB B,[X+data8]`   |`B<-B-[X+data8]`                                          |
|`SUBB [addr16]`     |`SUB B,[addr16]`    |`B<-B-[addr16]`                                           |
|`SWI`               |                    |Invoke software interrupt                                 |
|`TAB`               |                    |`B<-AP`                                                   |
|`TAP`               |                    |`SR<-A`                                                   |
|`TBA`               |                    |`A<-B`                                                    |
|`TPA`               |                    |`A<-SR`                                                   |
|`TSTA`              |`TST A`             |`A-0`                                                     |
|`TSTB`              |`TST B`             |`B-0`                                                     |
|`TST [X+data8]`     |`TST [X+data8]`     |`[X+data8]-0`                                             |
|`TST [addr16]`      |`TST [addr16]`      |`[addr16]-0`                                              |
|`TSX`               |                    |`X<-SP+1`                                                 |
|`TXS`               |                    |`SP<-X-1`                                                 |
|`WAI`               |                    |Wait for interrupt                                        |

Operands:

- `A` ... accumulator A
- `B` ... accumulator B
- `X` ... index register
- `SP` ... stack pointer
- `PC` ... program counter
- `SR` ... status register
- `data8` ... immediate 8 bit value
- `data16` ... immediate 16 bit value
- `{addr8}` ... direct addressing to the internal RAM
- `[X+data8]` ... index addresing
- `[addr16]` ... external addressing to the external RAM or ROM

Meaning of operations:

- `+` ... addition
- `-` ... subtraction
- `&` ... bitwise AND
- `|` ... bitwise OR
- `^` ... bitwise XOR

The status register `SR` holds following flags:

- `C` ... carry flag
- `V` ... overflow flag
- `Z` ... zero flag
- `N` ... negative flag
- `I` ... interrupt mask flag

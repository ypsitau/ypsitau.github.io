---
layout: jrasm-nonavi
title: jrasm - Assembler for JR-200
tracking_id: UA-38761061-3
---
{% assign jrasm_version = '1.0.0' %}

<div class="jumbotron">
  <div class="container-fluid">
	<h1 class="display-4">jrasm - Assembler for JR-200
	  <span class="float-right">
		<a class="btn btn-secondary" href="https://github.com/ypsitau/jrasm/releases/download/v{{ jrasm_version }}/jrasm-{{ jrasm_version }}.zip">
		  <i class="fas fa-download mr-2"></i>Download
		</a>
		<a class="btn btn-secondary" href="https://github.com/ypsitau/jrasm">
		  <i class="fab fa-github mr-2"></i>View on GitHub
		</a>
	  </span>
	</h1>
	<p class="lead">
	An assembler of Motorola 6800 MPU with features useful to program for JR-200.
	</p>
  </div>
</div>


## What's This?

This is an assembler of Motorola 6800 MPU, an 8bit CPU designed by Motorola in 1974.
And it has some features that I expect are useful to program for JR-200, a micro computer made by Panasonic in 1983.

This project has been inspired by JR-200 emulator VJR-200 developed by FIND,
which is available [here](http://www17.plala.or.jp/find_jr200/vjr200_en.html).

## Installation

A Windows executable is available [here](https://github.com/ypsitau/jrasm/releases/download/v{{ jrasm_version }}/jrasm-{{ jrasm_version }}.zip).
Expand its content in a directory that is included in `PATH` environment variable.

It has been tested in Windows 10 64bit.


## Let's Try It

Consider the following source file named `helloworld.asm`:

```
        .ORG    0x1000
loop:
        LDX     [ptr_src]
        LDAA    [X]
        BEQ     done
        INX
        STX     [ptr_src]
        LDX     [ptr_dst]
        STAA    [X]
        INX
        STX     [ptr_dst]
        BRA     loop
done:
        RTS
ptr_src:
        .DW     hello_world
ptr_dst:
        .DW     0xc100 + 9 + 20 * 0x20
hello_world:
        .DB     "Hello, world!", 0
```

To assemble it, just launch `jrasm` with the source file name.

```
> jrasm helloworld.asm
```

This creates a CJR file named `helloworld.cjr`. You can load the CJR file to a JR-200 emulator
or convert it into a WAV file that is fed to a real machine through a cassette recorder interface.

After loading it, you can call the program using JR-200 BASIC like follows:

```
U=USR($1000)
```

You see "Hello, world!" on the screen? Congratulations!

Specifying `-D` options prints the produced binary data with corresponding assembler codes
to the standard output.

```
> jrasm -D helloworld.asm
```

The result is:

```
                   .ORG 0x1000
loop:
    1000 FE 10 17  LDX  [ptr_src]
    1003 A6 00     LDAA [X]
    1005 27 0F     BEQ  done
    1007 08        INX 
    1008 FF 10 17  STX  [ptr_src]
    100B FE 10 19  LDX  [ptr_dst]
    100E A7 00     STAA [X]
    1010 08        INX 
    1011 FF 10 19  STX  [ptr_dst]
    1014 20 EA     BRA  loop
done:
    1016 39        RTS 
ptr_src:
    1017 10 1B     .DW  hello_world
ptr_dst:
    1019 C3 89     .DW  0xc100+9+20*0x20
hello_world:
    101B 48 65     .DB  "Hello, world!",0
    101D 6C 6C   
    101F 6F 2C   
    1021 20 77   
    1023 6F 72   
    1025 6C 64   
    1027 21 00   
```

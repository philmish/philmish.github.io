# Practical Malware Analysis Chapter 1 Lab 3


This lab turned out to be a bit more complicated than the two before. I spend a considerable amount of time figuring out how to unpack the provided sample, partly due to problems with setting up the tooling I wanted to use correctly (python3.10 backwards compatability can be a bitch).

In addition to [PE Bear](https://github.com/hasherezade/pe-bear) and [Detect it easy](https://github.com/horsicq/Detect-It-Easy) I used [PEiD](https://github.com/wolfram77web/app-peid) to identify the packer and a tool I found called `unipacker`. More about that later in the post.

## Questions

1. Are there any signature detections on VT ?
2. Are there indicators for packing or obfuscation ? If so, unpack/de-obfuscate the sample.
3. Do any imports hint at the samples functionality ? If so which imports are they and what do they tell you ?
4. What host-based or network-based indicators could be used to identify this malware on infected machines ?

## Initial analysis

As I did with the two samples before I ran `Detect it easy` against the provided sample. There were a few things I noticed intriguing.

1. There were no signatures detected for a compiler or linker
2. The time stamp showed as `1970-01-01 01:00:00`, a sign that it was tempered with
3. `Detect it easy` also couldn't extract any information on imports from the sample

### Extracted Hashes

- MD5 `9c5c27494c28ed0b14853b346b113145`
- SHA1 `290ab6f431f46547db2628c494ce615d6061ceb8`
- SHA256 `7983a582939924c70e3da2da80fd3352ebc90de7b8c4c427d484ff4f050f0aec`

### Imports

The only imports found by `PE Bear` are `LoadLibraryA` and `GetProcAddress` from `kernel32.dll`.
These are used for importing DLLs and their functionality.

### Is it packed ?

These discoveries made me take a look at the sample with `PEiD`, a program used to detect common packing formats on binaries.

![PEiD Output](images/prac_mal_ana_ch1_l3_peid.png "Figure 1. PEiD packer detection")

As we can see from the output, the sample is indeed packed. The packer used is called `FSG`. After a little bit of research I found [this article](https://www.mnin.org/write/2006_unpacking_fsg.pdf) which describes how to unpack an executbale packed with `FSG` with the help of a debugger.

### How FSG works

I am still in the phase of getting started with learning the concepts of x86 assembly so alot of this still went over my head, but I am trying to describe what I took from the article.

The article describes that the first instructions of the program is an `xchg`, which is used to make the stack pointer (`ESP`) to a DWORD, in the case of the article at offset `4094E8h`.
This DWORD is `4094CCh`, which points the stack pointer to a segment containing additional data.


After the first `xchg` instruction the two values on top of the stack are `00401000h` and `00407000h`. After the `xchg` the next instruction is a `popa`.
This instructions pops values of the stack into the general purpose registers. `popa` ,which is the next instruction, is used to pop `WORD` sized, so 16bit, values into the subregisters of the general purpose registers.
The order in which the sub regsiters are filled in this case are: `DI` `SI` `BP` `BX` and so on. So in case of the prgram we look at in the article means that the lower 16 bit of the `EDI` and `ESI` registers will be set with the values on top of the stack.


Researching those registers online (and by going ahead a bit in the book) I found out that those regsiters are often used with `rep` instructions, such as `movsb`.
These instructions are used to automatically move a bunch of bytes from one buffer to another.


This matches with the functionality further explained by the article. It states, that `EDI` and `ESI` are used to move bytes between two segments delimited by the two addresses in the registers after the `popa`, `00401000h` and `00407000h`.
It is also stated that in case of the sample looked at in the article, this goes on until the first segment contains the names of the `DLLs` and their exports used by the program.
After that `LoadLibraryA` and `GetProcAddress` are used to actually import them. This unveils the actual functionality of the program.


### So, can we unpack it ?

During my research about `FSG` I found that most of the things which came up are actual write-ups or videos about this lab.
Because it felt like cheating to me at first I left those alone.
But after reading up on the packer I was curious to find out what the unpacked version of the sample actually does.


I found [this video](https://www.youtube.com/watch?v=ee5_JUIEf8Q&ab_channel=Prof.ChrisDietrich) which demonstrates the usage of a tool written in python called `unipacker` for which the code can be found [here](https://github.com/unipacker/unipacker/tree/master)
Unipacker uses emulation to, like a debugger, step through a program for unpacking. It is based on `Capstone` one of the biggest disassembler library out there. It is also used in a lot of other tooling.


The use of the tool condenses the effort of unpacking the sample down to three instructions provided to unipacker. My assumption is that it steps through the program until the data moving and importing part is over and extracts the resulting PE from it, reconstructing the actual import table.


After some struggles with getting it to run with python3.10 I used `pyenv` to install python3.8 to my environment and `unipacker` ran without problems.
As python3.10 moved some imports from one module of the stdlib to another alot of older python code ran under 3.10 breaks on these older imports.
I encountered the same problem with some older code at work.

## The unpacked Sample

This section will be kept short. I used the usual tools to collect static information from the unpacked sample.

### Hashes

- MD5 `0b828a7ccf370a5c9d5ca4bcba6dbebe`
- SHA1 `08f610cd9d694fc08fe12356001d07bf2600c8ec`
- SHA256 `0fd9f5bcacddc526091630e269de92c50da8b7e85186d68a75418cade20475be`

### Strings

```text
Offset	Size	Type	String
0000004d	10	A	!Windows Program
0000020f	07	A	`.rdata
00000237	06	A	@.data
00000e2e	05	A	$s3
00000f34	0c	A	KERNEL32.dll
00000f42	0c	A	LoadLibraryA
00000f50	0e	A	GetProcAddress
00001013	05	A	Phh @
0000106e	06	A	L$$QVP
00002167	09	A	ole32.dll
00002171	0d	A	OleInitialize
0000217f	10	A	CoCreateInstance
00002190	0f	A	OleUninitialize
000021a5	0c	A	OLEAUT32.dll
000021c9	0a	A	MSVCRT.dll
000021d4	0d	A	__getmainargs
000021e2	0a	A	_controlfp
000021ed	10	A	_except_handler3
000021fe	0e	A	__set_app_type
0000220d	0a	A	__p__fmode
00002218	0c	A	__p__commode
00002225	05	A	_exit
0000222b	0b	A	_XcptFilter
0000223c	0d	A	__p___initenv
0000224a	09	A	_initterm
00002254	10	A	__setusermatherr
00002265	0c	A	_adjust_fdiv
00003010	2a	U	http://www.malwareanalysisbook.com/ad.html
00004015	05	A	Ph8j
00004025	05	A	3Bt>O
00004042	06	A	2]<,M
00004089	07	A	 S>VWe
000040ca	05	A	"Z,Y
000040e6	05	A	5pg
00004133	06	A	@^J%
00004144	06	A	I*G9>
000041b3	07	A	<d,Ll
000041c5	08	A	ole32.vd
000041d1	05	A	Init
000041ec	05	A	U!!C
000041f4	09	A	}OLEAUTLA
0000420a	0a	A	IMSVCRTT"b
00004215	07	A	_getmas
0000421d	07	A	yrcsco
00004228	06	A	fpex
00004235	07	A	|P2r3Us
0000423f	05	A	p|vuy
00004260	05	A	rh#
0000502e	05	A	$s3
00005134	0c	A	KERNEL32.dll
00005142	0c	A	LoadLibraryA
00005150	0e	A	GetProcAddress
00009050	09	A	ole32.dll
0000905a	0c	A	OLEAUT32.dll
00009067	0a	A	MSVCRT.dll
000090a0	0d	A	OleInitialize
000090b0	10	A	CoCreateInstance
000090c3	0f	A	OleUninitialize
00009145	0d	A	__getmainargs
00009155	0a	A	_controlfp
00009162	10	A	_except_handler3
00009175	0e	A	__set_app_type
00009186	0a	A	__p__fmode
00009193	0c	A	__p__commode
000091a2	05	A	_exit
000091aa	0b	A	_XcptFilter
000091bf	0d	A	__p___initenv
000091cf	09	A	_initterm
000091db	10	A	__setusermatherr
000091ee	0c	A	_adjust_fdiv
```

### Imports

The unpacked sample contains imports from `ole32.dll` and `oleauth32.dll`. Those are libraries used for the management and manipulation of `COM` objects.


`COM` or `Component Object Model` is a standard for binary interfaces, published in 1993 by Microsoft. [Wikipedia](https://en.wikipedia.org/wiki/Component_Object_Model)


According to the Wiki article it is the basis for several of Microsofts technologies. One which correlates to the `ole` libraries we saw, which might be interesting in the context of malware is the possibility for inter process communication and automation provided by `ole`.
These might be used to gain persistence by injecting code in automated startup procedures.

## Answers

1. There are multiple hits for the sample in its packed as well as in its unpacked version
2. The sample is packed with the `sfg (safe, fast good)` packer. To unpack it I used the emulation tool `unipack`. It performed operations usualy done with a debugger to extract the unpacked program code from the sample. After some setup the tool worked and extracted the unpacked program code successfuly
3. The unpacked sample imports the `ole32.dll` library, which is used to create and control COM Objects.  
4. After extracting strings from the unpacked sample I found a web address `http://www.malwareanalysisbook.com/ad.html` which could be used to monitor for malicious traffic. For host-based indicators I dont know if the interaction with COM Objects might provide a possibility to monitor for malicious activity from the sample on an infected host

### PS
After finishing my write-up I took a look at the solution provided by the book. Turns out the book didn't expect you to unpack the sample  

`¯\_(ツ)_/¯`


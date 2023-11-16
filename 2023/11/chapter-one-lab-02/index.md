# Practical Malware Analysis Chapter 1 Lab 2


As the first writeup got a little bit long, this one will be a little bit shorter. The second lab mainly deals with packing.

## Questions

1. Does the sample match any existing signatures on VT ?
2. Are there indicators that the sample is packed or obfuscated ? If it is packed unpack it if possible.
3. Do any imports hint at the program's functionality ? Which Imports are there and what do they tell you ?
4. What host and network based indicators could be used to identify this malware on infected machines ?

## Unpacking

Running the file through `Detect it easy` and `PE Bear` we see that the sections have weired names like `UPX1` and the signature suggests that this file is indeed packed with the upx packer.

![DetectItEasy](images/prac_mal_ana_ch1_l2_sig.png "Figure 1. Detect it easy signature")

![PEBear](images/prac_mal_ana_ch1_l2_sections.png "Figure 2. PE Bear Section View")

Luckily, it is really easy to unpack files packed with upx. All we need to do is to donwload the upx packer program and run the `unpack` command.

```powershell
.\upx.exe -d ..\Lab01-02.exe -o ..\Lab01-02-unpacked.exe
```

## Hashes

### Packed

- MD5 `8363436878404da0ae3e46991e355b83`
- Sha1 `5a016facbcb77e2009a01ea5c67b39af209c3fcb`
- SHA 256 `c876a332d7dd8da331cb8eee7ab7bf32752834d4b2b54eaa362674a2a48f64a6`

### Unpacked

- MD5 `ae4ca70697df5506bc610172cfc288e7`
- SHA1 `31e8a82e497058ff14049cf283b337ec51504819`
- SHA256 `8bcbe24949951d8aae6018b87b5ca799efe47aeb623e6e5d3665814c6d59aeae`

## Strings

### Packed

```
Offset	Size	Type	String
004d	28	A	!This program cannot be run in DOS mode.
04ab	07	A	" zRV$
058d	06	A	mCDmM
05fb	0a	A	MalService
0609	07	A	sHGL345
0611	08	A	http://w
061e	09	A	.mwarean
0628	0b	A	ysisbook.co
0636	17	A	om#Int6net Explo!r 8FEI
0663	16	A	SystemTimeToFileGetMo
0685	05	A	Cvg+
068b	0c	A	*Waitab'rEx
069c	07	A	Process
06a4	09	A	OpenMu$x
06c2	08	A	ObjectU4
06dc	05	A	[Vrtb
06e5	0b	A	CtrlDisp ch
072b	07	A	-msus
0736	05	A	5nm@_
073e	06	A	t_fd
0769	06	A	dlI37n
07db	06	A	lB`.rd
0979	09	A	`(XPTPSW
0a98	0c	A	KERNEL32.DLL
0aa5	0c	A	ADVAPI32.dll
0ab2	0a	A	MSVCRT.dll
0abd	0b	A	WININET.dll
0aca	0c	A	LoadLibraryA
0ad8	0e	A	GetProcAddress
0ae8	0e	A	VirtualProtect
0af8	0c	A	VirtualAlloc
0b06	0b	A	VirtualFree
0b14	0b	A	ExitProcess
0b22	0e	A	CreateServiceA
0b38	0d	A	InternetOpenA
```

### Unpacked

```
Offset	Size	Type	String
004d	28	A	!This program cannot be run in DOS mode.
01d8	05	A	.text
01ff	07	A	`.rdata
0227	06	A	@.data
1064	05	A	Vh(0@
1092	05	U	@jjjj
216c	0c	A	KERNEL32.DLL
2179	0c	A	ADVAPI32.dll
2186	0a	A	MSVCRT.dll
2191	0b	A	WININET.dll
21a0	14	A	SystemTimeToFileTime
21b6	12	A	GetModuleFileNameA
21ca	14	A	CreateWaitableTimerA
21e0	0b	A	ExitProcess
21ee	0a	A	OpenMutexA
21fa	10	A	SetWaitableTimer
220c	13	A	WaitForSingleObject
2222	0c	A	CreateMutexA
2230	0c	A	CreateThread
223e	0e	A	CreateServiceA
224e	1b	A	StartServiceCtrlDispatcherA
226c	0e	A	OpenSCManagerA
227c	05	A	_exit
2284	0b	A	_XcptFilter
2298	0d	A	__p___initenv
22a8	0d	A	__getmainargs
22b8	09	A	_initterm
22c4	10	A	__setusermatherr
22d6	0c	A	_adjust_fdiv
22e4	0c	A	__p__commode
22f2	0a	A	__p__fmode
22fe	0e	A	__set_app_type
230e	10	A	_except_handler3
2320	0a	A	_controlfp
232c	10	A	InternetOpenUrlA
233e	0d	A	InternetOpenA
3010	0a	A	MalService
301c	0a	A	Malservice
3028	06	A	HGL345
3030	22	A	http://www.malwareanalysisbook.com
3054	15	A	Internet Explorer 8.0
```

## Sandboxes

- [VT Report]([VirusTotal - File - c876a332d7dd8da331cb8eee7ab7bf32752834d4b2b54eaa362674a2a48f64a6](https://www.virustotal.com/gui/file/c876a332d7dd8da331cb8eee7ab7bf32752834d4b2b54eaa362674a2a48f64a6/detection))
- [Malware Bazaar](https://bazaar.abuse.ch/sample/c876a332d7dd8da331cb8eee7ab7bf32752834d4b2b54eaa362674a2a48f64a6/)
- [Unpac Me](https://www.unpac.me/results/37093850-1313-4a16-a801-6b1255661a31/#/)

## Imports from the unpacked sample

### Kernel32.dll

```
#	Thunk	Ordinal	Hint	Name
0	0000219e		0000	SystemTimeToFileTime
1	000021b4		0000	GetModuleFileNameA
2	000021c8		0000	CreateWaitableTimerA
3	000021de		0000	ExitProcess
4	000021ec		0000	OpenMutexA
5	000021f8		0000	SetWaitableTimer
6	0000220a		0000	WaitForSingleObject
7	00002220		0000	CreateMutexA
8	0000222e		0000	CreateThread
```

### Advapi32.dll

```
#	Thunk	Ordinal	Hint	Name
0	0000223c		0000	CreateServiceA
1	0000224c		0000	StartServiceCtrlDispatcherA
2	0000226a		0000	OpenSCManagerA
```

### Wininet.dll

```
#	Thunk	Ordinal	Hint	Name
0	0000232a		0000	InternetOpenUrlA
1	0000233c		0000	InternetOpenA
```

## Answers

1.  There are existing signatures for the packed as well as for the unpacked sample. Most of them label the sample as generic Win32 malware / Trojan
2. The sections of the PE file being named `UPX0` to `UPX2` hint at the fact that this sample is packed with upx. Luckily unpacking with UPX is as easy as calling the upx.exe with the right cli flags `-d`
3. The unpacked sample imports functions from `kernel32.dll`, `advapi32.dll` and `wininet.dll`. The `kernel32.dll` imports provide functionality for creating threads, creating and opening mutexes, exiting processes, retrieving filepaths for modules loaded by the current process, creating waitable timers and waiting for objects.
   The `advapi32.dll` imports functionality for creating a windows service, connecting a service to the ServiceControlDispatcher and opening the service control manager database. The last one requires administrator priviliges or it will fail.
   The `wininet.dll` imports provide functionality to initialize usage of the wininet functions and to open a resource via a FTP or HTTP URL.
   This could mean that the sample will try to create tasks as threads which will be awaited. It might persist by trying to establish itself as a Windows Service which is run on startup. The Network capabilities imported via `wininet.dll` are a hint that the sample might try to download files or data from a C2 server.
4. A network-based indicator is the URL `http://www.malwareanalysisbook.com` found in the extracted strings from the unpacked sample. A host-based indicator might be the Windows Service started by the maleware. The unpacked strings contain the names `MalService` and `Malservice` which could be the names used for the service created by the malware.  


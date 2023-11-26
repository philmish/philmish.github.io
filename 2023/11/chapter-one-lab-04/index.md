# Practical Malware Analysis Chapter 1 Lab 4


After falling down the rabbit hole of unpacking the previous sample, this one will be a bit shorter again.

## Questions


1. Are there any existing signatures for the sample on virus total ?
2. Are there indications that the file was obfuscated or packed ? If the file is packed, unpack it if possible.
3. When was the program compiled ?
4. Do any imports hint at the programs functionality ? If some which imports are they and what do they tell you ?
5. What host- and network-based indicators could be used to identify this malware on infected machines ?
6. This file has one resource in the resource section. Use resource hacker to examine that resource, and then use it to extract that resource. What can you learn from the resource ?

## Basic static analysis

### Online Resources

- [Virus Total Report](https://www.virustotal.com/gui/file/0fa1498340fca6c562cfa389ad3e93395f44c72fd128d7ba08579a69aaf3b126)

### Hashes

- MD5 `625ac05fd47adc3c63700c3b30de79ab`
- SHA1 `9369d80106dd245938996e245340a3c6f17587fe`
- SHA256 `9369d80106dd245938996e245340a3c6f17587fe`

### Strings

```text
Offset	Size	Type	String
004d	28	A	!This program cannot be run in DOS mode.
01e0	05	A	.text
0207	07	A	`.rdata
022f	06	A	@.data
0258	05	A	.rsrc
1284	05	A	Qhd0@
218a	0b	A	CloseHandle
2198	0b	A	OpenProcess
21a6	11	A	GetCurrentProcess
21ba	12	A	CreateRemoteThread
21d0	0e	A	GetProcAddress
21e2	0c	A	LoadLibraryA
21f2	07	A	WinExec
21fc	09	A	WriteFile
2208	0b	A	CreateFileA
2216	0e	A	SizeofResource
2228	0c	A	LoadResource
2238	0d	A	FindResourceA
2248	10	A	GetModuleHandleA
225c	14	A	GetWindowsDirectoryA
2274	09	A	MoveFileA
2280	0c	A	GetTempPathA
228e	0c	A	KERNEL32.dll
229e	15	A	AdjustTokenPrivileges
22b6	15	A	LookupPrivilegeValueA
22ce	10	A	OpenProcessToken
22e0	0c	A	ADVAPI32.dll
22f0	09	A	_snprintf
22fa	0a	A	MSVCRT.dll
2308	05	A	_exit
2310	0b	A	_XcptFilter
2326	0d	A	__p___initenv
2336	0d	A	__getmainargs
2346	09	A	_initterm
2352	10	A	__setusermatherr
2366	0c	A	_adjust_fdiv
2376	0c	A	__p__commode
2386	0a	A	__p__fmode
2394	0e	A	__set_app_type
23a6	10	A	_except_handler3
23ba	0a	A	_controlfp
23c8	08	A	_stricmp
3010	0c	A	winlogon.exe
3020	0a	A	<not real>
302c	10	A	SeDebugPrivilege
3040	0a	A	sfc_os.dll
304c	15	A	\system32\wupdmgr.exe
3078	12	A	EnumProcessModules
308c	09	A	psapi.dll
3098	12	A	GetModuleBaseNameA
30ac	09	A	psapi.dll
30b8	0d	A	EnumProcesses
30c8	09	A	psapi.dll
30d4	15	A	\system32\wupdmgr.exe
30f4	0a	A	\winup.exe
40ad	28	A	!This program cannot be run in DOS mode.
4128	05	A	Rich
4240	05	A	.text
4267	07	A	`.rdata
428f	06	A	@.data
50ed	05	A	Qh0@
5134	05	A	Rh<0@
5159	05	A	QhD0@
616a	14	A	GetWindowsDirectoryA
6182	07	A	WinExec
618c	0c	A	GetTempPathA
619a	0c	A	KERNEL32.dll
61aa	12	A	URLDownloadToFileA
61be	0a	A	urlmon.dll
61cc	09	A	_snprintf
61d6	0a	A	MSVCRT.dll
61e4	05	A	_exit
61ec	0b	A	_XcptFilter
6202	0d	A	__p___initenv
6212	0d	A	__getmainargs
6222	09	A	_initterm
622e	10	A	__setusermatherr
6242	0c	A	_adjust_fdiv
6252	0c	A	__p__commode
6262	0a	A	__p__fmode
6270	0e	A	__set_app_type
6282	10	A	_except_handler3
6296	0a	A	_controlfp
7070	0a	A	\winup.exe
7084	16	A	\system32\wupdmgrd.exe
70a4	33	A	http://www.practicalmalwareanalysis.com/updater.exe
```

### Imports

#### kernel32.dll

```text
#	Thunk	Ordinal	Hint	Name
0	000021ce		013e	GetProcAddress
1	000021e0		01c2	LoadLibraryA
2	000021f0		02d3	WinExec
3	000021fa		02df	WriteFile
4	00002206		0034	CreateFileA
5	00002214		0295	SizeofResource
6	000021b8		0046	CreateRemoteThread
7	00002236		00a3	FindResourceA
8	00002246		0126	GetModuleHandleA
9	0000225a		017d	GetWindowsDirectoryA
10	00002272		01dd	MoveFileA
11	0000227e		0165	GetTempPathA
12	000021a4		00f7	GetCurrentProcess
13	00002196		01ef	OpenProcess
14	00002188		001b	CloseHandle
15	00002226		01c7	LoadResource
```

#### advapi.dll

```text
#	Thunk	Ordinal	Hint	Name
0	000022cc		0142	OpenProcessToken
1	000022b4		00f5	LookupPrivilegeValueA
2	0000229c		0017	AdjustTokenPrivileges
```

At first glance, the simple does not seem to be packed. The signature shown by `Detect it easy` identifies the sample as a 32-Bit Windows PE file.

So far so good.

We can already spot a lot of interesting stuff here.

The imports detected by are also quite interesting. We can see imports from `advapi.dll` which can be used to look up and adjust privileges on security token.
We also can see imports from `kernel32.dll` for opening processes (`OpenProcess`), creating threads in other processes (`CreateRemoteThread`), writing (`WriteFile`) and moving (`MoveFileA`) files as well as functionality which can be used to obtain a handle to a resource in memory (`LoadResource`)


The extracted strings contain an URL, `http://www.practicalmalwareanalysis.com/updater.exe`, which looks like a link to another executable file.
Also, there are two occurences the known string contained in the DOS header of Microsoft PE files, `!This program cannot be run in DOS mode.`

Taking a closer look at the output of `Detect it easy` we can see that there is a resource contained in the binary which itself is a 32-bit PE file.
This also explains the second occurences of known section names such as `.text`.

## Extracting the resource

The book mentions [Resource Hacker](https://www.angusj.com/resourcehacker/) as a possible tool to be used to extract a resource, like icos, from binary files.
As I did't install it yet, I decided to take a more archaic approach by opening the sample in a Hex Editor, search for the `.rscr` section of the file and locate the Magic Bytes `MZ` to see where the attached PE File started.
After finding the location I just dumped all the bytes , beginning with `MZ`, to another file.


Loading the extracted file into `Detect it easy` shows us that this is indeed a 32-bit Windows PE file. 

### Hashes

- MD5 `ee5a2eaddb0050ea8d1c54a7811db9ab`
- SHA1 `8220bcd7f2850d55234bf633146d997c6d897688`
- SHA256 `bb1252dab9f573d7517083925db5fc6d8496afb56928cc848ad108c27542c448`


### Imports

#### kernel32.dll

```text
#	Thunk	Ordinal	Hint	Name
0	00002120		02d3	WinExec
1	0000212a		0165	GetTempPathA
2	00002108		017d	GetWindowsDirectoryA
```

#### urlmon.dll

```text
#	Thunk	Ordinal	Hint	Name
0	00002148		003e	URLDownloadToFileA
```

## Answers

1. There are existing signatures on VT. Most of them hint at a `Win32 Trojan/Download`
2. There seem to be no indicators that the sample is packed. But `detect it easy` shows us that a Win32 PE File is attached as a resource to the sample. After extracting it we can see that according to PE bear it is an executable. This might obfuscate the full functionality of the sample to people only looking at the `.text` section of the sample.
3. The File Header for the actual sample `Lab01-04.exe` shows a compilation timestamp of  `Friday, 30.08.2019 22:26:59 UTC`. The extracted resource shows a timestamp of `Sunday, 27.02.2011 00:16:59 UTC`.
4. The program's imports from `kernel32.dll` include functions for loading a executable file from memory with `LoadResource`, opening a process with `OpenProcess` as well as retrieving a handle to the current process with `GetCurrentProcess`. This could be used to run code attached to the resource section of the program. The sample also imports functionality for creating and moving Files on the system (`CreateFileA`, `MoveFileA`) which might be an indicator that the malware tries to persist itself on disk. Taking a look at the code extracted from the sample's resource we see that it uses `urlmon.dll` to import `URLDownloadToFileA`. This might be used to download the the `updater.exe` file from the URL found in the strings.
5. I found an URL in the extracted strings `http://www.practicalmalwareanalysis.com/updater.exe`, communication to this URL and the domain can be used to monitor the network for traffic to them
6. The extracted resource is a PE file itself, the extracted code contains imports from `urlmon.dll` which can be used to download a file from the internet (`URLDownloadToFileA`). The code also imports functions to retrieve the windows and temp directory as well as `WinExec`. It looks like the attached PE file from the resources is used to download a file and executing it on the system.







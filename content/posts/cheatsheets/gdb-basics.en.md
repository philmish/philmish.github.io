---
weight: 4
title: "GDB Basics Cheatsheet"
description: "The basic commands used for debugging with GDB over the Commandline"
date: "2023-05-16"
draft: false
author: "philmish"
images: []
tags: ["Cheatsheet", "Linux", "Debugging"]
categories: ["Cheatsheets"]
lightgallery: true
---

## Launching

### Launching against a binary
```shell
gdb ./path-to-binary
```

### Launch against a process ID
```
gdb -silent `pidof <binary-name>`
```

### Launch in TUI Mode
```bash
gdb -tui
```

## Commands

### Set breakpoint
```text
(gdb) b main // Breaks at main()
(gdb) break strcpy // Breaks at strcpy()
```

### List defined breakpoints
```text
(gdb) info b
```

### Continue execution
```text
(gdb) c
```

### Step into
```text
(gdb) s
```

### Show stored values
```text
(gdb) print $esp

(gdb) x/5x $esp-10 // in Hex
(gdb) x/5s $esp-10 //String
(gdb) x/5d $esp-10 //Decimal
(gdb) x/5i $esp-10 //Assembly Instructions
```

### Show where in the source file we are
```text
(gdb) list
```

### Show where execution is
```text
(gdb) where
```

### Show symbols
```text
(gdb) info file
```

### Show all defined functions
```text
(gdb) info functions
```

### Show function disassembly
```text
(gdb) disas <func-name>
```
#### Example
```text
(gdb) disas strcpy
Dump of assembler code for function strcpy:
0x42079dd0 <strcpy+0>:  push   %ebp
0x42079dd1 <strcpy+1>:  mov    %esp,%ebp
0x42079dd3 <strcpy+3>:  push   %esi
0x42079dd4 <strcpy+4>:  mov    0x8(%ebp),%esi
0x42079dd7 <strcpy+7>:  mov    0xc(%ebp),%edx
0x42079dda <strcpy+10>: mov    %esi,%eax
0x42079ddc <strcpy+12>: sub    %edx,%eax
0x42079dde <strcpy+14>: lea    0xffffffff(%eax),%ecx
0x42079de1 <strcpy+17>: jmp    0x42079df0 <strcpy+32>
0x42079de3 <strcpy+19>: nop
0x42079de4 <strcpy+20>: nop
0x42079dfb <strcpy+43>: mov    %esi,%eax
0x42079dfd <strcpy+45>: pop    %esi
0x42079dfe <strcpy+46>: pop    %ebp
0x42079dff <strcpy+47>: ret
End of assembler dump.
```

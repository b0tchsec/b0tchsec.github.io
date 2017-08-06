---
layout: post
title: "SHA2017 CTF - Junior : Find The Flag"
author: aagallag
date: 2017-08-06 15:18:00 -0700
slug: findtheflag
category:
- 2017
- sha2017
- sha2017-junior
---
**Category:** Binary

**Points:** 1

**Description:**

```
Find The Flag (1) - 276 solves
There is a flag hidden in this binary. Can you find it?

findtheflag

06b09333154289204c50224c700a456a
```

The flag was simply contained in the binary as a string, no assembly required.

```
$ strings findtheflag 
/lib64/ld-linux-x86-64.so.2
libc.so.6
puts
__libc_start_main
__gmon_start__
GLIBC_2.2.5
UH-x    `
fffff.
[]A\A]A^A_
flag{b760866fa6f035548be127b7525dbb66}
There is a flag hidden in this binary. Can you find it?
;*3$"
GCC: (Debian 4.9.2-10) 4.9.2
GCC: (Debian 4.8.4-1) 4.8.4
.symtab
.strtab
.shstrtab
.interp
.note.ABI-tag
.note.gnu.build-id
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rela.dyn
.rela.plt
.init
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.jcr
.dynamic
.got
.got.plt
.data
.bss
.comment
crtstuff.c
__JCR_LIST__
deregister_tm_clones
register_tm_clones
__do_global_dtors_aux
completed.6661
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
findtheflag.c
__FRAME_END__
__JCR_END__
__init_array_end
_DYNAMIC
__init_array_start
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
_ITM_deregisterTMCloneTable
data_start
puts@@GLIBC_2.2.5
_edata
_fini
__libc_start_main@@GLIBC_2.2.5
__data_start
__gmon_start__
__dso_handle
_IO_stdin_used
__libc_csu_init
_end
_start
__bss_start
main
_Jv_RegisterClasses
__TMC_END__
_ITM_registerTMCloneTable
_init
```

# Flag
**flag{b760866fa6f035548be127b7525dbb66}**

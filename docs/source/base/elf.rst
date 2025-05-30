ELF
#######


overview
==========

`ARM ELF File Format <https://developer.arm.com/documentation/>`_

elf含3类object file: relocatable, executable, shared object

elf基本结构：
- linking view: elf header, program header table(optional), section 1 ... n, section header table
- execution view: elf header, program header table, segment 1 ... n, section header table(optional)

系统预留部分特殊的section (reserved names)，例如.symtab, .strtab。linker/debuger场景，通过section header table访问这些section.

loader场景，通过program header table访问segment，执行程序。

generic elf file layout
==========================

elf format: 
- elf header
- { program header table, text segment (code), data segment (initialized r-w data), bss segment (uninitialized data, 在image初始化/程序启动时置0 }
- { ".symtab" section (用于底层debug), ".strtab" section (用于symbol table的entry的textual names), ".shstrtab" section(所有section的textual names), debug sections (源码相关的debug信息, arm symbolic debugger, dwarf), section header table }

注意：executable 可以不含section相关的内容，只含program segment相关的内容 

scatter-loaded executables
==============================

arm linker 从一个 load region 生成 section header table

section name与load region name相同

如果load region含code/data/uninitialized data，那么会相应生成多个同名section，只是内容不同。

不同section的地址信息在section header table里记录。

note
========

可见，scatter-loaded 如果载入恶意内容，就。。。



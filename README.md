# xorloser's IDA console stuff
(aka PS3 and Xbox360 support for IDA)

Currently made for the Windows version of IDA v7.5 sp3.

Please note that it is recommended to open the original SELF or XEX files directly
without first converting them to ELFs or EXEs. This is because a SELF/XEX contains
extra data not present in the ELF/EXE.

## Install
To install, copy the directory structure inside the accompanying "IDA" folder into your IDA installation directory.

## Plugin Notes

### PlayStation 3 SELF/ELF loader (ps3.dll/ps364.dll)
Loads PS3 SELF files directly into IDA without requiring any preprocessing such as extraction or decryption.
Attempts to utilise as much info as possible when setting up the disasm, automating
various things like table setup, syscalls, module calls etc.

This will also load ELF files however it is recommended to load the original SELF file
for best results as SELF files contain extra data that is not present in the ELF file.

This uses the user editable ps3.xml file for info on NIDs used to name imported and exported
functions and variables.

### SPU processor module (spu.dll/spu64.dll)
This package now contains my SPU processor module (spu.dll and spu64.dll).

If you install this alongside the *official* SPU module then IDA will choose to use
one plugin or the other in a way that seems random. So it is recommended to only
have one SPU module ready for use at one time. The way I do this is to add the ".bak"
suffix to the filename of whichever module I don't want to use currently.
For this reason my bundled SPU processor modules come with the ".bak" suffix already applied.
So if you wish to use them instead of the official module then you should remove this suffix
and then rename "spu.py" to "spu.py.bak".

I have not bothered to release this before since there is already a perfectly good
SPU module bundled with IDA. I wrote my module before this "official SPU" support was added,
and mainly kept it around since I had existing IDBs that relied on it.

### PPC to C (PPC2C.dll/PPC2C64.dll)
This is not a decompiler!
This is a plugin that attempts to make some of the tricky PPC instructions easier to understand.
There are various versions of this plugin to be found on the Internet.
So if you don't already have it, then use this version as a starting point.

This will add comments of C code that represent PPC instructions, such as:

	bc  14, 4*cr7+eq, loc_800037A8 # if(cr7 is equal) goto loc_800037A8
	clrlwi %r0, %r0, 31            # %r0 = %r0 & 1
	rldicr %r10, %r10, 24,39       # %r10 = ((%r10 << 24) | (%r10 >> 40)) & 0xFFFFFFFFFF000000
	rldicl %r4, %r4, 0,48          # %r4 = %r4 & 0xFFFF

An example of some lines you might want to add to plugins.cfg in IDA:

	PPC_To_C:_Current_Line          PPC2C         F10      0 ; convert the current line to C
	PPC_To_C:_Entire_Function       PPC2C         Ctrl-F10 1 ; convert the current function to C


### PPC Helper (PPCHelper.dll/PPCHelper64.dll)
This is a very basic plugin that attempts to rename function args and stack variables so as to make your disasm job that little it easier.
An example of some lines you might want to add to plugins.cfg in IDA:

	PPC_Helper          PPCHelper         F10      0

For an example of what it does, look at the "SPU Helper" example below. It works in a similar fashion.

### SPU Helper (SPUHelper.dll/SPUHelper64.dll)
Only works with my custom SPU module!
This is a very basic plugin that attempts to rename function args and stack variables so as to make your disasm job that little it easier.
An example of some lines you might want to add to plugins.cfg in IDA:

	SPU_Helper          SPUHelper         F10      0

An example of what it does:

Before:
```
	# =============== S U B R O U T I N E =======================================
	func1_before:
	.equ var_50, -0x50
	.equ var_30, -0x30
	.equ var_20, -0x20
	.equ var_10, -0x10
	.equ arg_10,  0x10

	stqd    s0, var_10(sp)
	lr      s0, r3
	stqd    s1, var_20(sp)
	lr      s1, r5
	stqd    s2, var_30(sp)
	lr      s2, r4
	stqd    lr, arg_10(sp)
	stqd    sp, var_50(sp)
```
After:
```
	# =============== S U B R O U T I N E =======================================
	func1_after:
	.equ save_sp, -0x50
	.equ save_s2, -0x30
	.equ save_s1, -0x20
	.equ save_s0, -0x10
	.equ save_lr,  0x10

	arg0 = s0
	arg2 = s1
	arg1 = s2
	ret = r8

	stqd    arg0, save_s0(sp)
	lr      arg0, r3
	stqd    arg2, save_s1(sp)
	lr      arg2, r5
	stqd    arg1, save_s2(sp)
	lr      arg1, r4
	stqd    lr, save_lr(sp)
	stqd    sp, save_sp(sp)
```


### Xbox360 XEX loader (xex.dll/xex64.dll)
Loads Xbox360 XEX files directly into IDA without requiring any preprocessing such as extraction or decryption.
Attempts to utilise as much info as possible when setting up the disasm, automating
various things like table setup, syscalls, module calls etc.

This uses the user editable Xbox360.xml file for info on IDs used to name imported and exported
functions and variables.


## History
In the event that I do any more updates of this package of files, the versions will be based on the date that the package was made available.

### 2021/03/19
* Fixed PS3 symbol handling (hopefully didnt break it in other parts.<br>
  Previously symbol offsets ('st_value's) were added to their section start address if the SELF file was for PPU.<br>
  This has been changed so that symbol offsets ('st_value's) are added to their section start address if the SELF file is a PRX (ie relocatable).<br>

### 2021/03/03
Contains some PS3 fixes after various people starting using them and reported issues back to me.
* Fixed 32bit ELF symbol handling (present in SPU files with symbols)
* Fixed custom handling of SPU ELF files where SELF extra data is missing.
* Fixed support for the official SPU module with the PS3 SELF/ELF loader.
* Added SPU module.
* Added DWARF debug info auto-loading (usually only present in debug builds)
* Added warning and suggestion to use SELF file if loading a badly preconverted ELF file.

### 2021/01/29
The first release in ages, mainly due to a friend asking for builds of these various plugins that would work with IDA v7.5 sp3.

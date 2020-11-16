# Disable BD PROCHOT on MBR bootstrap code

## What is this?

This is the MBR version of the [Disable BD PROCHOT UEFI extension](https://github.com/arter97/DisablePROCHOT)

It is made by modifying Windows's MBR bootstrap code, which is used in Windows 7, 8/8.1 and 10 (therefore, it is compatible with all these versions of Windows).

This mod uses bootstrap code from [here](https://github.com/egormkn/mbr-boot-manager)

## Why?

> ThrottleStop is loaded after the OS has finished booting, which means your entire OS loading is still done extremely slowly.  
>  
> This doesn't mean ThrottleStop is no longer needed.  
>  
> Entering ACPI S3 state(suspend) causes the BD PROCHOT MSR bit getting re-enabled. You need to use some userspace tool for disabling BD PROCHOT for such cases.  
>  
> In case of Windows, use ThrottleStop.  
> In case of macOS, try SimpleMSR

## How to install

**Compatibility**: Windows 7 / 8 / 8.1 / 10

1. Make sure you are using MBR boot mode (or "Legacy" boot mode on modern mainboard BIOS)
2. Download [BOOTICE](https://www.majorgeeks.com/files/details/bootice_64_bit.html) and run as Administrator.
3. Select the disk, then Process MBR -> Restore MBR
4. Choose `boot-windows.bin` as restore file
5. Make sure that "Keep signature and partition table untouched" is selected
6. Choose "Restore" and reboot your computer

## How does it work?

The following code is added into `windows.asm`

```
MOV ECX, 0x1FC
MOV EAX, 0x4005E
XOR EDX, EDX
WRMSR
```

In order to fit the bootstrap code into 446 bytes, The following messages have been changed:

```
LOADING_ERROR_OFFSET: EQU ($ - $$) % 0x100
LOADING_ERROR: DB "Error loading OS", 0     => Original: "Error loading operating system"
MISSING_ERROR_OFFSET: EQU ($ - $$) % 0x100
MISSING_ERROR: DB "Missing OS", 0           => Original: "Missing operating system"
```

Compile the assembly file using `nasm.exe -f bin windows.asm -o boot-windows.bin`
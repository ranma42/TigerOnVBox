## Known issues
The following issues have been identified and are not currently considered
critical, as they can cause some inconvenience or introduce limitations, but
they should not prevent the normal usage of the virtual machine.

Please report:
 * any issues that you encounter that are not listed here;
 * any of these issues that are critical for your use case;
 * any fix you find for these issues.

It is impossible to guarantee that eventually issues will be fixed, but tracking
them makes it easier for everybody to understand what to expect. Thank you in
advance for your help and contributions!

###  → About This Mac shows the processor as "Unknown"
*About This Mac* retrieves the processor type information from the DMI,
specifically from the Apple-specific *OEM Processor Type* SMBIOS table
([definition](https://opensource.apple.com/source/AppleSMBIOS/AppleSMBIOS-42/SMBIOS.h.auto.html)).
VirtualBox does not provide a way to override non-standard SMBIOS values,
therefore it is currently impossible to change the contents of this table.

A possible workaround for those interested only in getting the UI right is to
edit the `UnknownCPUKind` association in
`/System/Library/CoreServices/Resources/English.lproj/AppleSystemInfo.Strings`
(or the appropriate localized file, depending on your language settings).

### The virtual machine will not boot with 4+ GB of memory
The EFI32 boot loader starts successfully with up to 3488 MB of RAM, but
assigning more than that results in a virtual machine that halts in the EFI
environment.

### Changing the virtual machine settings resets the EFI firmware image
When changing some of the virtual machine settings (such as the number of
processors or the amount of memory), VirtualBox resets the firmware setting to
`EFI` instead of `EFI32`. If the virtual machine is configured as 64-bits
(**Version: Mac OS X (64-bit)**), this will prevent it from booting
successfully.

This can be fixed by running
```
VBoxManage modifyvm Tiger --firmware efi32
```

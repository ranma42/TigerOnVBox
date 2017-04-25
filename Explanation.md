## Post-installation tuning
Although not strictly needed, I find it convenient to tune some settings. These
are optional and independent from each other, but they are documented in case
you trip on the same inconveniences as I did.

### EFI32
Mac OS X 10.4 can only boot from a 32-bit EFI. For this reason, if the virtual
machine is configured as **Version: Mac OS X (64-bit)**, its bootloader must be
configured as 32-bits with the command:
```
VBoxManage modifyvm Tiger --firmware efi32
```

### CPU profile and CPUID settings
The kernel is unable to correctly parse some of the CPUID information of modern
CPUs, leading to a hard freeze while loading modules or to a kernel panic in the
kernel initialization. Luckily VirtualBox makes it possible to override most
CPUID information.

In order to understand what happens, some knowledge about the `CPUID`
instruction is required. It returns "CPU identification" (and feature
information) about the processor. The contents of the `EAX` register (and in
some cases of the `ECX` register) selects the leaf (and sub-leaf), i.e. what
information should be returned. The `EAX`, `EBX`, `ECX` and `EDX` registers are
used to store the output.

VirtualBox makes it possible to override the values returned by this instruction
in two ways:
 * by requesting a different processor to be emulated
```
VBoxManage modifyvm Tiger --cpu-profile 'Intel Pentium 4 3.00GHz'
```
 * by overriding a specific leaf
```
VBoxManage modifyvm <VM_Name> --cpuidset <leaf> <eax> <ebx> <ecx> <edx> 
```

The CPUID leaf `EAX=0` returns the maximum value for basic CPUID leaves in `EAX`
and a vendor identification string in `EBX-EDX-ECX`. In some cases (for example
to perform live migration of virtual machines over different processors), it is
desirable to restrict the CPU features as much as possible, but in our case we
are aiming at making them available to the virtual machine. Specifically, in
order to enable multiple processors, we need to enable at least the leaf `EAX=4`
(the vendor identification can be left as-is).
```
# Set CPUID EAX=0 to return 00000004 "Genu" "ntel" "ineI"
VBoxManage modifyvm Tiger --cpuidset 00000000 00000004 756e6547 6c65746e 49656e69
```

The sub-leaves of `EAX=4` contain cache information which the kernel has trouble
parsing and unfortunately there seem to be no way to override a sub-leaf in
VirtualBox without overriding the whole CPU profile. Among the profiles
currently shipped in VirtualBox, Intel Pentium 4 3.00GHz is the only one that
seem to boot fine with the leaf limit set to 4.

Setting the CPU profile to an old Pentium 4 will likely prevent VirtualBox from
exposing several of the features available on your CPU. The overrides for
`EAX=1` and `EAX=0x80000001` are used to re-enable any feature supported by your
CPU. Actually, the overrides will enable even features that are not supported by
the host CPU, but VirtualBox takes care of filtering them out.
```
# Set CPUID EAX=1 to return the Pentium 4 version/brand information
# but override the features, enabling anything except XSAVE
VBoxManage modifyvm Tiger --cpuidset 00000001 00000f43 00020800 fbffffff ffffffff
# Set CPUID EAX=0x80000001 to return any feature as enables
VBoxManage modifyvm Tiger --cpuidset 80000001 00000000 00000000 ffffffff ffffffff
```
Note that `XSAVE` (bit 26 of `ECX` for `EAX=1`) is not enabled. This is required
because VirtualBox will complain at boot if that bit is enabled on a processor
without leaf `EAX=0xd`. These feature overrides are very aggressive and other
features might have to be disabled just like `XSAVE` in future.

### DMI data
The kernel and most of the applications do not seem to check DMI data, with the
exception of *System Profiler*. With the default setting, it cannot show the
basic hardware information, apparently because it tries to parse the BIOS
version in a specific way. Namely, it tokenizes it using `.` as a separator and
shows the first, third and fourth tokens. Therefore, in order to get the
*Hardware* page working, the BIOS version is overridden with:
```
VBoxManage setextradata Tiger VBoxInternal/Devices/efi/0/Config/DmiBIOSVersion EFI32..Virtual.Box
```
In general, it would be possible to override it with
```
VBoxManage setextradata Tiger VBoxInternal/Devices/efi/0/Config/DmiBIOSVersion AAA.BBB.CCC.DDD.EEE
```
This results in the *System Profiler → Hardware* page showing "Boot ROM Version:
AAA.CCC.DDD"

Additionally, the virtual machine is set up to use the default DMI data from
VirtualBox instead of propagating the host data:
```
VBoxManage setextradata Tiger VBoxInternal/Devices/efi/0/Config/DmiUseHostInfo 0
```
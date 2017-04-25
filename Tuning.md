## Post-installation tuning
Although not strictly needed, I find it convenient to tune some settings. These
are optional and independent from each other, but they are documented in case
you trip on the same inconveniences as I did.

### Remove the optical drive
Apart from the initial installation, the optical drive is likely not going to be
needed. It looks like its presence contributes to the chance of a slow boot when
enumerating the devices, therefore I strongly suggest to remove it.

To remove it, go to *Settings... → Storage*, select the optical drive and then
*Remove Attachment*.

In case you need it again, go to *Settings... → Storage*, select the controller
(*Controller SATA*) and then *Add Optical Drive*.

### Change the display resolution
The lack of guest additions makes it harder to change the resolution of the
display in an arbitrary way, but there is a simple method to choose between some
predefined settings. From the command-line, run:

```
VBoxManage setextradata Tiger VBoxInternal2/EfiGopMode n
```

where `n` is one of 0-5, respectively representing:

0. 640x480
1. 800x600
2. 1024x768
3. 1280x1024
4. 1440x900
5. 1920x1200

### Change the host key
VirtualBox captures the **host key** and instead of passing it to the virtual
machine, it uses it to detach the keyboard from the guest OS and as a key
modifier for the VirtualBox operations.

The left command key (left-⌘) is the default host key. I find it more convenient
to set the host key to the right command key, as I normally use the left one in
the host OS (to copy & paste, open *Spotlight* and so on) and being able to use
the same key combinations in the guest OS makes switching between the two
environments feel much more seamless.

### Change system settings
Most system settings can be changed at any time, either before or after the
installation process, including:
 * the number of processors
 * the amount of memory
 * whether the system is 32-bits or 64-bits

Remember that binaries built for x86_64 will not be able to run when the virtual
machine is configured as **Version: Mac OS X (32-bit)**. Instead, a virtual
machine configured as **Version: Mac OS X (64-bit)** will be able to run both
i386 and x86_64 binaries.

**WARNING:** when changing some of the virtual machine settings, VirtualBox
resets the firmware setting to `EFI` instead of `EFI32`. If the virtual machine
is configured as 64-bits (**Version: Mac OS X (64-bit)**), this will prevent it
from booting successfully. To fix this, run:
```
VBoxManage modifyvm Tiger --firmware efi32
```

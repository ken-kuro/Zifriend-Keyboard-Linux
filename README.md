# Zifriend-Keyboard-Linux
Workarounds for Zifriend Keyboards on Linux

## TLDR:
1. Find your VendorID and ProductID of your keyboard in Windows using Device Manager
2. Add `usbcore.quirks=VendorID:ProductID:flags` to kernel parameters, I recommend trying `gki` for the flags
3. Reboot your system, and with the new kernel parameters applied, the keyboard should start working, and you should be able to find it listed in `lsusb`

## Details:
The Zifriend Keyboards (also known as SAMA, Gamestop, Cyberlinx,... as they're rebranded versions), such as the ZA68/ZA68pro, ZA63/ZA63pro, and others, are known to have issues working with Linux, you can easily find online posts discussing this problem, for example: [Here're some Reddit posts on the topic](https://www.reddit.com/search/?q=zifriend+linux&type=link). The manafacturer's manuals also state that only MacOS and Windows are officialy supported, [here's the link to the driver and manual page](https://www.zifriend.net/pages/drivers-manuals).

I recently purchased a ZA63 Pro keyboard. It's not perfect, but for the price, it seems like a good choice. It boasts a complete layout with three connection modes, hot-swappable pre-lubed switches, per-key RGB lighting, side RGB lighting, and n-key rollover – all for just $15! However, there's a problem.

The seller advertised Linux support, and I didn't think to double-check since all my previous keyboards, from mechanical to basic USB models, worked flawlessly with Linux. But after installing Arch Linux, the ZA63 Pro wouldn't work in wired mode. Initially, I suspected a kernel issue. I completed the installation, upgraded everything, but the keyboard remained unresponsive. It wasn't listed in lsusb or libinput list-devices. The dmesg output indicated errors:

```
[  341.170852] usb 1-4: new full-speed USB device number 12 using xhci_hcd
[  346.541131] usb 1-4: unable to read config index 0 descriptor/start: -71
[  346.541138] usb 1-4: can't read configurations, error -71
[  346.663850] usb 1-4: new full-speed USB device number 13 using xhci_hcd
[  352.086587] usb 1-4: unable to read config index 0 descriptor/start: -71
[  352.086594] usb 1-4: can't read configurations, error -71
[  352.088555] usb usb1-port4: attempt power cycle
```

Interestingly, the keyboard functioned perfectly in Windows, BIOS, GRUB, and even Linux itself in wireless (2.4Ghz or BT) mode. This was frustrating because the wired mode is necessary to bypass the keyboard's sleep function.

Trying Fedora yielded the same error. Thankfully, I stumbled upon information about [usbcore.quirks](https://github.com/torvalds/linux/blob/master/drivers/usb/core/quirks.c), you can learn more about it [here](https://www.kernel.org/doc/html/v5.0/admin-guide/kernel-parameters.html). Further investigation revealed similar issues with other Chinese keyboards and some Corsair (CKB-next) models. The solution involved specifying a `usbcore.quirks` flag related to power management, device qualifier, etc. Experimentation might be needed to find the optimal flag for your specific keyboard. In my case, the common fix `usbcore.quirks=VendorID:ProductID:gki` worked.

However, there's a catch: how do you obtain the VendorID and ProductID when the device isn't recognized by the system and doesn't show up in lsusb or libinput list-devices? As a Linux enthusiast, this might sound strange, but here's where Windows comes in handy (yes, I also happen to be a [gamer](https://discord.com/users/581475300777394190) – a god-level one, of course 😏). With a dual-boot setup, I simply booted into Windows, plugged in the keyboard, and went to Device Manager > Keyboard. There, I found the device details, including the Hardware IDs (VendorID and ProductID). You can find more information on this process [here](https://kb.synology.com/en-in/DSM/tutorial/How_do_I_check_the_PID_VID_of_my_USB_device). In my case, the ZA63Pro's IDs were `5566:0008`

If your device is showing up as **HID Device** without specifying a keyboard name in Windows Device Manager. Then it will also not display the VID and PID in the details section. To get the correct VID and PID, run this command on **Windows powershell** from any windows manchine when the keyboard is plugeed in.

```
Get-PnpDevice -PresentOnly | Where-Object { $_.InstanceId -match ‘^USB’ } | Format-Table -Wrap -AutoSize
```

And from the list discard the devices which has `VID 3151` and `PID 3020`. They are the windows default. Look for the **HID device** with other VID and PID. Also looks like ZiFriend, uses same VID and PID for all variant of keyboards. Even ZA68 also shares the same VID : 5566 and PID : 0008

The solution is to add `usbcore.quirks=5566:0008:gki`  (replacing the values with your specific VendorID, ProductID, and potentially the flag) to the [kernel parameters](https://wiki.archlinux.org/title/kernel_parameters). Since I use GRUB, I edited /etc/default/grub, added the parameter to GRUB_CMDLINE_LINUX_DEFAULT

```bash
sudo nano /etc/default/grub

GRUB_CMDLINE_LINUX_DEFAULT="usbcore.quirks=YOUR_VID:YOUR_PID:gki"

```

regenerated grub.cfg

```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

and voila! My keyboard now works perfectly in wired mode.


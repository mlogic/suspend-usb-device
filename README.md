# suspend-usb-device
An easy-to-use script to properly put an USB device into suspend mode that can then be unplugged safely

suspend-usb-device depends upon the `sdparm` package.

Safely remove an USB hard drive in GNU/Linux

[Modified from the blog](https://elliotli.blogspot.com/2009/01/safely-remove-usb-hard-drive-in-linux.html)

I bought a new USB hard drive, Western Digital My Book® Essential Edition™ 1TB, and use it with GNU/Linux. Here I'm discussing about how to safely remove (then disconnect) it from a GNU/Linux system.

The problem: according to its user manual, you should disconnect 
the drive safely when it is not active, but in a GNU/Linux system how to do 
this is not very straightforward. You may notice that after you unmount 
it (whether through command-line or a desktop environment), the drive is
 still spinning and it's LED on. If you read it's user manual carefully 
you'll find that the manufacturer never said it's safe to disconnect it 
from your system in such condition.

The quick solution to this problem:

* send SYNC, then STOP command to the device, this can be done easily in GNU/Linux by unbinding the device
* suspend the USB port by echoing a "suspend" to the 
  "/sys/bus/usb/devices/$DEVICE/power/level", where $DEVICE corresponding 
  to the device of your USB device.

To put it simple, I wrote a script to do this for you. Download and try it. For developers, you can get it from the github repo.

If you saw an error message like this when running it:

```
bash: /sys/bus/usb/devices/5-8/power/level: No such file or directory
```

that means your kernel doesn't support USB suspend. You can just 
disconnect your device now, it's already safer enough to unplug it.

To those who is interested, discussion of technical detail following.

From the hardware aspect, the drive should only be removed when the following requirements are met:

* there's no pending I/O request and software cache flushed
* it's hardware cache flushed
* it's driver spins down (means "not spinning")
* (optional) the USB port is put into "suspend" mode

The user's manual read (page 6, section "Turning Off/Disconnecting the Device"): "(for Windows systems): Right-click the Safely Remove Hardware icon in your system tray and select Safely Remove Hardware. You may hear the drive power down before the Power LED turns off.
 The drive is now shut down properly, and you may disconnect the drive 
safely. (for Macintosh systems) Drag the My Book icon to the Trash icon 
for proper dismount. You may hear the drive power down as the Power LED flashes. When the Power LED is steady, you may press the Power button once or disconnect the drive’s power cord to turn off the drive safely."
 (there's a mistake, this device doesn't have a Power button) If you 
read carefully you can tell that Windows and Macintosh don't do the same
 things when told to remove an USB device. As the bold text as shown, in
 a Windows system it will be turned off while in a Macintosh system you hear the drive power down but the Power LED would finally become steady
 rather than off. Therefore we know that Windows does all 4 steps and 
Macintosh do the first 3 steps only when told to remove a USB device. 
(BTW, it's funny that you have to use your ear before disconnecting a 
device.)

(This is only the case for Windows XP. For Windows Vista seems only the 
first 3 steps are done thus the drive remained active after you removing
 it from the system.)

Some may argue that they always disconnect their drives directly after 
unmounting (step 1 only) and never experience any damage, but as an 
operating system developer and a serious user, I always follow the 
manufacturer's manual or specification whenever it make sense and 
possible because I know electronic devices are fragile, rigid and obtuse
 if you don't use them as the way they are designed. For a well-designed
 and robust piece of hardware, theoretically you can disconnect the 
device safely when the first two steps are done. The firmware on the 
device would notice the USB cable was unplugged and shut down the disk 
drive properly. If not, the drive would be stopped sharply as if you 
pulled it's power when it's running and this damages the hardware 
gradually. As an outsider of the manufacturer, we hardly know whether 
it's well-designed or not so the only safe principle is to follow it's 
manual, means we should not disconnect the device until at least all 
three steps are done.

At the Linux kernel part. To finish all the 4 steps, you have to unmount
 the device, unbind the device from the driver, then tell the USB core 
driver to put that device into suspend mode, as shown in the solution 
above. And you have to be running a kernel with CONFIG_USB_SUSPEND 
enabled.

CONFIG_USB_SUSPEND is not enabled by default in a vanilla kernel. I'm pushing it in this discussion.

Properly removing a device from a running GNU/Linux should be done by either
 the HAL or desktop manager. If you can, please push the developers you 
know working on any DM that missing this feature to implement this 
function properly.

Try my script, and write me if you found a problem. Welcome patches.

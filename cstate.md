# 0. Introduction/Setup.
This tutorial is out of the ordinary because it relates to a very specific niche issue I was having with my Debian 12, and Debian 13 (when I upgraded it) system. In short, my system would randomly freeze. It was unresponsive and had to be shut down forcefully with the power button as it never recovered from the freeze on its own. This also seemed to happen on a Debian 12 home server I was running, which was headless and didn't have GPU drivers. This allowed me to rule out GPU driver issues which is what I had initially suspected. After some research and trial by elimination, I figured the issue was probably to do with c-states. Processor c-states are different states one's CPU can go in and out of, basically the higher the state the more "asleep" your CPU is. This feature exists to save power and likely to extend the processor's lifespan. However, Debian, at least systems I installed which had early generation AMD Ryzen processors, seems to have some sort of trouble managing the c-state resulting in my computer freezing up. I ended up spending countless hours on this issue but finally was able to figure out how to prevent my system from going into higher c-states. I turned off c-state management in my BIOS but my system overrode that and did it anyway much to my dismay. So, I scoured the internet and eventually found the arcane knowledge of how to do it. I now share this information publicly for potentially someone else having the issue, or more likely I'm putting this here in case I need to do this again.

# 1. Set The Max C-State.
Edit grub's configuration file. I recommend nano but any text editor will do. Grub is the boot loader, and we can pass through it the flag needed to limit the C-State. The boot loader loads the Kernel, which is the main program your OS consists of. The Kernel is also presumably what decides to manage C-States anyway despite me disabling that in the BIOS. Luckily, this is Linux so the user gets to tell the Kernel what to do. As long as said user has root privileges.

> sudo nano /etc/default/grub

Add `intel_idle.max_cstate=1` inside of the `GRUB_CMDLINE_LINUX_DEFAULT` setting. Make sure it's inside the quotes, and there is a space between it and whatever was there to start with, if anything.

Now, run the following command to apply the new grub configuration.

> sudo update-grub

Finally, reboot the system using either the following command or whatever power menu your desktop environment gives you if you have one.

> sudo reboot

Verify that the changes actually worked with the following command.

> cat /sys/module/intel_idle/parameters/max_cstate

Now the pain has finally gone away.

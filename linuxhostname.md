# 0. Introduction/Setup.
This short tutorial is about changing your Linux computer's hostname. To my knowledge, this will work on pretty much any Linux distribution. I tested it on Debian 13 only, so no guarantees. For context, your computer's hostname is something other than an IP address that your computer uses to identify itself on a network to other computers. Of course, an IP/MAC address will still be used to communicate with your computer, but the hostname will be mapped to said IP address. So basically the hostname is primarily for humans, as it's far eaiser for most people to remember than an IP address. The hostname will also appear in your Linux terminal, right after your user's name and the @ sign.

# 1. Changing the hostname.
Your hostname will simply be stored in the following file. Remember that whatever you choose must be 1 to 63 characters, only lowercase letters, numbers, and dashes, and it can't start with a dash. For the purposes of this tutorial, I will choose `newhostname`. Run the following command:

> nano /etc/hostname

The `/etc/hostname` file should now include your hostname and nothing but your hostname. We aren't done, however. We now need to edit another file, which is used to map IP addresses to hostnames. Essentially we need to tell the computer that our new hostname belongs to the computer. Run the following command:

> nano /etc/hosts

Add the following line, replacing `newhostname` with whatever you chose as your new hostname:

```
127.0.0.1   newhostname
```

Now you should reboot so that your changes take effect.

> sudo reboot

Finally, you may open a terminal window to verify that the changes were successful. You should see something like `yourusername@newhostname:~$ ` with the hostname being whatever you just set it to.

# 0. Introduction/Setup.
This tutorial explains how to perform a Man-In-The-Middle (MITM) attack using a technique called ARP spoofing. We will act as the attacker, positioning ourselves in between two machines and spying on all traffic between them. While this tutorial is only concerned with eavesdropping, the attacker may also perform other attacks given the ability to redirect traffic, drop traffic, and send arbitrary spoofed traffic to either machine. ARP spoofing basically involves advertising to both victims using ARP messages that the attacker machine is, in fact, the other respective victim machine. Of course, it's a bit more complicated than that but for the purposes of this tutorial I assume that you know what ARP is and how it works. If not, plenty of information can be found about it online.

Note that performing this attack against machines/networks that you don't own or have permission to attack is illegal in most juristictions. I do not condone any illegal or unethical activities performed using knowledge from this tutorial. This tutorial is for educational purposes only. Be sure to perform the attack only in a home lab setting or in a lab setting where you have explicit permission to do it.

I used a Kali Linux virtual machine on my home network. I went into the VirtualBox network settings for the machine and changed it from "NAT" to "Bridged Adapter" mode. This allows it to essentially function as another machine connected to your network. This can be dangerous, if your virtual machine or machines on your network are compromised they will be able to potentially spread malware and/or attack one another. I had my home router and another machine on the network act as the victim machines. Overall this makes for a pretty realistic scenario.

Other, more secure setups are absolutely possible and recommended. For instance, one could set up two victim virtual machines along with the Kali VM and set all of their network adapters to "Internal Network" and give their internal networks the same name. This would isolate all 3 VMs from everything else, while allowing them to communicate with each other. One would then have to either assign static IPs to the machines, or one of the machines would need to be a DHCP enabled router. Assigning static IPs is probably quicker and easier.

# 1. Enable IPv4 and IPv6 forwarding.
We need to allow our attacker machine to perform the same role as a router or switch. That is, the attacker machine needs to be able to forward network traffic on behalf of other machines. By default, computers are configured to not do this, which is a problem. The following commands temporarily enable forwarding for both IPv4 and IPv6. The IPv6 part is important as it is often overlooked, and it's pretty common for some traffic on a local network to use IPv6. This configuration will not survive a reboot. If you want these changes to be persistent, they have to be written to a configuration file which will vary depending on which Linux distribution you use. The following commands worked for me on Kali Linux, and will likely work on most distributions.

>sudo sysctl -w net.ipv4.ip_forward=1
>sudo sysctl -w net.ipv6.conf.all.forwarding=1

# 2. Get IPs of victim machine and router.
For the attack, all we need now are the IP addresses of the victim machines. These can be obtained by running the following command which displays the ARP information that the attacker machine and most computers store. You will see IP addresses along with MAC addresses for each machine, and probably hostnames as well. We only care about the IP addresses.

>arp -a

Just to be safe, run the following command which displays some network information about the attacker machine. In particular, take note of the interface name. You should see an interface named "lo" which is the loopback interface and can be ignored. Look for another active interface, one that has an IP assigned to it. This will probably be named "eth0" if your setup is similar to mine. If not, take note of that interface name and replace "eth0" with whatever it is in the next step.

>ip addr

# 3. Perform the spoofing.
We will now run a tool which constantly sends ARP responses in order to trick victim machines into thinking that the attacker machine is the other victim. We need to open two terminal windows, and continuously run two instances of the tool in order to send these messages to both machines, respectively. In the following commands replace "victimip" and "routerip" with the appropriate IP addresses for your setup.

>sudo arpspoof -i eth0 -t victimip routerip
>sudo arpspoof -i eth0 -t routerip victimip

Leave these terminal windows running. If these commands fail because the tool isn't installed, run the following command to install it and then try again.

>sudo apt-get install dsniff

If your distro uses a package manager other than apt, you'll have to research and figure out what the name of the package is on yours because that can vary between package managers. I highly recommend just using Kali Linux since this tool comes preinstalled.

# 4. Open wireshark to eavesdrop on communications between the two machines.
Now you should be able to eavesdrop on traffic between the two victim machines. Open wireshark, a packet sniffing tool, in order to verify this. If it isn't installed, a quick internet search will reveal how to install it. Within wireshark, double click on the interface from earlier. You should now be able to see all of the traffic going between the two machines. Success!

# 0. Introduction/Setup.
Hardening is an effective and standard technique used to secure devices, especially servers. This tutorial is aimed primarily at desktops and home servers, but can be applied to production servers as well. While there are various ways to go about hardening a server, this tutorial specifically focuses on using firewall rules to limit access to network services running on the server. We also limit what external network services can be accessed by programs running on the server. The principle of least privilege will be applied, meaning we will be allowing only what is needed for the server to function. The following packages are required and installable on Debian/Ubuntu based systems with the following command:

> sudo apt-get install iptables iptables-persistent

On other Linux distributions, other firewall programs are the standard. For example, on Fedora/Red Hat distros you would likely want to use firewalld instead. This tutorial pertains to Debian-based distributions, however, where iptables is the standard.

iptables is a relatively simple rule-based firewall program that essentially ties in to the Linux kernel's networking in order to filter network traffic. iptables has some state management capabilities which we will be using later on in the tutorial. I.E. iptables keeps track of ongoing network connections and is aware of when a connection is brand new, incoming, outgoing, etc. It is also possible to use a program called ufw (uncomplicated firewall) which is essentially just a wrapper around iptables. I don't personally find iptables to be particularly complicated so I prefer to cut out the middleman and just use iptables.

# 1. Set up IPv4 firewall rules.
Either physically log into the server or log into the server via SSH. Whatever the case may be, be logged into the server either as root or as an account with superuser privileges (sudo). If you have a desktop environment (not recommended for a server), open your terminal emulator. Otherwise, you should already be in the command line in one form or another.

iptables stores firewall rules in "chains." By default, there are 3 chains. `INPUT`, `FORWARD`, and `OUTPUT`. `INPUT` rules are applied to incoming packets, `OUTPUT` to outgoing packets. `FORWARD` is for packets not destined for our machine, but that our machine is expected to send elsewhere. `FORWARD` is for routers or machines supposed to behave like routers so for desktops/servers it's usually safe to totally block. This is achieved with the following command:

> sudo iptables -P FORWARD DROP

The `-P` flag means "policy" and sets the default behavior for the chain. In this case, we are dropping all traffic that is supposed to be forwarded. Also the `sudo` can be omitted in this and every command if you are logged in as the root user.

If you are SSHing into a server, you will want to create a rule allowing SSH. If you don't do this before blocking incoming traffic by default, you will find yourself unable to access the machine except physically. This is achieved with the following commands:

> sudo iptables -A INPUT -p tcp --dport=22 -j ACCEPT

> sudo iptables -A OUTPUT -p tcp --sport=22 -m --state ESTABLISHED,RELATED -j ACCEPT

The `-A` flag means "append" so we are adding a new rule to a specified chain. The `-p` flag means protocol, in this case being TCP as SSH runs over TCP. The port flags are destination and source port, respectively. The `-j` flag specifies what should be done when packets matching this rule are encountered.

Note that outgoing connections from the SSH port will only be allowed if they are part of an existing connection already. So an external client must start the connection before any outgoing traffic can be sent. That policy is what is specified with the `--state ESTABLISHED,RELATED` flag and is iptables' somewhat simple stateful packet capability.

Now, one will want to block all incoming traffic by default:

> sudo iptables -P INPUT DROP

If you want to run a network application which is only accessible from within one's own machine, run the following commands. I've had no issues not having this on servers, but on a desktop you likely want these rules.

> sudo iptables -A INPUT -i lo -j ACCEPT

> sudo iptables -A OUTPUT -o lo -j ACCEPT

These commands apply specifically to the "loopback" (lo) interface. The `-i` and `-o` flags specify an input and output interface, respectively. This can be used to set complicated policies for routers, allowing traffic from only certain interfaces. In this case we are just using this to allow internal traffic, network traffic from one process on a host to another.

## A. Lax firewall setup (mainly for desktops, less secure).
The only thing that needs to be changed here is allowing incoming packets that are part of an existing outgoing connection. This means that the host can start all the outgoing connections it wants, but will not accept unsolicited incoming packets. So, network services hosted on the machine will be unreachable from anywhere but the machine itself (if you applied the previous rule). This is ideal for most desktop users.

> sudo iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT

## B. Strict firewall setup (mainly for servers, more secure).
For servers, simply disallow all outgoing traffic as well.

> sudo iptables -P OUTPUT DROP

Now, we are dropping all incoming and outgoing traffic by default. Anything which you want your server to be able to host or access must have a rule set up to explicitly allow it. I will cover some common examples here, but one must tailor one's rules to their own server and allow only what is absolutely needed.

### Allow incoming HTTP connections (optional).
This is for a server hosting an HTTP server.

> sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

> sudo iptables -A OUTPUT -p tcp --sport 80 -m state --state ESTABLISHED,RELATED -j ACCEPT

For HTTPS, run the same commands but with 443 as the port number. For any protocol or service running over TCP, the above commands are a good reference and one need only change the port number.

### Allow incoming DNS queries (optional).
This is for a server which hosts a DNS server.

> sudo iptables -A INPUT -p udp --dport 53 -j ACCEPT

> sudo iptables -A OUTPUT -p udp --sport 53 -m --state ESTABLISHED,RELATED -j ACCEPT

Once again, this allows DNS queries to be answered by the server. Replace the port number to allow any other service that runs over UDP.

### Allow pings (optional).
The following commands will allow one to ping the server (e.g. using the ping command). Some administrators may want to allow this so that they can easily see whether a server is online or not.

> sudo iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

> sudo iptables -A OUTPUT -p icmp --icmp-type echo-reply -m --state ESTABLISHED,RELATED -j ACCEPT

### Allow package updates/upgrades (optional).
One may notice that the server is no longer able to update or upgrade with apt (or anything else). On many real production servers, this wouldn't be a problem as the server would never update but rather be imaged with a hopefully thoroughly tested updated server image. For home servers, it's easiest to simply use a package manager to apply updates as usual. This can be allowed with the following commands:

> sudo iptables -A INPUT -p udp --sport 53 -m --state ESTABLISHED,RELATED -j ACCEPT

> sudo iptables -A OUTPUT -p udp --dport 53 -j ACCEPT

> sudo iptables -A INPUT -p tcp --sport 80 -m --state ESTABLISHED,RELATED -j ACCEPT

> sudo iptables -A OUTPUT -p tcp --dport 80 -j ACCEPT

> sudo iptables -A INPUT -p tcp --sport 443 -m --state ESTABLISHED,RELATED -j ACCEPT

> sudo iptables -A OUTPUT -p tcp --dport 443 -j ACCEPT

These rules are the reverse of the previous rules. They allow the server to make an outgoing connection to a DNS, HTTP, and HTTPS server respectively. This is needed in order to update one's packages. Note that any malware which makes a reverse connection to a server of any of these types would now work on your server. Therefore there is a serious convenience versus security tradeoff with these particular rules.

# 2. Set up IPv6 firewall rules.
IPv6 firewall rules can be configured in exactly the same manner as IPv4 firewall rules. The only difference here is that the command is `ip6tables` and not `iptables`. With IPv6, there are more complicated issues depending on how one's IPv6 network is set up. These issues are outside of the scope of this tutorial. I will simply present a way to leave IPv6 enabled, and a way to disable (by blocking) it. If you are not using IPv6, it is best to block it. If you are running a desktop and/or are unsure if you're using it or not, then leaving it enabled may be best.

## IPv6 enabled (lax).
These rules are suitable for desktops only.

> sudo ip6tables -P INPUT DROP

> sudo ip6tables -P FORWARD DROP

> sudo ip6tables -P OUTPUT ACCEPT

> sudo ip6tables -A INPUT -m --state RELATED,ESTABLISHED -j ACCEPT

> sudo ip6tables -A INPUT -i lo -j ACCEPT

> sudo ip6tables -A OUTPUT -o lo -j ACCEPT

## IPv6 disabled (strict).
> sudo ip6tables -P INPUT DROP

> sudo ip6tables -P FORWARD DROP

> sudo ip6tables -P OUTPUT DROP

# 3. Save firewall rules.
As of current, all of your firewall rules won't persist after a reboot. One can make them permanent with the following command:

> sudo iptables-save

For the IPv6 rules:

> sudo ip6tables-save

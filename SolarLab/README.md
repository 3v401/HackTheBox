CrackMapExec (CME) is a tool used to automate tasks related to penetration testing within Windows network environments such as Active Directory.
It can interact with different network services such as SMB (Server Message Block), RDP (Remote Desktop Protocol),
WINRM (Windows Remote Management), and LDAP (Lightweight Directory Access Protocol). Some of its main applications are:

1. `Network Reconnaissance`: Enumerating network shares, users, groups, and policies within a Windows domain.
2. `Credential Testing`: Validating credentials and finding where they can be used within the network.
3. `Lateral Movement`: Executing commands on remote systems to move laterally within a network.
4. `Information Gathering`: Gathering detailed information about Active Directory objects and configurations.
5. `Post-Exploitation`: After gaining initial access, CME can help in maintaining access, further exploiting systems, and gathering data.

Observe that the open ports found with nmap where 80, 135, 139 and 445). Ports 139 and 445 are SMB ports. So it is a good way to start using CME to find SMB shares that allow applications to read and write files and request services from server programs in the computer network.
SMB can be a rich source of information if it is misconfigured, especially if shares are accessible to guest or unauthenticated users.
Also access to SMB shares might provide a foothold into the system, allowing you to upload or download files. This can be used to plant further exploits or gather more data for privilege escalation.
To install type `sudo apt install crackmapexec`.

So, we don't have a username and password initially. A good way would be to try two attempts, one with username empty, and another with username "guest". If without username and password no results are obtained, it means anonymous access is not permitted on the target SMB servers. In that case, you would need some authentication/credentials to proceed further. `guest` is a usual keyword in Windows operating systems. It is designed to provide temporary and restricted access to the system without the need for a password. Many Windows systems have the guest account enabled by default, though with limited permissions. Other keywords like `administrator`, `system`, or `user` are usually secured with strong passwords due to security reasons. An annonymous attempt didn't show successful results, so a `guest` account was run. Type:

```
crackmapexec smb solarlab.htb -u Guest -p "" --shares
```

1. `crackmapexec` calls the tool.
2. `smb` specifies the target service to test (SMB), a network file sharing protocol commonly used in Windows environments
3. `solarlab.htb` Target host to scan
4. `-u` user flag (guest)
5. `-p` password flag (empty)
6. `--shares` enumerate and display the shares available on the target SMB server

This attempt succeeded in enumerating shares. The guest account, which often has limited access rights, was allowed to log in and read the share information.

(pic4)

-----------------------------------

When downloading the files from the smb connection execute:

sudo apt-get install libreoffice --fix-missing

(I had many errors without --fix-missing)

Username: BlakeB
Password: Thiscan...

I chose this one because both Alexander and Claudia have the same structure in the "signa" accounts.


Once logged in, click on Leave Request and fill the data.
I tried a XSS attack like in Healess with no success.
So I searched on Google: `ReportHub vulnerability Github`. On the second link you will find the following github publication:
https://github.com/c53elyas/CVE-2023-33733


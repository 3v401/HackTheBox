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

When downloading the files from the smb connection I tried to open them with libreoffice. Nonetheless I wasn't able so I opened them with an online tool for xlsx (Search on Google "Open xlsx/docx file online" and pick the option you like the most). You will encounter that there is no meaningful information in the docx, only in the xlsx:

(pic7)

Username and password of the three members of SolarLab (Alexander, Claudia and Blake) for the sited Fidelity, Signa and unknown.

(explain how did you reason to find report.solarlab.htb_6791)

Once you enter to `report.solarlab.htb:6791/login` you will access a login site. We know there must be an account for Alexander, Claudia and Blake. We start with Blake introducing `blake.byte` as username and `ThisCanB3...` as password. We get back `User not found.`. We try now with `AlexanderK`and `danenaci...` and we get `User authentication error`. Doing the same with ClaudiaS we get "User authentication error". So from here we infere that AlexanderK and ClaudiaS must exist as users, but we don't know their password (and doing a brutte force attack without hints/masks to know their passwords would take O(k^n) complexity).

The best option would be to attack Blake since there is a chance that the password of that excel is the one for Blake in the solarlab report. Let's try that password with the same structure as the other users, i.e., First Name + First Letter of Surname. Therefore user: `BlakeB` password: `ThisCanB3...`.

(pic10)

Bingo! We got inside. This is a more "thoughtful path" to get inside. Nonetheless, in real life it is not commong to be like that. So we will use a more realistic scenario (to get inside)

#### Alternative getting inside report.solarlab.htb

Our goal is to find a username for Blake using the password from the excel. A good way to perform a brutte force login attack is with [Hydra](https://www.kali.org/tools/hydra/). 

Hydra is a network login cracker. It is designed to perform brute-force attacks to guess passwords/usernames and gain unauthorized access to systems and services. Some of its key features are:

1. `Multi-protocol support`: HTTP, HTTPS, FTP, SSH, SMTP, SNMP, Telnet and more.
2. `Parallel attack capability`: Hydra can launch multiple parallel attacks using multiple threads speeding up the brute-force process
3. `Customizable`: Users can specify custom login forms, failure conditions and other parameters.

The command that we are going to use is:dra -L

```
sudo hyDRA -L user2.txt -p ThisCanB3typedeasily1@ 10.10.11.16 -s 6791 http-post-form "/login:username=^USER^&PASSWORD=^pass^:F=User not found." -t64
```

Where does this command come from? Well, to prepare it I read the documentation of the following github [link](https://github.com/gnebbia/hydra_notes) that explains some key features of Hydra and how to use it, and the second [link2](https://infinitelogins.com/2020/02/22/how-to-brute-force-websites-using-hydra/) that explained how to develop the command for your specific site. Because Hydra depends on which site you are at, remember, it is very `customizable`.


I chose this one because both Alexander and Claudia have the same structure in the "signa" accounts.


Once logged in, click on Leave Request and fill the data.
I tried a XSS attack like in Healess with no success.
So I searched on Google: `ReportHub vulnerability Github`. On the second link you will find the following github publication:
https://github.com/c53elyas/CVE-2023-33733


To obtain the list of open ports run the following command:

```
ports=$(nmap -p- --min-rate=1000 -T3 10.10.11.4 | grep '^[0-9]' | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
```

You will obtain the following list of commands

(pic1)

If you run `nmap -p$ports -sV 10.10.11.4` after the previous port retrieve, nmap will return a negative response appearing to be `10.10.11.4` down or blocking ICMP (ping) probes. When `nmap` doesn't receive a response to its ping probes, it assumes the host is down and doesn't proceed with the port scan.

Administrators often configure servers to ignore or block ICMP (ping) requests as a security measure to make the server less detectable and harder to probe by potential attackers. This technique is known as `stealth` or `cloaking` the server. This is done to reduce exposure, prevent reconnaissance, or mitigate DDOs attacks.

To analyse even when the server is in `stealth` mode run:

```
nmap -p$ports -sV -Pn 10.10.11.4
```

(pic2)

From this scan, it can be inferred that the host is a Windows machine that runs various network services, including DNS, Kerberos (authentication), LDAP (directory services), and RPC (remote procedure calls for interprocess communication), among others. The presence of Active Directory services and multiple RPC services suggests the machine is a server within a Windows domain environment, likely performing several roles related to network and user management. The variety of open ports and services could indicate a well-utilized server, possibly used in a corporate or enterprise setting.

Nonetheless, observe that the most common versions are `Microsoft Windows RPC` (9 entries) and `Wildfire XMPP Client` (2 entries). One could thing that **multiple open ports related to mwrpc imples that the target's infrastructure is highly dependent on this protocol**. This clustering of mwrpc-related ports strengthens the inference that this service is likely a critical component on the target system.

Shall we focus on `MWRPC`, `XMPP`, `LDAP`...? mwrpc and XMPP are the most common used services, so the server is highly dependent on these services. To decide on which service to focus first we must think:

1. XMPP servers (especially specific implementations like Openfire) can have known vulnerabilities or common misconfigurations that might be easier to exploit. In contrast, MSRPC (Microsoft Windows Remote Procedure Call) services are crucial and common in Windows environments and there might not be as many straightforward vulnerabilities. **Exploiting RPC services often require more specific conditions**.

Locally resolve the address:

(pic3)

Locally resolving the address means mapping a human-readable domain name (hostname) to an IP address on your local machine. This allows your computer to recognize and connect to a specific server or service using a more memorable hostname instead of a numerical IP address.

```
echo 10.129.230.215 jab.htb | sudo tee -a /etc/hosts
```



Additionally, the previous scan reveals several open ports that are typically associated with XMPP (Extensible Messaging and Presence Protocol) services used for instant messaging, presence information, and other communication services. 


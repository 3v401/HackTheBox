Start pinging the target IP:

```
ping 10.10.11.8
```

You will get response outcome meaning that the server is active. Let's start with an nmap scap to see what ports are opened and which information can be retrieved.

```
nmap -sC -sV 10.10.11.8 -oN output_nmap.txt
```

This command returns the output of nmap into `output_nmap.txt`file.

`-sC`: Default Script Scan. Runs a set of standard nmap scripts at the target host (Checking for open proxy servers, identifying HTTP servers, detecting SSL/TLS certificate details... etc). This command adds depth to the scan by running various scripts that can uncover additional information and potential vulnerabilities.

`-sV`: Version Detection. Probes open ports to determine what service and version is running on them. It sends various probes to the open ports and analyzing the responses (e.g., Identifying application and version running on each open port). It ensures you get accurate service and version information for the open ports, which can be crucial for identifying specific vulnerabilities and understanding the network environment better.
Concatenating `-sC` and `-sV` in `nmap` scans is a common practice because it provides a detailed analysis of the target.

The summary of the outcome is the following:

1. Host IP: `10.10.11.8`
2. Open ports:
   a. 22/tcp: SSH (OpenSSH 9.2p1)
   b. 5000/tcp: Web server
3. Operating System: Linux

The outcome shows that the open SSH service on port 22 and the web server on port 5000 might be targets for further investigation or exploitation.

![Alt text](nmap_headless.png)

As there is a webserver host. The most interesting way is to access from a browser `http://10.10.11.8:5000`. It automatically redirects to `http://10.10.11.8:5000/support`. We observe that the site is reachable and contains a form to send. Usually forms are good infection points which attackers can leverage to bypass systems or interact with them.

(pic2)
(pic3)

Instead of trying common scripts in javascript to interact with the website and make them vulnerable. Let's scan what this domain has to offer. There is a famous tool in Kali Linux called `Gobuster`.

`Gobuster` is a tool for brute-forcing directories, files, DNS subdomains and virtual hosts. It is commonly used in pentesting to discover hidden resources on a web server that might not be immediately visible through normal browsing or standard `web crawlers`. To install it type: `sudo apt install gobuster`. Then run the following command:
```
gobuster dir -u http://10.10.11.8:5000 -w /usr/share/wordlists/dirb/big.txt
```
gobuster uses the `dir` flag to scan for directories on the `-u` (url flag) on the domain specified using the `-w` worldlist flag located in `/usr/.../big.txt`. It can be observed that there are two available URLs `/dashboard` (Status:500, , i.e., Internal Server Error) and `/support` (Status: 200, i.e., OK). Our URL of interest is `/dashboard`. The reason why it isn't accessible can be because server configuration errors, resource limits... the most plausible situation is permission issues. It is highly likely that we don't have persmissions to access such URL so let's dig more into it.

(pic4)
(pic5)

So to access `http://10.10.11.8:5000/dashboard`we need to be able to login or have admin privileges. A good way to login is to find a login page (and use brute force for example or any additional hits in such login page). Nonetheless, we only have a non-accessible URL. There must be a way to bypass the server and make them think that we are authorized to access the URL. A good way to bypass/trick the server is to give an admin cookie or session id. The differences between admin cookie and session id:

(pic6)


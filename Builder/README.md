The --min-rate=1000 option in nmap is used to specify the minimum number of packets sent per second during the scan. This is particularly useful for speeding up the scan, especially when dealing with a large number of ports (like when scanning all 65,535 TCP ports)

Comparison with Other Timing Templates

    -T0 (Paranoid): Extremely slow and stealthy, used to avoid detection by IDS/IPS systems.
    -T1 (Sneaky): Very slow and stealthy, similar to -T0 but slightly faster.
    -T2 (Polite): Slow but polite, waits longer between sending packets to avoid causing disruption.
    -T3 (Normal): Default setting, balances speed and performance without being too aggressive.
    -T4 (Aggressive): Faster than the default, uses shorter timeouts and more parallelism.
    -T5 (Insane): Extremely fast and aggressive, can overwhelm networks and be detected easily.

(pic1)

We don' t have access credentials to the SSH service, so let' s try with the opened (accepts connections) port 8080. We can observe that the specific HTTP server software is Jetty 10.0.18. The title indicates that the web application running on this port is Jenkins (a popular automation server used for CI/CD). We also observe a "robots.txt" file which is used by websites to give instructions to web crawlers and spiders about which pages should not be indexed. You can observe in the next row the term "_/" which means that the root directory is disallowed, meaning that the entire site shouldn' t be indexed by search engines. http-open-proxy indcates that the scan detected the server might be an open proxy, which means it coould potentially allow unauthorized users to relay their requests through this server. An http open proxy is a type of proxy server that allows any external user to send requests through it to other servers. This can include browsing websites, sending emails, or connecting to other network services, effectively masking the user's original IP address.

The HTTP-server-header reveals the server software used (Jetty 10.0.8). Let's enter in 10.10.11.10:8080 and see what appears

(pic2)

Before scrolling and visiting sections, we have to detect the software version that the site is built on. We see that it is Jenkins 2.441.

#### Jenkins

[Jenkins](https://en.wikipedia.org/wiki/Jenkins_(software)) is a tool for automating the software development process, from code integration and testing to deployment. Its extensibility and integration capabilities make it a popular choice for teams practicing DevOps and agile methodologies.

When you access the webpage at `http://10.10.11.8:8080` on your Linux server and see that it uses Jenkins, it means that Jenkins is installed and running on that server. The Jenkins web interface is being served on port 8080, allowing you to interact with Jenkins through your web browser.

So, if you can interact with Jenkins to the server, let's look for "Jenkins 2.441 vulnerability" on the internet. After a quick search we find the following [URL](https://www.jenkins.io/security/advisory/2024-01-24/#SECURITY-3314)

`Arbitrary file read vulnerability through the CLI can lead to RCE`. RCE stands for Remote Code Execution.
It states that Jenkins has a built-in command line interface (CLI) to access Jenkins from a script or shell environment. Jenkins uses the `args4j` library to parse command arguments and options on the Jenkins controller when processing CLI commands. This command parser has a feature that replaces an `@` character followed by a file path in an argument with the file’s contents (`expandAtFiles`). This feature is enabled by default and Jenkins 2.441 and earlier, LTS 2.426.2 and earlier does not disable it.

So, how do we get a Jenkins CLI? Typing on Google `get jenkins cli` we access the Jenkins documentation, read the description and access to section [Downloading the client](https://www.jenkins.io/doc/book/managing/cli/#downloading-the-client), which is what we want to interact with the server.

The documentation states that the CLI client can be downloaded directly from a Jenkins controller at the URL /jnlpJars/jenkins-cli.jar, in effect `JENKINS_URL/jnlpJars/jenkins-cli.jar`. So we connect to the host server and introduce such URL:

(pic3)

Now that we have the Jenkins-CLI, how do we use Jenkins-CLI? How do we insert the payload into the webserver?

##### How to use Jenkins-CLI

The general syntax of using Jenkins-CLI.jar can be found [here](https://www.jenkins.io/doc/book/managing/cli/#using-the-client). The command should be:

```
java -jar jenkins-cli.jar [-s JENKINS_URL] [global options...] command [command options...] [arguments...]
```

##### How to insert Payload into Jenkins 2.441

Searching `CVE-2024-23897 PoC` we find the following Splunk [site](https://www.splunk.com/en_us/blog/security/security-insights-jenkins-cve-2024-23897-rce.html). Reading the section `Payload Body Explanation` we can observe the payload structure as:

`[Command Length][Command ('help')][File Path Length][File Path ('@/etc/passwd')][Other Parameters]`

From here we can infere that the injection should be something like `Jenkins-CLI command -s http://<JENKINS_SERVER>:8080 help "@/etc/passwd"`. Therefore, the injection should be:

```
java -jar jenkins-cli.jar -s http://<JENKINS_SERVER>:8080 help "@/etc/passwd"
```

Why the path `/etc/passwd`? Searching the `/etc/passwd` file on a Unix or Linux system is crucial for both administrative and security purposes because it contains vital information about all user accounts on the system. This file lists usernames, user IDs, group IDs, home directories, and default shells for each user, though it does not contain actual passwords, which are stored in `/etc/shadow` (we will check it later). For system administrators, `/etc/passwd` is essential for managing user accounts and auditing the system to identify any unusual or unauthorized entries. From a security perspective, attackers often target this file during reconnaissance to gather information about user accounts, which can be used for privilege escalation, targeted password attacks, and other exploitative activities. Understanding the details and configurations of user accounts helps both in maintaining system integrity and in identifying potential security vulnerabilities.

(pic7)

In this picture, we attempted to use the Jenkins CLI to read the /etc/passwd file by passing it as an argument to the help command. In Unix-like systems, @filename can be used to pass the contents of filename as an argument, and as a result, the contents of /etc/passwd were read and passed to the help command.

The help command interpreted the contents of `/etc/passwd` as multiple arguments, leading to the error message: ERROR: Too many arguments: daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin. This line is the first line from /etc/passwd, showing the entry for the daemon user. The daemon user on Unix-like systems is a special system account used to run background services (daemons). It typically has limited permissions and is not intended for interactive login. This account helps in organizing and running system processes securely without giving them root privileges.
There is also presence of the root user. The structure `root:x:0:0:root:/root:/bin/bash` from /etc/passwd provides key details about the root user account: the username (root), a placeholder indicating the password is stored in /etc/shadow (x), the user ID (0), the group ID (0), the GECOS field with user info (root), the home directory (/root), and the default shell (/bin/bash). This format helps the system manage user accounts, with each field separated by colons.

The CLI misinterpreted the file content as multiple arguments due to the way the help command processes its input. Let' s see the /etc/shadow to find the password:

(pic8)

No contents for `java -jar jenkins-cli.jar -s 'http://10.10.11.10:8080' help "@/etc/shadow"`.

?Nonetheless, we don' t find any example of how to implement it. So let's look on Google " CVE-2024-23897 proof of concept" and you will find the following site: https://github.com/3yujw7njai/CVE-2024-23897?









But how to get a client jenkins-cli.jar? Let' s type on google: "download the client jenkins-cli.jar" and select the Jenkins wiki (https://wiki.jenkins.io/JENKINS/Jenkins-CLI.html). You will observe the following statement:

(pic4)

So what we have to do is to connect to the jenkins server and add the additional URL to download:

(pic3)

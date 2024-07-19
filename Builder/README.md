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

Jenkins is a tool for automating the software development process, from code integration and testing to deployment. Its extensibility and integration capabilities make it a popular choice for teams practicing DevOps and agile methodologies.

When you access the webpage at `http://10.10.11.8:8080` on your Linux server and see that it uses Jenkins, it means that Jenkins is installed and running on that server. The Jenkins web interface is being served on port 8080, allowing you to interact with Jenkins through your web browser.

So, if you can interact with Jenkins to the server, let's look for "Jenkins 2.441 vulnerability" on the internet. After a quick search we find the following [URL](https://www.jenkins.io/security/advisory/2024-01-24/#SECURITY-3314)

RCE stands for Remote Code Execution.


   
?Nonetheless, we don' t find any example of how to implement it. So let's look on Google " CVE-2024-23897 proof of concept" and you will find the following site: https://github.com/3yujw7njai/CVE-2024-23897?









But how to get a client jenkins-cli.jar? Let' s type on google: "download the client jenkins-cli.jar" and select the Jenkins wiki (https://wiki.jenkins.io/JENKINS/Jenkins-CLI.html). You will observe the following statement:

(pic4)

So what we have to do is to connect to the jenkins server and add the additional URL to download:

(pic3)

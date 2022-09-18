---
layout: post
title:  SSH Tunnel
date:   2019-11-09
categories: techniques
---

{:.text-centered}
# || SSH Tunnel ||
![Banner](../../../../assets/img/ssh/tunnel.jpg){:.image-centered}

A very popular service used by almost all system administrator for remote administration as it is secure encrypted connection. For a pentester it is more useful than it seems.

Quick Basic: 
default ssh port : 22
Both windows and linux support ssh
ssh provides you remote tty shell
Login:

    ssh user@168.192.1.100 (or user@hostname.com)

Login with ssh key:

    ssh -i key user@hostname.com

That goes very basic of ssh,but ssh has a important feture - tunneling or ssh port forwarding,this is very useful in various condition and that we will discuss here.
### Local port forwarding:

Ok imagine a condition when you are in a remote server and you got a service listening on localhost on that server and you might find that you can exploit it(database server normally listens on localhost),but how can you access it? Yes,you can connect to it via a ssh tunnel.

    ssh -L 8080:localhost:8080 user@remotehost.net

-L stands for local,and it is called local port forwarding. First 8080 is port of your machine,2nd 8080 is port of remote machine.
Now,if you requst localhost:8080 from your browser that request will be send through that ssh tunnel to that remote host and you will be able to connect to that remote service.
So,what this tunnel does is whatever request you sent to your localhost it will be forwarded to remote localhost (I hope this line makes sence!)
Ok,now make it more complicated(not really,just joking!!).Send a request to a server through a second ssh proxy machine from your client. Your machine is client,second machine to which you are making tunnel is proxy and third machine is server.

    ssh -L 8000:server:80 user@proxy.net

then open localhost:8000 in browser and you will be connected to server. The machine to which you are making tunnel is working as proxy and and sending request to server on your behalf,so if your ip is blocked,but the ssh proxy machine can access the server,you can use this. Note that this is also working as a very basic vpn,it is totally encrypted and in your control.
But it is not fully functional vpn or proxy as here ip and port of the socket is fixed,to overcome this limitation you can use dynamic tunneling.
### Dynamic tunneling:
You can use your ssh server as a fully functional proxy through dynamic tunneling. It uses SOCKS proxy,a proxy protocol, under the ssh tunnel. 

    ssh -D 8080 user@server.net

This simple command will create a proxy,and configure your browser to send traffic through proxy 8080,then all the traffic of your browser
### Remote port forwarding:
Another type of port forwarding is there called remote port forwarding,useful in case when a service is brhind NAT and you want to expose it to public,this type can be used.

    ssh -R 4000:server.net:5000 user@server.net

It will forward all the request to server port 5000 to port 4000 of your machine. When your machine is behind  a NAT one can send a request to server:4000 and that will be forwarded to your machine.


Quick commands:

    ssh -L 8000:locahost:9000 user@hostname.net
    ssh -L localhost:8000:localhost:9000 user@hostname.net

(same as first one but in first case anyone can use this tunnel on your network,but in second case only you can connect)

    ssh -D 9001 user@hostname.net
    ssh -R 4000:server.net:5000 user@server.net

Hope you enjoyed; Happy Hacking.
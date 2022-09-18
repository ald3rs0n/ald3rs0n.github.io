---
layout: post
title:  Traverxec
date:   2020-04-11
categories: htb-writeup
---

{:.text-centered}
## TRAVERXEC@HTB

![Banner](../../../../assets/img/traverxec/Banner.png){:.image-centered}

Traverxec is an easy box from hackthebox. Ok let’s start.

### Quick Hack:

__User:__  Port Scan > 80/http > nostromo server > search for exploit > metasploit exploit > reverse shell > reading nostromo conf and manual > getting ssh creds in a directory > ssh as david > user.txt

__Root:__  david > reading server-stats.sh script > sudo without password for a perticular command > abusing it with vim > root.txt

### US3R
Run nmap to find open ports. Nmap will show result like this:

![nmap scan](../../../../assets/img/traverxec/nmap.png){:.image-left}

We have two open ports; 22 is ssh and 80 is http. Open http in browser. There is nothing interesting other than a name David White, which may be a potential user name. Just take a note. View source nothing there. Nmap says it is running nostromo 1.9.6,any 404 error also shows the same. 

![web](../../../../assets/img/traverxec/web.png){:.image-left}

Ok so let's learn about nostromo. After some reading I came to know that it is a web server. Ran searchsploit against it, got a Directory Traversal RCE but that is for an older version. I googled for nostromo 1.9.6 vulnarebilites and got a metasploit exploit for RCE. [Exploit](https://packetstormsecurity.com/files/155045/nostromo_code_exec.rb.txt)

Recently there is a python exploit too,but I didn't used that,metasploit worked fine for me. [Exploit python](https://packetstormsecurity.com/files/155802/nostromo-1.9.6-Remote-Code-Execution.html)

Ran the metasploit exploit and got a session as www-data but I transfered it to a netcat session and stebilized it.

![netcat shell](../../../../assets/img/traverxec/stabilized.png){:.image-left}

Now from this point you can run linenum or linpeas but it doesn't helps.Looking at /home directory,there is only one user david,we don't have read or write access to this directory,this is not normal,normally read access to this directory is given. Let's go to nostromo server directory to see if we can find there anything interesting. 

    cd /var/nostromo


Let's go to conf folder. There is a file nhttpd.conf


![nhttpd file](../../../../assets/img/traverxec/nhttpd.conf.png){:.image-left}

To understand this file we have to read the documentaion of nostromo server. Let's google it.

[nostromo documentation](http://www.nazgul.ch/dev/nostromo_man.html)

I read it several time to understand how everything works. At homedirs part it says it can serve users home directory as docroot. That is interesting as we don't have reading access to users home directory. There is another option to restrict access user home directory by setting up homedirs_public option in .conf file. Let's look at nhttpd.conf file again. It sets up homedirs_public to public_www.

![documentation](../../../../assets/img/traverxec/documentation.png){:.image-left}

So there must be a directory named public_www in users home directory.

    cd /home/david/public_www

![public_www](../../../../assets/img/traverxec/public_www.png){:.image-left}

It works.Ahhhhh,thats why there was no read access to this /david directory,so that we can't find it easily! List files, there is a directory name protected-file-area. Let's go there. There is a file name backup-ssh-identity-files.tgz .

![gzip](../../../../assets/img/traverxec/ssh.tgz.png){:.image-left}

This is a tar gzip file,name suggests it has ssh creds. Ok let's extract it to /tmp folder.

    tar zxvf backup-ssh-identity-files.tgz -C /tmp
    cd /tmp
    ls -la

it creates a folder /home,let's go there.

    cd home

there is a folder david and within it there is a folder .ssh let's go there. There are three files. id_rsa, id_rsa.pub, authorized_keys. Get this files and try to login with private key.

    ssh -i id-rsa david@10.10.10.165

ooops! It asks for passphrase. Ok,give this file to john to crack it.

    python ssh2john.py id_rsa > id_rsa.john
    john id_rsa.john --wordlist=/usr/share/wordlists/rockyou.txt

john cracks it very soon and the passphrase is "hunter".

Ok, now login with this crades and get the user flag.

![user flag](../../../../assets/img/traverxec/user.txt.png){:.image-left}


## R00T
Getting root is easy. David's home directory has two files. server-stats.head and server-stats.sh
Run the shell script. It prints nostromo server information. Ok let's check the script.

    cat server-stats.sh


![server](../../../../assets/img/traverxec/server-stats-output.png){:.image-left}

Everything is fine but last line is interesting. Running sudo without password. Copy that last line and run it from terminal it works fine. But how to exploit that? Ok let's remove the cat command from the end part and it still works. It prints server information,now press v when it is showing the server details. It opens in vim. now type 

    :!/bin/sh

It opens shell as root.

How do I know to exploit it in this way? This is a similar hack which was given in bandit game of overthewire. You can play that awesome game [here](https://overthewire.org/wargames/bandit/).
When I remove the cat command it opens in "more" or "less". And this is a feature of that tool that when you type 'v' it opens in vim. Vim is powerful text editor in linux,but most importantly you can run command from vim. And thats what I did,as vim was opened as root I just opened shell from there and became root. You can learn what you can do with vim from [gtfobins](https://gtfobins.github.io/).

How do I know it opened it "more" or "less"? Just by looking at the left below corner of the terminal.

![more](../../../../assets/img/traverxec/more.png){:.image-left}

![root flag](../../../../assets/img/networked/root.txt.png){:.image-left}

Hope you enjoyed; Happy Hacking.
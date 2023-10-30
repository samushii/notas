querying for computer info can be made in powershell, the ***Get-WmiObject*** cmdlet can query a wide variety of objects for information about the local and remote hosts

***Get-ComputerInfo***  can also be used

## Remote Desktop Protocol

rdp uses a client/server architecture to allow remote access of a windows machine, it uses port TCP/3389 by default [rdp must be previously allowed on the destination machine]

access from a windows host can be made through the native ***mstsc.exe***, linux has tools such as ***xfreerdp*** and ***remmina*** 

#### connecting to a windows target from linux using xfreerdp:
 
	xfreerdp /v:[target-ip]:[port] /u:[username] /p:[password]

<br>


## Permissions

***integrity control access control list*** (icacls) can be used to check and manage permissions for files 

basic ntfs permissions are:

- **F** : full access
- **D** : delete access
- **N** : no access 
- **M** : modify access 
- **RX** : read and execute access 
- **R** : read-only access 
- **W** : write-only access 

#### checking permissions for [dir]:

	icacls [dir]

#### grant permissions over [dir] for [user]:

	icacls [dir] /grant [user]:[permission]

<br>


## SMB

on linux, smbclient can be used to connect to a windows share
[a rule might need to be added on windows firewall allowing smb traffic, which uses default port TCP/445]

### connecting to share [sharename]:

	smbclient -L \\[server-ip]\[sharename] -U [username]
[the -L options is used to list share content,]

### mounting the share [sharename] to [dir]:

	sudo mount -t cifs -o username=[username],password=[password] \\[server-ip]\[sharename] [local-dir]
[cifs-utils package is required]

***Computer Management*** can be used to manage shares, permissions and files currently being used

***Event Viewer*** can be used to monitor logs about SMB 

<br>





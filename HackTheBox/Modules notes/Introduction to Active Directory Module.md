## Important Terminology

- ### NTDS.dit

	-  stored in ***C:\\Windows\\NTDS\\***
	- is a database that stores AD data such as user and group objects, group membership and password hashes for all users in the domain
	- if "Store password with reversible encryption" is enabled, then **cleartext** credentials will be stored 

<br>

## Kerberos, DNS, LDAP, MSRPC

### Kerberos

kerberos is the default authentication protocol for domain accounts, it decouples users' credentials from their request to consumable resources, ensuring passwords aren't transmitted over the network

it uses port **88 TCP/UDP** by default

#### authentication process:
1. User logs on and their password is converted to NTLM hash, which is used to encrypt the Ticket Granting Ticket (TGT) request
2. Kerberos Distribution Centre (KDC) on Domain Controller checks the request (AS_REQ), creates the TGT and delivers it to the user (AS_REP)
3. The user presents the TGT to the DC when requesting Ticket Granting Service Ticket (TGS) for a specific service instance, if it passes the validation the data in it is copied to the TGS (TGS_REQ)
4. The TGS is encrypted with the NTLM password hash of the service/computer account in which context the service instance is running, then it is delivered to the user (TGS_REP)
5. The user presents the TGS to the service and if valid, is able to connect to the service instance (AP_REQ)

<br>

### MSRPC

Microsoft Remote Procedure Call allows access to Active Directory systems using four key RPC interfaces:

| Interface Name | Description |
| - | - |
| lsarpc | set of rpc calls to the Local Security Authority (LSA) which manages local security policy on a computer, controls the audit policy and provides interactive authentication services |
| netlogon | is a windows process used to authenticate users and other services in the domain; it continuously runs in the background | 
| samr | remote SAM provides management functionality for the domain account database, storing information about users and groups; can be used for reconnaissance using tools such as BloodHound to create attack paths for how administrative access or full domain compromise could be achieved, by default all authenticated domain users can use these queries to gather information about the domain
| drsuapi | used to perform replication-related tasks accross DCs; can be used to create a copy of the AD database (NTDS.dit) file to retrieve password hashes to be cracked offline/used in pass-the-hash attacks

<br>





## Noteworthy privileges

| Privilege | Description | 
| - | - | 
| SeRemoteInteractiveLogonRight | This privilege could give our target user the right to log onto a host via Remote Desktop, which could potentially be used to obtain sensitive data or escalate privileges | 
| SeBackupPrivilege | could be used to obtain copies of sensitive files such as SAM and System registry hives and the NTDS.dit AD |
| SeDebugPrivilege | allows the user to debug and adjust the memory of a process, tools such as *mimikatz* could then be used to extract credentials from memory by reading the address space of LSASS process |
| SeImpersonatePrivilege  | allows the token impersonation of a privileged account such as NT AUTHORITY\\SYSTEM |
| SeLoadDriverPrivilege | allows loading and unloading drivers that could be used for privilege escalation or system compromising | 
| SeTakeOwnershipPrivilege | allows a process to take ownership of an object, at a basic level could be used to gain access to a file share/file that was not accessible before | 

<br>



## Active Directory Hardening

### LAPS

Microsoft Local Administrator Password Solution (LAPS) can be used to randomize and rotate local administrator passwords periodically

### Audit Policy Settings

logging and monitoring should be in place to detect any anomalous activities

### Group Policy Security Settings

GPOs should be in place enforcing security policies such as: 

	- Account Policies
	- Local Policies
	- Software Restriction Policies
	- Application Control Policies

### Update Management

WSUS can be installed as a role to minimize manual patching of windows computers, reducing downtime and human error (leaving systems unpatched)

### Group Managed Service Accounts (gMSA)

domain managed account that offers higher security than other types of service accounts, used with non-interactive applications, services and processes, also tasks that do not require credentials; it uses very long randomly generated passwords

### Security Groups

simplify rights assignment compared to individually managing each user by using groups with pre-set permissions

### Account Separation

administrators must have two separate accounts, one with standard user rights and other for execution of administrative tasks, also utilizing a **secure administrative host** improves security

### Password Complexity Policies + Passphrases + 2FA

ideally, password managers should be used, but in scenarios where they are not password complexity should be enforced (12 char length for standard users and even more for administrators/service accounts); implementation of two-factor authentication adds another layer of security and can limit lateral movement in the network

### Periodically Auditing and Removing Stale Users and Objects

it is important to audit AD periodically and remove/disable unused accounts

### Periodically Auditing permissions and access

ensuring users only have the right set of permissions for their day-to-day work

### Limiting Server Roles

installing only strictly necessary roles on sensitive hosts improves security by reducing the attack surface



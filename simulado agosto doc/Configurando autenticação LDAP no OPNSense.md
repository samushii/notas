
create firewall rule on domain controller allowing connection from the firewall on port 389

##### on opnsense

system>access>servers

create new

> type ldap
> hostname: dc ip
> port: 389
> user dn: get from ADSI
> password: user password
> search scope: entire subtree
> base dn: domain dn (ex: DC=caneco,DC=ad)
> initial template: microsoftAD
> user naming attribute: sam account name
> read properties check
> sync groups check 

system>access>users

> click on the cloud icon to import ad users
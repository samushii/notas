
passos:
1. criar grupos de acesso
2. instalar radius server
3. configurar radius server
4. configurar dispositivo cisco

##### instalar radius server

registrar o nps no active directory
criar radius client para o switch
	- nome
	- ip (é possivel utiizar um range de ips, como por exemplo todo o range da vlan de gerenciamento dos switches)
	- [shared secret template]

##### network policy
admin policy
	- conditions/windows groups/[cisco admins]
	- enable unencrypted authentication (pap), disable all others
	- radius attributes
		standard
		- remove default
		- add service-type/login
		vendor specific
		- cisco 
		- shell:priv-lvl=15


### configuração no switch

criar usuario administrador backup

username administrator privilege 15 secret 0 Senai@123456

aaa new-model
aaa authentication login default group radius local
aaa authorization exec default group radius local
radius server [nps-server-name] [nps-ip]
	address ipv4 [nps-server-address] auth port 1812 acct-port 1813
	key 0 [shared-secret-template]

ativar ssh
ip domain name [domain-name]
crypto key generate rsa general-keys modulus 2048 
line vty 0 15
	transport input ssh 
	login authentication default
	exit


Forensic 
	Windows 
		Caminhos Importantes
		
			C:\Windows\System32\config\												#Registro do windows
			C:\Users\<User>\Appdata\Local\Google\Chrome\User Data\Default\ 			#Chrome Informações
			C:\Users\<username>\AppData\Roaming\Mozilla\Firefox\Profiles\			#Firefox Informações
			C:\Users\<username>\AppData\Local\Microsoft\Edge\User Data\Default\		#Edge Informações
			C:\Windows\System32\winevt\Logs 										#Events Logs
			C:\$Recycle.Bin															#Lixeiras
			C:\Users\<usernam>\AppData\Local\Microsoft\Terminal 					#RDP Historico
			C:\Windows\Prefetch														#Prefetchs 
			C:\$mft
			
	Linux 
		Caminhos Importantes 
		
			/home/<username>/bash_history
			/var/log/
			/etc/hostname
			/etc/network/interfaces
			/etc/timezone
			/opt/
			/root/
			/etc/passwd
			/var/www
			/var/run 
			
		Coisas a verificar
		
			-Historico
			-Usuarios
			-Serviços 
			-Programas
			-Logs
			-Arquivos Suspeitos 
			-Rede 
			
	Trafego de rede
		Serviços importantes
		
			FTP
			DNS
			SSH
			HTTP
			RDP
		
Linux 
	Splunk	
		rpm -i splunk.rpm
		/opt/splunk/bin ./start 
		
		Pesquisas 
			FTP login 			= source=”c:\inetpub\logs\web\vsftpd\” ftp rule=”SUCCESS” rule=”FAILED” | stats 
			FTP arquivos 		= source=”c:\inetpub\logs\web\vsftp\” ftp UPLOAD | stats count by UPLOADS,Nome_arquivo
			WEB codigos erro 	= source=”c:\inetpub\logs\web\” GET NOT codigo=”200” | stats count by codigo 
			WEB acesso time 	= source=”c:\inetpub\logs\web\” GET | stat count by_time 
			Alerta FTP exe		= source=”c:\inetpub\logs\iss” exe 
			Alerta BF			= source=”c:\inetpub\logs\iss” user=* user=’’ | stats count(eval(action=”success”)) as successes count(eval(action=”failure”)) as failures by user, ComputerName | where successes >0 AND failures >500
			Dispositivos		= source=”/var/log/cisco/switch” Login stats count by dispositivo, hora, user, origem;
			Criticidade			= source=”/var/log/cisco/switch” stats count by severity

	Senhas 
		Chage
			chage -l <user>
			chage -M <days> <user>
			
		pwhistory 
			/etc/pam.d/system-auth
				"password	required	pam_history.so remember 5 use_authok"
		pwquality 
			minlen = tamanho 
			ucredit = letras maiusculas
			lcredit = letras minusculas 
			ocredit = outros caracteres
			
		faillock
			/etc/security/faillock
		
		permissão ou negação de usuarios ou ips 
			/etc/host.allow
			/etc/host.deny
			
	Syslog
		apt install rsyslog
		/etc/rsyslog/rsyslog.config 
		portas udp!! 
		facility 
		
	Proxy
		apt install nginx
		setsebool -P http_can_network_connect 1 
		getsebool -a | grep http 
		criar arquivo /etc/nginx/conf.d/<filename>.conf
			server{
				listen 80 default server;
				server_name_;
					location / { 
					return 301 https://$host$request_uri;
					}
			}
			server{
				listen 443
					location / {
					proxy_pass http://<server_ip>;
					}
			}
	Selinux
		sestatus
		/etc/selinux/config
	
	Firewall 
		firewall-cmd --add-port=80/tcp --permanent
		systemctl restart firewalld

	Freeradius
		apt install freeradius freeradius-utils -y

		nano /etc/freeradius/client.conf
	
			client <client name> {
				ipaddr = <ip_of_client>
				secret = <password_of_radius>
				require_message_authenticator = no 
				nas_type = cisco 
			}

		nano /etc/freeradius/users
	
			"<username>"	Cleartext-Password := "<password_of_client>"
					Service-Type = NAS-Prompt-User,
					Cisco-AVPair = "shell:priv-lvl=15"
			}	

		systemctl restart freeradius or radiusd
		systemctl enable freradius 
	
		firewall-cmd --add-port=1812/udp --permanent
		firewall-cmd --add-port=1813/udp --permanent

		systemctl restart firewalld

		Radius no switch

		username backup privilege 15 secret P@ssw0rd
	
		aaa new-model 
		radius server MYRADIUS
			address ipv4 <ip_of_server_radius> auth-port 1812 acc-port 1813
			key <key_of_radius> 

		aaa authentication login default group radius local 
		aaa authorization exec default group radius local
	
	Bind9
		yum install bind bind-utils 
		cp /etc/named.conf /etc/named.conf.orig

		vi /etc/named.conf --> 

		options {
     			   #listen-on port 53 { 127.0.0.1; };
     			   #listen-on-v6 port 53 { ::1; };
      		  directory       "/var/named";
		}

		allow-query  {localhost; 192.168.56.0/24}

		#Create a zone 

		vi /etc/named.conf

		zone "pollos.ltd" IN {
			type master;
			file "pollos.ltd.db";
			allow-update {none;};
			allow-query {any;}
		};

		#Create a Forwarder zone in /var/named
		"
			$TTL 86400
			@ IN SOA dns-primary.tecmint.lan. admin.tecmint.lan. (
			    2019061800 ;Serial
			    3600 ;Refresh
			    1800 ;Retry
			    604800 ;Expire
			    86400 ;Minimum TTL
			)
			
			;Name Server Information
			@ IN NS dns-primary.tecmint.lan.
			
			;IP for Name Server
			dns-primary IN A 192.168.56.100
			
			;A Record for IP address to Hostname 
			www IN A 192.168.56.5
			mail IN A 192.168.56.10
			docs  IN A 192.168.56.20
		"
		firewall-cmd --add-service=named --permanent
		firewall-cmd --add-port=53/udp --permanent
		
		systemctl restart firewalld 
		systemctl restart named 
		

		named-checkconf
		named-checkzone pollos.ltd /var/named/pollos.ltd.db

	Chown 
		chown root:root -r /var/named

	Snort 
		yum apt install snort -y 
		snort --version 

		nano /etc/snort/snort.conf
			ipvar HOME_NET 172.16.1.1/16
			
			#Editar as regras nativas 

				"sudo vim /root/.vimrc" "set number" "syntax on" 
				"sudo vim /etc/snort/snort.conf" ":578,696s/^/#" --> All default rules will are commented

 		sudo snort -T -i eth0 -c /etc/snort/snort.conf 

			#
	
			alert tcp $EXTERNAL_NET 80 -> $HOME_NET any
			(
			    msg:"Attack attempt!";
			    flow:to_client,established;
			    file_data;
			    content:"1337 hackz 1337",fast_pattern,nocase;
			    service:http;
			    sid:1;
			)

	Permissão Usuarios Linux
		/etc/sudoers #Permissão de sudo e comandos com sudo 

		#Usuario não pode logar localmente, somente via ssh
			"-:lionel:ALL" >> /etc/security/access.conf 
			"account required	pam_access.so" >> /etc/pam.d/system-auth
			"AllowUser lionel" >> /etc/ssh/sshd_config
		
		#Accessar apenas um diretorio 
			chown -R Lionel:Lionel <path>

	Grub 
		#Proteger grub com senha 
			grub2-setpassword <senha>

	Autidoria 
		sudo rum install audit 
			#Regra para auditoria em comando 
			“-a always,exit -F path=/usr/bin/ls -F perm=x -F auid>=1000 -F auid!=4294967295 -k ls-command” > /etc/audit/rules.d/ls-command ; ausearch -k ls-command;"
Windows Server 

	Shared 
		Criar share;
		Criar quota para subfolders na share;
		Criar file screening  e aplicar na share e subfolders;
		Selecionar todos os usuários, aba propriedades, sub-aba home, definir home folder com o nome do servidor e share ex:”\\WIN-DC\usuarios\%username%”, selecionar mapeamento de unidade para H;



Switch 

			en
			conf t
			hostname SW-1
			ip domain-name pollos.ltd
			banner-motd Apenas usuarios autorizados
			login block-for 60 attempts 3 with 60 
			enable password Senai@123456
			service password-encrypt
			line console 0 
				exec-timeout 15 
				exit
			crypto key generate rsa modulus 2048
			access-list 1 permit ip 192.168.17.0 255.255.255.0
			line vty 0 4 
				transport input ssh
				access-class 1
			int r e0/0-4
				switchport port-security
				switchport port-security maximum 2
			int e0/2 	
				switchport port-security sticky 
				switchport port-security violation restrict 
			int e1/1
				shutdown

			en
			conf t
			hostname HQ-SW
			ip domain-name POLLOS.LTD
			set ntp server <ip>
			service password-encrypt
			line console 0 
				exec-timeout 15 

			access-list 1 permit 192.168.0.0 255.255.255.0 
			access-list extend SSH 
				deny tcp any any eq 22
				permit ip any any
			line vty 0 4 
				transport input ssh 
				access-class 1
				access-class SSH

			crypto key generate rsa modulus 2048
			ip ssh port 2222 rotatory 1
			username userlocal1 privilege 15 secret Senai@123456
			username userlocal2 privilege 15 secret Senai@123456
			aaa new-model 
			radius server MYRADIUS
				address ipv4 <ip_server> auth 1812 acc-port 1813
				key P@ssw0rd
			aaa authentication login default group radius local 
			aaa authorization exec default group radius local 
			int r e0/2-3
			shutdown 
			ip dhcp snooping 
			ip dhcp snooping vlan 1
			int e0/1 <dhcp_interface>
			ip dhcp snooping trust
			ip arp inspection vlan 1
			banner-motd Somente Acesso Autorizado!
			aaa authentication dot1x default 
			int port 
			access-session port-control auto 
			end 
			show dot1x
			do wr

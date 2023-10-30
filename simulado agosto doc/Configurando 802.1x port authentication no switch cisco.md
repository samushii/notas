

aaa new-model
aaa authentication dot1x default group radius
aaa authorization network default group radius

radius server [NPS-SERVER]
	address ipv4 [nps-address] auth-port 1812 acct-port 1813
	key [nps-shared-secret]

dot1x system-auth-control

interface [interface-id]
	switchport mode access
	authentication port-control auto
	dot1x pae authenticator


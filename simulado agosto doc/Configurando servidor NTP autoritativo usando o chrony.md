
## 1. Configure o tempo local manualmente

date +%T -s 13:30:00
ou 
timedatectl set-time 13:30:00

## 2. Edite o arquivo /etc/chrony.conf

server 192.168.0.1 prefer iburst
driftfile /var/lib/chrony/drift
local stratum 1
allow 192.168.0.0/24

###### Opcional:

criar arquivo /etc/chrony/chrony.keys com uma linha contendo

1 MD5 [hash]

configurar as diretivas:

server 192.168.0.1 key 1
keyfile /etc/chrony.keys 

## 3. Reinicie o serviço

systemctl restart chronyd

chronyc makestop
chronyc ntpdata
timedatectl

# Configurando RHEL como cliente NTP

server 192.168.0.1 prefer iburst


server master iburst
keyfile /etc/chrony.keys


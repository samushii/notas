### DHCP Snooping

1. Ativar DHCP Snooping globalmente
2. Ativar DHCP por VLAN
3. (Opcional) Desabilitar a opção 82 nos pacotes DHCP
4. Configurar as interfaces confiáveis no switch
5. (Opcional) Configurar rate limit para pacotes DHCP request 

 
```switch-cli
configurações globais:
sw1(config)# ip dhcp snooping
sw1(config)# ip dhcp snooping vlan [vlan-number]
sw1(config)#no ip dhcp snooping information option

configurações de interaface:
sw1(config-if)#ip dhcp snooping trust
sw1(config-if)#ip dhcp snooping limit rate [number]

debug:
sw1#show ip dhcp snooping
```


### Dynamic ARP Inspection (DAI):

Obs.: A configuração do DAI depende da configuração do DHCP Snooping pois utiliza a dhcp snooping binding database para validação

1. Habilitar arp inspection por vlan
2. Configurar interfaces confiáveis

```switch-cli
configuração global:
sw1(config)#ip arp inspection vlan [vlan-number]

configuração de interface:
sw1(config-if)#ip arp inspection trust

debug:
sw1#show ip arp inspection statistics vlan [vlan-number]
```

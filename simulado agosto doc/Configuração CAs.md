
# Windows

## WIN-AD

### Configurando Autoridade de Certificação Raíz:

#### Passos:

- Instalar AD CS
- Configurar AD CS
	- Definir o tipo da CA ([Enterprise]/Standalone)
	- Definir a hierarquia da CA ([Root]/Subordinate)
	- Criar chave privada
	- Definir o Common Name ([Worldskills Lyon 2024]) e o Distinguished Name ([OU=TI,O=Pollos,L=Lyon,ST=ARA.C=FR,DC=pollos,DC=ltd])
	- Configurar extensões CDP e AIA utilizando uma share (localizada na Subordinate CA) para publicação e um website para validação pelos clientes
- Criar entrada no DNS para o ponto de publicação das CRLs e AIA
##### Observações sobre instalação da CA:

######  CDP

- num cenário com uma Root CA e uma Subordinate CA é necessário configurar a extensão CDP para utilizar um UNC file path para a publicação (a share na Subordinate CA por exemplo, que deve ter permissão "Full Control" para um grupo *domain local* que contenha o grupo *Cert Publishers*) com as checkboxes "Publish CRLs to this location" e "Publish Delta CRLs to this location") e uma url http a ser utilizada para validação com as checkboxes "Include in CRLs..." e "Include in the CDP extensions of issued certificates"

- não é recomendado usar uma url ldap:// e não é atualmente suportado utilizar uma url file://

- as urls usadas como AIA e CDP devem ser **obrigatoriamente** http e não https

- o nome do arquivo (.crl) deve ser simples e sem espaços e deve possuir as variáveis \<CRLNameSuffix> e \<DeltaCRLAllowed> antes da extensão

- o nome dos arquivos **deve** ser o mesmo em ambas urls (UNC e http://)

###### AIA

- uma Windows CA não suporta publicação de certificados em localizações customizadas portanto o arquivo deverá ser renomeado e copiado para a o diretório do website manualmente, por isso apenas uma url é necessária (http://)

- a url deve terminar com a extensão .crt e possuir a variável especial \<CertificateName> 


## BO-WIN-SRV

### Configurando Autoridade de Certificação Intermediária:

#### Passos:


- Instalar AD CS
- Configurar AD CS
	- Definir o tipo da CA ([Enterprise]/Standalone)
	- Definir a hierarquia da CA (Root/[Subordinate])
	- Criar chave privada
	- Definir o Common Name ([CA Intermediaria]]) e o Distinguished Name ([O=SENAI,ST=WS2024.C=BR,DC=pollos,DC=ltd])
	- Configurar extensões CDP e AIA utilizando uma share (localizada na Subordinate CA) para publicação e um website para validação pelos clientes
- Configurar  share que será usada como site e diretório virtual para local de publicação das CRLs e AIA 
	-  IIS -> Request Filtering -> Allow URL double escaping
	- Configurar permissões NTFS na share para que o grupo *Cert Publishers* tenha "Full Control" sobre a share, além disso convém desabilitar herança de permissões

### Configurando Certificate Autoenroll para Computadores e Usuários

#### Subordinate CA 

##### Usuários

- Server Manager -> Tools -> Certification Authority -> Certificate Templates [Manage]
- Fazer uma cópia do template User Certificate
	- Configurar nome, alterar modo de compatibilidade (o padrão é Windows Server 2003 e **não** deve ser usado), opções de criptografia (o padrão usa Legacy Crypto Service Provider portanto não é recomendado, KSP deve ser usando e de preferência SHA-256 para a assinatura)
	- Domain Users devem ter permissão read, enroll e autoenroll
	- Na aba "Subject Name" desmarcar a opção "Include e-mail name in subject name" e a checkbox "E-mail name"
- Configuração do Group Policy Object
	- Computer Configuration > Policies > Windows Settings > Security Settings > Public Key Policies/Certificate Services Client - Auto-Enrollment
		- Enabled
		- Ativar as opções "Renew expired certificates..." e "Update certificates that use certificate templates"
	- Computer Configuration > Policies > Windows Settings > Security Settings > Public Key Policies/Certificate Services Client - Certificate Enrollment Policy
		- Enabled
##### Computador

- Server Manager -> Tools -> Certification Authority -> Certificate Templates [Manage]
- Fazer uma cópia do template Computer Certificate
	- Configurar nome, alterar modo de compatibilidade (o padrão é Windows Server 2003 e **não** deve ser usado), opções de criptografia (o padrão usa Legacy Crypto Service Provider portanto não é recomendado, KSP deve ser usando e de preferência SHA-256 para a assinatura)
	- Domain Computers devem ter permissão read, enroll e autoenroll
- Configuração do Group Policy Object
	- Computer Configuration > Policies > Windows Settings > Security Settings > Public Key Policies/Certificate Services Client - Auto-Enrollment
		- Enabled
		- Ativar as opções "Renew expired certificates..." e "Update certificates that use certificate templates"
	- Computer Configuration > Policies > Windows Settings > Security Settings > Public Key Policies/Certificate Services Client - Certificate Enrollment Policy
		- Enabled

### Consertando problema "the revocation function was unable to check revocation because the revocation server was offline"

Esse problema apareceu após reiniciar os servidores que estavam rodando o AD CS, os passos que funcionaram para resolvê-lo foram os seguintes:

Desativando a checagem para que o serviço inicialize:
- certutil -setreg ca\\CRLFlags +CRLF_REVCHECK_IGNORE_OFFLINE

Com a CRL mais atual emitida pela Root CA no diretório CertEnroll:
- certutil -addstore -f Root [CRL-NAME].crl
- certutil -setreg chain\\chainCacheResyncFiletime @now
- certutil -setreg ca\\CRLFlags -CRLF_REVCHECK_IGNORE_OFFLINE
- Reiniciar o serviço


### Consertando problema "the request subject name is invalid or too long"

O AD CS limita o campo DN do certificado X.509 em 64 caracteres, mas pode acontecer de um certificado ter um DN maior que isso, portanto é necessário desativar o limite utilizando o seguinte comando

```windows-cmd
certutil -setreg ca\EnforceX500NameLengths 0
```


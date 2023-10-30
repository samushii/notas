(nesse cenário ambas as comunicações serão protegidas por SSL utilizando o mesmo certificado wildcard)
## Windows Server (IIS)

- Instalar roles *IIS - Web Server* e *IIS - Web Server - Security - IP Address and Domain Restrictions*
- Requisitar certificado baseado no template Web Server*
- Configurar site IIS
	- Criar diretório raiz (grupo Everyone deve ter permissão de leitura), criar arquivo index.html
	- Configurar binding para utilizar hostname presente no certificado
- Configurar restrições de IP (nesse caso apenas o IP do NGINX reverse proxy vai ter uma entrada allow, a outra entrada será deny 0.0.0.0 e bloqueará acesso de todos os outros hosts [incluindo o servidor local])

\*o certificado deve possuir a opção "Make private key exportable"
## Linux Server (NGINX)

Primeiramente o certificado criado deve ser exportado em formato .pfx protegido com senha na Issuing CA, e então copiado para o servidor Linux

o formato .pfx compreende o certificado e a chave privada encriptados, então é necessário separar os dois utilizando o comando openssl

### Extraindo a chave privada do arquivo .pfx

```bash
openssl pkcs12 -in [Certificado-PrivateKey].pfx -nocerts -out [PrivateKey].pkcs12
```

### Extraindo o certificado do arquivo .pfx

```bash
openssl pkcs12 -in [Certificado-PrivateKey].pfx -nokeys -out [Certificado].pem
```

### Descriptografando a chave privada

```bash
openssl rsa -in [PrivateKey].pkcs12 -out [PrivateKey].pem
```

### Configurando o Reverse Proxy

- Mover o certificado para /etc/pki/nginx/ e a chave privada para /etc/pki/nginx/private/
- Editar o arquivo nginx.conf

(o bloco "Settings for a TLS enabled server" possui um template dessas configurações, abaixo estão incluídas somente as que necessitam modificação)
```nginx
server{
	server_name [proxy-url];
	ssl_certificate /etc/pki/nginx/[Certificate].pem;
	ssl_certificate_key /etc/pki/nginx/private/[Private-Key].pem;

	location / {
		proxy_pass https://[iis-url];
	}
}
```

#### Redirecionando tráfego HTTP para HTTPS

```nginx
server{
	listen 80;
	server_name proxyz.caneco.ad;
	return 301 https://$host$request_uri;
}
```
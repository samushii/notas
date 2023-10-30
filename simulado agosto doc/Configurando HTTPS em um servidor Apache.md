
## 1. Diretivas mínimas de configuração

```
LoadModule ssl_module modules/mod_ssl.so

Listen 443
<VirtualHost *:443>
    ServerName www.example.com
    SSLEngine on
    SSLCertificateFile "/path/to/www.example.com.cert"
    SSLCertificateKeyFile "/path/to/www.example.com.key"
</VirtualHost>
```


## Configurando redirect HTTP para HTTPS

https://stackoverflow.com/questions/16200501/how-to-automatically-redirect-http-to-https-on-apache-servers


## Referência: 

https://httpd.apache.org/docs/2.4/ssl/ssl_howto.html



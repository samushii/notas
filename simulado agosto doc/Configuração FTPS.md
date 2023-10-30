
- Instalar role FTP Server
- Requisitar certificado*
- Configurar site FTP (porta, SSL e user isolation)
- Configurar autorização básica e autenticação para os grupos específicos
- No prompt de comando:
```powershell
sc sidtype ftpsvc unrestricted
net stop ftpsvc && net start ftpsvc
```

\* uma binding ftp não utiliza o campo virtual host, a validação do certificado será feita de acordo com o registro do dns e o campo SubjectAlternativeName no próprio certificado


#### Configurando user isolation

##### Usuários de domínio

- Criar seguinte estrutura de diretórios na raíz do site FTP
	[nome-do-domínio] - disable inheritance e convert implicit permissions to explicit
		[user1] - grupo da qual user1 é membro deve ter permissão modify (This folder, subfolders and files)
		[user2] - grupo da qual user2 é membro deve ter permissão modify (This folder, subfolder and files )

##### Usuários anônimos
- Criar seguinte estrutura de diretórios na raíz do site FTP
	LocalUser - disable inheritance e convert implicit permissions to explicit
		Public - grupo Everyone deve ter permissões de leitura (This folder only)
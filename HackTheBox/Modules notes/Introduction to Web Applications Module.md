
## Atacando aplicações web

| Vulnerabilidade | Exemplo |
| - | - |
| SQL Injection | Extrair usuários do Active Directory e realizando um spraying de credenciais contra uma VPN ou portal de e-mail.
| File Inclusion | Obter acesso ao código fonte ou diretório que expõe alguma funcionalidade adicional e pode ser usado para conseguir execução remota de código | 
| Insecure Direct Object Reference (IDOR) | Quando combinada com uma vulnerabilidade do tipo Broken Access Control, pode ser usado para acessar arquivos ou funcionalidade de outro usuário. Um exemplo seria editar a url da página de configuração do seu usuário **/user/701/edit-profile** para **/user/702/edit-profile** e ganhar acesso às configurações do usuário com id 702 |
| Broken Access Control | Tomando por exemplo uma funcionalidade de registro de conta em que os parâmetros de criação são enviados pelo método POST (username=bjones&password=P@ssw0rd&roleid=3), se a aplicação for vulnerável pode ser possível mudar o parâmetro **roleid** para 0 e conseguir cadastrar um usuário novo administrador |

<br>
## Cross-Site Scripting (XSS)

a injeção de código javascript em uma página pode permitir a obtenção de informações sensíveis/abuso de funcionalidades, existem três tipos principais de XSS:

| Tipo | Descrição | 
| - | - |
| Reflected XSS | entrada fornecida pelo usuário é mostrado na página após o processamento (exemplo: resultado de pesquisa ou mensagem de erro) |
| Stored XSS | entrada fornecida pelo usuário é armazenada numa base de dados e então apresentada após ser requisitada (exemplo: publicações ou comentários) | 
| DOM XSS | entrada fornecida pelo usuário é diretamente apresentada no navegador e escrita em uma objeto HTML DOM (exemplo: objeto nome de usuário ou título da página vulneráveis à injeção de comandos) |

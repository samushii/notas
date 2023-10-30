
#### Comandos úteis

```
wmic useraccount get name,sid \\ útil para determinar quais usuários deletaram quais arquivos
```

# Windows

## Eventos

o log de eventos do windows contém informações sobre quase todos os acontecimentos do sistema, podem ser localizados na pasta **C:\\Windows\\System32\\winevt\\Logs**

usando a ferramenta **evtxcmd.exe** em conjunto com a **timeline explorer.exe** é possível agrupar pelas coluna   e conseguir um relatório detalhado dos eventos registrados

Map Description + Executable Info
Map Description + User Name 

## Link Files

arquivos de link são criados sob inúmeras circunstâncias no windows [ por exemplo quando um documento é acessado pela primeira vez ], um arquivo de link guarda informações como:

- caminho original do arquivo alvo
- tamanho do arquivo alvo
- atributos do arquivo alvo (read-only, hidden etc)
-  nome do sistema, nome do volume, número de série do volume e até mesmo o endereço MAC do sistema na qual o arquivo link existe
- informação se o recurso acessado pelo link é local ou remoto

a data de criação do link geralmente indica quando o arquivo alvo foi acessado pela primeira vez, a data de modificação indica quando o arquivo alvo foi acessado por último

caminhos principais [ arquivos .lnk podem estar em qualquer lugar do sistema ]:

- C:\\Users\\%USERNAME%\\AppData\\Roaming\\Microsoft\\Office\\Recent [ documentos do office ]
- C:\\Users\\%USERNAME%\\AppData\\Roaming\\Microsoft\\Windows\\Recent
## Jump Lists

caminhos: 
- C:\\Users\\%USERNAME%\\AppData\\Roaming\\Microsoft\\Windows\\Recent\\CustomDestinations
- C:\\Users\\%USERNAME%\\AppData\\Roaming\\Microsoft\\Windows\\Recent\\AutomaticDestinations

extensões: .automaticdestinations-ms e .customdestinations-ms

ao acessar um programa pela barra de tarefas é possível enxergar atividades recentes e atividades comuns associadas ao software em seu menu de contexto

## ShellBags

um tipo de artefato que guarda definições de exibição de diretórios, pode ser utilizado para visualizar diretórios em drives montados que nem sequer estão mais presentes no sistema, indicando que um certo usuário com certeza possuía conhecimento sobre um determinado arquivo

a ferramenta **ShellbagsExplorer.exe** do Eric Zimmerman Tools pode ser utilizada para analisar esse tipo de artefato
## Memória

### volatility2

```
imageinfo \\ informações sobre a imagem analisada [perfil a ser utilizado, horário local] 

netscan \\ informações sobre conexões de rede [ processos maliciosos tendem a realizar conexões com 
servidores externos para exfiltração de data/download de malware ]

malfind \\ mostra um hexdump do espaço de memória de um processo e também um assembly dump [ procurar por cabeçalhos (MZ, DOS) no hexdump e "sanduíches" de código assembly ]

```
## Lixeira

um arquivo deletado cria dois arquivos na pasta oculta [ dir /a para visualizá-los no terminal ], o nome desses arquivos segude o formato de 6 digitos aleatórios com um prefixo indicando o tipo do arquivo

**C:\\\$Recycle.Bin\\{SID\*}\\\$I\*\*\*\*\*\****

> contém metadados do arquivo deletado

**C:\\\$Recycle.Bin\\{SID\*}\\\$R***\*\*\*\*\**

> contém o conteúdo do arquivo deletado


## Registro

### Hives padrão

C:\\Windows\\System32\\config\\DEFAULT
C:\\Windows\\System32\\config\\SAM
C:\\Windows\\System32\\config\\SECURITY
C:\\Windows\\System32\\config\\SOFTWARE
C:\\Windows\\System32\\config\\SYSTEM

### Hives de usuário

C:\\Users\\<username\>\\NTUSER.DAT
C:\\Users\\<username\>\\AppData\\Local\\Microsoft\\Windows\\USRCLASS.DAT


## Navegadores


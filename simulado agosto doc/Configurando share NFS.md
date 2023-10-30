
## BO-WIN-SRV

- Server Manager -> Add Role -> File and Storage Services/Server for NFS
- Adicione a feature "Client for NFS"
- Crie uma pasta -> Propriedades -> Manage NFS Sharing 
	- configurar permissão (Read-only/Read-Write)
	- essa permissão é configurada por **computador** 
### Montando share

#### Windows
(necessita da feature 'Client for NFS')
- cmd > mount \\\\*share-server*\\*share-name* Z: 
#### Linux 
(necessita do pacote 'nfs-utils')
- mkdir /mnt/nfs-share
- mount -t nfs *share-server*:/*share-name*


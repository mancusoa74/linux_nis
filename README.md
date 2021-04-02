# linux_nis

Questo repository contiene i passi per installare configurare un linux come SERVER NFS e NIS, e un linux come CLIENT NFS e NIS.

E' anche presente un playbook ansible per automatizzare la creazione del CLIENT.


## SERVER

Di seguiti i passi per installare e configurare Linux come NFS e NIS server:


1. installare NFS server
```
sudo apt install nfs-kernel-server
```

2. installare NIS server e definire dominio NIS
```
sudo apt install nis
```

3. creare la directory /export/home
```
sudo mkdir -p /export/home
```

4. configurare il file /etc/exports
```
/export/home (fsid=1,sync,rw,no_subtree_check,insecure)
```

5. far ripartire NFS server 
```
sudo systemctl restart nfs-kernel-server
```

6. configurare il file /etc/yp.conf per definire il nome del server nis
```
ypserver openmediavault.local
```

7. configurare il file /etc/ypserv.securenets  per definire chi ha accesso al server
```
255.255.0.0   192.168.0.0
```
8. configurare il file /etc/deafult/nis e impostare master
```
# Are we a NIS server and if so what kind (values: false, slave, master)?
NISSERVER=master

# Are we a NIS client?
NISCLIENT=true

# Location of the master NIS password file (for yppasswdd).
# If you change this make sure it matches with /var/yp/Makefile.
YPPWDDIR=/etc

# Do we allow the user to use ypchsh and/or ypchfn ? The YPCHANGEOK
# fields are passed with -e to yppasswdd, see it's manpage.
# Possible values: "chsh", "chfn", "chsh,chfn"
YPCHANGEOK=chsh

# NIS master server.  If this is configured on a slave server then ypinit
# will be run each time NIS is started.
NISMASTER=

# Additional options to be given to ypserv when it is started.
YPSERVARGS=

# Additional options to be given to ypbind when it is started.  
YPBINDARGS=

# Additional options to be given to yppasswdd when it is started.  Note
# that if -p is set then the YPPWDDIR above should be empty.
YPPASSWDDARGS=

# Additional options to be given to ypxfrd when it is started. 
YPXFRDARGS=

```

9. restart nis service con 
```
sudo systemctl restart  nis
```

10. inizializzare il server nis
```
/usr/lib/yp/ypinit  -m 
```

11. per ogni nuovo utente aggiunto al server, fare aggiornamento tabelle
```
make -C /var/yp
```

## CLIENT

Di seguiti i passi per installare e configurare Linux come NFS e NIS client:

1. installare nis client e nfs client e specificare il dominio nis
```
sudo apt install nis
sudo apt install nfs-client
```

2. creare /export/home
```
sudo mkdir -p /export/home
```

3. configurare il file /etc/fastab
```
openmediavault.local:/export/home /export/home nfs defaults 0 0
```

4. configurare il file /etc/yp.conf
```
domain agnelli.inf.local  server openmediavault.local
```

5. configurare il file /etc/nsswitch.conf
```
passwd:         files systemd nis 
group:          files systemd nis
shadow:         files nis
gshadow:        files

hosts:          files mdns4_minimal [NOTFOUND=return] dns myhostname nis
networks:       files

protocols:      db files
services:       db files
ethers:         db files
rpc:            db files

netgroup:       nis

```

6. far ripartire i servizi nis e rpcbind
```
sudo systemctl restart rpcbind
sudo systemctl restart nis
```
7. testare la configurazione con
```
yptest
```

8. configurare il file /etc/lightdm/lightdm.conf.d/70-linuxminf.conf

aggiungere le 3 seguenti linee

```
greeter-hide-user=false
greeter-show-manual-login=true
allow-guest=false
```

9. fare il reboot del pc
```
sudo reboot
```

## ANSIBLE

il file *install_nis_client.yml* è il playbook per installare e configurare un client NFS e NIS.

Aggiorna il file inventory con i dati corretti per il tuo client.

esegui il playbook in questo modo:

```
ansible-playbook -i inventory install_nis_client.yml
```
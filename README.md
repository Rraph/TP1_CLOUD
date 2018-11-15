# TP1_CLOUD

## 1. Les préparatifs.

### 1.1 Le réseau
Tout d'abord, nous avons configuré nos cartes réseaux afin de pouvoir communiquer entre nous.
Voici nos deux adresses ip fixes :
* Ip de Paulin : 10.33.10.1/24
* Ip de Raphaël : 10.33.10.2/24

### 1.2 Le vagrantfile
Pour rajouter un disque de 10gb et une interface bridgé nous avons du modifier le VagrantFile.
Nous avons également du le modifier afin de lancer 5 vagrantfiles en même temps. Tu pourras trouver notre dockerfile [ici](sources/Vagrantfile)

## 2. Docker Swarm

### 2.1 La clause metrics-addr

Nous avons choisis de mettre en place la clause "metrics-addr" dans le vagrantfile.

Nous avons rajoutez la ligne suivante pour éxecutez le script lors du vagrant up.

```
config.vm.provision :shell, :path => "scripts/config_docker.sh", :privileged => true
```

le script :

```
#!/bin/bash

dockerd=/etc/docker
mkdir -p $dockerd

cat << EOF | tee $dockerd/daemon.json
{
    "experimental": true,
    "metrics-addr": "0.0.0.0:9323"
}
EOF
systemctl restart docker
```

Une fois ceci fait, on effectue un vagrant up. On peut voir que le script est éxecuté au demmarage.

### 2.2 Ajout des 3 managers et 2 workers.

Sur core-01 lancez la commande

```
docker swarm init --advertise-addr <IP_CURRENT_VM>
```

Vous aves donc initilisé docker swarm et Core-01 est donc le premier manager.

nous allons promouvoir core-02 et core-03 en tant que managers. Lancez la commande suivante pour générer une commande à lancer sur les deux autres managers.

```
docker swarm join-token manager
```

Sur les deux derniers core, lancez la commande suivant pour générer une nouvelle commande à rentrer dans les machines core-04 et 05.

```
docker swarm join-token worker
```

## 3. NFS

### 3.1 Prérequis

Nous avons installé un serveur CentOS 7 avec deux cartes réseaux : une host only et une NAT.

Nous avons set un nouvel hostname dans le fichier /etc/hostaname

```
[root@server ~]# cat /etc/hostname
server.domain.com
```

Ne pas oublier de mettre à jour le serveur :

```
yum update -y && yum upgrade -y
```

### 3.2 Installation de NFS Server.

Tout d'abord, nous avons installé tout les paquets nfs sur notre serveur :

```
yum install nfs-utils
```

Créons maintenant le dossier que nous allons partager :

```
mkdir /var/partage
```

et donnos nous les permissions dessus :

```
chmod -R 755 /var/partage
chown nfsnobody:nfsnobody /var/partage
```

démarrons les services nfs et profitons-en pour les activés au reboot :

```
systemctl enable rpcbind &&
systemctl enable nfs-server &&
systemctl enable nfs-lock &&
systemctl enable nfs-idmap &&
systemctl start rpcbind &&
systemctl start nfs-server &&
systemctl start nfs-lock &&
systemctl start nfs-idmap
```

Allons ajouter au fichier /etc/exports le dossier que l'on partage et l'ip des machins clientes :

```
[root@server ~]# cat /etc/exports
/var/nfsshare   192.168.10.0/24(rw,sync,no_root_squash,no_all_squash)
```

restartons le service nfs-server :
```
systemctl restart nfs-server
```

Pour terminer, ajoutons les protocoles nécessaires pour le serveur nfs :
```
firewall-cmd --permanent --zone=public --add-service=nfs
firewall-cmd --permanent --zone=public --add-service=mountd
firewall-cmd --permanent --zone=public --add-service=rpc-bind
firewall-cmd --reload
```

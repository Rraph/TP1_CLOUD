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

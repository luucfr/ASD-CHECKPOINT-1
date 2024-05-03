# ASD-CHECKPOINT-1

## Partie 1 : Gestion des utilisateurs, groupes et droits

### 1.1 Manipulation : créer un compte utilisateur

```bash
#!/bin/bash

# Créer un compte utilisateur "wilder"
sudo useradd -m -d /home/wilder -g wilder -s /bin/bash wilder

# Créer un répertoire "/home/share" et donner les droits à l'utilisateur "wilder"
sudo mkdir /home/share
sudo chown wilder:wilder /home/share
sudo chmod 700 /home/share

# Créer un fichier "passwords.txt" dans "/home/share" via le compte "wilder"
sudo -u wilder touch /home/share/passwords.txt
sudo -u wilder chmod 600 /home/share/passwords.txt

# Créer un groupe "share"
sudo groupadd share

# Ajouter l'utilisateur "wilder" au groupe "share"
sudo usermod -aG share wilder

# Ajouter le groupe "share" à ton propre compte utilisateur
sudo usermod -aG share $USER

# Mettre les droits du groupe "share" en écriture et lecture sur le répertoire "/home/share" et son contenu
sudo chmod -R g+rw /home/share

# Chiffrer le répertoire "/home/share"
sudo apt-get install -y ecryptfs-utils
sudo mount -t ecryptfs /home/share /home/share -o key=passphrase:passwd_passwd,ecryptfs_cipher=aes,ecryptfs_key_bytes=16,ecryptfs_passthrough=no,ecryptfs_enable_filename_crypto=no

# Personnaliser la session de l'utilisateur "wilder" avec un message
echo "echo '============================='" >> /home/wilder/.bashrc
echo "echo '-- Bienvenue cher Wilder --'" >> /home/wilder/.bashrc
echo "echo '============================='" >> /home/wilder/.bashrc
echo "echo '- Hostname............: \$HOSTNAME'" >> /home/wilder/.bashrc
echo "echo '- Disk Space..........: \$(df -h / | awk 'NR==2 {print \$5}')'" >> /home/wilder/.bashrc
echo "echo '- Memory used.........: \$(free -m | awk 'NR==2 {print \$3}') MB'" >> /home/wilder/.bashrc
```

### 1.2 Question : quelles sont les adresses ip propre au conteneur ?

Par défaut, Docker attribue des adresses IP de la plage `172.17.0.0/16` aux conteneurs.

## Partie 2 : Sécurité du conteneur et durcissement ssh

### 2.1 Manipulation : sécuriser l'environnement distant

```bash
#!/bin/bash

# Nom ou ID du conteneur Docker
CONTAINER_NAME="nom_du_conteneur"

# Nouveau port SSH
NEW_SSH_PORT="2222"

# Modifier le port SSH du conteneur
docker exec $CONTAINER_NAME sed -i "s/#Port 22/Port $NEW_SSH_PORT/" /etc/ssh/sshd_config

# Redémarrer le service SSH dans le conteneur
docker exec $CONTAINER_NAME service ssh restart

# Bloquer tous les ports du conteneur avec un pare-feu logiciel (iptables)
docker exec $CONTAINER_NAME iptables -P INPUT DROP
docker exec $CONTAINER_NAME iptables -P OUTPUT DROP
docker exec $CONTAINER_NAME iptables -P FORWARD DROP

# Autoriser l'accès depuis le nouveau port SSH
docker exec $CONTAINER_NAME iptables -A INPUT -p tcp --dport $NEW_SSH_PORT -j ACCEPT
docker exec $CONTAINER_NAME iptables -A OUTPUT -p tcp --sport $NEW_SSH_PORT -j ACCEPT
docker exec $CONTAINER_NAME iptables -A OUTPUT -p tcp --dport $NEW_SSH_PORT -j ACCEPT
docker exec $CONTAINER_NAME iptables -A INPUT -p tcp --sport $NEW_SSH_PORT -j ACCEPT
```

### 2.2 Question : quel autre moyen simple peux-tu mettre en œuvre pour augmenter la sécurité du conteneur ?

Un autre moyen simple pour augmenter la sécurité du conteneur est de maintenir les logiciels et le système d'exploitation à jour, en appliquant régulièrement les correctifs de sécurité.

## Partie 3 : Scripting Bash

### 3.1 Manipulation : script bash de connexion distante

```bash
#!/bin/bash

# Mot de passe pour l'authentification SSH
SSH_PASSWORD="votre_mot_de_passe"

# Nom d'utilisateur et adresse IP du conteneur
CONTAINER_USER="wilder"
CONTAINER_IP="adresse_ip_du_conteneur"

sshpass -p "$SSH_PASSWORD" ssh "$CONTAINER_USER@$CONTAINER_IP"
```

### 3.2 Question : Qu'est censé faire le script Bash ci-dessous ?

Ce script surveille l'utilisation du processeur. Si l'utilisation du processeur dépasse 95 %, il envoie un e-mail à l'adresse spécifiée, indiquant que l'ordinateur est surchargé

### 3.2 Question : Est-ce que l'utilisateur "wilder" va pouvoir installer des paquets logiciels tels que apache ou nginx ? Que la réponse soit oui ou non, expliquer pourquoi en quelques mots

Non, l'utilisateur "wilder" ne pourra pas installer des paquets logiciels tels que Apache ou Nginx sans privilèges administratifs. Cela est dû au fait que l'installation de nouveaux paquets logiciels nécessite des droits d'administration sur le système. L'utilisateur "wilder" n'ayant pas ces privilèges, il ne pourra pas effectuer l'installation de ces paquets.

## Partie 4 : 10 questions sur le thème DevOps

### 4.1 Qu'est-ce que l'infrastructure as code (IaC) ?

L'infrastructure as code (IaC) est une méthode de gestion de l'infrastructure informatique à l'aide de fichiers de configuration textuels ou de scripts, permettant d'automatiser le déploiement, la gestion et la mise à jour de l'infrastructure.

### 4.2 Est-ce que Docker est une nécessité dans le milieu DevOps ? (Expliquer la réponse)

Non, Docker n'est pas une nécessité absolue en DevOps, mais il est souvent utilisé pour sa capacité à créer des environnements de développement cohérents et pour faciliter le déploiement d'applications.

### 4.3 Qu'est-ce qu'une pipeline CI/CD ?

Une pipeline CI/CD est un ensemble d'outils et de processus automatisés utilisés pour intégrer, tester et déployer régulièrement les changements de code dans un logiciel.

### 4.4 Quel outil (logiciel) utiliserais tu pour gérer des configurations serveurs à distance ?

J'utiliserais Ansible pour gérer des configurations serveurs à distance.

### 4.5 Que signifie le terme "scalabilité" pour le milieu DevOps ?

En DevOps, "scalabilité" signifie la capacité d'une infrastructure ou d'une application à s'adapter de manière efficace à une augmentation ou une diminution de la charge de travail, tout en maintenant des performances optimales.

### 4.6 Quel est le principal rôle d'un administrateur système DevOps en entreprise ?

Le principal rôle d'un administrateur système DevOps est de garantir le bon fonctionnement et la sécurité de l'infrastructure, tout en favorisant la collaboration entre les équipes de développement et d'exploitation.

### 4.7 Quel outil (plateforme) utiliserais tu pour créer une pipeline de déploiement logiciel ?

Pour créer une pipeline de déploiement logiciel, j'utiliserais Jenkins.

## 4.8 Quels types d'environnements mettrais tu en place avant une mise en production de logiciel ?

Avant une mise en production de logiciel, je mettrais en place les environnements suivants :

- Environnement de développement
- Environnement de test
- Environnement de pré-production (staging)

### 4.9 Qu'est-ce que signifie exactement la notion d'intégration continue (CI) ?

La pratique de l'intégration continue implique la fusion régulière des modifications de code dans un référentiel partagé, suivie de l'exécution automatique de tests pour garantir la qualité du code et détecter les erreurs précocement.

### 4.10 Que signifie la notion de "provisionning" pour un administrateur système DevOps ?

Le provisionning pour un administrateur système DevOps signifie le déploiement automatique et la configuration cohérente des ressources informatiques.

## Partie 5 : Administration système et réseau

### 5.1 Manipulation : installer un outil de surveillance de l'activité réseau d'un conteneur

```bash
#!/bin/bash

# Se connecter au conteneur distant
ssh -t user@adresse_du_conteneur 'sudo su'

# Installe iftop
apt-get update
apt-get install -y iftop

# Monitore l'interface "eth0" avec iftop
iftop -i eth0
```
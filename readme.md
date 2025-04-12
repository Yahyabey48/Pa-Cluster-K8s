# Mise en place d'un Cluster Kubernetes (K8s) avec Ansible

Ce projet a pour objectif de déployer un cluster Kubernetes (K8s) en utilisant un script Ansible. Le cluster sera composé de :

- **1 nœud master**
- **2 nœuds workers**

## Prérequis

Avant de commencer, assurez-vous d'avoir les éléments suivants :

- Ansible installé sur votre machine.
- Accès SSH configuré pour les nœuds du cluster.
- Les machines cibles doivent avoir une distribution Linux compatible avec Kubernetes.

## Étapes de déploiement

1. **Configurer l'inventaire Ansible** : Définissez les adresses IP des nœuds dans le fichier d'inventaire.
2. **Exécuter le script Ansible** : Lancez le script pour installer et configurer Kubernetes sur les nœuds.
3. **Vérifier le cluster** : Utilisez `kubectl` pour vérifier que le cluster est opérationnel.

## Structure du projet

- `ansible/` : Contient les playbooks et les rôles Ansible nécessaires pour le déploiement.
- `inventory/` : Fichier d'inventaire Ansible avec les informations des nœuds.

## Commandes utiles

- Lancer le playbook principal :
  ```bash
  ansible-playbook -i inventory/hosts ansible/main.yml


Verification :  kubectl get nodes 


Ajoutez ce contenu dans votre fichier [readme.md](http://_vscodecontentref_/1).

## Préparation des pré-requis

Insallation d'Ansible : 

Debian 12 (Bookworm)

->

Ubuntu 22.04 (Jammy)

jammy

$ UBUNTU_CODENAME=jammy
$ wget -O- "https://keyserver.ubuntu.com/pks/lookup?fingerprint=on&op=get&search=0x6125E2A8C77F2818FB7BD15B93C4A3FD7BB9C367" | sudo gpg --dearmour -o /usr/share/keyrings/ansible-archive-keyring.gpg
$ echo "deb [signed-by=/usr/share/keyrings/ansible-archive-keyring.gpg] http://ppa.launchpad.net/ansible/ansible/ubuntu $UBUNTU_CODENAME main" | sudo tee /etc/apt/sources.list.d/ansible.list
$ sudo apt update && sudo apt install ansible
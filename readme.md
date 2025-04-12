# Mise en place d'un Cluster Kubernetes (K8s) avec Ansible

Ce projet déploie un cluster Kubernetes complet sur des machines Debian 12 (Bookworm) en utilisant Ansible pour automatiser l'ensemble du processus d'installation et de configuration.

---

## Architecture du Cluster

- **1 nœud master** (`k8s-master`) : Gère le plan de contrôle Kubernetes  
- **2 nœuds workers** (`k8s-node1`, `k8s-node2`) : Exécutent les charges de travail applicatives  

---

## Fonctionnalités implémentées

- ✅ Installation complète de Docker avec configuration optimisée  
- ✅ Configuration du noyau Linux pour Kubernetes (modules, sysctl)  
- ✅ Désactivation du SWAP (requis pour Kubernetes)  
- ✅ Installation de Kubelet, Kubeadm et Kubectl  
- ✅ Initialisation du cluster sur le nœud master  
- ✅ Déploiement de Cilium comme plugin CNI (Container Network Interface)  
- ✅ Jonction automatique des nœuds workers au cluster  

---

## Structure du projet

### Fichiers principaux

1. **`setup-playbook.yaml`**  
   Ce fichier contient le playbook Ansible principal pour configurer les nœuds, installer Kubernetes et Docker, et déployer le cluster.  
   - Nettoyage des configurations Docker existantes.  
   - Installation des dépendances nécessaires.  
   - Configuration des modules noyau et des paramètres sysctl.  
   - Initialisation du cluster Kubernetes sur le nœud master.  
   - Déploiement de Cilium comme plugin réseau.  

2. **`hosts`**  
   Fichier d'inventaire Ansible définissant les groupes de nœuds (`master` et `workers`).  
   Exemple :  
   ```ini
   [master]
   k8s-master ansible_host=192.168.1.166 ansible_user=ansible

   [workers]
   k8s-node1 ansible_host=192.168.1.153 ansible_user=ansible
   k8s-node2 ansible_host=192.168.1.124 ansible_user=ansible

   [k8s-cluster:children]
   master
   workers
   ```

3. **`README.md`**  
   Documentation complète du projet, incluant les étapes de déploiement, les prérequis, et les commandes utiles pour la maintenance.

---

## Prérequis

### Sur la machine de contrôle

- **Ansible** (version 2.9+)  
- **Accès SSH** configuré avec authentification par clé vers tous les nœuds  

### Sur les nœuds cibles

- **Debian 12 (Bookworm)** fraîchement installé  
- **Accès root** ou utilisateur avec privilèges sudo  
- **Ports réseau** ouverts entre les nœuds (6443, 10250, etc.)  

---

## Installation d'Ansible sur Debian 12

```bash
# Configuration du dépôt Ansible pour Debian via Ubuntu Jammy
UBUNTU_CODENAME=jammy
wget -O- "https://keyserver.ubuntu.com/pks/lookup?fingerprint=on&op=get&search=0x6125E2A8C77F2818FB7BD15B93C4A3FD7BB9C367" | sudo gpg --dearmour -o /usr/share/keyrings/ansible-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/ansible-archive-keyring.gpg] http://ppa.launchpad.net/ansible/ansible/ubuntu $UBUNTU_CODENAME main" | sudo tee /etc/apt/sources.list.d/ansible.list
sudo apt update && sudo apt install ansible
```

---

## Préparation des nœuds Kubernetes

Avant d'exécuter le playbook Ansible, vous devez préparer les nœuds cibles avec un utilisateur approprié et configurer l'accès SSH.

### 1. Créer l'utilisateur Ansible sur chaque nœud

Se connecter en root à chaque nœud (master et workers) et créer l'utilisateur `ansible` :

```bash
# Création de l'utilisateur ansible
adduser ansible
# Ajouter l'utilisateur au groupe sudo
usermod -aG sudo ansible
# Configurer sudo sans mot de passe pour l'utilisateur ansible
echo "ansible ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/ansible
```

### 2. Configurer l'authentification par clé SSH

Sur votre machine de contrôle Ansible, générez une paire de clés SSH si ce n'est pas déjà fait :

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
```

Ensuite, copiez la clé publique vers chaque nœud :

```bash
# Pour chaque nœud (remplacez USER et HOSTNAME par les valeurs appropriées)
ssh-copy-id -i ~/.ssh/id_rsa.pub ansible@k8s-master
ssh-copy-id -i ~/.ssh/id_rsa.pub ansible@k8s-node1
ssh-copy-id -i ~/.ssh/id_rsa.pub ansible@k8s-node2
```

### 3. Configurer les noms d'hôtes

Sur chaque nœud, définissez le nom d'hôte approprié :

```bash
# Sur le nœud master
hostnamectl set-hostname k8s-master

# Sur le premier nœud worker
hostnamectl set-hostname k8s-node1

# Sur le deuxième nœud worker
hostnamectl set-hostname k8s-node2
```

### 4. Configurer le fichier `/etc/hosts` sur chaque nœud

Ajoutez les entrées suivantes au fichier `/etc/hosts` sur chaque nœud :

```bash
sudo bash -c 'cat >> /etc/hosts << EOF
192.168.1.166 k8s-master
192.168.1.153 k8s-node1
192.168.1.124 k8s-node2
EOF'
```

Remplacez les adresses IP par celles correspondant à votre environnement.

---

## Étapes de déploiement

1. Cloner ce dépôt

```bash
git clone <URL_du_repo>
cd Pa-Cluster-K8s
```

2. Configurer le fichier d'inventaire `hosts`

3. Exécuter le playbook Ansible

```bash
ansible-playbook -i hosts setup-playbook.yaml
```

4. Vérifier l'état du cluster

```bash
kubectl get nodes
kubectl get pods -A
```

---

## Commandes utiles pour la maintenance

```bash
# Vérifier l'état des nœuds
kubectl get nodes -o wide

# Vérifier les pods système
kubectl get pods -n kube-system

# Vérifier l'état de Cilium
kubectl -n kube-system get pods -l k8s-app=cilium

# Afficher les journaux des pods Cilium
kubectl -n kube-system logs -l k8s-app=cilium

# Obtenir des informations détaillées sur un nœud
kubectl describe node <nom-du-nœud>
```

---

## Références

- [Documentation officielle Kubernetes](https://kubernetes.io/docs/home/)  
- [Documentation Cilium](https://docs.cilium.io/)  
- [Documentation Ansible](https://docs.ansible.com/)  
- [Guide d'installation de Docker](https://docs.docker.com/engine/install/)  

---

**Note** : Ce projet est en cours de développement. Les contributions et suggestions sont les bienvenues !

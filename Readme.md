# Mini-Projet Ansible : Déploiement d'une Application

## Architecture du Projet

Le projet comporte deux architectures différentes :

### **Partie 1 : Déploiement Classique Nginx**
```
┌─────────────────────────────────┐
│    Serveur (10.0.37.5)          │
├─────────────────────────────────┤
│          Nginx                   │
│  (Port 80)                       │
│  ├─ index.html (généré)          │
│  └─ Fichiers statiques           │
└─────────────────────────────────┘
```

### **Partie 2 : Architecture Docker avec Proxy Inverse**
```
┌──────────────────────────────────────────────────┐
│        Serveur (10.0.37.5)                       │
├──────────────────────────────────────────────────┤
│                                                   │
│  ┌────────────────────────────────────────────┐ │
│  │    Docker Network: custom_net              │ │
│  ├────────────────────────────────────────────┤ │
│  │                                            │ │
│  │  ┌─────────────────┐  ┌──────────────┐    │ │
│  │  │  Nginx Proxy    │  │  Webapp1     │    │ │
│  │  │  (Port 80)  ────┼──→ (Port 80)    │    │ │
│  │  └─────────────────┘  └──────────────┘    │ │
│  │                                            │ │
│  └────────────────────────────────────────────┘ │
│         ↓                                       │
│  Webapp1 Port 8082 (accès direct)              │
└──────────────────────────────────────────────────┘
```

---

## Structure du Projet

```
mini-projet-ansible/
├── app-init/                       ← PARTIE 1 : Nginx simple
│   ├── ansible.cfg
│   ├── hosts
│   ├── nginx_playbook.yaml
│   ├── group_vars/all.yml
│   ├── host_vars/client1.yml
│   ├── templates/                  # Templates Jinja2
│   ├── vars/logos.yaml
│   └── files/playbook_stacker.zip
│
└── app-template/                   ← PARTIE 2 : Docker + Rôles
    ├── ansible.cfg
    ├── hosts
    ├── nginx_webapp_playbook.yaml
    ├── group_vars/all.yml
    ├── host_vars/client1.yml
    ├── nginx/                      # Rôle Nginx Proxy
    │   ├── tasks/main.yml
    │   ├── tasks/nginx.conf
    │   ├── defaults/main.yml
    │   └── vars/main.yml
    └── webapp/                     # Rôle Application Web
        ├── tasks/main.yml
        ├── templates/
        ├── defaults/main.yml
        └── vars/main.yml
```

---

## Étapes pour Déployer

### **Prérequis - Harmonisation des Versions**

⚠️ **IMPORTANT** : Les versions d'Ansible, Python et Docker sur la machine cliente doivent être harmonisées pour éviter les problèmes de compatibilité.

#### **Versions recommandées sur le serveur cible (10.0.37.5)**

```bash
# Vérifier les versions installées
python3 --version       # Minimum Python 3.6
ansible --version       # Minimum Ansible 2.9
docker --version        # Minimum Docker 19.0
```

**Tableau de compatibilité** :

| Ansible | Python | Docker | Status |
|---------|--------|--------|--------|
| 2.9.x   | 2.7    | Quelconque | ❌ Non supporté (manque de modules) |
| 2.9.x   | 3.6+   | 19.0+  | ✅ Supporté |
| 2.10+   | 3.6+   | 19.0+  | ✅ Recommandé |
| 3.x+    | 3.8+   | 20.0+  | ✅ Optimal |

#### **Installation recommandée sur le serveur cible**

**Sur CentOS 7/8** :
```bash
# Mettre à jour Python
sudo yum install -y python3 python3-pip

# Mettre à jour Ansible
sudo pip3 install --upgrade ansible

# Vérifier les versions
python3 --version
ansible --version
```

**Sur Ubuntu 20.04+** :
```bash
# Mettre à jour les paquets
sudo apt-get update
sudo apt-get install -y python3 python3-pip

# Mettre à jour Ansible
sudo pip3 install --upgrade ansible

# Vérifier les versions
python3 --version
ansible --version
```

### **Machine locale**

1. **Installer Ansible avec Python 3**
```bash
pip3 install ansible
```

2. **Vérifier la version**
```bash
ansible --version
# Doit afficher : Python 3.x.x
```

### **Serveurs cibles** (Ubuntu ou CentOS)

Vérifier que les versions sont cohérentes :
```bash
# Python 3 (minimum 3.6)
ssh admin@10.0.37.5 "python3 --version"

# Ansible 2.9+ 
ssh admin@10.0.37.5 "ansible --version"

# Docker 19.0+
ssh admin@10.0.37.5 "docker --version"
```

### **Partie 1 : Déploiement Nginx Simple**

```bash
# 1. Entrer dans le répertoire
cd app-init

# 2. Tester la connectivité
ansible -i hosts prod -m ping

# 3. Exécuter le playbook
ansible-playbook -i hosts nginx_playbook.yaml

# 4. Vérifier le résultat
curl http://10.0.37.5
```

### **Partie 2 : Déploiement Docker**

#### **Avant de commencer, installer Docker sur le serveur**

**Sur CentOS :**
```bash
ssh admin@10.0.37.5 "
  sudo yum install -y docker
  sudo systemctl start docker
  sudo usermod -aG docker admin
  sudo yum install python3-pip -y
"
```

**Sur Ubuntu :**
```bash
ssh admin@10.0.37.5 "
  sudo apt-get update
  sudo apt-get install -y docker.io
  sudo systemctl start docker
  sudo usermod -aG docker admin
  pip3 install docker
"
```
Harmoniser la version docker et python
sudo pip3 install docker requests
#### **Déployer avec Ansible**

```bash
# 1. Entrer dans le répertoire
cd ../app-template

# 2. Tester la connectivité
ansible -i hosts prod -m ping

# 3. Exécuter le playbook
ansible-playbook -i hosts nginx_webapp_playbook.yaml

# 4. Vérifier les conteneurs
ansible -i hosts prod -m command -a "docker ps"

# 5. Tester le proxy Nginx
curl http://10.0.37.5

# 6. Tester la webapp directe
curl http://10.0.37.5:8082
```

---

## Vérification Rapide

**Partie 1 (Nginx)** :
```bash
cd app-init
ansible -i hosts prod -m command -a "systemctl status nginx | head -3"
ansible -i hosts prod -m command -a "curl -I http://127.0.0.1 | head -1"
```

**Partie 2 (Docker)** :
```bash
cd ../app-template
ansible -i hosts prod -m command -a "docker ps --format 'table {{.Names}}\t{{.Status}}'"
ansible -i hosts prod -m command -a "curl -I http://127.0.0.1:8082 | head -1"
```

---

---

## Dépannage des Problèmes de Version

### **Erreur : "Unable to import community.docker.docker_network"**

**Cause** : Ansible 2.9 avec Python 2.7 ne supporte pas les modules community.docker

**Solution** :
```bash
# Sur le serveur cible
ssh admin@10.0.37.5 "
  sudo yum install -y python3 python3-pip
  sudo pip3 install --upgrade ansible
  ansible --version  # Doit afficher Python 3.x
"
```

### **Erreur : "Timeout when waiting for localhost:8082"**

**Cause** : Le conteneur Docker met trop longtemps à démarrer

**Solutions** :
```bash
# 1. Vérifier que Docker est bien installé et actif
ssh admin@10.0.37.5 "docker ps"

# 2. Vérifier les logs du conteneur
ssh admin@10.0.37.5 "docker logs webapp1"

# 3. Augmenter le timeout dans app-template/webapp/tasks/main.yml
# Chercher "wait_for" et augmenter le timeout à 120 secondes
```

### **Vérifier la compatibilité complète**

```bash
# Script de vérification complet
ssh admin@10.0.37.5 "
  echo '=== Python ==='
  python3 --version
  echo '=== Ansible ==='
  ansible --version
  echo '=== Docker ==='
  docker --version
  echo '=== Docker daemon ==='
  docker ps
"
```

---

## Configuration des Serveurs

Les deux parties utilisent l'inventaire du fichier `hosts` :
```
[prod]
client1 ansible_host=10.0.37.5
```

Les variables de connexion se trouvent dans `group_vars/all.yml` :
```yaml
ansible_user: admin
ansible_password: admin
ansible_sudo_pass: admin
```



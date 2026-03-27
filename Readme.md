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

### **Prérequis**

1. **Machine locale** : Installer Ansible
```bash
pip install ansible
```

2. **Serveurs cibles** (Ubuntu ou CentOS)
```bash
# Doit avoir Python 3, SSH et sudo d'activés
ssh admin@10.0.37.5 "python3 --version"
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



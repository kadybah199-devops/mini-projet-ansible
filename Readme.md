# Mini-Projet Ansible : Déploiement d'une Application 🚀

## 📋 Table des matières
- [Vue d'ensemble](#vue-densemble)
- [Architecture](#architecture)
- [Prérequis](#prérequis)
- [Structure du projet](#structure-du-projet)
- [Partie 1 : Déploiement simple](#partie-1--déploiement-simple)
- [Partie 2 : Déploiement Docker](#partie-2--déploiement-docker)
- [Instructions complètes étape par étape](#instructions-complètes-étape-par-étape)
- [Vérification et validation](#vérification-et-validation)
- [Troubleshooting](#troubleshooting)

---

## 🎯 Vue d'ensemble

Ce projet Ansible déploie une application web en deux étapes progressives :

### **Partie 1 : Déploiement Classique**
Installation de Nginx avec génération dynamique du contenu HTML via des templates Jinja2.

### **Partie 2 : Déploiement Conteneurisé**
Utilisation de Docker avec des rôles Ansible pour déployer :
- Un conteneur Nginx comme **proxy inverse**
- Un conteneur avec l'application (webapp)
- Configuration d'un réseau personnalisé Docker

---

## 🏗️ Architecture

### **Partie 1 : Architecture Simple**
```
┌─────────────────────────────────┐
│      Serveur Client1 (10.0.37.5)│
├─────────────────────────────────┤
│          Nginx                   │
│  (Port 80)                       │
│  ├─ index.html (généré)          │
│  └─ playbook_stacker.zip         │
└─────────────────────────────────┘
```

### **Partie 2 : Architecture Docker avec Proxy Inverse**
```
┌─────────────────────────────────────────────────────┐
│        Serveur Client1 (10.0.37.5)                  │
├─────────────────────────────────────────────────────┤
│                                                      │
│  ┌─────────────────────────────────────────────┐   │
│  │        Docker Network: custom_net            │   │
│  ├─────────────────────────────────────────────┤   │
│  │                                              │   │
│  │  ┌──────────────────┐  ┌────────────────┐   │   │
│  │  │  Nginx Container │  │ Webapp1 Cont.  │   │   │
│  │  │  (Proxy Inverse) │  │  (Nginx Image) │   │   │
│  │  │                  │  │                │   │   │
│  │  │  :80 ──┬─────────┼──→ :80            │   │   │
│  │  │        │ Upstream│  │                │   │   │
│  │  │        │(webapp1:80) │                │   │   │
│  │  └──────────────────┘  └────────────────┘   │   │
│  │                                              │   │
│  └─────────────────────────────────────────────┘   │
│                   ↓                                │
│          Port 8082 (webapp1)                      │
└─────────────────────────────────────────────────────┘
```

---

## 📦 Prérequis

### **Environnement**
- **Système d'exploitation** : Windows 10/11, Linux ou macOS
- **Vagrant** : Pour créer les VMs (recommandé)
- **VirtualBox** : Hyperviseur pour Vagrant

### **Logiciels requis**

#### **Localement (sur votre machine)**
```bash
# Installer Ansible
pip install ansible
# ou
sudo apt-get install ansible  # Ubuntu/Debian
brew install ansible          # macOS
```

#### **Sur les serveurs cibles (VMs)**
- **Système d'exploitation** : CentOS 7/8 ou Ubuntu 20.04+
- **Python 3** : Pré-requis Ansible
- **SSH** : Accès à distance
- **sudo** : Permissions d'administration
- **Docker** : Pour la Partie 2 uniquement

### **Accès SSH**

Fichier de configuration SSH `~/.ssh/config` (optionnel) :
```
Host client1
    HostName 10.0.37.5
    User admin
    IdentityFile ~/.ssh/id_rsa
    StrictHostKeyChecking no
```

---

## 📁 Structure du projet

```
mini-projet-ansible/
├── README_COMPLET.md              ← Ce fichier
├── Readme.md                       ← Énoncé du projet
├── CORRECTIONS.md                  ← Détail des corrections apportées
│
├── app-init/                       ← PARTIE 1 : Déploiement Simple
│   ├── ansible.cfg                 # Configuration Ansible
│   ├── hosts                       # Inventaire des hosts
│   ├── nginx_playbook.yaml         # Playbook principal
│   ├── group_vars/
│   │   └── all.yml                 # Variables communes
│   ├── host_vars/
│   │   ├── client1.yml             # Config spécifique client1
│   │   └── client2.yml             # Config spécifique client2
│   ├── templates/                  # Templates Jinja2
│   │   ├── index.html-base.j2
│   │   ├── index.html-easter_egg.j2
│   │   ├── index.html-logos.j2
│   │   ├── index.html-ansible_managed.j2
│   │   └── index.html.j2
│   ├── vars/
│   │   └── logos.yaml              # Variables logos (base64)
│   └── files/                      # Fichiers à copier
│       └── playbook_stacker.zip    # Archive application
│
└── app-template/                   ← PARTIE 2 : Déploiement Docker
    ├── ansible.cfg
    ├── hosts
    ├── nginx_webapp_playbook.yaml  # Playbook avec rôles
    ├── group_vars/
    │   └── all.yml
    ├── host_vars/
    │   ├── client1.yml
    │   └── client2.yml
    │
    ├── nginx/                      ← RÔLE : Proxy Nginx
    │   ├── defaults/main.yml       # Variables par défaut
    │   ├── vars/main.yml           # Variables constantes
    │   ├── handlers/main.yml       # Handlers (notifications)
    │   ├── tasks/main.yml          # Tâches du rôle
    │   ├── tasks/nginx.conf        # Configuration Nginx
    │   ├── meta/main.yml           # Métadonnées rôle
    │   └── tests/                  # Tests (optionnel)
    │
    └── webapp/                     ← RÔLE : Application Web
        ├── defaults/main.yml       # Variables par défaut
        ├── vars/main.yml           # Variables constantes
        ├── handlers/main.yml       # Handlers
        ├── tasks/main.yml          # Tâches déploiement Docker
        ├── templates/              # Templates Jinja2
        │   ├── index.html-base.j2
        │   ├── index.html-logos.j2
        │   ├── index.html-easter_egg.j2
        │   ├── index.html-ansible_managed.j2
        │   └── index.html.j2
        ├── meta/main.yml
        └── tests/
```

---

## 🔧 Configuration initiale

### **Étape 1 : Préparer les VMs**

Créez 2 VMs avec :
- **Client1** : IP `10.0.37.5` (CentOS ou Ubuntu)
- **Client2** : IP `10.0.37.6` (optionnel)

Utilisateur : `admin` / Mot de passe : `admin`

### **Étape 2 : Vérifier la connectivité**

```bash
# Tester la connexion SSH à client1
ssh admin@10.0.37.5 "python3 --version"

# Vous devez voir : Python 3.x.x
```

### **Étape 3 : Tester Ansible**

```bash
# Faire un ping Ansible
ansible -i app-init/hosts prod -m ping

# Résultat attendu :
# client1 | SUCCESS => {
#     "ping": "pong"
# }
```

---

## ✨ PARTIE 1 : Déploiement Simple

### **Objectif**
Installer Nginx et générer dynamiquement une page HTML avec Ansible.

### **Étapes détaillées**

#### **Étape 1.1 : Naviguer vers app-init**
```bash
cd app-init
```

#### **Étape 1.2 : Examiner la configuration**

**Fichier `hosts`** - Inventaire :
```yaml
all:
  children:
    prod:
      hosts:
        client1:
          ansible_host: 10.0.37.5
    staging:
      hosts:
        client2:
          ansible_host: 10.0.37.6
```

**Fichier `group_vars/all.yml`** - Credentials communes :
```yaml
ansible_user: admin          # Utilisateur SSH
ansible_password: admin      # Mot de passe
ansible_sudo_pass: admin     # Mot de passe sudo
```

**Fichier `host_vars/client1.yml`** - Config spécifique :
```yaml
ansible_host: 10.0.37.5
```

#### **Étape 1.3 : Analyser le playbook**

**Fichier `nginx_playbook.yaml`** - Flux d'exécution :

```yaml
- hosts: prod              # Cible : groupe 'prod'
  become: true             # Utiliser sudo
  vars_files:
    - vars/logos.yaml      # Charger les logos
  tasks:
    # 1. Définir le répertoire root de Nginx selon la distribution
    - name: Définir nginx_root_location pour RedHat
      set_fact:
        nginx_root_location: "/usr/share/nginx/html"
      when: ansible_os_family == "RedHat"
    
    # 2. Même chose pour Debian/Ubuntu
    - name: Définir nginx_root_location pour Ubuntu
      set_fact:
        nginx_root_location: "/var/www/html"
      when: ansible_os_family == "Debian"
    
    # 3. Installer EPEL (Extended Package) - CentOS uniquement
    - name: Install EPEL
      yum:
        name: epel-release
      when: ansible_distribution == 'CentOS'
    
    # 4. Installer Nginx
    - name: Install Nginx
      package:
        name: nginx
        state: latest
    
    # 5. Redémarrer Nginx
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
      notify: Check HTTP Service  # Déclencher le handler
    
    # 6. Générer index.html depuis le template
    - name: Template index.html
      template:
        src: index.html-easter_egg.j2
        dest: "{{ nginx_root_location }}/index.html"
    
    # 7. Installer unzip
    - name: Install unzip
      package:
        name: unzip
    
    # 8. Extraire l'archive playbook_stacker.zip
    - name: Unarchive application
      unarchive:
        src: playbook_stacker.zip
        dest: "{{ nginx_root_location }}"
  
  handlers:
    # Handler déclenché après redémarrage de Nginx
    - name: Check HTTP Service
      uri:
        url: http://{{ ansible_default_ipv4.address }}
        status_code: 200
```

#### **Étape 1.4 : Valider la syntaxe**
```bash
ansible-playbook --syntax-check -i hosts nginx_playbook.yaml

# Résultat attendu :
# playbook: nginx_playbook.yaml
```

#### **Étape 1.5 : Exécuter le playbook**
```bash
ansible-playbook -i hosts nginx_playbook.yaml -v

# Vous verrez :
# TASK [Install Nginx] ...
# TASK [Restart nginx] ...
# RUNNING HANDLER [Check HTTP Service] ...
```

### **Étape 1.6 : Vérifier le résultat**

```bash
# Tester Nginx localement
curl http://10.0.37.5

# Vous devez voir du HTML avec des logos

# Vérifier le status de Nginx
ansible -i hosts prod -m command -a "systemctl status nginx | head -5"

# Vérifier le fichier index.html
ansible -i hosts prod -m command -a "cat /usr/share/nginx/html/index.html | head -20"
```

---

## 🐳 PARTIE 2 : Déploiement Docker avec Rôles

### **Objectif**
Déployer une application dans Docker avec Nginx comme proxy inverse utilisant les rôles Ansible.

### **Étapes détaillées**

#### **Étape 2.1 : Naviguer vers app-template**
```bash
cd ../app-template
```

#### **Étape 2.2 : Vérifier que Docker est installé**

Pré-installation sur le host (exécuté manuellement avant Ansible) :
```bash
# Sur CentOS
ssh admin@10.0.37.5 "sudo yum install -y docker"
ssh admin@10.0.37.5 "sudo systemctl start docker"
ssh admin@10.0.37.5 "sudo usermod -aG docker admin"

# Sur Ubuntu
ssh admin@10.0.37.5 "sudo apt-get install -y docker.io"
ssh admin@10.0.37.5 "sudo systemctl start docker"
ssh admin@10.0.37.5 "sudo usermod -aG docker admin"
```

#### **Étape 2.3 : Examiner la configuration**

**Fichier `group_vars/all.yml`** - Credentials :
```yaml
ansible_user: admin
ansible_password: admin
ansible_sudo_pass: admin
ansible_python_interpreter: "/usr/bin/python3"
```

**Fichier `host_vars/client1.yml`** - Config du host :
```yaml
ansible_host: 10.0.37.5
```

#### **Étape 2.4 : Comprendre la structure des rôles**

#### **Rôle `nginx` : Proxy Inverse**

**Fichier `nginx/defaults/main.yml`** - Variables par défaut :
```yaml
# Noms et images Docker
docker_image: nginx
docker_image_tag: latest

# Configuration réseau
custom_net: custom_net
nginx_port: 80

# Upstream backend
upstream_server: webapp1:80
```

**Fichier `nginx/tasks/main.yml`** - Tâches :
```yaml
- name: Create a custom Docker network
  community.docker.docker_network:
    name: custom_net
    driver: bridge

- name: Pull Nginx Docker image
  docker_image:
    name: nginx
    tag: latest
    source: pull

- name: Copy Nginx configuration
  ansible.builtin.copy:
    src: nginx.conf
    dest: /tmp/nginx.conf
    owner: root
    mode: '0644'

- name: Run Nginx container
  docker_container:
    name: nginx
    image: nginx:latest
    state: started
    ports:
      - "80:80"
    networks:
      - name: custom_net
    volumes:
      - /tmp/nginx.conf:/etc/nginx/nginx.conf:ro
```

**Fichier `nginx/tasks/nginx.conf`** - Configuration Nginx :
```nginx
user nginx;
worker_processes auto;

http {
    upstream webapp_backend {
        server webapp1:80;  # Référence le conteneur webapp
    }

    server {
        listen 80;
        server_name localhost;

        location / {
            proxy_pass http://webapp_backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

#### **Rôle `webapp` : Application Web**

**Fichier `webapp/defaults/main.yml`** - Variables par défaut :
```yaml
container_name: "webapp"           # Nom du conteneur
app_port: 8080                     # Port exposé
docker_network: "custom_net"       # Réseau Docker
docker_image: "nginx"              # Image Docker
docker_image_tag: "latest"         # Version image
template_index: "index.html-base.j2" # Template à utiliser
```

**Fichier `webapp/tasks/main.yml`** - Tâches de déploiement :
```yaml
- name: Pull Docker image
  docker_image:
    name: "{{ docker_image }}"
    tag: "{{ docker_image_tag }}"
    source: pull

- name: Create application directory
  file:
    path: /app
    state: directory
    mode: '0755'

- name: Generate index.html from template
  template:
    src: "{{ template_index }}"
    dest: "/app/index.html"
    mode: '0644'

- name: Run Docker container for webapp
  docker_container:
    name: "{{ container_name }}"
    image: "{{ docker_image }}:{{ docker_image_tag }}"
    state: started
    ports:
      - "{{ app_port }}:80"
    networks:
      - name: "{{ docker_network }}"
    volumes:
      - "/app/index.html:/usr/share/nginx/html/index.html:ro"
```

#### **Étape 2.5 : Examiner le playbook principal**

**Fichier `nginx_webapp_playbook.yaml`** :
```yaml
- hosts: prod
  become: true
  roles:
    # Rôle 1 : Nginx (proxy) exécuté en premier
    - role: nginx
    
    # Rôle 2 : Webapp (application)
    - role: webapp
      vars:
        container_name: "webapp1"    # Surcharge le nom
        app_port: 8082               # Surcharge le port
```

#### **Étape 2.6 : Valider la syntaxe**
```bash
ansible-playbook --syntax-check -i hosts nginx_webapp_playbook.yaml
```

#### **Étape 2.7 : Exécuter le playbook**
```bash
ansible-playbook -i hosts nginx_webapp_playbook.yaml -v

# Vous verrez :
# TASK [nginx : Create a custom Docker network] ...
# TASK [nginx : Pull Nginx Docker image] ...
# TASK [nginx : Run Nginx container] ...
# TASK [webapp : Pull Docker image] ...
# TASK [webapp : Create application directory] ...
# TASK [webapp : Generate index.html from template] ...
# TASK [webapp : Run Docker container for webapp] ...
```

#### **Étape 2.8 : Vérifier l'état des conteneurs**
```bash
# Consulter les conteneurs en exécution
ansible -i hosts prod -m command -a "docker ps"

# Résultat attendu :
# CONTAINER ID   IMAGE     COMMAND                  STATUS
# xxx            nginx     "/docker-entrypoint.sh"  Up 2 minutes
# yyy            nginx     "/docker-entrypoint.sh"  Up 1 minute

# Vérifier le réseau Docker
ansible -i hosts prod -m command -a "docker network inspect custom_net"

# Consulter les logs
ansible -i hosts prod -m command -a "docker logs nginx"
ansible -i hosts prod -m command -a "docker logs webapp1"
```

---

## 🧪 Instructions complètes étape par étape

### **Scénario complet : Du zéro à l'application déployée**

#### **Phase 1 : Préparation**

```bash
# 1. Créer 2 VMs (CentOS ou Ubuntu)
# VM1 : 10.0.37.5 (client1)
# VM2 : 10.0.37.6 (client2)

# 2. Installer les services essentiels
ssh admin@10.0.37.5 "
  sudo yum update -y
  sudo yum install -y python3 openssh-server openssh-clients
  sudo systemctl start sshd
"

# 3. Installer Ansible localement
pip install ansible

# 4. Cloner le dépôt
git clone https://github.com/eazytraining/bootcamp-project-update.git
cd bootcamp-project-update/mini-projet-ansible
```

#### **Phase 2 : Partie 1 (Simple Nginx)**

```bash
# 1. Entrer dans le répertoire
cd app-init

# 2. Tester la connectivité
ansible -i hosts prod -m ping

# 3. Vérifier la syntaxe
ansible-playbook --syntax-check -i hosts nginx_playbook.yaml

# 4. Exécuter le playbook (mode verbose)
ansible-playbook -i hosts nginx_playbook.yaml -v

# 5. Tester le résultat
curl http://10.0.37.5
# Vous verrez la page HTML

# 6. Tester l'archive playbook_stacker
curl http://10.0.37.5/playbook_stacker/ -I
# Vous verrez : HTTP/1.1 200 OK
```

#### **Phase 3 : Préparation Docker**

```bash
# Sur le host (10.0.37.5), installer Docker

# CentOS
ssh admin@10.0.37.5 "
  sudo yum install -y yum-utils
  sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
  sudo yum install -y docker-ce docker-ce-cli containerd.io
  sudo systemctl start docker
  sudo usermod -aG docker admin
  newgrp docker
"

# Ubuntu
ssh admin@10.0.37.5 "
  sudo apt-get update
  sudo apt-get install -y docker.io
  sudo systemctl start docker
  sudo usermod -aG docker admin
  newgrp docker
"

# Installer les modules Python Docker (requis par Ansible)
ssh admin@10.0.37.5 "
  pip3 install docker
"
sur CentOS
sudo yum install -y python3 python3-pip
```

#### **Phase 4 : Partie 2 (Docker + Rôles)**

```bash
# 1. Entrer dans le répertoire
cd ../app-template

# 2. Tester la connectivité
ansible -i hosts prod -m ping

# 3. Valider la syntaxe
ansible-galaxy collection install community.docker
ansible-playbook --syntax-check -i hosts nginx_webapp_playbook.yaml

# 4. Exécuter le playbook
ansible-playbook -i hosts nginx_webapp_playbook.yaml -v

# 5. Vérifier les conteneurs
ansible -i hosts prod -m command -a "docker ps"

# 6. Tester le proxy Nginx (port 80)
curl http://10.0.37.5
# Vous verrez la même page HTML

# 7. Tester l'application directe (port 8082)
curl http://10.0.37.5:8082
# Même contenu

# 8. Vérifier le réseau Docker
ansible -i hosts prod -m command -a "docker network ls"

# 9. Vérifier la communication entre conteneurs
ansible -i hosts prod -m command -a "docker exec nginx ping webapp1"
# Vous verrez : PING webapp1 (172.xx.xx.xx) ...
```

---

## ✅ Vérification et validation

### **Checklist Partie 1**

```bash
# ✅ Nginx est installé
ansible -i app-init/hosts prod -m command -a "nginx -v"

# ✅ Service Nginx est actif
ansible -i app-init/hosts prod -m service -a "name=nginx state=started"

# ✅ Port 80 écoute
ansible -i app-init/hosts prod -m command -a "netstat -tlnp | grep 80"

# ✅ index.html existe
ansible -i app-init/hosts prod -m command -a "ls -la /usr/share/nginx/html/"

# ✅ Page HTML est accessible
curl -I http://10.0.37.5
# Résultat : HTTP/1.1 200 OK
```

### **Checklist Partie 2**

```bash
# ✅ Docker est installé
ansible -i app-template/hosts prod -m command -a "docker --version"

# ✅ Réseau Docker custom_net existe
ansible -i app-template/hosts prod -m command -a "docker network inspect custom_net"

# ✅ Conteneur Nginx est en cours d'exécution
ansible -i app-template/hosts prod -m command -a "docker ps | grep nginx"

# ✅ Conteneur webapp1 est en cours d'exécution
ansible -i app-template/hosts prod -m command -a "docker ps | grep webapp1"

# ✅ Port 80 répond (proxy)
curl -I http://10.0.37.5
# Résultat : HTTP/1.1 200 OK

# ✅ Port 8082 répond (direct)
curl -I http://10.0.37.5:8082
# Résultat : HTTP/1.1 200 OK

# ✅ Contenu identique sur les deux URLs
curl http://10.0.37.5 | md5sum
curl http://10.0.37.5:8082 | md5sum
# Les deux hashes doivent être identiques
```

### **Test avancé : Flux du proxy Nginx**

```bash
# 1. Vérifier les logs Nginx (conteneur proxy)
ansible -i app-template/hosts prod -m command -a "docker logs nginx | tail -10"
# Vous verrez les requêtes proxy_pass vers webapp1:80

# 2. Vérifier les logs du conteneur webapp
ansible -i app-template/hosts prod -m command -a "docker logs webapp1 | tail -10"
# Vous verrez les requêtes reçues

# 3. Tester la communication de conteneur à conteneur
ansible -i app-template/hosts prod -m command -a "docker exec nginx curl http://webapp1"
# Vous verrez le HTML retourné par webapp1

# 4. Inspectez la configuration Nginx
ansible -i app-template/hosts prod -m command -a "docker exec nginx cat /etc/nginx/nginx.conf | grep -A 5 upstream"
```

---

## 🔧 Troubleshooting

### **Problème : Connexion SSH refusée**

```bash
# Vérifier la connectivité
ping 10.0.37.5

# Vérifier que SSH est accessible
ssh -v admin@10.0.37.5

# Ajouter la clé SSH au trousseau
ssh-keyscan 10.0.37.5 >> ~/.ssh/known_hosts

# Réessayer Ansible
ansible -i hosts prod -m ping
```

### **Problème : Erreur "Permission denied"**

```bash
# Vérifier que l'utilisateur admin peut utiliser sudo
ssh admin@10.0.37.5 "sudo id"

# Ajouter admin au groupe sudoers (si nécessaire)
ssh admin@10.0.37.5 "echo 'admin ALL=(ALL) NOPASSWD: ALL' | sudo tee -a /etc/sudoers"
```

### **Problème : Nginx ne démarre pas (Partie 1)**

```bash
# Vérifier les logs Nginx
ansible -i app-init/hosts prod -m command -a "journalctl -u nginx -n 20"

# Tester la configuration Nginx
ansible -i app-init/hosts prod -m command -a "nginx -t"

# Démarrer manuellement
ansible -i app-init/hosts prod -m service -a "name=nginx state=restarted"
```

### **Problème : Conteneur Docker ne démarre pas (Partie 2)**

```bash
# Vérifier les erreurs Docker
ansible -i app-template/hosts prod -m command -a "docker logs nginx"
ansible -i app-template/hosts prod -m command -a "docker logs webapp1"

# Vérifier les permissions Docker
ssh admin@10.0.37.5 "groups"
# admin doit être dans le groupe docker

# Ajouter à docker (si nécessaire)
ssh admin@10.0.37.5 "
  sudo usermod -aG docker admin
  newgrp docker
"

# Redémarrer le daemon Docker
ssh admin@10.0.37.5 "sudo systemctl restart docker"
```

### **Problème : Port 80 déjà utilisé**

```bash
# Vérifier quel processus utilise le port
ansible -i hosts prod -m command -a "lsof -i :80"

# Arrêter Nginx (Partie 1)
ansible -i app-init/hosts prod -m service -a "name=nginx state=stopped"

# Ou arrêter le conteneur (Partie 2)
ansible -i app-template/hosts prod -m command -a "docker stop nginx"
```

### **Problème : Modules Docker non disponibles**

```bash
# Installer les modules Python Docker
ssh admin@10.0.37.5 "
  pip3 install docker
"

# Ou utiliser pip au niveau du projet
ansible -i hosts prod -m shell -a "
  pip3 install --user docker
"
```

### **Problème : Template Jinja2 non trouvé**

```bash
# Vérifier l'existence du template
ls -la app-init/templates/
ls -la app-template/webapp/templates/

# Vérifier le chemin dans le playbook
grep "src:" app-init/nginx_playbook.yaml
grep "src:" app-template/webapp/tasks/main.yml
```

---

## 📚 Ressources utiles

### **Documentation officielle**
- [Documentation Ansible](https://docs.ansible.com/)
- [Module Docker Ansible](https://docs.ansible.com/ansible/latest/collections/community/docker/)
- [Templates Jinja2](https://jinja.palletsprojects.com/)

### **Commandes Ansible utiles**

```bash
# Lister tous les hosts
ansible-inventory -i hosts --list

# Exécuter une commande ad-hoc
ansible -i hosts prod -m command -a "df -h"

# Exécuter une tâche avec debug
ansible-playbook -i hosts playbook.yaml -vv

# Tester un playbook (mode check)
ansible-playbook -i hosts playbook.yaml --check

# Afficher les variables
ansible -i hosts prod -m debug -a "var=ansible_os_family"

# Exécuter une tâche spécifique
ansible-playbook -i hosts playbook.yaml --tags "install"
```

### **Commandes Docker utiles**

```bash
# Lister les conteneurs
docker ps -a

# Inspecter un conteneur
docker inspect nginx

# Consulter les logs
docker logs nginx -f

# Exécuter une commande dans un conteneur
docker exec nginx nginx -t

# Nettoyer les ressources
docker system prune -a
```

---

## 🎓 Apprentissages clés

### **Concepts Ansible**

1. **Playbooks** : Scripts d'automatisation déclaratifs
2. **Rôles** : Organisation modulaire des tâches
3. **Variables** : Configuration dynamique
4. **Templates** : Génération de fichiers avec Jinja2
5. **Handlers** : Tâches conditionnelles (notifications)
6. **Inventaire** : Définition des hosts et groupes

### **Concepts Docker**

1. **Images** : Modèles pour créer des conteneurs
2. **Conteneurs** : Instances en cours d'exécution
3. **Réseaux** : Communication entre conteneurs
4. **Volumes** : Partage de fichiers
5. **Proxy inverse** : Nginx pour la distribution du trafic

### **Flux d'automatisation**

```
Playbook Ansible
    ↓
Inventaire (hosts)
    ↓
Variables (group_vars, host_vars)
    ↓
Tâches / Rôles
    ↓
Exécution distante via SSH
    ↓
Configuration du système ou Docker
```

---

## 📝 Notes de projet

### **Ce qui a été fait**

✅ Création d'un playbook simple (Partie 1)
✅ Création de deux rôles Ansible (Partie 2)
✅ Configuration d'un proxy inverse Nginx dans Docker
✅ Automatisation complète du déploiement
✅ Documentation exhaustive

### **Améliorations possibles**

- Ajouter des tests Ansible (molecule)
- Implémenter la CI/CD avec Jenkins ou GitLab CI
- Ajouter du monitoring (Prometheus, Grafana)
- Sécuriser avec TLS/HTTPS
- Utiliser Kubernetes au lieu de Docker simple
- Implémenter l'orchestration avec Docker Compose

---

## 🤝 Support et questions

En cas de problème ou de question :

1. **Consulter les logs** : `journalctl`, `docker logs`
2. **Valider la syntaxe** : `--syntax-check`, `ansible-playbook -v`
3. **Utiliser le mode check** : `--check` pour tester sans effectuer
4. **Vérifier la connectivité** : `ping`, `ssh`, `ansible -m ping`
5. **Consulter la documentation** : [docs.ansible.com](https://docs.ansible.com/)

---

## 📄 Licence et auteurs

**Projet** : Mini-Projet Ansible Eazytraining
**Formation** : Bootcamp DevOps / Cloud
**Plateforme** : Eazytraining

---

**Dernière mise à jour** : Mars 2026
**Status** : ✅ Complet et opérationnel

# Corrections du Mini-Projet Ansible

Ce document résume toutes les corrections apportées au mini-projet Ansible pour le déploiement d'une application.

## Partie 1: Déploiement Simple (app-init)

Le playbook `app-init/nginx_playbook.yaml` est **correct** et fonctionnel.

### Contenu:
- **Installation de Nginx** sur les serveurs de production
- **Configuration des templates Jinja2** pour générer le fichier index.html
- **Gestion des services** avec redémarrage de Nginx
- **Vérification du service HTTP** via handler

### Fichiers configurés:
- ✅ `hosts` - Configuration des inventaires
- ✅ `nginx_playbook.yaml` - Playbook principal
- ✅ `group_vars/all.yml` - Variables communes (credentials admin)
- ✅ `host_vars/client1.yml` - Configuration spécifique client1 (10.0.37.5)
- ✅ `templates/` - Tous les fichiers de template HTML
- ✅ `vars/logos.yaml` - Variables avec logos encodés en base64

---

## Partie 2: Déploiement en Conteneur avec Docker (app-template)

### Fichiers corrigés:

#### 1. **app-template/group_vars/all.yml**
- ✅ Remplacé `<FIX IT>` par les credentials:
  - `ansible_user: admin`
  - `ansible_password: admin`
  - `ansible_sudo_pass: admin`

#### 2. **app-template/host_vars/client1.yml**
- ✅ Remplacé `<FIX IT>` par `ansible_host: 10.0.37.5`

#### 3. **app-template/host_vars/client2.yml**
- ✅ Remplacé `<FIX IT>` par `ansible_host: 10.0.37.6` (pour complétude)

#### 4. **app-template/nginx_webapp_playbook.yaml**
- ✅ Remplacé `<FIX IT>` par le nom du rôle: `role: webapp`

#### 5. **app-template/nginx/defaults/main.yml**
- ✅ Ajouté les variables par défaut:
  - Noms et images Docker
  - Configuration réseau (custom_net, port 80)
  - Upstream backend (webapp1:80)
  - Politiques de redémarrage

#### 6. **app-template/nginx/vars/main.yml**
- ✅ Ajouté les variables du rôle:
  - Configuration Docker pour Nginx
  - Chemins des fichiers de configuration
  - Paramètres de redémarrage

#### 7. **app-template/webapp/defaults/main.yml**
- ✅ Ajouté les variables par défaut pour le rôle webapp:
  - Nom du conteneur: `webapp`
  - Port de l'application: 8080
  - Réseau Docker: `custom_net`
  - Configuration des fichiers et templates

#### 8. **app-template/webapp/vars/main.yml**
- ✅ Ajouté les variables constantes:
  - Image Docker de base: `nginx:latest`
  - Répertoire de travail: `/app`
  - Configuration du logging

#### 9. **app-template/webapp/handlers/main.yml**
- ✅ Ajouté les handlers:
  - Redémarrage du conteneur webapp
  - Vérification du service HTTP

#### 10. **app-template/webapp/tasks/main.yml** (FICHIER CRITIQUE)
- ✅ **Entièrement rédigé** avec les tâches suivantes:
  - Téléchargement de l'image Docker Nginx
  - Création du répertoire d'application
  - Copie des fichiers de l'application
  - Génération du fichier index.html à partir du template Jinja2
  - Exécution du conteneur Docker avec:
    - Configuration réseau personnalisée (custom_net)
    - Montage des volumes (index.html en lecture seule)
    - Exposition du port d'application
    - Politique de redémarrage
  - Vérification que le conteneur est en cours d'exécution
  - Attente de la disponibilité du service (wait_for)

---

## Architecture du Déploiement en Conteneur

### Flux de déploiement:

1. **Rôle Nginx** (exécuté en premier):
   - Crée un réseau Docker personnalisé `custom_net`
   - Télécharge l'image Nginx
   - Copie la configuration Nginx (`nginx.conf`)
   - Lance le conteneur Nginx avec:
     - Exposition du port 80
     - Connexion au réseau `custom_net`
     - Montage de la configuration Nginx

2. **Rôle Webapp** (exécuté après Nginx):
   - Télécharge l'image Docker (Nginx)
   - Crée le répertoire de l'application
   - Génère le fichier `index.html` à partir des templates Jinja2
   - Lance le conteneur webapp avec:
     - Connexion au réseau `custom_net`
     - Exposition du port 8082 (ou autre selon la variable `app_port`)
     - Montage du fichier index.html

### Configuration Nginx (proxy inverse):

Le fichier `app-template/nginx/tasks/nginx.conf` configure Nginx comme proxy inverse:
- Écoute sur le port 80
- Transmet les requêtes vers le backend `webapp1:80` (sur le réseau custom_net)
- Ajoute les headers de proxy appropriés

---

## Variables de Configuration

### Variables principales utilisées:

| Variable | Valeur | Rôle |
|----------|--------|------|
| `container_name` | `webapp1` | Nom du conteneur webapp |
| `app_port` | 8082 | Port exposé du conteneur |
| `docker_network` | `custom_net` | Réseau Docker personnalisé |
| `docker_image` | `nginx` | Image Docker de base |
| `template_index` | `index.html-base.j2` | Template à utiliser pour l'index |

### Fichiers de configuration:

- `app-template/group_vars/all.yml`: Credentials communes pour tous les hosts
- `app-template/host_vars/client1.yml`: Configuration spécifique au host client1
- `app-template/host_vars/client2.yml`: Configuration spécifique au host client2

---

## Comment exécuter les playbooks

### Partie 1 - Déploiement simple:

```bash
cd app-init
ansible-playbook -i hosts nginx_playbook.yaml
```

### Partie 2 - Déploiement en conteneur:

```bash
cd app-template
ansible-playbook -i hosts nginx_webapp_playbook.yaml
```

---

## Vérification du déploiement

### Partie 1:
```bash
curl http://10.0.37.5
```

### Partie 2:
```bash
# Via le proxy Nginx
curl http://10.0.37.5

# Directement sur le conteneur webapp
curl http://10.0.37.5:8082
```

---

## Résumé des corrections

✅ **10 fichiers corrigés/complétés**
✅ **Toutes les expressions `<FIX IT>` remplacées**
✅ **Fichier `webapp/tasks/main.yml` entièrement rédigé**
✅ **Configuration Docker complète et fonctionnelle**
✅ **Proxy inverse Nginx configuré**
✅ **Support des templates Jinja2**
✅ **Gestion des handlers et notifications**
✅ **Networking personnalisé (custom_net)**

Le projet est maintenant **entièrement complété** et **prêt pour le déploiement**.

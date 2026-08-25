# 🛡️ Centre d'Opérations Réseau (NOC) & SIEM - Automatisation Ansible

Un dépôt Ansible complet de niveau production pour le provisionnement, la configuration et la gestion d'un NOC (Network Operations Center), d'une infrastructure d'observabilité et d'un SIEM (Security Information and Event Management).

---

## 📑 Sommaire
1. [Présentation et Fonctionnalités](#-présentation-et-fonctionnalités)
2. [Écosystème et Stack Technique](#-écosystème-et-stack-technique)
3. [Architecture des Rôles Ansible](#-architecture-des-rôles-ansible)
4. [Prérequis et Environnements Supportés](#-prérequis-et-environnements-supportés)
5. [Matrice des Flux Réseau (Ports)](#-matrice-des-flux-réseau-ports)
6. [Guide de Déploiement](#-guide-de-déploiement)
7. [Guide d'Exploitation : Ajouter un Serveur](#-guide-dexploitation--ajouter-un-serveur)
8. [Dépannage Courant](#-dépannage-courant)

---

## 🚀 Présentation et Fonctionnalités

Ce projet vise à normaliser et automatiser le déploiement d'outils de supervision et de sécurité sur un parc hétérogène. 

**Fonctionnalités clés :**
- **Zéro-touch provisioning :** Installation des dépendances (Docker, paquets système) et de la stack complète en une seule ligne de commande.
- **Observabilité 360° :** Collecte des métriques (système et applicatives), centralisation des logs, et traçabilité.
- **Sécurité intégrée :** Déploiement des agents de détection d'intrusion (HIDS), surveillance d'intégrité des fichiers (FIM) et analyse des vulnérabilités.
- **Modularité :** Possibilité de déployer uniquement les agents sur de nouveaux serveurs (app/db) ou de reconstruire le serveur NOC centralisé.

---

## 🛠️ Écosystème et Stack Technique

Le projet déploie et configure les composants suivants :

### 1. Supervision et Métriques (NOC)
* **Prometheus :** Collecteur de métriques time-series.
* **Node Exporter :** Métriques matérielles et OS (CPU, RAM, Disque, Réseau).
* **PostgreSQL Exporter :** Métriques de base de données spécifiques pour les instances PostgreSQL.
* **cAdvisor :** Collecte des métriques pour les environnements conteneurisés.

### 2. Gestion des Journaux et Traces
* **Loki & Promtail :** Agrégation et requêtage de logs applicatifs (Nginx, Gunicorn, Django) et système.
* **Grafana Tempo & OTel Collector :** Traçabilité distribuée.

### 3. Visualisation et Alerting
* **Grafana :** Tableaux de bord unifiés (NOC et Sécurité).
* **Alertmanager :** Routage des alertes critiques (Webhook, Email).

### 4. SIEM et Sécurité (Wazuh)
* **Wazuh Manager & Indexer :** Cerveau de l'analyse de sécurité et indexation des événements.
* **Wazuh Agent :** Déployé sur tous les nœuds pour la réponse aux incidents et l'audit.

---

## 📂 Architecture des Rôles Ansible

L'organisation suit strictement les *best practices* Ansible, séparant la logique par rôle :

```text
├── inventory/
│   ├── group_vars/
│   │   ├── all/vault.yml          # Secrets chiffrés (mots de passe DB, clés API)
│   │   └── all/vars.yml           # Variables globales non sensibles
│   ├── host_vars/                 # Variables spécifiques (ex: IP publiques/privées)
│   └── hosts.ini                  # Groupes : [nocserv], [appserv], [dbserv], [k8s_nodes]
├── playbooks/
│   ├── site.yml                   # Déploiement complet (Master playbook)
│   ├── noc.yml                    # Provisionnement du serveur de supervision central
│   ├── db.yml                     # Configuration orientée bases de données (PostgreSQL)
│   └── app.yml                    # Configuration orientée applications (Nginx, Django)
└── roles/
    ├── common/                    # Sécurisation SSH, Sysctl, NTP, Fail2ban
    ├── infra/                     # Déploiement de Docker, configuration des daemons
    ├── manager/                   # Composants du NOC central (Grafana, Loki, Wazuh-Manager)
    └── agent/                     # Agents de collecte : Promtail, Wazuh-Agent, Node-Exporter
```

---

## ⚙️ Prérequis et Environnements Supportés

**Systèmes d'exploitation cibles validés :**
* Debian 13 (Recommandé pour sa stabilité)
* Ubuntu Server (20.04, 22.04, 24.04)
* Environnements virtualisés (VirtualBox, VMware) et Cloud (OpenStack).
* Nœuds d'un cluster **Kubernetes (k8s natif)**.

**Pré-requis sur le nœud de contrôle :**
* Ansible >= 2.10
* `ansible-galaxy collection install ansible.posix community.docker community.general`

---

## 🔀 Matrice des Flux Réseau (Ports)

Pour que l'infrastructure communique correctement, les ports suivants doivent être ouverts sur le réseau/pare-feu :

| Composant | Port | Protocole | Source | Destination | Description |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Grafana UI** | `3000` | TCP | Utilisateurs | `nocserv` | Interface d'administration |
| **Prometheus** | `9090` | TCP | `nocserv` | `nocserv` | Interface Prometheus |
| **Node Exporter** | `9100` | TCP | `nocserv` | `appserv`, `dbserv` | Collecte des métriques OS |
| **PostgreSQL Exp.** | `9187` | TCP | `nocserv` | `dbserv` | Métriques BDD |
| **Loki** | `3100` | TCP | `appserv`, `dbserv` | `nocserv` | Envoi des logs (Promtail -> Loki) |
| **Wazuh Auth** | `1515` | TCP | `appserv`, `dbserv` | `nocserv` | Enregistrement des agents Wazuh |
| **Wazuh Comms** | `1514` | TCP | `appserv`, `dbserv` | `nocserv` | Transmission des événements de sécurité |

---

## 🏃‍♂️ Guide de Déploiement

### 1. Préparation de l'environnement
Clonez le dépôt et installez les dépendances :
```bash
git clone <url-repo> noc-ansible
cd noc-ansible
ansible-galaxy install -r requirements.yml
```

### 2. Configuration de l'inventaire
Éditez le fichier `inventory/hosts.ini` :
```ini
[nocserv]
192.168.1.100 ansible_user=admin

[appserv]
192.168.1.101 ansible_user=admin # (Nginx, Gunicorn, Django)

[dbserv]
192.168.1.102 ansible_user=admin # (PostgreSQL)

[k8s_nodes]
192.168.1.110 ansible_user=admin # (Noeuds k8s natifs)
```

### 3. Gestion des Secrets (Ansible Vault)
Initialisez votre fichier de secrets :
```bash
cp inventory/group_vars/all/vault.example.yml inventory/group_vars/all/vault.yml
ansible-vault edit inventory/group_vars/all/vault.yml
```
*(Ajoutez-y les mots de passe Grafana, clés API Wazuh, identifiants PostgreSQL).*

### 4. Exécution des Playbooks
Déployer l'infrastructure centrale (NOC & SIEM) :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/noc.yml --ask-vault-pass
```

Déployer les agents sur les serveurs applicatifs et bases de données :
```bash
ansible-playbook -i inventory/hosts.ini playbooks/app.yml playbooks/db.yml --ask-vault-pass
```

---

## ➕ Guide d'Exploitation : Ajouter un Serveur

Lorsqu'un nouveau serveur applicatif (ex: une nouvelle instance web) est ajouté à la production :

1. Ajoutez l'adresse IP du serveur dans `inventory/hosts.ini` (sous `[appserv]`).
2. Assurez-vous que l'authentification SSH par clé est configurée :
   ```bash
   ssh-copy-id admin@<nouvelle-ip>
   ```
3. Exécutez le playbook ciblé sur ce nouveau nœud pour installer Docker, Promtail, Node-Exporter et l'Agent Wazuh :
   ```bash
   ansible-playbook -i inventory/hosts.ini playbooks/app.yml --limit <nouvelle-ip> --ask-vault-pass
   ```
4. Le nœud apparaîtra automatiquement dans les tableaux de bord Grafana et Wazuh d'ici 2 à 3 minutes.

---

## 🛠️ Dépannage Courant

**1. L'Agent Wazuh ne remonte pas dans le Manager :**
* Vérifiez les logs sur l'agent : `tail -f /var/ossec/logs/ossec.log`
* Assurez-vous que le port TCP/1514 est joignable depuis l'agent : `telnet <ip-manager> 1514`

**2. Les logs Nginx/Django n'apparaissent pas dans Grafana/Loki :**
* Vérifiez que l'utilisateur exécutant Promtail (via Docker ou systemd) a les droits de lecture sur `/var/log/nginx/` et le répertoire de logs applicatifs.
* Vérifiez la configuration du *scrape_config* de Promtail générée par Ansible.

**3. Erreurs de Virtualisation au déploiement (VirtualBox) :**
* Si vos nœuds cibles sont des machines virtuelles (VirtualBox), assurez-vous que les options de virtualisation imbriquée sont correctement configurées en fonction de l'hyperviseur hôte.

---

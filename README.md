# Projet NOC - Monitoring & Observabilité Linux

**Centre de Commande Grafana** — Infrastructure e-commerce (3 serveurs)

---

## 📋 Description du Projet

Ce projet consiste à concevoir et déployer un **NOC (Network Operations Center)** complet pour une infrastructure e-commerce composée de 3 serveurs Debian 13.

L’objectif principal est de faire de **Grafana** le **seul et unique centre de visualisation** pour les équipes SRE, NOC et la direction. Toutes les sources de données (métriques, logs, traces, sécurité) convergent vers Grafana.

**Durée du projet** : 1 semaine (lab accéléré)

---

## 🎯 Objectifs

- Concevoir une architecture de monitoring claire et scalable
- Déployer un stack d’observabilité complet (métriques, logs, traces, alerting)
- Centraliser toute la supervision dans Grafana
- Automatiser le déploiement avec **Ansible** (Infrastructure as Code)
- Réaliser des dashboards pertinents, des alertes et des runbooks
- Simuler des incidents et produire des post-mortems

---

## 🏗️ Architecture

### Serveurs

| Serveur     | Nom         | IP            | Rôle principal                  |
|-------------|-------------|---------------|---------------------------------|
| Serveur 1   | `nocserv`   | 192.168.6.10  | Monitoring / NOC (Grafana + Stack) |
| Serveur 2   | `dbserv`    | 192.168.6.30  | Base de données PostgreSQL      |
| Serveur 3   | `appserv`   | 192.168.6.20  | Application (Nginx + Django + Laravel) |

### Stack Technique

- **Visualisation** : Grafana (port 3030)
- **Métriques** : Prometheus + Node Exporter + PostgreSQL Exporter + cAdvisor
- **Logs** : Loki + Promtail
- **Traces** : Tempo
- **Sécurité** : Wazuh (Manager + Agents)
- **Alerting** : Alertmanager + Grafana Unified Alerting
- **Automatisation** : Ansible
- **OS** : Debian 13

---

## 📁 Structure du Projet

# Secure Self-Hosted Application

> Déploiement d'une application auto-hébergée au sein d'une infrastructure virtualisée sécurisée.

## Présentation

Ce projet documente le déploiement d'une application auto-hébergée (Immich) dans une infrastructure personnelle conçue pour reproduire les bonnes pratiques d'un environnement d'entreprise.

L'objectif ne se limite pas à rendre l'application accessible depuis Internet. L'infrastructure a été pensée pour être évolutive, segmentée et sécurisée afin de servir de base à d'autres projets d'administration systèmes, de cybersécurité et d'automatisation.

Ce dépôt fait partie de mon laboratoire personnel d'administration systèmes, réseaux et cybersécurité.

---

# Objectifs

- Déployer une application auto-hébergée dans un environnement virtualisé.
- Sécuriser l'exposition du service sur Internet.
- Isoler les services du réseau personnel.
- Limiter la surface d'attaque des interfaces d'administration.
- Concevoir une architecture facilement réutilisable pour d'autres applications.
- Documenter les choix techniques et les difficultés rencontrées.

---

# Architecture

```
                           Internet
                               │
                        Box / Routeur FAI
                               │
                    Redirection des ports 80/443
                               │
                         WAN - pfSense
                               │
                    ┌──────────┴──────────┐
                    │                     │
               OpenVPN               LAN Infrastructure
                    │                     │
                    │               Ubuntu Server
                    │                     │
                    │                  Docker
                    │                     │
                    │          Nginx Proxy Manager
                    │                     │
                    └────────────► Immich
```

---

# Infrastructure

| Élément | Rôle |
|---------|------|
| Proxmox VE | Plateforme de virtualisation |
| pfSense | Pare-feu, routage, OpenVPN |
| Ubuntu Server | Hôte Docker |
| Docker | Conteneurisation des services |
| Nginx Proxy Manager | Reverse Proxy et gestion HTTPS |
| Immich | Application auto-hébergée |

---

# Choix d'architecture

## Virtualisation

L'ensemble des services est hébergé sur Proxmox VE afin d'isoler les différents composants de l'infrastructure et de faciliter leur administration.

Cette approche permet de faire évoluer indépendamment chaque service sans impacter les autres.

---

## Pare-feu dédié

Le trafic entrant transite exclusivement par pfSense.

Le pare-feu assure :

- le routage ;
- le NAT ;
- le filtrage des flux ;
- l'hébergement du serveur OpenVPN.

Cette architecture centralise l'ensemble des règles de sécurité sur un point unique.

---

## Segmentation réseau

L'infrastructure est séparée du réseau utilisé quotidiennement.

Cette séparation permet :

- d'éviter qu'une erreur de configuration impacte les postes personnels ;
- de réaliser des expérimentations en limitant les risques ;
- de préparer l'ajout de nouveaux services.

---

## Reverse Proxy

Nginx Proxy Manager est utilisé comme point d'entrée unique des applications Web.

Cette architecture permet :

- la centralisation des accès HTTP/HTTPS ;
- la gestion simplifiée des certificats TLS ;
- la publication de plusieurs services derrière une même adresse IP publique.

---

# Contrôle des accès d'administration

L'infrastructure distingue les services destinés aux utilisateurs des interfaces d'administration.

Les interfaces d'administration ne sont **jamais accessibles directement depuis le réseau local**.

L'administration de :

- pfSense ;
- Proxmox VE ;
- l'interface d'administration d'Immich ;
- des serveurs Linux (SSH) ;

nécessite l'établissement préalable d'un tunnel **OpenVPN**.

Cette approche repose sur le principe que le réseau local ne doit pas être considéré comme un réseau de confiance.

Même en cas de compromission d'un poste connecté au réseau local, les interfaces d'administration restent inaccessibles sans authentification VPN.

Cette architecture permet de :

- réduire la surface d'attaque ;
- protéger les interfaces sensibles ;
- centraliser les accès administratifs ;
- chiffrer toutes les connexions d'administration.

---

# Difficultés rencontrées

## Architecture réseau initiale

Lors des premiers déploiements, certaines machines virtuelles étaient connectées au mauvais réseau.

Cette configuration empêchait pfSense d'assurer correctement son rôle de passerelle.

### Solution

- création d'un réseau dédié sous Proxmox ;
- migration des machines vers ce réseau ;
- utilisation de pfSense comme unique routeur de l'infrastructure.

---

## Publication du service

La publication d'Immich a nécessité plusieurs ajustements concernant :

- le NAT ;
- le reverse proxy ;
- les certificats TLS ;
- la résolution DNS.

Ces difficultés ont permis de mieux comprendre les interactions entre les différents composants de l'infrastructure.

---

## Validation des accès

Les premiers tests réalisés depuis le réseau local ne reflétaient pas le comportement réel des accès externes.

La validation finale a donc été effectuée depuis un réseau externe afin de vérifier le fonctionnement complet de la publication Internet.

---

# Résultat

L'infrastructure répond désormais aux objectifs définis.

✔ Application accessible en HTTPS

✔ Reverse Proxy opérationnel

✔ Pare-feu dédié

✔ Infrastructure segmentée

✔ Administration exclusivement via OpenVPN

✔ Architecture évolutive permettant l'ajout de nouveaux services

---

# Compétences mises en œuvre

## Administration Systèmes

- Ubuntu Server
- Docker
- Administration Linux

## Infrastructure & Réseau

- Proxmox VE
- Virtualisation
- Segmentation réseau
- NAT
- DNS
- Routage

## Cybersécurité

- pfSense
- Reverse Proxy
- TLS / HTTPS
- Contrôle des accès
- VPN
- Réduction de la surface d'attaque

---

# Évolutions prévues

Cette architecture constitue la base de mon laboratoire personnel.

Les prochaines évolutions prévues sont :

- déploiement d'une plateforme Wazuh ;
- automatisation des audits de sécurité avec n8n ;
- développement d'outils Python ;
- intégration de LLM locaux (Ollama) pour assister l'analyse et la génération de rapports ;
- ajout de nouveaux services auto-hébergés.

---

# Documentation

La documentation détaillée est disponible dans le dossier **docs/**.

- Installation de Proxmox
- Configuration réseau
- pfSense
- OpenVPN
- Docker
- Nginx Proxy Manager
- Déploiement d'Immich
- Journal des difficultés rencontrées

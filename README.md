#  Homelab - Secure Self-Hosted Application

## Présentation

Ce projet présente le déploiement d'une application auto-hébergée (Immich) au sein d'une infrastructure virtualisée sécurisée.

L'objectif n'était pas simplement de rendre l'application accessible depuis Internet, mais de concevoir une architecture évolutive reproduisant les principes d'une infrastructure d'entreprise : séparation des rôles, segmentation réseau, contrôle des flux entrants et possibilité d'accueillir de nouveaux services.

Ce projet constitue l'une des briques de mon laboratoire personnel d'administration systèmes, réseaux et cybersécurité.

---

# Objectifs

- Déployer une application auto-hébergée accessible depuis Internet
- Limiter la surface d'exposition du service
- Isoler l'infrastructure du réseau personnel
- Centraliser le filtrage réseau
- Mettre en place une architecture réutilisable pour d'autres applications
- Documenter les choix techniques et les difficultés rencontrées

---

# Architecture

```
                     Internet
                         │
                    Box FAI
               (NAT 80 / 443)
                         │
                    pfSense (WAN)
                         │
              ┌──────────┴──────────┐
              │                     │
         VPN Administration     LAN 10.0.10.0/24
                                      │
                                Ubuntu Server
                                      │
                            Docker Engine
                           ┌───────────────┐
                           │               │
                       Immich        Nginx Proxy Manager
```

Principes retenus :

- pfSense constitue le point d'entrée unique de l'infrastructure
- seules les connexions HTTP/HTTPS sont publiées
- les conteneurs Docker ne sont jamais exposés directement
- le reverse proxy assure la terminaison TLS et le routage vers les services

---

# Infrastructure

| Élément | Rôle |
|---------|------|
| Proxmox VE | Hyperviseur |
| pfSense | Pare-feu / Routeur |
| Ubuntu Server | Hôte Docker |
| Docker | Hébergement des services |
| Nginx Proxy Manager | Reverse Proxy |
| Immich | Application auto-hébergée |

---

# Choix d'architecture

## Virtualisation avec Proxmox

Le choix de Proxmox permet d'isoler les différents composants de l'infrastructure tout en facilitant leur administration et leur évolution.

## Pare-feu dédié

L'ensemble du trafic entrant transite par pfSense afin de centraliser :

- le routage
- le filtrage
- les règles NAT
- l'administration VPN

## Segmentation réseau

L'infrastructure est isolée du réseau utilisé quotidiennement.

Cette séparation permet :

- de limiter les impacts d'une erreur de configuration
- d'expérimenter en toute sécurité
- de préparer l'ajout de nouveaux services

## Reverse Proxy

L'utilisation de Nginx Proxy Manager permet :

- la centralisation des accès HTTP/HTTPS
- la gestion simplifiée des certificats TLS
- la publication de plusieurs applications derrière une même adresse IP

---

# Difficultés rencontrées

Le projet a permis de résoudre plusieurs problématiques classiques d'administration réseau.

### Mauvaise segmentation initiale

La machine Ubuntu était initialement connectée au réseau de la box Internet.

Conséquence :

- pfSense ne pouvait pas router correctement le trafic vers la VM.

Solution :

- création d'un bridge LAN dédié sous Proxmox ;
- migration de la VM vers ce réseau ;
- utilisation de pfSense comme passerelle unique.

---

### Double NAT

L'utilisation simultanée du NAT de la box Internet et de celui de pfSense a nécessité une reconfiguration des redirections de ports afin que le trafic traverse correctement le pare-feu.

---

### Validation des accès

Les tests réalisés depuis le réseau local étaient perturbés par le hairpin NAT.

La validation finale a donc été réalisée depuis un réseau externe (4G) afin de confirmer le fonctionnement réel de la publication Internet.

---

# Résultats

Le projet a permis d'obtenir une architecture répondant aux objectifs initiaux.

- Application accessible publiquement en HTTPS
- Reverse Proxy opérationnel
- Pare-feu dédié
- Infrastructure segmentée
- Conteneurs non exposés directement
- Architecture facilement extensible

---

# Compétences mobilisées

## Administration Systèmes

- Ubuntu Server
- Docker
- Netplan

## Infrastructure & Réseau

- Proxmox VE
- Segmentation réseau
- NAT
- Routage
- DNS

## Cybersécurité

- pfSense
- Reverse Proxy
- TLS
- Filtrage réseau

---

# Évolutions prévues

Cette architecture constitue la base du laboratoire.

Les prochaines évolutions prévues sont :

- déploiement d'une plateforme Wazuh ;
- automatisation des audits avec n8n ;
- intégration de workflows Python ;
- assistance par IA locale (Ollama et LLM) pour l'analyse des résultats ;
- hébergement de nouveaux services derrière la même architecture sécurisée.

---

# Documentation

Les étapes détaillées d'installation et de configuration sont disponibles dans le dossier `docs/`.

- Installation de Proxmox
- Configuration réseau
- Netplan
- Docker
- pfSense
- Nginx Proxy Manager
- Déploiement d'Immich

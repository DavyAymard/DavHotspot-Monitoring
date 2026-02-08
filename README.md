# DavHotspot-Monitoring : Supervision de Liaison Satellite Starlink

## 📡 Présentation du Projet
Ce dépôt contient l'infrastructure de monitoring complète déployée pour superviser un réseau critique situé à Bria (République Centrafricaine). Ce projet valide les compétences en ingénierie de données, supervision SNMP et résilience réseau dans le cadre d'un Master SSI.

## 🏗️ Architecture Globale
- **Plan de Contrôle** : Routeur MikroTik (Bria) via SNMP v2c.
- **Liaison de Données** : Starlink (Liaison satellite haute latence).
- **Plateforme de Gestion** : Stack Prometheus & Grafana conteneurisée sur ZadoVPS (Bangui).

## 📊 Indicateurs de Gouvernance IT (KPIs)
- **Disponibilité Réseau** : Suivi de l'Up-time et de la stabilité du lien satellite.
- **Performance WAN** : Débits asymétriques temps réel et audit de latence.
- **Audit des Ressources** : Consommation CPU, RAM et intégrité du stockage Flash.
- **Conformité & Sécurité** : Suivi de la version RouterOS pour le Patch Management.

## 🛡️ Sécurité par Design
- **Secrets** : Exclusion des configurations sensibles via `.gitignore`.
- **Cloisonnement** : Isolation des services via Docker.

---
**Auteur** : Davy Aymard LITSE - Professionnel TIC & Auditeur SSI.
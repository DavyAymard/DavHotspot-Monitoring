# Dossier d'Ingénierie : Supervision Résiliente et Audit SSI (Bria, RCA)

Ce document constitue le référentiel technique pour le déploiement d'une stack de monitoring (Prometheus/Grafana) destinée à la surveillance d'infrastructures critiques en environnement isolé.

## 1. Phase de Durcissement du Serveur (Hardening)

La plateforme de supervision ZadoVPS est le socle de confiance de l'audit. Son durcissement est la première étape de notre protocole de sécurité.

### 1.1 Sécurisation de la Couche OS (Debian/Ubuntu)

```bash
# Mise à jour des dépôts et application des patchs de sécurité
sudo apt update && sudo apt upgrade -y

# Configuration d'une politique de filtrage "Default Deny" via UFW
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Autorisations granulaires (Management & Flux)
sudo ufw allow 22/tcp    # SSH : Gestion à distance
sudo ufw allow 3000/tcp  # Grafana : Visualisation
sudo ufw allow 9090/tcp  # Prometheus : Moteur de calcul
sudo ufw enable

```

### 1.2 Isolation par Conteneurisation (Docker)

Nous utilisons Docker pour garantir l'étanchéité des services et simplifier le cycle de vie applicatif.

```bash
# Installation de Docker Engine et de l'orchestrateur Compose
sudo apt install docker.io docker-compose -y
sudo systemctl enable --now docker

```

## 2. Configuration de l'Agent SNMP (MikroTik)

Le routeur MikroTik à Bria agit comme un agent SNMP v2c. Son exposition est limitée par une communauté dédiée.

### 2.1 Identification des OIDs et Paramétrage CLI

Nous utilisons les compteurs **64-bit (High Capacity)** pour éviter le "counter rollover" (débordement) fréquent sur les liaisons haut débit comme Starlink.

* **Commande de configuration RouterOS** :

```bash
/snmp set enabled=yes contact="Davy Aymard LITSE" location="Bria-RCA"
/snmp community add name=bria_v2 read-access=yes

```

## 3. Orchestration et Ingénierie des Métriques

La stack est orchestrée pour assurer la persistance des données historiques, cruciale pour l'analyse d'incidents post-mortem.

### 3.1 Déploiement via Docker Compose

```yaml
services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus # Persistance des metrics
    ports: ["9090:9090"]

  snmp-exporter:
    image: prom/snmp-exporter
    volumes:
      - ./snmp_exporter/config:/etc/snmp_exporter
    command: ["--config.file=/etc/snmp_exporter/snmp.yml"]

  grafana:
    image: grafana/grafana
    ports: ["3000:3000"]

volumes:
  prometheus_data:

```

### 3.2 Correction du Typage SNMP (Le défi technique)

Un problème récurrent en monitoring est l'interprétation des compteurs en chaînes de caractères (labels). Nous avons manuellement forcé le type `counter` pour les flux réseau dans le fichier `snmp.yml` .

```yaml
metrics:
  - name: ifInOctets
    oid: 1.3.6.1.2.1.31.1.1.1.6    # OID HCIn (64-bit)
    type: counter
  - name: ifOutOctets
    oid: 1.3.6.1.2.1.31.1.1.1.10   # OID HCOut (64-bit)
    type: counter

```

## 4. Analyse de Données et Visualisation (PromQL)

Le dashboard Grafana transforme les Octets collectés en débits exploitables par les décideurs.

### 4.1 Formule de Conversion du Débit

Pour calculer la bande passante réelle instantanée, nous utilisons la fonction `rate` sur une fenêtre de 2 minutes, multipliée par 8 pour la conversion Octets -> Bits :


### 4.2 Audit Dynamique du Système

Nous utilisons les transformations **Labels to fields** pour extraire l'identité du matériel et la version de l'OS. Cela permet un audit constant du **Patch Management** (Version actuelle : 7.21.2) afin de prévenir l'exploitation de failles connues sur le MikroTik.

## 5. Gouvernance et Sécurité (Framework SSI)

* **Gestion des Secrets** : Les configurations contenant les communautés SNMP sont systématiquement exclues du versioning via `.gitignore`.
* **Monitoring Satellite** : L'analyse de la latence () permet de différencier une saturation réseau d'une instabilité physique de la liaison Starlink.

## Conclusion

Ce projet démontre la viabilité d'une stack de supervision open-source pour la gestion d'infrastructures distantes. Au-delà de la simple visualisation, cette implémentation répond aux exigences de la **Gouvernance IT** et de la **Cybersécurité** :

1. **Transparence opérationnelle** : Visibilité totale sur l'usage des ressources matérielles et réseau à Bria.
2. **Postures de Sécurité** : Application des principes de durcissement serveur et de gestion des vulnérabilités logicielles.
3. **Aide à la décision** : Capacité à corréler la charge utilisateur avec la qualité du service satellite pour optimiser les coûts et la performance.

En conclusion, ce laboratoire valide les compétences d'ingénierie nécessaires au déploiement de solutions résilientes dans des contextes à ressources limitées, tout en respectant les standards internationaux de sécurité.
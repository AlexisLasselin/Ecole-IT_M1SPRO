# Ecole-IT_M1SPRO

## Projet : Red Team vs Blue Team

**Métier ciblé** : analyste SOC / Red Teamer — Jury simulé : RSSI

**Scénario** : La PME fictive "CyberMed" (secteur santé) subit une attaque ciblée. L'équipe se divise en cellule Red (2-3 étudiants) et cellule Blue (2-3 étudiants) qui travaillent en simultané.

### Infrastructure déployée sur la VM Cible

Joomla 4.2.7 + MySQL — CMS vulnérable (auth bypass, permission bypass) — vulhub/joomla/CVE-2023-23752

Tomcat 8 — Désérialisation de session RCE — vulhub/tomcat/CVE-2020-9484

PostgreSQL — Élévation de privilèges — vulhub/postgres/CVE-2018-1058

ELK Stack (Elasticsearch + Logstash + Kibana) — SIEM pour la cellule Blue — docker-compose ELK

Suricata — IDS pour la détection en temps réel — installé sur la VM Cible

### Organisation des équipes

**Cellule Red (depuis la VM Kali)** : reconnaissance nmap, exploitation Metasploit/Burp Suite, post-exploitation, documentation de la kill chain

**Cellule Blue (sur la VM Cible)** : configuration des règles ELK/Suricata, monitoring en temps réel, détection des attaques Red, analyse Wireshark, rédaction des alertes

**J2-J3** : les deux cellules travaillent en simultané — Red attaque, Blue détecte et défend en temps réel

**J4** : chaque cellule rédige son rapport (attaque / défense), puis rapport consolidé commun

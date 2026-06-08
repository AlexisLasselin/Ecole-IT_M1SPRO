# Rapport Red Team - Jour 1

## Informations générales

IP de la vm cible : 10.156.115.137
IP de la vm Kali : 10.156.115.217

## Reconnaissance

### Scan de ports ouverts

```bash
nmap -sS -sV -O -p- 10.156.115.137
```

**Résultats :**

| Port     | Service      | Version                           |
| -------- | ------------ | --------------------------------- |
| 22/tcp   | ssh          | OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 |
| 5044/tcp | lxi-evntsvc? |                                   |
| 5601/tcp | http         | Elasticsearch Kibana              |
| 9200/tcp | http         | Elasticsearch REST API 8.13.4     |

**OS details**: Linux 4.15 - 5.19

De ce test initial, nous avons identifié les services suivants : SSH (port 22), un service inconnu sur le port 5044, Kibana (port 5601) et Elasticsearch (port 9200).
Nous avons également déterminé que la machine cible fonctionne sous Linux, avec une version du noyau comprise entre 4.15 et 5.19. Ces informations nous permettront de cibler nos prochaines étapes d'exploitation.

### Scan de vulnérabilités

```bash
nmap --script vuln -p22,5601,9200 10.156.115.137
```

**Résultats :**

```sh
PORT     STATE SERVICE
22/tcp   open  ssh
5601/tcp open  esmagent
9200/tcp open  elasticsearch
```

Le scan de vulnérabilités n'a pas révélé de failles exploitables sur les services identifiés. Cependant, nous allons approfondir notre analyse en utilisant d'autres outils et techniques pour tenter de trouver des vulnérabilités potentielles, notamment sur les services Elasticsearch et Kibana, qui sont souvent ciblés par les attaquants. Nous allons également enquêter sur le service inconnu sur le port 5044 pour déterminer sa nature et son potentiel de vulnérabilité.

### Analyse du service inconnu (port 5044)

Pour analyser le service inconnu sur le port 5044, nous avons utilisé la commande `nmap` avec des scripts de détection de services :

```bash
nmap -sV --script=banner -p5044 10.156.115.137
```

**Résultats :**

```sh
PORT     STATE SERVICE      VERSION
5044/tcp open  lxi-evntsvc?
```

Le service sur le port 5044 est identifié comme "lxi-evntsvc", mais sa nature exacte reste incertaine. Nous allons poursuivre notre investigation en utilisant des outils de reconnaissance plus avancés, tels que `netcat` ou `telnet`, pour tenter d'obtenir plus d'informations sur ce service et déterminer s'il présente des vulnérabilités exploitables.

### Analyse des services Elasticsearch et Kibana

Nous avons utilisé des outils spécifiques pour analyser les services Elasticsearch (port 9200) et Kibana (port 5601). Nous avons vérifié les versions de ces services et recherché des vulnérabilités connues associées à ces versions.

### Prochaines étapes

1. **Analyse approfondie des services Elasticsearch et Kibana** : Nous allons utiliser des outils spécifiques pour tester la sécurité de ces services, notamment en recherchant des vulnérabilités connues et en tentant d'exploiter les configurations par défaut.
2. **Investigation du service sur le port 5044** : Nous allons essayer de communiquer avec ce service pour comprendre sa fonction et identifier d'éventuelles failles de sécurité.
3. **Documentation de la kill chain** : Nous allons commencer à documenter nos actions et découvertes dans un rapport structuré, en suivant les étapes de la kill chain (reconnaissance, armement, livraison, exploitation, installation, commande et contrôle, actions sur les objectifs).
4. **Collaboration avec la cellule Blue** : Nous allons partager nos découvertes avec la cellule Blue pour les aider à configurer leurs défenses et à surveiller les activités suspectes en temps réel.

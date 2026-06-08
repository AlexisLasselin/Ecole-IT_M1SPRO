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

| Port     | Service | Version                           |
| -------- | ------- | --------------------------------- |
| 22/tcp   | ssh     | OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 |
| 5601/tcp | http    | Elasticsearch Kibana              |
| 9200/tcp | http    | Elasticsearch REST API 8.13.4     |

### Scan de vulnérabilités

```bash
nmap --script vuln -p22,5601,9200 10.156.115.137
```

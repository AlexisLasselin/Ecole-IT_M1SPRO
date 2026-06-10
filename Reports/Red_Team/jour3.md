# Rapport Red Team – Jour 3

## Débrief de la veille

Les analyses réalisées lors de la journée précédente ont permis d’identifier plusieurs éléments majeurs de l’infrastructure cible :

- Elasticsearch est accessible sans authentification.
- Kibana est également accessible sans authentification et permet l’utilisation des Dev Tools.
- Un CMS Joomla! 4.2.7 est exposé sur le port 8080.
- La machine héberge une stack ELK composée d’Elasticsearch, Kibana et Filebeat.
- Les premières observations indiquent la présence d’un capteur Suricata assurant la supervision du trafic réseau.

À ce stade, aucune compromission directe n’a été identifiée. Toutefois, l’exposition de la stack ELK ainsi que des services applicatifs représente une surface d’attaque importante permettant une analyse approfondie de l’infrastructure et de son niveau de supervision.

L’objectif de cette troisième journée est donc d’analyser les données indexées dans Elasticsearch afin d’évaluer la visibilité du SOC et d’identifier d’éventuels éléments exploitables.

## Cartographie des données Elasticsearch

Une exploration des données disponibles dans Elasticsearch a été réalisée via Kibana Dev Tools afin d’identifier les index actifs et les sources de logs.

```json
GET /.ds-filebeat-*/_search
{
  "size": 1,
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "@timestamp": {
        "order": "desc"
      }
    }
  ]
}
```

## Analyse des données collectées

L’analyse du document retourné permet de mettre en évidence plusieurs informations clés relatives à la supervision de la cible :

- L’adresse IP source `10.156.115.217` correspond à la machine d’attaque.
- L’adresse IP destination `10.156.115.137` correspond à la machine cible.
- Le dataset `suricata.eve` confirme que les événements réseau sont collectés via Suricata.
- Les logs sont centralisés dans Elasticsearch via Filebeat, indiquant une chaîne de collecte opérationnelle.
- Une alerte de faible criticité est générée, liée à une vérification de la journalisation.
- La signature `[FAIBLE] OWASP A09 - Verification Journalisation` indique l’utilisation de règles de détection basées sur OWASP.

Ces éléments confirment que les activités de la machine d’attaque sont visibles et correctement journalisées par le système de supervision.

## Interprétation globale de la supervision

L’analyse des données collectées montre que :

- La stack ELK est pleinement opérationnelle et centralise les logs réseau.
- Suricata détecte et journalise les activités de manière efficace.
- Les événements sont correctement indexés et consultables dans Elasticsearch.
- Le SOC dispose d’une visibilité sur les actions effectuées depuis la machine d’attaque.

Cependant, les alertes observées restent limitées à des événements de faible gravité. Cela peut indiquer soit un périmètre de détection restreint, soit l’absence de corrélation avancée entre les événements.

## Recherche des types de journaux collectés

Afin de mieux comprendre les types de données surveillées par le SOC, une analyse des index disponibles a été réalisée :

```json id="jour3es2"
GET /_cat/indices?v
```

L’analyse de la liste des index met en évidence plusieurs éléments :

- La présence d’un data stream Filebeat contenant un volume important d’événements.
- Plusieurs index dédiés aux alertes de sécurité et à l’observabilité.
- Des index internes liés à Kibana et aux composants système.

L’index Filebeat constitue la source principale de données, regroupant les journaux système, réseau et applicatifs.

## Analyse de la surface d’attaque applicative

En parallèle de l’analyse des logs, un service Apache Tomcat a été identifié sur la cible :

- Apache Tomcat 8.0.43 (port 8081)

Cette version est associée à la vulnérabilité CVE-2020-9484, une faille critique de désérialisation de session pouvant mener à une exécution de code à distance dans certaines conditions.

## Vulnérabilité CVE-2020-9484

La CVE-2020-9484 concerne une vulnérabilité de désérialisation de session dans Apache Tomcat.
Dans certaines configurations spécifiques, notamment lorsque la gestion des sessions persistantes est activée, un attaquant peut potentiellement exploiter ce mécanisme afin d’influencer le processus de désérialisation et conduire à une exécution de code à distance sur le serveur applicatif.

Cette vulnérabilité est critique car elle peut permettre une compromission complète du service Tomcat et potentiellement un accès au système hôte.

## Validation de la surface d’attaque

Dans le cadre de cette évaluation, l’objectif est de déterminer si la configuration du serveur Tomcat est susceptible de présenter les conditions nécessaires à l’exploitation de la CVE-2020-9484.

Cette phase ne constitue pas une exploitation directe, mais une analyse de la surface d’attaque disponible. Elle inclut notamment :

- la confirmation de la version du serveur Tomcat
- l’identification des services HTTP exposés
- l’analyse du comportement du serveur
- l’évaluation de la présence de mécanismes de gestion de sessions

Une première requête de reconnaissance a été effectuée afin d’observer le comportement du service :

```bash
curl -i http://10.156.115.137:8081/
```

## Analyse du comportement

La réponse du serveur confirme l’exposition d’un service Apache Tomcat 8.0.43 sur le port 8081.

À ce stade, aucune preuve d’exploitation n’est observée. L’analyse reste strictement observationnelle et vise uniquement à confirmer la surface d’attaque.

## Scénario d’exploitation potentiel

Sur la base des informations collectées, la vulnérabilité CVE-2020-9484 est considérée comme une hypothèse d’exploitation.

Dans un scénario théorique, si les conditions nécessaires sont réunies (notamment la gestion de sessions persistantes et une configuration vulnérable du mécanisme de désérialisation), un attaquant pourrait influencer le traitement des sessions côté serveur.

Cela pourrait conduire à une exécution de code arbitraire sur le serveur Tomcat.

## Impact potentiel

Si cette vulnérabilité était exploitable, les impacts seraient critiques :

- Exécution de code à distance (RCE)
- Compromission du service Tomcat
- Accès potentiel au système hôte
- Possibilité de pivot vers d’autres composants internes

## Conclusion du Jour 3

L’analyse réalisée durant cette journée met en évidence deux aspects majeurs :

1. La stack ELK permet une visibilité complète des activités réseau, confirmant la présence d’un SOC fonctionnel basé sur Suricata et Filebeat.
2. Un service Apache Tomcat potentiellement vulnérable a été identifié, représentant une surface d’attaque critique.

Ainsi, la cible présente à la fois :

- une bonne capacité de détection (SOC actif)
- deux surfaces d’attaque exploitables (Joomla! et service Tomcat vulnérable)

La prochaine étape consistera à corréler ces éléments afin d’évaluer un scénario de compromission complet de l’infrastructure.

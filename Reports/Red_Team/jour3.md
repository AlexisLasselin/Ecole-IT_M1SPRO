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

## Conclusion de la matinée du jour 3

L’analyse réalisée durant cette journée met en évidence deux aspects majeurs :

1. La stack ELK permet une visibilité complète des activités réseau, confirmant la présence d’un SOC fonctionnel basé sur Suricata et Filebeat.
2. Un service Apache Tomcat potentiellement vulnérable a été identifié, représentant une surface d’attaque critique.

Ainsi, la cible présente à la fois :

- une bonne capacité de détection (SOC actif)
- deux surfaces d’attaque exploitables (Joomla! et service Tomcat vulnérable)

La prochaine étape consistera à corréler ces éléments afin d’évaluer un scénario de compromission complet de l’infrastructure.

## Analyse de la surface applicative – Apache Tomcat

Suite aux analyses réalisées le matin, un service Apache Tomcat 8.0.43 a été identifié sur la cible, exposé sur le port 8081.
Cette version est associée à la vulnérabilité CVE-2020-9484, une faille critique liée à la désérialisation de sessions pouvant conduire à une exécution de code à distance dans certaines conditions de configuration.

L’objectif de cette phase est d’évaluer si le service présente une surface d’attaque compatible avec l’exploitation théorique de cette vulnérabilité.

### Reconnaissance du service Tomcat

Une phase de reconnaissance a été effectuée afin d’observer le comportement du service et d’identifier les informations exposées.

```bash
curl -i http://10.156.115.137:8081/
```

**Résultat :**

```plaintext
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Content-Type: text/html;charset=UTF-8
Transfer-Encoding: chunked
Date: Wed, 10 Jun 2026 12:28:30 GMT
```

La réponse du serveur confirme l’exposition d’un service Apache Tomcat 8.0.43 sur le port 8081, avec une configuration par défaut. Aucune information supplémentaire n’est révélée dans les en-têtes HTTP, ce qui est typique d’une configuration standard de Tomcat.

### Analyse des prérequis d’exploitation

Pour que la CVE-2020-9484 soit exploitable, plusieurs conditions doivent être réunies :

- La gestion de sessions persistantes doit être activée dans la configuration de Tomcat.
- Le mécanisme de désérialisation doit être vulnérable, ce qui est généralement le cas pour les versions antérieures à 8.5.51 et 9.0.31.
- Le service doit être accessible depuis l’extérieur, ce qui est confirmé par la reconnaissance.

Pour vérifier la présence de ces conditions, une analyse plus approfondie de la configuration du serveur serait nécessaire, notamment en examinant les fichiers de configuration de Tomcat (server.xml, context.xml) et en testant le comportement du mécanisme de gestion des sessions.

### Scénario d’exploitation potentiel de la CVE-2020-9484

Dans un scénario théorique, si les conditions d’exploitation sont réunies, un attaquant pourrait envoyer une requête spécialement conçue pour influencer le processus de désérialisation des sessions. Cela pourrait conduire à une exécution de code arbitraire sur le serveur Tomcat, permettant ainsi une compromission complète du service et potentiellement un accès au système hôte.

### Impact potentiel de l’exploitation de la CVE-2020-9484

L’exploitation de cette vulnérabilité pourrait avoir des conséquences graves, notamment :

- Exécution de code à distance (RCE)
- Compromission du service Tomcat
- Accès potentiel au système hôte
- Possibilité de pivot vers d’autres composants internes

### Corrélation avec la supervision observée

Les analyses réalisées précédemment sur la stack ELK ont démontré que les activités réseau sont surveillées et centralisées par le SOC à l'aide de Suricata, Filebeat et Elasticsearch.

Toute tentative d'exploitation visant le service Tomcat serait donc susceptible de générer des événements réseau visibles par les outils de supervision. En revanche, aucune donnée collectée jusqu'à présent ne permet de confirmer la présence d'une journalisation applicative spécifique à Tomcat dans Elasticsearch.

Cette situation suggère que le SOC dispose d'une visibilité satisfaisante sur le trafic réseau mais que la détection d'événements applicatifs liés à Tomcat pourrait être plus limitée.

### Conclusion de l'analyse Tomcat

L'identification d'un serveur Apache Tomcat 8.0.43 constitue un élément important de la surface d'attaque de la cible.

La présence d'une version affectée par la CVE-2020-9484 ne permet pas à elle seule de conclure à une vulnérabilité exploitable, plusieurs conditions de configuration devant être réunies pour permettre une compromission effective.

Néanmoins, compte tenu de l'impact potentiel associé à cette vulnérabilité, le service Tomcat doit être considéré comme une cible prioritaire pour les phases ultérieures de l'évaluation.

Les investigations futures devront permettre de déterminer si les conditions d'exploitation sont effectivement réunies et d'évaluer la capacité du SOC à détecter une activité malveillante visant ce service.

## Identification de services internes potentiellement exploitables

L'analyse de l'environnement et du syllabus de l'épreuve met en évidence la présence potentielle d'un serveur PostgreSQL vulnérable à la CVE-2018-1058. Cette vulnérabilité étant une élévation de privilèges, elle sera étudiée dans un second temps, après l'obtention d'un accès initial à l'application ou à la base de données.

## Pistes d'investigation pour la journée 4

- Analyse approfondie de la configuration du service Tomcat pour évaluer les conditions d'exploitation de la CVE-2020-9484.
- Recherche de mécanismes de gestion de sessions et de désérialisation dans Tomcat.
- Évaluation de la capacité du SOC à détecter des activités malveillantes ciblant le service Tomcat.
- Exploration de la surface d'attaque liée au CMS Joomla! 4.2.7, notamment en recherchant des vulnérabilités exploitables.
- Préparation d'une stratégie d'exploitation pour la CVE-2020-9484 en cas de confirmation de la vulnérabilité.
- Analyse de la configuration de PostgreSQL pour identifier les conditions d'exploitation de la CVE-2018-1058.

# Rapport Blue Team - Jour 1

## Informations générales

**Machine SOC :** 10.156.115.137

Durant cette première journée, l'objectif principal de la cellule Blue Team a été de mettre en place une plateforme de supervision de sécurité capable de collecter, centraliser et visualiser les événements générés sur la machine cible. L'environnement déployé repose sur Elasticsearch, Kibana, Filebeat et Suricata. Une personne de l'équipe est restée connectée en SSH afin d'assurer l'administration continue de la plateforme et le suivi des événements remontés en temps réel.

L'infrastructure a été configurée pour surveiller les activités réseau et les tentatives d'interaction réalisées par la cellule Red Team. Les événements collectés ont ensuite été centralisés dans Elasticsearch afin d'être visualisés et analysés depuis Kibana.

## Mise en place de la supervision

La première étape a consisté à vérifier le bon fonctionnement des services Elasticsearch et Kibana exposés sur les ports 9200 et 5601. Les journaux de sécurité générés par Suricata sont enregistrés dans le fichier `eve.json`. Afin d'exploiter ces données, le module Suricata de Filebeat a été activé puis configuré pour envoyer automatiquement les événements vers Elasticsearch.

Plusieurs vérifications ont été réalisées afin de s'assurer du bon fonctionnement de la chaîne de collecte :

```bash
sudo filebeat test config
sudo filebeat test output
sudo systemctl restart filebeat
```

La présence des événements dans Elasticsearch a été confirmée grâce aux commandes suivantes :

```bash
curl localhost:9200/_cat/indices?v
curl localhost:9200/_cat/data_streams?v
```

Cette phase a permis de valider la collecte des événements réseau et des alertes de sécurité produites par Suricata.

## Détection des activités de reconnaissance

Les premières actions de la Red Team ont été identifiées sous la forme de scans Nmap visant à cartographier les services exposés sur la machine cible. Les journaux collectés ont permis d'observer plusieurs tentatives de reconnaissance ciblant notamment Elasticsearch, Kibana ainsi que le service SSH.

Afin d'améliorer la détection, un ensemble de règles personnalisées a été développé dans le fichier `cybermed.rules`. Ces règles permettent de catégoriser plus clairement les événements observés et d'associer un niveau de gravité aux différentes activités détectées.

Parmi les événements observés figurent notamment :

* Scan Nmap SYN
* Enumeration Services
* Connexion SSH
* Accès aux interfaces Joomla
* Accès aux interfaces Tomcat
* Connexions PostgreSQL

Ces événements ont été associés à différents niveaux de gravité afin de faciliter leur interprétation dans Kibana.

## Amélioration des règles Suricata

Plusieurs règles personnalisées ont été créées afin de compléter les signatures existantes de Suricata. L'objectif était de détecter les techniques les plus courantes utilisées lors d'une phase de reconnaissance ou d'exploitation.

Les règles développées couvrent notamment :

* les scans de ports ;
* l'énumération de services ;
* les accès SSH ;
* l'accès aux interfaces d'administration Joomla ;
* l'accès aux interfaces Tomcat ;
* les connexions PostgreSQL ;
* plusieurs catégories du Top 10 OWASP ;
* certaines tentatives de WebShell ;
* certaines tentatives de Reverse Shell.

Les signatures ont été organisées avec différents niveaux de sévérité afin de rendre les alertes plus compréhensibles pour les analystes SOC.

## Problèmes rencontrés

L'un des principaux problèmes rencontrés au cours de cette journée concerne la visualisation des événements dans Kibana. Bien que les alertes soient correctement générées par Suricata, certaines informations importantes n'étaient pas affichées de manière explicite dans les tableaux de bord.

Les champs contenant le nom exact de l'attaque, la gravité ou certaines informations réseau n'étaient pas toujours visibles directement dans l'interface. Plusieurs ajustements ont donc été réalisés afin d'améliorer l'affichage et de rendre les événements plus compréhensibles.

Des travaux complémentaires seront nécessaires afin de produire un tableau de bord SOC plus lisible, mettant en évidence les catégories d'attaques, les niveaux de criticité et les adresses IP impliquées.

## Corrélation avec les activités Red Team

Les informations remontées par la plateforme de supervision ont permis de confirmer plusieurs activités réalisées par la cellule Red Team. Les scans Nmap effectués depuis la machine Kali ont été observés dans les journaux Suricata et corrélés avec les résultats communiqués par la cellule offensive.

Les services identifiés par la Red Team correspondent aux services effectivement exposés par la machine cible :

* SSH ;
* Elasticsearch ;
* Kibana ;
* Filebeat/Beats ;
* Joomla ;
* Tomcat ;
* PostgreSQL.

Cette corrélation confirme le bon fonctionnement de la chaîne de supervision mise en place.

## Veille de sécurité et vulnérabilités

Une phase de veille a également été réalisée afin d'identifier les vulnérabilités publiques associées aux services déployés dans l'environnement de test.

Les recherches se sont concentrées sur :

* Elasticsearch 8.13.4 ;
* Kibana 8.13.4 ;
* Joomla 4.2.7 ;
* Tomcat 8 ;
* PostgreSQL 9.6.

Les informations ont été recoupées avec les bases de référence telles que :
* Elastic Security Advisories ;
* OWASP Top 10.

Cette phase permettra d'orienter les futures activités de surveillance et de renforcer progressivement les mécanismes de détection.

## Conclusion

Cette première journée a permis de mettre en place les principaux composants de supervision de l'infrastructure et de valider leur fonctionnement. Les activités de reconnaissance menées par la cellule Red Team ont pu être observées et corrélées avec les événements collectés par Suricata.

Les prochaines étapes consisteront à améliorer la visualisation des alertes dans Kibana, enrichir les règles de détection, renforcer la catégorisation des événements et poursuivre la surveillance des différentes tentatives d'exploitation réalisées sur les services exposés.

# Rapport Blue Team – Jour 2

## Informations générales

**Machine SOC :** 10.156.115.137

Cette deuxième journée a été consacrée à l'amélioration de la supervision de l'infrastructure ainsi qu'à l'observation des premières tentatives d'analyse approfondie menées par la cellule Red Team. L'objectif principal était de renforcer la visibilité sur les activités réalisées contre les services exposés tout en conservant un environnement suffisamment accessible pour permettre la poursuite des exercices d'exploitation.

Au cours de la matinée, plusieurs modifications ont été réalisées sur la plateforme afin d'améliorer la collecte et la visualisation des événements de sécurité. Une attention particulière a été portée à Elasticsearch, Kibana et aux différents services applicatifs déployés sur la machine cible.

## Observation des activités Red Team

Les journaux collectés ont confirmé la poursuite des activités de reconnaissance initiées la veille. La cellule Red Team a notamment effectué plusieurs requêtes sur Elasticsearch afin d'obtenir des informations sur la configuration du cluster, les index présents ainsi que l'état général de la plateforme.

Les accès observés ont permis de constater que le cluster Elasticsearch était accessible sans authentification. Les requêtes envoyées par la Red Team ont notamment permis d'obtenir la liste des index présents ainsi que l'état de santé du cluster. Cette situation représente un risque important puisqu'elle facilite l'identification de la structure interne de l'environnement et peut permettre à un attaquant de cibler plus efficacement les composants exposés.

Parallèlement, plusieurs accès à Kibana ont été détectés. Les analystes ont constaté que la cellule Red Team était en mesure de consulter l'interface web et d'accéder à certaines fonctionnalités de navigation et d'exploration des données. Cette exposition confirme l'importance de renforcer les mécanismes d'authentification et de contrôle d'accès au sein de la stack ELK.

## Évolution de l'infrastructure supervisée

Durant cette deuxième journée, de nouveaux services ont été intégrés à l'environnement de test afin de fournir des surfaces d'attaque supplémentaires aux équipes offensives. Plusieurs conteneurs Docker ont été déployés, notamment des environnements Joomla!, Tomcat et PostgreSQL volontairement vulnérables dans le cadre pédagogique du projet.

La présence de ces services a nécessité l'adaptation des mécanismes de détection afin d'assurer une surveillance cohérente de l'ensemble de l'environnement. Les règles Suricata ont ainsi été enrichies afin de permettre la détection de plusieurs scénarios d'attaque visant ces applications.

Des catégories spécifiques ont été créées pour distinguer les événements liés à Joomla!, Tomcat, PostgreSQL, aux activités de reconnaissance réseau ainsi qu'aux tentatives d'accès aux différents services exposés.

## Amélioration des règles de détection

Afin d'obtenir une meilleure visibilité sur les activités observées, plusieurs règles personnalisées ont été développées et intégrées à Suricata.

Ces règles permettent notamment d'identifier :

* les scans Nmap ;
* les activités d'énumération de services ;
* les connexions SSH ;
* les accès aux interfaces d'administration Joomla ;
* les accès aux interfaces Tomcat ;
* les connexions PostgreSQL ;
* certaines tentatives de reverse shell ;
* plusieurs scénarios inspirés du Top 10 OWASP.

L'objectif n'était pas uniquement de générer des alertes mais également de rendre les événements plus compréhensibles pour les analystes SOC. Des niveaux de gravité ont donc été associés aux différentes catégories afin de faciliter la priorisation des événements observés dans Kibana.

## Difficultés de visualisation dans Kibana

L'une des principales difficultés rencontrées au cours de cette journée concerne l'exploitation des données dans Kibana.

Bien que les événements soient correctement collectés et indexés dans Elasticsearch, plusieurs problèmes d'affichage ont été observés lors de la création des tableaux de bord. Certains champs importants n'apparaissaient pas systématiquement dans les visualisations, notamment les informations liées aux adresses IP sources, aux catégories d'attaques ou aux niveaux de gravité.

Cette situation a nécessité plusieurs phases de vérification de la chaîne de traitement des logs. Les équipes ont analysé les index Elasticsearch, les champs ECS générés par Filebeat ainsi que la structure des événements Suricata afin d'identifier l'origine des incohérences observées.

À ce stade du projet, la collecte des événements est fonctionnelle mais les tableaux de bord nécessitent encore des ajustements afin d'obtenir une représentation claire et exploitable des alertes de sécurité.

## Gestion de l'accès aux services

Dans une logique de défense, plusieurs tests de filtrage réseau ont été réalisés afin de limiter l'accès aux services exposés. Des règles de pare-feu ont été mises en place pour contrôler les communications vers les conteneurs Docker hébergeant les différents services vulnérables.

Toutefois, ces restrictions ont temporairement été assouplies afin de permettre à la cellule Red Team de poursuivre ses investigations et ses tentatives d'exploitation dans le cadre du scénario pédagogique. L'objectif était de maintenir un équilibre entre la sécurisation de l'infrastructure et la poursuite des exercices offensifs nécessaires à l'évaluation des capacités de détection de la Blue Team.

## Veille sécurité et analyse des vulnérabilités

Une phase de veille a également été menée concernant les technologies déployées dans l'environnement.

Les recherches ont porté sur :

* Elasticsearch 8.13.4 ;
* Kibana 8.13.4 ;
* Joomla! 4.2.7 ;
* Apache HTTP Server 2.4.54 ;
* PHP 7.4.33 ;
* PostgreSQL 9.6 ;
* Tomcat 8.


Une attention particulière a été portée à la vulnérabilité CVE-2023-23752 affectant Joomla! 4.2.7. Les analyses réalisées montrent que la surface concernée est présente sur l'environnement mais qu'aucune exploitation directe n'a été observée durant cette journée.

## Recommandations

Les travaux réalisés lors de cette phase ont permis d'améliorer la visibilité des événements de sécurité au sein du SOC. Les règles Suricata personnalisées développées pour les principaux scénarios d'attaque doivent être régulièrement révisées afin de limiter les faux positifs et de conserver un niveau de détection pertinent.

Il est recommandé de poursuivre l'enrichissement des alertes en associant systématiquement les adresses IP sources, les niveaux de sévérité et les catégories d'attaque. Cette approche facilite l'analyse des événements et permet une meilleure priorisation des incidents.

La supervision doit également être renforcée autour des applications exposées, notamment Joomla, Tomcat et PostgreSQL. Une attention particulière doit être portée aux tentatives d'exploitation liées aux vulnérabilités du Top 10 OWASP ainsi qu'aux activités de post-exploitation telles que les reverse shells ou les webshells.

Enfin, il est recommandé de mettre en place une procédure de revue quotidienne des tableaux de bord Kibana afin de vérifier le bon fonctionnement de la collecte des journaux, d'identifier rapidement les anomalies et de garantir une détection efficace des activités menées par la Red Team.


## Conclusion

Cette deuxième journée a permis d'améliorer significativement la capacité de supervision de l'infrastructure tout en observant les premières phases d'investigation approfondie réalisées par la cellule Red Team.

Les activités menées ont confirmé l'exposition de plusieurs services sensibles ainsi que l'importance de la centralisation des journaux dans Elasticsearch. Les travaux ont également mis en évidence les limites actuelles des tableaux de bord Kibana, qui nécessitent encore des ajustements afin de fournir une visualisation complète des événements collectés.

Les prochaines étapes consisteront à finaliser les visualisations SOC, enrichir les mécanismes de détection, renforcer les règles Suricata et poursuivre la surveillance des tentatives d'exploitation ciblant les différents services déployés dans l'environnement.


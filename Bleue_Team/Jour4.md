# Rapport Blue Team - Jour 4

## Informations générales

IP de la VM CyberMed : 10.156.115.137

Au cours de cette quatrième journée, les travaux se sont concentrés sur le renforcement de la sécurité de la plateforme ELK (Elasticsearch, Kibana et Filebeat). Après plusieurs jours consacrés à la collecte et à la visualisation des événements de sécurité, l'objectif principal était de sécuriser l'accès aux outils de supervision tout en garantissant la continuité de la collecte des journaux et le fonctionnement des tableaux de bord Kibana.

## Sécurisation d'Elasticsearch et Kibana

Une analyse de la configuration a montré que les services Elasticsearch et Kibana étaient accessibles sans authentification. Dans cette configuration, toute personne ayant accès au réseau pouvait potentiellement consulter les données collectées, les index Elasticsearch ou les tableaux de bord Kibana.

Dans un premier temps, l'équipe a envisagé de bloquer complètement les ports 9200 et 5601 afin de restreindre l'accès aux services Elasticsearch et Kibana. Après réflexion, cette solution a été écartée car elle aurait compliqué l'accès à l'interface de supervision pour les analystes SOC et limité certaines opérations d'administration.

La décision a donc été prise de mettre en place une authentification native Elasticsearch afin de contrôler l'accès aux services. Cette approche permet de conserver l'accessibilité des plateformes tout en exigeant une authentification préalable avant toute consultation ou modification des données.

## Difficultés rencontrées

L'activation de la sécurité Elasticsearch a entraîné plusieurs problèmes de fonctionnement. Après l'activation du module de sécurité, Kibana affichait le message « Kibana server is not ready yet » et ne permettait plus l'accès à l'interface web.

Les investigations ont montré que Kibana tentait de s'authentifier avec un compte utilisateur ne disposant pas des autorisations nécessaires pour accéder aux index système. Plusieurs erreurs d'authentification ont également été observées concernant les comptes administrateurs créés lors des premiers essais de configuration.

Une phase de diagnostic a été menée afin de vérifier l'état des utilisateurs Elasticsearch, des mots de passe et des permissions associées. Le mot de passe du compte administrateur intégré a été réinitialisé afin de reprendre le contrôle du cluster et de corriger progressivement les erreurs de configuration.

## Mise en place des comptes de service

Une fois l'accès administrateur restauré, un compte dédié à l'administration SOC a été créé afin de limiter l'utilisation du compte administrateur par défaut. Un compte système spécifique a également été configuré pour permettre à Kibana de communiquer correctement avec Elasticsearch.

Les paramètres de connexion ont ensuite été mis à jour dans les différents composants de l'architecture afin de garantir le fonctionnement de l'ensemble de la chaîne de supervision.

Cette opération a permis de conserver les index existants, les tableaux de bord Kibana, les règles Suricata et les données historiques collectées lors des précédentes journées.

## Durcissement des applications Web

Dans le cadre des activités de sécurisation de l'infrastructure CyberMed, une analyse des identifiants par défaut des applications exposées a été réalisée.

### Sécurisation de Tomcat: http://10.156.115.137:8081/manager/html

L'application Tomcat utilisait les identifiants par défaut : 

* Utilisateur : tomcat
* Mot de passe : tomcat

Ces identifiants étant largement connus des attaquants, ils représentent un risque important d'accès non autorisé à l'interface d'administration. Le mot de passe a été remplacé par un mot de passe robuste afin de renforcer la sécurité du service.

### Sécurisation de Joomla : http://10.156.115.137:8080/administrator/index.php

L'application Joomla utilisait le compte administrateur avec les identifiants suivants : 

* Utilisateur : admin
* Mot de passe : admin

Afin de réduire les risques liés à l'utilisation d'identifiants faibles ou par défaut, le mot de passe du compte administrateur a été modifié et remplacé par un mot de passe complexe répondant aux exigences de sécurité de Joomla.

### Objectif de la mesure

Cette opération de durcissement vise à limiter les risques de compromission liés à l'utilisation d'identifiants par défaut, fréquemment exploités lors des phases de reconnaissance et d'attaque. Elle permet de renforcer le contrôle d'accès aux interfaces d'administration et de réduire la surface d'attaque de l'infrastructure.

## Validation du fonctionnement

Après correction de la configuration, plusieurs tests ont été réalisés afin de confirmer le bon fonctionnement de la plateforme.

Les services Elasticsearch et Kibana ont été relancés avec succès. Les journaux système ont confirmé le démarrage normal des différents composants ainsi que la disponibilité de l'interface Kibana.

Les tests de connexion à Elasticsearch ont permis de valider le fonctionnement de l'authentification. Les données collectées par Filebeat continuaient d'être transmises correctement vers Elasticsearch et restaient consultables depuis Kibana.

L'ensemble de la plateforme ELK est désormais protégé par authentification tout en conservant les fonctionnalités de supervision nécessaires aux activités du SOC.

## Recommandations

Il est recommandé de conserver le compte administrateur intégré uniquement comme compte de secours et d'utiliser des comptes dédiés pour les opérations quotidiennes du SOC.

La mise en place d'une politique de mots de passe robuste doit être maintenue afin de limiter les risques de compromission des accès à la plateforme de supervision.

Il est également recommandé de restreindre l'accès réseau aux ports 9200 et 5601 aux seules adresses IP autorisées lorsque l'architecture finale sera stabilisée. Durant cette phase du projet, le choix a été fait de privilégier l'authentification sur les interfaces Elasticsearch et Kibana plutôt qu'un blocage réseau complet afin de conserver la souplesse d'administration nécessaire aux opérations du SOC.

Les mots de passe par défaut des applications exposées doivent être systématiquement remplacés dès le déploiement de nouveaux services afin de réduire les risques de compromission.

Enfin, la configuration d'une clé de chiffrement Kibana ainsi que la création de rôles spécifiques avec des privilèges limités permettraient d'améliorer davantage le niveau de sécurité global de la solution.

## Conclusion

Cette quatrième journée a permis de renforcer significativement la sécurité de l'infrastructure de supervision en mettant en place une authentification centralisée sur Elasticsearch et Kibana. Le durcissement des applications Web exposées, notamment Tomcat et Joomla, a également permis de réduire la surface d'attaque de l'environnement CyberMed. Malgré plusieurs difficultés liées à la gestion des comptes et des permissions, l'ensemble des services a été rétabli avec succès sans perte de données ni interruption durable de la collecte des journaux. L'infrastructure est désormais plus adaptée à un fonctionnement de type SOC tout en restant opérationnelle pour les prochaines phases de tests Red Team.

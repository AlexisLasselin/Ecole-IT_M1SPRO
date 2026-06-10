# Rapport Blue Team - Jour 3

## Informations générales

IP de la VM CyberMed : 10.156.115.137

IP Red Team : 10.156.115.217 

IP Administrateur : 10.156.115.183 et 10.156.115.139

## Reconstruction de l'infrastructure de supervision

La journée a débuté par une réinitialisation de l'environnement de supervision afin de repartir d'une configuration propre. Les anciens index Elasticsearch et les anciennes données de démonstration ont été supprimés afin d'éviter toute confusion entre les événements historiques et les nouveaux événements générés pendant les exercices Red Team.

Cette opération a permis de reconstruire une chaîne de collecte cohérente composée de Suricata, Filebeat, Elasticsearch et Kibana. L'objectif principal était de garantir une remontée fiable des événements de sécurité et une visualisation claire des activités observées sur le réseau.

Une attention particulière a été portée à la configuration de Filebeat. Plusieurs erreurs de syntaxe ont été identifiées dans les modules de collecte, empêchant le démarrage correct du service. Après correction de la configuration et validation des fichiers YAML, la collecte des journaux Suricata a été rétablie avec succès.

## Enrichissement des événements de sécurité

Afin d'améliorer la lisibilité des événements dans Kibana, un enrichissement des données a été mis en place directement dans Filebeat. Les événements remontés contiennent désormais des informations complémentaires permettant d'identifier rapidement l'environnement surveillé.

Les journaux incluent notamment le nom du serveur CyberMed ainsi que l'adresse IP de la machine cible. Cette approche facilite l'identification des événements dans Kibana et permet une meilleure contextualisation des alertes lors des analyses réalisées par le SOC.

Les adresses IP de la Red Team et de l'administrateur ont également été identifiées afin de faciliter la distinction entre les activités légitimes et les activités offensives observées sur le réseau.

## Amélioration des règles de détection

Les règles Suricata ont été entièrement revues afin d'obtenir des alertes plus cohérentes et plus exploitables. Les signatures développées couvrent désormais plusieurs catégories d'attaques observables au cours des exercices Red Team.

Les détections concernent notamment les activités de reconnaissance réseau, les tentatives d'accès aux services exposés, les attaques visant Joomla, Tomcat et PostgreSQL, les vulnérabilités du Top 10 OWASP ainsi que les mécanismes de post-exploitation tels que les webshells et les reverse shells.

Une classification homogène des alertes a également été mise en place. Chaque événement dispose désormais d'un niveau de priorité compris entre 1 et 5. Cette normalisation permet de distinguer rapidement les événements les plus critiques et de faciliter leur traitement dans le SOC.

## Optimisation des tableaux de bord Kibana

Une partie importante des travaux a été consacrée à l'amélioration des tableaux de bord Kibana. Les visualisations ont été ajustées afin de mettre en évidence les informations réellement utiles à l'analyse des incidents.

Les événements générés par certaines règles informatives, notamment les signatures associées à Spotify ou à d'autres applications légitimes, ont été identifiés comme des sources importantes de bruit. Ces événements n'apportaient pas de valeur opérationnelle dans le contexte du projet et perturbaient la lecture des tableaux de bord.

Des actions de nettoyage ont donc été engagées afin de réduire les faux positifs et d'améliorer la pertinence des informations affichées. L'objectif était de faire ressortir en priorité les activités liées aux attaques et aux comportements anormaux observés sur le réseau.

Les visualisations ont également été adaptées afin d'afficher clairement les adresses IP sources, les adresses IP de destination, les catégories d'attaques et les niveaux de sévérité associés à chaque événement.

## Validation de la chaîne de détection

Plusieurs tests ont été réalisés afin de vérifier le bon fonctionnement de l'infrastructure de détection. Les événements générés par les activités Red Team ont été observés dans les journaux Suricata puis suivis tout au long de la chaîne de traitement jusqu'à leur affichage dans Kibana.

Cette validation a permis de confirmer le bon fonctionnement des composants de collecte, de traitement et de visualisation. Les alertes remontent désormais correctement dans Elasticsearch et peuvent être consultées en temps réel depuis les tableaux de bord du SOC.

## Recommandations

Les améliorations apportées au cours de cette journée ont permis d'obtenir une infrastructure de supervision plus stable et plus lisible. Il est recommandé de poursuivre le travail de réduction des faux positifs afin de limiter le bruit généré par les applications légitimes et de concentrer l'attention des analystes sur les événements réellement critiques.

Il est également conseillé de poursuivre l'enrichissement des événements remontés dans Elasticsearch afin d'améliorer la corrélation des alertes et de faciliter les investigations. La mise en place de nouvelles visualisations dédiées aux activités Red Team permettrait également d'améliorer la visibilité des actions offensives menées pendant les exercices.

Enfin, une surveillance régulière des services Suricata, Filebeat, Elasticsearch et Kibana doit être maintenue afin de garantir la continuité de la collecte des journaux et la disponibilité des capacités de détection du SOC.

## Conclusion

Cette troisième journée a permis de stabiliser et d'améliorer significativement l'infrastructure de supervision du SOC CyberMed. Les problèmes de collecte et de visualisation des journaux ont été corrigés, les règles de détection ont été optimisées et les tableaux de bord Kibana ont été rendus plus lisibles et plus pertinents pour l'analyse des activités de la Red Team. La chaîne de traitement composée de Suricata, Filebeat, Elasticsearch et Kibana fonctionne désormais correctement et permet une meilleure visibilité des événements de sécurité observés sur le réseau.

# Rapport Red Team - Jour 4

## Débrief de la veille

La journée précédente a permis de consolider la compréhension de l’infrastructure cible et de préciser les principales surfaces d’attaque exploitables :

- La stack ELK (Elasticsearch, Kibana et Filebeat) est accessible sans authentification, ce qui permet une consultation directe des logs et des événements de sécurité.
- Les analyses réalisées dans Kibana ont confirmé la présence d’un SOC fonctionnel basé sur Suricata, avec une chaîne de collecte des événements de type : /var/log/suricata/eve.json → Filebeat → Elasticsearch.
- Les activités menées depuis la machine d’attaque sont correctement visibles dans les journaux, ce qui confirme que le SOC dispose d’une visibilité réseau opérationnelle.
- Deux surfaces d’attaque applicatives majeures ont été identifiées :
  - Joomla! 4.2.7 exposé sur le port 8080 ;
  - Apache Tomcat 8.0.43 exposé sur le port 8081.
- L’analyse du service Tomcat a mis en évidence une correspondance potentielle avec la CVE-2020-9484, une vulnérabilité de désérialisation de sessions pouvant conduire à une exécution de code à distance sous certaines conditions de configuration.
- Aucune exploitation effective n’a toutefois été confirmée à ce stade : les investigations sont restées dans une phase d’évaluation de la surface d’attaque et des prérequis techniques.

## Objectifs du jour

L’objectif principal de cette journée est de poursuivre l’évaluation des surfaces d’attaque identifiées lors des phases précédentes et de déterminer lesquelles présentent les meilleures perspectives de compromission.

Les travaux seront organisés autour des axes suivants :

### 1. Validation de l’hypothèse d’exploitation de la CVE-2020-9484

- Analyser plus en détail le comportement et la configuration du service Apache Tomcat 8.0.43.
- Rechercher des indices permettant de confirmer ou d’infirmer la présence des conditions nécessaires à l’exploitation de la CVE-2020-9484.
- Évaluer la faisabilité théorique d’une compromission du serveur applicatif à travers cette vulnérabilité.

### 2. Exploration du CMS Joomla! 4.2.7

- Identifier les composants, modules et extensions exposés par l’application.
- Rechercher d’éventuelles vulnérabilités connues associées à la version déployée.
- Évaluer la présence de faiblesses applicatives pouvant conduire à un accès non autorisé ou à une compromission du CMS.

### 3. Évaluation des capacités de détection du SOC

- Continuer la surveillance des événements remontés dans Kibana et Elasticsearch.
- Vérifier si les activités réalisées durant les phases d’investigation génèrent des alertes de sécurité.
- Identifier les éventuels écarts entre les actions réalisées et les événements effectivement détectés par le SOC.

## Plan d’action

Afin d’optimiser les investigations, les travaux seront menés selon l’ordre de priorité suivant :

1. Analyse approfondie du service Apache Tomcat.
2. Exploration de la surface applicative Joomla!.
3. Vérification des événements remontés dans Elasticsearch et corrélation avec les actions réalisées.
4. Identification d’éventuelles nouvelles pistes d’exploitation pour les phases ultérieures de l’évaluation.

Oui, tout à fait. Le plus utile est de le faire à partir de **ce que tu as réellement observé**, afin que ton rapport reste crédible.

Je te propose cette structure :

## Investigation approfondie du service Tomcat

### Objectif de l'analyse

Après avoir identifié un serveur Apache Tomcat 8.0.43 exposé sur le port 8081, une phase d'investigation complémentaire est menée afin d'évaluer plus précisément sa surface d'attaque et les conditions potentielles d'exploitation associées à la CVE-2020-9484.

L'objectif n'est pas de confirmer immédiatement l'exploitation de la vulnérabilité, mais de déterminer si le service présente des caractéristiques compatibles avec ce scénario.

### Analyse des points d'accès exposés

Une série de requêtes de reconnaissance a été effectuée afin d'identifier les ressources accessibles et d'observer le comportement du serveur.

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

L'analyse des en-têtes HTTP confirme la présence d'un serveur Apache Tomcat. Aucune information détaillée concernant la configuration interne ou les composants déployés n'est directement exposée dans la réponse.

Cette limitation réduit les possibilités de collecte d'informations passives mais n'exclut pas la présence de fonctionnalités ou de composants vulnérables.

### Analyse des prérequis liés à la CVE-2020-9484

La vulnérabilité CVE-2020-9484 nécessite plusieurs conditions techniques spécifiques pour être exploitable.

Parmi les principaux prérequis figurent :

- l'utilisation d'un mécanisme de persistance des sessions ;
- une configuration vulnérable du traitement des objets sérialisés ;
- l'existence d'un contexte permettant l'influence des données de session.

À ce stade de l'évaluation, les informations collectées ne permettent pas de confirmer la présence de ces éléments. Une analyse plus approfondie de la configuration interne du serveur serait nécessaire pour conclure.

### Évaluation du niveau de risque

Bien que l'exploitabilité de la CVE-2020-9484 ne puisse pas être confirmée à ce stade, plusieurs facteurs justifient une attention particulière :

- la version observée de Tomcat est concernée par la vulnérabilité ;
- le service est directement exposé sur le réseau ;
- une compromission du serveur applicatif pourrait avoir un impact important sur l'ensemble du système.

Le risque associé à ce service est donc considéré comme significatif et mérite des investigations complémentaires.

### Conclusion intermédiaire

L'investigation du service Apache Tomcat confirme la présence d'une surface d'attaque potentiellement sensible. Toutefois, les éléments actuellement disponibles ne permettent pas de démontrer que les conditions d'exploitation de la CVE-2020-9484 sont réunies.

Le service reste néanmoins une cible prioritaire pour les phases ultérieures de l'évaluation en raison de l'impact potentiel associé à une compromission réussie.

## Priorisation des vecteurs d'attaque

À ce stade de l’évaluation, plusieurs surfaces d’attaque ont été identifiées au sein de l’infrastructure cible :

- Apache Tomcat 8.0.43 (CVE-2020-9484)
- CMS Joomla! 4.2.7
- Stack ELK accessible sans authentification
- Présence d’un service PostgreSQL (cible d’escalade)

Afin d’optimiser la démarche d’exploitation, une phase de priorisation est nécessaire pour déterminer le vecteur le plus pertinent pour un accès initial.

## Évaluation des vecteurs identifiés

### Apache Tomcat 8.0.43

Le service Tomcat présente un risque potentiel élevé en raison de la CVE-2020-9484.
Cependant, son exploitation dépend de conditions de configuration spécifiques qui n’ont pas pu être confirmées à ce stade de l’analyse.

Par conséquent, bien que l’impact potentiel soit critique, le niveau d’incertitude associé à l’exploitabilité reste important.

### CMS Joomla! 4.2.7

Le CMS Joomla! constitue une surface d’attaque web exposée et accessible publiquement.
Les CMS de ce type sont fréquemment ciblés en raison :

- de la présence potentielle de composants vulnérables ;
- d’extensions tierces non maintenues ;
- de mauvaises configurations applicatives.

Contrairement à Tomcat, ce vecteur présente une surface d’attaque plus large et plus directement exploitable dans un contexte Red Team.

### PostgreSQL

Le service PostgreSQL identifié dans le cadre du syllabus correspond à une vulnérabilité d’escalade de privilèges (CVE-2018-1058).
Cependant, cette vulnérabilité nécessite un accès préalable au système de base de données, ce qui implique qu’elle ne peut être exploitée qu’après compromission initiale d’un autre service.

### Choix du vecteur d’accès initial

Sur la base de l’analyse comparative, le CMS Joomla! 4.2.7 est retenu comme **vecteur principal d’accès initial**.

Ce choix est motivé par :

- une surface d’attaque directement accessible depuis le réseau externe ;
- une probabilité d’exploitation plus élevée que Tomcat dans un contexte inconnu ;
- une logique cohérente de chaîne d’attaque (web → système → base de données).

Tomcat est conservé comme vecteur secondaire potentiel en cas de blocage sur la voie principale.

### Préparation de la phase d’accès initial

La prochaine étape de l’évaluation consiste à :

- approfondir l’analyse du CMS Joomla! 4.2.7 ;
- identifier les composants, extensions et pages exposées ;
- rechercher des vulnérabilités exploitables connues ;
- déterminer un éventuel point d’entrée permettant une compromission initiale.

Cette phase marquera la transition entre l’analyse de surface et une phase d’exploitation contrôlée dans le cadre de l’évaluation Red Team.

## Validation active du vecteur d’accès initial (CMS Joomla! 4.2.7)

L’analyse fonctionnelle de l’application met en évidence une surface d’attaque typique d’un CMS :

- pages dynamiques générées côté serveur ;
- possibilité de composants additionnels (plugins / extensions) ;
- présence probable de fonctionnalités d’administration ;
- exposition potentielle de répertoires et fichiers sensibles.

Dans ce contexte, les CMS constituent généralement une cible privilégiée en raison de leur complexité et de la fréquence des vulnérabilités liées aux extensions tierces.

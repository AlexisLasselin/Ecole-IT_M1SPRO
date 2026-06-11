
## Durcissement des applications Web

Dans le cadre des activités de sécurisation de l'infrastructure CyberMed, une analyse des identifiants par défaut des applications exposées a été réalisée.

### Sécurisation de Tomcat

L'application Tomcat utilisait les identifiants par défaut :

* Utilisateur : tomcat
* Mot de passe : tomcat

Ces identifiants étant largement connus des attaquants, ils représentent un risque important d'accès non autorisé à l'interface d'administration. Le mot de passe a été remplacé par un mot de passe robuste afin de renforcer la sécurité du service.

### Sécurisation de Joomla

L'application Joomla utilisait le compte administrateur avec les identifiants suivants :

* Utilisateur : admin
* Mot de passe : admin

Afin de réduire les risques liés à l'utilisation d'identifiants faibles ou par défaut, le mot de passe du compte administrateur a été modifié et remplacé par un mot de passe complexe répondant aux exigences de sécurité de Joomla.

### Objectif de la mesure

Cette opération de durcissement vise à limiter les risques de compromission liés à l'utilisation d'identifiants par défaut, fréquemment exploités lors des phases de reconnaissance et d'attaque. Elle permet de renforcer le contrôle d'accès aux interfaces d'administration et de réduire la surface d'attaque de l'infrastructure.

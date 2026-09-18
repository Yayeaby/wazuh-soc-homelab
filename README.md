# Wazuh SOC Home-Lab

Simulation d'un environnement SOC (Security Operations Center) avec Wazuh (SIEM open source), construite pour s'entraîner à la détection d'intrusions, au threat hunting et à la classification d'attaques selon le framework MITRE ATT&CK.

Ce home-lab reproduit une chaîne d'attaque réaliste en 4 scénarios progressifs, du point d'entrée initial jusqu'à la propagation interne, avec analyse des alertes générées et mesure du temps de détection pour chacun.

## Objectifs du projet

- Déployer et configurer un SIEM (Wazuh) en environnement virtualisé
- Simuler des attaques réelles (brute force, exploitation web, malware, mouvement latéral)
- Analyser la détection, la corrélation d'événements et la classification MITRE ATT&CK
- Mesurer le délai entre le lancement d'une attaque et sa détection

## Architecture du lab

Trois machines composent le lab :

- Kali Linux (Red Team) : machine attaquante, outils Hydra et wordlists. Elle attaque la machine cible.
- wazuhagent (machine cible), adresse 192.168.121.175 : machine Ubuntu avec l'agent Wazuh installé, héberge aussi DVWA en conteneur Docker. Une fois compromise, elle sert de point de pivot vers le serveur.
- wazuhserver : Ubuntu Server avec l'installation Wazuh complète (Manager, Indexer, Dashboard). Reçoit les logs de l'agent et centralise la détection.

## Scénarios réalisés

### Scénario 1 — Attaque brute force SSH

Attaque automatisée par force brute (Hydra avec la wordlist rockyou.txt) contre le service SSH de la machine cible.

- Détection : règle 5763, "sshd: brute force trying to get access to the system", niveau 10
- Volume : 74 événements en quelques secondes, 938 alertes au total (787 échecs, 8 succès)
- Temps de détection : environ 6 minutes
- Voir le dossier scenario-1-bruteforce

### Scénario 2 — Exploitation web (DVWA)

Injection SQL et XSS réfléchi sur l'application volontairement vulnérable DVWA, en conteneur Docker.

- Injection SQL (1' OR '1'='1) permettant l'extraction de tous les utilisateurs de la base
- XSS réfléchi (script alert test) avec exécution confirmée côté navigateur
- Variantes testées : UNION SELECT, contournement d'authentification, XSS via balise img onerror
- Détection : règle 31106, "A web attack returned code 200", et règle 31101
- Temps de détection : environ 1 seconde
- Voir le dossier scenario-2-dvwa

### Scénario 3 — Dépôt de malware simulé (EICAR et VirusTotal)

Dépôt du fichier de test standard EICAR (inoffensif mais détecté par tous les antivirus) sur la machine cible.

- Détection via le module FIM (File Integrity Monitoring) combiné à l'intégration VirusTotal de Wazuh
- 65 moteurs antivirus sur VirusTotal identifient le fichier comme malveillant
- Détection : règle 87105, "VirusTotal: Alert - 65 engines detected this file", niveau 12
- Temps de détection : environ 15 secondes (latence de l'API VirusTotal)
- Voir le dossier scenario-3-malware

### Scénario 4 — Mouvement latéral

Utilisation de la machine cible déjà compromise comme point de rebond pour attaquer le serveur Wazuh lui-même.

- Création d'un compte cible (adminweb) sur le serveur
- Installation de Hydra depuis la machine pivot (et non depuis Kali), puis attaque SSH vers le serveur
- Mot de passe trouvé : adminweb / 123456
- Détection : classification MITRE ATT&CK automatique, tactiques Credential Access et Lateral Movement, techniques T1110.001 (Password Guessing) et T1021.004 (Remote Services: SSH), règle 5760
- Temps de détection : environ 17 secondes
- Voir le dossier scenario-4-lateral-movement

## Synthèse des temps de détection

- Scénario 1, brute force SSH : lancement à 12:27:24, détection à 12:33:11 (règle 5763), délai d'environ 6 minutes
- Scénario 2, injection SQL et XSS : requête envoyée à 20:11:15, détection à 20:11:16 (règle 31106), délai d'environ 1 seconde
- Scénario 3, malware simulé : dépôt du fichier à 23:04:19, détection à 23:04:34 (règle 87105), délai d'environ 15 secondes
- Scénario 4, mouvement latéral : lancement à 13:32:45, détection à 13:33:02 (règle 5760, MITRE), délai d'environ 17 secondes

Les attaques réseau et applicatives (brute force, XSS) sont détectées quasi instantanément par le moteur de corrélation local de Wazuh. Le scénario malware, dépendant d'un appel à l'API externe VirusTotal, introduit une latence liée au temps de réponse réseau. Le mouvement latéral, bien que plus complexe, reste détecté en quelques secondes grâce à la surveillance continue des journaux d'authentification.

## Outils et technologies utilisés

- Wazuh (Manager, Indexer, Dashboard), SIEM et XDR open source
- Hydra, pour l'attaque par force brute
- DVWA (Damn Vulnerable Web Application), pour les vulnérabilités web volontaires
- Docker, pour la conteneurisation de DVWA
- EICAR, fichier de test antivirus standard
- VirusTotal API, intégrée à Wazuh pour la détection de malware
- MITRE ATT&CK, framework de classification des tactiques et techniques d'attaque
- FIM (File Integrity Monitoring), pour la surveillance de l'intégrité des fichiers

## Documentation complète

Le rapport détaillé avec l'ensemble des captures commentées est disponible dans le fichier documentation_projet_wazuh.pdf à la racine du dépôt.

## Compétences démontrées

- Déploiement et configuration d'un SIEM en environnement virtualisé
- Analyse de logs et corrélation d'événements de sécurité
- Détection d'intrusions réseau et applicatives (OWASP Top 10)
- Threat hunting et investigation d'incidents
- Classification d'attaques selon MITRE ATT&CK
- Rédaction de rapports d'incidents et de recommandations

Projet réalisé dans le cadre d'un Master 1 Sécurité des Systèmes d'Information.



# Wazuh SOC Home-Lab — Détection d'attaques et analyse MITRE ATT&CK
## Objectif

Ce projet consiste à déployer un environnement SIEM complet avec Wazuh et à évaluer sa capacité de détection face à 4 scénarios d'attaque réalistes, dans une logique Red Team vs Blue Team. L'objectif est de mesurer le temps de détection (MTTD), d'analyser les règles déclenchées, et de cartographier les alertes selon le framework MITRE ATT&CK.

## Architecture du lab
Kali : machine attaquante (Hydra, nmap)
wazuhagent : VM cible Ubuntu, agent Wazuh installé, héberge aussi DVWA (Docker) pour les tests web
wazuhserver : Wazuh Manager/Indexer/Dashboard (all-in-one), analyse et corrèle les événements

## Scénarios d'attaque testés
#	  Scénario	                               Résumé
1	Brute force SSH:	Attaque par dictionnaire (Hydra + rockyou.txt) contre le service SSH de wazuhagent
2	Exploitation web (DVWA):	Injection SQL et XSS réfléchi sur une application volontairement vulnérable
3	Malware simulé (EICAR):	Dépôt d'un fichier de test standard, détecté par FIM (syscheck) puis confirmé malveillant via  l'intégration VirusTotal
4	Mouvement latéral:	Pivot en brute force depuis la machine compromise (wazuhagent) vers le manager, avec détection taguée MITRE ATT&CK

## Temps de détection (MTTD)
Scénario	                Règle Wazuh déclenchée	          Niveau	  MTTD
Brute force SSH	5763 — sshd: brute force trying to get access	10	~5 min 45 s
Exploitation web (DVWA)	31106 — A web attack returned code 200 6	Quelques secondes
Malware EICAR (FIM)	554 — File added to the system	          5-7	Quasi instantané (mode realtime)
Malware EICAR (VirusTotal)	Alerte VirusTotal	                 12	Quelques secondes après FIM
Mouvement latéral	5760 — sshd: authentication failed (MITRE T1110.001 / T1021.004)	5	< 2 s

## Compétences démontrées
SIEM / Wazuh : installation all-in-one, gestion d'agents, règles FIM (syscheck), intégrations (VirusTotal)
MITRE ATT&CK : cartographie des alertes par tactiques (Credential Access, Lateral Movement) et techniques (T1110, T1021)
Outils offensifs : Hydra (brute force), DVWA (SQLi/XSS), nmap
Conteneurisation : Docker (déploiement DVWA)
Administration Linux : Ubuntu Server, SSH, systemd, configuration réseau
Analyse et reporting : mesure de MTTD, documentation structurée d'un exercice Red Team / Blue Team


## Contenu du dépôt
documentation/ — rapport complet avec captures d'écran numérotées pour chaque étape
screenshots/ — captures brutes organisées par scénario

Projet réalisé dans le cadre d'un exercice pratique de sécurité SOC / Blue Team.

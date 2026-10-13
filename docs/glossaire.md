# Glossaire

| Terme | Définition |
|---|---|
| **Agent** | Petit programme installé sur une machine surveillée, qui collecte ses événements et les envoie au serveur Wazuh |
| **Manager** | Composant du serveur Wazuh qui reçoit les événements des agents et les compare aux règles |
| **Indexer** | Composant du serveur Wazuh qui stocke et indexe les alertes |
| **Dashboard** | Interface web de Wazuh pour consulter et rechercher les alertes |
| **SIEM** | *Security Information and Event Management* : collecte les journaux de plusieurs machines, les analyse avec des règles et génère des alertes |
| **XDR** | *Extended Detection and Response* : prolonge le SIEM en ajoutant la réaction (bloquer une adresse, isoler une machine) |
| **Réponse active** | Action automatique exécutée par un agent lorsqu'une règle se déclenche |
| **FIM** | *File Integrity Monitoring* : surveillance de l'intégrité des fichiers |
| **Checksum (empreinte)** | Valeur calculée à partir du contenu d'un fichier ; elle change si le contenu change |
| **Règle parente** | Règle à laquelle une règle personnalisée se rattache avec `<if_sid>` |
| **`full_log`** | Texte brut d'un événement, avant son découpage en champs |
| **Threat Hunting** | Recherche active de signes d'attaque dans les alertes et les journaux ; page de Wazuh qui regroupe toutes les alertes |
| **MITRE ATT&CK** | Catalogue mondial des techniques d'attaque, utilisé dans les SOC |
| **Force brute** | Essai d'un grand nombre de mots de passe successifs sur un service (T1110) |
| **PAM** | Système qui vérifie les mots de passe sous Linux |
| **SSH** | *Secure Shell* : accès à distance sécurisé (connexion chiffrée) à la ligne de commande d'une machine |
| **`sudo` / `root`** | `root` est le compte administrateur de Linux ; `sudo` exécute une commande avec ses droits |
| **Port** | Porte numérotée d'une machine ; un programme « écoute » sur un port pour recevoir des connexions |
| **Redirection de ports** | Règle qui relie un port de l'hôte à un port d'une VM |
| **Faux positif** | Alerte déclenchée par une activité légitime ou simulée |

## Niveaux de gravité Wazuh

| Catégorie | Niveaux | Signification |
|---|---|---|
| Critical | 15 ou plus | Incident très probablement en cours, réaction immédiate |
| High | 12 à 14 | Événement très grave ou comportement clairement malveillant, à traiter en priorité |
| Medium | 7 à 11 | Événement suspect à vérifier, pas forcément malveillant |
| Low | 0 à 6 | Informatif ou courant, utile comme contexte |

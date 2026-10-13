# Rapport d'incident : tentative de force brute SSH

> Incident simulé dans un environnement de test. Les détails techniques du scénario sont dans le [lab 2](labs/lab2-force-brute-ssh.md).

## Résumé

Le 11 octobre 2026, entre 13:51 et 13:52, une série d'échecs de connexion SSH sur la machine `ubuntu-agent` a déclenché la règle de détection de force brute (5763, niveau 10).

## Éléments clés

| Rubrique | Contenu |
|---|---|
| Machine visée | `ubuntu-agent` (192.168.56.102) |
| Compte visé | `celia` |
| Source | 192.168.56.1 (PC hôte, attaquant simulé) |
| Service visé | SSH (port 22) |
| Début / détection | 13:51:06 / 13:52:08 (heure du dashboard) |
| Règles déclenchées | 5760, 5503, 5762 (niveaux 4 et 5), 2502 et 5763 (niveau 10) |
| Gravité maximale | Niveau 10 : catégorie Medium (7 à 11) |
| MITRE ATT&CK | T1110.001 Password Guessing, T1110 Brute Force |

## Chronologie

| Heure | Événement |
|---|---|
| 13:51:06 | Premiers échecs d'authentification, puis règle 2502 (niveau 10) |
| 13:51:12 à 13:51:18 | Échecs répétés (règles 5760 et 5503) |
| 13:52:08 | Règle 5763 : force brute détectée |

## Analyse

Les échecs isolés (niveau 5) sont courants. C'est leur répétition en moins d'une minute qui a conduit Wazuh à les regrouper dans une alerte de niveau 10.

## Verdict

Attaque simulée par l'analyste dans son propre lab : faux positif volontaire. Dans un cas réel, la question décisive serait de savoir si un des essais a réussi : il faudrait rechercher une connexion SSH acceptée depuis la même adresse après la série d'échecs.

## Actions recommandées

- Bloquer l'adresse source : automatisé dans le [lab 5](labs/lab5-reponse-active.md).
- Remplacer l'authentification par mot de passe par une authentification par clé SSH.
- Limiter le nombre d'essais et restreindre l'accès SSH aux adresses nécessaires.
- Vérifier l'absence de connexion réussie depuis la même source.

# Lab 2 : attaque par force brute SSH

## Objectif

Jouer l'attaquant, dans un environnement isolé et sur ses propres machines, puis analyser les alertes comme le ferait un analyste SOC. Une attaque par force brute consiste à essayer de nombreux mots de passe successifs sur un service. Définitions : [glossaire](../glossaire.md).

> Cette attaque est réalisée uniquement contre ma propre VM, dans mon propre lab.

## Prérequis

### Serveur SSH sur Ubuntu

Jusqu'ici, la connexion SSH visait la VM Wazuh. Pour attaquer Ubuntu, celle-ci doit disposer d'un serveur SSH à l'écoute. Vérification, puis démarrage (sur Ubuntu) :

```
sudo systemctl status ssh --no-pager | head -n 5
sudo systemctl enable --now ssh
```

Si le service est introuvable, l'installer d'abord :

```
sudo apt install -y openssh-server
```

Dans ce lab, le service était installé mais inactif (`inactive (dead)`). `enable --now` l'active au démarrage et le lance immédiatement.

### Adresse de la cible

```
ip -4 a
```

Ubuntu possède deux adresses, une par carte réseau. L'attaque provient du PC hôte, présent sur le réseau privé hôte (192.168.56.1) : la cible est donc l'adresse de cette carte, ici `192.168.56.102`. L'adresse de la carte NAT (`10.0.3.x`) ne sert qu'à sortir vers internet et n'est pas joignable depuis le PC.

## Scénario

Depuis l'invite de commande du PC hôte :

```
ssh celia@192.168.X.X
```

1. Une première connexion avec le bon mot de passe réussit. Elle laisse elle aussi une trace dans les journaux (connexion acceptée). Déconnexion avec `exit`.
2. La commande est relancée en saisissant de mauvais mots de passe : 3 essais par connexion, jusqu'à `Permission denied`, répétés plusieurs fois.

Un échec isolé est banal. Ce sont les échecs répétés sur un court laps de temps qui ressemblent à une attaque.

## Détection

Les mauvais mots de passe sont écrits par Ubuntu dans son journal de sécurité (`/var/log/auth.log`). L'agent le lit et l'envoie au serveur, qui compare chaque ligne à ses règles.

Dans le dashboard : **Threat Hunting**, agent `ubuntu-agent`, onglet **Events**, recherche `sshd` (nom du programme du serveur SSH). Threat Hunting regroupe toutes les alertes, contrairement aux pages spécialisées comme File Integrity Monitoring.

### Règles déclenchées

| Règle | Description | Niveau | Signification |
|---|---|---|---|
| 5760 | sshd: authentication failed | 5 | Un échec de connexion SSH, peu inquiétant seul |
| 5503 | PAM: User login failed | 5 | Le même échec, vu par PAM (le système qui vérifie les mots de passe sous Linux) |
| 5762 | sshd: connection reset | 4 | Connexion coupée en cours de route, après 3 essais ratés |
| 2502 | syslog: User missed the password more than one time | 10 | Plusieurs erreurs de mot de passe |
| 5763 | sshd: brute force trying to get access to the system. Authentication failed. | 10 | Échecs SSH répétés en peu de temps : tentative de force brute |

### Chronologie (11 octobre 2026, heure du dashboard)

| Heure | Événement |
|---|---|
| 13:51:06 | Premiers échecs, suivis de la règle 2502 (niveau 10) |
| 13:51:12 à 13:51:18 | Échecs répétés |
| 13:52:08 | Règle 5763, force brute détectée |

Des signaux faibles et isolés (niveau 5) deviennent, une fois cumulés, une alerte plus grave (niveau 10, catégorie Medium) : c'est une **escalade**.

## Classification MITRE ATT&CK

MITRE ATT&CK est un catalogue des techniques d'attaque. **Brute Force** (T1110) est la technique générale, et elle contient des sous-techniques :

| Identifiant | Nom | Principe |
|---|---|---|
| T1110.001 | Password Guessing | Essayer des mots de passe à la suite sur un compte (scénario de ce lab) |
| T1110.003 | Password Spraying | Essayer un même mot de passe sur de nombreux comptes |
| T1110.004 | Credential Stuffing | Réutiliser des identifiants issus de fuites |

Dans le dashboard, les échecs isolés sont d'abord rattachés à la sous-technique **Password Guessing**. Quand la répétition déclenche la règle 5763, l'alerte est rattachée à la technique **Brute Force**.

## Analyse

La question clé d'un analyste est de savoir si l'un des essais a fini par réussir. Il faut donc rechercher une connexion SSH acceptée, depuis la même adresse, après la série d'échecs.

Ici, l'attaque est simulée : c'est un faux positif volontaire. La synthèse formalisée est dans le [rapport d'incident](../rapport-incident-force-brute.md).

## Ce que j'ai retenu

- Wazuh signale la force brute mais, par défaut, ne bloque rien : le blocage fait l'objet du [lab 5](lab5-reponse-active.md).
- Plusieurs règles se déclenchent pour un même événement, à des niveaux différents.
- Une alerte se lit avec son contexte : chronologie, répétition, technique MITRE.
- L'adresse à utiliser pour joindre une machine dépend de la carte réseau par laquelle on arrive.

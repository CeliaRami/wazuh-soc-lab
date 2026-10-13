# Lab 3 : usage de sudo (comptes privilégiés)

## Objectif

Sous Linux, `root` est le compte tout-puissant. La commande `sudo` permet à un utilisateur ordinaire d'exécuter une commande avec les droits de root. Un attaquant qui entre avec un compte ordinaire cherche presque toujours à obtenir ces droits (élévation de privilèges) : un SOC surveille donc les usages et les échecs de `sudo`. Définitions : [glossaire](../glossaire.md).

## Scénario

Sur la VM Ubuntu.

### 1. Un sudo réussi

```
sudo whoami
```

`whoami` affiche le compte qui exécute la commande. Résultat : `root` (sans `sudo`, ce serait l'utilisateur ordinaire).

### 2. Un sudo raté, volontairement

Après un `sudo` réussi, Ubuntu mémorise le mot de passe environ 15 minutes. `sudo -k` efface cette mémoire :

```
sudo -k
sudo whoami
```

Trois mots de passe incorrects sont saisis, jusqu'au message d'échec de `sudo`.

## Vérification de la source

Avant de chercher l'alerte, on vérifie ce qu'Ubuntu a réellement enregistré :

```
sudo journalctl _COMM=sudo -n 8 --no-pager
```

- `journalctl` lit le journal du système.
- `_COMM=sudo` ne garde que les lignes écrites par `sudo`.
- `-n 8` affiche les 8 plus récentes.

Le journal doit contenir la commande exécutée (`COMMAND=/usr/bin/whoami`) pour la réussite et des lignes d'échec d'authentification pour les mauvais mots de passe.

## Détection

Dans le dashboard : **Threat Hunting**, agent `ubuntu-agent`, onglet **Events**, recherche `sudo`, période **Last 24 hours**. Une alerte de réussite et des alertes d'échec ont été retrouvées.

### Champs analysés

La réussite a été relevée sur le serveur Wazuh (agent `wazuh-server`, redémarrage d'un service avec `sudo`) : c'est la même règle 5402 qui se déclenche pour tout `sudo` réussi vers `root`. L'échec a été relevé sur `ubuntu-agent`, lors du test avec trois mauvais mots de passe.

| Champ | Signification | Réussite | Échec |
|---|---|---|---|
| `rule.id` | Règle déclenchée | 5402 | 5404 |
| `rule.description` | Ce que Wazuh a compris | Successful sudo to ROOT executed. | Three failed attempts to run sudo |
| `rule.level` | Gravité de 0 à 15 | 3 (Low) | 10 (Medium) |
| `data.srcuser` | Compte qui a lancé `sudo` | `root` | `celia` |
| `data.dstuser` | Droits demandés | `root` | `root` |
| `data.command` | Commande lancée avec ces droits | `/usr/bin/systemctl restart wazuh-dashboard` | `/usr/bin/whoami` |
| `rule.mitre.technique` | Technique ATT&CK associée | Sudo and Sudo Caching (T1548.003) | Sudo and Sudo Caching (T1548.003) |

Le champ `full_log` de l'alerte d'échec contient le texte brut de l'événement : `celia : 3 incorrect password attempts ; TTY=pts/0 ; PWD=/home/celia ; USER=root ; COMMAND=/usr/bin/whoami`.

### Lecture

- **Escalade de gravité** : un `sudo` réussi est classé niveau 3 (Low). Trois échecs consécutifs déclenchent une règle distincte au niveau 10 (Medium). Pour comparaison, la documentation officielle de Wazuh décrit la règle 5401, « Failed attempt to run sudo », au niveau 5 pour un échec isolé (versions 3.x à 4.2, non vérifiée sur la 4.14.7) : la répétition fait donc monter la gravité.
- **Classification MITRE** : les deux alertes sont rattachées à la technique T1548.003 (Sudo and Sudo Caching), des tactiques Privilege Escalation et Defense Evasion.
- **Contexte** : `data.srcuser`, `data.command` et `data.pwd` répondent à « qui, quoi, depuis où » lors d'une enquête.

## Ce que j'ai retenu

- Un `sudo` réussi est un événement courant. Des échecs répétés d'une personne qui cherche à obtenir les droits root sont suspects.
- Contrôler le journal du système avant de chercher une alerte permet de distinguer « l'événement n'a pas eu lieu » de « Wazuh ne l'a pas détecté ».
- Comparer les alertes de réussite et d'échec montre comment Wazuh gradue la gravité selon le contexte.

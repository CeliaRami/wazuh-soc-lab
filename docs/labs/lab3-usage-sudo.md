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

| Champ | Signification | Réussite | Échec |
|---|---|---|---|
| `rule.id` | Règle déclenchée | **À COMPLÉTER** | **À COMPLÉTER** |
| `rule.description` | Ce que Wazuh a compris | **À COMPLÉTER** | **À COMPLÉTER** |
| `rule.level` | Gravité de 0 à 15 | **À COMPLÉTER** | **À COMPLÉTER** |
| `data.srcuser` | Compte qui a lancé `sudo` | **À COMPLÉTER** | **À COMPLÉTER** |
| `data.dstuser` | Droits demandés | **À COMPLÉTER** | **À COMPLÉTER** |
| `data.command` | Commande lancée avec ces droits | **À COMPLÉTER** | **À COMPLÉTER** |
| `rule.mitre.technique` | Technique ATT&CK associée | **À COMPLÉTER** | **À COMPLÉTER** |

## Ce que j'ai retenu

- Un `sudo` réussi est un événement courant. Des échecs répétés d'une personne qui cherche à obtenir les droits root sont suspects.
- Contrôler le journal du système avant de chercher une alerte permet de distinguer « l'événement n'a pas eu lieu » de « Wazuh ne l'a pas détecté ».
- Comparer les alertes de réussite et d'échec montre comment Wazuh gradue la gravité selon le contexte.

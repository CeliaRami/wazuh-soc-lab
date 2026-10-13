# Lab 1 : surveillance d'intégrité des fichiers (FIM)

## Objectif

Détecter qu'un fichier sensible a été modifié, ajouté ou supprimé. Un attaquant qui s'installe sur une machine modifie souvent des fichiers (configuration, scripts, comptes). L'agent calcule une empreinte (checksum) de chaque fichier surveillé et prévient le serveur Wazuh lorsqu'elle change. Définitions : [glossaire](../glossaire.md).

## Mise en place

Toutes les commandes de cette section s'exécutent sur la VM Ubuntu (celle de l'agent).

### 1. Créer un dossier de test

```
mkdir ~/fim-test
echo "contenu initial" > ~/fim-test/secret.txt
ls ~/fim-test
```

### 2. Demander à l'agent de surveiller ce dossier

La configuration de l'agent se trouve dans `/var/ossec/etc/ossec.conf`. Elle contient une section `<syscheck>`, le module de surveillance des fichiers. La commande suivante y ajoute le dossier à surveiller (remplacer `celia` par le nom d'utilisateur de la VM) :

```
sudo sed -i '0,/<syscheck>/{s|<syscheck>|<syscheck>\n    <directories realtime="yes" check_all="yes" report_changes="yes">/home/celia/fim-test</directories>|}' /var/ossec/etc/ossec.conf
```

| Option | Effet |
|---|---|
| `realtime="yes"` | Détection immédiate, sans attendre un scan périodique |
| `check_all="yes"` | Vérifie l'empreinte, la taille, les droits et le propriétaire |
| `report_changes="yes"` | L'alerte indique ce qui a changé dans le contenu |

Vérification (`sudo` est nécessaire : le fichier est protégé) :

```
sudo grep -n "fim-test" /var/ossec/etc/ossec.conf
```

### 3. Redémarrer l'agent

L'agent ne lit sa configuration qu'au démarrage. Le redémarrage n'est nécessaire qu'après une modification de `ossec.conf`, pas à chaque événement.

```
sudo systemctl restart wazuh-agent
sudo systemctl status wazuh-agent --no-pager | head -n 5
```

## Scénario

Trois événements provoqués sur Ubuntu :

```
echo "ligne ajoutee par un attaquant" >> ~/fim-test/secret.txt
echo "nouveau fichier" > ~/fim-test/nouveau.txt
rm ~/fim-test/nouveau.txt
```

`>>` ajoute du texte à la fin d'un fichier, `>` crée ou écrase un fichier, `rm` supprime.

## Résultats

Dans le dashboard : carte **File Integrity Monitoring**, agent `ubuntu-agent`, onglet **Events**. L'alerte peut mettre une à deux minutes à apparaître. La page **Discover** liste aussi toutes les alertes brutes.

| Action sur Ubuntu | `syscheck.event` | Interprétation |
|---|---|---|
| Ajout d'une ligne dans `secret.txt` | `modified` | Le contenu d'un fichier existant a changé |
| Création de `nouveau.txt` | `added` | Un fichier est apparu (outil ou script déposé ?) |
| Suppression de `nouveau.txt` | `deleted` | Un fichier a disparu (trace effacée ?) |

### Lecture de l'alerte « modified »

| Champ | Signification | Valeur observée |
|---|---|---|
| `rule.id` | Règle déclenchée | 550 |
| `rule.description` | Description de la règle | Integrity checksum changed |
| `rule.level` | Gravité de 0 à 15 | 7 (catégorie Medium) |
| `syscheck.path` | Fichier concerné | `/home/celia/fim-test/secret.txt` |
| `syscheck.mode` | Mode de détection | `realtime` |
| `syscheck.diff` | Changement de contenu (`>` : ligne ajoutée, `<` : ligne supprimée) | `1a2 > Ligne ajoutee par attaquant` |
| `syscheck.size_before` / `size_after` | Taille avant et après | 16 puis 44 octets |
| `syscheck.changed_attributes` | Attributs modifiés | `size, mtime, md5, sha1, sha256` |
| `syscheck.uname_after` | Compte à l'origine de la modification | `celia` |
| `full_log` | Texte brut de l'événement | `File '/home/celia/fim-test/secret.txt' modified` |

Les empreintes avant et après sont différentes : c'est la preuve que le contenu a changé. Le champ `full_log` sera réutilisé au [lab 4](lab4-regle-personnalisee.md) pour écrire une règle personnalisée.

## Observation complémentaire

Dans **Discover**, des alertes `netstat listening ports` apparaissent aussi pour `wazuh-server`. Wazuh surveille la liste des ports ouverts d'une machine et alerte quand elle change, par exemple après un changement de réseau ou un redémarrage. Tout ce qui change sur une machine génère donc une alerte, y compris ce que l'on provoque soi-même : le rôle de l'analyste est de distinguer le normal du suspect.

## Ce que j'ai retenu

- L'agent détecte les modifications, ajouts et suppressions de fichiers en temps réel.
- `report_changes` permet de voir exactement quelle ligne a été ajoutée.
- Le champ `syscheck.uname_after` répond à la question « qui ? » lors d'une enquête.
- Une alerte de niveau 7 (Medium) demande une vérification : une modification peut être légitime.

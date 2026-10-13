# Lab 4 : règle de détection personnalisée

## Objectif

Une règle est une consigne donnée à Wazuh : « si tu observes telle situation, déclenche une alerte de tel niveau avec tel message ». Ce lab consiste à en écrire une pour traiter plus sévèrement un fichier critique : `secret.txt` (voir le [lab 1](lab1-fim.md)) passe du niveau 7 de la règle générale 550 au niveau 12 (catégorie High).

Les règles s'exécutent sur le **serveur** Wazuh, dans `/var/ossec/etc/rules/local_rules.xml`, fichier prévu pour les règles personnalisées.

## Principe : deux filtres successifs

1. **Règle parente (550)** : « un fichier surveillé a changé » (niveau 7).
2. **Règle personnalisée (100010)** : « et le fichier concerné est `secret.txt` » (niveau 12).

La règle parente se lit dans le dashboard (champ `rule.id` de l'alerte « Integrity checksum changed »). Le fichier de règles contient par défaut une règle d'exemple (100001) servant de modèle.

## Mise en place (sur le serveur Wazuh)

### 1. Lire le fichier et sauvegarder

```
sudo cat /var/ossec/etc/rules/local_rules.xml
sudo cp /var/ossec/etc/rules/local_rules.xml /var/ossec/etc/rules/local_rules.xml.bak
```

### 2. Ajouter la règle

Le bloc complet est dans [configs/local_rules.xml](../../configs/local_rules.xml). Commande d'ajout à la fin du fichier, sur une seule ligne :

```
sudo bash -c 'printf "\n<group name=\"local,fim,\">\n  <rule id=\"100010\" level=\"12\">\n    <if_sid>550</if_sid>\n    <match>fim-test/secret.txt</match>\n    <description>Fichier critique secret.txt modifie</description>\n  </rule>\n</group>\n" >> /var/ossec/etc/rules/local_rules.xml'
```

| Élément | Rôle |
|---|---|
| `id="100010"` | Les numéros à partir de 100000 sont réservés aux règles personnalisées |
| `level="12"` | Gravité de 0 à 15 (12 : catégorie High) |
| `<if_sid>550</if_sid>` | La règle ne s'applique qu'après la règle parente 550 |
| `<match>` | Cherche ce texte dans le texte brut de l'événement (`full_log`) |
| `<description>` | Message affiché dans l'alerte |

`full_log` contient `File '/home/celia/fim-test/secret.txt' modified` : le texte recherché y figure.

### 3. Vérifier, tester la syntaxe, redémarrer

```
sudo grep -n "100010" /var/ossec/etc/rules/local_rules.xml
sudo /var/ossec/bin/wazuh-analysisd -t; echo "code: $?"
sudo systemctl restart wazuh-manager
```

- `grep` confirme que la règle est bien dans le fichier.
- `wazuh-analysisd -t` charge les règles et vérifie leur validité sans rien démarrer : `code: 0` signifie « aucune erreur ».
- Le manager lit ses règles au démarrage : le redémarrage est nécessaire.

## Test

Sur Ubuntu, après la reconnexion de l'agent (une à deux minutes) :

```
echo "test regle 2" >> ~/fim-test/secret.txt
```

Dans le dashboard : **Threat Hunting**, onglet **Events**, recherche `rule.id:100010`. La page File Integrity Monitoring ne montre que les alertes de la famille « fichiers » et peut ne pas afficher une règle personnalisée rangée dans un autre groupe.

## Résultat

L'alerte est apparue au **niveau 12** (High), au lieu du niveau 7 de la règle générale 550.

## Leçons

- Ma première tentative d'ajout, avec une commande sur plusieurs lignes, ne s'était pas écrite dans le fichier, **sans aucun message d'erreur**. Le test de syntaxe renvoyait `code: 0` car il validait un fichier qui ne contenait que l'exemple. Toujours vérifier après modification avec `grep`.
- Une première version de la règle utilisait `<field name="file">` pour la condition. Le nom de champ n'avait pas été vérifié. `<match>` sur `full_log` s'appuie sur un texte observé dans l'alerte.
- Après toute modification des règles : tester la syntaxe, puis redémarrer le manager.

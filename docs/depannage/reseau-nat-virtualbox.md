# Dépannage : le Réseau NAT de VirtualBox n'ouvrait pas les ports

## Symptôme

La connexion SSH vers le serveur Wazuh échouait :

```
ssh: connect to host 127.0.0.1 port 2229: Connection refused
```

Le port 2229 n'apparaissait jamais côté PC :

```
netstat -an | findstr 2229
```

## Diagnostic

| Vérification | Constat |
|---|---|
| Règles de redirection (2229 vers 22 et 443 vers 443) | Présentes et correctes |
| Mode réseau de la VM | Bien branchée sur le Réseau NAT `NatNetwork` |
| Deux réseaux NAT | `NatNetwork` et `NatNetwork1`, même plage 10.0.2.0/24 : doublon supprimé |
| `VBoxManage list natnets` | Réseau correct (DHCP activé) |
| `VBoxManage natnetwork start --netname NatNetwork` | Erreur : « Cannot start DHCP server because it is already running » |
| `tasklist \| findstr VBox` | Plusieurs `VBoxNetDHCP.exe` bloqués, aucun `VBoxNetNAT.exe` |

Le processus qui ouvre les ports du Réseau NAT (`VBoxNetNAT.exe`) ne tournait pas.

## Actions

1. Arrêt des processus DHCP bloqués : `taskkill /F /IM VBoxNetDHCP.exe`.
2. Redémarrage du réseau avec `VBoxManage natnetwork start`.
3. Résultat : `VBoxNetNAT.exe` démarrait, mais `netstat -ano` ne montrait aucun port en écoute pour son identifiant de processus : les règles de redirection n'étaient pas appliquées.
4. Suppression puis recréation de la règle SSH avec `VBoxManage natnetwork modify ... --port-forward-4`, puis redémarrage de la VM : le port restait fermé.

## Résolution

Passage au **NAT simple** avec redirections de ports configurées directement sur la VM (Configuration, Réseau, Carte 1, Avancé, Redirection de ports). Le port s'est ouvert dès le démarrage de la VM. La cause exacte du dysfonctionnement du Réseau NAT n'a pas été identifiée.

## Remarque

Un triangle jaune « Paramètre invalide détecté » dans la configuration de la VM ne concernait pas le réseau : il signalait la page Affichage (contrôleur graphique conseillé : VMSVGA), sans lien avec le problème.

## Ce que j'ai retenu

- Isoler le problème couche par couche : règle présente, port ouvert côté hôte, processus actif.
- Utiliser les outils en ligne de commande (`VBoxManage`, `tasklist`, `netstat`) pour obtenir le message d'erreur réel.
- Savoir abandonner une approche après vérification et choisir une alternative fiable.

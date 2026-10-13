# Installation

Ce document décrit le montage du lab, de l'import du serveur Wazuh à l'enregistrement de l'agent. L'architecture et le plan d'adressage sont décrits dans [01-architecture.md](01-architecture.md).

## 1. Serveur Wazuh

Import de l'OVA Wazuh 4.14.7 dans VirtualBox. Les identifiants par défaut de l'OVA sont indiqués dans la documentation officielle de Wazuh et doivent être changés pour tout usage réel.

## 2. Réseau de la VM Wazuh

- **Carte 1** : mode NAT, avec les deux redirections de ports (SSH et dashboard) décrites dans [01-architecture.md](01-architecture.md#redirections-de-ports-carte-nat-de-la-vm-wazuh).
- **Carte 2** : réseau privé hôte (`VirtualBox Host-Only Ethernet Adapter`).

Une redirection de ports ouvre un port sur le PC hôte et le relie à un port de la VM. Le port est actif uniquement lorsque la VM est démarrée.

Vérification du port SSH depuis le PC (Windows) :

```
netstat -an | findstr 2229
```

La ligne `127.0.0.1:2229 ... LISTENING` confirme l'ouverture du port.

## 3. Connexion SSH au serveur

```
ssh wazuh-user@127.0.0.1 -p 2229
```

SSH (Secure Shell) donne accès à la ligne de commande de la VM par une connexion chiffrée, avec copier-coller, ce que la console VirtualBox ne permet pas confortablement.

## 4. Vérification des services

Wazuh repose sur trois services qui doivent tous être à l'état `active (running)` :

```
sudo systemctl status wazuh-manager wazuh-indexer wazuh-dashboard --no-pager | grep -E "wazuh-|Active"
```

Lors du premier démarrage, le manager était en échec (`Result: timeout`) : le démarrage de la VM est lent. Relance des services dans l'ordre de dépendance :

```
sudo systemctl restart wazuh-indexer
sudo systemctl restart wazuh-manager
sudo systemctl restart wazuh-dashboard
```

L'indexer et le manager peuvent mettre une à deux minutes à passer à `active (running)`.

## 5. Accès au dashboard

Depuis le navigateur du PC : `https://127.0.0.1:8443`. Le certificat est auto-signé, le navigateur affiche donc un avertissement qu'il faut accepter dans ce contexte de lab.

Si le navigateur redirige vers `wazuh-server:8443`, la page échoue (`DNS_PROBE_FINISHED_NXDOMAIN`) : ce nom n'est connu que de la VM. Il faut saisir l'adresse en `127.0.0.1`.

## 6. Machine surveillée (Ubuntu)

Configuration réseau de la VM Ubuntu : voir [01-architecture.md](01-architecture.md#réseau).

Test de communication avec le serveur :

```
ping -c 4 192.168.56.101
```

Test de l'accès internet, nécessaire pour télécharger l'agent :

```
ping -c 2 8.8.8.8
ping -c 2 google.com
```

Dans ce lab, le ping vers `8.8.8.8` réussissait mais pas celui vers `google.com` : le serveur DNS attribué à la carte NAT ne répondait pas. Correction temporaire (perdue au redémarrage) :

```
sudo resolvectl dns enp0s8 8.8.8.8
```

`enp0s8` est le nom de l'interface NAT dans cette VM. Il peut différer et se lit avec `resolvectl status`.

## 7. Installation de l'agent

La commande est générée par le dashboard (Deploy new agent, Linux, DEB amd64, adresse du serveur `192.168.56.101`, nom `ubuntu-agent`) :

```
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.7-1_amd64.deb && sudo WAZUH_MANAGER='192.168.56.101' WAZUH_AGENT_NAME='ubuntu-agent' dpkg -i ./wazuh-agent_4.14.7-1_amd64.deb
```

L'installation ne démarre pas l'agent. Démarrage et activation au boot :

```
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

## 8. Vérification

- Sur Ubuntu : `sudo systemctl status wazuh-agent` doit afficher `active (running)`.
- Dans le dashboard : `ubuntu-agent` apparaît dans la liste des agents avec le statut `Active`.

## Problèmes rencontrés

- Le Réseau NAT de VirtualBox n'ouvrait pas les ports : [reseau-nat-virtualbox.md](depannage/reseau-nat-virtualbox.md)
- Le serveur Wazuh est tombé après le remplissage de son disque : [disque-plein.md](depannage/disque-plein.md)

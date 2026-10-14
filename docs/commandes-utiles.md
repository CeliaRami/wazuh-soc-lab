# Commandes utiles

## Sur le PC hôte (Windows)

- `netstat -an | findstr 2229` : vérifier que la redirection de port SSH est active (`LISTENING`)
- `ssh wazuh-user@127.0.0.1 -p 2229` : se connecter à la VM Wazuh
- `ssh celia@192.168.X.X` : se connecter à Ubuntu (ou provoquer des échecs de connexion)

## Services et état du système

- `systemctl status <service>` : voir l'état d'un service
- `systemctl restart <service>` : redémarrer un service
- `sudo systemctl enable --now ssh` : démarrer le serveur SSH et l'activer au démarrage
- `ip -4 a` : afficher les adresses IP de la machine
- `ping -c 2 <adresse>` : tester la connexion (2 paquets)
- `df -h` : voir l'espace libre des disques
- `sudo du -xh / --max-depth=2 | sort -rh | head` : trouver les dossiers qui occupent le plus de place

## Wazuh

- `sudo /var/ossec/bin/wazuh-analysisd -t` : tester la validité des règles et de la configuration sans rien démarrer (`code: 0` = OK)
- `sudo grep -E "ERROR|CRITICAL" /var/ossec/logs/ossec.log | tail -n 10` : lire les dernières erreurs du journal de Wazuh
- `sudo systemctl restart wazuh-manager` : recharger la configuration et les règles du serveur
- `sudo grep -n -A8 "active-response" /var/ossec/etc/ossec.conf` : chercher la configuration de réponse active

## Fichiers et recherche

- `sudo nano /var/ossec/etc/ossec.conf` : ouvrir un fichier de configuration (Ctrl+W pour chercher, Ctrl+X pour sortir)
- `sudo grep -n "mot" fichier` : chercher un mot dans un fichier protégé
- `sudo cp fichier fichier.bak` : sauvegarder un fichier avant de le modifier

## Journaux et sécurité

- `sudo journalctl _COMM=sudo -n 8 --no-pager` : lire les dernières lignes écrites par `sudo` dans le journal
- `sudo -k` : effacer la mémoire du mot de passe `sudo`
- `sudo iptables -L -n` : lister les règles du pare-feu (voir les adresses bloquées)

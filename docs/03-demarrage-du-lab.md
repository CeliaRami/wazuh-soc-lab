# Démarrage et arrêt du lab

Procédure pour relancer le lab après un arrêt du PC. Les commandes de vérification sont détaillées dans [02-installation.md](02-installation.md).

## Démarrage

1. **Démarrer la VM Wazuh** et attendre environ 3 minutes : les services mettent du temps à démarrer.
2. **Se connecter en SSH** depuis le PC : `ssh wazuh-user@127.0.0.1 -p 2229`.
3. **Vérifier les services** (manager, indexer, dashboard : tous `active`). S'il y en a un en échec, le relancer dans l'ordre : indexer, puis manager, puis dashboard. Voir [la vérification des services](02-installation.md#4-vérification-des-services).
4. **Vérifier l'espace disque** du serveur :

```
   df -h /
```

   Un disque presque plein empêche le manager de démarrer. Voir [disque-plein.md](depannage/disque-plein.md).
5. **Vérifier l'adresse IP** du serveur avec `ip -4 a`. Elle doit rester `192.168.56.101`. Si elle a changé, il faut la mettre à jour dans la configuration de l'agent (balise `<address>` du fichier `/var/ossec/etc/ossec.conf` sur Ubuntu), puis redémarrer l'agent.
6. **Ouvrir le dashboard** : `https://127.0.0.1:8443`.
7. **Démarrer la VM Ubuntu**. L'agent est activé au démarrage et doit se lancer seul. Vérification :

```
   sudo systemctl status wazuh-agent
```

   S'il n'est pas actif : `sudo systemctl start wazuh-agent`.
8. **Si internet ne fonctionne plus sur Ubuntu** (`ping google.com` échoue) : refaire la correction DNS décrite dans [02-installation.md](02-installation.md#6-machine-surveillée-ubuntu). Une solution permanente n'a pas été testée.
9. **Contrôler dans le dashboard** que `ubuntu-agent` est au statut `Active`.

## Arrêt

Éteindre proprement chaque VM avant d'éteindre le PC :

```
sudo poweroff
```

Un disque virtuel est un fichier sur le PC hôte. Une coupure brutale, alors que la VM écrit, risque de l'endommager. Le risque concerne surtout la VM Wazuh, qui écrit en continu (alertes, bases internes).

L'option « Sauvegarder l'état de la machine » de VirtualBox est plus rapide, mais elle fige aussi les services de Wazuh : un arrêt complet reste plus propre.

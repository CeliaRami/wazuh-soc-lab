# Dépannage : disque plein sur le serveur Wazuh

## Symptôme

Au redémarrage du manager (pour charger une règle) :

```
Job for wazuh-manager.service failed because the control process exited with error code.
```

## Diagnostic

Démarche : de l'erreur vers la cause, en mesurant avant de supprimer.

1. **Lire la vraie erreur** dans le journal de Wazuh :

```
   sudo grep -E "ERROR|CRITICAL" /var/ossec/logs/ossec.log | tail -n 10
```

   Résultat : `No space left on device`.

2. **Mesurer le disque** :

```
   df -h
```

   Résultat : `/dev/sda1`, 25 Go, 100 % utilisé, 20 Ko libres.

3. **Trouver le dossier le plus gros** :

```
   sudo du -xh / --max-depth=2 2>/dev/null | sort -rh | head -n 15
```

   Résultat : `/var/ossec` occupe 20 Go.

4. **Descendre dans le dossier** :

```
   sudo du -h /var/ossec --max-depth=2 2>/dev/null | sort -rh | head -n 12
```

   Résultat : `queue/vd_updater` (9,9 Go) et `queue/vd` (8,1 Go). Ces dossiers appartiennent au module de détection de vulnérabilités.

5. **Trouver la cause** :

```
   sudo du -h /var/ossec/queue/vd_updater --max-depth=3 2>/dev/null | sort -rh | head -n 10
```

   Résultat : `tmp/contents` occupe 9,6 Go de fichiers temporaires `*-api_file.json`, datés de quelques minutes avant les premières erreurs.

## Cause

Le module de détection de vulnérabilités télécharge régulièrement ses bases de CVE (failles connues). Ses fichiers temporaires auraient dû être supprimés après traitement et se sont accumulés jusqu'à remplir le disque. Le réglage était une mise à jour toutes les 60 minutes (`feed-update-interval`) ; je n'ai pas confirmé que c'est la cause exacte. Un disque de 25 Go est par ailleurs petit pour cette fonction.

## Résolution

1. Arrêter le manager :

```
   sudo systemctl stop wazuh-manager
```

2. Supprimer uniquement ces fichiers temporaires (suppression définitive) :

```
   sudo find /var/ossec/queue/vd_updater/tmp/contents -name '*-api_file.json' -delete
```

   Résultat : disque à 65 %, 8,8 Go libres. Les données utiles du module (`rocksdb`, `queue/vd`) n'ont pas été touchées.

3. Sauvegarder la configuration, puis désactiver temporairement la détection de vulnérabilités :

```
   sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.bak
   sudo grep -n -A8 "<vulnerability-detection>" /var/ossec/etc/ossec.conf
   sudo sed -i '113s/yes/no/' /var/ossec/etc/ossec.conf
```

   La ligne 113 est ciblée explicitement : la balise `<enabled>yes</enabled>` apparaît aussi pour l'indexer, qu'il ne fallait pas désactiver.

4. Redémarrer et contrôler :

```
   sudo systemctl start wazuh-manager
   df -h /
```

## Ce que j'ai retenu

- Lire le journal avant d'agir : le message `failed` est générique, la cause est dans `ossec.log`.
- Mesurer avant de supprimer, et ne supprimer que des fichiers temporaires identifiés.
- Sauvegarder un fichier de configuration avant de le modifier.
- Surveiller l'espace disque (`df -h /`) d'un serveur de journalisation, dimensionné ici trop juste.

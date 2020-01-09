# Logstash Grok patterns pour Sun Oracle Messaging Server

Sun oracle Messaging Server est un serveur de messagerie.

## Description
Dans ce dépôt, nous proposons des patterns Grok pour parser (structurer) ses logs via Logstash:
- `sun-oracle.grok` : contient les patterns Grok
- `pipeline.exemple` : est un fichier d'exemple de pipeline logstash montrant une méthode d'utilisation de ces patterns via Logtash.
- `log-oracle.log` : contient des exemples de log de Sun oracle Messaging Server qui ont été testés.
- `rsyslog.conf.example` : Contient un exemple de configuration de Rsyslog pour envoyer les logs de Sun Oracle Messaging vers Logtash en précisant les chemins vers les fichiers de logs (grâce au module `imfile`).

## Remarque
- L'utilisation de ce dépôt nécessite la compréhension du fonctionnement de Logstash.
- Dans `pipeline.exemple`, le répertoire du pattern doit être adapté.
- En production, en général, les logs de messagerie sun oracle ne sont pas stockés dans `/var/log/maillog`. Cependant grâce à des outils comme Rsyslog, on peut spécifier ses fichiers de log (grâce au module `imfile`) avant leur envoi vers Logstash. Dans ce cas, il faut décommenter plutôt le deuxième pattern `SUN` dans `sun-oracle.grok` pour éviter les erreurs de parsing car Rsyslog va ajouter des tags sur les lignes de log.

## Help
- Construction des patterns à l'aide du site [Grok Constructor](https://grokconstructor.appspot.com/do/match)
- Doc Sun Oracle Messaging accessible [ici](https://docs.oracle.com/cd/E63708_01/doc.801/e63711/toc.htm)

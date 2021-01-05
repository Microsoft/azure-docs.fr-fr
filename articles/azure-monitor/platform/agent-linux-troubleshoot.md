---
title: Résoudre les problèmes liés à l’agent Linux pour Azure Log Analytics | Microsoft Docs
description: Décrit les symptômes, les causes et les solutions aux problèmes courants liés à l’agent Log Analytics pour Linux dans Azure Monitor.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 11/21/2019
ms.openlocfilehash: 13959c4a3c798656efdc72b5c8e5f96e4fb2392a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011895"
---
# <a name="how-to-troubleshoot-issues-with-the-log-analytics-agent-for-linux"></a>Guide pratique pour résoudre les problèmes liés à l’agent Log Analytics pour Linux 

Cet article fournit une assistance sur les erreurs que vous pourriez rencontrer avec l’agent Log Analytics pour Linux dans Azure Monitor et suggère des solutions possibles pour les résoudre.

Si aucune de ces étapes ne fonctionne, les canaux de support suivants sont également disponibles :

* Les clients bénéficiant d’un support Premier peuvent ouvrir une demande de support auprès du [Support Premier](https://premier.microsoft.com/).
* Les clients titulaires d’un contrat de support Azure peuvent ouvrir une demande de support [sur le portail Azure](https://manage.windowsazure.com/?getsupport=true).
* Diagnostiquez les problèmes liés à OMI avec le [Guide de dépannage pour OMI](https://github.com/Microsoft/omi/blob/master/Unix/doc/diagnose-omi-problems.md).
* Déposez un [problème GitHub](https://github.com/Microsoft/OMS-Agent-for-Linux/issues).
* Visitez la page de commentaires de Log Analytics pour consulter les idées et bogues soumis sur [https://aka.ms/opinsightsfeedback](https://aka.ms/opinsightsfeedback) ou en partager de nouveaux. 

## <a name="log-analytics-troubleshooting-tool"></a>Outil de résolution des problèmes Log Analytics

L’outil de résolution des problèmes de l’agent Log Analytics Linux est un script conçu pour faciliter la recherche et le diagnostic des problèmes liés à l’agent Log Analytics. Il est automatiquement inclus avec l’agent lors de l’installation. L’exécution de l’outil doit être la première étape du diagnostic d’un problème.

### <a name="how-to-use"></a>Utilisation
L’outil de dépannage peut être exécuté en collant la commande suivante dans une fenêtre de terminal sur un ordinateur avec l’agent Log Analytics : `sudo /opt/microsoft/omsagent/bin/troubleshooter`

### <a name="manual-installation"></a>Installation manuelle
L’outil de résolution des problèmes est inclus automatiquement lors de l’installation de l’agent Log Analytics. Toutefois, si l’installation échoue de quelque façon que ce soit, vous pouvez également l’installer manuellement en suivant les étapes ci-dessous.

1. Copiez le pack de l’utilitaire de résolution des problèmes sur votre ordinateur : `wget https://raw.github.com/microsoft/OMS-Agent-for-Linux/master/source/code/troubleshooter/omsagent_tst.tar.gz`
2. Décompressez le pack : `tar -xzvf omsagent_tst.tar.gz`
3. Exécutez l’installation manuelle : `sudo ./install_tst`

### <a name="scenarios-covered"></a>Scénarios couverts
Voici une liste de scénarios vérifiés par l’outil de résolution des problèmes :

1. L’agent est défectueux, la pulsation ne fonctionne pas correctement
2. L’agent ne démarre pas, impossible de se connecter aux services Log Analytics
3. Le Syslog de l’agent ne fonctionne pas
4. L’agent a une utilisation élevée du processeur/de la mémoire
5. L’agent a des problèmes d’installation
6. Les journaux personnalisés de l’agent ne fonctionnent pas
7. Collecter les journaux de l’agent

Pour plus d’informations, consultez notre [documentation GitHub](https://github.com/microsoft/OMS-Agent-for-Linux/blob/master/docs/Troubleshooting-Tool.md).

 >[!NOTE]
 >Si vous rencontrez un problème, exécutez l’outil collecteur de journaux. Le fait d’avoir les journaux dès le départ permettra à notre équipe de support de résoudre plus rapidement votre problème.

## <a name="purge-and-re-install-the-linux-agent"></a>Vider et réinstaller l’agent Linux

Nous avons vu qu’une nouvelle réinstallation de l’agent permet de résoudre la plupart des problèmes. En fait, il peut s’agir de la première suggestion de notre équipe de support technique pour que l’agent passe dans un état non altéré. Le fait d’exécuter l’utilitaire de résolution des problèmes, de collecter des journaux et de tenter une nouvelle réinstallation vous permet de résoudre plus rapidement les problèmes.

1. Téléchargez le script de vidage :
- `$ wget https://raw.githubusercontent.com/microsoft/OMS-Agent-for-Linux/master/tools/purge_omsagent.sh`
2. Exécutez le script de vidage (avec les autorisations sudo) :
- `$ sudo sh purge_omsagent.sh`

## <a name="important-log-locations-and-log-collector-tool"></a>Emplacement de journaux importants et outil collecteur de journaux

 Fichier | Path
 ---- | -----
 Fichier journal de l’agent Log Analytics pour Linux | `/var/opt/microsoft/omsagent/<workspace id>/log/omsagent.log`
 Fichier journal de configuration de l’agent Log Analytics | `/var/opt/microsoft/omsconfig/omsconfig.log`

 Nous vous recommandons d’utiliser notre outil collecteur de journaux d’activité pour récupérer les journaux d’activité importants à des fins de dépannage ou avant de soumettre un problème GitHub. Vous pouvez en savoir plus sur l’outil et son exécution [ici](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/master/tools/LogCollector/OMS_Linux_Agent_Log_Collector.md).

## <a name="important-configuration-files"></a>Fichiers de configuration importants

 Category | Emplacement du fichier
 ----- | -----
 syslog | `/etc/syslog-ng/syslog-ng.conf` ou `/etc/rsyslog.conf` ou `/etc/rsyslog.d/95-omsagent.conf`
 Performances, Nagios, Zabbix, sortie Log Analytics et agent général | `/etc/opt/microsoft/omsagent/<workspace id>/conf/omsagent.conf`
 Configurations supplémentaires | `/etc/opt/microsoft/omsagent/<workspace id>/conf/omsagent.d/*.conf`

 >[!NOTE]
 >La modification des fichiers de configuration pour les compteurs de performances et Syslog est remplacée si la collection est configurée à partir du [menu Données dans les paramètres avancés Log Analytics ](./agent-data-sources.md#configuring-data-sources) dans le portail Azure pour votre espace de travail. Pour désactiver la configuration de tous les agents, désactivez la collecte dans les **Paramètres avancés** Log Analytics, ou bien, pour un seul agent, exécutez la commande suivante :  
> `sudo su omsagent -c 'python /opt/microsoft/omsconfig/Scripts/OMS_MetaConfigHelper.py --disable'`

## <a name="installation-error-codes"></a>Codes d’erreur d’installation

| Code d'erreur | Signification |
| --- | --- |
| NOT_DEFINED | Les dépendances nécessaires n’étant pas installées, le plug-in auoms auditd n’est pas installé. | L’installation d’auoms a échoué, installez le package auditd. |
| 2 | Option non valide fournie au bundle de l’interpréteur de commandes. Exécutez `sudo sh ./omsagent-*.universal*.sh --help` pour l’utilisation. |
| 3 | Aucune option fournie au bundle de l’interpréteur de commandes. Exécutez `sudo sh ./omsagent-*.universal*.sh --help` pour l’utilisation. |
| 4 | Paramètres de proxy non valide ou type de package non valide ; les packages omsagent-*rpm*.sh peuvent être installés uniquement sur des systèmes basés sur RPM, et les packages omsagent -*deb*.sh peuvent être installés uniquement sur des systèmes Debian. Nous vous recommandons d’utiliser le programme d’installation universel de la [dernière version](../learn/quick-collect-linux-computer.md#install-the-agent-for-linux). Par ailleurs, lisez ces informations pour vérifier vos paramètres de proxy. |
| 5 | Le bundle de l’interpréteur de commandes doit être exécuté en tant qu’utilisateur root ou une erreur 403 a été retournée pendant l’intégration. Exécutez votre commande à l’aide de `sudo`. |
| 6 | Architecture du package non valide ou erreur 200 retournée pendant l’intégration ; les packages omsagent-*x64.sh ne peuvent être installés que sur des systèmes 64 bits, et les packages omsagent-* x86.sh ne peuvent être installés que sur des systèmes 32 bits. Téléchargez le package approprié pour votre architecture à partir de la [dernière version](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/latest). |
| 17 | Échec de l’installation du package OMS. Examinez le résultat de la commande pour déterminer l’échec de root. |
| 19 | Échec de l’installation du package OMI. Examinez le résultat de la commande pour déterminer l’échec de root. |
| 20 | Échec de l’installation du package SCX. Examinez le résultat de la commande pour déterminer l’échec de root. |
| 21 | Échec de l’installation des kits Provider. Examinez le résultat de la commande pour déterminer l’échec de root. |
| 22 | Échec de l’installation du package fourni. Examinez le résultat de la commande pour déterminer l’échec de root. |
| 23 | Package SCX ou OMI déjà installé. Utilisez `--upgrade` au lieu de `--install` pour installer le bundle de l’interpréteur de commandes. |
| 30 | Erreur d’application interne. Déposez un [problème GitHub](https://github.com/Microsoft/OMS-Agent-for-Linux/issues) en indiquant les détails de la sortie. |
| 55 | Version d’openssl non prise en charge OU connexion impossible à Azure Monitor OU dpkg est verrouillé OU programme curl manquant. |
| 61 | Bibliothèque de ctypes Python manquante. Installez la bibliothèque ou le package de ctypes Python (python-ctypes). |
| 62 | Programme tar manquant ; installez tar. |
| 63 | Programme sed manquant ; installez sed. |
| 64 | Programme curl manquant ; installez curl. |
| 65 | Programme gpg manquant ; installez gpg. |

## <a name="onboarding-error-codes"></a>Codes d’erreur d’intégration

| Code d'erreur | Signification |
| --- | --- |
| 2 | Option non valide fournie au script omsadmin. Exécutez `sudo sh /opt/microsoft/omsagent/bin/omsadmin.sh -h` pour l’utilisation. |
| 3 | Configuration non valide fournie au script omsadmin. Exécutez `sudo sh /opt/microsoft/omsagent/bin/omsadmin.sh -h` pour l’utilisation. |
| 4 | Proxy non valide fourni au script omsadmin. Vérifiez le proxy et consultez notre [documentation sur l’utilisation d’un proxy HTTP](log-analytics-agent.md#firewall-requirements). |
| 5 | Erreur HTTP 403 provenant d’Azure Monitor. Consultez la sortie complète du script omsadmin pour plus d’informations. |
| 6 | Erreur non HTTP 200 provenant d’Azure Monitor. Consultez la sortie complète du script omsadmin pour plus d’informations. |
| 7 | Impossible de se connecter à Azure Monitor. Consultez la sortie complète du script omsadmin pour plus d’informations. |
| 8 | Erreur d’intégration de l’espace de travail Log Analytics. Consultez la sortie complète du script omsadmin pour plus d’informations. |
| 30 | Erreur de script interne. Déposez un [problème GitHub](https://github.com/Microsoft/OMS-Agent-for-Linux/issues) en indiquant les détails de la sortie. |
| 31 | Erreur de génération d’ID d’agent. Déposez un [problème GitHub](https://github.com/Microsoft/OMS-Agent-for-Linux/issues) en indiquant les détails de la sortie. |
| 32 | Erreur de génération de certificats. Consultez la sortie complète du script omsadmin pour plus d’informations. |
| 33 | Erreur de génération d’une métaconfiguration pour omsconfig. Déposez un [problème GitHub](https://github.com/Microsoft/OMS-Agent-for-Linux/issues) en indiquant les détails de la sortie. |
| 34 | Script de génération de métaconfiguration absent. Recommencez l’intégration avec `sudo sh /opt/microsoft/omsagent/bin/omsadmin.sh -w <Workspace ID> -s <Workspace Key>`. |

## <a name="enable-debug-logging"></a>Activer l’enregistrement du débogage
### <a name="oms-output-plugin-debug"></a>Débogage dans le plug-in de sortie OMS
 FluentD prend en charge des niveaux de journalisation spécifiques au plug-in, ce qui vous permet de spécifier différents niveaux du journal en relation avec les entrées et sorties. Pour spécifier un autre niveau du journal pour une sortie OMS, modifiez la configuration générale de l’agent à l’emplacement `/etc/opt/microsoft/omsagent/<workspace id>/conf/omsagent.conf`.  

 Dans le plug-in de sortie OMS, avant la fin du fichier de configuration, changez la propriété `log_level` en remplaçant `info` par `debug` :

 ```
 <match oms.** docker.**>
  type out_oms
  log_level debug
  num_threads 5
  buffer_chunk_limit 5m
  buffer_type file
  buffer_path /var/opt/microsoft/omsagent/<workspace id>/state/out_oms*.buffer
  buffer_queue_limit 10
  flush_interval 20s
  retry_limit 10
  retry_wait 30s
</match>
 ```

La journalisation du débogage vous permet de voir les chargements par lot sur Azure Monitor, répartis par type, nombre d’éléments de données et temps d’envoi :

*Exemple de journal activé pour le débogage :*

```
Success sending oms.nagios x 1 in 0.14s
Success sending oms.omi x 4 in 0.52s
Success sending oms.syslog.authpriv.info x 1 in 0.91s
```

### <a name="verbose-output"></a>Sortie détaillée.
Au lieu d’utiliser le plug-in de sortie OMS, vous pouvez acheminer les éléments de données directement vers `stdout`, qui est visible dans le fichier journal de l’agent Log Analytics pour Linux.

Dans le fichier de configuration générale de l’agent Log Analytics sur `/etc/opt/microsoft/omsagent/<workspace id>/conf/omsagent.conf`, commentez le plug-in de sortie OMS en ajoutant le symbole `#` au début de chaque ligne :

```
#<match oms.** docker.**>
#  type out_oms
#  log_level info
#  num_threads 5
#  buffer_chunk_limit 5m
#  buffer_type file
#  buffer_path /var/opt/microsoft/omsagent/<workspace id>/state/out_oms*.buffer
#  buffer_queue_limit 10
#  flush_interval 20s
#  retry_limit 10
#  retry_wait 30s
#</match>
```

Sous le plug-in de sortie, supprimez les marques de commentaire de la section suivante en supprimant le symbole `#` au début de chaque ligne :

```
<match **>
  type stdout
</match>
```

## <a name="issue--unable-to-connect-through-proxy-to-azure-monitor"></a>Problème :  impossible de se connecter via le proxy à Azure Monitor

### <a name="probable-causes"></a>Causes probables
* Le proxy spécifié pendant l’intégration est incorrect
* Les points de terminaison des services Azure Monitor et Azure Automation ne figurent pas dans la liste approuvée de votre centre de données. 

### <a name="resolution"></a>Résolution
1. Réintégrez Azure Monitor avec l’agent Log Analytics pour Linux à l’aide de la commande suivante avec l’option `-v` activée. Cela autorise une sortie détaillée de l’agent qui se connecte via le proxy à Azure Monitor. 
`/opt/microsoft/omsagent/bin/omsadmin.sh -w <Workspace ID> -s <Workspace Key> -p <Proxy Conf> -v`

2. Pour vérifier que vous avez correctement configuré l’agent de manière à communiquer via un serveur proxy, consultez la section [Mettre à jour les paramètres du proxy](agent-manage.md#update-proxy-settings).    

3. Vérifiez que les points de terminaison décrits dans la liste [Configuration requise du pare-feu réseau](log-analytics-agent.md#firewall-requirements) Azure Monitor sont correctement ajoutés à une liste verte. Si vous utilisez Azure Automation, un lien d'accès aux étapes de configuration du réseau est également disponible ci-dessus.

## <a name="issue-you-receive-a-403-error-when-trying-to-onboard"></a>Problème : vous voyez une erreur 403 lorsque vous tentez d’effectuer l’intégration

### <a name="probable-causes"></a>Causes probables
* La date et l’heure sont incorrectes sur le serveur Linux 
* L’ID et la clé d’espace de travail utilisés sont incorrects

### <a name="resolution"></a>Résolution

1. Vérifiez l’heure sur votre serveur Linux avec la commande date. Si l’heure est à +/-15 minutes de l’heure actuelle, l’intégration échoue. Pour corriger ce problème, mettez à jour de la date et/ou le fuseau horaire de votre serveur Linux. 
2. Vérifiez que vous avez installé la dernière version de l’agent Log Analytics pour Linux.  Celle-ci vous avertit si le décalage horaire est à l’origine de l’échec d’intégration.
3. Relancez l’intégration avec la bonne clé et le bon identifiant d’espace de travail en suivant les instructions d’installation fournies précédemment dans cet article.

## <a name="issue-you-see-a-500-and-404-error-in-the-log-file-right-after-onboarding"></a>Problème : vous voyez une erreur 500 et 404 dans le fichier journal juste après l’intégration
Il s’agit d’un problème connu qui se produit lors du premier chargement de données Linux dans un espace de travail Log Analytics. Cela n’affecte pas les données envoyées ni l’expérience du service.


## <a name="issue-you-see-omiagent-using-100-cpu"></a>Problème : Vous voyez que l’omiagent utilise 100 % du processeur

### <a name="probable-causes"></a>Causes probables
Une régression dans le package nss-pem [v1.0.3-5.el7](https://centos.pkgs.org/7/centos-x86_64/nss-pem-1.0.3-7.el7.x86_64.rpm.html) a provoqué un problème de performances, récurrent dans les distributions Redhat/Centos 7.x. Pour en savoir plus sur ce problème, consultez la documentation suivante : Bogue [1667121 sur la régression des performances dans libcurl](https://bugzilla.redhat.com/show_bug.cgi?id=1667121).

Les bogues relatifs aux performances ne sont pas réguliers, et sont difficiles à reproduire. Si vous rencontrez ce genre de problèmes avec omiagent, vous devez utiliser le script omiHighCPUDiagnostics.sh afin de collecter les traces de l’omiagent lorsqu’il dépasse un certain seuil.

1. Télécharger le script <br/>
`wget https://raw.githubusercontent.com/microsoft/OMS-Agent-for-Linux/master/tools/LogCollector/source/omiHighCPUDiagnostics.sh`

2. Exécuter les diagnostics pendant 24 heures avec un seuil du processus de 30 % <br/>
`bash omiHighCPUDiagnostics.sh --runtime-in-min 1440 --cpu-threshold 30`

3. La pile d’appel sera vidée dans le fichier omiagent_trace. Si vous remarquez plusieurs appels de fonction Curl et NSS, suivez les étapes de résolution ci-dessous.

### <a name="resolution-step-by-step"></a>Résolution (étape par étape)

1. Mettez à niveau le package nss-pem vers la version [v1.0.3-5.el7_6.1](https://centos.pkgs.org/7/centos-x86_64/nss-pem-1.0.3-7.el7.x86_64.rpm.html). <br/>
`sudo yum upgrade nss-pem`

2. Si nss-pem n’est pas disponible pour la mise à niveau (c’est principalement le cas sur Centos), passez curl à la version antérieure 7.29.0-46. Si par inadvertance vous exécutez une « mise à jour yum », curl sera mis à niveau vers la version 7.29.0-51 et le problème surviendra à nouveau. <br/>
`sudo yum downgrade curl libcurl`

3. Redémarrez OMI : <br/>
`sudo scxadmin -restart`

## <a name="issue-you-are-not-seeing-any-data-in-the-azure-portal"></a>Problème : vous ne voyez pas les données dans le portail Azure

### <a name="probable-causes"></a>Causes probables

- Échec de l’intégration à Azure Monitor
- La connexion à Azure Monitor est bloquée
- Les données de l’agent Log Analytics pour Linux sont sauvegardées

### <a name="resolution"></a>Résolution
1. Vérifiez si l’intégration d’Azure Monitor a réussi en vous assurant que le fichier suivant existe : `/etc/opt/microsoft/omsagent/<workspace id>/conf/omsadmin.conf`
2. Relancer l’intégration à l’aide des instructions de la ligne de commande `omsadmin.sh`
3. Si vous utilisez un proxy, reportez-vous à la procédure de résolution de proxy fournie précédemment.
4. Dans certains cas, quand l’agent Log Analytics pour Linux ne peut pas communiquer avec le service, les données de l’agent sont mises en file d’attente dans la taille de mémoire tampon saturée de 50 Mo. Il est nécessaire de redémarrer l’agent en exécutant la commande suivante : `/opt/microsoft/omsagent/bin/service_control restart [<workspace id>]`. 

    >[!NOTE]
    >Ce problème est résolu dans l’agent version 1.1.0-28 et versions ultérieures.


## <a name="issue-you-are-not-seeing-forwarded-syslog-messages"></a>Problème : vous ne voyez pas les messages Syslog transférés 

### <a name="probable-causes"></a>Causes probables
* La configuration appliquée au serveur Linux n’autorise pas la collecte des fonctionnalités et/ou niveaux du journal envoyés.
* Le message Syslog n’est pas transmis correctement au serveur Linux.
* Le nombre de messages transmis par seconde est trop élevé pour être géré avec la configuration de base de l’agent Log Analytics pour Linux.

### <a name="resolution"></a>Résolution
* Vérifiez que la configuration de l’espace de travail Log Analytics pour Syslog possède toutes les fonctionnalités nécessaires et les niveaux corrects du journal d’activité. Consultez [Configurer la collecte Syslog dans le portail Azure](./data-sources-syslog.md#configure-syslog-in-the-azure-portal).
* Vérifiez que les démons de messagerie Syslog natifs (`rsyslog`, `syslog-ng`) peuvent recevoir les messages transférés
* Vérifiez les paramètres de pare-feu sur le serveur Syslog pour vous assurer que les messages ne sont pas bloqués.
* Simulez un message Syslog pour Log Analytics à l’aide de la commande `logger`.
  * `logger -p local0.err "This is my test message"`

## <a name="issue-you-are-receiving-errno-address-already-in-use-in-omsagent-log-file"></a>Problème : vous recevez une erreur Errno indiquant qu’une adresse est déjà en cours d’utilisation dans le fichier journal omsagent
Si vous voyez `[error]: unexpected error error_class=Errno::EADDRINUSE error=#<Errno::EADDRINUSE: Address already in use - bind(2) for "127.0.0.1" port 25224>` dans omsagent.log.

### <a name="probable-causes"></a>Causes probables
Cette erreur indique que l’extension Linux Diagnostic (LAD) est installée à côté de l’extension de machine virtuelle Linux Log Analytics, et qu’elle utilise le même port pour la collecte de données syslog qu’omsagent.

### <a name="resolution"></a>Résolution
1. En tant qu’utilisateur root, exécutez les commandes suivantes (25224 étant un exemple, il est possible que dans votre environnement vous voyiez un autre numéro de port utilisé par LAD) :

    ```
    /opt/microsoft/omsagent/bin/configure_syslog.sh configure LAD 25229

    sed -i -e 's/25224/25229/' /etc/opt/microsoft/omsagent/LAD/conf/omsagent.d/syslog.conf
    ```

    Vous devez ensuite modifier le fichier config `rsyslogd` ou `syslog_ng` correct et changer la configuration liée à LAD pour écrire sur le port 25229.

2. Si la machine virtuelle exécute `rsyslogd`, le fichier à modifier est : `/etc/rsyslog.d/95-omsagent.conf` (s’il existe, sinon `/etc/rsyslog`). Si la machine virtuelle exécute `syslog_ng`, le fichier à modifier est : `/etc/syslog-ng/syslog-ng.conf`.
3. Redémarrez omsagent (`sudo /opt/microsoft/omsagent/bin/service_control restart`).
4. Redémarrez le service syslog.

## <a name="issue-you-are-unable-to-uninstall-omsagent-using-purge-option"></a>Problème : vous ne parvenez pas à désinstaller omsagent en utilisant l’option de vidage

### <a name="probable-causes"></a>Causes probables

* L’extension de diagnostic Linux est installée
* L’extension de diagnostic Linux a été installée et désinstallée, mais vous voyez toujours une erreur indiquant qu’omsagent est utilisé par mdsd et qu’il ne peut pas être supprimé.

### <a name="resolution"></a>Résolution
1. Désinstallez l’extension de diagnostic Linux (LAD).
2. Supprimez les fichiers de l’extension de diagnostic Linux de la machine s’ils sont présents à l’emplacement suivant : `/var/lib/waagent/Microsoft.Azure.Diagnostics.LinuxDiagnostic-<version>/` et `/var/opt/microsoft/omsagent/LAD/`.

## <a name="issue-you-cannot-see-data-any-nagios-data"></a>Problème : vous ne pouvez pas voir de données Nagios 

### <a name="probable-causes"></a>Causes probables
* L’utilisateur omsagent n’a pas les autorisations nécessaires pour lire le fichier journal Nagios
* Les marques de commentaire associées au filtre et à la source Nagios n’ont pas été supprimées du fichier omsagent.conf

### <a name="resolution"></a>Résolution
1. Ajoutez l’utilisateur omsagent pour lire à partir du fichier Nagios en suivant ces [instructions](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/master/docs/OMS-Agent-for-Linux.md#nagios-alerts).
2. Dans le fichier de configuration générale de l’agent Log Analytics pour Linux situé à l’emplacement `/etc/opt/microsoft/omsagent/<workspace id>/conf/omsagent.conf`, vérifiez que la source et le filtre Nagios sont **tous deux** exempts de marques de commentaire.

    ```
    <source>
      type tail
      path /var/log/nagios/nagios.log
      format none
      tag oms.nagios
    </source>

    <filter oms.nagios>
      type filter_nagios_log
    </filter>
    ```

## <a name="issue-you-are-not-seeing-any-linux-data"></a>Problème : vous ne pouvez pas voir de données Linux 

### <a name="probable-causes"></a>Causes probables
* Échec de l’intégration à Azure Monitor
* La connexion à Azure Monitor est bloquée
* La machine virtuelle a été redémarrée
* Le package OMI a été mis à niveau manuellement vers une version plus récente que ce qu’a installé le package de l’agent Log Analytics pour Linux
* La ressource DSC journalise une erreur de type *classe introuvable* dans le fichier journal `omsconfig.log`
* Les données de l’agent Log Analytics sont sauvegardées
* DSC journalise le message *Aucune configuration actuelle n’existe. Exécutez la commande Start-DscConfiguration avec le paramètre -Path pour commencer par spécifier un fichier de configuration et créer une configuration actuelle.* dans le fichier journal `omsconfig.log`, mais aucun message de journal n’existe concernant les opérations `PerformRequiredConfigurationChecks`.

### <a name="resolution"></a>Résolution
1. Installez toutes les dépendances telles que le package auditd.
2. Vérifiez si l’intégration à Azure Monitor a réussi en vous assurant que le fichier suivant existe : `/etc/opt/microsoft/omsagent/<workspace id>/conf/omsadmin.conf`.  Si ce n’est pas le cas, relancez l’intégration avec les [instructions](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/master/docs/OMS-Agent-for-Linux.md#onboarding-using-the-command-line) de la ligne de commande omsadmin.sh.
4. Si vous utilisez un proxy, consultez les étapes de résolution de problèmes de proxy décrites plus haut.
5. Dans certains systèmes de distribution Azure, le démon de serveur OMI omid ne démarre pas après le redémarrage de la machine virtuelle. Ainsi, vous ne voyez pas les données de solution pour Audit, ChangeTracking ou UpdateManagement. La solution de contournement consiste à démarrer manuellement le serveur omi en exécutant `sudo /opt/omi/bin/service_control restart`.
6. Une fois le package OMI mis à niveau manuellement vers une version plus récente, il doit être redémarré manuellement pour que l’agent Log Analytics continue à fonctionner correctement. Cette étape est nécessaire pour certaines distributions où le serveur OMI ne démarre pas automatiquement après sa mise à niveau. Exécutez `sudo /opt/omi/bin/service_control restart` pour redémarrer OMI.
7. Si vous voyez une erreur de type *classe introuvable* pour les ressources DSC dans omsconfig.log, exécutez `sudo /opt/omi/bin/service_control restart`.
8. Dans certains cas, quand l’agent Log Analytics pour Linux ne peut pas parler à Azure Monitor, les données de l’agent sont sauvegardées dans la taille de mémoire tampon saturée : 50 Mo. Il est nécessaire de redémarrer l’agent en exécutant la commande suivante : `/opt/microsoft/omsagent/bin/service_control restart`.

    >[!NOTE]
    >Ce problème est résolu dans l’agent version 1.1.0-28 ou ultérieure.
    >

* Si le fichier journal `omsconfig.log` n’indique pas que des opérations `PerformRequiredConfigurationChecks` s’exécutent régulièrement sur le système, un problème affecte peut-être le service/travail cron. Assurez-vous que le travail cron existe sous `/etc/cron.d/OMSConsistencyInvoker`. Au besoin, exécutez les commandes suivantes pour créer le travail cron :

    ```
    mkdir -p /etc/cron.d/
    echo "*/15 * * * * omsagent /opt/omi/bin/OMSConsistencyInvoker >/dev/null 2>&1" | sudo tee /etc/cron.d/OMSConsistencyInvoker
    ```

    En outre, assurez-vous que le service cron est en cours d’exécution. Vous pouvez utiliser `service cron status` avec Debian, Ubuntu et SUSE, ou `service crond status` avec RHEL, CentOS et Oracle Linux pour vérifier l’état de ce service. Si le service n’existe pas, vous pouvez installer les fichiers binaires et le démarrer à l’aide des commandes suivantes :

    **Ubuntu/Debian**

    ```
    # To Install the service binaries
    sudo apt-get install -y cron
    # To start the service
    sudo service cron start
    ```

    **SUSE**

    ```
    # To Install the service binaries
    sudo zypper in cron -y
    # To start the service
    sudo systemctl enable cron
    sudo systemctl start cron
    ```

    **RHEL/CeonOS**

    ```
    # To Install the service binaries
    sudo yum install -y crond
    # To start the service
    sudo service crond start
    ```

    **Oracle Linux**

    ```
    # To Install the service binaries
    sudo yum install -y cronie
    # To start the service
    sudo service crond start
    ```

## <a name="issue-when-configuring-collection-from-the-portal-for-syslog-or-linux-performance-counters-the-settings-are-not-applied"></a>Problème : quand vous configurez la collection à partir du portail pour les compteurs de performances Syslog ou Linux, les paramètres ne sont pas appliqués

### <a name="probable-causes"></a>Causes probables
* L’agent Log Analytics pour Linux n’a pas récupéré la configuration la plus récente.
* Les paramètres changés dans le portail n’ont pas été appliqués.

### <a name="resolution"></a>Résolution
**Contexte :** `omsconfig` est l’agent de configuration de l’agent Log Analytics pour Linux qui cherche la nouvelle configuration côté portail toutes les cinq minutes. Cette configuration est ensuite appliquée aux fichiers de configuration de l’agent Log Analytics pour Linux situés à l’emplacement /etc/opt/microsoft/omsagent/conf/omsagent.conf.

* Dans certains cas, l’agent de configuration de l’agent Log Analytics pour Linux n’est pas en mesure de communiquer avec le service de configuration du portail, de sorte que la configuration la plus récente n’est pas appliquée.
  1. Vérifiez que l’agent `omsconfig` est installé en exécutant `dpkg --list omsconfig` ou `rpm -qi omsconfig`.  S’il n’est pas installé, réinstallez la dernière version de l’agent Log Analytics pour Linux.

  2. Vérifiez que l’agent `omsconfig` peut communiquer avec Azure Monitor en exécutant la commande suivante : `sudo su omsagent -c 'python /opt/microsoft/omsconfig/Scripts/GetDscConfiguration.py'`. Cette commande retourne la configuration que l’agent reçoit du service, dont les paramètres Syslog, les compteurs de performances Linux et les journaux d’activité personnalisés. Si cette commande échoue, exécutez la commande suivante : `sudo su omsagent -c 'python /opt/microsoft/omsconfig/Scripts/PerformRequiredConfigurationChecks.py'`. Cette commande force l’agent omsconfig à parler à Azure Monitor pour récupérer la configuration la plus récente.

## <a name="issue-you-are-not-seeing-any-custom-log-data"></a>Problème : vous ne pouvez pas voir de données de journal personnalisé 

### <a name="probable-causes"></a>Causes probables
* Échec de l’intégration à Azure Monitor.
* Le paramètre **Appliquer la configuration suivante à mes serveurs Linux** n’a pas été sélectionné.
* L’agent omsconfig n’a pas récupéré la dernière configuration de journal personnalisé du service.
* L’utilisateur `omsagent` de l’agent Log Analytics pour Linux ne parvient pas à accéder au journal personnalisé parce qu’il n’a pas les autorisations ou qu’il ne le trouve pas.  Les erreurs suivantes peuvent apparaître :
 * `[DATETIME] [warn]: file not found. Continuing without tailing it.`
 * `[DATETIME] [error]: file not accessible by omsagent.`
* Problème connu lié à une condition de concurrence corrigé dans la version 1.1.0-217 de l’agent Log Analytics pour Linux

### <a name="resolution"></a>Résolution
1. Vérifiez si l’intégration à Azure Monitor a réussi en vous assurant que le fichier suivant existe : `/etc/opt/microsoft/omsagent/<workspace id>/conf/omsadmin.conf`. Si ce n’est pas le cas, effectuez l’une des deux opérations suivantes :  

  1. Relancez l’intégration avec les [instructions](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/master/docs/OMS-Agent-for-Linux.md#onboarding-using-the-command-line) de la ligne de commande omsadmin.sh.
  2. Sous **Paramètres avancés** dans le portail Azure, vérifiez que le paramètre **Appliquer la configuration suivante à mes serveurs Linux** est activé.  

2. Vérifiez que l’agent `omsconfig` peut communiquer avec Azure Monitor en exécutant la commande suivante : `sudo su omsagent -c 'python /opt/microsoft/omsconfig/Scripts/GetDscConfiguration.py'`.  Cette commande retourne la configuration que l’agent reçoit du service, dont les paramètres Syslog, les compteurs de performances Linux et les journaux d’activité personnalisés. Si cette commande échoue, exécutez la commande suivante : `sudo su omsagent -c 'python /opt/microsoft/omsconfig/Scripts/PerformRequiredConfigurationChecks.py'`. Cette commande force l’agent omsconfig à parler à Azure Monitor pour récupérer la configuration la plus récente.

**Contexte** : au lieu que l’agent Log Analytics pour Linux opère en tant qu’utilisateur privilégié - `root`, l’agent opère en tant qu’utilisateur `omsagent`. Dans la plupart des cas, une autorisation explicite doit être accordée à cet utilisateur pour la lecture de certains fichiers. Pour accorder l’autorisation à l’utilisateur `omsagent`, exécutez les commandes suivantes :

1. Ajoutez l’utilisateur `omsagent` à un groupe spécifique : `sudo usermod -a -G <GROUPNAME> <USERNAME>`
2. Accordez l’accès en lecture universel au fichier requis : `sudo chmod -R ugo+rx <FILE DIRECTORY>`

Il existe un problème connu lié à une condition de concurrence affectant les versions de l’agent Log Analytics pour Linux antérieures à 1.1.0-217. Après mise à jour vers le dernier agent, exécutez la commande suivante pour obtenir la dernière version du plug-in de sortie : `sudo cp /etc/opt/microsoft/omsagent/sysconf/omsagent.conf /etc/opt/microsoft/omsagent/<workspace id>/conf/omsagent.conf`.

## <a name="issue-you-are-trying-to-reonboard-to-a-new-workspace"></a>Problème : vous tentez d’effectuer une réintégration dans un nouvel espace de travail
Quand vous tentez de réintégrer un agent dans un espace de travail, la configuration de l’agent Log Analytics doit être nettoyée avant la réintégration. Pour nettoyer l’ancienne configuration de l’agent, exécutez le bundle de l’interpréteur de commandes avec `--purge`

```
sudo sh ./omsagent-*.universal.x64.sh --purge
```
ou

```
sudo sh ./onboard_agent.sh --purge
```

Vous pouvez poursuivre la réintégration après avoir utilisé l’option `--purge`

## <a name="log-analytics-agent-extension-in-the-azure-portal-is-marked-with-a-failed-state-provisioning-failed"></a>L’extension de l’agent Log Analytics dans le portail Azure est marquée avec un état d’échec : Échec de l’approvisionnement

### <a name="probable-causes"></a>Causes probables
* L’agent Log Analytics a été supprimé du système d’exploitation.
* Le service de l’agent Log Analytics est arrêté, désactivé ou n’est pas configuré.

### <a name="resolution"></a>Résolution 
Effectuez les étapes suivantes pour résoudre le problème.
1. Supprimez l’extension du portail Azure.
2. Installez l’agent en suivant les [instructions](../learn/quick-collect-linux-computer.md).
3. Redémarrez l’agent en exécutant la commande suivante : `sudo /opt/microsoft/omsagent/bin/service_control restart`.
* Patientez quelques minutes ; l’état du provisionnement passe alors à **Approvisionnement réussi**.


## <a name="issue-the-log-analytics-agent-upgrade-on-demand"></a>Problème : Mise à jour de l’agent Log Analytics à la demande

### <a name="probable-causes"></a>Causes probables

Les packages de l’agent Log Analytics sur l’hôte sont obsolètes.

### <a name="resolution"></a>Résolution 
Effectuez les étapes suivantes pour résoudre le problème.

1. Recherchez la dernière version dans cette [page](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/).
2. Téléchargez le script d’installation (1.4.2-124, à titre d’exemple de version) :

    ```
    wget https://github.com/Microsoft/OMS-Agent-for-Linux/releases/download/OMSAgent_GA_v1.4.2-124/omsagent-1.4.2-124.universal.x64.sh
    ```

3. Mettez à niveau les packages en exécutant `sudo sh ./omsagent-*.universal.x64.sh --upgrade`.

---
title: Échec du démarrage d’Apache HBase Master dans Azure HDInsight
description: Échec du démarrage d’Apache HBase Master (HMaster) dans Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 08/14/2019
ms.openlocfilehash: 032c25969bf477e1163b8db2aca631044c457939
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92539970"
---
# <a name="apache-hbase-master-hmaster-fails-to-start-in-azure-hdinsight"></a>Échec du démarrage d’Apache HBase Master (HMaster) dans Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

## <a name="scenario-atomic-renaming-failure"></a>Scénario : Échec de renommage atomique

### <a name="issue"></a>Problème

Fichiers inattendus identifiés au cours du processus de démarrage.

### <a name="cause"></a>Cause

Pendant le processus de démarrage, HMaster effectue de nombreuses étapes d’initialisation, notamment le déplacement des données du dossier temporaire (. tmp) vers le dossier de données. HMaster examine également le dossier de journaux d’activité WAL (write-ahead log) pour voir s’il existe des serveurs régionaux ne répondant pas.

HMaster effectue une commande de liste de base sur les dossiers WAL. S’il rencontre un fichier inattendu dans l’un de ces dossiers, il envoie une exception et ne démarre pas.

### <a name="resolution"></a>Résolution

Vérifiez la pile des appels et essayez de déterminer quel dossier pourrait être responsable du problème (par exemple le dossier de journaux WAL ou le dossier .tmp). Ensuite, dans Cloud Explorer ou à l’aide de commandes HDFS, tentez de localiser le fichier posant problème. En règle générale, il s’agit d’un fichier `*-renamePending.json`. (Le fichier `*-renamePending.json` est un fichier journal utilisé pour implémenter l’opération de renommage atomique au niveau du pilote WASB. En raison des bogues de cette implémentation, ces fichiers peuvent rester après le blocage du processus, etc.) Forcez la suppression de ce fichier soit dans Cloud Explorer, soit à l’aide de commandes HDFS.

Dans certains cas, cet emplacement contient également un fichier temporaire nommé `$$$.$$$`. Vous devez utiliser la commande HDFS `ls` pour voir ce fichier ; vous ne pouvez pas le voir dans Cloud Explorer. Pour supprimer ce fichier, utilisez la commande HDFS `hdfs dfs -rm /\<path>\/\$\$\$.\$\$\$`.

Après avoir exécuté ces commandes, HMaster devrait démarrer immédiatement.

---

## <a name="scenario-no-server-address-listed"></a>Scénario : Aucune adresse de serveur listée

### <a name="issue"></a>Problème

Vous pouvez voir un message indiquant que la table `hbase: meta` n’est pas en ligne. L’exécution de `hbck` peut indiquer `hbase: meta table replicaId 0 is not found on any region.` Dans les journaux d’activité HMaster, vous pouvez voir le message: `No server address listed in hbase: meta for region hbase: backup <region name>`.  

### <a name="cause"></a>Cause

HMaster n’a pas pu être initialisé après le redémarrage de HBase.

### <a name="resolution"></a>Résolution

1. Entrez la commande suivante dans l’interpréteur de commandes HBase (modifiez les valeurs le cas échéant) :

    ```hbase
    scan 'hbase:meta'
    delete 'hbase:meta','hbase:backup <region name>','<column name>'
    ```

1. Supprimez l’entrée `hbase: namespace`. Cette entrée peut être à l’origine de l’erreur signalée quand la table `hbase: namespace` est analysée.

1. Redémarrez le service HMaster actif à partir de l’interface utilisateur Ambari pour rétablir le fonctionnement de HBase.

1. Dans l’interpréteur de commandes HBase, exécutez la commande suivante pour afficher toutes les tables hors connexion :

    ```hbase
    hbase hbck -ignorePreCheckPermission -fixAssignments
    ```

---

## <a name="scenario-javaioioexception-timedout"></a>Scénario : java.io.IOException: Timedout

### <a name="issue"></a>Problème

HMaster arrive à expiration avec une exception irrécupérable similaire à `java.io.IOException: Timedout 300000ms waiting for namespace table to be assigned`.

### <a name="cause"></a>Cause

Vous pourriez rencontrer ce problème si plusieurs tables et régions n’ont pas été vidées lors du redémarrage de vos services HMaster. Le délai d’attente est un défaut connu de HMaster. Les tâches générales de démarrage du cluster peuvent prendre beaucoup de temps. HMaster s’arrête si la table namespace n’a pas encore été affectée. Les tâches de démarrage fastidieuses se produisent quand une grande quantité de données n’a pas été vidée et qu’un délai d’attente de cinq minutes est insuffisant.

### <a name="resolution"></a>Résolution

1. Dans l’interface utilisateur d’Apache Ambari, accédez à **HBase** > **Configs** . Dans le fichier `hbase-site.xml` personnalisé, ajoutez le paramètre suivant :

    ```
    Key: hbase.master.namespace.init.timeout Value: 2400000  
    ```

1. Redémarrez les services requis (HMaster et éventuellement d’autres services HBase).

---

## <a name="scenario-frequent-region-server-restarts"></a>Scénario : Redémarrages fréquents de serveurs régionaux

### <a name="issue"></a>Problème

Les nœuds redémarrent périodiquement. Dans les journaux des serveurs régionaux, vous pouvez voir des entrées similaires à celles-ci :

```
2017-05-09 17:45:07,683 WARN  [JvmPauseMonitor] util.JvmPauseMonitor: Detected pause in JVM or host machine (eg GC): pause of approximately 31000ms
2017-05-09 17:45:07,683 WARN  [JvmPauseMonitor] util.JvmPauseMonitor: Detected pause in JVM or host machine (eg GC): pause of approximately 31000ms
2017-05-09 17:45:07,683 WARN  [JvmPauseMonitor] util.JvmPauseMonitor: Detected pause in JVM or host machine (eg GC): pause of approximately 31000ms
```

### <a name="cause"></a>Cause

Longue pause de nettoyage de la mémoire de machine virtuelle Java de `regionserver`. La pause entraîne l’absence de réponse de `regionserver` et l’impossibilité d’envoyer une pulsation à HMaster dans les 40 secondes de délai d’expiration de la session zk. HMaster estime que `regionserver` est inactif et l’abandonne, puis redémarre.

### <a name="resolution"></a>Résolution

Modifiez le délai d’expiration de la session Zookeeper ; non seulement le paramètre `hbase-site``zookeeper.session.timeout`, mais également le paramètre Zookeeper `zoo.cfg``maxSessionTimeout` doit être modifié.

1. Dans l’interface utilisateur Ambari, accédez à **HBase -> Configs -> Paramètres** , dans la section Délais d’expiration, modifiez la valeur du délai d’expiration de la session Zookeeper.

1. Dans l’interface utilisateur Ambari, accédez à **Zookeeper -> Configs -> Custom (Personnalisée)** `zoo.cfg`, puis ajoutez ou modifiez le paramètre suivant. Vérifiez que la valeur est identique à HBase `zookeeper.session.timeout`.

    ```
    Key: maxSessionTimeout Value: 120000  
    ```

1. Redémarrez les services requis.

---

## <a name="scenario-log-splitting-failure"></a>Scénario : Échec du fractionnement du journal

### <a name="issue"></a>Problème

HMaster n’apparaît pas sur un cluster HBase.

### <a name="cause"></a>Cause

Paramètres HDFS et HBase mal configurés pour un compte de stockage secondaire.

### <a name="resolution"></a>Résolution

Définissez hbase.rootdir: wasb://@.blob.core.windows.net/hbase et redémarrez les services sur Ambari.

---

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article ou si vous ne parvenez pas à le résoudre, utilisez un des canaux suivants pour obtenir de l’aide :

* Obtenez des réponses de la part d’experts Azure en faisant appel au [Support de la communauté Azure](https://azure.microsoft.com/support/community/).

* Connectez-vous à [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client. Connexion de la communauté Azure aux ressources appropriées : réponses, support technique et experts.

* Si vous avez besoin d’une aide supplémentaire, vous pouvez envoyer une requête de support à partir du [Portail Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Sélectionnez **Support** dans la barre de menus, ou ouvrez le hub **Aide + Support** . Pour plus d’informations, consultez [Création d’une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). L’accès au support relatif à la gestion et à la facturation des abonnements est inclus avec votre abonnement Microsoft Azure. En outre, le support technique est fourni avec l’un des [plans de support Azure](https://azure.microsoft.com/support/plans/).
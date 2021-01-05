---
title: Résoudre les problèmes liés aux actions de script dans Azure HDInsight
description: Étapes générales de résolution des problèmes liés aux actions de script dans Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: troubleshooting
ms.custom: seoapr2020
ms.date: 04/21/2020
ms.openlocfilehash: ef9322c17a20ab5bfcf348649a1272dd4f301c5c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000672"
---
# <a name="troubleshoot-script-actions-in-azure-hdinsight"></a>Résoudre les problèmes liés aux actions de script dans Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

## <a name="viewing-logs"></a>Consultation des journaux

Vous pouvez utiliser l’interface utilisateur web d’Apache Ambari pour afficher les informations enregistrées par des actions de script. Si le script échoue lors de la création du cluster, les journaux se trouvent dans le compte de stockage de cluster par défaut. Cette section fournit des informations sur la façon de récupérer les journaux d’activité à l’aide de ces deux options.

### <a name="apache-ambari-web-ui"></a>Interface utilisateur web d’Apache Ambari

1. Dans un navigateur web, accédez à `https://CLUSTERNAME.azurehdinsight.net`, où `CLUSTERNAME` est le nom de votre cluster.

1. Dans la barre située en haut de la page, sélectionnez l’entrée **ops**. Une liste affiche les opérations en cours et précédentes effectuées sur le cluster via Ambari.

    ![Barre de l’interface utilisateur web d’Ambari avec ops sélectionné](./media/troubleshoot-script-action/hdi-apache-ambari-nav.png)

1. Recherchez les entrées comportant **run\_customscriptaction** dans la colonne **Operations**. Ces entrées sont créées lors de l’exécution des actions de script.

    ![Apache Ambari - Action de script, opérations](./media/troubleshoot-script-action/ambari-script-action.png)

    Pour voir les sorties **STDOUT** et **STDERR**, sélectionnez l’entrée **run\customscriptaction** et suivez les différents liens. Une sortie est générée à chaque exécution du script. Elle contient des informations potentiellement utiles.

### <a name="default-storage-account"></a>Compte de stockage par défaut

Si la création du cluster échoue en raison d’une erreur de script, les journaux d’activité sont conservés dans le compte de stockage du cluster.

* Les journaux d’activité de stockage sont disponibles dans `\STORAGE_ACCOUNT_NAME\DEFAULT_CONTAINER_NAME\custom-scriptaction-logs\CLUSTER_NAME\DATE`.

    ![Journaux d’actions de script](./media/troubleshoot-script-action/script-action-logs-in-storage.png)

    Sous ce répertoire, les journaux d’activité sont organisés séparément pour le **nœud principal**, le **nœud worker** et le **nœud zookeeper**. Regardez les exemples suivants :

    * **Nœud principal** : `<ACTIVE-HEADNODE-NAME>.cloudapp.net`

    * **Nœud worker** : `<ACTIVE-WORKERNODE-NAME>.cloudapp.net`

    * **Nœud zookeeper** : `<ACTIVE-ZOOKEEPERNODE-NAME>.cloudapp.net`

* Toutes les valeurs **stdout** et **stderr** de l’hôte correspondant sont chargées vers le compte de stockage. Il existe un fichier **output-\*.txt** et un fichier **errors-\*.txt** pour chaque action de script. Le fichier **output-*.txt** contient des informations sur l’URI du script exécuté sur l’ordinateur hôte. Le texte suivant constitue un exemple de ces informations :

    ```output
    'Start downloading script locally: ', u'https://hdiconfigactions.blob.core.windows.net/linuxrconfigactionv01/r-installer-v01.sh'
    ```

* Vous pouvez créer plusieurs fois un cluster d’action de script portant le même nom. Dans ce cas, vous pouvez différencier les journaux d’activité correspondants par le nom de dossier **DATE**. Par exemple, la structure de dossiers d’un cluster, **mycluster**, créé à différentes dates ressemble aux entrées de journaux suivantes :

    `\STORAGE_ACCOUNT_NAME\DEFAULT_CONTAINER_NAME\custom-scriptaction-logs\mycluster\2015-10-04` `\STORAGE_ACCOUNT_NAME\DEFAULT_CONTAINER_NAME\custom-scriptaction-logs\mycluster\2015-10-05`

* Si vous créez un cluster d’action de script avec le même nom le même jour, vous pouvez utiliser le préfixe unique pour identifier les fichiers journaux correspondants.

* Si vous créez un cluster vers minuit (0 h 00), il est possible que les fichiers journaux s’étendent sur deux jours. Dans ce cas, vous voyez deux dossiers de date différente pour le même cluster.

* Le chargement des fichiers journaux vers le conteneur par défaut peut prendre jusqu’à cinq minutes, en particulier si les clusters sont de grande taille. Par conséquent, si vous souhaitez accéder aux journaux d’activité, vous ne devez pas immédiatement supprimer le cluster en cas d’échec d’une action de script.

## <a name="ambari-watchdog"></a>Agent de surveillance Ambari

Ne changez pas le mot de passe de l’agent de surveillance Ambari (hdinsightwatchdog) sur votre cluster HDInsight Linux. Une modification de mot de passe annule la possibilité d’exécuter de nouvelles actions de script sur le cluster HDInsight.

## <a name="cant-import-name-blobservice"></a>Impossible d’importer le nom BlobService

__Symptômes__. L’action de script échoue. Un texte semblable à l’erreur suivante s’affiche lorsque vous observez l’opération dans Ambari :

```
Traceback (most recent call list):
  File "/var/lib/ambari-agent/cache/custom_actions/scripts/run_customscriptaction.py", line 21, in <module>
    from azure.storage.blob import BlobService
ImportError: cannot import name BlobService
```

__Cause__. Cette erreur se produit si vous mettez à niveau le client Stockage Azure Python qui est inclus dans le cluster HDInsight. HDInsight attend le client de stockage Azure 0.20.0.

__Résolution__. Pour résoudre cette erreur, connectez-vous manuellement à chaque nœud de cluster avec `ssh`. Exécutez la commande suivante pour réinstaller la version correcte du client de stockage :

```bash
sudo pip install azure-storage==0.20.0
```

Pour plus d’informations sur la connexion au cluster via SSH, consultez [Se connecter à HDInsight (Apache Hadoop) avec SSH](hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="history-doesnt-show-the-scripts-used-during-cluster-creation"></a>L’historique n’affiche pas les scripts utilisés pendant la création du cluster

Si votre cluster a été créé avant le 15 mars 2016, il est possible que l’historique des actions de script n’affiche pas de script. Le redimensionnement du cluster provoque l’affichage des scripts dans l’historique des actions de script.

Deux exceptions :

* Votre cluster a été créé avant le 1er septembre 2015. Il s’agit de la date à laquelle les actions de script ont été introduites. Tout cluster créé avant cette date n’aurait pas pu utiliser les actions de script pour la création du cluster.

* Vous avez utilisé plusieurs actions de script durant la création du cluster. Ou vous avez donné le même nom à plusieurs scripts, ou utilisé le même nom et le même URI, mais des paramètres différents pour plusieurs scripts. Dans tous ces cas, l’erreur suivante s’affiche :

    ```
    No new script actions can be run on this cluster because of conflicting script names in existing scripts. Script names provided at cluster creation must be all unique. Existing scripts are run on resize.
    ```

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../includes/hdinsight-troubleshooting-next-steps.md)]
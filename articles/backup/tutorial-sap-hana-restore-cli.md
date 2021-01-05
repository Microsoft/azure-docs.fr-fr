---
title: Tutoriel - Restauration de base de données SAP HANA sur Azure à l’aide de l’interface CLI
description: Dans ce tutoriel, vous allez découvrir comment restaurer des bases de données SAP HANA s’exécutant sur une machine virtuelle Azure à partir d’un coffre Recovery Services de Sauvegarde Azure à l’aide d’Azure CLI.
ms.topic: tutorial
ms.date: 12/4/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 0e524bfe090f0d67b76c13e876f44e83986aeb9e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91334801"
---
# <a name="tutorial-restore-sap-hana-databases-in-an-azure-vm-using-azure-cli"></a>Tutoriel : Restaurer des bases de données SAP HANA sur une machine virtuelle Azure à l’aide d’Azure CLI

Azure CLI permet de créer et de gérer des ressources Azure à partir de la ligne de commande ou par le biais de scripts. Ce document explique comment restaurer une base de données SAP HANA sauvegardée sur une machine virtuelle Azure à l’aide d’Azure CLI. Vous pouvez également effectuer ces étapes à l’aide du [portail Azure](./sap-hana-db-restore.md).

Utilisez [Azure Cloud Shell](tutorial-sap-hana-backup-cli.md) pour exécuter les commandes CLI.

À la fin de ce tutoriel, vous serez en mesure d’effectuer les opérations suivantes :

> [!div class="checklist"]
>
> * Afficher les points de restauration d’une base de données sauvegardée
> * Restaurer une base de données

Ce tutoriel part du principe que vous disposez d’une base de données SAP HANA s’exécutant sur une machine virtuelle Azure qui est sauvegardée à l’aide de Sauvegarde Azure. Si vous avez appliqué la procédure de l’article [Sauvegarder une base de données SAP HANA dans Azure à l’aide de l’interface CLI](tutorial-sap-hana-backup-cli.md) pour sauvegarder votre base de données SAP HANA, vous utiliserez les ressources suivantes :

* Un groupe de ressources nommé *saphanaResourceGroup*
* Un coffre nommé *saphanaVault*
* Un conteneur protégé nommé *VMAppContainer;Compute;saphanaResourceGroup;saphanaVM*
* Un élément/base de données sauvegardé nommé *saphanadatabase;hxe;hxe*
* Des ressources dans la région *westus2*

## <a name="view-restore-points-for-a-backed-up-database"></a>Afficher les points de restauration d’une base de données sauvegardée

Pour afficher la liste de tous les points de récupération d’une base de données, utilisez l’applet de commande [az backup recoverypoint list](/cli/azure/backup/recoverypoint#az-backup-recoverypoint-show-log-chain) :

```azurecli-interactive
az backup recoverypoint list --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --item-name saphanadatabase;hxe;hxe \
   --output table
```

La liste des points de récupération se présente comme suit :

```output
Name                      Time                               BackupManagementType   Item Name               RecoveryPointType
-------------------       ---------------------------------  ---------------------  ----------------------  ------------------
7660777527047692711       2019-12-10T04:00:32.346000+00:00   AzureWorkload          SAPHanaDtabase;hxe;hxe  Full
7896624824685666836       2019-12-15T10:33:32.346000+00:00   AzureWorkload          SAPHanaDtabase;hxe;hxe  Differential
DefaultRangeRecoveryPoint                                    AzureWorkload          SAPHanaDtabase;hxe;hxe  Log
```

Comme vous pouvez le voir, la liste ci-dessus contient trois points de récupération : un pour la sauvegarde complète, un pour la sauvegarde différentielle et un pour la sauvegarde des journaux.

>[!NOTE]
>Vous pouvez également afficher les points de début et de fin de chaque chaîne de sauvegarde de journaux non interrompue à l’aide de l’applet de commande [az backup recoverypoint show-log-chain](/cli/azure/backup/recoverypoint#az-backup-recoverypoint-show-log-chain).

## <a name="prerequisites-to-restore-a-database"></a>Prérequis pour restaurer une base de données

Avant de restaurer une base de données, vérifiez que les prérequis suivants sont remplis :

* Vous pouvez restaurer la base de données uniquement sur une instance de SAP HANA qui se trouve dans la même région.
* L’instance cible doit être inscrite auprès du même coffre que la source.
* Sauvegarde Azure ne peut pas identifier deux instances de SAP HANA différentes sur la même machine virtuelle. Ainsi, il est impossible de restaurer des données d’une instance vers une autre sur la même machine virtuelle.

## <a name="restore-a-database"></a>Restaurer une base de données

Sauvegarde Azure peut restaurer des bases de données SAP HANA s’exécutant sur des machines virtuelles Azure comme suit :

* Restaurer à une date ou une heure spécifique (à la seconde), en utilisant les sauvegardes de fichiers journaux. Sauvegarde Azure détermine automatiquement les sauvegardes différentielles complètes appropriées, et la chaîne des sauvegardes de fichiers journaux nécessaires pour restaurer en fonction de la date/heure sélectionnée.
* Restaurer en fonction d’une sauvegarde complète ou différentielle spécifique à un point de récupération spécifique.

Pour restaurer une base de données, utilisez l’applet de commande [az restore restore-azurewl](/cli/azure/backup/restore#az-backup-restore-restore-azurewl), qui nécessite entre autres un objet de configuration de récupération comme entrée. Vous pouvez générer cet objet à l’aide de l’applet de commande [az backup recoveryconfig show](/cli/azure/backup/recoveryconfig#az-backup-recoveryconfig-show). L’objet de configuration de récupération contient tous les détails nécessaires pour effectuer une restauration, notamment le mode de restauration : **OriginalWorkloadRestore** ou **AlternateWorkloadRestore**.

>[!NOTE]
> **OriginalWorkloadRestore** : restaure les données sur la même instance de SAP HANA que la source d’origine. Cette option remplace la base de données d’origine. <br>
> **AlternateWorkloadRestore** : restaure la base de données vers un autre emplacement et conserve la base de données source d’origine.

## <a name="restore-to-alternate-location"></a>Restaurer à un autre emplacement

Pour restaurer une base de données à un autre emplacement, utilisez **AlternateWorkloadRestore** comme mode de restauration. Vous devez ensuite choisir le point de restauration, qui peut être un point antérieur dans le temps ou l’un des points de restauration précédents.

Dans ce tutoriel, vous allez restaurer à un point de restauration précédent. [Affichez la liste des points de restauration](#view-restore-points-for-a-backed-up-database) pour la base de données et choisissez celui vers lequel vous souhaitez effectuer la restauration. Ce tutoriel utilise le point de restauration nommé *7660777527047692711*.

À l’aide du nom du point de restauration et du mode de restauration ci-dessus, nous allons créer l’objet de configuration de récupération avec l’applet de commande [az backup recoveryconfig show](/cli/azure/backup/recoveryconfig#az-backup-recoveryconfig-show). Examinons ce que signifie chacun des autres paramètres de cette applet de commande :

* **--target-item-name** : il s’agit du nom qu’aura la base de données restaurée. Ici, nous avons utilisé le nom *restored_database*.
* **--target-server-name** : il s’agit du nom d’un serveur SAP HANA qui est correctement inscrit auprès d’un coffre Recovery Services et se trouve dans la même région que la base de données à restaurer. Dans ce tutoriel, nous allons restaurer la base de données sur le serveur SAP HANA que nous avons protégé, nommé *hxehost*.
* **--target-server-type** : pour la restauration des bases de données SAP HANA, vous devez utiliser **HANAInstance**.

```azurecli-interactive

az backup recoveryconfig show --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --item-name saphanadatabase;hxe;hxe \
    --restore-mode AlternateWorkloadRestore \
    --rp-name 7660777527047692711 \
    --target-item-name restored_database \
    --target-server-name hxehost \
    --target-server-type HANAInstance \
    --workload-type SAPHANA \
    --output json
```

La réponse à la requête ci-dessus est un objet de configuration de récupération qui ressemble à ceci :

```output
{"restore_mode": "AlternateLocation", "container_uri": " VMAppContainer;Compute;saphanaResourceGroup;saphanaVM ", "item_uri": "SAPHanaDatabase;hxe;hxe", "recovery_point_id": "7660777527047692711", "item_type": "SAPHana", "source_resource_id": "/subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/saphanaResourceGroup/providers/Microsoft.Compute/virtualMachines/saphanavm", "database_name": null, "container_id": null, "alternate_directory_paths": null}
```

À présent, pour restaurer la base de données, exécutez l’applet de commande [az restore restore-azurewl](/cli/azure/backup/restore#az-backup-restore-restore-azurewl). Pour utiliser cette commande, nous allons entrer la sortie JSON ci-dessus enregistrée dans un fichier nommé *recoveryconfig.json*.

```azurecli-interactive
az backup restore restore-azurewl --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --restore-config recoveryconfig.json \
    --output table
```

La sortie se présente ainsi :

```output
Name                                  Resource
------------------------------------  -------
5b198508-9712-43df-844b-977e5dfc30ea  SAPHANA
```

La réponse vous donnera le nom du travail. Vous pouvez utiliser ce nom pour effectuer le suivi de l’état du travail à l’aide de l’applet de commande [az backup job show](/cli/azure/backup/job#az-backup-job-show).

## <a name="restore-and-overwrite"></a>Restaurer et remplacer

Pour restaurer à l’emplacement d’origine, nous allons utiliser **OrignialWorkloadRestore** comme mode de restauration. Vous devez ensuite choisir le point de restauration, qui peut être un point antérieur dans le temps ou l’un des points de restauration précédents.

Pour ce tutoriel, nous allons choisir le point dans le temps précédent « 28-11-2019-09:53:00 » pour la restauration. Vous pouvez spécifier ce point de restauration aux formats suivants : jj-mm-aaaa, jj-mm-aaaa-hh:mm:ss. Pour choisir un point dans le temps valide pour la restauration, utilisez l’applet de commande [az backup recoverypoint show-log-chain](/cli/azure/backup/recoverypoint#az-backup-recoverypoint-show-log-chain), qui liste les intervalles de sauvegardes de chaîne de journaux non interrompues.

```azurecli-interactive
az backup recoveryconfig show --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --item-name saphanadatabase;hxe;hxe \
    --restore-mode OriginalWorkloadRestore \
    --log-point-in-time 28-11-2019-09:53:00 \
    --output json
```

La réponse à la requête ci-dessus est un objet de configuration de récupération qui ressemble à ceci :

```output
{"restore_mode": "OriginalLocation", "container_uri": " VMAppContainer;Compute;saphanaResourceGroup;saphanaVM ", "item_uri": "SAPHanaDatabase;hxe;hxe", "recovery_point_id": "DefaultRangeRecoveryPoint", "log_point_in_time": "28-11-2019-09:53:00", "item_type": "SAPHana", "source_resource_id": "/subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/saphanaResourceGroup/providers/Microsoft.Compute/virtualMachines/saphanavm", "database_name": null, "container_id": null, "alternate_directory_paths": null}"
```

À présent, pour restaurer la base de données, exécutez l’applet de commande [az restore restore-azurewl](/cli/azure/backup/restore#az-backup-restore-restore-azurewl). Pour utiliser cette commande, nous allons entrer la sortie JSON ci-dessus enregistrée dans un fichier nommé *recoveryconfig.json*.

```azurecli-interactive
az backup restore restore-azurewl --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --restore-config recoveryconfig.json \
    --output table
```

La sortie se présente ainsi :

```output
Name                                  Resource
------------------------------------  --------
5b198508-9712-43df-844b-977e5dfc30ea  SAPHANA
```

La réponse vous donnera le nom du travail. Vous pouvez utiliser ce nom pour effectuer le suivi de l’état du travail à l’aide de l’applet de commande [az backup job show](/cli/azure/backup/job#az-backup-job-show).

## <a name="restore-as-files"></a>Restaurer sous forme de fichiers

Pour restaurer les données de sauvegarde sous forme de fichiers plutôt que sous forme de base de données, nous allons utiliser **RestoreAsFiles** comme mode de restauration. Choisissez ensuite le point de restauration, qui peut être un point antérieur dans le temps ou l’un des points de restauration précédents. Une fois les fichiers vidés sous un chemin d’accès spécifié, vous pouvez placer ces fichiers sur n’importe quel ordinateur SAP HANA sur lequel vous souhaitez les restaurer sous forme de base de données. Étant donné que vous pouvez déplacer ces fichiers sur n’importe quel ordinateur, vous pouvez désormais restaurer les données entre les abonnements et les régions.

Pour ce tutoriel, nous allons choisir le point dans le temps précédent `28-11-2019-09:53:00` vers lequel effectuer la restauration, ainsi que l’emplacement de vidage des fichiers de sauvegarde comme `/home/saphana/restoreasfiles` sur le même serveur SAP HANA. Vous pouvez spécifier ce point de restauration dans l’un des formats suivants : **jj-mm-aaaa** ou **jj-mm-aaaa-hh:mm:ss**. Pour choisir un point dans le temps valide pour la restauration, utilisez l’applet de commande [az backup recoverypoint show-log-chain](/cli/azure/backup/recoverypoint#az-backup-recoverypoint-show-log-chain), qui liste les intervalles de sauvegardes de chaîne de journaux non interrompues.

À l’aide du nom du point de restauration ci-dessus et du mode de restauration, créons l’objet de configuration de récupération avec l’applet de commande [az backup recoveryconfig show](/cli/azure/backup/recoveryconfig#az-backup-recoveryconfig-show). Examinons ce que signifie chacun des autres paramètres de cette applet de commande :

* **--target-container-name** : il s’agit du nom d’un serveur SAP HANA qui est correctement inscrit auprès d’un coffre Recovery Services et qui se trouve dans la même région que la base de données à restaurer. Dans ce tutoriel, nous allons restaurer la base de données sous forme de fichiers sur le serveur SAP HANA que nous avons protégé, nommé *hxehost*.
* **--rp-name** : pour une limite de restauration dans le temps, le nom du point de restauration est **DefaultRangeRecoveryPoint**

```azurecli-interactive
az backup recoveryconfig show --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --item-name saphanadatabase;hxe;hxe \
    --restore-mode RestoreAsFiles \
    --log-point-in-time 28-11-2019-09:53:00 \
    --rp-name DefaultRangeRecoveryPoint \
    --target-container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --filepath /home/saphana/restoreasfiles \
    --output json
```

La réponse à la requête ci-dessus est un objet de configuration de récupération qui ressemble à ceci :

```output
{
  "alternate_directory_paths": null,
  "container_id": "/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/saphanaResourceGroup/providers/Microsoft.RecoveryServices/vaults/SAPHANAVault/backupFabrics/Azure/protectionContainers/VMAppContainer;Compute;SAPHANA;hanamachine",
  "container_uri": "VMAppContainer;compute;saphana;hanamachine",
  "database_name": null,
  "filepath": "/home/",
  "item_type": "SAPHana",
  "item_uri": "SAPHanaDatabase;hxe;hxe",
  "log_point_in_time": "04-07-2020-09:53:00",
  "recovery_mode": "FileRecovery",
  "recovery_point_id": "DefaultRangeRecoveryPoint",
  "restore_mode": "AlternateLocation",
  "source_resource_id": "/subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/saphanaResourceGroup/providers/Microsoft.Compute/virtualMachines/hanamachine"
}
```

À présent, pour restaurer la base de données sous forme de fichiers, exécutez l’applet de commande [az restore restore-azurewl](/cli/azure/backup/restore#az-backup-restore-restore-azurewl). Pour utiliser cette commande, nous allons entrer la sortie JSON ci-dessus qui est enregistrée dans un fichier nommé *recoveryconfig.json*.

```azurecli-interactive
az backup restore restore-azurewl --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --restore-config recoveryconfig.json \
    --output json
```

La sortie se présente ainsi :

```output
{
  "eTag": null,
  "id": "/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/SAPHANARESOURCEGROUP/providers/Microsoft.RecoveryServices/vaults/SAPHANAVault/backupJobs/608e737e-c001-47ca-8c37-57d909c8a704",
  "location": null,
  "name": "608e737e-c001-47ca-8c37-57d909c8a704",
  "properties": {
    "actionsInfo": [
      "Cancellable"
    ],
    "activityId": "7ddd3c3a-c0eb-11ea-a5f8-54ee75ec272a",
    "backupManagementType": "AzureWorkload",
    "duration": "0:00:01.781847",
    "endTime": null,
    "entityFriendlyName": "HXE [hxehost]",
    "errorDetails": null,
    "extendedInfo": {
      "dynamicErrorMessage": null,
      "propertyBag": {
        "Job Type": "Restore as files"
      },
      "tasksList": [
        {
          "status": "InProgress",
          "taskId": "Transfer data from vault"
        }
      ]
    },
    "jobType": "AzureWorkloadJob",
    "operation": "Restore",
    "startTime": "2020-07-08T07:20:29.336434+00:00",
    "status": "InProgress",
    "workloadType": "SAPHanaDatabase"
  },
  "resourceGroup": "saphanaResourceGroup",
  "tags": null,
  "type": "Microsoft.RecoveryServices/vaults/backupJobs"
}
```

La réponse vous donnera le nom du travail. Vous pouvez utiliser ce nom pour effectuer le suivi de l’état du travail à l’aide de l’applet de commande [az backup job show](/cli/azure/backup/job#az-backup-job-show).

Les fichiers qui sont vidés dans le conteneur cible sont :

* Fichiers de sauvegarde de base de données
* Fichiers catalogue
* Fichiers de métadonnées JSON (pour chaque fichier de sauvegarde impliqué)

En règle générale, un chemin d’accès de partage réseau, ou le chemin d’un partage de fichiers Azure monté lorsqu’il est spécifié comme chemin de destination, permet un accès plus facile à ces fichiers par d’autres ordinateurs du même réseau ou avec le même partage de fichiers Azure monté sur ces fichiers.

>[!NOTE]
>Pour restaurer les fichiers de sauvegarde de base de données sur un partage de fichiers Azure monté sur la machine virtuelle inscrite cible, assurez-vous que le compte racine dispose des autorisations de lecture ou d’écriture sur le partage de fichiers Azure.

En fonction du type de point de restauration choisi (**Limite dans le temps** ou **Différentielle et complète**), vous verrez un ou plusieurs dossiers créés dans le chemin de destination. L’un des dossiers nommés `Data_<date and time of restore>` contient les sauvegardes complètes et différentielles, et l’autre dossier nommé `Log` contient les sauvegardes de fichiers journaux.

Déplacez ces fichiers restaurés sur le serveur SAP HANA sur lequel vous souhaitez les restaurer en tant que base de données. Effectuez ensuite ces étapes pour restaurer la base de données :

1. Définissez des autorisations sur le dossier/répertoire dans lequel les fichiers de sauvegarde sont stockés à l’aide de la commande suivante :

    ```bash
    chown -R <SID>adm:sapsys <directory>
    ```

1. Exécutez l’ensemble de commandes suivant en tant que `<SID>adm`.

    ```bash
    su - <sid>adm
    ```

1. Générez le fichier catalogue pour la restauration. Extrayez le **BackupId** du fichier de métadonnées JSON pour la sauvegarde complète et qui sera utilisé ultérieurement dans l’opération de restauration. Assurez-vous que les sauvegardes complètes et celles de fichiers journaux se trouvent dans des dossiers différents et supprimez les fichiers catalogue et les fichiers de métadonnées JSON dans ces dossiers.

    ```bash
    hdbbackupdiag --generate --dataDir <DataFileDir> --logDirs <LogFilesDir> -d <PathToPlaceCatalogFile>
    ```

    Dans la commande ci-dessus :

    * `<DataFileDir>` : le dossier qui contient les sauvegardes complètes
    * `<LogFilesDir>` : le dossier qui contient les sauvegardes de fichiers journaux
    * `<PathToPlaceCatalogFile>` : le dossier dans lequel le fichier catalogue généré doit être placé

1. Restaurez à l’aide du fichier catalogue qui vient d’être généré via HANA Studio ou exécutez la requête de restauration HDBSQL avec ce catalogue nouvellement généré. Les requêtes HDBSQL sont répertoriées ci-dessous :

    * Pour restaurer à une limite dans le temps :

        Si vous créez une base de données restaurée, exécutez la commande HDBSQL pour créer une base de données `<DatabaseName>`, puis arrêtez la base de données pour la restauration. Toutefois, si vous restaurez uniquement une base de données existante, exécutez la commande HDBSQL pour arrêter la base de données.

        Exécutez ensuite la commande suivante pour restaurer la base de données :

        ```hdbsql
        RECOVER DATABASE FOR <DatabaseName> UNTIL TIMESTAMP '<TimeStamp>' CLEAR LOG USING SOURCE '<DatabaseName@HostName>'  USING CATALOG PATH ('<PathToGeneratedCatalogInStep3>') USING LOG PATH (' <LogFileDir>') USING DATA PATH ('<DataFileDir>') USING BACKUP_ID <BackupIdFromJsonFile> CHECK ACCESS USING FILE
        ```

        * `<DatabaseName>` : nom de la nouvelle base de données ou de la base de données existante que vous souhaitez restaurer
        * `<Timestamp>` : horodatage exact de la limite de restauration dans le temps
        * `<DatabaseName@HostName>` : nom de la base de données dont la sauvegarde est utilisée pour la restauration et l’**hôte**/le nom du serveur SAP HANA sur lequel réside cette base de données. L’option `USING SOURCE <DatabaseName@HostName>` spécifie que la sauvegarde de données (utilisée pour la restauration) est celle d’une base de données avec un SID ou un nom différent de celui de la machine SAP HANA cible. Il n’est donc pas nécessaire de spécifier les restaurations effectuées sur le même serveur HANA à partir duquel la sauvegarde est effectuée.
        * `<PathToGeneratedCatalogInStep3>` : chemin du fichier catalogue généré à l’**étape 3**
        * `<DataFileDir>` : le dossier qui contient les sauvegardes complètes
        * `<LogFilesDir>` : le dossier qui contient les sauvegardes de fichiers journaux
        * `<BackupIdFromJsonFile>` : **BackupId** extrait à l’**étape 3**

    * Pour restaurer une sauvegarde complète ou différentielle particulière :

        Si vous créez une base de données restaurée, exécutez la commande HDBSQL pour créer une base de données `<DatabaseName>`, puis arrêtez la base de données pour la restauration. Toutefois, si vous restaurez uniquement une base de données existante, exécutez la commande HDBSQL pour arrêter la base de données :

        ```hdbsql
        RECOVER DATA FOR <DatabaseName> USING BACKUP_ID <BackupIdFromJsonFile> USING SOURCE '<DatabaseName@HostName>'  USING CATALOG PATH ('<PathToGeneratedCatalogInStep3>') USING DATA PATH ('<DataFileDir>')  CLEAR LOG
        ```

        * `<DatabaseName>` : nom de la nouvelle base de données ou de la base de données existante que vous souhaitez restaurer
        * `<Timestamp>` : horodatage exact de la limite de restauration dans le temps
        * `<DatabaseName@HostName>` : nom de la base de données dont la sauvegarde est utilisée pour la restauration et l’**hôte**/le nom du serveur SAP HANA sur lequel réside cette base de données. L’option `USING SOURCE <DatabaseName@HostName>` spécifie que la sauvegarde de données (utilisée pour la restauration) est celle d’une base de données avec un SID ou un nom différent de celui de la machine SAP HANA cible. Il n’est donc pas nécessaire de spécifier les restaurations effectuées sur le même serveur HANA à partir duquel la sauvegarde est effectuée.
        * `<PathToGeneratedCatalogInStep3>` : chemin du fichier catalogue généré à l’**étape 3**
        * `<DataFileDir>` : le dossier qui contient les sauvegardes complètes
        * `<LogFilesDir>` : le dossier qui contient les sauvegardes de fichiers journaux
        * `<BackupIdFromJsonFile>` : **BackupId** extrait à l’**étape 3**

## <a name="next-steps"></a>Étapes suivantes

* Pour découvrir comment gérer des bases de données SAP HANA qui sont sauvegardées à l’aide d’Azure CLI, consultez le tutoriel [Gérer une base de données SAP HANA sur une machine virtuelle Azure à l’aide de l’interface CLI](tutorial-sap-hana-backup-cli.md).

* Pour découvrir comment restaurer une base de données SAP HANA exécutée sur une machine virtuelle Azure à l’aide du portail Azure, consultez [Restaurer une base de données SAP HANA sur des machines virtuelles Azure](./sap-hana-db-restore.md).

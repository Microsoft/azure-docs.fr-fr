---
title: Gérer les sauvegardes de partage de fichiers Azure avec l'interface de ligne de commande Azure
description: Découvrez comment utiliser l'interface de ligne de commande Azure pour gérer et superviser les partages de fichiers Azure sauvegardés par Sauvegarde Azure.
ms.topic: conceptual
ms.date: 01/15/2020
ms.openlocfilehash: 5a8a785016845b836a102663a959e4b2f28696b6
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94566450"
---
# <a name="manage-azure-file-share-backups-with-the-azure-cli"></a>Gérer les sauvegardes de partage de fichiers Azure avec l'interface de ligne de commande Azure

L’interface de ligne de commande Azure permet de gérer les ressources Azure. Elle constitue un excellent outil pour générer une automatisation personnalisée afin d'utiliser des ressources Azure. Cet article explique comment effectuer les tâches pour gérer et superviser les partages de fichiers Azure sauvegardés par [Sauvegarde Azure](./backup-overview.md). Vous pouvez également suivre ces étapes avec le [portail Azure](https://portal.azure.com/).

## <a name="prerequisites"></a>Prérequis

Cet article suppose que vous disposez déjà d’un partage de fichiers Azure sauvegardé par [Sauvegarde Azure](./backup-overview.md). Si ce n'est pas le cas, consultez [Sauvegarder des partages de fichiers Azure avec l’interface de ligne de commande](backup-afs-cli.md) afin de configurer la sauvegarde pour vos partage de fichiers. Pour cet article, vous allez utiliser les ressources suivantes :
   -  **Groupe de ressources** : *azurefiles*
   -  **RecoveryServicesVault** : *azurefilesvault*
   -  **Compte de stockage** : *afsaccount*
   -  **Partage de fichiers** : *azurefiles*
  
  [!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]
   - Cet tutoriel nécessite la version 2.0.18 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="monitor-jobs"></a>Surveiller des travaux

Lorsque vous déclenchez une opération de sauvegarde ou de restauration, le service de sauvegarde crée une tâche de suivi. Pour superviser les travaux terminés ou en cours d’exécution, utilisez la cmdlet [az backup job list](/cli/azure/backup/job#az-backup-job-list). L’interface de ligne de commande vous permet également d’[interrompre un travail en cours d’exécution](/cli/azure/backup/job#az-backup-job-stop) ou d’[attendre qu’un travail se termine](/cli/azure/backup/job#az-backup-job-wait).

L’exemple suivant affiche l’état des travaux de sauvegarde pour le coffre Recovery Services *azurefilesvault* :

```azurecli-interactive
az backup job list --resource-group azurefiles --vault-name azurefilesvault
```

```output
[
  {
    "eTag": null,
    "id": "/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupJobs/d477dfb6-b292-4f24-bb43-6b14e9d06ab5",
    "location": null,
    "name": "d477dfb6-b292-4f24-bb43-6b14e9d06ab5",
    "properties": {
      "actionsInfo": null,
      "activityId": "3cef43ed-0af4-43e2-b9cb-1322c496ccb4",
      "backupManagementType": "AzureStorage",
      "duration": "0:00:29.718011",
      "endTime": "2020-01-13T08:05:29.180606+00:00",
      "entityFriendlyName": "azurefiles",
      "errorDetails": null,
      "extendedInfo": null,
      "jobType": "AzureStorageJob",
      "operation": "Backup",
      "startTime": "2020-01-13T08:04:59.462595+00:00",
      "status": "Completed",
      "storageAccountName": "afsaccount",
      "storageAccountVersion": "MicrosoftStorage"
    },
    "resourceGroup": "azurefiles",
    "tags": null,
    "type": "Microsoft.RecoveryServices/vaults/backupJobs"
  },
  {
    "eTag": null,
    "id": "/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupJobs/1b9399bf-c23c-4caa-933a-5fc2bf884519",
    "location": null,
    "name": "1b9399bf-c23c-4caa-933a-5fc2bf884519",
    "properties": {
      "actionsInfo": null,
      "activityId": "2663449c-94f1-4735-aaf9-5bb991e7e00c",
      "backupManagementType": "AzureStorage",
      "duration": "0:00:28.145216",
      "endTime": "2020-01-13T08:05:27.519826+00:00",
      "entityFriendlyName": "azurefilesresource",
      "errorDetails": null,
      "extendedInfo": null,
      "jobType": "AzureStorageJob",
      "operation": "Backup",
      "startTime": "2020-01-13T08:04:59.374610+00:00",
      "status": "Completed",
      "storageAccountName": "afsaccount",
      "storageAccountVersion": "MicrosoftStorage"
    },
    "resourceGroup": "azurefiles",
    "tags": null,
    "type": "Microsoft.RecoveryServices/vaults/backupJobs"
  }
]
```

## <a name="modify-policy"></a>Modifier la stratégie

Vous pouvez modifier une stratégie de sauvegarde pour changer la fréquence de sauvegarde ou la durée de rétention à l’aide de la cmdlet [az backup item set-policy](/cli/azure/backup/item#az-backup-item-set-policy).

Pour modifier la stratégie, définissez les paramètres suivants :

* **--container-name** : Nom du compte de stockage hébergeant le partage de fichiers. Pour récupérer le **nom** ou **nom convivial** de votre conteneur, utilisez la commande [az backup container list](/cli/azure/backup/container#az-backup-container-list).
* **--name** : correspond au nom du partage de fichiers pour lequel vous souhaitez modifier la stratégie. Pour récupérer le **nom** ou **nom convivial** de votre élément sauvegardé, utilisez la commande [az backup item list](/cli/azure/backup/item#az-backup-item-list).
* **--policy-name**: correspond au nom de la stratégie de sauvegarde que vous souhaitez définir pour votre partage de fichiers. Vous pouvez utiliser la commande [az backup policy list](/cli/azure/backup/policy#az-backup-policy-list) pour afficher toutes les stratégies de votre coffre.

L’exemple suivant définit la stratégie de sauvegarde *schedule2* pour le partage de fichiers *azurefiles* présent dans le compte de stockage *afsaccount*.

```azurecli-interactive
az backup item set-policy --policy-name schedule2 --name azurefiles --vault-name azurefilesvault --resource-group azurefiles --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --name "AzureFileShare;azurefiles" --backup-management-type azurestorage --out table
```

Vous pouvez également exécuter la commande précédente en utilisant les noms conviviaux pour le conteneur (container) et l’élément (item) en fournissant les deux paramètres supplémentaires suivants :

* **--backup-management-type**: *azurestorage*
* **--workload-type**: *azurefileshare*

```azurecli-interactive
az backup item set-policy --policy-name schedule2 --name azurefiles --vault-name azurefilesvault --resource-group azurefiles --container-name afsaccount --name azurefiles --backup-management-type azurestorage --out table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
fec6f004-0e35-407f-9928-10a163f123e5  azurefiles
```

L’attribut **name** figurant dans la sortie correspond au nom du travail créé par le service de sauvegarde pour votre opération de changement de stratégie. Pour suivre l’état de ce travail, utilisez la cmdlet [az backup job show](/cli/azure/backup/job#az-backup-job-show).

## <a name="stop-protection-on-a-file-share"></a>Arrêter la protection sur un partage de fichiers

Il existe deux façons de suspendre la protection des partages de fichiers Azure :

* Arrêter tous les travaux de sauvegarde à venir et *supprimer* tous les points de récupération
* Arrêter tous les travaux de sauvegarde à venir mais *conserver* les points de récupération

Le fait de laisser des points de récupération dans l'espace de stockage peut avoir un coût car les instantanés sous-jacents créés par Sauvegarde Azure sont conservés. Cela présente cependant l’avantage de vous permettre de restaurer ultérieurement le partage de fichiers, si vous le souhaitez. Pour plus d’informations sur les coûts de conservation des points de récupération, voir les [détails de la tarification](https://azure.microsoft.com/pricing/details/storage/files). Si vous décidez de supprimer tous les points de récupération, vous ne pouvez plus restaurer le partage de fichiers.

Pour arrêter la protection du partage de fichiers, définissez les paramètres suivants :

* **--container-name** : Nom du compte de stockage hébergeant le partage de fichiers. Pour récupérer le **nom** ou **nom convivial** de votre conteneur, utilisez la commande [az backup container list](/cli/azure/backup/container#az-backup-container-list).
* **--item-name** : correspond au nom du partage de fichiers pour lequel vous souhaitez arrêter la protection. Pour récupérer le **nom** ou **nom convivial** de votre élément sauvegardé, utilisez la commande [az backup item list](/cli/azure/backup/item#az-backup-item-list).

### <a name="stop-protection-and-retain-recovery-points"></a>Arrêter la protection et conserver les points de récupération

Pour arrêter la protection en conservant les données, utilisez la cmdlet [az backup protection disable](/cli/azure/backup/protection#az-backup-protection-disable).

L’exemple suivant arrête la protection du partage de fichiers *azurefiles*, mais conserve tous les points de récupération.

```azurecli-interactive
az backup protection disable --vault-name azurefilesvault --resource-group azurefiles --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name “AzureFileShare;azurefiles” --out table
```

Vous pouvez également exécuter la commande précédente en utilisant le nom convivial pour le conteneur (container) et l’élément (item) en fournissant les deux paramètres supplémentaires suivants :

* **--backup-management-type**: *azurestorage*
* **--workload-type**: *azurefileshare*

```azurecli-interactive
az backup protection disable --vault-name azurefilesvault --resource-group azurefiles --container-name afsaccount --item-name azurefiles --workload-type azurefileshare --backup-management-type Azurestorage --out table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
fec6f004-0e35-407f-9928-10a163f123e5  azurefiles
```

L’attribut **name** figurant dans la sortie correspond au nom du travail créé par le service de sauvegarde pour votre opération d’arrêt de la protection. Pour suivre l’état de ce travail, utilisez la cmdlet [az backup job show](/cli/azure/backup/job#az-backup-job-show).

### <a name="stop-protection-without-retaining-recovery-points"></a>Arrêter la protection sans conserver les points de récupération

Pour arrêter la protection sans conserver les points de récupération, utilisez la cmdlet [az backup protection disable](/cli/azure/backup/protection#az-backup-protection-disable) avec l’option **delete-backup-data** définie sur **true**.

L’exemple suivant arrête la protection du partage de fichiers *azurefiles* sans conserver les points de récupération.

```azurecli-interactive
az backup protection disable --vault-name azurefilesvault --resource-group azurefiles --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name “AzureFileShare;azurefiles” --delete-backup-data true --out table
```

Vous pouvez également exécuter la commande précédente en utilisant le nom convivial pour le conteneur (container) et l’élément (item) en fournissant les deux paramètres supplémentaires suivants :

* **--backup-management-type**: *azurestorage*
* **--workload-type**: *azurefileshare*

```azurecli-interactive
az backup protection disable --vault-name azurefilesvault --resource-group azurefiles --container-name afsaccount --item-name azurefiles --workload-type azurefileshare --backup-management-type Azurestorage --delete-backup-data true --out table
```

## <a name="resume-protection-on-a-file-share"></a>Reprendre la protection sur un partage de fichiers

Si vous avez arrêté la protection d’un partage de fichiers Azure mais conservé les points de récupération, vous pouvez reprendre la protection ultérieurement. Si vous ne conservez pas les points de récupération, vous ne pourrez pas reprendre la protection.

Pour reprendre la protection du partage de fichiers, définissez les paramètres suivants :

* **--container-name** : Nom du compte de stockage hébergeant le partage de fichiers. Pour récupérer le **nom** ou **nom convivial** de votre conteneur, utilisez la commande [az backup container list](/cli/azure/backup/container#az-backup-container-list).
* **--item-name** : correspond au nom du partage de fichiers pour lequel vous souhaitez reprendre la protection. Pour récupérer le **nom** ou **nom convivial** de votre élément sauvegardé, utilisez la commande [az backup item list](/cli/azure/backup/item#az-backup-item-list).
* **--policy-name**: correspond au nom de la stratégie de sauvegarde pour laquelle vous souhaitez reprendre la protection du partage de fichiers.

L’exemple suivant utilise la cmdlet [az backup protection reresume](/cli/azure/backup/protection#az-backup-protection-resume) pour reprendre la protection du partage de fichiers *azurefiles* à l’aide de la stratégie de sauvegarde *Schedule1*.

```azurecli-interactive
az backup protection resume --vault-name azurefilesvault --resource-group azurefiles --container-name "StorageContainer;Storage;AzureFiles;afsaccount” --item-name “AzureFileShare;azurefiles” --policy-name schedule2 --out table
```

Vous pouvez également exécuter la commande précédente en utilisant le nom convivial pour le conteneur (container) et l’élément (item) en fournissant les deux paramètres supplémentaires suivants :

* **--backup-management-type**: *azurestorage*
* **--workload-type**: *azurefileshare*

```azurecli-interactive
az backup protection resume --vault-name azurefilesvault --resource-group azurefiles --container-name afsaccount --item-name azurefiles --workload-type azurefileshare --backup-management-type Azurestorage --policy-name schedule2 --out table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
75115ab0-43b0-4065-8698-55022a234b7f  azurefiles
```

L’attribut **name** figurant dans la sortie correspond au nom du travail créé par le service de sauvegarde pour votre opération de reprise de la protection. Pour suivre l’état de ce travail, utilisez la cmdlet [az backup job show](/cli/azure/backup/job#az-backup-job-show).

## <a name="unregister-a-storage-account"></a>Annuler l’inscription d’un compte de stockage

Si vous souhaitez protéger vos partages de fichiers dans un compte de stockage particulier à l’aide d’un coffre Recovery Services différent, commencez par [Arrêter la protection de tous les partages de fichiers](#stop-protection-on-a-file-share) dans ce compte de stockage. Annulez ensuite l’inscription du compte du coffre Recovery Services actuellement utilisé pour la protection.

Vous devez fournir un nom de conteneur pour annuler l’inscription du compte de stockage. Pour récupérer le **nom** ou **nom convivial** de votre conteneur, utilisez la commande [az backup container list](/cli/azure/backup/container#az-backup-container-list).

L’exemple suivant annule l’inscription du compte de stockage *afsaccount* de *azurefilesvault* à l'aide de la cmdlet [az backup container unregister](/cli/azure/backup/container#az-backup-container-unregister).

```azurecli-interactive
az backup container unregister --vault-name azurefilesvault --resource-group azurefiles --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --out table
```

Vous pouvez également exécuter la cmdlet précédente en utilisant le nom convivial pour le conteneur (container), en fournissant le paramètre supplémentaire suivant :

* **--backup-management-type**: *azurestorage*

```azurecli-interactive
az backup container unregister --vault-name azurefilesvault --resource-group azurefiles --container-name afsaccount --backup-management-type azurestorage --out table
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d'informations, consultez [Résoudre les problèmes liés à la sauvegarde des partages de fichiers Azure](troubleshoot-azure-files.md).

---
title: Restaurer des partages de fichiers Azure avec l'interface de ligne de commande Azure
description: Découvrir comment utiliser l'interface de ligne de commande Azure pour restaurer des partages de fichiers Azure sauvegardés dans le coffre Recovery Services
ms.topic: conceptual
ms.date: 01/16/2020
ms.openlocfilehash: a025de7bfb9db037b2008d69be7782feabb482f3
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94562319"
---
# <a name="restore-azure-file-shares-with-the-azure-cli"></a>Restaurer des partages de fichiers Azure avec l'interface de ligne de commande Azure

L’interface de ligne de commande Azure permet de gérer les ressources Azure. Elle constitue un excellent outil pour générer une automatisation personnalisée afin d'utiliser des ressources Azure. Cet article explique comment restaurer l’intégralité d’un partage de fichiers ou des fichiers spécifiques à partir d’un point de restauration créé par [Sauvegarde Azure](./backup-overview.md) à l’aide de l'interface de ligne de commande Azure. Vous pouvez également effectuer ces étapes avec [Azure PowerShell](./backup-azure-afs-automation.md) ou dans le [portail Azure](backup-afs.md).

À la fin de ce tutoriel, vous aurez appris à effectuer les opérations ci-dessous avec l'interface de ligne de commande Azure :

* Afficher les points de restauration d’un partage de fichiers Azure sauvegardé
* Restaurer un partage de fichiers Azure complet
* restaurer des fichiers ou dossiers individuels ;

>[!NOTE]
> Sauvegarde Azure prend désormais en charge la restauration de plusieurs fichiers ou dossiers vers l’emplacement d’origine ou un autre emplacement à l’aide d’Azure CLI. Pour en savoir plus, reportez-vous à la section [Restaurer plusieurs fichiers ou dossiers à l’emplacement d’origine ou à un autre emplacement](#restore-multiple-files-or-folders-to-original-or-alternate-location) de ce document.

## <a name="prerequisites"></a>Prérequis

Cet article suppose que vous disposez déjà d’un partage de fichiers Azure sauvegardé par Sauvegarde Azure. Si ce n'est pas le cas, consultez [Sauvegarder des partages de fichiers Azure avec l’interface de ligne de commande](backup-afs-cli.md) afin de configurer la sauvegarde pour vos partages de fichiers. Pour cet article, vous allez utiliser les ressources suivantes :

| Partage de fichiers | Compte de stockage | Région | Détails |
|---|---|---|---|
| *azurefiles* | *afsaccount* | USAEst | Source d’origine sauvegardée à l’aide de Sauvegarde Azure |
| *azurefiles1* | *afaccount1* | USAEst | Source de destination utilisée pour la récupération à un autre emplacement |

Vous pouvez utiliser une structure similaire pour vos partages de fichiers afin de tester les différents types de restauration expliqués dans cet article.

[!INCLUDE [azure-cli-prepare-your-environment-h3.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

 - Ce tutoriel nécessite Azure CLI version 2.0.18 ou ultérieure. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="fetch-recovery-points-for-the-azure-file-share"></a>Extraire des points de récupération pour le partage de fichiers Azure

Utilisez l’applet de commande [az backup recoverypoint list](/cli/azure/backup/recoverypoint#az-backup-recoverypoint-list) pour lister tous les points de récupération du partage de fichiers sauvegardé.

L’exemple suivant extrait la liste des points de récupération du partage de fichiers *azurefiles* dans le compte de stockage *afsaccount*.

```azurecli-interactive
az backup recoverypoint list --vault-name azurefilesvault --resource-group azurefiles --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --backup-management-type azurestorage --item-name "AzureFileShare;azurefiles" --workload-type azurefileshare --out table
```

Vous pouvez également exécuter la cmdlet précédente en utilisant le nom convivial pour le conteneur (container) et l’élément (item) en fournissant les deux paramètres supplémentaires suivants :

* **--backup-management-type**: *azurestorage*
* **--workload-type**: *azurefileshare*

```azurecli-interactive
az backup recoverypoint list --vault-name azurefilesvault --resource-group azurefiles --container-name afsaccount --backup-management-type azurestorage --item-name azurefiles --workload-type azurefileshare --out table
```

Le jeu de résultats correspond à une liste de points de récupération avec des détails d’heure et de cohérence pour chaque point de restauration.

```output
Name                Time                        Consistency
------------------  -------------------------   --------------------
932887541532871865  2020-01-05T07:08:23+00:00   FileSystemConsistent
932885927361238054  2020-01-05T07:08:10+00:00   FileSystemConsistent
932879614553967772  2020-01-04T21:33:04+00:00   FileSystemConsistent
```

L’attribut **Name** figurant dans la sortie correspond au nom du point de récupération qui peut être utilisé comme valeur pour le paramètre **--rp-name** dans les opérations de récupération.

## <a name="full-share-recovery-by-using-the-azure-cli"></a>Récupération de partage complet avec l'interface de ligne de commande Azure

Vous pouvez utiliser cette option de restauration pour restaurer l’intégralité du partage de fichiers dans l’emplacement d’origine ou dans un autre emplacement.

Définissez les paramètres suivants pour effectuer des opérations de restauration :

* **--container-name** : nom du compte de stockage hébergeant le partage de fichiers d’origine sauvegardé. Pour récupérer le nom ou nom convivial de votre conteneur, utilisez la commande [az backup container list](/cli/azure/backup/container#az-backup-container-list).
* **--item-name** : nom du partage de fichiers d’origine sauvegardé que vous souhaitez utiliser pour l’opération de restauration. Pour récupérer le nom ou nom convivial de votre élément sauvegardé, utilisez la commande [az backup item list](/cli/azure/backup/item#az-backup-item-list).

### <a name="restore-a-full-share-to-the-original-location"></a>Restaurer l’intégralité d’un partage à l’emplacement d’origine

En cas de restauration à l’emplacement d’origine, vous n’avez pas besoin de spécifier les paramètres liés à la cible. Seul **Résoudre le conflit** doit être fourni.

L’exemple suivant utilise la cmdlet [az backup restore restore-azurefileshare](/cli/azure/backup/restore#az-backup-restore-restore-azurefileshare) avec le mode de restauration défini sur *originallocation* pour restaurer le partage de fichiers *azurefiles* à l’emplacement d’origine. Vous utilisez le point de récupération 932883129628959823 obtenu à l'étape [Extraire des points de récupération pour le partage de fichiers Azure](#fetch-recovery-points-for-the-azure-file-share) :

```azurecli-interactive
az backup restore restore-azurefileshare --vault-name azurefilesvault --resource-group azurefiles --rp-name 932887541532871865   --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name "AzureFileShare;azurefiles" --restore-mode originallocation --resolve-conflict overwrite --out table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
6a27cc23-9283-4310-9c27-dcfb81b7b4bb  azurefiles
```

L’attribut **name** figurant dans la sortie correspond au nom du travail créé par le service de sauvegarde pour votre opération de restauration. Pour suivre l’état de ce travail, utilisez la cmdlet [az backup job show](/cli/azure/backup/job#az-backup-job-show).

### <a name="restore-a-full-share-to-an-alternate-location"></a>Restaurer l’intégralité d’un partage dans un autre emplacement

Vous pouvez utiliser cette option pour restaurer un partage de fichiers dans un autre emplacement et conserver le partage de fichiers d’origine tel quel. Spécifiez les paramètres suivants pour la récupération dans un autre emplacement :

* **--target-storage-account** : Compte de stockage sur lequel le contenu sauvegardé est restauré. Le compte de stockage cible doit se trouver au même emplacement que le coffre.
* **--target-file-share** : Partage de fichiers du compte de stockage cible sur lequel le contenu sauvegardé est restauré.
* **--target-folder** : Dossier situé sous le partage de fichiers sur lequel les données sont restaurées. Si le contenu sauvegardé doit être restauré dans le dossier racine, indiquez les valeurs du dossier cible sous forme de chaîne vide.
* **--resolve-conflict** : Instruction en cas de conflit avec les données restaurées. Accepte **Remplacer** ou **Ignorer**.

L’exemple suivant utilise [az backup restore restore-azurefileshare](/cli/azure/backup/restore#az-backup-restore-restore-azurefileshare) avec un mode de restauration comme *alternatelocation* pour restaurer le partage de fichiers *azurefiles* du compte de stockage *afsaccount* sur le partage de fichiers *azurefiles1* du compte de stockage *afaccount1*.

```azurecli-interactive
az backup restore restore-azurefileshare --vault-name azurefilesvault --resource-group azurefiles --rp-name 932883129628959823 --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name "AzureFileShare;azurefiles" --restore-mode alternatelocation --target-storage-account afaccount1 --target-file-share azurefiles1 --target-folder restoredata --resolve-conflict overwrite --out table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
babeb61c-d73d-4b91-9830-b8bfa83c349a  azurefiles
```

L’attribut **name** figurant dans la sortie correspond au nom du travail créé par le service de sauvegarde pour votre opération de restauration. Pour suivre l’état de ce travail, utilisez la cmdlet [az backup job show](/cli/azure/backup/job#az-backup-job-show).

## <a name="item-level-recovery"></a>Récupération au niveau de l'élément

Vous pouvez utiliser cette option de restauration pour restaurer des fichiers ou dossiers individuels à l’emplacement d’origine ou dans un autre emplacement.

Définissez les paramètres suivants pour effectuer des opérations de restauration :

* **--container-name** : nom du compte de stockage hébergeant le partage de fichiers d’origine sauvegardé. Pour récupérer le nom ou nom convivial de votre conteneur, utilisez la commande [az backup container list](/cli/azure/backup/container#az-backup-container-list).
* **--item-name** : nom du partage de fichiers d’origine sauvegardé que vous souhaitez utiliser pour l’opération de restauration. Pour récupérer le nom ou nom convivial de votre élément sauvegardé, utilisez la commande [az backup item list](/cli/azure/backup/item#az-backup-item-list).

Spécifiez les paramètres suivants pour les éléments que vous voulez récupérer :

* **SourceFilePath** : Chemin d'accès absolu du fichier, à restaurer dans le partage de fichiers, sous forme de chaîne. Ce chemin est le même que celui utilisé dans les commandes CLI [az storage file download](/cli/azure/storage/file#az-storage-file-download) ou [az storage file show](/cli/azure/storage/file#az-storage-file-show).
* **SourceFileType** : Indiquez si un répertoire ou un fichier est sélectionné. Accepte **Répertoire** ou **Fichier**.
* **ResolveConflict** : Instruction en cas de conflit avec les données restaurées. Accepte **Remplacer** ou **Ignorer**.

### <a name="restore-individual-files-or-folders-to-the-original-location"></a>Restaurer des fichiers ou dossiers individuels à l’emplacement d’origine

Utilisez la cmdlet [az backup restore restore-azurefiles](/cli/azure/backup/restore#az-backup-restore-restore-azurefiles) avec le mode de restauration défini sur *originallocation* pour restaurer des fichiers ou dossiers spécifiques à leur emplacement d’origine.

L’exemple suivant restaure le fichier *RestoreTest.txt* à son emplacement d’origine : le partage de fichiers *azurefiles*.

```azurecli-interactive
az backup restore restore-azurefiles --vault-name azurefilesvault --resource-group azurefiles --rp-name 932881556234035474 --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name "AzureFileShare;azurefiles" --restore-mode originallocation  --source-file-type file --source-file-path "Restore/RestoreTest.txt" --resolve-conflict overwrite  --out table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
df4d9024-0dcb-4edc-bf8c-0a3d18a25319  azurefiles
```

L’attribut **name** figurant dans la sortie correspond au nom du travail créé par le service de sauvegarde pour votre opération de restauration. Pour suivre l’état de ce travail, utilisez la cmdlet [az backup job show](/cli/azure/backup/job#az-backup-job-show).

### <a name="restore-individual-files-or-folders-to-an-alternate-location"></a>Restaurer des fichiers ou dossiers individuels dans un autre emplacement

Pour restaurer des fichiers ou dossiers spécifiques dans un autre emplacement, utilisez la cmdlet [az backup restore restore-azurefiles](/cli/azure/backup/restore#az-backup-restore-restore-azurefiles) avec le mode de restauration défini sur *alternatelocation*, puis spécifiez les paramètres liés à la cible suivants :

* **--target-storage-account** : Compte de stockage sur lequel le contenu sauvegardé est restauré. Le compte de stockage cible doit se trouver au même emplacement que le coffre.
* **--target-file-share** : Partage de fichiers du compte de stockage cible sur lequel le contenu sauvegardé est restauré.
* **--target-folder** : Dossier situé sous le partage de fichiers sur lequel les données sont restaurées. Si le contenu sauvegardé doit être restauré dans un dossier racine, indiquez une chaîne vide comme valeur du dossier cible.

L’exemple suivant restaure le fichier *RestoreTest.txt* initialement présent dans le partage de fichiers *azurefiles* dans un autre emplacement : le dossier *restoredata* dans le partage de fichiers *azurefiles1* hébergé dans le compte de stockage *afaccount1*.

```azurecli-interactive
az backup restore restore-azurefiles --vault-name azurefilesvault --resource-group azurefiles --rp-name 932881556234035474 --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name "AzureFileShare;azurefiles" --restore-mode alternatelocation --target-storage-account afaccount1 --target-file-share azurefiles1 --target-folder restoredata --resolve-conflict overwrite --source-file-type file --source-file-path "Restore/RestoreTest.txt" --out table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
df4d9024-0dcb-4edc-bf8c-0a3d18a25319  azurefiles
```

L’attribut **name** figurant dans la sortie correspond au nom du travail créé par le service de sauvegarde pour votre opération de restauration. Pour suivre l’état de ce travail, utilisez la cmdlet [az backup job show](/cli/azure/backup/job#az-backup-job-show).

## <a name="restore-multiple-files-or-folders-to-original-or-alternate-location"></a>Restaurer plusieurs fichiers ou dossiers à l’emplacement d’origine ou à un autre emplacement

Pour effectuer une restauration pour plusieurs éléments, transmettez la valeur du paramètre **source-file-path** en tant que chemins **séparés par des espaces** de tous les fichiers ou dossiers que vous souhaitez restaurer.

L’exemple suivant restaure les fichiers *Restore.txt* et *AFS testing Report.docx* à leur emplacement d’origine.

```azurecli-interactive
az backup restore restore-azurefiles --vault-name azurefilesvault --resource-group azurefiles --rp-name 932889937058317910 --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name "AzureFileShare;azurefiles" --restore-mode originallocation  --source-file-type file --source-file-path "Restore Test.txt" "AFS Testing Report.docx" --resolve-conflict overwrite  --out table
```

Vous devez obtenir un résultat semblable à ce qui suit :

```output
Name                                          ResourceGroup
------------------------------------          ---------------
649b0c14-4a94-4945-995a-19e2aace0305          azurefiles
```

L’attribut **name** figurant dans la sortie correspond au nom du travail créé par le service de sauvegarde pour votre opération de restauration. Pour suivre l’état de ce travail, utilisez la cmdlet [az backup job show](/cli/azure/backup/job#az-backup-job-show).

Si vous souhaitez restaurer plusieurs éléments à un autre emplacement, utilisez la commande ci-dessus en spécifiant les paramètres liés à la cible, comme expliqué dans la section [Restaurer des fichiers ou dossiers individuels dans un autre emplacement](#restore-individual-files-or-folders-to-an-alternate-location).

## <a name="next-steps"></a>Étapes suivantes

Découvrir comment [Gérer les sauvegardes de partage de fichiers Azure avec l'interface de ligne de commande Azure](manage-afs-backup-cli.md)

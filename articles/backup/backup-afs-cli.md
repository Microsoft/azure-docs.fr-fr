---
title: Sauvegarder des partages de fichiers Azure à l'aide d'Azure CLI
description: Apprenez à utiliser l'interface de ligne de commande Azure (Azure CLI) pour sauvegarder des partages de fichiers Azure dans le coffre Recovery Services
ms.topic: conceptual
ms.date: 01/14/2020
ms.openlocfilehash: 34eea8daa6a0a8920c842178664055838b06a78a
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94565889"
---
# <a name="back-up-azure-file-shares-with-azure-cli"></a>Sauvegarder des partages de fichiers Azure à l'aide d'Azure CLI

L'interface de ligne de commande Azure (Azure CLI) fournit une expérience de ligne de commande dédiée à la gestion des ressources Azure. Elle constitue un excellent outil pour générer une automatisation personnalisée afin d'utiliser des ressources Azure. Cet article explique comment sauvegarder des partages de fichiers Azure à l'aide d'Azure CLI. Vous pouvez également effectuer ces étapes avec [Azure PowerShell](./backup-azure-afs-automation.md) ou dans le [portail Azure](backup-afs.md).

À la fin de ce tutoriel, vous aurez appris à effectuer les opérations ci-dessous avec Azure CLI :

* Créer un coffre Recovery Services
* Activer la sauvegarde des partages de fichiers Azure
* Déclencher une sauvegarde à la demande des partages de fichiers

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

 - Cet tutoriel nécessite la version 2.0.18 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-recovery-services-vault"></a>Créer un coffre Recovery Services

Un coffre Recovery Services est une entité qui vous offre une vue d'ensemble de tous les éléments de sauvegarde ainsi qu'une capacité de gestion de ceux-ci. Lorsque le travail de sauvegarde d’une ressource protégée s’exécute, il crée un point de récupération à l’intérieur du coffre Recovery Services. Vous pouvez ensuite utiliser un de ces points de récupération pour restaurer des données à un moment donné dans le temps.

Pour créer un coffre Recovery Services, procédez comme suit :

1. Un coffre est placé dans un groupe de ressources. Si vous n'avez pas de groupe de ressources, créez-en un à l'aide de la commande [az group create](/cli/azure/group#az-group-create). Dans ce tutoriel, nous allons créer le nouveau groupe de ressources *azurefiles* dans la région USA Est.

    ```azurecli-interactive
    az group create --name AzureFiles --location eastus --output table
    ```

    ```output
    Location    Name
    ----------  ----------
    eastus      AzureFiles
    ```

1. Utilisez la cmdlet [az backup vault create](/cli/azure/backup/vault#az-backup-vault-create) pour créer le coffre. Spécifiez pour le coffre le même emplacement que pour le groupe de ressources.

    L'exemple suivant explique comment créer un coffre Recovery Services nommé *azurefilesvault* dans la région USA Est.

    ```azurecli-interactive
    az backup vault create --resource-group azurefiles --name azurefilesvault --location eastus --output table
    ```

    ```output
    Location    Name                ResourceGroup
    ----------  ----------------    ---------------
    eastus      azurefilesvault     azurefiles
    ```

## <a name="enable-backup-for-azure-file-shares"></a>Activer la sauvegarde des partages de fichiers Azure

Cette section part du principe que vous disposez déjà d'un partage de fichiers Azure pour lequel vous souhaitez configurer la sauvegarde. Si vous n'avez pas de partage de fichiers Azure, créez-en un à l'aide de la commande [az storage share create](/cli/azure/storage/share#az-storage-share-create).

Pour activer la sauvegarde de partages de fichiers, vous devez créer une stratégie de protection qui détermine quand exécuter un travail de sauvegarde et combien de temps les points de récupération doivent être stockés. Vous pouvez créer une stratégie de sauvegarde à l'aide de la cmdlet [az backup policy create](/cli/azure/backup/policy#az-backup-policy-create).

L'exemple suivant utilise la cmdlet [az backup protection enable-for-azurefileshare](/cli/azure/backup/protection#az-backup-protection-enable-for-azurefileshare) pour activer la sauvegarde du partage de fichiers *azurefiles* sur le compte de stockage *afsaccount* à l'aide de la stratégie de sauvegarde *schedule 1* :

```azurecli-interactive
az backup protection enable-for-azurefileshare --vault-name azurefilesvault --resource-group  azurefiles --policy-name schedule1 --storage-account afsaccount --azure-file-share azurefiles  --output table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
0caa93f4-460b-4328-ac1d-8293521dd928  azurefiles
```

L’attribut **Name** figurant dans la sortie correspond au nom du travail créé par le service de sauvegarde pour votre opération d’**activation de la sauvegarde**. Pour suivre l'état de ce travail, utilisez la cmdlet [az backup job show](/cli/azure/backup/job#az-backup-job-show).

## <a name="trigger-an-on-demand-backup-for-file-share"></a>Déclencher une sauvegarde à la demande des partages de fichiers

Si vous souhaitez déclencher une sauvegarde à la demande de votre partage de fichiers au lieu d'attendre que la stratégie de sauvegarde exécute le travail à l'heure planifiée, utilisez la cmdlet [az backup protection backup-now](/cli/azure/backup/protection#az-backup-protection-backup-now).

Pour déclencher une sauvegarde à la demande, vous devez définir les paramètres suivants :

* **--container-name** est le nom du compte de stockage hébergeant le partage de fichiers. Pour récupérer le **nom** ou **nom convivial** de votre conteneur, utilisez la commande [az backup container list](/cli/azure/backup/container#az-backup-container-list).
* **--item-name** est le nom du partage de fichiers pour lequel vous souhaitez déclencher une sauvegarde à la demande. Pour récupérer le **nom** ou **nom convivial** de votre élément sauvegardé, utilisez la commande [az backup item list](/cli/azure/backup/item#az-backup-item-list).
* **--retain-until** permet de spécifier la date jusqu'à laquelle vous souhaitez conserver le point de récupération. La valeur doit être définie au format UTC (jj-mm-aaaa).

L'exemple suivant déclenche une sauvegarde à la demande du partage de fichiers *azurefiles* sur le compte de stockage *afsaccount* avec conservation jusqu'au *20-01-2020*.

```azurecli-interactive
az backup protection backup-now --vault-name azurefilesvault --resource-group azurefiles --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name "AzureFileShare;azurefiles" --retain-until 20-01-2020 --output table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
9f026b4f-295b-4fb8-aae0-4f058124cb12  azurefiles
```

L’attribut **Name** figurant dans la sortie correspond au nom du travail créé par le service de sauvegarde pour votre opération de « sauvegarde à la demande ». Pour suivre l'état d'un travail, utilisez la cmdlet [az backup job show](/cli/azure/backup/job#az-backup-job-show).

## <a name="next-steps"></a>Étapes suivantes

* Apprenez à [Restaurer des partages de fichiers Azure à l'aide de l'interface CLI](restore-afs-cli.md)
* Apprenez à [Gérer les sauvegardes de partages de fichiers Azure à l'aide de l'interface CLI](manage-afs-backup-cli.md)

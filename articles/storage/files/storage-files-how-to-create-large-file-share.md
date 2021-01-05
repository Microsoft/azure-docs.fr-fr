---
title: Activer et créer des partages de fichiers volumineux - Azure Files
description: Dans cet article, vous allez apprendre à activer et à créer des partages de fichiers volumineux.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 05/29/2020
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 5acb8e347046780c84f0aa324b997abb5e2aa840
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94629221"
---
# <a name="enable-and-create-large-file-shares"></a>Activer et créer des partages de fichiers volumineux

Lorsque vous activez des partages de fichiers volumineux sur votre compte de stockage, vos partages de fichiers peuvent être mis à l’échelle jusqu’à 100 Tio, tout en renforçant les IOPS et les limites de débit pour les partages standard. Vous pouvez également activer cette mise à l’échelle sur vos comptes de stockage existants pour vos partages de fichiers existants. Consultez [Partage de fichiers et objectifs de mise à l’échelle des fichiers](storage-files-scale-targets.md#azure-files-scale-targets) pour obtenir des détails. 

## <a name="prerequisites"></a>Prérequis

- Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
- Si vous envisagez d’utiliser Azure CLI, [installez-en la dernière version](/cli/azure/install-azure-cli?view=azure-cli-latest).
- Si vous envisagez d’utiliser le module Azure PowerShell, [installez-en la dernière version](/powershell/azure/install-az-ps?view=azps-4.6.0).

## <a name="restrictions"></a>Restrictions

Pour le moment, vous ne pouvez utiliser qu’un stockage localement redondant (LRS) ou un stockage redondant interzone (ZRS) sur des comptes activés pour le partage de fichiers volumineux. Vous ne pouvez pas utiliser un stockage géoredondant interzone (GZRS), un stockage géoredondant (GRS), un stockage géoredondant avec accès en lecture (RA-GRS) ou un stockage géoredondant interzone avec accès en lecture (RA-GZRS).

L’activation de partages de fichiers volumineux sur un compte est un processus irréversible. Une fois que vous l’avez activé, vous ne pouvez plus convertir votre compte en GZRS, GRS, RA-GRS ou RA-GZRS.

## <a name="create-a-new-storage-account"></a>Création d’un nouveau compte de stockage

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans le portail Azure, sélectionnez **Tous les services**. 
1. Dans la liste des ressources, entrez **Comptes de stockage**. Lorsque vous écrivez, au fur et à mesure de votre saisie, la liste est filtrée. Sélectionnez **Comptes de stockage**.
1. Dans la fenêtre **Comptes de stockage** qui apparaît, sélectionnez **Ajouter**.
1. Sélectionnez l’abonnement que vous allez utiliser pour créer le compte de stockage.
1. Sous le champ **Groupe de ressources**, sélectionnez **Créer**. Entrez le nom de votre nouveau groupe de ressources.

    ![Capture d’écran montrant comment créer un groupe de ressources sur le portail](media/storage-files-how-to-create-large-file-share/create-large-file-share.png)

1. Ensuite, entrez un nom pour votre compte de stockage. Le nom doit être unique dans tout Azure. Le nom doit comporter entre 3 et 24 caractères, et être constitué seulement de chiffres et de lettres minuscules.
1. Sélectionnez l’emplacement de votre compte de stockage.
1. Définissez la réplication sur **Stockage localement redondant** ou sur **Stockage redondant interzone**.
1. Laissez ces champs définis sur leur valeur par défaut :

   |Champ  |Valeur  |
   |---------|---------|
   |Modèle de déploiement     |Gestionnaire de ressources         |
   |Performances     |Standard         |
   |Type de compte     |StorageV2 (usage général v2)         |
   |Niveau d’accès     |À chaud         |

1. Sélectionnez **Avancé**, puis activez la case d’option **Activé** à droite **Partages de fichiers volumineux**.
1. Cliquez sur **Vérifier + créer** pour passer en revue vos paramètres de compte de stockage et créer le compte.

    ![Capture d’écran avec la case d’option « Activé » sur un nouveau compte de stockage dans le portail Azure](media/storage-files-how-to-create-large-file-share/large-file-shares-advanced-enable.png)

1. Sélectionnez **Create** (Créer).

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Tout d’abord, [installez la dernière version d’Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest) pour pouvoir activer les partages de fichiers volumineux.

Pour créer un compte de stockage avec des partages de fichiers volumineux activés, utilisez la commande suivante. Remplacez `<yourStorageAccountName>`, `<yourResourceGroup>` et `<yourDesiredRegion>` par vos informations.

```azurecli-interactive
## This command creates a large file share–enabled account. It will not support GZRS, GRS, RA-GRS, or RA-GZRS.
az storage account create --name <yourStorageAccountName> -g <yourResourceGroup> -l <yourDesiredRegion> --sku Standard_LRS --kind StorageV2 --enable-large-file-share
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Tout d’abord, [installez la dernière version de PowerShell](/powershell/azure/install-az-ps?view=azps-3.0.0) pour pouvoir activer les partages de fichiers volumineux.

Pour créer un compte de stockage avec des partages de fichiers volumineux activés, utilisez la commande suivante. Remplacez `<yourStorageAccountName>`, `<yourResourceGroup>` et `<yourDesiredRegion>` par vos informations.

```powershell
## This command creates a large file share–enabled account. It will not support GZRS, GRS, RA-GRS, or RA-GZRS.
New-AzStorageAccount -ResourceGroupName <yourResourceGroup> -Name <yourStorageAccountName> -Location <yourDesiredRegion> -SkuName Standard_LRS -EnableLargeFileShare;
```
---

## <a name="enable-large-files-shares-on-an-existing-account"></a>Activer les partages de fichiers volumineux sur un compte existant

Vous pouvez aussi activer les partages de fichiers volumineux sur vos comptes existants. Si vous activez les partages de fichiers volumineux, vous ne pourrez plus convertir votre compte en GZRS, GRS, RA-GRS ou RA-GZRS. L’activation des partages de fichiers volumineux est irréversible sur ce compte de stockage.

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Ouvrez le [portail Azure](https://portal.azure.com) et accédez au compte de stockage où vous voulez activer les partages de fichiers volumineux.
1. Ouvrez le compte de stockage et sélectionnez **Configuration**.
1. Sélectionnez **Activé** pour **Partages de fichiers volumineux**, puis sélectionnez **Enregistrer**.
1. Sélectionnez **Vue d’ensemble** puis **Actualiser**.

![Sélection de la case d’option « Activé » sur un compte de stockage existant dans le portail Azure](media/storage-files-how-to-create-large-file-share/enable-large-file-shares-on-existing.png)

Vous avez maintenant activé les partages de fichiers volumineux sur votre compte de stockage. Ensuite, vous devez [mettre à jour le quota du partage existant](#expand-existing-file-shares) pour tirer parti de l’augmentation de la capacité et de l’échelle.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour activer les partages de fichiers volumineux sur votre compte existant, utilisez la commande suivante. Remplacez `<yourStorageAccountName>` et `<yourResourceGroup>` par vos informations.

```azurecli-interactive
az storage account update --name <yourStorageAccountName> -g <yourResourceGroup> --enable-large-file-share
```

Vous avez maintenant activé les partages de fichiers volumineux sur votre compte de stockage. Ensuite, vous devez [mettre à jour le quota du partage existant](#expand-existing-file-shares) pour tirer parti de l’augmentation de la capacité et de l’échelle.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour activer les partages de fichiers volumineux sur votre compte existant, utilisez la commande suivante. Remplacez `<yourStorageAccountName>` et `<yourResourceGroup>` par vos informations.

```powershell
Set-AzStorageAccount -ResourceGroupName <yourResourceGroup> -Name <yourStorageAccountName> -EnableLargeFileShare
```

Vous avez maintenant activé les partages de fichiers volumineux sur votre compte de stockage. Ensuite, vous devez [mettre à jour le quota du partage existant](#expand-existing-file-shares) pour tirer parti de l’augmentation de la capacité et de l’échelle.

---

## <a name="create-a-large-file-share"></a>Créer un partage de fichiers volumineux

Une fois que vous avez activé les partages de fichiers volumineux sur votre compte de stockage, vous pouvez créer des partages de fichiers dans ce compte avec des quotas plus élevés. 

# <a name="portal"></a>[Portail](#tab/azure-portal)

La création d’un partage de fichiers volumineux est presque identique à la création d’un partage de fichiers standard. La principale différence est que vous pouvez définir un quota allant jusqu’à 100 Tio.

1. À partir de votre compte de stockage, sélectionnez **Partages de fichiers**.
1. Sélectionner **+Partage de fichiers**.
1. Entrez un nom pour votre partage de fichiers. Vous pouvez également définir la taille de quota de votre choix, jusqu’à 100 Tio. Sélectionnez ensuite **Créer**. 

![Interface utilisateur du portail Azure montrant les zones Nom et Quota](media/storage-files-how-to-create-large-file-share/large-file-shares-create-share.png)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour créer un partage de fichiers volumineux, utilisez la commande suivante. Remplacez `<yourStorageAccountName>`, `<yourStorageAccountKey>` et `<yourFileShareName>` par vos informations.

```azurecli-interactive
az storage share create --account-name <yourStorageAccountName> --account-key <yourStorageAccountKey> --name <yourFileShareName>
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour créer un partage de fichiers volumineux, utilisez la commande suivante. Remplacez `<YourStorageAccountName>`, `<YourStorageAccountKey>` et `<YourStorageAccountFileShareName>` par vos informations.

```powershell
##Config
$storageAccountName = "<YourStorageAccountName>"
$storageAccountKey = "<YourStorageAccountKey>"
$shareName="<YourStorageAccountFileShareName>"
$ctx = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
New-AzStorageShare -Name $shareName -Context $ctx
```
---

## <a name="expand-existing-file-shares"></a>Étendre des partages de fichiers existants

Une fois que vous avez activé les partages de fichiers volumineux sur votre compte de stockage, vous pouvez aussi étendre les partages de fichiers existants dans ce compte avec un quota plus élevé. 

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. À partir de votre compte de stockage, sélectionnez **Partages de fichiers**.
1. Cliquez avec le bouton droit sur votre partage de fichiers, puis sélectionnez **Quota**.
1. Entrez la nouvelle taille souhaitée, puis sélectionnez **OK**.

![Interface utilisateur du portail Azure avec le quota des partages de fichiers existants](media/storage-files-how-to-create-large-file-share/update-large-file-share-quota.png)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour définir le quota sur la taille maximale, utilisez la commande suivante. Remplacez `<yourStorageAccountName>`, `<yourStorageAccountKey>` et `<yourFileShareName>` par vos informations.

```azurecli-interactive
az storage share update --account-name <yourStorageAccountName> --account-key <yourStorageAccountKey> --name <yourFileShareName> --quota 102400
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour définir le quota sur la taille maximale, utilisez la commande suivante. Remplacez `<YourStorageAccountName>`, `<YourStorageAccountKey>` et `<YourStorageAccountFileShareName>` par vos informations.

```powershell
##Config
$storageAccountName = "<YourStorageAccountName>"
$storageAccountKey = "<YourStorageAccountKey>"
$shareName="<YourStorageAccountFileShareName>"
$ctx = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
# update quota
Set-AzStorageShareQuota -ShareName $shareName -Context $ctx -Quota 102400
```
---

## <a name="next-steps"></a>Étapes suivantes

* [Connecter et monter un partage de fichiers sur Windows](storage-how-to-use-files-windows.md)
* [Connecter et monter un partage de fichiers sur Linux](storage-how-to-use-files-linux.md)
* [Connecter et monter un partage de fichiers sur macOS](storage-how-to-use-files-mac.md)
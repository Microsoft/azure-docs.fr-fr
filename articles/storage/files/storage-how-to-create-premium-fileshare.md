---
title: Créer un partage de fichiers Azure premium
description: Découvrez comment créer un partage de fichiers Azure Premium à l’aide du portail Azure, du module Azure PowerShell ou de l’interface de ligne de commande Azure.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 08/26/2020
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 1ec8e4a945f8b8277d05c11bf3673d2e4ab15f9a
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94626790"
---
# <a name="how-to-create-an-azure-premium-file-share"></a>Guide pratique pour créer un partage de fichiers Premium Azure

Les partages de fichiers premium sont proposés sur les supports de stockage SSD et sont utiles pour les charges de travail gourmandes en E/S, notamment l’hébergement de bases de données et le calcul haute performance (HPC). Les partages de fichiers premium sont hébergés dans un type de compte de stockage spécial, appelé compte FileStorage. Les partages de fichiers premium sont conçus pour les applications d’entreprise évolutives et à hautes performances, en fournissant des partages à latence faible et homogène, nombre élevé d’IOPS et débit élevé.

Cet article vous montre comment créer ce type de compte en utilisant le [portail Azure](https://portal.azure.com/), le module Azure PowerShell et l’interface de ligne de commande Azure.

## <a name="prerequisites"></a>Prérequis

- Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
- Si vous envisagez d’utiliser Azure CLI, [installez-en la dernière version](/cli/azure/install-azure-cli?view=azure-cli-latest).
- Si vous envisagez d’utiliser le module Azure PowerShell, [installez-en la dernière version](/powershell/azure/install-az-ps?view=azps-4.6.0).

## <a name="create-a-filestorage-storage-account"></a>Créer un compte de stockage FileStorage

Chaque compte de stockage doit appartenir à un groupe de ressources Azure. Un groupe de ressources est un conteneur logique servant à grouper vos services Azure. Lorsque vous créez un compte de stockage, vous avez le choix entre créer un groupe de ressources ou utiliser un groupe de ressources existant. Les partages de fichiers Premium nécessitent un compte FileStorage.

# <a name="portal"></a>[Portail](#tab/azure-portal)

### <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com/).

À présent, vous êtes prêt à créer votre compte de stockage.

1. Dans le portail Azure, sélectionnez **Comptes de stockage** dans le menu de gauche.

    ![Sur la page principale du portail Azure, sélectionnez le compte de stockage](media/storage-how-to-create-premium-fileshare/azure-portal-storage-accounts.png)

1. Sur la fenêtre **Comptes de stockage**, sélectionnez **Ajouter**.
1. Sélectionnez l’abonnement dans lequel créer le compte de stockage.
1. Sous le champ **Groupe de ressources**, sélectionnez **Créer**. Entrez un nom pour votre nouveau groupe de ressources, comme indiqué dans l’image suivante.

1. Ensuite, entrez un nom pour votre compte de stockage. Le nom choisi doit être unique dans tout Azure. Le nom doit aussi contenir entre 3 et 24 caractères, et uniquement des lettres minuscules et des chiffres.
1. Sélectionnez l’emplacement de votre compte de stockage ou utilisez l’emplacement par défaut.
1. Pour **Performances**, sélectionnez **Premium**.

    Vous devez sélectionner **Premium** pour que **FileStorage** soit une option disponible dans la liste déroulante **Type de compte**.

1. Sélectionnez **Type de compte** et choisissez **FileStorage**.
1. Laissez **Réplication** défini sur sa valeur par défaut, **Stockage localement redondant (LRS)** .

    ![Création d'un compte de stockage pour un partage de fichiers premium](media/storage-how-to-create-premium-fileshare/create-filestorage-account.png)

1. Cliquez sur **Vérifier + créer** pour passer en revue vos paramètres de compte de stockage et créer le compte.
1. Sélectionnez **Create** (Créer).

Une fois que votre ressource de compte de stockage a été créée, accédez-y.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

### <a name="sign-in-to-azure"></a>Connexion à Azure

Utilisez la commande `Connect-AzAccount` et suivez les instructions à l’écran pour l’authentification.

```powershell
Connect-AzAccount
```

### <a name="create-a-resource-group"></a>Créer un groupe de ressources

Pour créer un groupe de ressources avec PowerShell, utilisez la commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) :

```powershell
# put resource group in a variable so you can use the same group name going forward,
# without hardcoding it repeatedly
$resourceGroup = "storage-how-to-resource-group"
$location = "westus2"
New-AzResourceGroup -Name $resourceGroup -Location $location
```

### <a name="create-a-filestorage-storage-account"></a>Créer un compte de stockage FileStorage

Pour créer un compte de stockage FileStorage à partir de PowerShell, utilisez la commande [New-AzStorageAccount](/powershell/module/az.storage/New-azStorageAccount) :

```powershell
$storageAcct = New-AzStorageAccount -ResourceGroupName $resourceGroup -Name "fileshowto" -SkuName "Premium_LRS" -Location "westus2" -Kind "FileStorage"
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour démarrer Azure Cloud Shell, connectez-vous au [portail Azure](https://portal.azure.com).

Si vous souhaitez vous connecter à votre installation locale de l’interface CLI, vérifiez que vous avez la version la plus récente, puis connectez-vous :

```azurecli
az login
```

### <a name="create-a-resource-group"></a>Créer un groupe de ressources

Pour créer un groupe de ressources avec Azure CLI, utilisez la commande [az group create](/cli/azure/group).

```azurecli-interactive
az group create `
    --name files-howto-resource-group `
    --location westus2
```

### <a name="create-a-filestorage-storage-account"></a>Créer un compte de stockage FileStorage

Pour créer un compte de stockage FileStorage à partir d’Azure CLI, utilisez la commande [az storage account create](/cli/azure/storage/account).

```azurecli-interactive
az storage account create `
    --name fileshowto `
    --resource-group files-howto-resource-group `
    --location westus `
    --sku Premium_LRS `
    --kind FileStorage
```

### <a name="get-the-storage-account-key"></a>Obtenir la clé du compte de stockage

Les clés de compte de stockage contrôlent l’accès aux ressources dans un compte de stockage. Dans cet article, nous utilisons la clé afin de créer un partage de fichiers premium. Les clés sont automatiquement créées en même temps que le compte de stockage. Vous pouvez récupérer les clés de votre compte de stockage avec la commande [az storage account keys list](/cli/azure/storage/account/keys) :

```azurecli-interactive
STORAGEKEY=$(az storage account keys list \
    --resource-group "myResourceGroup" \
    --account-name $STORAGEACCT \
    --query "[0].value" | tr -d '"')
```
---

## <a name="create-a-premium-file-share"></a>Créer un partage de fichiers premium

Le compte FileStorage étant créé, vous pouvez créer un partage de fichiers Premium dans ce compte de stockage.

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Dans le menu de gauche du compte de stockage, faites défiler jusqu’à la section **Service de fichiers**, puis sélectionnez **Fichiers**.
1. Sélectionnez **Partage de fichiers** pour créer un partage de fichiers premium.
1. Saisissez un nom et le quota souhaité pour votre partage de fichiers, puis sélectionnez **Créer**.

> [!NOTE]
> Les tailles de partage provisionnées sont spécifiées par le quota de partage. Les partages de fichiers sont facturés selon la taille provisionnée. Pour plus d’informations, consultez la [page relative aux prix appliqués](https://azure.microsoft.com/pricing/details/storage/files/).

   ![Créer un partage de fichiers premium](media/storage-how-to-create-premium-fileshare/create-premium-file-share.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour créer un partage de fichiers Premium avec le module Azure PowerShell, utilisez l’applet de commande [New-AzStorageShare](/powershell/module/az.storage/New-AzStorageShare).

> [!NOTE]
> Les tailles de partage provisionnées sont spécifiées par le quota de partage. Les partages de fichiers sont facturés selon la taille provisionnée. Pour plus d’informations, consultez la [page relative aux prix appliqués](https://azure.microsoft.com/pricing/details/storage/files/).

```powershell
New-AzStorageShare `
   -Name myshare `
   -EnabledProtocol SMB `
   -Context $storageAcct.Context
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour créer un partage de fichiers Premium avec l’interface de ligne de commande Azure, utilisez la commande [az storage share create](/cli/azure/storage/share).

> [!NOTE]
> Les tailles de partage provisionnées sont spécifiées par le quota de partage. Les partages de fichiers sont facturés selon la taille provisionnée. Pour plus d’informations, consultez la [page relative aux prix appliqués](https://azure.microsoft.com/pricing/details/storage/files/).

```azurecli-interactive
az storage share create \
    --account-name $STORAGEACCT \
    --account-key $STORAGEKEY \
    --enabled-protocol SMB \
    --name "myshare" 
```
---

## <a name="clean-up-resources"></a>Nettoyer les ressources

# <a name="portal"></a>[Portail](#tab/azure-portal)

Si vous souhaitez nettoyer les ressources créées dans cet article, supprimez le groupe de ressources. La suppression du groupe de ressources efface également le compte de stockage associé et d’autres ressources liées au groupe de ressources.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Si vous souhaitez nettoyer les ressources créées dans cet article, supprimez le groupe de ressources. La suppression du groupe de ressources efface également le compte de stockage associé et d’autres ressources liées au groupe de ressources.

Pour supprimer le groupe de ressources et les ressources associées, y compris le nouveau compte de stockage, utilisez la commande [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) : 

```powershell
Remove-AzResourceGroup -Name $resourceGroup
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Si vous souhaitez nettoyer les ressources créées dans cet article, supprimez le groupe de ressources. La suppression du groupe de ressources efface également le compte de stockage associé et d’autres ressources liées au groupe de ressources.

Pour supprimer le groupe de ressources et les ressources associées, y compris le nouveau compte de stockage, utilisez la commande [az group delete](/cli/azure/group).

```azurecli-interactive
az group delete --name myResourceGroup
```
---

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez créé un partage de fichiers premium. Pour en savoir plus sur les performances que ce compte offre, allez à la section sur le niveau de performances du guide de planification.

> [!div class="nextstepaction"]
> [Niveaux de partage de fichiers](storage-files-planning.md#storage-tiers)
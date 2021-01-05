---
title: Créer un partage NFS - Azure Files
description: Découvrez comment créer un partage de fichiers Azure qui peut être monté à l’aide du protocole NFS.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 12/04/2020
ms.author: rogarana
ms.subservice: files
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: 3cf22ee22c35b850aff33290a59a7043bb57c984
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96620940"
---
# <a name="how-to-create-an-nfs-share"></a>Comment créer un partage NFS

Les partages de fichiers Azure sont des partages de fichiers entièrement gérés qui résident dans le cloud. Ils sont accessibles via le protocole SMB (Server Message Block) ou le protocole NFS (Network File System). Cet article explique comment créer un partage de fichiers qui utilise le protocole NFS. Pour plus d’informations sur les deux protocoles, voir [Protocoles de partage de fichiers Azure](storage-files-compare-protocols.md).

## <a name="limitations"></a>Limites

[!INCLUDE [files-nfs-limitations](../../../includes/files-nfs-limitations.md)]

### <a name="regional-availability"></a>Disponibilité régionale

[!INCLUDE [files-nfs-regional-availability](../../../includes/files-nfs-regional-availability.md)]

## <a name="prerequisites"></a>Prérequis

- Créez un [compte FileStorage](storage-how-to-create-premium-fileshare.md).

    > [!IMPORTANT]
    > Les partages NFS ne sont accessibles qu’à partir de réseaux approuvés. Les connexions à votre partage NFS doivent provenir de l’une des sources suivantes :

    - [Créez un point de terminaison privé](storage-files-networking-endpoints.md#create-a-private-endpoint) (recommandé) ou [restreignez l’accès à votre point de terminaison public](storage-files-networking-endpoints.md#restrict-public-endpoint-access).
    - [Configurer un VPN point à site (P2S) sous Linux pour une utilisation avec Azure Files](storage-files-configure-p2s-vpn-linux.md).
    - [Configurer un VPN site à site pour une utilisation avec Azure Files](storage-files-configure-s2s-vpn.md).
    - Configurez [ExpressRoute](../../expressroute/expressroute-introduction.md).
- Si vous envisagez d’utiliser Azure CLI, [installez-en la dernière version](/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="register-the-nfs-41-protocol"></a>Inscrire le protocole NFS 4.1

Si vous utilisez le module Azure PowerShell ou Azure CLI, inscrivez votre fonctionnalité à l’aide des commandes suivantes :

### <a name="powershell"></a>PowerShell

```azurepowershell
Connect-AzAccount
$context = Get-AzSubscription -SubscriptionId <yourSubscriptionIDHere>
Set-AzContext $context
Register-AzProviderFeature -FeatureName AllowNfsFileShares -ProviderNamespace Microsoft.Storage
Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
```

### <a name="azure-cli"></a>Azure CLI

```azurecli
az login
az feature register --name AllowNfsFileShares \
                    --namespace Microsoft.Storage \
                    --subscription <yourSubscriptionIDHere>
az provider register --namespace Microsoft.Storage
```

## <a name="verify-feature-registration"></a>Vérifier l’inscription des fonctionnalités

L’approbation d’inscription peut prendre jusqu’à une heure. Pour vérifier que l’inscription est terminée, utilisez les commandes suivantes :

### <a name="powershell"></a>PowerShell

```azurepowershell
Get-AzProviderFeature -ProviderNamespace Microsoft.Storage -FeatureName AllowNfsFileShares
```

### <a name="azure-cli"></a>Azure CLI

```azurecli
az feature show --name AllowNfsFileShares --namespace Microsoft.Storage --subscription <yourSubscriptionIDHere>
```

## <a name="verify-storage-account-kind"></a>Vérifier le type de compte de stockage

Actuellement, seuls les comptes FileStorage peuvent créer des partages NFS. 

# <a name="portal"></a>[Portail](#tab/azure-portal)

Pour vérifier le type de compte de stockage dont vous disposez, accédez à celui-ci dans le portail Azure. Ensuite, à partir de votre compte de stockage, sélectionnez **Propriétés**. Dans le panneau Propriétés, examinez la valeur sous **Type de compte** : la valeur doit être **FileStorage**.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Pour vérifier que vous avez un compte FileStorage, vous pouvez utiliser la commande suivante :

```azurepowershell
$accountKind=Get-AzStorageAccount -ResourceGroupName "yourResourceGroup" -Name "yourStorageAccountName"
$accountKind.Kind
```

La sortie doit être **FileStorage**. Si ce n’est pas le cas, le type de votre compte de stockage est incorrect. Pour créer un compte **FileStorage**, consultez [Guide pratique pour créer un partage de fichiers Premium Azure](storage-how-to-create-premium-fileshare.md).

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
Pour vérifier que vous avez un compte FileStorage, vous pouvez utiliser la commande suivante :

```azurecli
az storage account show -g yourResourceGroup -n yourStorageAccountName
```

La sortie doit contenir **"kind": "FileStorage"** . Si ce n’est pas le cas, le type de votre compte de stockage est incorrect. Pour créer un compte **FileStorage**, consultez [Guide pratique pour créer un partage de fichiers Premium Azure](storage-how-to-create-premium-fileshare.md).

---
## <a name="create-an-nfs-share"></a>Créer un partage NFS

# <a name="portal"></a>[Portail](#tab/azure-portal)

Maintenant que vous avez créé un compte FileStorage et configuré la mise en réseau, vous pouvez créer un partage de fichiers NFS. Le processus est similaire à la création d’un partage SMB ; vous sélectionnez simplement **NFS** au lieu de **SMB** lors de la création du partage.

1. Accédez à votre compte de stockage et sélectionnez **Partages de fichiers**.
1. Sélectionnez **+ Partage de fichiers** pour créer un partage de fichiers.
1. Nommez votre partage de fichiers, puis sélectionnez une capacité approvisionnée.
1. Pour **Protocole** sélectionnez **NFS (préversion)** .
1. Pour **Squash racine** effectuez une sélection.

    - Squash racine (par défaut) : l’accès pour le superutilisateur distant (racine) est mappé à l’UID (65534) et au GID (65534).
    - Pas de squash racine : le superutilisateur distant (racine) reçoit l’accès en tant que racine.
    - Tous les types squash : tous les accès utilisateur sont mappés à l’UID (65534) et au GID (65534).
    
1. Sélectionnez **Create** (Créer).

    :::image type="content" source="media/storage-files-how-to-create-mount-nfs-shares/create-nfs-file-share.png" alt-text="Capture d’écran du panneau de création de partage de fichiers":::

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Vérifiez que le .NET Framework est installé. Consultez [Télécharger .NET Framework](https://dotnet.microsoft.com/download/dotnet-framework).
 
1. Vérifiez que la version de PowerShell installée est `5.1` ou une version ultérieure à l’aide de la commande suivante.    

   ```powershell
   echo $PSVersionTable.PSVersion.ToString() 
   ```
    
   Pour mettre à niveau votre version de PowerShell, consultez [Mise à niveau des instances Windows PowerShell existantes](/powershell/scripting/install/installing-windows-powershell?view=powershell-6#upgrading-existing-windows-powershell).
    
1. Installez la dernière version du module PowershellGet.

   ```powershell
   install-Module PowerShellGet –Repository PSGallery –Force  
   ```

1. Fermez, puis rouvrez la console PowerShell.

1. Installez le module **Az.Storage** en préversion **2.5.2-preview**.

   ```powershell
   Install-Module Az.Storage -Repository PsGallery -RequiredVersion 2.5.2-preview -AllowClobber -AllowPrerelease -Force  
   ```

   Pour plus d’informations sur l’installation des modules PowerShell, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps?view=azps-3.0.0).
   
1. Pour créer un partage de fichiers Premium avec le module Azure PowerShell, utilisez l’applet de commande [New-AzRmStorageShare](/powershell/module/az.storage/new-azrmstorageshare).

> [!NOTE]
> Les tailles de partage provisionnées sont spécifiées par le quota de partage. Les partages de fichiers sont facturés selon la taille provisionnée. Pour plus d’informations, consultez la [page relative aux prix appliqués](https://azure.microsoft.com/pricing/details/storage/files/).

  ```powershell
  New-AzRmStorageShare `
   -ResourceGroupName $resourceGroupName `
   -StorageAccountName $storageAccountName `
   -Name myshare `
   -EnabledProtocol NFS `
   -RootSquash RootSquash `
   -Context $storageAcct.Context
  ```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour créer un partage de fichiers Premium avec l’interface de ligne de commande Azure, utilisez la commande [az storage share create](/cli/azure/storage/share-rm).

> [!NOTE]
> Les tailles de partage provisionnées sont spécifiées par le quota de partage. Les partages de fichiers sont facturés selon la taille provisionnée. Pour plus d’informations, consultez la [page relative aux prix appliqués](https://azure.microsoft.com/pricing/details/storage/files/).

```azurecli-interactive
az storage share-rm create \
    --storage-account $STORAGEACCT \
    --enabled-protocol NFS \
    --root-squash RootSquash \
    --name "myshare" 
```
---

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez créé un partage NFS, pour l’utiliser, vous devez le monter sur votre client Linux. Pour plus d’informations, consultez [Comment monter un partage NFS](storage-files-how-to-mount-nfs-shares.md).

Si vous rencontrez des problèmes, consultez [Résoudre les problèmes liés aux partages de fichiers Azure NFS](storage-troubleshooting-files-nfs.md).
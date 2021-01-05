---
title: Activer SMB Multichannel
description: Découvrez comment activer SMB Multichannel sur des partages de fichiers Azure Premium.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 11/16/2020
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 2444ec28a2618b638f78926e214de468f56c5e52
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "95995467"
---
# <a name="enable-smb-multichannel-on-a-filestorage-account-preview"></a>Activer SMB Multichannel sur un compte FileStorage (préversion) 

Les comptes Azure FileStorage prennent en charge SMB Multichannel (préversion), ce qui augmente les performances d’un client SMB 3.x en établissant plusieurs connexions réseau à vos partages de fichiers Premium. Cet article fournit des instructions pas à pas pour activer SMB Multichannel sur un compte de stockage existant. Pour plus d’informations sur Azure Files SMB Multichannel, consultez Performances de SMB Multichannel.

## <a name="limitations"></a>Limites

[!INCLUDE [storage-files-smb-multi-channel-restrictions](../../../includes/storage-files-smb-multi-channel-restrictions.md)]

### <a name="regional-availability"></a>Disponibilité régionale

[!INCLUDE [storage-files-smb-multi-channel-regions](../../../includes/storage-files-smb-multi-channel-regions.md)]

## <a name="prerequisites"></a>Prérequis

- [Créez un compte FileStorage](storage-how-to-create-premium-fileshare.md).
- Si vous envisagez d’utiliser le module Azure PowerShell, [installez la préversion 3.0.1 du module](https://www.powershellgallery.com/packages/Az.Storage/3.0.1-preview).

## <a name="getting-started"></a>Prise en main

Ouvrez une fenêtre PowerShell et connectez-vous à votre abonnement Azure. À partir de là, vous pouvez vous inscrire à la préversion de SMB Multichannel avec les commandes suivantes.

```azurepowershell
Connect-AzAccount
# Setting your active subscription to the one you want to register for the preview. 
# Replace the <subscription-id> placeholder with your subscription id. 
$context = Get-AzSubscription -SubscriptionId <your-subscription-id> 
Set-AzContext $context

Register-AzProviderFeature -FeatureName AllowSMBMultichannel -ProviderNamespace Microsoft.Storage 
Register-AzResourceProvider -ProviderNamespace Microsoft.Storage 
```

> [!NOTE]
> L’inscription peut prendre jusqu’à une heure.

### <a name="verify-that-feature-registration-is-complete"></a>Vérifier que l’inscription des fonctionnalités est terminée

Dans la mesure où l’activation de la fonctionnalité sur votre compte de stockage peut prendre jusqu’à une heure, vous pouvez utiliser la commande suivante pour vérifier qu’elle est inscrite pour votre abonnement :

```azurepowershell
Get-AzProviderFeature -FeatureName AllowSMBMultichannel -ProviderNamespace Microsoft.Storage
```


## <a name="enable-smb-multichannel"></a>Activer SMB Multichannel 
Une fois que vous avez créé un compte FileStorage, vous pouvez suivre les instructions pour mettre à jour les paramètres SMB Multichannel pour votre compte de stockage.

# <a name="portal"></a>[Portail](#tab/azure-portal)
1. Connectez-vous au portail Azure et accédez au compte de stockage FileStorage sur lequel vous souhaitez configurer SMB Multichannel.
1. Sélectionnez **Partages de fichiers** sous **Service Fichier**, puis sélectionnez **Paramètres du partage de fichiers**.
1. Basculez **SMB Multichannel** sur **actif** (ou **inactif** pour désactiver), puis sélectionnez **enregistrer**.

:::image type="content" source="media/storage-files-enable-smb-multichannel/enable-smb-multichannel-on-storage-account.png" alt-text="Capture d’écran du compte de stockage, SMB Multichannel est activé.":::

Si l’option SMB Multichannel n’est pas visible sous **Paramètres du partage de fichiers** ou si vous voyez une erreur Échec de la mise à jour du paramètre lors de la mise à jour de la configuration, vérifiez que votre abonnement est inscrit et que votre compte se trouve dans l’une des [régions prises en charge](#regional-availability) avec un type de compte et une réplication pris en charge.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour activer SMB Multichannel à l’aide du module Azure PowerShell, vous devez [installer la préversion 3.0.1](https://www.powershellgallery.com/packages/Az.Storage/3.0.1-preview) du module.

Définissez les variables `$resourceGroupName` et `$storageAccountName` sur votre groupe de ressources et votre compte de stockage avant d’exécuter ces commandes PowerShell.

```azurepowershell
# Enable SMB Multichannel on the premium storage account that's in one of the supported regions
Update-AzStorageFileServiceProperty -ResourceGroupName $resourceGroupName -StorageAccountName $storageAccountName -EnableSmbMultichannel $true 
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
Azure CLI ne prend pas encore en charge la configuration de SMB Multichannel. Consultez les instructions du portail pour configurer SMB Multichannel sur le compte de stockage.

---

> [!NOTE]
> Toutes les modifications apportées aux paramètres de configuration SMB Multichannel s’appliquent à tous les partages de fichiers sous le compte de stockage. Toutefois, vous devez remonter le partage sur votre client pour que les modifications soient prises en compte.


## <a name="next-steps"></a>Étapes suivantes 

- [Remontez votre partage de fichiers](storage-how-to-use-files-windows.md) pour tirer parti de SMB Multichannel.
- [Résolvez les problèmes liés à SMB Multichannel](storage-troubleshooting-files-performance.md#smb-multichannel-option-not-visible-under-file-share-settings).
- Pour en savoir plus sur les améliorations, consultez [Performances de SMB Multichannel](storage-files-smb-multichannel-performance.md)
 - Pour en savoir plus sur la fonctionnalité Windows SMB Multichannel, consultez [Gérer SMB Multichannel](/azure-stack/hci/manage/manage-smb-multichannel).

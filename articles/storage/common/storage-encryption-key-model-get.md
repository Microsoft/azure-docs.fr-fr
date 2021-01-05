---
title: Déterminer le modèle de clé de chiffrement utilisé pour le compte de stockage
titleSuffix: Azure Storage
description: Utilisez le portail Azure, PowerShell ou Azure CLI pour vérifier le mode de gestion des clés de chiffrement pour le compte de stockage. Les clés peuvent être gérées par Microsoft (configuration par défaut) ou par le client. Les clés gérées par le client doivent être stockées dans Azure Key Vault.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 03/13/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: f59e4238c983cdb336500a68c52730ae5346b1c7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91612419"
---
# <a name="determine-which-azure-storage-encryption-key-model-is-in-use-for-the-storage-account"></a>Déterminer le modèle de clé de chiffrement Stockage Azure utilisé pour le compte de stockage

Les données de votre compte de stockage sont automatiquement chiffrées par Stockage Azure. Le chiffrement Stockage Azure offre deux options de gestion des clés de chiffrement au niveau du compte de stockage :

- **Clés gérées par Microsoft.** Par défaut, Microsoft gère les clés utilisées pour chiffrer votre compte de stockage.
- **Clés gérées par le client.** Vous pouvez éventuellement choisir de gérer les clés de chiffrement pour votre compte de stockage. Les clés gérées par le client doivent être stockées dans Azure Key Vault.

De plus, vous pouvez fournir une clé de chiffrement au niveau d’une demande individuelle pour certaines opérations de stockage d’objets blob. Quand une clé de chiffrement est spécifiée sur la demande, elle remplace la clé de chiffrement qui est active sur le compte de stockage. Pour plus d’informations, consultez [Spécifier une clé fournie par le client sur une demande au stockage d’objets blob](../blobs/storage-blob-customer-provided-key.md).

Pour plus d’informations sur les clés de chiffrement, consultez [Chiffrement du Stockage Azure pour les données au repos](storage-service-encryption.md).

## <a name="check-the-encryption-key-model-for-the-storage-account"></a>Vérifier le modèle de clé de chiffrement du compte de stockage

Pour déterminer si un compte de stockage utilise des clés gérées par Microsoft ou des clés gérées par le client pour le chiffrement, utilisez l’une des méthodes suivantes.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

Pour vérifier le modèle de chiffrement associé au compte de stockage à l’aide du portail Azure, effectuez les étapes suivantes  :

1. Dans le portail Azure, accédez à votre compte de stockage.
1. Sélectionnez le paramètre **Chiffrement** et notez-le.

L’illustration suivante montre un compte de stockage chiffré avec des clés gérées par Microsoft :

![Afficher le compte chiffré avec des clés managées par Microsoft](media/storage-encryption-key-model-get/microsoft-managed-encryption-key-setting-portal.png)

L’illustration suivante montre un compte de stockage chiffré avec des clés gérées par le client :

![Capture d’écran montrant le paramètre de clé de chiffrement dans le portail Azure](media/storage-encryption-key-model-get/customer-managed-encryption-key-setting-portal.png)

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour vérifier le modèle de chiffrement associé au compte de stockage à l’aide de PowerShell, appelez la commande [Get-AzStorageAccount](/powershell/module/az.storage/get-azstorageaccount), puis vérifiez la propriété **KeySource** relative au compte.

```powershell
$account = Get-AzStorageAccount -ResourceGroupName <resource-group> `
    -Name <storage-account>
$account.Encryption.KeySource
```

Si la valeur de la propriété **KeySource** est `Microsoft.Storage`, le compte est chiffré avec des clés gérées par Microsoft. Si la valeur de la propriété **KeySource** est `Microsoft.Keyvault`, le compte est chiffré avec des clés gérées par le client.

# <a name="azure-cli"></a>[Azure CLI](#tab/cli)

Pour vérifier le modèle de chiffrement associé au compte de stockage à l’aide d’Azure CLI, appelez la commande [az storage account show](/cli/azure/storage/account#az-storage-account-show), puis vérifiez la propriété **keySource** relative au compte.

```azurecli-interactive
key_source=$(az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --query encryption.keySource \
    --output tsv)
```

Si la valeur de la propriété **keySource** est `Microsoft.Storage`, le compte est chiffré avec des clés gérées par Microsoft. Si la valeur de la propriété **keySource** est `Microsoft.Keyvault`, le compte est chiffré avec des clés gérées par le client.

---

## <a name="next-steps"></a>Étapes suivantes

- [Chiffrement du stockage Azure pour les données au repos](storage-service-encryption.md)
- [Clés gérées par le client pour le chiffrement du Stockage Azure](customer-managed-keys-overview.md)

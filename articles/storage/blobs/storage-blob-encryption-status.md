---
title: Vérifier l’état de chiffrement d’un objet blob - Stockage Azure
description: Découvrez comment utiliser le portail Azure, PowerShell ou Azure CLI pour vérifier si un objet blob donné est chiffré. Si un objet blob n’est pas chiffré, découvrez comment utiliser AzCopy pour forcer le chiffrement en téléchargeant et en rechargeant l’objet blob.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 11/26/2019
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: 6d0392cd8e94ba8a9026f557b90e740fbed7f50c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84809088"
---
# <a name="check-the-encryption-status-of-a-blob"></a>Vérifier l’état de chiffrement d’un objet blob

Chaque objet blob de blocs, objet blob d’ajouts ou objet blob de pages qui a été écrit dans le Stockage Azure après le 20 octobre 2017 est chiffré avec le chiffrement du Stockage Azure. Les objets blob créés avant cette date continuent à être chiffrés à l’aide d’un processus en arrière-plan.

Cet article explique comment déterminer si un objet blob donné a été chiffré.

## <a name="check-a-blobs-encryption-status"></a>Vérifier l’état de chiffrement d’un objet blob

Utilisez le portail Azure, PowerShell ou Azure CLI pour déterminer si un objet blob est chiffré sans code.

### <a name="azure-portal"></a>[Azure portal](#tab/portal)

Pour utiliser le portail Azure en vue de vérifier si un objet blob a été chiffré, effectuez les étapes suivantes :

1. Dans le portail Azure, accédez à votre compte de stockage.
1. Sélectionnez **Conteneurs** pour accéder à la liste des conteneurs du compte.
1. Localisez l’objet blob, puis affichez son onglet **Vue d’ensemble**.
1. Consultez la propriété **Serveur chiffré**. Si sa valeur est **true**, comme indiqué dans l’image suivante, l’objet blob est chiffré. Notez que les propriétés de l’objet blob incluent également la date et l’heure de sa création.

    ![Capture d’écran montrant comment vérifier la propriété Serveur chiffré dans le portail Azure](media/storage-blob-encryption-status/blob-encryption-property-portal.png)

### <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour utiliser PowerShell en vue de vérifier si un objet blob a été chiffré, vérifiez sa propriété **IsServerEncrypted**. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```powershell
$account = Get-AzStorageAccount -ResourceGroupName <resource-group> `
    -Name <storage-account>
$blob = Get-AzStorageBlob -Context $account.Context `
    -Container <container> `
    -Blob <blob>
$blob.ICloudBlob.Properties.IsServerEncrypted
```

Pour déterminer quand l’objet blob a été créé, vérifiez la valeur de la propriété **Création le** :

```powershell
$blob.ICloudBlob.Properties.IsServerEncrypted
```

### <a name="azure-cli"></a>[Azure CLI](#tab/cli)

Pour utiliser Azure CLI en vue de vérifier si un objet blob a été chiffré, vérifiez sa propriété **IsServerEncrypted**. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```azurecli-interactive
az storage blob show \
    --account-name <storage-account> \
    --container-name <container> \
    --name <blob> \
    --query "properties.serverEncrypted"
```

Pour déterminer quand l’objet blob a été créé, vérifiez la valeur de la propriété **Création le**.

---

## <a name="force-encryption-of-a-blob"></a>Forcer le chiffrement d’un objet blob

Si un objet blob créé avant le 20 octobre 2017 n’a pas encore été chiffré par le processus en arrière-plan, vous pouvez forcer son chiffrement pour qu’il se produise immédiatement en téléchargeant et en rechargeant l’objet blob. Un moyen simple de le faire consiste à utiliser AzCopy.

Pour télécharger un objet blob dans votre système de fichiers local avec AzCopy, utilisez la syntaxe suivante :

```
azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-path>' '<local-file-path>'

Example:
azcopy copy 'https://storagesamples.blob.core.windows.net/sample-container/blob1.txt' 'C:\temp\blob1.txt'
```

Pour recharger l’objet blob dans le Stockage Azure avec AzCopy, utilisez la syntaxe suivante :

```
azcopy copy '<local-file-path>' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-name>'

Example:
azcopy copy 'C:\temp\blob1.txt' 'https://storagesamples.blob.core.windows.net/sample-container/blob1.txt'
```

Pour plus d’informations sur l’utilisation d’AzCopy afin de copier des données d’objets blob, consultez [Transférer des données avec AzCopy et le Stockage Blob](../common/storage-use-azcopy-blobs.md).

## <a name="next-steps"></a>Étapes suivantes

[Chiffrement du stockage Azure pour les données au repos](../common/storage-service-encryption.md)

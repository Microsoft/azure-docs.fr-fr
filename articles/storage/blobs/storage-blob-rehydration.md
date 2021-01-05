---
title: Réalimenter les données d’objets blob à partir du niveau Archive
description: Réalimentez vos blobs à partir du stockage d’archive pour pouvoir accéder aux données des blobs. Copiez un blob archivé sur un niveau de service en ligne.
services: storage
author: mhopkins-msft
ms.author: mhopkins
ms.date: 04/08/2020
ms.service: storage
ms.subservice: blobs
ms.topic: conceptual
ms.reviewer: hux
ms.openlocfilehash: f74d4ffdd724039354a311234317dac889cd7cfe
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95545928"
---
# <a name="rehydrate-blob-data-from-the-archive-tier"></a>Réalimenter les données d’objets blob à partir du niveau Archive

Lorsqu’un objet blob se trouve dans le niveau d’accès Archive, il est considéré comme étant hors connexion et ne peut être lu ni modifié. Les métadonnées de l’objet blob restent en ligne et disponible, ce qui vous permet de répertorier l’objet blob et ses propriétés. La lecture et la modification des données d’objets blob sont uniquement disponibles avec des niveaux en ligne tels que chaud ou froid. Il existe deux options pour récupérer et accéder aux données stockées dans le niveau d’accès Archive.

1. [Réalimenter un blob archivé dans un niveau en ligne](#rehydrate-an-archived-blob-to-an-online-tier) : Réalimenter un blob d’archive dans un niveau chaud ou froid en modifiant son niveau à l’aide de l’opération [Définir le niveau du blob](/rest/api/storageservices/set-blob-tier).
2. [Copier un blob archivé dans un niveau en ligne](#copy-an-archived-blob-to-an-online-tier) : Créer une copie d’un blob d’archive à l’aide de l’opération [Copier le blob](/rest/api/storageservices/copy-blob). Spécifiez un autre nom d’objet blob et un niveau de destination chaud ou froid.

 Pour plus d’informations sur les niveaux, consultez [Stockage Blob Azure : niveaux d’accès chaud, froid et archive](storage-blob-storage-tiers.md).

## <a name="rehydrate-an-archived-blob-to-an-online-tier"></a>Réalimenter un objet blob archivé dans un niveau en ligne

[!INCLUDE [storage-blob-rehydration](../../../includes/storage-blob-rehydrate-include.md)]

## <a name="copy-an-archived-blob-to-an-online-tier"></a>Copier un objet blob archivé dans un niveau en ligne

Si vous ne souhaitez pas réalimenter votre blob d’archive, vous pouvez choisir d’effectuer une opération [Copier le blob](/rest/api/storageservices/copy-blob). Votre blob d’origine reste inchangé dans le niveau archive pendant qu’un nouveau blob est créé dans le niveau chaud ou froid en ligne pour que vous travailliez dessus. Dans l’opération Copier le blob, vous pouvez également définir la propriété facultative *x-ms-réhydrate-priorité* sur Standard ou Haute pour spécifier la priorité à laquelle vous souhaitez créer votre copie de blob.

La copie d’un blob à partir d’une archive peut prendre plusieurs heures, selon la priorité de réalimentation sélectionnée. En arrière-plan, l’opération **Copier le blob** lit votre blob source d’archive pour créer un blob en ligne dans le niveau de destination sélectionné. Le nouveau blob peut être visible lorsque vous répertoriez les blobs, mais les données ne sont pas disponibles tant que la lecture du blob d’archive source n’est pas terminée et que les données ne sont pas écrites dans le nouveau blob de destination en ligne. Le nouveau blob est une copie indépendante et toute modification ou suppression de celui-ci ne se répercute pas sur le blob d’archive source.

> [!IMPORTANT]
> Ne supprimez pas l’objet BLOB source tant que la copie n’est pas terminée avec succès sur la destination. Si l’objet BLOB source est supprimé, l’objet BLOB de destination peut ne pas terminer la copie et sera vide. Vous pouvez vérifier *x-ms-Copy-Status* pour déterminer l’état de l’opération de copie.

Les blobs d’archive peuvent uniquement être copiés vers des niveaux de destination en ligne au sein du même compte de stockage. La copie d’un blob d’archive vers un autre blob d’archive n’est pas prise en charge. Le tableau suivant indique les fonctionnalités de CopyBlob.

|                                           | **Source de niveau chaud**   | **Source de niveau froid** | **Source de niveau d’archive**    |
| ----------------------------------------- | --------------------- | -------------------- | ------------------- |
| **Destination de niveau chaud**                  | Prise en charge             | Prise en charge            | Pris en charge dans le même compte ; en attente de réactivation               |
| **Destination de niveau froid**                 | Prise en charge             | Prise en charge            | Pris en charge dans le même compte ; en attente de réactivation               |
| **Destination du niveau d’archive**              | Prise en charge             | Prise en charge            | Non pris en charge         |

## <a name="pricing-and-billing"></a>Tarification et facturation

La réalimentation d’objets blob hors du niveau archive vers les niveaux chaud ou froid est facturée comme une opération de lecture et de récupération des données. L’utilisation de la haute priorité a des coûts d’exploitation et d’extraction de données plus élevés par rapport à la priorité standard. La réalimentation à priorité élevée apparaît sous la forme d’un élément de ligne distinct sur votre facture. Si une requête de priorité élevée pour retourner un blob d’archive de quelques gigaoctets prend plus de 5 heures, le tarif de récupération haute priorité n’est pas facturé. Toutefois, les tarifs de récupération standard s’appliquent toujours, car la réalimentation a été traitée en prioritaire par rapport à d’autres requêtes.

La copie d’objets blob depuis le niveau archive vers les niveaux chaud ou froid est facturée comme une opération de lecture et de récupération des données. Une opération d’écriture est facturée pour la création de la nouvelle copie du blob. Aucun frais de suppression précoce n’est appliqué lorsque vous copiez vers un objet blob en ligne car l’objet blob source reste inchangé dans le niveau archive. Les frais de récupération à priorité élevée s’appliquent si cette option est sélectionnée.

Les objets blob dans le niveau Archive doivent être stockés pendant un minimum de 180 jours. La suppression ou la réactivation d’objets blob archivés avant 180 jours entraînent des frais de suppression précoce.

> [!NOTE]
> Pour plus d’informations sur la tarification de la réalimentation des données et des objets blob de blocs, consultez [Présentation de la tarification Stockage Azure](https://azure.microsoft.com/pricing/details/storage/blobs/). Pour plus d’informations sur les frais de transfert de données sortantes, consultez la page [Détails de la tarification – Transferts de données](https://azure.microsoft.com/pricing/details/data-transfers/).

## <a name="quickstart-scenarios"></a>Scénarios de démarrage rapide

### <a name="rehydrate-an-archive-blob-to-an-online-tier"></a>Réalimenter un objet blob d’archive dans un niveau en ligne
# <a name="portal"></a>[Portail](#tab/azure-portal)
1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans la Portail Azure, recherchez et sélectionnez **Toutes les ressources**.

1. Sélectionnez votre compte de stockage.

1. Sélectionnez votre conteneur, puis sélectionnez votre objet blob.

1. Dans **Propriétés de l’objet blob**, sélectionnez **Modifiez le niveau**.

1. Sélectionnez le niveau d’accès **Chaud** ou **Froid**. 

1. Sélectionnez une priorité de réalimentation de **Standard** ou **Haute**.

1. Sélectionnez **Enregistrer** en bas.

![Modifier le niveau du compte de stockage](media/storage-tiers/blob-access-tier.png)
![Vérifier l’état de réactivation](media/storage-tiers/rehydrate-status.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Le script PowerShell suivant peut être utilisé pour modifier le niveau d’objet blob d’un objet blob d’archive. La variable `$rgName` doit être initialisée avec le nom de votre groupe de ressources. La variable `$accountName` doit être initialisée avec le nom de votre compte de stockage. La variable `$containerName` doit être initialisée avec le nom de votre conteneur. La variable `$blobName` doit être initialisée avec le nom de votre objet blob. 
```powershell
#Initialize the following with your resource group, storage account, container, and blob names
$rgName = ""
$accountName = ""
$containerName = ""
$blobName = ""

#Select the storage account and get the context
$storageAccount =Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName
$ctx = $storageAccount.Context

#Select the blob from a container
$blob = Get-AzStorageBlob -Container $containerName -Blob $blobName -Context $ctx

#Change the blob’s access tier to Hot using Standard priority rehydrate
$blob.ICloudBlob.SetStandardBlobTier("Hot", "Standard")
```
---

### <a name="copy-an-archive-blob-to-a-new-blob-with-an-online-tier"></a>Copier un objet blob d’archive vers un nouvel objet blob avec un niveau en ligne
Le script PowerShell suivant peut être utilisé pour copier un objet blob d’archive vers un nouvel objet blob sur un même compte de stockage. La variable `$rgName` doit être initialisée avec le nom de votre groupe de ressources. La variable `$accountName` doit être initialisée avec le nom de votre compte de stockage. Les variables `$srcContainerName` et `$destContainerName` doivent être initialisées avec le nom de vos conteneurs. Les variables `$srcBlobName` et `$destBlobName` doivent être initialisées avec le nom de vos objets blob. 
```powershell
#Initialize the following with your resource group, storage account, container, and blob names
$rgName = ""
$accountName = ""
$srcContainerName = ""
$destContainerName = ""
$srcBlobName = ""
$destBlobName = ""

#Select the storage account and get the context
$storageAccount =Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName
$ctx = $storageAccount.Context

#Copy source blob to a new destination blob with access tier hot using standard rehydrate priority
Start-AzStorageBlobCopy -SrcContainer $srcContainerName -SrcBlob $srcBlobName -DestContainer $destContainerName -DestBlob $destBlobName -StandardBlobTier Hot -RehydratePriority Standard -Context $ctx
```

## <a name="next-steps"></a>Étapes suivantes

* [En savoir plus sur les niveaux de Stockage Blob](storage-blob-storage-tiers.md)
* [Vérifier la tarification du niveau chaud, froid et archive dans les comptes de stockage d’objets blob et GPv2 par région](https://azure.microsoft.com/pricing/details/storage/)
* [Gérer le cycle de vie du stockage Blob Azure](storage-lifecycle-management-concepts.md)
* [Vérifier la tarification des transferts de données](https://azure.microsoft.com/pricing/details/data-transfers/)
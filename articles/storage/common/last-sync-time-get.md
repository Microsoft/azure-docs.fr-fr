---
title: Vérifier la propriété Heure de la dernière synchronisation pour un compte de stockage
titleSuffix: Azure Storage
description: Découvrez comment vérifier la propriété Heure de la dernière synchronisation pour un compte de stockage géorépliqué. La propriété Heure de la dernière synchronisation indique la dernière fois où toutes les écritures de la région primaire ont été écrites avec succès dans la région secondaire.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 05/28/2020
ms.author: tamram
ms.reviewer: artek
ms.subservice: common
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 242700c05053aa9d07e3a561a21986c8451a46c7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91612442"
---
# <a name="check-the-last-sync-time-property-for-a-storage-account"></a>Vérifier la propriété Heure de la dernière synchronisation pour un compte de stockage

Lorsque vous configurez un compte de stockage, vous pouvez spécifier que vos données sont copiées dans une région secondaire qui se trouve à des centaines de kilomètres de la région primaire. La géoréplication offre une durabilité pour vos données en cas de panne importante dans la région primaire, par exemple en raison d’une catastrophe naturelle. Si vous activez également l’accès en lecture pour la région secondaire, vos données restent disponibles pour les opérations de lecture si la région primaire n’est plus disponible. Vous pouvez concevoir votre application de manière à ce qu’elle passe sans interruption à la lecture de la région secondaire si la région primaire ne répond pas.

Le stockage géoredondant (GRS) et le stockage géoredondant interzone (GZRS) répliquent vos données de façon asynchrone dans une région secondaire. Pour l’accès en lecture à la région secondaire, activez le stockage géographiquement redondant avec accès en lecture (RA-GRS) ou le stockage géographiquement redondant interzone avec accès en lecture (RA-GZRS). Pour plus d’informations sur les différentes options de redondance offertes par Stockage Azure, consultez [Redondance de Stockage Azure](storage-redundancy.md).

Cet article explique comment vérifier la propriété **Heure de la dernière synchronisation** de votre compte de stockage afin de pouvoir évaluer tout écart entre les régions primaire et secondaire.

## <a name="about-the-last-sync-time-property"></a>À propos de la propriété Heure de la dernière synchronisation

Étant donné que la géoréplication est asynchrone, il est possible que les données écrites dans la région primaire n’aient pas encore été écrites dans la région secondaire au moment où une panne se produit. La propriété **Heure de la dernière synchronisation** indique la dernière fois où les données de la région primaire ont été écrites avec succès dans la région secondaire. Toutes les écritures effectuées dans la région primaire avant l’heure de la dernière synchronisation sont disponibles en lecture à partir de l’emplacement secondaire. Les écritures effectuées dans la région primaire après l’heure de la dernière synchronisation peuvent ou non être déjà disponibles en lecture.

La propriété **Heure de la dernière synchronisation** correspond à une valeur de date/heure GMT.

## <a name="get-the-last-sync-time-property"></a>Obtenir la propriété Heure de la dernière synchronisation

Vous pouvez utiliser PowerShell ou Azure CLI pour récupérer la valeur de la propriété **Heure de la dernière synchronisation**.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour obtenir l’heure de la dernière synchronisation du compte de stockage à l’aide de PowerShell, installez la version 1.11.0 ou une version ultérieure du module [Az.Storage](https://www.powershellgallery.com/packages/Az.Storage). Vérifiez ensuite la propriété **GeoReplicationStats.LastSyncTime** du compte de stockage. N’oubliez pas de remplacer les valeurs d’espace réservé par vos propres valeurs :

```powershell
$lastSyncTime = $(Get-AzStorageAccount -ResourceGroupName <resource-group> `
    -Name <storage-account> `
    -IncludeGeoReplicationStats).GeoReplicationStats.LastSyncTime
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour obtenir l’heure de la dernière synchronisation du compte de stockage à l’aide d’Azure CLI, vérifiez la propriété **geoReplicationStats.lastSyncTime** du compte de stockage. Utilisez le paramètre `--expand` afin de retourner des valeurs pour les propriétés imbriquées sous **geoReplicationStats**. N’oubliez pas de remplacer les valeurs d’espace réservé par vos propres valeurs :

```azurecli-interactive
$lastSyncTime=$(az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --expand geoReplicationStats \
    --query geoReplicationStats.lastSyncTime \
    --output tsv)
```

---

## <a name="see-also"></a>Voir aussi

- [Redondance de Stockage Azure](storage-redundancy.md)
- [Modifier l’option de redondance d’un compte de stockage](redundancy-migration.md)
- [Utiliser la géoredondance pour concevoir des applications hautement disponibles](geo-redundant-design.md)

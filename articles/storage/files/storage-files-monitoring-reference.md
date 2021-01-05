---
title: Informations de référence sur les données de supervision d’Azure Files | Microsoft Docs
description: Informations de référence sur les journaux et les métriques pour la supervision des données d’Azure Files.
author: normesta
services: azure-monitor
ms.service: azure-monitor
ms.topic: reference
ms.date: 10/02/2020
ms.author: normesta
ms.subservice: logs
ms.custom: monitoring
ms.openlocfilehash: 3a3f4faf8278553153ae2186ea74d20702c603b3
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96780924"
---
# <a name="azure-files-monitoring-data-reference"></a>Informations de référence sur les données de supervision d’Azure Files

Pour plus d’informations sur la collecte et l’analyse des données de supervision concernant Azure Files, consultez [Supervision d’Azure Files](storage-files-monitoring.md).

## <a name="metrics"></a>Mesures

Les tableaux suivants listent les métriques de plateforme collectées pour Azure Files. 

### <a name="capacity-metrics"></a>Métriques de capacité

Les valeurs de métriques de capacité sont actualisées quotidiennement (jusqu’à 24 heures). Le fragment de temps définit l’intervalle de temps pour lequel des valeurs de mesures sont présentées. Le fragment de temps pris en charge pour toutes les mesures de capacité est d’une heure (PT1H).

Azure Files fournit les métriques de capacité suivantes dans Azure Monitor.

#### <a name="account-level"></a>Niveau du compte

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-account-capacity-metrics.md)]

#### <a name="azure-files"></a>Azure Files

Ce tableau présente les [métriques Azure Files](../../azure-monitor/platform/metrics-supported.md#microsoftstoragestorageaccountsfileservices).

| Métrique | Description |
| ------------------- | ----------------- |
| FileCapacity | Quantité de stockage de fichiers utilisée par le compte de stockage. <br/><br/> Unité : Octets <br/> Type d’agrégation : Average <br/> Exemple de valeur : 1 024 |
| FileCount   | Nombre de fichiers dans le compte de stockage. <br/><br/> Unité : Count <br/> Type d’agrégation : Average <br/> Exemple de valeur : 1 024 |
| FileShareCapacityQuota | Limite supérieure de la quantité de stockage pouvant être utilisée par le service Azure Files, en octets. <br/><br/> Unité : Octets <br/> Type d’agrégation : Average <br/> Exemple de valeur : 1 024|
| FileShareCount | Nombre de partages de fichiers dans le compte de stockage. <br/><br/> Unité : Count <br/> Type d’agrégation : Average <br/> Exemple de valeur : 1 024 |
| FileShareProvisionedIOPS | Nombre d’IOPS provisionnées sur un partage de fichiers. Cette métrique s’applique uniquement au stockage de fichiers Premium. <br/><br/> Unité : octets <br/> Type d’agrégation : Average |
| FileShareSnapshotCount | Nombre d’instantanés présents sur le partage dans le service Azure Files du compte de stockage. <br/><br/> Unité : nombre <br/> Type d’agrégation : Average | 
|FileShareSnapshotSize|Quantité de stockage utilisée par les instantanés dans le service Azure Files du compte de stockage. <br/><br/> Unité : Octets <br/> Type d’agrégation : Average|

### <a name="transaction-metrics"></a>Métriques de transaction

Des métriques de transaction sont émises lors de chacune des requêtes adressées à un compte de stockage entre Stockage Azure et Azure Monitor. En l'absence d'activité sur votre compte de stockage, il n'y aura pas de données sur les métriques de transaction au cours la période. Toutes les métriques de transaction sont disponibles aussi bien au niveau du compte qu’au niveau du service Azure Files. Le fragment de temps définit l’intervalle de temps auquel des valeurs de mesures sont présentées. Les fragments de temps pris en charge pour toutes les mesures de transaction sont PT1H et PT1M.

[!INCLUDE [Transaction metrics](../../../includes/azure-storage-account-transaction-metrics.md)]

<a id="metrics-dimensions"></a>

## <a name="metrics-dimensions"></a>Dimensions de mesures

Azure Files prend en charge les dimensions suivantes pour les métriques dans Azure Monitor.

> [!NOTE] 
> La dimension Partage de fichiers n’est pas disponible pour les partages de fichiers standard (uniquement pour les partages de fichiers Premium). Lors de l’utilisation de partages de fichiers standard, les métriques fournies concernent tous les partages de fichiers présents dans le compte de stockage. Pour obtenir les métriques par partage pour les partages de fichiers standard, créez un partage de fichiers par compte de stockage.

[!INCLUDE [Metrics dimensions](../../../includes/azure-storage-account-metrics-dimensions.md)]

## <a name="resource-logs-preview"></a>Journaux de ressources (préversion)

> [!NOTE]
> Les journaux de stockage Azure dans Azure Monitor sont en préversion publique et sont disponibles pour le test en préversion dans toutes les régions de cloud public. Cette préversion active les journaux pour les objets blob (y compris Azure Data Lake Storage Gen2), les fichiers, les files d’attente, les tables, les comptes de stockage Premium dans les comptes de stockage à usage général v1 et à usage général v2. Les comptes de stockage classiques ne sont pas pris en charge.

Le tableau suivant liste les propriétés des journaux de ressources de stockage Azure lorsqu’elles sont collectées dans les journaux Azure Monitor ou dans le Stockage Azure. Ces propriétés décrivent l’opération, le service et le type d’autorisation qui ont servi à effectuer l’opération.

### <a name="fields-that-describe-the-operation"></a>Champs qui décrivent l’opération


[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-logs-properties-operation.md)]

### <a name="fields-that-describe-how-the-operation-was-authenticated"></a>Champs qui décrivent comment l’opération a été authentifiée

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-logs-properties-authentication.md)]

### <a name="fields-that-describe-the-service"></a>Champs qui décrivent le service

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-logs-properties-service.md)]

## <a name="see-also"></a>Voir également

- Pour obtenir une description de la supervision du Stockage Azure, consultez [Supervision d’Azure Files](storage-files-monitoring-reference.md).
- Pour plus d’informations sur le monitoring des ressources Azure, voir [Monitoring des ressources Azure avec Azure Monitor](../../azure-monitor/insights/monitor-azure-resource.md).
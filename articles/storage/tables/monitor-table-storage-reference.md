---
title: Informations de référence sur les données de supervision du service Stockage Table Azure | Microsoft Docs
description: Informations de référence sur les journaux et les métriques pour la supervision des données à partir du service Stockage Table Azure.
author: normesta
services: azure-monitor
ms.service: azure-monitor
ms.topic: reference
ms.date: 10/02/2020
ms.author: normesta
ms.subservice: logs
ms.custom: monitoring
ms.openlocfilehash: b9920956a8983a1c71c1f91cd1c19a6e84be6aee
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96780295"
---
# <a name="azure-table-storage-monitoring-data-reference"></a>Informations de référence sur les données de supervision du service Stockage Table Azure

Consultez [Supervision du stockage Azure](monitor-table-storage.md) pour plus d’informations sur la collecte et l’analyse des données de supervision du stockage Azure.

## <a name="metrics"></a>Mesures

Les tableaux suivants répertorient les métriques de plateforme collectées pour le stockage Azure. 

### <a name="capacity-metrics"></a>Métriques de capacité

Les valeurs de mesures de capacité sont envoyées à Azure Monitor toutes les heures. Les valeurs sont actualisées tous les jours. Le fragment de temps définit l’intervalle de temps pour lequel des valeurs de mesures sont présentées. Le fragment de temps pris en charge pour toutes les mesures de capacité est d’une heure (PT1H).

Stockage Azure fournit les mesures de capacité suivantes dans Azure Monitor.

#### <a name="account-level"></a>Niveau du compte

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-account-capacity-metrics.md)]

#### <a name="table-storage"></a>Stockage de tables

Ce tableau montre des [métriques de Stockage Table](../../azure-monitor/platform/metrics-supported.md#microsoftstoragestorageaccountstableservices).

| Métrique | Description |
| ------------------- | ----------------- |
| TableCapacity | Quantité de stockage de tables utilisée par le compte de stockage. <br/><br/> Unité : Octets <br/> Type d’agrégation : Average <br/> Exemple de valeur : 1 024 |
| TableCount   | Nombre de tables dans le compte de stockage. <br/><br/> Unité : Count <br/> Type d’agrégation : Average <br/> Exemple de valeur : 1 024 |
| TableEntityCount | Nombre d’entités de table dans le compte de stockage. <br/><br/> Unité : Count <br/> Type d’agrégation : Average <br/> Exemple de valeur : 1 024 |

### <a name="transaction-metrics"></a>Métriques de transaction

Des métriques de transaction sont émises lors de chacune des requêtes adressées à un compte de stockage entre Stockage Azure et Azure Monitor. En l'absence d'activité sur votre compte de stockage, il n'y aura pas de données sur les métriques de transaction au cours la période. Toutes les métriques de transaction sont disponibles au niveau du compte et du service Stockage Table. Le fragment de temps définit l’intervalle de temps auquel des valeurs de mesures sont présentées. Les fragments de temps pris en charge pour toutes les mesures de transaction sont PT1H et PT1M.

[!INCLUDE [Transaction metrics](../../../includes/azure-storage-account-transaction-metrics.md)]

<a id="metrics-dimensions"></a>

## <a name="metrics-dimensions"></a>Dimensions de mesures

Stockage Azure prend en charge les dimensions suivantes pour les mesures dans Azure Monitor.

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

- Consultez [Supervision du service Stockage Table Azure](monitor-table-storage.md) pour obtenir une description de la supervision du stockage Azure.
- Pour plus d’informations sur le monitoring des ressources Azure, voir [Monitoring des ressources Azure avec Azure Monitor](../../azure-monitor/insights/monitor-azure-resource.md).
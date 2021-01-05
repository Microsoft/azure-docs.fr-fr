---
title: Informations de référence sur les données de supervision du Stockage blob Azure | Microsoft Docs
description: Informations de référence sur les journaux et les métriques pour la supervision des données du Stockage blob Azure.
author: normesta
services: azure-monitor
ms.service: azure-monitor
ms.topic: reference
ms.date: 10/02/2020
ms.author: normesta
ms.subservice: logs
ms.custom: monitoring
ms.openlocfilehash: 16ae2f9e74202aff47e58a22dbe21a28d8280a7e
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96780720"
---
# <a name="azure-blob-storage-monitoring-data-reference"></a>Informations de référence sur les données de supervision du Stockage blob Azure

Consultez [Supervision du stockage Azure](monitor-blob-storage.md) pour plus d’informations sur la collecte et l’analyse des données de supervision du stockage Azure.

## <a name="metrics"></a>Mesures

Les tableaux suivants répertorient les métriques de plateforme collectées pour le stockage Azure. 

### <a name="capacity-metrics"></a>Métriques de capacité

Les valeurs de métriques de capacité sont actualisées quotidiennement (jusqu’à 24 heures). Le fragment de temps définit l’intervalle de temps pour lequel des valeurs de mesures sont présentées. Le fragment de temps pris en charge pour toutes les mesures de capacité est d’une heure (PT1H).

Stockage Azure fournit les mesures de capacité suivantes dans Azure Monitor.

#### <a name="account-level"></a>Niveau du compte

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-account-capacity-metrics.md)]

#### <a name="blob-storage"></a>Stockage d'objets blob

Ce tableau montre des [métriques de Stockage Blob](../../azure-monitor/platform/metrics-supported.md#microsoftstoragestorageaccountsblobservices).

| Métrique | Description |
| ------------------- | ----------------- |
| BlobCapacity | Total de stockage d’objets blob utilisé dans le compte de stockage. <br/><br/> Unité : Octets <br/> Type d’agrégation : Average <br/> Exemple de valeur : 1 024 <br/> Dimensions : **BlobType** et **BlobTier** ([définition](#metrics-dimensions)) |
| BlobCount    | Nombre d’objets blob stockés dans le compte de stockage. <br/><br/> Unité : Count <br/> Type d’agrégation : Average <br/> Exemple de valeur : 1 024 <br/> Dimensions : **BlobType** et **BlobTier** ([définition](#metrics-dimensions)) |
| BlobProvisionedSize | Quantité de stockage provisionnée dans le compte de stockage. Cette métrique s’applique uniquement aux comptes de stockage Premium. <br/><br/> Unité : octets <br/> Type d’agrégation : Average |
| ContainerCount    | Nombre de conteneurs dans le compte de stockage. <br/><br/> Unité : Count <br/> Type d’agrégation : Average <br/> Exemple de valeur : 1 024 |
| IndexCapacity     | Espace utilisé par l'index hiérarchique d'ADLS Gen2 <br/><br/> Unité : Octets <br/> Type d’agrégation : Average <br/> Exemple de valeur : 1 024 |

### <a name="transaction-metrics"></a>Métriques de transaction

Des métriques de transaction sont émises lors de chacune des requêtes adressées à un compte de stockage entre Stockage Azure et Azure Monitor. En l'absence d'activité sur votre compte de stockage, il n'y aura pas de données sur les métriques de transaction au cours la période. Toutes les métriques de transaction sont disponibles au niveau du compte et du service de stockage d’objets blob. Le fragment de temps définit l’intervalle de temps auquel des valeurs de mesures sont présentées. Les fragments de temps pris en charge pour toutes les mesures de transaction sont PT1H et PT1M.

[!INCLUDE [Transaction metrics](../../../includes/azure-storage-account-transaction-metrics.md)]

<a id="metrics-dimensions"></a>

## <a name="metrics-dimensions"></a>Dimensions de mesures

Stockage Azure prend en charge les dimensions suivantes pour les mesures dans Azure Monitor.

### <a name="dimensions-available-to-all-storage-services"></a>Dimensions disponibles dans tous les services de stockage

[!INCLUDE [Metrics dimensions](../../../includes/azure-storage-account-metrics-dimensions.md)]

### <a name="dimensions-specific-to-blob-storage"></a>Dimensions spécifiques au stockage d’objets blob

| Nom de la dimension | Description |
| ------------------- | ----------------- |
| **BlobType** | Type d’objet blob pour les mesures d’objet Blob uniquement. Les valeurs prises en charge sont **BlockBlob**, **PageBlob**, et **Azure Data Lake Storage**. Les blobs d’ajout sont inclus dans **BlockBlob**. |
| **BlobTier** | Le Stockage Azure propose différents niveaux d’accès qui vous permettent de stocker vos objets blob de la manière la plus économique. Pour en savoir plus, consultez [Niveau du stockage Azure d’objets blob](../blobs/storage-blob-storage-tiers.md). Les valeurs prises en charge incluent : <br/> <li>**Hot** : Niveau de stockage chaud</li> <li>**Cool** : Niveau de stockage froid</li> <li>**Archivage** : Niveau de stockage archive</li> <li>**Premium** : Niveau Premium pour les objets blob de blocs</li> <li>**P4/P6/P10/P15/P20/P30/P40/P50/P60** : Types de niveau pour les objets blob de pages premium</li> <li>**Standard** : Type de niveau pour les objets blob de pages standard</li> <li>**Untiered** : Type de niveau pour un compte de stockage à usage général v1</li> |

Pour les mesures prenant en charge des dimensions, vous devez spécifier la valeur de la dimension pour afficher les valeurs de mesures correspondantes. Par exemple, si vous examinez la valeur **Transactions** pour des réponses réussies, vous devez filtrer la dimension **ResponseType** avec **Success**. Si vous examinez la valeur **BlobCount** pour BlockBlob, vous devez filtrer la dimension **BlobType** avec **BlockBlob**.

## <a name="resource-logs-preview"></a>Journaux de ressources (préversion)

> [!NOTE]
> Les journaux de stockage Azure dans Azure Monitor sont en préversion publique et sont disponibles pour le test en préversion dans toutes les régions de cloud public. Cette préversion active les journaux pour les objets blob (y compris Azure Data Lake Storage Gen2), les fichiers, les files d’attente, les tables, les comptes de stockage Premium dans les comptes de stockage à usage général v1 et à usage général v2. Les comptes de stockage classiques ne sont pas pris en charge.

Le tableau suivant liste les propriétés des journaux de ressources de stockage Azure lorsqu’elles sont collectées dans les journaux Azure Monitor ou dans le Stockage Azure. Ces propriétés décrivent l’opération, le service et le type d’autorisation qui ont servi à effectuer l’opération.

### <a name="fields-that-describe-the-operation"></a>Champs qui décrivent l’opération

```json
{
    "time": "2019-02-28T19:10:21.2123117Z",
    "resourceId": "/subscriptions/12345678-2222-3333-4444-555555555555/resourceGroups/mytestrp/providers/Microsoft.Storage/storageAccounts/testaccount1/blobServices/default",
    "category": "StorageWrite",
    "operationName": "PutBlob",
    "operationVersion": "2017-04-17",
    "schemaVersion": "1.0",
    "statusCode": 201,
    "statusText": "Success",
    "durationMs": 5,
    "callerIpAddress": "192.168.0.1:11111",
    "correlationId": "ad881411-201e-004e-1c99-cfd67d000000",
    "location": "uswestcentral",
    "uri": "http://mystorageaccount.blob.core.windows.net/cont1/blobname?timeout=10"
}
```

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-logs-properties-operation.md)]

### <a name="fields-that-describe-how-the-operation-was-authenticated"></a>Champs qui décrivent comment l’opération a été authentifiée

```json
{
    "identity": {
        "authorization": [
            {
                "action": "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read",
                "principals": [
                    {
                        "id": "fde5ba15-4355-4223-b811-cccccccccccc",
                        "type": "User"
                    }
                ],
                "roleAssignmentId": "ecf75cb8-491c-4a25-ad6e-aaaaaaaaaaaa",
                "roleDefinitionId": "b7e6dc6d-f1e8-4753-8033-ffffffffffff"
            }
        ],
        "requester": {
            "appId": "691458b9-1327-4635-9f55-bbbbbbbbbbbb",
            "audience": "https://storage.azure.com/",
            "objectId": "fde5ba15-4355-4223-b811-cccccccccccc",
            "tenantId": "72f988bf-86f1-41af-91ab-dddddddddddd",
            "tokenIssuer": "https://sts.windows.net/72f988bf-86f1-41af-91ab-eeeeeeeeeeee/"
           },
        "type": "OAuth"
    },
}

```

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-logs-properties-authentication.md)]

### <a name="fields-that-describe-the-service"></a>Champs qui décrivent le service

```json
{
    "properties": {
        "accountName": "testaccount1",
        "requestUrl": "https://testaccount1.blob.core.windows.net:443/upload?restype=container&comp=list&prefix=&delimiter=%2F&marker=&maxresults=30&include=metadata&_=1551405598426",
        "userAgentHeader": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.140 Safari/537.36 Edge/17.17134",
        "referrerHeader": "blob:https://ms.portal.azure.com/6f50025f-3b88-488d-b29e-3c592a31ddc9",
        "clientRequestId": "",
        "etag": "",
        "serverLatencyMs": 63,
        "serviceType": "blob",
        "operationCount": 0,
        "requestHeaderSize": 2658,
        "requestBodySize": 0,
        "responseHeaderSize": 295,
        "responseBodySize": 2018,
        "contentLengthHeader": 0,
        "requestMd5": "",
        "serverMd5": "",
        "lastModifiedTime": "",
        "conditionsUsed": "",
        "smbTreeConnectID" : "0x3",
        "smbPersistentHandleID" : "0x6003f",
        "smbVolatileHandleID" : "0xFFFFFFFF00000065",
        "smbMessageID" : "0x3b165",
        "smbCreditsConsumed" : "0x3",
        "smbCommandDetail" : "0x2000 bytes at offset 0xf2000",
        "smbFileId" : " 0x9223442405598953",
        "smbSessionID" : "0x8530280128000049",
        "smbCommandMajor" : "0x6",
        "smbCommandMinor" : "DirectoryCloseAndDelete"
    }

}
```

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-logs-properties-service.md)]

## <a name="see-also"></a>Voir aussi

- Consultez [Supervision du stockage Azure](monitor-blob-storage.md) pour obtenir une description de la supervision du stockage Azure.
- Pour plus d’informations sur le monitoring des ressources Azure, voir [Monitoring des ressources Azure avec Azure Monitor](../../azure-monitor/insights/monitor-azure-resource.md).
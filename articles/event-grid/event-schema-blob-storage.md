---
title: Stockage Blob Azure en tant que source Event Grid
description: Décrit les propriétés fournies pour les événements de stockage Blob avec Azure Event Grid.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 1a81b30fcb775f5e8bc99bda70307f7f1aed9796
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452552"
---
# <a name="azure-blob-storage-as-an-event-grid-source"></a>Stockage Blob Azure en tant que source Event Grid

Cet article fournit les propriétés et les schémas des événements de stockage Blob.  Pour une présentation des schémas d’événements, consultez [Schéma d’événements Azure Event Grid](event-schema.md). Cet article fournit également une liste de démarrages rapides et de tutoriels permettant d’utiliser Stockage Blob Azure comme source d’événement.


>[!NOTE]
> Seuls les comptes de stockage de type **StorageV2 (v2 universel)** , **BlockBlobStorage** et **BlobStorage** prennent en charge l’intégration d’événements. Le type **Stockage (v1 universel)** ne prend *pas* en charge l’intégration à Event Grid.

## <a name="event-grid-event-schema"></a>Schéma d’événement Event Grid

### <a name="list-of-events-for-blob-rest-apis"></a>Liste des événements pour des API REST d’objets blob

Ces événements sont déclenchés quand un client crée, remplace ou supprime un objet blob en appelant des API REST d’objets blob.

> [!NOTE]
> Les conteneurs `$logs` et `$blobchangefeed` ne sont pas intégrés à Event Grid, si bien que l’activité dans ces conteneurs ne génère pas d’événements. De plus, l’utilisation du point de terminaison DFS *`(abfss://URI) `* pour des comptes d’espace de noms activé non hiérarchique ne génère pas d’événements, mais le point de terminaison d’objet blob *`(wasb:// URI)`* génère des événements.

 |Nom d'événement |Description|
 |----------|-----------|
 |**Microsoft.Storage.BlobCreated** |Déclenché quand un objet blob est créé ou remplacé. <br>Plus précisément, cet événement est déclenché quand des clients utilisent les opérations `PutBlob`, `PutBlockList` ou `CopyBlob` qui sont disponibles dans l’API REST d’objet blob.   |
 |**Microsoft.Storage.BlobDeleted** |Déclenché quand un objet blob est supprimé. <br>Plus précisément, cet événement est déclenché quand des clients appellent l’opération `DeleteBlob` qui est disponible dans l’API REST d’objet blob. |

> [!NOTE]
> Concernant **Stockage Blob Azure**, si vous souhaitez que l'événement **Microsoft.Storage.BlobCreated** ne soit déclenché que lorsqu'un objet blob de blocs est complètement validé, filtrez l'événement pour les appels d'API REST `CopyBlob`, `PutBlob` et `PutBlockList`. Ces appels d’API déclenchent l’événement **Microsoft.Storage.BlobCreated** uniquement quand les données sont entièrement validées dans un objet blob de blocs. Pour savoir comment créer un filtre, consultez [Filtrer des événements pour Event Grid](./how-to-filter-events.md).

### <a name="list-of-the-events-for-azure-data-lake-storage-gen-2-rest-apis"></a>Liste des événements pour les API REST Azure Data Lake Storage Gen2

Ces événements sont déclenchés si vous activez un espace de noms hiérarchique sur le compte de stockage et que des clients utilisent les API REST d'Azure Data Lake Storage Gen2. Pour plus d’informations sur Azure Data Lake Storage Gen2, consultez [Présentation d’Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md).

|Nom d'événement|Description|
|----------|-----------|
|**Microsoft.Storage.BlobCreated** | Déclenché quand un objet blob est créé ou remplacé. <br>Plus précisément, cet événement est déclenché quand des clients utilisent les opérations `CreateFile` et `FlushWithClose` qui sont disponibles dans l’API REST Azure Data Lake Storage Gen2. |
|**Microsoft.Storage.BlobDeleted** |Déclenché quand un objet blob est supprimé. <br>Plus précisément, cet événement est également déclenché quand des clients appellent l’opération `DeleteFile` qui est disponible dans l’API REST Azure Data Lake Storage Gen2. |
|**Microsoft.Storage.BlobRenamed**|Déclenché quand un objet blob est renommé. <br>Plus précisément, cet événement est déclenché quand des clients utilisent l’opération `RenameFile` qui est disponible dans l’API REST Azure Data Lake Storage Gen2.|
|**Microsoft.Storage.DirectoryCreated**|Déclenché quand un répertoire est créé. <br>Plus précisément, cet événement est déclenché quand des clients utilisent l’opération `CreateDirectory` qui est disponible dans l’API REST Azure Data Lake Storage Gen2.|
|**Microsoft.Storage.DirectoryRenamed**|Déclenché quand un répertoire est renommé. <br>Plus précisément, cet événement est déclenché quand des clients utilisent l’opération `RenameDirectory` qui est disponible dans l’API REST Azure Data Lake Storage Gen2.|
|**Microsoft.Storage.DirectoryDeleted**|Déclenché quand un répertoire est supprimé. <br>Plus précisément, cet événement est déclenché quand des clients utilisent l’opération `DeleteDirectory` qui est disponible dans l’API REST Azure Data Lake Storage Gen2.|

> [!NOTE]
> Concernant **Azure Data Lake Storage Gen2**, si vous souhaitez que l'événement **Microsoft.Storage.BlobCreated** ne soit déclenché que lorsqu'un objet blob de blocs est complètement validé, filtrez l'événement pour l'appel d'API REST `FlushWithClose`. Cet appel d’API déclenche l’événement **Microsoft.Storage.BlobCreated** uniquement quand les données sont entièrement validées dans un objet blob de blocs. Pour savoir comment créer un filtre, consultez [Filtrer des événements pour Event Grid](./how-to-filter-events.md).

<a name="example-event"></a>
### <a name="the-contents-of-an-event-response"></a>Le contenu d’une réponse à un événement

Quand un événement est déclenché, le service Event Grid envoie les données relatives à cet événement au point de terminaison d’abonnement.

Cette section contient un exemple de ce à quoi ces données ressembleraient pour chaque événement de stockage d’objet blob.

### <a name="microsoftstorageblobcreated-event"></a>Événement Microsoft.Storage.BlobCreated

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/test-container/blobs/new-file.txt",
  "eventType": "Microsoft.Storage.BlobCreated",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "PutBlockList",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "eTag": "\"0x8D4BCC2E4835CD0\"",
    "contentType": "text/plain",
    "contentLength": 524288,
    "blobType": "BlockBlob",
    "url": "https://my-storage-account.blob.core.windows.net/testcontainer/new-file.txt",
    "sequencer": "00000000000004420000000000028963",
    "storageDiagnostics": {
      "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstorageblobcreated-event-data-lake-storage-gen2"></a>Événement Microsoft.Storage.BlobCreated (Data Lake Storage Gen2)

Si le compte de stockage d'objets blob dispose d'un espace de noms hiérarchique, les données sont semblables à l'exemple précédent, à l'exception des changements suivants :

* La clé `dataVersion` est définie sur la valeur `2`.

* La clé `data.api` est définie sur la chaîne `CreateFile` ou `FlushWithClose`.

* La clé `contentOffset` est incluse dans le jeu de données.

> [!NOTE]
> Si des applications utilisent l’opération `PutBlockList` pour charger un nouvel objet blob dans le compte, les données ne contiennent pas ces changements.

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/new-file.txt",
  "eventType": "Microsoft.Storage.BlobCreated",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "CreateFile",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "eTag": "\"0x8D4BCC2E4835CD0\"",
    "contentType": "text/plain",
    "contentLength": 0,
    "contentOffset": 0,
    "blobType": "BlockBlob",
    "url": "https://my-storage-account.dfs.core.windows.net/my-file-system/new-file.txt",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "2",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstorageblobdeleted-event"></a>Événement Microsoft.Storage.BlobDeleted

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/testcontainer/blobs/file-to-delete.txt",
  "eventType": "Microsoft.Storage.BlobDeleted",
  "eventTime": "2017-11-07T20:09:22.5674003Z",
  "id": "4c2359fe-001e-00ba-0e04-58586806d298",
  "data": {
    "api": "DeleteBlob",
    "requestId": "4c2359fe-001e-00ba-0e04-585868000000",
    "contentType": "text/plain",
    "blobType": "BlockBlob",
    "url": "https://my-storage-account.blob.core.windows.net/testcontainer/file-to-delete.txt",
    "sequencer": "0000000000000281000000000002F5CA",
    "storageDiagnostics": {
      "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstorageblobdeleted-event-data-lake-storage-gen2"></a>Événement Microsoft.Storage.BlobDeleted (Data Lake Storage Gen2)

Si le compte de stockage d'objets blob dispose d'un espace de noms hiérarchique, les données sont semblables à l'exemple précédent, à l'exception des changements suivants :

* La clé `dataVersion` est définie sur la valeur `2`.

* La clé `data.api` est définie sur la chaîne `DeleteFile`.

* La clé `url` contient le chemin `dfs.core.windows.net`.

> [!NOTE]
> Si des applications utilisent l’opération `DeleteBlob` pour supprimer un objet blob du compte, les données ne contiennent pas ces changements.

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/file-to-delete.txt",
  "eventType": "Microsoft.Storage.BlobDeleted",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
    "data": {
    "api": "DeleteFile",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "contentType": "text/plain",
    "blobType": "BlockBlob",
    "url": "https://my-storage-account.dfs.core.windows.net/my-file-system/file-to-delete.txt",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "2",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstorageblobrenamed-event"></a>Événement Microsoft.Storage.BlobRenamed

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/my-renamed-file.txt",
  "eventType": "Microsoft.Storage.BlobRenamed",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "RenameFile",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "destinationUrl": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-renamed-file.txt",
    "sourceUrl": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-original-file.txt",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstoragedirectorycreated-event"></a>Événement Microsoft.Storage.DirectoryCreated

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/my-new-directory",
  "eventType": "Microsoft.Storage.DirectoryCreated",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "CreateDirectory",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "url": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-new-directory",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstoragedirectoryrenamed-event"></a>Événement Microsoft.Storage.DirectoryRenamed

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/my-renamed-directory",
  "eventType": "Microsoft.Storage.DirectoryRenamed",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "RenameDirectory",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "destinationUrl": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-renamed-directory",
    "sourceUrl": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-original-directory",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstoragedirectorydeleted-event"></a>Événement Microsoft.Storage.DirectoryDeleted

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/directory-to-delete",
  "eventType": "Microsoft.Storage.DirectoryDeleted",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "DeleteDirectory",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "url": "https://my-storage-account.dfs.core.windows.net/my-file-system/directory-to-delete",
    "recursive": "true", 
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

### <a name="event-properties"></a>Propriétés d’événement

Un événement contient les données générales suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| topic | string | Chemin d’accès complet à la source de l’événement. Ce champ n’est pas modifiable. Event Grid fournit cette valeur. |
| subject | string | Chemin de l’objet de l’événement, défini par le serveur de publication. |
| eventType | string | Un des types d’événements inscrits pour cette source d’événement. |
| eventTime | string | L’heure à quelle l’événement est généré selon l’heure UTC du fournisseur. |
| id | string | Identificateur unique de l’événement. |
| data | object | Données d’événement de stockage Blob. |
| dataVersion | string | Version du schéma de l’objet de données. Le serveur de publication définit la version du schéma. |
| metadataVersion | string | Version du schéma des métadonnées d’événement. Event Grid définit le schéma des propriétés de niveau supérieur. Event Grid fournit cette valeur. |

L’objet de données comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| api | string | Opération qui a déclenché l’événement. |
| clientRequestId | string | ID de requête fourni par le client pour l’opération d’API de stockage. Cet ID peut être utilisé pour mettre en corrélation les journaux de diagnostic Stockage Azure en utilisant le champ « client-request-id » dans les journaux, et peut être fourni dans des requêtes du client à l’aide de l’en-tête « x-ms-client-request-id ». Consultez [Format de journal](/rest/api/storageservices/storage-analytics-log-format). |
| requestId | string | ID de requête généré par le service pour l’opération de l’API de stockage. Peut être utilisé pour mettre en corrélation les journaux de diagnostic de stockage Azure en utilisant le champ « request-id-header » dans les journaux d’activité et est retourné lors de l’initialisation de l’appel d’API dans l’en-tête ’x-ms-request-id’. Consultez [Format de journal](/rest/api/storageservices/storage-analytics-log-format). |
| eTag | string | Valeur que vous pouvez utiliser pour exécuter des opérations de manière conditionnelle. |
| contentType | string | Type de contenu spécifié pour l’objet blob. |
| contentLength | entier | Taille de l’objet blob en octets. |
| blobType | string | Type d’objet blob. Les valeurs valides sont « BlockBlob » ou « PageBlob ». |
| contentOffset | nombre | Décalage, en octets, d’une opération d’écriture effectuée au point où l’application de déclenchement d’événement a effectué l’écriture dans le fichier. <br>Apparaît uniquement pour les événements déclenchés sur les comptes de stockage d’objets blob qui ont un espace de noms hiérarchique.|
| destinationUrl |string | URL du fichier qui existera une fois l’opération terminée. Par exemple, si un fichier est renommé, la propriété `destinationUrl` contient l’URL du nouveau nom de fichier. <br>Apparaît uniquement pour les événements déclenchés sur les comptes de stockage d’objets blob qui ont un espace de noms hiérarchique.|
| sourceUrl |string | URL du fichier qui existe avant que l'opération ne soit effectuée. Par exemple, si un fichier est renommé, `sourceUrl` contient l'URL du nom de fichier d'origine avant l'opération de changement de nom. <br>Apparaît uniquement pour les événements déclenchés sur les comptes de stockage d’objets blob qui ont un espace de noms hiérarchique. |
| url | string | Chemin de l’objet blob. <br>Si le client utilise une API REST Blob, la structure de l'URL est la suivante :`<storage-account-name>.blob.core.windows.net\<container-name>\<file-name>`. <br>Si le client utilise une API REST Data Lake Storage, la structure de l'URL est la suivante : `<storage-account-name>.dfs.core.windows.net/<file-system-name>/<file-name>`. |
| recursive | string | `True` pour exécuter l'opération sur tous les répertoires enfants ; sinon, `False`. <br>Apparaît uniquement pour les événements déclenchés sur les comptes de stockage d’objets blob qui ont un espace de noms hiérarchique. |
| sequencer | string | Une valeur de chaîne opaque représentant l’ordre logique des événements pour n’importe quel nom d’objet Blob particulier.  Les utilisateurs peuvent utiliser la comparaison de chaînes standard pour comprendre l’ordre relatif de deux événements sur le même nom d’objet Blob. |
| storageDiagnostics | object | Des données de diagnostic occasionnellement incluses par le service de stockage Azure. Elles doivent, le cas échéant, être ignorées par les consommateurs d’événements. |

## <a name="tutorials-and-how-tos"></a>Tutoriels et guides pratiques
|Intitulé  |Description  |
|---------|---------|
| [Démarrage rapide : Acheminer des événements de stockage Blob vers un point de terminaison web avec Azure CLI](../storage/blobs/storage-blob-event-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Montre comment utiliser Azure CLI pour envoyer des événements de stockage d’objets blob à un webhook. |
| [Démarrage rapide : Acheminer des événements de stockage Blob vers un point de terminaison web avec PowerShell](../storage/blobs/storage-blob-event-quickstart-powershell.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Montre comment utiliser Azure PowerShell pour envoyer des événements de stockage d’objets blob à un webhook. |
| [Démarrage rapide : Créer et acheminer des événements de stockage d’objets blob avec le portail Azure](blob-event-quickstart-portal.md) | Montre comment utiliser le portail pour envoyer des événements de stockage d’objets blob à un webhook. |
| [Azure CLI : S’abonner aux événements d’un compte de stockage Blob](./scripts/event-grid-cli-blob.md) | Exemple de script pour s’abonner aux événements d’un compte de stockage Blob. Il envoie l’événement à un webhook. |
| [PowerShell : S’abonner aux événements d’un compte de stockage Blob](./scripts/event-grid-powershell-blob.md) | Exemple de script pour s’abonner aux événements d’un compte de stockage Blob. Il envoie l’événement à un webhook. |
| [Modèle Resource Manager : Créer un abonnement et un stockage d’objets blob](https://github.com/Azure/azure-quickstart-templates/tree/master/101-event-grid-subscription-and-storage) | Déploie un compte de stockage blob Azure et s’abonne aux événements pour ce compte de stockage. Il envoie les événements vers un webhook. |
| [Vue d’ensemble : Réaction aux événements de Stockage Blob](../storage/blobs/storage-blob-event-overview.md) | Vue d’ensemble de l’intégration du stockage Blob à Event Grid. |

## <a name="next-steps"></a>Étapes suivantes

* Pour une présentation d’Azure Event Grid, consultez [Présentation d’Event Grid](overview.md).
* Pour plus d’informations sur la création d’un abonnement Azure Event Grid, consultez [Schéma d’abonnement à Event Grid](subscription-creation-schema.md).
* Pour plus d’informations sur l’utilisation des événements de stockage Blob, consultez [Itinéraire d’événements de stockage Blob - Azure CLI](../storage/blobs/storage-blob-event-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json). 

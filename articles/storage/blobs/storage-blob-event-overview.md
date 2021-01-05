---
title: Réaction aux événements Stockage Blob Azure (préversion) | Microsoft Docs
description: Utilisez Azure Event Grid pour vous abonner et réagir à des événements de stockage Blob. Comprendre le modèle d’événement, le filtrage des événements et les pratiques pour la consommation des événements.
author: normesta
ms.author: normesta
ms.date: 04/06/2020
ms.topic: conceptual
ms.service: storage
ms.subservice: blobs
ms.reviewer: dineshm
ms.openlocfilehash: 00a7a081f29458ae81d8d8ea4dd8f7abef42f78f
ms.sourcegitcommit: 84e3db454ad2bccf529dabba518558bd28e2a4e6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96519006"
---
# <a name="reacting-to-blob-storage-events"></a>Réaction aux événements de Stockage Blob

Les événements de stockage Azure permettent aux applications de réagir à des événements, tels que la création et la suppression d’objets. et sans qu’il soit nécessaire de faire appel à du code complexe ou à des services d’interrogation coûteux et inefficaces. L’avantage est que vous paierez uniquement pour ce que vous utiliserez.

Les événements Stockage Blob sont envoyés (push) avec [Azure Event Grid](https://azure.microsoft.com/services/event-grid/) aux abonnés comme Azure Functions, Azure Logic Apps, ou même à votre propre écouteur http personnalisé. Event Grid fournit des services de livraison d’événements fiables à vos applications par le biais de stratégies enrichies de nouvelle tentative et de livraison de lettres mortes.

Pour voir la liste complète des événements pris en charge par Stockage Blob, consultez l’article [Schéma des événements Stockage Blob](../../event-grid/event-schema-blob-storage.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

Les scénarios d’événements de stockage d’objets Blob courants incluent le traitement d’images ou de vidéos, l’indexation pour la recherche ou n’importe quel flux de travail orienté fichier. Les téléchargements de fichier asynchrones sont parfaitement adaptés aux événements. Lorsque les modifications sont peu fréquentes, mais que votre scénario requiert une réactivité immédiate, une architecture basée sur des événements peut être particulièrement efficace.

Si vous souhaitez essayer les événements Stockage Blob, consultez l’un des articles de démarrage rapide suivants :

|Si vous souhaitez utiliser cet outil :    |Lisez l’article : |
|--|-|
|Portail Azure    |[Démarrage rapide : Acheminer des événements de stockage Blob vers un point de terminaison web avec le portail Azure](../../event-grid/blob-event-quickstart-portal.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)|
|PowerShell    |[Démarrage rapide : Acheminer des événements de stockage vers un point de terminaison web avec PowerShell](./storage-blob-event-quickstart-powershell.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)|
|Azure CLI    |[Démarrage rapide : Acheminer des événements de stockage vers un point de terminaison web avec Azure CLI](./storage-blob-event-quickstart.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)|

Pour consulter des exemples détaillés de réaction aux événements de stockage Blob à l’aide de fonctions Azure, consultez les articles suivants :

- [Tutoriel : Utiliser des événements Azure Data Lake Storage Gen2 pour mettre à jour une table Databricks Delta](data-lake-storage-events.md).
- [Tutoriel : Automatiser le redimensionnement des images chargées à l’aide d’Event Grid](../../event-grid/resize-images-on-storage-blob-upload-event.md?tabs=dotnet)

>[!NOTE]
> Seuls les comptes de stockage de type **StorageV2 (v2 universel)** , **BlockBlobStorage** et **BlobStorage** prennent en charge l’intégration d’événements. Le type **Stockage (v1 universel)** ne prend *pas* en charge l’intégration à Event Grid.

## <a name="the-event-model"></a>Le modèle d’événement

Event Grid utilise les [abonnements aux événements](../../event-grid/concepts.md#event-subscriptions) pour acheminer les messages d’événements vers les abonnés. Cette image illustre la relation entre les éditeurs d’événements, les abonnements aux événements et les gestionnaires d’événements.

![Modèle de Event Grid](./media/storage-blob-event-overview/event-grid-functional-model.png)

Tout d’abord, abonnez un point de terminaison à un événement. Ensuite, lorsqu’un événement est déclenché, le service Event Grid envoie les données relatives à cet événement au point de terminaison.

Consultez l’article [Schéma d’événements de stockage Blob](../../event-grid/event-schema-blob-storage.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) pour obtenir :

> [!div class="checklist"]
> * Une liste complète des événements de stockage Blob et savoir comment chaque événement est déclenché.
> * Un exemple des données que Event Grid peut envoyer pour chacun de ces événements.
> * L’objectif de chaque paire de clé-valeur qui apparaît dans les données.

## <a name="filtering-events"></a>Filtrage des événements

Les [événements d’objets BLOB peuvent être filtrés](/cli/azure/eventgrid/event-subscription) par le type d’événement, le nom du conteneur ou le nom de l’objet qui a été créé/supprimé. Les filtres dans Event Grid correspondent au début ou à la fin de l’objet, de sorte que les événements avec un objet correspondant sont dirigés vers l’abonné.

Pour en savoir plus sur l’application de filtres, consultez [Filtrer des événements pour Event Grid](../../event-grid/how-to-filter-events.md).

Le sujet des événements de stockage d’objets Blob utilise le format suivant :

```
/blobServices/default/containers/<containername>/blobs/<blobname>
```

Pour faire correspondre tous les événements pour un compte de stockage, vous pouvez laisser les filtres de sujet vides.

Pour faire correspondre les événements à partir d’objets Blob créés dans un ensemble de conteneurs partageant un préfixe, utilisez un filtre `subjectBeginsWith`, comme :

```
/blobServices/default/containers/containerprefix
```

Pour faire correspondre les événements d’objet Blob créés dans un conteneur spécifique, utilisez un filtre `subjectBeginsWith`, comme :

```
/blobServices/default/containers/containername/
```

Pour faire correspondre les événements d’objet Blob créés dans un conteneur spécifique partageant un préfixe de nom d’objet Blob, utilisez un filtre `subjectBeginsWith`, comme :

```
/blobServices/default/containers/containername/blobs/blobprefix
```

Pour faire correspondre les événements d’objet Blob créés dans un conteneur spécifique partageant un suffixe d’objet Blob, utilisez un filtre `subjectEndsWith`, comme « .log » ou « .jpg ». Pour plus d’informations, consultez [Concepts d’Event Grid](../../event-grid/concepts.md#event-subscriptions).

## <a name="practices-for-consuming-events"></a>Pratiques pour la consommation d’événements

Les applications qui gèrent des événements de stockage d’objets Blob doivent suivre certaines pratiques recommandées :
> [!div class="checklist"]
> * Comme plusieurs abonnements peuvent être configurés pour acheminer les événements vers le même gestionnaire d’événements, il est important de ne pas considérer que les événements proviennent d’une source particulière, mais de vérifier le sujet du message pour vous assurer qu’il provient d’un compte de stockage que vous attendez.
> * De même, vérifiez que vous êtes prêt à traiter son eventType, et ne supposez pas que tous les événements reçus seront aux types que vous attendez.
> * Les messages pouvant arriver après un certain temps, utilisez les champs etag pour comprendre si vos informations sur les objets sont toujours à jour. Pour savoir comment utiliser le champ etag, consultez [Gestion de l’accès concurrentiel dans le Stockage Blob](../common/storage-concurrency.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#managing-concurrency-in-blob-storage).
> * Les messages pouvant arriver dans le désordre, utilisez les champs de séquence pour comprendre l’ordre des événements sur un objet particulier. Le champ de séquence est une valeur de chaîne qui représente l’ordre logique des événements pour n’importe quel nom d’objet blob. Vous pouvez utiliser la comparaison de chaînes standard pour comprendre l’ordre relatif de deux événements sur le même nom d’objet blob.
> * Les événements de stockage garantissent au moins une livraison aux abonnés, assurant ainsi la sortie de tous les messages. Toutefois, en raison de nouvelles tentatives ou de la disponibilité des abonnements, des messages dupliqués peuvent parfois se produire. Pour en savoir plus sur la livraison de messages et les nouvelles tentatives, consultez [Livraison et nouvelle tentative de livraison de messages avec Event Grid](../../event-grid/delivery-and-retry.md).
> * Utilisez le champ blobType pour comprendre le type d’opération autorisé sur l’objet Blob, et les types de bibliothèque client que vous devez utiliser pour accéder à l’objet Blob. Les valeurs valides sont `BlockBlob` ou `PageBlob`. 
> * Utilisez le champ URL avec les constructeurs `CloudBlockBlob` et `CloudAppendBlob` pour accéder à l’objet Blob.
> * Ignorez les champs que vous ne comprenez pas. Cette pratique vous aidera à prendre en charge les nouvelles fonctionnalités qui peuvent être ajoutées à l’avenir.
> * Si vous souhaitez vous assurer que l’événement **Microsoft.Storage.BlobCreated** n’est déclenché que lorsqu’un objet blob de blocs est entièrement validé, filtrez l’événement pour les appels d’API REST `CopyBlob`, `PutBlob`, `PutBlockList` ou `FlushWithClose`. Ces appels d’API déclenchent l’événement **Microsoft.Storage.BlobCreated** uniquement quand les données sont entièrement validées dans un objet blob de blocs. Pour savoir comment créer un filtre, consultez [Filtrer des événements pour Event Grid](../../event-grid/how-to-filter-events.md).


## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur Event Grid et essayer les événements de stockage d’objets Blob :

- [À propos d’Event Grid](../../event-grid/overview.md)
- [Schéma des événements Stockage Blob](../../event-grid/event-schema-blob-storage.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
- [Acheminer des événements de stockage Blob Azure vers un point de terminaison Web personnalisé ](storage-blob-event-quickstart.md)
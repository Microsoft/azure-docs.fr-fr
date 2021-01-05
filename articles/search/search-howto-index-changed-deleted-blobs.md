---
title: Blobs modifiés et supprimés
titleSuffix: Azure Cognitive Search
description: Après la création d’index de recherche initial qui importe à partir du Stockage Blob Azure, l’indexation suivante peut récupérer uniquement les blobs modifiés ou supprimés. Cet article explique cela en détail.
manager: nitinme
author: MarkHeff
ms.author: maheff
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 09/25/2020
ms.openlocfilehash: 2e73039418233c97fc20242ed7af7df14c5b47ee
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91534775"
---
# <a name="how-to-set-up-change-and-deletion-detection-for-blobs-in-azure-cognitive-search-indexing"></a>Comment configurer la détection des modifications et des suppressions de blobs dans l’indexation de Recherche cognitive Azure

Après la création d’un index de recherche initial, vous pouvez configurer des travaux subséquents de l’indexeur afin de récupérer uniquement les documents qui ont été créés ou supprimés depuis l’exécution initiale. Pour le contenu de recherche en provenance du Stockage Blob Azure, la détection des modifications se produit automatiquement lorsque vous utilisez une planification pour déclencher l’indexation. Par défaut, le service réindexe uniquement les blobs modifiés, déterminés par l’horodateur `LastModified`. Contrairement à d’autres sources de données prises en charge par les indexeurs de recherche, les blobs ont toujours un horodateur, ce qui évite d’avoir à configurer manuellement une stratégie de détection des modifications.

Bien que la détection des modifications soit acquise, la détection des suppressions ne l’est pas. Si vous souhaitez détecter des documents supprimés, veillez à adopter une approche de « suppression réversible ». Si vous supprimez complètement les objets blob, les documents correspondants ne seront pas supprimés de l’index de recherche.

Il existe deux façons d’implémenter l’approche de suppression réversible. Les deux sont décrites ci-dessous.

## <a name="native-blob-soft-delete-preview"></a>Suppression réversible native de blobs (préversion)

> [!IMPORTANT]
> La prise en charge de la suppression réversible native de blobs est disponible en préversion. Les fonctionnalités en préversion sont fournies sans contrat de niveau de service et ne sont pas recommandées pour les charges de travail de production. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). L’[API REST version 2020-06-30-Preview](./search-api-preview.md) fournit cette fonctionnalité. Il n’y a actuellement pas de prise en charge du portail ou du SDK .NET.

> [!NOTE]
> Lors de l’utilisation de la stratégie de suppression réversible native de blobs, les clés de document des documents de votre index doivent être une propriété blob ou des métadonnées blob.

Dans cette méthode, vous allez utiliser la fonctionnalité de [suppression réversible native de blobs](../storage/blobs/soft-delete-blob-overview.md) offerte par le stockage Blob Azure. Si la suppression réversible native de blobs est activée sur votre compte de stockage, votre source de données a un jeu natif de stratégies de suppression réversible, et si l’indexeur trouve un blob qui a été transféré à un état Supprimé de manière réversible, l’indexeur supprime ce document de l’index. La stratégie de suppression réversible native de blobs n’est pas prise en charge lors de l’indexation de blobs à partir d’Azure Data Lake Storage Gen2.

Utiliser les étapes suivantes :

1. Activez la [suppression réversible native pour le stockage Blob Azure](../storage/blobs/soft-delete-blob-overview.md). Nous vous recommandons de définir la stratégie de rétention sur une valeur bien supérieure à celle de la planification d’intervalle de votre indexeur. Ainsi, en cas de problème lors de l’exécution de l’indexeur ou si vous avez un grand nombre de documents à indexer, l’indexeur a tout le temps nécessaire pour traiter les blobs supprimés de manière réversible. Les indexeurs Recherche cognitive Azure suppriment un document de l’index uniquement s’ils traitent le blob alors que son état est Supprimé de manière réversible.

1. Configurez une stratégie de détection des suppressions réversibles natives de blobs sur la source de données. Voici un exemple. Étant donné qu’il s’agit d’une fonctionnalité d’évaluation, vous devez utiliser l’API REST en préversion.

1. Exécutez l’indexeur ou configurez l’indexeur pour qu’il s’exécute selon une planification. Lorsque l’indexeur s’exécute et traite le blob, le document est supprimé de l’index.

    ```http
    PUT https://[service name].search.windows.net/datasources/blob-datasource?api-version=2020-06-30-Preview
    Content-Type: application/json
    api-key: [admin key]
    {
        "name" : "blob-datasource",
        "type" : "azureblob",
        "credentials" : { "connectionString" : "<your storage connection string>" },
        "container" : { "name" : "my-container", "query" : null },
        "dataDeletionDetectionPolicy" : {
            "@odata.type" :"#Microsoft.Azure.Search.NativeBlobSoftDeleteDeletionDetectionPolicy"
        }
    }
    ```

### <a name="reindexing-un-deleted-blobs-using-native-soft-delete-policies"></a>Réindexation de blobs dont le suppression a été annulée (à l’aide de stratégies de suppression réversible natives)

Si vous supprimez un blob du Stockage Blob Azure à l’aide de la suppression réversible native activée sur votre compte de stockage, le blob passe à l’état supprimé de manière réversible, vous donnant la possibilité d’annuler sa suppression pendant la période de rétention. Si vous annulez une suppression après que l’indexeur l’a traitée, l’indexeur n’indexera pas toujours le blob restauré. Cela est dû au fait que l’indexeur détermine les blobs à indexer en fonction du timestamp `LastModified` du blob. Lorsque la suppression de manière réversible d’un blob est annulée, son horodateur `LastModified` n’est pas mis à jour. Par conséquent, si l’indexeur a déjà traité des blobs dont les horodateurs `LastModified` sont plus récents, il ne réindexe pas le blob dont la suppression a été annulée. 

Pour vous assurer qu’un blob dont la suppression a été annulée est réindexé, vous devez mettre à jour l’horodateur `LastModified` du blob. Pour ce faire, vous pouvez réenregistrer les métadonnées de ce blob. Vous n’avez pas besoin de modifier les métadonnées, mais le réenregistrement de celles-ci a pour effet de mettre à jour l’horodateur `LastModified` du blob de sorte que l’indexeur sait qu’il doit le réindexer.

## <a name="soft-delete-using-custom-metadata"></a>Suppression réversible à l’aide de métadonnées personnalisées

Dans cette méthode, vous utiliserez les métadonnées d’un blob pour indiquer à quel moment un document doit être supprimé de l’index de recherche. Cette méthode requiert deux actions distinctes, la suppression du document de recherche de l’index, puis la suppression du blob dans le service Stockage Azure.

Utiliser les étapes suivantes :

1. Ajoutez une paire clé-valeur de métadonnées personnalisées au blob pour indiquer à Recherche cognitive Azure qu’il est logiquement supprimé.

1. Configurez une stratégie de détection des colonnes de suppression réversible sur la source de données. Voici un exemple.

1. Une fois que l’indexeur a traité le blob et supprimé le document de l’index, vous pouvez supprimer le blob du Stockage Blob Azure.

Par exemple, la stratégie suivante considère qu’un objet blob est supprimé s’il présente une propriété de métadonnées `IsDeleted` avec la valeur `true` :

```http
    PUT https://[service name].search.windows.net/datasources/blob-datasource?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
        "name" : "blob-datasource",
        "type" : "azureblob",
        "credentials" : { "connectionString" : "<your storage connection string>" },
        "container" : { "name" : "my-container", "query" : null },
        "dataDeletionDetectionPolicy" : {
            "@odata.type" :"#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",
            "softDeleteColumnName" : "IsDeleted",
            "softDeleteMarkerValue" : "true"
        }
    }
```

### <a name="reindexing-un-deleted-blobs-using-custom-metadata"></a>Réindexation de blobs dont la suppression a été annulée (à l’aide de métadonnées personnalisées)

Une fois qu’un indexeur a traité un blob supprimé, et supprimé de l’index le document de recherche correspondant, il ne réexaminera pas ce blob en cas de restauration ultérieure si l’horodateur `LastModified` du blob est antérieur à la dernière exécution de l’indexeur.

Si vous souhaitez réindexer ce document, modifiez la valeur `"softDeleteMarkerValue" : "false"` pour ce blob, puis réexécutez l’indexeur.

## <a name="help-us-make-azure-cognitive-search-better"></a>Aidez-nous à améliorer Recherche cognitive Azure

Si vous avez des suggestions de fonctionnalités ou des idées d’amélioration, faites-le-nous savoir [UserVoice](https://feedback.azure.com/forums/263029-azure-search/). Si vous avez besoin d’aide pour utiliser la fonctionnalité existante, publiez votre question sur [Stack Overflow](https://stackoverflow.microsoft.com/questions/tagged/18870).

## <a name="next-steps"></a>Étapes suivantes

* [Indexeurs dans Recherche cognitive Azure](search-indexer-overview.md)
* [Comment configurer un indexeur de blobs](search-howto-indexing-azure-blob-storage.md)
* [Vue d’ensemble de l’indexation de blobs](search-blob-storage-integration.md)
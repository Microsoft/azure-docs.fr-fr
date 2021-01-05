---
title: Personnaliser un modèle de personne avec l’API Video Indexer
titleSuffix: Azure Media Services
description: Découvrez comment personnaliser un modèle de personne avec l’API Video Indexer.
services: media-services
author: anikaz
manager: johndeu
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 01/14/2020
ms.author: kumud
ms.openlocfilehash: 09366dea1a0d77052b6f99e9f5ab52c270e341b4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87047017"
---
# <a name="customize-a-person-model-with-the-video-indexer-api"></a>Personnaliser un modèle de personne avec l’API Video Indexer

Video Indexer prend en charge la détection de visage et la reconnaissance de célébrités dans du contenu vidéo. La fonctionnalité de reconnaissance de célébrités couvre environ un million de visages en fonction de la source de données couramment demandée comme IMDB, Wikipédia et les principaux influenceurs LinkedIn. Les visages qui ne sont pas reconnus par la fonctionnalité de reconnaissance de célébrités sont détectés, mais sont laissés sans nom. Après avoir chargé votre vidéo sur Video Indexer et obtenu des résultats, vous pouvez revenir en arrière et nommer les visages qui n’ont pas été reconnus. Une fois que vous étiquetez un visage avec un nom, le visage et le nom sont ajoutés au modèle de personne de votre compte. Video Indexer peut alors reconnaître ce visage dans vos vidéos, futures et anciennes.

Vous pouvez utiliser l’API Video Indexer pour modifier des visages détectés dans une vidéo, comme décrit dans cette rubrique. Vous pouvez également utiliser le site web Video Indexer, comme décrit dans [Personnaliser un modèle de personne avec le site web Video Indexer](customize-person-model-with-api.md).

## <a name="managing-multiple-person-models"></a>Gestion de plusieurs modèles de personne

Video Indexer prend en charge plusieurs modèles de personne par compte. Cette fonctionnalité est actuellement disponible uniquement via les API Video Indexer.

Si votre compte est adapté à différents scénarios d’utilisation, vous souhaiterez peut-être créer plusieurs modèles de personne par compte. Par exemple, si votre contenu est lié au sport, vous pouvez créer un modèle de personne distinct par sport (football, basketball, soccer, etc.).

Une fois un modèle créé, vous pouvez l’utiliser en fournissant l’ID d’un modèle de personne spécifique lors du chargement /de l’indexation ou de la réindexation d’une vidéo. La formation du modèle sur un nouveau visage dans une vidéo met à jour le modèle personnalisé spécifique auquel la vidéo était associée.

Chaque compte possède une limite de 50 modèles de personne. Si vous n’avez pas besoin de la prise en charge de plusieurs modèles de personne, n’affectez aucun ID de modèle de personne à votre vidéo lors de son chargement/indexation ou de sa réindexation. Dans ce cas, Video Indexer utilise le modèle de personne personnalisé par défaut dans votre compte.

## <a name="create-a-new-person-model"></a>Créer un nouveau modèle de personne

Pour créer un nouveau modèle de personne sur le compte spécifié, utilisez l'API [Créer un modèle de personne](https://api-portal.videoindexer.ai/docs/services/operations/operations/Create-Person-Model?).

La réponse fournit le nom et l’ID généré du modèle de personne que vous venez de créer au format de l’exemple ci-dessous.

```json
{
    "id": "227654b4-912c-4b92-ba4f-641d488e3720",
    "name": "Example Person Model"
}
```

Utilisez ensuite la valeur **id** pour le paramètre **personModelId** lors du [chargement d’une vidéo à indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Upload-video?) ou de la [réindexation d’une vidéo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Re-index-video?).

## <a name="delete-a-person-model"></a>Supprimer un modèle de personne

Pour supprimer un modèle de personne personnalisé sur le compte spécifié, utilisez l'API [Supprimer un modèle de personne](https://api-portal.videoindexer.ai/docs/services/operations/operations/Delete-Person-Model?).

Une fois le modèle de personne supprimé, l’index des vidéos actuelles utilisant le modèle supprimé restera inchangé jusqu’à ce que vous les réindexiez. Au moment de la réindexation, les visages nommés dans le modèle supprimé ne sont pas reconnus par Video Indexer dans vos vidéos actuelles indexées à l’aide de ce modèle, mais les visages sont tout de même détectés. Les vidéos actuelles indexées à l’aide du modèle supprimé utiliseront désormais le modèle de personne par défaut de votre compte. Si des faces du modèle supprimé sont également nommées dans le modèle par défaut de votre compte, ces visages continueront d’être reconnus dans les vidéos.

Aucun contenu n’est retourné en cas de suppression effective du modèle de personne.

## <a name="get-all-person-models"></a>Obtenir tous les modèles de personne

Pour obtenir tous les modèles de personne sur le compte spécifié, utilisez l'API [Obtenir un modèle de personne](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Person-Models?).

La réponse fournit une liste de tous les modèles de personne de votre compte (dont le modèle de personne par défaut du compte spécifié), ainsi que le nom et l’ID de chacun au format de l’exemple ci-dessous.

```json
[
    {
        "id": "59f9c326-b141-4515-abe7-7d822518571f",
        "name": "Default"
    }, 
    {
        "id": "9ef2632d-310a-4510-92e1-cc70ae0230d4",
        "name": "Test"
    }
]
```

Vous pouvez choisir le modèle que vous souhaitez utiliser pour une vidéo en utilisant la valeur `id` du modèle de personne pour le paramètre `personModelId` lors du [chargement d’une vidéo à indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Upload-video?) ou de la [réindexation d’une vidéo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Re-index-video?).

## <a name="update-a-face"></a>Mettre à jour un visage

Cette commande vous permet de mettre à jour un visage dans votre vidéo avec un nom à l’aide de l’ID de la vidéo et l’ID du visage. Cette action met ensuite à jour le modèle de personne auquel la vidéo était associée lors de son chargement/indexation ou de sa réindexation. Si aucun modèle de personne n’était attribué, elle met à jour le modèle de personne du compte par défaut.

Le système reconnaît alors les occurrences du même visage dans vos autres vidéos actuelles partageant le même modèle de personne. La reconnaissance du visage dans vos autres vidéos actuelles peut prendre un certain temps pour entrer en vigueur, car il s’agit d’un processus de traitement par lots.

Vous pouvez mettre à jour un visage que Video Indexer a reconnu comme une célébrité avec un nouveau nom. Le nouveau nom affecté est prioritaire sur la reconnaissance de célébrités intégrée.

Pour mettre à jour le visage, utilisez l'API [Mettre à jour un visage dans une vidéo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Update-Video-Face?).

Les noms étant uniques pour les modèles de personne, si vous affectez la même valeur de paramètre `name` à deux visages du même modèle de personne, Video Indexer identifie les visages comme appartenant à la même personne et les rapproche quand vous réindexez votre vidéo.

## <a name="next-steps"></a>Étapes suivantes

[Personnaliser un modèle de personne à l’aide du site web Video Indexer](customize-person-model-with-website.md)

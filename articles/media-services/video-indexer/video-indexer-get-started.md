---
title: S’inscrire à Video Indexer et charger votre première vidéo - Azure
titleSuffix: Azure Media Services
description: Découvrez comment s’inscrire et charger votre première vidéo à l’aide du portail Video Indexer.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: quickstart
ms.date: 10/30/2020
ms.author: juliako
ms.openlocfilehash: 7099b0d303965b987fb316170738cc82568dc6d6
ms.sourcegitcommit: 4bee52a3601b226cfc4e6eac71c1cb3b4b0eafe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94505595"
---
# <a name="quickstart-how-to-sign-up-and-upload-your-first-video"></a>Démarrage rapide : Comment s’inscrire et charger votre première vidéo

Ce guide de démarrage rapide vous montre comment vous connecter au site web Video Indexer, et comment charger votre première vidéo.

Lorsque vous créez un compte Video Indexer, vous pouvez choisir un compte d’essai gratuit (où vous obtenez un certain nombre de minutes d’indexation gratuites) ou une option payante (où vous n’êtes pas limités par le quota). Avec l’essai gratuit, Video Indexer fournit jusqu’à 600 heures d’indexation gratuite aux utilisateurs du site web et jusqu’à 2 400 heures d’indexation gratuite aux utilisateurs de l’API. Avec l’option payante, vous créez un compte Video Indexer [connecté à votre abonnement Azure et un compte Azure Media Services](connect-to-azure.md). Vous payez pour les minutes indexées ; pour plus d’informations, consultez [Tarifs Media Services](https://azure.microsoft.com/pricing/details/media-services/). 

## <a name="sign-up-for-video-indexer"></a>S’inscrire à Video Indexer

Pour commencer à développer avec Video Indexer, accédez au site web [Video Indexer](https://www.videoindexer.ai/) et inscrivez-vous.

Une fois que vous avez commencé à utiliser Video Indexer, l’ensemble de vos données stockées et du contenu chargé fait l’objet d’un chiffrement à clé gérée (au repos) par Microsoft.

> [!NOTE]
> Prenez connaissance des [changements planifiés relatifs à l’authentification sur le site web Video Indexer](release-notes.md#planned-video-indexer-website-authenticatication-changes).

## <a name="upload-a-video-using-the-video-indexer-website"></a>Charger une vidéo à l’aide du site web Video Indexer

### <a name="supported-file-formats-for-video-indexer"></a>Formats de fichiers pris en charge pour Video Indexer

Pour obtenir la liste des formats de fichiers que vous pouvez utiliser avec Video Indexer, consultez l’article [conteneur d’entrée/formats de fichiers](../latest/media-encoder-standard-formats.md#input-containerfile-formats).

### <a name="upload-a-video"></a>Charger une vidéo

1. Connectez-vous au site web [Video Indexer](https://www.videoindexer.ai/).
1. Pour charger une vidéo, appuyez sur le bouton ou le lien **Charger**.

    > [!NOTE]
    > Le nom de la vidéo ne doit pas dépasser 80 caractères.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/video-indexer-get-started/video-indexer-upload.png" alt-text="Charger":::
1. Une fois votre vidéo chargée, Video Indexer démarre l’indexation et l’analyse. La progression s’affiche. 

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/video-indexer-get-started/progress.png" alt-text="Progression du chargement":::
1. Au terme de l’analyse réalisée par Video Indexer, vous recevrez un e-mail contenant un lien vers votre vidéo et une brève description du contenu de celle-ci. Par exemple : personnes, mots prononcés et écrits, rubriques et entités nommées.
1. Vous pouvez retrouver ultérieurement votre vidéo dans la liste des bibliothèques et effectuer différentes opérations. Par exemple : recherche, réindexation, modification.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/video-indexer-get-started/uploaded.png" alt-text="Chargement du téléchargement":::
 
## <a name="see-also"></a>Voir aussi

Pour plus d’informations, consultez [Charger et indexer des vidéos](upload-index-videos.md).

Une fois que vous avez chargé et indexé une vidéo, vous pouvez commencer à utiliser le [site web Video Indexer](video-indexer-view-edit.md) ou [Video Indexer Developer Portal](video-indexer-use-apis.md) pour voir les insights de la vidéo. 

[Commencer à utiliser des API](video-indexer-use-apis.md)

## <a name="next-steps"></a>Étapes suivantes

Pour avoir une présentation détaillée, accédez à notre [lab de présentation](https://github.com/Azure-Samples/media-services-video-indexer/blob/master/IntroToVideoIndexer.md). 

Au terme de l’atelier, vous aurez une bonne compréhension du genre d’informations qui peuvent être extraites des contenus vidéo et audio, et vous serez plus à même d’identifier les opportunités de l’intelligence de contenu, de présenter l’intelligence artificielle vidéo sur Azure et d’expérimenter plusieurs scénarios sur Video Indexer.


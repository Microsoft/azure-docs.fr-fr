---
title: Rechercher des moments précis dans des vidéos avec Video Indexer
titleSuffix: Azure Media Services
description: Découvrez comment rechercher des moments précis dans des vidéos avec Video Indexer.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 11/23/2019
ms.author: juliako
ms.openlocfilehash: a1ca7c677e5f1eb55d74cca45e757676674f303c
ms.sourcegitcommit: 2e9643d74eb9e1357bc7c6b2bca14dbdd9faa436
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96030536"
---
# <a name="search-for-exact-moments-in-videos-with-video-indexer"></a>Rechercher des moments précis dans des vidéos avec Video Indexer

Cette rubrique montre comment utiliser le site web Video Indexer pour rechercher des moments précis dans des vidéos.

1. Accédez au site web [Video Indexer](https://www.videoindexer.ai/), puis connectez-vous.
1. Spécifiez les mots clés de recherche, et la recherche sera effectuée parmi toutes les vidéos de la bibliothèque de votre compte. 

    Vous pouvez filtrer votre recherche en sélectionnant **Filtres**. Dans l’exemple ci-dessous, nous effectuons une recherche sur « Microsoft » qui s’affiche sous forme de texte à l’écran uniquement (OCR).

    :::image type="content" source="./media/video-indexer-search/filter.png" alt-text="Filtrer, texte uniquement":::
1. Appuyez sur **Search** pour afficher le résultat.

    :::image type="content" source="./media/video-indexer-search/results.png" alt-text="Résultats de recherche de vidéos":::

    Si vous sélectionnez l’un des résultats, le lecteur vous amène à ce moments précis de la vidéo.
1. Affichez et recherchez les insights résumés de la vidéo en cliquant sur **Lecture** dans la vidéo ou en sélectionnant l’un des résultats de la recherche d’origine. 

    Vous pouvez afficher, rechercher et modifier les **insights**. Lorsque vous sélectionnez l’un des insights, le lecteur vous amène à ce moments précis de la vidéo.  

    :::image type="content" source="./media/video-indexer-search/insights.png" alt-text="Afficher, rechercher et modifier les insights de la vidéo":::

    Si vous incorporez la vidéo par le biais de widgets Video Indexer, vous pouvez obtenir la vue Lecteur/Insights et la synchronisation dans votre application. Pour plus d’informations, consultez [Incorporer des widgets Video Indexer dans votre application](video-indexer-embed-widgets.md).
1. Vous pouvez afficher, rechercher et modifier les transcriptions en cliquant sur l’onglet **Chronologie**. 

    :::image type="content" source="./media/video-indexer-search/timeline.png" alt-text="Afficher, rechercher et modifier les transcriptions de la vidéo":::

    Pour modifier le texte, sélectionnez **Modifier** en haut à droite et modifiez le texte comme vous le souhaitez. 

    Vous pouvez également traduire et télécharger les transcriptions en sélectionnant l’option appropriée dans l’angle supérieur droit. 

## <a name="embed-download-create-projects"></a>Incorporer, télécharger, créer des projets

Vous pouvez incorporer votre vidéo en sélectionnant **</> Incorporer** sous votre vidéo. Pour plus d’informations, consultez [Incorporer des widgets visuels dans votre application](video-indexer-embed-widgets.md).

Vous pouvez télécharger la vidéo source, les insights de la vidéo et les transcriptions en cliquant sur **Télécharger** sous votre vidéo.

Vous pouvez créer un clip basé sur votre vidéo avec des lignes et des moments spécifiques en cliquant sur **Ouvrir dans l’éditeur**, en modifiant la vidéo, puis en enregistrant le projet. Pour plus d’informations, consultez [Ajouter des clips vidéo à votre projet](use-editor-create-project.md).

:::image type="content" source="./media/video-indexer-search/embed-download-create-projects.png" alt-text="Incorporer, télécharger, créer des projets de la vidéo":::

## <a name="next-steps"></a>Étapes suivantes

[Traiter du contenu avec l’API REST de Video Indexer](video-indexer-use-apis.md)

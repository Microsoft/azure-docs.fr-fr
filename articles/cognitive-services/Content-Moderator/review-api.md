---
title: Concepts des révisions, flux de travail et travaux - Content Moderator
titleSuffix: Azure Cognitive Services
description: 'Dans cet article, vous découvrirez les concepts fondamentaux de l’outil de révision : révisions, flux de travail et travaux.'
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 03/14/2019
ms.author: pafarley
ms.openlocfilehash: 43c39c40af6e02861211a8666fefa57c34072f32
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905194"
---
# <a name="content-moderation-reviews-workflows-and-jobs"></a>Révisions, flux de travail et travaux avec modération du contenu

Content Moderator combine la modération assistée par ordinateur et les capacités d’intervention humaine pour créer un processus de modération optimal adapté à des scénarios concrets. Il utilise pour cela l’[outil de révision](https://contentmoderator.cognitive.microsoft.com) dans le cloud. Dans ce guide, vous allez découvrir les concepts fondamentaux de l’outil de révision : révisions, workflows et travaux.

## <a name="reviews"></a>Révisions

Dans une révision, le contenu est chargé vers l’outil de révision. Vous pouvez l’afficher en cliquant sur son type de contenu sous l’onglet **Review** du tableau de bord. À partir de l’écran de révision, vous pouvez modifier les étiquettes appliquées et appliquer vos propres étiquettes personnalisées, le cas échéant. Lorsque vous soumettez une révision, les résultats sont envoyés à un point de terminaison de rappel spécifié, et le contenu est retiré du site.

> [!div class="mx-imgBorder"]
> ![Le menu déroulant Révision est en évidence. Il affiche les types de contenus suivants : image, texte et vidéo.](./Review-Tool-user-Guide/images/review-tab.png)

### <a name="manage-reviews"></a>Gérer les révisions

Dans le tableau de bord, accédez à **Admin** -> **Manage Reviews** pour afficher l’écran d’administration. Ici, vous pouvez voir une liste de toutes les révisions (en attente et terminées).

Le bouton **Actions** à trois points sur chaque révision vous permet d’accéder à l’écran de révision ou d’inspecter l’historique de cette révision.

> [!div class="mx-imgBorder"]
> ![Site de l’outil de révision, écran Review](./Review-Tool-user-Guide/images/manage-reviews.png)

Utilisez la barre d’outils **Search** pour trier les révisions d’après diverses catégories, telles que l’état de la révision, les étiquettes, le type de contenu, les sous-équipes, les utilisateurs affectés et la date de création/modification.

> [!div class="mx-imgBorder"]
> ![La barre d’outils Rechercher s’affiche. Elle comporte différentes zones de liste modifiable pour l’entrée de critères de recherche, tels que l’état de révision et les balises.](./Review-Tool-user-Guide/images/review-search.png)

Consultez le [guide de l’outil de révision](./review-tool-user-guide/review-moderated-images.md) pour commencer à créer des révisions, ou le [guide de la console d’API](./try-review-api-review.md) pour apprendre à le faire par programmation.

## <a name="workflows"></a>Workflows

Un flux de travail est un filtre personnalisé basé sur le cloud pour le contenu. Flux de travail peut se connecter à une variété de services pour filtrer le contenu de différentes façons et puis exécutez l’action appropriée. Avec le connecteur Content Moderator, un flux de travail peut automatiquement appliquer des balises de modération et créer révisions avec le contenu soumis.

### <a name="view-workflows"></a>Afficher des flux de travail

Pour afficher vos workflows existants, accédez à l’[outil de révision](https://contentmoderator.cognitive.microsoft.com/) et sélectionnez **Admin** > **Workflows**.

> [!div class="mx-imgBorder"]
> ![Flux de travail par défaut](images/default-workflow-list.png)

Les workflows sont définis en tant que chaînes JSON, ce qui les rend accessibles par programmation. Si vous choisissez l’option **Modifier** pour votre flux de travail, puis choisissez sélectionnez l’onglet **JSON**, une expression JSON similaire à ce qui suit s’affiche :

```json
{
    "Type": "Logic",
    "If": {
        "ConnectorName": "moderator",
        "OutputName": "isAdult",
        "Operator": "eq",
        "Value": "true",
        "Type": "Condition"
        },
    "Then": {
    "Perform": [
    {
        "Name": "createreview",
        "CallbackEndpoint": null,
        "Tags": []
    }
    ],
    "Type": "Actions"
    }
}
```

Consultez le [guide de l’outil de révision](./review-tool-user-guide/workflows.md) pour commencer à créer et à utiliser des flux de travail, ou le [guide de la console d’API](./try-review-api-workflow.md) pour apprendre à le faire par programmation.

## <a name="jobs"></a>travaux

Un travail de modération est une forme de wrapper des fonctionnalités de modération du contenu, de workflows et de révision. Le travail analyse votre contenu à l’aide de l’API Modération des images ou de l’API Modération du texte de Content Moderator, puis compare le résultat obtenu au flux de travail désigné. Selon les résultats de flux de travail, il crée ou non une révision pour le contenu dans l’[outil de révision](./review-tool-user-guide/human-in-the-loop.md). Même s’il est possible de créer et de configurer des révisions et des flux de travail avec leurs API respectives, l'API de travail vous permet d'obtenir un rapport détaillé de l'ensemble du processus (qui peut être envoyé à un point de terminaison de rappel spécifié).

Consultez le [guide de la console d’API](./try-review-api-job.md) pour commencer à utiliser des travaux.

## <a name="next-steps"></a>Étapes suivantes

* Testez la [console d’API Job](try-review-api-job.md) et utilisez les exemples de code API REST. Si vous connaissez déjà Visual Studio et C#, consultez aussi le [guide de démarrage des tâches .NET](moderation-jobs-quickstart-dotnet.md). 
* Pour les révisions, prenez en main la [console d’API Révision](try-review-api-review.md), et utilisez les exemples de code de l’API REST. Consultez ensuite les sections relatives aux révisions du [guide de démarrage rapide .NET](./client-libraries.md?pivots=programming-language-csharp%253fpivots%253dprogramming-language-csharp).
* Pour des révisions vidéo, utilisez le [guide de démarrage rapide pour révision vidéo](video-reviews-quickstart-dotnet.md), et apprenez à [ajouter des transcriptions à la révision vidéo](video-transcript-reviews-quickstart-dotnet.md).
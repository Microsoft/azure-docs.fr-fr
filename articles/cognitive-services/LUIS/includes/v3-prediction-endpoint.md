---
title: Comment obtenir un point de terminaison de prédiction V3
titleSuffix: Azure Cognitive Services
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: include
ms.date: 05/05/2020
ms.openlocfilehash: 85fcd0308083350a02afd4eadac2b361337f3b33
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93128159"
---
1. Dans le portail LUIS, dans la section **Gérer** (menu en haut à droite), dans la page **Ressources Azure** (menu de gauche), sous l’onglet **Ressources de prédiction** , copiez l’exemple de requête **Example Query** en bas de la page. L’URL contient votre ID d’application, votre clé et le nom de l’emplacement. L’URL du point de terminaison de prédiction V3 a la forme suivante : `https://YOUR-RESOURCE-NAME.api.cognitive.microsoft.com/luis/prediction/v3.0/apps/APP-ID/slots/SLOT-NAME/predict?subscription-key=YOUR-PREDICTION-KEY&<optional-name-value-pairs>&query=YOUR_QUERY_HERE`

    :::image type="content" source="../media/prediction-resources-example-query.png" alt-text="exemple de requête dans la section des ressources de prédiction" lightbox="../media/prediction-resources-example-query.png":::
    
    Collez l’URL dans un nouvel onglet de navigateur. Si vous ne voyez pas l’URL, vous n’avez pas de ressource de prédiction et devrez en créer une. 

    

    


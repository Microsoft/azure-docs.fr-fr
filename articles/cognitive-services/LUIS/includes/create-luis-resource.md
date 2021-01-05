---
title: Créer une ressource LUIS
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 11/20/2020
ms.author: aahi
ms.openlocfilehash: ee7fd384a198c5eff672b14b6cb479aac26cfe54
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95972508"
---
<a name="create-luis-resources"></a>

## <a name="create-luis-resources-in-the-azure-portal"></a>Créer des ressources LUIS sur le portail Azure

1. Utilisez [ce lien](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesLUISAllInOne) pour commencer à créer des ressources LUIS sur le portail Azure.

1. Entrez tous les paramètres obligatoires :

    |Nom|Objectif|
    |--|--|
    |Abonnement | Abonnement auquel la ressource sera facturée.|
    |Resource group| Nom de groupe de ressources personnalisé que vous choisissez ou créez. Les groupes de ressources vous permettent de regrouper des ressources Azure pour l’accès et la gestion.|
    |Nom| nom personnalisé que vous choisissez. Il est utilisé comme sous-domaine personnalisé pour vos requêtes de point de terminaison de création et de prédiction.|
    |Emplacement de création|Région associée à votre modèle.|
    |Niveau tarifaire de création|Détermine le nombre maximal de transactions par seconde et par mois.|
    |Lieu de prédiction|Région associée à votre runtime de point de terminaison de prédiction publié.|
    |Niveau tarifaire de prédiction|Détermine le nombre maximal de transactions par seconde et par mois.|

    > [!div class="mx-imgBorder"]
    > [![Capture d’écran montrant l’onglet Informations de base sous Créer.](../media/luis-how-to-azure-subscription/create-resource-in-azure-small.png)](../media/luis-how-to-azure-subscription/create-resource-in-azure-small.png#lightbox)

1. Sélectionnez **Vérifier + créer** et attendez que la ressource soit créée.
1. Une fois les deux ressources créées, toujours dans le portail Azure, sélectionnez la nouvelle ressource de création. Ensuite, sélectionnez **Clés et point de terminaison** pour obtenir l’**URL du point de terminaison** de création et la **clé** pour la création programmatique.

> [!TIP]
> Pour utiliser les ressources, sur le portail LUIS, [affectez-les](../luis-how-to-azure-subscription.md#assign-an-authoring-resource-in-the-luis-portal-for-all-apps).

---
title: Publier l'application - LUIS
titleSuffix: Azure Cognitive Services
description: Quand vous avez terminé la création et les tests de votre application LUIS active, mettez-la à disposition de votre application cliente en la publiant sur le point de terminaison.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 05/17/2020
ms.openlocfilehash: b72f1fd64cca0fa77ebc486670a512c5228e1146
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91541473"
---
# <a name="publish-your-active-trained-app-to-a-staging-or-production-endpoint"></a>Publier votre application active, formée sur un point de terminaison intermédiaire ou de production

Quand vous avez terminé la génération, l’entraînement et les tests de votre application LUIS active, mettez-la à disposition de votre application cliente en la publiant sur le point de terminaison.

## <a name="publishing"></a>Publication
1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Ouvrez votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).
1. Pour publier sur le point de terminaison, sélectionnez **Publish** (Publier) dans le volet en haut à droite.

    ![Bouton publier en haut, barre de navigation à droite](./media/luis-how-to-publish-app/publish-top-nav-bar.png)

1. Sélectionnez vos paramètres pour le point de terminaison de prédiction publié, puis sélectionnez **Publier**.

    ![Sélectionnez Paramètres de publication, puis le bouton Publier.](./media/luis-how-to-publish-app/publish-pop-up.png)

### <a name="publishing-slots"></a>Emplacements de publication

Sélectionnez l’emplacement correct quand la fenêtre contextuelle s’affiche :

* Staging
* Production

L’utilisation des deux emplacements de publication vous permet d’avoir deux versions différentes de votre application disponibles aux points de terminaison publiés, ou la même version sur deux points de terminaison différents.

### <a name="publishing-regions"></a>Régions de publication

L’application est publiée sur toutes les régions associées aux ressources de point de terminaison de prédiction LUIS ajoutées dans le portail LUIS à partir de la page **Gérer** ->  **[Ressources Azure](luis-how-to-azure-subscription.md#assign-a-resource-to-an-app)** .

Par exemple, pour une application créée sur [www.luis.ai](https://www.luis.ai), si vous créez une ressource LUIS dans deux régions **westus** et **eastus**, et ajoutez celles-ci à l’application en tant que ressources, l’application est publiée dans les deux régions. Pour plus d’informations sur les régions LUIS, consultez [Régions](luis-reference-regions.md).

> [!TIP]
> Il existe 3 régions de création. Vous devez créer dans la région sur laquelle vous souhaitez publier. Si vous devez publier sur toutes les régions, vous devez gérer votre processus de création et le modèle entraîné résultant dans l’ensemble des 3 régions de création.


## <a name="configuring-publish-settings"></a>Configuration des paramètres de publication

Après avoir sélectionné l’emplacement, configurez les paramètres de publication suivants :

* analyse de sentiments
* [Correction orthographique](luis-tutorial-bing-spellcheck.md) - point de terminaison de prédiction v2 uniquement
* Préparation vocale

Après publication, ces paramètres sont accessibles via la page **Paramètres de publication** de la section **Gérer**. Vous pouvez modifier les paramètres pour chaque publication. Si vous annulez une publication, les modifications que vous avez apportées lors de la publication sont également annulées.

### <a name="when-your-app-is-published"></a>Quand votre application est publiée

Une fois votre application correctement publiée, une notification de réussite s’affiche en haut du navigateur. La notification comprend également un lien vers les points de terminaison.

Si vous avez besoin de l’URL de point de terminaison, sélectionnez le lien. Vous pouvez également accéder aux URL de point de terminaison en sélectionnant **Gérer** dans le menu supérieur, puis **Ressources Azure** dans le menu gauche.

## <a name="sentiment-analysis"></a>analyse de sentiments

<a name="enable-sentiment-analysis"></a>

L’analyse des sentiments permet à LUIS de s’intégrer avec l’[Analyse de texte](https://azure.microsoft.com/services/cognitive-services/text-analytics/) pour fournir une analyse des sentiments et des expressions clés.

Vous ne devez pas nécessairement fournir une clé d’Analyse de texte, et votre compte Azure ne sera pas facturé pour ce service.

Les données de sentiment correspondent à un score compris entre 1 et 0 indiquant le sentiment, positif (plus proche de 1) ou négatif (plus proche de 0), des données. L’étiquette de sentiment de `positive`, `neutral` et `negative` est fonction de la culture prise en charge. Actuellement, seul l’anglais prend en charge les étiquettes de sentiment.

Pour plus d’informations sur la réponse du point de terminaison JSON avec l’analyse des sentiments, voir [Analyse des sentiments](luis-reference-prebuilt-sentiment.md).

## <a name="spelling-correction"></a>Correction orthographique

[!INCLUDE [Not supported in V3 API prediction endpoint](./includes/v2-support-only.md)]

Les corrections orthographiques sont effectuées avant la prédiction d’énoncé utilisateur LUIS. Vous pouvez voir toutes les modifications apportées à l’énoncé d’origine, y compris en lien avec l’orthographe, dans la réponse.

## <a name="speech-priming"></a>Préparation vocale

La préparation vocale est le processus d’envoi du modèle LUIS aux services Speech avant la synthèse vocale. Cela permet au service de fournir une conversion vocale plus précise pour votre modèle. Cela permet de faire passer les demandes du service Speech et les réponses du service LUIS dans un seul appel. La latence globale est ainsi réduite.

## <a name="next-steps"></a>Étapes suivantes

* Consultez [Gérer les clés](./luis-how-to-azure-subscription.md) pour ajouter des clés à la clé d’abonnement Azure dans LUIS et pour savoir comment définir la clé de vérification orthographique Bing et inclure toutes les intentions dans les résultats.
* Voir [Former et tester votre application](luis-interactive-test.md) pour obtenir des instructions sur la façon de tester votre application publiée dans la console de test.


---
title: Créer un compte Cognitive Services dans le portail Azure
titleSuffix: Azure Cognitive Services
description: Commencez à utiliser Azure Cognitive Services en créant et en vous abonnant à une ressource dans le Portail Azure.
services: cognitive-services
author: aahill
manager: nitinme
keywords: services cognitifs, intelligence cognitive, solutions cognitives, services ia
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 09/14/2020
ms.author: aahi
ms.openlocfilehash: 3fd1d9e708a5f1a500440f20c2947ddfe4bc7460
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94368914"
---
# <a name="quickstart-create-a-cognitive-services-resource-using-the-azure-portal"></a>Démarrage rapide : Créer un compte Cognitive Services dans le portail Azure

Ce guide de démarrage rapide vous montre comment commencer à utiliser Azure Cognitive Services. Après avoir créé une ressource Cognitive Services dans le portail Azure, vous obtenez un point de terminaison et une clé pour authentifier vos applications.

La solution Azure Cognitive Services correspond à des services cloud avec des API REST et des kits SDK de bibliothèque de client destinés à aider les développeurs à intégrer une intelligence cognitive dans des applications sans connaissances ni compétences directes en intelligence artificielle (IA) ou en science des données. Azure Cognitive Services permet aux développeurs d’ajouter facilement des fonctionnalités cognitives dans leurs applications à l’aide de solutions cognitives capables de voir, entendre, parler, comprendre et même commencer à raisonner.


[!INCLUDE [cognitive-services-subscription-types](../../includes/cognitive-services-subscription-types.md)]

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure valide - [Créer un abonnement gratuitement](https://azure.microsoft.com/free/cognitive-services/)

## <a name="create-a-new-azure-cognitive-services-resource"></a>Créer une ressource Azure Cognitive Services

1. Crée une ressource.

    #### <a name="multi-service-resource"></a>[Ressource multiservice](#tab/multiservice)

    La ressource multiservice est nommée **Cognitive Services** dans le portail. [Créez une ressource Cognitive Services](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAllInOne).

    À ce stade, la ressource multiservice permet d’accéder aux services Cognitive Services suivants :

    - Vision par ordinateur
    - Content Moderator
    - Face
    - Language Understanding (LUIS)
    - Analyse de texte
    - Convertisseur
    - Recherche Bing v7 <br>(Web, Image, Actualités, Vidéo, Visuel)
    - Recherche personnalisée Bing
    - Recherche d’entité Bing
    - Suggestion automatique Bing
    - Vérification orthographique Bing

    #### <a name="single-service-resource"></a>[Ressource monoservice](#tab/singleservice)

    Utilisez les liens ci-dessous afin de créer une ressource pour les services Cognitive Services disponibles :

    | Vision                      | Speech                  | Langage                          | Décision             | Recherche                 |
    |-----------------------------|-------------------------|-----------------------------------|----------------------|------------------------|
    | [Vision par ordinateur](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision)         | [Services Speech](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesSpeechServices)     | [Lecteur immersif](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesImmersiveReader)              | [Détecteur d’anomalies](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAnomalyDetector) | [API Recherche Bing V7](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesBingSearch-v7) |
    | [Service Custom Vision](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesCustomVision) | [Reconnaissance de l’orateur](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesSpeakerRecognition) | [Language Understanding (LUIS)](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesLUISAllInOne) | [Content Moderator](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesContentModerator) | [Recherche personnalisée Bing](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesBingCustomSearch) |
    | [Visage](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFace)                    |                         | [QnA Maker](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker)                     | [Personalizer](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesPersonalizer)     | [Recherche d’entités Bing](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesBingEntitySearch) |
    | [Ink Recognizer](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesInkRecognizer)        |                         | [Analyse de texte](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics)                |  [Metrics Advisor](https://go.microsoft.com/fwlink/?linkid=2142156)                    | [Vérification orthographique Bing](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesBingSpellCheck-v7)   |
    |           |                         | [Translator](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextTranslation)               |                      | [Suggestion automatique Bing](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesBingAutosuggest-v7)                       |

    **_

3. Dans la page _ *Créer**, renseignez les informations suivantes :

    #### <a name="multi-service-resource"></a>[Ressource multiservice](#tab/multiservice)

    |    |    |
    |--|--|
    | **Nom** | Nom descriptif de votre ressource Cognitive Services. Par exemple, *MyCognitiveServicesResource*. |
    | **Abonnement** | Sélectionnez l’un de vos abonnements Azure disponibles. |
    | **Lieu** | Emplacement de votre instance Cognitive Services. Des emplacements différents peuvent entraîner une latence. Toutefois, cela n’aura pas d’impact sur la disponibilité d’exécution de votre ressource. |
    | **Niveau tarifaire** | Le coût associé à votre compte Cognitive Services dépend des options que vous choisissez, ainsi que de votre utilisation. Pour plus d'informations, consultez le [détail des tarifs](https://azure.microsoft.com/pricing/details/cognitive-services/) de l’API.
    | **Groupe de ressources** | Groupe de ressources Azure comprenant votre ressource Cognitive Services. Vous pouvez créer un groupe ou l’ajouter à un groupe préexistant. |

    ![Écran Création de ressources pour les ressources multiservices](media/cognitive-services-apis-create-account/resource_create_screen-multi.png)

    Cliquez sur **Créer**.

    #### <a name="single-service-resource"></a>[Ressource monoservice](#tab/singleservice)

    |    |    |
    |--|--|
    | **Nom** | Nom descriptif de votre ressource Cognitive Services. Par exemple, *TextAnalyticsResource*. |
    | **Abonnement** | Sélectionnez l’un de vos abonnements Azure disponibles. |
    | **Lieu** | Emplacement de votre instance Cognitive Services. Des emplacements différents peuvent entraîner une latence. Toutefois, cela n’aura pas d’impact sur la disponibilité d’exécution de votre ressource. |
    | **Niveau tarifaire** | Le coût associé à votre compte Cognitive Services dépend des options que vous choisissez, ainsi que de votre utilisation. Pour plus d'informations, consultez le [détail des tarifs](https://azure.microsoft.com/pricing/details/cognitive-services/) de l’API.
    | **Groupe de ressources** | Groupe de ressources Azure comprenant votre ressource Cognitive Services. Vous pouvez créer un groupe ou l’ajouter à un groupe préexistant. |

    ![Écran Création de ressource pour la ressource d’un seul service](media/cognitive-services-apis-create-account/resource_create_screen.png)

    Cliquez sur **Créer**.

    **_

[!INCLUDE [Register Azure resource for subscription](./includes/register-resource-subscription.md)]

## <a name="get-the-keys-for-your-resource"></a>Obtenir les clés pour votre ressource

1. Une fois votre ressource déployée avec succès, cliquez sur _ *Accéder à la ressource** sous **Étapes suivantes**.

    ![Rechercher Cognitive Services](media/cognitive-services-apis-create-account/resource-next-steps.png)

2. Dans le volet de démarrage rapide qui s’ouvre, vous pouvez accéder à votre clé et à votre point de terminaison.

    ![Obtenir la clé et le point de terminaison](media/cognitive-services-apis-create-account/get-cog-serv-keys.png)

[!INCLUDE [cognitive-services-environment-variables](../../includes/cognitive-services-environment-variables.md)]

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez nettoyer et supprimer un abonnement Cognitive Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources supprime également toutes les autres ressources se trouvant dans le groupe.

1. Sur le portail Azure, développez le menu de gauche pour ouvrir le menu des services, et sélectionnez **Groupes de ressources** pour afficher la liste de vos groupes de ressources.
2. Localisez le groupe de ressources contenant la ressource à supprimer.
3. Cliquez avec le bouton droit sur la liste des groupes de ressources. Sélectionnez **Supprimer le groupe de ressources** et confirmez.

## <a name="see-also"></a>Voir aussi

* [Authentifier des requêtes auprès d’Azure Cognitive Services](authentication.md)
* [Qu’est-ce qu’Azure Cognitive Services ?](./what-are-cognitive-services.md)
* [Créer des ressources à l’aide de la bibliothèque de client du portail de gestion Microsoft Azure](.\cognitive-services-apis-create-account-client-library.md)
* [Prise en charge en langage naturel](language-support.md)
* [Prise en charge des conteneurs Docker](cognitive-services-container-support.md)
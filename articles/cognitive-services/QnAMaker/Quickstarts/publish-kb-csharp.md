---
title: 'Démarrage rapide : Publier une base de connaissances REST, C# – QnA Maker'
description: Ce démarrage rapide de C# basé sur REST publie votre base de connaissances et crée un point de terminaison qui peut être appelé dans votre application ou bot conversationnel.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.date: 02/08/2020
ROBOTS: NOINDEX,NOFOLLOW
ms.custom: RESTCURL2020FEB27, devx-track-csharp
ms.topic: how-to
ms.openlocfilehash: abaccfb1a1f36e1f7f47d3f5acefb08e2091cb39
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96351178"
---
# <a name="quickstart-publish-a-knowledge-base-in-qna-maker-using-c"></a>Démarrage rapide : Publier une base de connaissances dans QnA Maker à l'aide de C#

Ce guide de démarrage rapide basé sur REST vous aide à publier votre base de connaissances par programmation. La publication envoie la dernière version de la base de connaissances à un index de Recherche cognitive Azure dédié, et crée un point de terminaison pouvant être appelé dans votre application ou bot conversationnel.

Ce démarrage rapide fait appel aux API QnA Maker :
* [Publier](/rest/api/cognitiveservices/qnamaker/knowledgebase/publish) : cette API ne nécessite aucune information dans le corps de la requête.

## <a name="prerequisites"></a>Prérequis

* La dernière [**édition de Visual Studio Community**](https://www.visualstudio.com/downloads/).
* Vous devez disposer d’un [service QnA Maker](../How-To/set-up-qnamaker-service-azure.md). Pour récupérer votre clé et votre point de terminaison (qui incluent le nom de la ressource), sélectionnez **Démarrage rapide** pour votre ressource dans le portail Azure.
* ID de la base de connaissances QnA Maker figurant dans l’URL du paramètre de chaîne de requête `kbid`, comme illustré ci-dessous.

    ![ID de base de connaissances QnA Maker](../media/qnamaker-quickstart-kb/qna-maker-id.png)

    Si vous n’avez pas encore de base de connaissances, vous pouvez créer un exemple à utiliser pour ce démarrage rapide : [Créer une base de connaissances](create-new-kb-csharp.md).

> [!NOTE]
> Les fichiers solution complets sont disponibles dans le dépôt GitHub [**Azure-Samples/cognitive-services-qnamaker-csharp**](https://github.com/Azure-Samples/cognitive-services-qnamaker-csharp/tree/master/documentation-samples/quickstarts/publish-knowledge-base).

## <a name="create-knowledge-base-project"></a>Créer un projet de base de connaissances

1. Ouvrez Visual Studio 2019 Community Edition.
1. Créez un projet **Application console (.NET Core)** et nommez-le `QnaMakerQuickstart`. Acceptez les valeurs par défaut des autres paramètres.

## <a name="add-required-dependencies"></a>Ajouter les dépendances nécessaires

En haut de Program.cs, remplacez l’instruction unique using par les lignes suivantes pour ajouter les dépendances nécessaires au projet :

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/QnAMaker/rest/publish-kb.cs" id="dependencies":::

## <a name="add-required-constants"></a>Ajouter les constantes nécessaires

Dans la classe **Program**, ajoutez les constantes nécessaires pour accéder à QnA Maker.

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/QnAMaker/rest/publish-kb.cs" id="constants":::

## <a name="add-the-main-method-to-publish-the-knowledge-base"></a>Ajouter la méthode Main pour publier la base de connaissances

Après les constantes nécessaires, ajoutez le code suivant, qui adresse une requête HTTPS à l’API QnA Maker afin de publier une base de connaissances et reçoit la réponse :

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/QnAMaker/rest/publish-kb.cs" id="post":::

L’appel API renvoie un état 204 si la publication réussit sans aucun contenu dans le corps de la réponse.

## <a name="build-and-run-the-program"></a>Créez et exécutez le projet.

Générez et exécutez le programme. Il envoie automatiquement la requête à l’API QnA Maker visant à publier la base de connaissances, puis la réponse est affichée dans la fenêtre de console.

Une fois votre base de connaissances publiée, vous pouvez l’interroger à partir du point de terminaison à l’aide d’une application client ou d’un chatbot.

[!INCLUDE [Clean up files and knowledge base](../../../../includes/cognitive-services-qnamaker-quickstart-cleanup-resources.md)]

## <a name="next-steps"></a>Étapes suivantes

Une fois la base de connaissances publiée, il faut que l’[URL du point de terminaison génère une réponse](./get-answer-from-knowledge-base-csharp.md).

> [!div class="nextstepaction"]
> [Documentation de référence pour l’API REST QnA Maker (V4)](/rest/api/cognitiveservices/qnamaker4.0/knowledgebase)
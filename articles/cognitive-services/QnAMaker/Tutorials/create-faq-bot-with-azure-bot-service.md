---
title: 'Tutoriel : Créer un bot de forum aux questions avec Azure Bot Service'
description: Dans ce tutoriel, créez un bot de forum aux questions sans code avec QnA Maker et Azure Bot Service.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: tutorial
ms.date: 08/31/2020
ms.openlocfilehash: 356b954c65086ff65264ce7a62b038cd7a8eac3d
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350957"
---
# <a name="tutorial-create-an-faq-bot-with-azure-bot-service"></a>Tutoriel : Créer un bot de forum aux questions avec Azure Bot Service
Créez un bot de forum aux questions avec QnA Maker et Azure [Bot Service](https://azure.microsoft.com/services/bot-service/) sans code.

Dans ce tutoriel, vous allez apprendre à :

<!-- green checkmark -->
> [!div class="checklist"]
> * Lier une base de connaissances QnA Maker à une instance Azure Bot Service
> * Déployer le bot
> * Converser avec le bot dans un webchat
> * Activer le bot dans les canaux pris en charge

## <a name="create-and-publish-a-knowledge-base"></a>Créer et publier une base de connaissances

Suivez le guide de [démarrage rapide](../Quickstarts/create-publish-knowledge-base.md) pour créer une base de connaissances. Une fois la base de connaissances publiée, vous pouvez accéder à la page ci-dessous.

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

![Capture d’écran de la réussite de la publication](../media/qnamaker-create-publish-knowledge-base/publish-knowledge-base-to-endpoint.png)

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

![Capture d’écran de la réussite de la publication (version managée)](../media/qnamaker-create-publish-knowledge-base/publish-knowledge-base-to-endpoint-managed.png)

---

## <a name="create-a-bot"></a>Créer un bot

Après la publication, vous pouvez créer un bot à partir de la page **Publish** (Publier) :

* Vous pouvez créer plusieurs bots rapidement, tous pointant vers la même base de connaissances pour différentes régions ou différents plans tarifaires pour les bots individuels.
* Si vous ne voulez qu’un seul bot pour la base de connaissances, utilisez le lien **Afficher tous vos bots sur le portail Azure** pour voir la liste de vos bots actuel.

Quand vous apportez des modifications à la base de connaissances et que vous republiez, aucune autre action n’est nécessaire avec le bot. Il est déjà configuré pour fonctionner avec la base de connaissances et il fonctionne avec toutes les modifications futures de la base de connaissances. Chaque fois que vous publiez une base de connaissances, tous les bots qui y sont connectés sont automatiquement mis à jour.

1. Dans le portail QnA Maker, dans la page **Publish**, sélectionnez **Create bot**. Ce bouton apparaît seulement une fois que vous avez publié la base de connaissances.

     # <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

    ![Capture d’écran de la création d’un bot](../media/qnamaker-create-publish-knowledge-base/create-bot-from-published-knowledge-base-page.png)

    # <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

    ![Capture d’écran de la création d’un bot (préversion managée)](../media/qnamaker-create-publish-knowledge-base/create-bot-from-published-knowledge-base-page-managed.png)

    ---
    

1. Un nouvel onglet de navigateur s’ouvre pour le portail Azure, avec la page de création d’Azure Bot Service. Configurez le service bot Azure. Le bot et QnA Maker peuvent partager le plan du service Web App, mais ils ne peuvent pas partager l’application web. Cela signifie que le **nom de l’application** pour le bot doit être différent du nom de l’application pour le service QnA Maker.

    * **À faire**
        * Changer le descripteur du bot, s’il n’est pas unique.
        * Sélectionner le langage du SDK. Une fois le bot créé, vous pouvez télécharger le code dans votre environnement de développement local et poursuivre le processus de développement.
    * **À ne pas faire**
        * Modifier les paramètres suivants sur le portail Azure au moment de créer le bot. Ils sont préremplis pour votre base de connaissances existante :
           * Clé d’authentification QnA
           * Plan et emplacement App Service


1. Une fois le bot créé, ouvrez la ressource **Service Bot**.
1. Sous **Gestion du bot**, sélectionnez **Tester dans le Web Chat**.
1. À l’invite de chat **Tapez votre message**, entrez :

    `Azure services?`

    Le chatbot répond avec une réponse provenant de votre base de connaissances.

    :::image type="content" source="../media/qnamaker-create-publish-knowledge-base/test-web-chat.png" alt-text="Entrez une requête d’utilisateur dans le web chat de test.":::
1. Activez le bot dans des [canaux supplémentaires](/azure/bot-service/bot-service-manage-channels?preserve-view=true&view=azure-bot-service-4.0) pris en charge.
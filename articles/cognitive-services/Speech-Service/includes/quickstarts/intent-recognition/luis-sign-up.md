---
author: trevorbye
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 01/27/2020
ms.author: trbye
ms.openlocfilehash: a41c9dc888e85baf021712e3302da7ae6106db47
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "81422109"
---
Pour terminer le démarrage rapide de reconnaissance de l’intention, vous devez créer un compte LUIS et un projet à partir de la préversion du portail LUIS. Ce démarrage rapide nécessite uniquement un abonnement LUIS. Il n’est *pas* nécessaire de disposer d’un abonnement au service Speech.

La première chose à faire est de créer un compte et une application LUIS à partir de la préversion du portail LUIS. L’application LUIS que vous créez utilisera un domaine prédéfini pour la domotique, qui fournit des intentions, des entités et des exemples d’énoncés. À la fin du processus, vous disposerez d’un point de terminaison LUIS s’exécutant dans le cloud et que vous pourrez appeler à l’aide du kit SDK Speech. 

Pour créer votre application LUIS, suivez ces instructions :

* <a href="https://docs.microsoft.com/azure/cognitive-services/luis/luis-get-started-create-app" target="_blank">Démarrage rapide : Générer une application de domaine prédéfinie<span class="docon docon-navigate-external x-hidden-focus"></span></a>

Quand vous aurez terminé, vous aurez besoin de quatre éléments :

* Effectuer une republication avec **Préparation vocale** activé
* Votre **clé primaire** LUIS
* Votre **localisation** LUIS
* Votre **ID d’application** LUIS

Voici comment vous pouvez trouver ces informations sur la [préversion du portail LUI](https://preview.luis.ai/) :

1. À partir du portail LUIS en préversion, sélectionnez votre application, puis sélectionnez le bouton **Publier**.

2. Sélectionnez l’emplacement de **Production**. Si vous utilisez `en-US`, basculez l’option **Préparation vocale** sur la position **Activé**. Cliquez ensuite sur le bouton **Publier**.

    > [!IMPORTANT]
    > La **Préparation vocale** est vivement recommandée, car elle améliore la précision de la reconnaissance vocale.

    > [!div class="mx-imgBorder"]
    > ![Publier LUIS sur le point de terminaison](../../../media/luis/publish-app-popup.png)

3. À partir de la préversion du portail LUIS, sélectionnez **Manage** (Gérer), puis sélectionnez **Azure Resources** (Ressources Azure). Dans cette page, vous trouverez votre clé et votre localisation LUIS (parfois appelée _région_).

   > [!div class="mx-imgBorder"]
   > ![Clé et localisation LUIS](../../../media/luis/luis-key-region.png)

4. Après vous être procuré la clé et la localisation, vous aurez besoin de l’ID d’application. Sélectionnez **Application Settings** (Paramètres d’application) – votre ID d’application est disponible dans cette page.

   > [!div class="mx-imgBorder"]
   > ![ID d’application LUIS](../../../media/luis/luis-app-id.png)

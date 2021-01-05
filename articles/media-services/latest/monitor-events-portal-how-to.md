---
title: Superviser les événements Azure Media Services avec Event Grid à partir du portail
description: Cet article explique comment s’abonner à Event Grid dans le but de superviser les événements Azure Media Services.
services: media-services
documentationcenter: na
author: IngridAtMicrosoft
manager: femila
editor: ''
tags: ''
keywords: azure media services, flux, diffusion, en direct, hors connexion
ms.service: media-services
ms.devlang: multiple
ms.topic: how-to
ms.tgt_pltfrm: multiple
ms.workload: media
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: 65459dbdf8eaa92525fa20eda158f4beacf58780
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89294371"
---
# <a name="create-and-monitor-media-services-events-with-event-grid-using-the-azure-portal"></a>Créer et superviser des événements Azure Media Services avec Event Grid à partir du portail

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Azure Event Grid est un service de gestion d’événements pour le cloud. Ce service utilise les [abonnements aux événements](../../event-grid/concepts.md#event-subscriptions) pour acheminer les messages d’événements vers les abonnés. Les événements Media Services contiennent toutes les informations dont vous avez besoin pour répondre aux modifications de vos données. Vous pouvez identifier un événement Media Services, car la propriété eventType commence par « Microsoft.Media ». Pour plus d’informations, consultez les [schémas d’événement Media Services](media-services-event-schemas.md).

Dans cet article, vous allez utiliser le portail Azure pour vous abonner à des événements pour votre compte Azure Media Services. Vous déclencherez ensuite des événements pour en examiner le résultat. En règle générale, vous envoyez des événements à un point de terminaison qui traite les données d’événement et entreprend des actions. Dans cet article, nous allons envoyer des événements à une application web qui collecte et affiche les messages.

Une fois que vous avez fini, vous voyez que les données d’événement ont été envoyées à l’application web.

## <a name="prerequisites"></a>Prérequis 

* Avoir un abonnement Azure actif.
* Créez un nouveau compte Azure Media Services, comme décrit dans [ce démarrage rapide](./create-account-howto.md).

## <a name="create-a-message-endpoint"></a>Créer un point de terminaison de message

Avant de nous abonner aux événements pour le compte Media Services, créons le point de terminaison pour le message d’événement. En règle générale, le point de terminaison entreprend des actions en fonction des données d’événement. Dans cet article, vous allez déployer une [application web prédéfinie](https://github.com/Azure-Samples/azure-event-grid-viewer) qui affiche les messages d’événement. La solution déployée comprend un plan App Service, une offre App Service Web Apps et du code source en provenance de GitHub.

1. Sélectionnez **Déployer sur Azure** pour déployer la solution sur votre abonnement. Dans le portail Azure, indiquez des valeurs pour les paramètres.

   [![Image affichant le bouton intitulé « Déployer sur Azure ».](https://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fazure-event-grid-viewer%2Fmaster%2Fazuredeploy.json)

1. Le déploiement peut prendre quelques minutes. Une fois le déploiement réussi, affichez votre application web pour vérifier qu’elle s’exécute. Dans un navigateur web, accédez à : `https://<your-site-name>.azurewebsites.net`

Si vous basculez vers le site « Azure Event Grid Viewer », vous constatez qu’il n’y a pas encore d’événements.
   
[!INCLUDE [event-grid-register-provider-portal.md](../../../includes/event-grid-register-provider-portal.md)]

## <a name="subscribe-to-media-services-events"></a>S’abonner à des événements Media Services

Vous vous abonnez à une rubrique pour communiquer à Event Grid les événements qui vous intéressent, et où les envoyer.

1. Sur le portail, sélectionnez votre compte Media Services, puis **Événements**.
1. Pour envoyer des événements à votre application de visionneuse, utilisez un webhook pour le point de terminaison. 

   ![Sélectionner Webhook](./media/monitor-events-portal/select-web-hook.png)

1. L’abonnement aux événements est prérempli de valeurs pour votre compte Media Services. 
1. Sélectionnez « Webhook » pour **Type de point de terminaison**.
1. Dans cette rubrique, nous laissons la case **S’abonner à tous les types d’événements** cochée. Cependant, vous pouvez la décocher et filtrer sur certains types d’événements. 
1. Cliquez sur le lien **Sélectionner un point de terminaison**.

    Pour le point de terminaison du webhook, indiquez l’URL de votre application web et ajoutez `api/updates` à l’URL de la page d’accueil. 

1. Appuyez sur **Confirmer la sélection**.
1. Appuyez sur **Créer**.
1. Donnez un nom à votre abonnement.

   ![Sélectionner des journaux d’activité](./media/monitor-events-portal/create-subscription.png)

1. Affichez à nouveau votre application web, et notez qu’un événement de validation d’abonnement lui a été envoyé. 

    Event Grid envoie l’événement de validation pour que le point de terminaison puisse vérifier qu’il souhaite recevoir des données d’événement. Le point de terminaison doit définir `validationResponse` sur `validationCode`. Pour en savoir plus, consultez la page [Sécurité et authentification pour Event Grid](../../event-grid/security-authentication.md). Vous pouvez examiner le code de l’application web pour voir comment il valide l’abonnement.

À présent, déclenchons un événement pour voir comment Event Grid distribue le message à votre point de terminaison.

## <a name="send-an-event-to-your-endpoint"></a>Envoyer un événement à votre point de terminaison

Vous pouvez déclencher des événements pour le compte Media Services en exécutant un travail d’encodage. Vous pouvez suivre [ce démarrage rapide](stream-files-dotnet-quickstart.md) pour encoder un fichier et commencer à envoyer des événements. Si vous vous êtes abonné à tous les événements, vous obtenez un écran similaire à celui-ci :

> [!TIP]
> Sélectionnez l’icône en forme d’œil pour développer les données d’événements. N’actualisez pas la page si vous voulez afficher tous les événements.

![Afficher l’événement d’abonnement](./media/monitor-events-portal/view-subscription-event.png)

## <a name="next-steps"></a>Étapes suivantes

[Charger, encoder et diffuser en continu](stream-files-tutorial-with-api.md)

---
title: Gérer les événements Service Bus via Event Grid à l’aide d’Azure Functions
description: Cet article explique comment gérer les événements Service Bus via Event Grid à l’aide d’Azure Functions.
documentationcenter: .net
author: spelluru
ms.topic: tutorial
ms.date: 06/23/2020
ms.author: spelluru
ms.custom: devx-track-csharp
ms.openlocfilehash: afc0a5bf9b83363d1f4baab955b55148fe3a8498
ms.sourcegitcommit: 6a770fc07237f02bea8cc463f3d8cc5c246d7c65
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95818449"
---
# <a name="tutorial-respond-to-azure-service-bus-events-received-via-azure-event-grid-by-using-azure-functions"></a>Tutoriel : Répondre aux événements Azure Service Bus reçus via Azure Event Grid à l’aide d’Azure Functions
Dans ce tutoriel, vous allez apprendre à répondre aux événements Azure Service Bus qui sont reçus via Azure Event Grid à l’aide d’Azure Functions et d’Azure Logic Apps. 

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> * Création d’un espace de noms Service Bus
> * Préparer un exemple d’application pour envoyer des messages
> * Envoyer des messages à la rubrique Service Bus
> * Recevoir des messages à l’aide de Logic Apps
> * Configurer une fonction de test sur Azure
> * Connecter la fonction et l’espace de noms via Event Grid
> * Recevoir des messages à l’aide d’Azure Functions

[!INCLUDE [service-bus-event-grid-prerequisites](../../includes/service-bus-event-grid-prerequisites.md)]

## <a name="additional-prerequisites"></a>Autres composants requis
Installez [Visual Studio 2019](https://www.visualstudio.com/vs) et incluez la charge de travail **Développement Azure**. Cette charge de travail comprend les **outils Azure Functions** dont vous avez besoin pour créer, générer et déployer des projets Azure Functions dans Visual Studio. 

## <a name="deploy-the-function-app"></a>Déployer l’application de fonction 

>[!NOTE]
> Pour plus d’informations sur la création et le déploiement d’une application Azure Functions, consultez [Développer des fonctions Azure Functions avec Visual Studio](../azure-functions/functions-develop-vs.md).

1. Ouvrez le fichier **ReceiveMessagesOnEvent.cs** à partir du projet **FunctionApp1** de la solution **SBEventGridIntegration.sln**. 
1. Remplacez `<SERCICE BUS NAMESPACE - CONNECTION STRING>` par la chaîne de connexion de votre espace de noms Service Bus. Elle doit être identique à celle que vous avez utilisée dans le fichier **Program.cs** du projet **MessageSender** de la même solution. 
1. Cliquez avec le bouton droit sur **FunctionApp1**, puis sélectionnez **Publier**. 
1. Dans la page **Publier**, sélectionnez **Démarrer**. Ces étapes peuvent être différentes de celles que vous voyez, mais le processus de publication doit être similaire. 
1. Dans l’Assistant **Publication**, dans la page **Cible**, sélectionnez **Azure** pour **Cible**. 
1. Dans la page **Cible spécifique**, sélectionnez **Application de fonction Azure (Windows)** . 
1. Dans la page **Instance de Functions**, sélectionnez **Créer une instance d’Azure Functions**. 
1. Dans la page **Application de fonction (Windows)** , procédez comme suit :
    1. Entrez un **nom** pour l’application de fonction.
    1. Sélectionnez un **abonnement** Azure.
    1. Sélectionnez un **groupe de ressources** existant ou créez-en un. Pour ce tutoriel, sélectionnez le groupe de ressources contenant l’espace de noms Service Bus. 
    1. Sélectionnez un **type de plan** pour App Service.
    1. Sélectionnez un **emplacement**. Sélectionnez le même emplacement que l’espace de noms Service Bus. 
    1. Sélectionnez un compte de **stockage Azure** existant ou sélectionnez **Nouveau** pour créer un compte de stockage à utiliser par l’application Functions. 
    1. Sélectionnez **Créer** pour créer l’application Functions. 
1. De retour dans la page **Instance de Functions** de l’Assistant **Publication**, sélectionnez **Terminer**. 
1. Dans la page **Publier** de Visual Studio, sélectionnez **Publier** pour publier l’application Functions sur Azure. 

    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/publish-function-app.png" alt-text="Publier l’application Functions":::    
1. Dans la fenêtre **Sortie**, vérifiez que les opérations de génération et de publication ont réussi. 
1. Maintenant, dans la page **Publier**, sélectionnez **Gérer dans le portail Azure**. 
1. Dans le portail Azure, dans la page **Application de fonction**, sélectionnez **Fonctions** dans le menu de gauche, puis vérifiez que vous voyez deux fonctions : 

    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/functions.png" alt-text="Deux fonctions pour gérer les déclencheurs Event Grid et HTTP":::

    > [!NOTE]
    > `EventGridTriggerFunction` utilise un [déclencheur Event Grid](../azure-functions/functions-bindings-event-grid-trigger.md) tandis que `HTTPTriggerFunction` utilise un [déclencheur HTTP](../azure-functions/functions-bindings-http-webhook-trigger.md).
1. Sélectionnez **EventGridTriggerFunction** dans la liste. Nous vous recommandons d’utiliser le déclencheur Event Grid avec Azure Functions, car il présente quelques avantages par rapport à l’utilisation du déclencheur HTTP. Pour plus d’informations, consultez [Fonction Azure en tant que gestionnaire d’événements pour des événements Event Grid](../event-grid/handler-functions.md).
1. Dans la page **Fonction** pour **EventGridTriggerFunction**, sélectionnez **Superviser** dans le menu de gauche. 
1. Sélectionnez **Configurer** pour configurer Application Insights afin de capturer le journal d’invocations. Vous utilisez cette page pour superviser les exécutions de fonction lors de la réception d’événements Service Bus à partir d’Event Grid. 
1. Dans la page **Application Insights**, entrez un nom pour la ressource, sélectionnez un **emplacement** pour celle-ci, puis cliquez sur **OK**. 
1. Sélectionnez la fonction **EventGridTriggerFunction** en haut (menu de navigation) pour revenir à la page **Fonction**. 
1. Vérifiez que vous êtes dans la page **Superviser**. 

    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/monitor-before.png" alt-text="Page Superviser pour la fonction avant les invocations de fonction":::
    
    Laissez cette page ouverte sous un onglet de votre navigateur web. Vous actualiserez plus tard cette page pour voir les invocations liées à cette fonction.

## <a name="connect-the-function-and-namespace-via-event-grid"></a>Connecter la fonction et l’espace de noms via Event Grid
Dans cette section, vous allez relier la fonction et l’espace de noms Service Bus à l’aide du portail Azure. 

Pour créer un abonnement Azure Event Grid, effectuez les étapes suivantes :

1. Dans le portail Azure, accédez à votre espace de noms puis, dans le volet gauche, sélectionnez **Événements**. La fenêtre de votre espace de noms s’ouvre, avec deux abonnements Event Grid affichés dans le volet droit. 
    
    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/service-bus-events-page.png" alt-text="Service Bus - page d’événements":::
2. Sélectionnez **+ Abonnement aux événements** dans la barre d’outils. 
3. Dans la page **Créer un abonnement aux événements**, effectuez les étapes suivantes :
    1. Entrez un **nom** pour l’abonnement. 
    2. Entrez un **nom** pour la **rubrique système**. Les rubriques sur le système sont des rubriques créées pour les ressources Azure, telles que le compte de stockage Azure et Azure Service Bus. Pour en savoir plus sur les rubriques système, consultez [Vue d’ensemble des rubriques système](../event-grid/system-topics.md).
    2. Sélectionnez **Fonction Azure** pour **Type de point de terminaison**, puis cliquez sur **Sélectionner un point de terminaison**. 

        ![Service Bus - abonnement Event Grid](./media/service-bus-to-event-grid-integration-example/event-grid-subscription-page.png)
    3. Dans la page **Sélectionner une fonction Azure**, sélectionnez l’abonnement, le groupe de ressources, l’application de fonction, l’emplacement et la fonction, puis sélectionnez **Confirmer la sélection**. 

        ![Fonction - sélectionner le point de terminaison](./media/service-bus-to-event-grid-integration-example/function-select-endpoint.png)
    4. Dans la page **Créer un abonnement aux événements**, basculez vers l’onglet **Filtres**, puis effectuez les tâches suivantes :
        1. Sélectionnez **Activer le filtrage d’objet**.
        2. Entrez le nom de l’abonnement dans la rubrique Service Bus (**S1**) que vous avez créée précédemment.
        3. Cliquez sur le bouton **Créer**. 

            ![Filtre d’abonnement aux événements](./media/service-bus-to-event-grid-integration-example/event-subscription-filter.png)
4. Passez dans l’onglet **Abonnements aux événements** de la page **Événements** et vérifiez que vous voyez l’abonnement aux événements dans la liste.

    ![Abonnement aux événements dans la liste](./media/service-bus-to-event-grid-integration-example/event-subscription-in-list.png)

## <a name="monitor-the-functions-app"></a>Superviser l’application Functions
Les messages que vous avez envoyés à la rubrique Service Bus précédemment sont transférés à l’abonnement (S1). Event Grid transfère les messages au niveau de l’abonnement à la fonction Azure. Au cours de cette étape du tutoriel, vous vérifiez que la fonction a été appelée et vous affichez les messages d’information journalisés. 

1. Dans la page de votre application de fonction Azure, basculez vers l’onglet **Superviser** s’il n’est pas déjà activé. Vous devez voir une entrée pour chaque message posté dans la rubrique Service Bus. Si ce n’est pas le cas, actualisez la page au bout de quelques minutes. 

    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/function-monitor.png" alt-text="Page Superviser pour la fonction après les invocations":::
2. Sélectionnez l’invocation dans la liste pour afficher les détails. 

    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/invocation-details.png" alt-text="Détails de l’invocation de la fonction" lightbox="./media/service-bus-to-event-grid-integration-example/invocation-details.png":::

    Vous pouvez également utiliser l’onglet **Journaux** de la page **Superviser** pour voir les informations de journalisation au fur et à mesure que les messages sont envoyés. Il peut y avoir un peu de délai, patientez quelques minutes pour voir les messages journalisés. 

    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/function-logs.png" alt-text="Journaux d’activité de fonction" lightbox="./media/service-bus-to-event-grid-integration-example/function-logs.png":::

## <a name="troubleshoot"></a>Dépanner
Si vous ne voyez aucune invocation de fonction après avoir attendu un certain temps et actualisé la page, procédez comme suit : 

1. Vérifiez que les messages ont atteint la rubrique Service Bus. Consultez le compteur des **messages entrants** dans la page **Rubrique Service Bus**. En l’occurrence, comme j’ai exécuté l’application **MessageSender** à deux reprises, je vois 10 messages (5 messages pour chaque exécution).

    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/topic-incoming-messages.png" alt-text="Page Rubrique Service Bus : messages entrants":::    
1. Vérifiez qu’il n’y a **aucun message actif** au niveau de l’abonnement Service Bus. 
    Si vous ne voyez pas d’événements dans cette page, vérifiez que la page **Abonnement Service Bus** n’affiche aucun **Nombre de messages actifs**. Si le nombre de ce compteur est supérieur à zéro, les messages au niveau de l’abonnement ne sont pas transmis à la fonction de gestionnaire (gestionnaire d’abonnement aux événements) pour une raison quelconque. Vérifiez que vous avez correctement configuré l’abonnement aux événements. 

    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/subscription-active-message-count.png" alt-text="Nombre de messages actifs au niveau de l’abonnement Service Bus":::    
1. Vous pouvez également voir les **événements remis** dans la page **Événements** de l’espace de noms Service Bus. 

    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/event-subscription-page.png" alt-text="Page événements : événements remis" lightbox="./media/service-bus-to-event-grid-integration-example/invocation-details.png":::
1. Vous pouvez également voir que les événements sont remis dans la page **Abonnement aux événements**. Vous pouvez accéder à cette page en sélectionnant l’abonnement aux événements dans la page **Événements**. 
    
    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/event-subscription-delivered-events.png" alt-text="Page Abonnement aux événements : événements remis":::
    


## <a name="next-steps"></a>Étapes suivantes

* Apprenez-en plus sur [Azure Event Grid](../event-grid/index.yml).
* Apprenez-en plus sur [Azure Functions](../azure-functions/index.yml).
* Apprenez-en plus sur [la fonctionnalité Logic Apps d’Azure App Service](../logic-apps/index.yml).
* En savoir plus sur [Azure Service Bus](/azure/service-bus/).


[2]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid2.png
[3]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid3.png
[7]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid7.png
[8]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid8.png
[9]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid9.png
[10]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid10.png
[11]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid11.png
[12]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid12.png
[12-1]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid12-1.png
[12-2]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid12-2.png
[13]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid13.png
[14]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid14.png
[15]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid15.png
[16]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid16.png
[17]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid17.png
[18]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid18.png
[20]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgridportal.png
[21]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgridportal2.png
---
title: 'Démarrage rapide : Envoyer des SMS dans Azure Logic Apps à l’aide d’Azure Communication Services'
titleSuffix: An Azure Communication Services quickstart
description: Dans ce guide de démarrage rapide, découvrez comment envoyer des SMS dans les workflows Azure Logic Apps à l’aide du connecteur Azure Communication Services.
author: tophpalmer
manager: anvalent
services: azure-communication-services
ms.author: chpalm
ms.date: 10/06/2020
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: ed9af178c10d033806b05f73372a0642bb28cab4
ms.sourcegitcommit: 23aa0cf152b8f04a294c3fca56f7ae3ba562d272
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2020
ms.locfileid: "91801643"
---
# <a name="quickstart-send-sms-messages-in-azure-logic-apps-with-azure-communication-services"></a>Démarrage rapide : Envoyer des SMS dans Azure Logic Apps avec Azure Communication Services

À l’aide du connecteur [Azure Communication Services SMS](../../overview.md) et d’[Azure Logic Apps](../../../logic-apps/logic-apps-overview.md), vous pouvez créer des workflows automatisés, ou *applications logiques*, qui peuvent envoyer des SMS. Ce guide de démarrage rapide montre comment envoyer automatiquement des SMS en réponse à un événement déclencheur, qui est la première étape d’un workflow d’application logique. Un événement déclencheur peut être un e-mail entrant, une planification de périodicité, un événement de ressource [Azure Event Grid](../../../event-grid/overview.md) ou tout autre [déclencheur pris en charge par Azure Logic Apps](/connectors/connector-reference/connector-reference-logicapps-connectors).

:::image type="content" source="./media/logic-app/azure-communication-services-connector.png" alt-text="Capture d’écran montrant le portail Azure, ouvert sur le concepteur d’application logique, et un exemple d’application logique qui utilise l’action Envoyer un SMS pour le connecteur Azure Communication Services":::

Même si ce guide de démarrage rapide se concentre sur l’utilisation du connecteur pour répondre à un déclencheur, vous pouvez également utiliser le connecteur pour répondre à d’autres actions, qui sont les étapes qui suivent le déclencheur dans un workflow. Si vous débutez avec Logic Apps, consultez [Qu’est-ce qu’Azure Logic Apps ?](../../../logic-apps/logic-apps-overview.md) avant de commencer.

> [!NOTE]
> Le fait de suivre ce guide de démarrage rapide entraîne une petite dépense de quelques cents USD tout au plus dans votre compte Azure.

## <a name="prerequisites"></a>Prérequis

- Un compte Azure avec un abonnement actif ou [créez gratuitement un compte Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

- Une ressource Azure Communication Services active ou [créez une ressource Communication Services](../create-communication-resource.md).

- Une ressource Logic Apps active (application logique), ou [créez une application logique vide, mais avec le déclencheur que vous souhaitez utiliser](../../../logic-apps/quickstart-create-first-logic-app-workflow.md). Comme le connecteur Azure Communication Services SMS ne fournit que des actions, votre application logique nécessite au minimum un déclencheur.

  Ce guide de démarrage rapide utilise le déclencheur **À la réception d’un e-mail**, qui est disponible avec le [connecteur Office 365 Outlook](/connectors/office365/).

- Un numéro de téléphone permettant de recevoir des SMS ou [obtenez un numéro de téléphone](./get-phone-number.md).

## <a name="add-an-sms-action"></a>Ajouter une action SMS

Pour ajouter l’action **Envoyer un SMS** en tant que nouvelle étape dans votre workflow à l’aide du connecteur Azure Communication Services SMS, procédez comme suit dans le [portail Azure](https://portal.azure.com) avec le workflow de votre application logique ouvert dans le concepteur d’application logique :

1. Dans le concepteur, sous l’étape à laquelle vous souhaitez ajouter la nouvelle action, choisissez **Nouvelle étape**. Vous pouvez également ajouter la nouvelle action entre des étapes, déplacer votre pointeur sur la flèche située entre ces étapes, sélectionner le signe plus ( **+** ), puis sélectionner **Ajouter une action**.

1. Dans la zone de recherche **Choisir une opération**, entrez `Azure Communication Services`. Dans la liste des actions, sélectionnez **Envoyer un SMS**.

   :::image type="content" source="./media/logic-app/select-send-sms-action.png" alt-text="Capture d’écran montrant le portail Azure, ouvert sur le concepteur d’application logique, et un exemple d’application logique qui utilise l’action Envoyer un SMS pour le connecteur Azure Communication Services":::

1. Créez maintenant une connexion à votre ressource Communication Services.

   1. Entrez un nom pour la connexion.

   1. Sélectionnez votre ressource Azure Communication Services.

   1. Sélectionnez **Create** (Créer).

   :::image type="content" source="./media/logic-app/send-sms-configuration.png" alt-text="Capture d’écran montrant le portail Azure, ouvert sur le concepteur d’application logique, et un exemple d’application logique qui utilise l’action Envoyer un SMS pour le connecteur Azure Communication Services":::

1. Dans l’action **Envoyer un SMS**, entrez les informations suivantes : 

   * Numéros de téléphone source et de destination. À des fins de test, vous pouvez utiliser votre propre numéro de téléphone comme numéro de téléphone de destination.

   * Contenu du message que vous souhaitez envoyer, par exemple, « Hello from Logic Apps ! ».

   Voici une action **Envoyer un SMS** avec des exemples d’informations :

   :::image type="content" source="./media/logic-app/send-sms-action.png" alt-text="Capture d’écran montrant le portail Azure, ouvert sur le concepteur d’application logique, et un exemple d’application logique qui utilise l’action Envoyer un SMS pour le connecteur Azure Communication Services":::

1. Lorsque c’est chose faite, dans la barre d’outils du concepteur, sélectionnez **Enregistrer**.

Ensuite, exécutez votre application logique à des fins de test.

## <a name="test-your-logic-app"></a>Tester votre application logique

Pour lancer manuellement votre application logique, sélectionnez **Exécuter** dans la barre d’outils du concepteur. Vous pouvez aussi attendre que votre application logique se déclenche. Dans les deux cas, l’application logique doit envoyer un SMS au numéro de téléphone de destination que vous avez spécifié. Pour plus d’informations sur l’exécution de votre application logique, consultez les [instructions permettant d’exécuter votre application logique](../../../logic-apps/quickstart-create-first-logic-app-workflow.md#run-your-logic-app).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour enlever un abonnement Communication Services, supprimez la ressource ou le groupe de ressources Communication Services. La suppression du groupe de ressources efface également les autres ressources dans ce groupe. Pour plus d’informations, consultez les [instructions permettant de nettoyer les ressources Communication Services](../create-communication-resource.md#clean-up-resources).

Pour nettoyer le workflow de votre application logique et les ressources associées, consultez les [instructions permettant de nettoyer des ressources Logic Apps](../../../logic-apps/quickstart-create-first-logic-app-workflow.md#clean-up-resources).

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à envoyer des SMS avec Azure Logic Apps et Azure Communication Services. Pour approfondir vos connaissances, poursuivez avec l’abonnement aux événements SMS :

> [!div class="nextstepaction"]
> [S’abonner aux événements SMS](./handle-sms-events.md)

Pour plus d’informations sur SMS dans les Azure Communication Services, consultez les articles suivants :

- [Concepts relatifs au SMS](../../concepts/telephony-sms/concepts.md)
- [Planifier votre solution de téléphonie et SMS](../../concepts/telephony-sms/plan-solution.md)
- [SDK SMS](../../concepts/telephony-sms/sdk-features.md)

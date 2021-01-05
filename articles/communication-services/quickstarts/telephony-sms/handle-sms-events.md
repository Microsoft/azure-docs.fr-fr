---
title: Démarrage rapide - Gérer les événements SMS
titleSuffix: An Azure Communication Services quickstart
description: Découvrez comment gérer les événements SMS avec Azure Communication Services.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 09/30/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: d6409c005e006372f55e77aeb4d977e6b1c45832
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96936271"
---
# <a name="quickstart-handle-sms-events"></a>Démarrage rapide : Gérer les événements SMS

[!INCLUDE [Public Preview Notice](../../includes/public-preview-include.md)]
[!INCLUDE [Regional Availability Notice](../../includes/regional-availability-include.md)]

Commencez avec Azure Communication Services en utilisant Azure Event Grid pour gérer les événements SMS Communication Services. 

## <a name="about-azure-event-grid"></a>À propos d’Azure Event Grid

[Azure Event Grid](../../../event-grid/overview.md) est un service d’événement cloud. Dans cet article, vous allez apprendre à vous abonner à des [événements de service de communication](../../concepts/event-handling.md) et à déclencher un événement pour voir le résultat. En règle générale, vous envoyez des événements à un point de terminaison qui traite les données d’événement et entreprend des actions. Dans cet article, nous allons envoyer les événements à une application web qui collecte et affiche les messages.

## <a name="prerequisites"></a>Prérequis
- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- Une ressource Azure Communication Services. Pour plus d’informations, consultez le guide de démarrage rapide [Créer une ressource Azure Communication](../create-communication-resource.md).
- Un numéro de téléphone permettant de recevoir des SMS. [Obtenez un numéro de téléphone](./get-phone-number.md).

## <a name="setting-up"></a>Configuration

### <a name="enable-event-grid-resource-provider"></a>Activer le fournisseur de ressources Event Grid

Si vous n’avez jamais utilisé Event Grid dans votre abonnement Azure auparavant, vous devrez peut-être inscrire le fournisseur de ressources Event Grid en effectuant les étapes mentionnées ci-dessous :

Dans le portail Azure :

1. Sélectionnez **Abonnements** dans le menu de gauche.
2. Sélectionnez l’abonnement que vous utilisez pour Event Grid.
3. Dans le menu de gauche, sous **Paramètres**, sélectionnez **Fournisseurs de ressources**.
4. Recherchez **Microsoft.EventGrid**.
5. Si l’inscription n’est pas encore faite, sélectionnez **S’inscrire**. 

L’inscription peut prendre un certain temps. Sélectionnez **Actualiser** pour mettre à jour l’état. Lorsque l’**état** est **Inscrit**, vous êtes prêt à continuer.

### <a name="event-grid-viewer-deployment"></a>Déploiement de la visionneuse Event Grid

Pour ce guide de démarrage rapide, nous allons utiliser l’[exemple de la visionneuse Event Grid](/samples/azure-samples/azure-event-grid-viewer/azure-event-grid-viewer/) pour voir les événements en quasi-temps réel. Cela offre à l’utilisateur l’expérience d’un flux en temps réel. De plus, la charge utile de chaque événement doit également être disponible pour inspection.  

## <a name="subscribe-to-the-sms-events-using-web-hooks"></a>S’abonner aux événements SMS à l’aide de webhooks

Dans le portail, accédez à votre ressource Azure Communication Services que vous avez créée. Dans la ressource Communication Services, sélectionnez **Événements** dans le menu de gauche de la page **Communication Services**.

:::image type="content" source="./media/handle-sms-events/select-events.png" alt-text="Capture d’écran montrant la sélection du bouton d’abonnement aux événements dans la page des événements d’une ressource.":::

Appuyez sur **Ajouter un abonnement à un événement** pour accéder à l’Assistant Création.

Dans la page **Créer un abonnement aux événements**, entrez un **nom** pour l’abonnement aux événements.

Vous pouvez vous abonner à des événements spécifiques pour indiquer à Event Grid les événements SMS que vous voulez suivre, et où les envoyer. Dans le menu déroulant, sélectionnez les événements auxquels vous voulez vous abonner. Pour les SMS, vous avez le choix entre `SMS Received` et `SMS Delivery Report Received`. 

Si vous êtes invité à fournir un **Nom de rubrique système**, n’hésitez pas à fournir une chaîne unique. Ce champ n’a aucun impact sur votre expérience et est utilisé à des fins de télémétrie interne.

Consultez la liste complète des [événements pris en charge par Azure Communication Services](../../concepts/event-handling.md).

:::image type="content" source="./media/handle-sms-events/select-events-create-eventsub.png" alt-text="Capture d’écran montrant les types d’événements SMS Received (SMS reçu) et SMS Delivery Report Received (Rapport de remise de SMS reçu) sélectionnés.":::

Sélectionnez **Webhook** pour **Type de point de terminaison**. 

:::image type="content" source="./media/handle-sms-events/select-events-create-linkwebhook.png" alt-text="Capture d’écran montrant le champ Type de point de terminaison défini sur Webhook.":::

Pour **Point de terminaison**, cliquez sur **Sélectionner un point de terminaison**, puis entrez l’URL de votre application web.

Dans le cas présent, nous allons utiliser l’URL de l’[exemple de la visionneuse Event Grid](/samples/azure-samples/azure-event-grid-viewer/azure-event-grid-viewer/) que nous avons configurée précédemment dans le guide de démarrage rapide. L’URL de l’exemple sera au format suivant : `https://{{site-name}}.azurewebsites.net/api/updates`

Sélectionnez ensuite **Confirmer la sélection**.

:::image type="content" source="./media/handle-sms-events/select-events-create-selectwebhook-epadd.png" alt-text="Capture d’écran montrant la confirmation d’un point de terminaison webhook.":::

## <a name="viewing-sms-events"></a>Affichage d’événements SMS

### <a name="triggering-sms-events"></a>Déclenchement d’événements SMS

Pour afficher des déclencheurs d’événements, nous devons en premier lieu générer des événements.

- Les événements `SMS Received` sont générés quand le numéro de téléphone Communication Services reçoit un SMS. Pour déclencher un événement, envoyez simplement un message à partir de votre téléphone au numéro de téléphone attaché à votre ressource Communication Services.
- Les événements `SMS Delivery Report Received` sont générés quand vous envoyez un SMS à un utilisateur à l’aide d’un numéro de téléphone Communication Services. Pour déclencher un événement, vous devez activer `Delivery Report` dans les options du [SMS envoyé](../telephony-sms/send.md). Essayez d’envoyer un message à votre téléphone avec `Delivery Report` (Rapport de remise). Effectuer cette action entraîne une petite dépense de quelques cents USD tout au plus dans votre compte Azure.

Consultez la liste complète des [événements pris en charge par Azure Communication Services](../../concepts/event-handling.md).

### <a name="receiving-sms-events"></a>Réception d’événements SMS

Une fois que vous avez effectué l’une des actions ci-dessus, vous remarquerez que les événements `SMS Received` et `SMS Delivery Report Received` sont envoyés à votre point de terminaison. Ces événements s’affichent dans l’[exemple de la visionneuse Event Grid](/samples/azure-samples/azure-event-grid-viewer/azure-event-grid-viewer/) que nous avons configuré au début. Vous pouvez appuyer sur l’icône représentant un œil en regard de l’événement pour voir l’intégralité de la charge utile. Les événements se présentent comme suit :

:::image type="content" source="./media/handle-sms-events/sms-received.png" alt-text="Capture d’écran montrant le schéma Event Grid pour un événement SMS Received (SMS reçu).":::

:::image type="content" source="./media/handle-sms-events/sms-delivery-report-received.png" alt-text="Capture d’écran montrant le schéma Event Grid pour un événement SMS Delivery Report (Rapport de remise de SMS).":::

Découvrez-en plus sur les [schémas d’événements et d’autres concepts liés aux événements](../../concepts/event-handling.md).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous voulez nettoyer et supprimer un abonnement Communication Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources efface également les autres ressources qui y sont associées. Apprenez-en davantage sur le [nettoyage des ressources](../create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris comment consommer des événements SMS. Vous pouvez recevoir des messages SMS en créant un abonnement Event Grid.

> [!div class="nextstepaction"] 
> [Envoyer un SMS](../telephony-sms/send.md)

Vous voudrez peut-être aussi :

 - [Découvrir les concepts de gestion des événements](../../concepts/event-handling.md)
 - [Découvrir Event Grid](../../../event-grid/overview.md)
---
title: Notifications dans Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Envoyez des notifications aux utilisateurs d’applications reposant sur Azure Communication Services.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 09/30/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: a52188dc5058dbc74d3b03fba860b98540cd4a41
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96608501"
---
# <a name="communication-services-notifications"></a>Notifications Communication Services

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]

Les bibliothèques de client Azure Communication Services Chat et Calling créent un canal de messagerie en temps réel qui permet l’envoi (push) des messages de signalisation aux clients connectés de manière efficace et fiable. Cela vous permet de créer des fonctionnalités de communication riches en temps réel dans vos applications sans avoir à implémenter une logique d’interrogation HTTP compliquée. Toutefois, sur les applications mobiles, ce canal de signalisation reste connecté uniquement quand votre application est active au premier plan. Si vous souhaitez que vos utilisateurs reçoivent des appels entrants ou des messages de conversation lorsque votre application est en arrière-plan, vous devez utiliser des notifications Push.

Les notifications Push vous permettent d’envoyer des informations à partir de votre application vers les appareils mobiles des utilisateurs. Vous pouvez utiliser des notifications Push pour afficher une boîte de dialogue, émettre un signal sonore ou afficher l’interface utilisateur des appels entrants. Azure Communication Services fournit des intégrations avec [Azure Event Grid](../../event-grid/overview.md) et [Azure Notification Hubs](../../notification-hubs/notification-hubs-push-notification-overview.md) qui vous permettent d’ajouter des notifications Push à vos applications.

## <a name="trigger-push-notifications-via-azure-event-grid"></a>Déclencher des notifications Push via Azure Event Grid

Azure Communication Services s’intègre à [Azure Event Grid](https://azure.microsoft.com/services/event-grid/) pour fournir des notifications d’événements en temps réel de manière fiable, scalable et sécurisée. Vous pouvez tirer parti de cette intégration pour créer un service de notification qui fournisse des notifications Push mobiles à vos utilisateurs en créant un abonnement Event Grid qui déclenche une [fonction Azure](../../azure-functions/functions-overview.md) ou un webhook.

:::image type="content" source="./media/notifications/acs-events-int.png" alt-text="Schéma montrant comment Communication Services s’intègre à Event Grid.":::

Apprenez-en davantage sur la [gestion des événements dans Azure Communication Services](./event-handling.md).

## <a name="deliver-push-notifications-via-azure-notification-hubs"></a>Envoyer des notifications Push via Azure Notification Hubs

Vous pouvez connecter un hub de notification Azure à votre ressource Communication Services afin d’envoyer automatiquement des notifications Push à l’appareil mobile d’un utilisateur lorsqu’il reçoit un appel entrant. Vous devez utiliser ces notifications Push pour réveiller votre application à partir de l’arrière-plan et afficher l’interface utilisateur permettant à l’utilisateur d’accepter ou de refuser l’appel. 

:::image type="content" source="./media/notifications/acs-anh-int.png" alt-text="Schéma montrant comment Communication Services s’intègre à Azure Notifications Hub.":::

Communication Services utilise Azure Notification Hub en tant que service direct pour communiquer avec les différents services de notification Push spécifiques à la plateforme à l’aide de l’API [Envoi direct](/rest/api/notificationhubs/direct-send). Cela vous permet de réutiliser vos ressources et configurations Azure Notification Hub existantes pour fournir des notifications d’appel fiables et à faible latence à vos applications.

> [!NOTE]
> Seules les notifications Push d’appel sont prises en charge.

### <a name="notification-hub-provisioning"></a>Provisionnement des hubs de notification 

Pour envoyer des notifications Push aux appareils clients à l’aide de Notification Hubs, [créez un hub de notification](../../notification-hubs/create-notification-hub-portal.md) dans le même abonnement que votre ressource Communication Services. Azure Notification Hubs doit être configuré pour le service de notification de plateforme que vous souhaitez utiliser. Pour savoir comment obtenir des notifications Push dans votre application cliente provenant de Notification Hubs, consultez [Prise en main de Notification Hubs](../../notification-hubs/notification-hubs-android-push-notification-google-fcm-get-started.md) et sélectionnez votre plateforme de client cible dans la liste déroulante en haut de la page.

> [!NOTE]
> Les plateformes APNs et FCM sont prises en charge.  
La plateforme APNs doit être configurée avec le mode d’authentification par jeton. Le mode d'authentification par certificat n'est pas pris en charge. 

Une fois que votre hub de notification est configuré, vous pouvez l’associer à votre ressource Communication Services en fournissant une chaîne de connexion pour le hub à l’aide du client Azure Resource Manager ou par le biais du portail Azure. La chaîne de connexion doit contenir des autorisations « Envoyer ». Nous vous recommandons de créer une autre stratégie d’accès avec des autorisations « Envoyer » uniquement, en particulier pour votre hub. Découvrez plus en détail les [stratégies de sécurité et d’accès de Notification Hubs](../../notification-hubs/notification-hubs-push-notification-security.md)

#### <a name="using-the-azure-resource-manager-client-to-configure-the-notification-hub"></a>Utilisation du client Azure Resource Manager pour configurer le hub de notification

Pour vous connecter à Azure Resource Manager, exécutez la commande suivante et connectez-vous à l’aide de vos informations d’identification.

```console
armclient login
```

 Une fois connecté, exécutez la commande suivante pour provisionner le hub de notification :

```console
armclient POST /subscriptions/<sub_id>/resourceGroups/<resource_group>/providers/Microsoft.Communication/CommunicationServices/<resource_id>/linkNotificationHub?api-version=2020-08-20-preview "{'connectionString': '<connection_string>','resourceId': '<resource_id>'}"
```

#### <a name="using-the-azure-portal-to-configure-the-notification-hub"></a>Utilisation du portail Azure pour configurer le hub de notification

Dans le portail, accédez à votre ressource Azure Communication Services. Dans la ressource Communication Services, sélectionnez Notifications Push dans le menu de gauche de la page Communication Services et connectez le hub de notification que vous avez provisionné précédemment. Vous devez fournir votre chaîne de connexion et votre ID de ressource ici :

:::image type="content" source="./media/notifications/acs-anh-portal-int.png" alt-text="Capture d’écran montrant les paramètres de notifications Push dans le portail Azure.":::

> [!NOTE]
> Si la chaîne de connexion du hub de notification Azure est mise à jour, la ressource Communication Services doit également être mise à jour.  
Toute modification relative à la façon dont le hub est lié est reflétée dans le plan de données (par exemple lors de l’envoi d’une notification) pendant une période maximale de ``10`` minutes. Cela s’applique également lorsque le hub est lié pour la première fois **si** des notifications ont été envoyées avant.

#### <a name="device-registration"></a>Enregistrement de l’appareil 

Reportez-vous au [guide de démarrage rapide des appels vocaux](../quickstarts/voice-video-calling/getting-started-with-calling.md) pour découvrir comment inscrire votre handle d’appareil auprès de Communication Services.

## <a name="next-steps"></a>Étapes suivantes

* Pour une présentation d’Azure Event Grid, consultez [Présentation d’Event Grid](../../event-grid/overview.md).
* Pour en savoir plus sur les concepts d’Azure Notification Hub, consultez [Documentation Azure Notification Hubs](../../notification-hubs/index.yml)
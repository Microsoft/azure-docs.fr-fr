---
title: Routage et expressions de balise dans Azure Notification Hubs
description: Découvrez le routage et les expressions de balise pour Azure Notification Hubs.
services: notification-hubs
documentationcenter: .net
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: 0fffb3bb-8ed8-4e0f-89e8-0de24a47f644
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: dotnet
ms.topic: article
ms.date: 12/09/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 12/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 4ff8225522e79e2be40682fb5e4823777dde2aa0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88998300"
---
# <a name="routing-and-tag-expressions"></a>Routage et expressions de balise

## <a name="overview"></a>Vue d’ensemble

Les expressions de balise vous permettent de cibler des ensembles spécifiques d'appareils, ou plus précisément d'inscriptions, lors de l'envoi d'une notification push via Notification Hubs.

## <a name="targeting-specific-registrations"></a>Ciblage d'inscriptions spécifiques

La seule façon de cibler des inscriptions de notification spécifiques consiste à les associer à des balises, puis à cibler ces balises. Comme indiqué dans la rubrique [Gestion des inscriptions](notification-hubs-push-notification-registration-management.md), pour recevoir des notifications Push, une application doit inscrire un handle d’appareil auprès d’un hub de notification. Une fois que l’application a créé une inscription sur un hub de notification, le back-end d’application peut lui envoyer des notifications Push. Le serveur principal d'application peut choisir les inscriptions à cibler avec une notification spécifique en procédant ainsi :

1. **Diffusion**: toutes les inscriptions dans le concentrateur de notification reçoivent la notification.
2. **Balise**: toutes les inscriptions qui contiennent la balise spécifiée reçoivent la notification.
3. **Expression de balise**: toutes les inscriptions dont le jeu de balises correspond à l'expression spécifiée reçoivent la notification.

## <a name="tags"></a>Balises

Une balise est une chaîne de 120 caractères au maximum, contenant des caractères alphanumériques ainsi que les caractères non alphanumériques suivants : « `_` », « `@` », « `#` », « `.` », « `:` », « `-` ». L'exemple suivant montre une application à partir de laquelle vous pouvez recevoir des notifications toast concernant des groupes musicaux spécifiques. Dans ce scénario, une méthode simple pour router des notifications consiste à étiqueter les inscriptions avec des balises représentant les différents groupes de musique, comme dans la figure suivante :

![Vue d’ensemble des étiquettes](./media/notification-hubs-tags-segment-push-message/notification-hubs-tags.png)

Dans la figure, le message étiqueté **Beatles** est uniquement envoyé à la tablette inscrite avec la balise **Beatles**.

Pour plus d'informations sur la création d’inscriptions pour des balises, consultez [Gestion des inscriptions](notification-hubs-push-notification-registration-management.md).

Vous pouvez envoyer des notifications à des balises à l'aide des méthodes d’envoi de notifications de la classe `Microsoft.Azure.NotificationHubs.NotificationHubClient` dans le Kit de développement logiciel (SDK) [Microsoft Azure Notification Hubs](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/) . Vous pouvez également utiliser Node.js ou les API REST Notifications Push.  Voici un exemple utilisant le Kit de développement logiciel (SDK).

```csharp
Microsoft.Azure.NotificationHubs.NotificationOutcome outcome = null;

// Windows 8.1 / Windows Phone 8.1
var toast = @"<toast><visual><binding template=""ToastText01""><text id=""1"">" +
"You requested a Beatles notification</text></binding></visual></toast>";
outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, "Beatles");

// Windows 10
toast = @"<toast><visual><binding template=""ToastGeneric""><text id=""1"">" +
"You requested a Wailers notification</text></binding></visual></toast>";
outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, "Wailers");
```

Les balises ne doivent pas être préprovisionnées et peuvent référencer plusieurs concepts propres à l’application. Par exemple, les utilisateurs de cet exemple d'application peuvent publier des commentaires sur les groupes et recevoir des notifications toast non seulement concernant les commentaires sur leurs groupes favoris, mais également pour tous les commentaires de leurs amis, quel que soit le groupe qu’ils commentent. La figure suivante illustre un exemple de ce scénario :

![Étiquettes - Amis](./media/notification-hubs-tags-segment-push-message/notification-hubs-tags2.png)

Dans cet exemple, Alice s’intéresse à l’actualité des Beatles, et Bob à celle des Wailers. Bob est également intéressé par les commentaires de Charlie, et Charlie s’intéresse aux Wailers. Lorsqu’une notification est envoyée concernant un commentaire de Charlie sur les Beatles, Notification Hubs l’envoie à Alice et Bob.

Même s’il est possible d’encoder plusieurs éléments dans des balises (par exemple, `band_Beatles` ou `follows_Charlie`), les balises sont de simples chaînes et non des propriétés avec des valeurs. Une inscription correspond uniquement à la présence ou à l’absence d’une balise spécifique.

Pour obtenir un didacticiel complet et détaillé sur la façon d'utiliser des balises pour l'envoi à des groupes d'intérêt, consultez la rubrique [Dernières nouvelles](notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md).

> [!NOTE]
> Azure Notification Hubs prend en charge un maximum de 60 balises par inscription.

## <a name="using-tags-to-target-users"></a>Utilisation de balises pour cibler des utilisateurs

Vous pouvez aussi utiliser des balises pour identifier tous les appareils associés à un utilisateur particulier. Vous pouvez marquer une inscription avec une balise contenant l’ID utilisateur, comme dans la figure suivante :

![Étiquette - Utilisateurs](./media/notification-hubs-tags-segment-push-message/notification-hubs-tags3.png)

Dans la figure, le message étiqueté `user_Alice` atteint tous les appareils marqués avec `user_Alice`.

## <a name="tag-expressions"></a>Expressions de balise

Dans certains cas, une notification doit cibler un jeu d’inscriptions identifié non pas par une balise unique, mais par une expression booléenne avec des balises.

Examinons une application de sports qui envoie un rappel à tous les abonnés habitant Boston qu’un match opposera les Red Sox aux Cardinals. Si l'application cliente inscrit des balises spécifiques à ces équipes et à ce lieu, la notification doit être ciblée pour tous les abonnés de Boston qui s’intéressent aux Red Sox ou aux Cardinals. Cette condition peut être exprimée avec l'expression booléenne suivante :

```csharp
(follows_RedSox || follows_Cardinals) && location_Boston
```

![Expressions de balise](./media/notification-hubs-tags-segment-push-message/notification-hubs-tags4.png)

Les expressions de balise prennent en charge les opérateurs booléens courants comme `AND` (`&&`), `OR` (`||`) et `NOT` (`!`) ; elles peuvent également contenir des parenthèses. Les expressions de balise utilisant uniquement des opérateurs `OR` peuvent référencer 20 balises ; une expression avec des opérateurs `AND` mais sans opérateur `OR` peut référencer 10 balises ; sinon, les expressions de balise sont limitées à 6 balises.

Voici un exemple d’envoi de notifications effectué avec des expressions de balise et le kit SDK :

```csharp
Microsoft.Azure.NotificationHubs.NotificationOutcome outcome = null;

String userTag = "(location_Boston && !follows_Cardinals)";

// Windows 8.1 / Windows Phone 8.1
var toast = @"<toast><visual><binding template=""ToastText01""><text id=""1"">" +
"You want info on the Red Sox</text></binding></visual></toast>";
outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, userTag);

// Windows 10
toast = @"<toast><visual><binding template=""ToastGeneric""><text id=""1"">" +
"You want info on the Red Sox</text></binding></visual></toast>";
outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, userTag);
```

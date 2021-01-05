---
title: Envoyer des notifications planifiées | Microsoft Docs
description: Cette rubrique décrit l'utilisation de notifications planifiées avec Azure Notification Hubs.
services: notification-hubs
documentationcenter: .net
keywords: notifications push,notification push,planification de notifications push
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: 6b718c75-75dd-4c99-aee3-db1288235c1a
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-android
ms.devlang: dotnet
ms.topic: article
ms.date: 01/04/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 01/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 56eedda7f79fedce1e34ad837c92006e5cd8f191
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88998269"
---
# <a name="how-to-send-scheduled-notifications"></a>Procédure : Envoyer des notifications planifiées

Si vous avez un scénario dans lequel vous souhaitez envoyer une notification à un moment donné dans le futur, mais que vous ne disposez pas d'un moyen simple d'activer votre code principal pour envoyer la notification. Notification Hubs de niveau Standard prend en charge une fonctionnalité qui vous permet de planifier des notifications jusqu’à 7 jours à l’avance.


## <a name="schedule-your-notifications"></a>Planifier vos notifications
Lorsque vous envoyez une notification, utilisez simplement la classe [`ScheduledNotification`](/dotnet/api/microsoft.azure.notificationhubs.schedulednotification?view=azure-dotnet#microsoft_azure_notificationhubs_schedulednotification) dans le Kit de développement logiciel (SDK) Notification Hubs, comme indiqué dans l'exemple suivant :

```csharp
Notification notification = new AppleNotification("{\"aps\":{\"alert\":\"Happy birthday!\"}}");
var scheduled = await hub.ScheduleNotificationAsync(notification, new DateTime(2014, 7, 19, 0, 0, 0));
```

## <a name="cancel-scheduled-notifications"></a>Annuler des notifications planifiées
Vous pouvez également annuler une notification précédemment planifiée à l'aide de son ID de notification :

```csharp
await hub.CancelNotificationAsync(scheduled.ScheduledNotificationId);
```

Il n'existe aucune limite quant au nombre de notifications planifiées que vous pouvez envoyer.

## <a name="next-steps"></a>Étapes suivantes

Consultez les didacticiels suivants :

 - [Notifications push vers tous les appareils inscrits](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md)
 - [Notifications Push vers des appareils spécifiques](notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md)
 - [Notifications push localisées](notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification.md)
 - [Notifications Push vers des utilisateurs spécifiques](notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md) 
 - [Notifications Push en fonction de la localisation](notification-hubs-push-bing-spatial-data-geofencing-notification.md)

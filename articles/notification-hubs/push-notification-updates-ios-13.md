---
title: Mises à jour liées à iOS 13 dans Azure Notification Hubs | Microsoft Docs
description: Découvrez les changements cassants liés à iOS 13 apportées dans Azure Notification Hubs
author: sethmanheim
ms.author: sethm
ms.date: 10/16/2019
ms.topic: article
ms.service: notification-hubs
ms.reviewer: jowargo
ms.lastreviewed: 10/16/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: df8560bec3671a9f05628ee6ed8ea95c31e9b16f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88998048"
---
# <a name="azure-notification-hubs-updates-for-ios-13"></a>Mises à jour Azure Notification Hubs pour iOS 13

Apple a récemment apporté des modifications à son service push public ; ces modifications sont principalement alignées sur la version d’iOS 13 et Xcode. Cet article décrit l’impact de ces modifications sur Azure Notification Hubs.

## <a name="apns-push-payload-changes"></a>Modifications de la charge utile d’envoi (push) APNS

### <a name="apns-push-type"></a>Type d’envoi (push) APNS

Apple demande maintenant aux développeurs d’identifier les notifications en tant qu’alerte ou notifications en arrière-plan via le nouvel en-tête `apns-push-type` dans l’API APNS. Selon la [documentation d’Apple](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/sending_notification_requests_to_apns) : « La valeur de cet en-tête doit refléter précisément le contenu de la charge utile de votre notification. En cas de non-concordance ou si l’en-tête est manquant sur les systèmes requis, Apple Push Notification Service peut retourner une erreur, retarder la remise de la notification ou la supprimer entièrement. »

Les développeurs doivent maintenant définir cet en-tête dans les applications qui envoient des notifications via Azure Notification Hubs. En raison d’une limitation technique, les clients doivent utiliser l’authentification basée sur les jetons pour les informations d’identification APNS avec les demandes qui incluent cet attribut. Si vous utilisez l’authentification basée sur les certificats pour vos informations d’identification APNS, vous devez passer à l’utilisation de l’authentification basée sur les jetons.

Les exemples de code suivants montrent comment définir cet attribut d’en-tête dans les demandes de notification envoyées via Azure Notification Hubs.

#### <a name="template-notifications---net-sdk"></a>Notifications de modèles - kit SDK .NET

```csharp
var hub = NotificationHubClient.CreateFromConnectionString(...);
var headers = new Dictionary<string, string> {{"apns-push-type", "alert"}};
var tempprop = new Dictionary<string, string> {{"message", "value"}};
var notification = new TemplateNotification(tempprop);
notification.Headers = headers;
await hub.SendNotificationAsync(notification);
```

#### <a name="native-notifications---net-sdk"></a>Notifications natives - kit SDK .NET

```csharp
var hub = NotificationHubClient.CreateFromConnectionString(...);
var headers = new Dictionary<string, string> {{"apns-push-type", "alert"}};
var notification = new AppleNotification("notification text", headers);
await hub.SendNotificationAsync(notification);
```

#### <a name="direct-rest-calls"></a>Appels REST directs

```csharp
var request = new HttpRequestMessage(method, $"<resourceUri>?api-version=2017-04");
request.Headers.Add("Authorization", createToken(resourceUri, KEY_NAME, KEY_VALUE));
request.Headers.Add("ServiceBusNotification-Format", "apple");
request.Headers.Add("apns-push-type", "alert");
```

Pour vous aider durant cette transition, quand Azure Notification Hubs détecte une notification pour laquelle `apns-push-type` n’est pas défini, le service déduit le type d’envoi (push) à partir de la demande de notification et définit automatiquement la valeur. Souvenez-vous que vous devez configurer Azure Notification Hubs pour utiliser l’authentification basée sur les jetons afin de définir l’en-tête requis. Pour plus d’informations, consultez [Authentification basée sur un jeton (HTTP/2) pour APNS](./notification-hubs-push-notification-http2-token-authentication.md).

## <a name="apns-priority"></a>Priorité APNS

Un autre changement mineur, mais qui nécessite une modification de l’application back-end qui envoie les notifications, concerne l’exigence selon laquelle, pour les notifications en arrière-plan, l’en-tête `apns-priority` doit maintenant avoir la valeur 5. De nombreuses applications attribuent à l’en-tête `apns-priority` la valeur 10 (indiquant une remise immédiate) ou ne la configurent pas et obtiennent la valeur par défaut (qui est également 10).

La définition de cette valeur sur 10 n’est plus autorisée pour les notifications en arrière-plan et vous devez définir cette valeur pour chaque demande. Apple n’effectuera pas la livraison des notifications en arrière-plan si cette valeur est manquante. Par exemple :

```csharp
var hub = NotificationHubClient.CreateFromConnectionString(...);
var headers = new Dictionary<string, string> {{"apns-push-type", "background"}, { "apns-priority", "5" }};
var notification = new AppleNotification("notification text", headers);
await hub.SendNotificationAsync(notification);
```

## <a name="sdk-changes"></a>Modifications du kit SDK

Pendant des années, les développeurs iOS ont utilisé l’attribut `description` des données `deviceToken` envoyées au délégué de jeton Push pour extraire le jeton Push qu’une application back-end utilise pour envoyer des notifications à l’appareil. Avec Xcode 11, cet attribut `description` a été remplacé par un autre format. Le code que les développeurs utilisaient pour cet attribut est désormais interrompu. Nous avons mis à jour le kit SDK Azure Notification Hubs pour prendre en compte cette modification. Vous devez donc mettre à jour le kit SDK utilisé par vos applications vers la version 2.0.4 ou ultérieure du [kit SDK iOS pour Azure Notification Hubs](https://github.com/Azure/azure-notificationhubs-ios).

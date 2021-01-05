---
author: mikeparker104
ms.author: miparker
ms.date: 06/02/2020
ms.service: notification-hubs
ms.topic: include
ms.openlocfilehash: 98c9d83424e7d347bb6e1166a3b00304d39e0cce
ms.sourcegitcommit: 971a3a63cf7da95f19808964ea9a2ccb60990f64
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/19/2020
ms.locfileid: "85095279"
---
Une [API web ASP.NET Core](https://dotnet.microsoft.com/apps/aspnet/apis) back-end est utilisée pour gérer l’[inscription d’appareils](https://docs.microsoft.com/azure/notification-hubs/notification-hubs-push-notification-registration-management#what-is-device-registration) pour le client à l’aide de l’approche d’[installation](https://docs.microsoft.com/azure/notification-hubs/notification-hubs-push-notification-registration-management#installations) la plus récente et la plus appropriée. Le service envoie également des notifications Push en mode multiplateforme. 

Ces opérations sont gérées à l’aide du [SDK Notification Hubs pour les opérations back-end](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/). Vous trouverez plus d’informations sur l’approche globale dans la documentation sur l’[inscription à partir de votre back-end d’application](https://docs.microsoft.com/azure/notification-hubs/notification-hubs-push-notification-registration-management#registration-management-from-a-backend).

---
title: Authentification basée sur un jeton (HTTP/2) pour APNS dans Azure Notification Hubs | Microsoft Docs
description: Découvrez comment utiliser la nouvelle authentification par jeton pour APNS.
services: notification-hubs
documentationcenter: .net
author: sethmanheim
manager: femila
editor: jwargo
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: dotnet
ms.topic: article
ms.date: 11/14/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 02/13/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 4f8de6389a04448579672b84e91f0bb4dd0f4ce2
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96460466"
---
# <a name="token-based-http2-authentication-for-apns"></a>Authentification basée sur un jeton (HTTP/2) pour APNS

## <a name="overview"></a>Vue d’ensemble

Cet article explique comment utiliser le nouveau protocole APNS HTTP/2 avec une authentification basée sur un jeton.

Les principaux avantages de l’utilisation du nouveau protocole sont notamment :

* La génération de jetons est relativement simple (par rapport aux certificats)
* Plus de date d’expiration : vous contrôlez vos jetons d’authentification et leur révocation
* Les charges utiles peuvent maintenant atteindre 4 Ko
* Les commentaires sont synchrones
* Vous utilisez le dernier protocole d’Apple : les certificats utilisent toujours le protocole binaire, qui est marqué pour dépréciation

L’utilisation de ce nouveau mécanisme peut être effectuée en deux étapes :

* Obtenez les informations nécessaires auprès du portail du compte de développeur Apple.
* Configurez votre hub de notification avec les nouvelles informations.

Notification Hubs est maintenant configuré pour utiliser le nouveau système d’authentification avec APNS.

Notez que si vous avez migré depuis l'utilisation des informations d'identification de certificat pour APNS, les propriétés du jeton écrasent votre certificat dans notre système, mais votre application continue de recevoir des notifications de manière transparente.

## <a name="obtaining-authentication-information-from-apple"></a>Obtention d’informations d’authentification auprès d’Apple

Pour activer l’authentification basée sur un jeton, vous devez obtenir les propriétés suivantes auprès de votre compte de développeur Apple :

### <a name="key-identifier"></a>Identificateur de clé

L’identificateur de clé peut être obtenu à partir de la page **Clés** sous **Certificats, identificateurs et profils**, dans votre compte de développeur Apple :

![Certificats](./media/notification-hubs-push-notification-http2-token-authentification/keys.png)

![Identificateurs](./media/notification-hubs-push-notification-http2-token-authentification/obtaining-auth-information-from-apple.png)

### <a name="application-identifier-and-application-name"></a>Identificateur de l’application et nom de l’application

Le nom et l'identificateur de l'application sont également disponibles dans la page **Certificats, identificateurs et profils** du compte développeur :

![Certificats et ID](./media/notification-hubs-push-notification-http2-token-authentification/app-name.png)

### <a name="configure-via-the-net-sdk-or-the-azure-portal"></a>Configurer via le Kit SDK .NET ou le portail Azure

Vous pouvez configurer votre hub pour utiliser l’authentification basée sur un jeton à l’aide de notre [dernier Kit SDK client](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs), ou dans le portail Azure. Pour activer l’authentification basée sur un jeton dans le portail, connectez-vous au portail Azure et accédez au panneau Hub de notification **Paramètres > Apple (APNS)** . Sélectionnez **Jeton** dans la propriété **Mode d’authentification** pour mettre à jour votre hub avec toutes les propriétés de jeton appropriées.

![Configurer le jeton](./media/notification-hubs-push-notification-http2-token-authentification/azure-portal-apns-settings.png)

* Entrez les propriétés que vous avez récupérées à partir de votre compte de développeur Apple.
* Choisissez le mode d’application (**Production** ou **Bac à sable (sandbox)** ).
* Cliquez sur le bouton **Enregistrer** pour mettre à jour vos informations d’identification APNS.

Les informations d'identification basées sur des jetons sont composées des champs suivants :

* **ID de la clé** : Identificateur de la clé privée générée dans le portail des développeurs Apple ; par exemple, `2USFGKSKLT`.
* **ID de l’équipe** : Aussi appelé « Préfixe » ou « Préfixe de l'application ». C'est l'identificateur de l'organisation dans le portail des développeurs Apple ; par exemple, `S4V3D7CHJR`.
* **ID de l’offre groupée** : Aussi appelé « ID de l’application ». C'est l'identificateur de l’offre groupée pour l'application ; par exemple, `com.example.myapp`. Notez que vous ne pouvez utiliser qu’une seule clé pour une application. Cette valeur est mappée à l'en-tête HTTP `apns-topic` lors de l'envoi d'une notification, et sert à cibler l'application spécifique. Vous ne pouvez pas définir explicitement la valeur de `apns-topic`.
* **Jeton** : Aussi appelé « clé » « clé privée ». Il est obtenu à partir du fichier .p8 généré sur le portail des développeurs Apple. La clé doit avoir l'APNS activé (sélectionné dans le portail des développeurs Apple lors de la génération de la clé). L’en-tête ou le pied de page PEM doit être supprimé de la valeur lorsque vous la fournissez au portail NH ou à l’API.
* **Point de terminaison** : Il s'agit d'un bouton bascule dans la volet du portail Notification Hubs, et d'un champ de chaîne dans l'API. Les valeurs valides sont `https://api.development.push.apple.com:443/3/device` ou `https://api.sandbox.push.apple.com:443/3/device`. Notification Hubs utilise cette valeur soit pour l'environnement de production, soit pour l'environnement de bac à sable (sandbox), afin d’envoyer des notifications. Elle doit correspondre à l’autorisation `aps-environment` dans l'application , sinon, les jetons d’appareils APNS générés ne correspondent pas à l'environnement, et les notifications ne sont pas envoyées.

Voici un exemple de code illustrant l’utilisation correcte :

```csharp
NamespaceManager nm = NamespaceManager.CreateFromConnectionString(_endpoint);
string token = "YOUR PRIVATE KEY HERE";
string keyId = "YOUR KEY ID HERE";
string appName = "YOUR APP NAME HERE";
string appId = "YOUR APP ID HERE";
NotificationHubDescription desc = new NotificationHubDescription("PATH TO YOUR HUB");
desc.ApnsCredential = new ApnsCredential(token, keyId, appId, appName);
desc.ApnsCredential.Endpoint = @"https://api.development.push.apple.com:443/3/device";
nm.UpdateNotificationHubAsync(desc);
```

## <a name="next-steps"></a>Étapes suivantes

* [Créer un hub de notification Azure dans le portail Azure](create-notification-hub-portal.md)
* [Configurer un hub de notification dans le portail Azure](create-notification-hub-portal.md)

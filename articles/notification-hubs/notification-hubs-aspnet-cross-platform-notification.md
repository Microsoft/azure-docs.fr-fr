---
title: Envoyer des notifications interplateformes aux utilisateurs avec Azure Notification Hubs (ASP.NET)
description: Découvrez comment utiliser des modèles Notification Hubs pour envoyer, dans une même demande, une notification indépendante de la plateforme qui cible toutes les plateformes.
services: notification-hubs
documentationcenter: ''
author: sethmanheim
manager: femila
editor: thsomasu
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows
ms.devlang: multiple
ms.topic: article
ms.date: 09/14/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 10/02/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 3f8f9357fff4773467dd93046ece8533514eb444
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92313899"
---
# <a name="send-cross-platform-notifications-with-azure-notification-hubs"></a>Envoi de notifications multiplateformes avec Azure Notification Hubs

Ce tutoriel se base sur le précédent [Envoyer des notifications à des utilisateurs spécifiques à l’aide d’Azure Notification Hubs]. Ce tutoriel décrit comment envoyer des notifications Push à tous les appareils inscrits pour un utilisateur authentifié spécifique. Cette approche nécessitait que plusieurs demandes envoient une notification à chaque plateforme cliente prise en charge. Azure Notification Hubs prend en charge des modèles avec lesquels vous pouvez spécifier le mode de réception des notifications pour un appareil déterminé. Cette méthode simplifie l’envoi de notifications interplateformes.

Cet article montre comment exploiter les modèles pour envoyer une notification qui cible toutes les plateformes. Cet article utilise une seule demande pour envoyer une notification indépendante de la plateforme. Pour plus d’informations sur les modèles, consultez [Vue d’ensemble de Notification Hubs][Templates].

> [!IMPORTANT]
> Les projets Windows Phone version 8.1 et antérieures ne sont pas pris en charge dans Visual Studio 2019. Pour en savoir plus, consultez [Plateforme cible et compatibilité dans Visual Studio 2019](/visualstudio/releases/2019/compatibility).

> [!NOTE]
> Avec Notification Hubs, un appareil peut inscrire plusieurs modèles en utilisant la même balise. Dans ce cas, un message entrant qui cible cette balise déclenche l’envoi de plusieurs notifications à destination de l’appareil (une pour chaque modèle). Ce processus vous permet d’afficher un même message dans plusieurs notifications visuelles, par exemple, sous la forme d’un badge et d’une notification toast dans une application du Windows Store.

## <a name="send-cross-platform-notifications-using-templates"></a>Envoyer des notifications multiplateformes à l’aide de modèles

Cette section utilise l’exemple de code que vous avez créé dans le tutoriel [Envoyer des notifications à des utilisateurs spécifiques à l’aide d’Azure Notification Hubs]. Vous pouvez télécharger l’exemple complet sur [GitHub](https://github.com/Azure/azure-notificationhubs-dotnet/tree/master/Samples/NotifyUsers).

Pour envoyer des notifications interplateformes en utilisant des modèles, procédez comme suit :

1. Dans **l’Explorateur de solutions** de Visual Studio, développez le dossier **Contrôleurs**, puis ouvrez le fichier *RegisterController.cs*.

1. Recherchez le bloc de code dans la méthode `Put` qui crée une inscription, puis remplacez le contenu de `switch` par le code suivant :

    ```csharp
    switch (deviceUpdate.Platform)
    {
        case "mpns":
            var toastTemplate = "<?xml version=\"1.0\" encoding=\"utf-8\"?>" +
                "<wp:Notification xmlns:wp=\"WPNotification\">" +
                    "<wp:Toast>" +
                        "<wp:Text1>$(message)</wp:Text1>" +
                    "</wp:Toast> " +
                "</wp:Notification>";
            registration = new MpnsTemplateRegistrationDescription(deviceUpdate.Handle, toastTemplate);
            break;
        case "wns":
            toastTemplate = @"<toast><visual><binding template=""ToastText01""><text id=""1"">$(message)</text></binding></visual></toast>";
            registration = new WindowsTemplateRegistrationDescription(deviceUpdate.Handle, toastTemplate);
            break;
        case "apns":
            var alertTemplate = "{\"aps\":{\"alert\":\"$(message)\"}}";
            registration = new AppleTemplateRegistrationDescription(deviceUpdate.Handle, alertTemplate);
            break;
        case "fcm":
            var messageTemplate = "{\"data\":{\"message\":\"$(message)\"}}";
            registration = new FcmTemplateRegistrationDescription(deviceUpdate.Handle, messageTemplate);
            break;
        default:
            throw new HttpResponseException(HttpStatusCode.BadRequest);
    }
    ```

    Ce code permet d’appeler la méthode propre à la plateforme pour créer une inscription de modèle et non une inscription native. Sachant que les inscriptions de modèles sont dérivées d’inscriptions natives, vous n’avez pas besoin de modifier les inscriptions existantes.

1. Dans **l’Explorateur de solutions**, dans le dossier **Contrôleurs**, ouvrez le fichier **NotificationsController.cs**. Remplacez la méthode `Post` par le code suivant :

    ```csharp
    public async Task<HttpResponseMessage> Post()
    {
        var user = HttpContext.Current.User.Identity.Name;
        var userTag = "username:" + user;

        var notification = new Dictionary<string, string> { { "message", "Hello, " + user } };
        await Notifications.Instance.Hub.SendTemplateNotificationAsync(notification, userTag);

        return Request.CreateResponse(HttpStatusCode.OK);
    }
    ```

    Ce code envoie une notification à toutes les plateformes en même temps. Vous ne spécifiez pas de charge utile native. Notification Hubs génère et remet la charge utile appropriée à chaque appareil avec la valeur de balise fournie, comme spécifié dans les modèles inscrits.

1. Republiez votre projet d’API web.

1. Réexécutez l’application cliente pour vérifier que l’inscription a abouti.

1. Vous pouvez aussi déployer l’application cliente sur un deuxième appareil, puis exécuter l’application. Une notification s’affiche sur chaque appareil.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez terminé ce didacticiel, vous trouverez des informations supplémentaires sur Notification Hubs et les modèles dans les articles suivants :

* Pour un scénario différent sur l’utilisation des modèles, consultez le tutoriel [Notifications Push vers des appareils Windows spécifiques exécutant des applications de plateforme Windows universelle][Use Notification Hubs to send breaking news].
* Pour plus d’informations sur les modèles, consultez [Vue d’ensemble de Notification Hubs][Templates].

<!-- Anchors. -->

<!-- Images. -->

<!-- URLs. -->
[Push to users ASP.NET]: notification-hubs-aspnet-backend-ios-apple-apns-notification.md
[Push to users Mobile Services]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Visual Studio 2012 Express for Windows 8]: https://visualstudio.microsoft.com/downloads/

[Use Notification Hubs to send breaking news]: notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md
[Azure Notification Hubs]: https://go.microsoft.com/fwlink/p/?LinkId=314257
[Envoyer des notifications à des utilisateurs spécifiques à l’aide d’Azure Notification Hubs]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Templates]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Notification Hub How to for Windows Store]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
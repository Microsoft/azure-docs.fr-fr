---
title: Envoyer des notifications Push localisées vers iOS à l’aide d’Azure Notification Hubs | Microsoft Docs
description: Découvrez comment utiliser des notifications Push localisées vers des appareils iOS à l’aide d’Azure Notification Hubs.
services: notification-hubs
documentationcenter: ios
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: 484914b5-e081-4a05-a84a-798bbd89d428
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: ios
ms.devlang: objective-c
ms.topic: article
ms.date: 01/04/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 01/04/2019
ms.openlocfilehash: a78d3a76e2b13a120e9e744e181c95bfcb330f27
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92313912"
---
# <a name="tutorial-send-localized-push-notifications-to-ios-using-azure-notification-hubs"></a>Tutoriel : Envoyer des notifications Push localisées vers iOS à l’aide d’Azure Notification Hubs

> [!div class="op_single_selector"]
> * [Windows Store C#](notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification.md)
> * [iOS](notification-hubs-ios-xplat-localized-apns-push-notification.md)

Ce tutoriel montre comment utiliser la fonctionnalité de [modèles](notification-hubs-templates-cross-platform-push-messages.md) d’Azure Notification Hubs pour diffuser des notifications relatives aux dernières nouvelles qui ont été localisées par langue et par appareil. Dans ce tutoriel, vous commencez avec l’application iOS que vous avez créée dans le cadre du tutoriel [Utilisation de Notification Hubs pour envoyer les dernières nouvelles]. Lorsque vous aurez terminé, vous pourrez vous inscrire aux catégories qui vous intéressent, spécifier une langue dans laquelle recevoir les notifications et recevoir uniquement des notifications Push pour les catégories sélectionnées dans cette langue.

Ce scénario comporte deux parties :

* L'application iOS permet aux appareils clients d'indiquer une langue et de s'abonner à différentes catégories de dernières nouvelles.
* Le serveur principal diffuse les notifications à l’aide des fonctionnalités de **balise** et de **modèle** d’Azure Notification Hubs.

Dans ce tutoriel, vous effectuez les étapes suivantes :

> [!div class="checklist"]
> * Mettre à jour l’interface utilisateur de l’application
> * Générer l’application iOS
> * Envoyer des notifications de modèle localisé à partir de l’application console .NET
> * Envoyer des notifications de modèle localisé à partir de l’appareil

## <a name="overview"></a>Vue d’ensemble

Dans le tutoriel [Utilisation de Notification Hubs pour envoyer les dernières nouvelles] , vous avez créé une application qui se sert de **balises** pour s'abonner aux notifications relatives à différentes catégories de nouvelles. Cependant, de nombreuses applications sont destinées à plusieurs marchés et doivent donc être localisées. Cela signifie que le contenu des notifications proprement dites doit lui aussi être localisé et envoyé au bon ensemble d’appareils. Ce tutoriel vous montre comment utiliser la fonctionnalité de **modèle** de Notification Hubs pour facilement envoyer des notifications de dernières nouvelles localisées.

> [!NOTE]
> Pour envoyer des notifications localisées, vous pouvez notamment créer plusieurs versions de chaque balise. Par exemple, pour prendre en charge l'anglais, le français et le mandarin, vous auriez besoin de trois balises différentes pour les nouvelles internationales : « world_en », « world_fr » et « world_ch ». Il faudrait ensuite que vous envoyiez une version localisée des nouvelles internationales à chacune de ces balises. Dans cette rubrique, vous utilisez des modèles afin d'éviter la prolifération de balises et d'éliminer la nécessité d'envoyer plusieurs messages.

Les modèles permettent de spécifier comment un appareil particulier reçoit une notification. Le modèle spécifie le format de charge utile exact en se référant aux propriétés qui font partie du message envoyé par le serveur principal de votre application. Aux fins de notre exemple, vous allez envoyer un message de paramètres régionaux contenant toutes les langues prises en charge :

```json
{
    "News_English": "...",
    "News_French": "...",
    "News_Mandarin": "..."
}
```

Ensuite, vous allez vous assurer que les appareils s'inscrivent avec un modèle qui se réfère à la bonne propriété. Par exemple, une application iOS qui souhaite s’abonner aux nouvelles françaises utilisera la syntaxe suivante :

```json
{
    aps: {
        alert: "$(News_French)"
    }
}
```

Pour plus d’informations sur les modèles, consultez l’article [Modèles](notification-hubs-templates-cross-platform-push-messages.md).

## <a name="prerequisites"></a>Prérequis

* Compléter le tutoriel [Notifications Push à des appareils iOS spécifique](notification-hubs-ios-xplat-segmented-apns-push-notification.md) et rendre le code disponible car ce tutoriel s’appuie directement sur ce code.
* Visual Studio 2019 est facultatif.

## <a name="update-the-app-user-interface"></a>Mettre à jour l’interface utilisateur de l’application

Dans cette section, vous allez modifier l’application de dernières nouvelles que vous avez créée à la rubrique [Utilisation de Notification Hubs pour envoyer les dernières nouvelles] pour envoyer les dernières nouvelles localisées à l’aide de modèles.

Dans votre `MainStoryboard_iPhone.storyboard`, ajoutez un contrôle segmenté avec les trois langues suivantes : anglais, français et mandarin.

![Création de la table de montage séquentiel de l’interface utilisateur iOS][13]

Puis, assurez-vous d’ajouter un IBOutlet dans votre ViewController.h comme indiqué dans l’image suivante :

![Créer des sorties pour les commutateurs][14]

## <a name="build-the-ios-app"></a>Générer l’application iOS

1. Dans `Notification.h`, ajoutez la méthode `retrieveLocale`, puis modifiez le magasin et les méthodes d’abonnement comme indiqué dans le code suivant :

    ```objc
    - (void) storeCategoriesAndSubscribeWithLocale:(int) locale categories:(NSSet*) categories completion: (void (^)(NSError* error))completion;

    - (void) subscribeWithLocale:(int) locale categories:(NSSet*) categories completion:(void (^)(NSError *))completion;

    - (NSSet*) retrieveCategories;

    - (int) retrieveLocale;
    ```
    Dans `Notification.m`, modifiez la méthode `storeCategoriesAndSubscribe`, en ajoutant le paramètre `locale` et en le stockant dans les valeurs par défaut de l’utilisateur :

    ```objc
    - (void) storeCategoriesAndSubscribeWithLocale:(int) locale categories:(NSSet *)categories completion:(void (^)(NSError *))completion {
        NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];
        [defaults setValue:[categories allObjects] forKey:@"BreakingNewsCategories"];
        [defaults setInteger:locale forKey:@"BreakingNewsLocale"];
        [defaults synchronize];

        [self subscribeWithLocale: locale categories:categories completion:completion];
    }
    ```

    Puis, modifiez la méthode *subscribe* afin d'inclure les paramètres régionaux :

    ```objc
    - (void) subscribeWithLocale: (int) locale categories:(NSSet *)categories completion:(void (^)(NSError *))completion{
        SBNotificationHub* hub = [[SBNotificationHub alloc] initWithConnectionString:@"<connection string>" notificationHubPath:@"<hub name>"];

        NSString* localeString;
        switch (locale) {
            case 0:
                localeString = @"English";
                break;
            case 1:
                localeString = @"French";
                break;
            case 2:
                localeString = @"Mandarin";
                break;
        }

        NSString* template = [NSString stringWithFormat:@"{\"aps\":{\"alert\":\"$(News_%@)\"},\"inAppMessage\":\"$(News_%@)\"}", localeString, localeString];

        [hub registerTemplateWithDeviceToken:self.deviceToken name:@"localizednewsTemplate" jsonBodyTemplate:template expiryTemplate:@"0" tags:categories completion:completion];
    }
    ```

    Vous utilisez la méthode `registerTemplateWithDeviceToken` au lieu de `registerNativeWithDeviceToken`. Quand vous inscrivez un modèle, vous devez fournir le modèle json ainsi qu’un nom pour le modèle (car l’application peut vouloir inscrire différents modèles). Prenez soin d’inscrire vos catégories sous forme de balises, car nous voulons être certains de recevoir les notifications pour ces nouvelles.

    Ajoutez une méthode pour extraire les paramètres régionaux des paramètres utilisateur par défaut :

    ```objc
    - (int) retrieveLocale {
        NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];

        int locale = [defaults integerForKey:@"BreakingNewsLocale"];

        return locale < 0?0:locale;
    }
    ```

2. La classe `Notifications` étant modifiée, vous devez vous assurer que le paramètre `ViewController` utilise le nouveau paramètre `UISegmentControl`. Ajoutez la ligne suivante dans la méthode `viewDidLoad` pour garantir l’affichage des paramètres régionaux sélectionnés :

    ```objc
    self.Locale.selectedSegmentIndex = [notifications retrieveLocale];
    ```

    Ensuite, dans la méthode `subscribe`, changez votre appel à `storeCategoriesAndSubscribe` à l’aide du code suivant :

    ```objc
    [notifications storeCategoriesAndSubscribeWithLocale: self.Locale.selectedSegmentIndex categories:[NSSet setWithArray:categories] completion: ^(NSError* error) {
        if (!error) {
            UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Notification" message:
                                    @"Subscribed!" delegate:nil cancelButtonTitle:
                                    @"OK" otherButtonTitles:nil, nil];
            [alert show];
        } else {
            NSLog(@"Error subscribing: %@", error);
        }
    }];
    ```

3. Enfin, vous devez mettre à jour la méthode `didRegisterForRemoteNotificationsWithDeviceToken` dans votre AppDelegate.m afin que vous puissiez correctement actualiser votre abonnement quand votre application démarre. Changez votre appel à la méthode `subscribe` de notifications à l’aide du code suivant :

    ```obj-c
    NSSet* categories = [self.notifications retrieveCategories];
    int locale = [self.notifications retrieveLocale];
    [self.notifications subscribeWithLocale: locale categories:categories completion:^(NSError* error) {
        if (error != nil) {
            NSLog(@"Error registering for notifications: %@", error);
        }
    }];
    ```

## <a name="optional-send-localized-template-notifications-from-net-console-app"></a>(facultatif) Envoyer des notifications de modèle localisé à partir de l’application console .NET

[!INCLUDE [notification-hubs-localized-back-end](../../includes/notification-hubs-localized-back-end.md)]

## <a name="optional-send-localized-template-notifications-from-the-device"></a>(facultatif) Envoyer des notifications de modèle localisé à partir de l’appareil

Si vous n’avez pas accès à Visual Studio ou si vous souhaitez simplement essayer d’envoyer les notification de modèle localisé directement depuis l’application sur votre appareil. Vous pouvez ajouter les paramètres de modèle localisé à la méthode `SendNotificationRESTAPI` que vous avez défini précédemment, dans le tutoriel.

```objc
- (void)SendNotificationRESTAPI:(NSString*)categoryTag
{
    NSURLSession* session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration
                                defaultSessionConfiguration] delegate:nil delegateQueue:nil];

    NSString *json;

    // Construct the messages REST endpoint
    NSURL* url = [NSURL URLWithString:[NSString stringWithFormat:@"%@%@/messages/%@", HubEndpoint,
                                        HUBNAME, API_VERSION]];

    // Generated the token to be used in the authorization header.
    NSString* authorizationToken = [self generateSasToken:[url absoluteString]];

    //Create the request to add the template notification message to the hub
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    [request setHTTPMethod:@"POST"];

    // Add the category as a tag
    [request setValue:categoryTag forHTTPHeaderField:@"ServiceBusNotification-Tags"];

    // Template notification
    json = [NSString stringWithFormat:@"{\"messageParam\":\"Breaking %@ News : %@\","
            \"News_English\":\"Breaking %@ News in English : %@\","
            \"News_French\":\"Breaking %@ News in French : %@\","
            \"News_Mandarin\":\"Breaking %@ News in Mandarin : %@\","
            categoryTag, self.notificationMessage.text,
            categoryTag, self.notificationMessage.text,  // insert English localized news here
            categoryTag, self.notificationMessage.text,  // insert French localized news here
            categoryTag, self.notificationMessage.text]; // insert Mandarin localized news here

    // Signify template notification format
    [request setValue:@"template" forHTTPHeaderField:@"ServiceBusNotification-Format"];

    // JSON Content-Type
    [request setValue:@"application/json;charset=utf-8" forHTTPHeaderField:@"Content-Type"];

    //Authenticate the notification message POST request with the SaS token
    [request setValue:authorizationToken forHTTPHeaderField:@"Authorization"];

    //Add the notification message body
    [request setHTTPBody:[json dataUsingEncoding:NSUTF8StringEncoding]];

    // Send the REST request
    NSURLSessionDataTask* dataTask = [session dataTaskWithRequest:request
                completionHandler:^(NSData *data, NSURLResponse *response, NSError *error)
        {
        NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*) response;
            if (error || httpResponse.statusCode != 200)
            {
                NSLog(@"\nError status: %d\nError: %@", httpResponse.statusCode, error);
            }
            if (data != NULL)
            {
                //xmlParser = [[NSXMLParser alloc] initWithData:data];
                //[xmlParser setDelegate:self];
                //[xmlParser parse];
            }
        }];

    [dataTask resume];
}
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez envoyé des notifications localisées à des appareils iOS. Pour savoir comment envoyer des notifications Push à des utilisateurs spécifiques d’applications iOS, passez au tutoriel suivant :

> [!div class="nextstepaction"]
>[Notifications Push vers des utilisateurs spécifiques](notification-hubs-aspnet-backend-ios-apple-apns-notification.md)

<!-- Images. -->
[13]: ./media/notification-hubs-ios-send-localized-breaking-news/ios_localized1.png
[14]: ./media/notification-hubs-ios-send-localized-breaking-news/ios_localized2.png

<!-- URLs. -->
[How To: Service Bus Notification Hubs (iOS Apps)]: /previous-versions/azure/reference/dn223264(v=azure.100)
[Utilisation de Notification Hubs pour envoyer les dernières nouvelles]: notification-hubs-ios-xplat-segmented-apns-push-notification.md
[Mobile Service]: /develop/mobile/tutorials/get-started
[Notify users with Notification Hubs: ASP.NET]: notification-hubs-aspnet-backend-ios-apple-apns-notification.md
[Notify users with Notification Hubs: Mobile Services]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Submit an app page]: https://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: https://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: https://go.microsoft.com/fwlink/p/?LinkId=262253
[Get started with Mobile Services]: /develop/mobile/tutorials/get-started/#create-new-service
[Get started with data]: /develop/mobile/tutorials/get-started-with-data-ios
[Get started with authentication]: /develop/mobile/tutorials/get-started-with-users-ios
[Get started with push notifications]: /develop/mobile/tutorials/get-started-with-push-ios
[Push notifications to app users]: /develop/mobile/tutorials/push-notifications-to-users-ios
[Authorize users with scripts]: /develop/mobile/tutorials/authorize-users-in-scripts-ios
[JavaScript and HTML]: ../get-started-with-push-js.md
[Windows Developer Preview registration steps for Mobile Services]: ../mobile-services-windows-developer-preview-registration.md
[wns object]: /previous-versions/azure/reference/jj860484(v=azure.100)
[Notification Hubs Guidance]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Notification Hubs How-To for iOS]: /previous-versions/azure/reference/dn223264(v=azure.100)
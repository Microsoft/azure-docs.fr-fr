---
title: Envoyer des notifications Push vers des appareils iOS spécifiques avec Azure Notification Hubs | Microsoft Docs
description: Dans ce didacticiel, vous découvrirez comment utiliser Azure Notification Hubs pour envoyer des notifications Push à des appareils iOS spécifiques.
services: notification-hubs
documentationcenter: ios
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-ios
ms.devlang: objective-c
ms.topic: article
ms.date: 11/07/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 11/07/2019
ms.openlocfilehash: 585bdfdd7033f75e5beeba7246c8fbdd03a5e6e8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86530030"
---
# <a name="tutorial-send-push-notifications-to-specific-ios-devices-using-azure-notification-hubs"></a>Tutoriel : Envoyer des notifications Push vers des appareils iOS spécifiques avec Azure Notification Hubs

[!INCLUDE [notification-hubs-selector-breaking-news](../../includes/notification-hubs-selector-breaking-news.md)]

## <a name="overview"></a>Vue d’ensemble

Ce didacticiel vous montre comment utiliser Azure Notification Hubs pour diffuser des notifications relatives aux dernières nouvelles vers une application iOS. Lorsque vous avez terminé, vous pouvez vous inscrire aux catégories de dernières nouvelles qui vous intéressent et recevoir uniquement des notifications Push pour ces catégories. Ce scénario est un modèle courant pour de nombreuses applications pour lesquelles des notifications doivent être envoyées à des groupes d’utilisateurs qui ont signalé antérieurement un intérêt, par exemple, lecteur RSS, applications pour fans de musique, etc.

Les scénarios de diffusion sont activés en incluant une ou plusieurs *balises* durant la création d’une inscription dans le hub de notification. Lorsque des notifications sont envoyées à une balise, les appareils qui se sont inscrits à cette balise reçoivent la notification. Les balises étant de simples chaînes, il n’est pas nécessaire de les mettre en service à l’avance. Pour plus d’informations sur les balises, consultez [Routage et expressions de balise Notification Hubs](notification-hubs-tags-segment-push-message.md).

Dans ce tutoriel, vous effectuez les étapes suivantes :

> [!div class="checklist"]
> * Ajouter une sélection de catégories à l’application
> * Envoyer des notifications avec balises
> * Envoyer des notifications à partir de l’appareil
> * Exécution de l’application et génération de notifications

## <a name="prerequisites"></a>Prérequis

Cette rubrique s’appuie sur l’application que vous avez créée dans le [Tutoriel : Notifications Push vers des applications iOS avec Azure Notification Hubs][get-started]. Avant de commencer ce tutoriel, vous devez suivre le [Tutoriel : Notifications Push vers des applications iOS avec Azure Notification Hubs][get-started].

## <a name="add-category-selection-to-the-app"></a>Ajout d’une sélection de catégories à l’application

La première étape consiste à ajouter à votre storyboard existant les éléments d'interface utilisateur qui permettent à l'utilisateur de sélectionner les catégories à inscrire. Les catégories sélectionnées par un utilisateur sont stockées sur l'appareil. Lorsque l'application démarre, une inscription d'appareil est créée dans votre Notification Hub avec les catégories sélectionnées sous forme de balises.

1. Dans votre **MainStoryboard_iPhone.storyboard**, ajoutez les composants suivants à partir de la bibliothèque d’objets :

   * une étiquette intitulée « Dernières nouvelles » ;
   * des étiquettes portant les intitulés de catégories « Monde », « Politiques », « Entreprise », « Technologies », « Science », « Sports » ;
   * six commutateurs, un par catégorie, chacun défini sur **l’État** **désactivé** par défaut ;
   * un bouton intitulé « S’abonner ».

     Votre storyboard doit ressembler à ce qui suit :

     ![Générateur d’interface Xcode][3]

2. Dans l'éditeur de l'Assistant, créez des outlets pour tous les commutateurs et appelez-les "WorldSwitch", "PoliticsSwitch", "BusinessSwitch", "TechnologySwitch", "ScienceSwitch", "SportsSwitch".

3. Créez une Action pour le bouton intitulé `subscribe` ; votre `ViewController.h` doit contenir le code suivant :

    ```objc
    @property (weak, nonatomic) IBOutlet UISwitch *WorldSwitch;
    @property (weak, nonatomic) IBOutlet UISwitch *PoliticsSwitch;
    @property (weak, nonatomic) IBOutlet UISwitch *BusinessSwitch;
    @property (weak, nonatomic) IBOutlet UISwitch *TechnologySwitch;
    @property (weak, nonatomic) IBOutlet UISwitch *ScienceSwitch;
    @property (weak, nonatomic) IBOutlet UISwitch *SportsSwitch;

    - (IBAction)subscribe:(id)sender;
    ```

4. Créez une **classe Cocoa Touch** appelée `Notifications`. Copiez le code suivant dans la section de l’interface du fichier Notifications.h :

    ```objc
    @property NSData* deviceToken;

    - (id)initWithConnectionString:(NSString*)listenConnectionString HubName:(NSString*)hubName;

    - (void)storeCategoriesAndSubscribeWithCategories:(NSArray*)categories
                completion:(void (^)(NSError* error))completion;

    - (NSSet*)retrieveCategories;

    - (void)subscribeWithCategories:(NSSet*)categories completion:(void (^)(NSError *))completion;
    ```

5. Ajoutez la directive import suivante au fichier Notifications.m :

    ```objc
    #import <WindowsAzureMessaging/WindowsAzureMessaging.h>
    ```

6. Copiez le code suivant dans la section d’implémentation du fichier Notifications.m.

    ```objc
    SBNotificationHub* hub;

    - (id)initWithConnectionString:(NSString*)listenConnectionString HubName:(NSString*)hubName{

        hub = [[SBNotificationHub alloc] initWithConnectionString:listenConnectionString
                                    notificationHubPath:hubName];

        return self;
    }

    - (void)storeCategoriesAndSubscribeWithCategories:(NSSet *)categories completion:(void (^)(NSError *))completion {
        NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];
        [defaults setValue:[categories allObjects] forKey:@"BreakingNewsCategories"];

        [self subscribeWithCategories:categories completion:completion];
    }

    - (NSSet*)retrieveCategories {
        NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];

        NSArray* categories = [defaults stringArrayForKey:@"BreakingNewsCategories"];

        if (!categories) return [[NSSet alloc] init];
        return [[NSSet alloc] initWithArray:categories];
    }

    - (void)subscribeWithCategories:(NSSet *)categories completion:(void (^)(NSError *))completion
    {
        //[hub registerNativeWithDeviceToken:self.deviceToken tags:categories completion: completion];

        NSString* templateBodyAPNS = @"{\"aps\":{\"alert\":\"$(messageParam)\"}}";

        [hub registerTemplateWithDeviceToken:self.deviceToken name:@"simpleAPNSTemplate" 
            jsonBodyTemplate:templateBodyAPNS expiryTemplate:@"0" tags:categories completion:completion];
    }
    ```

    Cette classe utilise le stockage local pour stocker et récupérer les catégories de nouvelles que cet appareil reçoit. Elle comporte également une méthode pour s’inscrire à ces catégories à l’aide de l’inscription de [modèle](notification-hubs-templates-cross-platform-push-messages.md) .

7. Dans le fichier `AppDelegate.h`, ajoutez une instruction import pour `Notifications.h` et ajoutez une propriété d’une instance de la classe `Notifications` :

    ```objc
    #import "Notifications.h"

    @property (nonatomic) Notifications* notifications;
    ```

8. Dans la méthode `didFinishLaunchingWithOptions` du fichier `AppDelegate.m`, ajoutez ce code pour initialiser l’instance de notifications au début de la méthode.  
    Dans `HUBNAME` et `HUBLISTENACCESS` (définis dans `hubinfo.h`), le nom du hub de notification et la chaîne de connexion pour *DefaultListenSharedAccessSignature* obtenus précédemment doivent avoir remplacé les espaces réservés `<hub name>` et `<connection string with listen access>`.

    ```objc
    self.notifications = [[Notifications alloc] initWithConnectionString:HUBLISTENACCESS HubName:HUBNAME];
    ```

    > [!NOTE]
    > Les informations d’identification distribuées avec une application cliente n’étant généralement pas sécurisées, vous ne devez distribuer que la clé d’accès d’écoute avec votre application cliente. L'accès d'écoute permet à votre application de s'inscrire à des notifications, mais les inscriptions existantes ne peuvent pas être modifiées et les notifications ne peuvent pas être envoyées. La clé d’accès complet est utilisée dans un service de serveur principal sécurisé pour l’envoi de notifications et la modification d’inscriptions existantes.

9. Dans la méthode `didRegisterForRemoteNotificationsWithDeviceToken` du fichier `AppDelegate.m`, remplacez le code de la méthode par le code suivant pour transmettre le jeton d’appareil à la classe `notifications`. La classe `notifications` effectue l’enregistrement pour les notifications avec les catégories. Si l’utilisateur modifie les sélections de catégorie, appelez la méthode `subscribeWithCategories` en réponse au bouton **S’abonner** pour mettre à jour les sections.

    > [!NOTE]
    > Étant donné que le jeton d’appareil attribué par Apple Push Notification Service (APN) peut être modifié à tout moment, vous devez vous inscrire fréquemment aux notifications pour éviter les échecs de notification. Cet exemple s’inscrit aux notifications chaque fois que l’application démarre. Pour les applications exécutées fréquemment, plus d'une fois par jour, vous pouvez probablement ignorer l'inscription afin de préserver la bande passante si moins d'un jour s'est écoulé depuis l'inscription précédente.

    ```objc
    self.notifications.deviceToken = deviceToken;

    // Retrieves the categories from local storage and requests a registration for these categories
    // each time the app starts and performs a registration.

    NSSet* categories = [self.notifications retrieveCategories];
    [self.notifications subscribeWithCategories:categories completion:^(NSError* error) {
        if (error != nil) {
            NSLog(@"Error registering for notifications: %@", error);
        }
    }];
    ```

    À ce stade, il ne doit pas y avoir d’autre code dans la méthode `didRegisterForRemoteNotificationsWithDeviceToken`.

10. Les méthodes suivantes doivent déjà être présentes dans `AppDelegate.m` depuis la fin du tutoriel [Prise en main de Notification Hubs][get-started]. Si ce n’est pas le cas, veuillez les ajouter.

    ```objc
    - (void)MessageBox:(NSString *)title message:(NSString *)messageText
    {

        UIAlertView *alert = [[UIAlertView alloc] initWithTitle:title message:messageText delegate:self
            cancelButtonTitle:@"OK" otherButtonTitles: nil];
        [alert show];
    }

    - (void)application:(UIApplication *)application didReceiveRemoteNotification:
       (NSDictionary *)userInfo {
       NSLog(@"%@", userInfo);
       [self MessageBox:@"Notification" message:[[userInfo objectForKey:@"aps"] valueForKey:@"alert"]];
     }
    ```

    Cette méthode gère les notifications reçues lorsque l'application est en cours d'exécution en affichant tout simplement une **UIAlert**.

11. Dans `ViewController.m`, ajoutez une instruction `import` pour `AppDelegate.h` et copiez le code suivant dans la méthode `subscribe` générée par XCode. Ce code met à jour l’inscription aux notifications pour utiliser les nouvelles balises de catégories choisies par l’utilisateur dans l’interface utilisateur.

    ```objc
    #import "Notifications.h"

    NSMutableArray* categories = [[NSMutableArray alloc] init];

    if (self.WorldSwitch.isOn) [categories addObject:@"World"];
    if (self.PoliticsSwitch.isOn) [categories addObject:@"Politics"];
    if (self.BusinessSwitch.isOn) [categories addObject:@"Business"];
    if (self.TechnologySwitch.isOn) [categories addObject:@"Technology"];
    if (self.ScienceSwitch.isOn) [categories addObject:@"Science"];
    if (self.SportsSwitch.isOn) [categories addObject:@"Sports"];

    Notifications* notifications = [(AppDelegate*)[[UIApplication sharedApplication]delegate] notifications];

    [notifications storeCategoriesAndSubscribeWithCategories:categories completion: ^(NSError* error) {
        if (!error) {
            UIAlertView *alert = [[UIAlertView alloc] initWithTitle:"Notification" message:"Subscribed" delegate:self
            cancelButtonTitle:@"OK" otherButtonTitles: nil];
            [alert show];
        } else {
            NSLog(@"Error subscribing: %@", error);
        }
    }];
    ```

    Cette méthode crée un `NSMutableArray` de catégories et utilise la classe `Notifications` pour stocker la liste dans le stockage local et inscrit les balises correspondantes auprès de votre hub de notification. Lorsque des catégories sont modifiées, l'inscription est à nouveau créée avec les nouvelles catégories.

12. Dans `ViewController.m`, ajoutez le code suivant dans la méthode `viewDidLoad` pour définir l’interface utilisateur en fonction des catégories enregistrées.

    ```objc
    // This updates the UI on startup based on the status of previously saved categories.

    Notifications* notifications = [(AppDelegate*)[[UIApplication sharedApplication]delegate] notifications];

    NSSet* categories = [notifications retrieveCategories];

    if ([categories containsObject:@"World"]) self.WorldSwitch.on = true;
    if ([categories containsObject:@"Politics"]) self.PoliticsSwitch.on = true;
    if ([categories containsObject:@"Business"]) self.BusinessSwitch.on = true;
    if ([categories containsObject:@"Technology"]) self.TechnologySwitch.on = true;
    if ([categories containsObject:@"Science"]) self.ScienceSwitch.on = true;
    if ([categories containsObject:@"Sports"]) self.SportsSwitch.on = true;
    ```

L’application peut désormais stocker un ensemble de catégories dans le stockage local de l’appareil utilisé pour s’inscrire auprès du Notification Hub au démarrage de l’application. L’utilisateur peut changer la sélection des catégories au démarrage et cliquer sur la méthode `subscribe` pour mettre à jour l’inscription de l’appareil. Ensuite, vous mettez à jour l’application pour envoyer les notifications de dernières nouvelles directement dans l’application elle-même.

## <a name="optional-send-tagged-notifications"></a>(Facultatif) Envoyer des notifications avec balises

Si vous n’avez pas accès à Visual Studio, vous pouvez passer à la section suivante et envoyer des notifications à partir de l’application elle-même. Vous pouvez également envoyer la notification de modèle concernée sur le [Azure portal], dans l’onglet Débogage de votre centre Notification Hubs.

[!INCLUDE [notification-hubs-send-categories-template](../../includes/notification-hubs-send-categories-template.md)]

## <a name="optional-send-notifications-from-the-device"></a>(Facultatif) Envoyer des notifications depuis l’appareil

Normalement, les notifications doivent être envoyées par un service principal. Toutefois, vous pouvez envoyer des notifications de dernières nouvelles directement à partir de l’application. Pour cela, vous mettez à jour la méthode `SendNotificationRESTAPI` que vous avez définie dans le didacticiel [Prise en main de Notification Hubs][get-started].

1. Dans `ViewController.m`, mettez à jour la méthode `SendNotificationRESTAPI` comme suit pour qu’elle accepte un paramètre pour la balise de catégorie et envoie une notification de [modèle](notification-hubs-templates-cross-platform-push-messages.md) adaptée.

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
        json = [NSString stringWithFormat:@"{\"messageParam\":\"Breaking %@ News : %@\"}",
                categoryTag, self.notificationMessage.text];

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

2. Dans `ViewController.m`, mettez à jour l’action `Send Notification` comme indiqué dans le code suivant. Ainsi, les notifications sont envoyées à l’aide de chaque balise de manière individuelle vers plusieurs plateformes.

    ```objc
    - (IBAction)SendNotificationMessage:(id)sender
    {
        self.sendResults.text = @"";

        NSArray* categories = [NSArray arrayWithObjects: @"World", @"Politics", @"Business",
                                @"Technology", @"Science", @"Sports", nil];

        // Lets send the message as breaking news for each category to WNS, FCM, and APNS
        // using a template.
        for(NSString* category in categories)
        {
            [self SendNotificationRESTAPI:category];
        }
    }
    ```

3. Régénérez votre projet et vérifiez qu’il n’existe aucune erreur de génération.

## <a name="run-the-app-and-generate-notifications"></a>Exécution de l’application et génération de notifications

1. Cliquez sur le bouton Exécuter pour générer le projet et démarrer l’application. Sélectionnez certaines options de dernières nouvelles pour vous y abonner, puis appuyez sur le bouton **S’abonner** . Vous devez voir une boîte de dialogue indiquant les notifications auxquelles vous êtes abonné.

    ![Exemple de notification sur iOS][1]

    Lorsque vous sélectionnez **S’abonner**, l'application convertit les catégories sélectionnées en balises et demande une nouvelle inscription de l'appareil aux balises sélectionnées depuis le hub de notification.

2. Entrez un message à envoyer comme dernières nouvelles, puis appuyez sur le bouton **Envoyer une notification** . Vous pouvez également exécuter l’application console .NET pour générer des notifications.

    ![Changer les préférences de notification sur iOS][2]

3. Chaque appareil abonné aux dernières nouvelles reçoit les notifications de dernières nouvelles que vous venez d’envoyer.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez envoyé des notifications de diffusion à des appareils iOS spécifiques que sont enregistrés pour les catégories. Pour savoir comment envoyer (push) des notifications localisées, passez au didacticiel suivant :

> [!div class="nextstepaction"]
>[Notifications push localisées](notification-hubs-ios-xplat-localized-apns-push-notification.md)

<!-- Images. -->
[1]: ./media/notification-hubs-ios-send-breaking-news/notification-hub-breakingnews-subscribed.png
[2]: ./media/notification-hubs-ios-send-breaking-news/notification-hub-breakingnews-ios1.png
[3]: ./media/notification-hubs-ios-send-breaking-news/notification-hub-breakingnews-ios2.png

<!-- URLs. -->
[How To: Service Bus Notification Hubs (iOS Apps)]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Use Notification Hubs to broadcast localized breaking news]: notification-hubs-ios-xplat-localized-apns-push-notification.md
[Mobile Service]: /develop/mobile/tutorials/get-started
[Notify users with Notification Hubs]: notification-hubs-aspnet-backend-ios-notify-users.md
[Notification Hubs Guidance]: /previous-versions/azure/azure-services/dn530749(v=azure.100)
[Notification Hubs How-To for iOS]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[get-started]: ios-sdk-get-started.md
[Azure portal]: https://portal.azure.com

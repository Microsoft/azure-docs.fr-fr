---
title: Inscription de l’utilisateur actif aux notifications Push au moyen de l’API Web | Microsoft Docs
description: Découvrez comment demander l’inscription aux notifications Push dans une application iOS avec Azure Notification Hubs quand l’inscription est réalisée par l’API web ASP.NET.
services: notification-hubs
documentationcenter: ios
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: ios
ms.devlang: objective-c
ms.topic: article
ms.date: 01/04/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 01/04/2019
ms.openlocfilehash: 0819f5196fffca25a840dc16d1df04cdd0a55029
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86223310"
---
# <a name="register-the-current-user-for-push-notifications-by-using-aspnet"></a>Inscription de l’utilisateur actif aux notifications Push à l’aide d’ASP.NET

> [!div class="op_single_selector"]
> * [iOS](notification-hubs-ios-aspnet-register-user-from-backend-to-push-notification.md)

## <a name="overview"></a>Vue d’ensemble

Cette rubrique montre comment demander une inscription aux notifications Push avec Azure Notification Hubs lorsque l’inscription est réalisée par l’API Web ASP.NET. Cette rubrique s'inscrit dans le prolongement du didacticiel [Notification des utilisateurs via Notification Hubs]. Vous devez avoir suivi les étapes de ce didacticiel permettant de créer le service mobile authentifié. Pour plus d'informations sur les scénarios de notification des utilisateurs, consultez la rubrique [Notification des utilisateurs via Notification Hubs].

## <a name="update-your-app"></a>Mise à jour de votre application

1. Dans MainStoryboard_iPhone.storyboard, ajoutez les composants suivants de la bibliothèque d'objets :

   * **Étiquette** : « Envoi d’une notification Push à l’utilisateur avec Notification Hubs »
   * **Étiquette** : « InstallationId »
   * **Étiquette** : « Utilisateur »
   * **Champ texte** : « Utilisateur »
   * **Étiquette** : « Mot de passe »
   * **Champ texte** : « Mot de passe »
   * **Bouton** : « Connexion »

     À ce stade, votre storyboard a normalement l’aspect suivant :

     ![Capture d’écran de l’application MainStoryboard_iPhone.storyboard avec l’ajout des composants.][0]

2. Dans l’éditeur de l’Assistant, créez des outlets pour tous les contrôles commutés et appelez-les, connectez les champs texte au View Controller (délegué), puis créez une **Action** pour le bouton **login**.

    ![Capture d’écran de l’éditeur assistant dans l’application MainStoryboard_iPhone.storyboard][1]

    Le fichier BreakingNewsViewController.h doit désormais contenir le code suivant :

    ```objc
    @property (weak, nonatomic) IBOutlet UILabel *installationId;
    @property (weak, nonatomic) IBOutlet UITextField *User;
    @property (weak, nonatomic) IBOutlet UITextField *Password;

    - (IBAction)login:(id)sender;
    ```
3. Créez une classe nommée `DeviceInfo`, puis copiez le code suivant dans la section de l’interface du fichier DeviceInfo.h :

    ```objc
    @property (readonly, nonatomic) NSString* installationId;
    @property (nonatomic) NSData* deviceToken;
    ```
4. Copiez le code suivant dans la section d'implémentation du fichier DeviceInfo.m :

    ```objc
    @synthesize installationId = _installationId;

    - (id)init {
        if (!(self = [super init]))
            return nil;

        NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
        _installationId = [defaults stringForKey:@"PushToUserInstallationId"];
        if(!_installationId) {
            CFUUIDRef newUUID = CFUUIDCreate(kCFAllocatorDefault);
            _installationId = (__bridge_transfer NSString *)CFUUIDCreateString(kCFAllocatorDefault, newUUID);
            CFRelease(newUUID);

            //store the install ID so we don't generate a new one next time
            [defaults setObject:_installationId forKey:@"PushToUserInstallationId"];
            [defaults synchronize];
        }

        return self;
    }

    - (NSString*)getDeviceTokenInHex {
        const unsigned *tokenBytes = [[self deviceToken] bytes];
        NSString *hexToken = [NSString stringWithFormat:@"%08X%08X%08X%08X%08X%08X%08X%08X",
                                ntohl(tokenBytes[0]), ntohl(tokenBytes[1]), ntohl(tokenBytes[2]),
                                ntohl(tokenBytes[3]), ntohl(tokenBytes[4]), ntohl(tokenBytes[5]),
                                ntohl(tokenBytes[6]), ntohl(tokenBytes[7])];
        return hexToken;
    }
    ```

5. Dans PushToUserAppDelegate.h, ajoutez le singleton de propriété suivant :

    ```objc
    @property (strong, nonatomic) DeviceInfo* deviceInfo;
    ```
6. Dans la méthode `didFinishLaunchingWithOptions` du fichier PushToUserAppDelegate.m, ajoutez le code suivant :

    ```objc
    self.deviceInfo = [[DeviceInfo alloc] init];

    [[UIApplication sharedApplication] registerForRemoteNotificationTypes: UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound];
    ```

    La première ligne initialise le singleton `DeviceInfo`. La seconde ligne lance l’inscription aux notifications Push, déjà présente si vous avez préalablement suivi le tutoriel [Prise en main de Notification Hubs].
7. Dans PushToUserAppDelegate.m, implémentez la méthode `didRegisterForRemoteNotificationsWithDeviceToken` dans AppDelegate et ajoutez le code suivant :

    ```objc
    self.deviceInfo.deviceToken = deviceToken;
    ```

    Cela définit le jeton d’appareil de la requête.

   > [!NOTE]
   > À ce stade, il ne doit pas y avoir d’autre code dans cette méthode. S’il existe déjà un appel à la méthode `registerNativeWithDeviceToken` que vous avez ajoutée quand vous avez suivi le tutoriel [Envoyer des notifications Push aux applications iOS avec Azure Notification Hubs](ios-sdk-get-started.md), vous devez placer l’appel en commentaire ou le supprimer.

8. Dans le fichier `PushToUserAppDelegate.m`, ajoutez la méthode de gestionnaire suivante :

    ```objc
    * (void) application:(UIApplication *) application didReceiveRemoteNotification:(NSDictionary *)userInfo {
       NSLog(@"%@", userInfo);
       UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Notification" message:
                             [userInfo objectForKey:@"inAppMessage"] delegate:nil cancelButtonTitle:
                             @"OK" otherButtonTitles:nil, nil];
       [alert show];
    }
    ```

    Cette méthode affiche une alerte dans l'interface utilisateur lorsque votre application reçoit des notifications alors qu'elle est en cours d'exécution.

9. Ouvrez le fichier `PushToUserViewController.m` et revenez au clavier dans l’implémentation suivante :

    ```objc
    - (BOOL)textFieldShouldReturn:(UITextField *)theTextField {
        if (theTextField == self.User || theTextField == self.Password) {
            [theTextField resignFirstResponder];
        }
        return YES;
    }
    ```

10. Dans la méthode `viewDidLoad` du fichier `PushToUserViewController.m`, initialisez l’étiquette `installationId` comme suit :

    ```objc
    DeviceInfo* deviceInfo = [(PushToUserAppDelegate*)[[UIApplication sharedApplication]delegate] deviceInfo];
    Self.installationId.text = deviceInfo.installationId;
    ```

11. Ajoutez les propriétés suivantes à l’interface du fichier `PushToUserViewController.m` :

    ```objc
    @property (readonly) NSOperationQueue* downloadQueue;
    - (NSString*)base64forData:(NSData*)theData;
    ```

12. Ajoutez ensuite l'implémentation suivante :

    ```objc
    - (NSOperationQueue *)downloadQueue {
        if (!_downloadQueue) {
            _downloadQueue = [[NSOperationQueue alloc] init];
            _downloadQueue.name = @"Download Queue";
            _downloadQueue.maxConcurrentOperationCount = 1;
        }
        return _downloadQueue;
    }

    // base64 encoding
    - (NSString*)base64forData:(NSData*)theData
    {
        const uint8_t* input = (const uint8_t*)[theData bytes];
        NSInteger length = [theData length];

        static char table[] = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=";

        NSMutableData* data = [NSMutableData dataWithLength:((length + 2) / 3) * 4];
        uint8_t* output = (uint8_t*)data.mutableBytes;

        NSInteger i;
        for (i=0; i < length; i += 3) {
            NSInteger value = 0;
            NSInteger j;
            for (j = i; j < (i + 3); j++) {
                value <<= 8;

                if (j < length) {
                    value |= (0xFF & input[j]);
                }
            }

            NSInteger theIndex = (i / 3) * 4;
            output[theIndex + 0] =                    table[(value >> 18) & 0x3F];
            output[theIndex + 1] =                    table[(value >> 12) & 0x3F];
            output[theIndex + 2] = (i + 1) < length ? table[(value >> 6)  & 0x3F] : '=';
            output[theIndex + 3] = (i + 2) < length ? table[(value >> 0)  & 0x3F] : '=';
        }

        return [[NSString alloc] initWithData:data encoding:NSASCIIStringEncoding];
    }
    ```

13. Copiez le code suivant dans la méthode de gestionnaire `login` créée par XCode :

    ```objc
    DeviceInfo* deviceInfo = [(PushToUserAppDelegate*)[[UIApplication sharedApplication]delegate] deviceInfo];

    // build JSON
    NSString* json = [NSString stringWithFormat:@"{\"platform\":\"ios\", \"instId\":\"%@\", \"deviceToken\":\"%@\"}", deviceInfo.installationId, [deviceInfo getDeviceTokenInHex]];

    // build auth string
    NSString* authString = [NSString stringWithFormat:@"%@:%@", self.User.text, self.Password.text];

    NSMutableURLRequest* request = [NSMutableURLRequest requestWithURL:[NSURL URLWithString:@"http://nhnotifyuser.azurewebsites.net/api/register"]];
    [request setHTTPMethod:@"POST"];
    [request setHTTPBody:[json dataUsingEncoding:NSUTF8StringEncoding]];
    [request addValue:[@([json lengthOfBytesUsingEncoding:NSUTF8StringEncoding]) description] forHTTPHeaderField:@"Content-Length"];
    [request addValue:@"application/json" forHTTPHeaderField:@"Content-Type"];
    [request addValue:[NSString stringWithFormat:@"Basic %@",[self base64forData:[authString dataUsingEncoding:NSUTF8StringEncoding]]] forHTTPHeaderField:@"Authorization"];

    // connect with POST
    [NSURLConnection sendAsynchronousRequest:request queue:[self downloadQueue] completionHandler:^(NSURLResponse* response, NSData* data, NSError* error) {
        // add UIAlert depending on response.
        if (error != nil) {
            NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*)response;
            if ([httpResponse statusCode] == 200) {
                UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Back-end registration" message:@"Registration successful" delegate:nil cancelButtonTitle: @"OK" otherButtonTitles:nil, nil];
                [alert show];
            } else {
                NSLog(@"status: %ld", (long)[httpResponse statusCode]);
            }
        } else {
            NSLog(@"error: %@", error);
        }
    }];
    ```

    Cette méthode obtient à la fois un ID d'installation et un canal pour les notifications Push et les envoie avec le type d'appareil à la méthode d'API Web authentifiée qui crée une inscription dans Notification Hubs. Cette API Web a été définie dans le cadre du didacticiel [Notification des utilisateurs via Notification Hubs].

Maintenant que l'application cliente est à jour, retournez au didacticiel [Notification des utilisateurs via Notification Hubs] et mettez le service mobile à jour pour qu'il envoie des notifications à l'aide de Notification Hubs.

<!-- Anchors. -->

<!-- Images. -->
[0]: ./media/notification-hubs-ios-aspnet-register-user-push-notifications/notification-hub-user-aspnet-ios1.png
[1]: ./media/notification-hubs-ios-aspnet-register-user-push-notifications/notification-hub-user-aspnet-ios2.png

<!-- URLs. -->
[Notification des utilisateurs via Notification Hubs]: notification-hubs-aspnet-backend-ios-apple-apns-notification.md
[Prise en main de Notification Hubs]: ios-sdk-get-started.md

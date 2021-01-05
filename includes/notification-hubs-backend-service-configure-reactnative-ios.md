---
author: alexeystrakh
ms.author: alstrakh
ms.date: 06/11/2020
ms.service: notification-hubs
ms.topic: include
ms.openlocfilehash: e67f53e26f2081e1aa12f7c0e6a219c240f5027a
ms.sourcegitcommit: bcb962e74ee5302d0b9242b1ee006f769a94cfb8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/07/2020
ms.locfileid: "86060444"
---
### <a name="configure-required-ios-packages"></a>Configurer les packages iOS requis

Le package est [automatiquement lié](https://github.com/react-native-community/cli/blob/master/docs/autolinking.md) lors de la génération de l’application. Il vous suffit d’installer les pods natifs :

```bash
npx pod-install
```

### <a name="configure-infoplist-and-entitlementsplist"></a>Configurer Info.plist et Entitlements.plis

1. Accédez à votre dossier « PushDemo/ios », ouvrez l’espace de travail « PushDemo.xcworkspace », sélectionnez le projet supérieur « PushDemo » et sélectionnez l’onglet « Signature et fonctionnalités ».

1. Mettez à jour l’identificateur de bundle pour qu’il corresponde à la valeur utilisée dans le profil de provisionnement.

1. Ajoutez deux nouvelles fonctionnalités à l’aide du bouton « + » :

    - Fonctionnalité en mode arrière-plan et notifications à distance de cycles.
    - Fonctionnalité des notifications Push

### <a name="handle-push-notifications-for-ios"></a>Gérer les notifications Push pour iOS

1. Ouvrez « AppDelegate.h » et ajoutez l’importation suivante :

    ```objective-c
    #import <UserNotifications/UNUserNotificationCenter.h>
    ```

1. Mettez à jour la liste des protocoles pris en charge par « AppDelegate » en ajoutant `UNUserNotificationCenterDelegate` :

    ```objective-c
    @interface AppDelegate : UIResponder <UIApplicationDelegate, RCTBridgeDelegate, UNUserNotificationCenterDelegate>
    ```

1. Ouvrez « AppDelegate.m » et configurez tous les rappels iOS requis :

    ```objective-c
    #import <UserNotifications/UserNotifications.h>
    #import <RNCPushNotificationIOS.h>

    ...

    // Required to register for notifications
    - (void)application:(UIApplication *)application didRegisterUserNotificationSettings:(UIUserNotificationSettings *)notificationSettings
    {
     [RNCPushNotificationIOS didRegisterUserNotificationSettings:notificationSettings];
    }

    // Required for the register event.
    - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
    {
     [RNCPushNotificationIOS didRegisterForRemoteNotificationsWithDeviceToken:deviceToken];
    }

    // Required for the notification event. You must call the completion handler after handling the remote notification.
    - (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo
    fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler
    {
      [RNCPushNotificationIOS didReceiveRemoteNotification:userInfo fetchCompletionHandler:completionHandler];
    }

    // Required for the registrationError event.
    - (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error
    {
     [RNCPushNotificationIOS didFailToRegisterForRemoteNotificationsWithError:error];
    }

    // IOS 10+ Required for localNotification event
    - (void)userNotificationCenter:(UNUserNotificationCenter *)center
    didReceiveNotificationResponse:(UNNotificationResponse *)response
             withCompletionHandler:(void (^)(void))completionHandler
    {
      [RNCPushNotificationIOS didReceiveNotificationResponse:response];
      completionHandler();
    }

    // IOS 4-10 Required for the localNotification event.
    - (void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification
    {
     [RNCPushNotificationIOS didReceiveLocalNotification:notification];
    }

    //Called when a notification is delivered to a foreground app.
    -(void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler
    {
      completionHandler(UNAuthorizationOptionSound | UNAuthorizationOptionAlert | UNAuthorizationOptionBadge);
    }
    ```

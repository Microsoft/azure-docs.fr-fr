---
title: Diagnostiquer les problèmes de perte des notifications dans Azure Notification Hubs
description: Découvrez comment diagnostiquer les problèmes courants liés à la perte des notifications dans Azure Notification Hubs.
services: notification-hubs
documentationcenter: Mobile
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: b5c89a2a-63b8-46d2-bbed-924f5a4cce61
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: NA
ms.devlang: multiple
ms.topic: article
ms.date: 02/25/2020
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 04/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: b5139f75084eb0646db2fc8b05b04aaf3ddb2a12
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89010781"
---
# <a name="diagnose-dropped-notifications-in-azure-notification-hubs"></a>Diagnostiquer les problèmes de perte des notifications dans Azure Notification Hubs

L’une des questions les plus fréquentes concernant Azure Notification Hubs concerne les notifications qui sont envoyées à partir d’une application et qui n’apparaissent pas sur les appareils clients. Les clients veulent savoir où se trouvent ces notifications, pourquoi elles se sont perdues et comment résoudre ce problème. Cet article explique les raisons pour lesquelles les notifications peuvent être perdues ou ne pas apparaître sur les appareils. Il indique également comment déterminer la cause racine du problème.

En premier lieu, il est essentiel de comprendre la manière dont Notification Hubs envoie (push) les notifications vers un appareil.

![Architecture de Notification Hubs][0]

Dans un flux de notification d’envoi type, le message est envoyé par le *serveur d’applications backend* à Notification Hubs. Notification Hubs traite toutes les inscriptions. Il prend en compte les balises configurées et les expressions de balise pour déterminer les cibles. Les cibles correspondent à toutes les inscriptions qui doivent recevoir la notification Push. Ces inscriptions peuvent s’étendre sur toutes les plateformes que nous prenons en charge : Android, Baidu (appareils Android en Chine), Fire OS (Amazon) iOS, Windows et Windows Phone.

Une fois les cibles établies, Notification Hubs envoie (push) des notifications au *service de notifications Push* correspondant à la plateforme de l’appareil. Il peut s’agir, par exemple, d’Apple Push Notification Service (APNs) pour iOS et macOS ou de Firebase Cloud Messaging (FCM) pour les appareils Android. Notification Hubs envoie les notifications regroupées en plusieurs lots d’inscriptions. Il s’authentifie auprès du service de notifications Push correspondant, en fonction des informations d’identification que vous avez définies dans le Portail Azure, sous **Configurer un hub de notification**. Le service de notifications Push transmet alors les notifications aux *appareils clients* correspondants.

Le dernier tronçon de remise des notifications se situe entre le service de notifications Push de la plateforme et l’appareil. La remise des notifications peut échouer lors de l’une des quatre phases du processus de notifications Push (client, serveur d’applications principal, Notification Hubs et service de notifications Push de la plateforme). Pour plus d’informations sur l’architecture de Notification Hubs, consultez [Vue d’ensemble de Notification Hubs].

L’échec de la remise des notifications peut se produire lors de la phase de test ou de préproduction initiale. À ce stade, la perte des notifications peut indiquer un problème de configuration. Si un échec de remise des notifications se produit lors de la phase de production, il se peut qu’une partie ou la totalité des notifications soient perdues. Cela indique un problème plus complexe lié à l’application ou au type de messagerie.

La section ci-après présente les scénarios, des plus courants aux plus rares, dans lesquels les notifications peuvent être perdues.

## <a name="notification-hubs-misconfiguration"></a>Configuration incorrecte de Notification Hubs

Pour envoyer des notifications au service de notifications Push adapté, Notification Hubs doit s’authentifier dans le contexte de votre application. Vous devez créer un compte de développeur avec le service de notification de la plateforme cible (Microsoft, Apple, Google, etc.). Ensuite, vous devez inscrire votre application auprès du système d’exploitation, grâce auquel vous obtenez un jeton ou une clé qui vous permet de travailler avec le PNS (Platform Notification System) cible.

Vous devez ajouter ces informations d’identification de plateforme dans le portail Azure. Si aucune notification n’arrive sur l’appareil, la première chose à faire consiste à vérifier que les informations d’identification configurées dans Notification Hubs sont correctes. Les informations d’identification doivent correspondre à celles de l’application qui a été créée sous un compte de développeur propre à la plateforme.

Pour obtenir des instructions pas à pas pour ce processus, consultez [Bien démarrer avec Azure Notification Hubs].

Voici certaines erreurs de configuration courantes :

### <a name="notification-hub-name-location"></a>Emplacement du nom du hub de notification

Vérifiez que le nom de votre hub de notification (sans fautes de frappe) est le même dans chacun de ces emplacements :

* Là où vous vous êtes inscrit auprès du client
* Là où vous envoyez des notifications à partir du serveur principal
* Là où vous avez configuré les informations d’identification du service de notifications Push

Vérifiez que vous utilisez les chaînes de configuration de signature d’accès partagé correctes sur le client et sur le serveur d’applications principal. En règle générale, vous devez utiliser **DefaultListenSharedAccessSignature** sur le client et **DefaultFullSharedAccessSignature** sur le serveur d’applications principal. Cette opération autorise l’envoi de notifications à Notification Hubs.

### <a name="apn-configuration"></a>Configuration du nom du point d’accès (APN)

Vous devez disposer de deux hubs différents : l’un pour la production, et l’autre pour les tests. Vous devez charger le certificat que vous utilisez dans un environnement de bac à sable dans un hub différent du certificat/hub que vous utiliserez en production. N’essayez pas de charger plusieurs types de certificats dans un même hub. Cela entraînerait l’échec de la remise des notifications.

Si vous chargez par inadvertance plusieurs types de certificats dans un même hub, supprimez ce dernier et créez-en un autre. Si vous ne pouvez pas supprimer le hub pour une raison quelconque, vous devez au moins supprimer toutes les inscriptions effectuées auprès de ce hub.

### <a name="fcm-configuration"></a>Configuration de Firebase Cloud Messaging (FCM)

1. Vérifiez que la *clé serveur* que vous avez obtenue auprès de Firebase correspond à celle que vous avez inscrite dans le Portail Azure.

   ![Clé serveur Firebase][3]

2. Vérifiez que vous avez configuré **l’ID de projet** dans le client. La valeur de **l’ID de projet** se trouve dans le tableau de bord Firebase.

   ![ID de projet Firebase][1]

## <a name="application-issues"></a>Problèmes de l’application

### <a name="tags-and-tag-expressions"></a>Balises et expressions de balise

Si vous utilisez des balises ou des expressions de balise pour segmenter votre public, il est possible qu’aucune cible ne soit trouvée lorsque vous envoyez la notification. Cette erreur repose sur les balises ou expressions de balise que vous spécifiez dans votre appel d’envoi.

Lorsque vous envoyez une notification, examinez vos inscriptions pour vérifier que les balises correspondent. Ensuite, vérifiez uniquement l’accusé de réception des clients qui disposent de ces inscriptions.

Par exemple, supposons que toutes vos inscriptions auprès de Notification Hubs utilisent la balise « Politique ». Si vous envoyez par la suite une notification avec la balise « Sports », la notification ne sera envoyée à aucun appareil. Un cas complexe peut impliquer des expressions de balise, lorsque vous vous êtes inscrit à l’aide de « Balise A » *ou* de « Balise B », alors que vous cibliez « Balise A && Balise B ». La section de conseils d’autodiagnostic plus loin dans cet article vous explique comment vérifier vos inscriptions et leurs balises.

### <a name="template-issues"></a>Problèmes liés aux modèles

Si vous utilisez des modèles, suivez bien les instructions décrites dans la section [Modèles].

### <a name="invalid-registrations"></a>Inscriptions non valides

Si le hub de notification a été configuré de manière adéquate, et que les balises ou expressions de balise ont été correctement utilisées, des cibles valides sont trouvées. Les notifications doivent être envoyées à ces cibles. Notification Hubs déclenche ensuite plusieurs traitements en parallèle. Chaque lot envoie des messages à un ensemble d’inscriptions.

> [!NOTE]
> Étant donné que Notification Hubs traite plusieurs lots en parallèle, l’ordre dans lequel les notifications sont remises n’est pas garanti.

Notification Hubs est optimisé pour le modèle de remise de messages de type « une fois maximum ». Nous tentons une déduplication de sorte qu’aucune notification ne soit remise plus d’une fois au même périphérique. Le processus contrôle les inscriptions pour vérifier qu’un seul message est envoyé par identificateur d’appareil avant son envoi au service de notifications Push.

Chaque lot est envoyé au service de notifications Push qui, à son tour, accepte et valide les inscriptions. Au cours de ce processus, il est possible que le service de notifications Push détecte une erreur au niveau d’une ou de plusieurs inscriptions dans un lot. Le service de notifications Push renvoie alors une erreur à Notification Hubs, et le processus s’arrête. Le service de notifications Push abandonne alors l’intégralité du lot. Cela est particulièrement vrai pour Apple Push Notification Service (APNs), qui utilise un protocole de flux TCP.

Dans ce cas, l’inscription défaillante est supprimée de la base de données. Ensuite, la remise des notifications est retentée pour les appareils restants du lot.

Pour plus d’informations sur les messages d’erreur concernant l’échec d’une tentative de remise pour une inscription, vous pouvez utiliser les API REST de Notification Hubs [Per Message Telemetry: Get Notification message telemetry](/rest/api/notificationhubs/get-notification-message-telemetry) (Télémétrie par message : obtention de la télémétrie des messages de notification) et [PNS feedback](/previous-versions/azure/reference/mt705560(v=azure.100)) (Commentaires du service de notifications Push). Pour obtenir un exemple de code, consultez [l’exemple d’API REST d’envoi](https://github.com/Azure/azure-notificationhubs-dotnet/tree/master/Samples/SendRestExample/).

## <a name="push-notification-service-issues"></a>Problèmes liés au service de notifications Push

Une fois que le service de notifications Push a reçu la notification, il remet la notification à l’appareil. À ce stade, Notification Hubs n’a aucun contrôle sur la remise de la notification à l’appareil.

Les services de notifications de plateforme étant très performants, les notifications arrivent généralement sur les appareils en quelques secondes. Si le service de notifications Push est soumis à une limitation, Notification Hubs applique une stratégie d’interruption exponentielle. Si le service de notifications Push reste inaccessible pendant 30 minutes, l’une des stratégies mises en place consiste à faire expirer et supprimer définitivement ces messages.

Si un service de notifications Push tente de remettre une notification alors que l’appareil est hors connexion, la notification est stockée par le service pendant une période limitée. La notification est envoyée à l’appareil lorsque celui-ci est de nouveau disponible.

Chaque application ne stocke qu’une seule notification récente. Si plusieurs notifications sont envoyées pendant qu’un appareil se trouve hors connexion, chaque nouvelle notification provoque la suppression de la précédente. La conservation de la dernière notification uniquement est appelée *fusion des notifications* dans APNs et *réduction* dans FCM. (FCM utilise une clé de réduction.) Lorsque l’appareil reste hors connexion pendant une longue période, les notifications qui étaient stockées pour l’appareil sont supprimées. Pour plus d’informations, consultez les articles [Présentation d’Apple Push Notification Service] (Présentation d’Apple Push Notification Service) et [À propos des messages Firebase Cloud Messaging] (À propos des messages Firebase Cloud Messaging).

Avec Notification Hubs, vous pouvez transmettre une clé de fusion par le biais d’un en-tête HTTP à l’aide de l’API SendNotification générique. Par exemple, pour le SDK .NET, vous devez utiliser `SendNotificationAsync`. L’API SendNotification accepte également les en-têtes HTTP qui sont transmis tels quels au service de notifications Push correspondant.

## <a name="self-diagnosis-tips"></a>Conseils d’autodiagnostic

Vous trouverez ci-après différentes méthodes permettant de déterminer la cause racine des pertes de notifications dans Notification Hubs.

### <a name="verify-credentials"></a>Vérification des informations d’identification

#### <a name="push-notification-service-developer-portal"></a>Portail des développeurs du service de notifications Push

Vérifiez les informations d’identification dans le portail des développeurs du service de notifications Push correspondant (Apple Push Notification Service, Firebase Cloud Messaging, service de notification Windows, etc.). Pour plus d’informations, consultez [Didacticiel : Envoyer des notifications vers des applications de plateforme Windows universelle avec Azure Notification Hubs](./notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md).

#### <a name="azure-portal"></a>Portail Azure

Pour vérifier et comparer les informations d’identification avec celles que vous avez obtenues par le biais du portail des développeurs du service de notifications Push, dans le Portail Azure, accédez à l’onglet **Stratégies d’accès**.

![Portail Azure - Stratégies d’accès][4]

### <a name="verify-registrations"></a>Vérification des inscriptions

#### <a name="visual-studio"></a>Visual Studio

Dans Visual Studio, vous pouvez vous connecter à Azure par l’intermédiaire de l’Explorateur de serveurs pour afficher et gérer un ensemble de services Azure, y compris Notifications Hubs. Ce raccourci est particulièrement utile pour votre environnement de développement et de test.

![Explorateur de serveurs Visual Studio.][9]

![Explorateur de serveurs](media/notification-hubs-push-notification-fixer/vsserverexplorer2.png)

Vous pouvez visualiser et gérer toutes les inscriptions de votre hub. Les inscriptions peuvent être classées par plateforme, par type (native ou basée sur un modèle), par balise, par identificateur de service de notifications Push, par ID d’inscription et par date d’expiration. Dans cette page, vous pouvez également modifier une inscription. Cette fonctionnalité est particulièrement utile pour la modification des balises.

Dans **l’Explorateur de serveurs**, cliquez avec le bouton droit sur votre hub de notification, puis sélectionnez **Diagnostiquer**. 

![Explorateur de serveurs Visual Studio : menu Diagnostiquer](./media/notification-hubs-push-notification-fixer/diagnose-menu.png)

La page ci-après apparaît :

![Visual Studio : page Diagnostiquer](./media/notification-hubs-push-notification-fixer/diagnose-page.png)

Basculez vers la page **Inscriptions d’appareil** :

![Visual Studio : page Inscriptions d’appareil](./media/notification-hubs-push-notification-fixer/VSRegistrations.png)

Vous pouvez utiliser la page **Test d’envoi** pour envoyer un message de notification de test :

![Visual Studio : Testez les envois.](./media/notification-hubs-push-notification-fixer/test-send-vs.png)

> [!NOTE]
> Utilisez Visual Studio pour modifier les inscriptions uniquement pendant les phases de développement ou de test, et avec un nombre limité d’inscriptions. Si vous avez besoin de modifier vos inscriptions en bloc, utilisez la fonctionnalité d’exportation et d’importation des inscriptions décrite dans l’article [Exportation et modification d’inscriptions en bloc](/previous-versions/azure/azure-services/dn790624(v=azure.100)).

#### <a name="service-bus-explorer"></a>Explorateur Service Bus

De nombreux clients utilisent [Service Bus Explorer](https://github.com/paolosalvatori/ServiceBusExplorer) pour afficher et gérer leurs hubs de notification. Service Bus Explorer est un projet open source. 

### <a name="verify-message-notifications"></a>Vérification des notifications de messages

#### <a name="azure-portal"></a>Portail Azure

Pour envoyer une notification de test à vos clients sans backend de service opérationnel, sous **Support + dépannage**, sélectionnez **Envoi de test**.

![Fonctionnalité Envoi de test dans Azure][7]

#### <a name="visual-studio"></a>Visual Studio

Vous pouvez également envoyer des notifications de test à partir de Visual Studio.

![Fonctionnalité Envoi de test dans Visual Studio][10]

Pour plus d’informations sur l’utilisation de Notification Hubs avec l’Explorateur de serveurs Visual Studio, consultez les articles suivants :

* [How to view device registrations for notification hubs](/previous-versions/windows/apps/dn792122(v=win.10)) (Affichage des inscriptions d’appareils pour les hubs de notification)
* [Présentation approfondie : Visual Studio 2013 Update 2 RC and Azure SDK 2.3]
* [Announcing release of Visual Studio 2013 Update 3 and Azure SDK 2.4]

### <a name="debug-failed-notifications-and-review-notification-outcome"></a>Résoudre les échecs de remise de notification et analyser les résultats des notifications

#### <a name="enabletestsend-property"></a>Propriété EnableTestSend

Lorsque vous envoyez une notification par le biais de Notification Hubs, la notification est d’abord mise en file d’attente. Notification Hubs détermine les cibles qui conviennent, puis envoie la notification au service de notifications Push. Si vous utilisez l’API REST ou l’un des Kits de développement logiciel (SDK) clients, le renvoi de votre appel d’envoi signifie uniquement que le message est placé dans la file d’attente de Notification Hubs. Ce renvoi ne fournit aucun insight sur ce qui s’est passé lorsque Notification Hubs a finalement envoyé la notification au service de notifications Push.

Si votre notification n’arrive pas jusqu’à l’appareil client, une erreur s’est peut-être produite au moment où Notification Hubs a tenté de remettre la notification au service de notifications Push. Par exemple, la taille de la charge utile peut dépasser le seuil maximal autorisé par le service de notifications Push, ou les informations d’identification configurées dans Notification Hubs peuvent ne pas être valides.

Pour comprendre ce qu’il s’est passé lors d’une erreur du service de notifications Push, vous pouvez utiliser la propriété [EnableTestSend]. Cette propriété est activée automatiquement lorsque vous envoyez des messages de test à partir du portail ou du client Visual Studio. Vous pouvez visualiser des informations de débogage détaillées à l’aide de cette propriété, ainsi que par le biais des API. Actuellement, vous pouvez l’utiliser dans le SDK .NET. Il est prévu que cette propriété soit ajoutée à tous les Kits de développement logiciel (SDK) clients.

Pour utiliser la propriété `EnableTestSend` avec l’appel REST, ajoutez un paramètre de chaîne de requête appelé *test* à la fin de votre appel d’envoi. Par exemple :

```text
https://mynamespace.servicebus.windows.net/mynotificationhub/messages?api-version=2013-10&test
```

#### <a name="net-sdk-example"></a>Exemple de Kit de développement logiciel (SDK) .NET

Voici un exemple d’utilisation du SDK .NET pour envoyer une notification (toast) sous forme de fenêtre contextuelle native :

```csharp
NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString(connString, hubName);
var result = await hub.SendWindowsNativeNotificationAsync(toast);
Console.WriteLine(result.State);
```

À la fin de l’exécution, `result.State` indique seulement la valeur `Enqueued`. Les résultats ne fournissent aucun insight sur ce qui est arrivé à votre notification Push.

Ensuite, vous pouvez utiliser la propriété booléenne `EnableTestSend`. Utilisez la propriété `EnableTestSend` quand vous initialisez `NotificationHubClient` pour obtenir des informations détaillées sur les erreurs du service de notifications Push qui se produisent lors de l’envoi de la notification. Le retour de l’appel d’envoi nécessite davantage de temps, car cet appel doit attendre que Notification Hubs ait remis la notification au service de notifications Push.

```csharp
    bool enableTestSend = true;
    NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString(connString, hubName, enableTestSend);

    var outcome = await hub.SendWindowsNativeNotificationAsync(toast);
    Console.WriteLine(outcome.State);

    foreach (RegistrationResult result in outcome.Results)
    {
        Console.WriteLine(result.ApplicationPlatform + "\n" + result.RegistrationId + "\n" + result.Outcome);
    }
```

#### <a name="sample-output"></a>Exemple de sortie

```text
DetailedStateAvailable
windows
7619785862101227384-7840974832647865618-3
The Token obtained from the Token Provider is wrong
```

Ce message indique que les informations d’identification configurées dans Notification Hubs ne sont pas valides, ou qu’il existe un problème avec les inscriptions dans le hub. Supprimez cette inscription et laissez le client la recréer avant d’envoyer le message.

> [!NOTE]
> L’utilisation de la propriété `EnableTestSend` entraîne une limitation importante. Utilisez cette option uniquement dans un environnement de développement et de test, et avec un ensemble limité d’inscriptions. Les notifications de débogage sont envoyées à un maximum de 10 appareils. En outre, le traitement des envois de débogage est limité à 10 envois par minute.

### <a name="review-telemetry"></a>Révision de la télémétrie

#### <a name="azure-portal"></a>Portail Azure

Le portail vous permet d’obtenir un aperçu rapide de toutes les activités sur votre hub de notification.

1. Sous l’onglet **Vue d’ensemble**, vous pouvez voir les inscriptions, les notifications et les erreurs regroupées par plateforme.

   ![Tableau de bord de Notification Hubs][5]

2. Sous l’onglet **Surveiller**, vous pouvez ajouter de nombreuses autres métriques relatives à la plateforme. Vous pouvez ainsi obtenir des informations plus précises sur les erreurs qui sont renvoyées lorsque Notification Hubs tente d’envoyer la notification au service de notifications Push.

   ![Journal d’activités - Portail Azure][6]

3. Commencez par consultez les **messages entrants**, les **opérations d’inscription** et les **notifications réussies**. Ensuite, accédez à l’onglet de la plateforme pour examiner les erreurs relatives au service de notifications Push.

4. Si les paramètres d’authentification du hub de notification sont incorrects, le message **Erreur d’authentification PNS** s’affiche. Ce message indique qu’il est nécessaire de vérifier les informations d’identification du service de notifications Push.

#### <a name="programmatic-access"></a>Accès par programme

Pour plus d’informations sur l’accès par programmation, consultez l’article [Accès par programme](/previous-versions/azure/azure-services/dn458823(v=azure.100)).

> [!NOTE]
> Plusieurs fonctionnalités de télémétrie, comme l’exportation et l’importation des inscriptions, et l’accès à la télémétrie via des API, sont disponibles uniquement avec le niveau de service Standard. Si vous essayez d’utiliser ces fonctionnalités à partir du niveau de service Gratuit ou De base, vous obtiendrez un message d’exception en cas d’utilisation du Kit de développement logiciel (SDK). Si vous utilisez les fonctionnalités directement à partir des API REST, vous obtiendrez une erreur HTTP 403 (Interdit).
>
> Pour utiliser les fonctionnalités de télémétrie, commencez par vérifier dans le Portail Azure que vous utilisez le niveau de service Standard.  

<!-- IMAGES -->
[0]: ./media/notification-hubs-push-notification-fixer/Architecture.png
[1]: ./media/notification-hubs-push-notification-fixer/FCMConfigure.png
[3]: ./media/notification-hubs-push-notification-fixer/FCMServerKey.png
[4]: ../../includes/media/notification-hubs-portal-create-new-hub/notification-hubs-connection-strings-portal.png
[5]: ./media/notification-hubs-push-notification-fixer/PortalDashboard.png
[6]: ./media/notification-hubs-push-notification-fixer/PortalAnalytics.png
[7]: ./media/notification-hubs-ios-get-started/notification-hubs-test-send.png
[8]: ./media/notification-hubs-push-notification-fixer/VSRegistrations.png
[9]: ./media/notification-hubs-push-notification-fixer/vsserverexplorer.png
[10]: ./media/notification-hubs-push-notification-fixer/VSTestNotification.png

<!-- LINKS -->
[Vue d’ensemble de Notification Hubs]: notification-hubs-push-notification-overview.md
[Bien démarrer avec Azure Notification Hubs]: notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md
[Modèles]: /previous-versions/azure/azure-services/dn530748(v=azure.100)
[Présentation d’Apple Push Notification Service]: https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html
[À propos des messages Firebase Cloud Messaging]: https://firebase.google.com/docs/cloud-messaging/concept-options
[Export and modify registrations in bulk]: /previous-versions/azure/azure-services/dn790624(v=azure.100)
[Service Bus Explorer code]: https://code.msdn.microsoft.com/windowsazure/Service-Bus-Explorer-f2abca5a
[View device registrations for notification hubs]: /previous-versions/windows/apps/dn792122(v=win.10)
[Présentation approfondie : Visual Studio 2013 Update 2 RC and Azure SDK 2.3]: https://azure.microsoft.com/blog/2014/04/09/deep-dive-visual-studio-2013-update-2-rc-and-azure-sdk-2-3/#NotificationHubs
[Announcing release of Visual Studio 2013 Update 3 and Azure SDK 2.4]: https://azure.microsoft.com/blog/2014/08/04/announcing-release-of-visual-studio-2013-update-3-and-azure-sdk-2-4/
[EnableTestSend]: /dotnet/api/microsoft.azure.notificationhubs.notificationhubclient.enabletestsend?view=azure-dotnet
[Programmatic telemetry access]: /previous-versions/azure/azure-services/dn458823(v=azure.100)

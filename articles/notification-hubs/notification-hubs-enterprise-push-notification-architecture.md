---
title: Notification Hubs architecture Push d’entreprise
description: En savoir plus sur l’utilisation d’Azur Notification Hubs dans un environnement d’entreprise
services: notification-hubs
documentationcenter: ''
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: 903023e9-9347-442a-924b-663af85e05c6
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows
ms.devlang: dotnet
ms.topic: article
ms.date: 01/04/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 01/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 018315b7ed468e24fb922337848d14703ffdcd4d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89003624"
---
# <a name="enterprise-push-architectural-guidance"></a>Guide architectural des notifications Push d’entreprise

Les entreprises se tournent aujourd’hui progressivement vers la création d’applications mobiles pour leurs clients (externes) ou leurs collaborateurs (internes). Ils disposent de systèmes principaux, qu’il s’agisse de grands systèmes ou de certaines applications métiers, qui doivent être intégrés à l’architecture de l’application mobile. Ce guide vous présente comment réussir au mieux cette intégration, en recommandant des solutions possibles pour les scénarios habituels.

Une exigence fréquente est l’envoi de notifications Push aux utilisateurs via leurs applications mobiles, lorsqu’un événement se produit dans les systèmes principaux. Par exemple, un client d’une banque disposant d’une application bancaire sur un iPhone souhaite être averti lorsque son compte est débité d’un montant supérieur à un seuil défini ou un scénario intranet dans lequel un employé du service financier disposant d’une application d’approbation du budget sur un téléphone Windows Phone souhaite être averti lorsqu’il reçoit une demande d’approbation.

Le traitement des comptes bancaires ou des approbations est généralement effectué dans un système principal, qui doit envoyer une notification Push à l’utilisateur. Il peut y avoir plusieurs systèmes principaux qui doivent tous générer le même genre de logique afin d’envoyer une notification Push lorsqu’un événement la déclenche. Ici, la complexité réside dans l’intégration de plusieurs systèmes principaux avec un système Push unique où les utilisateurs peuvent être abonnés à différentes notifications et même peut-être à plusieurs applications mobiles, par exemple dans le cas d’applications mobiles intranet où une application mobile peut recevoir des notifications à partir de plusieurs systèmes principaux. Les systèmes principaux n’ont pas besoin de connaître une technologie ou une sémantique Push. Une solution courante a donc généralement été d’introduire un composant qui interroge les systèmes principaux pour tout événement vous intéressant et qui est chargé d’envoyer les messages Push au client.

Une solution encore plus efficace consiste à utiliser le modèle Azure Service Bus - Rubrique/abonnement, qui réduit la complexité tout en rendant la solution évolutive.

Voici l’architecture générale de la solution (généralisée pour plusieurs applications mobiles mais également applicable lorsqu’il n’y a qu’une seule application mobile)

## <a name="architecture"></a>Architecture

![Diagramme de l’architecture d’entreprise montrant le flux dans les événements, les abonnements et les messages de transmission de type push.][1]

L'élément clé de ce diagramme architectural est Azure Service Bus, qui fournit un modèle de programmation des rubriques/abonnements (la page [Programmation Service Bus Pub/Sub]apporte plus d'informations à ce sujet). Le récepteur, dans ce cas le serveur principal Mobile (généralement [Azure Mobile Service], qui initie une transmission de type push vers les applications mobiles), ne reçoit pas directement les messages des systèmes principaux mais d'une couche d'abstraction intermédiaire fournie par [Azure Service Bus] qui permet au serveur principal Mobile de recevoir des messages d'un ou plusieurs systèmes principaux. Une rubrique Service Bus doit être créée pour chacun des systèmes principaux, par exemple Comptabilité, RH, Finance, qui sont essentiellement des « rubriques » d’intérêt qui génèrent des messages à envoyer en tant que notification Push. Les systèmes principaux envoient les messages à ces rubriques. Un service principal Mobile peut s’abonner à une ou plusieurs de ces rubriques en créant un abonnement Service Bus. Cela autorise le serveur principal mobile à recevoir une notification du système principal correspondant. Le serveur principal mobile continue à écouter les messages sur ses abonnements et dès qu’un message arrive, il le reprend et l’envoie sous forme de notification à son hub de notification. Pour finir, les hubs de notification remettent ensuite le message à l’application mobile. Voici la liste des composants clés :

1. les systèmes principaux (systèmes métiers/hérités)
   * Crée une rubrique Service Bus
   * Envoie un message
1. Serveur principal mobile
   * Crée l’abonnement au service
   * Reçoit le message (du système principal)
   * Envoie une notification aux clients (via Azure Notification Hub)
1. Application mobile
   * Reçoit et affiche une notification

### <a name="benefits"></a>Avantages

1. Le découplage entre l’expéditeur (les systèmes principaux) et le récepteur (application/service mobile via Notification Hub) permet à des systèmes principaux supplémentaires d’être intégrés, et ce avec un minimum de modifications.
1. Cela permet également à plusieurs applications mobiles de recevoir des événements à partir d’un ou plusieurs systèmes principaux.  

## <a name="sample"></a>Exemple

### <a name="prerequisites"></a>Prérequis

Complétez les tutoriels suivants pour vous familiariser avec les concepts et les étapes habituelles de création et de configuration :

1. [Programmation Service Bus Pub/Sub] : ce tutoriel explique en détail comment utiliser des rubriques/abonnements Service Bus, comment créer un espace de noms pour contenir des rubriques/abonnements et comment envoyer et recevoir des messages à partir de ces rubriques/abonnements.
2. [Notification Hubs : didacticiel Windows Universal] : ce tutoriel explique comment configurer une application Windows Store et Notification Hubs pour vous inscrire et recevoir des notifications.

### <a name="sample-code"></a>Exemple de code

L'exemple de code complet est disponible dans la page [Exemples de Notification Hub]. Il est divisé en trois composants :

1. **EnterprisePushBackendSystem**

    a. Ce projet utilise le package NuGet **WindowsAzure.ServiceBus**. Il repose sur la [programmation Service Bus Pub/Sub].

    b. Il s’agit d’une console d’application C# simple pour simuler un système métier qui lance le message à remettre à l’application mobile.

    ```csharp
    static void Main(string[] args)
    {
        string connectionString =
            CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

        // Create the topic
        CreateTopic(connectionString);

        // Send message
        SendMessage(connectionString);
    }
    ```

    c. `CreateTopic` est utilisé pour créer la rubrique Service Bus.

    ```csharp
    public static void CreateTopic(string connectionString)
    {
        // Create the topic if it does not exist already

        var namespaceManager =
            NamespaceManager.CreateFromConnectionString(connectionString);

        if (!namespaceManager.TopicExists(sampleTopic))
        {
            namespaceManager.CreateTopic(sampleTopic);
        }
    }
    ```

    d. `SendMessage` est utilisé pour envoyer les messages à cette rubrique Service Bus. Ce code envoie simplement un ensemble de messages aléatoires à la rubrique. Normalement, un système principal envoie des messages lorsqu’un événement se produit.

    ```csharp
    public static void SendMessage(string connectionString)
    {
        TopicClient client =
            TopicClient.CreateFromConnectionString(connectionString, sampleTopic);

        // Sends random messages every 10 seconds to the topic
        string[] messages =
        {
            "Employee Id '{0}' has joined.",
            "Employee Id '{0}' has left.",
            "Employee Id '{0}' has switched to a different team."
        };

        while (true)
        {
            Random rnd = new Random();
            string employeeId = rnd.Next(10000, 99999).ToString();
            string notification = String.Format(messages[rnd.Next(0,messages.Length)], employeeId);

            // Send Notification
            BrokeredMessage message = new BrokeredMessage(notification);
            client.Send(message);

            Console.WriteLine("{0} Message sent - '{1}'", DateTime.Now, notification);

            System.Threading.Thread.Sleep(new TimeSpan(0, 0, 10));
        }
    }
    ```
2. **ReceiveAndSendNotification**

    a. Ce projet utilise les packages NuGet *WindowsAzure.ServiceBus* et **Microsoft.Web.WebJobs.Publish**. Il repose sur la [programmation Service Bus Pub/Sub].

    b. L’application de console suivante s'exécute comme une [tâche web Azure], étant donné qu'elle doit s'exécuter en continu pour écouter les messages des systèmes métiers/principaux. Cette application fait partie de votre serveur principal Mobile.

    ```csharp
    static void Main(string[] args)
    {
        string connectionString =
                 CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

        // Create the subscription that receives messages
        CreateSubscription(connectionString);

        // Receive message
        ReceiveMessageAndSendNotification(connectionString);
    }
    ```

    c. `CreateSubscription` permet de créer un abonnement Service Bus pour la rubrique où le système principal envoie les messages. Selon le scénario commercial, ce composant crée un ou plusieurs abonnements aux rubriques correspondantes (par exemple, certains peuvent recevoir les messages du système de RH, du système des Finances, etc.)

    ```csharp
    static void CreateSubscription(string connectionString)
    {
        // Create the subscription if it does not exist already
        var namespaceManager =
            NamespaceManager.CreateFromConnectionString(connectionString);

        if (!namespaceManager.SubscriptionExists(sampleTopic, sampleSubscription))
        {
            namespaceManager.CreateSubscription(sampleTopic, sampleSubscription);
        }
    }
    ```

    d. `ReceiveMessageAndSendNotification` est utilisé pour lire le message à partir de la rubrique en utilisant son abonnement et, si la lecture aboutit, une notification est créée (dans notre exemple, il s'agit d'une notification toast native Windows) pour être envoyée à l'application mobile à l'aide d'Azure Notification Hubs.

    ```csharp
    static void ReceiveMessageAndSendNotification(string connectionString)
    {
        // Initialize the Notification Hub
        string hubConnectionString = CloudConfigurationManager.GetSetting
                ("Microsoft.NotificationHub.ConnectionString");
        hub = NotificationHubClient.CreateClientFromConnectionString
                (hubConnectionString, "enterprisepushservicehub");

        SubscriptionClient Client =
            SubscriptionClient.CreateFromConnectionString
                    (connectionString, sampleTopic, sampleSubscription);

        Client.Receive();

        // Continuously process messages received from the subscription
        while (true)
        {
            BrokeredMessage message = Client.Receive();
            var toastMessage = @"<toast><visual><binding template=""ToastText01""><text id=""1"">{messagepayload}</text></binding></visual></toast>";

            if (message != null)
            {
                try
                {
                    Console.WriteLine(message.MessageId);
                    Console.WriteLine(message.SequenceNumber);
                    string messageBody = message.GetBody<string>();
                    Console.WriteLine("Body: " + messageBody + "\n");

                    toastMessage = toastMessage.Replace("{messagepayload}", messageBody);
                    SendNotificationAsync(toastMessage);

                    // Remove message from subscription
                    message.Complete();
                }
                catch (Exception)
                {
                    // Indicate a problem, unlock message in subscription
                    message.Abandon();
                }
            }
        }
    }
    static async void SendNotificationAsync(string message)
    {
        await hub.SendWindowsNativeNotificationAsync(message);
    }
    ```

    e. Pour publier cette application sous forme de **travail web**, cliquez avec le bouton droit sur la solution dans Visual Studio et sélectionnez **Publier en tant que travail web**

    ![Capture d’écran des options de clic droit affichées avec la fonction Publier sous forme de tâche web Azure en rouge.][2]

    f. Sélectionnez votre profil de publication et créez un site web Azure s’il n’existe pas déjà : il héberge ce travail web. Quand vous disposez du site web, utilisez **Publier**.

    :::image type="complex" source="./media/notification-hubs-enterprise-push-architecture/PublishAsWebJob.png" alt-text="Capture d’écran montrant le flux de travail pour créer un site sur Azure.":::
    Capture d’écran de la boîte de dialogue Publier le site Web avec l’option sites Web Microsoft Azure sélectionnée, une flèche verte pointant vers la boîte de dialogue Sélectionner un site Web existant avec la nouvelle option en rouge, et une flèche verte pointant vers la boîte de dialogue Créer un site sur Microsoft Azure avec le nom du site et les options de création présentées en rouge.
    :::image-end:::

    g. Configurez le travail sur « Exécuter en continu ». Ainsi, quand vous vous connectez au [Azure portal], vous devriez voir ce qui suit :

    ![Capture d’écran du portail Azure avec les tâches web principales de transmission de type push d’entreprise affichées et les valeurs Name, Planification et Journaux d'activité en rouge.][4]

3. **EnterprisePushMobileApp**

    a. Cette application est une application Windows Store qui reçoit des notifications toast du WebJob en cours d’exécution dans le cadre de votre serveur principal Mobile et les affiche. Ce code repose sur [Notification Hubs : didacticiel Windows Universal].  

    b. Assurez-vous que la réception des notifications toast est activée dans votre application.

    c. Assurez-vous que le code d'inscription Notification Hubs suivant est appelé au démarrage de l'application (après avoir remplacé les valeurs `HubName` et `DefaultListenSharedAccessSignature` :

    ```csharp
    private async void InitNotificationsAsync()
    {
        var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

        var hub = new NotificationHub("[HubName]", "[DefaultListenSharedAccessSignature]");
        var result = await hub.RegisterNativeAsync(channel.Uri);

        // Displays the registration ID so you know it was successful
        if (result.RegistrationId != null)
        {
            var dialog = new MessageDialog("Registration successful: " + result.RegistrationId);
            dialog.Commands.Add(new UICommand("OK"));
            await dialog.ShowAsync();
        }
    }
    ```

### <a name="running-the-sample"></a>Exécution de l’exemple

1. Assurez-vous que votre WebJob est exécuté avec succès et planifié pour s’exécuter en continu.
2. Exécutez **EnterprisePushMobileApp**, qui démarre l’application Windows Store.
3. Exécutez l’application console **EnterprisePushBackendSystem** qui simule le serveur principal métier et envoie des messages. Vous devez voir apparaître des notifications toast similaires à l’image suivante :

    ![Capture d’écran d’une console exécutant l’application système principal push d’entreprise et le message envoyé par l’application.][5]

4. Les messages ont été envoyés aux rubriques Service Bus qui ont été analysées par les abonnements Service Bus dans votre tâche Web. Lors de la réception d’un message, une notification a été créée et envoyée à l’application mobile. Vous pouvez consulter les journaux d’activité WebJob pour confirmer le traitement quand vous accédez au lien Journaux d’activité dans le [Azure portal] pour votre tâche Web :

    ![Capture d’écran de la boîte de dialogue Détails de tâche web continue avec le message envoyé en rouge.][6]

<!-- Images -->
[1]: ./media/notification-hubs-enterprise-push-architecture/architecture.png
[2]: ./media/notification-hubs-enterprise-push-architecture/WebJobsContextMenu.png
[3]: ./media/notification-hubs-enterprise-push-architecture/PublishAsWebJob.png
[4]: ./media/notification-hubs-enterprise-push-architecture/WebJob.png
[5]: ./media/notification-hubs-enterprise-push-architecture/Notifications.png
[6]: ./media/notification-hubs-enterprise-push-architecture/WebJobsLog.png

<!-- Links -->
[Exemples de Notification Hub]: https://github.com/Azure/azure-notificationhubs-samples
[Azure Mobile Service]: https://azure.microsoft.com/documentation/services/mobile-services/
[Azure Service Bus]: ../service-bus-messaging/service-bus-messaging-overview.md
[Programmation Service Bus Pub/Sub]: ../service-bus-messaging/service-bus-dotnet-how-to-use-topics-subscriptions.md
[tâche web Azure]: ../app-service/webjobs-create.md
[Notification Hubs : didacticiel Windows Universal]: ./notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md
[Azure portal]: https://portal.azure.com/

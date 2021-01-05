---
title: Mettre à jour l’inventaire avec le portail Azure et des rubriques/abonnements
description: Dans ce tutoriel, vous découvrez comment envoyer et recevoir des messages à partir d’une rubrique et d’un abonnement et comment ajouter et utiliser les règles de filtre à l’aide de .NET
author: spelluru
ms.author: spelluru
ms.date: 10/15/2020
ms.topic: tutorial
ms.custom: devx-track-csharp
ms.openlocfilehash: f2c8d107c6de4965472c3fb04ff626841fb1f6ea
ms.sourcegitcommit: 6a770fc07237f02bea8cc463f3d8cc5c246d7c65
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95810737"
---
# <a name="tutorial-update-inventory-using-azure-portal-and-topicssubscriptions"></a>Tutoriel : mise à jour de l’inventaire à l’aide du portail Azure et des rubriques/abonnements

Microsoft Azure Service Bus est un service de messagerie cloud multi-locataire qui envoie des informations entre les applications et les services. Le fonctionnement asynchrone vous offre un service de messagerie répartie flexible, avec messagerie premier entré, premier sorti (FIFO) et fonctionnalités de publication/abonnement. Ce tutoriel montre comment utiliser les rubriques et abonnements Service Bus dans un scénario d’inventaire de vente au détail, avec des canaux de publication/abonnement à l’aide du portail Azure et de .NET.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> * Créer une rubrique Service Bus et un ou plusieurs abonnements à cette rubrique à l’aide du portail Azure
> * Ajouter des filtres de rubrique à l’aide de code .NET
> * Créer deux messages avec un contenu différent
> * Envoyer les messages et vérifiez qu’ils arrivent dans les abonnements attendus
> * Recevoir des messages des abonnements

Un exemple de ce scénario est une mise à jour d’assortiment d’inventaire pour plusieurs magasins de détail. Dans ce scénario, chaque magasin ou ensemble de magasins, reçoit les messages destinés à mettre à jour leurs assortiments. Ce tutoriel montre comment implémenter ce scénario à l’aide des filtres et des abonnements. Tout d’abord, vous créez une rubrique avec 3 abonnements, ajoutez des règles et des filtres, puis vous envoyez et recevez des messages à partir de la rubrique et des abonnements.

![topic](./media/service-bus-tutorial-topics-subscriptions-portal/about-service-bus-topic.png)

Si vous n'avez pas d'abonnement Azure, vous pouvez créer un [compte gratuit][] avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour suivre ce didacticiel, vérifiez que les éléments suivants sont installés :

- [Visual Studio 2017 Update 3 (version 15.3, 26730.01)](https://www.visualstudio.com/vs) ou ultérieur.
- [Kit de développement logiciel (SDK) NET Core](https://www.microsoft.com/net/download/windows), version 2.0 ou ultérieure.

## <a name="service-bus-topics-and-subscriptions"></a>Rubriques et abonnements Service Bus

Chaque [abonnement à une rubrique](service-bus-messaging-overview.md#topics) peut recevoir une copie de chaque message. Les rubriques sont entièrement compatibles au niveau de la sémantique et du protocole avec les files d’attente Service Bus. Les rubriques Service Bus prennent en charge un large éventail de règles de sélection avec des conditions de filtre et avec des actions facultatives qui permettent de définir ou de modifier les propriétés des messages. Chaque fois qu’une règle correspond, elle génère un message. Pour en savoir plus sur les règles, filtres et actions, consultez ce [lien](topic-filters.md).

[!INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

[!INCLUDE [service-bus-create-topics-three-subscriptions-portal](../../includes/service-bus-create-topics-three-subscriptions-portal.md)]



## <a name="create-filter-rules-on-subscriptions"></a>Créer des règles de filtre sur les abonnements

Une fois que l’espace de noms et les rubriques/abonnements sont configurés, et que vous disposez des informations d’identification nécessaires, vous êtes prêt à créer des règles de filtre sur les abonnements, puis à envoyer et recevoir des messages. Vous pouvez consulter le code dans [ce dossier d’exemples GitHub](https://github.com/Azure/azure-service-bus/tree/master/samples/Java/azure-servicebus/TopicFilters).

## <a name="send-and-receive-messages"></a>Envoyer et recevoir des messages

Pour exécuter le code, procédez comme suit :

1. Dans une invite de commandes ou une invite PowerShell, clonez le [référentiel GitHub Service Bus](https://github.com/Azure/azure-service-bus/) en exécutant la commande suivante :

   ```shell
   git clone https://github.com/Azure/azure-service-bus.git
   ```

2. Accédez à l’exemple de dossier `azure-service-bus\samples\DotNet\Azure.Messaging.ServiceBus\BasicSendReceiveTutorialwithFilters`.

3. Récupérez la chaîne de connexion que vous avez copiée dans le Bloc-notes dans la section Obtenir les informations d’identification de gestion de ce tutoriel. Vous avez également besoin du nom de la rubrique que vous avez créé dans la section précédente.

4. Saisissez ensuite la commande suivante dans l’invite de commandes :

   ```shell
   dotnet build
   ```

5. Accédez au dossier `BasicSendReceiveTutorialwithFilters\bin\Debug\netcoreapp3.1`.

6. Tapez la commande suivante pour exécuter le programme. Veillez à remplacer `myConnectionString` par la valeur que vous avez obtenue précédemment, et `myTopicName` par le nom de la rubrique que vous avez créée :

   ```shell
   dotnet BasicSendReceiveTutorialwithFilters.dll -ConnectionString "myConnectionString" -TopicName "myTopicName"
   ``` 
7. Suivez d’abord les instructions dans la console pour sélectionner la création de filtres. La création de filtres implique la suppression des filtres par défaut. Lorsque vous utilisez PowerShell ou CLI, vous n’avez pas besoin de supprimer les filtres par défaut, mais si faites cela dans le code, vous devez les supprimer. Les commandes de console 1 et 3 vous aident à gérer les filtres sur les abonnements que vous avez créés précédemment :

   - Exécutez 1 : pour supprimer les filtres par défaut.
   - Exécutez 2 : pour ajouter vos propres filtres.
   - Exécutez 3 : pour supprimer vos propres filtres (facultatif). Notez que cela ne recréera pas les filtres par défaut.

     ![Affichage de la sortie de 2](./media/service-bus-tutorial-topics-subscriptions-portal/create-rules.png)

8. Après la création des filtres, vous pouvez envoyer des messages. Appuyez sur 4 et observez l’envoi de 10 messages à la rubrique :

    ![Envoi de la sortie](./media/service-bus-tutorial-topics-subscriptions-portal/send-output.png)

9. Appuyez sur 5 et observez la réception des messages. Si vous n’obtenez pas 10 messages en retour, appuyez sur « m » pour afficher le menu, puis appuyez de nouveau sur 5.

    ![Recevoir la sortie](./media/service-bus-tutorial-topics-subscriptions-portal/receive-output.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsqu’il n’est plus nécessaire, supprimez l’espace de noms et la file d’attente. Pour ce faire, sélectionnez ces ressources sur le portail et cliquez sur **Supprimer**.

## <a name="understand-the-sample-code"></a>Comprendre l’exemple de code

Cette section contient plus de détails sur ce que fait l’exemple de code.

### <a name="get-connection-string-and-topic"></a>Obtention de la chaîne de connexion et de la rubrique

Tout d’abord, le code déclare un ensemble de variables qui déterminent l’exécution restante du programme.

```csharp
string ServiceBusConnectionString;
string TopicName;

static string[] Subscriptions = { "S1", "S2", "S3" };
static IDictionary<string, string[]> SubscriptionFilters = new Dictionary<string, string[]> {
    { "S1", new[] { "StoreId IN('Store1', 'Store2', 'Store3')", "StoreId = 'Store4'"} },
    { "S2", new[] { "sys.To IN ('Store5','Store6','Store7') OR StoreId = 'Store8'" } },
    { "S3", new[] { "sys.To NOT IN ('Store1','Store2','Store3','Store4','Store5','Store6','Store7','Store8') OR StoreId NOT IN ('Store1','Store2','Store3','Store4','Store5','Store6','Store7','Store8')" } }
};
// You can have only have one action per rule and this sample code supports only one action for the first filter, which is used to create the first rule. 
static IDictionary<string, string> SubscriptionAction = new Dictionary<string, string> {
    { "S1", "" },
    { "S2", "" },
    { "S3", "SET sys.Label = 'SalesEvent'"  }
};
static string[] Store = { "Store1", "Store2", "Store3", "Store4", "Store5", "Store6", "Store7", "Store8", "Store9", "Store10" };
static string SysField = "sys.To";
static string CustomField = "StoreId";
static int NrOfMessagesPerStore = 1; // Send at least 1.
```

Les noms de la rubrique et de la chaîne de connexion sont transmis via des paramètres de ligne de commande comme indiqué, puis sont lus dans la méthode `Main()` :

```csharp
static void Main(string[] args)
{
    string ServiceBusConnectionString = "";
    string TopicName = "";

    for (int i = 0; i < args.Length; i++)
    {
        if (args[i] == "-ConnectionString")
        {
            Console.WriteLine($"ConnectionString: {args[i + 1]}");
            ServiceBusConnectionString = args[i + 1]; // Alternatively enter your connection string here.
        }
        else if (args[i] == "-TopicName")
        {
            Console.WriteLine($"TopicName: {args[i + 1]}");
            TopicName = args[i + 1]; // Alternatively enter your queue name here.
        }
    }

    if (ServiceBusConnectionString != "" && TopicName != "")
    {
        Program P = StartProgram(ServiceBusConnectionString, TopicName);
        P.PresentMenu().GetAwaiter().GetResult();
    }
    else
    {
        Console.WriteLine("Specify -Connectionstring and -TopicName to execute the example.");
        Console.ReadKey();
    }
}
```

### <a name="remove-default-filters"></a>Supprimer les filtres par défaut

Lorsque vous créez un abonnement, Service Bus crée un filtre par défaut par abonnement. Ce filtre permet de recevoir tous les messages envoyés à la rubrique. Si vous souhaitez utiliser des filtres personnalisés, vous pouvez supprimer le filtre par défaut, comme indiqué dans le code suivant :

```csharp
private async Task RemoveDefaultFilters()
{
    Console.WriteLine($"Starting to remove default filters.");

    try
    {
        var client = new ServiceBusAdministrationClient(ServiceBusConnectionString);
        foreach (var subscription in Subscriptions)
        {
            await client.DeleteRuleAsync(TopicName, subscription, CreateRuleOptions.DefaultRuleName);
            Console.WriteLine($"Default filter for {subscription} has been removed.");
        }

        Console.WriteLine("All default Rules have been removed.\n");
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
    }

    await PresentMenu();
}
```

### <a name="create-filters"></a>Création de filtres

Le code suivant ajoute les filtres personnalisés définis dans ce tutoriel :

```csharp
private async Task CreateCustomFilters()
{
    try
    {
        for (int i = 0; i < Subscriptions.Length; i++)
        {
            var client = new ServiceBusAdministrationClient(ServiceBusConnectionString);
            string[] filters = SubscriptionFilters[Subscriptions[i]];
            if (filters[0] != "")
            {
                int count = 0;
                foreach (var myFilter in filters)
                {
                    count++;

                    string action = SubscriptionAction[Subscriptions[i]];
                    if (action != "")
                    {
                        await client.CreateRuleAsync(TopicName, Subscriptions[i], new CreateRuleOptions
                        {
                            Filter = new SqlRuleFilter(myFilter),
                            Action = new SqlRuleAction(action),
                            Name = $"MyRule{count}"
                        });
                    }
                    else
                    {
                        await client.CreateRuleAsync(TopicName, Subscriptions[i], new CreateRuleOptions
                        {
                            Filter = new SqlRuleFilter(myFilter),
                            Name = $"MyRule{count}"
                        });
                    }
                }
            }

            Console.WriteLine($"Filters and actions for {Subscriptions[i]} have been created.");
        }

        Console.WriteLine("All filters and actions have been created.\n");
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
    }

    await PresentMenu();
}
```

### <a name="remove-your-custom-created-filters"></a>Supprimer vos filtres personnalisés

Si vous souhaitez supprimer tous les filtres de votre abonnement, le code suivant montre comment procéder :

```csharp
private async Task CleanUpCustomFilters()
{
    foreach (var subscription in Subscriptions)
    {
        try
        {
            var client = new ServiceBusAdministrationClient(ServiceBusConnectionString);
            IAsyncEnumerator<RuleProperties> rules = client.GetRulesAsync(TopicName, subscription).GetAsyncEnumerator();
            while (await rules.MoveNextAsync())
            {
                await client.DeleteRuleAsync(TopicName, subscription, rules.Current.Name);
                Console.WriteLine($"Rule {rules.Current.Name} has been removed.");
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.ToString());
        }
    }
    Console.WriteLine("All default filters have been removed.\n");

    await PresentMenu();
}
```

### <a name="send-messages"></a>Envoyer des messages

L’envoi de messages à une rubrique est similaire à l’envoi de messages à une file d’attente. Cet exemple montre comment envoyer des messages, à l’aide d’une liste des tâches et d’un traitement asynchrone :

```csharp
public async Task SendMessages()
{
    try
    {
        await using var client = new ServiceBusClient(ServiceBusConnectionString);
        var taskList = new List<Task>();
        for (int i = 0; i < Store.Length; i++)
        {
            taskList.Add(SendItems(client, Store[i]));
        }

        await Task.WhenAll(taskList);
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
    }
    Console.WriteLine("\nAll messages sent.\n");
}

private async Task SendItems(ServiceBusClient client, string store)
{
    // create the sender
    ServiceBusSender tc = client.CreateSender(TopicName);

    for (int i = 0; i < NrOfMessagesPerStore; i++)
    {
        Random r = new Random();
        Item item = new Item(r.Next(5), r.Next(5), r.Next(5));

        // Note the extension class which is serializing an deserializing messages
        ServiceBusMessage message = item.AsMessage();
        message.To = store;
        message.ApplicationProperties.Add("StoreId", store);
        message.ApplicationProperties.Add("Price", item.GetPrice().ToString());
        message.ApplicationProperties.Add("Color", item.GetColor());
        message.ApplicationProperties.Add("Category", item.GetItemCategory());

        await tc.SendMessageAsync(message);
        Console.WriteLine($"Sent item to Store {store}. Price={item.GetPrice()}, Color={item.GetColor()}, Category={item.GetItemCategory()}"); ;
    }
}
```

### <a name="receive-messages"></a>Recevoir des messages

Là encore, les messages sont reçus via une liste des tâches, et le code utilise le traitement par lot. Vous pouvez envoyer et recevoir à l’aide du traitement par lot, mais cet exemple ne montre que la réception par lots. En réalité, vous ne quitteriez pas la boucle, mais continueriez le bouclage et définiriez une valeur timespan supérieure, par exemple une minute. L’appel de réception au répartiteur est maintenu ouvert pendant ce laps de temps et si des messages arrivent, ils sont retournés immédiatement et un nouvel appel de réception est émis. Ce concept est appelé *interrogation longue*. Utiliser la pompe de réception que vous pouvez visualiser dans le [démarrage rapide](service-bus-quickstart-portal.md), et dans plusieurs autres exemples dans le référentiel, est une option plus classique.

```csharp
public async Task Receive()
{
    var taskList = new List<Task>();
    for (var i = 0; i < Subscriptions.Length; i++)
    {
        taskList.Add(this.ReceiveMessages(Subscriptions[i]));
    }

    await Task.WhenAll(taskList);
}

private async Task ReceiveMessages(string subscription)
{
    await using var client = new ServiceBusClient(ServiceBusConnectionString);
    ServiceBusReceiver receiver = client.CreateReceiver(TopicName, subscription);

    // In reality you would not break out of the loop like in this example but would keep looping. The receiver keeps the connection open
    // to the broker for the specified amount of seconds and the broker returns messages as soon as they arrive. The client then initiates
    // a new connection. So in reality you would not want to break out of the loop. 
    // Also note that the code shows how to batch receive, which you would do for performance reasons. For convenience you can also always
    // use the regular receive pump which we show in our Quick Start and in other github samples.
    while (true)
    {
        try
        {
            //IList<Message> messages = await receiver.ReceiveAsync(10, TimeSpan.FromSeconds(2));
            // Note the extension class which is serializing an deserializing messages and testing messages is null or 0.
            // If you think you did not receive all messages, just press M and receive again via the menu.
            IReadOnlyList<ServiceBusReceivedMessage> messages = await receiver.ReceiveMessagesAsync(maxMessages: 100);

            if (messages.Any())
            {
                foreach (ServiceBusReceivedMessage message in messages)
                {
                    lock (Console.Out)
                    {
                        Item item = message.As<Item>();
                        IReadOnlyDictionary<string, object> myApplicationProperties = message.ApplicationProperties;
                        Console.WriteLine($"StoreId={myApplicationProperties["StoreId"]}");
                        if (message.Subject != null)
                        {
                            Console.WriteLine($"Subject={message.Subject}");
                        }
                        Console.WriteLine(
                            $"Item data: Price={item.GetPrice()}, Color={item.GetColor()}, Category={item.GetItemCategory()}");
                    }

                    await receiver.CompleteMessageAsync(message);
                }
            }
            else
            {
                break;
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.ToString());
        }
    }
}
```

> [!NOTE]
> Vous pouvez gérer les ressources Service Bus à l'aide de [Service Bus Explorer](https://github.com/paolosalvatori/ServiceBusExplorer/). Service Bus Explorer permet aux utilisateurs de se connecter à un espace de noms Service Bus et de gérer les entités de messagerie en toute simplicité. L’outil fournit des fonctionnalités avancées telles que la fonction d’importation/exportation ou la possibilité de tester une rubrique, des files d’attente, des abonnements, des services de relais, des hubs de notification et des hubs d’événements. 

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez provisionné des ressources à l’aide du portail Azure, puis envoyé et reçu des messages à partir d’une rubrique Service Bus et de ses abonnements. Vous avez appris à :

> [!div class="checklist"]
> * Créer une rubrique Service Bus et un ou plusieurs abonnements à cette rubrique à l’aide du portail Azure
> * Ajouter des filtres de rubrique à l’aide de code .NET
> * Créer deux messages avec un contenu différent
> * Envoyer les messages et vérifiez qu’ils arrivent dans les abonnements attendus
> * Recevoir des messages des abonnements

Pour plus d’exemples d’envoi et de réception de messages, démarrez avec [les exemples Service Bus sur GitHub](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/GettingStarted).

Passez au prochain tutoriel pour en savoir plus sur les fonctionnalités de publication/abonnement de Service Bus.

> [!div class="nextstepaction"]
> [Répondre à des événements via Event Grid](service-bus-to-event-grid-integration-example.md)

[compte gratuit]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
[fully qualified domain name]: https://wikipedia.org/wiki/Fully_qualified_domain_name
[Azure portal]: https://portal.azure.com/

[connection-string]: ./media/service-bus-tutorial-topics-subscriptions-portal/connection-string.png
[service-bus-flow]: ./media/service-bus-tutorial-topics-subscriptions-portal/about-service-bus-topic.png

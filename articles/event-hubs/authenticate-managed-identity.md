---
title: Authentification d’une identité managée avec Azure Active Directory
description: Cet article fournit des informations sur l’authentification d’une identité managée avec Azure Active Directory pour accéder aux ressources Azure Event Hubs.
ms.topic: conceptual
ms.date: 06/23/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: c6b43cc48663be28d12fa788d92286be6f47ef08
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95993531"
---
# <a name="authenticate-a-managed-identity-with-azure-active-directory-to-access-event-hubs-resources"></a>Authentifier une identité managée avec Azure Active Directory pour accéder aux ressources Event Hubs
Azure Event Hubs prend en charge l’authentification Azure Active Directory (Azure AD) avec des [identités managées pour ressources Azure](../active-directory/managed-identities-azure-resources/overview.md). Les identités managées pour ressources Azure peuvent autoriser l’accès à des ressources Event Hubs en utilisant les informations d’identification Azure AD d’applications s’exécutant dans des machines virtuelles Azure, des applications de fonction, le service Virtual Machine Scale Sets et d’autres services. En utilisant des identités managées pour ressources Azure et Azure AD Authentication, vous pouvez éviter de stocker des informations d’identification avec les applications qui s’exécutent dans le cloud.

Cet article montre comment autoriser l’accès à un hub d’événements en utilisant une identité managée à partir d’une machine virtuelle Azure.

## <a name="enable-managed-identities-on-a-vm"></a>Activer les identités managées sur une machine virtuelle
Avant de pouvoir utiliser les identités managées pour ressources Azure en vue d’autoriser l’accès aux ressources Event Hubs à partir de votre machine virtuelle, vous devez les activer sur la machine virtuelle. Pour savoir comment activer des identités managées pour ressources Azure, consultez un de ces articles :

- [Azure portal](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)
- [Azure PowerShell](../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)
- [Azure CLI](../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)
- [Modèle Azure Resource Manager](../active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm.md)
- [Bibliothèques clientes Azure Resource Manager](../active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm.md)

## <a name="grant-permissions-to-a-managed-identity-in-azure-ad"></a>Octroyer des autorisations à une identité managée dans Azure AD
Pour autoriser une requête auprès du service Event Hubs à partir d’une identité managée dans votre application, commencez par configurer les paramètres du contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour cette identité managée. Azure Event Hubs définit les rôles Azure qui englobent les autorisations d’envoi et de lecture à partir d’Event Hubs. Quand le rôle Azure est attribué à une identité managée, celle-ci est autorisée à accéder aux données Event Hubs selon l’étendue appropriée.

Pour plus d’informations sur l’attribution des rôles Azure, consultez [Autoriser l’accès aux ressources Event Hubs à l’aide d’Azure Active Directory](authorize-access-azure-active-directory.md).

## <a name="use-event-hubs-with-managed-identities"></a>Utiliser Event Hubs avec des identités managées
Pour utiliser Event Hubs avec des identités managées, vous devez attribuer le rôle et l'étendue appropriée à l'identité. La procédure décrite dans cette section utilise une application simple qui s’exécute sous une identité managée et accède aux ressources Event Hubs.

Ici, nous utilisons un exemple d’application web hébergée dans [Azure App Service](https://azure.microsoft.com/services/app-service/). Pour obtenir des instructions pas à pas sur la création d’une application web, consultez [Créer une application web ASP.NET Core dans Azure](../app-service/quickstart-dotnetcore.md)

Une fois que vous avez créé l’application, suivez ces étapes : 

1. Accédez à **Paramètres** et sélectionnez **Identité**. 
1. Définissez **État** sur **Activé**. 
1. Sélectionnez **Enregistrer** pour enregistrer le paramètre. 

    ![Identité managée pour une application web](./media/authenticate-managed-identity/identity-web-app.png)

Une fois ce paramètre activé, une identité de service est créée dans votre annuaire Azure Active Directory (Azure AD) et configurée dans l’hôte App Service.

À présent, attribuez cette identité de service à un rôle dans l’étendue requise dans vos ressources Event Hubs.

### <a name="to-assign-azure-roles-using-the-azure-portal"></a>Pour attribuer des rôles Azure à l’aide du portail Azure
Pour attribuer un rôle aux ressources Event Hubs, accédez à cette ressource dans le portail Azure. Affichez les paramètres Contrôle d’accès (IAM) pour la ressource et suivez ces instructions pour gérer les attributions de rôle :

> [!NOTE]
> Les étapes suivantes vous permettent d’attribuer un rôle d’identité de service à vos espaces de noms Event Hubs. Vous pouvez suivre les mêmes étapes pour attribuer un rôle étendu à n’importe quelle ressource Event Hubs. 

1. Dans le portail Azure, accédez à votre espace de noms Event Hubs, puis affichez la **vue d’ensemble** de l’espace de noms. 
1. Sélectionnez **Contrôle d’accès (IAM)** dans le menu de gauche pour afficher les paramètres du contrôle d’accès pour le hub d’événements.
1.  Sélectionnez l’onglet **Attributions de rôles** pour afficher la liste des attributions de rôles.
3.  Sélectionnez **Ajouter** pour ajouter un nouveau rôle.
4.  Dans la page **Ajouter une attribution de rôle**, sélectionnez les rôles Event Hubs que vous souhaitez attribuer. Recherchez ensuite l’identité de service que vous avez inscrite pour attribuer le rôle.
    
    ![Page Ajouter une attribution de rôle](./media/authenticate-managed-identity/add-role-assignment-page.png)
5.  Sélectionnez **Enregistrer**. L’identité à laquelle vous avez attribué le rôle apparaît sous ce dernier. Par exemple, l’image suivante montre que l’identité de service a le rôle Propriétaire des données Azure Event Hubs.
    
    ![Identité attribuée à un rôle](./media/authenticate-managed-identity/role-assigned.png)

Une fois que vous avez attribué le rôle, l’application web a accès aux ressources Event Hubs sous l’étendue définie. 

### <a name="test-the-web-application"></a>Tester l’application web
1. Créez un espace de noms Event Hubs et un hub d’événements. 
2. Déployer l’application web sur Azure. Pour obtenir des liens vers l’application web sur GitHub, voir la section à onglets suivante. 
3. Assurez-vous que SendReceive.aspx est défini en tant que document par défaut pour l’application web. 
3. Activez **identité** pour l’application web. 
4. Attribuez cette identité au rôle **Propriétaire de données Event Hub** au niveau de l’espace de noms ou du hub d’événements. 
5. Exécutez l’application web, entrez les noms de l’espace de noms et du hub d’événements, un message, puis sélectionnez **Envoyer**. Pour recevoir l’événement, sélectionnez **Recevoir**. 

#### <a name="azuremessagingeventhubs-latest"></a>[Azure.Messaging.EventHubs (dernier)](#tab/latest)
Vous pouvez maintenant lancer votre application web et faire pointer votre navigateur vers l’exemple de page aspx. Vous trouverez l’exemple d’application web qui envoie des données aux ressources Event Hubs et reçoit des données de ces ressources dans le [dépôt GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Azure.Messaging.EventHubs/ManagedIdentityWebApp).

Installez le package le plus récent à partir de [NuGet](https://www.nuget.org/packages/Azure.Messaging.EventHubs/) et commencez à envoyer des événements à Event Hubs en utilisant **EventHubProducerClient** et à recevoir des événements en utilisant **EventHubConsumerClient**. 

> [!NOTE]
> Pour obtenir un exemple Java qui utilise une identité managée pour publier des événements sur un Event Hub, consultez [l’exemple PublishEventsWithAzureIdentity.java sur GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/eventhubs/azure-messaging-eventhubs/src/samples/java/com/azure/messaging/eventhubs).

```csharp
protected async void btnSend_Click(object sender, EventArgs e)
{
    await using (EventHubProducerClient producerClient = new EventHubProducerClient(txtNamespace.Text, txtEventHub.Text, new DefaultAzureCredential()))
    {
        // create a batch
        using (EventDataBatch eventBatch = await producerClient.CreateBatchAsync())
        {

            // add events to the batch. only one in this case. 
            eventBatch.TryAdd(new EventData(Encoding.UTF8.GetBytes(txtData.Text)));

            // send the batch to the event hub
            await producerClient.SendAsync(eventBatch);
        }

        txtOutput.Text = $"{DateTime.Now} - SENT{Environment.NewLine}{txtOutput.Text}";
    }
}
protected async void btnReceive_Click(object sender, EventArgs e)
{
    await using (var consumerClient = new EventHubConsumerClient(EventHubConsumerClient.DefaultConsumerGroupName, $"{txtNamespace.Text}.servicebus.windows.net", txtEventHub.Text, new DefaultAzureCredential()))
    {
        int eventsRead = 0;
        try
        {
            using CancellationTokenSource cancellationSource = new CancellationTokenSource();
            cancellationSource.CancelAfter(TimeSpan.FromSeconds(5));

            await foreach (PartitionEvent partitionEvent in consumerClient.ReadEventsAsync(cancellationSource.Token))
            {
                txtOutput.Text = $"Event Read: { Encoding.UTF8.GetString(partitionEvent.Data.Body.ToArray()) }{ Environment.NewLine}" + txtOutput.Text;
                eventsRead++;
            }
        }
        catch (TaskCanceledException ex)
        {
            txtOutput.Text = $"Number of events read: {eventsRead}{ Environment.NewLine}" + txtOutput.Text;
        }
    }
}
```

#### <a name="microsoftazureeventhubs-legacy"></a>[Microsoft.Azure.EventHubs (hérité)](#tab/old)
Vous pouvez maintenant lancer votre application web et faire pointer votre navigateur vers l’exemple de page aspx. Vous trouverez l’exemple d’application web qui envoie des données aux ressources Event Hubs et reçoit des données de ces ressources dans le [dépôt GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/Rbac/ManagedIdentityWebApp).

Installez le package le plus récent à partir de [NuGet](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/) et commencez à échanger des données avec des hubs d’événements en utilisant EventHubClient comme dans le code suivant : 

```csharp
var ehClient = EventHubClient.CreateWithManagedIdentity(new Uri($"sb://{EventHubNamespace}/"), EventHubName);
```
---

## <a name="event-hubs-for-kafka"></a>Event Hubs pour Kafka
Vous pouvez utiliser les applications Apache Kafka pour envoyer et recevoir des messages dans Azure Event Hubs à l’aide d’une identité managée OAuth. Consultez l’exemple suivant sur GitHub : [Event Hubs pour Kafka - Envoyer et recevoir des messages à l’aide d’une identité managée OAuth](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/oauth/java/managedidentity).

## <a name="samples"></a>Exemples
- Exemples **Azure.Messaging.EventHubs**
    - [.NET](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Azure.Messaging.EventHubs/ManagedIdentityWebApp)
    - [Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/eventhubs/azure-messaging-eventhubs/src/samples/java/com/azure/messaging/eventhubs)
- [Exemples Microsoft.Azure.EventHubs](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/Rbac). 
    
    Ces exemples utilisent l'ancienne bibliothèque **Microsoft.Azure.EventHubs**, mais vous pouvez facilement mettre celle-ci à jour à l'aide de la dernière bibliothèque **Azure.Messaging.EventHubs**. Pour migrer l'exemple de l'ancienne bibliothèque vers la nouvelle, consultez le [Guide de migration de Microsoft.Azure.EventHubs vers Azure.Messaging.EventHubs](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/eventhub/Azure.Messaging.EventHubs/MigrationGuide.md).
    Cet exemple a été mis à jour pour utiliser la dernière bibliothèque **Azure.Messaging.EventHubs**.
- [Event Hubs pour Kafka - Envoyer et recevoir des messages à l’aide d’une identité managée OAuth](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/oauth/java/managedidentity)


## <a name="next-steps"></a>Étapes suivantes
- Consultez l’article suivant pour en savoir plus sur les identités managées pour ressources Azure : [Que sont les identités managées pour les ressources Azure ?](../active-directory/managed-identities-azure-resources/overview.md)
- Consultez les articles associés suivants :
    - [Authentifier les requêtes adressées à Azure Event Hubs à partir d’une application à l’aide d’Azure Active Directory](authenticate-application.md)
    - [Authentifier les requêtes adressées à Azure Event Hubs à l’aide de signatures d’accès partagé](authenticate-shared-access-signature.md)
    - [Autoriser l’accès aux ressources Event Hubs à l’aide d’Azure Active Directory](authorize-access-azure-active-directory.md)
    - [Authentifier l’accès aux ressources Event Hubs avec des signatures d’accès partagé](authorize-access-shared-access-signature.md)

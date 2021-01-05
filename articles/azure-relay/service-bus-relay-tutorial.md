---
title: Exposer un service REST WCF local aux clients à l’aide d’Azure Relay
description: Ce tutoriel explique comment exposer un service WCF REST local à un client externe avec Azure WCF Relay.
ms.topic: tutorial
ms.custom: devx-track-dotnet
ms.date: 06/23/2020
ms.openlocfilehash: bb2b9b5ed7c263762cc24b8eb2e6d66215147c4c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88935702"
---
# <a name="tutorial-expose-an-on-premises-wcf-rest-service-to-external-client-by-using-azure-wcf-relay"></a>Tutoriel : Exposer un service WCF REST local sur un client externe en utilisant Azure WCF Relay

Ce didacticiel vous montre comment créer une application cliente et un service WCF Relay à l’aide d’Azure Relay. Pour obtenir un didacticiel similaire utilisant la [messagerie Service Bus](../service-bus-messaging/service-bus-messaging-overview.md), consultez [Bien démarrer avec les files d’attente Service Bus](../service-bus-messaging/service-bus-dotnet-get-started-with-queues.md).

Ce didacticiel vous offre une présentation des étapes de création d’une application cliente et d’une application de service WCF Relay. À l’instar de leurs homologues WCF d’origine, un service est une construction qui expose un ou plusieurs points de terminaison. Chaque point de terminaison expose une ou plusieurs opérations de service. Le point de terminaison de service spécifie une adresse où le service peut être trouvé, une liaison contenant les informations qu’un client doit communiquer avec le service et le contrat qui définit la fonctionnalité fournie par le service à ses clients. La principale différence entre WCF et WCF Relay est que le point de terminaison est exposé dans le cloud et non localement sur votre ordinateur.

Après avoir parcouru les sections de ce didacticiel, vous disposez d’un service opérationnel. Vous disposez également d’un client qui peut appeler les opérations du service. 

Dans ce didacticiel, vous allez effectuer les tâches suivantes :

> [!div class="checklist"]
>
> * Installer la configuration requise pour ce didacticiel.
> * Créer un espace de noms Relay
> * Définir un contrat de service WCF
> * Implémenter le contrat WCF
> * Héberger le service WCF pour s’enregistrer auprès du service de relais
> * Créer un client WCF pour le contrat de service
> * Configurer le client WCF
> * Implémenter le client WCF
> * Exécutez les applications.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce didacticiel, vous avez besoin de ce qui suit :

* Un abonnement Azure. Si vous n’en avez pas, [créez un compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* [Visual Studio 2015 ou version ultérieure](https://www.visualstudio.com). Les exemples de ce didacticiel utilisent Visual Studio 2019.
* SDK Azure pour .NET. Installez-le à partir de la [page de téléchargement des SDK](https://azure.microsoft.com/downloads/).

## <a name="create-a-relay-namespace"></a>Créer un espace de noms Relay

La première étape consiste à créer un espace de noms et à obtenir une clé de [signature d’accès partagé (SAP)](../service-bus-messaging/service-bus-sas.md). Un espace de noms fournit une limite d’application pour chaque application exposée via le service de relais. Une clé SAS est automatiquement générée par le système lors de la création d’un espace de noms de service. La combinaison de l’espace de noms de service et de la clé SAP fournit à Azure des informations d’identification permettant d’authentifier l’accès à une application.

[!INCLUDE [relay-create-namespace-portal](../../includes/relay-create-namespace-portal.md)]

## <a name="define-a-wcf-service-contract"></a>Définition d’un contrat de service WCF

Le contrat de service spécifie les opérations prises en charge par le service. Les opérations sont des méthodes ou des fonctions de service Web. Les contrats sont créés en définissant une interface C++, C# ou Visual Basic. Chaque méthode dans l'interface correspond à une opération de service spécifique. L’attribut [ServiceContractAttribute](/dotnet/api/system.servicemodel.servicecontractattribute) doit être appliqué à chaque interface et l’attribut [OperationContractAttribute](/dotnet/api/system.servicemodel.operationcontractattribute) doit être appliqué à chaque opération. Si une méthode présente dans une interface contenant l’attribut [ServiceContractAttribute](/dotnet/api/system.servicemodel.servicecontractattribute) ne comporte pas l’attribut [OperationContractAttribute](/dotnet/api/system.servicemodel.operationcontractattribute), cette méthode n’est pas exposée. Le code servant à effectuer ces tâches est fourni dans l’exemple qui suit la procédure. Pour une description plus approfondie des contrats et des services, consultez [Conception et implémentation de services](/dotnet/framework/wcf/designing-and-implementing-services).

### <a name="create-a-relay-contract-with-an-interface"></a>Création d’un contrat de relais avec une interface

1. Lancez Microsoft Visual Studio en tant qu'administrateur. Pour ce faire, cliquez avec le bouton droit sur l’icône de programme Visual Studio, puis sélectionnez **Exécuter en tant qu’administrateur**.
1. Dans Visual Studio, sélectionnez **Créer un nouveau projet**.
1. Dans **Créer un projet**, sélectionnez **Application console (.NET Framework)** pour C#, puis **Suivant**.
1. Nommez le projet *EchoService* et sélectionnez **Créer**.

   ![Créer une application console][2]

1. Dans **l’Explorateur de solutions**, cliquez avec le bouton droit sur le projet, puis sélectionnez **Gérer les packages NuGet**. Dans **Gestionnaire de package NuGet**, sélectionnez **Parcourir**, puis recherchez et choisissez **WindowsAzure.ServiceBus**. Sélectionnez **Installer** et acceptez les conditions d’utilisation.

    ![Package Service Bus][3]

   Ce package ajoute automatiquement des références aux bibliothèques Service Bus et au `System.ServiceModel` de WCF. [System.ServiceModel](/dotnet/api/system.servicemodel) est l’espace de noms qui vous permet d’accéder par programme aux fonctionnalités WCF de base. Service Bus utilise la plupart des objets et attributs de WCF pour définir des contrats de service.

1. Ajoutez les instructions `using` suivantes au début de *Program.cs* :

    ```csharp
    using System.ServiceModel;
    using Microsoft.ServiceBus;
    ```

1. Remplacez le nom par défaut de l'espace de noms `EchoService` par `Microsoft.ServiceBus.Samples`.

   > [!IMPORTANT]
   > Ce didacticiel utilise l’espace de noms C# `Microsoft.ServiceBus.Samples` , qui est l’espace de noms du type géré par contrat utilisé dans le fichier de configuration à l’étape [Configurer le client WCF](#configure-the-wcf-client). Vous pouvez spécifier n’importe quel espace de noms lorsque vous générez cet exemple. Toutefois, le didacticiel ne fonctionnera que si vous modifiez les espaces de noms du contrat et le service en conséquence, dans le fichier de configuration d’application de service. L’espace de noms spécifié dans le fichier *App.config* doit être identique à l’espace de noms spécifié dans vos fichiers C#.
   >

1. Définissez directement après la déclaration d’espace de noms `Microsoft.ServiceBus.Samples`, mais à l’intérieur de l’espace de noms, une nouvelle interface nommée `IEchoContract` et appliquez l’attribut `ServiceContractAttribute` à cette interface avec une valeur d’espace de noms de `https://samples.microsoft.com/ServiceModel/Relay/`. Collez le code suivant après la déclaration de l’espace de noms :

    ```csharp
    [ServiceContract(Name = "IEchoContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IEchoContract
    {
    }
    ```

    La valeur de l'espace de noms diffère de l'espace de noms que vous utilisez dans l’ensemble de votre code. En revanche, la valeur de l’espace de noms est utilisée comme identificateur unique pour ce contrat. Spécifier explicitement l'espace de noms empêche l'ajout au nom du contrat de la valeur d'espace de noms par défaut.

   > [!NOTE]
   > En règle générale, l’espace de noms de contrat de service contient un schéma d’affectation de noms qui inclut des informations de version. L’inclusion des informations de version dans l’espace de noms de contrat de service permet aux services d’isoler les modifications majeures en définissant un contrat de service avec un nouvel espace de noms et en l’exposant sur un point de terminaison. De cette manière, les clients peuvent continuer à utiliser l’ancien contrat de service sans avoir à procéder à la mise à jour. Les informations de version peuvent se composer d’une date ou un numéro de version. Pour plus d’informations, consultez la rubrique [Contrôle de version des services](/dotnet/framework/wcf/service-versioning). Dans ce didacticiel, le schéma d’affectation de noms de l’espace de noms de contrat de service ne contient pas les informations de version.
   >

1. Dans l’interface `IEchoContract`, déclarez une méthode pour l’unique opération exposée par le contrat `IEchoContract` dans l’interface, puis appliquez l’attribut `OperationContractAttribute` à la méthode que vous souhaitez exposer dans le cadre du contrat WCF Relay public, de la façon suivante :

    ```csharp
    [OperationContract]
    string Echo(string text);
    ```

1. Directement après la définition de l’interface `IEchoContract`, déclarez un canal qui hérite à la fois de `IEchoContract` et de l’interface `IClientChannel`, comme indiqué ici :

    ```csharp
    public interface IEchoChannel : IEchoContract, IClientChannel { }
    ```

    Un canal est l’objet WCF par le biais duquel l’hôte et le client se transmettent des informations. Par la suite , vous allez écrire du code par rapport au canal pour reprendre les informations entre les deux applications.

1. Sélectionnez **Générer** > **Générer la solution** ou sélectionnez Ctrl+Maj+B pour confirmer que votre travail est correct jusqu’à présent.

### <a name="example-of-a-wcf-contract"></a>Exemple de contrat WCF

Le code suivant montre une interface de base qui définit un contrat WCF Relay.

```csharp
using System;
using System.ServiceModel;

namespace Microsoft.ServiceBus.Samples
{
    [ServiceContract(Name = "IEchoContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IEchoContract
    {
        [OperationContract]
        String Echo(string text);
    }

    public interface IEchoChannel : IEchoContract, IClientChannel { }

    class Program
    {
        static void Main(string[] args)
        {
        }
    }
}
```

Maintenant que l’interface est créée, vous pouvez implémenter l’interface.

## <a name="implement-the-wcf-contract"></a>Implémentation du contrat WCF

La création d’un relais Azure nécessite la création au préalable du contrat à l’aide d’une interface. Pour plus d’informations sur la création de l’interface, consultez la section précédente. La procédure suivante implémente l’interface. Cette tâche implique la création d’une classe nommée `EchoService` qui met en œuvre l’interface `IEchoContract` définie par l’utilisateur. Une fois l’interface implémentée, vous configurez l’interface à l’aide d’un fichier de configuration *App.config*. Le fichier de configuration contient les informations nécessaires à l’application. Ces informations comprennent le nom du service, le nom du contrat et le type de protocole utilisé pour communiquer avec le service de relais. Le code utilisé pour effectuer ces tâches est fourni dans l'exemple suivant la procédure. Pour obtenir une description plus générale sur la manière d’implémenter un contrat de service, consultez [Implémentation de contrats de service](/dotnet/framework/wcf/implementing-service-contracts).

1. Créez une classe nommée `EchoService` directement après la définition de l’interface `IEchoContract`. La classe `EchoService` implémente l’interface `IEchoContract`.

    ```csharp
    class EchoService : IEchoContract
    {
    }
    ```

    Comme pour d'autres implémentations d'interface, vous pouvez implémenter la définition dans un autre fichier. Toutefois, pour ce didacticiel, l’implémentation se situe dans le même fichier que la définition d’interface et la méthode `Main()`.

1. Appliquez l’attribut [ServiceBehaviorAttribute](/dotnet/api/system.servicemodel.servicebehaviorattribute) à l’interface `IEchoContract`. L’attribut spécifie le nom du service ainsi que son espace de noms. Ensuite, la classe `EchoService` apparaît comme suit :

    ```csharp
    [ServiceBehavior(Name = "EchoService", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    class EchoService : IEchoContract
    {
    }
    ```

1. Implémentez la méthode `Echo` définie dans l’interface `IEchoContract` dans la classe `EchoService`.

    ```csharp
    public string Echo(string text)
    {
        Console.WriteLine("Echoing: {0}", text);
        return text;
    }
    ```

1. Sélectionnez **Générer** > **Générer la solution** ou sélectionnez Ctrl+Maj+B.

### <a name="define-the-configuration-for-the-service-host"></a>Définition de la configuration de l’hôte de service

Le fichier de configuration est similaire à un fichier de configuration WCF. Il inclut le nom du service, le point de terminaison et la liaison. Le point de terminaison correspond à l'emplacement qu’Azure Relay expose pour permettre aux clients et aux hôtes de communiquer. La liaison correspond au type de protocole utilisé pour communiquer. La principale différence réside dans le fait que le point de terminaison de service configuré fait référence à la liaison [NetTcpRelayBinding](/dotnet/api/microsoft.servicebus.nettcprelaybinding), qui ne fait pas partie de .NET Framework. [NetTcpRelayBinding](/dotnet/api/microsoft.servicebus.nettcprelaybinding) est l’une des liaisons définies par le service.

1. Dans l'**Explorateur de solutions**, double-cliquez sur le fichier **App.config** pour l’ouvrir dans l’éditeur Visual Studio.
1. Dans l’élément `<appSettings>`, remplacez les espaces réservés par le nom de votre espace de noms de service et par la clé SAP que vous avez copiée à l’étape précédente.
1. Dans les balises `<system.serviceModel>`, ajouter un élément `<services>`. Vous pouvez définir plusieurs applications de relais dans un même fichier de configuration. Toutefois, ce didacticiel n’en définit qu’un seul.

    ```xml
    <?xmlversion="1.0"encoding="utf-8"?>
    <configuration>
      <system.serviceModel>
        <services>

        </services>
      </system.serviceModel>
    </configuration>
    ```

1. Dans l’élément `<services>`, ajoutez un élément `<service>` pour définir le nom du service.

    ```xml
    <service name="Microsoft.ServiceBus.Samples.EchoService">
    </service>
    ```

1. Dans l’élément `<service>`, définissez l’emplacement du contrat de point de terminaison, ainsi que le type de liaison de ce dernier.

    ```xml
    <endpoint contract="Microsoft.ServiceBus.Samples.IEchoContract" binding="netTcpRelayBinding"/>
    ```

    Le point de terminaison définit l’emplacement où le client recherchera l’application hôte. Plus tard, le didacticiel utilise cette étape pour créer une URI qui expose entièrement l’hôte via Azure Relay. La liaison déclare que nous utilisons TCP comme protocole pour communiquer avec le service de relais.

1. Sélectionnez **Générer** > **Générer la solution** ou sélectionnez Ctrl+Maj+B pour confirmer que votre travail est correct jusqu’à présent.

### <a name="example-of-implementation-of-a-service-contract"></a>Exemple d’implémentation d’un contrat de service

Le code suivant illustre l’implémentation du contrat de service.

```csharp
[ServiceBehavior(Name = "EchoService", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]

    class EchoService : IEchoContract
    {
        public string Echo(string text)
        {
            Console.WriteLine("Echoing: {0}", text);
            return text;
        }
    }
```

Le code suivant montre le format de base du fichier *App.config* associé à l’hôte de service.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <system.serviceModel>
    <services>
      <service name="Microsoft.ServiceBus.Samples.EchoService">
        <endpoint contract="Microsoft.ServiceBus.Samples.IEchoContract" binding="netTcpRelayBinding" />
      </service>
    </services>
    <extensions>
      <bindingExtensions>
        <add name="netTcpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.NetTcpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
      </bindingExtensions>
    </extensions>
  </system.serviceModel>
</configuration>
```

## <a name="host-and-run-the-wcf-service-to-register-with-the-relay-service"></a>Hébergement et exécution du service WCF pour s’enregistrer auprès du service de relais

Cette étape décrit comment exécuter un service Azure Relay.

### <a name="create-the-relay-credentials"></a>Création des informations d’identification du relais

1. Dans `Main()`, créez deux variables dans lesquelles stocker l’espace de noms et la clé SAS qui sont lues à partir de la fenêtre de console.

    ```csharp
    Console.Write("Your Service Namespace: ");
    string serviceNamespace = Console.ReadLine();
    Console.Write("Your SAS key: ");
    string sasKey = Console.ReadLine();
    ```

    La clé SAP sera utilisée ultérieurement pour accéder à votre projet. L’espace de noms est transmis en tant que paramètre à `CreateServiceUri` pour créer une URI de service.

1. À l’aide d’un objet [TransportClientEndpointBehavior](/dotnet/api/microsoft.servicebus.transportclientendpointbehavior), déclarez que vous utiliserez une clé SAP en tant que type d’informations d’identification. Ajoutez le code suivant directement après le code ajouté à l’étape précédente.

    ```csharp
    TransportClientEndpointBehavior sasCredential = new TransportClientEndpointBehavior();
    sasCredential.TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider("RootManageSharedAccessKey", sasKey);
    ```

### <a name="create-a-base-address-for-the-service"></a>Création d’une adresse de base pour le service

Après le code ajouté à la section précédente, créez une instance `Uri` pour l’adresse de base du service. Cette URI spécifie le schéma Service Bus, l’espace de noms et le chemin d’accès de l’interface de service.

```csharp
Uri address = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, "EchoService");
```

La valeur « sb » est une abréviation pour le schéma de Service Bus. Elle indique que nous utilisons le protocole TCP. Ce schéma était précédemment indiqué dans le fichier de configuration, lorsque [NetTcpRelayBinding](/dotnet/api/microsoft.servicebus.nettcprelaybinding) a été spécifié en tant que liaison.

Pour ce didacticiel, l’URI est `sb://putServiceNamespaceHere.windows.net/EchoService`.

### <a name="create-and-configure-the-service-host"></a>Création et configuration de l’hôte de service

1. Toujours dans `Main()`, définissez le mode connectivité sur `AutoDetect`.

    ```csharp
    ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.AutoDetect;
    ```

    Le mode de connectivité décrit le protocole que le service utilise pour communiquer avec le service de relais ; HTTP ou TCP. Avec la valeur `AutoDetect` par défaut, le service tente de se connecter à Azure Relay via TCP s’il est disponible et HTTP dans le cas contraire. Ce résultat diffère du protocole du service spécifié pour la communication client. Ce protocole est déterminé par la liaison utilisée. Par exemple, un service peut utiliser la liaison [BasicHttpRelayBinding](/dotnet/api/microsoft.servicebus.basichttprelaybinding), qui indique que son point de terminaison communique avec les clients via HTTP. Ce même service peut spécifier `ConnectivityMode.AutoDetect` de manière que le service communique avec Azure Relay via TCP.

1. Créez l’hôte de service, en utilisant l’URI créée précédemment dans cette section.

    ```csharp
    ServiceHost host = new ServiceHost(typeof(EchoService), address);
    ```

    L’hôte de service est l’objet WCF qui instancie le service. Ici, vous transmettez le type de service que vous souhaitez créer, un type `EchoService`, ainsi que l’adresse à laquelle vous souhaitez exposer le service.

1. En haut du fichier *Program.cs*, ajoutez des références à [System.ServiceModel.Description](/dotnet/api/system.servicemodel.description) et [Microsoft.ServiceBus.Description](/dotnet/api/microsoft.servicebus.description).

    ```csharp
    using System.ServiceModel.Description;
    using Microsoft.ServiceBus.Description;
    ```

1. De retour dans `Main()`, configurez le point de terminaison pour activer l’accès public.

    ```csharp
    IEndpointBehavior serviceRegistrySettings = new ServiceRegistrySettings(DiscoveryType.Public);
    ```

    Cette étape informe le service de relais que votre application peut être trouvée publiquement en examinant le flux Atom de votre projet. Si vous définissez `DiscoveryType` sur `private`, un client peut toujours accéder au service. Toutefois, le service n’apparaît pas lorsqu’il recherche l’espace de noms `Relay`. Au lieu de ça, le client doit connaître le chemin d’accès du point de terminaison au préalable.

1. Appliquez les informations d’identification de service aux points de terminaison de service définis dans le fichier *App.config* :

    ```csharp
    foreach (ServiceEndpoint endpoint in host.Description.Endpoints)
    {
        endpoint.Behaviors.Add(serviceRegistrySettings);
        endpoint.Behaviors.Add(sasCredential);
    }
    ```

    Comme indiqué précédemment, il se peut que vous ayez déclaré plusieurs services et points de terminaison dans le fichier de configuration. Si c’est le cas, ce code traverse le fichier de configuration et recherche chaque point de terminaison auquel vos informations d’identification doivent être appliquées. Pour ce didacticiel, le fichier de configuration n’a qu’un seul point de terminaison.

### <a name="open-the-service-host"></a>Ouverture de l’hôte de service

1. Toujours dans `Main()`, ajoutez la ligne suivante pour ouvrir le service.

    ```csharp
    host.Open();
    ```

1. Informez l’utilisateur que le service est en cours d’exécution et expliquez comment arrêter le service.

    ```csharp
    Console.WriteLine("Service address: " + address);
    Console.WriteLine("Press [Enter] to exit");
    Console.ReadLine();
    ```

1. Lorsque vous avez terminé, fermez l'hôte de service.

    ```csharp
    host.Close();
    ```

1. Sélectionnez Ctrl+Maj+B pour générer le projet.

### <a name="example-that-hosts-a-service-in-a-console-application"></a>Exemple qui héberge un service dans une application console

Une fois terminé, votre code de service doit se présenter ainsi. Il comprend le contrat de service et l’implémentation des étapes précédentes du didacticiel, et héberge le service dans une application console.

```csharp
using System;
using System.ServiceModel;
using System.ServiceModel.Description;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Description;

namespace Microsoft.ServiceBus.Samples
{
    [ServiceContract(Name = "IEchoContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IEchoContract
    {
        [OperationContract]
        String Echo(string text);
    }

    public interface IEchoChannel : IEchoContract, IClientChannel { };

    [ServiceBehavior(Name = "EchoService", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    class EchoService : IEchoContract
    {
        public string Echo(string text)
        {
            Console.WriteLine("Echoing: {0}", text);
            return text;
        }
    }

    class Program
    {
        static void Main(string[] args)
        {

            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.AutoDetect;         

            Console.Write("Your Service Namespace: ");
            string serviceNamespace = Console.ReadLine();
            Console.Write("Your SAS key: ");
            string sasKey = Console.ReadLine();

           // Create the credentials object for the endpoint.
            TransportClientEndpointBehavior sasCredential = new TransportClientEndpointBehavior();
            sasCredential.TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider("RootManageSharedAccessKey", sasKey);

            // Create the service URI based on the service namespace.
            Uri address = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, "EchoService");

            // Create the service host reading the configuration.
            ServiceHost host = new ServiceHost(typeof(EchoService), address);

            // Create the ServiceRegistrySettings behavior for the endpoint.
            IEndpointBehavior serviceRegistrySettings = new ServiceRegistrySettings(DiscoveryType.Public);

            // Add the Relay credentials to all endpoints specified in configuration.
            foreach (ServiceEndpoint endpoint in host.Description.Endpoints)
            {
                endpoint.Behaviors.Add(serviceRegistrySettings);
                endpoint.Behaviors.Add(sasCredential);
            }

            // Open the service.
            host.Open();

            Console.WriteLine("Service address: " + address);
            Console.WriteLine("Press [Enter] to exit");
            Console.ReadLine();

            // Close the service.
            host.Close();
        }
    }
}
```

## <a name="create-a-wcf-client-for-the-service-contract"></a>Créer un client WCF pour le contrat de service

La tâche suivante consiste à créer une application cliente et à définir le contrat de service que vous allez implémenter ultérieurement. Ces étapes ressemblent aux étapes utilisées pour créer un service : définition d’un contrat, modification d’un fichier *App.config* à l’aide des informations d’identification servant à se connecter au service de relais et ainsi de suite. Le code utilisé pour effectuer ces tâches est fourni dans l'exemple suivant la procédure.

1. Créer un nouveau projet dans la solution Visual Studio en cours pour le client :

   1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur la solution actuelle (et non sur le projet) et sélectionnez **Ajouter** > **Nouveau projet**.
   1. Dans **Ajouter un nouveau projet**, sélectionnez **Application console (.NET Framework)** pour C#, puis **Suivant**.
   1. Nommez le projet *EchoClient* et sélectionnez **Créer**.

1. Dans l’**Explorateur de solutions**, dans le projet **EchoClient**, double-cliquez sur **Program.cs** pour ouvrir le fichier dans l’éditeur si ce n’est déjà fait.
1. Remplacez le nom par défaut de l'espace de noms `EchoClient` par `Microsoft.ServiceBus.Samples`.
1. Installez le [package NuGet Service Bus](https://www.nuget.org/packages/WindowsAzure.ServiceBus) :

   1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur **EchoClient**, puis sélectionnez **Gérer les packages NuGet**.
   1. Sélectionnez **Parcourir**, puis recherchez et sélectionnez **WindowsAzure.ServiceBus**. Sélectionnez **Installer** et acceptez les conditions d’utilisation.

      ![Installer le package Service Bus][4]

1. Ajoutez une instruction `using` pour l’espace de noms [System.ServiceModel](/dotnet/api/system.servicemodel) dans le fichier *Program.cs*.

    ```csharp
    using System.ServiceModel;
    ```

1. Ajoutez la définition de contrat de service à l’espace de noms, comme illustré dans l’exemple suivant. Cette définition est identique à celle qui est utilisée dans le projet **Service**. Ajoutez ce code en haut de l’espace de noms `Microsoft.ServiceBus.Samples`.

    ```csharp
    [ServiceContract(Name = "IEchoContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IEchoContract
    {
        [OperationContract]
        string Echo(string text);
    }

    public interface IEchoChannel : IEchoContract, IClientChannel { }
    ```

1. Sélectionnez Ctrl+Maj+B pour générer le client.

### <a name="example-of-the-echoclient-project"></a>Exemple de projet EchoClient

Le code suivant montre l’état actuel du fichier *Program.cs* dans le projet **EchoClient**.

```csharp
using System;
using Microsoft.ServiceBus;
using System.ServiceModel;

namespace Microsoft.ServiceBus.Samples
{

    [ServiceContract(Name = "IEchoContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IEchoContract
    {
        [OperationContract]
        string Echo(string text);
    }

    public interface IEchoChannel : IEchoContract, IClientChannel { }


    class Program
    {
        static void Main(string[] args)
        {
        }
    }
}
```

## <a name="configure-the-wcf-client"></a>Configurer le client WCF

Dans cette étape, vous allez créer un fichier *App.config* d’une application cliente de base qui accède au service que vous avez créé précédemment dans ce didacticiel. Ce fichier *App.config* définit le contrat, la liaison et le nom du point de terminaison. Le code utilisé pour effectuer ces tâches est fourni dans l'exemple suivant la procédure.

1. Dans l’**Explorateur de solutions**, dans le projet **EchoClient**, double-cliquez sur **App.config** pour ouvrir le fichier dans l’éditeur de Visual Studio.
1. Dans l’élément `<appSettings>`, remplacez les espaces réservés par le nom de votre espace de noms de service et par la clé SAP que vous avez copiée à l’étape précédente.
1. Dans l’élément `system.serviceModel`, ajoutez un élément `<client>`.

    ```xml
    <?xmlversion="1.0"encoding="utf-8"?>
    <configuration>
      <system.serviceModel>
        <client>
        </client>
      </system.serviceModel>
    </configuration>
    ```

    Ce code montre que vous définissez une application client de type WCF.

1. Dans l ’élément `client`, définissez le nom, le contrat et le type de liaison du point de terminaison.

    ```xml
    <endpoint name="RelayEndpoint"
                    contract="Microsoft.ServiceBus.Samples.IEchoContract"
                    binding="netTcpRelayBinding"/>
    ```

    Ce code définit le nom du point de terminaison. Il définit également le contrat défini dans le service et le fait que l’application cliente utilise TCP pour communiquer avec Azure Relay. Le nom de point de terminaison est utilisé dans l’étape suivante pour associer cette configuration de point de terminaison à l’URI de service.

1. Sélectionnez **Fichier** > **Enregistrer tout**.

### <a name="example-of-the-appconfig-file"></a>Exemple de fichier App.config

Le code suivant montre le fichier *App.config* correspondant au client Echo.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <system.serviceModel>
    <client>
      <endpoint name="RelayEndpoint"
                      contract="Microsoft.ServiceBus.Samples.IEchoContract"
                      binding="netTcpRelayBinding"/>
    </client>
    <extensions>
      <bindingExtensions>
        <add name="netTcpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.NetTcpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
      </bindingExtensions>
    </extensions>
  </system.serviceModel>
</configuration>
```

## <a name="implement-the-wcf-client"></a>Implémentation du client WCF

Dans cette section, vous allez mettre en œuvre une application cliente de base qui accède au service que vous avez créé précédemment dans ce didacticiel. Comme le service, le client effectue de nombreuses opérations similaires pour accéder à Azure Relay :

* Définit le mode connectivité sur.
* Crée l’URI qui localise le service hôte.
* Définit les informations d’identification de sécurité.
* Applique les informations d’identification à la connexion.
* Ouvre la connexion.
* Effectue les tâches spécifiques à l’application.
* Ferme la connexion.

Toutefois, une des principales différences réside dans le fait que l’application cliente utilise un canal pour se connecter au service de relais. Le service utilise un appel à **ServiceHost**. Le code utilisé pour effectuer ces tâches est fourni dans l'exemple suivant la procédure.

### <a name="implement-a-client-application"></a>Implémentation d’une application cliente

1. Définissez le mode connectivité sur `AutoDetect`. Ajoutez le code suivant à la méthode `Main()` de l’application **EchoClient**.

    ```csharp
    ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.AutoDetect;
    ```

1. Définissez des variables pour stocker les valeurs de l’espace de noms de service et la clé SAS qui sont lues depuis la console.

    ```csharp
    Console.Write("Your Service Namespace: ");
    string serviceNamespace = Console.ReadLine();
    Console.Write("Your SAS Key: ");
    string sasKey = Console.ReadLine();
    ```

1. Créez l’URI qui définit l’emplacement de l’hôte dans votre projet Relay.

    ```csharp
    Uri serviceUri = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, "EchoService");
    ```

1. Créez l’objet d’informations d’identification de votre point de terminaison d’espace de noms du service.

    ```csharp
    TransportClientEndpointBehavior sasCredential = new TransportClientEndpointBehavior();
    sasCredential.TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider("RootManageSharedAccessKey", sasKey);
    ```

1. Création de la structure de canaux qui charge la configuration décrite dans le fichier *App.config*.

    ```csharp
    ChannelFactory<IEchoChannel> channelFactory = new ChannelFactory<IEchoChannel>("RelayEndpoint", new EndpointAddress(serviceUri));
    ```

    Une structure de canaux est un objet WCF qui crée un canal via lequel les services et les applications clients communiquent.

1. Appliquez les informations d’identification.

    ```csharp
    channelFactory.Endpoint.Behaviors.Add(sasCredential);
    ```

1. Créer et ouvrir le canal au service.

    ```csharp
    IEchoChannel channel = channelFactory.CreateChannel();
    channel.Open();
    ```

1. Écrire l’interface utilisateur de base et les fonctionnalités pour l’écho.

    ```csharp
    Console.WriteLine("Enter text to echo (or [Enter] to exit):");
    string input = Console.ReadLine();
    while (input != String.Empty)
    {
        try
        {
            Console.WriteLine("Server echoed: {0}", channel.Echo(input));
        }
        catch (Exception e)
        {
            Console.WriteLine("Error: " + e.Message);
        }
        input = Console.ReadLine();
    }
    ```

    Le code utilise l’instance de l’objet de canal en tant que proxy pour le service.

1. Fermez le canal ainsi que la structure.

    ```csharp
    channel.Close();
    channelFactory.Close();
    ```

### <a name="example-code-for-this-tutorial"></a>Exemple de code pour ce tutoriel

Une fois terminé, votre code doit se présenter ainsi. Ce code montre comment créer une application client, comment appeler les opérations du service et comment fermer le client une fois l’appel d’opération terminé.

```csharp
using System;
using Microsoft.ServiceBus;
using System.ServiceModel;

namespace Microsoft.ServiceBus.Samples
{
    [ServiceContract(Name = "IEchoContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IEchoContract
    {
        [OperationContract]
        String Echo(string text);
    }

    public interface IEchoChannel : IEchoContract, IClientChannel { }

    class Program
    {
        static void Main(string[] args)
        {
            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.AutoDetect;


            Console.Write("Your Service Namespace: ");
            string serviceNamespace = Console.ReadLine();
            Console.Write("Your SAS Key: ");
            string sasKey = Console.ReadLine();



            Uri serviceUri = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, "EchoService");

            TransportClientEndpointBehavior sasCredential = new TransportClientEndpointBehavior();
            sasCredential.TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider("RootManageSharedAccessKey", sasKey);

            ChannelFactory<IEchoChannel> channelFactory = new ChannelFactory<IEchoChannel>("RelayEndpoint", new EndpointAddress(serviceUri));

            channelFactory.Endpoint.Behaviors.Add(sasCredential);

            IEchoChannel channel = channelFactory.CreateChannel();
            channel.Open();

            Console.WriteLine("Enter text to echo (or [Enter] to exit):");
            string input = Console.ReadLine();
            while (input != String.Empty)
            {
                try
                {
                    Console.WriteLine("Server echoed: {0}", channel.Echo(input));
                }
                catch (Exception e)
                {
                    Console.WriteLine("Error: " + e.Message);
                }
                input = Console.ReadLine();
            }

            channel.Close();
            channelFactory.Close();

        }
    }
}
```

## <a name="run-the-applications"></a>Exécution des applications

1. Sélectionnez Ctrl+Maj+B pour générer la solution. Cette action génère le projet client et le projet de service que vous avez créés aux étapes précédentes.
1. Avant d’exécuter l’application cliente, assurez-vous que l’application de service est en cours d’exécution. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur la solution **EchoService**, puis sélectionnez **Propriétés**.
1. Dans **Page de propriétés**, **Propriétés communes** > **Projet de démarrage**, puis sélectionnez **Plusieurs projets de démarrage**. Assurez-vous que **EchoService** apparaît en premier dans la liste.
1. Définissez le champ **Action** sur **Démarrer** pour les projets **EchoService** et **EchoClient**.

    ![Pages de propriétés du projet][5]

1. Sélectionnez **Dépendances du projet**. Dans **Projets**, sélectionnez **EchoClient**. Dans **Dépend de**, assurez-vous que l’option **EchoService** est sélectionnée.

    ![Dépendances du projet][6]

1. Sélectionnez **OK** pour fermer **Pages de propriétés**.
1. Sélectionnez F5 pour exécuter les deux projets.
1. Les deux fenêtres de console s’ouvrent et vous invitent à renseigner le nom de l’espace de noms. Le service doit d’abord s’exécuter. Dans la fenêtre de console **EchoService**, entrez l’espace de noms, puis sélectionnez Entrée.
1. Ensuite, la console vous invite à entrer votre clé SAS. Entrez la clé SAS et sélectionnez Entrée.

    Voici un exemple de sortie de la fenêtre de console. Les valeurs ici sont simplement des exemples.

    `Your Service Namespace: myNamespace`

    `Your SAS Key: <SAS key value>`

    L’application de service imprime l’adresse sur laquelle il écoute dans la fenêtre de console, comme indiqué dans l’exemple suivant.

    `Service address: sb://mynamespace.servicebus.windows.net/EchoService/`

    `Press [Enter] to exit`

1. Dans la fenêtre de console **EchoClient**, entrez les mêmes informations que celles que vous avez entrées précédemment pour l’application de service. Saisissez des valeurs identiques pour l’espace de noms de service et la clé SAP pour l’application cliente.
1. Après avoir saisi ces valeurs, le client ouvre un canal au service et vous invite à saisir du texte, comme indiqué dans l’exemple de sortie de console suivante.

    `Enter text to echo (or [Enter] to exit):`

    Saisissez du texte à envoyer à l’application de service, puis sélectionnez Entrée. Ce texte est envoyé au service via l’opération de service Echo et apparaît dans la fenêtre de console de service comme dans l’exemple de sortie suivant.

    `Echoing: My sample text`

    L’application cliente reçoit la valeur de retour de l’opération `Echo`, qui est le texte d’origine, et l’imprime sur la fenêtre de console. Le texte suivant est un exemple de sortie de la fenêtre de console du client.

    `Server echoed: My sample text`

1. Vous pouvez continuer d’envoyer des messages texte du client au service de cette manière. Une fois que vous avez terminé, sélectionnez Entrée dans les fenêtres du client et de la console de service pour arrêter les des deux applications.

## <a name="next-steps"></a>Étapes suivantes

Accéder au didacticiel suivant :

> [!div class="nextstepaction"]
>[Exposer un service REST WCF local sur un client WCF à l’extérieur de votre réseau](service-bus-relay-rest-tutorial.md)

[2]: ./media/service-bus-relay-tutorial/configure-echoservice-console-app.png
[3]: ./media/service-bus-relay-tutorial/install-nuget-service-bus.png
[4]: ./media/service-bus-relay-tutorial/install-nuget-service-bus-client.png
[5]: ./media/service-bus-relay-tutorial/set-projects.png
[6]: ./media/service-bus-relay-tutorial/set-depend.png

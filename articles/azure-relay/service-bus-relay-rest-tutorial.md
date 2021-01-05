---
title: 'Tutoriel : Didacticiel REST utilisant Azure Relay'
description: 'Tutoriel : Créez une application hôte Azure Relay qui expose une interface de type REST.'
ms.topic: tutorial
ms.custom: devx-track-csharp
ms.date: 06/23/2020
ms.openlocfilehash: 0620f55650d0e4da0cd7a616649df952f3017455
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88922325"
---
# <a name="tutorial-azure-wcf-relay-rest-tutorial"></a>Tutoriel : Didacticiel Azure WCF Relay REST

Ce didacticiel explique comment créer une application hôte Azure Relay qui expose une interface de type REST. REST permet à un client web, par exemple un navigateur web, d’accéder aux API Service Bus via des requêtes HTTP.

Ce didacticiel utilise le modèle de programmation REST WCF (Windows Communication Foundation) pour construire un service REST dans Azure Relay. Pour plus d’informations, consultez [Modèle de programmation REST WCF](/dotnet/framework/wcf/feature-details/wcf-web-http-programming-model) et [Conception et implémentation de services](/dotnet/framework/wcf/designing-and-implementing-services).

Dans ce didacticiel, vous allez effectuer les tâches suivantes :

> [!div class="checklist"]
>
> * Installer la configuration requise pour ce didacticiel.
> * Créer un espace de noms Relay
> * Définir un contrat de service WCF basé sur REST
> * Implémenter le contrat WCF basé sur REST
> * Héberger et exécuter le service WCF basé sur REST
> * Exécuter et tester le pipeline

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce didacticiel, vous avez besoin de ce qui suit :

* Un abonnement Azure. Si vous n’en avez pas, [créez un compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* [Visual Studio 2015 ou version ultérieure](https://www.visualstudio.com). Les exemples de ce didacticiel utilisent Visual Studio 2019.
* SDK Azure pour .NET. Installez-le à partir de la [page de téléchargement des SDK](https://azure.microsoft.com/downloads/).

## <a name="create-a-relay-namespace"></a>Créer un espace de noms Relay

Pour commencer à utiliser les fonctionnalités de relais dans Azure, vous devez d’abord créer un espace de noms de service. Un espace de noms fournit un conteneur d’étendue pour l’adressage des ressources Azure au sein de votre application. Suivez les [instructions fournies ici](relay-create-namespace-portal.md) pour créer un espace de noms Relay.

## <a name="define-a-rest-based-wcf-service-contract-to-use-with-azure-relay"></a>Définir un contrat de service WCF basé sur REST à utiliser avec Azure Relay

Lorsque vous créez un service de type REST WCF, vous devez définir le contrat. Le contrat spécifie les opérations prises en charge par l'hôte. Une opération de service ressemble à une méthode de service web. Définissez un contrat avec interface C++, C# ou Visual Basic. Chaque méthode dans l'interface correspond à une opération de service spécifique. Appliquez l’attribut [ServiceContractAttribute](/dotnet/api/system.servicemodel.servicecontractattribute) à chaque interface et l’attribut [OperationContractAttribute](/dotnet/api/system.servicemodel.operationcontractattribute) à chaque opération. 

> [!TIP]
> Si une méthode d'interface dotée de l’attribut [ServiceContractAttribute](/dotnet/api/system.servicemodel.servicecontractattribute) ne contient pas l’attribut [OperationContractAttribute](/dotnet/api/system.servicemodel.operationcontractattribute), cette méthode n’est pas exposée. Le code utilisé pour effectuer ces tâches s'affiche dans l'exemple suivant la procédure.

La principale différence entre un contrat WCF et un contrat de type REST est l’ajout d’une propriété à l’attribut [OperationContractAttribute](/dotnet/api/system.servicemodel.operationcontractattribute) : [WebGetAttribute](/dotnet/api/system.servicemodel.web.webgetattribute). Cette propriété vous permet de mapper une méthode dans votre interface à une méthode de l'autre côté de l'interface. Cet exemple utilise l’attribut [WebGetAttribute](/dotnet/api/system.servicemodel.web.webgetattribute) pour lier une méthode `HTTP GET`. Cette approche permet à Service Bus de récupérer et d’interpréter correctement les commandes envoyées à l’interface.

### <a name="to-create-a-contract-with-an-interface"></a>Création d’un contrat avec une interface

1. Lancez Microsoft Visual Studio en tant qu'administrateur. Pour ce faire, cliquez avec le bouton droit sur l’icône de programme Visual Studio, puis sélectionnez **Exécuter en tant qu’administrateur**.
1. Dans Visual Studio, sélectionnez **Créer un nouveau projet**.
1. Dans **Créer un projet**, sélectionnez **Application console (.NET Framework)** pour C#, puis **Suivant**.
1. Nommez le projet *ImageListener*. Utilisez la valeur par défaut **Emplacement**, puis sélectionnez **Créer**.

   Pour un projet C#, Visual Studio crée un fichier *Program.cs*. Cette classe contient une méthode `Main()` vide, requise pour créer correctement un projet d’application console.

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet **ImageListener**, puis sélectionnez **Gérer les packages NuGet**.
1. Sélectionnez **Parcourir**, puis recherchez et sélectionnez **WindowsAzure.ServiceBus**. Sélectionnez **Installer** et acceptez les conditions d’utilisation.

    Cette étape ajoute des références à Service Bus et *System.ServiceModel.dll*. Ce package ajoute automatiquement des références aux bibliothèques Service Bus et au `System.ServiceModel` de WCF.

1. Ajoutez explicitement une référence à `System.ServiceModel.Web.dll` vers le projet. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur **Références** sous le dossier de projet, puis sélectionnez **Ajouter une référence**.
1. Dans la référence **Ajouter une référence**, sélectionnez **Infrastructure** et entrez *System.ServiceModel.Web* dans **Rechercher**. Cochez la case **System.ServiceModel.Web**, puis cliquez sur **OK**.

Apportez ensuite les modifications de code suivantes au projet :

1. Ajoutez les instructions `using` suivantes en haut du fichier *Program.cs*.

    ```csharp
    using System.ServiceModel;
    using System.ServiceModel.Channels;
    using System.ServiceModel.Web;
    using System.IO;
    ```

    * [System.ServiceModel](/dotnet/api/system.servicemodel) est l’espace de noms qui permet l’accès par programme aux fonctionnalités WCF de base. WCF Relay utilise la plupart des objets et attributs de WCF pour définir des contrats de service. Vous allez utiliser cet espace de noms dans la plupart de vos applications de relais.
    * [System.ServiceModel.Channels](/dotnet/api/system.servicemodel.channels) permet de définir le canal, qui est l’objet via lequel vous communiquez avec Azure Relay et le navigateur web client.
    * [System.ServiceModel.Web](/dotnet/api/system.servicemodel.web) contient les types qui vous permettent de créer des applications web.

1. Renommez l’espace de noms `ImageListener` sur `Microsoft.ServiceBus.Samples`.

    ```csharp
    namespace Microsoft.ServiceBus.Samples
    {
        ...
    ```

1. Directement après l’accolade d’ouverture de la déclaration d’espace de noms, définissez une nouvelle interface nommée `IImageContract` et appliquez l’attribut `ServiceContractAttribute` à l’interface avec une valeur de `https://samples.microsoft.com/ServiceModel/Relay/RESTTutorial1`. 

    ```csharp
    [ServiceContract(Name = "ImageContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/RESTTutorial1")]
    public interface IImageContract
    {
    }
    ```

    La valeur de l'espace de noms diffère de l'espace de noms que vous utilisez dans l’ensemble de votre code. La valeur de l'espace de noms correspond à un identificateur unique pour ce contrat et doit contenir des informations de version. Pour plus d’informations, consultez la rubrique [Contrôle de version des services](/dotnet/framework/wcf/service-versioning). Spécifier explicitement l'espace de noms empêche l'ajout au nom du contrat de la valeur d'espace de noms par défaut.

1. Dans l’interface `IImageContract`, déclarez une méthode pour une seule opération que le contrat `IImageContract` expose dans l’interface, puis appliquez l’attribut `OperationContract` à la méthode que vous souhaitez exposer dans le cadre du contrat Service Bus public.

    ```csharp
    public interface IImageContract
    {
        [OperationContract]
        Stream GetImage();
    }
    ```

1. Dans l’attribut `OperationContract`, ajoutez la valeur `WebGet`.

    ```csharp
    public interface IImageContract
    {
        [OperationContract, WebGet]
        Stream GetImage();
    }
    ```

   L'ajout de la valeur `WebGet` permet au service de relais d’acheminer les demandes HTTP GET vers `GetImage`, et de translater les valeurs `GetImage` renvoyées dans une réponse `HTTP GETRESPONSE`. Plus loin dans ce didacticiel, vous utiliserez un navigateur web pour accéder à cette méthode et afficher l'image dans le navigateur.

1. Directement après la définition de `IImageContract`, déclarez un canal qui hérite à la fois des interfaces `IImageContract` et `IClientChannel` :

    ```csharp
    public interface IImageChannel : IImageContract, IClientChannel { }
    ```

   Un canal est l'objet WCF par le biais duquel le service et le client se transmettent des informations. Ensuite, vous allez créer le canal dans votre application hôte. Azure Relay utilise ensuite ce canal pour transmettre les requêtes HTTP GET du navigateur vers votre implémentation `GetImage`. Le relais utilise également ce canal pour extraire la valeur `GetImage`renvoyée et la convertir en `HTTP GETRESPONSE` pour le navigateur client.

1. Sélectionnez **Générer** > **Générer la solution** pour confirmer que votre travail est correct jusqu’à présent.

### <a name="example-that-defines-a-wcf-relay-contract"></a>Exemple définissant un contrat WCF Relay

Le code suivant montre une interface de base qui définit un contrat WCF Relay.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.ServiceModel;
using System.ServiceModel.Channels;
using System.ServiceModel.Web;
using System.IO;

namespace Microsoft.ServiceBus.Samples
{

    [ServiceContract(Name = "IImageContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IImageContract
    {
        [OperationContract, WebGet]
        Stream GetImage();
    }

    public interface IImageChannel : IImageContract, IClientChannel { }

    class Program
    {
        static void Main(string[] args)
        {
        }
    }
}
```

## <a name="implement-the-rest-based-wcf-service-contract"></a>Implémenter le contrat de service WCF basé sur REST

Pour créer un service WCF Relay de type REST, commencez par créer le contrat à l’aide d’une interface. L'étape suivante consiste à implémenter l'interface. Cette procédure implique la création d’une classe nommée `ImageService` qui implémente l’interface `IImageContract` définie par l’utilisateur. Une fois le contrat implémenté, vous configurez ensuite l'interface à l'aide d'un fichier *App.config*. Le fichier de configuration contient les informations nécessaires à l’application. Ces informations comprennent le nom du service, le nom du contrat et le type de protocole utilisé pour communiquer avec le service de relais. Le code utilisé pour effectuer ces tâches s'affiche dans l'exemple suivant la procédure.

Comme pour les étapes précédentes, il existe très peu de différences entre l’implémentation d’un contrat de type REST et un contrat WCF Relay.

### <a name="to-implement-a-rest-style-service-bus-contract"></a>Implémentation d’un contrat Service Bus de type REST

1. Créez une classe nommée `ImageService` directement après la définition de l’interface `IImageContract`. La classe `ImageService` implémente l’interface `IImageContract`.

    ```csharp
    class ImageService : IImageContract
    {
    }
    ```

    Comme pour d'autres implémentations d'interface, vous pouvez implémenter la définition dans un autre fichier. Toutefois, pour ce didacticiel, l’implémentation apparaît dans le même fichier que la définition d’interface et la méthode `Main()`.

1. Appliquez l’attribut [ServiceBehaviorAttribute](/dotnet/api/system.servicemodel.servicebehaviorattribute) à la classe `IImageService` pour indiquer que la classe est une implémentation de contrat WCF.

    ```csharp
    [ServiceBehavior(Name = "ImageService", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    class ImageService : IImageContract
    {
    }
    ```

    Comme mentionné précédemment, cet espace de noms n'est pas un espace de noms standard. Il fait partie de l'architecture WCF qui identifie le contrat. Pour plus d'informations, consultez la page [Noms des contrats de données](/dotnet/framework/wcf/feature-details/data-contract-names/).

1. Ajoutez une image *.jpg* à votre projet. Ce fichier correspond à une image que le service affiche dans le navigateur de réception.

   1. Cliquez avec le bouton droit sur votre projet et sélectionnez **Ajouter**.
   1. Sélectionnez ensuite **Élément existant**.
   1. Utilisez **Ajouter un élément existant** pour accéder à un fichier .jpg approprié, puis sélectionnez **Ajouter**. Lorsque vous ajoutez le fichier, sélectionnez **Tous les fichiers** dans la liste déroulante en regard de **Nom du fichier**.

   Le reste de ce didacticiel suppose que le nom de l'image est *image.jpg*. Si votre fichier est différent, vous devez renommer l’image ou modifier votre code pour compenser.

1. Pour vous assurer que le service en cours d’exécution est capable de trouver le fichier image, cliquez avec le bouton droit sur le fichier image dans l'**Explorateur de solutions**, puis sélectionnez **Propriétés**. Dans **Propriétés**, définissez la valeur **Copier dans le répertoire de sortie** sur **Copier si plus récent**.

1. Utilisez la procédure décrite dans [Création d’un contrat avec une interface](#to-create-a-contract-with-an-interface) pour ajouter une référence à l’assembly *System.Drawing.dll* du projet.

1. Ajoutez les instructions connexes `using` suivantes :

    ```csharp
    using System.Drawing;
    using System.Drawing.Imaging;
    using Microsoft.ServiceBus;
    using Microsoft.ServiceBus.Web;
    ```

1. Dans la classe `ImageService`, ajoutez le constructeur suivant qui charge le bitmap et prépare son envoi vers le navigateur client :

    ```csharp
    class ImageService : IImageContract
    {
        const string imageFileName = "image.jpg";
   
        Image bitmap;
   
        public ImageService()
        {
            this.bitmap = Image.FromFile(imageFileName);
        }
    }
    ```

1. Directement après le code précédent, ajoutez la méthode `GetImage` suivante à la classe `ImageService` afin de retourner un message HTTP contenant l’image.

    ```csharp
    public Stream GetImage()
    {
        MemoryStream stream = new MemoryStream();
        this.bitmap.Save(stream, ImageFormat.Jpeg);
   
        stream.Position = 0;
        WebOperationContext.Current.OutgoingResponse.ContentType = "image/jpeg";
   
        return stream;
    }
    ```

    Cette implémentation utilise `MemoryStream` pour récupérer l’image et la préparer pour la diffusion en continu vers le navigateur. La diffusion en continu commence à la position zéro, déclare le contenu du flux au format *.jpg* et diffuse l'information.

1. Sélectionnez **Générer** > **Générer la solution**.

### <a name="to-define-the-configuration-for-running-the-web-service-on-service-bus"></a>Définition de la configuration pour l’exécution du service web sur Service Bus

1. Dans l'**Explorateur de solutions**, double-cliquez sur le fichier **App.config** pour l’ouvrir dans l’éditeur Visual Studio.

    Le fichier *App.config* comprend le nom du service, le point de terminaison et la liaison. Le point de terminaison correspond à l'emplacement qu’Azure Relay expose pour permettre aux clients et aux hôtes de communiquer. La liaison correspond au type de protocole utilisé pour communiquer. La principale différence réside dans le fait que le point de terminaison de service configuré fait référence à une liaison [WebHttpRelayBinding](/dotnet/api/microsoft.servicebus.webhttprelaybinding).

1. L’élément XML `<system.serviceModel>` est un élément WCF qui définit un ou plusieurs services. Ici, il sert à définir le nom du service et le point de terminaison. En bas de l’élément `<system.serviceModel>`, mais toujours au sein de `<system.serviceModel>`, ajoutez un élément `<bindings>` avec le contenu suivant :

    ```xml
    <bindings>
        <!-- Application Binding -->
        <webHttpRelayBinding>
            <binding name="default">
                <security relayClientAuthenticationType="None" />
            </binding>
        </webHttpRelayBinding>
    </bindings>
    ```

    Ce contenu définit les liaisons utilisées dans l'application. Vous pouvez définir plusieurs liaisons, mais pour ce didacticiel, vous n’en définissez qu'une seule.

    Le code précédent définit une liaison WCF Relay [WebHttpRelayBinding](/dotnet/api/microsoft.servicebus.webhttprelaybinding) avec `relayClientAuthenticationType`définir sur `None`. Ce paramètre indique qu'un point de terminaison utilisant cette liaison ne nécessite aucune information d'identification client.

1. Après l’élément `<bindings>`, ajoutez un élément `<services>`. Comme pour les liaisons, vous pouvez définir plusieurs services dans un même fichier de configuration. Toutefois, pour ce didacticiel, vous n’en définissez qu'un seul.

    ```xml
    <services>
        <!-- Application Service -->
        <service name="Microsoft.ServiceBus.Samples.ImageService"
             behaviorConfiguration="default">
            <endpoint name="RelayEndpoint"
                    contract="Microsoft.ServiceBus.Samples.IImageContract"
                    binding="webHttpRelayBinding"
                    bindingConfiguration="default"
                    behaviorConfiguration="sbTokenProvider"
                    address="" />
        </service>
    </services>
    ```

    Ce contenu configure un service qui utilise la valeur par défaut `webHttpRelayBinding`définie précédemment. Il utilise également la valeur par défaut `sbTokenProvider`, définie dans l’étape suivante.

1. Après l’élément `<services>`, créez un élément `<behaviors>` avec le contenu suivant, en remplaçant `SAS_KEY` par la clé avec signature d’accès partagé (SAP). Pour obtenir une clé SAP à partir du [portail Azure][Azure portal], consultez [Obtenir des informations d'identification d'administration](service-bus-relay-tutorial.md#get-management-credentials).

    ```xml
    <behaviors>
        <endpointBehaviors>
            <behavior name="sbTokenProvider">
                <transportClientEndpointBehavior>
                    <tokenProvider>
                        <sharedAccessSignature keyName="RootManageSharedAccessKey" key="YOUR_SAS_KEY" />
                    </tokenProvider>
                </transportClientEndpointBehavior>
            </behavior>
            </endpointBehaviors>
            <serviceBehaviors>
                <behavior name="default">
                    <serviceDebug httpHelpPageEnabled="false" httpsHelpPageEnabled="false" />
                </behavior>
            </serviceBehaviors>
    </behaviors>
    ```

1. Toujours dans le fichier *App.config*, dans l’élément `<appSettings>`, remplacez la valeur de la chaîne de connexion entière par la chaîne de connexion obtenue précédemment à partir du portail.

    ```xml
    <appSettings>
       <!-- Service Bus specific app settings for messaging connections -->
       <add key="Microsoft.ServiceBus.ConnectionString"
           value="Endpoint=sb://yourNamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=YOUR_SAS_KEY"/>
    </appSettings>
    ```

1. Select **Générer** > **Générer la solution** pour générer la solution complète.

### <a name="example-that-implements-the-rest-based-wcf-service-contract"></a>Exemple d'implémentation du contrat de service WCF basé sur REST

Le code suivant montre l’implémentation du contrat et du service pour un service REST qui s’exécute sur Service Bus à l’aide de la liaison `WebHttpRelayBinding`.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.ServiceModel;
using System.ServiceModel.Channels;
using System.ServiceModel.Web;
using System.IO;
using System.Drawing;
using System.Drawing.Imaging;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Web;

namespace Microsoft.ServiceBus.Samples
{


    [ServiceContract(Name = "ImageContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IImageContract
    {
        [OperationContract, WebGet]
        Stream GetImage();
    }

    public interface IImageChannel : IImageContract, IClientChannel { }

    [ServiceBehavior(Name = "ImageService", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    class ImageService : IImageContract
    {
        const string imageFileName = "image.jpg";

        Image bitmap;

        public ImageService()
        {
            this.bitmap = Image.FromFile(imageFileName);
        }

        public Stream GetImage()
        {
            MemoryStream stream = new MemoryStream();
            this.bitmap.Save(stream, ImageFormat.Jpeg);

            stream.Position = 0;
            WebOperationContext.Current.OutgoingResponse.ContentType = "image/jpeg";

            return stream;
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
        }
    }
}
```

L'exemple suivant montre le fichier *App.config* associé au service.

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <startup> 
        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5.2"/>
    </startup>
    <system.serviceModel>
        <extensions>
            <!-- In this extension section we are introducing all known service bus extensions. User can remove the ones they don't need. -->
            <behaviorExtensions>
                <add name="connectionStatusBehavior"
                    type="Microsoft.ServiceBus.Configuration.ConnectionStatusElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="transportClientEndpointBehavior"
                    type="Microsoft.ServiceBus.Configuration.TransportClientEndpointBehaviorElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="serviceRegistrySettings"
                    type="Microsoft.ServiceBus.Configuration.ServiceRegistrySettingsElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
            </behaviorExtensions>
            <bindingElementExtensions>
                <add name="netMessagingTransport"
                    type="Microsoft.ServiceBus.Messaging.Configuration.NetMessagingTransportExtensionElement, Microsoft.ServiceBus,  Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="tcpRelayTransport"
                    type="Microsoft.ServiceBus.Configuration.TcpRelayTransportElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="httpRelayTransport"
                    type="Microsoft.ServiceBus.Configuration.HttpRelayTransportElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="httpsRelayTransport"
                    type="Microsoft.ServiceBus.Configuration.HttpsRelayTransportElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="onewayRelayTransport"
                    type="Microsoft.ServiceBus.Configuration.RelayedOnewayTransportElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
            </bindingElementExtensions>
            <bindingExtensions>
                <add name="basicHttpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.BasicHttpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="webHttpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.WebHttpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="ws2007HttpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.WS2007HttpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="netTcpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.NetTcpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="netOnewayRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.NetOnewayRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="netEventRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.NetEventRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="netMessagingBinding"
                    type="Microsoft.ServiceBus.Messaging.Configuration.NetMessagingBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
            </bindingExtensions>
        </extensions>
      <bindings>
        <!-- Application Binding -->
        <webHttpRelayBinding>
          <binding name="default">
            <security relayClientAuthenticationType="None" />
          </binding>
        </webHttpRelayBinding>
      </bindings>
      <services>
        <!-- Application Service -->
        <service name="Microsoft.ServiceBus.Samples.ImageService"
             behaviorConfiguration="default">
          <endpoint name="RelayEndpoint"
                  contract="Microsoft.ServiceBus.Samples.IImageContract"
                  binding="webHttpRelayBinding"
                  bindingConfiguration="default"
                  behaviorConfiguration="sbTokenProvider"
                  address="" />
        </service>
      </services>
      <behaviors>
        <endpointBehaviors>
          <behavior name="sbTokenProvider">
            <transportClientEndpointBehavior>
              <tokenProvider>
                <sharedAccessSignature keyName="RootManageSharedAccessKey" key="YOUR_SAS_KEY" />
              </tokenProvider>
            </transportClientEndpointBehavior>
          </behavior>
        </endpointBehaviors>
        <serviceBehaviors>
          <behavior name="default">
            <serviceDebug httpHelpPageEnabled="false" httpsHelpPageEnabled="false" />
          </behavior>
        </serviceBehaviors>
      </behaviors>
    </system.serviceModel>
    <appSettings>
        <!-- Service Bus specific app settings for messaging connections -->
        <add key="Microsoft.ServiceBus.ConnectionString"
            value="Endpoint=sb://yourNamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=YOUR_SAS_KEY>"/>
    </appSettings>
</configuration>
```

## <a name="host-the-rest-based-wcf-service-to-use-azure-relay"></a>Héberger le service WCF basé sur REST pour utiliser Azure Relay

Cette section décrit comment exécuter un service web à l’aide d’une application console avec WCF Relay. Une liste complète du code écrit dans cette section s'affiche dans l'exemple suivant la procédure.

### <a name="to-create-a-base-address-for-the-service"></a>Création d’une adresse de base pour le service

1. Dans la déclaration de fonction `Main()`, créez une variable pour stocker l’espace de noms de votre projet. Assurez-vous de remplacer `yourNamespace` par le nom de l’espace de noms de service Relay créé précédemment.

    ```csharp
    string serviceNamespace = "yourNamespace";
    ```

    Service Bus utilise le nom de votre espace de noms pour créer une adresse URI unique.

1. Créez une instance `Uri` pour l’adresse de base du service basé sur l’espace de noms.

    ```csharp
    Uri address = ServiceBusEnvironment.CreateServiceUri("https", serviceNamespace, "Image");
    ```

### <a name="to-create-and-configure-the-web-service-host"></a>Création et configuration de l'hôte de service web

Toujours dans `Main()`, créez l'hôte de service web en utilisant l'adresse URI créée précédemment dans cette section.
  
```csharp
WebServiceHost host = new WebServiceHost(typeof(ImageService), address);
```

L'hôte de service est l'objet WCF qui instancie l'application hôte. Cet exemple transmet le type d’hôte que vous souhaitez créer, un élément `ImageService`, ainsi que l’adresse à laquelle vous souhaitez exposer l’application hôte.

### <a name="to-run-the-web-service-host"></a>Exécution de l'hôte de service web

1. Toujours dans `Main()`, ajoutez la ligne suivante pour ouvrir le service.

    ```csharp
    host.Open();
    ```

    Le service est maintenant actif.

1. Affichez un message indiquant que le service est en cours d'exécution et expliquant comment arrêter ce service.

    ```csharp
    Console.WriteLine("Copy the following address into a browser to see the image: ");
    Console.WriteLine(address + "GetImage");
    Console.WriteLine();
    Console.WriteLine("Press [Enter] to exit");
    Console.ReadLine();
    ```

1. Lorsque vous avez terminé, fermez l'hôte de service.

    ```csharp
    host.Close();
    ```

### <a name="example-of-the-service-contract-and-implementation"></a>Exemple de contrat de service et d'implémentation

L'exemple suivant inclut le contrat de service et l'implémentation des étapes précédentes du didacticiel et héberge le service dans une application console. Compilez le code suivant dans un fichier exécutable nommé *ImageListener.exe*.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.ServiceModel;
using System.ServiceModel.Channels;
using System.ServiceModel.Web;
using System.IO;
using System.Drawing;
using System.Drawing.Imaging;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Web;

namespace Microsoft.ServiceBus.Samples
{

    [ServiceContract(Name = "ImageContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IImageContract
    {
        [OperationContract, WebGet]
        Stream GetImage();
    }

    public interface IImageChannel : IImageContract, IClientChannel { }

    [ServiceBehavior(Name = "ImageService", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    class ImageService : IImageContract
    {
        const string imageFileName = "image.jpg";

        Image bitmap;

        public ImageService()
        {
            this.bitmap = Image.FromFile(imageFileName);
        }

        public Stream GetImage()
        {
            MemoryStream stream = new MemoryStream();
            this.bitmap.Save(stream, ImageFormat.Jpeg);

            stream.Position = 0;
            WebOperationContext.Current.OutgoingResponse.ContentType = "image/jpeg";

            return stream;
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            string serviceNamespace = "InsertServiceNamespaceHere";
            Uri address = ServiceBusEnvironment.CreateServiceUri("https", serviceNamespace, "Image");

            WebServiceHost host = new WebServiceHost(typeof(ImageService), address);
            host.Open();

            Console.WriteLine("Copy the following address into a browser to see the image: ");
            Console.WriteLine(address + "GetImage");
            Console.WriteLine();
            Console.WriteLine("Press [Enter] to exit");
            Console.ReadLine();

            host.Close();
        }
    }
}
```

## <a name="run-and-test-the-service"></a>Exécuter et tester le pipeline

Après avoir créé la solution, procédez comme suit pour exécuter l'application :

1. Sélectionnez F5 ou accédez à l’emplacement du fichier exécutable *ImageListener\bin\Debug\ImageListener.exe* pour exécuter le service. Ne fermez pas l’application en cours d’exécution car elle est requise pour l'étape suivante.
1. Copiez et collez l'adresse à partir de l'invite de commande dans un navigateur pour afficher l'image.
1. Lorsque vous avez terminé, sélectionnez Entrée dans la fenêtre d’invite de commande pour fermer l’application.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez créé une application qui utilise le service Azure Relay, consultez les articles suivants pour en savoir plus :

* [Qu’est-ce qu’Azure Relay ?](relay-what-is-it.md)
* [Exposer un service WCF REST local sur un client externe en utilisant Azure WCF Relay](service-bus-relay-tutorial.md)

[Azure portal]: https://portal.azure.com

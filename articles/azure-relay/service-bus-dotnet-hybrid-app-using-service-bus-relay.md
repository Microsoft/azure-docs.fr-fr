---
title: Application hybride Azure Windows Communication Foundation (WCF) Relay locale/cloud (.NET) | Microsoft Docs
description: Découvrir comment exposer un service WCF local sur une application web dans le cloud à l’aide d’Azure Relay
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 06/23/2020
ms.openlocfilehash: 22b582634b623b39545eca225c8df0130606c2bb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90983892"
---
# <a name="expose-an-on-premises-wcf-service-to-a-web-application-in-the-cloud-by-using-azure-relay"></a>Exposer un service WCF local sur une application web dans le cloud à l’aide d’Azure Relay

Cet article montre comment créer une application cloud hybride avec Microsoft Azure et Visual Studio. Vous créez une application qui utilise plusieurs ressources Azure dans le cloud. Dans ce tutoriel, vous allez voir comment :

* créer ou adapter un service Web existant qui sera utilisé par une solution Web ;
* Utiliser le service Azure Windows Communication Foundation (WCF) Relay pour partager des données entre une application Azure et un service web hébergé ailleurs.

Dans ce didacticiel, vous allez effectuer les tâches suivantes :

> [!div class="checklist"]
>
> * Installer la configuration requise pour ce didacticiel.
> * Passez en revue le scénario.
> * Créez un espace de noms.
> * Créer un serveur local.
> * Créer une application ASP .NET.
> * Exécutez l’application localement.
> * Déployer l’application web sur Azure.
> * Exécuter l’application sur Azure.

## <a name="prerequisites"></a>Conditions préalables requises

Pour effectuer ce didacticiel, vous avez besoin de ce qui suit :

* Un abonnement Azure. Si vous n’en avez pas, [créez un compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* [Visual Studio 2015 ou version ultérieure](https://www.visualstudio.com). Les exemples de ce didacticiel utilisent Visual Studio 2019.
* SDK Azure pour .NET. Installez-le à partir de la [page de téléchargement des SDK](https://azure.microsoft.com/downloads/).

## <a name="how-azure-relay-helps-with-hybrid-solutions"></a>Avantages de l’utilisation d’Azure Relay pour les solutions hybrides

Les solutions d’entreprise sont généralement composées d’une combinaison de code personnalisé et de fonctionnalités existantes. Le code personnalisé permet de gérer les exigences d’entreprise nouvelles et uniques. Les solutions et systèmes déjà en place fournissent des fonctionnalités existantes.

Les architectes de solutions commencent à utiliser le cloud, car celui-ci permet de gérer plus facilement les exigences de mise à l'échelle tout en offrant des coûts opérationnels faibles. Ainsi, ils découvrent que les ressources des services existants qu’ils souhaiteraient utiliser comme des blocs de construction pour leurs solutions se situent à l’intérieur du pare-feu d’entreprise et hors de portée pour une solution cloud. Bon nombre de services internes ne sont pas créés ni hébergés pour être facilement exposés dans le périmètre du réseau d’entreprise.

[Azure Relay](https://azure.microsoft.com/services/service-bus/) rend accessibles les services web WCF existants de manière sécurisée aux solutions qui résident à l’extérieur du périmètre de l’entreprise, sans exiger de modifications intrusives de l’infrastructure de réseau d’entreprise. Ces services de relais sont toujours hébergés dans leur environnement existant, mais ils délèguent l’écoute des sessions et demandes entrantes au service de relais hébergé sur le cloud. Azure Relay protège également ces services de tout accès non autorisé à l’aide de l’authentification par [signature d’accès partagé](../service-bus-messaging/service-bus-sas.md) (SAP).

## <a name="review-the-scenario"></a>Passer en revue le scénario

Dans ce tutoriel, vous créez un site web ASP.NET qui vous permet de voir une liste de produits dans la page d’inventaire des produits.

![Scénario][0]

Ce didacticiel part du principe que vous disposez des informations sur les produits dans un système local existant, et utilise Azure Relay pour atteindre ce système. Un service web qui s’exécute dans une application console simple simule cette situation. Il contient un ensemble de produits en mémoire. Vous pouvez exécuter cette application console sur votre propre ordinateur et déployer le rôle Web dans Azure. Vous verrez ainsi comment le rôle Web exécuté dans le centre de données Azure appelle votre ordinateur. Cet appel se produit même si votre ordinateur se trouve très probablement derrière au moins un pare-feu et une couche de Traduction d’adresses réseau (NAT).

## <a name="set-up-the-development-environment"></a>Configuration de l’environnement de développement

Avant de commencer à développer votre application Azure, téléchargez les outils et configurez votre environnement de développement :

1. Installez le Kit de développement logiciel (SDK) Azure pour .NET depuis la page des [téléchargements SDK](https://azure.microsoft.com/downloads/).
1. Dans la colonne **.NET**, choisissez la version correspondant à votre version de [Visual Studio](https://www.visualstudio.com). Ce didacticiel utilise Visual Studio 2019.
1. Lorsque vous êtes invité à exécuter ou à enregistrer le programme d’installation, sélectionnez **Exécuter**.
1. Dans la boîte de dialogue **Web Platform Installer**, sélectionnez **Installer**, puis poursuivez l’installation.

Une fois l’installation terminée, vous disposez de tous les éléments nécessaires pour commencer le développement de l’application. Le Kit de développement logiciel (SDK) comprend des outils qui vous permettent de facilement développer des applications Azure dans Visual Studio.

## <a name="create-a-namespace"></a>Créer un espace de noms

La première étape consiste à créer un espace de noms et à obtenir une clé de [signature d’accès partagé (SAP)](../service-bus-messaging/service-bus-sas.md). Un espace de noms fournit une limite d’application pour chaque application exposée via le service de relais. Une clé SAS est automatiquement générée par le système lors de la création d’un espace de noms de service. La combinaison de l’espace de noms de service et de la clé SAP fournit à Azure des informations d’identification permettant d’authentifier l’accès à une application.

[!INCLUDE [relay-create-namespace-portal](../../includes/relay-create-namespace-portal.md)]

## <a name="create-an-on-premises-server"></a>Création d’un serveur local

Pour commencer, vous créez un système local de catalogue de produits (simulation).  Ce projet est une application console Visual Studio et utilise le [package NuGet Service Bus](https://www.nuget.org/packages/WindowsAzure.ServiceBus/) pour inclure les bibliothèques et les paramètres de configuration Service Bus. <a name="create-the-project"></a>

1. Lancez Microsoft Visual Studio en tant qu'administrateur. Pour ce faire, cliquez avec le bouton droit sur l’icône de programme Visual Studio, puis sélectionnez **Exécuter en tant qu’administrateur**.
1. Dans Visual Studio, sélectionnez **Créer un nouveau projet**.
1. Dans **Créer un projet**, sélectionnez **Application console (.NET Framework)** pour C#, puis **Suivant**.
1. Nommez le projet *ProductsServer* et sélectionnez **Créer**.

   ![Configurer votre nouveau projet][11]

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet **ProductsServer**, puis sélectionnez **Gérer les packages NuGet**.
1. Sélectionnez **Parcourir**, puis recherchez et sélectionnez **WindowsAzure.ServiceBus**. Sélectionnez **Installer** et acceptez les conditions d’utilisation.

   ![Sélectionner un package NuGet][13]

   Les assemblys client nécessaires sont maintenant référencés.

1. Ajoutez une nouvelle classe pour votre contrat de produit. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet **ProductsServer**, puis sélectionnez **Ajouter** > **une classe**.
1. Dans **Nom**, entrez le nom *ProductsContract.cs* et sélectionnez **Ajouter**.

Apportez les modifications de code suivantes à votre solution :

1. Dans *ProductsContract.cs*, remplacez la définition d’espace de noms existante par le code suivant, qui définit le contrat du service.

    ```csharp
    namespace ProductsServer
    {
        using System.Collections.Generic;
        using System.Runtime.Serialization;
        using System.ServiceModel;

        // Define the data contract for the service
        [DataContract]
        // Declare the serializable properties.
        public class ProductData
        {
            [DataMember]
            public string Id { get; set; }
            [DataMember]
            public string Name { get; set; }
            [DataMember]
            public string Quantity { get; set; }
        }

        // Define the service contract.
        [ServiceContract]
        interface IProducts
        {
            [OperationContract]
            IList<ProductData> GetProducts();

        }

        interface IProductsChannel : IProducts, IClientChannel
        {
        }
    }
    ```

1. Dans *Program.cs*, remplacez la définition d’espace de noms par le code suivant, qui ajoute le service de profil et l’hôte correspondant.

    ```csharp
    namespace ProductsServer
    {
        using System;
        using System.Linq;
        using System.Collections.Generic;
        using System.ServiceModel;

        // Implement the IProducts interface.
        class ProductsService : IProducts
        {

            // Populate array of products for display on website
            ProductData[] products =
                new []
                    {
                        new ProductData{ Id = "1", Name = "Rock",
                                         Quantity = "1"},
                        new ProductData{ Id = "2", Name = "Paper",
                                         Quantity = "3"},
                        new ProductData{ Id = "3", Name = "Scissors",
                                         Quantity = "5"},
                        new ProductData{ Id = "4", Name = "Well",
                                         Quantity = "2500"},
                    };

            // Display a message in the service console application
            // when the list of products is retrieved.
            public IList<ProductData> GetProducts()
            {
                Console.WriteLine("GetProducts called.");
                return products;
            }

        }

        class Program
        {
            // Define the Main() function in the service application.
            static void Main(string[] args)
            {
                var sh = new ServiceHost(typeof(ProductsService));
                sh.Open();

                Console.WriteLine("Press ENTER to close");
                Console.ReadLine();

                sh.Close();
            }
        }
    }
    ```

1. Dans l'**Explorateur de solutions**, double-cliquez sur le fichier **App.config** pour l’ouvrir dans l’éditeur Visual Studio. En bas de l’élément `<system.ServiceModel>` (mais toujours au sein de `<system.ServiceModel>`), ajoutez le code XML suivant. Veillez à remplacer `yourServiceNamespace` par le nom de votre espace de noms, et `yourKey` par la clé SAS que vous avez récupérée précédemment dans le portail :

    ```xml
    <system.serviceModel>
    ...
      <services>
         <service name="ProductsServer.ProductsService">
           <endpoint address="sb://yourServiceNamespace.servicebus.windows.net/products" binding="netTcpRelayBinding" contract="ProductsServer.IProducts" behaviorConfiguration="products"/>
         </service>
      </services>
      <behaviors>
         <endpointBehaviors>
           <behavior name="products">
             <transportClientEndpointBehavior>
                <tokenProvider>
                   <sharedAccessSignature keyName="RootManageSharedAccessKey" key="yourKey" />
                </tokenProvider>
             </transportClientEndpointBehavior>
           </behavior>
         </endpointBehaviors>
      </behaviors>
    </system.serviceModel>
    ```

    > [!NOTE]
    > L’erreur provoquée par `transportClientEndpointBehavior` est simplement un avertissement qui ne bloque pas le déroulement de cet exemple.

1. Toujours dans *App.config*, dans l’élément `<appSettings>`, remplacez la valeur de la chaîne de connexion par la chaîne de connexion obtenue précédemment dans le portail.

    ```xml
    <appSettings>
       <!-- Service Bus specific app settings for messaging connections -->
       <add key="Microsoft.ServiceBus.ConnectionString"
           value="Endpoint=sb://yourNamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=yourKey"/>
    </appSettings>
    ```

1. Appuyez sur Ctrl+Maj+B ou sélectionnez **Générer** > **Générer la solution** pour générer l’application et vérifier que votre travail est correct.

## <a name="create-an-aspnet-application"></a>Création d’une application ASP.NET

Dans cette section, vous générez une application ASP.NET simple qui affiche des données récupérées de votre service de produit.

### <a name="create-the-project"></a>Créer le projet

1. Vérifiez que Visual Studio est exécuté en tant qu’administrateur.
1. Dans Visual Studio, sélectionnez **Créer un nouveau projet**.
1. Dans **Créer un projet**, sélectionnez **Application web ASP.NET (.NET Framework)** pour C#, puis **Suivant**.
1. Nommez le projet *ProductsPortal* et sélectionnez **Créer**.
1. Dans **Créer une nouvelle application web ASP.NET**, sélectionnez **MVC**, puis **Modifier** sous **Authentification**.

   ![Sélectionner l’application web ASP .NET][16]

1. Dans **Modifier l’authentification**, sélectionnez **Aucune authentification**, puis **OK**. Pour ce didacticiel, vous déployez une application qui n’a pas besoin de connexion de l’utilisateur.

    ![Spécifier l’authentification][18]

1. De retour dans **Créer une application web ASP.NET**, sélectionnez **Créer** pour créer l’application MVC.
1. Configurez les ressources Azure d’une nouvelle application web. Suivez les étapes décrites dans [Publier votre application web](../app-service/quickstart-dotnet-framework.md#launch-the-publish-wizard). Ensuite, revenez à ce didacticiel et passez à l’étape suivante.
1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur **Modèles** et sélectionnez **Ajouter** > **une classe**.
1. Nommez la classe *Product.cs*, puis sélectionnez **Ajouter**.

    ![Créer le modèle de produit][17]

### <a name="modify-the-web-application"></a>Modification de l’application Web

1. Dans le fichier *Product.cs* dans Visual Studio, remplacez la définition d’espace de noms existante par le code suivant :

   ```csharp
    // Declare properties for the products inventory.
    namespace ProductsWeb.Models
    {
       public class Product
       {
           public string Id { get; set; }
           public string Name { get; set; }
           public string Quantity { get; set; }
       }
    }
    ```

1. Dans l’**Explorateur de solutions**, développez **Controllers** et double-cliquez sur **HomeController.cs** pour ouvrir le fichier dans Visual Studio.
1. Dans *HomeController.cs*, remplacez la définition d’espace de noms existante par le code suivant :

    ```csharp
    namespace ProductsWeb.Controllers
    {
        using System.Collections.Generic;
        using System.Web.Mvc;
        using Models;

        public class HomeController : Controller
        {
            // Return a view of the products inventory.
            public ActionResult Index(string Identifier, string ProductName)
            {
                var products = new List<Product>
                    {new Product {Id = Identifier, Name = ProductName}};
                return View(products);
            }
         }
    }
    ```

1. Dans l’**Explorateur de solutions**, développez **Views** > **Shared**, puis double-cliquez sur **_Layout.cshtml** pour ouvrir le fichier dans l’éditeur de Visual Studio.
1. Remplacez toutes les occurrences de `My ASP.NET Application` par *Northwind Traders Products*.
1. Supprimez les liens `Home`, `About` et `Contact`. Dans l’exemple suivant, supprimez le code en surbrillance.

    ![Supprimer les éléments de liste générés][41]

1. Dans l’**Explorateur de solutions**, développez **Views** > **Home**, puis double-cliquez sur **Index.cshtml** pour ouvrir le fichier dans l’éditeur de Visual Studio. Remplacez tout le contenu du fichier par le code suivant :

   ```html
   @model IEnumerable<ProductsWeb.Models.Product>

   @{
            ViewBag.Title = "Index";
   }

   <h2>Prod Inventory</h2>

   <table>
             <tr>
                 <th>
                     @Html.DisplayNameFor(model => model.Name)
                 </th>
                 <th></th>
                 <th>
                     @Html.DisplayNameFor(model => model.Quantity)
                 </th>
             </tr>

   @foreach (var item in Model) {
             <tr>
                 <td>
                     @Html.DisplayFor(modelItem => item.Name)
                 </td>
                 <td>
                     @Html.DisplayFor(modelItem => item.Quantity)
                 </td>
             </tr>
   }

   </table>
   ```

1. Pour vérifier la qualité de votre travail, appuyez sur Ctrl+Maj+B pour générer le projet.

### <a name="run-the-app-locally"></a>Exécutez l’application localement.

Exécutez l'application afin de vérifier qu'elle fonctionne.

1. Assurez-vous que **ProductsPortal** est le projet actif. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le nom du projet, puis sélectionnez **Définir comme projet de démarrage**.
1. Dans Visual Studio, appuyez sur F5.

Votre application doit s’exécuter dans un navigateur :

![Capture d’écran montrant un exemple de l’application s’exécutant dans un navigateur avec l’URL en surbrillance.][21]

## <a name="put-the-pieces-together"></a>Assemblage des éléments

La prochaine étape consiste à raccorder le serveur de produits local et l’application ASP.NET.

1. S’il n’est pas déjà ouvert, ouvrez dans Visual Studio le projet **ProductsPortal** que vous avez créé dans la section [Création d’une application ASP.NET](#create-an-aspnet-application).
1. Comme vous l’avez fait dans la section [Création d’un serveur local](#create-an-on-premises-server), ajoutez le package NuGet aux références du projet. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet **ProductsPortal**, puis sélectionnez **Gérer les packages NuGet**.
1. Recherchez *WindowsAzure.ServiceBus* et sélectionnez l’élément **WindowsAzure.ServiceBus**. Ensuite, terminez l’installation et fermez cette boîte de dialogue.
1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet **ProductsPortal**, puis sélectionnez **Ajouter** > **un élément existant**.
1. Accédez au fichier *ProductsContract.cs* depuis le projet de console **ProductsServer**. Mettez en surbrillance *ProductsContract.cs*. Cliquez sur la flèche vers le bas en regard de **Ajouter**, puis sélectionnez **Ajouter en tant que lien**.

   ![Ajouter en tant que lien][24]

1. Ouvrez à présent le fichier *HomeController.cs* dans l’éditeur Visual Studio, puis remplacez la définition d’espace de noms existante par le code suivant. Assurez-vous de remplacer `yourServiceNamespace` par le nom de votre espace de noms de service et `yourKey` par votre clé SAS. Ce code permet au client d’appeler le service local, en renvoyant le résultat de l’appel.

   ```csharp
   namespace ProductsWeb.Controllers
   {
       using System.Linq;
       using System.ServiceModel;
       using System.Web.Mvc;
       using Microsoft.ServiceBus;
       using Models;
       using ProductsServer;

       public class HomeController : Controller
       {
           // Declare the channel factory.
           static ChannelFactory<IProductsChannel> channelFactory;

           static HomeController()
           {
               // Create shared access signature token credentials for authentication.
               channelFactory = new ChannelFactory<IProductsChannel>(new NetTcpRelayBinding(),
                   "sb://yourServiceNamespace.servicebus.windows.net/products");
               channelFactory.Endpoint.Behaviors.Add(new TransportClientEndpointBehavior {
                   TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider(
                       "RootManageSharedAccessKey", "yourKey") });
           }

           public ActionResult Index()
           {
               using (IProductsChannel channel = channelFactory.CreateChannel())
               {
                   // Return a view of the products inventory.
                   return this.View(from prod in channel.GetProducts()
                                    select
                                        new Product { Id = prod.Id, Name = prod.Name,
                                            Quantity = prod.Quantity });
               }
           }
       }
   }
   ```

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur la solution **ProductsPortal**. Veillez à cliquer sur la solution et non sur le projet. Sélectionnez **Ajouter** > **un projet existant**.
1. Accédez au projet **ProductsServer**, puis double-cliquez sur le fichier solution *ProductsServer.csproj* pour l’ajouter.
1. **ProductsServer** doit être en cours d’exécution pour afficher les données sur **ProductsPortal**. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur la solution **ProductsPortal**, puis sélectionnez **Propriétés** pour afficher les **Pages de propriétés**.
1. Sélectionnez **Propriétés communes** > **Projet de démarrage**, puis sélectionnez **Plusieurs projets de démarrage**. Vérifiez que **ProductsServer** et **ProductsPortal** apparaissent dans cet ordre et que **Action** est défini sur **Démarrer** pour les deux.

      ![Plusieurs projets de démarrage][25]

1. Sélectionnez **Propriétés courantes** > **Dépendances du projet** dans le volet gauche.
1. Pour **Projets**, sélectionnez **ProductsPortal**. Vérifiez que **ProductsServer** est sélectionné.

    ![Dépendances du projet][26]

1. Pour **Projets**, sélectionnez **ProductsServer**. Vérifiez que **ProductsPortal** n’est pas sélectionné, puis cliquez sur **OK** pour enregistrer vos modifications.

## <a name="run-the-project-locally"></a>Exécutez le projet localement.

Pour tester l’application localement dans Visual Studio, appuyez sur F5. Le serveur local **ProductsServer** doit démarrer, puis l’application **ProductsPortal** doit se lancer dans une fenêtre de navigateur. À présent, vous voyez que l’inventaire de produits répertorie des données récupérées du système local de service de produit.

![Application web][10]

Sélectionnez **Actualiser** dans la page **ProductsPortal**. Chaque fois que vous actualisez la page, l’application de serveur affiche un message lors de l’appel de `GetProducts()` à partir de **ProductsServer**.

Fermez les deux applications avant de passer à la section suivante.

## <a name="deploy-the-productsportal-project-to-an-azure-web-app"></a>Déploiement du projet ProductsPortal dans une application web Azure

L’étape suivante consiste à publier une nouvelle fois l’application web Azure frontale **ProductsPortal** :

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet **ProductsPortal**, puis sélectionnez **Publier**. Dans la page de **publication**, sélectionnez **Publier**.

   > [!NOTE]
   > Un message d’erreur peut s’afficher dans la fenêtre du navigateur lorsque le projet web **ProductsPortal** est démarré automatiquement après le déploiement. Ce comportement est normal. Il s’explique par le fait que l’application **ProductsServer** n’est pas encore en cours d’exécution.
   >

1. Copiez l’URL de l’application web déployée. Vous aurez besoin de l’URL plus tard. Vous pouvez également obtenir cette URL dans la fenêtre **Activité d’Azure App Service** dans Visual Studio :

   ![URL de l’application déployée][9]

1. Fermez la fenêtre du navigateur pour arrêter l’application en cours d’exécution.

<a name="set-productsportal-as-web-app"></a>Avant d’exécuter l’application dans le cloud, vous devez vous assurer que **ProductsPortal** est démarré dans Visual Studio en tant qu’application web.

1. Dans Visual Studio, cliquez avec le bouton droit sur le projet **ProductsPortal**, puis sélectionnez **Propriétés**.
1. Sélectionnez **Web**. Sous **Action de démarrage**, sélectionnez **URL de démarrage**. Entrez l’URL de votre application web déployée précédemment, dans cet exemple, `https://productsportal20190906122808.azurewebsites.net/`.

    ![URL de démarrage][27]

1. Sélectionnez **Fichier** > **Enregistrer tout**.
1. Sélectionnez **Générer** > **Générer de nouveau la solution**.

## <a name="run-the-application"></a>Exécution de l'application

Appuyez sur F5 pour générer et exécuter l’application. Le serveur local, qui est l’application console **ProductsServer**, doit démarrer, puis l’application **ProductsPortal** doit se lancer dans une fenêtre de navigateur, comme illustré ci-dessous :

   ![Exécuter l’application web sur Azure][1]

L’inventaire de produits répertorie les données récupérées à partir du système local du service du produit et les affiche dans l’application web. Vérifiez l’URL pour vous assurer que l’application **ProductsPortal** est exécutée dans le cloud comme une application web Azure.

   > [!IMPORTANT]
   > L’application console **ProductsServer** doit être en cours d’exécution et en mesure de fournir les données à l’application **ProductsPortal**. Si le navigateur affiche une erreur, patientez quelques secondes, le temps que **ProductsServer** se charge et affiche le message suivant, puis actualisez le navigateur.
   >

Dans le navigateur, actualisez la page **ProductsPortal**. Chaque fois que vous actualisez la page, l’application de serveur affiche un message lors de l’appel de `GetProducts()` à partir de **ProductsServer**.

![Sortie mise à jour][38]

## <a name="next-steps"></a>Étapes suivantes

Accéder au didacticiel suivant :

> [!div class="nextstepaction"]
>[Exposer un service WCF sur site à un client WCF à l’extérieur de votre réseau](service-bus-relay-tutorial.md)

[0]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hybrid.png
[1]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/run-web-app.png
[NuGet]: https://nuget.org

[11]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/configure-productsserver.png
[13]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/install-nuget-service-bus-productsserver.png
[15]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-2.png
[16]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/choose-web-application-template.png
[17]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/add-class-productsportal.png
[18]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/change-authentication.png
[9]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/web-publish-activity.png
[10]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/run-web-app-locally.png

[21]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/run-web-app-locally-no-content.png
[24]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/add-existing-item-link.png
[25]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-13.png
[26]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-14.png
[27]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/launch-app-as-web-app.png

[36]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/App2.png
[37]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-service1.png
[38]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-service2.png
[41]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-multi-tier-40.png
[43]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-43.png

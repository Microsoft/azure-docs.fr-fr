---
title: Prise en main de la bibliothèque Azure CDN pour .NET | Microsoft Docs
description: Apprenez à écrire des applications .NET pour gérer Azure CDN à l’aide de Visual Studio.
services: cdn
documentationcenter: .net
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: 63cf4101-92e7-49dd-a155-a90e54a792ca
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 01/23/2017
ms.author: mazha
ms.custom: has-adal-ref, devx-track-csharp
ms.openlocfilehash: a812704c42a4da5ddf89fe6c5ba0c9a684047f75
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88919486"
---
# <a name="get-started-with-the-azure-cdn-library-for-net"></a>Prise en main de la bibliothèque Azure CDN pour .NET
> [!div class="op_single_selector"]
> * [Node.JS](cdn-app-dev-node.md)
> * [.NET](cdn-app-dev-net.md)
>
>

Vous pouvez utiliser la [bibliothèque Azure CDN pour .NET](/dotnet/api/overview/azure/cdn) pour automatiser la création et la gestion des points de terminaison et profils CDN.  Ce didacticiel présente la création d’une application console .NET simple, qui exécute plusieurs des opérations disponibles.  Il n’a pas vocation à décrire en détail tous les aspects de la bibliothèque Azure CDN pour .NET.

Pour suivre ce didacticiel, vous avez besoin de Visual Studio 2015.  [Visual Studio Community 2015](https://www.visualstudio.com/products/visual-studio-community-vs.aspx) est disponible gratuitement en téléchargement.

> [!TIP]
> Le [projet achevé de ce didacticiel](https://code.msdn.microsoft.com/Azure-CDN-Management-1f2fba2c) est disponible en téléchargement sur MSDN.
>
>

[!INCLUDE [cdn-app-dev-prep](../../includes/cdn-app-dev-prep.md)]

## <a name="create-your-project-and-add-nuget-packages"></a>Créer votre projet et ajouter des packages Nuget
Maintenant que nous avons créé un groupe de ressources pour nos profils CDN et autorisé l’application Azure AD à gérer les points de terminaison et profils CDN au sein de ce groupe, nous pouvons créer notre application.

Dans Visual Studio 2015, cliquez sur **Fichier**, **Nouveau**, **Projet...** pour ouvrir la boîte de dialogue Nouveau projet.  Développez **Visual C#** , puis sélectionnez **Windows** dans le volet de gauche.  Cliquez sur **Application console** dans le volet central.  Nommez votre projet, puis cliquez sur **OK**.

![Nouveau projet](./media/cdn-app-dev-net/cdn-new-project.png)

Notre projet va utiliser certaines bibliothèques Azure contenues dans des packages Nuget.  Ajoutons-les à ce projet.

1. Dans le menu **Outils**, sélectionnez **Gestionnaire de package NuGet**, puis **Console du Gestionnaire de package**.

    ![Gérer les packages NuGet](./media/cdn-app-dev-net/cdn-manage-nuget.png)
2. Dans la Console du Gestionnaire de package, exécutez la commande suivante pour installer la bibliothèque **ADAL (Active Directory Authentication Library)** :

    `Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory`
3. Exécutez la commande suivante pour installer la bibliothèque **Azure CDN Management Library**:

    `Install-Package Microsoft.Azure.Management.Cdn`

## <a name="directives-constants-main-method-and-helper-methods"></a>Directives, constantes, méthode principale et méthodes d’assistance
Rédigeons la structure de base de notre programme.

1. Dans l’onglet Program.cs, remplacez les directives `using` au début par les éléments suivants :

    ```csharp
    using System;
    using System.Collections.Generic;
    using Microsoft.Azure.Management.Cdn;
    using Microsoft.Azure.Management.Cdn.Models;
    using Microsoft.Azure.Management.Resources;
    using Microsoft.Azure.Management.Resources.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using Microsoft.Rest;
    ```
2. Nous devons définir certaines constantes que nos méthodes utiliseront.  Dans la classe `Program` mais avant la méthode `Main`, ajoutez le code suivant.  Veillez à remplacer les espaces réservés, notamment les **&lt;éléments entre chevrons&gt;** , par vos propres valeurs, si nécessaire.

    ```csharp
    //Tenant app constants
    private const string clientID = "<YOUR CLIENT ID>";
    private const string clientSecret = "<YOUR CLIENT AUTHENTICATION KEY>"; //Only for service principals
    private const string authority = "https://login.microsoftonline.com/<YOUR TENANT ID>/<YOUR TENANT DOMAIN NAME>";

    //Application constants
    private const string subscriptionId = "<YOUR SUBSCRIPTION ID>";
    private const string profileName = "CdnConsoleApp";
    private const string endpointName = "<A UNIQUE NAME FOR YOUR CDN ENDPOINT>";
    private const string resourceGroupName = "CdnConsoleTutorial";
    private const string resourceLocation = "<YOUR PREFERRED AZURE LOCATION, SUCH AS Central US>";
    ```
3. De plus, au niveau de la classe, vous devez définir ces deux variables.  Nous les utiliserons ultérieurement pour déterminer si notre profil et notre point de terminaison existent déjà.

    ```csharp
    static bool profileAlreadyExists = false;
    static bool endpointAlreadyExists = false;
    ```
4. Remplacez la méthode `Main` comme suit :

   ```csharp
   static void Main(string[] args)
   {
       //Get a token
       AuthenticationResult authResult = GetAccessToken();

       // Create CDN client
       CdnManagementClient cdn = new CdnManagementClient(new TokenCredentials(authResult.AccessToken))
           { SubscriptionId = subscriptionId };

       ListProfilesAndEndpoints(cdn);

       // Create CDN Profile
       CreateCdnProfile(cdn);

       // Create CDN Endpoint
       CreateCdnEndpoint(cdn);

       Console.WriteLine();

       // Purge CDN Endpoint
       PromptPurgeCdnEndpoint(cdn);

       // Delete CDN Endpoint
       PromptDeleteCdnEndpoint(cdn);

       // Delete CDN Profile
       PromptDeleteCdnProfile(cdn);

       Console.WriteLine("Press Enter to end program.");
       Console.ReadLine();
   }
   ```
5. Certaines de nos autres méthodes posent à l’utilisateur des questions fermées (de type Oui/non).  Ajoutez la méthode suivante pour faciliter l’opération :

    ```csharp
    private static bool PromptUser(string Question)
    {
        Console.Write(Question + " (Y/N): ");
        var response = Console.ReadKey();
        Console.WriteLine();
        if (response.Key == ConsoleKey.Y)
        {
            return true;
        }
        else if (response.Key == ConsoleKey.N)
        {
            return false;
        }
        else
        {
            // They pressed something other than Y or N.  Let's ask them again.
            return PromptUser(Question);
        }
    }
    ```

Maintenant que la structure de base de notre programme est écrite, nous devons créer les méthodes appelées par la méthode `Main` .

## <a name="authentication"></a>Authentification
Pour pouvoir utiliser la bibliothèque Azure CDN Management Library, nous devons authentifier notre principal de service et obtenir un jeton d’authentification.  Cette méthode utilise la bibliothèque ADAL pour récupérer le jeton.

```csharp
private static AuthenticationResult GetAccessToken()
{
    AuthenticationContext authContext = new AuthenticationContext(authority);
    ClientCredential credential = new ClientCredential(clientID, clientSecret);
    AuthenticationResult authResult =
        authContext.AcquireTokenAsync("https://management.core.windows.net/", credential).Result;

    return authResult;
}
```

Si vous utilisez l’authentification d’utilisateurs individuels, la méthode `GetAccessToken` se présentera légèrement différemment.

> [!IMPORTANT]
> N’utilisez ce code que si vous privilégiez l’authentification d’utilisateurs individuels au principal du service.
>
>

```csharp
private static AuthenticationResult GetAccessToken()
{
    AuthenticationContext authContext = new AuthenticationContext(authority);
    AuthenticationResult authResult = authContext.AcquireTokenAsync("https://management.core.windows.net/",
        clientID, new Uri("http://<redirect URI>"), new PlatformParameters(PromptBehavior.RefreshSession)).Result;

    return authResult;
}
```

Veillez à remplacer `<redirect URI>` par l’URI de redirection que vous avez saisie lors de l’inscription de l’application dans Azure AD.

## <a name="list-cdn-profiles-and-endpoints"></a>Répertorier les profils CDN et points de terminaison
Nous sommes maintenant prêts à effectuer des opérations CDN.  La première action de notre méthode est de répertorier tous les profils et points de terminaison dans notre groupe de ressources. Si elle trouve une correspondance avec les noms de profil et de point de terminaison spécifiés dans nos constantes, elle la mémorise pour éviter de créer des doublons.

```csharp
private static void ListProfilesAndEndpoints(CdnManagementClient cdn)
{
    // List all the CDN profiles in this resource group
    var profileList = cdn.Profiles.ListByResourceGroup(resourceGroupName);
    foreach (Profile p in profileList)
    {
        Console.WriteLine("CDN profile {0}", p.Name);
        if (p.Name.Equals(profileName, StringComparison.OrdinalIgnoreCase))
        {
            // Hey, that's the name of the CDN profile we want to create!
            profileAlreadyExists = true;
        }

        //List all the CDN endpoints on this CDN profile
        Console.WriteLine("Endpoints:");
        var endpointList = cdn.Endpoints.ListByProfile(p.Name, resourceGroupName);
        foreach (Endpoint e in endpointList)
        {
            Console.WriteLine("-{0} ({1})", e.Name, e.HostName);
            if (e.Name.Equals(endpointName, StringComparison.OrdinalIgnoreCase))
            {
                // The unique endpoint name already exists.
                endpointAlreadyExists = true;
            }
        }
        Console.WriteLine();
    }
}
```

## <a name="create-cdn-profiles-and-endpoints"></a>Créer des profils CDN et des points de terminaison
Maintenant, nous allons créer un profil.

```csharp
private static void CreateCdnProfile(CdnManagementClient cdn)
{
    if (profileAlreadyExists)
    {
        Console.WriteLine("Profile {0} already exists.", profileName);
    }
    else
    {
        Console.WriteLine("Creating profile {0}.", profileName);
        ProfileCreateParameters profileParms =
            new ProfileCreateParameters() { Location = resourceLocation, Sku = new Sku(SkuName.StandardVerizon) };
        cdn.Profiles.Create(profileName, profileParms, resourceGroupName);
    }
}
```

Après le profil, nous allons créer un point de terminaison.

```csharp
private static void CreateCdnEndpoint(CdnManagementClient cdn)
{
    if (endpointAlreadyExists)
    {
        Console.WriteLine("Profile {0} already exists.", profileName);
    }
    else
    {
        Console.WriteLine("Creating endpoint {0} on profile {1}.", endpointName, profileName);
        EndpointCreateParameters endpointParms =
            new EndpointCreateParameters()
            {
                Origins = new List<DeepCreatedOrigin>() { new DeepCreatedOrigin("Contoso", "www.contoso.com") },
                IsHttpAllowed = true,
                IsHttpsAllowed = true,
                Location = resourceLocation
            };
        cdn.Endpoints.Create(endpointName, endpointParms, profileName, resourceGroupName);
    }
}
```

> [!NOTE]
> L’exemple ci-dessus attribue au point de terminaison une origine nommée *Contoso* avec un nom d’hôte `www.contoso.com`.  Vous devez remplacer celui-ci par le nom d’hôte de votre propre origine.
>
>

## <a name="purge-an-endpoint"></a>Vider un point de terminaison
En supposant que le point de terminaison a été créé, une tâche courante que nous pouvons effectuer dans notre programme consiste à vider le contenu de notre point de terminaison.

```csharp
private static void PromptPurgeCdnEndpoint(CdnManagementClient cdn)
{
    if (PromptUser(String.Format("Purge CDN endpoint {0}?", endpointName)))
    {
        Console.WriteLine("Purging endpoint. Please wait...");
        cdn.Endpoints.PurgeContent(endpointName, profileName, resourceGroupName, new List<string>() { "/*" });
        Console.WriteLine("Done.");
        Console.WriteLine();
    }
}
```

> [!NOTE]
> Dans l’exemple ci-dessus, la chaîne `/*` indique que je souhaite vider tous les éléments à la racine du chemin d’accès du point de terminaison.  Cela revient à cocher la case **Purge All** (Purger tout) dans la boîte de dialogue de vidage du portail Azure. Dans la méthode `CreateCdnProfile`, j’ai créé notre profil comme un profil **Azure CDN de Verizon** à l’aide du code `Sku = new Sku(SkuName.StandardVerizon)`, pour que l’opération aboutisse.  Toutefois, les profils **Azure CDN de Akamai** ne prennent pas en charge l’option **Purge All** (Purger tout). Donc, si j’utilisais un profil Akamai pour ce didacticiel, je devrais inclure les chemins d’accès spécifiques à vider.
>
>

## <a name="delete-cdn-profiles-and-endpoints"></a>Supprimer des profils CDN et des points de terminaison
Les dernières méthodes supprimeront notre point de terminaison et notre profil.

```csharp
private static void PromptDeleteCdnEndpoint(CdnManagementClient cdn)
{
    if(PromptUser(String.Format("Delete CDN endpoint {0} on profile {1}?", endpointName, profileName)))
    {
        Console.WriteLine("Deleting endpoint. Please wait...");
        cdn.Endpoints.DeleteIfExists(endpointName, profileName, resourceGroupName);
        Console.WriteLine("Done.");
        Console.WriteLine();
    }
}

private static void PromptDeleteCdnProfile(CdnManagementClient cdn)
{
    if(PromptUser(String.Format("Delete CDN profile {0}?", profileName)))
    {
        Console.WriteLine("Deleting profile. Please wait...");
        cdn.Profiles.DeleteIfExists(profileName, resourceGroupName);
        Console.WriteLine("Done.");
        Console.WriteLine();
    }
}
```

## <a name="running-the-program"></a>Exécution du programme
Nous pouvons maintenant compiler et exécuter le programme en cliquant sur le bouton **Démarrer** dans Visual Studio.

![Exécution du programme](./media/cdn-app-dev-net/cdn-program-running-1.png)

Lorsque le programme atteint l’invite ci-dessus, vous pouvez revenir à votre groupe de ressources dans le portail Azure et vérifier que le profil a été créé.

![Opération réussie.](./media/cdn-app-dev-net/cdn-success.png)

Nous pouvons alors confirmer les invites pour exécuter le reste du programme.

![Fin du programme](./media/cdn-app-dev-net/cdn-program-running-2.png)

## <a name="next-steps"></a>Étapes suivantes
Pour voir le projet achevé obtenu à partir de cette procédure pas à pas, [téléchargez l’exemple](https://code.msdn.microsoft.com/Azure-CDN-Management-1f2fba2c).

Pour trouver de la documentation supplémentaire sur la bibliothèque Azure CDN Management Library pour .NET, consultez la [référence sur MSDN](/dotnet/api/overview/azure/cdn).

Gérez vos ressources CDN avec [PowerShell](cdn-manage-powershell.md).

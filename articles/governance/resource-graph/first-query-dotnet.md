---
title: 'Démarrage rapide : Votre première requête .NET Core'
description: Dans ce guide de démarrage rapide, vous suivez les étapes pour activer les packages NuGet Resource Graph pour .NET Core et vous exécutez votre première requête.
ms.date: 10/14/2020
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.openlocfilehash: 5f825b22712956c0b94531ddb163301167f30f00
ms.sourcegitcommit: 1b47921ae4298e7992c856b82cb8263470e9e6f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92057465"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-net-core"></a>Démarrage rapide : Exécuter votre première requête Resource Graph à l’aide de .NET Core

La première étape pour utiliser Azure Resource Graph consiste à vérifier que les packages nécessaires pour .NET Core sont installés. Ce démarrage rapide décrit le processus d’ajout des packages à votre installation .NET Core.

Au terme de ce processus, vous aurez ajouté les packages à votre installation .NET Core et vous pourrez exécuter votre première requête Resource Graph.

## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.
- Un principal de service Azure, y compris les _clientId_ et _clientSecret_. Si vous n’avez pas de principal de service à utiliser avec Resource Graph ou souhaitez en créer un, consultez [Authentification des bibliothèques de gestion Azure pour .NET](/dotnet/azure/sdk/authentication#mgmt-auth).
  Ignorez l’étape d’installation des packages .NET Core, car nous l’effectuerons dans les prochaines étapes.

## <a name="create-the-resource-graph-project"></a>Créer le projet Resource Graph

Pour permettre à .NET Core d’interroger Azure Resource Graph, créez une application console et installez les packages nécessaires.

1. Vérifiez que la version la plus récente de .NET Core est installée (**3.1.5** au minimum). Si ce n’est pas le cas, téléchargez-la à l’adresse [dotnet.microsoft.com](https://dotnet.microsoft.com/download/dotnet-core).

1. Initialisez une nouvelle application console .NET Core nommée « argQuery » :

   ```dotnetcli
   dotnet new console --name "argQuery"
   ```

1. Remplacez les répertoires dans le dossier du nouveau projet et installez les packages nécessaires pour Azure Resource Graph :

   ```dotnetcli
   # Add the Resource Graph package for .NET Core
   dotnet add package Microsoft.Azure.Management.ResourceGraph --version 2.0.0

   # Add the Azure app auth package for .NET Core
   dotnet add package Microsoft.Azure.Services.AppAuthentication --version 1.5.0
   ```

1. Remplacez le `program.cs` par défaut par le code suivant et enregistrez le fichier mis à jour :

   ```csharp
   using System;
   using System.Collections.Generic;
   using System.Threading.Tasks;
   using Microsoft.IdentityModel.Clients.ActiveDirectory;
   using Microsoft.Rest;
   using Microsoft.Azure.Management.ResourceGraph;
   using Microsoft.Azure.Management.ResourceGraph.Models;

   namespace argQuery
   {
       class Program
       {
           static async Task Main(string[] args)
           {
               string strTenant = args[0];
               string strClientId = args[1];
               string strClientSecret = args[2];
               string strSubscriptionId = args[3];
               string strQuery = args[4];

               AuthenticationContext authContext = new AuthenticationContext("https://login.microsoftonline.com/" + strTenant);
               AuthenticationResult authResult = await authContext.AcquireTokenAsync("https://management.core.windows.net", new ClientCredential(strClientId, strClientSecret));
               ServiceClientCredentials serviceClientCreds = new TokenCredentials(authResult.AccessToken);

               ResourceGraphClient argClient = new ResourceGraphClient(serviceClientCreds);
               QueryRequest request = new QueryRequest();
               request.Subscriptions = new List<string>(){ strSubscriptionId };
               request.Query = strQuery;

               QueryResponse response = argClient.Resources(request);
               Console.WriteLine("Records: " + response.Count);
               Console.WriteLine("Data:\n" + response.Data);
           }
       }
   }
   ```

1. Générez et publiez l’application console`argQuery` :

   ```dotnetcli
   dotnet build
   dotnet publish -o {run-folder}
   ```

## <a name="run-your-first-resource-graph-query"></a>Exécuter votre première requête Resource Graph

L’application console .NET Core étant générée et publiée, nous pouvons tester une requête Resource Graph simple. La requête retourne les cinq premières ressources Azure avec le nom (**Name**) et le **Type** de chaque ressource.

Dans chaque appel de `argQuery`, vous devez remplacer les variables utilisées par vos propres valeurs :

- Remplacer `{tenantId}` par l’ID de votre locataire
- `{clientId}` - Remplacer l’ID client de votre principal de service
- `{clientSecret}` - Remplacer le secret client de votre principal de service
- Remplacer `{subscriptionId}` par votre ID d’abonnement

1. Remplacez les répertoires par le `{run-folder}` que vous avez défini à l’aide de la commande `dotnet publish` précédente.

1. Exécutez votre première requête Azure Resource Graph à l’aide de l’application console .NET Core compilée :

   ```bash
   argQuery "{tenantId}" "{clientId}" "{clientSecret}" "{subscriptionId}" "Resources | project name, type | limit 5"
   ```

   > [!NOTE]
   > Comme cet exemple de requête ne fournit pas un modificateur de tri tel que `order by`, l’exécution répétée de cette requête peut produire un ensemble différent de ressources.

1. Remplacez le dernier paramètre par `argQuery.exe` et la requête par `order by` suivi de la propriété **Name** :

   ```bash
   argQuery "{tenantId}" "{clientId}" "{clientSecret}" "{subscriptionId}" "Resources | project name, type | limit 5 | order by name asc"
   ```

   > [!NOTE]
   > Comme précédemment, l’exécution répétée de cette requête peut produire un ensemble différent de ressources. L’ordre des commandes de requête est important. Dans cet exemple, `order by` vient après `limit`. Cet ordre de commande limite d’abord les résultats de la requête, puis les classe.

1. Remplacez le dernier paramètre par `argQuery.exe` et changez la requête en `order by` pour classer d’abord les résultats selon la propriété **Name**, puis limitez (`limit`) les résultats aux cinq premiers :

   ```bash
   argQuery "{tenantId}" "{clientId}" "{clientSecret}" "{subscriptionId}" "Resources | project name, type | order by name asc | limit 5"
   ```

Si votre environnement ne change pas et si vous exécutez plusieurs fois la requête finale, les résultats retournés sont cohérents et classés en fonction de la propriété **Name**, mais toujours limités aux cinq premiers.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez supprimer l’application console .NET Core et les packages installés, vous pouvez le faire en supprimant le dossier du projet `argQuery`.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé une application console .NET Core avec les packages Resource Graph nécessaires et exécuté votre première requête. Pour plus d’informations sur le langage Resource Graph, passez à la page des détails du langage de requête.

> [!div class="nextstepaction"]
> [Obtenir plus d’informations sur le langage de requête](./concepts/query-language.md)
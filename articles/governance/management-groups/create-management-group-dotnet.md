---
title: 'Démarrage rapide : Créer un groupe d’administration avec .NET Core'
description: Dans ce démarrage rapide, vous allez utiliser .NET Core pour créer un groupe d’administration afin de hiérarchiser vos ressources.
ms.date: 09/30/2020
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.openlocfilehash: 138998fdc23fd8a296ca50093e2952017888041f
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91604540"
---
# <a name="quickstart-create-a-management-group-with-net-core"></a>Démarrage rapide : Créer un groupe d’administration avec .NET Core

Les groupes d’administration sont des conteneurs qui vous aident à gérer l’accès, la stratégie et la conformité dans plusieurs abonnements. Créez ces conteneurs pour construire une hiérarchie efficace utilisable avec [Azure Policy](../policy/overview.md) et les [contrôles d’accès en fonction du rôle Azure](../../role-based-access-control/overview.md). Pour plus d’informations sur les groupes d’administration, consultez [Organiser vos ressources avec des groupes d’administration Azure](overview.md).

La création du premier groupe d’administration dans l’annuaire peut nécessiter jusqu’à 15 minutes. En effet, des processus s’exécutent la première fois pour configurer le service des groupes d’administration dans Azure pour votre annuaire. Vous recevez une notification quand le processus est terminé. Pour plus d’informations, consultez la [configuration initiale des groupes d’administration](./overview.md#initial-setup-of-management-groups).

## <a name="prerequisites"></a>Prérequis

- Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

- Un principal de service Azure, y compris les _clientId_ et _clientSecret_. Si vous n’avez pas de principal de service à utiliser avec Azure Policy ou si souhaitez en créer un, consultez [Authentification des bibliothèques de gestion Azure pour .NET](/dotnet/azure/sdk/authentication#mgmt-auth).
  Ignorez l’étape d’installation des packages .NET Core, car nous l’effectuerons dans les prochaines étapes.

- Tout utilisateur Azure AD dans le locataire peut créer un groupe d’administration sans l’autorisation d’écriture du groupe d’administration attribuée à cet utilisateur si la [protection de la hiérarchie](./how-to/protect-resource-hierarchy.md#setting---require-authorization) n’est pas activée. Ce nouveau groupe d’administration devient alors un enfant du groupe d’administration racine ou le [groupe d’administration par défaut](./how-to/protect-resource-hierarchy.md#setting---default-management-group), et le rôle « Propriétaire » est attribué au créateur. Le service de groupe d’administration offre cette possibilité afin que les attributions de rôles ne soient pas nécessaires au niveau de la racine. Aucun utilisateur n’a accès au groupe d’administration racine lors de sa création. Pour éviter l’obstacle que constitue la recherche des administrateurs généraux Azure AD pour commencer à utiliser des groupes d’administration, nous autorisons la création des groupes d’administration initiaux au niveau racine.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="application-setup"></a>Configuration des applications

Pour permettre à .NET Core de gérer des groupes d’administration, créez une application console et installez les packages nécessaires.

1. Vérifiez que la version la plus récente de .NET Core est installée (au minimum, la version **3.1.8**). Si ce n’est pas le cas, téléchargez-la à l’adresse [dotnet.microsoft.com](https://dotnet.microsoft.com/download/dotnet-core).

1. Initialisez une nouvelle application console .NET Core nommée « mgCreate » :

   ```dotnetcli
   dotnet new console --name "mgCreate"
   ```

1. Remplacez les répertoires dans le dossier du nouveau projet, et installez les packages requis pour Azure Policy :

   ```dotnetcli
   # Add the Azure Policy package for .NET Core
   dotnet add package Microsoft.Azure.Management.ManagementGroups --version 1.1.1-preview

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
   using Microsoft.Azure.Management.ManagementGroups;
   using Microsoft.Azure.Management.ManagementGroups.Models;
   
   namespace mgCreate
   {
       class Program
       {
           static async Task Main(string[] args)
           {
               string strTenant = args[0];
               string strClientId = args[1];
               string strClientSecret = args[2];
               string strGroupId = args[3];
               string strDisplayName = args[4];
   
               var authContext = new AuthenticationContext($"https://login.microsoftonline.com/{strTenant}");
               var authResult = await authContext.AcquireTokenAsync(
                      "https://management.core.windows.net",
                      new ClientCredential(strClientId, strClientSecret));
   
               using (var client = new ManagementGroupsAPIClient(new TokenCredentials(authResult.AccessToken)))
               {
                   var mgRequest = new CreateManagementGroupRequest
                   {
                       DisplayName = strDisplayName
                   };
                   var response = await client.ManagementGroups.CreateOrUpdateAsync(strGroupId, mgRequest);
               }
           }
       }
   }
   ```

1. Générez et publiez l’application console`mgCreate` :

   ```dotnetcli
   dotnet build
   dotnet publish -o {run-folder}
   ```

## <a name="create-the-management-group"></a>Créer le groupe d’administration

Dans ce démarrage rapide, vous allez créer un groupe d’administration dans le groupe d’administration racine.

1. Remplacez les répertoires par le `{run-folder}` que vous avez défini à l’aide de la commande `dotnet publish` précédente.

1. Entrez la commande suivante dans le terminal :

   ```bash
   mgCreate.exe `
      "{tenantId}" `
      "{clientId}" `
      "{clientSecret}" `
      "{groupID}" `
      "{displayName}"
   ```

Les commandes précédentes utilisent les informations suivantes :

- Remplacer `{tenantId}` par l’ID de votre locataire
- `{clientId}` - Remplacer l’ID client de votre principal de service
- `{clientSecret}` - Remplacer le secret client de votre principal de service
- `{groupID}` - Remplacer par l’ID de votre nouveau groupe d’administration
- `{displayName}` - Remplacer par le nom convivial de votre nouveau groupe d’administration

Le résultat est un nouveau groupe d’administration dans le groupe d’administration racine.

## <a name="clean-up-resources"></a>Nettoyer les ressources

- Supprimez le nouveau groupe d’administration via le portail.

- Si vous souhaitez supprimer l’application console .NET Core et les packages installés, supprimez le dossier de projet `mgCreate`.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un groupe d’administration pour organiser votre hiérarchie de ressources. Le groupe d’administration peut contenir des abonnements ou d’autres groupes d’administration.

Pour en savoir plus sur les groupes d’administration et sur la gestion de votre hiérarchie de ressources, consultez :

> [!div class="nextstepaction"]
> [Gérer vos ressources avec des groupes d’administration](./manage.md)
---
title: 'Démarrage rapide : Nouvelle attribution de stratégie à l’aide de .NET Core'
description: Dans ce démarrage rapide, vous allez utiliser .NET Core pour créer une attribution Azure Policy afin d’identifier des ressources non conformes.
ms.date: 09/30/2020
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.openlocfilehash: c4c8f8e9df544b6fc00b5b7701435f5a606f9764
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91604550"
---
# <a name="quickstart-create-a-policy-assignment-to-identify-non-compliant-resources-with-net-core"></a>Démarrage rapide : Créer une attribution de stratégie pour identifier des ressources non conformes avec .NET Core

La première étape pour comprendre la conformité dans Azure consiste à identifier l’état de vos ressources. Dans ce démarrage rapide, vous créerez une affectation de stratégie pour identifier les machines virtuelles qui n’utilisent pas de disques managés. Lorsque vous aurez terminé, vous identifierez les machines virtuelles qui ne sont _pas conformes_.

La bibliothèque .NET Core est utilisée pour gérer les ressources Azure. Ce guide explique comment utiliser la bibliothèque .NET Core pour Azure Policy afin de créer une attribution de stratégie.

## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.
- Un principal de service Azure, y compris les _clientId_ et _clientSecret_. Si vous n’avez pas de principal de service à utiliser avec Azure Policy ou si souhaitez en créer un, consultez [Authentification des bibliothèques de gestion Azure pour .NET](/dotnet/azure/sdk/authentication#mgmt-auth).
  Ignorez l’étape d’installation des packages .NET Core, car nous l’effectuerons dans les prochaines étapes.

## <a name="create-the-azure-policy-project"></a>Créer le projet Azure Policy

Pour permettre à .NET Core de gérer Azure Resource Graph, créez une application console et installez les packages nécessaires.

1. Vérifiez que la version la plus récente de .NET Core est installée (au minimum, la version **3.1.8**). Si ce n’est pas le cas, téléchargez-la à l’adresse [dotnet.microsoft.com](https://dotnet.microsoft.com/download/dotnet-core).

1. Initialisez une nouvelle application console .NET Core nommée « policyAssignment » :

   ```dotnetcli
   dotnet new console --name "policyAssignment"
   ```

1. Remplacez les répertoires dans le dossier du nouveau projet, et installez les packages requis pour Azure Policy :

   ```dotnetcli
   # Add the Azure Policy package for .NET Core
   dotnet add package Microsoft.Azure.Management.ResourceManager --version 3.10.0-preview

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
   using Microsoft.Azure.Management.ResourceManager;
   using Microsoft.Azure.Management.ResourceManager.Models;
   
   namespace policyAssignment
   {
       class Program
       {
           static async Task Main(string[] args)
           {
               string strTenant = args[0];
               string strClientId = args[1];
               string strClientSecret = args[2];
               string strSubscriptionId = args[3];
               string strName = args[4];
               string strDisplayName = args[5];
               string strPolicyDefID = args[6];
               string strDescription = args[7];
               string strScope = args[8];
   
               var authContext = new AuthenticationContext($"https://login.microsoftonline.com/{strTenant}");
               var authResult = await authContext.AcquireTokenAsync(
                   "https://management.core.windows.net",
                   new ClientCredential(strClientId, strClientSecret));

               using (var client = new PolicyClient(new TokenCredentials(authResult.AccessToken)))
               {
                   var policyAssignment = new PolicyAssignment
                   {
                       DisplayName = strDisplayName,
                       PolicyDefinitionId = strPolicyDefID,
                       Description = strDescription
                   };
                   var response = await client.PolicyAssignments.CreateAsync(strScope, strName, policyAssignment);
               }
           }
       }
   }
   ```

1. Générez et publiez l’application console`policyAssignment` :

   ```dotnetcli
   dotnet build
   dotnet publish -o {run-folder}
   ```

## <a name="create-a-policy-assignment"></a>Créer une affectation de stratégie

Dans ce guide de démarrage rapide, vous allez créer une attribution de stratégie et attribuer la définition **Auditer les machines virtuelles qui n'utilisent aucun disque managé** (`06a78e20-9358-41c9-923c-fb736d382a4d`). Cette stratégie identifie les ressources qui ne sont pas conformes aux conditions définies dans sa définition.

1. Remplacez les répertoires par le `{run-folder}` que vous avez défini à l’aide de la commande `dotnet publish` précédente.

1. Entrez la commande suivante dans le terminal :

   ```bash
   policyAssignment.exe `
      "{tenantId}" `
      "{clientId}" `
      "{clientSecret}" `
      "{subscriptionId}" `
      "audit-vm-manageddisks" `
      "Audit VMs without managed disks Assignment" `
      "/providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d" `
      "Shows all virtual machines not using managed disks" `
      "{scope}"
   ```

Les commandes précédentes utilisent les informations suivantes :

- Remplacer `{tenantId}` par l’ID de votre locataire
- `{clientId}` - Remplacer l’ID client de votre principal de service
- `{clientSecret}` - Remplacer le secret client de votre principal de service
- Remplacer `{subscriptionId}` par votre ID d’abonnement
- **name** : nom unique de l’objet d’attribution de stratégie. L’exemple ci-dessus utilise _audit-vm-manageddisks_.
- **displayName** : nom d’affichage pour l’attribution de stratégie. Dans ce cas, nous allons utiliser l’affectation _Audit VMs without managed disks_ (Auditer les machines virtuelles sans disques managés).
- **policyDefID** : chemin de la définition de stratégie, sur lequel vous vous basez pour créer l’attribution. Dans ce cas, il s’agit de l’ID de la définition de stratégie _Auditer les machines virtuelles qui n’utilisent pas de disques managés_.
- **description** : explication plus détaillée de ce que fait la stratégie ou de la raison pour laquelle elle est attribuée à cette étendue.
- **scope** : une étendue détermine les ressources ou le groupe de ressources sur lesquels l'attribution de stratégie est appliquée. Cette étendue peut aller d'un groupe d'administration à une ressource individuelle. Veillez à remplacer `{scope}` par l'un des modèles suivants :
  - Groupe d'administration : `/providers/Microsoft.Management/managementGroups/{managementGroup}`
  - Abonnement : `/subscriptions/{subscriptionId}`
  - Groupe de ressources : `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}`
  - Ressource : `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/[{parentResourcePath}/]`

Vous êtes maintenant prêt à identifier les ressources non conformes pour comprendre l’état de conformité de votre environnement.

## <a name="identify-non-compliant-resources"></a>Identifier des ressources non conformes

Maintenant que votre attribution de stratégie est créée, vous pouvez identifier les ressources qui ne sont pas conformes.

1. Initialisez une nouvelle application console .NET Core nommée « policyAssignment » :

   ```dotnetcli
   dotnet new console --name "policyCompliance"
   ```

1. Remplacez les répertoires dans le dossier du nouveau projet, et installez les packages requis pour Azure Policy :

   ```dotnetcli
   # Add the Azure Policy package for .NET Core
   dotnet add package Microsoft.Azure.Management.PolicyInsights --version 3.1.0

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
   using Microsoft.Azure.Management.PolicyInsights;
   using Microsoft.Azure.Management.PolicyInsights.Models;
   
   namespace policyAssignment
   {
       class Program
       {
           static async Task Main(string[] args)
           {
               string strTenant = args[0];
               string strClientId = args[1];
               string strClientSecret = args[2];
               string strSubscriptionId = args[3];
               string strName = args[4];
   
               var authContext = new AuthenticationContext($"https://login.microsoftonline.com/{strTenant}");
               var authResult = await authContext.AcquireTokenAsync(
                   "https://management.core.windows.net",
                   new ClientCredential(strClientId, strClientSecret));
   
               using (var client = new PolicyInsightsClient(new TokenCredentials(authResult.AccessToken)))
               {
                   var policyQueryOptions = new QueryOptions
                   {
                       Filter = $"IsCompliant eq false and PolicyAssignmentId eq '{strName}'",
                       Apply = "groupby(ResourceId)"
                   };
   
                   var response = await client.PolicyStates.ListQueryResultsForSubscriptionAsync(
                       "latest", strSubscriptionId, policyQueryOptions);
                   Console.WriteLine(response.Odatacount);
               }
           }
       }
   }
   ```

1. Générez et publiez l’application console`policyAssignment` :

   ```dotnetcli
   dotnet build
   dotnet publish -o {run-folder}
   ```

1. Remplacez les répertoires par le `{run-folder}` que vous avez défini à l’aide de la commande `dotnet publish` précédente.

1. Entrez la commande suivante dans le terminal :

   ```bash
   policyCompliance.exe `
      "{tenantId}" `
      "{clientId}" `
      "{clientSecret}" `
      "{subscriptionId}" `
      "audit-vm-manageddisks"
   ```

Les commandes précédentes utilisent les informations suivantes :

- Remplacer `{tenantId}` par l’ID de votre locataire
- `{clientId}` - Remplacer l’ID client de votre principal de service
- `{clientSecret}` - Remplacer le secret client de votre principal de service
- Remplacer `{subscriptionId}` par votre ID d’abonnement
- **name** : nom unique de l’objet d’attribution de stratégie. L’exemple ci-dessus utilise _audit-vm-manageddisks_.

Les résultats dans `response` correspondent à ce que vous voyez sous l’onglet **Conformité des ressources** d’une attribution de stratégie dans l’affichage du portail Azure.

## <a name="clean-up-resources"></a>Nettoyer les ressources

- Supprimez l’attribution de stratégie _Audit VMs without managed disks Assignment_ (Auditer les machines virtuelles sans attribution de disques managés) via le portail. La définition de stratégie étant intégrée, il n’y a aucune définition à supprimer.

- Si vous souhaitez supprimer les applications console .NET Core et les packages installés, supprimez les dossiers de projet `policyAssignment` et `policyCompliance`.

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez affecté une définition de stratégie pour identifier les ressources non conformes de votre environnement Azure.

Pour en savoir plus sur l'attribution de définitions de stratégie visant à vérifier que les nouvelles ressources sont conformes, suivez le tutoriel :

> [!div class="nextstepaction"]
> [Création et gestion des stratégies](./tutorials/create-and-manage.md)

---
title: 'Tutoriel : Accès d’une application web à Microsoft Graph en tant qu’application | Azure'
description: Dans ce tutoriel, vous allez apprendre à accéder à des données dans Microsoft Graph à l’aide d’identités managées.
services: microsoft-graph, app-service-web
author: rwike77
manager: CelesteDG
ms.service: app-service-web
ms.topic: tutorial
ms.workload: identity
ms.date: 12/16/2020
ms.author: ryanwi
ms.reviewer: stsoneff
ms.custom: azureday1
ms.openlocfilehash: 3d6e9d1d66425655f0db5751a28746b6a6a31c86
ms.sourcegitcommit: 8c3a656f82aa6f9c2792a27b02bbaa634786f42d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97631763"
---
# <a name="tutorial-access-microsoft-graph-from-a-secured-app-as-the-app"></a>Tutoriel : Accéder à Microsoft Graph à partir d’une application sécurisée en tant qu’application

Découvrez comment accéder à Microsoft Graph à partir d’une application web s’exécutant sur Azure App Service.

:::image type="content" alt-text="Diagramme illustrant l’accès à Microsoft Graph." source="./media/scenario-secure-app-access-microsoft-graph/web-app-access-graph.svg" border="false":::

Vous souhaitez appeler Microsoft Graph pour le compte de l’application web. Un moyen sûr d’accorder à votre application web l’accès aux données consiste à utiliser une [identité managée affectée par le système](../active-directory/managed-identities-azure-resources/overview.md). Une identité managée d’Azure Active Directory permet à App Service d’accéder aux ressources par le biais du contrôle d’accès en fonction du rôle (RBAC) sans demander d’informations d’identification d’application. Après avoir affecté une identité managée à votre application web, Azure s’occupe de la création et de la distribution d’un certificat. Vous n’avez pas à vous soucier de la gestion des secrets ou des informations d’identification d’application.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Créer une identité managée affectée par le système sur une application web.
> * Ajouter des autorisations d’API Microsoft Graph à une identité managée.
> * Appeler Microsoft Graph à partir d’une application web à l’aide d’identités managées.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

* Une application web s’exécutant sur Azure App Service et pour laquelle le [module d’authentification/autorisation App Service est activé](scenario-secure-app-authentication-app-service.md)

## <a name="enable-managed-identity-on-app"></a>Activer une identité managée sur l’application

Si vous créez et publiez votre application web à l’aide de Visual Studio, l’identité managée a été activée pour vous sur votre application. Dans votre service d’application, sélectionnez **Identité** dans le volet gauche, puis **Affectée par le système**. Vérifiez qu’**État** a la valeur **Activé**. Si ce n’est pas le cas, sélectionnez **Enregistrer**, puis **Oui** pour activer l’identité managée affectée par le système. Lorsque l’identité managée est activée, l’état prend la valeur **Activé** et l’ID d’objet est disponible.

Notez la valeur de l’**ID d’objet**, dont vous aurez besoin à l’étape suivante.

:::image type="content" alt-text="Capture d’écran montrant l’identité affectée par le système." source="./media/scenario-secure-app-access-microsoft-graph/create-system-assigned-identity.png":::

## <a name="grant-access-to-microsoft-graph"></a>Accorder l’accès à Microsoft Graph

Lors de l’accès à Microsoft Graph, l’identité managée doit disposer des autorisations appropriées pour l’opération qu’elle souhaite effectuer. Il n’existe actuellement aucune option permettant d’attribuer de telles autorisations par le biais du portail Azure. Le script suivant ajoute les autorisations d’API Microsoft Graph demandées à l’objet de principal de service d’identité managée.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
# Install the module. (You need admin on the machine.)
# Install-Module AzureAD.

# Your tenant ID (in the Azure portal, under Azure Active Directory > Overview).
$TenantID="<tenant-id>"
$resourceGroup = "securewebappresourcegroup"
$webAppName="SecureWebApp-20201102125811"

# Get the ID of the managed identity for the web app.
$spID = (Get-AzWebApp -ResourceGroupName $resourceGroup -Name $webAppName).identity.principalid

# Check the Microsoft Graph documentation for the permission you need for the operation.
$PermissionName = "User.Read.All"

Connect-AzureAD -TenantId $TenantID

# Get the service principal for Microsoft Graph.
# First result should be AppId 00000003-0000-0000-c000-000000000000
$GraphServicePrincipal = Get-AzureADServicePrincipal -SearchString "Microsoft Graph" | Select-Object -first 1

# Assign permissions to the managed identity service principal.
$AppRole = $GraphServicePrincipal.AppRoles | `
Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}

New-AzureAdServiceAppRoleAssignment -ObjectId $spID -PrincipalId $spID `
-ResourceId $GraphServicePrincipal.ObjectId -Id $AppRole.Id
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az login

webAppName="SecureWebApp-20201106120003"

spId=$(az resource list -n $webAppName --query [*].identity.principalId --out tsv)

graphResourceId=$(az ad sp list --display-name "Microsoft Graph" --query [0].objectId --out tsv)

appRoleId=$(az ad sp list --display-name "Microsoft Graph" --query "[0].appRoles[?value=='User.Read.All' && contains(allowedMemberTypes, 'Application')].id" --output tsv)

uri=https://graph.microsoft.com/v1.0/servicePrincipals/$spId/appRoleAssignments

body="{'principalId':'$spId','resourceId':'$graphResourceId','appRoleId':'$appRoleId'}"

az rest --method post --uri $uri --body $body --headers "Content-Type=application/json"
```

---

Après avoir exécuté le script, vous pouvez vérifier dans le [portail Azure](https://portal.azure.com) que les autorisations d’API demandées sont affectées à l’identité managée.

Accédez à **Azure Active Directory**, puis sélectionnez **Applications d’entreprise**. Ce volet affiche tous les principaux de service de votre locataire. Dans **Toutes les applications**, sélectionnez le principal de service de l’identité managée. 

Si vous suivez ce tutoriel, il existe deux principaux de service avec le même nom d’affichage (par exemple, SecureWebApp2020094113531). Le principal de service qui a une **URL de la page d’accueil** représente l’application web dans votre locataire. Le principal de service sans l’**URL de la page d’accueil** représente l’identité managée affectée par le système pour votre application web. La valeur de l’**ID d’objet** de l’identité managée correspond à l’ID d’objet de l’identité managée que vous avez créé plus tôt.

Sélectionnez le principal de service pour l’identité managée.

:::image type="content" alt-text="Capture d’écran montrant l’option Toutes les applications." source="./media/scenario-secure-app-access-microsoft-graph/enterprise-apps-all-applications.png":::

Dans **Vue d’ensemble**, sélectionnez **Autorisations** ; vous verrez alors les autorisations ajoutées pour Microsoft Graph.

:::image type="content" alt-text="Capture d’écran montrant le volet Autorisations." source="./media/scenario-secure-app-access-microsoft-graph/enterprise-apps-permissions.png":::

## <a name="call-microsoft-graph-net"></a>Appeler Microsoft Graph (.NET)

La classe [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential) est utilisée pour obtenir les informations d’identification d’un jeton pour votre code afin d’autoriser les demandes d’accès à Microsoft Graph. Créez une instance de la classe [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential), qui utilise l’identité managée pour extraire des jetons et les attacher au client de service. L’exemple de code suivant obtient les informations d’identification du jeton authentifiées, et les utilise pour créer un objet de client de service, qui obtient les utilisateurs du groupe.

Pour voir ce code dans un exemple d’application, consultez l’[exemple sur GitHub](https://github.com/Azure-Samples/ms-identity-easyauth-dotnet-storage-graphapi/tree/main/3-WebApp-graphapi-managed-identity).

### <a name="install-the-microsoftgraph-client-library-package"></a>Installer le package de la bibliothèque cliente Microsoft.Graph

Installez le [package NuGet Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph) dans votre projet à l’aide de l’interface de ligne de commande .NET Core ou de la console du gestionnaire de package dans Visual Studio.

# <a name="command-line"></a>[Ligne de commande](#tab/command-line)

Ouvrez une ligne de commande et basculez vers le répertoire qui contient votre fichier projet.

Exécutez les commandes d’installation.

```dotnetcli
dotnet add package Microsoft.Graph
```

# <a name="package-manager"></a>[Gestionnaire de package](#tab/package-manager)

Ouvrez le projet/la solution dans Visual Studio, puis ouvrez la console à l’aide de la commande **Outils** > **Gestionnaire de package NuGet** > **Console du gestionnaire de package**.

Exécutez les commandes d’installation.
```powershell
Install-Package Microsoft.Graph
```

---

### <a name="example"></a>Exemple

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Azure.Identity;
using Microsoft.Graph.Core;
using System.Net.Http.Headers;

...

public IList<MSGraphUser> Users { get; set; }

public async Task OnGetAsync()
{
    // Create the Microsoft Graph service client with a DefaultAzureCredential class, which gets an access token by using the available Managed Identity.
    var credential = new DefaultAzureCredential();
    var token = credential.GetToken(
        new Azure.Core.TokenRequestContext(
            new[] { "https://graph.microsoft.com/.default" }));

    var accessToken = token.Token;
    var graphServiceClient = new GraphServiceClient(
        new DelegateAuthenticationProvider((requestMessage) =>
        {
            requestMessage
            .Headers
            .Authorization = new AuthenticationHeaderValue("bearer", accessToken);

            return Task.CompletedTask;
        }));

    List<MSGraphUser> msGraphUsers = new List<MSGraphUser>();
    try
    {
        var users =await graphServiceClient.Users.Request().GetAsync();
        foreach(var u in users)
        {
            MSGraphUser user = new MSGraphUser();
            user.userPrincipalName = u.UserPrincipalName;
            user.displayName = u.DisplayName;
            user.mail = u.Mail;
            user.jobTitle = u.JobTitle;

            msGraphUsers.Add(user);
        }
    }
    catch(Exception ex)
    {
        string msg = ex.Message;
    }

    Users = msGraphUsers;
}
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous avez terminé ce tutoriel et que vous n’avez plus besoin de l’application web ni des ressources associées, [nettoyez les ressources que vous avez créées](scenario-secure-app-clean-up-resources.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
>
> * Créer une identité managée affectée par le système sur une application web.
> * Ajouter des autorisations d’API Microsoft Graph à une identité managée.
> * Appeler Microsoft Graph à partir d’une application web à l’aide d’identités managées.

Découvrez comment connecter une [application .NET Core](tutorial-dotnetcore-sqldb-app.md), une [application Python](tutorial-python-postgresql-app.md), une [application Java](tutorial-java-spring-cosmosdb.md) ou une [application Node.js](tutorial-nodejs-mongodb-app.md) à une base de données.
---
title: 'Tutoriel : Utiliser Azure Key Vault avec une application web Azure dans .NET'
description: Dans ce tutoriel, vous allez configurer une application web Azure dans une application ASP.NET Core pour lire un secret de votre coffre de clés.
services: key-vault
author: msmbaldwin
manager: rajvijan
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 05/06/2020
ms.author: mbaldwin
ms.custom: devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: 6bb1aafd942046faa77072d99af043ebd43b4a8a
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97589965"
---
# <a name="tutorial-use-a-managed-identity-to-connect-key-vault-to-an-azure-web-app-in-net"></a>Tutoriel : Utiliser une identité managée pour connecter Key Vault à une application web Azure dans .NET

[Azure Key Vault](./overview.md) fournit un moyen de stocker des informations d’identification et d’autres secrets avec une sécurité renforcée. Mais votre code a besoin de s’authentifier auprès de Key Vault pour les récupérer. Les [identités managées pour les ressources Azure](../../active-directory/managed-identities-azure-resources/overview.md) facilitent la résolution de ce problème en fournissant aux services Azure une identité managée automatiquement dans Azure AD (Azure Active Directory). Vous pouvez utiliser cette identité pour vous authentifier sur n’importe quel service prenant en charge l’authentification Azure AD, y compris Key Vault, sans avoir à afficher les informations d’identification dans votre code.

Dans ce tutoriel, vous allez utiliser une identité managée pour authentifier une application web Azure auprès d’un coffre de clés Azure. Vous allez utiliser la [bibliothèque de client de secrets Azure Key Vault Secret pour .NET](/dotnet/api/overview/azure/key-vault) et [Azure CLI](/cli/azure/get-started-with-azure-cli). Les mêmes principes de base s’appliquent quand vous utilisez le langage de développement de votre choix, Azure PowerShell et/ou le portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce démarrage rapide, les éléments suivants sont requis :

* Un abonnement Azure. [Créez-en un gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Le [SDK .NET Core 3.1 (ou version ultérieure)](https://dotnet.microsoft.com/download/dotnet-core/3.1).
* Une installation [Git](https://www.git-scm.com/downloads).
* [Azure CLI](/cli/azure/install-azure-cli) ou [Azure PowerShell](/powershell/azure/).
* [Azure Key Vault](./overview.md). Vous pouvez créer un coffre de clés à l’aide du [portail Azure](quick-create-portal.md), d’[Azure CLI](quick-create-cli.md) ou d’[Azure PowerShell](quick-create-powershell.md).
* Un [secret](../secrets/about-secrets.md) de coffre de clés. Vous pouvez créer un secret avec le [portail Azure](../secrets/quick-create-portal.md), [PowerShell](../secrets/quick-create-powershell.md) ou [Azure CLI](../secrets/quick-create-cli.md).

## <a name="create-a-net-core-app"></a>Créer une application .NET Core
Cette étape consiste à configurer le projet .NET Core local.

Dans une fenêtre de terminal sur votre machine, créez un répertoire nommé `akvwebapp` et faites-en le répertoire actuel.

```bash
mkdir akvwebapp
cd akvwebapp
```

Créez une application .NET Core à l’aide de la commande [dotnet new web](/dotnet/core/tools/dotnet-new) :

```bash
dotnet new web
```

Exécutez l’application localement pour voir à quoi elle doit ressembler quand vous la déployez sur Azure :

```bash
dotnet run
```

Dans un navigateur web, accédez à l’application à l’adresse `http://localhost:5000`.

Vous voyez apparaître le message « Hello World! » de l’exemple d’application.

## <a name="deploy-the-app-to-azure"></a>Déploiement de l’application dans Azure

Dans cette étape, vous allez déployer votre application .NET Core sur Azure App Service à l’aide de Git local. Pour plus d’informations sur la création et le déploiement d’applications, consultez [Créer une application web ASP.NET Core dans Azure](../../app-service/quickstart-dotnetcore.md).

### <a name="configure-the-local-git-deployment"></a>Configurer le déploiement Git local

Dans la fenêtre de terminal, sélectionnez **Ctrl+C** pour fermer le serveur web.  Initialisez un dépôt Git pour le projet .NET Core :

```bash
git init
git add .
git commit -m "first commit"
```

Vous pouvez utiliser le protocole FTP et Git local pour déployer une application web Azure en faisant appel à un *utilisateur de déploiement*. Une fois que vous avez créé votre utilisateur de déploiement, vous pouvez l’utiliser pour tous vos déploiements Azure. Votre nom d’utilisateur et votre mot de passe de déploiement au niveau du compte sont différents de vos informations d’identification de l’abonnement Azure. 

Pour configurer l’utilisateur de déploiement, exécutez la commande [az webapp deployment user set](/cli/azure/webapp/deployment/user?#az-webapp-deployment-user-set). Choisissez un nom d’utilisateur et un mot de passe conformes à ces instructions : 

- Le nom d’utilisateur doit être unique au sein de Azure. Pour les envois (push) Git locaux, il ne peut pas contenir d’arobase (@). 
- Le mot de passe doit comporter au moins huit caractères et inclure deux des trois éléments suivants : lettres, chiffres et symboles. 

```azurecli-interactive
az webapp deployment user set --user-name "<username>" --password "<password>"
```

La sortie JSON affiche le mot de passe comme étant `null`. Si vous obtenez une erreur `'Conflict'. Details: 409`, modifiez le nom d’utilisateur. Si vous obtenez une erreur `'Bad Request'. Details: 400`, utilisez un mot de passe plus fort. 

Enregistrez votre nom d’utilisateur et votre mot de passe afin de pouvoir les utiliser pour déployer vos applications web.

### <a name="create-a-resource-group"></a>Créer un groupe de ressources

Un groupe de ressources est un conteneur logique dans lequel vous déployez et gérez des ressources Azure. Créez un groupe de ressources pour y mettre à la fois votre coffre de clés et votre application web à l’aide de la commande [az group create](/cli/azure/group?#az-group-create) :

```azurecli-interactive
az group create --name "myResourceGroup" -l "EastUS"
```

### <a name="create-an-app-service-plan"></a>Créer un plan App Service

Créez un [plan App Service](../../app-service/overview-hosting-plans.md) avec la commande [az appservice plan create](/cli/azure/appservice/plan) d’Azure CLI. Cet exemple crée un plan App Service nommé `myAppServicePlan` dans le niveau tarifaire `FREE` :

```azurecli-interactive
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku FREE
```

Quand le plan App Service est créé, Azure CLI affiche des informations similaires à celles que vous voyez ici :

<pre>
{ 
  "adminSiteName": null,
  "appServicePlanName": "myAppServicePlan",
  "geoRegion": "West Europe",
  "hostingEnvironmentProfile": null,
  "id": "/subscriptions/0000-0000/resourceGroups/myResourceGroup/providers/Microsoft.Web/serverfarms/myAppServicePlan",
  "kind": "app",
  "location": "West Europe",
  "maximumNumberOfWorkers": 1,
  "name": "myAppServicePlan",
  &lt; JSON data removed for brevity. &gt;
  "targetWorkerSizeId": 0,
  "type": "Microsoft.Web/serverfarms",
  "workerTierName": null
} 
</pre>

Pour plus d’informations, consultez [Gérer un plan App Service dans Azure](../../app-service/app-service-plan-manage.md).

### <a name="create-a-web-app"></a>Créer une application web

Créez une [application web Azure](../../app-service/overview.md) dans le plan App Service `myAppServicePlan`. 

> [!Important]
> Comme un coffre de clés, une application web Azure doit porter un nom unique. Remplacez `<your-webapp-name>` par le nom de votre application web dans les exemples suivants.


```azurecli-interactive
az webapp create --resource-group "myResourceGroup" --plan "myAppServicePlan" --name "<your-webapp-name>" --deployment-local-git
```

Quand l’application web est créée, Azure CLI présente une sortie similaire à celle que vous voyez ici :

<pre>
Local git is configured with url of 'https://&lt;username&gt;@&lt;your-webapp-name&gt;.scm.azurewebsites.net/&lt;ayour-webapp-name&gt;.git'
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "clientCertExclusionPaths": null,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "&lt;your-webapp-name&gt;.azurewebsites.net",
  "deploymentLocalGitUrl": "https://&lt;username&gt;@&lt;your-webapp-name&gt;.scm.azurewebsites.net/&lt;your-webapp-name&gt;.git",
  "enabled": true,
  &lt; JSON data removed for brevity. &gt;
}
</pre>


L’URL du Git distant est indiquée dans la propriété `deploymentLocalGitUrl`, au format `https://<username>@<your-webapp-name>.scm.azurewebsites.net/<your-webapp-name>.git`. Enregistrez cette URL. Vous en aurez besoin ultérieurement.

Accédez à votre nouvelle application en utilisant la commande suivante. Remplacez `<your-webapp-name>` par le nom de votre application.

```bash
https://<your-webapp-name>.azurewebsites.net
```

Vous allez voir la page web par défaut d’une nouvelle application web Azure.

### <a name="deploy-your-local-app"></a>Déployer votre application locale

De retour dans la fenêtre de terminal locale, ajoutez un dépôt distant Azure dans votre dépôt Git local. Dans la commande suivante, remplacez `<deploymentLocalGitUrl-from-create-step>` par l’URL du Git distant que vous avez enregistré dans la section [Créer une application web](#create-a-web-app).

```bash
git remote add azure <deploymentLocalGitUrl-from-create-step>
```

Utilisez la commande suivante pour effectuer un envoi (push) au dépôt distant Azure afin de déployer votre application. Quand le gestionnaire d’informations d’identification de Git vous invite à entrer des informations d’identification, utilisez celles que vous avez dans la section [Configurer le déploiement Git local](#configure-the-local-git-deployment).

```bash
git push azure main
```

Cette commande peut prendre quelques minutes pour s’exécuter. Pendant son exécution, elle présente des informations semblables à celles que vous voyez ici :
<pre>
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 285 bytes | 95.00 KiB/s, done.
Total 3 (delta 2), reused 0 (delta 0), pack-reused 0
remote: Deploy Async
remote: Updating branch 'main'.
remote: Updating submodules.
remote: Preparing deployment for commit id 'd6b54472f7'.
remote: Repository path is /home/site/repository
remote: Running oryx build...
remote: Build orchestrated by Microsoft Oryx, https://github.com/Microsoft/Oryx
remote: You can report issues at https://github.com/Microsoft/Oryx/issues
remote:
remote: Oryx Version      : 0.2.20200114.13, Commit: 204922f30f8e8d41f5241b8c218425ef89106d1d, ReleaseTagName: 20200114.13
remote: Build Operation ID: |imoMY2y77/s=.40ca2a87_
remote: Repository Commit : d6b54472f7e8e9fd885ffafaa64522e74cf370e1
.
.
.
remote: Deployment successful.
remote: Deployment Logs : 'https://&lt;your-webapp-name&gt;.scm.azurewebsites.net/newui/jsonviewer?view_url=/api/deployments/d6b54472f7e8e9fd885ffafaa64522e74cf370e1/log'
To https://&lt;your-webapp-name&gt;.scm.azurewebsites.net:443/&lt;your-webapp-name&gt;.git
   d87e6ca..d6b5447  main -> main
</pre>

Accédez à (ou actualisez) l’application déployée à l’aide de votre navigateur web :

```bash
http://<your-webapp-name>.azurewebsites.net
```

Vous voyez apparaître le message « Hello World! » que vous avez vu précédemment quand vous avez accédé à `http://localhost:5000`.
 
## <a name="configure-the-web-app-to-connect-to-key-vault"></a>Configurer l’application web pour se connecter au coffre de clés

Dans cette section, vous allez configurer l’accès web au coffre de clés et mettre à jour le code de votre application pour récupérer un secret auprès de Key Vault.

### <a name="create-and-assign-a-managed-identity"></a>Créer et attribuer une identité gérée

Dans ce tutoriel, nous allons utiliser une [identité managée](../../active-directory/managed-identities-azure-resources/overview.md) pour l’authentification auprès de Key Vault. Une identité managée gère automatiquement les informations d’identification de l’application.

Dans Azure CLI, pour créer l’identité de l’application, exécutez la commande [az webapp-identity assign](/cli/azure/webapp/identity?#az-webapp-identity-assign) :

```azurecli-interactive
az webapp identity assign --name "<your-webapp-name>" --resource-group "myResourceGroup"
```

La commande retourne cet extrait de code JSON :

```json
{
  "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "SystemAssigned"
}
```

Pour permettre à votre application web d’effectuer des opérations **get** et **list** sur votre coffre de clés, passez `principalId` à la commande Azure CLI [az keyvault set-policy](/cli/azure/keyvault?#az-keyvault-set-policy) :

```azurecli-interactive
az keyvault set-policy --name "<your-keyvault-name>" --object-id "<principalId>" --secret-permissions get list
```

Vous pouvez également affecter des stratégies d’accès à l’aide du [portail Azure](./assign-access-policy-portal.md) ou de [PowerShell](./assign-access-policy-powershell.md).

### <a name="modify-the-app-to-access-your-key-vault"></a>Modifier l’application pour accéder à votre coffre de clés

Dans ce tutoriel, vous allez utiliser la [bibliothèque de client de secrets Azure Key Vault](https://docs.microsoft.com/dotnet/api/overview/azure/security.keyvault.secrets-readme) à des fins de démonstration. Vous pouvez également utiliser la [bibliothèque de client de certificats Azure Key Vault](https://docs.microsoft.com/dotnet/api/overview/azure/security.keyvault.certificates-readme) ou la [bibliothèque de client de clés Azure Key Vault](https://docs.microsoft.com/dotnet/api/overview/azure/security.keyvault.keys-readme).

#### <a name="install-the-packages"></a>Installer les packages

Dans la fenêtre de terminal, installez la bibliothèque de client de secrets Azure Key Vault pour .NET et les packages de la bibliothèque de client d’identités Azure :

```console
dotnet add package Azure.Identity
dotnet add package Azure.Security.KeyVault.Secrets
```

#### <a name="update-the-code"></a>Mettez à jour le code

Recherchez et ouvrez le fichier Startup.cs dans votre projet akvwebapp. 

Ajoutez ces lignes à l’en-tête :

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;
using Azure.Core;
```

Ajoutez les lignes suivantes avant l’appel de `app.UseEndpoints`, en mettant à jour l’URI pour refléter la valeur `vaultUri` de votre coffre de clés. Ce code utilise [DefaultAzureCredential()](/dotnet/api/azure.identity.defaultazurecredential) pour l’authentification auprès du coffre de clés, qui utilise un jeton de l’identité managée pour s’authentifier. Pour plus d’informations sur l’authentification auprès du coffre de clés, consultez le [Guide du développeur](./developers-guide.md#authenticate-to-key-vault-in-code). Le code utilise également un backoff exponentiel pour les nouvelles tentatives en cas de limitation du coffre de clés. Pour plus d’informations sur les limites de transaction du coffre de clés, consultez [Aide sur la limitation de requêtes Azure Key Vault](./overview-throttling.md).

```csharp
SecretClientOptions options = new SecretClientOptions()
    {
        Retry =
        {
            Delay= TimeSpan.FromSeconds(2),
            MaxDelay = TimeSpan.FromSeconds(16),
            MaxRetries = 5,
            Mode = RetryMode.Exponential
         }
    };
var client = new SecretClient(new Uri("https://<your-unique-key-vault-name>.vault.azure.net/"), new DefaultAzureCredential(),options);

KeyVaultSecret secret = client.GetSecret("<mySecret>");

string secretValue = secret.Value;
```

Mettez à jour la ligne `await context.Response.WriteAsync("Hello World!");` pour qu’elle ressemble à ceci :

```csharp
await context.Response.WriteAsync(secretValue);
```

Veillez à enregistrer vos modifications avant de passer à l’étape suivante.

#### <a name="redeploy-your-web-app"></a>Redéployez votre application web

Maintenant que vous avez mis à jour votre code, vous pouvez le redéployer sur Azure à l’aide de ces commandes Git :

```bash
git add .
git commit -m "Updated web app to access my key vault"
git push azure main
```

## <a name="go-to-your-completed-web-app"></a>Accéder à votre application web terminée

```bash
http://<your-webapp-name>.azurewebsites.net
```

Là où « Hello World » s’affichait, vous devez maintenant voir la valeur de votre secret.

## <a name="next-steps"></a>Étapes suivantes

- [Utiliser Azure Key Vault avec des applications déployées sur une machine virtuelle dans .NET](./tutorial-net-virtual-machine.md)
- En savoir plus sur les [identités managées pour les ressources Azure](../../active-directory/managed-identities-azure-resources/overview.md)
- Consulter le [Guide du développeur](./developers-guide.md)
- [Sécuriser l’accès à un coffre de clés](./secure-your-key-vault.md)

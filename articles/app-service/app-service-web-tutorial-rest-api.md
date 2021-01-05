---
title: 'Tutoriel : Héberger des API RESTful avec CORS'
description: Découvrir comment Azure App Service vous aide à héberger vos API RESTful avec prise en charge de CORS. App Service peut héberger des applications web front-end et des API back-end.
ms.assetid: a820e400-06af-4852-8627-12b3db4a8e70
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 04/28/2020
ms.custom: devx-track-csharp, mvc, devcenter, seo-javascript-september2019, seo-javascript-october2019, seodec18
ms.openlocfilehash: 9481b6d2740d27b8c3d1309e205edda6017868fa
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005745"
---
# <a name="tutorial-host-a-restful-api-with-cors-in-azure-app-service"></a>Tutoriel : Héberger une API RESTful avec CORS dans Azure App Service

[Azure App Service](overview.md) offre un service d’hébergement web hautement évolutif appliquant des mises à jour correctives automatiques. De plus, App Service intègre la prise en charge du [partage des ressources cross-origin (CORS)](https://wikipedia.org/wiki/Cross-Origin_Resource_Sharing) pour les API RESTful. Ce didacticiel montre comment déployer une application ASP.NET Core API sur App Service avec prise en charge CORS. Vous configurez l’application à l’aide d’outils en ligne de commande, et vous la déployez à l’aide de Git. 

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer des ressources App Service à l’aide d’Azure CLI
> * Déployer une API RESTful sur Azure à l’aide de Git
> * Activer la prise en charge de CORS sur App Service

Vous pouvez suivre les étapes de ce tutoriel sur macOS, Linux, Windows.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel :

* <a href="https://git-scm.com/" target="_blank">Installez Git</a>
 * <a href="https://dotnet.microsoft.com/download/dotnet-core/3.1" target="_blank">Installer la dernière version du kit SDK .NET Core 3.1</a>

## <a name="create-local-aspnet-core-app"></a>Créer l’application ASP.NET Core locale

Cette étape consiste à configurer le projet ASP.NET Core local. App Service prend en charge le même flux de travail pour les API écrites dans d’autres langages.

### <a name="clone-the-sample-application"></a>Clonage de l’exemple d’application

Dans la fenêtre de terminal, `cd` vers un répertoire de travail.  

Exécutez la commande suivante pour cloner l’exemple de référentiel : 

```bash
git clone https://github.com/Azure-Samples/dotnet-core-api
```

Ce référentiel contient une application créée en fonction du didacticiel suivant : [Pages d’aide d’API web ASP.NET Core avec Swagger/OpenAPI](/aspnet/core/tutorials/web-api-help-pages-using-swagger?tabs=visual-studio). Elle utilise un générateur de Swagger pour traiter l’[interface utilisateur Swagger](https://swagger.io/swagger-ui/) et le point de terminaison JSON de Swagger.

### <a name="run-the-application"></a>Exécution de l'application

Exécutez la commande suivante pour installer les packages requis, migrer les bases de données et démarrer l’application.

```bash
cd dotnet-core-api
dotnet restore
dotnet run
```

Accédez à `http://localhost:5000/swagger` dans un navigateur pour vous familiariser avec l’interface utilisateur Swagger.

![API ASP.NET Core exécuté en local](./media/app-service-web-tutorial-rest-api/azure-app-service-local-swagger-ui.png)

Accédez à `http://localhost:5000/api/todo` pour visualiser une liste des tâches JSON.

Accédez à `http://localhost:5000` et familiarisez-vous avec l’application de navigateur. Ensuite, vous allez pointer l’application de navigateur vers une API distante dans l’App Service pour tester la fonctionnalité CORS. Le code pour l’application de navigateur se trouve dans le répertoire _wwwroot_ du référentiel.

Pour arrêter ASP.NET Core à tout moment, appuyez sur `Ctrl+C` dans le terminal.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="deploy-app-to-azure"></a>Déployer des applications dans Azure

Dans cette étape, vous déployez votre application .NET Core connectée à SQL Database sur App Service.

### <a name="configure-local-git-deployment"></a>Configurer le déploiement Git local

[!INCLUDE [Configure a deployment user](../../includes/configure-deployment-user-no-h.md)]

### <a name="create-a-resource-group"></a>Créer un groupe de ressources

[!INCLUDE [Create resource group](../../includes/app-service-web-create-resource-group-no-h.md)]

### <a name="create-an-app-service-plan"></a>Créer un plan App Service

[!INCLUDE [Create app service plan](../../includes/app-service-web-create-app-service-plan-no-h.md)]

### <a name="create-a-web-app"></a>Créer une application web

[!INCLUDE [Create web app](../../includes/app-service-web-create-web-app-dotnetcore-win-no-h.md)] 

### <a name="push-to-azure-from-git"></a>Effectuer une transmission de type push vers Azure à partir de Git

[!INCLUDE [app-service-plan-no-h](../../includes/app-service-web-git-push-to-azure-no-h.md)]

<pre>
Enumerating objects: 83, done.
Counting objects: 100% (83/83), done.
Delta compression using up to 8 threads
Compressing objects: 100% (78/78), done.
Writing objects: 100% (83/83), 22.15 KiB | 3.69 MiB/s, done.
Total 83 (delta 26), reused 0 (delta 0)
remote: Updating branch 'master'.
remote: Updating submodules.
remote: Preparing deployment for commit id '509236e13d'.
remote: Generating deployment script.
remote: Project file path: .\TodoApi.csproj
remote: Generating deployment script for ASP.NET MSBuild16 App
remote: Generated deployment script files
remote: Running deployment command...
remote: Handling ASP.NET Core Web Application deployment with MSBuild16.
remote: .
remote: .
remote: .
remote: Finished successfully.
remote: Running post deployment command(s)...
remote: Triggering recycle (preview mode disabled).
remote: Deployment successful.
To https://&lt;app_name&gt;.scm.azurewebsites.net/&lt;app_name&gt;.git
 * [new branch]      master -> master
</pre>

### <a name="browse-to-the-azure-app"></a>Accéder à l’application Azure

Accédez à `http://<app_name>.azurewebsites.net/swagger` dans un navigateur et familiarisez-vous avec l’interface utilisateur Swagger.

![API ASP.NET Core exécuté dans Azure App Service](./media/app-service-web-tutorial-rest-api/azure-app-service-browse-app.png)

Accédez à `http://<app_name>.azurewebsites.net/swagger/v1/swagger.json` pour voir le _swagger.json_ de l’API déployée.

Accédez à `http://<app_name>.azurewebsites.net/api/todo` pour voir l’API déployée en fonctionnement.

## <a name="add-cors-functionality"></a>Ajoutez des fonctionnalités CORS

Ensuite, activez la prise en charge intégrée de CORS dans App Service pour votre API.

### <a name="test-cors-in-sample-app"></a>Testez CORS dans l’exemple d’application

Dans votre référentiel local, ouvrez _wwwroot/index.html_.

Dans la ligne 51, définissez la variable `apiEndpoint` sur l’URL de l’API déployée (`http://<app_name>.azurewebsites.net`). Remplacez _\<appname>_ par le nom de votre application dans App Service.

Dans la fenêtre de votre terminal local, exécutez à nouveau l’exemple d’application.

```bash
dotnet run
```

Accédez à l’application de navigateur à `http://localhost:5000`. Ouvrez la fenêtre d’outils de développement dans votre navigateur (`Ctrl`+`Shift`+`i` dans Chrome pour Windows) et inspectez l’onglet **Console**. Vous devriez maintenant voir le message d’erreur, `No 'Access-Control-Allow-Origin' header is present on the requested resource`.

![Erreur CORS dans le navigateur client](./media/app-service-web-tutorial-rest-api/azure-app-service-cors-error.png)

En raison de l’incompatibilité de domaine entre l’application de navigateur (`http://localhost:5000`) et la ressource à distance (`http://<app_name>.azurewebsites.net`), et du fait que votre API dans App Service n’envoie pas l’en-tête `Access-Control-Allow-Origin`, votre navigateur a empêché le chargement du contenu inter-domaines dans votre application de navigateur.

En production, votre application de navigateur aurait une URL publique au lieu de l’URL de l’hôte local, mais la façon d’activer CORS est identique pour les deux types d’URL.

### <a name="enable-cors"></a>Activez CORS 

Dans Cloud Shell, activez CORS pour votre URL de client à l’aide de la commande [`az webapp cors add`](/cli/azure/webapp/cors#az-webapp-cors-add). Remplacez l’espace réservé _&lt;app-name>_ .

```azurecli-interactive
az webapp cors add --resource-group myResourceGroup --name <app-name> --allowed-origins 'http://localhost:5000'
```

Vous pouvez définir plusieurs URL de client dans `properties.cors.allowedOrigins` (`"['URL1','URL2',...]"`). Vous pouvez également activer toutes les URL client avec `"['*']"`.

> [!NOTE]
> Si votre application exige l’envoi d’informations d’identification, telles que des cookies ou des jetons d’authentification, le navigateur peut exiger l’en-tête `ACCESS-CONTROL-ALLOW-CREDENTIALS` dans la réponse. Pour ce faire, dans App Service, définissez `properties.cors.supportCredentials` sur `true` dans votre configuration CORS. Cela ne peut pas être activé lorsque `allowedOrigins` inclut `'*'`.

### <a name="test-cors-again"></a>Testez CORS à nouveau

Actualisez l’application de navigateur à `http://localhost:5000`. Le message d’erreur dans la fenêtre **Console** a désormais disparu, et vous pouvez afficher les données de l’API déployée et interagir avec elles. Votre API distante prend désormais en charge CORS vers votre application de navigateur exécuté en local. 

![Succès de CORS dans le navigateur client](./media/app-service-web-tutorial-rest-api/azure-app-service-cors-success.png)

Félicitations, vous exécutez une API dans Azure App Service avec prise en charge de CORS.

## <a name="app-service-cors-vs-your-cors"></a>Le CORS App Service et votre CORS

Vous pouvez utiliser vos propres utilitaires CORS au lieu de CORS App Service pour plus de souplesse. Par exemple, vous souhaitez peut-être spécifier différentes origines autorisées pour des méthodes ou des itinéraires différents. Étant donné que CORS App Service vous permet de spécifier un ensemble d’origines acceptées pour tous les itinéraires et méthodes de l’API, vous pouvez utiliser votre propre code CORS. ASP.NET Core [L’activation de demandes de Cross-Origin (CORS)](/aspnet/core/security/cors).

> [!NOTE]
> N’essayez pas d’utiliser conjointement CORS App Service et votre propre code CORS. Utilisés conjointement, CORS App Service est prioritaire et votre propre code CORS n’a aucun effet.
>
>

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

<a name="next"></a>
## <a name="next-steps"></a>Étapes suivantes

Vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer des ressources App Service à l’aide d’Azure CLI
> * Déployer une API RESTful sur Azure à l’aide de Git
> * Activer la prise en charge de CORS sur App Service

Passez au tutoriel suivant pour découvrir comment authentifier et autoriser des utilisateurs.

> [!div class="nextstepaction"]
> [Tutoriel : Authentifier et autoriser des utilisateurs de bout en bout](tutorial-auth-aad.md)

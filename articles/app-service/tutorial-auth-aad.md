---
title: 'Tutoriel : Authentifier les utilisateurs E2E'
description: Apprenez à utiliser les fonctionnalités d’authentification et d’autorisation d’App Service pour sécuriser vos applications App Service de bout en bout (E2E), notamment l’accès aux API distantes.
keywords: app service, azure app service, authentification, autorisation, sécurisé, sécurité, plusieurs niveaux, azure active directory, azure ad
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 04/29/2020
ms.custom: devx-track-csharp, seodec18, devx-track-azurecli
zone_pivot_groups: app-service-platform-windows-linux
ms.openlocfilehash: 8c3fca6cf7782c3aaac91388a8f8395e288f5ea5
ms.sourcegitcommit: 65a4f2a297639811426a4f27c918ac8b10750d81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96558740"
---
# <a name="tutorial-authenticate-and-authorize-users-end-to-end-in-azure-app-service"></a>Tutoriel : Authentifier et autoriser des utilisateurs de bout en bout dans Azure App Service

::: zone pivot="platform-windows"  

[Azure App Service](overview.md) offre un service d’hébergement web hautement évolutif appliquant des mises à jour correctives automatiques. En outre, App Service prend, de base, en charge [l’authentification et l’autorisation des utilisateurs](overview-authentication-authorization.md). Ce tutoriel montre comment sécuriser vos applications avec l’authentification et l’autorisation App Service. Il utilise comme exemple une application ASP.NET Core avec un front-end Angular.js. L’authentification et l’autorisation App Service prennent en charge tous les runtimes de langage, et vous pouvez apprendre comment les appliquer à votre langage préféré en suivant le tutoriel.

::: zone-end

::: zone pivot="platform-linux"

[Azure App Service](overview.md) fournit un service d’hébergement web hautement évolutif appliquant des mises à jour correctives automatiques à l’aide du système d’exploitation Linux. En outre, App Service prend, de base, en charge [l’authentification et l’autorisation des utilisateurs](overview-authentication-authorization.md). Ce tutoriel montre comment sécuriser vos applications avec l’authentification et l’autorisation App Service. Il utilise comme exemple une application ASP.NET Core avec un front-end Angular.js. L’authentification et l’autorisation App Service prennent en charge tous les runtimes de langage, et vous pouvez apprendre comment les appliquer à votre langage préféré en suivant le tutoriel.

::: zone-end

![Authentification et autorisation simples](./media/tutorial-auth-aad/simple-auth.png)

Il vous montre également comment sécuriser une application multiniveau, en accédant à une API de serveur principal sécurisée pour le compte de l’utilisateur authentifié, [à partir du code du serveur](#call-api-securely-from-server-code) et [à partir du code du navigateur](#call-api-securely-from-browser-code).

![Authentification et autorisation avancées](./media/tutorial-auth-aad/advanced-auth.png)

Il s’agit uniquement de quelques scénarios d’authentification et d’autorisation possibles dans App Service. 

Voici une liste plus complète de ce que vous allez apprendre dans ce tutoriel :

> [!div class="checklist"]
> * Activer l’authentification et l’autorisation intégrées
> * Sécuriser des applications contre des requêtes non authentifiées
> * Utiliser Azure Active Directory en tant que fournisseur d’identité
> * Accéder à une application à distance pour le compte de l’utilisateur connecté
> * Sécuriser des appels entre des services avec un jeton d’authentification
> * Utiliser des jetons d’accès à partir du code du serveur
> * Utiliser des jetons d’accès à partir du code du client (navigateur)

Vous pouvez suivre les étapes de ce tutoriel sur macOS, Linux, Windows.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel :

- <a href="https://git-scm.com/" target="_blank">Installez Git</a>
- <a href="https://dotnet.microsoft.com/download/dotnet-core/3.1" target="_blank">Installez la dernière version du kit SDK .NET Core 3.1</a>
[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="create-local-net-core-app"></a>Création de l’application .NET Core locale

Cette étape consiste à configurer le projet .NET Core local. Vous utilisez le même projet pour déployer une application API principale et une application web frontale.

### <a name="clone-and-run-the-sample-application"></a>Cloner et exécuter l’exemple d’application

Exécutez les commandes suivantes pour cloner l’exemple de référentiel et l’exécuter.

```bash
git clone https://github.com/Azure-Samples/dotnet-core-api
cd dotnet-core-api
dotnet run
```

Accédez à `http://localhost:5000` et essayez d’ajouter, de modifier et de supprimer des éléments de liste de tâches. 

![API ASP.NET Core exécuté en local](./media/tutorial-auth-aad/local-run.png)

Pour arrêter ASP.NET Core à tout moment, appuyez sur `Ctrl+C` dans le terminal.

## <a name="deploy-apps-to-azure"></a>Déployer des applications dans Azure

Dans cette étape, vous déployez le projet dans deux applications App Service. La première est l’application frontale, et la deuxième est l’application principale.

### <a name="configure-a-deployment-user"></a>Configuration d’un utilisateur de déploiement

[!INCLUDE [Configure deployment user](../../includes/configure-deployment-user-no-h.md)]

### <a name="create-azure-resources"></a>Créer des ressources Azure

::: zone pivot="platform-windows"  

Dans Cloud Shell, exécutez les commandes suivantes pour créer deux applications web Windows. Remplacez _\<front-end-app-name>_ et _\<back-end-app-name>_ par deux noms d’applications globaux uniques (les caractères valides sont `a-z`, `0-9` et `-`). Pour plus d’informations sur chaque commande, consultez [Héberger une API RESTful avec CORS dans Azure App Service](app-service-web-tutorial-rest-api.md).

```azurecli-interactive
az group create --name myAuthResourceGroup --location "West Europe"
az appservice plan create --name myAuthAppServicePlan --resource-group myAuthResourceGroup --sku FREE
az webapp create --resource-group myAuthResourceGroup --plan myAuthAppServicePlan --name <front-end-app-name> --deployment-local-git --query deploymentLocalGitUrl
az webapp create --resource-group myAuthResourceGroup --plan myAuthAppServicePlan --name <back-end-app-name> --deployment-local-git --query deploymentLocalGitUrl
```

::: zone-end

::: zone pivot="platform-linux"

Dans Cloud Shell, exécutez les commandes suivantes pour créer deux applications web. Remplacez _\<front-end-app-name>_ et _\<back-end-app-name>_ par deux noms d’applications globaux uniques (les caractères valides sont `a-z`, `0-9` et `-`). Pour plus d’informations sur chaque commande, consultez [Créer une application .NET Core dans Azure App Service](quickstart-dotnetcore.md).

```azurecli-interactive
az group create --name myAuthResourceGroup --location "West Europe"
az appservice plan create --name myAuthAppServicePlan --resource-group myAuthResourceGroup --sku FREE --is-linux
az webapp create --resource-group myAuthResourceGroup --plan myAuthAppServicePlan --name <front-end-app-name> --runtime "DOTNETCORE|3.1" --deployment-local-git --query deploymentLocalGitUrl
az webapp create --resource-group myAuthResourceGroup --plan myAuthAppServicePlan --name <back-end-app-name> --runtime "DOTNETCORE|3.1" --deployment-local-git --query deploymentLocalGitUrl
```

::: zone-end

> [!NOTE]
> Enregistrez les URL des référentiels Git distants pour votre application frontale et votre application principale, qui sont affichées dans la sortie de `az webapp create`.
>

### <a name="push-to-azure-from-git"></a>Effectuer une transmission de type push vers Azure à partir de Git

Dans la _fenêtre de terminal local_, exécutez les commandes Git suivantes pour effectuer le déploiement vers l’application principale. Remplacez _\<deploymentLocalGitUrl-of-back-end-app>_ par l’URL du dépôt Git distant que vous avez enregistrée à la section [Créer des ressources Azure](#create-azure-resources). Lorsque vous êtes invité à saisir les informations d’identification par Git Credential Manager, assurez-vous d’entrer [vos informations d’identification de déploiement](deploy-configure-credentials.md), et non celles que vous utilisez pour vous connecter au portail Azure.

```bash
git remote add backend <deploymentLocalGitUrl-of-back-end-app>
git push backend master
```

Dans la fenêtre de terminal local, exécutez les commandes Git suivantes pour déployer le même code vers l’application frontale. Remplacez _\<deploymentLocalGitUrl-of-front-end-app>_ par l’URL du dépôt Git distant que vous avez enregistrée à la section [Créer des ressources Azure](#create-azure-resources).

```bash
git remote add frontend <deploymentLocalGitUrl-of-front-end-app>
git push frontend master
```

### <a name="browse-to-the-apps"></a>Accéder aux applications

Accédez aux URL suivantes dans un navigateur et observez les deux applications en action.

```
http://<back-end-app-name>.azurewebsites.net
http://<front-end-app-name>.azurewebsites.net
```

:::image type="content" source="./media/tutorial-auth-aad/azure-run.png" alt-text="Capture d’écran d’un exemple d’API REST Azure App Service dans une fenêtre de navigateur, qui présente une application de liste de tâches.":::

> [!NOTE]
> En cas de redémarrage de votre application, vous avez peut-être remarqué que de nouvelles données ont été effacées. Ce comportement par défaut se déclenche en raison de l’utilisation d’une base de données en mémoire par l’application ASP.NET Core.
>
>

## <a name="call-back-end-api-from-front-end"></a>Appeler une API principale du serveur frontal

Dans cette étape, vous pointez le code du serveur de l’application frontale pour accéder à l’API principale. Vous activerez ultérieurement l’accès authentifié à partir du serveur frontal vers le serveur principal.

### <a name="modify-front-end-code"></a>Modifier le code du serveur frontal

Dans le référentiel local, ouvrez _Controllers/TodoController.cs_. Au début de la classe `TodoController`, ajoutez les lignes suivantes et remplacez _\<back-end-app-name>_ par le nom de votre application back-end :

```cs
private static readonly HttpClient _client = new HttpClient();
private static readonly string _remoteUrl = "https://<back-end-app-name>.azurewebsites.net";
```

Recherchez la méthode dotée de `[HttpGet]` et remplacez le code à l’intérieur des accolades par :

```cs
var data = await _client.GetStringAsync($"{_remoteUrl}/api/Todo");
return JsonConvert.DeserializeObject<List<TodoItem>>(data);
```

La première ligne effectue un appel `GET /api/Todo` à l’application d’API principale.

Ensuite, recherchez la méthode dotée de `[HttpGet("{id}")]` et remplacez le code à l’intérieur des accolades par :

```cs
var data = await _client.GetStringAsync($"{_remoteUrl}/api/Todo/{id}");
return Content(data, "application/json");
```

La première ligne effectue un appel `GET /api/Todo/{id}` à l’application d’API principale.

Ensuite, recherchez la méthode dotée de `[HttpPost]` et remplacez le code à l’intérieur des accolades par :

```cs
var response = await _client.PostAsJsonAsync($"{_remoteUrl}/api/Todo", todoItem);
var data = await response.Content.ReadAsStringAsync();
return Content(data, "application/json");
```

La première ligne effectue un appel `POST /api/Todo` à l’application d’API principale.

Ensuite, recherchez la méthode dotée de `[HttpPut("{id}")]` et remplacez le code à l’intérieur des accolades par :

```cs
var res = await _client.PutAsJsonAsync($"{_remoteUrl}/api/Todo/{id}", todoItem);
return new NoContentResult();
```

La première ligne effectue un appel `PUT /api/Todo/{id}` à l’application d’API principale.

Ensuite, recherchez la méthode dotée de `[HttpDelete("{id}")]` et remplacez le code à l’intérieur des accolades par :

```cs
var res = await _client.DeleteAsync($"{_remoteUrl}/api/Todo/{id}");
return new NoContentResult();
```

La première ligne effectue un appel `DELETE /api/Todo/{id}` à l’application d’API principale.

Enregistrez toutes vos modifications. Dans la fenêtre de terminal local, déployez vos modifications à l’application frontale avec les commandes Git suivantes :

```bash
git add .
git commit -m "call back-end API"
git push frontend master
```

### <a name="check-your-changes"></a>Vérifier vos modifications

Accédez à `http://<front-end-app-name>.azurewebsites.net` et ajoutez quelques éléments, tels que `from front end 1` et `from front end 2`.

Accédez à `http://<back-end-app-name>.azurewebsites.net` pour voir les éléments ajoutés à partir de l’application frontale. Ajoutez également quelques éléments, tels que `from back end 1` et `from back end 2`, puis actualisez l’application frontale pour voir si elle reflète les modifications.

:::image type="content" source="./media/tutorial-auth-aad/remote-api-call-run.png" alt-text="Capture d’écran d’un exemple d’API REST Azure App Service dans une fenêtre de navigateur, qui montre une application de liste de tâches avec des éléments ajoutés à partir de l’application front-end.":::

## <a name="configure-auth"></a>Configurer l’authentification

Dans cette étape, vous activez l’authentification et l’autorisation pour les deux applications. Vous configurez également l’application frontale pour générer un jeton d’accès que vous pouvez utiliser pour effectuer des appels authentifiés à l’application principale.

Vous utilisez Azure Active Directory en tant que fournisseur d’identité. Pour plus d’informations, consultez [Configurer votre application App Service pour utiliser une connexion Azure Active Directory](configure-authentication-provider-aad.md).

### <a name="enable-authentication-and-authorization-for-back-end-app"></a>Activer l’authentification et l’autorisation pour l’application principale

Dans le menu du [portail Azure](https://portal.azure.com), sélectionnez **Groupes de ressources** ou recherchez et sélectionnez *Groupes de ressources* dans n’importe quelle page.

Dans **Groupes de ressources**, recherchez et sélectionnez votre groupe de ressources. Dans **Vue d’ensemble**, sélectionnez la page de gestion de votre application back-end.

:::image type="content" source="./media/tutorial-auth-aad/portal-navigate-back-end.png" alt-text="Capture d’écran de la fenêtre Groupes de ressources montrant la vue d’ensemble d’un exemple de groupe de ressources et la page de gestion de l’applications back-end sélectionnée.":::

Dans le menu de gauche de votre application back-end, sélectionnez **Authentification/Autorisation**, puis activez l’authentification App Service en sélectionnant **On** (Activé).

Dans **Mesure à prendre quand une demande n’est pas authentifiée**, sélectionnez **Se connecter avec Azure Active Directory**.

Sous **Fournisseurs d’authentification**, cliquez sur **Azure Active Directory**.

:::image type="content" source="./media/tutorial-auth-aad/configure-auth-back-end.png" alt-text="Capture d’écran du menu de gauche de l’application back-end montrant l’élément Authentification/Autorisation sélectionné et des paramètres sélectionnés dans le menu de droite.":::

Sélectionnez **Express**, puis acceptez les paramètres par défaut pour créer une application AD, et sélectionnez **OK**.

Sur la page **Authentification/Autorisation**, sélectionnez **Enregistrer**.

Lorsque vous voyez la notification avec le message `Successfully saved the Auth Settings for <back-end-app-name> App`, actualisez la page du portail.

Sélectionnez à nouveau **Azure Active Directory**, puis sélectionnez **Application Azure AD**.

Copiez l’**ID de client** de l’application Azure AD dans le Bloc-notes. Vous aurez besoin de cette valeur ultérieurement.

:::image type="content" source="./media/tutorial-auth-aad/get-application-id-back-end.png" alt-text="Capture d’écran de la fenêtre Paramètres Azure Active Directory montrant l’application Azure AD et la fenêtre Applications Azure AD montrant l’ID client à copier.":::

Si vous vous arrêtez ici, vous disposez d’une application autonome qui est déjà sécurisée par l’authentification et l’autorisation App Service. Les sections restantes vous montrent comment sécuriser une solution multi-application en « transmettant » l’utilisateur authentifié du front-end au back-end. 

### <a name="enable-authentication-and-authorization-for-front-end-app"></a>Activer l’authentification et l’autorisation pour l’application frontale

Suivez les mêmes étapes que pour l’application principale, mais ignorez la dernière étape. Vous n’avez pas besoin de l’ID de client pour l’application front-end.

Si vous le souhaitez, rendez-vous à l’adresse `http://<front-end-app-name>.azurewebsites.net`. Vous êtes à présent redirigé vers une page de connexion sécurisée. Une fois que vous êtes connecté, *vous ne pouvez toujours pas accéder aux données depuis l’application back-end* parce que celle-ci nécessite désormais la connexion Azure Active Directory à partir de l’application front-end. Vous devez effectuer trois opérations :

- Accorder l’accès du serveur frontal au serveur principal
- Configurer App Service pour renvoyer un jeton utilisable
- Utiliser le jeton dans votre code

> [!TIP]
> Si vous rencontrez des erreurs et reconfigurez les paramètres d’authentification/d’autorisation de votre application, les jetons dans le magasin de jetons ne peuvent pas être régénérés à partir des nouveaux paramètres. Pour vous assurer que vos jetons sont régénérés, vous devez vous déconnecter et vous reconnecter à votre application. Pour ce faire, vous pouvez simplement utiliser votre navigateur en mode privé, et fermer et ouvrir à nouveau le navigateur en mode privé après avoir modifié les paramètres dans vos applications.

### <a name="grant-front-end-app-access-to-back-end"></a>Accorder l’accès d’une application frontale au serveur principal

Maintenant que vous avez activé l’authentification et l’autorisation pour vos deux applications, chacune d’elles est soutenue par une application AD. Dans cette étape, vous donnez à l’application frontale les autorisations pour accéder au serveur principal pour le compte de l’utilisateur. (Techniquement, vous donnez à _l’application AD_ du serveur frontal les autorisations pour accéder à _l’application AD_ du serveur principal pour le compte de l’utilisateur.)

Dans le menu du [portail Azure](https://portal.azure.com), sélectionnez **Azure Active Directory** ou recherchez et sélectionnez *Azure Active Directory* dans n’importe quelle page.

Sélectionnez **Inscriptions d’applications** > **Applications détenues** > **Afficher toutes les applications de cet annuaire**. Sélectionnez le nom de votre application frontale, puis sélectionnez **Autorisations de l’API**.

:::image type="content" source="./media/tutorial-auth-aad/add-api-access-front-end.png" alt-text="Capture d’écran de la fenêtre Microsoft - Inscriptions d’applications avec Applications détenues, un nom d’application front-end et Autorisations d’API sélectionnés.":::

Sélectionnez **Ajouter une autorisation**, puis **API utilisées par mon organisation** >  **\<back-end-app-name>** .

Dans la page **Demander des autorisations d’API** pour l’application back-end, sélectionnez **Autorisations déléguées** et **user_impersonation**, puis sélectionnez **Ajouter des autorisations**.

:::image type="content" source="./media/tutorial-auth-aad/select-permission-front-end.png" alt-text="Capture d’écran de la page Demander des autorisations d’API montrant Autorisations déléguées, user_impersonation et le bouton Ajouter une autorisation sélectionnés.":::

### <a name="configure-app-service-to-return-a-usable-access-token"></a>Configurer App Service pour renvoyer un jeton d’accès utilisable

L’application front-end dispose maintenant des autorisations nécessaires pour accéder à l’application back-end en tant qu’utilisateur connecté. Dans cette étape, vous configurez l’authentification et l’autorisation App Service pour obtenir un jeton d’accès utilisable pour accéder au serveur principal. Pour cette étape, vous avez besoin de l’ID de client du back-end, que vous avez copié dans la section [Activer l’authentification et l’autorisation pour l’application back-end](#enable-authentication-and-authorization-for-back-end-app).

Accédez à [Azure Resource Explorer](https://resources.azure.com) et, à l’aide de l’arborescence de ressources, localisez votre application web front-end.

[Azure Resource Explorer](https://resources.azure.com) est désormais ouvert avec votre application front-end sélectionnée dans l’arborescence des ressources. En haut de la page, cliquez sur **Lecture/écriture** pour apporter des modifications à vos ressources Azure.

:::image type="content" source="./media/tutorial-auth-aad/resources-enable-write.png" alt-text="Capture d’écran des boutons Lecture seule et Lecture/écriture situés en haut de la page Azure Resource Explorer, avec le bouton Lecture/écriture sélectionné.":::

Dans le navigateur de gauche, descendez dans la hiérarchie jusqu’à **conf** > **authsettings**.

Dans l’affichage **authsettings**, cliquez sur **Modifier**. Définissez `additionalLoginParams` sur la chaîne JSON suivante, en utilisant l’ID de client que vous avez copié. 

```json
"additionalLoginParams": ["response_type=code id_token","resource=<back-end-client-id>"],
```

:::image type="content" source="./media/tutorial-auth-aad/additional-login-params-front-end.png" alt-text="Capture d’écran d’un exemple de code dans la vue authsettings montrant la chaîne additionalLoginParams avec un exemple d’ID client.":::

Enregistrez vos paramètres en cliquant sur **PUT**.

Vos applications sont désormais configurées. Le serveur frontal est maintenant prêt à accéder au serveur principal avec un jeton d’accès approprié.

Pour plus d’informations sur la configuration du jeton d’accès pour d’autres fournisseurs, consultez [Actualiser les jetons de fournisseur d’identité](app-service-authentication-how-to.md#refresh-identity-provider-tokens).

## <a name="call-api-securely-from-server-code"></a>Appeler une API de façon sécurisée à partir du code du serveur

Dans cette étape, vous activez votre code du serveur modifié précédemment pour effectuer des appels authentifiés à l’API de serveur principal.

Votre application front-end a désormais l’autorisation nécessaire et ajoute également l’ID de client du back-end aux paramètres de connexion. Par conséquent, elle peut obtenir un jeton d’accès pour l’authentification avec l’application principale. App Service fournit ce jeton dans votre code du serveur en injectant un en-tête `X-MS-TOKEN-AAD-ACCESS-TOKEN` à chaque requête authentifiée (consultez [Retrieve tokens in app code](app-service-authentication-how-to.md#retrieve-tokens-in-app-code) (Récupérer des jetons dans le code d’application)).

> [!NOTE]
> Ces en-têtes sont injectés pour tous les langages pris en charge. Vous y accédez à l’aide du modèle standard pour chaque langage respectif.

Dans le référentiel local, ouvrez une nouvelle fois _Controllers/TodoController.cs_. Dans le constructeur `TodoController(TodoContext context)`, ajoutez le code suivant :

```cs
public override void OnActionExecuting(ActionExecutingContext context)
{
    base.OnActionExecuting(context);

    _client.DefaultRequestHeaders.Accept.Clear();
    _client.DefaultRequestHeaders.Authorization =
        new AuthenticationHeaderValue("Bearer", Request.Headers["X-MS-TOKEN-AAD-ACCESS-TOKEN"]);
}
```

Ce code ajoute l’en-tête HTTP standard `Authorization: Bearer <access-token>` à tous les appels d’API à distance. Dans le pipeline d’exécution de la requête MVC ASP.NET Core, `OnActionExecuting` s’exécute juste avant l’action respective, afin que chacun de vos appels d’API sortants présente désormais le jeton d’accès.

Enregistrez toutes vos modifications. Dans la fenêtre de terminal local, déployez vos modifications à l’application frontale avec les commandes Git suivantes :

```bash
git add .
git commit -m "add authorization header for server code"
git push frontend master
```

Connectez-vous à `https://<front-end-app-name>.azurewebsites.net` une nouvelle fois. Sur la page de contrat d’utilisation des données utilisateur, cliquez sur **Accepter**.

Vous devez maintenant être en mesure de créer, lire, mettre à jour et supprimer des données à partir de l’application principale comme avant. Désormais, la seule différence est que les deux applications sont protégées par l’authentification et l’autorisation App Service, y compris les appels entre les services.

Félicitations ! Votre code du serveur accède maintenant aux données de serveur principal pour le compte de l’utilisateur authentifié.

## <a name="call-api-securely-from-browser-code"></a>Appeler une API de façon sécurisée à partir du code du navigateur

Dans cette étape, vous pointez l’application Angular.js frontale vers l’API principale. De cette manière, vous apprenez à récupérer le jeton d’accès et à effectuer des appels d’API à l’application principale avec celui-ci.

Alors que le code du serveur a accès aux en-têtes de requête, le code du client peut accéder à `GET /.auth/me` pour obtenir les mêmes jetons d’accès (consultez [Retrieve tokens in app code](app-service-authentication-how-to.md#retrieve-tokens-in-app-code) (Récupérer des jetons dans le code d’application)).

> [!TIP]
> Cette section utilise les méthodes HTTP standard pour illustrer les appels HTTP sécurisés. Toutefois, vous pouvez utiliser la [Bibliothèque d’authentification Microsoft pour JavaScript](https://github.com/AzureAD/microsoft-authentication-library-for-js) afin de simplifier le modèle d’application Angular.js.
>

### <a name="configure-cors"></a>Configuration de CORS

Dans Cloud Shell, activez CORS pour votre URL de client à l’aide de la commande [`az webapp cors add`](/cli/azure/webapp/cors#az-webapp-cors-add). Remplacez les valeurs des espaces réservés _\<back-end-app-name>_ et _\<front-end-app-name>_ .

```azurecli-interactive
az webapp cors add --resource-group myAuthResourceGroup --name <back-end-app-name> --allowed-origins 'https://<front-end-app-name>.azurewebsites.net'
```

Cette étape n’est pas liée à l’authentification et à l’autorisation. Toutefois, vous en avez besoin pour que votre navigateur autorise les appels d’API entre les domaines à partir de votre application Angular.js. Pour plus d’informations, consultez [Ajoutez des fonctionnalités CORS](app-service-web-tutorial-rest-api.md#add-cors-functionality).

### <a name="point-angularjs-app-to-back-end-api"></a>Pointer l’application Angular.js vers l’API de serveur principal

Dans le référentiel local, ouvrez _wwwroot/index.html_.

À la ligne 51, définissez la variable `apiEndpoint` sur l’URL HTTPS de votre application back-end (`https://<back-end-app-name>.azurewebsites.net`). Remplacez _\<back-end-app-name>_ par le nom de votre application dans App Service.

Dans le référentiel local, ouvrez _wwwroot/app/scripts/todoListSvc.js_ ; vous pouvez constater que `apiEndpoint` est ajouté à tous les appels d’API. Votre application Angular.js appelle maintenant les API de serveur principal. 

### <a name="add-access-token-to-api-calls"></a>Ajouter un jeton d’accès aux appels d’API

Dans _wwwroot/app/scripts/todoListSvc.js_, au-dessus de la liste des appels d’API (au-dessus de la ligne `getItems : function(){`), ajoutez la fonction suivante à la liste :

```javascript
setAuth: function (token) {
    $http.defaults.headers.common['Authorization'] = 'Bearer ' + token;
},
```

Cette fonction est appelée pour définir l’en-tête `Authorization` par défaut avec le jeton d’accès. Vous l’appelez à l’étape suivante.

Dans le référentiel local, ouvrez _wwwroot/app/scripts/app.js_ et recherchez le code suivant :

```javascript
$routeProvider.when("/Home", {
    controller: "todoListCtrl",
    templateUrl: "/App/Views/TodoList.html",
}).otherwise({ redirectTo: "/Home" });
```

Remplacez l’intégralité du bloc de code par le code suivant :

```javascript
$routeProvider.when("/Home", {
    controller: "todoListCtrl",
    templateUrl: "/App/Views/TodoList.html",
    resolve: {
        token: ['$http', 'todoListSvc', function ($http, todoListSvc) {
            return $http.get('/.auth/me').then(function (response) {
                todoListSvc.setAuth(response.data[0].access_token);
                return response.data[0].access_token;
            });
        }]
    },
}).otherwise({ redirectTo: "/Home" });
```

La nouvelle modification ajoute le mappage `resolve` qui appelle `/.auth/me` et définit le jeton d’accès. Il permet de s’assurer que vous avez le jeton d’accès avant d’instancier le contrôleur `todoListCtrl`. De cette façon, tous les appels d’API par le contrôleur incluent le jeton.

### <a name="deploy-updates-and-test"></a>Déployer des mises à jour et des tests

Enregistrez toutes vos modifications. Dans la fenêtre de terminal local, déployez vos modifications à l’application frontale avec les commandes Git suivantes :

```bash
git add .
git commit -m "add authorization header for Angular"
git push frontend master
```

Accédez à nouveau à `https://<front-end-app-name>.azurewebsites.net`. Vous devez maintenant être en mesure de créer, lire, mettre à jour et supprimer des données à partir de l’application principale, et ce, directement dans l’application Angular.js.

Félicitations ! Votre code du client accède maintenant aux données de serveur principal pour le compte de l’utilisateur authentifié.

## <a name="when-access-tokens-expire"></a>Date d’expiration des jetons d’accès

Votre jeton d’accès expire après un certain laps de temps. Pour plus d’informations sur la façon d’actualiser vos jetons d’accès sans obliger les utilisateurs à se réauthentifier auprès de votre application, consultez [Actualiser les jetons de fournisseur d’identité](app-service-authentication-how-to.md#refresh-identity-provider-tokens).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Au cours des étapes précédentes, vous avez créé des ressources Azure au sein d’un groupe de ressources. Si vous ne pensez pas avoir besoin de ces ressources à l’avenir, supprimez le groupe de ressources en exécutant la commande suivante dans Cloud Shell :

```azurecli-interactive
az group delete --name myAuthResourceGroup
```

L’exécution de cette commande peut prendre une minute.

<a name="next"></a>
## <a name="next-steps"></a>Étapes suivantes

Vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Activer l’authentification et l’autorisation intégrées
> * Sécuriser des applications contre des requêtes non authentifiées
> * Utiliser Azure Active Directory en tant que fournisseur d’identité
> * Accéder à une application à distance pour le compte de l’utilisateur connecté
> * Sécuriser des appels entre des services avec un jeton d’authentification
> * Utiliser des jetons d’accès à partir du code du serveur
> * Utiliser des jetons d’accès à partir du code du client (navigateur)

Passez au didacticiel suivant pour découvrir comment mapper un nom DNS personnalisé à votre application.

> [!div class="nextstepaction"]
> [Mapper un nom DNS personnalisé existant à Azure App Service](app-service-web-tutorial-custom-domain.md)

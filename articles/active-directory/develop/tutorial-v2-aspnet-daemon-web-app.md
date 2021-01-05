---
title: 'Tutoriel : Créer un démon multilocataire qui accède à des données métier Microsoft Graph | Azure'
titleSuffix: Microsoft identity platform
description: Dans ce tutoriel, vous allez voir comment appeler une API web ASP.NET protégée par Azure Active Directory à partir d’une application de bureau Windows (WPF). Le client WPF authentifie un utilisateur, demande un jeton d’accès et appelle l’API web.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 12/10/2019
ms.author: jmprieur
ms.custom: aaddev, identityplatformtop40, scenarios:getting-started, languages:ASP.NET
ms.openlocfilehash: c1d448fe9da72654ac1600009e66c88c5e7b93b4
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97509425"
---
# <a name="tutorial-build-a-multi-tenant-daemon-that-uses-the-microsoft-identity-platform"></a>Tutoriel : Créer un démon multilocataire qui utilise la plateforme d’identités Microsoft

Dans ce tutoriel, vous téléchargez et exécutez une application web démon ASP.NET qui illustre l’utilisation de l’octroi d’informations d’identification du client OAuth 2.0 pour qu’un jeton d’accès appelle l’API Microsoft Graph.

Dans ce tutoriel :

> [!div class="checklist"]
> * Intégrer une application démon à la plateforme d’identités Microsoft
> * Accorder des autorisations d’application directement à l’application par un administrateur
> * Obtenir un jeton d’accès pour appeler l’API Microsoft Graph
> * Appeler l’API Microsoft Graph

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

- [Visual Studio 2017 ou 2019](https://visualstudio.microsoft.com/downloads/).
- Un locataire Azure AD. Pour plus d’informations, consultez [Obtenir un locataire Azure AD](quickstart-create-new-tenant.md).
- Un ou plusieurs comptes d’utilisateur dans votre locataire Azure AD. Cet exemple ne fonctionne pas avec un compte Microsoft. Si vous vous êtes connecté au [portail Azure](https://portal.azure.com) avec un compte Microsoft et que vous n’avez encore jamais créé de compte d’utilisateur dans votre annuaire, faites-le maintenant.

## <a name="scenario"></a>Scénario

L’application est générée comme une application ASP.NET MVC. Elle utilise le middleware (intergiciel) OWIN OpenID Connect pour connecter les utilisateurs.

Le composant « démon » de cet exemple est un contrôleur d’API : `SyncController.cs`. Lorsque le contrôleur est appelé, il tire (pull) une liste d’utilisateurs du locataire Azure Active Directory (Azure AD) à partir de Microsoft Graph. `SyncController.cs` est déclenché par un appel AJAX dans l’application web. Il utilise la [bibliothèque MSAL pour .NET](msal-overview.md) afin d’acquérir un jeton d’accès pour Microsoft Graph.

L’application est une application multilocataire destinée aux clients professionnels Microsoft. Elle doit donc leur fournir un moyen de s’abonner ou de se connecter aux données de leur entreprise. Durant le processus de connexion, un administrateur d’entreprise accorde d’abord des *autorisations d’application* directement à l’application afin que celle-ci puisse accéder aux données d’entreprise de manière non interactive, sans nécessiter la présence d’un utilisateur connecté. Dans cet exemple, la majeure partie de la logique montre comment réaliser ce processus de connexion à l’aide du point de terminaison de [consentement administrateur](v2-permissions-and-consent.md#using-the-admin-consent-endpoint) de la plateforme d’identités.

![Le diagramme illustre l’application UserSync avec trois éléments locaux se connectant à Azure : Start dot Auth acquérant un jeton de manière interactive pour se connecter à Azure AD, AccountController obtenant le consentement de l’administrateur pour se connecter à Azure AD et SyncController lisant un utilisateur pour une connexion à Microsoft Graph.](./media/tutorial-v2-aspnet-daemon-webapp/topology.png)

Pour plus d’informations sur les concepts utilisés dans cet exemple, lisez la [documentation relative au protocole d’informations d’identification de client pour le point de terminaison de la plateforme d’identités](v2-oauth2-client-creds-grant-flow.md).

## <a name="clone-or-download-this-repository"></a>Cloner ou télécharger ce dépôt

À partir de votre interpréteur de commandes ou de votre ligne de commande, entre la commande suivante :

```Shell
git clone https://github.com/Azure-Samples/active-directory-dotnet-daemon-v2.git
```

Ou [téléchargez l’exemple dans un fichier zip](https://github.com/Azure-Samples/ms-identity-aspnet-daemon-webapp/archive/master.zip).

## <a name="register-your-application"></a>Inscrivez votre application

Cet exemple comporte un projet. Pour inscrire l’application auprès de votre locataire Azure AD, vous pouvez choisir l’une des méthodes suivantes :

- Effectuer les étapes décrites dans [Inscrire l’exemple d’application auprès de votre locataire Azure AD](#register-the-client-app-dotnet-web-daemon-v2) et [Configurer l’exemple pour utiliser votre locataire Azure AD](#choose-the-azure-ad-tenant)
- Utilisez des scripts PowerShell pour :
  - Créer *automatiquement* les applications Azure AD et les objets associés (mots de passe, autorisations et dépendances)
  - Modifier les fichiers de configuration des projets Visual Studio.

Si vous souhaitez utiliser la méthode automatisée :

1. Dans Windows, exécutez PowerShell et accédez à la racine du répertoire cloné.
1. Exécutez cette commande :

   ```PowerShell
   Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process -Force
   ```

1. Exécutez le script pour créer votre application Azure AD et configurer le code de l’exemple d’application en conséquence :

   ```PowerShell
   .\AppCreationScripts\Configure.ps1
   ```

   D’autres méthodes d’exécution des scripts sont décrites dans [Scripts de création d’application](https://github.com/Azure-Samples/ms-identity-aspnet-daemon-webapp/blob/master/AppCreationScripts/AppCreationScripts.md).

1. Ouvrez la solution Visual Studio, puis sélectionnez **Démarrer** pour exécuter le code.

Si vous ne souhaitez pas utiliser l’automatisation, suivez les étapes décrites dans les sections ci-dessous.

### <a name="choose-the-azure-ad-tenant"></a>Choisir le locataire Azure AD

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Si vous avez accès à plusieurs locataires, utilisez le filtre **Répertoire + abonnement** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: dans le menu du haut pour sélectionner le locataire dans lequel vous voulez inscrire une application.


### <a name="register-the-client-app-dotnet-web-daemon-v2"></a>Inscrire l’application cliente (dotnet-web-daemon-v2)

1. Recherchez et sélectionnez **Azure Active Directory**.
1. Sous **Gérer**, sélectionnez **Inscriptions d’applications** > **Nouvelle inscription**.
1. Entrez un **nom** pour votre application (par exemple, `dotnet-web-daemon-v2`). Les utilisateurs de votre application peuvent voir ce nom, et vous pouvez le changer ultérieurement.
1. Dans la section **Types de comptes pris en charge**, sélectionnez **Comptes dans un annuaire organisationnel**.
1. Dans la section **URI de redirection (facultatif)** , sélectionnez **Web** dans la zone de liste modifiable et entrez `https://localhost:44316/` et `https://localhost:44316/Account/GrantPermissions` comme URI de redirection.

    S’il y a plus de deux URI de redirection, vous devrez les ajouter à partir de l’onglet **Authentification** une fois que l’application aura été créée.
1. Sélectionnez **Inscrire** pour créer l’application.
1. Dans la page **Vue d’ensemble** de l’application, recherchez la valeur de l’**ID d’application (client)** et notez-la. Vous en aurez besoin pour configurer le fichier de configuration Visual Studio pour ce projet.
1. Sous **Gérer**, sélectionnez **Authentification**.
1. Définissez **URL de déconnexion** sur `https://localhost:44316/Account/EndSession`.
1. Dans la section **Octroi implicite**, sélectionnez **Jetons d’accès** et **Jetons d’ID**. Cet exemple nécessite l’activation du [flux d’octroi implicite](v2-oauth2-implicit-grant-flow.md) pour la connexion de l’utilisateur et l’appel d’une API.
1. Sélectionnez **Enregistrer**.
1. Sous **Gérer**, sélectionnez **Certificats et secrets**.
1. Dans la section **Secrets client**, sélectionnez **Nouveau secret client**. 
1. Entrez une description pour la clé (par exemple, **secret de l’application**).
1. Sélectionnez une durée pour la clé : **Dans 1 an**, **Dans 2 ans** ou **N’expire jamais**.
1. Sélectionnez **Ajouter**. Enregistrez la valeur de la clé dans un endroit sûr. Vous aurez besoin de cette clé plus tard pour configurer le projet dans Visual Studio.
1. Sous **Gérer**, sélectionnez **Autorisations de l’API** > **Ajouter une autorisation**.
1. Dans la section **API Microsoft couramment utilisées**, sélectionnez **Microsoft Graph**.
1. Dans la section **Autorisations d’application**, vérifiez que les autorisations appropriées sont sélectionnées : **User.Read.All**.
1. Sélectionnez **Ajouter des autorisations**.

## <a name="configure-the-sample-to-use-your-azure-ad-tenant"></a>Configurer l’exemple pour utiliser votre locataire Azure AD

Dans les étapes suivantes, **ClientID** équivaut à « ID d’application » ou à **AppId**.

Ouvrez la solution dans Visual Studio pour configurer les projets.

### <a name="configure-the-client-project"></a>Configurer le projet client

Si vous avez utilisé les scripts d’installation, les modifications suivantes ont été automatiquement appliquées.

1. Ouvrez le fichier **UserSync\Web.Config**.
1. Recherchez la clé d’application **ida:ClientId**. Remplacez la valeur existante par l’ID d’application de l’application **dotnet-web-daemon-v2** que vous avez copié à partir du portail Azure.
1. Recherchez la clé d’application **ida:ClientSecret**. Remplacez la valeur existante par la clé que vous avez enregistrée lors de la création de l’application **dotnet-web-daemon-v2** dans le portail Azure.

## <a name="run-the-sample"></a>Exécution de l'exemple

Nettoyez la solution et regénérez-la, exécutez l’application UserSync, puis connectez-vous en tant qu’administrateur dans votre locataire Azure AD. Si vous n’avez pas de locataire Azure AD dédié aux tests, vous pouvez [suivre ces instructions](quickstart-create-new-tenant.md) pour en obtenir un.

Quand vous vous connectez, l’application vous demande d’abord votre autorisation pour vous connecter et lire votre profil utilisateur. Ce consentement permet à l’application de vérifier que vous êtes bien un utilisateur professionnel.

![Consentement de l’utilisateur](./media/tutorial-v2-aspnet-daemon-webapp/firstconsent.png)

L’application essaie ensuite de synchroniser une liste d’utilisateurs à partir de votre locataire Azure AD, par le biais de Microsoft Graph. Si elle n’y parvient pas, elle vous invite (en tant qu’administrateur du locataire) à connecter votre locataire à l’application.

L’application vous demande ensuite l’autorisation de lire la liste des utilisateurs de votre locataire.

![Consentement de l’administrateur](./media/tutorial-v2-aspnet-daemon-webapp/adminconsent.PNG)

Une fois que vous avez accordé l’autorisation, vous êtes déconnecté de l’application. Cette déconnexion garantit que les jetons d’accès existants pour Microsoft Graph seront supprimés du cache de jetons. Une fois que vous serez reconnecté, le nouveau jeton obtenu aura les autorisations nécessaires pour effectuer des appels à Microsoft Graph.


Lorsque vous accordez l’autorisation, l’application est en mesure de rechercher les utilisateurs à tout moment. Vous pouvez le vérifier en sélectionnant le bouton **Synchroniser les utilisateurs**, puis en actualisant la liste des utilisateurs. Essayez d’ajouter ou de supprimer un utilisateur, puis de resynchroniser la liste (notez toutefois que l’application synchronise uniquement la première page des utilisateurs).

## <a name="about-the-code"></a>À propos du code

Le code correspondant à cet exemple se trouve dans les fichiers suivants :

- **App_Start\Startup.Auth.cs**, **Controllers\AccountController.cs** : Connexion initiale. En particulier, les actions sur le contrôleur ont un attribut **Authorize** qui force l’utilisateur à se connecter. L’application utilise le [flux de code d’autorisation](v2-oauth2-auth-code-flow.md) pour connecter l’utilisateur.
- **Controllers\SyncController.cs** : synchronisation de la liste des utilisateurs avec le magasin local en mémoire.
- **Controllers\UserController.cs** : affichage de la liste des utilisateurs à partir du magasin local en mémoire.
- **Controllers\AccountController.cs** : obtention des autorisations de l’administrateur du locataire à l’aide du point de terminaison de consentement de l’administrateur.

## <a name="re-create-the-sample-app"></a>Recréer l’exemple d’application

1. Dans Visual Studio, créez un projet **Visual C#** **Application web ASP.NET (.NET Framework)** .
1. Dans l’écran suivant, choisissez le modèle de projet **MVC**. Ensuite, ajoutez des références de dossier et de base pour **API web**, car vous ajouterez un contrôleur d’API Web plus tard. Conservez le mode d’authentification sélectionné par défaut pour le projet : **Aucune authentification**.
1. Sélectionnez le projet dans la fenêtre **Explorateur de solutions**, puis appuyez sur la touche **F4**.
1. Dans les propriétés du projet, définissez **SSL activé** sur **True**. Notez les informations qui se trouvent dans **URL SSL**. Vous devrez les fournir au moment de configurer l’inscription de cette application dans le portail Azure.
1. Ajoutez les packages NuGet middleware ASP.NET OWIN suivants :
   - Microsoft.Owin.Security.ActiveDirectory
   - Microsoft.Owin.Security.Cookies
   - Microsoft.Owin.Host.SystemWeb
   - Microsoft.IdentityModel.Protocol.Extensions
   - Microsoft.Owin.Security.OpenIdConnect
   - Microsoft.Identity.Client
1. Dans le dossier **App_Start** :
   1. Créez une classe appelée **Startup.Auth.cs**.
   1. Supprimez **.App_Start** du nom de l’espace de noms.
   1. Remplacez le code de la classe **Startup** par le code de l’exemple d’application figurant dans le même fichier.
   Veillez à copier l’intégralité de la définition de classe. La définition passe de **public class Startup** à **public partial class Startup**.
1. Dans **Startup.Auth.cs**, corrigez les références manquantes en ajoutant les instructions **using** suggérées par la fonctionnalité IntelliSense de Visual Studio.
1. Cliquez avec le bouton droit sur le projet, sélectionnez **Ajouter**, puis sélectionnez **Classe**.
1. Dans la zone de recherche, entrez **OWIN**. La **classe OWIN Startup** s’affiche comme la sélection. Sélectionnez-la, puis nommez la classe **Startup.cs**.
1. Dans **Startup.cs**, remplacez le code de la classe **Startup** par le code de l’exemple d’application figurant dans le même fichier. Encore une fois, notez que la définition passe de **public class Startup** à **public partial class Startup**.
1. Dans le dossier **Models**, ajoutez une nouvelle classe appelée **MsGraphUser.cs**. Remplacez l’implémentation par le contenu du fichier du même nom que celui de l’exemple.
1. Ajoutez une nouvelle instance de **Contrôleur MVC 5 - Vide** que vous nommerez **AccountController**. Remplacez l’implémentation par le contenu du fichier du même nom que celui de l’exemple.
1. Ajoutez une nouvelle instance de **Contrôleur MVC 5 - Vide** que vous nommerez **UserController**. Remplacez l’implémentation par le contenu du fichier du même nom que celui de l’exemple.
1. Ajoutez une nouvelle instance de **Contrôleur Web API 2 – Vide** que vous nommerez **SyncController**. Remplacez l’implémentation par le contenu du fichier du même nom que celui de l’exemple.
1. Pour l’interface utilisateur, dans le dossier **Views\Account**, ajoutez trois instances de **Vues vides (sans modèle)** que vous nommerez **GrantPermissions**, **Index** et **UserMismatch**. Ajoutez la vue nommée **Index** dans le dossier **Views\User**. Remplacez l’implémentation par le contenu du fichier du même nom que celui de l’exemple.
1. Mettez à jour **Shared\_Layout.cshtml** et **Home\Index.cshtml** pour lier correctement les différentes vues.

## <a name="deploy-the-sample-to-azure"></a>Déployer l’exemple sur Azure

Ce projet comporte des projets d’API web et d’applications web. Pour les déployer sur des sites web Azure, effectuez les étapes suivantes pour chacun d’eux :

1. Créez un site web Azure.
1. Publiez l’application web et les API web sur le site web.
1. Mettez à jour les clients pour qu’ils appellent le site web au lieu d’IIS Express.

### <a name="create-and-publish-dotnet-web-daemon-v2-to-an-azure-website"></a>Créer et publier dotnet-web-daemon-v2 sur un site web Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans le coin supérieur gauche, sélectionnez **Créer une ressource**.
1. Sélectionnez **Web** > **Application web**, puis donnez un nom à votre site web. Par exemple, nommez-le **dotnet-web-daemon-v2-contoso.azurewebsites.net**.
1. Sélectionnez les informations concernant l’**abonnement**, le **groupe de ressources**, ainsi que le **plan App Service et l’emplacement**. **Système d’exploitation** est défini sur **Windows** et **Publier** est défini sur **Code**.
1. Sélectionnez **Créer**, puis attendez la fin de la création du service d’application.
1. Lorsque la notification **Déploiement réussi** s’affiche, sélectionnez **Accéder à la ressource** pour accéder au service d’application nouvellement créé.
1. Une fois le site web créé, recherchez-le dans le **tableau de bord**, puis sélectionnez-le pour ouvrir l’écran **Vue d’ensemble** du service d’application.
1. Sous l’onglet **Vue d’ensemble** du service d’application, téléchargez le profil de publication en sélectionnant le lien **Obtenir le profil de publication**, puis enregistrez-le. Vous pouvez utiliser d’autres mécanismes de déploiement, comme le déploiement à partir du contrôle de code source.
1. Passez à Visual Studio, puis :
   1. Accédez au projet **dotnet-web-daemon-v2**.
   1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet, puis sélectionnez **Publier**.
   1. Sélectionnez **Importer le profil** dans la barre inférieure, puis importez le profil de publication que vous avez téléchargé précédemment.
1. Sélectionnez **Configurer**.
1. Sous l’onglet **Connexion**, mettez à jour l’URL de destination afin qu’elle utilise « HTTPS ». Par exemple, utilisez `https://dotnet-web-daemon-v2-contoso.azurewebsites.net`. Sélectionnez **Suivant**.
1. Sous l’onglet **Paramètres**, vérifiez que l’option **Activer l’authentification d’organisation** n’est pas sélectionnée.
1. Sélectionnez **Enregistrer**. Sélectionnez **Publier** dans l’écran principal.

Visual Studio publie le projet et ouvre automatiquement l’URL du projet dans un navigateur. Si la page web par défaut du projet s’affiche, la publication a réussi.

### <a name="update-the-azure-ad-tenant-application-registration-for-dotnet-web-daemon-v2"></a>Mettre à jour l’inscription de l’application locataire Azure AD pour dotnet-web-daemon-v2

1. Revenez au [portail Azure](https://portal.azure.com).
1. Dans le volet de gauche, sélectionnez le service **Azure Active Directory**, puis **Inscriptions d’applications**.
1. Sélectionnez l’application **dotnet-web-daemon-v2**.
1. Dans la page **Authentification** de votre application, mettez à jour les champs **URL de déconnexion** avec l’adresse de votre service. Par exemple, utilisez `https://dotnet-web-daemon-v2-contoso.azurewebsites.net`.
1. Dans le menu **Personnalisation**, mettez à jour l’**URL de la page d’accueil** avec l’adresse de votre service. Par exemple, utilisez `https://dotnet-web-daemon-v2-contoso.azurewebsites.net`.
1. Enregistrez la configuration.
1. Ajoutez la même URL à la liste d’URL du menu **Authentification** > **URI de redirection**. Si vous avez plusieurs URL de redirection, vérifiez qu’il existe une nouvelle entrée correspondant à l’URI du service d’application pour chaque URL de redirection.

## <a name="clean-up-resources"></a>Nettoyer les ressources
Quand vous n’en avez plus besoin, supprimez l’objet d’application que vous avez créé à l’étape [Inscrivez votre application](#register-your-application).  Pour supprimer l’application, suivez les instructions fournies dans [Supprimer une application créée par vous ou votre organisation](quickstart-remove-app.md#remove-an-application-authored-by-you-or-your-organization).

## <a name="get-help"></a>Obtenir de l’aide

Accédez à [Stack Overflow](http://stackoverflow.com/questions/tagged/msal) pour obtenir un support de la part de la communauté.
Posez vos questions sur Stack Overflow d’abord et parcourez les problèmes déjà rencontrés pour voir si quelqu’un d’autre a déjà posé la même question.
Vérifiez que vos questions ou commentaires comportent bien les étiquettes « adal », « msal » et « dotnet ».

Si vous trouvez un bogue dans l’exemple, signalez-le sur [GitHub Issues](https://github.com/Azure-Samples/ms-identity-aspnet-daemon-webapp/issues).

Si vous trouvez un bogue dans MSAL.NET, signalez-le sur [MSAL.NET GitHub Issues](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/issues).

Pour faire une suggestion, accédez à la [page User Voice](https://feedback.azure.com/forums/169401-azure-active-directory).

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur la création d’applications démon qui utilisent la plateforme d’identités Microsoft pour accéder à des API web protégées :

> [!div class="nextstepaction"]
> [Scénario : Application démon appelant des API web](scenario-daemon-overview.md)

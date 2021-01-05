---
title: Modifications apportées à un projet MVC quand vous vous connectez à Azure AD
description: Décrit les conséquences sur votre projet MVC lorsque vous vous connectez à Azure AD à l'aide des services connectés Visual Studio
author: ghogen
manager: jillfra
ms.prod: visual-studio-windows
ms.technology: vs-azure
ms.workload: azure-vs
ms.topic: how-to
ms.date: 03/12/2018
ms.author: ghogen
ms.custom: devx-track-csharp, aaddev, vs-azure
ms.openlocfilehash: ab72512b2e5d915dbd4b8a3d7d8446a5932eef9e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88165445"
---
# <a name="what-happened-to-my-mvc-project-visual-studio-azure-active-directory-connected-service"></a>Qu'est-il arrivé à mon projet MVC (service connecté Azure Active Directory Visual Studio) ?

> [!div class="op_single_selector"]
> - [Prise en main](vs-active-directory-dotnet-getting-started.md)
> - [Que s'est-il passé ?](vs-active-directory-dotnet-what-happened.md)

Cet article identifie les modifications exactes apportées à un projet MVC ASP.NET lors de l’ajout du [Service connecté Azure Active Directory avec Visual Studio](vs-active-directory-add-connected-service.md).

Pour plus d’informations sur l’utilisation du service connecté, consultez la page [Bien démarrer](vs-active-directory-dotnet-getting-started.md).

## <a name="added-references"></a>Références ajoutées

Affecte le fichier projet (références * .NET) et `packages.config` (références NuGet).

| Type | Informations de référence |
| --- | --- |
| .NET ; NuGet | Microsoft.IdentityModel.Protocol.Extensions |
| .NET ; NuGet | Microsoft.Owin |
| .NET ; NuGet | Microsoft.Owin.Host.SystemWeb |
| .NET ; NuGet | Microsoft.Owin.Security |
| .NET ; NuGet | Microsoft.Owin.Security.Cookies |
| .NET ; NuGet | Microsoft.Owin.Security.OpenIdConnect |
| .NET ; NuGet | Owin |
| .NET        | System.IdentityModel |
| .NET ; NuGet | System.IdentityModel.Tokens.Jwt |
| .NET        | System.Runtime.Serialization |

Références supplémentaires si l’option **Lire les données d’annuaire** est sélectionnée :

| Type | Informations de référence |
| --- | --- |
| .NET ; NuGet | EntityFramework |
| .NET        | EntityFramework.SqlServer (Visual Studio 2015 uniquement) |
| .NET ; NuGet | Microsoft.Azure.ActiveDirectory.GraphClient |
| .NET ; NuGet | Microsoft.Data.Edm |
| .NET ; NuGet | Microsoft.Data.OData |
| .NET ; NuGet | Microsoft.Data.Services.Client |
| .NET ; NuGet | Microsoft.IdentityModel.Clients.ActiveDirectory |
| .NET        | Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms (Visual Studio 2015 uniquement) |
| .NET ; NuGet | System.Spatial |

Les références suivantes sont supprimées (projets ASP.NET 4 uniquement, comme dans Visual Studio 2015) :

| Type | Informations de référence |
| --- | --- |
| .NET ; NuGet | Microsoft.AspNet.Identity.Core |
| .NET ; NuGet | Microsoft.AspNet.Identity.EntityFramework |
| .NET ; NuGet | Microsoft.AspNet.Identity.Owin |

## <a name="project-file-changes"></a>Modifications apportées au fichier projet

- Définissez la propriété `IISExpressSSLPort` sur un nombre distinct.
- Définissez la propriété `WebProject_DirectoryAccessLevelKey` sur 0, ou 1 si l’option **Lire les données d’annuaire** est sélectionnée.
- Définissez la propriété `IISUrl` sur `https://localhost:<port>/`, où `<port>` correspond à la valeur `IISExpressSSLPort`.

## <a name="webconfig-or-appconfig-changes"></a>Modifications apportées à web.config ou à app.config

- Ajout des entrées de configuration suivantes :

    ```xml
    <appSettings>
        <add key="ida:ClientId" value="<ClientId from the new Azure AD app>" />
        <add key="ida:AADInstance" value="https://login.microsoftonline.com/" />
        <add key="ida:Domain" value="<your selected Azure domain>" />
        <add key="ida:TenantId" value="<the Id of your selected Azure AD tenant>" />
        <add key="ida:PostLogoutRedirectUri" value="<project start page, such as https://localhost:44335>" />
    </appSettings>
    ```

- Ajout des éléments `<dependentAssembly>` sous le nœud `<runtime><assemblyBinding>` pour `System.IdentityModel.Tokens.Jwt` et `Microsoft.IdentityModel.Protocol.Extensions`.

Modifications supplémentaires si l’option **Lire les données d’annuaire** est sélectionnée :

- Ajout de l’entrée de configuration sous `<appSettings>` :

    ```xml
    <add key="ida:ClientSecret" value="<Azure AD app's new client secret>" />
    ```

- Ajout des éléments suivants sous `<configuration>` ; les valeurs de project-mdf-file et de project-catalog-id varieront :

    ```xml
    <configSections>
      <!-- For more information on Entity Framework configuration, visit https://go.microsoft.com/fwlink/?LinkID=237468 -->
      <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
    </configSections>

    <connectionStrings>
      <add name="DefaultConnection" connectionString="Data Source=(localdb)\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\<project-mdf-file>.mdf;Initial Catalog=<project-catalog-id>;Integrated Security=True" providerName="System.Data.SqlClient" />
    </connectionStrings>

    <entityFramework>
      <defaultConnectionFactory type="System.Data.Entity.Infrastructure.LocalDbConnectionFactory, EntityFramework">
        <parameters>
          <parameter value="mssqllocaldb" />
        </parameters>
      </defaultConnectionFactory>
      <providers>
        <provider invariantName="System.Data.SqlClient" type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" />
      </providers>
    </entityFramework>
    ```

- Ajout des éléments `<dependentAssembly>` sous le nœud `<runtime><assemblyBinding>` pour `Microsoft.Data.Services.Client`, `Microsoft.Data.Edm` et `Microsoft.Data.OData`.

## <a name="code-changes-and-additions"></a>Ajouts et modifications de code

- Ajout de l’attribut `[Authorize]` à `Controllers/HomeController.cs` et tous les autres contrôleurs existants.

- Ajout d’une classe de démarrage de l’authentification, `App_Start/Startup.Auth.cs`, comportant la logique de démarrage de l’authentification Azure AD. Si l’option **Lire les données d’annuaire** est sélectionnée, ce fichier contient également le code permettant de recevoir un code OAuth et de l’échanger contre un jeton d’accès.

- Ajout d’une classe de contrôleur, `Controllers/AccountController.cs`, comportant les méthodes `SignIn` et `SignOut`.

- Ajout d’une vue partielle, `Views/Shared/_LoginPartial.cshtml`, contenant un lien d’action pour `SignIn` et `SignOut`.

- Ajout d’une vue partielle, `Views/Account/SignoutCallback.cshtml`, contenant du HTML pour l’interface utilisateur de déconnexion.

- Mise à jour de la méthode `Startup.Configuration` pour inclure un appel à `ConfigureAuth(app)` si la classe existait déjà ; sinon ajout d’une classe `Startup` qui comporte des appels à la méthode.

- Ajout de `Connected Services/AzureAD/ConnectedService.json` (Visual Studio 2017) ou de `Service References/Azure AD/ConnectedService.json` (Visual Studio 2015), contenant des informations dont Visual Studio se sert pour suivre l’ajout du service connecté.

- Si l’option **Lire les données d’annuaire** est sélectionnée, ajout de `Models/ADALTokenCache.cs` et de `Models/ApplicationDbContext.cs` pour prendre en charge la mise en cache des jetons. Ajout également d’un contrôleur et d’une vue supplémentaires pour illustrer l’accès aux informations de profil utilisateur à l’aide des API Graph Azure : `Controllers/UserProfileController.cs`, `Views/UserProfile/Index.cshtml` et `Views/UserProfile/Relogin.cshtml`.

### <a name="file-backup-visual-studio-2015"></a>Sauvegarde de fichiers (Visual Studio 2015)

Lors de l’ajout du service connecté, Visual Studio 2015 sauvegarde les fichiers modifiés et supprimés. Tous les fichiers affectés sont enregistrés dans le dossier `Backup/AzureAD`. Visual Studio 2017 et les versions ultérieures ne créent pas de sauvegardes.

- `Startup.cs`
- `App_Start\IdentityConfig.cs`
- `App_Start\Startup.Auth.cs`
- `Controllers\AccountController.cs`
- `Controllers\ManageController.cs`
- `Models\IdentityModels.cs`
- `Models\ManageViewModels.cs`
- `Views\Shared\_LoginPartial.cshtml`

## <a name="changes-on-azure"></a>Modifications apportées à Azure

- Création d’une application Azure AD dans le domaine sélectionné lors de l’ajout du service connecté.
- Mise à jour de l’application pour inclure l’autorisation **Lire les données d’annuaire** si cette option est sélectionnée.

[En savoir plus sur Azure Active Directory](https://azure.microsoft.com/services/active-directory/).

## <a name="next-steps"></a>Étapes suivantes

- [Scénarios d’authentification pour Azure Active Directory](./authentication-vs-authorization.md)
- [Ajouter la connexion avec Microsoft à une application web ASP.NET](quickstart-v2-aspnet-webapp.md)
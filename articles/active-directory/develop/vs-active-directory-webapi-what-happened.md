---
title: Changements apportés aux projets WebAPI lors de la connexion à Azure AD
description: Décrit ce qui se passe dans votre projet WebAPI quand vous vous connectez à Azure AD à l’aide de Visual Studio
author: ghogen
manager: jillfra
ms.workload: azure-vs
ms.prod: visual-studio-windows
ms.technology: vs-azure
ms.topic: how-to
ms.date: 03/12/2018
ms.author: ghogen
ms.custom: devx-track-csharp, aaddev, vs-azure
ms.openlocfilehash: a070bee7a9fa836eeac7c739cf2757295533ad7f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88165360"
---
# <a name="what-happened-to-my-webapi-project-visual-studio-azure-active-directory-connected-service"></a>Qu’est-il arrivé à mon projet WebApi (service connecté Azure Active Directory de Visual Studio)

> [!div class="op_single_selector"]
> - [Prise en main](vs-active-directory-webapi-getting-started.md)
> - [Que s'est-il passé ?](vs-active-directory-webapi-what-happened.md)

Cet article identifie les modifications exactes apportées aux projets ASP.NET Azure API, Application monopage ASP.NET et ASP.NET Azure API lors de l’ajout du [service connecté Azure Active Directory à l’aide de Visual Studio](vs-active-directory-add-connected-service.md). S’applique également aux projets ASP.NET Azure Mobile Service dans Visual Studio 2015.

Pour plus d’informations sur l’utilisation du service connecté, consultez la page [Bien démarrer](vs-active-directory-webapi-getting-started.md).

## <a name="added-references"></a>Références ajoutées

Affecte les références *.NET du fichier projet et `packages.config` (références NuGet).

| Type | Informations de référence |
| --- | --- |
| .NET ; NuGet | Microsoft.Owin |
| .NET ; NuGet | Microsoft.Owin.Host.SystemWeb |
| .NET ; NuGet | Microsoft.Owin.Security |
| .NET ; NuGet | Microsoft.Owin.Security.ActiveDirectory |
| .NET ; NuGet | Microsoft.Owin.Security.Jwt |
| .NET ; NuGet | Microsoft.Owin.Security.OAuth |
| .NET ; NuGet | Owin |
| .NET ; NuGet | System.IdentityModel.Tokens.Jwt |

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
| .NET        | Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms<br>(Visual Studio 2015 uniquement) |
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
        <add key="ida:Tenant" value="<your selected Azure domain>" />
        <add key="ida:Audience" value="<your selected domain + / + project name>" />
    </appSettings>
    ```

- Visual Studio 2017 uniquement : vous avez également ajouté l’entrée suivante sous `<appSettings>` »

    ```xml
    <add key="ida:MetadataAddress" value="<domain URL + /federationmetadata/2007-06/federationmetadata.xml>" />
    ```

- Vous avez ajouté les éléments `<dependentAssembly>` sous le nœud `<runtime><assemblyBinding>` pour `System.IdentityModel.Tokens.Jwt`.

- Si vous avez sélectionné l’option **Lire les données de l’annuaire**, vous avez ajouté l’entrée de configuration suivante sous `<appSettings>` :

    ```xml
    <add key="ida:Password" value="<Your Azure AD app's new password>" />
    ```

## <a name="code-changes-and-additions"></a>Ajouts et modifications de code

- Ajout de l’attribut `[Authorize]` à `Controllers/ValueController.cs` et tous les autres contrôleurs existants.

- Vous avez ajouté une classe de démarrage de l’authentification, `App_Start/Startup.Auth.cs`, qui contient la logique de démarrage pour l’authentification Azure AD ou modifié en conséquence. Si l’option **Lire les données d’annuaire** est sélectionnée, ce fichier contient également le code permettant de recevoir un code OAuth et de l’échanger contre un jeton d’accès.

- (Visual Studio 2015 avec l’application ASP.NET 4 uniquement) Vous avez supprimé `App_Start/IdentityConfig.cs` et ajouté `Controllers/AccountController.cs`, `Models/IdentityModel.cs` et `Providers/ApplicationAuthProvider.cs`.

- Ajout de `Connected Services/AzureAD/ConnectedService.json` (Visual Studio 2017) ou de `Service References/Azure AD/ConnectedService.json` (Visual Studio 2015), contenant des informations dont Visual Studio se sert pour suivre l’ajout du service connecté.

### <a name="file-backup-visual-studio-2015"></a>Sauvegarde de fichiers (Visual Studio 2015)

Lors de l’ajout du service connecté, Visual Studio 2015 sauvegarde les fichiers modifiés et supprimés. Tous les fichiers affectés sont enregistrés dans le dossier `Backup/AzureAD`. Visual Studio 2017 ne crée pas de sauvegardes.

- `Startup.cs`
- `App_Start\IdentityConfig.cs`
- `App_Start\Startup.Auth.cs`
- `Controllers\AccountController.cs`
- `Controllers\ManageController.cs`
- `Models\IdentityModels.cs`
- `Models\ApplicationOAuthProvider.cs`

## <a name="changes-on-azure"></a>Modifications apportées à Azure

- Création d’une application Azure AD dans le domaine sélectionné lors de l’ajout du service connecté.
- Mise à jour de l’application pour inclure l’autorisation **Lire les données d’annuaire** si cette option est sélectionnée.

[En savoir plus sur Azure Active Directory](https://azure.microsoft.com/services/active-directory/).

## <a name="next-steps"></a>Étapes suivantes

- [Scénarios d’authentification pour Azure Active Directory](./authentication-vs-authorization.md)
- [Ajouter la connexion avec Microsoft à une application web ASP.NET](quickstart-v2-aspnet-webapp.md)
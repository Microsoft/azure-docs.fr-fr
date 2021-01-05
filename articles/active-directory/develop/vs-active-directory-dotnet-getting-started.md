---
title: Bien démarrer avec Azure AD dans les projets .NET MVC | Azure
description: Guide pratique pour bien démarrer avec Azure Active Directory dans les projets MVC .NET après connexion à un annuaire ou création d’un annuaire Azure AD à l’aide des Services connectés de Visual Studio.
author: ghogen
manager: jillfra
ms.prod: visual-studio-windows
ms.technology: vs-azure
ms.workload: azure-vs
ms.topic: how-to
ms.date: 03/12/2018
ms.author: ghogen
ms.custom: devx-track-csharp, aaddev, vs-azure
ms.openlocfilehash: 15a7c873e4d1e5c962a89b03f2a5cafc88843192
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88165462"
---
# <a name="getting-started-with-azure-active-directory-aspnet-mvc-projects"></a>Bien démarrer avec Azure Active Directory (projets MVC ASP.NET)

> [!div class="op_single_selector"]
> - [Prise en main](vs-active-directory-dotnet-getting-started.md)
> - [Que s'est-il passé ?](vs-active-directory-dotnet-what-happened.md)

Cet article offre une aide complémentaire une fois Active Directory ajouté à un projet MVC ASP.NET avec la commande **Projet > Services connectés** de Visual Studio. Si vous n’avez pas déjà ajouté le service à votre projet, vous pourrez le faire à tout moment.

Consultez la page [Qu’est-il arrivé à mon projet MVC ?](vs-active-directory-dotnet-what-happened.md) pour connaître les modifications apportées à votre projet lors de l’ajout du service connecté.

## <a name="requiring-authentication-to-access-controllers"></a>Demander une authentification pour l'accès aux contrôleurs

Tous les contrôleurs de votre projet ont été ornés de l’attribut `[Authorize]`. Cet attribut permet de demander à l’utilisateur de s’authentifier avant d’accéder à ces contrôleurs. Pour autoriser un accès anonyme au contrôleur, cet attribut doit être supprimé du contrôleur. Pour définir plus précisément les autorisations, appliquez cet attribut à chaque méthode nécessitant une autorisation, au lieu de l’appliquer à la classe de contrôleur.

## <a name="adding-signin--signout-controls"></a>Ajouter des contrôles SignIn/SignOut

Pour ajouter les commandes Connexion/Déconnexion à l’un de vos affichages, vous pouvez utiliser l’affichage partiel `_LoginPartial.cshtml`. Voici un exemple d’ajout de la fonctionnalité à l’affichage `_Layout.cshtml` standard. (Notez le dernier élément de la classe div avec navbar-collapse) :

```html
<!DOCTYPE html>
 <html>
 <head>
     <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@ViewBag.Title - My ASP.NET Application</title>
    @Styles.Render("~/Content/css")
    @Scripts.Render("~/bundles/modernizr")
</head>
<body>
    <div class="navbar navbar-inverse navbar-fixed-top">
        <div class="container">
            <div class="navbar-header">
                <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                </button>
                @Html.ActionLink("Application name", "Index", "Home", new { area = "" }, new { @class = "navbar-brand" })
            </div>
            <div class="navbar-collapse collapse">
                <ul class="nav navbar-nav">
                    <li>@Html.ActionLink("Home", "Index", "Home")</li>
                    <li>@Html.ActionLink("About", "About", "Home")</li>
                    <li>@Html.ActionLink("Contact", "Contact", "Home")</li>
                </ul>
                @Html.Partial("_LoginPartial")
            </div>
        </div>
    </div>
    <div class="container body-content">
        @RenderBody() 
        <hr />
        <footer>
            <p>&copy; @DateTime.Now.Year - My ASP.NET Application</p>
        </footer>
    </div>
    @Scripts.Render("~/bundles/jquery")
    @Scripts.Render("~/bundles/bootstrap")
    @RenderSection("scripts", required: false)
</body>
</html>
```

## <a name="next-steps"></a>Étapes suivantes

- [Scénarios d’authentification pour Azure Active Directory](./authentication-vs-authorization.md)
- [Ajouter la connexion avec Microsoft à une application web ASP.NET](quickstart-v2-aspnet-webapp.md)
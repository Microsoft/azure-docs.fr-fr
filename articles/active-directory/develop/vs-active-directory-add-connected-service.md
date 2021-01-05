---
title: Utilisation du service connecté Active Directory (Visual Studio)
description: Ajouter un annuaire Azure Active Directory à l’aide de la boîte de dialogue Visual Studio Ajouter des services connectés
author: ghogen
manager: jillfra
ms.prod: visual-studio-windows
ms.technology: vs-azure
ms.custom: devx-track-csharp, aaddev, vs-azure
ms.workload: azure-vs
ms.topic: how-to
ms.date: 03/12/2018
ms.author: ghogen
ms.openlocfilehash: a1ba7db72743ac122a697bf271e783ec64e041e8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88165479"
---
# <a name="add-an-azure-active-directory-by-using-connected-services-in-visual-studio"></a>Ajouter un annuaire Azure Active Directory à l’aide de services connectés dans Visual Studio

En utilisant Azure Active Directory (Azure AD), vous pouvez prendre en charge l’authentification unique (SSO) pour les applications web MVC ASP.NET ou l’authentification Active Directory dans les services API web. Avec l’authentification Azure AD, vos utilisateurs peuvent se servir de leur compte sur Azure Active Directory pour se connecter à vos applications web. Grâce à l’authentification Azure AD avec l’API web, la sécurité des données pendant l’exposition d’une API à partir d’une application web est renforcée. Avec Azure AD, vous n’avez pas besoin de gérer un système d’authentification distinct doté de ses propres dispositifs de gestion de comptes et d’utilisateurs.

Cet article et ceux qui l’accompagnent fournissent des détails sur l’utilisation de la fonctionnalité Service connecté de Visual Studio pour Active Directory. Cette fonctionnalité est disponible dans Visual Studio 2015 et versions ultérieures.

Actuellement, le service connecté Active Directory ne prend pas en charge les applications ASP.NET Core.

## <a name="prerequisites"></a>Prérequis

- Compte Azure : si vous n’avez pas de compte Azure, vous pouvez [vous inscrire à un essai gratuit](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A261C142F) ou [activer les avantages de votre abonnement Visual Studio](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A261C142F).
- **Visual Studio 2015** ou version ultérieure. [Téléchargez maintenant Visual Studio](https://aka.ms/vsdownload?utm_source=mscom&utm_campaign=msdocs).

### <a name="connect-to-azure-active-directory-using-the-connected-services-dialog"></a>Se connecter à Azure Active Directory à l’aide de la boîte de dialogue Services connectés

1. Dans Visual Studio, créez ou ouvrez un projet d’API Web ASP.NET ou un projet MVC ASP.NET. Vous pouvez utiliser les modèles MVC, API Web, Application monopage, Application API Azure, Application mobile Azure et Service mobile Azure.

1. Sélectionnez la commande de menu **Projet > Ajouter un service connecté** ou double-cliquez sur le nœud **Services connectés** figurant sous le projet dans l’Explorateur de solutions.

1. Sur la page **Services connectés**, sélectionnez **Authentification avec Azure Active Directory**.

    ![Page des services connectés](./media/vs-azure-active-directory/connected-services-add-active-directory.png)

1. Dans la page **Introduction**, sélectionnez **Suivant**. Si vous rencontrez des erreurs dans cette page, consultez [Diagnostic d’erreurs avec l’Assistant Connexion Azure Active Directory](vs-active-directory-error.md).

    ![Page d’introduction](./media/vs-azure-active-directory/configure-azure-ad-wizard-1.png)

1. Dans la page **Authentification unique**, sélectionnez un domaine dans la liste déroulante **Domaine**. La liste contient tous les domaines accessibles aux comptes répertoriés dans la boîte de dialogue Paramètres du compte de Visual Studio (**Fichier > Paramètres du compte**). Vous pouvez également entrer un nom de domaine si vous ne trouvez pas celui que vous recherchez, par exemple `mydomain.onmicrosoft.com`. Vous pouvez opter pour la création d’une application Azure Active Directory ou utiliser les paramètres d’une application Azure Active Directory existante. Sélectionnez **Suivant** lorsque vous avez terminé.

    ![Page de l’authentification unique](./media/vs-azure-active-directory/configure-azure-ad-wizard-2.png)

1. Dans la page **Accès aux annuaires**, sélectionnez l’option **Accéder en lecture aux données d’annuaire** si vous le souhaitez. Les développeurs incluent en général cette option.

    ![Page d’accès à l’annuaire](./media/vs-azure-active-directory/configure-azure-ad-wizard-3.png)

1. Sélectionnez **Terminer** pour commencer à modifier votre projet afin d’activer l’authentification Azure AD. Visual Studio affiche la progression pendant ce temps :

    ![Progression du service connecté Active Directory](./media/vs-azure-active-directory/active-directory-connected-service-output.png)

1. Une fois le processus terminé, Visual Studio ouvre l’un des articles suivants dans votre navigateur, en fonction de votre type de projet :

    - [Prise en main des projets .NET MVC](vs-active-directory-dotnet-getting-started.md)
    - [Prise en main des projets WebAPI](vs-active-directory-webapi-getting-started.md)

1. Vous pouvez également voir le domaine Active Directory dans le [Portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).

## <a name="how-your-project-is-modified"></a>Modifications apportées à votre projet

Quand vous ajoutez le service connecté à l’Assistant, Visual Studio ajoute Azure Active Directory et les références associées à votre projet. Les fichiers de configuration et les fichiers de code dans votre projet sont également modifiés pour prendre en charge Azure AD. Les modifications spécifiques que Visual Studio apporte dépendent du type de projet. Consultez les articles suivants pour plus de détails :

- [Qu’est-il arrivé à mon projet .NET MVC ?](vs-active-directory-dotnet-what-happened.md)
- [Qu’est-il arrivé à mon projet d’API web ?](vs-active-directory-webapi-what-happened.md)

## <a name="next-steps"></a>Étapes suivantes

- [Scénarios d’authentification pour Azure Active Directory](./authentication-vs-authorization.md)
- [Ajouter la connexion avec Microsoft à une application web ASP.NET](quickstart-v2-aspnet-webapp.md)
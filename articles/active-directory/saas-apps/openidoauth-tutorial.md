---
title: Configurer une application OpenID/OAuth à partir de la galerie d’applications Azure AD | Microsoft Docs
description: Étapes pour configurer une application OpenID/OAuth à partir de la galerie d’applications Azure AD
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/30/2019
ms.author: jeedes
ms.custom: has-adal-ref
ms.openlocfilehash: 17d78b4d6db9e65868995d8ea7835a0d911e8589
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96348690"
---
# <a name="configure-an-openidoauth-application-from-the-azure-ad-app-gallery"></a>Configurer une application OpenID/OAuth à partir de la galerie d’applications Azure AD

## <a name="process-of-adding-an-openid-application-from-the-gallery"></a>Processus d’ajout d’une application OpenID à partir de la galerie

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** > **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Sélectionnez **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez le nom de l’application. Dans le panneau de résultats, sélectionnez l’application souhaitée et inscrivez-vous à l’application.

    ![Openid dans la liste des résultats](common/search-new-app.png)


1. Dans la page Nom de l’application, cliquez sur le bouton **S’inscrire**.

    ![Bouton Ajouter](./media/openidoauth-tutorial/addbutton.png)

    > [!NOTE]
    > L’administrateur des locataires doit sélectionner le bouton S’inscrire et donner son consentement pour l’application. L’application est alors ajoutée au locataire du client, où vous pouvez effectuer les configurations. Il est inutile d’ajouter l’application explicitement.

5. Vous êtes redirigé vers la page Connexion de l’application ou Azure Active Directory (Azure AD) pour entrer les informations d’identification de connexion.

6. Une fois que l’authentification a été effectuée, vous acceptez le consentement dans la page de consentement. La page d’accueil de l’application s’affiche.

    > [!NOTE]
    > Vous ne pouvez ajouter qu’une instance de l’application. Si vous en avez déjà ajouté une et que vous avez tenté de fournir une nouvelle fois le consentement, l’application n’est pas ajoutée à nouveau dans le locataire. Vous ne pouvez donc utiliser en toute logique qu’une seule instance de l’application dans le locataire.

1. Suivez la vidéo ci-dessous pour ajouter une application OpenID à partir de la galerie.
    >[!VIDEO https://www.microsoft.com/videoplayer/embed/RE4HoNI]

## <a name="authentication-flow-using-openid-connect"></a>Flux d’authentification à l’aide d’OpenID Connect

Le flux d’authentification le plus élémentaire comprend les étapes suivantes :

![Flux d’authentification à l’aide d’OpenID Connect](./media/openidoauth-tutorial/authenticationflow.png)

### <a name="multitenant-application"></a>Application mutualisée
Une application mutualisée est destinée à être utilisée dans plusieurs organisations, et pas dans une seule. Il s’agit généralement d’applications SaaS (software-as-a-service) écrites par un éditeur de logiciels indépendant.

Les applications mutualisées doivent être approvisionnées dans chaque répertoire où elles seront utilisées. Leur inscription nécessite le consentement de l’utilisateur ou de l’administrateur. Ce processus de consentement démarre quand une application a été enregistrée dans l’annuaire et accède à l’API Graph ou à une autre API web. Lorsqu’un utilisateur ou un administrateur d’une autre organisation s’inscrit pour utiliser l’application, une boîte de dialogue contenant les autorisations nécessaires à l’application s’affiche.

L’utilisateur ou l’administrateur peut alors donner son consentement pour l’application. Le consentement permet à l’application d’accéder aux données indiquées, et enregistre l’application dans le répertoire.

> [!NOTE]
> Si vous mettez votre application à la disposition des utilisateurs dans plusieurs répertoires, vous devez disposer d’un mécanisme permettant de déterminer le locataire dans lequel ils se trouvent. Une application à locataire unique ne doit regarder que dans son propre répertoire pour y rechercher un utilisateur. Une application mutualisée doit identifier un utilisateur spécifique dans tous les répertoires d’Azure AD.
>
> À cet effet, Azure AD fournit un point de terminaison d’authentification commun vers lequel une application mutualisée peut diriger les demandes de connexion, plutôt que vers un point de terminaison propre au locataire. Ce point de terminaison est `https://login.microsoftonline.com/common` pour tous les répertoires dans Azure AD. Un point de terminaison propre à un locataire peut être `https://login.microsoftonline.com/contoso.onmicrosoft.com`.
>
> Il est important de tenir compte du point de terminaison commun lorsque vous développez votre application. Vous avez besoin de la logique nécessaire pour gérer plusieurs locataires pendant la connexion, la déconnexion et la validation des jetons.

Par défaut, Azure AD promeut les applications mutualisées. Elles sont facilement accessibles entre les organisations et d’une utilisation simple une fois que vous en avez accepté le consentement.

## <a name="consent-framework"></a>Framework de consentement

Vous pouvez utiliser l’infrastructure de consentement d’Azure AD pour développer des applications clientes natives et web mutualisées. Ces applications autorisent la connexion au moyen de comptes d’utilisateurs d’un locataire Azure AD différent de celui où l’application a été inscrite. Elles peuvent également avoir besoin d’accéder à des API web, par exemple :
- API Microsoft Graph pour accéder à Azure AD, à Intune et aux services de Microsoft 365.
- API des autres services Microsoft
- Vos propres API web

L’infrastructure est basée sur le consentement d’un utilisateur ou d’un administrateur à l’inscription d’une application dans son répertoire. Cette inscription peut impliquer l’accès aux données de répertoire. Une fois le consentement donné, l’application cliente peut appeler l’API Microsoft Graph au nom de l’utilisateur et utiliser les informations en fonction des besoins.

L’[API Microsoft Graph](https://developer.microsoft.com/graph/) permet d’accéder aux données de Microsoft 365, par exemple :

- Calendriers et messages d’Exchange
- Sites et listes de SharePoint
- Documents de OneDrive
- Blocs-notes de OneNote
- Tâches du Planificateur
- Classeurs d’Excel

L’API Graph fournit également un accès aux utilisateurs et groupes d’Azure AD et aux autres objets de données provenant d’autres services de cloud computing Microsoft.

Les étapes suivantes vous montrent comment l’expérience de consentement fonctionne pour le développeur d’applications et pour l’utilisateur :

1. Supposons que l’une de vos applications clientes web doive demander des autorisations spécifiques pour accéder à une ressource ou à une API. Le Portail Azure est utilisé pour déclarer les demandes d’autorisation au moment de la configuration. Comme d’autres paramètres de configuration, elles sont incluses dans les inscriptions Azure AD de l’application. Pour le chemin des demandes d’autorisations, vous devez effectuer les étapes ci-dessous :

    a. Cliquez sur **Inscriptions d’applications** à gauche du menu et ouvrez votre application en tapant son nom dans la zone de recherche.

    ![Capture d’écran montrant l’élément « Inscriptions d’applications » sélectionné dans le menu de gauche, et la zone de recherche d’« ID d’application » mise en évidence.](./media/openidoauth-tutorial/application.png)

    b. Cliquez sur **Afficher les autorisations de l’API**.

    ![Capture d’écran montrant la page « Appeler une API » avec le bouton « Afficher les autorisations de l’API » sélectionné.](./media/openidoauth-tutorial/api-permission.png)

    c. Cliquez sur **Ajouter une autorisation**.

    ![Capture d’écran montrant la page « Autorisations de l’API » avec le bouton « Ajouter une autorisation » sélectionné.](./media/openidoauth-tutorial/add-permission.png)

    d. Cliquez sur **Microsoft Graph**.

    ![Capture d’écran montrant la page « Demander des autorisations d’API » avec la sélection de l’onglet « API Microsoft » et de la vignette « Microsoft Graph ».](./media/openidoauth-tutorial/microsoft-graph.png)

    e. Sélectionnez les options appropriées dans **Autorisations déléguées** et **Autorisations d’application**.

    ![API Graph](./media/openidoauth-tutorial/graphapi.png)

2. Considérez que les autorisations de votre application ont été mises à jour. L’application est en cours d’exécution, et un utilisateur s’apprête à l’utiliser pour la première fois. L’application doit tout d’abord obtenir un code d’autorisation du point de terminaison /authorize d’Azure AD. Le code d’autorisation peut ensuite être utilisé pour obtenir un nouvel accès et un jeton d’actualisation.

3. Si l’utilisateur n’est pas déjà authentifié, le point de terminaison /authorize d’Azure AD l’invite à se connecter.

    ![Capture d’écran de l’invite de connexion pour le compte](./media/openidoauth-tutorial/authentication.png)

4. Une fois l’utilisateur connecté, Azure AD détermine s’il est nécessaire d’afficher une page de consentement pour l’utilisateur. La détermination est différente selon que l’utilisateur (ou l’administrateur de son organisation) a déjà accordé ou non son consentement à l’application.

   Si le consentement n’a pas été accordé, Azure AD invite l’utilisateur à le fournir et affiche les autorisations requises pour le fonctionnement. Les autorisations qui s’affichent dans la boîte de dialogue de consentement correspondent à celles qui sont sélectionnées dans les autorisations déléguées du portail Azure.

    ![Page de consentement](./media/openidoauth-tutorial/consentpage.png)

Un utilisateur standard peut donner son consentement pour certaines autorisations. D’autres autorisations nécessitent le consentement de l’administrateur d’un locataire.

## <a name="difference-between-admin-consent-and-user-consent"></a>Différence entre le consentement administrateur et le consentement de l’utilisateur

En tant qu’administrateur, vous pouvez également donner votre consentement pour les autorisations déléguées d’une application pour le compte de tous les utilisateurs de votre client. Le consentement administrateur évite que la boîte de dialogue de consentement ne s’affiche pour tous les utilisateurs du locataire. Les utilisateurs disposant du rôle d’administrateur peuvent indiquer leur consentement dans le portail Azure. Dans la page **Paramètres** de l’application, sélectionnez **Autorisations requises** > **Accorder le consentement administrateur**.

![Bouton Accorder des autorisations](./media/openidoauth-tutorial/grantpermission.png)

> [!NOTE]
> Pour les applications monopage (SPA) qui utilisent ADAL.js, vous devez maintenant accorder un consentement explicite à l’aide du bouton **Accorder le consentement administrateur**. Sinon, l’application échoue lorsque le jeton d’accès est demandé.

Les autorisations d’application uniquement nécessitent toujours le consentement de l’administrateur d’un locataire. Si votre application demande une autorisation application seule et qu’un utilisateur tente de se connecter à l’application, un message d’erreur s’affiche. Le message indique que l’utilisateur n’est pas en mesure de donner son consentement.

Si votre application utilise des autorisations qui nécessitent un consentement administrateur, vous devez y intégrer une option comme un bouton ou un lien afin que l’administrateur puisse démarrer l’action. La requête que votre application envoie pour cette action est une demande d’autorisation OAuth2/OpenID Connect ordinaire. Cette requête inclut le paramètre de chaîne de requête *prompt=admin_consent*.

Une fois que l’administrateur a donné son consentement et que le principal de service est créé dans le locataire du client, les demandes de connexion ultérieures n’ont pas besoin du paramètre *prompt=admin_consent*. Comme l’administrateur a décidé que les autorisations demandées sont acceptables, les autres utilisateurs du locataire n’ont plus à donner leur consentement par la suite.

L’administrateur d’un client peut empêcher les utilisateurs standard de donner son consentement aux applications. Si cette fonctionnalité est désactivée, le consentement de l’administrateur est toujours requis pour que l’application soit utilisée dans le client. Si vous souhaitez tester votre application avec le consentement de l’utilisateur final désactivé, vous trouverez le commutateur de configuration dans le [portail Azure](https://portal.azure.com/). Il se trouve dans la section [Paramètres utilisateur](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/UserSettings/menuId/) sous **Applications d’entreprise**.

Le paramètre *prompt=admin_consent* peut également être utilisé par les applications qui demandent des autorisations ne nécessitant pas le consentement de l’administrateur. Exemple d’application qui requiert une expérience : l’administrateur des locataires « s’inscrit » une fois et aucun autre utilisateur n’est invité à donner son consentement à compter de ce moment.

Imaginez qu’une application nécessite un consentement administrateur, et qu’un administrateur se connecte sans envoyer le paramètre *prompt=admin_consent*. Si l’administrateur donne son consentement pour l’application, celui-ci s’applique uniquement pour son compte d’utilisateur. Les utilisateurs standard ne peuvent toujours pas se connecter ni donner leur consentement pour l’application. Cette fonctionnalité s’avère particulièrement utile si vous souhaitez donner à l’administrateur des locataires la possibilité d’explorer votre application avant d’autoriser l’accès à d’autres utilisateurs.

## <a name="next-steps"></a>Étapes suivantes

[Configurer l’authentification unique (SSO) basée sur OIDC pour une application de votre locataire Azure Active Directory (Azure AD)](https://docs.microsoft.com/azure/active-directory/manage-apps/add-application-portal-setup-oidc-sso)

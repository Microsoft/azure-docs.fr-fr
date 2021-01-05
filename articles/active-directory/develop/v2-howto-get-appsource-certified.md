---
title: Obtenir une certification AppSource pour Azure Active Directory | Microsoft Docs
description: Découvrez comment faire certifier votre application AppSource pour Azure Active Directory.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 07/23/2020
ms.author: ryanwi
ms.reviewer: jeedes
ms.custom: aaddev
ms.openlocfilehash: 65d6151536eb92e2ab9b740a99984fb0b5977cab
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91256872"
---
# <a name="get-appsource-certified-for-azure-active-directory"></a>Obtenir une certification AppSource pour Azure Active Directory

[Microsoft AppSource](https://appsource.microsoft.com/) est une destination pour les utilisateurs professionnels permettant de découvrir, d’essayer et de gérer des applications SaaS métier (applications SaaS autonomes et module complémentaire pour des produits SaaS Microsoft existant).

Pour répertorier une application SaaS autonome sur AppSource, votre application doit accepter l’authentification unique des comptes professionnels d’une société ou d’une organisation qui dispose d’Azure Active Directory (Azure AD). Le processus de connexion doit utiliser les protocoles [OpenID Connect](v2-protocols-oidc.md) ou [OAuth 2.0](v2-oauth2-auth-code-flow.md). L’intégration SAML n’est pas acceptée pour la certification AppSource.

## <a name="multi-tenant-applications"></a>Applications multilocataires

Une *application multilocataire* est une application qui accepte les connexions des utilisateurs de toutes les entreprises ou organisations qui disposent d’Azure AD sans qu’une instance, une configuration ou un déploiement distincts ne soient nécessaires. AppSource recommande que les applications implémentent une architecture mutualisée pour activer l’expérience d’essai gratuit *d’un seul clic*.

Pour activer une architecture mutualisée sur votre application, suivez ces étapes :
1. Définissez la propriété `Multi-Tenanted` sur `Yes` dans informations d’inscription de votre application au sein du [portail Azure](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps). Par défaut, les applications créées dans le portail Azure sont configurées comme étant *[à locataire unique](#single-tenant-applications)* .
1. Mettez à jour votre code pour envoyer des demandes au point de terminaison `common`. Pour ce faire, mettez à jour le point de terminaison de `https://login.microsoftonline.com/{yourtenant}` à `https://login.microsoftonline.com/common*`.
1. Pour certaines plateformes, comme ASP.NET, vous devez également mettre à jour votre code afin d’accepter plusieurs émetteurs.

Pour plus d’informations sur l’architecture mutualisée, consultez [Comment connecter un utilisateur Azure Active Directory (Azure AD) à l’aide du modèle d’application mutualisée](howto-convert-app-to-be-multi-tenant.md).

### <a name="single-tenant-applications"></a>Applications à locataire unique

Une *application à locataire unique* est une application qui accepte uniquement les connexions des utilisateurs d’une instance Azure AD définie. Les utilisateurs externes (y compris les comptes professionnels ou scolaires d’autres organisations ou les comptes personnels) peuvent se connecter à une application à locataire unique après l’ajout de chaque utilisateur en tant que compte invité à l’instance Azure AD auprès de laquelle l’application est inscrite. 

Vous pouvez ajouter des utilisateurs en tant que comptes invités à Azure AD via la [collaboration Azure AD B2B](../external-identities/what-is-b2b.md) et vous pouvez le faire [par programmation](../../active-directory-b2c/code-samples.md). Lorsque vous utilisez B2B, les utilisateurs peuvent créer un portail en libre-service ne nécessitant pas d’invitation pour s’y connecter. Pour plus d’informations, consultez [Portail d’inscription en libre-service pour Azure AD B2B Collaboration](../external-identities/self-service-portal.md).

Les applications à locataire unique peuvent activer l’expérience *Me contacter*, mais si vous souhaitez activer l’expérience d’essai gratuit/d’un simple clic qu’AppSource recommande, activez l’architecture mutualisée de votre application à la place.

## <a name="appsource-trial-experiences"></a>Expérience d’essai gratuit AppSource

### <a name="free-trial-customer-led-trial-experience"></a>Essai gratuit (expérience d’essai gratuit menée par le client)

L’essai gratuit mené par le client est l’expérience recommandée par AppSource, car elle offre un accès d’un simple clic à votre application. L’exemple suivant montre à quoi ressemble cette expérience :

1.  Un utilisateur trouve votre application sur le site web AppSource, puis sélectionne l’option **Essai gratuit**.

    ![Montre Essai gratuit pour l’expérience d’essai gratuit menée par le client](./media/active-directory-devhowto-appsource-certified/customer-led-trial-step1.png)

2.  AppSource redirige l’utilisateur vers une URL de votre site web. Votre site web lance le processus *d’authentification unique* automatiquement (au chargement de la page).

    ![Montre la façon dont l’utilisateur est redirigé vers une URL de votre site web](./media/active-directory-devhowto-appsource-certified/customer-led-trial-step2.png)

3.  L’utilisateur est redirigé vers la page de connexion de Microsoft et fournit ses informations d’identification pour se connecter.

    ![Montre la page de connexion de Microsoft](./media/active-directory-devhowto-appsource-certified/customer-led-trial-step3.png)

4. L’utilisateur donne son consentement pour votre application.

    ![Exemple : Page de consentement pour une application](./media/active-directory-devhowto-appsource-certified/customer-led-trial-step4.png)

5.  Le processus de connexion se termine et l’utilisateur est redirigé vers votre site web.  L’utilisateur commence l’essai gratuit.

    ![Montre l’expérience vue par l’utilisateur lorsqu’il est redirigé vers votre site](./media/active-directory-devhowto-appsource-certified/customer-led-trial-step5.png)

### <a name="contact-me-partner-led-trial-experience"></a>Me contacter (expérience d’essai gratuit menée par le partenaire)

Vous pouvez utiliser l’expérience d’essai gratuit menée par le partenaire quand une opération manuelle ou à long terme doit se produire pour approvisionner l’utilisateur/la société : par exemple, votre application doit approvisionner des machines virtuelles, des instances de base de données ou des opérations prenant beaucoup de temps. Dans ce cas, après que l’utilisateur a sélectionné le bouton **Demander une version d’évaluation** et a rempli un formulaire, AppSource vous envoie les informations de contact de l’utilisateur. Lorsque vous recevez ces informations, vous pouvez approvisionner l’environnement et envoyer les instructions à l’utilisateur pour qu’il puisse accéder à l’expérience d’essai gratuit :<br/><br/>

1. Un utilisateur trouve votre application sur le site web AppSource, puis sélectionne l’option **Me contacter**.

    ![Montre l’option Me contacter pour l’expérience d’essai gratuit menée par le partenaire](./media/active-directory-devhowto-appsource-certified/partner-led-trial-step1.png)

2. L’utilisateur remplit un formulaire avec ses coordonnées.

    ![Montre un exemple de formulaire avec des coordonnées](./media/active-directory-devhowto-appsource-certified/partner-led-trial-step2.png)

3. Vous recevez les informations de l’utilisateur, configurez une instance d’évaluation et envoyez à l’utilisateur le lien hypertexte permettant d’accéder à votre application.

    ![Montre l’espace réservé pour les informations de l’utilisateur](./media/active-directory-devhowto-appsource-certified/usercontact.png)

4. L’utilisateur accède à votre application et termine le processus d’authentification unique.

    ![Montre l’écran de connexion de l’application](./media/active-directory-devhowto-appsource-certified/partner-led-trial-step3.png)

5. L’utilisateur donne son consentement pour votre application.

    ![Montre un exemple de page de consentement pour une application](./media/active-directory-devhowto-appsource-certified/partner-led-trial-step4.png)

6. Le processus de connexion se termine et l’utilisateur est redirigé vers votre site web. L’utilisateur commence l’essai gratuit.

    ![Montre l’expérience vue par l’utilisateur lorsqu’il est redirigé vers votre site](./media/active-directory-devhowto-appsource-certified/customer-led-trial-step5.png)

### <a name="more-information"></a>Informations complémentaires

Pour plus d’informations sur l’expérience d’essai gratuit AppSource, regardez [cette vidéo](https://aka.ms/trialexperienceforwebapps). 

## <a name="get-support"></a>Obtenir de l’aide

Pour l’intégration Azure AD, nous utilisons [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-active-directory+appsource) avec la communauté pour proposer de l’aide.

Nous vous recommandons vivement de poser vos questions sur Stack Overflow d’abord et de parcourir les problèmes existants pour voir si quelqu’un d’autre a déjà posé la même question. Assurez-vous que vos questions ou commentaires portent les mentions [`[azure-active-directory]` et `[appsource]`](https://stackoverflow.com/questions/tagged/azure-active-directory+appsource).

Utilisez la section Commentaires suivante pour donner votre avis et nous aider à affiner et à mettre en forme notre contenu.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur la création d’applications qui prennent en charge les connexions Azure AD, consultez [Scénarios d’authentification pour Azure AD](authentication-flows-app-scenarios.md).
- Pour plus d’informations sur comment répertorier votre application SaaS dans AppSource, consultez [les informations sur les partenaires AppSource](https://appsource.microsoft.com/partners).
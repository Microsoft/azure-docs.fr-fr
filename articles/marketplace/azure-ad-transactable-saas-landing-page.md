---
title: Créer la page d'accueil de votre offre SaaS à vendre dans le Marketplace commercial
description: Découvrez comment créer une page d’accueil pour votre offre SaaS à vendre.
author: mingshen-ms
ms.author: mingshen
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 09/02/2020
ms.openlocfilehash: 04137fef640da46ca8876811e127e109a8c3d445
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93348302"
---
# <a name="build-the-landing-page-for-your-transactable-saas-offer-in-the-commercial-marketplace"></a>Créer la page d'accueil de votre offre SaaS à vendre dans le Marketplace commercial

Cet article vous guide tout au long du processus de création d’une page d’accueil pour une application SaaS qui sera vendue sur le Marketplace commercial Microsoft.

## <a name="overview"></a>Vue d’ensemble

Vous pouvez considérer la page d’accueil comme le « vestibule » de votre offre SaaS (software as a service). Une fois que l’acheteur s’est abonné à une offre, le Marketplace commercial le dirige vers la page d’accueil pour activer et configurer son abonnement à votre application SaaS. Considérez ceci comme une étape de confirmation de commande qui permet à l’acheteur de voir ce qu’il a acheté et de confirmer les détails de son compte. À l’aide d’Azure Active Directory (Azure AD) et de Microsoft Graph, vous allez activer l’authentification unique (SSO) pour l’acheteur et obtenir des informations importantes sur lui que vous pouvez utiliser pour confirmer et activer son abonnement, notamment son nom, son adresse e-mail et son organisation.

Étant donné que les informations nécessaires pour activer l’abonnement sont limitées et fournies par Azure AD et Microsoft Graph, il ne devrait pas être nécessaire de demander des informations qui nécessitent plus que le consentement de base. Si vous avez besoin de détails sur l’utilisateur qui nécessitent un consentement supplémentaire pour votre application, vous devriez demander ces informations une fois l’activation de l’abonnement terminée. Ceci permet l’activation fluide de l’abonnement pour l’acheteur et diminue le risque d’abandon.

En général, la page d’accueil comprend les éléments suivants :

- Nom de l’offre et du plan achetés ainsi que conditions de facturation.
- Détails du compte de l’acheteur, notamment le prénom et le nom, l’organisation et l’adresse e-mail.
- Demandez à l’acheteur de confirmer ou de remplacer différents détails du compte.
- Guidez l’acheteur lors des étapes suivant l’activation. Par exemple, recevoir un e-mail de bienvenue, gérer l’abonnement, obtenir de l’aide ou lire la documentation.

> [!NOTE]
> L’acheteur sera également dirigé vers la page d’accueil dans le cadre de la gestion de son abonnement après l’activation. Une fois l’abonnement de l’acheteur activé, vous devez utiliser l’authentification unique pour permettre à l’utilisateur de se connecter. Il est recommandé de diriger l’utilisateur vers une page de profil ou de configuration de compte.

Les sections suivantes vous guideront tout au long du processus de création d’une page d’accueil :

1. [Créer une inscription d’application Azure AD](#create-an-azure-ad-app-registration) pour la page d’accueil.
1. [Utiliser un exemple de code comme point de départ](#use-a-code-sample-as-a-starting-point) pour votre application.
1. [Utiliser deux applications Azure AD pour améliorer la sécurité de la production](#use-two-azure-ad-apps-to-improve-security-in-production).
1. [Résoudre le jeton d’identification d’achat du marketplace](#resolve-the-marketplace-purchase-identification-token) ajouté à l’URL par le marketplace commercial.
1. [Lire les informations des revendications encodées dans le jeton d’ID](#read-information-from-claims-encoded-in-the-id-token) qui a été envoyé par Azure AD avec la requête après la connexion.
1. [Utiliser l’API Microsoft Graph](#use-the-microsoft-graph-api) pour recueillir des informations supplémentaires le cas échéant.

## <a name="create-an-azure-ad-app-registration"></a>Créer une inscription d’application Azure AD

Le marketplace commercial est entièrement intégrée à Azure AD. Les acheteurs arrivent sur le marketplace authentifiés avec un [compte Azure AD ou un compte Microsoft (MSA)](../active-directory/fundamentals/active-directory-whatis.md#terminology). Après l’achat, l’acheteur passe du marketplace commercial à l’URL de votre page d’accueil pour activer et gérer son abonnement à votre application SaaS. Vous devez laisser l’acheteur se connecter à votre application avec Azure AD SSO. (L’URL de la page d’accueil est spécifiée dans la page [Configuration technique](plan-saas-offer.md#technical-information) de l’offre).

La première étape de l’utilisation de l’identité consiste à s’assurer que votre page d’accueil est inscrite en tant qu’application Azure AD. Inscrire l’application vous permet d’utiliser Azure AD pour authentifier les utilisateurs et demander l’accès aux ressources utilisateur. Cela peut être considéré comme la définition de l’application, ce qui permet au service de savoir comment émettre des jetons pour l’application en fonction des paramètres de l’application.

### <a name="register-a-new-application-using-the-azure-portal"></a>Inscrire une nouvelle application à l’aide du Portail Azure

Pour commencer, suivez les instructions pour l’[inscription d’une nouvelle application](../active-directory/develop/quickstart-register-app.md). Pour permettre aux utilisateurs d’autres entreprise de visiter l’application, vous devez choisir l’une des options multilocataires lorsqu’il vous est demandé qui peut utiliser l’application.

Si vous envisagez d’interroger l’API Microsoft Graph, [configurez votre nouvelle application de manière à accéder aux API web](../active-directory/develop/quickstart-configure-app-access-web-apis.md). Lorsque vous sélectionnez les autorisations d’API pour cette application, la valeur par défaut **user.Read** suffit pour collecter des informations de base sur l’acheteur afin de rendre le processus d’intégration lisse et automatique. Ne demandez aucune autorisation d’API nommée **besoin du consentement de l’administrateur** , car cela empêchera tous les utilisateurs non-administrateurs de visiter votre page d’accueil.

Si vous avez besoin d’autorisations élevées dans le cadre de votre processus d’intégration ou de configuration, envisagez d’utiliser la fonctionnalité de [consentement incrémentiel](../active-directory/azuread-dev/azure-ad-endpoint-comparison.md) d’Azure AD afin que tous les acheteurs envoyés par le marketplace puissent interagir initialement avec la page d’accueil.

## <a name="use-a-code-sample-as-a-starting-point"></a>Utiliser un exemple de code comme point de départ

Nous avons fourni plusieurs exemples d’applications qui implémentent un site web simple avec la connexion Azure AD activée. Une fois que votre application est inscrite dans Azure AD, le panneau **Démarrage rapide** propose une liste de types d’applications et de piles de développement courants, comme le montre la figure 1. Choisissez ce qui correspond à votre environnement et suivez les instructions de téléchargement et d’installation.

**_Figure 1 : Panneau Démarrage rapide dans le portail Azure_* _

:::image type="content" source="./media/azure-ad-saas/azure-ad-quickstart-blade.png" alt-text="Montre le panneau de démarrage rapide dans le portail Azure.":::

Après avoir téléchargé le code et configuré votre environnement de développement, modifiez les paramètres de configuration dans l’application pour refléter l’ID d’application, l’ID de locataire et la clé secrète client que vous avez enregistrés lors de la procédure précédente. Notez que les étapes exactes diffèrent en fonction de l’exemple que vous utilisez.

## <a name="use-two-azure-ad-apps-to-improve-security-in-production"></a>Utiliser deux applications Azure AD pour améliorer la sécurité de la production

Cet article présente une version simplifiée de l’architecture permettant d’implémenter une page d’accueil pour votre offre SaaS dans le marketplace. Lors de l’exécution de la page en production, nous vous recommandons d’améliorer la sécurité en communiquant avec les API de traitement SaaS uniquement par le biais d’une autre application sécurisée. Cela nécessite la création de deux nouvelles applications :

- Tout d’abord, l’application de page d’accueil multilocataire décrite jusqu’à présent, à l’exception de la fonctionnalité permettant de contacter les API de traitement SaaS. Cette fonctionnalité sera déchargée vers une autre application, comme décrit ci-dessous.
- Deuxièmement, une application qui possède les communications avec les API de traitement SaaS. Cette application doit être un locataire unique, utilisée uniquement par votre organisation, et une liste de contrôle d’accès peut être établie pour que l’accès aux API soit possible uniquement à partir de cette application.

Cela permet à la solution de fonctionner dans des scénarios qui observent le principe de [séparation des préoccupations](/dotnet/architecture/modern-web-apps-azure/architectural-principles#separation-of-concerns). Par exemple, la page d’accueil utilise la première application Azure AD inscrite pour se connecter à l’utilisateur. Une fois que l’utilisateur est connecté, la page d’arrivée utilise la deuxième application Azure AD pour demander un jeton d’accès afin d’appeler celui de l’API de traitement SaaS et appeler l’opération de résolution.

## <a name="resolve-the-marketplace-purchase-identification-token"></a>Résoudre le jeton d’identification d’achat du marketplace

Lorsque l’acheteur est envoyé à votre page d’accueil, un jeton est ajouté au paramètre URL. Ce jeton est différent de celui émis par Azure AD et du jeton d’accès utilisé pour l’authentification de service à service. Il est utilisé comme entrée pour l’appel de résolution des [ASP de réalisation SaaS](./partner-center-portal/pc-saas-fulfillment-api-v2.md#resolve-a-purchased-subscription) afin d’obtenir les détails de l’abonnement. Comme pour tous les appels aux API de traitement SaaS, votre demande de service à service sera authentifiée avec un jeton d’accès basé sur l’utilisateur de l’ID d’application Azure AD de l’application pour l’authentification de service à service.

> [!NOTE]
> Dans la plupart des cas, il est préférable d’effectuer cet appel à partir d’une deuxième application à locataire unique. Consultez [Utiliser deux applications Azure AD pour améliorer la sécurité de la production](#use-two-azure-ad-apps-to-improve-security-in-production) plus haut dans cet article.

### <a name="request-an-access-token"></a>Demander un jeton d’accès

Pour authentifier votre application avec les API de traitement SaaS, vous avez besoin d’un jeton d’accès qui peut être généré en appelant le point de terminaison OAuth d’Azure AD. Consultez [Procédure à suivre pour récupérer le jeton d’autorisation de l’éditeur](./partner-center-portal/pc-saas-registration.md#how-to-get-the-publishers-authorization-token).

### <a name="call-the-resolve-endpoint"></a>Appeler le point de terminaison de résolution

Les API de traitement SaaS implémentent le point de terminaison [résoudre](./partner-center-portal/pc-saas-fulfillment-api-v2.md#resolve-a-purchased-subscription), qui peut être appelé pour confirmer la validité du jeton du marketplace et pour retourner des informations sur l’abonnement.

## <a name="read-information-from-claims-encoded-in-the-id-token"></a>Lire les informations des revendications encodées dans le jeton d’ID

Dans le cadre du flux [OpenID Connect](../active-directory/develop/v2-protocols-oidc.md), Azure AD ajoute un [jeton d’ID](../active-directory/develop/id-tokens.md) à la demande lorsque l’acheteur est envoyé à la page d’accueil. Ce jeton contient plusieurs éléments d’informations de base qui peuvent être utiles lors du processus d’activation, notamment les informations affichées dans ce tableau.

| Valeur | Description |
| ------------ | ------------- |
| aud | Public concerné par ce jeton. Dans ce cas, il doit correspondre à l’ID de votre application et être validé. |
| preferred_username | Nom d’utilisateur principal de l’utilisateur visiteur. Il peut s’agir d’une adresse e-mail, d’un numéro de téléphone ou d’un autre identificateur. |
| email | Adresse e-mail de l’utilisateur. Notez que ce champ peut être vide. |
| name | Valeur contrôlable de visu qui identifie le sujet du jeton. Dans ce cas, il s’agit du nom de l’acheteur. |
| oid | Identificateur dans le système d’identité Microsoft qui identifie l’utilisateur de façon unique dans toutes les applications. Microsoft Graph renverra cette valeur en tant que propriété d’ID pour un compte d’utilisateur donné. |
| tid | Identificateur représentant le client Azure AD d’où provient l’acheteur. Dans le cas d’une identité MSA, ce sera toujours ``9188040d-6c67-4c5b-b112-36a304b66dad``. Pour en savoir plus, consultez la note dans la section suivante : Utiliser l'API Microsoft Graph. |
| sub | Identificateur qui identifie l’utilisateur de façon unique dans cette application spécifique. |
|||

## <a name="use-the-microsoft-graph-api"></a>Utiliser l’API Microsoft Graph

Le jeton d’ID contient des informations de base pour identifier l’acheteur, mais votre processus d’activation peut nécessiter des détails supplémentaires, tels que l’entreprise de l’acheteur, pour finaliser le processus d’intégration. Utilisez l’[API Microsoft Graph](/graph/use-the-api) pour demander ces informations afin de ne pas forcer l’utilisateur à entrer de nouveau ces détails. Les autorisations standard _ *User.Read* * incluent par défaut les informations suivantes.

| Valeur | Description |
| ------------ | ------------- |
| displayName | Nom affiché dans le carnet d'adresses pour l'utilisateur. |
| givenName | Prénom de l’utilisateur. |
| jobTitle | Fonction de l’utilisateur. |
| mail | Adresse SMTP de l’utilisateur. |
| mobilePhone | Numéro de portable principal de l’utilisateur. |
| preferredLanguage | Code ISO 639-1 de la langue par défaut de l’utilisateur. |
| surname | Nom de l’utilisateur. |
|||

Des propriétés supplémentaires, telles que le nom de la société de l’utilisateur ou l’emplacement de l’utilisateur (pays), peuvent être sélectionnées pour être incluses dans la demande. Pour plus d’informations, consultez [propriétés du type de ressource utilisateur](/graph/api/resources/user?view=graph-rest-1.0&preserve-view=true#properties).

La plupart des applications inscrites à Azure AD accordent des permissions déléguées de lecture des informations de l’utilisateur à partir du locataire Azure AD de son entreprise. Toute demande d’informations de ce type adressée à Microsoft Graph doit être accompagnée d’un jeton d’accès pour l’authentification. Les étapes spécifiques pour générer le jeton d’accès dépendent de la pile de technologies que vous utilisez, mais l’exemple de code contient un exemple. Pour plus d’informations, consultez [Obtenir l’accès pour le compte d’un utilisateur](/graph/auth-v2-user).

> [!NOTE]
> Les comptes du locataire MSA (avec ID de locataire ``9188040d-6c67-4c5b-b112-36a304b66dad``) ne retourneront pas plus d’informations que ce qui a déjà été collecté avec le jeton d’ID. Vous pouvez donc ignorer cet appel à l’API Graph pour ces comptes.

## <a name="next-steps"></a>Étapes suivantes

- [Créer une offre SaaS dans la place de marché commerciale](create-new-saas-offer.md)
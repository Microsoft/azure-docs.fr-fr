---
title: Nouvelle expérience d’inscriptions d’applications dans Azure AD B2C
description: Présentation de la nouvelle expérience d’inscriptions d’applications dans Azure AD B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 05/25/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 7c9e3344ed819f607feeab88ac43a9dc286a61e0
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97109094"
---
# <a name="the-new-app-registrations-experience-for-azure-active-directory-b2c"></a>Nouvelle expérience d’inscriptions d’applications pour Azure Active Directory B2C

Nouvelle expérience **[d’inscriptions d’applications](https://aka.ms/b2cappregistrations)** pour Azure Active Directory B2C (Azure AD B2C) est à présent disponible. Si vous êtes plus à l’aise avec l’expérience **Applications** pour l’inscription d’applications pour Azure AD B2C, appelée ici « l’expérience héritée », ce guide vous permet de commencer à utiliser la nouvelle expérience.

## <a name="overview"></a>Vue d’ensemble
Auparavant, vous deviez gérer vos applications Azure AD B2C destinées aux consommateurs séparément du reste de vos applications à l’aide de l’expérience héritée. Cela signifiait différentes expériences de création d’applications à différents emplacements dans Azure.

La nouvelle expérience affiche toutes les inscriptions d’applications Azure AD B2C et les inscriptions d’applications Azure AD dans un même emplacement et permet de les gérer de manière cohérente. De la création d’une application destinée aux clients à la gestion d’une application avec des autorisations Microsoft Graph pour la gestion des ressources, vous n’avez besoin d’apprendre qu’une seule façon de faire les choses.

Vous pouvez accéder à la nouvelle expérience en accédant à **Inscriptions d’applications** dans un locataire Azure AD B2C à partir des services **Azure AD B2C** ou **Azure Active Directory** dans le portail Azure.

L’expérience d’inscriptions d’applications Azure AD B2C est basée sur l’[expérience générale d’inscriptions d’applications](https://developer.microsoft.com/identity/blogs/new-app-registrations-experience-is-now-generally-available/) pour tout locataire Azure AD, mais elle est personnalisée pour les locataires Azure AD B2C.

## <a name="whats-not-changing"></a>Ce qui ne change pas
- Vos applications et les configurations associées peuvent être trouvées telles quelles dans la nouvelle expérience. Vous n’avez pas besoin de réinscrire les applications et les utilisateurs de vos applications n’ont pas besoin de se reconnecter.

> [!NOTE]
> Pour afficher toutes les applications que vous avez créées précédemment, accédez au panneau **Inscriptions d’applications** et sélectionnez l’onglet **Toutes les applications**. Cela permet d’afficher les applications créées dans l’expérience héritée et la nouvelle expérience, ainsi que celles créées dans le service Azure AD.

## <a name="key-new-features"></a>Nouvelles fonctionnalités clés

-   Une **liste d’applications unifiée** affiche toutes vos applications qui s’authentifient auprès d’Azure AD B2C et d’Azure AD dans un emplacement pratique. En outre, vous pouvez tirer parti des fonctionnalités déjà disponibles pour les applications Azure AD, notamment la **date de création**, l’état **Certificats et secrets**, la barre de recherche et bien plus encore.

-   L’**inscription d’applications combinée** vous permet d’inscrire rapidement une application, qu’il s’agisse d’une application orientée client ou d’une application visant à accéder à Microsoft Graph.

- Le volet **Points de terminaison** vous permet d’identifier rapidement les points de terminaison appropriés pour votre scénario, notamment la configuration OpenID Connect, les métadonnées SAML, l’API Microsoft Graph et les [points de terminaison de flux d’utilisateurs OAuth 2.0](tokens-overview.md#endpoints).

- **Autorisations de l’API** et **Exposer une API** permettent une gestion plus étendue de l’étendue, des permissions et du consentement. Désormais, vous pouvez également attribuer des autorisations MS Graph et Azure AD Graph à une application.

-   **Propriétaires** et **Manifeste** sont désormais disponibles pour les applications qui s’authentifient auprès d’Azure AD B2C. Vous pouvez ajouter des propriétaires pour vos inscriptions et modifier directement les propriétés de l’application [à l’aide de l’éditeur de manifeste](../active-directory/develop/reference-app-manifest.md).


## <a name="new-supported-account-types"></a>Nouveaux types de comptes pris en charge

Dans la nouvelle expérience, vous sélectionnez un type de compte de support parmi les options suivantes :
- Comptes dans cet annuaire organisationnel uniquement
- Comptes dans un annuaire organisationnel (tout annuaire Azure AD – Multilocataire)
- Comptes dans un fournisseur d’identité ou annuaire organisationnel (pour authentifier les utilisateurs avec des flux d’utilisateurs)

Pour comprendre les différents types de comptes, sélectionnez **M’aider à choisir** dans l’expérience de création.

Dans l’expérience héritée, les applications étaient toujours créées en tant qu’applications orientées client. Pour ces applications, le type de compte est défini sur **Comptes dans un fournisseur d’identité ou annuaire organisationnel (pour authentifier les utilisateurs avec des flux d’utilisateurs)** .
> [!NOTE]
> Cette option est requise pour pouvoir exécuter les flux d’utilisateurs Azure AD B2C afin d’authentifier les utilisateurs de cette application. Découvrez [comment inscrire une application pour l’utiliser avec des flux d’utilisateurs](tutorial-register-applications.md).

Vous pouvez également recourir à cette option pour utiliser Azure AD B2C en tant que fournisseur de services SAML. [Plus d’informations](identity-provider-adfs.md)

## <a name="applications-for-devops-scenarios"></a>Applications pour les scénarios DevOps
Vous pouvez utiliser les autres types de comptes pour créer une application permettant de gérer vos scénarios DevOps, par exemple en utilisant Microsoft Graph pour charger des stratégies Identity Experience Framework ou approvisionner des utilisateurs. Découvrez [comment inscrire une application Microsoft Graph pour gérer les ressources Azure AD B2C](microsoft-graph-get-started.md).

Vous ne verrez peut-être pas toutes les autorisations de Microsoft Graph, car la plupart de ces autorisations ne s’appliquent pas aux utilisateurs consommateurs d’Azure B2C. [En savoir plus sur la gestion des utilisateurs à l’aide de Microsoft Graph](manage-user-accounts-graph-api.md).

## <a name="admin-consent-and-offline_accessopenid-scopes"></a>Consentement administrateur et étendues offline_access+openid
<!-- Azure AD B2C doesn't support user consent. That is, when a user signs into an application, the user doesn't see a screen requesting consent for the application permissions. All permissions have to be granted through admin consent.  -->

L’étendue **openid** est nécessaire pour qu’Azure AD B2C puisse connecter des utilisateurs à une application. L’étendue **offline_access** est nécessaire pour émettre des jetons d’actualisation pour un utilisateur. Ces étendues ont été ajoutées précédemment et ont reçu le consentement administrateur par défaut. À présent, vous pouvez facilement ajouter des autorisations pour ces étendues pendant le processus de création en vous assurant que l’option **Accorder le consentement administrateur aux autorisations openid et offline_access** est sélectionnée. Sinon, les autorisations de Microsoft Graph peuvent être ajoutées avec le consentement administrateur dans les paramètres **Autorisations de l’API** pour une application existante.

En savoir plus sur [les autorisations et le consentement](../active-directory/develop/v2-permissions-and-consent.md).

## <a name="platformsauthentication-reply-urlsredirect-uris"></a>Plateformes/Authentification : URL de réponse/URl de redirection
Dans l’expérience héritée, les différents types de plateformes étaient gérés sous **Propriétés** en tant qu’URL de réponse pour les API/applications web et en tant qu’URI de redirection pour les clients natifs. Les « clients natifs » sont également appelés « clients publics » et incluent des applications pour iOS, macOS, Android et d’autres types d’applications mobiles et de bureau.

Dans la nouvelle expérience, les URL de réponse et les URI de redirection sont tous deux appelés URI de redirection et se trouvent dans la section **Authentification** d’une application. Les inscriptions d’applications ne se limitent pas aux applications web ou natives. Vous pouvez utiliser le même processus d’inscription d’applications pour tous ces types de plateformes en inscrivant les URI de redirection correspondants.

Les URI de redirection doivent être associés à un type d’application, qu’elle soit web ou publique (mobile et de bureau). [En savoir plus sur les URI de redirection](../active-directory/develop/quickstart-register-app.md#add-a-redirect-uri)

<!-- Whether an application should be treated as a public client is inferred at run-time from the Redirect URI platform type, if possible. The **Treat application as a public client** setting should be set to **Yes** for flows that might not use a redirect URI, such as ROPC flows. -->

Les plateformes **iOS/macOS** et **Android** sont un type de client public. Elles offrent un moyen simple de configurer des applications iOS/macOS ou Android avec des URI de redirection correspondants pour une utilisation avec MSAL. En savoir plus sur les [options de configuration d’application](../active-directory/develop/msal-client-applications.md).


## <a name="application-certificates--secrets"></a>Certificats et secrets d’application

Dans la nouvelle expérience, au lieu du panneau **Clés**, vous utilisez le panneau **Certificats et secrets** pour gérer les certificats et les secrets. Les informations d’identification permettent aux applications de s’identifier auprès du service d’authentification lors de la réception de jetons à un emplacement adressable sur le web (à l’aide d’un schéma HTTPS). Nous vous recommandons d’utiliser un certificat au lieu d’une clé secrète client pour les scénarios d’informations d’identification du client lors de l’authentification auprès d’Azure AD. Les certificats ne peuvent pas être utilisés pour l’authentification auprès d’Azure AD B2C.


## <a name="features-not-applicable-in-azure-ad-b2c-tenants"></a>Fonctionnalités non disponibles dans les locataires Azure AD B2C
Les capacités d’inscriptions d’applications Azure AD suivantes ne s’appliquent pas aux locataires Azure AD B2C :
- **Rôles et administrateurs** : non disponible actuellement pour Azure AD B2C.
- **Personnalisation** : la personnalisation de l’interface utilisateur et de l’expérience utilisateur est configurée dans l’expérience **Personnalisation de l’entreprise** ou dans le cadre d’un flux d’utilisateurs. Apprenez à [personnaliser l’interface utilisateur dans Azure Active Directory B2C](customize-ui-with-html.md).
- **Vérification du domaine de l’éditeur** : votre application est inscrite sur *.onmicrosoft.com*, qui n’est pas un domaine vérifié. En outre, le domaine de l’éditeur est principalement utilisé pour accorder le consentement de l’utilisateur, qui ne s’applique pas aux applications Azure AD B2C pour l’authentification des utilisateurs. [En savoir plus sur le domaine de l’éditeur](../active-directory/develop/howto-configure-publisher-domain.md).
- **Configuration de jeton** : le jeton est configuré dans le cadre d’un flux d’utilisateurs plutôt que dans une application.
- L’expérience **Démarrages rapides** n’est actuellement pas disponible pour les locataires Azure AD B2C.
<!-- - The **Integration assistant** blade is currently not available for Azure AD B2C tenants. -->

## <a name="limitations"></a>Limites
La nouvelle expérience présente les limites suivantes :
- À ce stade, Azure AD B2C ne fait pas de différence entre la possibilité d’émettre des jetons d’accès ou d’ID pour des flux implicites. Les deux types de jetons sont disponibles pour le flux d’octroi implicite si l’option **Jetons d’ID** est sélectionnée dans le panneau **Authentification**.
<!-- - Azure AD B2C doesn't currently support the single-page application "SPA" app type.  -->
- L’interface utilisateur ne gère pas la modification de la valeur des comptes pris en charge. Vous devrez utiliser le manifeste de l’application, sauf si vous basculez entre une application Azure AD à locataire unique et une application mutualisée.

## <a name="next-steps"></a>Étapes suivantes

Pour prendre en main la nouvelle expérience d’inscriptions d’applications :
* Découvrez [comment inscrire une application web](tutorial-register-applications.md).
* Découvrez [comment inscrire une API web](add-web-api-application.md).
* Découvrez [comment inscrire une application cliente native](add-native-application.md).
* Découvrez [comment inscrire une application Microsoft Graph pour gérer les ressources Azure AD B2C](microsoft-graph-get-started.md).
* Découvrez [comment utiliser Azure AD B2C en tant que fournisseur de services SAML](identity-provider-adfs.md).
* En savoir plus sur les [types d’applications](application-types.md).
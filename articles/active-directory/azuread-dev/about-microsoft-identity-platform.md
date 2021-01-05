---
title: Évolution de la plateforme d’identités Microsoft - Azure
description: En savoir plus sur la plateforme d’identités Microsoft, une évolution de la plateforme de développeur et de service d’identité Azure Active Directory (Azure AD).
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: overview
ms.workload: identity
ms.date: 12/09/2019
ms.author: ryanwi
ms.reviewer: agirling, saeeda, benv, marsma
ms.custom: aaddev
ROBOTS: NOINDEX
ms.openlocfilehash: ab0057f55ddb5bf76f6fa74b0211b5ea5af24288
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96608212"
---
# <a name="evolution-of-microsoft-identity-platform"></a>Évolution de la plateforme d’identités Microsoft

La [plateforme d’identités Microsoft](../develop/index.yml) est une évolution de la plateforme de développement Azure Active Directory (Azure AD). Elle permet aux développeurs de générer des applications qui connectent les utilisateurs et obtiennent des jetons pour appeler des API comme Microsoft Graph ou des API que des développeurs ont créées. Elle se compose d’un service d’authentification, de bibliothèques open source, de fonctionnalités d’inscription et de configuration d’application (via un portail des développeurs et une API d’application), d’une documentation de développement exhaustive, d’exemples de démarrage rapide, d’exemples de code, de didacticiels, de guides pratiques et autres contenus destinés aux développeurs. La plateforme d’identités Microsoft prend en charge les protocoles standard tels qu’OAuth 2.0 et OpenID Connect.

De nombreux développeurs ont déjà utilisé la plateforme Azure AD v1.0 pour authentifier les comptes professionnels et scolaires (provisionnés par Azure AD) : ils demandent des jetons au point de terminaison Azure AD v1.0 par le biais de la bibliothèque d’authentification Azure AD (ADAL), du portail Azure pour l’inscription et la configuration de l’application, et de l’API Microsoft Graph pour la configuration programmatique de l’application.

Avec la plateforme d’identités Microsoft unifiée (v2.0), vous pouvez écrire du code une seule fois et authentifier n’importe quelle identité Microsoft dans votre application. Pour plusieurs plateformes, la bibliothèque d’authentification Microsoft (MSAL) open source et totalement prise en charge est recommandée pour être utilisée sur les points de terminaison de la plateforme d’identités. La bibliothèque MSAL est simple à utiliser, offre d’excellentes expériences d’authentification unique (SSO) pour vos utilisateurs, vous aide à atteindre des niveaux de fiabilité et de performances élevés, et est développée avec Microsoft Secure Development Lifecycle (SDL). Lorsque vous appelez des API, vous pouvez configurer votre application pour tirer parti du consentement incrémentiel, qui vous permet de retarder la demande de consentement pour les étendues plus invasives jusqu’à ce que l’utilisation de l’application garantisse cela lors de l’exécution.  La bibliothèque MSAL prend également en charge Azure Active Directory B2C, donc vos clients utilisent leurs identités de compte local, d’entreprise ou de réseau social préférées pour bénéficier d’un accès par authentification unique à vos applications et API.

Avec la plateforme d’identités Microsoft, atteignez également ces types d’utilisateurs :

- Comptes professionnels et scolaires (comptes Azure AD approvisionnés)
- Comptes personnels (comme Outlook.com ou Hotmail.com)
- Vos clients qui ont déjà leur propre adresse e-mail ou identité de réseau social (comme LinkedIn, Facebook, Google) via MSAL et Azure AD B2C

Vous pouvez utiliser le portail Azure pour inscrire et configurer votre application, et utiliser l’API Microsoft Graph pour la configuration d’application par programme.

Mettez à jour votre application à votre rythme. Les applications générées avec des bibliothèques ADAL sont toujours prises en charge. Les portefeuilles d’applications mixtes, composés d’applications générées avec des bibliothèques ADAL et d’applications générées avec des bibliothèques MSAL, sont également pris en charge. Cela signifie que les applications qui utilisent les bibliothèques ADAL et MSAL les plus récentes fournissent l’authentification unique pour l’ensemble du portefeuille, au moyen du cache de jeton partagé entre ces bibliothèques. Les applications mises à jour de la bibliothèque ADAL vers la bibliothèque MSAL conserveront l’état de connexion utilisateur lors de la mise à niveau.

## <a name="microsoft-identity-platform-experience"></a>Expérience de la plateforme d’identités Microsoft

Le diagramme suivant illustre l’expérience d’identité Microsoft à un haut niveau, notamment l’expérience d’inscription d’application, les kits de développement logiciel (SDK), les points de terminaison et les identités prises en charge.

![Plateforme d’identités Microsoft actuelle](./media/about-microsoft-identity-platform/about-microsoft-identity-platform.svg)

### <a name="app-registration-experience"></a>Expérience d’inscription d’application

L’expérience d’ **[inscription d’application](https://go.microsoft.com/fwlink/?linkid=2083908)** du portail Azure est la seule expérience de portail qui vous permet de gérer toutes les applications que vous avez intégrées à la plateforme d’identités Microsoft. Si vous utilisez le portail d’inscription d’application, commencez plutôt à utiliser l’inscription d’application du portail Azure.

Pour l’intégration à Azure AD B2C (lors de l’authentification d’identités sociales ou locales), vous devrez inscrire votre application dans un locataire Azure AD B2C. Cette expérience fait également partie du portail Azure.

Utilisez l’[API d’application](/graph/api/resources/application) pour configurer par programmation vos applications intégrées à la plateforme d’identités Microsoft pour l’authentification de n’importe quelle identité Microsoft.

### <a name="msal-libraries"></a>Bibliothèques MSAL

Vous pouvez utiliser la bibliothèque MSAL pour générer des applications qui authentifient toutes les identités Microsoft. Les bibliothèques MSAL en .NET et JavaScript sont en disponibilité générale. Les bibliothèques MSAL pour iOS et Android sont en préversion et conviennent à une utilisation en environnement de production. Nous offrons la même prise en charge de niveau de production pour les bibliothèques MSAL en préversion que pour les versions de MSAL et ADAL qui sont en disponibilité générale.

Vous pouvez également utiliser les bibliothèques MSAL pour intégrer votre application à Azure AD B2C.

### <a name="microsoft-identity-platform-endpoint"></a>Point de terminaison de la plateforme d’identités Microsoft

Le point de terminaison de la plateforme d’identités Microsoft (v2.0) est certifiée OIDC. Il fonctionne avec les bibliothèques MSAL (Microsoft Authentication Libraries) ou n’importe quelle autre bibliothèque conforme aux normes. Il implémente des étendues contrôlables de visu, conformément aux normes du secteur.

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage dans la [documentation de la plateforme d’identités Microsoft](../develop/index.yml).

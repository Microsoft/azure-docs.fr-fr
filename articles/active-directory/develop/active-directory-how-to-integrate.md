---
title: Intégration à la plateforme d’identités Microsoft | Azure
titleSuffix: Microsoft identity platform
description: Découvrez les avantages de l’intégration de votre application à Plateforme d’identités Microsoft et obtenez des ressources pour des fonctionnalités telles que la connexion simplifiée, la gestion d’identités, l’authentification multifacteur et le contrôle d’accès.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 10/01/2020
ms.author: ryanwi
ms.reviewer: jmprieur
ms.custom: aaddev, seoapril2019
ms.openlocfilehash: e9724bb2e5377f84904dc77c33375c39f3077dd4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91631407"
---
# <a name="integrating-with-microsoft-identity-platform"></a>Intégration à Plateforme d’identités Microsoft

[!INCLUDE [active-directory-devguide](../../../includes/active-directory-devguide.md)]

Dans cet article, vous découvrez les avantages de l’intégration de votre application à Plateforme d’identités Microsoft et obtenez des ressources pour l’intégration. Plateforme d’identités Microsoft et Azure Active Directory (AD) offrent aux organisations une gestion professionnelle des identités pour les applications cloud. L’intégration à Plateforme d’identités Microsoft offre à vos utilisateurs une expérience de connexion rationalisée et permet à votre application d’être conforme à la stratégie informatique.

## <a name="how-to-integrate"></a>Processus d’intégration

Il existe plusieurs façons d’intégrer votre application à Plateforme d’identités Microsoft. Bénéficiez d’autant de ces scénarios nécessaires à votre application.

### <a name="support-microsoft-identity-platform-as-a-way-to-sign-in-to-your-application"></a>Prendre en charge Plateforme d’identités Microsoft comme moyen de connexion à votre application

**Réduction des problèmes de connexion et réduction des coûts de prise en charge.** En utilisant Plateforme d’identités Microsoft pour se connecter à votre application, vos utilisateurs n’auront pas à mémoriser un nom et mot de passe de plus. En tant que développeur, vous aurez un mot de passe en moins à stocker et à protéger. Ne pas avoir à gérer les réinitialisations de mots de passe oubliés peut être en soi une économie considérable. Plateforme d’identités Microsoft alimente les connexions pour certaines des applications cloud les plus populaires au monde, notamment Microsoft 365 et Microsoft Azure. Avec des centaines de millions d’utilisateurs dans des millions d’organisations, il est fort probable que votre utilisateur soit déjà connecté à Plateforme d’identités Microsoft. En savoir plus sur [l’ajout de la prise en charge de la connexion à Plateforme d’identités Microsoft](./authentication-vs-authorization.md).

**Simplifiez l’inscription à votre application.**  Lors de l’inscription de votre application, Plateforme d’identités Microsoft peut envoyer des informations essentielles sur un utilisateur pour vous permettre de remplir au préalable votre formulaire d’inscription ou de le supprimer complètement. Les utilisateurs peuvent s’inscrire à votre application à l'aide de leur compte Azure AD via une expérience de consentement courante, similaire à celles des réseaux sociaux et des applications mobiles. Tout utilisateur peut s’inscrire et se connecter à une application qui est intégrée à Plateforme d’identités Microsoft sans avoir besoin de l’intervention des services informatiques. En savoir plus sur [l’inscription de votre application pour la connexion d’un compte Azure AD](../../app-service/configure-authentication-provider-aad.md) .

### <a name="browse-for-users-manage-user-provisioning-and-control-access-to-your-application"></a>Recherche des utilisateurs, gestion de l’attribution de privilèges d’accès des utilisateurs et contrôle de l’accès à votre application

**Rechercher des utilisateurs dans le répertoire.**  Utilisez l’API Microsoft Graph pour aider les utilisateurs à rechercher d’autres personnes dans leur organisation lorsque vous invitez d’autres personnes ou que vous leur accordez l’accès, au lieu de leur demander de taper leurs adresses e-mail. Les utilisateurs peuvent effectuer leur recherche dans une interface similaire à un carnet d’adresses, notamment en affichant les détails de la hiérarchie organisationnelle. Apprenez-en davantage sur l’[API Microsoft Graph](/graph/overview).

**Réutilisation des groupes Active Directory et des listes de distribution déjà gérées par votre client.**  Azure AD contient les groupes que votre client utilise déjà pour la distribution d’e-mail et la gestion des accès. À l’aide de l’API Microsoft Graph, réutilisez ces groupes au lieu de demander à votre client de créer et de gérer un ensemble de groupes distinct dans votre application. Les informations sur les groupes peuvent également être envoyées à votre application dans les jetons de connexion. Apprenez-en davantage sur l’[API Microsoft Graph](/graph/overview).

**Utilisez Plateforme d’identités Microsoft pour contrôler qui a accès à votre application.**  Les administrateurs et les propriétaires d’applications dans Azure AD peuvent attribuer l’accès aux applications à des utilisateurs et des groupes spécifiques. À l’aide de l’API Microsoft Graph, vous pouvez lire cette liste et l’utiliser pour contrôler le provisionnement et le déprovisionnement de ressources et l’accès au sein de votre application.

**Utilisez Plateforme d’identités Microsoft pour le contrôle d’accès en fonction des rôles.**  Les administrateurs et les propriétaires d’applications peuvent affecter des utilisateurs et des groupes aux rôles que vous définissez lorsque vous enregistrez votre application dans Plateforme d’identités Microsoft. Les informations sur les rôles sont envoyées à votre application dans des jetons de connexion et peuvent également être lues à l’aide de l’API Microsoft Graph. En savoir plus sur [l’utilisation de Plateforme d’identités Microsoft pour l’autorisation](https://cloudblogs.microsoft.com/enterprisemobility/2014/12/18/azure-active-directory-now-with-group-claims-and-application-roles/).

### <a name="get-access-to-users-profile-calendar-email-contacts-files-and-more"></a>Accéder au profil des utilisateurs, au calendrier, aux e-mails, aux contacts, aux fichiers et plus encore

**Plateforme d’identités Microsoft est le serveur d’autorisation pour Microsoft 365 et d’autres services métiers de Microsoft.**  Si vous prenez en charge Plateforme d’identités Microsoft pour la connexion à votre application ou si vous prenez en charge la liaison de vos comptes d’utilisateur actuels aux comptes d’utilisateur Azure AD à l’aide d’OAuth 2.0, vous pouvez demander l’accès en lecture et en écriture au profil utilisateur, au calendrier, aux e-mails, aux contacts, aux fichiers et autres informations de l’utilisateur. Vous pouvez écrire des événements en toute transparence dans le calendrier de l'utilisateur et lire ou écrire des fichiers dans leur OneDrive. Apprenez-en davantage sur [les API Microsoft 365](/graph/overview).

### <a name="promote-your-application-in-the-azure-and-microsoft-365-marketplaces"></a>Promotion de votre application dans les Places de marché Azure et Microsoft 365

**Promouvez votre application aux millions d'organisations qui utilisent déjà Azure AD.**  Les utilisateurs qui recherchent ces marketplace utilisent déjà un ou plusieurs services cloud, ce qui les définit en tant que clients de service cloud. En savoir plus sur la promotion de votre application dans [Azure Marketplace](https://azure.microsoft.com/marketplace/partner-program/).

**Lorsque des utilisateurs s’inscrivent à votre application, cela apparaîtra dans leur panneau d'accès Azure AD et le lanceur d'applications Microsoft 365.**  Les utilisateurs pourront rapidement et facilement retourner à votre application plus tard, ce qui améliore ainsi l’engagement utilisateur. En savoir plus sur le [panneau d'accès Azure AD](../user-help/my-apps-portal-end-user-access.md).

### <a name="secure-device-to-service-and-service-to-service-communication"></a>Sécurisation de la communication de périphérique à service et de service à service

**L’utilisation de Plateforme d’identités Microsoft pour la gestion des identités des services et des périphériques réduit la quantité de code que vous devez écrire et permet au service informatique de gérer les accès.**  Les services et périphériques peuvent obtenir des jetons de Plateforme d’identités Microsoft à l’aide d’OAuth et utiliser ces jetons pour accéder aux API web. Grâce à Plateforme d’identités Microsoft, vous pouvez éviter d’écrire un code d’authentification complexe. Étant donné que les identités des services et des périphériques sont stockées dans Azure AD, le service informatique peut gérer les clés et la révocation dans un même endroit plutôt que de faire cela séparément dans votre application.

## <a name="benefits-of-integration"></a>Avantages de l’intégration

L’intégration à Plateforme d’identités Microsoft s’accompagne d’avantages qui ne nécessitent pas l’écriture de code supplémentaire.

### <a name="integration-with-enterprise-identity-management"></a>Intégration avec la gestion des identités d’entreprise

**Aidez votre application à se conformer aux stratégies informatiques.**  Les organisations intègrent leurs systèmes de gestion des identités d’entreprise à Plateforme d’identités Microsoft. Ainsi, lorsqu’une personne quitte une organisation, elle perd automatiquement l’accès à votre application sans que le service informatique ait besoin de prendre des mesures supplémentaires. Le service informatique peut gérer quels utilisateurs peuvent accéder à votre application et déterminer quelles sont les stratégies d'accès requises (par exemple, une authentification multifacteur), réduisant ainsi la nécessité d'écrire du code pour se conformer aux stratégies d'entreprise complexes. Azure AD fournit aux administrateurs un journal d'audit détaillé des utilisateurs qui se sont connectés à votre application, ce qui permet au service informatique d’effectuer un suivi de l’utilisation.

**Azure AD étend Active Directory au cloud afin que votre application puisse être intégrée à Active Directory.**  De nombreuses organisations dans le monde utilisent Active Directory comme leur principal système de gestion des identités et des connexions et ont besoin que leurs applications fonctionnent avec Active Directory. L’intégration avec Azure AD permet d’intégrer votre application à Active Directory.

### <a name="advanced-security-features"></a>Fonctionnalités de sécurité avancées

**Authentification multifacteur**  Plateforme d’identités Microsoft fournit une authentification multifacteur native. Les administrateurs informatiques peuvent avoir besoin d'une authentification multifacteur pour accéder à votre application afin que vous n'ayez pas à écrire le code vous-même. En savoir plus sur l’ [authentification multifacteur](https://azure.microsoft.com/documentation/services/multi-factor-authentication/).

**Détection de connexion anormale**  Plateforme d’identités Microsoft traite plus d’un milliard de connexions par jour, tout en utilisant des algorithmes d’apprentissage automatique pour détecter des activités suspectes et signaler tout problème éventuel aux administrateurs informatiques. En prenant en charge la connexion à Plateforme d’identités Microsoft, votre application bénéficie de cette protection. En savoir plus sur [l’affichage des rapports d'accès Azure Active Directory](../reports-monitoring/overview-reports.md).

**Accès conditionnel**.  Outre l'authentification multifacteur, les administrateurs peuvent exiger que les utilisateurs remplissent certaines conditions avant qu’ils puissent se connecter à votre application. Les conditions qui peuvent être définies incluent la plage d'adresses IP d’un périphérique client, l'appartenance à des groupes spécifiques et l'état du périphérique utilisé pour l'accès. En savoir plus sur [l’accès conditionnel d’Azure Active Directory](../conditional-access/overview.md).

### <a name="easy-development"></a>Développement facile

**Protocoles standard de l’industrie.**  Microsoft s'engage à prendre en charge les normes industrielles. La plateforme d’identité Microsoft prend en charge les protocoles standard tels qu’OAuth 2.0 et OpenID Connect 1.0. Découvrez en plus sur les [protocoles d’authentification de plateforme d’identité Microsoft](active-directory-v2-protocols.md).

**Bibliothèques open source.**  Microsoft fournit les bibliothèques open source entièrement prises en charge pour les langages et plateformes courants afin d’accélérer le développement. Le code source est acquis sous licence sous Apache 2.0 et vous êtes libre de répliquer et de contribuer aux projets. En savoir plus sur [Microsoft Authentication Library (MSAL)](reference-v2-libraries.md).

### <a name="worldwide-presence-and-high-availability"></a>Présence dans le monde entier et haute disponibilité

**Azure AD est déployé dans des centres de données dans le monde entier et est géré et surveillé en permanence.**  Azure AD est le système de gestion des identités de Microsoft Azure et Microsoft 365 et est déployé dans 28 centres de données dans le monde entier. Les données d’Active Directory sont garanties pour être répliquées sur au moins trois centres de données. Les équilibrages de charge globaux garantissent aux utilisateurs un accès à la copie la plus fidèle d'Azure AD contenant leurs données. Ils retransmettent automatiquement les demandes vers d’autres centres de données si un problème est détecté.

## <a name="next-steps"></a>Étapes suivantes

[Commencer à écrire du code](v2-overview.md#getting-started).

[Connecter les utilisateurs à l’aide de la plateforme d’identité Microsoft](./authentication-vs-authorization.md)
---
title: Qu’est-ce que la gestion des applications dans Azure Active Directory
description: Vue d’ensemble de l’utilisation de Azure Active Directory (AD) comme système de gestion des identités et des accès (IAM) pour vos applications cloud et locales.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: overview
ms.workload: identity
ms.date: 07/01/2020
ms.author: kenwith
ms.reviewer: arvinh
ms.collection: M365-identity-device-management
ms.openlocfilehash: a157396318075522d5b3263c9cf0f749eafc2476
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94658806"
---
# <a name="what-is-application-management"></a>Qu’est-ce que la gestion des applications ?

Azure AD est un système de gestion des identités et des accès (IAM). Il offre un emplacement unique pour stocker des informations sur les identités numériques. Vous pouvez configurer vos applications logicielles de façon à ce qu’elles utilisent Azure AD comme emplacement de stockage des informations utilisateur. 

Azure AD doit être configuré pour être intégré à une application. En d’autres termes, il doit savoir quelles applications l’utilisent comme système d’identité. Le processus qui consiste à tenir Azure AD informé concernant ces applications, et qui lui indique comment il doit les gérer, est appelé « gestion des applications ».

Vous gérez les applications dans le panneau **Applications d’entreprise** situé dans la section Gérer du portail Azure Active Directory.

![Option Applications d’entreprise sous la section Gérer du portail Azure AD.](media/what-is-application-management/enterprise-applications-in-nav.png)

## <a name="what-is-an-identity-and-access-management-iam-system"></a>Qu’est-ce qu’un système de gestion des identités et des accès (IAM) ?
Une application est un élément logiciel qui est utilisé dans un but précis. La plupart des applications exigent que les utilisateurs se connectent afin que l’application puisse fournir une expérience personnalisée pour cet utilisateur particulier. En d’autres termes, l’application doit connaître l’identité de l’utilisateur qui l’utilise. En effet, elle sait quelles fonctionnalités proposer ou supprimer pour l’utilisateur.

Si chaque application a gardé une trace des utilisateurs séparément, le résultat est un ensemble de noms d’utilisateur et de connexions différents pour chaque application. Une application ne peut rien savoir des utilisateurs d’autres applications.

Un système d’identité centralisé résout ce problème en fournissant un emplacement unique pour stocker les informations utilisateur qui peuvent ensuite être utilisées par toutes les applications. Ces systèmes sont connus sous le nom de « systèmes de gestion des identités et des accès (IAM) ». Azure Active Directory est le système IAM pour le cloud Microsoft.

>[!TIP]
>Un système IAM fournit un emplacement unique pour effectuer le suivi d’identités d’utilisateurs. Azure AD est le système IAM pour le cloud Microsoft.


## <a name="why-manage-applications-with-a-cloud-solution"></a>Pourquoi gérer des applications avec une solution cloud ?

Souvent, les organisations comptent des centaines d’applications qui sont indispensables aux utilisateurs pour effectuer leur travail. Les utilisateurs accèdent à ces applications à partir de nombreux appareils et emplacements. De nouvelles applications sont ajoutées, développées et supprimées tous les jours. Avec autant d’applications et de points d’accès, il n’a jamais été aussi pertinent d’utiliser une solution basée sur le cloud pour gérer l’accès des utilisateurs à toutes ces applications.

>[!TIP]
>La galerie d’applications Azure AD contient de nombreuses applications courantes déjà préconfigurées pour fonctionner avec Azure AD en tant que fournisseur d’identité.

## <a name="how-does-azure-ad-work-with-applications"></a>Comment Azure AD fonctionne-t-il avec les applications ?

Azure AD simplifie la façon dont vous gérez vos applications en fournissant un seul système d’identité pour vos applications cloud et locales. Vous pouvez ajouter vos applications SaaS (software as a service), vos applications locales et vos applications métier (LOB) à Azure AD. Les utilisateurs se connectent une fois pour accéder de manière sécurisée et fluide à ces applications ainsi qu’aux applications Microsoft 365 et d’autres applications métier de Microsoft. Vous pouvez réduire les coûts d’administration en [automatisant le provisionnement des utilisateurs](../app-provisioning/user-provisioning.md). Vous pouvez aussi utiliser des stratégies d’authentification multifacteur et d’accès conditionnel pour fournir un accès sécurisé aux applications.

![Diagramme montrant des applications fédérées via Azure AD](media/what-is-application-management/app-management-overview.png)

## <a name="what-types-of-applications-can-i-integrate-with-azure-ad"></a>Quels types d’applications puis-je intégrer à Azure AD ?

Il existe quatre types d’applications principaux que vous pouvez ajouter à vos **applications d’entreprise** et gérer avec Azure AD :

- **Applications de la galerie Azure AD** : Azure AD a une galerie avec des milliers d’applications préintégrées pour l’authentification unique avec Azure AD. Certaines applications utilisées par votre organisation sont probablement dans la galerie. [Découvrez plus d’informations sur la planification de l’intégration de votre application](plan-an-application-integration.md) ou obtenez la procédure d’intégration détaillée pour chaque application dans les [tutoriels sur les applications SaaS](/azure/active-directory/saas-apps/).

- **Applications locales avec le proxy d’application** : Avec le proxy d’application Azure AD, vous pouvez intégrer vos applications web locales à Azure AD pour prendre en charge l’authentification unique. Les utilisateurs finaux peuvent alors accéder à vos applications web locales de la même manière qu’ils accèdent à Microsoft 365 et à d’autres applications SaaS. Consultez [Fournir un accès à distance aux applications locales par le biais du proxy d’application d’Azure AD](application-proxy.md).

- **Applications personnalisées** : Quand vous créez vos propres applications métier, vous pouvez les intégrer à Azure AD pour prendre en charge l’authentification unique. En inscrivant votre application dans Azure AD, vous pouvez contrôler sa stratégie d’authentification. Pour plus d’informations, consultez l’[aide pour les développeurs](developer-guidance-for-integrating-applications.md).

- **Applications hors galerie** : Apportez vos propres applications ! Prenez en charge l’authentification unique pour les autres applications en les ajoutant à Azure AD. Il existe plusieurs façons d’intégrer une application. Certaines d’entre elles sont indiquées ci-dessous. Pour plus d’informations, consultez [Configurer l’authentification unique SAML](configure-saml-single-sign-on.md).

>[!TIP]
>Vous pouvez intégrer Azure AD à une application, même si elle n’est pas déjà préconfigurée et qu’elle ne figure pas déjà dans la galerie d’applications. Vous pouvez **intégrer Azure AD avec n’importe lequel** des éléments suivants :
> - Tout lien web, ou toute application, qui restitue un **champ de nom d’utilisateur et de mot de passe**.
> - Toute application qui prend en charge les **protocoles SAML ou OpenID Connect**.
> - Toute application qui prend en charge le système **SCIM (System for Cross-domain Identity Management)**.

## <a name="manage-risk-with-conditional-access-policies"></a>Gérer les risques avec les stratégies d’accès conditionnel

Le couplage de l’authentification unique (SSO) Azure AD avec l’[accès conditionnel](../conditional-access/concept-conditional-access-cloud-apps.md) offre des niveaux élevés de sécurité pour l’accès aux applications. Les fonctionnalités de sécurité comprennent la protection des identités à l’échelle du cloud, le contrôle d’accès en fonction des risques, l’authentification multifacteur native et les stratégies d’accès conditionnel. Ces fonctionnalités offrent des stratégies de contrôle précises, basées sur les applications ou sur les groupes qui ont besoin d’un niveau plus élevé de sécurité.

## <a name="improve-productivity-with-single-sign-on"></a>Améliorer la productivité avec l’authentification unique

L’activation de l’authentification unique (SSO) sur les applications et Microsoft 365 offre une expérience de connexion supérieure aux utilisateurs en réduisant ou en éliminant des invites de connexion. L’environnement utilisateur semble plus cohésif et est moins perturbant sans la multiplication des invites ou l’obligation de gérer plusieurs mots de passe. Le groupe professionnel peut gérer et approuver l’accès au moyen de l’appartenance dynamique et en libre-service. Autoriser les bonnes personnes dans l’entreprise à gérer l’accès à une application améliore la sécurité du système des identités.

SSO améliore la sécurité. *Sans l’authentification unique*, les administrateurs doivent créer et mettre à jour des comptes d’utilisateur pour chaque application séparément, ce qui demande du temps. Les utilisateurs doivent aussi conserver plusieurs informations d’identification différentes pour accéder à leurs applications. Par conséquent, ils ont tendance à noter leurs mots de passe ou à utiliser d’autres solutions de gestion de mots de passe, ce qui crée d’autres risques sur la sécurité des données. [Découvrez plus d’informations sur l’authentification unique](what-is-single-sign-on.md).

## <a name="address-governance-and-compliance"></a>Assurer la gouvernance et la conformité

Avec Azure AD, vous pouvez superviser les connexions aux applications par le biais de rapports tirant parti des outils SIEM (Security Incident and Event Monitoring). Vous pouvez accéder aux rapports à partir du portail ou des API. Auditez par programmation toute personne qui a accès à vos applications, puis supprimez l’accès aux utilisateurs inactifs par l’intermédiaire des révisions d’accès.

## <a name="manage-costs"></a>Gérer les coûts

En migrant vers Azure AD, vous pouvez réduire les coûts et supprimer la contrainte que représente la gestion de votre infrastructure locale. Azure AD fournit également un accès en libre-service aux applications, ce qui fait gagner du temps aux administrateurs et aux utilisateurs. L’authentification unique élimine les mots de passe propres à l’application. L’authentification unique permet de réduire les coûts liés à la réinitialisation des mots de passe des applications et à la perte de productivité lors de la récupération des mots de passe.

Pour les applications axées sur les ressources humaines, ou d’autres applications ayant un grand nombre d’utilisateurs, vous pouvez tirer parti du provisionnement d’applications pour automatiser le processus de provisionnement et d’annulation du provisionnement des utilisateurs. Consultez [Qu’est-ce que le provisionnement d’applications ?](../app-provisioning/user-provisioning.md)

## <a name="next-steps"></a>Étapes suivantes

- [Série de guides de démarrage rapide sur la gestion des applications](view-applications-portal.md)
- [Bien démarrer avec l’intégration d’applications](plan-an-application-integration.md)
- [Découvrir comment automatiser l’attribution d’utilisateurs](../app-provisioning/user-provisioning.md)
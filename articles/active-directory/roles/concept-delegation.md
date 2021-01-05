---
title: Comprendre la délégation de rôle d'administrateur - Azure Active Directory | Microsoft Docs
description: modèles de délégation, exemples et rôle de sécurité dans Azure Active Directory
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: conceptual
ms.date: 11/05/2020
ms.author: curtand
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6f4512bea533b6394fcef025ddc19bb7989292a4
ms.sourcegitcommit: 0d171fe7fc0893dcc5f6202e73038a91be58da03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93378361"
---
# <a name="delegate-administration-in-azure-active-directory"></a>Déléguer l’administration dans Azure Active Directory

Croissance des organisations et complexité vont de pair. Une réponse courante consiste à réduire une partie de la charge de travail de la gestion de l’accès avec les rôles d’administrateur Azure Active Directory (AD). Vous pouvez attribuer le plus petit des privilèges aux utilisateurs afin qu’ils puissent accéder à leurs applications et effectuer leurs tâches. Même si vous n’affectez pas le rôle d’administrateur général à chaque propriétaire d’application, vous placez la gestion des applications sous la responsabilité des administrateurs généraux existants. De nombreuses raisons peuvent amener une organisation à opter pour une administration plus décentralisée. Cet article peut vous aider à planifier la délégation dans votre organisation.

<!--What about reporting? Who has which role and how do I audit?-->

## <a name="centralized-versus-delegated-permissions"></a>Autorisations centralisées et autorisations déléguées

À mesure que grandit une organisation, il peut être difficile de suivre les utilisateurs qui ont des rôles d’administrateur spécifiques. Si un employé a des droits d’administrateur qu’il ne devrait pas détenir, une organisation peut être plus vulnérable aux violations de la sécurité. En règle générale, le nombre d’administrateurs que vous prenez en charge et le niveau de granularité de leurs autorisations dépendent de la taille et de la complexité de votre déploiement.

* Dans les déploiements petits ou de preuve de concept, un ou quelques administrateurs effectuent toutes les tâches ; il n’existe aucune délégation. Dans ce cas, créez chaque administrateur en lui attribuant le rôle Administrateur général.
* Dans les déploiements plus grands comportant davantage de machines, d’applications et de postes de travail, plus de délégation est nécessaire. Plusieurs administrateurs peuvent avoir des responsabilités fonctionnelles plus spécifiques (rôles). Par exemple, certains peuvent être administrateurs d’identités privilégiées, d’autres administrateurs d’application. En outre, un administrateur peut gérer uniquement certains groupes d’objets tels que les appareils.
* Les déploiements encore plus grands peuvent nécessiter des autorisations plus granulaires et, éventuellement, des administrateurs ayant des rôles non conventionnels ou hybrides.

Dans le portail Azure AD, vous pouvez [voir tous les membres de n’importe quel rôle](manage-roles-portal.md), ce qui peut vous aider à vérifier rapidement le déploiement et à déléguer les autorisations.

Si vous êtes intéressé par la délégation de l’accès aux ressources Azure plutôt que par l’accès administratif dans Azure AD, consultez [Attribuer un rôle Azure](../../role-based-access-control/role-assignments-portal.md).

## <a name="delegation-planning"></a>Planification de la délégation

Le développement d’un modèle de délégation qui répond à vos besoins demande du travail. Le développement d’un modèle de délégation étant un processus de conception itératif, nous vous suggérons de suivre ces étapes :

* Définir les rôles dont vous avez besoin
* Déléguer l’administration des applications
* Accorder la possibilité d’inscrire des applications
* Déléguer l’appartenance des applications
* Développer un plan de sécurité
* Établir des comptes d’urgence
* Sécuriser vos rôles d’administrateur
* Rendre l’élévation de privilège temporaire

## <a name="define-roles"></a>Définir des rôles

Déterminez les tâches Active Directory qui sont exécutées par les administrateurs et comment elles sont mappées aux rôles. Vous pouvez [voir des descriptions détaillées des rôles](manage-roles-portal.md) dans le portail Azure.

Chaque tâche doit être évaluée du point de vue de la fréquence, de l’importance et de la difficulté. Ces critères sont des aspects vitaux de la définition de tâche, car ils déterminent si une autorisation doit être déléguée :

* Les tâches que vous effectuez de façon routinière, qui présentent un risque limité et qui sont très faciles à effectuer sont d’excellents candidats à la délégation.
* Les tâches que vous effectuez rarement, mais qui ont un impact important dans l’organisation et qui nécessitent des niveaux élevés de compétences doivent faire l’objet d’une attention particulière en vue de leur délégation. Au lieu de cela, vous pouvez [temporairement élever un compte au rôle requis](../privileged-identity-management/pim-configure.md) ou réattribuer la tâche.

## <a name="delegate-app-administration"></a>Déléguer l’administration des applications

La prolifération d’applications au sein de votre organisation peut restreindre votre modèle de délégation. S’il fait peser la gestion de l’accès aux applications sur l’administrateur général, sa surcharge est susceptible d’augmenter au fil du temps. Si vous avez accordé à des personnes le rôle Administrateur général pour des opérations telles que la configuration d’applications d’entreprise, vous pouvez désormais affecter ces opérations aux rôles les moins privilégiés suivants. Cette action permet d’améliorer votre situation de sécurité et de réduire le risque d’erreurs malheureuses. Les rôles d’administrateur d’application les plus privilégiés sont les suivants :

* Le rôle **Administrateur d’application** , qui donne la possibilité de gérer toutes les applications dans l’annuaire, y compris les inscriptions, les paramètres d’authentification unique, la gestion des licences et les affectations d’utilisateurs et de groupes, les paramètres de proxy d’application et le consentement. Il n’accorde pas la possibilité de gérer l’accès conditionnel.
* Le rôle **Administrateur d’application cloud** , qui accorde toutes les capacités de l’administrateur d’application, à ceci près qu’il n’accorde pas l’accès aux paramètres de proxy d’application (car il n’a aucune autorisation locale).

## <a name="delegate-app-registration"></a>Déléguer l’inscription des applications

Par défaut, tous les utilisateurs peuvent créer des inscriptions d’application. Pour accorder de manière sélective la possibilité de créer des inscriptions d’application :

* Définissez le paramètre **Les utilisateurs peuvent inscrire des applications** sur Non dans **Paramètres utilisateur**.
* Affectez l’utilisateur au rôle Développeur d’applications.

Pour accorder de manière sélective la possibilité d’autoriser une application à accéder aux données :

* Définissez le paramètre **Les utilisateurs peuvent autoriser les applications à accéder aux données de l’entreprise en leur nom** sur Non dans **Paramètres utilisateur**.
* Affectez l’utilisateur au rôle Développeur d’applications.

Quand un développeur d’applications crée une inscription d’application, il est automatiquement ajouté en tant que premier propriétaire.

## <a name="delegate-app-ownership"></a>Déléguer l’appartenance des applications

Pour affiner la délégation de l’accès aux applications, vous pouvez attribuer la propriété à des applications d’entreprise spécifiques. Vous complétez ainsi la prise en charge existante de l’affectation des propriétaires d’inscription d’application. La propriété est affectée par application d’entreprise dans le panneau Applications d’entreprise. L’avantage est que les propriétaires peuvent gérer uniquement les applications d’entreprise qu’ils possèdent. Par exemple, vous pouvez affecter un propriétaire pour l’application Salesforce ; ce propriétaire peut ainsi gérer l’accès à Salesforce et sa configuration, à l’exclusion de toute autre application. Une application d’entreprise peut avoir de nombreux propriétaires, et un utilisateur peut être le propriétaire de nombreuses applications d’entreprise. Il existe deux rôles de propriétaire d’application :

* Le rôle **Propriétaire d’application d’entreprise** accorde la possibilité de gérer les applications d’entreprise détenues par l’utilisateur, y compris les paramètres d’authentification unique, les affectations d’utilisateurs et de groupes et l’ajout de propriétaires. Il n’accorde pas la possibilité de gérer les paramètres de proxy d’application ou l’accès conditionnel.
* Le rôle **Propriétaire d’inscription d’application** accorde la possibilité de gérer les inscriptions d’application pour l’application détenue par l’utilisateur, y compris le manifeste d’application et l’ajout de propriétaires.

## <a name="develop-a-security-plan"></a>Développer un plan de sécurité

Azure AD fournit un guide détaillé de la planification et de l’exécution d’un plan de sécurité sur vos rôles d’administrateur Azure AD, [Sécurisation de l’accès privilégié pour les déploiements hybrides et cloud](security-planning.md).

## <a name="establish-emergency-accounts"></a>Établir des comptes d’urgence

Pour conserver l’accès à votre magasin de gestion d’identité quand un problème survient, préparez des comptes d’accès d’urgence comme indiqué dans [Créer des comptes d’administration de l’accès d’urgence](security-emergency-access.md).

## <a name="secure-your-administrator-roles"></a>Sécuriser vos rôles d’administrateur

Les attaquants qui obtiennent le contrôle de comptes privilégiés peuvent causer des dommages considérables ; vous devez donc protéger ces comptes en premier à l’aide de la [stratégie d’accès de référence](https://cloudblogs.microsoft.com/enterprisemobility/2018/06/22/baseline-security-policy-for-azure-ad-admin-accounts-in-public-preview/) qui est disponible par défaut pour toutes les organisations Azure AD (en préversion publique). La stratégie met en œuvre l’authentification multifacteur sur les comptes Azure AD privilégiés. Les rôles Azure AD suivants sont couverts par la stratégie de référence Azure AD :

* Administrateur général
* Administrateur SharePoint
* Administrateur Exchange
* Administrateur de l’accès conditionnel
* Administrateur de sécurité

## <a name="elevate-privilege-temporarily"></a>Élever le privilège temporairement

Pour la plupart des activités quotidiennes, tous les utilisateurs ne requièrent pas des droits d’administrateur général, et tous ne doivent pas être définitivement affectés au rôle Administrateur général. Ainsi, quand les utilisateurs ont besoin des autorisations d’un administrateur général, ils doivent activer l’attribution de rôle dans Azure AD [Privileged Identity Management](../privileged-identity-management/pim-configure.md) sur leur propre compte ou sur un autre compte d’administration.

## <a name="next-steps"></a>Étapes suivantes

Pour des informations de référence sur les descriptions de rôle Azure AD, consultez [Attribuer des rôles d’administrateur dans Azure AD](permissions-reference.md).
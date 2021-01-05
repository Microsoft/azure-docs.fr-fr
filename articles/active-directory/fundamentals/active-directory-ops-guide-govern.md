---
title: Guide de référence des opérations de gouvernance Azure Active Directory
description: Ce guide de référence des opérations décrit les vérifications et les actions à entreprendre pour sécuriser la gestion de la gouvernance.
services: active-directory
author: martincoetzer
manager: daveba
tags: azuread
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.subservice: fundamentals
ms.date: 10/31/2019
ms.author: martinco
ms.openlocfilehash: d4df373f78a9c74584d0e4046f7532a2190f3a3f
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92370965"
---
# <a name="azure-active-directory-governance-operations-reference-guide"></a>Guide de référence des opérations de gouvernance Azure Active Directory

Cette section du [Guide de référence des opérations Azure AD](active-directory-ops-guide-intro.md) décrit les vérifications et les mesures à prendre pour évaluer et approuver l’accès accordé aux identités privilégiées et non privilégiées, et pour auditer et contrôler les modifications apportées à l’environnement.

> [!NOTE]
> Ces suggestions sont valables à la date de publication mais peuvent évoluer. Les organisations doivent évaluer en continu leurs pratiques de gouvernance, au fil de l’évolution des produits et des services Microsoft.

## <a name="key-operational-processes"></a>Processus opérationnels clés

### <a name="assign-owners-to-key-tasks"></a>Affecter les propriétaires à des tâches clés

La gestion d’Azure Active Directory nécessite l’exécution continue de tâches et de processus opérationnels clés, qui peuvent ne pas faire partie d’un projet de déploiement. Il est cependant important de configurer ces tâches pour optimiser votre environnement. Les tâches clés et leurs propriétaires recommandés sont listés ci-après :

| Tâche | Propriétaire |
| :- | :- |
| Archiver les journaux d’audit d’Azure AD dans le système SIEM | Équipe des opérations InfoSec |
| Détection des applications qui sont gérées de façon non conforme | Équipe des opérations IAM |
| Passer régulièrement en revue l’accès aux applications | Équipe d’architecture InfoSec |
| Passer régulièrement en revue l’accès aux identités externes | Équipe d’architecture InfoSec |
| Passer régulièrement en revue qui a accès aux rôles privilégiés | Équipe d’architecture InfoSec |
| Définir des barrières de sécurité pour activer les rôles privilégiés | Équipe d’architecture InfoSec |
| Passer régulièrement en revue les octrois de consentement | Équipe d’architecture InfoSec |
| Concevoir des catalogues et des packages d’accès pour les applications et les ressources pour les employés de l’organisation | Propriétaires d’application |
| Définir des stratégies de sécurité pour affecter des utilisateurs aux packages d’accès | Équipe InfoSec + Propriétaires d’application |
| Si les stratégies incluent des workflows d’approbation, passer régulièrement en revue les approbations des workflows | Propriétaires d’application |
| Examiner les exceptions dans les stratégies de sécurité, comme les stratégies d’accès conditionnel, via des révisions des accès | Équipe des opérations InfoSec |

Quand vous passerez votre liste en revue, vous devrez peut-être affecter un propriétaire à des tâches qui en sont dépourvues, ou modifier la propriété des tâches avec propriétaires qui ne sont pas conformes aux suggestions ci-dessus.

#### <a name="owner-recommended-reading"></a>Lectures recommandées pour les propriétaires

- [Attribution de rôles d’administrateur dans Azure Active Directory](../roles/permissions-reference.md)
- [Gouvernance dans Azure](../../governance/index.yml)

### <a name="configuration-changes-testing"></a>Test des modifications de configuration

Le test de certaines modifications appelle des considérations spéciales, qui vont de techniques simples, comme le déploiement d’un sous-ensemble cible d’utilisateurs, au déploiement d’une modification dans un locataire de test parallèle. Si vous n’avez pas implémenté de stratégie de test, vous devez définir une approche des tests basée sur les recommandations du tableau ci-dessous :

| Scénario| Recommandation |
|-|-|
|Modification du type d’authentification, de l’authentification fédérée en PHS/PTA, ou vice versa| Utilisez un [déploiement échelonné](../hybrid/how-to-connect-staged-rollout.md) pour tester l’impact de la modification du type d’authentification.|
|Déploiement d’une nouvelle stratégie d’accès conditionnel ou d’une stratégie de protection des identités|Créez une stratégie d’autorité de certification et affectez-la à des utilisateurs de test.|
|Intégration d’un environnement de test d’une application|Ajoutez l’application à un environnement de production, masquez-la dans le panneau MyApps et affectez-la à utilisateurs de test pendant la phase d’assurance qualité.|
|Modification des règles de synchronisation|Effectuez les modifications dans un annuaire Azure AD Connect de test avec la même configuration que celle qui est actuellement en production (ceci est connu sous le nom de « mode de préproduction ») et analysez les résultats de CSExport. Si vous êtes satisfait, passez en production quand vous êtes prêt.|
|Modification de la personnalisation de la marque|Testez dans un locataire de test distinct.|
|Déploiement d’une nouvelle fonctionnalité|Si la fonctionnalité prend en charge le déploiement sur un ensemble d’utilisateurs cible, identifiez les utilisateurs pilotes et commencez par ceux-ci. Par exemple, la réinitialisation de mot de passe en libre-service et l’authentification multifacteur peuvent cibler des utilisateurs ou des groupes spécifiques.|
|Basculer une application depuis un fournisseur d’identité local, par exemple Active Directory, vers Azure AD|Si l’application prend en charge plusieurs configurations de fournisseur d’identité local, par exemple Salesforce, configurez les deux et testez Azure AD pendant une fenêtre de modification (au cas où l’application introduit une page HRD). Si l’application ne prend pas en charge plusieurs fournisseurs d’identité locaux, planifiez les tests pendant une fenêtre de contrôle des modifications et un temps d’arrêt du programme.|
|Mettre à jour des règles de groupes dynamiques|Créez un groupe dynamique parallèle avec la nouvelle règle. Comparez aux résultats prévus, par exemple exécutez PowerShell avec la même condition.<br>Si le test réussit, échangez les emplacements où l’ancien groupe était utilisé (si possible).|
|Migrer des licences de produits|Reportez-vous à [Modifier la licence pour un seul utilisateur au sein d’un groupe sous licence dans Azure Active Directory](../enterprise-users/licensing-groups-change-licenses.md).|
|Modifier des règles AD FS, comme l’autorisation, l’émission, l’authentification MFA|Utilisez la revendication de groupe pour cibler un sous-ensemble d’utilisateurs.|
|Modifier l’expérience d’authentification AD FS ou des modifications similaires à l’échelle d’une batterie de serveurs|Créez une batterie de serveurs parallèle avec le même nom d’hôte, implémentez les modifications de configuration, testez à partir de clients en utilisant un fichier HOSTS, des règles de routage NLB ou un routage similaire.<br>Si la plateforme cible ne prend pas en charge les fichiers HOSTS (par exemple des appareils mobiles), contrôlez la modification.|

## <a name="access-reviews"></a>Révisions d’accès

### <a name="access-reviews-to-applications"></a>Révisions de l’accès aux applications

Au fil du temps, les utilisateurs peuvent accumuler des accès à des ressources quand ils changent d’équipe et de fonction. Il est important que les propriétaires des ressources examinent régulièrement les accès aux applications et suppriment les privilèges qui ne sont plus nécessaires tout au long du cycle de vie des utilisateurs. Les [révisions d’accès](../governance/access-reviews-overview.md) d’Azure AD permettent aux organisations de gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. Les propriétaires de ressources doivent passer régulièrement en revue les accès des utilisateurs pour vérifier que seules les personnes autorisées continuent de bénéficier d’un accès. Dans l’idéal, vous devez envisager d’utiliser des révisions d’accès Azure AD pour cette tâche.

![Page de démarrage des révisions d'accès](./media/active-directory-ops-guide/active-directory-ops-img15.png)

> [!NOTE]
> Chaque utilisateur qui interagit avec les révisions d’accès doit posséder une licence Azure AD Premium P2 payante.

### <a name="access-reviews-to-external-identities"></a>Révisions des accès aux identités externes

Il est crucial que l’accès aux identités externes reste limité aux seules ressources nécessaires et seulement pour la période de temps nécessaire. Établissez un processus de révision d’accès automatisé standard pour toutes les identités externes et l’accès aux applications en utilisant des [révisions d’accès](../governance/access-reviews-overview.md) Azure AD. Si un processus existe déjà en local, envisagez d’utiliser des révisions d’accès Azure AD. Une fois qu’une application a été mise hors service ou qu’elle n’est plus utilisée, supprimez toutes les identités externes qui avaient accès à cette application.

> [!NOTE]
> Chaque utilisateur qui interagit avec les révisions d’accès doit posséder une licence Azure AD Premium P2 payante.

## <a name="privileged-account-management"></a>Gestion des comptes privilégiés

### <a name="privileged-account-usage"></a>Utilisation des comptes privilégiés

Les pirates informatiques ciblent souvent des comptes administrateur et d’autres éléments avec des accès privilégiés pour obtenir rapidement un accès à des données et à des systèmes sensibles.  Comme les utilisateurs disposant de rôles privilégiés ont tendance à s’accumuler au fil du temps, il est important de vérifier et de gérer régulièrement les accès administrateur et de fournir un accès privilégié juste-à-temps à Azure AD et aux ressources Azure.

Si aucun processus n’existe dans votre organisation pour gérer les comptes privilégiés, ou si vous avez actuellement des administrateurs qui utilisent leurs comptes d’utilisateur standard pour gérer des services et des ressources, vous devez commencer immédiatement à utiliser des comptes distincts, par exemple un pour les activités quotidiennes normales, et l’autre, configuré avec l’authentification multifacteur, pour les accès privilégiés. Mieux encore, si votre organisation dispose d’un abonnement Azure AD Premium P2, vous devez déployer immédiatement [Azure AD Privileged Identity Management](../privileged-identity-management/pim-configure.md#license-requirements) (PIM). De même, vous devez aussi examiner ces comptes privilégiés et [attribuer des rôles moins privilégiés](../roles/security-planning.md) quand c’est applicable.

Un autre aspect de la gestion des comptes privilégiés à implémenter consiste à définir des [révisions d’accès](../governance/access-reviews-overview.md) pour ces comptes, manuellement ou de façon [automatisée via Privileged Identity Management](../privileged-identity-management/pim-how-to-perform-security-review.md).

#### <a name="privileged-account-management-recommended-reading"></a>Lectures recommandées pour la gestion des comptes privilégiés

- [Rôles dans Azure AD Privileged Identity Management](../privileged-identity-management/pim-roles.md)

### <a name="emergency-access-accounts"></a>Comptes d’accès d’urgence

Les organisations doivent créer des [comptes d’urgence](../roles/security-emergency-access.md) pour se préparer à gérer Azure AD dans les cas où l’authentification cesse de fonctionner :

- Panne de composants des infrastructures d’authentification (AD FS, AD local, service MFA)
- Rotation du personnel d’administration

Pour empêcher que votre locataire ne soit verrouillé par inadvertance car vous ne pouvez pas vous connecter ou activer un compte d’utilisateur individuel existant en tant qu’administrateur, vous devez créer deux comptes d’urgence ou plus, et vérifier qu’ils sont implémentés et alignés avec les [bonnes pratiques de Microsoft](../roles/security-planning.md) et les [procédures d’urgence](../roles/security-planning.md#break-glass-what-to-do-in-an-emergency).

### <a name="privileged-access-to-azure-ea-portal"></a>Accès privilégié au portail Azure Contrat Entreprise

Le [portail Azure Contrat Entreprise (Azure EA)](https://azure.microsoft.com/blog/create-enterprise-subscription-experience-in-azure-portal-public-preview/) vous permet de créer des abonnements Azure sur un Contrat Entreprise principal, qui est un rôle puissant au sein de l’entreprise. Il est courant de démarrer la création de ce portail avant même d’avoir Azure AD en place : il est donc nécessaire d’utiliser des identités Azure AD pour le verrouiller, de supprimer les comptes personnels du portail, de vérifier que la délégation appropriée est en place et de limiter le risque de blocage.

Pour être clair, si le niveau d’autorisation du portail Contrat Entreprise est actuellement défini sur « mode mixte », vous devez supprimer les [comptes Microsoft](https://support.skype.com/en/faq/FA12059/what-is-a-microsoft-account) de tous les accès privilégiés dans ce portail et le configurer pour qu’il utilise seulement des comptes Azure AD. Si les rôles délégués du portail Contrat Entreprise ne sont pas configurés, vous devez également rechercher et implémenter des rôles délégués pour les départements et les comptes.

#### <a name="privileged-access-recommended-reading"></a>Lecture recommandée pour l’accès privilégié

- [Autorisations des rôles d'administrateur dans Azure Active Directory](../roles/permissions-reference.md)

## <a name="entitlement-management"></a>Gestion des droits d’utilisation

La [gestion des droits d’utilisation](../governance/entitlement-management-overview.md) permet aux propriétaires d’application de regrouper les ressources et de les affecter à des personnes spécifiques de l’organisation (internes et externes). La gestion des droits d’utilisation autorise l’inscription et la délégation en libre-service aux propriétaires d’entreprise, tout en conservant les stratégies de gouvernance pour accorder l’accès, définir les durées d’accès et autoriser les workflows d’approbation. 

> [!NOTE]
> La gestion des droits d’utilisation Azure AD nécessite des licences Azure AD Premium P2.

## <a name="summary"></a>Résumé

Une gouvernance des identités sécurisée comprend huit aspects. Cette liste vous aide à identifier les actions à entreprendre pour évaluer et approuver l’accès accordé aux identités privilégiées et non privilégiées, et pour auditer et contrôler les modifications apportées à l’environnement.

- Affectez des propriétaires aux tâches clés.
- Implémentez une stratégie de test.
- Utilisez les révisions d’accès Azure AD pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles.
- Établissez un processus de révision d’accès automatisé standard pour tous les types d’identités externes et l’accès aux applications.
- Établissez un processus de révision d’accès de façon à vérifier et à gérer régulièrement les accès administrateur, et fournissez un accès privilégié juste-à-temps à Azure AD et aux ressources Azure.
- Provisionnez des comptes d’urgence pour être prêt à gérer les pannes inattendues d’Azure AD.
- Verrouillez l’accès au portail Azure Contrat Entreprise.
- Implémentez la gestion des droits d’utilisation pour fournir un accès contrôlé à une collection de ressources.

## <a name="next-steps"></a>Étapes suivantes

Bien démarrer avec les [Vérifications et actions opérationnelles d’Azure AD](active-directory-ops-guide-ops.md).
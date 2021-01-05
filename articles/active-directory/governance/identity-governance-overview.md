---
title: Identity Governance - Azure Active Directory | Microsoft Docs
description: Azure Active Directory Identity Governance vous permet de bénéficier de la visibilité et des processus appropriés pour répondre aux besoins de votre organisation en termes de sécurité et de productivité des employés.
services: active-directory
documentationcenter: ''
author: ajburnle
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: markwahl-msft
ms.collection: M365-identity-device-management
ms.openlocfilehash: e02df83d4b7874a1d158aae45f1619eb543e0aec
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92362448"
---
# <a name="what-is-azure-ad-identity-governance"></a>Qu’est-ce qu’Azure AD Identity Governance ?

Azure Active Directory (Azure AD) Identity Governance vous permet de bénéficier de la visibilité et des processus appropriés pour répondre aux besoins de votre organisation en termes de sécurité et de productivité des employés. Il offre des fonctionnalités permettant de faire en sorte que chacun dispose de droits d’accès aux ressources adaptés. Ces fonctionnalités et celles d’Azure AD et d’Enterprise Mobility + Security permettent de limiter les risques d’accès en protégeant, en surveillant et en auditant l’accès aux ressources critiques, tout en garantissant la productivité des employés et des partenaires.  

Identity Governance offre aux organisations la possibilité d’effectuer les tâches suivantes vis-à-vis des employés, des partenaires et des fournisseurs, pour différents services et applications en local comme dans le cloud :

- Gouverner le cycle de vie des identités
- Gouverner le cycle de vie des accès
- Sécuriser l’accès privilégié pour l’administration

Plus précisément, ce service vise à aider les organisations à répondre à ces quatre questions clés :

- Quels utilisateurs doivent pouvoir accéder à quelles ressources ?
- Que font les utilisateurs de cet accès ?
- Des contrôles organisationnels efficaces sont-ils disponibles pour gérer l’accès ?
- Des auditeurs peuvent-ils vérifier que les contrôles fonctionnent ?

## <a name="identity-lifecycle"></a>Cycle de vie des identités

Identity Governance aide les organisations à trouver un équilibre entre *productivité* (vitesse à laquelle des personnes peuvent accéder aux ressources dont elles ont besoin, par exemple pour joindre mon organisation) et *sécurité* (comment leur accès doit évoluer au fil du temps, par exemple à la suite de modifications de leur statut professionnel).  La gestion du cycle de vie des identités constitue le fondement d’Identity Governance. Une gouvernance efficace à grande échelle implique la modernisation de l’infrastructure de gestion du cycle de vie des identités pour les applications.

![Cycle de vie des identités](./media/identity-governance-overview/identity-lifecycle.png)

Pour de nombreuses organisations, le cycle de vie des identités pour les employés est lié à la représentation de l’utilisateur concerné dans un système de GCH (gestion du capital humain).  Azure AD Premium tient automatiquement à jour les identités d’utilisateur pour les personnes représentées dans les systèmes Workday et SuccessFactors, et ce, dans Active Directory et dans Azure Active Directory, comme décrit dans le [Guide de planification d’une application RH cloud pour le provisionnement d’utilisateurs Azure Active Directory](../app-provisioning/plan-cloud-hr-provision.md).  Azure AD Premium inclut également [Microsoft Identity Manager](/microsoft-identity-manager/), qui peut importer des enregistrements à partir de systèmes locaux de gestion du capital humain (GCH) comme que SAP HCM, Oracle eBusiness et Oracle PeopleSoft.

De plus en plus de scénarios impliquent une collaboration avec des personnes extérieures à votre organisation. La collaboration [Azure AD B2B](/azure/active-directory/b2b/) permet de partager en toute sécurité les applications et services de votre organisation avec des utilisateurs invités et des partenaires externes à partir de n’importe quelle organisation, tout en conservant le contrôle sur vos propres données d’entreprise.  La [gestion des droits d’utilisation Azure AD](entitlement-management-overview.md) permet de sélectionner les utilisateurs de l’organisation autorisés à demander l’accès et à être ajoutés en tant qu’invités B2B à l’annuaire de l’organisation, et garantit que ces invités sont supprimés lorsqu’ils n’ont plus besoin de l’accès.

## <a name="access-lifecycle"></a>Cycle de vie des accès

Les organisations ont besoin d’un processus pour gérer l’accès au-delà de ce qui a été initialement configuré pour un utilisateur lors de la création de son identité.  En outre, les entreprises doivent pouvoir procéder efficacement à une mise à l’échelle pour être en mesure de développer et d’appliquer des contrôles et une stratégie d’accès de manière continue.

![Cycle de vie des accès](./media/identity-governance-overview/access-lifecycle.png)

En règle générale, le service informatique délègue aux décideurs d’entreprise la prise de décision en matière d’approbation d’accès.  De plus, le service informatique peut impliquer les utilisateurs eux-mêmes.  Par exemple, les utilisateurs accédant à des données client confidentielles dans l’application marketing d’une société en Europe doivent connaître les stratégies de l’entreprise. Les utilisateurs invités d’une organisation n’ont peut-être pas connaissance de ses exigences en matière de traitement des données.

Les organisations peuvent automatiser le processus de cycle de vie des accès à l’aide de technologies comme les [groupes dynamiques](../enterprise-users/groups-dynamic-membership.md), moyennant l’approvisionnement des utilisateurs sur des [applications SaaS](../saas-apps/tutorial-list.md) ou des [applications intégrées à SCIM](../app-provisioning/use-scim-to-provision-users-and-groups.md).  Les organisations peuvent également contrôler les [utilisateurs invités pouvant accéder aux applications locales](../external-identities/hybrid-cloud-to-on-premises.md).  Ces droits d’accès peuvent ensuite être révisés régulièrement par le biais de [révisions d’accès Azure AD](access-reviews-overview.md) récurrentes.   La [gestion des droits d’utilisation Azure AD](entitlement-management-overview.md) permet également de définir la manière dont les utilisateurs demandent l’accès pour différents packages d’appartenance aux groupes et aux équipes, rôles d’application et rôles SharePoint Online.

Lorsqu’un utilisateur tente d’accéder à des applications, Azure AD applique des stratégies [d’accès conditionnel](../conditional-access/index.yml). Par exemple, les stratégies d’accès conditionnel peuvent inclure l’affichage de [conditions d’utilisation](../conditional-access/terms-of-use.md) et un processus [garantissant que l’utilisateur a accepté ces conditions](../conditional-access/require-tou.md) avant de lui permettre d’accéder à une application.

## <a name="privileged-access-lifecycle"></a>Cycle de vie des accès privilégiés

Les autres fournisseurs ont toujours décrit l’accès privilégié comme une fonctionnalité distincte d’Identity Governance. Toutefois, chez Microsoft, nous pensons que la gouvernance de l’accès privilégié est une composante essentielle d’Identity Governance, notamment en raison du risque d’utilisation inappropriée pesant sur les organisations, inhérent à ces droits d’administrateur. Les employés, fournisseurs et sous-traitants qui bénéficient de droits d’administration doivent être gouvernés.

![Cycle de vie des accès privilégiés](./media/identity-governance-overview/privileged-access-lifecycle.png)

[Azure AD Privileged Identity Management (PIM)](../privileged-identity-management/pim-configure.md) offre des contrôles supplémentaires conçus pour sécuriser les droits d’accès aux ressources dans Azure AD, Azure et d’autres services Microsoft Online Services.  L’accès juste-à-temps et les fonctionnalités d’alerte de changement de rôle fournies par Azure AD PIM, en plus de l’authentification multifacteur et de l’accès conditionnel, offrent un ensemble complet de contrôles de gouvernance contribuant à sécuriser les ressources de votre société (annuaire, Microsoft 365 et rôles de ressources Azure). Comme avec d’autres formes d’accès, les organisations peuvent utiliser les révisions d’accès pour configurer une recertification récurrente des accès pour tous les utilisateurs bénéficiant de rôles d’administrateur.

## <a name="governance-capabilities-in-other-azure-ad-features"></a>Capacités de gouvernance dans d’autres fonctionnalités Azure AD

Outre les fonctionnalités listées ci-dessus, les fonctionnalités Azure AD supplémentaires fréquemment utilisées pour fournir des scénarios de gouvernance des identités sont les suivantes :

| Fonctionnalité | Scénario |Fonctionnalité
| ------- | --------------------- |-----|
|Cycle de vie des identités (employés)|Les administrateurs peuvent activer le provisionnement de comptes d’utilisateur à partir d’un système Workday ou SuccessFactors, d’un système RH cloud ou d’un système de ressources humaines (RH) local.|[Attribution d’utilisateurs d’un système RH cloud vers Azure AD](../app-provisioning/plan-cloud-hr-provision.md)|
|Cycle de vie des identités (invités)|Les administrateurs peuvent activer l’intégration en libre-service d’utilisateurs invités à partir d’un autre locataire Azure AD, une fédération directe, un code secret à usage unique ou des comptes Google.  Les utilisateurs invités sont automatiquement provisionnés et déprovisionnés sous réserve de stratégies de cycle de vie.|[Gestion des droits d’utilisation](entitlement-management-overview.md) avec [B2B](../external-identities/what-is-b2b.md)|
|Gestion des droits d’utilisation|Les propriétaires de ressources peuvent créer des packages d’accès contenant des applications, Teams, des groupes Azure AD et Microsoft 365, ainsi que des sites SharePoint Online.|[Gestion des droits d’utilisation](entitlement-management-overview.md)|
|Demandes d’accès|Les utilisateurs finaux peuvent faire une demande d’appartenance à un groupe ou d’accès à des applications. Les utilisateurs finaux, dont les invités d’autres organisations, peuvent demander l’accès à des packages d’accès.|[Gestion des droits d’utilisation](entitlement-management-overview.md)|
|Workflow|Les propriétaires de ressources peuvent définir les approbateurs et les approbateurs d’escalade pour les demandes d’accès, et les approbateurs pour les demandes d’activation de rôle.  |[Gestion des droits d’utilisation](entitlement-management-overview.md) et [PIM](../privileged-identity-management/pim-configure.md)|
|Gestion des stratégies et des rôles|L’administrateur peut définir des stratégies d’accès conditionnel pour l’accès aux applications au moment de l’exécution.  Les propriétaires de ressources peuvent définir des stratégies pour l’accès de l’utilisateur par le biais de packages d’accès.|Stratégies d’[accès conditionnel](../conditional-access/overview.md) et de [gestion des droits d’utilisation](entitlement-management-overview.md)|
|Certification d’accès|Les administrateurs peuvent activer la nouvelle certification d’accès récurrente pour : Les applications SaaS ou les appartenances aux groupes cloud, Azure AD ou les attributions de rôles de ressources Azure. Supprimer automatiquement l’accès à des ressources, bloquer l’accès invité et supprimer des comptes invités.|[Révisions d’accès](access-reviews-overview.md), également exposées dans [PIM](../privileged-identity-management/pim-how-to-start-security-review.md)|
|Traitement et provisionnement|Le provisionnement et le déprovisionnement automatiques dans les applications connectées à Azure AD, notamment par le biais de SCIM et dans les sites SharePoint Online. |[Attribution d’utilisateurs](../app-provisioning/user-provisioning.md)|
|Création de rapports et analytique|Les administrateurs peuvent récupérer les journaux d’audit de l’activité récente de provisionnement et de connexion des utilisateurs. Intégration à Azure Monitor et « Qui a accès » par le biais de packages d’accès.|[Rapports Azure AD](../reports-monitoring/overview-reports.md) et [supervision](../reports-monitoring/overview-monitoring.md)|
|Accès privilégié|Workflows d’accès, d’alerte et d’approbation juste-à-temps et planifiés pour les rôles Azure AD (dont les rôles personnalisés) et les rôles de ressources Azure.|[Azure AD PIM](../privileged-identity-management/pim-configure.md)|
|Audit|Les administrateurs peuvent être alertés de la création de comptes d’administrateur.|[Alertes Azure AD PIM](../privileged-identity-management/pim-how-to-configure-security-alerts.md)|

## <a name="getting-started"></a>Prise en main

Pour commencer à utiliser la gestion des droits d’utilisation, les révisions d’accès, Privileged Identity Management et les conditions d’utilisation, voir l’onglet Bien démarrer **d’Identity Governance** sur le Portail Azure.

![Prise en main d’Identity Governance](./media/identity-governance-overview/getting-started.png)


Si vous avez des commentaires sur les fonctionnalités de gouvernance des identités, cliquez sur **Vous avez des commentaires ?** dans le portail Azure pour les envoyer. L’équipe les examine régulièrement.

Il n’existe pas de solution ou de recommandation parfaite pour tous les clients. Cependant, vous pouvez vous appuyer sur les guides de configuration suivants afin d’appréhender les stratégies de base recommandées par Microsoft pour renforcer la sécurité et la productivité des employés.

- [Configurations des identités et de l’accès aux appareils](/microsoft-365/enterprise/microsoft-365-policies-configurations)
- [Sécurisation de l’accès privilégié](../roles/security-planning.md)

## <a name="appendix---least-privileged-roles-for-managing-in-identity-governance-features"></a>Annexe : rôles les moins privilégiés pour la gestion dans les fonctionnalités Identity Governance

Il est recommandé d’utiliser le rôle le moins privilégié pour effectuer des tâches d’administration dans Identity Governance. Nous vous recommandons d’utiliser Azure AD PIM pour activer un rôle en fonction des besoins afin d’effectuer ces tâches. Voici les rôles d’annuaire les moins privilégiés pour configurer les fonctionnalités d’Identity Governance :

| Fonctionnalité | Rôle moins privilégié |
| ------- | --------------------- |
| Gestion des droits d’utilisation | Administrateur d’utilisateurs (à l’exception de l’ajout de sites SharePoint Online aux catalogues, qui nécessite un administrateur général) |
| Révisions d’accès | Administrateur d’utilisateurs (à l’exception des révisions d’accès des rôles Azure ou Azure AD, qui requièrent un administrateur de rôle privilégié) |
|Privileged Identity Management | Administrateur de rôle privilégié |
| Conditions d’utilisation | Administrateur de la sécurité ou administrateur de l’accès conditionnel |

## <a name="next-steps"></a>Étapes suivantes

- [Présentation de la gestion des droits d’utilisation Azure AD](entitlement-management-overview.md)
- [Présentation des révisions d’accès Azure AD](access-reviews-overview.md)
- [Qu’est-ce qu’Azure AD Privileged Identity Management ?](../privileged-identity-management/pim-configure.md)
- [Que puis-je faire avec Conditions d’utilisation ?](../conditional-access/terms-of-use.md)
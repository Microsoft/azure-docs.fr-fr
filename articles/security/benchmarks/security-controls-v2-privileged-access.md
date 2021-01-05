---
title: Benchmark de sécurité Azure v2 – Accès privilégié
description: 'Benchmark de sécurité Azure v2 : Accès privilégié'
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 09/20/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: d6676187f87cecb7f876150d1582cde9051dd251
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92367480"
---
# <a name="security-control-v2-privileged-access"></a>Contrôle de sécurité V2 : Accès privilégié

L’accès privilégié recouvre les contrôles permettant de protéger l’accès privilégié à votre locataire Azure et à vos ressources. Cela comprend une série de contrôles visant à protéger votre modèle d’administration, vos comptes d’administration et les stations de travail à accès privilégié contre les risques délibérés et involontaires.

## <a name="pa-1-protect-and-limit-highly-privileged-users"></a>PA-1 : Protéger et limiter les utilisateurs disposant de privilèges élevés

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| PA-1 | 4.3, 4.8 | AC-2 |

Limitez le nombre de comptes d’utilisateur dotés de privilèges élevés et protégez ces comptes à un niveau élevé. Les rôles intégrés les plus critiques dans Azure AD sont Administrateur général et Administrateur de rôle privilégié, car les utilisateurs affectés à ces deux rôles peuvent déléguer des rôles Administrateur. Avec ces privilèges, les utilisateurs peuvent lire et modifier directement ou indirectement chaque ressource dans votre environnement Azure :

- Administrateur général ou Administrateur d’entreprise : Les utilisateurs disposant de ce rôle ont accès à toutes les fonctionnalités d’administration d’Azure AD, ainsi qu’aux services qui utilisent des identités Azure AD.

- Administrateur de rôle privilégié : Les utilisateurs avec ce rôle peuvent gérer les attributions de rôles dans Azure AD et Azure AD Privileged Identity Management (PIM). De plus, ce rôle permet de gérer tous les aspects de PIM et des unités administratives.

Remarque : Vous pouvez avoir d’autres rôles critiques qui doivent être régis si vous utilisez des rôles personnalisés avec certaines autorisations attribuées. Vous pouvez également appliquer des contrôles similaires au compte Administrateur des ressources métier critiques.  

Vous pouvez activer l’accès privilégié juste-à-temps (JAT) aux ressources Azure et Azure AD en utilisant Azure AD Privileged Identity Management (PIM). JAT accorde des autorisations temporaires pour effectuer des tâches privilégiées uniquement lorsque les utilisateurs en ont besoin. PIM peut également générer des alertes de sécurité en cas d’activité suspecte ou non sécurisée dans votre organisation Azure AD.

- [Autorisations de rôle Administrateur dans Azure AD](../../active-directory/roles/permissions-reference.md)

- [Utiliser les alertes de sécurité Azure Privileged Identity Management](../../active-directory/privileged-identity-management/pim-how-to-configure-security-alerts.md)

- [Sécurisation de l’accès privilégié pour les déploiements hybrides et cloud dans Azure AD](../../active-directory/roles/security-planning.md)

**Responsabilité**  : Customer

**Parties prenantes de la sécurité des clients** ( [En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Gestion des identités et des clés](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

- [Architecture de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

- [Gestion de la conformité de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Opérations de sécurité](/azure/cloud-adoption-framework/organize/cloud-security-operations-center)

## <a name="pa-2-restrict-administrative-access-to-business-critical-systems"></a>PA-2 : Limiter l’accès administratif aux systèmes critiques de l’entreprise

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| PA-2 | 13.2, 2.10 | AC-2, SC-3, SC-7 |

Isolez l’accès aux systèmes critiques en restreignant les comptes disposant d’un accès privilégié aux abonnements et aux groupes d’administration dans lesquels ils se trouvent. Veillez également à limiter l’accès aux systèmes de gestion, d’identité et de sécurité qui ont un accès administratif à vos ressources critiques, comme les contrôleurs de domaine Active Directory (DC), les outils de sécurité et les outils de gestion du système avec les agents installés sur les systèmes critiques de l’entreprise. Les attaquants qui compromettent ces systèmes de gestion et de sécurité peuvent immédiatement les armer pour compromettre les ressources stratégiques de l’entreprise. 

Pour assurer un contrôle d’accès cohérent, tous les types de contrôle d’accès doivent être alignés sur la stratégie de segmentation de votre entreprise. 

Veillez à attribuer des comptes privilégiés distincts des comptes d’utilisateur standard utilisés pour les tâches de messagerie, de navigation et de productivité.

- [Composants Azure et modèle de référence](/security/compass/microsoft-security-compass-introduction#azure-components-and-reference-model-2151)

- [Accès aux groupes d’administration](../../governance/management-groups/overview.md#management-group-access)

- [Administrateurs d’abonnements Azure](../../cost-management-billing/manage/add-change-subscription-administrator.md)

**Responsabilité**  : Customer

**Parties prenantes de la sécurité des clients** ( [En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Gestion des identités et des clés](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

- [Gestion de la conformité de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Architecture de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

## <a name="pa-3-review-and-reconcile-user-access-regularly"></a>PA-3 : Examiner et rapprocher régulièrement les accès utilisateur

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| PA-3 | 4.1, 16.9, 16.10 | AC-2 |

Vérifiez régulièrement les comptes d’utilisateur et les affectations d’accès pour vous assurer de leur validité. Vous pouvez utiliser les révisions d’accès Azure AD pour réviser les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. Les rapports Azure AD peuvent fournir des journaux pour vous aider à découvrir les comptes obsolètes. Vous pouvez également utiliser Azure AD Privileged Identity Management pour créer un workflow de rapport de révision d’accès afin de faciliter le processus de révision.
En outre, Azure Privileged Identity Management peut être configuré pour alerter en cas de création d’un nombre excessif de comptes d’administrateur et pour identifier les comptes d’administrateur obsolètes ou mal configurés. 

Remarque : Certains services Azure prennent en charge des utilisateurs et rôles locaux qui ne sont pas gérés par le biais d’Azure AD. Vous devez gérer ces utilisateurs séparément.

- [Créer une révision d’accès des rôles de ressources Azure dans Privileged Identity Management (PIM)](../../active-directory/privileged-identity-management/pim-resource-roles-start-access-review.md)

- [Utilisation des révisions d’accès et des identités Azure AD](../../active-directory/governance/access-reviews-overview.md)

**Responsabilité**  : Customer

**Parties prenantes de la sécurité des clients** ( [En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Gestion des identités et des clés](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

- [Sécurité des applications et DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Gestion de la conformité de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

## <a name="pa-4-set-up-emergency-access-in-azure-ad"></a>PA-4 : Configurer l’accès d’urgence dans Azure AD

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| PA-4 | 16 | AC-2, CP-2 |

Pour empêcher le verrouillage accidentel de votre organisation Azure AD, configurez un compte d’accès d’urgence pour conserver un accès lorsque les comptes administratifs normaux ne peuvent pas être utilisés. Les comptes d’accès d’urgence sont généralement hautement privilégiés et ne doivent pas être attribués à des personnes spécifiques. Les comptes d’accès d’urgence sont limités à des cas d’urgence ou à des scénarios « de secours » où il est impossible d’utiliser des comptes d’administration normaux.
Vous devez vous assurer que les informations d’identification (telles que le mot de passe, le certificat ou la carte à puce) des comptes d’accès d’urgence restent sécurisées et connues des seules personnes autorisées à les utiliser en cas d’urgence.

- [Gérer des comptes d’accès d’urgence dans Azure AD](../../active-directory/roles/security-emergency-access.md)

**Responsabilité**  : Customer

**Parties prenantes de la sécurité des clients** ( [En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Gestion des identités et des clés](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

- [Sécurité des applications et DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Gestion de la conformité de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Opérations de sécurité (SecOps)](/azure/cloud-adoption-framework/organize/cloud-security-operations-center)

## <a name="pa-5-automate-entitlement-management"></a>PA-5 : Automatiser la gestion des droits d'utilisation

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| PA-5 | 16 | AC-2, AC-5, PM-10 |

Utilisez les fonctionnalités de gestion des droits d’utilisation d’Azure AD pour automatiser les workflows de demandes d’accès, notamment les attributions, les révisions et l’expiration des accès. L’approbation en deux ou plusieurs étapes est également prise en charge.
- [Présentation des révisions d’accès Azure AD](../../active-directory/governance/access-reviews-overview.md) 

- [Présentation de la gestion des droits d’utilisation Azure AD](../../active-directory/governance/entitlement-management-overview.md)

**Responsabilité**  : Customer

**Parties prenantes de la sécurité des clients** ( [En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Gestion des identités et des clés](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

- [Sécurité des applications et DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Gestion de la conformité de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

## <a name="pa-6-use-privileged-access-workstations"></a>PA-6 : Utiliser des stations de travail d’accès privilégié

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| PA-6 | 4.6, 11.6, 12.12 | AC-2, SC-3,  SC-7 |

Les stations de travail sécurisées et isolées sont extrêmement importantes pour la sécurité des rôles sensibles comme les administrateurs, développeurs et opérateurs de service critique. Utilisez des stations de travail utilisateur hautement sécurisées et/ou Azure Bastion pour les tâches d’administration. Utilisez Azure Active Directory, Microsoft Defender Advanced Threat Protection (MDATP) et/ou Microsoft Intune pour déployer une station de travail utilisateur sécurisée et gérée pour les tâches d’administration. Les stations de travail sécurisées peuvent être gérées de manière centralisée pour appliquer une configuration sécurisée, notamment une authentification forte, des lignes de base logicielles et matérielles et un accès réseau et logique restreint. 

- [Comprendre les stations de travail d’accès privilégié](../../active-directory/devices/concept-azure-managed-workstation.md)

- [Déployer une station de travail d’accès privilégié](../../active-directory/devices/howto-azure-managed-workstation.md)

**Responsabilité**  : Customer

**Parties prenantes de la sécurité des clients** ( [En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Sécurité des applications et DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Opérations de sécurité (SecOps)](/azure/cloud-adoption-framework/organize/cloud-security-operations-center)

- [Gestion des identités et des clés](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

## <a name="pa-7-follow-just-enough-administration-least-privilege-principle"></a>PA-7 : Suivre le principe Just Enough Administration (privilèges minimum)

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| PA-7 | 14.6 | AC-2, AC-3, SC-3 |

Le contrôle d’accès en fonction du rôle Azure (Azure RBAC) vous permet de gérer l’accès aux ressources Azure par le biais d’attributions de rôles. Vous pouvez affecter ces rôles à des utilisateurs et regrouper des principaux de service et des identités managées. Les rôles intégrés prédéfinis pour certaines ressources peuvent être inventoriés ou interrogés par le biais d’outils tels qu’Azure CLI, Azure PowerShell et le portail Azure. Les privilèges que vous attribuez aux ressources par le biais d’Azure RBAC doivent toujours être limités à ce qui est requis par les rôles. Les privilèges limités complètent l’approche juste-à-temps (JIT) d’Azure AD Privileged Identity Management (PIM), et ces privilèges doivent être révisés régulièrement.
Utilisez des rôles intégrés pour allouer les autorisations et créez un rôle personnalisé uniquement lorsque cela est nécessaire. 

- [Présentation du contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md)

- [Comment configurer Azure RBAC](../../role-based-access-control/role-assignments-portal.md)

- [Utilisation des révisions d’accès et des identités Azure AD](../../active-directory/governance/access-reviews-overview.md)

**Responsabilité**  : Customer

**Parties prenantes de la sécurité des clients** ( [En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Sécurité des applications et DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Gestion de la conformité de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management) 

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)    

- [Gestion des identités et des clés](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

## <a name="pa-8-choose-approval-process-for-microsoft-support"></a>PA-8 : Choisir le processus d’approbation pour le support Microsoft 

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| PA-8 | 16 | AC-2, AC-3, AC-4 |

Dans le cadre des scénarios de support où Microsoft a besoin d’accéder aux données client, Customer Lockbox fournit une capacité qui vous permet de consulter et d’approuver ou refuser explicitement chaque demande d’accès aux données client.

- [Présentation de Customer Lockbox](../fundamentals/customer-lockbox-overview.md)

**Responsabilité**  : Customer

**Parties prenantes de la sécurité des clients** ( [En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Sécurité des applications et DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Gestion de la conformité de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management) 

- [Gestion des identités et des clés](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)


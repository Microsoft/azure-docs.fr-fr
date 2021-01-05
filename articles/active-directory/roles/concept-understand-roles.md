---
title: Présentation des concepts de rôle Azure Active Directory
description: Découvrez à quoi correspondent les rôles intégrés et personnalisés Azure Active Directory avec étendue des ressources dans Azure Active Directory.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: overview
ms.date: 11/20/2020
ms.author: curtand
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 968ca6fa293e7bcc01bae6f48164c48c19b8267b
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96501669"
---
# <a name="understand-roles-in-azure-active-directory"></a>Présentation des rôles dans Azure Active Directory

Il existe environ 60 rôles intégrés Azure Active Directory (Azure AD), qui sont des rôles avec un ensemble fixe d’autorisations de rôle. Pour compléter les rôles intégrés, Azure AD prend également en charge des rôles personnalisés. Utilisez des rôles personnalisés pour sélectionner les autorisations de rôle souhaitées. Par exemple, vous pouvez en créer un pour gérer des ressources Azure AD spécifiques, telles que des applications ou des principaux de service.

Cet article explique ce que sont les rôles Azure AD et comment ils peuvent être utilisés.

## <a name="how-azure-ad-roles-are-different-from-other-microsoft-365-roles"></a>Différences entre les rôles Azure AD et les autres rôles Microsoft 365

Il existe de nombreux services différents dans Microsoft 365, comme Azure AD et Intune. Certains de ces services disposent de leurs propres systèmes de contrôle d’accès en fonction du rôle, plus spécifiquement :

- Azure AD
- Exchange
- Intune
- Security Center
- Centre de conformité
- Microsoft Cloud App Security
- Commerce

D’autres services, tels que Teams, SharePoint et Managed Desktop, n’ont pas de systèmes de contrôle d’accès en fonction du rôle distincts. Ils utilisent des rôles Azure AD pour leur accès administratif. Azure dispose de son propre système de contrôle d’accès en fonction du rôle pour les ressources Azure, comme les machines virtuelles, et ce système diffère des rôles Azure AD.

Par « système distinct de contrôle d’accès en fonction du rôle », nous voulons dire qu’il existe un magasin de données différent dans lequel sont stockées les définitions de rôles et les attributions de rôles. De même, les contrôles d’accès se produisent à un point de décision de stratégie différent. Pour plus d’informations, consultez [Rôles pour les services Microsoft 365 dans Azure AD](m365-workload-docs.md) et [Rôles d’administrateur d’abonnement classique, rôles Azure et rôles Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md).

## <a name="why-some-azure-ad-roles-are-for-other-services"></a>Pourquoi certains rôles Azure AD sont-ils pour d’autres services ?

Microsoft 365 a un certain nombre de systèmes de contrôle d’accès en fonction du rôle qui ont été développés indépendamment au fil du temps, chacun avec son propre portail de service. Pour faciliter la gestion des identités dans Microsoft 365 à partir du portail Azure AD, nous avons ajouté des rôles intégrés propres aux services, chacun d’eux accordant un accès administratif à un service Microsoft 365. Le rôle d’Administrateur Exchange dans Azure AD constitue un exemple de cet ajout. Ce rôle équivaut au [groupe de rôles Gestion de l’organisation](/exchange/organization-management-exchange-2013-help) dans le système de contrôle d’accès en fonction du rôle d’Exchange, et peut gérer tous les aspects d’Exchange. De même, nous avons ajouté le rôle d’Administrateur Intune, d’Administrateur Teams, d’Administrateur SharePoint, et ainsi de suite. Les rôles propres au serveur sont l’une des catégories de rôles intégrés Azure AD décrites dans la section suivante.

## <a name="categories-of-azure-ad-roles"></a>Catégories de rôles Azure AD

Les rôles intégrés Azure AD diffèrent quant à l’emplacement où ils peuvent être utilisés. Ils peuvent appartenir aux trois grandes catégories suivantes.

- **Rôles propres à Azure AD** : ces rôles accordent des autorisations pour gérer des ressources dans Azure AD uniquement. Par exemple, les rôles Administrateur d’utilisateurs, Administrateur d’application, Administrateur de groupes accordent tous des autorisations pour gérer des ressources qui résident dans Azure AD.
- **Rôles propres à un service** : pour les principaux services Microsoft 365 (non-Azure AD), nous avons créé des rôles propres au service qui accordent des autorisations pour gérer toutes les fonctionnalités au sein du service.  Par exemple, les rôles Administrateur Microsoft Exchange, Administrateur Intune, Administrateur SharePoint et Administrateur  Teams peuvent gérer les fonctionnalités avec leurs services respectifs. L’Administrateur Exchange peut gérer les boîtes aux lettres, l’Administrateur Intune peut gérer les stratégies des appareils, l’Administrateur SharePoint peut gérer les collections de sites, l’Administrateur Teams peut gérer les qualités des appels, et ainsi de suite.
- **Rôles inter-services** : certains rôles s’appliquent à plusieurs services. Il existe deux rôles généraux : Administrateur général et Lecteur général. Tous les services Microsoft 365 honorent ces deux rôles. Il existe également des rôles liés à la sécurité, comme Administrateur de la sécurité et Lecteur Sécurité, qui accordent l’accès à plusieurs services de sécurité au sein de Microsoft 365. Par exemple, à l’aide des rôles d’Administrateur de sécurité dans Azure AD, vous pouvez gérer le Centre de sécurité Microsoft 365, Microsoft Defender Advanced Threat Protection et Microsoft Cloud App Security. De même, le rôle d’Administrateur de conformité vous permet de gérer les paramètres de conformité dans le Centre de conformité Microsoft 365, Exchange, et ainsi de suite.

![Les trois catégories de rôles intégrés Azure AD](./media/concept-understand-roles/role-overlap-diagram.png)

Le tableau suivant a pour but d’aider à la compréhension de ces catégories de rôles. Les catégories sont nommées de manière arbitraire, et ne sont pas censées impliquer d’autres fonctionnalités au-delà des [autorisations de rôle documentées](permissions-reference.md).

Category | Role
---- | ----
Rôles propres à Azure AD | Administrateur d’application<br>Développeur d’applications<br>Administrateur d’authentification<br>Administrateur de jeux de clés B2C IEF<br>Administrateur de stratégies B2C IEF<br>Administrateur d'applications cloud<br>Administrateur d’appareil cloud<br>Administrateur de l’accès conditionnel<br>Administrateurs d’appareils<br>Lecteurs de répertoires<br>Comptes de synchronisation d’annuaires<br>Enregistreurs de répertoire<br>Administrateur de flux d’utilisateurs ID externe<br>Administrateur d’attribut de flux d’utilisateurs ID externe<br>Administrateur de fournisseurs d’identité externes<br>Administrateur de groupes<br>Inviteur<br>Administrateur du support technique<br>Administrateur d’identité hybride<br>Administrateur de licence<br>Prise en charge de niveau 1 de partenaire<br>Prise en charge de niveau 2 de partenaire<br>Administrateur de mots de passe<br>Administrateur d’authentification privilégié<br>Administrateur de rôle privilégié<br>Lecteur de rapports<br>Administrateur de compte utilisateur
Rôles inter-services | Administrateur d’entreprise<br>Administrateur de conformité<br>Administrateur des données de conformité<br>Lecteur général<br>Administrateur de sécurité<br>Opérateur de sécurité<br>Lecteur Sécurité<br>Administrateur de support de service
Rôles propres à un service | Administrateur Azure DevOps<br>Administrateur Azure Information Protection<br>Administrateur de facturation<br>Administrateur de services CRM<br>Approbateur d’accès à Customer LockBox<br>Administrateur Desktop Analytics<br>Administrateur de services Exchange<br>Administrateur Insights<br>Leader d’entreprise Insights<br>Administrateur de services Intune<br>Administrateur Kaizala<br>Administrateur de services Lync<br>Lecteur de confidentialité du Centre de messages<br>Lecteur du Centre de messages<br>Utilisateur de Commerce moderne<br>Administrateur réseau<br>Administrateur d’applications Office<br>Administrateur du service Power BI<br>Administrateur de plateforme Power<br>Administrateur d’imprimantes<br>Technicien en charge des imprimantes<br>Administrateur de recherche<br>Éditeur de recherche<br>Administrateur de services SharePoint<br>Administrateur des communications Teams<br>Ingénieur de support des communications Teams<br>Spécialiste du support des communications Teams<br>Administrateur d’appareils Teams<br>Administrateur du service Teams

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble du contrôle d’accès en fonction du rôle Azure AD](custom-overview.md)
- Créer des attributions de rôles à l’aide du [portail Azure, d’Azure AD PowerShell et de l’API Graph](custom-create.md)
- [Afficher les affectations d’un rôle](custom-view-assignments.md)

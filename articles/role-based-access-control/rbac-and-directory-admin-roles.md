---
title: Rôles d’administrateur d’abonnement classique, rôles Azure et rôles Azure AD
description: 'Décrit les différents rôles dans Azure : rôles d’administrateur d’abonnements classique, rôles Azure et rôles Azure Active Directory (Azure AD)'
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.assetid: 174f1706-b959-4230-9a75-bf651227ebf6
ms.service: role-based-access-control
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.date: 07/07/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.custom: it-pro;
ms.openlocfilehash: cc4f775c3cf4c016173c485d57f203b4665c6393
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92370642"
---
# <a name="classic-subscription-administrator-roles-azure-roles-and-azure-ad-roles"></a>Rôles d’administrateur d’abonnement classique, rôles Azure et rôles Azure AD

Si vous débutez sur Azure, vous trouverez peut-être un peu difficile de comprendre l’ensemble des différents rôles dans Azure. Cet article vous aide en décrivant les rôles suivants et en indiquant quand les utiliser :
- Rôles d’administrateur d’abonnements classique
- Rôles Azure
- Rôles Azure Active Directory (Azure AD)

## <a name="how-the-roles-are-related"></a>Relation entre les rôles

Pour mieux comprendre les rôles dans Azure, connaître leur historique peut aider. Quand Azure a été publié au départ, l’accès aux ressources était géré avec seulement trois rôles d’administrateur : administrateur de comptes, administrateur de services et coadministrateur. Par la suite, le contrôle d’accès en fonction du rôle Azure (Azure RBAC) a été ajouté. Le contrôle RBAC Azure est un système d’autorisations plus récent qui fournit une gestion précise des accès aux ressources Azure. Le contrôle RBAC Azure compte de nombreux rôles intégrés, peut être assigné à différentes étendues et vous permet de créer vos propres rôles personnalisés. Pour gérer les ressources dans Azure AD, comme les utilisateurs, les groupes et les domaines, il existe plusieurs rôles Azure AD.

Le diagramme suivant est une vue d’ensemble des relations entre les rôles d’administrateur d’abonnements classique, les rôles Azure et les rôles Azure AD.

![Les différents rôles dans Azure](./media/rbac-and-directory-admin-roles/rbac-admin-roles.png)


## <a name="classic-subscription-administrator-roles"></a>Rôles d’administrateur d’abonnements classique

Dans Azure, les trois rôles d’administrateur d’abonnements classique sont Administrateur de comptes, Administrateur de services et Coadministrateur. Les administrateurs d’abonnements classiques ont un accès total à l’abonnement Azure. Ils peuvent gérer les ressources en utilisant le portail Azure, les API Azure Resource Manager et les API du modèle de déploiement classique. Le compte qui est utilisé pour l’inscription à Azure est automatiquement défini en tant qu’administrateur de compte et administrateur de services fédérés. Par la suite, des coadministrateurs peuvent être ajoutés. Les administrateurs de services et les coadministrateurs ont un accès équivalent aux utilisateurs qui ont reçu le rôle Propriétaire (rôle Azure) dans le cadre de l’abonnement. Le tableau suivant décrit les différences entre ces trois rôles d’administrateurs d’abonnements classiques.

| Administrateur d’abonnements classiques | Limite | Autorisations | Notes |
| --- | --- | --- | --- |
| Administrateur de comptes | 1 par compte Azure | <ul><li>Gérer la facturation sur le [portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)</li><li>Gestion de tous les abonnements d’un compte</li><li>Création de nouveaux abonnements</li><li>Annulation d’abonnements</li><li>Changement du mode de facturation d’un abonnement</li><li>Changement d’administrateur de services</li></ul> | Le concept est qu’il est propriétaire de la facturation de l’abonnement.<br>L’Administrateur de compte n’a pas accès au portail Azure. |
| Administrateur de services | 1 par abonnement Azure | <ul><li>Gestion des services dans le [portail Azure](https://portal.azure.com)</li><li>Annuler l’abonnement</li><li>Attribution d’utilisateurs au rôle Coadministrateur</li></ul> | Par défaut, pour un nouvel abonnement, l’administrateur de compte est également l’administrateur de services fédérés.<br>L’administrateur de services dispose de l’accès équivalent à un utilisateur qui se voit attribuer le rôle Propriétaire sur l’étendue de l’abonnement.<br>L’Administrateur de service a un accès complet au portail Azure. |
| Coadministrateur | 200 par abonnement | <ul><li>Mêmes privilèges d’accès que l’administrateur de services, à ceci près qu’il ne peut pas changer l’association des abonnements aux annuaires Azure</li><li>Attribution des utilisateurs au rôle Coadministrateur, mais ne peut pas changer d’administrateur de services</li></ul> | Le coadministrateur dispose de l’accès équivalent à un utilisateur qui se voit attribuer le rôle Propriétaire sur l’étendue de l’abonnement. |

Sur le portail Azure, vous pouvez gérer les Coadministrateurs ou afficher l’Administrateur de Service sous l’onglet **Administrateurs classiques** .

![Administrateurs d’abonnement Azure Classic sur le portail Azure](./media/rbac-and-directory-admin-roles/subscription-view-classic-administrators.png)

Sur le portail Azure, vous pouvez afficher ou modifier l’Administrateur de Service ou afficher l’Administrateur de compte dans le panneau des propriétés de votre abonnement.

![Administrateur de comptes et administrateur de services dans le portail Azure](./media/rbac-and-directory-admin-roles/account-admin.png)

Pour plus d’informations, consultez [Administrateurs d’abonnement Azure Classic](classic-administrators.md).

### <a name="azure-account-and-azure-subscriptions"></a>Compte Azure et abonnements Azure

Un compte Azure représente une relation de facturation. Un compte Azure est une identité d’utilisateur, un ou plusieurs abonnements Azure et un ensemble de ressources Azure associé. La personne qui crée le compte est l’administrateur de comptes de tous les abonnements créés dans ce compte. Cette personne est également l’administrateur de services par défaut de l’abonnement.

Les abonnements Azure vous permettent d’organiser l’accès aux ressources Azure. Ils vous permettent également de contrôler le signalement, la facturation et le paiement des ressources utilisées. Comme chaque abonnement peut avoir une configuration de facturation et de paiement différente, vous pouvez avoir différents abonnements et différents plans par bureau, service, projet, etc. Chaque service appartient à un abonnement, et l’ID d’abonnement peut être nécessaire pour les opérations de programmation.

Chaque abonnement est associé à un annuaire Azure AD. Pour identifier l’annuaire auquel est associé l’abonnement, ouvrez **Abonnements** dans le portail Azure, puis sélectionnez un abonnement pour voir l’annuaire.

Les comptes et les abonnements sont gérés sur le [portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade).

## <a name="azure-roles"></a>Rôles Azure

Le contrôle RBAC Azure est un système d’autorisations basé sur [Azure Resource Manager](../azure-resource-manager/management/overview.md) qui offre une gestion précise des accès aux ressources Azure, comme les ressources de calcul et de stockage. Le contrôle RBAC Azure compte plus de 70 rôles intégrés. Il existe quatre rôles Azure fondamentaux. Les trois premiers s’appliquent à tous les types de ressources :

| Rôle Azure | Autorisations | Notes |
| --- | --- | --- |
| [Propriétaire](built-in-roles.md#owner) | <ul><li>Accès total à toutes les ressources</li><li>Délégation de l’accès à d’autres personnes</li></ul> | L’administrateur de services et les coadministrateurs se voient attribuer le rôle Propriétaire dans l’étendue de l’abonnement<br>S’applique à tous les types de ressources. |
| [Contributeur](built-in-roles.md#contributor) | <ul><li>Création et gestion de tous les types de ressources Azure</li><li>Créer un locataire dans Azure Active Directory</li><li>Ne peut pas accorder l’accès à d’autres personnes</li></ul> | S’applique à tous les types de ressources. |
| [Lecteur](built-in-roles.md#reader) | <ul><li>Consultation des ressources Azure</li></ul> | S’applique à tous les types de ressources. |
| [Administrateur de l'accès utilisateur](built-in-roles.md#user-access-administrator) | <ul><li>Gestion de l’accès utilisateur aux ressources Azure</li></ul> |  |

Les autres rôles intégrés permettent de gérer des ressources Azure spécifiques. Par exemple, le rôle [Contributeur de machines virtuelles](built-in-roles.md#virtual-machine-contributor) permet à l’utilisateur de créer et de gérer des machines virtuelles. Pour obtenir la liste de tous les rôles intégrés, consultez [Rôles intégrés Azure](built-in-roles.md).

Le contrôle RBAC Azure est pris en charge uniquement par le portail Azure et les API Azure Resource Manager. Les utilisateurs, les groupes et les applications qui se voient attribuer des rôles Azure ne peuvent pas utiliser les [API du modèle de déploiement classique Azure](../azure-resource-manager/management/deployment-models.md).

Dans le portail Azure, les attributions de rôles avec le contrôle RBAC Azure s’affichent dans le panneau **Contrôle d’accès (IAM)** . Ce panneau se trouve dans le portail, notamment pour les groupes d’administration, les abonnements, les groupes de ressources et diverses ressources.

![Panneau Contrôle d’accès IAM dans le portail Azure](./media/rbac-and-directory-admin-roles/access-control-role-assignments.png)

Quand vous cliquez sur l’onglet **Rôles** , vous voyez la liste des rôles intégrés et personnalisés.

![Rôles intégrés dans le portail Azure](./media/rbac-and-directory-admin-roles/roles-list.png)

Pour plus d’informations, consultez [Ajouter ou supprimer des attributions de rôles Azure à l’aide du portail Azure](role-assignments-portal.md).

## <a name="azure-ad-roles"></a>Rôles Azure AD

Les rôles Azure AD sont utilisés pour gérer les ressources Azure AD d’un annuaire, par exemple pour créer ou changer des utilisateurs, attribuer des rôles d’administration à d’autres personnes, réinitialiser les mots de passe des utilisateurs, gérer les licences utilisateur et gérer les domaines. Le tableau suivant décrit quelques-uns des rôles Azure AD plus importants.

| Rôle Azure AD | Autorisations | Notes |
| --- | --- | --- |
| [Administrateur général](../active-directory/roles/permissions-reference.md#company-administrator-permissions) | <ul><li>Gestion de l’accès à toutes les fonctionnalités d’administration dans Azure Active Directory, ainsi que les services qui sont fédérés à Azure Active Directory</li><li>Attribution des rôles d’administrateur à d’autres personnes</li><li>Réinitialisation des mots de passe des utilisateurs et de tous les autres administrateurs</li></ul> | La personne qui s’inscrit au locataire Azure Active Directory devient administrateur général. |
| [Administrateur d’utilisateurs](../active-directory/roles/permissions-reference.md#user-administrator) | <ul><li>Création et gestion de tous les aspects liés aux utilisateurs et aux groupes</li><li>Gestion des tickets de support</li><li>Suivi de l’intégrité des services</li><li>Changement des mots de passe des utilisateurs, des administrateurs du support technique et autres administrateurs d’utilisateurs</li></ul> |  |
| [Administrateur de facturation](../active-directory/roles/permissions-reference.md#billing-administrator) | <ul><li>Achats</li><li>Gérer les abonnements</li><li>Gestion des tickets de support</li><li>Suivi de l’intégrité des services</li></ul> |  |

Dans le portail Azure, vous pouvez voir la liste des rôles Azure AD dans le panneau **Rôles et administrateurs** . Pour obtenir une liste de tous les rôles Azure AD, consultez [Autorisations des rôles d’administrateur dans Azure Active Directory](../active-directory/roles/permissions-reference.md).

![Rôles Azure AD dans le portail Azure](./media/rbac-and-directory-admin-roles/directory-admin-roles.png)

## <a name="differences-between-azure-roles-and-azure-ad-roles"></a>Différences entre les rôles Azure et les rôles Azure AD

Globalement, les rôles Azure contrôlent les autorisations pour gérer les ressources Azure, tandis que les rôles Azure AD contrôlent les autorisations pour gérer les ressources Azure Active Directory. Le tableau suivant compare quelques différences.

| Rôles Azure | Rôles Azure AD |
| --- | --- |
| Gérer l’accès aux ressources Azure | Gérer l’accès aux ressources Azure Active Directory |
| Prise en charge des rôles personnalisés | Prise en charge des rôles personnalisés |
| L’étendue peut être spécifiée à plusieurs niveaux (groupe d’administration, abonnement, groupe de ressources, ressource) | L’étendue est au niveau du locataire |
| Les informations sur les rôles sont accessibles dans le portail Azure, Azure CLI, Azure PowerShell, les modèles Azure Resource Manager et l’API REST | Les informations sur les rôles sont accessibles dans le portail d’administration Azure, le centre d’administration Office 365, Microsoft Graph et AzureAD PowerShell |

### <a name="do-azure-roles-and-azure-ad-roles-overlap"></a>Les rôles Azure et les rôles Azure AD se chevauchent-ils ?

Par défaut, les rôles Azure et les rôles Azure AD ne couvrent pas Azure et Azure AD. Toutefois, si un administrateur général élève son accès en choisissant le commutateur **Gestion de l’accès pour les ressources Azure** dans le portail Azure, il reçoit le rôle [Administrateur de l’accès utilisateur](built-in-roles.md#user-access-administrator) (rôle Azure) sur tous les abonnements d’un locataire spécifique. Le rôle Administrateur de l’accès utilisateur permet à l’utilisateur d’accorder à d’autres utilisateurs l’accès aux ressources Azure. Ce commutateur peut être utile pour récupérer l’accès à un abonnement. Pour plus d’informations, consultez [Élever l’accès pour gérer tous les abonnements et groupes d’administration Azure](elevate-access-global-admin.md).

Plusieurs rôles Azure AD couvrent Azure AD et Microsoft 365, tels que les rôles d’administrateur général et d’administrateur d’utilisateurs. Par exemple, si vous êtes membre du rôle Administrateur général, vous disposez des fonctionnalités de l’administrateur général dans Azure AD et Microsoft 365, avec par exemple la possibilité d’apporter des changements dans Microsoft Exchange et Microsoft SharePoint. Toutefois, par défaut, l’administrateur général n’a pas accès aux ressources Azure.

![Rôles RBAC Azure et rôles Azure AD](./media/rbac-and-directory-admin-roles/azure-office-roles.png)

## <a name="next-steps"></a>Étapes suivantes

- [Qu’est-ce que le contrôle d’accès en fonction du rôle Azure (RBAC Azure) ?](overview.md)
- [Autorisations des rôles d'administrateur dans Azure Active Directory](../active-directory/roles/permissions-reference.md)
- [Administrateurs d’abonnement Azure Classic](classic-administrators.md)

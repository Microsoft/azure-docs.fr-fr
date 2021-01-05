---
title: Rechercher et filtrer des membres et propriétaires de groupes (préversion) – Azure Active Directory | Microsoft Docs
description: Rechercher et filtrer des membres et propriétaires de groupes dans le portail Azure.
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
ms.service: active-directory
ms.subservice: enterprise-users
ms.workload: identity
ms.topic: how-to
ms.date: 12/02/2020
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: d78a2a4e0f14d99a7a1ecada915857f59422bb58
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96547370"
---
# <a name="search-groups-and-members-preview-in-azure-active-directory"></a>Rechercher des groupes et des membres (préversion) dans Azure Active Directory

Cet article explique comment rechercher des membres et des propriétaires d’un groupe, ainsi que comment utiliser des filtres de recherche dans le cadre de la préversion d’amélioration des groupes dans le portail Azure Active Directory (Azure AD). De nombreuses améliorations ont été apportées à l’expérience des groupes pour vous aider à gérer vos groupes, ainsi que leurs membres et propriétaires, rapidement et facilement. Pour plus d’informations sur les préversions, consultez [Conditions d’utilisation supplémentaires pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Les modifications apportées à cette préversion sont les suivantes :

- Nouvelles fonctionnalités de recherche de groupes, telles que la recherche de sous-chaînes dans des noms de groupes
- Nouvelles options de filtrage et de tri sur des listes de membres et de propriétaires
- Nouvelles fonctionnalités de recherche pour les listes de membres et de propriétaires
- Comptage de groupes plus précis pour les grands groupes

## <a name="enabling-and-managing-the-preview"></a>Activation et gestion de la préversion

Nous avons facilité l’accès à la préversion :

  1. Connectez-vous au [portail Azure](https://portal.azure.com), puis sélectionnez **Groupes**.
  2. Dans la page Groupes – Tous les groupes, sélectionnez la bannière en haut pour accéder à la préversion.

Vous pouvez également découvrir les fonctionnalités et améliorations les plus récentes en sélectionnant le lien **Informations d’aperçu** dans la page **Tous les groupes**. Lorsque vous accédez à la préversion, vous pouvez voir la balise de la préversion sur toutes les pages de groupes qui ont fait l’objet d’améliorations et font partie de la préversion. Certaines pages de groupes n’ont pas été mises à jour dans le cadre de cette préversion.

Si vous rencontrez des problèmes, vous pouvez revenir à l’expérience héritée en sélectionnant la bannière en haut de la page **Tous les groupes**. Vos commentaires sont les bienvenus, car ils nous permettent d’améliorer l’expérience.

## <a name="group-search-and-sorting"></a>Recherche et tri de groupe

La recherche dans une liste de groupes a été améliorée. Lorsque vous entrez une chaîne à rechercher, la fonction de recherche exécute automatiquement la commande `startswith` et effectue une recherche substring dans la liste des noms de groupes. La recherche de sous-chaîne porte uniquement sur des mots entiers et n’inclut pas de caractères spéciaux. La recherche substring respecte la casse.

![Nouvelles recherches de sous-chaînes dans la page Tous les groupes](./media/groups-members-owners-search/groups-search-preview.png)

Par exemple, une recherche portant sur le mot « stratégie » retourne les chaînes « stratégie MDM – Ouest » et « groupe de stratégies ». Un groupe nommé « Nouvelle_stratégie » n’est pas retourné.

- Vous pouvez également effectuer la même recherche sur des listes d’appartenance à des groupes.
- Vous pouvez désormais trier la liste des groupes par nom à l’aide des flèches situées à droite de l’en-tête de la colonne des noms dans l’ordre croissant ou décroissant.

## <a name="group-member-search-and-filtering"></a>Recherche et filtrage de membres de groupe

### <a name="search-group-member-and-owner-lists"></a>Rechercher dans les listes de membres et de propriétaires de groupes

Vous pouvez désormais rechercher des membres d’un groupe spécifique par leur nom et effectuer la même recherche dans la liste des propriétaires du groupe. Dans la nouvelle expérience, si vous entrez une chaîne dans la zone de recherche, une commande startswith est effectuée automatiquement. Par exemple, la recherche de « Scott » renvoie Scott Wilkinson.

![nouvelles recherches de sous-chaînes sur les listes de membres et propriétaires de groupes](./media/groups-members-owners-search/members-list.png)

### <a name="filter-member-and-owners-list"></a>Filtrer une liste de membres et de propriétaires

En plus de la recherche, vous pouvez désormais filtrer les listes de membres et de propriétaires par type d’utilisateur. Il s’agit des informations figurant dans la colonne Type d’utilisateur de la liste. Par conséquent, vous pouvez filtrer la liste par membres et invités afin de déterminer si le groupe comprend des invités.

### <a name="view-and-manage-membership"></a>Afficher et gérer l’appartenance

En plus de l’affichage des membres directs d’un groupe spécifique, vous pouvez désormais afficher la liste de tous les membres du groupe dans la page Membres. Cette liste inclut tous les membres uniques d’un groupe, y compris les membres transitifs.

Vous pouvez également rechercher et filtrer individuellement dans les listes des membres directs et de tous les membres. Le filtrage de la liste tous les membres n’affecte pas les filtres appliqués à la liste des membres directs.

## <a name="improved-group-member-counts"></a>Comptages de membres de groupes amélioré

Nous avons amélioré la page **Vue d’ensemble** de groupe pour fournir des comptages de membres pour les groupes de toutes tailles. Vous pouvez voir le nombre de membres même pour des groupes comptant plus de 1 000 membres. Vous pouvez maintenant voir le nombre total de membres directs d’un groupe et le nombre total d’appartenances (tous les membres uniques du groupe, membres transitifs inclus) sur la page **Vue d’ensemble**.

![Plus grande précision des comptages d’appartenances aux groupes](./media/groups-members-owners-search/member-numbers.png)

## <a name="next-steps"></a>Étapes suivantes

Ces articles fournissent des informations supplémentaires sur l’utilisation des groupes dans Azure AD.

- [Afficher vos groupes et vos membres](../fundamentals/active-directory-groups-view-azure-portal.md)
- [Ajouter une appartenance à un groupe](../fundamentals/active-directory-groups-membership-azure-portal.md)
- [Gérer les règles dynamiques pour les utilisateurs dans un groupe](groups-create-rule.md)
- [Modifier vos paramètres de groupe](../fundamentals/active-directory-groups-settings-azure-portal.md)
- [Gérer l’accès aux ressources à l’aide des groupes](../fundamentals/active-directory-manage-groups.md)
- [Gérer l’accès aux applications SaaS en utilisant des groupes](groups-saasapps.md)
- [Gérer des groupes au moyen de commandes PowerShell](../enterprise-users/groups-settings-v2-cmdlets.md)
- [Ajouter un abonnement Azure à Azure Active Directory](../fundamentals/active-directory-how-subscriptions-associated-directory.md)

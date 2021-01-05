---
title: Déléguer la gouvernance des accès aux créateurs de catalogue dans la gestion des droits d’utilisation Azure AD - Azure Active Directory
description: Découvrez comment déléguer la gouvernance des accès des administrateurs informatiques aux créateurs de catalogue et chefs de projet pour qu’ils puissent gérer les accès eux-mêmes.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: mwahl
ms.collection: M365-identity-device-management
ms.openlocfilehash: e41f372cd0105896765d5a267456b6bda767dd4d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87798424"
---
# <a name="delegate-access-governance-to-catalog-creators-in-azure-ad-entitlement-management"></a>Déléguer la gouvernance des accès aux créateurs de catalogue dans la gestion des droits d’utilisation Azure AD

Un catalogue est un conteneur de ressources et de packages d’accès. Vous créez un catalogue lorsque vous souhaitez regrouper des ressources et packages d’accès liés. Par défaut, un administrateur général ou un administrateur d’utilisateurs peut [créer un catalogue](entitlement-management-catalog-create.md) et peut ajouter des utilisateurs supplémentaires en tant que propriétaires du catalogue.

Pour déléguer à des utilisateurs qui ne sont pas administrateurs, pour qu’ils puissent créer leurs propres catalogues, vous pouvez ajouter ces utilisateurs au rôle de créateur de catalogue défini par la gestion des droits d’utilisation Azure AD. Vous pouvez ajouter des utilisateurs individuels ou un groupe, dont les membres sont alors en mesure de créer des catalogues.  Après la création d’un catalogue, les utilisateurs peuvent ajouter par la suite des ressources dont ils sont propriétaires dans leur catalogue.

## <a name="as-an-it-administrator-delegate-to-a-catalog-creator"></a>En tant qu’administrateur informatique, déléguer à un créateur de catalogue

Suivez ces étapes pour affecter un utilisateur au rôle de créateur de catalogue.

**Rôle prérequis :** Administrateur général ou Administrateur d’utilisateurs

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, dans la section **Gestion des droits d’utilisation**, cliquez sur **Paramètres**.

1. Cliquez sur **Modifier**.

    ![Paramètres pour ajouter des créateurs de catalogue](./media/entitlement-management-delegate-catalog/settings-delegate.png)

1. Dans la section **Déléguer la gestion des droits d’utilisation**, cliquez sur **Ajouter des créateurs de catalogue** pour sélectionner les utilisateurs ou groupes auxquels vous voulez déléguer cette gestion des droits d’utilisation.

1. Cliquez sur **Sélectionner**.

1. Cliquez sur **Enregistrer**.

## <a name="allow-delegated-roles-to-access-the-azure-portal"></a>Autoriser les rôles délégués à accéder au Portail Azure

Pour permettre aux rôles délégués, tels que les créateurs de catalogue et les gestionnaires de packages d’accès, d’accéder au Portail Azure afin de gérer les packages d’accès, vous devez vérifier le paramètre du portail d’administration.

**Rôle prérequis :** Administrateur général ou Administrateur d’utilisateurs

1. Dans le Portail Azure, cliquez sur **Azure Active Directory**, puis sur **Utilisateurs**.

1. Dans le menu de gauche, cliquez sur **Paramètres utilisateur**.

1. Vérifiez que **Limiter l’accès au portail d’administration Azure AD** est défini sur **Non**.

    ![Paramètres utilisateur Azure AD - Portail d’administration](./media/entitlement-management-delegate-catalog/user-settings.png)

## <a name="next-steps"></a>Étapes suivantes

- [Créer et gérer un catalogue de ressources](entitlement-management-catalog-create.md)
- [Déléguer la gouvernance des accès aux gestionnaires de package d’accès](entitlement-management-delegate-managers.md)


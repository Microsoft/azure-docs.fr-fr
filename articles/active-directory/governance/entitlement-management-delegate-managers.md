---
title: Déléguer la gouvernance des accès aux gestionnaires de package dans le cadre de la gestion des droits d’utilisation Azure AD – Azure Active Directory
description: Découvrez comment déléguer la gouvernance des accès des administrateurs informatiques aux gestionnaires de package et chefs de projet pour qu’ils puissent gérer les accès eux-mêmes.
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
ms.openlocfilehash: e81389ec953829115062a31a019caf040850cbea
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87798633"
---
# <a name="delegate-access-governance-to-access-package-managers-in-azure-ad-entitlement-management"></a>Déléguer la gouvernance des accès aux gestionnaires de package dans le cadre de la gestion des droits d’utilisation Azure AD

Pour déléguer la création et la gestion des packages d’accès dans un catalogue, vous ajoutez des utilisateurs au rôle de gestionnaire de package d’accès. Les gestionnaires de package d’accès doivent connaître les besoins des utilisateurs quant à l’accès aux ressources d’un catalogue. Par exemple, si un catalogue est utilisé pour un projet, alors un coordinateur de projet peut être gestionnaire de package d’accès pour ce catalogue.  Les gestionnaires de package d’accès ne peuvent pas ajouter de ressources à un catalogue, mais ils peuvent gérer les packages d’accès et les stratégies contenus dans un catalogue.  Lors de la délégation à un gestionnaire de package d’accès, cette personne obtient alors les responsabilités suivantes :

- Déterminer quels rôles un utilisateur va avoir sur les ressources contenues dans un catalogue
- Déterminer qui va avoir besoin d’un accès
- Déterminer qui a besoin d’approuver les demandes d’accès
- Déterminer combien de temps va durer le projet

Cette vidéo fournit une vue d’ensemble de la délégation de la gouvernance des accès du propriétaire de catalogue au gestionnaire de package d’accès.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE3Lq08]

## <a name="as-a-catalog-owner-delegate-to-an-access-package-manager"></a>En tant que propriétaire de catalogue, déléguer à un gestionnaire de package d’accès

Suivez ces étapes pour affecter un utilisateur au rôle de gestionnaire de package d’accès :

**Rôle prérequis :** administrateur général, administrateur d’utilisateurs ou propriétaire de catalogue.

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, cliquez sur **Catalogues**, puis ouvrez le catalogue auquel vous souhaitez ajouter des administrateurs.

1. Dans le menu de gauche, cliquez sur **Rôles et administrateurs**.

    ![Rôles et administrateurs de catalogues](./media/entitlement-management-shared/catalog-roles-administrators.png)

1. Cliquez sur **Ajouter des gestionnaires de package d’accès** pour sélectionner les membres de ces rôles.

1. Cliquez sur **Sélectionner** pour ajouter ces membres.

## <a name="remove-an-access-package-manager"></a>Supprimer un gestionnaire de package d’accès

Suivez ces étapes pour retirer un utilisateur du rôle de gestionnaire de package d’accès :

**Rôle prérequis :** administrateur général, administrateur d’utilisateurs ou propriétaire de catalogue.

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, cliquez sur **Catalogues**, puis ouvrez le catalogue auquel vous souhaitez ajouter des administrateurs.

1. Dans le menu de gauche, cliquez sur **Rôles et administrateurs**.

1. Ajoutez une coche en regard d’un gestionnaire de package d’accès à supprimer.

1. Cliquez sur **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

- [Créer un package d’accès](entitlement-management-access-package-create.md)

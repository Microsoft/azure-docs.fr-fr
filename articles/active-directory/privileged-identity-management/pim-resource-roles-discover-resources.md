---
title: Découvrir les ressources Azure à gérer dans PIM - Azure AD | Microsoft Docs
description: Apprenez à découvrir les ressources Azure à gérer dans Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: pim
ms.date: 09/29/2020
ms.author: curtand
ms.collection: M365-identity-device-management
ms.openlocfilehash: be48e6e175beae751003895e60322a458cfbc8bd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91568078"
---
# <a name="discover-azure-resources-to-manage-in-privileged-identity-management"></a>Découvrir des ressources Azure à gérer dans Privileged Identity Management

Azure Active Directory (Azure AD) Privileged Identity Management (PIM) vous permet d'améliorer la protection de vos ressources Azure. Cela est utile pour :

- Les organisations qui utilisent déjà Privileged Identity Management pour protéger des rôles Azure AD.
- Les propriétaires d’abonnements et de groupes d’administration qui tentent de sécuriser des ressources de production.

Lors de la configuration initiale de Privileged Identity Management (PIM) pour des ressources Azure, vous devez découvrir et sélectionner les ressources à protéger avec Privileged Identity Management. Il n’existe aucune limite quant à la quantité de ressources que vous pouvez gérer avec Privileged Identity Management (PIM). Toutefois, nous vous recommandons de commencer avec vos ressources de production les plus critiques.

## <a name="discover-resources"></a>Découvrir des ressources

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Ouvrez **Azure AD Privileged Identity Management**.

1. Sélectionnez **Ressources Azure**.

    Si c’est la première fois que vous utilisez Privileged Identity Management (PIM) pour des ressources Azure, vous voyez s’afficher un volet **Découvrir les ressources**.

    ![Volet Découvrir les ressources sans ressources répertoriées, pour une première expérience](./media/pim-resource-roles-discover-resources/discover-resources-first-run.png)

    Si un autre administrateur au sein de votre organisation gère déjà des ressources Azure dans Privileged Identity Management (PIM), vous voyez s’afficher la liste des ressources actuellement gérées.

    ![Volet Découvrir les ressources répertoriant les ressources actuellement gérées](./media/pim-resource-roles-discover-resources/discover-resources.png)

1. Sélectionnez **Découvrir les ressources** pour lancer l’expérience de découverte.

    ![Le volet de découverte liste les ressources qui peuvent être gérées, comme les abonnements et les groupes d’administration](./media/pim-resource-roles-discover-resources/discovery-pane.png)

1. Dans le volet **Découverte**, utilisez les options **Filtre d’état de ressource** et **Sélectionner le type de ressource** pour filtrer les groupes d’administration ou abonnements pour lesquels vous disposez d’une autorisation d’écriture. Le plus simple est probablement de commencer avec **Tous**.

   Vous pouvez rechercher et sélectionner des ressources de groupe d’administration ou d’abonnement à gérer dans Privileged Identity Management. Quand vous gérez un groupe d’administration ou un abonnement dans Privileged Identity Management (PIM), vous pouvez également gérer ses ressources enfants.

   > [!Note]
   > Lorsque vous ajoutez une nouvelle ressource Azure enfant à un groupe d’administration géré par PIM, vous pouvez mettre la ressource enfant sous gestion en la recherchant dans PIM.

1. Sélectionnez les ressources non gérées que vous souhaitez gérer.

1. Sélectionnez **Gérer les ressources** pour commencer à gérer les ressources sélectionnées.

    > [!NOTE]
    > Une fois qu’un groupe d’administration ou abonnement est managé, il ne peut pas ne pas être managé. Cela empêche tout autre administrateur de ressources de supprimer des paramètres de Privileged Identity Management (PIM).

    ![Volet de découverte incluant une ressource sélectionnée et l’option Gérer les ressources mise en surbrillance](./media/pim-resource-roles-discover-resources/discovery-manage-resource.png)

1. Si vous voyez un message de confirmation de l’intégration de la ressource sélectionnée pour la gestion, sélectionnez **Oui**.

    ![Message confirmant l’intégration des ressources sélectionnées à des fins de gestion](./media/pim-resource-roles-discover-resources/discovery-manage-resource-message.png)

## <a name="next-steps"></a>Étapes suivantes

- [Configurer les paramètres des rôles de ressources Azure dans Privileged Identity Management](pim-resource-roles-configure-role-settings.md)
- [Attribuer des rôles de ressources Azure dans Privileged Identity Management](pim-resource-roles-assign-roles.md)

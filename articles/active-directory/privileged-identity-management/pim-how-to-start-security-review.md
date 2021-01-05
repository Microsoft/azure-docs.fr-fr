---
title: Créer une révision d’accès des rôles Azure AD dans PIM - Azure AD | Microsoft Docs
description: Découvrez comment créer une révision d’accès des rôles de ressources Azure dans Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: pim
ms.date: 10/22/2019
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9d9fdc44681c8773d7b6f724174a74e76e57939e
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92369758"
---
# <a name="create-an-access-review-of-azure-ad-roles-in-privileged-identity-management"></a>Créer une révision d’accès des rôles Azure AD dans Privileged Identity Management

Pour réduire les risques associés aux attributions de rôles obsolètes, vous devez vérifier les accès régulièrement. Vous pouvez utiliser Azure AD Privileged Identity Management (PIM) pour créer des révisions d’accès pour les rôles Azure AD privilégiés. Vous pouvez également configurer des révisions d’accès périodiques qui interviennent automatiquement.

Cet article explique comment créer une ou plusieurs révisions d’accès pour les rôles Azure AD privilégiés.

## <a name="prerequisites"></a>Prérequis

[Administrateur de rôle privilégié](../roles/permissions-reference.md#privileged-role-administrator)

## <a name="open-access-reviews"></a>Ouvrir les révisions d’accès

1. Connectez-vous au [portail Azure](https://portal.azure.com/) à l’aide d’un nom d’utilisateur qui est membre du rôle Administrateur de rôle privilégié.

1. Ouvrez **Azure AD Privileged Identity Management** .

1. Sélectionnez des **rôles Azure AD** .

1. Sous Gérer, sélectionnez **Révisions d’accès** , puis sélectionnez **Nouveau** .

    ![Rôles d’Azure AD : liste des révisions d’accès indiquant l’état de toutes les révisions](./media/pim-how-to-start-security-review/access-reviews.png)

[!INCLUDE [Privileged Identity Management access reviews](../../../includes/active-directory-privileged-identity-management-access-reviews.md)]

## <a name="start-the-access-review"></a>Démarrer la révision d’accès

Une fois que vous avez spécifié les paramètres pour une révision d’accès, sélectionnez **Démarrer** . La révision d’accès s’affiche dans votre liste, avec un indicateur de son état.

![Liste des révisions d’accès indiquant l’état des révisions démarrées](./media/pim-how-to-start-security-review/access-reviews-list.png)

Par défaut, Azure AD envoie un e-mail aux réviseurs peu de temps après le démarrage de la révision. Si vous ne souhaitez pas qu’Azure AD envoie cet e-mail, veillez à informer les réviseurs qu’une révision d’accès leur a été assignée. Vous pouvez leur montrer les instructions relatives à la [révision d’accès des rôles Azure AD](pim-how-to-perform-security-review.md).

## <a name="manage-the-access-review"></a>Gestion de la révision d’accès

Vous pouvez suivre la progression des révisions des réviseurs sur la page **Vue d’ensemble** de la révision d’accès. Aucun droit d’accès n’est modifié dans le répertoire avant que [la révision ne soit terminée](pim-how-to-complete-review.md).

![Page de vue d’ensemble des révisions d’accès montrant les détails de la révision](./media/pim-how-to-start-security-review/access-review-overview.png)

S’il s’agit d’une révision unique, une fois la période de révision d’accès terminée, ou si l’administrateur interrompt la révision d’accès, suivez les étapes de la [réalisation d’une révision d’accès des rôles Azure AD](pim-how-to-complete-review.md) pour voir et appliquer les résultats.  

Pour gérer une série de révisions d’accès, accédez à la révision d’accès et vous y voyez les occurrences à venir dans les révisions planifiées, pour lesquelles vous pouvez modifier la date de fin, ou ajouter/supprimer des réviseurs en conséquence.

En fonction de vos sélections dans **Paramètres de saisie semi-automatique** , l’application automatique est exécutée après la date de fin de la révision ou lorsque vous arrêtez manuellement la révision. La révision passe alors de l’état **Terminé** à divers états intermédiaires, comme **Application en cours** , pour arriver enfin à l’état **Appliqué** . Les utilisateurs dont l’accès est refusé doivent perdre leurs rôles au bout de quelques minutes.

## <a name="next-steps"></a>Étapes suivantes

- [Revoir l’accès aux rôles Azure AD](pim-how-to-perform-security-review.md)
- [Terminer une révision d’accès des rôles Azure AD](pim-how-to-complete-review.md)
- [Créer une révision d’accès des rôles de ressources Azure](pim-resource-roles-start-access-review.md)

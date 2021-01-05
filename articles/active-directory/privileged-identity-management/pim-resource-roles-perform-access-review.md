---
title: Réviser l’accès aux rôles de ressources Azure dans PIM - Azure AD | Microsoft Docs
description: Découvrez comment réviser les rôles de ressources Azure dans Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: pim
ms.date: 11/08/2019
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8091890d174ae68f012a6ec24685a0e705100f8b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84743675"
---
# <a name="review-access-to-azure-resource-roles-in-privileged-identity-management"></a>Réviser l’accès aux rôles de ressources Azure dans Privileged Identity Management

Les révisions d’accès dans Privileged Identity Management (PIM) peuvent aider à sécuriser l’accès aux rôles privilégiés dans Azure Active Directory (Azure AD). Cet article décrit les étapes à suivre pour réviser vos attributions de rôle privilégié dans le cadre d’une révision d’accès Azure AD.

Si vous êtes assigné à un rôle administratif, vous devrez peut-être demander une révision d’accès par votre administrateur pour confirmer que vous avez besoin d’un rôle. La demande de confirmation peut être fournie dans un e-mail contenant un lien ou vous pouvez confirmer via le [portail Azure](https://portal.azure.com).

Si vous êtes un administrateur de rôle privilégié intéressé par les révisions d’accès, consultez [Comment démarrer une révision de sécurité](pim-resource-roles-start-access-review.md)pour plus de détails.

## <a name="approve-or-deny-access"></a>Approuver ou refuser l'accès

Vous pouvez approuver ou refuser l’accès en fonction de si vous utilisez ou non ce rôle. Choisissez **Approuver** si vous souhaitez conserver le rôle, ou **Refuser** si vous n’avez plus besoin de l’accès. La modification du statut prendra effet une fois que le réviseur aura appliqué les résultats.

Procédez comme suit pour rechercher et terminer la révision de l’accès :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Sélectionnez **Azure Active Directory**, puis ouvrez **Privileged Identity Management**.
1. Sélectionnez **Réviser l’accès**.

   ![Capture d’écran de l’application Privileged Identity Management, avec sélection du panneau Réviser l’accès](media/pim-resource-roles-perform-access-review/rbac-access-review-complete.png)

1. Sélectionnez la révision à terminer.
1. Cliquez sur **Approuver** ou **Refuser**. Dans la zone **Indiquer un motif**, entrez une justification métier pour votre décision, si nécessaire.

   ![Capture d’écran de la page Détails de la révision](media/pim-resource-roles-perform-access-review/rbac-access-review-choice.png)

## <a name="next-steps"></a>Étapes suivantes

- [Effectuer une révision d’accès de mes rôles Azure AD dans Privileged Identity Management](pim-how-to-perform-security-review.md)

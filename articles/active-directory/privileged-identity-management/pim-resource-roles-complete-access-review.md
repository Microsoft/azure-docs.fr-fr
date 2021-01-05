---
title: Terminer une révision d’accès des rôles de ressources Azure dans PIM - Azure AD | Microsoft Docs
description: Découvrez comment terminer une révision d’accès des rôles de ressources Azure dans Azure AD Privileged Identity Management.
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
ms.date: 11/08/2019
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1601c9fc750df1f8e18bdaa072ede1b5ee164329
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84743811"
---
# <a name="complete-an-access-review-of-azure-resource-roles-in-privileged-identity-management"></a>Terminer une révision d’accès des rôles de ressources Azure dans Privileged Identity Management

Les administrateurs de rôle privilégié peuvent réviser l’accès privilégié après avoir [démarré une révision d’accès](pim-resource-roles-start-access-review.md). Privileged Identity Management (PIM) dans Azure Active Directory (Azure AD) envoie automatiquement un e-mail invitant les utilisateurs à réviser leur accès. Si un utilisateur ne reçoit pas d’e-mail, vous pouvez lui envoyer les instructions relatives à l’[exécution d’une révision d’accès](pim-resource-roles-perform-access-review.md).

À la fin de la période de révision d’accès, ou aussitôt que tous des utilisateurs ont terminé leur auto-examen, suivez les étapes décrites dans cet article pour gérer la révision et afficher les résultats.

## <a name="manage-access-reviews"></a>Gérer les révisions d’accès

1. Accédez au [portail Azure](https://portal.azure.com/). Dans le tableau de bord, sélectionnez le service **Ressources Azure**.

2. Sélectionnez votre ressource.

3. Sélectionnez la section **Révisions de l’accès** du tableau de bord.

    ![Ressources Azure : liste Révisions d’accès montrant le rôle, le propriétaire, la date de début, la date de fin et l’état](media/pim-resource-roles-complete-access-review/rbac-access-review-home-list.png)

4. Sélectionnez la révision d’accès que vous souhaitez gérer.

La page des détails de la révision d’accès comporte plusieurs options pour la gestion de cette révision. Les options disponibles sont les suivantes :

![Options de gestion d’une révision : Arrêter, Réinitialiser, Appliquer, Supprimer](media/pim-resource-roles-complete-access-review/rbac-access-review-menu.png)

### <a name="stop"></a>Arrêter

Toutes les révisions d’accès ont une date de fin. Sélectionnez **Arrêter** pour terminer l’opération plus tôt. Les utilisateurs qui n’ont pas terminé leur révision à ce stade ne pourront pas la terminer si vous l’arrêtez. Vous ne pouvez pas redémarrer une révision une fois qu’elle a été arrêtée.

### <a name="reset"></a>Réinitialiser

Vous pouvez réinitialiser une révision d’accès afin de supprimer toutes les décisions qui lui sont associées. Une fois la révision d’accès réinitialisée, tous les utilisateurs sont à nouveau marqués comme non révisés.

### <a name="apply"></a>Appliquer

Une fois une révision d’accès terminée, utilisez le bouton **Appliquer** pour implémenter le résultat de la révision. Si l’accès d’un utilisateur a été refusé lors de la révision, cette étape entraîne la suppression de son attribution de rôle.  

### <a name="delete"></a>DELETE

Si la révision ne vous intéresse plus, supprimez-la. Sélectionnez **Supprimer** pour supprimer la révision du service Privileged Identity Management.

## <a name="results"></a>Résultats

Dans la page **Résultats**, affichez et téléchargez la liste des résultats de votre révision.

![Page Résultats répertoriant les utilisateurs, les résultats, la raison, les réviseurs, les personnes ayant effectué l’application et les résultats de l’application](media/pim-resource-roles-complete-access-review/rbac-access-review-results.png)

## <a name="reviewers"></a>Réviseurs

Consultez et ajoutez des réviseurs à votre révision d’accès existant. Rappelez aux réviseurs d’effectuer leur révision.

![Page Réviseurs répertoriant les noms et les noms d’utilisateur principal](media/pim-resource-roles-complete-access-review/rbac-access-review-reviewers.png)

## <a name="next-steps"></a>Étapes suivantes

- [Commencer une révision d’accès des rôles de ressources Azure dans Privileged Identity Management](pim-resource-roles-start-access-review.md)
- [Effectuer une révision d’accès de mes rôles de ressources Azure dans Privileged Identity Management](pim-resource-roles-perform-access-review.md)

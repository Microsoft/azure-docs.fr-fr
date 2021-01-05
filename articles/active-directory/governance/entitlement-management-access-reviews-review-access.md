---
title: Révision de l’accès d’un package d’accès dans la gestion des droits d’utilisation Azure AD
description: Découvrez comment effectuer une révision d’accès pour les packages d’accès de gestion des droits d’utilisation dans les révisions d’accès Azure Active Directory (préversion).
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: d1115c08214db19227b4b8d7be671ce4da1cf2b1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87798595"
---
# <a name="review-access-of-an-access-package-in-azure-ad-entitlement-management"></a>Révision de l’accès d’un package d’accès dans la gestion des droits d’utilisation Azure AD

La gestion des droits d'utilisation d’Azure AD simplifie la gestion de l’accès aux groupes, aux applications et aux sites SharePoint pour les entreprises. Cet article explique comment effectuer des révisions d’accès pour d’autres utilisateurs affectés à un package d’accès en tant que réviseur désigné.

## <a name="prerequisites"></a>Conditions préalables requises

Pour examiner les attributions de packages d’accès actives des utilisateurs, vous devez remplir les conditions préalables requises pour effectuer une révision d’accès :
- Azure AD Premium P2
- Administrateur général
- Administrateur d’utilisateurs, propriétaire de catalogue ou gestionnaire de package d’accès désigné

Pour plus d’informations, consultez [Exigences des licences](entitlement-management-overview.md#license-requirements).


## <a name="open-the-access-review"></a>Ouvrir la révision d’accès

Utilisez la procédure suivante pour rechercher et ouvrir la révision d’accès :

1. Vous pourriez recevoir un e-mail de Microsoft vous invitant à réviser les accès. Retrouvez cet e-mail pour ouvrir la révision d’accès. Voici un exemple d’e-mail vous invitant à réviser l’accès :
    
    ![E-mail pour le réviseur de révision d’accès](./media/entitlement-management-access-reviews-review-access/review-access-reviewer-email.png)

1. Cliquez sur le lien **Réviser l’accès utilisateur** pour ouvrir la révision d’accès. 

1. Si vous n’avez pas reçu l’e-mail, vous trouverez les révisions d’accès en attente en accédant directement à https://myaccess.microsoft.com.  (Pour le gouvernement des États-Unis, utilisez `https://myaccess.microsoft.us` à la place.)

1. Cliquez sur **Révisions d’accès** dans la barre de navigation de gauche pour afficher la liste des révisions d’accès en attente qui vous sont affectées.
    
    ![Sélectionnez les révisions d’accès sur Mon Accès](./media/entitlement-management-access-reviews-review-access/review-access-myaccess-select-access-review.png)

1. Cliquez sur la révision que vous souhaitez commencer.
    
    ![Sélectionner la révision d’accès](./media/entitlement-management-access-reviews-review-access/review-access-select-access-review.png)

## <a name="perform-the-access-review"></a>Effectuer la révision d’accès

Une fois la révision d’accès ouverte, vous verrez les noms des utilisateurs que vous devez examiner. Il y a deux manières d’approuver ou de refuser l’accès  :
- Vous pouvez manuellement approuver ou refuser l’accès pour un ou plusieurs utilisateurs
- Vous pouvez accepter les recommandations du système

### <a name="manually-approve-or-deny-access-for-one-or-more-users"></a>Approuver ou refuser manuellement l’accès pour un ou plusieurs utilisateurs
1. Passez en revue la liste des utilisateurs et déterminez quels utilisateurs doivent continuer à avoir accès.

    ![Liste des utilisateurs à examiner](./media/entitlement-management-access-reviews-review-access/review-access-list-of-users.png)

1. Pour approuver ou refuser l’accès, sélectionnez la case d’option à gauche du nom de l’utilisateur.

1. Sélectionnez **Approuver** ou **Refuser** dans la barre située au-dessus des noms des utilisateurs.

    ![Sélectionnez l’utilisateur](./media/entitlement-management-access-reviews-review-access/review-access-select-users.png)

1. Si vous avez un doute, vous pouvez cliquer sur le bouton **Je ne sais pas**.

    Si vous effectuez cette sélection, l’utilisateur conserve l’accès et cette sélection est insérée dans les journaux d’audit. Le journal affiche tous les autres utilisateurs pour lesquels vous avez terminé la révision.

1. Vous pourriez être amené à justifier votre décision. Saisissez une raison, puis cliquez sur **Envoyer**.

    ![Approuver ou refuser l'accès](./media/entitlement-management-access-reviews-review-access/review-access-decision-approve.png)

1. Vous pouvez modifier votre décision à tout moment avant la fin de la révision. Pour ce faire, sélectionnez l’utilisateur dans la liste et modifiez la décision. Par exemple, vous pouvez approuver l’accès pour un utilisateur que vous avez précédemment refusé.

S’il existe plusieurs réviseurs, la dernière réponse envoyée est enregistrée. Prenons un exemple où un administrateur désigne deux réviseurs : Alice et Bob. Alice ouvre d’abord la révision et approuve l’accès. Avant la fin de la révision, Bob ouvre la révision et refuse l’accès. Dans ce cas, la dernière décision de refuser l’accès est enregistrée.

>[!NOTE]
>Si l’accès est refusé à un utilisateur, il n’est pas immédiatement supprimé du package d’accès. L’utilisateur est supprimé du package d’accès à la fin de la révision ou si un administrateur arrête la révision.

### <a name="approve-or-deny-access-using-the-system-generated-recommendations"></a>Approuver ou refuser l’accès à l’aide des recommandations générées par le système

Pour réviser plus rapidement l’accès de plusieurs utilisateurs, vous pouvez utiliser les recommandations générées par le système, en acceptant les recommandations d’un simple clic. Les recommandations sont générées en fonction de l’activité de connexion de l’utilisateur.

1.  Dans la barre en haut de la page, cliquez sur **Accepter les recommandations**.
    
    ![Sélectionnez Accepter les recommandations](./media/entitlement-management-access-reviews-review-access/review-access-use-recommendations.png)
    
    Vous obtenez un résumé des actions recommandées.

1.  Cliquez sur **Envoyer** pour accepter les recommandations.

## <a name="next-steps"></a>Étapes suivantes

- [Auto-évaluation des packages d’accès](entitlement-management-access-reviews-self-review.md)
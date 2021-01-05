---
title: Auto-révision d’un package d’accès dans la gestion des droits d’utilisation Azure AD
description: Découvrez comment réviser l’accès utilisateur pour les packages d’accès de gestion des droits d’utilisation dans les révisions d’accès Azure Active Directory (préversion).
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
ms.openlocfilehash: 31c44f2423cdc5c43638fe2515757bcb11a9814c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87798441"
---
# <a name="self-review-of-an-access-package-in-azure-ad-entitlement-management"></a>Auto-révision d’un package d’accès dans la gestion des droits d’utilisation Azure AD

La gestion des droits d'utilisation d’Azure AD simplifie la gestion de l’accès aux groupes, aux applications et aux sites SharePoint pour les entreprises. Cet article décrit comment un utilisateur effectue une auto-révision de ses packages d’accès attribués.

## <a name="open-the-access-review"></a>Ouvrir la révision d’accès

Pour effectuer une révision d’accès, vous devez d’abord ouvrir la révision d’accès. Utilisez la procédure suivante pour rechercher et ouvrir la révision d’accès :

1. Vous pourriez recevoir un e-mail de Microsoft vous invitant à réviser les accès. Retrouvez cet e-mail pour ouvrir la révision d’accès. Voici un exemple d’e-mail demandant une révision d’accès : 
    
    ![E-mail d’auto-révision de révision d’accès](./media/entitlement-management-access-reviews-review-access/self-review-reviewer-email.png)

1. Cliquez sur le lien **Revoir l'accès**.

1. Vous pouvez également accéder directement à https://myaccess.microsoft.com pour trouver vos révisions d’accès en attente si vous ne recevez pas d’e-mail.  (Pour le gouvernement des États-Unis, utilisez `https://myaccess.microsoft.us` à la place.)

1. Cliquez sur **Révisions d’accès** dans la barre de navigation de gauche pour afficher la liste des révisions d’accès en attente qui vous sont affectées.


1.  Cliquez sur la révision que vous souhaitez commencer.

## <a name="perform-the-access-review"></a>Effectuer la révision d’accès

Une fois que vous ouvrez la révision d’accès, vous pouvez voir votre accès. Utilisez la procédure suivante pour effectuer la révision d’accès :

1.  Déterminez si vous avez toujours besoin d’accéder au package d’accès. Par exemple, si le projet sur lequel vous travaillez n’est pas terminé, vous avez toujours besoin de l’accès pour continuer à travailler dessus.

1.  Cliquez sur **Oui** pour conserver votre accès ou sur **Non** pour supprimer votre accès.
    >[!NOTE]
    >Si vous avez indiqué que vous n’avez plus besoin d’un accès, vous n’êtes pas immédiatement supprimé du package d’accès. Vous êtes supprimé du package d’accès à la fin de la révision ou si un administrateur arrête la révision.

1.  Si vous avez cliqué sur **Oui**, vous devrez peut-être inclure une justification dans la zone de **Motif**.

1.  Cliquez sur **Envoyer**.

Vous pouvez revenir à la révision si vous changez d’avis et décidez de modifier votre réponse avant la fin de la révision.

## <a name="next-steps"></a>Étapes suivantes

- [Révision de l’accès aux packages d’accès](entitlement-management-access-reviews-review-access.md) 

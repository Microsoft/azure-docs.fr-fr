---
title: Examiner l’accès aux rôles Azure AD dans PIM - Azure AD | Microsoft Docs
description: Découvrez comment examiner l’accès des rôles Azure Active Directory dans Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: pim
ms.date: 04/24/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4dd0b63e08f44167b89b6e2d491722f3708b4036
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90055433"
---
# <a name="review-access-to-azure-ad-roles-in-privileged-identity-management"></a>Examiner l’accès aux rôles Azure AD dans Privileged Identity Management

Le service Privileged Identity Management (PIM) simplifie la gestion par les entreprises de l’accès privilégié aux ressources dans Azure Active Directory et d’autres services en ligne Microsoft, tels que Microsoft 365 ou Microsoft Intune. Suivez les étapes décrites dans cet article pour réussir l’auto-révision de vos rôles attribués.

Si vous êtes affecté à un rôle d’administrateur, l’administrateur de rôle privilégié de votre organisation peut vous demander de confirmer régulièrement que vous avez toujours besoin de ce rôle pour votre travail. Vous pouvez recevoir un e-mail contenant un lien, ou accéder directement au [portail Azure](https://portal.azure.com) et commencer.

Si vous êtes un administrateur de rôle privilégié ou administrateur global intéressé par les révisions d’accès, consultez [Comment démarrer une révision d’accès](pim-how-to-start-security-review.md)pour plus de détails.

## <a name="add-a-pim-dashboard-tile"></a>Ajouter une vignette de tableau de bord PIM

Si le service Privileged Identity Management n’est pas épinglé à votre tableau de bord dans le portail Azure, effectuez les étapes suivantes.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Sélectionnez votre nom d’utilisateur dans le coin supérieur droit du portail Azure, puis choisissez l’organisation Azure AD que vous allez utiliser.
3. Sélectionnez **Tous les services** et utilisez la zone de texte Filtre pour rechercher **Azure AD Privileged Identity Management**.
4. Cochez **Épingler au tableau de bord**, puis cliquez sur **Créer**. L’application Privileged Identity Management s’ouvre.

## <a name="approve-or-deny-access"></a>Approuver ou refuser l'accès

Lorsque vous acceptez ou refusez l’accès, vous indiquez simplement au réviseur si vous utilisez toujours ce rôle ou non. Choisissez **Approuver** si vous souhaitez conserver le rôle, ou **Refuser** si vous n’avez plus besoin de l’accès. Votre état ne change pas tout de suite car le réviseur doit d’abord appliquer les résultats.
Procédez comme suit pour rechercher et terminer la révision de l’accès :

1. Dans le service Privileged Identity Management, sélectionnez **Examiner l’accès privilégié**. Si vous avez des révisions d’accès en attente, ces révisions apparaissent dans la page **Révisions d’accès** Azure AD.
2. Sélectionnez la révision à terminer.
3. Sauf si vous avez créé la révision, vous serez l’unique utilisateur de cette révision. Cochez la case en regard de votre nom.
4. Choisissez **Approuver** ou **Refuser**. Vous devrez peut-être motiver votre choix dans la zone de texte **Indiquez une raison** .  
5. Fermez le panneau **Réviser les rôles Azure AD** .

## <a name="next-steps"></a>Étapes suivantes

- [Effectuer une révision d’accès des rôles de ressources Azure dans PIM](pim-resource-roles-perform-access-review.md)

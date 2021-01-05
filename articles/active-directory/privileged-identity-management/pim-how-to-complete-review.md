---
title: Effectuer une révision d’accès des rôles Azure AD dans PIM - Azure AD | Microsoft Docs
description: Découvrez comment effectuer une révision d’accès des rôles Azure AD dans Azure AD Privileged Identity Management (PIM), puis afficher les résultats.
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: pim
ms.date: 11/08/2019
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: fa7e8089fbbf2ee653100a05383fdbdc877ffda4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84742214"
---
# <a name="complete-an-access-review-of-azure-ad-roles-in-privileged-identity-management"></a>Effectuer une révision d’accès des rôles Azure AD dans Privileged Identity Management

Les administrateurs de rôle privilégié peuvent examiner l’accès privilégié dès le [démarrage d’une révision d’accès](pim-how-to-start-security-review.md).  Privileged Identity Management (PIM) envoie automatiquement un e-mail demandant aux utilisateurs dans votre organisation Azure Active Directory (Azure AD) d’examiner leur accès. Si un utilisateur n’a pas reçu d’e-mail, vous pouvez lui envoyer les instructions contenues dans [Guide pratique pour effectuer une révision d’accès](pim-how-to-perform-security-review.md).

À la fin de la période de révision d’accès, ou dès que tous les utilisateurs ont terminé leur auto-examen, suivez les étapes décrites dans cet article pour gérer la révision et afficher les résultats.

## <a name="manage-access-reviews"></a>Gérer les révisions d’accès

1. Accédez au [Portail Azure](https://portal.azure.com/) et sélectionnez le service **Azure AD Privileged Identity Management** sur votre tableau de bord.
1. Sélectionnez la section **Révisions de l’accès** du tableau de bord.
1. Sélectionnez la révision d’accès que vous souhaitez gérer.

Le panneau de détails de la révision d’accès comporte plusieurs options de gestion de cette révision.

![Boutons de révision d’accès Privileged Identity Management - capture d’écran](./media/pim-how-to-complete-review/review-buttons.png)

### <a name="remind"></a>Rappeler

Si une vérification de l’accès est configurée de sorte que les utilisateurs doivent eux-mêmes la réviser, le bouton **Rappeler** envoie une notification.

### <a name="stop"></a>Arrêter

Toutes les révisions d’accès ont une date de fin, mais vous pouvez utiliser le bouton **Arrêter** pour les terminer plus tôt. Si les utilisateurs n’ont pas été révisés à cette date, ils ne pourront plus le faire une fois la révision arrêtée. Vous ne pouvez pas redémarrer une révision arrêtée.

### <a name="apply"></a>Appliquer

Lorsqu’une vérification de l’accès est terminé, soit parce qu’elle a atteint la date de fin ou après un arrêt manuel, le bouton **Appliquer** implémente le résultat de la révision. Si l’accès d’un utilisateur a été refusé lors de la révision, cette étape supprimera l’affectation de son rôle.  

### <a name="export"></a>Exporter

Si vous souhaitez appliquer manuellement les résultats de la révision d’accès, vous pouvez exporter la révision. Le bouton **Exporter** fait démarrer le téléchargement d'un fichier CSV. Vous pouvez gérer les résultats dans Excel ou d'autres programmes qui s'ouvrent les fichiers CSV.

### <a name="delete"></a>DELETE

Si vous n'êtes plus intéressé par la révision, supprimez-la. Le bouton **Supprimer** supprime la révision du service Privileged Identity Management.

> [!IMPORTANT]
> Vous ne serez pas obligé de confirmer cette modification destructrice, donc vérifiez que vous souhaitez supprimer cette révision.

## <a name="next-steps"></a>Étapes suivantes

- [Commencer une révision d’accès des rôles Azure AD dans Privileged Identity Management](pim-how-to-start-security-review.md)
- [Effectuer une révision d’accès de mes rôles Azure AD dans Privileged Identity Management](pim-how-to-perform-security-review.md)

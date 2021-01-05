---
title: Approuver les demandes d’activation de membres et propriétaires de groupe dans Privileged Identity Management - Azure AD
description: Découvrez comment approuver ou refuser des demandes pour des groupes avec attribution de rôle dans Azure AD PIM (Privileged Identity Management).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: pim
ms.date: 07/27/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0a3b6dad33ce678478a201dc8b7b612f4b8507c0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91536985"
---
# <a name="approve-activation-requests-for-privileged-access-group-members-and-owners-preview"></a>Approuver des demandes d’activation de membres et propriétaires de groupe d’accès privilégié (préversion)

Avec PIM (Privileged Identity Management) dans Azure AD (Azure Active Directory), vous pouvez configurer les membres et propriétaires de groupe d’accès privilégié pour qu’ils nécessitent l’approbation des activations et désigner des utilisateurs ou groupes de votre organisation Azure AD comme approbateurs délégués. Nous vous recommandons de sélectionner plusieurs approbateurs pour chaque groupe afin de réduire la charge de travail pour l’administrateur de rôle privilégié. Les approbateurs délégués ont 24 heures pour approuver les demandes. Si une demande n’est pas approuvée sous 24 heures, l’utilisateur éligible doit soumettre une nouvelle demande. Le délai d'approbation de 24 heures n’est pas configurable.

Suivez les étapes décrites dans cet article pour approuver ou refuser des demandes de rôles de ressources Azure.

## <a name="view-pending-requests"></a>Afficher les demandes en attente

En tant qu’approbateur délégué, vous recevez une notification par e-mail quand une demande de rôle de ressource Azure est en attente d’approbation. Vous pouvez voir les demandes en attente dans Privileged Identity Management.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Ouvrez **Azure AD Privileged Identity Management**.

1. Sélectionnez **Approuver les demandes**.

    ![Approuver les demandes : page Ressources Azure montrant une demande de révision](./media/groups-approval-workflow/groups-select-request.png)

    Dans la section **Demandes d’activations de rôles**, vous voyez la liste des demandes en attente d’approbation.

## <a name="approve-requests"></a>Approuver les demandes

1. Recherchez et sélectionnez la demande que vous souhaitez approuver et sélectionnez **Approuver**.

    ![Capture d’écran montrant la page « Approuver les demandes » avec les boutons « Approuver » et « Confirmer » en surbrillance.](./media/groups-approval-workflow/groups-confirm-approval.png)

1. Dans la zone**Justification**, entrez la justification professionnelle.

1. Sélectionnez **Confirmer**. Votre approbation génère une notification Azure.

## <a name="deny-requests"></a>Refuser des demandes

1. Recherchez et sélectionnez la demande que vous souhaitez refuser et sélectionnez **Refuser**.

    ![Approuver les demandes : volet Approuver ou refuser avec des détails et la zone Justification](./media/groups-approval-workflow/groups-confirm-denial.png)

1. Dans la zone**Justification**, entrez la justification professionnelle.

1. Sélectionnez **Confirmer**. Votre refus génère une notification Azure.

## <a name="workflow-notifications"></a>Notifications de flux de travail

Voici quelques informations sur les notifications de flux de travail :

- Les approbateurs sont avertis par e-mail quand une demande d’attribution de groupe est en attente de leur examen. Les notifications par e-mail comportent un lien direct vers la demande, grâce auquel l’approbateur peut approuver ou refuser cette demande.
- Les demandes sont traitées par le premier approbateur qui les approuve ou les rejette.
- Lorsqu’un approbateur répond à la demande, tous les approbateurs sont informés de l’action.

>[!Note]
>Un administrateur qui estime qu’un utilisateur approuvé ne doit pas être actif peut supprimer l’attribution de groupe active dans Privileged Identity Management. Bien que les administrateurs de ressources ne soient informés des demandes en attente que s’ils sont approbateurs, ils peuvent voir et annuler les demandes en attente de tous les utilisateurs en les affichant dans Privileged Identity Management.

## <a name="next-steps"></a>Étapes suivantes

- [Étendre ou renouveler des attributions de groupe dans Privileged Identity Management](pim-resource-roles-renew-extend.md)
- [Notifications par e-mail dans Privileged Identity Management](pim-email-notifications.md)
- [Approuver ou rejeter des demandes d’attributions de groupe dans Privileged Identity Management](azure-ad-pim-approval-workflow.md)

---
title: Approuver des requêtes de rôles de ressources Azure dans PIM - Azure AD | Microsoft Docs
description: Découvrez comment approuver ou rejeter les demandes de rôles de ressources Azure dans Azure AD Privileged Identity Management (PIM).
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
ms.openlocfilehash: 096da0547806680f62b569f7c7809a7bbb04cb86
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84742077"
---
# <a name="approve-or-deny-requests-for-azure-resource-roles-in-privileged-identity-management"></a>Approuver ou rejeter des requêtes de rôles de ressources Azure AD dans Privileged Identity Management

Avec Privileged Identity Management (PIM) dans Azure Active Directory (Azure AD), vous pouvez configurer des rôles afin d’exiger une approbation pour les activations, et désigner des utilisateurs ou groupes de votre organisation Azure AD comme approbateurs délégués. Nous vous recommandons de sélectionner plusieurs approbateurs pour chaque rôle afin de réduire la charge de travail pour l’administrateur de rôle privilégié. Les approbateurs délégués ont 24 heures pour approuver les demandes. Si une demande n’est pas approuvée sous 24 heures, l’utilisateur éligible doit soumettre une nouvelle demande. Le délai d'approbation de 24 heures n’est pas configurable.

Suivez les étapes décrites dans cet article pour approuver ou refuser des demandes de rôles de ressources Azure.

## <a name="view-pending-requests"></a>Afficher les demandes en attente

En tant qu’approbateur délégué, vous recevez une notification par e-mail quand une demande de rôle de ressource Azure est en attente d’approbation. Ces demandes en attente sont affichées dans Privileged Identity Management.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Ouvrez **Azure AD Privileged Identity Management**.

1. Sélectionnez **Approuver les demandes**.

    ![Approuver les demandes : page Ressources Azure montrant une demande de révision](./media/pim-resource-roles-approval-workflow/resources-approve-requests.png)

    Dans la section **Demandes d’activations de rôles**, vous voyez la liste des demandes en attente d’approbation.

## <a name="approve-requests"></a>Approuver les demandes

1. Recherchez et sélectionnez la demande que vous souhaitez approuver. Une page Approuver ou rejeter s’affiche.

    ![Approuver les demandes : volet Approuver ou refuser avec des détails et la zone Justification](./media/pim-resource-roles-approval-workflow/resources-approve-pane.png)

1. Dans la zone**Justification**, entrez la justification professionnelle.

1. Sélectionnez **Approuver**. Vous allez recevoir une notification Azure de votre approbation.

    ![Notification d’approbation montrant que la demande a été approuvée](./media/pim-resource-roles-approval-workflow/resources-approve-notification.png)

## <a name="deny-requests"></a>Refuser des demandes

1. Recherchez et sélectionnez la demande que vous souhaitez rejeter. Une page Approuver ou rejeter s’affiche.

    ![Approuver les demandes : volet Approuver ou refuser avec des détails et la zone Justification](./media/pim-resource-roles-approval-workflow/resources-approve-pane.png)

1. Dans la zone**Justification**, entrez la justification professionnelle.

1. Sélectionner **Rejeter**. Une notification s’affiche avec votre refus.

## <a name="workflow-notifications"></a>Notifications de flux de travail

Voici quelques informations sur les notifications de flux de travail :

- Les approbateurs sont avertis par courrier électronique lorsqu’une demande concernant un rôle est en attente de leur examen. Les notifications par e-mail comportent un lien direct vers la demande, grâce auquel l’approbateur peut approuver ou refuser cette demande.
- Les demandes sont traitées par le premier approbateur qui les approuve ou les rejette.
- Lorsqu’un approbateur répond à la demande, tous les approbateurs sont informés de l’action.
- Les administrateurs de ressources sont avertis de l’activation d’un utilisateur approuvé dans leur rôle.

>[!Note]
>Un administrateur de ressources qui estime qu’un utilisateur approuvé ne doit pas être actif peut supprimer l’attribution de rôle active dans Privileged Identity Management. Bien que les administrateurs de ressources ne soient informés des demandes en attente que s’ils sont approbateurs, ils peuvent voir et annuler les demandes en attente de tous les utilisateurs en les affichant dans Privileged Identity Management.

## <a name="next-steps"></a>Étapes suivantes

- [Étendre ou renouveler des rôles de ressources Azure dans Privileged Identity Management](pim-resource-roles-renew-extend.md)
- [Notifications par e-mail dans Privileged Identity Management](pim-email-notifications.md)
- [Approuver ou rejeter des requêtes de rôles Azure AD dans Privileged Identity Management](azure-ad-pim-approval-workflow.md)

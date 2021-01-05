---
title: Approuver ou rejeter des demandes de rôles Azure AD dans PIM – Azure AD | Microsoft Docs
description: Découvrez comment approuver ou rejeter des demandes de rôles Azure AD dans Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: pim
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/07/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: d74a243ab10d0fffca960dc5149999560a7b54cd
ms.sourcegitcommit: f311f112c9ca711d88a096bed43040fcdad24433
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94980584"
---
# <a name="approve-or-deny-requests-for-azure-ad-roles-in-privileged-identity-management"></a>Approuver ou rejeter des demandes de rôles Azure AD dans Privileged Identity Management

Avec Azure Active Directory (Azure AD) Privileged Identity Management (PIM), vous pouvez configurer des rôles afin d’exiger une approbation pour les activations. Vous pouvez également choisir un ou plusieurs utilisateurs ou groupes en tant qu’approbateurs délégués. Les approbateurs délégués ont 24 heures pour approuver les demandes. Si une demande n’est pas approuvée sous 24 heures, l’utilisateur éligible doit soumettre une nouvelle demande. Le délai d'approbation de 24 heures n’est pas configurable.

## <a name="determine-your-version-of-pim"></a>Déterminer votre version de PIM

Depuis novembre 2019, la partie Rôles Azure AD de Privileged Identity Management est mise à jour vers une nouvelle version qui correspond à l’expérience des rôles Azure. Cela permet de créer des fonctionnalités supplémentaires et d’apporter des [modifications à l’API existante](azure-ad-roles-features.md#api-changes). Pendant que la nouvelle version est déployée, les procédures que vous suivez dans cet article dépendent de la version de Privileged Identity Management que vous possédez actuellement. Pour déterminer la version de Privileged Identity Management dont vous disposez, procédez de la manière décrite dans cette section. Une fois que vous connaissez votre version de Privileged Identity Management, vous pouvez sélectionner les procédures de cet article qui correspondent à cette version.

1. Connectez-vous au [portail Azure](https://portal.azure.com/) à l’aide d’un utilisateur avec le rôle [Administrateur de rôle privilégié](../roles/permissions-reference.md#privileged-role-administrator).
1. Ouvrez **Azure AD Privileged Identity Management**. Si une bannière figure en haut de la page de présentation, suivez les instructions sous l’onglet **Nouvelle version** de cet article. Sinon, suivez les instructions sous l’onglet **Version précédente**.

    [![Sélectionnez Azure AD > Privileged Identity Management.](media/pim-how-to-add-role-to-user/pim-new-version.png)](media/pim-how-to-add-role-to-user/pim-new-version.png#lightbox)

Suivez les étapes décrites dans cet article pour approuver ou refuser des demandes de rôles requête HTTP.

## <a name="new-version"></a>[Nouvelle version](#tab/new)

### <a name="view-pending-requests"></a>Afficher les demandes en attente

En tant qu’approbateur délégué, vous recevez une notification par e-mail quand une demande de rôle Azure AD est en attente d’approbation. Ces demandes en attente sont affichées dans Privileged Identity Management.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Ouvrez **Azure AD Privileged Identity Management**.

1. Sélectionnez **Approuver les demandes**.

    ![Approuver les demandes : page montrant une demande de révision des rôles Azure AD](./media/azure-ad-pim-approval-workflow/resources-approve-pane.png)

    Dans la section **Demandes d’activations de rôles**, vous voyez la liste des demandes en attente d’approbation.

### <a name="approve-requests"></a>Approuver les demandes

1. Recherchez et sélectionnez la demande que vous souhaitez approuver. Une page Approuver ou rejeter s’affiche.

    ![Capture d’écran montrant la page « Approuver les demandes – Rôles Azure AD ».](./media/azure-ad-pim-approval-workflow/resources-approve-pane.png)

1. Dans la zone **Justification**, entrez la justification professionnelle.

1. Sélectionnez **Approuver**. Vous allez recevoir une notification Azure de votre approbation.

    ![Notification d’approbation montrant que la demande a été approuvée](./media/pim-resource-roles-approval-workflow/resources-approve-pane.png))

### <a name="deny-requests"></a>Refuser des demandes

1. Recherchez et sélectionnez la demande que vous souhaitez rejeter. Une page Approuver ou rejeter s’affiche.

    ![Approuver les demandes : volet Approuver ou refuser avec des détails et la zone Justification](./media/pim-resource-roles-approval-workflow/resources-approve-pane.png)

1. Dans la zone **Justification**, entrez la justification professionnelle.

1. Sélectionner **Rejeter**. Une notification s’affiche avec votre refus.

### <a name="workflow-notifications"></a>Notifications de flux de travail

Voici quelques informations sur les notifications de flux de travail :

- Les approbateurs sont avertis par courrier électronique lorsqu’une demande concernant un rôle est en attente de leur examen. Les notifications par e-mail comportent un lien direct vers la demande, grâce auquel l’approbateur peut approuver ou refuser cette demande.
- Les demandes sont traitées par le premier approbateur qui les approuve ou les rejette.
- Lorsqu’un approbateur répond à la demande, tous les approbateurs sont informés de l’action.
- Les Administrateurs généraux et les Administrateurs de rôle privilégié sont avertis de l’activation d’un utilisateur approuvé dans leur rôle.

>[!NOTE]
>Un Administrateur général ou un Administrateur de rôle privilégié qui estime qu’un utilisateur approuvé ne doit pas être actif peut supprimer l’attribution du rôle actif dans Privileged Identity Management. Bien que les administrateurs ne soient informés des demandes en attente que s’ils sont approbateurs, ils peuvent voir et annuler toute demande en attente des utilisateurs en affichant les demandes en attente dans Privileged Identity Management.

## <a name="previous-version"></a>[Version précédente](#tab/previous)

### <a name="view-pending-requests"></a>Afficher les demandes en attente

En tant qu’approbateur délégué, vous recevez une notification par e-mail quand une demande de rôle Azure AD est en attente d’approbation. Ces demandes en attente sont affichées dans Privileged Identity Management.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Ouvrez **Azure AD Privileged Identity Management**.

1. Cliquez sur **Rôles Azure AD**.

1. Cliquez sur **Approuver les demandes**.

    ![Rôles Azure AD - Approuver les demandes](./media/azure-ad-pim-approval-workflow/approve-requests.png)

    La liste des demandes en attente de votre approbation s’affiche.

### <a name="approve-requests"></a>Approuver les demandes

1. Sélectionnez les demandes que vous souhaitez approuver, puis cliquez sur **Approuver** pour ouvrir le volet Approuver des demandes sélectionnées.

    ![Approuver la liste des demandes à l’aide de l’option Approuver mise en surbrillance](./media/azure-ad-pim-approval-workflow/pim-approve-requests-list.png)

1. Dans la zone **Approuver le motif**, tapez une raison.

    ![Volet Approuver des demandes sélectionnées avec option Approuver le motif](./media/azure-ad-pim-approval-workflow/pim-approve-selected-requests.png)

1. Cliquez sur **Approuver**.

    Le symbole d’état sera actualisé avec votre approbation.

    ![Volet Approuver des demandes sélectionnées après avoir cliqué sur Approuver](./media/azure-ad-pim-approval-workflow/pim-approve-status.png)

### <a name="deny-requests"></a>Refuser des demandes

1. Sélectionnez les demandes que vous souhaitez refuser, puis cliquez sur **Refuser** pour ouvrir le volet Refuser des demandes sélectionnées.

    ![Liste Approuver les demandes avec option Refuser mise en surbrillance](./media/azure-ad-pim-approval-workflow/pim-deny-requests-list.png)

1. Dans la zone **Refuser le motif**, tapez une raison.

    ![Volet Refuser des demandes sélectionnées avec motif de refus](./media/azure-ad-pim-approval-workflow/pim-deny-selected-requests.png)

1. Sélectionner **Rejeter**.

    Le symbole d’état sera actualisé avec votre refus.

---

## <a name="next-steps"></a>Étapes suivantes

- [Notifications par e-mail dans Privileged Identity Management](pim-email-notifications.md)
- [Approuver ou rejeter des demandes de rôles de ressources Azure dans Privileged Identity Management](pim-resource-roles-approval-workflow.md)

---
title: Configurer les paramètres des rôles de ressources Azure dans PIM - Azure AD | Microsoft Docs
description: Découvrez comment configurer les paramètres des rôles de ressources Azure dans Azure AD Privileged Identity Management (PIM).
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
ms.date: 01/01/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4ded81389fae5954c0e488f634945b4f1c1fe4c8
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96009039"
---
# <a name="configure-azure-resource-role-settings-in-privileged-identity-management"></a>Configurer les paramètres des rôles de ressource Azure dans Privileged Identity Management

Lorsque vous configurez les paramètres des rôles de ressources Azure, vous définissez les paramètres par défaut appliqués aux attributions de rôles de ressources Azure dans Azure Active Directory (Azure AD) Privileged Identity Management (PIM). Utilisez les procédures suivantes pour configurer le flux de travail d’approbation et spécifier qui peut approuver ou refuser les demandes.

## <a name="open-role-settings"></a>Ouvrir les paramètres de rôle

Suivez ces étapes pour ouvrir les paramètres pour un rôle de ressource Azure.

1. Connectez-vous au [portail Azure](https://portal.azure.com/) à l’aide d’un utilisateur avec le rôle [Administrateur de rôle privilégié](../roles/permissions-reference.md#privileged-role-administrator).

1. Ouvrez **Azure AD Privileged Identity Management**.

1. Sélectionnez **Ressources Azure**.

1. Sélectionnez la ressource que vous souhaitez gérer, telle qu’un abonnement ou un groupe d’administration.

    ![Page de ressources Azure qui répertorie les ressources pouvant être gérées](./media/pim-resource-roles-configure-role-settings/resources-list.png)

1. Sélectionnez **Paramètres du rôle**.

    ![Page Paramètres de rôle répertoriant les rôles de ressources Azure](./media/pim-resource-roles-configure-role-settings/resources-role-settings.png)

1. Sélectionnez le rôle dont vous souhaitez configurer les paramètres.

    ![Page Détails des paramètres de rôle répertoriant plusieurs paramètres d’affectation et d’activation](./media/pim-resource-roles-configure-role-settings/resources-role-setting-details.png)

1. Sélectionnez **Modifier** pour ouvrir le volet **Paramètres de rôle**. Le premier onglet vous permet de mettre à jour la configuration de l’activation de rôles dans Privileged Identity Management.

    ![Page Modifier les paramètres de rôle avec l’onglet Activation ouvert](./media/pim-resource-roles-configure-role-settings/role-settings-activation-tab.png)

1. Sélectionnez l’onglet **Affectation** ou le bouton **Suivant : Affectation** au bas de la page pour ouvrir l’onglet du paramètre d’affectation. Ces paramètres gèrent les affectations de rôles effectuées dans l’interface Privileged Identity Management.

    ![Onglet Attribution de rôle dans la page des paramètres de rôle](./media/pim-resource-roles-configure-role-settings/role-settings-assignment-tab.png)

1. Utilisez l’onglet **Notification** ou le bouton **Suivant : Activation** au bas de la page pour accéder à l’onglet du paramètre de notification pour ce rôle. Ces paramètres gèrent tous les e-mails de notification relatifs à ce rôle.

    ![Onglet Notifications de rôle dans la page des paramètres de rôle](./media/pim-resource-roles-configure-role-settings/role-settings-notification-tab.png)

    Sous l’onglet **Notifications** dans la page des paramètres de rôle, l’option Privileged Identity Management permet de contrôler précisément qui reçoit telle notification.

    - **Désactivation d’un e-mail**<br>Vous pouvez désactiver certains e-mails en désactivant la case à cocher du destinataire par défaut et en supprimant les éventuels destinataires supplémentaires.  

    - **Limiter les e-mails à des adresses e-mail spécifiées**<br>Vous pouvez désactiver les e-mails envoyés aux destinataires par défaut en désactivant la case à cocher du destinataire par défaut. Vous pouvez ensuite ajouter d’autres adresses e-mail comme destinataires supplémentaires. Si vous souhaitez ajouter plusieurs adresses e-mail, séparez-les par un point-virgule (;).

    - **Envoyer des e-mails à la fois aux destinataires par défaut et à des destinataires supplémentaires**<br>Vous pouvez envoyer des e-mails à la fois à des destinataires par défaut et à des destinataires supplémentaires en activant la case à cocher du destinataire par défaut et en ajoutant les adresses e-mail des destinataires supplémentaires.

    - **E-mails critiques uniquement**<br>Pour chaque type d’e-mail, vous pouvez activer la case à cocher pour recevoir uniquement les e-mails critiques. Cela signifie que Privileged Identity Management continue d’envoyer des e-mails aux destinataires configurés uniquement lorsqu’une action immédiate est requise. Par exemple, les e-mails qui demandent à l’utilisateur d’étendre son attribution de rôle ne seront pas déclenchés, tandis que ceux qui demandent à un administrateur d’approuver une demande d’extension seront déclenchés.

1. Sélectionnez le bouton **Mettre à jour** quand vous voulez pour mettre à jour les paramètres de rôle.

## <a name="assignment-duration"></a>Durée de l’attribution

Vous pouvez choisir entre deux options de durée d’attribution pour chaque type d’attribution (éligible et actif) lorsque vous configurez les paramètres d’un rôle. Ces options deviennent la durée maximale par défaut lorsqu’un utilisateur est attribué au rôle dans Privileged Identity Management.

Vous pouvez choisir l’une de ces options de durée d’attribution **éligible** :

| | |
| --- | --- |
| **Autoriser une attribution éligible permanente** | Les administrateurs de ressources peuvent accorder une attribution éligible permanente. |
| **Faire expirer les attributions éligibles après** | Les administrateurs de ressources peuvent exiger que toutes les attributions éligibles aient une date de début et une date de fin spécifiées. |

Vous pouvez choisir l’une de ces options de durée d’attribution **active** :

| | |
| --- | --- |
| **Autoriser une attribution active permanente** | Les administrateurs de ressources peuvent accorder une attribution active permanente. |
| **Faire expirer les attributions actives après** | Les administrateurs de ressources peuvent exiger que toutes les attributions actives aient une date de début et une date de fin spécifiées. |

> [!NOTE]
> Toutes les attributions qui ont une date de fin spécifiée peuvent être renouvelées par les administrateurs de ressources. De plus, les utilisateurs peuvent lancer des demandes en libre-service afin d’[étendre ou renouveler des attributions de rôles](pim-resource-roles-renew-extend.md).

## <a name="require-multi-factor-authentication"></a>Exiger une authentification multifacteur

Privileged Identity Management permet également l'implémentation facultative d'Azure AD Multi-Factor Authentication dans deux scénarios distincts.

### <a name="require-multi-factor-authentication-on-active-assignment"></a>Demander l'authentification multifacteur lors de l'attribution active

Dans certains cas, vous pouvez attribuer un utilisateur ou un groupe à un rôle pour une courte durée (une journée, par exemple). Les utilisateurs attribués n’ont alors pas besoin de demander l’activation. Dans ce scénario, Privileged Identity Management ne peut pas appliquer l’authentification multifacteur lorsque l’utilisateur utilise son attribution de rôle, car il est déjà actif dans le rôle depuis l’attribution.

Pour garantir que l’administrateur de ressources qui réalise l’attribution est bien celui qu’il prétend être, vous pouvez appliquer l’authentification multifacteur lors de l’attribution active en cochant la case **Demander l’authentification multifacteur lors de l’attribution active**.

### <a name="require-multi-factor-authentication-on-activation"></a>Exiger Multi-Factor Authentication lors de l’activation

Vous pouvez exiger des utilisateurs éligibles à un rôle qu'ils s'authentifient à l'aide d'Azure AD Multi-Factor Authentication pour effectuer l'activation. L’authentification multifacteur garantit, avec une certitude raisonnable, que l’utilisateur est bien celui qu’il prétend être. L’application de cette option permet de protéger les ressources critiques au cas où le compte d’utilisateur pourrait être compromis.

Pour exiger l’authentification multifacteur avant l’activation, cochez la case **Exiger l’authentification multifacteur lors de l’activation**.

Pour plus d’informations, consultez [Authentification multifacteur et Privileged Identity Management](pim-how-to-require-mfa.md).

## <a name="activation-maximum-duration"></a>Durée maximum d’activation

Utilisez le curseur **Durée maximum d’activation** pour définir la durée maximale, en heures, pendant laquelle un rôle reste actif avant d’expirer. Cette valeur peut être comprise entre 1 et 24 heures.

## <a name="require-justification"></a>Demander une justification

Vous pouvez exiger que les utilisateurs saisissent une justification métier lorsqu’ils s’activent. Pour demander une justification, cochez la case **Demander une justification lors de l'affectation active** ou la case **Demander une justification lors de l’activation**.

## <a name="require-approval-to-activate"></a>Demander une approbation pour activation

Si vous souhaitez exiger l’approbation pour activer un rôle, suivez ces étapes.

1. Cochez la case **Exiger une approbation pour activer**.

1. Cliquez sur **Sélectionner des approbateurs** pour ouvrir la page **Sélectionner un membre ou un groupe**.

    ![Sélectionner un volet d’utilisateur ou de groupe pour sélectionner les approbateurs](./media/pim-resource-roles-configure-role-settings/resources-role-settings-select-approvers.png)

1. Sélectionnez au moins un utilisateur ou un groupe, puis cliquez sur **Sélectionner**. Vous pouvez ajouter n’importe quelle combinaison d’utilisateurs et de groupes. Vous devez sélectionner au moins un approbateur. Il n’existe aucun approbateur par défaut.

    Vos sélections apparaissent dans la liste des approbateurs sélectionnés.

1. Une fois que vous avez spécifié tous vos paramètres de rôle, sélectionnez **Mettre à jour** pour enregistrer vos modifications.

## <a name="next-steps"></a>Étapes suivantes

- [Attribuer des rôles de ressources Azure dans Privileged Identity Management](pim-resource-roles-assign-roles.md)
- [Configurer les alertes de sécurité pour les rôles de ressources Azure dans Privileged Identity Management](pim-resource-roles-configure-alerts.md)

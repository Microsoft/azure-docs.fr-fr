---
title: Configurer les paramètres des groupes d’accès privilégié dans PIM – Azure Active Directory | Microsoft Docs
description: Découvrez comment configurer les paramètres des groupes assignables à un rôle dans Azure AD Privileged Identity Management (PIM).
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
ms.date: 07/27/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: abb7f93437cd45914d3824e9f557241ba0d71162
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94835388"
---
# <a name="configure-privileged-access-group-settings-preview-in-privileged-identity-management"></a>Configurer les paramètres de groupe d’accès privilégié (préversion) dans Privileged Identity Management

Les paramètres de rôle sont les paramètres par défaut appliqués aux attributions d’accès privilégié au propriétaire aux membres du groupe dans Privileged Identity Management (PIM). Procédez comme suit pour configurer le workflow d’approbation afin de spécifier qui peut approuver ou refuser des demandes d’élévation de privilèges.

## <a name="open-role-settings"></a>Ouvrir les paramètres de rôle

Procédez comme suit pour ouvrir les paramètres d’un rôle de groupe d’accès privilégié Azure.

1. Connectez-vous au [portail Azure](https://portal.azure.com/) à l’aide d’un utilisateur avec le rôle [Administrateur de rôle privilégié](../roles/permissions-reference.md#privileged-role-administrator).

1. Ouvrez **Azure AD Privileged Identity Management**.

1. Sélectionnez **Accès privilégié (préversion)** .

1. Sélectionnez le groupe à gérer.

    ![Groupes d’accès privilégié filtrés par un nom de groupe](./media/groups-role-settings/group-select.png)

1. Sélectionnez **Paramètres**.

    ![Page Paramètres répertoriant les paramètres de groupe pour le groupe sélectionné](./media/groups-role-settings/group-settings-select-role.png)

1. Sélectionnez le rôle Propriétaire ou Membre dont vous souhaitez consulter ou modifier les paramètres. Vous pouvez afficher les paramètres actuels du rôle dans la page **Détails des paramètres de rôle**.

    ![Page Détails des paramètres de rôle répertoriant plusieurs paramètres d’affectation et d’activation](./media/groups-role-settings/group-role-setting-details.png)

1. Sélectionnez **Modifier** pour ouvrir la page **Modifier les paramètres de rôle**. L’onglet **Activation** vous permet de modifier les paramètres d’activation de rôle, notamment d’autoriser ou non les attributions actives et éligibles permanentes.

    ![Page Modifier les paramètres de rôle avec l’onglet Activation ouvert](./media/groups-role-settings/role-settings-activation-tab.png)

1. Sélectionnez l’onglet **Attribution** pour ouvrir l’onglet Paramètres d’attribution. Ces paramètres contrôlent les paramètres d’attribution Privileged Identity Management de ce rôle.

    ![Onglet Attribution de rôle dans la page des paramètres de rôle](./media/groups-role-settings/role-settings-assignment-tab.png)

1. Utilisez l’onglet **Notification** ou le bouton **Suivant : Activation** au bas de la page pour accéder à l’onglet du paramètre de notification pour ce rôle. Ces paramètres gèrent tous les e-mails de notification relatifs à ce rôle.

    ![Onglet Notifications de rôle dans la page des paramètres de rôle](./media/groups-role-settings/role-settings-notification-tab.png)

1. Sélectionnez le bouton **Mettre à jour** quand vous voulez pour mettre à jour les paramètres de rôle.

Sous l’onglet **Notifications** dans la page des paramètres de rôle, l’option Privileged Identity Management permet de contrôler précisément qui reçoit telle notification.

- **Désactivation d’un e-mail**<br>Vous pouvez désactiver certains e-mails en désactivant la case à cocher du destinataire par défaut et en supprimant les éventuels destinataires supplémentaires.  
- **Limiter les e-mails à des adresses e-mail spécifiées**<br>Vous pouvez désactiver les e-mails envoyés aux destinataires par défaut en désactivant la case à cocher du destinataire par défaut. Vous pouvez ensuite ajouter d’autres adresses e-mail comme destinataires supplémentaires. Si vous souhaitez ajouter plusieurs adresses e-mail, séparez-les par un point-virgule (;).
- **Envoyer des e-mails à la fois aux destinataires par défaut et à des destinataires supplémentaires**<br>Vous pouvez envoyer des e-mails à la fois à des destinataires par défaut et à des destinataires supplémentaires en activant la case à cocher du destinataire par défaut et en ajoutant les adresses e-mail des destinataires supplémentaires.
- **E-mails critiques uniquement**<br>Pour chaque type d’e-mail, vous pouvez activer la case à cocher pour recevoir uniquement les e-mails critiques. Cela signifie que Privileged Identity Management continue d’envoyer des e-mails aux destinataires configurés uniquement lorsqu’une action immédiate est requise. Par exemple, les e-mails qui demandent à l’utilisateur d’étendre son attribution de rôle ne seront pas déclenchés, tandis que ceux qui demandent à un administrateur d’approuver une demande d’extension seront déclenchés.

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

    ![Sélectionner un volet d’utilisateur ou de groupe pour sélectionner les approbateurs](./media/groups-role-settings/group-settings-select-approvers.png)

1. Sélectionnez au moins un utilisateur ou un groupe, puis cliquez sur **Sélectionner**. Vous pouvez ajouter n’importe quelle combinaison d’utilisateurs et de groupes. Vous devez sélectionner au moins un approbateur. Il n’existe aucun approbateur par défaut.

    Vos sélections apparaissent dans la liste des approbateurs sélectionnés.

1. Une fois que vous avez spécifié tous vos paramètres de rôle, sélectionnez **Mettre à jour** pour enregistrer vos modifications.

## <a name="next-steps"></a>Étapes suivantes

- [Attribuer l’appartenance à un groupe d’accès privilégié ou la propriété de celui-ci dans PIM](groups-assign-member-owner.md)

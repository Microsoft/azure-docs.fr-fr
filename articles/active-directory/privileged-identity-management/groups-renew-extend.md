---
title: Renouveler un propriétaire de groupe expiré d’affectations de membres dans Privileged Identity Management – Azure AD | Microsoft Docs
description: Découvrez comment étendre ou renouveler des affectations de groupes affectables à un rôle dans Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: pim
ms.date: 06/30/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0cfbb0be22dee4550050d6af10314f3a3bb1f583
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87505617"
---
# <a name="extend-or-renew-privileged-access-group-assignments-preview-in-privileged-identity-management"></a>Étendre ou renouveler des affectations de groupe d’accès privilégié (préversion) dans Privileged Identity Management

Azure Active Directory (Azure AD) Privileged Identity Management (PIM) fournit des contrôles permettant de gérer le cycle de vie d’accès et d’affectation pour des groupes d’accès privilégié. Les administrateurs peuvent attribuer des rôles à l’aide de propriétés de date et d’heure de début et de fin. À l’approche de la fin de l’attribution, Privileged Identity Management envoie des notifications par e-mail aux utilisateurs ou aux groupes concernés ainsi qu’aux administrateurs de la ressource pour garantir le maintien de l’accès approprié. Les attributions peuvent être renouvelées et rester visibles à l’état expiré pendant 30 jours, même si l’accès n’est pas étendu.

## <a name="who-can-extend-and-renew"></a>Qui peut étendre et renouveler ?

Seuls des administrateurs de la ressource peuvent étendre ou renouveler des affectations de groupe d’accès privilégié. L’utilisateur ou le groupe affecté peut demander l’extension d’affectations arrivant à expiration et le renouvellement d’affectations ayant déjà expiré.

## <a name="when-notifications-are-sent"></a>Critères d’envoi des notifications

Privileged Identity Management envoie des notifications par e-mail aux administrateurs et aux utilisateurs affectés d’affectations de groupe d’accès privilégié qui expirent :

- dans les 14 jours précédant l’expiration ;
- un jour avant l’expiration ;
- quand une affectation expire.

Les administrateurs reçoivent des notifications quand un utilisateur ou groupe demande une extension ou un renouvellement d’une affectation venant à expiration ou expirée. Quand un administrateur résout la demande, tous les administrateurs et l’utilisateur demandeur sont avertis de l’approbation ou du refus.

## <a name="extend-group-assignments"></a>Ajouter des affectations de groupes

Les étapes suivantes décrivent la procédure de demande, de résolution et d’administration d’une extension ou d’un renouvellement d’affectation de groupe.

### <a name="self-extend-expiring-assignments"></a>Étendre automatiquement les attributions arrivant à expiration

Les utilisateurs affectés à un groupe d’accès privilégié peuvent étendre des affectations de groupe arrivant à expiration directement à partir de l’onglet **Éligible** ou **Active** de la page **Affectations** du groupe. Des utilisateurs ou des groupes peuvent demander l’extension d’affectations éligibles et actives qui expirent au cours des 14 prochains jours.

![Page Mes rôles répertoriant des affectations éligibles avec une colonne Action](media/groups-renew-extend/self-extend-group-assignment.png)

Quand la date et heure de fin de l’affectation sont à maximum 14 jours, la commande **Étendre** est disponible. Pour demander une extension d’affectation de groupe, sélectionnez **Étendre** afin d’ouvrir le formulaire de demande.

![Volet Étendre l’affectation de groupe avec une zone Raison et des détails](media/groups-renew-extend/extend-request-details-group-assignment.png)

>[!NOTE]
>Nous vous recommandons de préciser en détail la raison pour laquelle l’extension est nécessaire, ainsi que la durée de l’extension demandée (si vous la connaissez).

Quelques instants après, les administrateurs reçoivent une notification par e-mail leur demandant d’examiner la demande d’extension. Si une demande d’extension a déjà été envoyée, une notification Azure apparaît dans le portail.

Pour afficher l’état de votre demande ou annuler celle-ci, ouvrez la page **Demandes en attente** pour l’affectation de groupe.

![Affectations de groupe d’accès privilégié – Page demandes en attente présentant le lien pour annuler](media/groups-renew-extend/group-assignment-extend-cancel-request.png)

### <a name="admin-approved-extension"></a>Extensions approuvées par l’administrateur

Quand un utilisateur ou un groupe envoie une demande d’extension d’une affectation de groupe, les administrateurs reçoivent une notification par e-mail contenant les détails de l’affectation d’origine et le motif de la demande. La notification inclut un lien direct vers la requête pour que l’administrateur puisse l’approuver ou la refuser.

Pour approuver ou rejeter les demandes, les administrateurs peuvent suivre le lien contenu dans l’e-mail ou accéder au portail d’administration de Privileged Identity Management et sélectionner **Approuver les demandes** dans le menu de gauche.

![Affectations de groupe d’accès privilégié – Page Approuver les demandes répertoriant les demandes et des liens pour les approuver ou refuser](media/groups-renew-extend/group-assignment-extend-admin-approve.png)

Lorsqu’un administrateur sélectionne **Approuver** ou **Rejeter**, les détails de la demande s’affichent, ainsi qu’un champ permettant de donner une justification professionnelle pour les journaux d’audit.

![Demande d’approbation d’affectation de groupe avec la raison du demandeur, le type d’affectation, l’heure de début, l’heure de fin et le motif](media/groups-renew-extend/group-assignment-extend-admin-approve-reason.png)

Quand ils approuvent une demande d’extension d’affectation de groupe, les administrateurs de ressources peuvent choisir de nouvelles dates de début et de fin et un nouveau type d’affectation. Une modification du type d’attribution peut être nécessaire s’ils souhaitent accorder un accès limité afin d’effectuer une tâche spécifique (un jour, par exemple). Dans cet exemple, l’administrateur peut modifier l’attribution de **Éligible** à **Actif**, ce qui signifie qu’il peut donner l’accès au demandeur sans l’obliger à s’activer.

### <a name="admin-initiated-extension"></a>Extension initiée par l’administrateur

Si un utilisateur affecté à un groupe ne demande pas d’extension pour une affectation de groupe, un administrateur peut étendre celle-ci au nom de l’utilisateur. Des extensions administratives d’affectation de groupe ne nécessitent pas d’approbation, mais des notifications sont envoyées à tous les autres administrateurs une fois l’affectation étendue.

Pour étendre une affectation de groupe, accédez à la vue de l’affectation dans Privileged Identity Management. Trouvez l’attribution qui a besoin d’une extension. Ensuite, sélectionnez **Étendre** dans la colonne d’action.

![Page Affectations répertoriant les affectations de groupes éligibles avec des liens pour les étendre](media/groups-renew-extend/group-assignment-extend-admin-approve.png)

## <a name="renew-group-assignments"></a>Renouveler des affectations de groupes

Bien que semblable d’un point de vue conceptuel au processus de demande d’extension, le processus de renouvellement d’une affectation de groupe arrivée à expiration est différent. Les étapes suivantes permettent aux administrateurs de renouveler l’accès à des affectations expirées si nécessaire.

### <a name="self-renew"></a>Renouvellement autonome

Les utilisateurs qui n’ont plus accès aux ressources peuvent accéder à 30 jours d’historique d’attributions arrivées à expiration. Pour cela, ils accèdent à **Mes rôles** dans le volet à gauche, puis sélectionnent l’onglet **Affectations expirées**.

![Page Mes rôles – Onglet Affectations expirées](media/groups-renew-extend/groups-renew-from-my-roles.png)

La liste d’affectations affiche par défaut les **Affectations éligibles**. Utilisez le menu déroulant pour basculer entre les affectations attribués Éligibles et Actives.

Pour demander le renouvellement d’une affectation de groupe figurant dans la liste, sélectionnez l’action **Renouveler**. Ensuite, précisez le motif de la demande. Il est utile d’indiquer une durée en plus d’un contexte supplémentaire ou d’une justification professionnelle pour aider l’administrateur de la ressource à l’approuver ou la rejeter.

![Volet Renouveler une affectation de groupe montrant la zone Raison](media/groups-renew-extend/groups-renew-request-form.png)

Une fois que la demande envoyée, les administrateurs de ressources sont informés de l’existence d’une demande de renouvellement d’affectation de groupe en attente.

### <a name="admin-approves"></a>Approbations d’administrateur

Les administrateurs de ressources peuvent accéder à la demande de renouvellement à partir du lien contenu dans la notification par e-mail ou en accédant à Privileged Identity Management sur le Portail Azure et en sélectionnant **Approuver les demandes** dans le menu de gauche.

Lorsqu’un administrateur sélectionne **Approuver** ou **Rejeter**, les détails de la demande s’affichent, ainsi qu’un champ permettant de donner une justification professionnelle pour les journaux d’audit.

Quand ils approuvent une demande de renouvellement d’affectation de groupe, les administrateurs de ressources doivent entrer de nouvelles dates de début et de fin et un nouveau type d’affectation.

## <a name="next-steps"></a>Étapes suivantes

- [Approuver ou refuser des demandes d’affectation de groupe d’accès privilégié dans Privileged Identity Management](groups-approval-workflow.md)
- [Configurer les paramètres de groupe d’accès privilégié dans Privileged Identity Management](groups-role-settings.md)

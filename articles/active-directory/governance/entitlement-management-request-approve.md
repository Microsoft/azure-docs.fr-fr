---
title: Approuver ou refuser des demandes d'accès - Gestion des droits d'utilisation Azure AD
description: Découvrez comment utiliser le portail Mon Accès pour approuver ou refuser les demandes d’accès à un package d’accès dans la gestion des droits d’utilisation Azure Active Directory.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: mamtakumar
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: mamkumar
ms.collection: M365-identity-device-management
ms.openlocfilehash: fddb3b171e5a26273cb2e0045f11e3a4dbb48c5f
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97655099"
---
# <a name="approve-or-deny-access-requests-in-azure-ad-entitlement-management"></a>Approuver ou refuser des demandes d’accès dans la gestion des droits d’utilisation Azure AD

Avec la gestion des droits d’utilisation Azure AD, vous pouvez configurer des stratégies pour exiger l’approbation des packages d’accès et choisir un ou plusieurs approbateurs. Cet article décrit comment les approbateurs désignés peuvent approuver ou refuser les demandes pour les packages d’accès.

## <a name="open-request"></a>Ouvrir la demande

La première étape pour approuver ou refuser les demandes d’accès consiste à rechercher et ouvrir la demande d’accès en attente d’approbation. Il existe deux manières d’ouvrir la demande d’accès.

**Rôle prérequis :** Approbateur

1. Recherchez un e-mail de Microsoft Azure vous demandant d’approuver ou de refuser une demande. Voici un exemple d’e-mail :

    ![E-mail concernant l’approbation de la demande pour un package d’accès](./media/entitlement-management-shared/approver-request-email.png)

1. Cliquez sur le lien **Approuver ou refuser la demande** pour ouvrir la demande d’accès.

1. Connectez-vous au portail Mon Accès.

Si vous n’avez pas reçu l’e-mail, vous pouvez localiser les demandes d’accès en attente d’approbation comme suit.

1. Connectez-vous au portail Mon Accès sur [https://myaccess.microsoft.com](https://myaccess.microsoft.com).  (Pour le gouvernement des États-Unis, le domaine figurant dans le lien du portail Mon Accès sera `myaccess.microsoft.us`.)

1. Dans le menu de gauche, cliquez sur **Approbations** pour afficher la liste des demandes d’accès en attente d’approbation.

1. Dans l’onglet **En attente**, recherchez la demande.

## <a name="view-requestors-answers-to-questions-preview"></a>Afficher les réponses du demandeur aux questions (préversion)

1. Accédez à l’onglet **Approbations** dans Mon accès.

1. Accédez à la demande que vous souhaitez approuver, puis cliquez sur **Détails**. Vous pouvez également cliquer sur **Approuver** ou **Refuser** si vous êtes prêt à prendre une décision.

1. Cliquez sur **Détails de la demande**.

    ![Portail Mon Accès - Demande d’accès - Clic sur les détails de la demande](./media/entitlement-management-request-approve/requestor-information-request-details.png)

1. Les informations fournies par le demandeur se trouvent au bas du panneau.

    ![Capture d’écran montrant les détails pour la requête](./media/entitlement-management-request-approve/requestor-information-requestor-answers.png)

1. En fonction des informations fournies par le demandeur, vous pouvez approuver ou refuser la demande. Consultez les étapes de la rubrique Approuver ou refuser une demande pour obtenir de l’aide.

## <a name="approve-or-deny-request"></a>Approuver ou refuser la demande

Une fois que vous avez ouvert une demande d’accès en attente d’approbation, vous pouvez consulter les détails qui vous aideront à prendre la décision.

**Rôle prérequis :** Approbateur

1. Cliquez sur le lien **Afficher** pour ouvrir le volet Demande d’accès.

1. Cliquez sur **Détails** pour afficher les détails de la demande d’accès.

    Les détails incluent le nom d’utilisateur, l’organisation, la date de début et de fin le cas échéant, la justification professionnelle, la date de soumission de la demande et la date d’expiration de la demande.

1. Cliquez sur **Approuver** ou **Refuser**.

1. Si nécessaire, indiquez un motif.

    ![Capture d’écran montrant la page dans laquelle vous acceptez ou refusez la demande.](./media/entitlement-management-request-approve/my-access-approve-request.png)

1. Cliquez sur **Soumettre** pour soumettre votre décision.

    Si une stratégie est configurée avec plusieurs approbateurs, un seul approbateur doit prendre une décision concernant l’approbation en attente. Une fois qu’un approbateur a rendu sa décision concernant la demande d’accès, celle-ci est terminée et n’est plus disponible pour les autres approbateurs. Les autres approbateurs peuvent voir la décision et l’approbateur qui est intervenu dans leur portail Mon Accès. À ce stade, seule l’approbation en une étape est prise en charge.

    Si aucun des approbateurs configurés ne peut approuver ou refuser la demande d’accès, celle-ci expire après la période définie. L’utilisateur est averti que sa demande d’accès a expiré et qu’il doit la soumettre à nouveau.

## <a name="next-steps"></a>Étapes suivantes

- [Demander l’accès à un package d’accès](entitlement-management-request-access.md)
- [Processus de requête et notifications par e-mail](entitlement-management-process.md)

---
title: Portail d’inscription en libre-service pour B2B Collaboration - Azure AD
description: Découvrez comment personnaliser le workflow d’intégration pour les utilisateurs d’Azure Active Directory B2B en fonction des besoins de votre organisation.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 02/12/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8a5b800e78448afcc970010535ba12b543d3cc74
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96860505"
---
# <a name="self-service-for-azure-ad-b2b-collaboration-sign-up"></a>Inscription en libre-service pour Azure AD B2B Collaboration

Les clients peuvent faire beaucoup d’opérations avec les fonctionnalités intégrées qui sont exposées par le biais du [portail Azure](https://portal.azure.com) et du [panneau d’accès aux applications](https://myapps.microsoft.com) pour les utilisateurs. Cependant, vous devrez peut-être personnaliser le flux de travail d’intégration pour les utilisateurs B2B en fonction des besoins de votre organisation.

## <a name="azure-ad-entitlement-management-for-b2b-guest-user-sign-up"></a>Gestion des droits d’utilisation Azure AD pour l’inscription des utilisateurs invités B2B

En tant qu’organisation invitante, vous ne pouvez pas savoir à l’avance quels collaborateurs externes auront besoin d’accéder à vos ressources. Vos devez disposer d’un moyen de permettre à des utilisateurs d’entreprises partenaires de s’inscrire avec les stratégies que vous contrôlez. Si vous souhaitez autoriser les utilisateurs d’autres organisations à demander l’accès, et qu’après approbation ils soient provisionnés avec des comptes invités et affectés à des groupes, des applications et des sites SharePoint Online, vous pouvez utiliser la [gestion des droits d’utilisation Azure AD](../governance/entitlement-management-overview.md) pour configurer des stratégies qui [gèrent l’accès pour les utilisateurs externes](../governance/entitlement-management-external-users.md#how-access-works-for-external-users).

## <a name="azure-active-directory-b2b-invitation-api"></a>API d’invitation B2B Azure Active Directory

Les organisations peuvent utiliser l’[API gestionnaire d’invitation Microsoft Graph](/graph/api/resources/invitation) pour créer leurs propres expériences d’intégration pour les utilisateurs invités B2B. Quand vous souhaitez proposer une inscription d’utilisateur invité en libre-service B2B, nous vous recommandons d’utiliser la [gestion des droits d’utilisation Azure AD](../governance/entitlement-management-overview.md). Si par contre vous souhaitez créer votre propre expérience, vous pouvez utiliser l’[API de création d’invitation](/graph/api/invitation-post?tabs=http) pour envoyer automatiquement votre e-mail d’invitation personnalisé directement à l’utilisateur B2B, par exemple. Votre application peut aussi utiliser l’inviteRedeemUrl retourné dans la réponse de création pour créer votre propre invitation (par le biais du mécanisme de communication de votre choix) pour l’utilisateur invité.

## <a name="next-steps"></a>Étapes suivantes

* [Qu'est-ce que la collaboration B2B d'Azure AD ?](what-is-b2b.md)
* [Prix des identités externes](external-identities-pricing.md)
* [Questions fréquemment posées (FAQ) sur Azure Active Directory B2B Collaboration](faq.md)

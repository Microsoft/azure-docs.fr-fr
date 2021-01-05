---
title: Audit et création de rapports relatifs à un utilisateur B2B Collaboration – Azure AD
description: Les propriétés de l’utilisateur invité sont configurables dans Azure Active Directory B2B Collaboration
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 05/11/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: a805f643e5f93270fe4721f527073baa9a1e4c8d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87907307"
---
# <a name="auditing-and-reporting-a-b2b-collaboration-user"></a>Audit et création de rapports relatifs à un utilisateur B2B Collaboration
Avec les utilisateurs invités, vous disposez de fonctionnalités d’audit identiques à celles des utilisateurs membres. 

## <a name="access-reviews"></a>Révisions d’accès
Vous pouvez utiliser les révisions d’accès pour vérifier périodiquement si les utilisateurs invités ont toujours besoin d’accéder à vos ressources. La fonctionnalité **Révisions d’accès** est disponible dans **Azure Active Directory** sous **Identités externes** > **Révisions d’accès**. Vous pouvez également rechercher des « révisions d’accès » à partir de **Tous les services** dans le portail Azure. Pour apprendre à utiliser les révisions d’accès, consultez [Gérer l’accès invité avec les révisions d’accès Azure AD](../governance/manage-guest-access-with-access-reviews.md).

## <a name="audit-logs"></a>Journaux d’audit

Les journaux d’audit d’Azure AD fournissent des enregistrements de l’activité du système et des utilisateurs, y compris les activités initiées par les utilisateurs invités. Pour accéder aux journaux d’audit, dans **Azure Active Directory**, sous **Supervision**, sélectionnez **Journaux d’audit**. Voici un exemple de l’historique d’invitation et d’échange pour l’invité Sam Oogle :

![Capture d’écran affichant un exemple de sortie d’un journal d’audit](./media/auditing-and-reporting/audit-log.png)

Vous pouvez explorer chacun de ces événements pour en obtenir les détails. Examinons, par exemple, les détails de l’acceptation.

![Capture d’écran affichant un exemple de sortie de détails d’activités](./media/auditing-and-reporting/activity-details.png)

Vous pouvez également exporter ces journaux d’activité à partir d’Azure AD et utiliser l’outil de création de rapports de votre choix afin d’obtenir des rapports personnalisés.

### <a name="next-steps"></a>Étapes suivantes

- [Propriétés de l’utilisateur B2B Collaboration](user-properties.md)


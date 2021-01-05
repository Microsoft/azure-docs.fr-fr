---
title: Limitations de B2B Collaboration - Azure Active Directory | Microsoft Docs
description: Limitations actuelles d’Azure Active Directory B2B Collaboration
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 05/29/2019
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: elisolMS
ms.collection: M365-identity-device-management
ms.openlocfilehash: e4f960819aa208dcc8d3e476fc45a766452b612c
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96168948"
---
# <a name="limitations-of-azure-ad-b2b-collaboration"></a>Limitations d’Azure AD B2B Collaboration
Azure Active Directory (Azure AD) B2B Collaboration subit actuellement les limitations décrites dans le présent article.

## <a name="possible-double-multi-factor-authentication"></a>Risque de redondance de l’authentification multifacteur
Avec Azure AD B2B, vous pouvez appliquer l’authentification multifacteur au niveau de l’organisation de la ressource (l’organisation à l’origine de l’invitation). Les raisons de cette approche sont détaillées dans l’article [Conditional access for B2B collaboration users](conditional-access.md) (Accès conditionnel pour les utilisateurs de B2B Collaboration). Si un partenaire a déjà configuré et appliqué l’authentification multifacteur, ses utilisateurs devront peut-être effectuer l’authentification une fois dans leur organisation d’origine, puis de nouveau dans les vôtres.

## <a name="instant-on"></a>Activation instantanée
Dans les flux B2B Collaboration, nous ajoutons des utilisateurs au répertoire et les mettons à jour de manière dynamique pendant l’échange d’invitation, l’affectation d’application, etc. Les mises à jour et les écritures se produisent d’ordinaire dans une instance de répertoire et doivent être répliquées entre toutes les instances. La réplication est terminée une fois toutes les instances mises à jour. Parfois, lorsque l’objet est écrit ou mis à jour dans une instance et quand l’appel pour récupérer cet objet se fait vers une autre instance, des latences de réplication peuvent se produire. Si cela se produit, actualisez ou recommencez. Si vous écrivez une application à l’aide de notre API, effectuer de nouvelles tentatives avec des temporisations peut être une pratique judicieuse et préventive pour atténuer ce problème.

## <a name="azure-ad-directories"></a>Répertoires Azure AD
Azure AD B2B est soumis aux limites de répertoire du service Azure AD. Pour plus d’informations sur le nombre de répertoires qu’un utilisateur peut créer et sur le nombre de répertoires auxquels un utilisateur ou un utilisateur invité peuvent appartenir, consultez l’article [Restrictions et limites de service Azure Active Directory](../enterprise-users/directory-service-limits-restrictions.md).

## <a name="national-clouds"></a>Clouds nationaux
[Les clouds nationaux](../develop/authentication-national-cloud.md) sont des instances d’Azure physiquement isolées. B2B Collaboration n’est pas pris en charge au-delà des limites du cloud national. Par exemple, si votre client Azure se trouve dans le cloud public général, vous ne pouvez pas inviter un utilisateur dont le compte se trouve dans un cloud national. Pour collaborer avec cet utilisateur, demandez-lui une autre adresse e-mail ou créez-lui un compte d’utilisateur membre dans votre répertoire.

## <a name="azure-us-government-clouds"></a>Clouds Azure US Government
Dans le cloud Azure - Gouvernement des États-Unis, la collaboration B2B est prise en charge entre locataires se trouvant à l’intérieur du cloud et prenant en charge la collaboration B2B. Les locataires Azure - Gouvernement des États-Unis qui prennent en charge la collaboration B2B peuvent également collaborer avec des utilisateurs sociaux à l’aide de comptes Microsoft ou Google. Si vous invitez un utilisateur extérieur à ces groupes (par exemple, si l’utilisateur se trouve dans un locataire qui ne fait pas partie du cloud Azure - Gouvernement des États-Unis ou qui ne prend pas encore en charge la collaboration B2B), l’invitation échoue ou l’utilisateur ne peut pas utiliser l’invitation. Pour plus d’informations sur les autres limitations, consultez [Différences entre Azure Active Directory Premium P1 et P2](../../azure-government/compare-azure-government-global-azure.md#azure-active-directory-premium-p1-and-p2).

### <a name="how-can-i-tell-if-b2b-collaboration-is-available-in-my-azure-us-government-tenant"></a>Comment puis-je savoir si B2B Collaboration est disponible dans mon locataire Azure US Government ?
Pour savoir si votre locataire du cloud Azure US Government prend en charge B2B Collaboration, procédez comme suit :

1. Dans un navigateur, accédez à l’URL suivante, en remplaçant *&lt;tenantname&gt;* par le nom de votre locataire :

   `https://login.microsoftonline.com/<tenantname>/v2.0/.well-known/openid-configuration`

2. Recherchez `"tenant_region_scope"` dans la réponse JSON :

   - Si `"tenant_region_scope":"USGOV”` apparaît, c’est que B2B est pris en charge.
   - Si `"tenant_region_scope":"USG"` apparaît, c’est que B2B n’est pas pris en charge.

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants sur Azure AD B2B Collaboration :

- [Qu'est-ce que la collaboration B2B d'Azure AD ?](what-is-b2b.md)
- [Déléguer des invitations B2B Collaboration](delegate-invitations.md)
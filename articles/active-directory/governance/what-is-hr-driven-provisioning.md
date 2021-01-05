---
title: Qu’est-ce que le provisionnement piloté par les RH avec Azure Active Directory ? | Microsoft Docs
description: Présente une vue d’ensemble du provisionnement piloté par les RH.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: overview
ms.date: 10/30/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1ffe8c1397525348e472e965a407909fee36152d
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96168445"
---
# <a name="what-is-hr-driven-provisioning"></a>Qu’est-ce que le provisionnement piloté par les RH ?

![Provisionnement piloté par les RH](./media/what-is-hr-driven-provisioning/cloud2a.png)

Le provisionnement piloté par les RH est le processus de création d’identités numériques selon un système de ressources humaines.  Les systèmes RH deviennent la source de noms de ces identités numériques nouvellement créées et il s’agit souvent du point de départ pour de nombreux processus de provisionnement.  Par exemple, si un nouvel employé rejoint votre entreprise, il est créé dans le système de ressources humaines.  La création déclenche le provisionnement d’un compte d’utilisateur dans Active Directory, puis Azure AD Connect provisionne ce compte pour Azure AD, etc.

Le provisionnement piloté par les RH peut être local ou cloud.

## <a name="on-premises-based-hr-provisioning"></a>Provisionnement piloté par les RH local
Le provisionnement piloté par les RH local est effectué avec un système RH local et un moyen de provisionner de nouvelles identités numériques.

Les systèmes RH sont fournis dans divers packages et bundles de logiciels, et peuvent utiliser des serveurs SQL, des annuaires LDAP, etc.

Actuellement, les solutions de provisionnement piloté par les RH local de Microsoft utilisent Microsoft Identity Manager pour déclencher le provisionnement lors de la création d’une identité dans ces systèmes RH.

À l’aide de MIM, vous pouvez provisionner des utilisateurs de vos systèmes RH locaux dans Active Directory ou Azure AD.

Pour plus d’informations sur Microsoft Identity Manager et les systèmes qu’il prend en charge, consultez la documentation [Microsoft Identity Manager](/microsoft-identity-manager/microsoft-identity-manager-2016).

[!INCLUDE [active-directory-hr-provisioning.md](../../../includes/active-directory-hr-provisioning.md)]



## <a name="next-steps"></a>Étapes suivantes 
- [Qu’est-ce que la gestion du cycle de vie des identités](what-is-identity-lifecycle-management.md)
- [Présentation du provisionnement](what-is-provisioning.md)
- [Qu’est-ce que le provisionnement d’application ?](what-is-app-provisioning.md)
- [Qu’est-ce que le provisionnement inter-annuaires ?](what-is-inter-directory-provisioning.md)
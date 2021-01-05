---
title: Nouveautés d’Azure AD dans Microsoft 365 Government | Microsoft Docs
description: Découvrez certaines modifications apportées à Azure Active Directory (Azure AD) dans l’instance cloud Microsoft 365 Government, susceptibles de vous concerner.
services: active-directory
author: eross-msft
manager: daveba
ms.author: lizross
ms.reviewer: sumitp
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 05/07/2019
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: a0efc4bc8f89b0fbefbba171d80a3f8a1ed5e7f6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89318928"
---
# <a name="whats-new-for-azure-active-directory-in-microsoft-365-government"></a>Nouveautés d’Azure Active Directory dans Microsoft 365 Government

Nous avons apporté quelques modifications à Azure Active Directory (Azure AD) dans l’instance cloud Microsoft 365 Government. Ces modifications s'appliquent aux clients utilisant les services suivants :

- Microsoft Azure Government

- Microsoft 365 Government - GCC High

- Microsoft 365 Government - DoD

Cet article ne s’applique pas aux clients Microsoft 365 Government – GCC.

## <a name="changes-to-the-initial-domain-name"></a>Modifications apportées au nom de domaine initial

Lors de l'inscription initiale de votre organisation à un service en ligne Microsoft 365 Government, vous avez été invité à choisir le nom de domaine de votre organisation, `<your-domain-name>.onmicrosoft.com`. Si vous disposez déjà d’un nom de domaine avec le suffixe .com, rien ne change.

En revanche, si vous vous inscrivez à un nouveau service Microsoft 365 Government, il vous sera demandé de choisir un nom de domaine avec le suffixe `.us`. Il s'agira donc de `<your-domain-name>.onmicrosoft.us`.

>[!Note]
>Cette modification ne s’applique pas aux clients gérés par des fournisseurs de services cloud.

## <a name="changes-to-portal-access"></a>Modifications apportées à l’accès au portail

Nous avons mis à jour les points de terminaison du portail pour Microsoft Azure Government, Microsoft 365 Government - GCC High et Microsoft 365 Government – DoD, comme indiqué dans la [table de mappage des points de terminaison](#endpoint-mapping).

Jusqu'à présent, les clients pouvaient se connecter à l’aide des portails Azure (portal.azure.com) et Office 365 (portal.office.com) dans le monde entier. Avec cette mise à jour, les clients doivent désormais se connecter à l’aide des portails spécifiques Microsoft Azure Government, Microsoft 365 Government - GCC High et Microsoft 365 Government - DoD.

## <a name="endpoint-mapping"></a>Mappage de point de terminaison

La table suivante répertorie les points de terminaison pour tous les clients :

| Nom | Détails du point de terminaison |
|------|------------------|
| Portails |Microsoft Azure Government : https://portal.azure.us<p>Microsoft 365 Government - GCC High : https://portal.office365.us<p>Microsoft 365 Government - DoD: https://portal.apps.mil |
| Point de terminaison d'autorité Azure Active Directory | https://login.microsoftonline.us |
| API Microsoft Graph pour Microsoft 365 Government - GCC High | https://graph.microsoft.us |
| API Microsoft Graph pour Microsoft 365 Government - DoD | https://dod-graph.microsoft.us |
| Points de terminaison des services Azure Government | Pour plus d'informations, consultez [Guide du développeur Azure Government](../../azure-government/documentation-government-developer-guide.md) |
| Points de terminaison de Microsoft 365 Government - GCC High | Pour plus d’informations, consultez [Points de terminaison d'Office 365 US Government - GCC High](/office365/enterprise/office-365-u-s-government-gcc-high-endpoints) |
| Microsoft 365 Government - DoD | Pour plus d’informations, consultez [Points de terminaison d'Office 365 US Government DoD](/office365/enterprise/office-365-u-s-government-dod-endpoints) |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, voir les articles suivants :

- [Qu’est-ce qu’Azure Government ?](../../azure-government/documentation-government-welcome.md)

- [Mise à jour du point de terminaison AAD Azure Government](https://devblogs.microsoft.com/azuregov/azure-government-aad-authority-endpoint-update/)

- [Points de terminaison de Microsoft Graph dans le cloud US Government](https://developer.microsoft.com/graph/blogs/new-microsoft-graph-endpoints-in-us-government-cloud/)

- [Office 365 US Government GCC High et DoD](/office365/servicedescriptions/office-365-platform-service-description/office-365-us-government/gcc-high-and-dod)
---
title: Renforcer la résilience en utilisant Évaluation continue de l’accès dans Azure Active Directory
description: Guide destiné aux architectes et aux administrateurs informatiques sur l’utilisation de CAE
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 11/30/2020
ms.author: baselden
ms.reviewer: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8ad36c2a7f47948d9362b85e78355e6046cda703
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95918979"
---
# <a name="build-resilience-by-using-continuous-access-evaluation"></a>Renforcer la résilience en utilisant Évaluation continue de l’accès

[Évaluation continue de l’accès](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-continuous-access-evaluation) (CAE) permet aux applications Azure AD de s’abonner à des événements critiques qui peuvent ensuite être évalués et appliqués. Cela comprend l’évaluation des événements suivants :

* Le compte d’utilisateur est supprimé ou désactivé.

* Le mot de passe d’un utilisateur a été modifié.

* MFA est activée pour l’utilisateur.

* L’administrateur révoque explicitement un jeton.

* Un risque pour l’utilisateur élevé est détecté.

Par conséquent, les applications peuvent rejeter des jetons non expirés en fonction des événements signalés par Azure AD, comme illustré dans le diagramme suivant.

![diagramme conceptuel de CAE](./media/resilience-with-cae/admin-resilience-continuous-access-evaluation.png)

## <a name="how-does-cae-help"></a>Comment la fonctionnalité CAE aide-t-elle ?

Ce mécanisme permet à Azure AD d’émettre des jetons de plus longue durée, tout en permettant aux applications de révoquer l’accès et de forcer la réauthentification uniquement lorsque cela est nécessaire. Le résultat net de ce schéma est une diminution des appels pour l’acquisition de jetons, ce qui signifie que le flux de bout en bout est plus résilient. 

Pour utiliser CAE, le service et le client doivent tous deux être compatibles avec la fonctionnalité. Les services Microsoft 365 tels qu’Exchange Online, Teams et SharePoint Online prennent en charge CAE. Côté client, les expériences basées sur un navigateur qui utilisent ces services Office 365 (par exemple, l’application web Outlook) et des versions spécifiques des clients natifs d’Office 365 sont compatibles avec la fonctionnalité CAE. D’autres services de cloud computing Microsoft deviendront compatibles avec CAE.

Microsoft collabore avec le secteur d’activité pour créer des [normes](https://openid.net/wg/sse/) qui permettront aux applications tierces d’utiliser cette fonctionnalité. Vous pouvez également développer des applications prenant en charge CAE. Pour plus d’informations, consultez Comment renforcer la résilience de votre application.

## <a name="how-do-i-implement-cae"></a>Comment implémenter CAE ?

* [Activez CAE](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-continuous-access-evaluation) dans la configuration de sécurité Azure AD.

* Assurez-vous que votre organisation utilise des [versions compatibles](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-continuous-access-evaluation) des applications natives Microsoft Office.

* [Optimisez vos invites de réauthentification](https://docs.microsoft.com/azure/active-directory/authentication/concepts-azure-multi-factor-authentication-prompts-session-lifetime).

 
## <a name="next-steps"></a>Étapes suivantes
Ressources de résilience pour les administrateurs et les architectes
 
* [Renforcer la résilience grâce à la gestion des informations d’identification](resilience-in-credentials.md)

* [Renforcer la résilience grâce aux états des appareils](resilience-with-device-states.md)

* [Renforcer la résilience de l’authentification des utilisateurs externes](resilience-b2b-authentication.md)

* [Renforcer la résilience de votre authentification hybride](resilience-in-hybrid.md)

* [Renforcer la résilience de l’accès aux applications avec Proxy d’application](resilience-on-premises-access.md)

Ressources de résilience pour les développeurs

* [Renforcer la résilience de l’IAM dans vos applications](resilience-app-development-overview.md)

* [Renforcer la résilience de vos systèmes CIAM](resilience-b2c.md)

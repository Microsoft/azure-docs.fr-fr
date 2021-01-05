---
title: Renforcer la résilience de votre infrastructure IAM avec Azure Active Directory
description: Guide destiné aux architectes et aux administrateurs informatiques qui ne renforcent pas la résilience aux perturbations de leur infrastructure IAM.
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
ms.openlocfilehash: f65ab02e06319519548eaa2c02120691a0ceef02
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498555"
---
# <a name="build-resilience-in-your-identity-and-access-management-infrastructure"></a>Renforcer la résilience de votre infrastructure de gestion des identités et des accès

Azure Active Directory est un système mondial de gestion des identités et des accès dans le cloud qui fournit des services essentiels tels que l’authentification et l’autorisation pour accéder aux ressources de votre organisation. Ce document fournit des conseils pour comprendre, inclure et atténuer le risque d’interruption des services d’authentification ou d’autorisation pour les ressources qui dépendent d’Azure Active Directory (Azure AD). 

L’ensemble de documents est conçu pour :

* Les architectes d’identité

* Les propriétaires d’un service d’identité

* Les équipes d’opérations d’identité

Reportez-vous également à la documentation pour [les développeurs d’applications](https://aka.ms/azureadresilience/developer) et pour [les systèmes Azure AD B2C](resilience-b2c.md).

## <a name="what-is-resilience"></a>Qu’est-ce que la résilience ?

Dans le contexte de votre infrastructure d’identité, la résilience est la capacité à résister à des interruptions de service, telles que l’authentification et l’autorisation, ou à la défaillance d’autres composants avec un impact minimal ou inexistant sur votre activité, vos utilisateurs et vos opérations. L’impact d’une interruption peut être grave, et la résilience requiert une planification diligente.

## <a name="why-worry-about-disruption"></a>Pourquoi s’inquiéter des interruptions ?

Chaque appel au système d’authentification est soumis à une interruption si un composant de l’appel échoue. Lorsque l’authentification est interrompue, en raison des défaillances des composants sous-jacents, vos utilisateurs n’accèdent pas à leurs applications. Il est donc important pour votre résilience de réduire le nombre d’appels d’authentification et le nombre de dépendances dans ces appels. Les développeurs d’applications peuvent exercer un certain contrôle sur la fréquence à laquelle les jetons sont demandés. Par exemple, collaborez avec vos développeurs pour vous assurer qu’ils utilisent des identités managées Azure AD pour leurs applications, dans la mesure du possible. 

Dans un système d’authentification basé sur des jetons comme Azure AD, l’application d’un utilisateur (client) doit acquérir un jeton de sécurité auprès du système d’identité pour pouvoir accéder à une application ou à une autre ressource. Pendant la période de validité, un client peut présenter le même jeton plusieurs fois pour accéder à l’application.

Lorsque le jeton présenté à l’application expire, l’application rejette le jeton et le client doit acquérir un nouveau jeton auprès d’Azure AD. L’acquisition d’un nouveau jeton peut nécessiter une interaction avec l’utilisateur, comme les invites à fournir des informations d’identification ou la satisfaction d’autres exigences du système d’authentification. La réduction de la fréquence des appels d’authentification avec des jetons à longue durée de vie réduit les interactions inutiles. Cependant, vous devez trouver un équilibre entre la durée de vie des jetons et le risque créé par moins d’évaluations de stratégie. Pour plus d’informations sur la gestion des durées de vie des jetons, consultez cet article sur [l’optimisation des invites de réauthentification](https://docs.microsoft.com/azure/active-directory/authentication/concepts-azure-multi-factor-authentication-prompts-session-lifetime).

## <a name="ways-to-increase-resilience"></a>Moyens d’accroître la résilience
Le diagramme suivant illustre six méthodes concrètes permettant d’accroître la résilience. Chaque méthode est expliquée en détail dans les articles liés dans la section Étapes suivantes de cet article.
  
![Diagramme montrant une vue d’ensemble de la résilience administrative](./media/resilience-in-infrastructure/admin-resilience-overview.png)

## <a name="next-steps"></a>Étapes suivantes
Ressources de résilience pour les administrateurs et les architectes
 
* [Renforcer la résilience grâce à la gestion des informations d’identification](resilience-in-credentials.md)

* [Renforcer la résilience grâce aux états des appareils](resilience-with-device-states.md)

* [Renforcer la résilience en utilisant Évaluation continue de l’accès (CAE)](resilience-with-continuous-access-evaluation.md)

* [Renforcer la résilience de l’authentification des utilisateurs externes](resilience-b2b-authentication.md)

* [Renforcer la résilience de votre authentification hybride](resilience-in-hybrid.md)

* [Renforcer la résilience de l’accès aux applications avec Proxy d’application](resilience-on-premises-access.md)

Ressources de résilience pour les développeurs

* [Renforcer la résilience de l’IAM dans vos applications](resilience-app-development-overview.md)

* [Renforcer la résilience de vos systèmes CIAM](resilience-b2c.md)

---
title: Renforcer la résilience de l’accès aux applications avec Proxy d’application
description: Guide destiné aux architectes et aux administrateurs informatiques sur l’utilisation de Proxy d’application pour un accès résilient aux applications locales
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
ms.openlocfilehash: f8bfc3fb239f30911eddf0aa27496a465e36c486
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95918981"
---
# <a name="build-resilience-in-application-access-with-application-proxy"></a>Renforcer la résilience de l’accès aux applications avec Proxy d’application

Le Proxy d’application est une fonctionnalité d’Azure AD qui permet aux utilisateurs d’accéder à des applications web locales à partir d’un client distant. Proxy d’application inclut le service Proxy d’application dans le cloud et les connecteurs Proxy d’Application, qui s’exécutent sur un serveur local. 

Les utilisateurs accèdent aux ressources locales par le biais d’une URL publiée via Proxy d’application. Ils sont redirigés vers la page de connexion Azure AD. Le service Proxy d’application d’Azure AD envoie ensuite un jeton au connecteur Proxy d’application dans le réseau d’entreprise, qui transmet le jeton à Windows Server Active Directory. L’utilisateur authentifié peut alors accéder à la ressource locale. Dans le diagramme ci-dessous, des [connecteurs](../manage-apps/application-proxy-connectors.md) sont présentés dans un [groupe de connecteurs](../manage-apps/application-proxy-connector-groups.md).

> [!IMPORTANT]
> Lorsque vous publiez vos applications via Proxy d’application, vous devez implémenter la [planification de la capacité et la redondance appropriée pour les connecteurs Proxy d’application](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-connectors#capacity-planning).

![Diagramme d’architecture de l’application y](./media/resilience-on-prem-access/admin-resilience-app-proxy.png))

## <a name="how-do-i-implement-application-proxy"></a>Comment implémenter Proxy d’application ?

Pour implémenter l’accès à distance à l’aide de Proxy d’application Azure AD, consultez les ressources suivantes.

* [Planification d’un déploiement de proxy d’application](../manage-apps/application-proxy-deployment-plan.md)

* [Meilleures pratiques pour la haute disponibilité et l’équilibrage de charge](../manage-apps/application-proxy-high-availability-load-balancing.md)

* [Configurer des serveurs proxy](../manage-apps/application-proxy-configure-connectors-with-proxy-servers.md)

* [Concevoir une stratégie résiliente de contrôle d’accès](../authentication/concept-resilient-controls.md)

## <a name="next-steps"></a>Étapes suivantes
Ressources de résilience pour les administrateurs et les architectes
 
* [Renforcer la résilience grâce à la gestion des informations d’identification](resilience-in-credentials.md)

* [Renforcer la résilience grâce aux états des appareils](resilience-with-device-states.md)

* [Renforcer la résilience en utilisant Évaluation continue de l’accès (CAE)](resilience-with-continuous-access-evaluation.md)

* [Renforcer la résilience de l’authentification des utilisateurs externes](resilience-b2b-authentication.md)

* [Renforcer la résilience de votre authentification hybride](resilience-in-hybrid.md)

Ressources de résilience pour les développeurs

* [Renforcer la résilience de l’IAM dans vos applications](resilience-app-development-overview.md)

* [Renforcer la résilience de vos systèmes CIAM](resilience-b2c.md)

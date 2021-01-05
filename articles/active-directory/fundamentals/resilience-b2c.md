---
title: Renforcer la résilience de la gestion des identités et des accès client à l’aide d’Azure AD B2C | Microsoft Docs
description: Méthodes pour renforcer la résilience de la gestion des identités et des accès client à l’aide d’Azure AD B2C
services: active-directory
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: how-to
author: gargi-sinha
ms.author: gasinh
manager: martinco
ms.reviewer: ''
ms.date: 11/30/2020
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: ba7b8ff2555fd7014cf16f66745721c6425ff868
ms.sourcegitcommit: 2e9643d74eb9e1357bc7c6b2bca14dbdd9faa436
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96029521"
---
# <a name="build-resilience-in-your-customer-identity-and-access-management-with-azure-active-directory-b2c"></a>Renforcer la résilience de votre gestion des identités et des accès client avec Azure Active Directory B2C

[Azure Active Directory (AD) B2C](https://docs.microsoft.com/azure/active-directory-b2c/overview) est une plateforme de gestion des identités et des accès client (CIAM) conçue pour vous aider à lancer avec succès vos applications critiques destinées aux clients. Nous disposons de nombreuses fonctionnalités de [résilience](https://azure.microsoft.com/blog/advancing-azure-active-directory-availability/) intégrées qui sont conçues pour aider notre service à s’adapter à vos besoins et à améliorer la résilience en cas d’éventuelles situations de panne. En outre, lors du lancement d’une application critique, il est important de tenir compte de différents éléments de conception et de configuration de votre application, ainsi que de la façon dont l’application est configurée dans Azure AD B2C pour vous assurer d’avoir un comportement résilient en réponse à des scénarios de panne ou de défaillance. Dans cet article, nous allons aborder certaines des meilleures pratiques pour vous aider à renforcer la résilience.

Un service résilient est un service qui continue à fonctionner malgré les interruptions. Vous pouvez contribuer à améliorer la résilience dans votre service en :

- comprenant tous les composants

- éliminant les points de défaillance uniques

- isolant les composants défaillants pour limiter leur impact

- assurant la redondance avec des mécanismes de basculement rapide et des chemins de récupération

Lors du développement de votre application, nous vous recommandons de réfléchir à la manière [d’accroître la résilience de l’authentification et de l’autorisation dans vos applications](resilience-app-development-overview.md) avec les composants d’identité de votre solution. Cet article tente d’aborder les améliorations apportées à la résilience spécifique aux applications Azure AD B2C. Nos recommandations sont regroupées par fonctions CIAM.

![Image illustrant les composants de la gestion des identités et des accès client](media/resilience-b2c/high-level-components.png) Dans les sections suivantes, nous vous guiderons pour renforcer la résilience dans les domaines suivants :

- [Expérience de l’utilisateur final](resilient-end-user-experience.md) : Activez un plan de secours pour votre flux d’authentification et atténuez l’impact potentiel d’une interruption du service d’authentification Azure AD B2C.

- [Interfaces avec processus externes](resilient-external-processes.md) : Renforcez la résilience de vos applications et interfaces en récupérant des erreurs.  

- [Meilleures pratiques de développement](resilience-b2c-developer-best-practices.md) : Évitez la fragilité due à des problèmes courants de stratégie personnalisée et améliorez la gestion des erreurs dans les domaines tels que les interactions avec les vérificateurs de revendications, les applications tierces et les API REST.

- [Supervision et analytique](resilience-with-monitoring-alerting.md) : Évaluez l’intégrité de votre service en surveillant les indicateurs clés et détectez les défaillances et les interruptions de performances par le biais d’alertes.

- [Renforcer la résilience de votre infrastructure d’authentification](resilience-in-infrastructure.md)

- [Augmenter la résilience de l’authentification et de l’autorisation dans vos applications](resilience-app-development-overview.md)

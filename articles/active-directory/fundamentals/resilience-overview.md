---
title: Création d’une gestion résiliente des identités et des accès avec Azure Active Directory
description: Guide d’intégration de la résilience à la perturbation des systèmes d’identité destiné aux architectes, aux administrateurs informatiques et aux développeurs.
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
ms.openlocfilehash: 39021fecb79f6b8e7536daad592378ed998760a7
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96454339"
---
# <a name="building-resilience-into-identity-and-access-management-with-azure-active-directory"></a>Intégration de la résilience à la gestion des identités et des accès avec Azure Active Directory

La gestion des identités et des accès (IAM) est un framework de processus, de stratégies et de technologies qui facilitent la gestion des identités et de leurs accès. Elle comprend les nombreux composants qui prennent en charge l’authentification et l’autorisation des comptes d’utilisateur et d’autres comptes dans le système.

La résilience IAM correspond à la capacité à supporter la perturbation des composants du système et à récupérer avec un impact minimal sur les activités, les utilisateurs, les clients et les opérations. En réduisant les dépendances, la complexité et les points de défaillance uniques tout en garantissant une gestion complète des erreurs, vous augmentez votre résilience.

Les perturbations peuvent provenir de n’importe quel composant des systèmes IAM. Pour créer un système IAM résilient, partez du principe qu’il s’en produira et préparez-vous-y. 

Lorsque vous planifiez la résilience de votre solution IAM, tenez compte des éléments suivants : 

* Vos applications qui reposent sur votre système IAM

* Les infrastructures publiques utilisées par vos appels d’authentification, notamment les opérateurs de télécommunications, les fournisseurs de services Internet et les fournisseurs de clés publiques

* Vos fournisseurs d’identité cloud et locaux

* Les autres services qui reposent sur votre système IAM et les API qui les connectent

* Tout autre composant local de votre système

Quelle que soit leur source, il est important de reconnaître les imprévus et de s’y préparer. Toutefois, le fait d’ajouter des systèmes d’identité supplémentaires, avec les dépendances et la complexité qui en résultent, est susceptible de réduire la résilience au lieu de l’augmenter.

Pour renforcer la résilience dans vos systèmes, consultez les articles suivants :

* [Intégration de la résilience dans une infrastructure IAM](resilience-in-infrastructure.md)

* [Renforcer la résilience de l’IAM dans vos applications](resilience-app-development-overview.md)

* [Renforcer la résilience de vos systèmes de gestion des accès et des identités clients](resilience-b2c.md)

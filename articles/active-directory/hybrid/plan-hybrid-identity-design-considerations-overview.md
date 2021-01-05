---
title: 'Considérations relatives à la conception d’identités hybrides Azure Active Directory : vue d’ensemble | Microsoft Docs'
description: Vue d'ensemble et plan du contenu du guide des considérations relatives à la conception des identités hybrides
documentationcenter: ''
services: active-directory
author: billmath
manager: daveba
editor: ''
ms.assetid: 100509c4-0b83-4207-90c8-549ba8372cf7
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/30/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: e7f8dd49f3668b8f68753681123a04d21edac46c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95997730"
---
# <a name="azure-active-directory-hybrid-identity-design-considerations"></a>Considérations relatives à la conception d'identités hybrides Azure Active Directory
Les périphériques grand public se multiplient dans le monde de l'entreprise et les applications cloud software-as-a-service (SaaS) sont faciles à adopter. Par conséquent, le maintien du contrôle d'accès des utilisateurs aux applications sur les plateformes cloud et les centres de données internes est difficile.  

Les solutions d'identité de Microsoft regroupent des fonctionnalités, locales et cloud, de création d'une identité d'utilisateur unique pour l'authentification et l'autorisation d'accès à toutes les ressources, indépendamment de l'emplacement. Ce concept est appelé Identité hybride. Il existe différentes possibilités de conception et de configuration de l'identité hybride avec les solutions Microsoft et, dans certains cas, il peut être difficile de déterminer quelle est la combinaison qui répond le mieux aux besoins de votre organisation. 

Ce guide des considérations relatives à la conception des identités hybrides vous aidera à comprendre comment concevoir la solution d'identité hybride idéale en fonction des besoins métier et technologiques de votre organisation.  Ce guide décrit en détail une série d’étapes et de tâches que vous pouvez suivre pour concevoir une solution d’identité hybride qui réponde aux exigences spécifiques de votre organisation. Au fil des étapes et des tâches, le guide présente les technologies et les options appropriées à la disposition des entreprises pour satisfaire aux exigences de niveau de qualité fonctionnelle et de qualité de service (par exemple de disponibilité, d'évolutivité, de performances, de facilité de gestion et de sécurité). 

Plus précisément, les objectifs du guide des considérations relatives à la conception des identités hybrides sont de répondre aux questions suivantes : 

* Quelles sont les questions à poser et les réponses à obtenir pour mettre en place une conception d'identité hybride spécifique pour un domaine technologique ou une catégorie de problème qui réponde à mes exigences ?
* Quelle séquence d'activités dois-je effectuer pour concevoir une solution d'identité hybride pour ce domaine technologique ou cette catégorie de problème ? 
* Quelles options de technologie et de configuration d'identité hybride sont disponibles pour m'aider à répondre à mes exigences ? Quelles sont les compromis entre ces options afin que je puisse sélectionner la meilleure option pour mon entreprise ?

## <a name="who-is-this-guide-intended-for"></a>À qui ce guide est-il destiné ?
 Directeurs informatiques, directeurs des systèmes d’information, architectes d’entreprise et responsables de l’architecture informatique chargés de concevoir une solution d’identité hybride pour les organisations de taille moyenne ou grande.

## <a name="how-can-this-guide-help-you"></a>En quoi ce guide peut-il vous aider ?
Vous pouvez utiliser ce guide pour apprendre à concevoir une solution d’identité hybride en mesure d’intégrer un système cloud de gestion des identités à votre solution d’identité locale actuelle. 

Le schéma suivant illustre un exemple de solution d'identité hybride permettant aux administrateurs informatiques d'intégrer leur solution actuelle Windows Server Active Directory, locale, avec Microsoft Azure Active Directory pour permettre aux utilisateurs d'utiliser l'authentification unique sur l'ensemble des applications cloud et locales.

![Exemple](media/plan-hybrid-identity-design-considerations/hybridID-example.png)

L’illustration ci-dessus est un exemple de solution d’identité hybride qui exploite les services cloud et les intègre aux fonctionnalités locales afin de garantir une expérience unique pour le processus d’authentification de l’utilisateur final et de faciliter la gestion de ces ressources par le service informatique. Bien que cet exemple puisse être courant, le modèle d’identité hybride de chaque organisation est probablement différent de l’exemple illustré dans la Figure 1 en raison de la diversité des exigences. 

Ce guide propose une série d'étapes et de tâches que vous pouvez suivre pour concevoir une solution d'identité hybride qui réponde aux exigences spécifiques de votre organisation. Au fil des étapes et des tâches suivantes, le guide présente les technologies et les options appropriées à votre disposition pour satisfaire aux exigences de niveau de qualité fonctionnelle et de qualité de service de votre organisation.

**Hypothèses** : Vous avez un peu d’expérience avec Windows Server, Active Directory Domain Services et Azure Active Directory. Dans ce document, on suppose que vous cherchez à savoir en quoi ces solutions peuvent répondre aux besoins de votre entreprise de façon autonome ou dans une solution intégrée.

## <a name="design-considerations-overview"></a>Présentation des considérations relatives à la conception
Ce document propose un ensemble d'étapes et de tâches que vous pouvez suivre pour concevoir la solution d'identité hybride idéale en fonction de vos exigences. Les étapes sont présentées dans un ordre précis. Cependant, les considérations relatives à la conception que vous découvrirez dans les étapes suivantes pourront vous amener à revenir sur les décisions que vous aurez prises dans les étapes précédentes, en raison de conflits sur les choix de conception. Tout est mis en œuvre pour vous avertir des risques de conflits de conception tout au long du document. 

Vous n'atteindrez la conception idéale en fonction de vos exigences qu'après avoir répété les étapes autant de fois que nécessaire pour intégrer toutes les considérations présentes dans le document. 

| Phase d'identité hybride | Liste des rubriques |
| --- | --- |
| Déterminer les exigences en matière d'identité |[Déterminer les besoins métier](plan-hybrid-identity-design-considerations-business-needs.md)<br> [Déterminer les exigences de synchronisation de répertoire](plan-hybrid-identity-design-considerations-directory-sync-requirements.md)<br> [Déterminer les exigences d’authentification multifacteur](plan-hybrid-identity-design-considerations-multifactor-auth-requirements.md)<br> [Définir une stratégie d'adoption des identités hybrides](plan-hybrid-identity-design-considerations-identity-adoption-strategy.md) |
| Planifier l'amélioration de la sécurité des données grâce à une solution d'identité solide |[Déterminer les exigences de protection des données](plan-hybrid-identity-design-considerations-dataprotection-requirements.md) <br> [Déterminer les exigences de gestion de contenu](plan-hybrid-identity-design-considerations-contentmgt-requirements.md)<br> [Déterminer les exigences de contrôle d’accès](plan-hybrid-identity-design-considerations-accesscontrol-requirements.md)<br> [Déterminer les exigences de réponse aux incidents](plan-hybrid-identity-design-considerations-incident-response-requirements.md) <br> [Définir les options de protection des données](plan-hybrid-identity-design-considerations-data-protection-strategy.md) |
| Planifier le cycle de vie des identités hybrides |[Déterminer les tâches de gestion des identités hybrides](plan-hybrid-identity-design-considerations-hybrid-id-management-tasks.md) <br> [Gestion de la synchronisation](plan-hybrid-identity-design-considerations-hybrid-id-management-tasks.md)<br> [Déterminer la stratégie d'adoption de la gestion des identités hybrides](plan-hybrid-identity-design-considerations-lifecycle-adoption-strategy.md) |

## <a name="next-steps"></a>Étapes suivantes
[Déterminer les exigences en matière d’identité](plan-hybrid-identity-design-considerations-business-needs.md)


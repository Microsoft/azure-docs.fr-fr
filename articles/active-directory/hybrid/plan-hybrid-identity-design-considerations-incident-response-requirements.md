---
title: 'Conception d’identités hybrides : exigences en matière de réponse aux incidents Azure | Microsoft Docs'
description: Déterminer les fonctionnalités de surveillance et de création de rapports pour la solution d'identité hybride pouvant être exploitées par les services informatiques pour prendre des mesures permettant d'identifier et d'atténuer les menaces.
documentationcenter: ''
services: active-directory
author: billmath
manager: daveba
editor: ''
ms.assetid: a3d2a459-599b-4b67-8e51-7369ee25082d
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/18/2017
ms.subservice: hybrid
ms.author: billmath
ms.custom: seohack1
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8c68be4e46693621f77cb25dd8f34f8e0b7d75dd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89278409"
---
# <a name="determine-incident-response-requirements-for-your-hybrid-identity-solution"></a>Déterminer les exigences de réponse aux incidents pour votre solution d'identité hybride
Les organisations de taille moyenne ou grande auront probablement mis en place une [réponse aux incidents de sécurité](/previous-versions/tn-archive/cc700825(v=technet.10)) pour aider les services informatiques à agir en fonction du niveau de l'incident. Le système de gestion des identités est un composant important du processus de réponse aux incidents. Il peut en effet être utilisé pour aider à identifier l'auteur d’une action spécifique dirigée contre la cible. La solution d'identité hybride doit être en mesure de fournir des capacités de surveillance et de création de rapports pouvant être exploitées par les services informatiques pour prendre des mesures permettant d'identifier et d'atténuer la menace. Un plan de réponse aux incidents typique devra comprendre les phases suivantes :

1. Évaluation initiale.
2. Communication de l'incident.
3. Contrôle des dommages et réduction des risques.
4. Identification de la compromission et de la gravité.
5. Conservation des preuves.
6. Notification aux parties appropriées.
7. Récupération du système.
8. Documentation.
9. Évaluation des dommages et des coûts.
10. Révision des processus et du plan.

Lors de l'identification de ce qui a été compromis et du niveau de gravité, il sera nécessaire d'identifier les systèmes compromis, les fichiers auxquels on aura eu accès et la sensibilité de ces fichiers. Votre système d'identité hybride doit être en mesure de répondre à ces exigences pour vous aider à identifier l'utilisateur qui a effectué ces modifications. 

## <a name="monitoring-and-reporting"></a>Surveillance et création de rapports
Souvent, le système d'identité peut également être une aide pendant la phase d'évaluation initiale, surtout s'il est doté de fonctionnalités d'audit et de création de rapports intégrées. Lors de l'évaluation initiale, tout administrateur informatique doit être en mesure d'identifier une activité suspecte, ou le système doit être en mesure de déclencher automatiquement le système d'identité sur la base d'une tâche préconfigurée. De nombreuses activités peuvent indiquer une attaque possible, mais dans d'autres cas, un système mal configuré peut entraîner plusieurs faux positifs dans un système de détection d'intrusion. 

Le système de gestion d'identité doit aider les administrateurs informatiques à identifier et signaler ces activités suspectes. Il est généralement possible de satisfaire à ces exigences techniques en surveillant tous les systèmes et en disposant d'une fonctionnalité de création de rapports pouvant mettre en évidence les menaces potentielles. Utilisez les questions ci-dessous pour vous aider à concevoir votre solution d'identité hybride en tenant compte des exigences de réponse aux incidents :

* Votre entreprise a-t-elle mis en place une réponse aux incidents de sécurité ?
  * Si oui, le système de gestion d'identité actuel est-il utilisé dans le cadre du processus ?
* Votre entreprise a-t-elle besoin d'identifier des tentatives de connexion suspectes d'utilisateurs sur différents appareils ?
* Votre entreprise a-t-elle besoin de détecter une compromission potentielle d'informations d'identification utilisateur ?
* Votre entreprise a-t-elle besoin d'auditer l'accès et l'action de l'utilisateur ?
* Votre entreprise a-t-elle besoin de savoir à quel moment un utilisateur réinitialise son mot de passe ?

## <a name="policy-enforcement"></a>Application de stratégies
Pendant la phase de réduction des risques et de contrôle des dommages, il est important de réduire rapidement les effets réels et potentiels d'une attaque. La mesure que vous prendrez à ce stade peut faire la différence entre un dommage mineur et majeur. La réponse exacte dépendra de votre organisation et de la nature de l'attaque subie. S'il ressort de l'évaluation initiale qu'un compte a été compromis, vous devrez appliquer une stratégie pour bloquer ce compte. Ce n'est qu'un exemple d'utilisation du système de gestion d'identité. Utilisez les questions ci-dessous pour vous aider à concevoir votre solution d'identité hybride en tenant compte de la manière dont les stratégies seront appliquées pour réagir à un incident en cours :

* Votre entreprise a-t-elle mis en place des stratégies pour empêcher les utilisateurs d'accéder au réseau si nécessaire ?
  * Si oui, la solution actuelle pourra-t-elle être intégrée dans le système de gestion d'identité hybride que vous vous apprêtez à adopter ?
* Votre entreprise a-t-elle besoin d’appliquer l’accès conditionnel pour les utilisateurs qui sont en quarantaine ? 

> [!NOTE]
> Veillez à noter chaque réponse et à comprendre le raisonnement derrière la réponse. [Définir la stratégie de protection des données](plan-hybrid-identity-design-considerations-data-protection-strategy.md) examinera les options disponibles et les avantages/inconvénients de chaque option.  En répondant à chacune de ces questions, vous sélectionnerez l’option correspondant le mieux à vos besoins métier.
> 
> 

## <a name="next-steps"></a>Étapes suivantes
[Définir les options de protection des données](plan-hybrid-identity-design-considerations-data-protection-strategy.md)

## <a name="see-also"></a>Voir aussi
[Présentation des considérations relatives à la conception](plan-hybrid-identity-design-considerations-overview.md)
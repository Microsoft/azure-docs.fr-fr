---
title: Détecter des fuites de mémoire - Détection intelligente Azure Application Insights
description: Surveiller les applications avec Azure Application Insights pour détecter les fuites de mémoire potentielles.
ms.topic: conceptual
ms.date: 12/12/2017
ms.openlocfilehash: 3fe58cd7d61246c5565cd89fa782c8a977f09499
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86539888"
---
# <a name="memory-leak-detection-preview"></a>Détection des fuites de mémoire (version préliminaire)

Application Insights analyse automatiquement la consommation de mémoire des processus de votre application et peut vous avertir des fuites de mémoire potentielles ou d’une consommation accrue de la mémoire.

Cette fonctionnalité ne requiert aucune configuration spéciale, autre que la [configuration des compteurs de performances](./performance-counters.md) pour votre application. Elle est active lorsque votre application génère suffisamment de données de télémétrie sur les compteurs de performances mémoire (par exemple, les octets privés).

## <a name="when-would-i-get-this-type-of-smart-detection-notification"></a>Quand reçoit-on ce type de notification de détection intelligente ?
Une notification classique suit une augmentation constante de la consommation de mémoire sur une période prolongée, pour un ou plusieurs processus et/ou une ou plusieurs machines, qui font partie intégrante de votre application. Des algorithmes Machine Learning sont utilisés pour la détection d’une consommation de mémoire accrue correspondant au modèle d’une fuite de mémoire.

## <a name="does-my-app-really-have-a-problem"></a>Mon application rencontre-t-elle réellement un problème ?
Non, une notification ne signifie pas que votre application rencontre réellement un problème. Bien que les modèles de fuites de mémoire indiquent généralement un problème au niveau de l’application, ces modèles peuvent être liés à votre processus spécifique, ou peuvent avoir une justification naturelle, et peuvent être ignorés.

## <a name="how-do-i-fix-it"></a>Comment la corriger ?
Les notifications incluent des informations de diagnostic qui facilitent le processus d’analyse de diagnostic :
1. **Triage.** La notification vous montre la quantité d’augmentation de mémoire (en Go) et l’intervalle de temps au cours duquel celle-ci s’est produite. Ceci vous permet d’attribuer une priorité au problème.
2. **Portée.** Combien de machines présentent un modèle de fuite de mémoire ? Combien d’exceptions ont été déclenchées au cours de la fuite de mémoire potentielle ? Ces informations peuvent être obtenues dans la notification.
3. **Diagnostic**. La détection contient le modèle de fuite de mémoire, montrant la consommation de mémoire du processus au fil du temps. Pour mieux diagnostiquer le problème, vous pouvez également utiliser les éléments liés et les rapports pointant vers des informations de prise en charge.

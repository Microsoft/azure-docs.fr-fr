---
author: PatrickFarley
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: include
ms.date: 07/17/2019
ms.author: pafarley
ms.openlocfilehash: 07e7cc991f127bf4bb4f466c0108962786e45bce
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018355"
---
Notez le curseur **Probability Threshold** (Seuil de probabilité) dans le volet gauche de l’onglet **Performance**. Il s’agit du niveau de confiance qu’une prédiction doit avoir pour être considérée comme correcte (dans le cadre du calcul de la précision et du rappel). 

Quand vous interprétez des appels de prédiction avec un seuil de probabilité élevé, ils ont tendance à retourner des résultats avec une précision élevée au détriment des rappels ; les classifications détectées sont correctes, mais beaucoup restent non détectées. Un seuil de probabilité faible a l’effet opposé : la plupart des classifications réelles sont détectées, mais il y a plus de faux positifs dans ce jeu. En ayant ceci à l’esprit, vous devez définir le seuil de probabilité en fonction des besoins spécifiques de votre projet. Ultérieurement, lors de la réception des résultats de la prédiction côté client, vous devez utiliser la même valeur de seuil de probabilité que celle utilisée ici.
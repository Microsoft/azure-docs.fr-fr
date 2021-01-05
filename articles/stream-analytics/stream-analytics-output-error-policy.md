---
title: Stratégies d’erreur de sortie dans Azure Stream Analytics
description: Découvrez plus d’informations sur les stratégies de gestion des erreurs de sortie disponibles dans Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 12/04/2018
ms.custom: seodec18
ms.openlocfilehash: 923e8d1ffc3e606c16226b4b2ccbdcfaf4d3e9f1
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93123419"
---
# <a name="azure-stream-analytics-output-error-policy"></a>Stratégies d’erreur de sortie d’Azure Stream Analytics
Cet article décrit les stratégies de gestion des erreurs de sortie qui peuvent être configurées dans Azure Stream Analytics.

Les stratégies de gestion des erreurs de sortie s’appliquent seulement aux erreurs de conversion de données qui se produisent quand l’événement de sortie généré par une tâche Stream Analytics n’est pas conforme au schéma du récepteur cible. Vous pouvez configurer cette stratégie en choisissant **Recommencer** ou **Supprimer**. Dans le portail Azure, quand vous êtes dans une tâche Stream Analytics, sous **Configurer** , sélectionnez **Stratégie d’erreur** pour effectuer votre sélection.

![Emplacement des stratégies d’erreur de sortie d’Azure Stream Analytics](./media/stream-analytics-output-error-policy/stream-analytics-error-policy-locate.png)


## <a name="retry"></a>Recommencer
Quand une erreur se produit, Azure Stream Analytics effectue indéfiniment de nouvelles tentatives d’écriture de l’événement jusqu’à la réussite de l’écriture. Il n’existe pas de délai d’expiration pour les nouvelles tentatives. Au final, le traitement de tous les événements suivants est bloqué par l’événement qui fait l’objet des nouvelles tentatives. Cette option est la stratégie de gestion des erreurs de sortie par défaut.

## <a name="drop"></a>Supprimer
Azure Stream Analytics supprime tous les événements de sortie qui entraînent une erreur de conversion de données. Les événements supprimés ne peuvent pas être récupérés pour un retraitement ultérieur.


Toutes les erreurs temporaires (par exemple les erreurs de réseau) font l’objet de nouvelles tentatives, quelle que soit la configuration de la stratégie de gestion des erreurs de sortie.


## <a name="next-steps"></a>Étapes suivantes
[Guide de dépannage pour Azure Stream Analytics](./stream-analytics-troubleshoot-query.md)
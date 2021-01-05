---
title: Obtenir une géo-redondance pour des travaux Azure Stream Analytics
description: Cet article explique comment obtenir une géo-redondance des travaux Azure Stream Analytics au lieu d’un géo-basculement.
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 08/26/2020
ms.openlocfilehash: 44b0394773485f8054eddc94c7d7e9b91baebc6d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88950821"
---
# <a name="achieve-geo-redundancy-for-azure-stream-analytics-jobs"></a>Obtenir une géo-redondance pour des travaux Azure Stream Analytics

Azure Stream Analytics n’assure pas de géo-basculement automatique, mais vous pouvez obtenir une géo-redondance en déployant des travaux Stream Analytics identiques dans plusieurs régions Azure. Chaque travail se connecte à des sources d’entrée et de sortie locales. Votre application doit envoyer des données d’entrée aux deux entrées régionales et rapprocher les deux sorties régionales. Les travaux Stream Analytics sont deux entités distinctes.

Le diagramme suivant illustre un exemple de déploiement de travail Stream Analytics géo-redondant avec une entrée de hub d’événements et une sortie de base de données Azure.

:::image type="content" source="media/geo-redundancy/geo-redundant-jobs.png" alt-text="diagramme des travaux Stream Analytics géo-redondants":::

## <a name="primarysecondary-strategy"></a>Stratégie primaire/secondaire

Votre application doit gérer la région dont la base de données de sortie est considérée comme primaire, et la région dont la base de données de sortie est considérée comme secondaire. En cas de défaillance de la région primaire, l’application bascule vers la base de données secondaire et démarre la lecture des mises à jour à partir de cette base de données. Le mécanisme réel qui permet de réduire les lectures en double dépend de votre application. Vous pouvez simplifier ce processus en écrivant des informations supplémentaires dans la sortie. Par exemple, vous pouvez ajouter un horodateur ou un ID de séquence à chaque sortie pour faire en sorte que le fait d’ignorer les lignes en double soit une opération triviale. Une fois la région primaire restaurée, elle rejoint la base de données secondaire à l’aide d’une mécanique similaire.

Bien que différents types d’entrées et de sorties autorisent différentes options de géo-réplication, nous vous recommandons d’utiliser le modèle décrit dans cet article pour obtenir la géo-redondance, car il offre la flexibilité et le contrôle nécessaires tant pour les producteurs que pour les consommateurs d’événements.

## <a name="next-steps"></a>Étapes suivantes

* [Surveiller et gérer les travaux Azure Stream Analytics avec PowerShell](stream-analytics-monitor-and-manage-jobs-use-powershell.md)
* [Débogage piloté par les données dans Azure Stream Analytics](stream-analytics-job-diagram-with-metrics.md)
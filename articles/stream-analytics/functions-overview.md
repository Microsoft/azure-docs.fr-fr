---
title: Fonctions définies par l’utilisateur dans Azure Stream Analytics
description: Cet article est une vue d’ensemble des fonctions définies par l’utilisateur dans Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 04/07/2020
ms.openlocfilehash: d167c603ada885a1a4917c66bab110e4ce38cab4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82598366"
---
# <a name="user-defined-functions-in-azure-stream-analytics"></a>Fonctions définies par l’utilisateur dans Azure Stream Analytics

Le langage de requête de type SQL dans Azure Stream Analytics facilite la mise en œuvre d’une logique d’analytique en temps réel sur les données de streaming. Stream Analytics offre une flexibilité supplémentaire par le biais de fonctions personnalisées qui sont appelées dans votre requête. L’exemple de code suivant est une fonction définie par l’utilisateur appelée `sampleFunction` qui accepte un paramètre, chaque enregistrement d’entrée que le travail reçoit, et le résultat est écrit dans la sortie en tant que `sampleResult`.

```sql
SELECT 
    UDF.sampleFunction(InputStream) AS sampleResult 
INTO 
    output 
FROM 
    InputStream 
```

## <a name="types-of-functions"></a>Types de fonctions

Azure Stream Analytics prend en charge les quatre types de fonction suivants : 

* Fonctions JavaScript définies par l’utilisateur 
* Agrégats JavaScript définis par l’utilisateur 
* Fonctions C# définies par l’utilisateur (à l’aide de Visual Studio) 
* Azure Machine Learning 

Vous pouvez utiliser ces fonctions pour des scénarios tels que l’évaluation en temps réel à l’aide de modèles Machine Learning, de manipulations de chaînes, de calculs mathématiques complexes, d’encodage et de décodage de données. 

## <a name="limitations"></a>Limites

Les fonctions définies par l’utilisateur sont sans état et la valeur de retour ne peut être qu’une valeur scalaire. Vous ne pouvez pas appeler des points de terminaison REST externes à partir de ces fonctions définies par l’utilisateur, car cela impacterait probablement les performances de votre travail. 

Azure Stream Analytics ne conserve pas un enregistrement de tous les appels de fonctions et des résultats retournés. Pour garantir la répétabilité (par exemple, la réexécution de votre travail à partir d’un horodatage plus ancien reproduit les mêmes résultats), n’utilisez pas de fonctions telles que `Date.GetData()` ou `Math.random()`, car ces fonctions ne retournent pas le même résultat pour chaque appel.  

## <a name="resource-logs"></a>Journaux d’activité de ressources

Toutes les erreurs d’exécution sont considérées comme irrécupérables et remontées par le biais des journaux d’activité et de ressources. Il est recommandé que votre fonction gère toutes les exceptions et erreurs et retourne un résultat valide à votre requête. Cela empêche votre travail de passer à un [état d’échec](job-states.md).  

## <a name="exception-handling"></a>Gestion des exceptions

Toute exception pendant le traitement des données est considérée comme un échec catastrophique lors de l’utilisation de données dans Azure Stream Analytics. Les fonctions définies par l’utilisateur ont un potentiel plus élevé de lever des exceptions et de provoquer l’arrêt du traitement. Pour éviter ce problème, utilisez un bloc *try-catch* en JavaScript ou C# afin d’intercepter les exceptions pendant l’exécution du code. Les exceptions interceptées peuvent être journalisées et traitées sans entraîner de défaillance du système. Il est recommandé de toujours encapsuler votre code personnalisé dans un bloc *try-catch* pour éviter de lever des exceptions inattendues pour le moteur de traitement.

## <a name="next-steps"></a>Étapes suivantes

* [Fonctions JavaScript définies par l’utilisateur dans Azure Stream Analytics](stream-analytics-javascript-user-defined-functions.md)
* [Agrégats JavaScript définis par l’utilisateur Azure Stream Analytics](stream-analytics-javascript-user-defined-aggregates.md)
* [Développer des fonctions .NET Standard définies par l’utilisateur pour des travaux Azure Stream Analytics](stream-analytics-edge-csharp-udf-methods.md)
* [Intégration d’Azure Stream Analytics à Azure Machine Learning](machine-learning-udf.md)

---
title: Gérer les erreurs et les exceptions dans les workflows
description: Apprenez à gérer les erreurs et les exceptions qui surviennent dans les tâches automatisées et les workflows créés à l’aide d’Azure Logic Apps
services: logic-apps
ms.suite: integration
author: dereklee
ms.author: deli
ms.reviewer: klam, estfan, logicappspm
ms.date: 01/11/2020
ms.topic: article
ms.openlocfilehash: d4bff4ee7980002d911426ed46ffef6fc28c43e9
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96920750"
---
# <a name="handle-errors-and-exceptions-in-azure-logic-apps"></a>Gérer les erreurs et les exceptions dans Azure Logic Apps

Le mode de gestion approprié des temps d’arrêt ou des problèmes dus à des dépendances de systèmes peut se révéler être un défi dans une architecture d’intégration. Pour vous aider à créer des intégrations fiables et résilientes qui gèrent correctement les problèmes et les défaillances, Logic Apps fournit une expérience de qualité pour la gestion des erreurs et des exceptions.

<a name="retry-policies"></a>

## <a name="retry-policies"></a>Stratégies de nouvelle tentative

Pour bénéficier de la gestion des erreurs et des exceptions la plus simple, vous pouvez utiliser une *stratégie de nouvelle tentative* dans toute action ou déclencheur prenant en charge cette fonctionnalité (voir par exemple [Action HTTP](../logic-apps/logic-apps-workflow-actions-triggers.md#http-trigger)). Une stratégie de nouvelle tentative spécifie si et comment l’action ou le déclencheur réessaie d’effectuer une requête quand la requête d’origine expire ou échoue, c’est-à-dire toute requête entraînant une réponse 408, 429 ou 5xx. Si aucune autre stratégie de nouvelle tentative n’est utilisée, la stratégie par défaut est utilisée.

Voici les types de stratégie de nouvelle tentative :

| Type | Description |
|------|-------------|
| **Par défaut** | Cette stratégie envoie jusqu’à quatre nouvelles tentatives à intervalles *exponentiellement croissants*, mis à l’échelle toutes les 7,5 secondes, mais limités à une valeur comprise entre 5 et 45 secondes. |
| **Intervalle exponentiel**  | Cette stratégie attend un intervalle aléatoire sélectionné parmi une plage à croissance exponentielle avant d’envoyer la requête suivante. |
| **Intervalle fixe**  | Cette stratégie attend l’intervalle spécifié avant d’envoyer la requête suivante. |
| **Aucun**  | Ne renvoie pas la requête. |
|||

Pour plus d’informations sur les limitations de la stratégie de nouvelle tentative, consultez [Limites et configuration de Logic Apps](../logic-apps/logic-apps-limits-and-config.md#http-limits).

### <a name="change-retry-policy"></a>Changer la stratégie de nouvelle tentative

Pour sélectionner une autre stratégie de nouvelle tentative, effectuez les étapes suivantes :

1. Ouvrez votre application logique dans le Concepteur d’applications logiques.

1. Ouvrez les **Paramètres** d’une action ou d’un déclencheur.

1. Si l’action ou le déclencheur prend en charge les stratégies de nouvelle tentative, sous **Stratégie de nouvelle tentative**, sélectionnez le type souhaité.

Vous pouvez aussi spécifier manuellement la stratégie de nouvelle tentative dans la section `inputs` pour une action ou un déclencheur qui prend en charge les stratégies de nouvelle tentative. Si vous ne spécifiez pas de stratégie de nouvelle tentative, l’action utilise la stratégie par défaut.

```json
"<action-name>": {
   "type": "<action-type>",
   "inputs": {
      "<action-specific-inputs>",
      "retryPolicy": {
         "type": "<retry-policy-type>",
         "interval": "<retry-interval>",
         "count": <retry-attempts>,
         "minimumInterval": "<minimum-interval>",
         "maximumInterval": "<maximum-interval>"
      },
      "<other-action-specific-inputs>"
   },
   "runAfter": {}
}
```

*Obligatoire*

| Valeur | Type | Description |
|-------|------|-------------|
| <*retry-policy-type*> | String | Type de stratégie de nouvelles tentatives à utiliser : `default`, `none`, `fixed` ou `exponential` |
| <*retry-interval*> | String | Intervalle de nouvelle tentative, où la valeur doit être au [format ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations). L’intervalle minimal par défaut est `PT5S` et l’intervalle maximal est `PT1D`. Quand vous utilisez la stratégie d’intervalle exponentiel, vous pouvez spécifier différentes valeurs minimales et maximales. |
| <*retry-attempts*> | Integer | Nombre de nouvelles tentatives, qui doit être compris entre 1 et 90 |
||||

*Facultatif*

| Valeur | Type | Description |
|-------|------|-------------|
| <*minimum-interval*> | String | Pour la stratégie à intervalle exponentiel, il s’agit du plus petit intervalle pour l’intervalle sélectionné de manière aléatoire au [format ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations) |
| <*maximum-interval*> | String | Pour la stratégie à intervalle exponentiel, il s’agit du plus grand intervalle pour l’intervalle sélectionné de manière aléatoire au [format ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations) |
||||

Voici quelques informations supplémentaires sur les différents types de stratégies.

<a name="default-retry"></a>

### <a name="default"></a>Default

Si vous ne spécifiez pas de stratégie de nouvelle tentative, l’action utilise la stratégie par défaut, qui est en fait une [stratégie d’intervalle exponentiel](#exponential-interval) qui envoie jusqu’à quatre nouvelles tentatives à des intervalles à croissance exponentielle mis à l’échelle toutes les 7,5 secondes. L’intervalle est limité entre 5 et 45 secondes.

Bien que cela ne soit pas défini explicitement dans votre action ou déclencheur, voici comment se comporte la stratégie par défaut dans un exemple d’action HTTP :

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "GET",
      "uri": "http://myAPIendpoint/api/action",
      "retryPolicy" : {
         "type": "exponential",
         "interval": "PT7S",
         "count": 4,
         "minimumInterval": "PT5S",
         "maximumInterval": "PT1H"
      }
   },
   "runAfter": {}
}
```

### <a name="none"></a>None

Pour spécifier que l’action ou le déclencheur n’effectue pas de nouvelle tentative en cas d’échec de requête, affectez la valeur `none` à <*retry-policy-type*>.

### <a name="fixed-interval"></a>Intervalle fixe

Pour spécifier que l’action ou le déclencheur attend l’intervalle spécifié avant d’envoyer la requête suivante, affectez la valeur `fixed` à <*retry-policy-type*>.

*Exemple*

Cette stratégie de nouvelle tentative tente d’obtenir les dernières actualités deux fois de plus après l’échec de la première requête, avec un délai de 30 secondes entre chaque tentative :

```json
"Get_latest_news": {
   "type": "Http",
   "inputs": {
      "method": "GET",
      "uri": "https://mynews.example.com/latest",
      "retryPolicy": {
         "type": "fixed",
         "interval": "PT30S",
         "count": 2
      }
   }
}
```

<a name="exponential-interval"></a>

### <a name="exponential-interval"></a>Intervalle exponentiel

Pour spécifier que l’action ou le déclencheur attend un intervalle aléatoire avant d’envoyer la requête suivante, affectez la valeur `exponential` à <*retry-policy-type*>. L’intervalle aléatoire est sélectionné parmi une plage à croissance exponentielle. Si vous le souhaitez, vous pouvez également substituer les intervalles minimaux et maximaux par défaut en spécifiant vos propres intervalles minimaux et maximaux.

**Plage des variables aléatoires**

Ce tableau montre comment Logic Apps génère une variable aléatoire uniforme dans la plage spécifiée pour chaque nouvelle tentative jusque et y compris le nombre de tentatives :

| Nombre de nouvelles tentatives | Intervalle minimal | Intervalle maximal |
|--------------|------------------|------------------|
| 1 | max(0, <*intervalle-minimal*>) | min(interval, <*intervalle-maximal*>) |
| 2 | max(interval, <*intervalle-minimal*>) | min(2 * interval, <*intervalle-maximal*>) |
| 3 | max(2 * interval, <*intervalle-minimal*>) | min(4 * interval, <*intervalle-maximal*>) |
| 4 | max(4 * interval, <*intervalle-minimal*>) | min(8 * interval, <*intervalle-maximal*>) |
| .... | .... | .... |
||||

<a name="control-run-after-behavior"></a>

## <a name="catch-and-handle-failures-by-changing-run-after-behavior"></a>Intercepter et gérer les échecs en modifiant le comportement « exécuter après »

Quand vous ajoutez des actions dans le concepteur d’applications logiques, vous déclarez implicitement l’ordre dans lequel exécuter ces actions. Une fois son exécution terminée, une action est marquée avec un état tel que `Succeeded`, `Failed`, `Skipped` ou `TimedOut`. Dans chaque définition d’action, la propriété `runAfter` spécifie l’action predécesseur qui doit d’abord se terminer ainsi que les états autorisés pour ce prédécesseur avant que l’action suivante puisse s’exécuter. Par défaut, une action que vous ajoutez dans le concepteur s’exécute uniquement quand le prédécesseur se termine avec l’état `Succeeded`.

Quand une action lève une erreur ou une exception non gérée, elle est marquée `Failed` et toute action successeur est marquée comme `Skipped`. Si ce comportement se produit pour une action qui a des branches parallèles, le moteur Logic Apps suit les autres branches pour déterminer leurs états d’achèvement. Par exemple, si une branche se termine par une action `Skipped`, l’état d’achèvement de cette branche est basé sur l’état du prédécesseur de cette action ignorée. Une fois l’exécution de l’application logique terminée, le moteur détermine l’état de l’exécution complète en évaluant tous les états des branches. Si une branche se termine par un échec, la totalité de l’exécution de l’application logique est marquée `Failed`.

![Exemples qui montrent comment les états d’exécution sont évalués](./media/logic-apps-exception-handling/status-evaluation-for-parallel-branches.png)

Pour vous assurer qu’une action peut toujours être exécutée malgré l’état de son prédécesseur, [personnalisez le comportement « exécuter après » d’une action](#customize-run-after) afin de gérer les états d’échec du prédécesseur.

<a name="customize-run-after"></a>

### <a name="customize-run-after-behavior"></a>Personnaliser le comportement « exécuter après »

Vous pouvez personnaliser le comportement « exécuter après » d’une action afin qu’elle s’exécute quand l’état du prédécesseur est `Succeeded`, `Failed`, `Skipped` ou `TimedOut`. Par exemple, pour envoyer un e-mail une fois que l’action prédécesseur Excel Online `Add_a_row_into_a_table` est marquée `Failed`, plutôt que `Succeeded`, modifiez le comportement « exécuter après » en effectuant l’une des étapes suivantes :

* En mode Conception, sélectionnez le bouton des points de suspension ( **...** ), puis **Configurer la propriété Exécuter après**.

  ![Configurer le comportement « exécuter après » pour une action](./media/logic-apps-exception-handling/configure-run-after-property-setting.png)

  La forme de l’action indique l’état par défaut nécessaire pour l’action prédécesseur, qui est **Ajouter une ligne à un tableau** dans cet exemple :

  ![Comportement « exécuter après » par défaut pour une action](./media/logic-apps-exception-handling/change-run-after-property-status.png)

  Remplacez le comportement « exécuter après » par l’état de votre choix, qui est **a échoué** dans cet exemple :

  ![Remplacer le comportement « exécuter après » par « a échoué »](./media/logic-apps-exception-handling/run-after-property-status-set-to-failed.png)

  Pour spécifier que l’action s’exécute si l’action prédécesseur est marquée comme `Failed`, `Skipped` ou `TimedOut`, sélectionnez les autres états :

  ![Remplacer le comportement « exécuter après » par un autre état](./media/logic-apps-exception-handling/run-after-property-multiple-statuses.png)

* En mode Code, dans la définition JSON de l’action, modifiez la propriété `runAfter`, qui suit la syntaxe suivante :

  ```json
  "<action-name>": {
     "inputs": {
        "<action-specific-inputs>"
     },
     "runAfter": {
        "<preceding-action>": [
           "Succeeded"
        ]
     },
     "type": "<action-type>"
  }
  ```

  Pour cet exemple, remplacez la valeur de la propriété `runAfter` `Succeeded` par `Failed` :

  ```json
  "Send_an_email_(V2)": {
     "inputs": {
        "body": {
           "Body": "<p>Failed to&nbsp;add row to &nbsp;@{body('Add_a_row_into_a_table')?['Terms']}</p>",,
           "Subject": "Add row to table failed: @{body('Add_a_row_into_a_table')?['Terms']}",
           "To": "Sophia.Owen@fabrikam.com"
        },
        "host": {
           "connection": {
              "name": "@parameters('$connections')['office365']['connectionId']"
           }
        },
        "method": "post",
        "path": "/v2/Mail"
     },
     "runAfter": {
        "Add_a_row_into_a_table": [
           "Failed"
        ]
     },
     "type": "ApiConnection"
  }
  ```

  Pour spécifier que l’action s’exécute si l’action prédécesseur est marquée comme `Failed`, `Skipped` ou `TimedOut`, ajoutez les autres états :

  ```json
  "runAfter": {
     "Add_a_row_into_a_table": [
        "Failed", "Skipped", "TimedOut"
     ]
  },
  ```

<a name="scopes"></a>

## <a name="evaluate-actions-with-scopes-and-their-results"></a>Évaluer les actions avec des étendues et leurs résultats

Comme pour l’exécution d’étapes après des actions individuelles avec la propriété `runAfter`, vous pouvez regrouper des actions dans une [étendue](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md). Vous pouvez utiliser des étendues lorsque vous souhaitez regrouper des actions de manière logique, évaluer l’état d’agrégation de l’étendue et effectuer des actions en fonction de cet état. Une fois que toutes les actions d’une étendue ont été exécutées, l’étendue récupère son propre état.

Pour vérifier l’état d’une étendue, vous pouvez utiliser les mêmes critères que ceux utilisés pour vérifier l’état d’exécution d’une application logique, tels que `Succeeded`, `Failed`, et ainsi de suite.

Par défaut, quand toutes les actions de l’étendue réussissent, l’état de l’étendue est marqué `Succeeded`. Si la dernière action d’une étendue se termine avec l’état `Failed` ou `Aborted`, l’état de l’étendue est marqué `Failed`.

Pour intercepter des exceptions dans une étendue `Failed` et exécuter des actions qui gèrent ces erreurs, vous pouvez utiliser la propriété `runAfter` de cette étendue `Failed`. Ainsi, si *n’importe quelle* action figurant dans l’étendue échoue, et que vous utilisez la propriété `runAfter` pour cette étendue, vous pouvez créer une seule action pour intercepter des échecs.

Pour les limites sur les étendues, consultez [Limites et configuration](../logic-apps/logic-apps-limits-and-config.md).

<a name="get-results-from-failures"></a>

### <a name="get-context-and-results-for-failures"></a>Obtenir le contexte et les résultats des échecs

Bien que l’interception des échecs d’une étendue soit très utile, vous aurez peut-être également besoin du contexte pour identifier précisément les actions qui ont échoué, ainsi que les codes d’erreur ou d’état renvoyés.

La fonction [`result()`](../logic-apps/workflow-definition-language-functions-reference.md#result) fournit le contexte des résultats de toutes les actions dans une étendue. La fonction `result()` accepte un seul paramètre, qui est le nom de l’étendue, et retourne un tableau contenant tous les résultats des actions dans cette étendue. Ces objets d’action incluent les mêmes attributs que l’objet `actions()`, comme l’heure de début, l’heure de fin, l’état, les entrées, les ID de corrélation et les sorties de l’action. Pour envoyer le contexte pour les actions qui ont échoué dans une étendue, vous pouvez facilement associer une expression `@result()` à la propriété `runAfter`.

Pour exécuter une action pour chaque action dans une étendue dont le résultat est `Failed` et pour filtrer le tableau de résultats sur les actions ayant échoué, vous pouvez associer une expression `@result()` à une action [**Filtrer le tableau**](logic-apps-perform-data-operations.md#filter-array-action) et à une boucle [**For each**](../logic-apps/logic-apps-control-flow-loops.md). Vous pouvez prendre le tableau des résultats filtrés et effectuer une action pour chaque échec à l’aide de la boucle `For_each`.

Cet exemple, suivi d’une explication détaillée, envoie une requête HTTP POST avec le corps de réponse pour toutes les actions qui ont échoué dans l’étendue « My_Scope » :

```json
"Filter_array": {
   "type": "Query",
   "inputs": {
      "from": "@result('My_Scope')",
      "where": "@equals(item()['status'], 'Failed')"
   },
   "runAfter": {
      "My_Scope": [
         "Failed"
      ]
    }
},
"For_each": {
   "type": "foreach",
   "actions": {
      "Log_exception": {
         "type": "Http",
         "inputs": {
            "method": "POST",
            "body": "@item()['outputs']['body']",
            "headers": {
               "x-failed-action-name": "@item()['name']",
               "x-failed-tracking-id": "@item()['clientTrackingId']"
            },
            "uri": "http://requestb.in/"
         },
         "runAfter": {}
      }
   },
   "foreach": "@body('Filter_array')",
   "runAfter": {
      "Filter_array": [
         "Succeeded"
      ]
   }
}
```

Voici la procédure détaillée pour décrire ce qui se produit dans cet exemple :

1. Pour obtenir le résultat de toutes les actions contenues dans « My_Scope », l’action **Filter Array** utilise cette expression de filtre : `@result('My_Scope')`

1. La condition de l’action **Filtrer le tableau** est tout élément `@result()` dont l’état est égal à `Failed`. Cette condition filtre le tableau ayant tous les résultats d’action de « My_Scope » afin d’obtenir un tableau contenant uniquement les résultats d’action ayant échoué.

1. Exécution d’une action en boucle `For_each` sur les résultats du *tableau filtré*. Cette étape exécute une action pour chaque résultat d’action ayant échoué filtré précédemment.

   Si une action dans l’étendue échoue, les actions de la boucle `For_each` s’exécutent une seule fois. Plusieurs actions ayant échoué peuvent provoquer une action par échec.

1. Envoi d’une requête HTTP POST sur le corps de réponse de l’élément `For_each`, qui est l’expression `@item()['outputs']['body']`.

   La forme de l’élément `@result()` est identique à la forme `@actions()` et peut être analysée de la même façon.

1. Deux en-têtes personnalisés avec le nom de l’action qui a échoué (`@item()['name']`) sont également inclus, ainsi que l’ID de suivi du client d’exécution qui a échoué (`@item()['clientTrackingId']`).

Pour référence, voici un exemple d’un seul élément `@result()`, montrant le `name`, le `body`, et les propriétés `clientTrackingId` analysés dans l’exemple précédent. En dehors d’une action `For_each`, `@result()` retourne un tableau de ces objets.

```json
{
   "name": "Example_Action_That_Failed",
   "inputs": {
      "uri": "https://myfailedaction.azurewebsites.net",
      "method": "POST"
   },
   "outputs": {
      "statusCode": 404,
      "headers": {
         "Date": "Thu, 11 Aug 2016 03:18:18 GMT",
         "Server": "Microsoft-IIS/8.0",
         "X-Powered-By": "ASP.NET",
         "Content-Length": "68",
         "Content-Type": "application/json"
      },
      "body": {
         "code": "ResourceNotFound",
         "message": "/docs/folder-name/resource-name does not exist"
      }
   },
   "startTime": "2016-08-11T03:18:19.7755341Z",
   "endTime": "2016-08-11T03:18:20.2598835Z",
   "trackingId": "bdd82e28-ba2c-4160-a700-e3a8f1a38e22",
   "clientTrackingId": "08587307213861835591296330354",
   "code": "NotFound",
   "status": "Failed"
}
```

Vous pouvez utiliser les expressions décrites plus haut dans cet article pour exécuter différents modèles de gestion des exceptions. Vous pouvez choisir d’exécuter une seule action de gestion des exceptions en dehors de l’étendue qui accepte l’intégralité du tableau filtré d’échecs, et de supprimer l’action `For_each`. Vous pouvez également inclure d’autres propriétés utiles à partir de la réponse `\@result()` comme décrit précédemment.

## <a name="set-up-azure-monitor-logs"></a>Configurer les journaux Azure Monitor

Les précédents modèles sont très utiles pour gérer les erreurs et les exceptions d’une exécution, mais vous pouvez également identifier les erreurs et y répondre indépendamment de l’exécution elle-même. [Azure Monitor](../azure-monitor/overview.md) vous permet d'envoyer facilement tous les événements du workflow, y compris les états d'exécution et d'action, à un [espace de travail Log Analytics](../azure-monitor/platform/data-platform-logs.md), à un [compte de stockage Azure](../storage/blobs/storage-blobs-overview.md) ou à [Azure Event Hubs](../event-hubs/event-hubs-about.md).

Vous pouvez surveiller les journaux d’activité et les mesures ou les publier dans n’importe quel outil de supervision de votre choix pour évaluer les états d’exécution. Vous avez également la possibilité de transmettre tous les événements via Event Hubs à [Azure Stream Analytics](https://azure.microsoft.com/services/stream-analytics/). Dans Stream Analytics, vous pouvez écrire des requêtes actives basées sur des anomalies, moyennes ou échecs dans les journaux de diagnostic. Vous pouvez utiliser Stream Analytics pour envoyer des informations à d’autres sources de données, telles que des files d’attente, des rubriques, SQL, Azure Cosmos DB ou Power BI.

## <a name="next-steps"></a>Étapes suivantes

* [Voir comment un client conçoit la gestion des erreurs avec Azure Logic Apps](../logic-apps/logic-apps-scenario-error-and-exception-handling.md)
* [Consultez d’autres exemples et scénarios Logic Apps](../logic-apps/logic-apps-examples-and-scenarios.md)

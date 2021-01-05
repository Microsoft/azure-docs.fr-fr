---
title: Utilisation de fonctions dans les requêtes de journal Azure Monitor | Microsoft Docs
description: Cet article décrit l’utilisation des fonctions pour appeler une requête à partir d’une autre requête de journal dans Azure Monitor.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/31/2020
ms.openlocfilehash: 678a6f0dc19d966f3d15e713008c19c8fbb96f5e
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024362"
---
# <a name="using-functions-in-azure-monitor-log-queries"></a>Utilisation de fonctions dans les requêtes de journal Azure Monitor

Pour utiliser une requête de journal avec une autre requête, vous pouvez l’enregistrer en tant que fonction. Cela vous permet de simplifier les requêtes complexes en les divisant en plusieurs parties et de réutiliser le code commun avec plusieurs requêtes.

## <a name="create-a-function"></a>Créer une fonction

Pour créer une fonction avec Log Analytics sur le portail Azure, cliquez sur **Enregistrer**, puis fournissez les informations indiquées dans le tableau suivant.

| Paramètre | Description |
|:---|:---|
| Nom           | Nom d’affichage de la requête dans l’**Explorateur de requêtes**. |
| Enregistrer sous        | Fonction |
| Alias de fonction | Nom court pour utiliser la fonction dans d’autres requêtes. Ne peut pas contenir d’espaces et doit être unique. |
| Category       | Catégorie pour organiser les fonctions et les requêtes enregistrées dans l’**Explorateur de requêtes**. |




## <a name="use-a-function"></a>Utiliser une fonction
Utilisez une fonction en incluant son alias dans une autre requête. Elle peut être utilisée comme toute autre table.

## <a name="function-parameters"></a>Paramètres de fonction 
Vous pouvez ajouter des paramètres à une fonction pour pouvoir fournir des valeurs pour certaines variables lors de son appel. Actuellement, la seule façon de créer une fonction avec des paramètres consiste à utiliser un modèle Resource Manager. Pour un exemple, consultez [Exemples de modèle Resource Manager pour les requêtes de journal dans Azure Monitor](../samples/resource-manager-log-queries.md#parameterized-function).

## <a name="example"></a>Exemple
L’exemple de requête suivante retourne toutes les mises à jour de sécurité manquantes signalées le dernier jour. Enregistrez cette requête en tant que fonction avec l’alias _security_updates_last_day_. 

```Kusto
Update
| where TimeGenerated > ago(1d) 
| where Classification == "Security Updates" 
| where UpdateState == "Needed"
```

Créez une autre requête et référencez la fonction _security_updates_last_day_ pour rechercher les mises à jour de sécurité nécessaires concernant SQL.

```Kusto
security_updates_last_day | where Title contains "SQL"
```

## <a name="next-steps"></a>Étapes suivantes
Reportez-vous à d'autres leçons pour écrire des requêtes de journal Azure Monitor :

- [Opérations de chaîne](/azure/data-explorer/kusto/query/samples?&pivots=azuremonitor#string-operations)
- [Opérations de date et d’heure](/azure/data-explorer/kusto/query/samples?&pivots=azuremonitor#date-and-time-operations)
- [Fonctions d’agrégation](/azure/data-explorer/kusto/query/samples?&pivots=azuremonitor#aggregations)
- [Agrégations avancées](/azure/data-explorer/write-queries#advanced-aggregations)
- [JSON et structures de données](/azure/data-explorer/kusto/query/samples?&pivots=azuremonitor#json-and-data-structures)
- [Joins](/azure/data-explorer/kusto/query/samples?&pivots=azuremonitor#joins)
- [Graphiques](/azure/data-explorer/kusto/query/samples?&pivots=azuremonitor#charts)

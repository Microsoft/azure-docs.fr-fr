---
title: Sortie de stockage de table à partir d’Azure Stream Analytics
description: Cet article décrit le stockage de table Azure comme sortie pour Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: ea775ef472687485dbd2f30c4f60adc33c0eaa73
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93127346"
---
# <a name="table-storage-output-from-azure-stream-analytics"></a>Sortie de stockage de table à partir d’Azure Stream Analytics

[stockage de tables Azure](../storage/common/storage-introduction.md) offre un stockage hautement disponible et massivement évolutif, afin qu'une application puisse être mise à l'échelle automatiquement pour répondre à la demande des utilisateurs. Le stockage de tables correspond au magasin de clés/d'attributs NoSQL de Microsoft, que vous pouvez utiliser pour les données structurées, avec moins de contraintes au niveau du schéma. Le stockage des données sur les tables Azure permet d’assurer leur persistance et une récupération efficace.

Le tableau suivant répertorie les noms de propriétés et leur description pour la création d’une sortie de table.

| Nom de la propriété | Description |
| --- | --- |
| Alias de sortie |Nom convivial utilisé dans les requêtes pour diriger la sortie de requête vers ce stockage de tables. |
| Compte de stockage |Nom du compte de stockage où vous envoyez votre sortie. |
| Clé du compte de stockage |Clé d’accès associée au compte de stockage. |
| Nom de la table |Nom de la table. La table est créée si elle n’existe pas. |
| Clé de partition |Nom de la colonne de sortie contenant la clé de partition. La clé de partition est un identificateur unique pour la partition dans une table qui constitue la première partie de la clé primaire d’une entité. C’est une valeur de chaîne qui peut atteindre 1 Ko. |
| Clé de ligne |Nom de la colonne de sortie contenant la clé de ligne. La clé de ligne est un identificateur unique pour une entité dans une partition. Elle constitue la deuxième partie de la clé primaire d'une entité. La clé de ligne est une valeur de chaîne qui peut atteindre 1 Ko. |
| Taille du lot |Nombre d’enregistrements d’une opération par lot. La valeur par défaut (100) est suffisante pour la plupart des travaux. Pour plus d’informations sur la modification de ce paramètre, consultez la spécification [TableBatchOperation Class](/java/api/com.microsoft.azure.storage.table.tablebatchoperation) (Classe TableBatchOperation). |

## <a name="partitioning"></a>Partitionnement

La clé de partition est une colonne de sortie. Le nombre d’enregistreurs de sortie suit le partitionnement d’entrée de [requêtes entièrement mises en parallèle ](stream-analytics-scale-jobs.md).

## <a name="output-batch-size"></a>Taille de lot de sortie

Pour obtenir la taille maximale du message, consultez [Limites de stockage Azure](../azure-resource-manager/management/azure-subscription-service-limits.md#storage-limits). La valeur par défaut est 100 entités par transaction, mais vous pouvez la configurer sur une valeur inférieure si besoin.

## <a name="next-steps"></a>Étapes suivantes

* [Démarrage rapide : Créer un travail Stream Analytics à l’aide du portail Azure](stream-analytics-quick-create-portal.md)
* [Démarrage rapide : Créer un travail Azure Stream Analytics avec l’interface de ligne de commande Azure](quick-create-azure-cli.md)
* [Démarrage rapide : Créer une tâche Azure Stream Analytics à l’aide d’un modèle Resource Manager](quick-create-azure-resource-manager.md)
* [Démarrage rapide : Créer un travail Stream Analytics à l’aide d’Azure PowerShell](stream-analytics-quick-create-powershell.md)
* [Démarrage rapide : Créer une tâche Azure Stream Analytics à l’aide de Visual Studio](stream-analytics-quick-create-vs.md)
* [Démarrage rapide : Créer une tâche Azure Stream Analytics dans Visual Studio Code](quick-create-visual-studio-code.md)
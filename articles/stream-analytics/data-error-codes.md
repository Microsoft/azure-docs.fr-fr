---
title: Codes d'erreur de données - Azure Stream Analytics
description: Résolvez les problèmes Azure Stream Analytics liés aux codes d’erreur de données.
ms.author: mamccrea
author: mamccrea
ms.topic: troubleshooting
ms.date: 05/07/2020
ms.service: stream-analytics
ms.openlocfilehash: 56d7527eebb91bd09895f6cd0238721574df1015
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86037322"
---
# <a name="azure-stream-analytics-data-error-codes"></a>Codes d'erreur de données Azure Stream Analytics

Vous pouvez utiliser les journaux d’activité et les journaux de ressources pour déboguer les comportements inattendus à partir de votre tâche Azure Stream Analytics. Cet article répertorie la description de chaque code d’erreur de données. Des erreurs de données se produisent lorsque le flux contient des données incorrectes, par exemple un schéma d’enregistrement inattendu.

## <a name="inputdeserializationerror"></a>InputDeserializationError

* **Cause** : Une erreur s’est produite lors de la désérialisation des données d’entrée.

## <a name="inputeventtimestampnotfound"></a>InputEventTimestampNotFound

* **Cause** : Stream Analytics ne parvient pas à obtenir de timestamp pour la ressource. 

## <a name="inputeventtimestampbyovervaluenotfound"></a>InputEventTimestampByOverValueNotFound

* **Cause** : Stream Analytics ne parvient pas à obtenir la valeur de `TIMESTAMP BY OVER COLUMN`.

## <a name="inputeventlatebeyondthreshold"></a>InputEventLateBeyondThreshold

* **Cause** : Un événement d’entrée a été envoyé au-delà de la tolérance configurée.

## <a name="inputeventearlybeyondthreshold"></a>InputEventEarlyBeyondThreshold

* **Cause** : Une heure d'arrivée d'événement d'entrée est antérieure au seuil de timestamp de l'application d'événement d'entrée.

## <a name="azurefunctionmessagesizeexceeded"></a>AzureFunctionMessageSizeExceeded

* **Cause** : La sortie des messages vers Azure Functions dépasse la limite de taille.

## <a name="eventhuboutputrecordexceedssizelimit"></a>EventHubOutputRecordExceedsSizeLimit

* **Cause** : Un enregistrement de sortie dépasse la limite de taille maximale lors de l’écriture dans Event Hub.

## <a name="cosmosdboutputinvalidid"></a>CosmosDBOutputInvalidId

* **Cause** : La valeur ou le type d’une colonne spécifique n’est pas valide.
* **Recommandation** : Fournissez des chaînes non vides uniques de moins de 255 caractères.

## <a name="cosmosdboutputinvalididcharacter"></a>CosmosDBOutputInvalidIdCharacter

* **Cause** : L'ID de document de l'enregistrement de sortie contient un caractère non valide.

## <a name="cosmosdboutputmissingid"></a>CosmosDBOutputMissingId

* **Cause** : L'enregistrement de sortie ne contient pas la colonne \[id] à utiliser comme propriété de clé primaire.

## <a name="cosmosdboutputmissingidcolumn"></a>CosmosDBOutputMissingIdColumn

* **Cause** : L’enregistrement de sortie ne contient pas la propriété d’ID de document. 
* **Recommandation** : Vérifiez que la sortie de la requête contient la colonne avec une chaîne non vide unique de moins de « 255 » caractères.

## <a name="cosmosdboutputmissingpartitionkey"></a>CosmosDBOutputMissingPartitionKey

* **Cause** : L'enregistrement de sortie ne contient pas de colonne à utiliser comme propriété de clé de la partition.

## <a name="cosmosdboutputsinglerecordtoolarge"></a>CosmosDBOutputSingleRecordTooLarge

* **Cause** : Une écriture d'enregistrement individuelle dans Cosmos DB est trop grande.

## <a name="sqldatabaseoutputdataerror"></a>SQLDatabaseOutputDataError

* **Cause** : Stream Analytics ne peut écrire d’événements dans SQL Database en raison de problèmes dans les données.

## <a name="next-steps"></a>Étapes suivantes

* [Troubleshoot input connections](stream-analytics-troubleshoot-input.md) (Résoudre les problèmes liés aux connexions d’entrée)
* [Résoudre les problèmes liés aux sorties Azure Stream Analytics](stream-analytics-troubleshoot-output.md)
* [Résoudre les problèmes liés aux requêtes Azure Stream Analytics](stream-analytics-troubleshoot-query.md)
* [Résoudre les problèmes liés à Azure Stream Analytics à l’aide des journaux de ressources](stream-analytics-job-diagnostic-logs.md)
* [Erreurs de données Azure Stream Analytics](data-errors.md)

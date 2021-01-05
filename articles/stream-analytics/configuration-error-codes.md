---
title: Codes d’erreur de configuration - Azure Stream Analytics
description: Résolvez les problèmes Azure Stream Analytics liés aux codes d’erreur de configuration.
ms.author: mamccrea
author: mamccrea
ms.topic: troubleshooting
ms.date: 05/07/2020
ms.service: stream-analytics
ms.openlocfilehash: 80179506c133de92b56d476c9aa99d55c3e3bbd9
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93305807"
---
# <a name="azure-stream-analytics-configuration-error-codes"></a>Codes d’erreur de configuration Azure Stream Analytics

Vous pouvez utiliser les journaux d’activité et les journaux de ressources pour déboguer les comportements inattendus à partir de votre tâche Azure Stream Analytics. Cet article répertorie la description de chaque code d’erreur de configuration. Les erreurs de configuration sont liées à la configuration de votre tâche ou aux configurations d’entrée et de sortie.

## <a name="eventhubunauthorizedaccess"></a>EventHubUnauthorizedAccess

* **Cause**  : Event Hub a généré une erreur *Accès non autorisé*.

## <a name="eventhubreceiverepochconflict"></a>EventHubReceiverEpochConflict

* **Cause**  : Plusieurs récepteurs Event Hub dotés de valeurs d'époque différentes sont présents.
* **Recommandation**  : Assurez-vous que *Service Bus Explorer* ou une application *EventProcessorHost* n’est pas connecté lors de l’exécution de votre tâche Stream Analytics.

## <a name="eventhubreceiverquotaexceeded"></a>EventHubReceiverQuotaExceeded

* **Cause**  : Stream Analytics ne peut se connecter à une partition car le nombre maximal de destinataires autorisés par partition dans un groupe de consommateurs a été atteint.
* **Recommandation**  : Assurez-vous que les autres tâches Stream Analytics ou Service Bus Explorer n’utilisent pas le même groupe de consommateurs.

## <a name="eventhuboutputthrottled"></a>EventHubOutputThrottled

* **Cause**  : Une erreur s’est produite lors de l’écriture de données dans Event Hub en raison d'une limitation.
* **Recommandation**  : Si cela arrive fréquemment, mettez à niveau le débit.

## <a name="eventhuboutputinvalidconnectionconfig"></a>EventHubOutputInvalidConnectionConfig

* **Cause**  : La configuration de connexion fournie est incorrecte.
* **Recommandation**  : Corrigez la configuration, puis redémarrez la tâche.

## <a name="eventhuboutputinvalidhostname"></a>EventHubOutputInvalidHostname

* **Cause**  : L’hôte Event Hub est inaccessible.
* **Recommandation**  : Vérifiez que le nom d’hôte fourni est correct.

## <a name="eventhuboutputunexpectedpartitioncount"></a>EventHubOutputUnexpectedPartitionCount

* **Cause**  : L’expéditeur EventHub a rencontré un nombre de partitions EventHub inattendu.
* **Recommandation**  : Redémarrez votre tâche Stream Analytics si le nombre de partitions EventHub a changé.

## <a name="cosmosdbpartitionkeynotfound"></a>CosmosDBPartitionKeyNotFound

* **Cause**  : Stream Analytics n'est pas parvenu à trouver la clé de partition d’une collection Cosmos DB spécifique dans la base de données.
* **Recommandation**  : Assurez-vous qu’une clé de partition valide est spécifiée pour la collection dans Cosmos DB.

## <a name="cosmosdbinvalidpartitionkeycolumn"></a>CosmosDBInvalidPartitionKeyColumn

* **Cause**  : Erreur générée lorsqu'une clé de partition ne correspond ni à un nœud terminal ni au niveau supérieur.

## <a name="cosmosdbinvalididcolumn"></a>CosmosDBInvalidIdColumn

* **Cause**  : La sortie de la requête ne peut pas contenir la colonne \[id] si une autre colonne est choisie comme propriété de clé primaire.

## <a name="cosmosdbdatabasenotfound"></a>CosmosDBDatabaseNotFound

* **Cause**  : Stream Analytics ne parvient pas à trouver une base de données CosmosDB.

## <a name="cosmosdbcollectionnotfound"></a>CosmosDBCollectionNotFound

* **Cause**  : Stream Analytics ne parvient pas à trouver une collection Cosmos DB spécifique dans une base de données.

## <a name="cosmosdboutputwritethrottling"></a>CosmosDBOutputWriteThrottling

* **Cause**  : Une erreur s’est produite lors de l’écriture de données en raison d’une limitation de Cosmos DB.
* **Recommandation**  : Mettez à niveau le niveau de performance de la collection et ajustez les performances de votre base de données.

## <a name="sqldatabaseconnectionstringerror"></a>SQLDatabaseConnectionStringError

* **Cause**  : La tâche Stream Analytics a rencontré une erreur d’authentification.
* **Recommandation**  : Assurez-vous que la chaîne de connexion SQL Database est correcte.

## <a name="sqldatabasemanagedidentityauthenticationerror"></a>SQLDatabaseManagedIdentityAuthenticationError

* **Cause**  : La tâche Stream Analytics a rencontré une erreur d’authentification. 
* **Recommandation**  : Assurez-vous que le nom du compte est correctement configuré et que l’identité managée de la tâche peut accéder à SQL Database.

## <a name="sqldatabaseoutputnotableerror"></a>SQLDatabaseOutputNoTableError

* **Cause**  : Stream Analytics ne parvient pas à trouver les informations de schéma d'une table spécifique.

## <a name="sqldwoutputinvalidserviceedition"></a>SQLDWOutputInvalidServiceEdition

* **Cause**  : SQL Database n'est pas pris en charge.
* **Recommandation**  : Utilisez un pool SQL dédié.

## <a name="next-steps"></a>Étapes suivantes

* [Troubleshoot input connections](stream-analytics-troubleshoot-input.md) (Résoudre les problèmes liés aux connexions d’entrée)
* [Résoudre les problèmes liés aux sorties Azure Stream Analytics](stream-analytics-troubleshoot-output.md)
* [Résoudre les problèmes liés aux requêtes Azure Stream Analytics](stream-analytics-troubleshoot-query.md)
* [Résoudre les problèmes liés à Azure Stream Analytics à l’aide des journaux de ressources](stream-analytics-job-diagnostic-logs.md)
* [Erreurs de données Azure Stream Analytics](data-errors.md)

---
title: Codes d’erreur externe - Azure Stream Analytics
description: Résolvez les problèmes Azure Stream Analytics liés aux codes d’erreur externe.
ms.author: mamccrea
author: mamccrea
ms.topic: troubleshooting
ms.date: 05/07/2020
ms.service: stream-analytics
ms.openlocfilehash: 61d6556467bc6f1494e11ce3a01bf5266b740f3f
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93305791"
---
# <a name="azure-stream-analytics-external-error-codes"></a>Codes d’erreur externe Azure Stream Analytics

Vous pouvez utiliser les journaux d’activité et les journaux de ressources pour déboguer les comportements inattendus à partir de votre tâche Azure Stream Analytics. Cet article répertorie la description de chaque code d’erreur externe. Les erreurs externes sont des erreurs génériques déclenchées par un service en amont ou en aval que Stream Analytics ne parvient pas à distinguer en tant qu'erreur de données, erreur de configuration ou erreur de disponibilité externe.

## <a name="adapterinitializationerror"></a>AdapterInitializationError

* **Cause**  : Une erreur s'est produite lors de l'initialisation de l'adaptateur.

## <a name="adapterfailedtowriteevents"></a>AdapterFailedToWriteEvents

* **Cause**  : Une erreur s'est produite lors de l'écriture de données sur un adaptateur.

## <a name="azurefunctionhttperror"></a>AzureFunctionHttpError

* **Cause**  : Une erreur HTTP a été renvoyée par des fonctions Azure.

## <a name="azurefunctionfailedtosendmessage"></a>AzureFunctionFailedToSendMessage

* **Cause**  : Stream Analytics n'est pas parvenu à écrire d'événements dans Azure Function.

## <a name="azurefunctionredirecterror"></a>AzureFunctionRedirectError

* **Cause**  : Une erreur de redirection s’est produire lors de la sortie vers Azure Functions.

## <a name="azurefunctionclienterror"></a>AzureFunctionClientError

* **Cause**  : Une erreur client s’est produite lors de la sortie vers Azure Functions.

## <a name="azurefunctionservererror"></a>AzureFunctionServerError

* **Cause**  : Une erreur de serveur s’est produite lors de la sortie vers Azure Functions.

## <a name="azurefunctionhttptimeouterror"></a>AzureFunctionHttpTimeOutError

* **Cause**  : L'écriture vers Azure Functions a échoué car la requête HTTP a dépassé le délai d’expiration. 
* **Recommandation**  : Vérifiez vos journaux Azure Functions pour de possibles retards.

## <a name="eventhubargumenterror"></a>EventHubArgumentError

* **Cause**  : Les décalages d’entrée ne sont pas valides. Cela peut être dû à un basculement.
* **Recommandation**  : Redémarrez votre tâche Stream Analytics à partir de la dernière heure de sortie.

## <a name="eventhubfailedtowriteevents"></a>EventHubFailedToWriteEvents

* **Cause**  : Une erreur s'est produite lors de l'envoi de données à Event Hub.

## <a name="cosmosdbconnectionfailureaftermaxretries"></a>CosmosDBConnectionFailureAfterMaxRetries

* **Cause**  : Stream Analytics n'est pas parvenu à se connecter à un compte Cosmos DB au terme du nombre maximal de nouvelles tentatives.

## <a name="cosmosdbfailureaftermaxretries"></a>CosmosDBFailureAfterMaxRetries

* **Cause**  : Stream Analytics n'est pas parvenu à interroger la base de données et la collection Cosmos DB au terme du nombre maximal de nouvelles tentatives.

## <a name="cosmosdbfailedtocreatestoredprocedure"></a>CosmosDBFailedToCreateStoredProcedure

* **Cause**  : CosmosDB n'est pas parvenu à créer de procédure stockée au terme de plusieurs tentatives.

## <a name="cosmosdboutputrequesttimeout"></a>CosmosDBOutputRequestTimeout

* **Cause**  : La procédure stockée upsert a renvoyé une erreur. 

## <a name="sqldatabaseoutputinitializationerror"></a>SQLDatabaseOutputInitializationError

* **Cause**  : Stream Analytics ne peut initialiser la sortie SQL Database.

## <a name="sqldatabaseoutputwriteerror"></a>SQLDatabaseOutputWriteError

* **Cause**  : Stream Analytics ne peut écrire d’événements dans la sortie SQL Database.

## <a name="sqldwoutputinitializationerror"></a>SQLDWOutputInitializationError

* **Cause**  : Une erreur s’est produite lors de l’initialisation d’une sortie de pool SQL dédié.

## <a name="sqldwoutputwriteerror"></a>SQLDWOutputWriteError

* **Cause**  : Une erreur s’est produite lors de l’écriture de la sortie dans un pool SQL dédié.

## <a name="next-steps"></a>Étapes suivantes

* [Troubleshoot input connections](stream-analytics-troubleshoot-input.md) (Résoudre les problèmes liés aux connexions d’entrée)
* [Résoudre les problèmes liés aux sorties Azure Stream Analytics](stream-analytics-troubleshoot-output.md)
* [Résoudre les problèmes liés aux requêtes Azure Stream Analytics](stream-analytics-troubleshoot-query.md)
* [Résoudre les problèmes liés à Azure Stream Analytics à l’aide des journaux de ressources](stream-analytics-job-diagnostic-logs.md)
* [Erreurs de données Azure Stream Analytics](data-errors.md)

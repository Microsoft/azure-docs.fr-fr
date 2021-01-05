---
author: normesta
ms.service: storage
ms.topic: include
ms.date: 09/28/2020
ms.author: normesta
ms.openlocfilehash: 2474b8920c5387c7896b413f229c2f5b06cdafb1
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011137"
---
| Propriété | Description |
|:--- |:---|
|**time** | Heure UTC (Universal Time Coordinated) à laquelle la demande a été reçue par le stockage. Par exemple : `2018/11/08 21:09:36.6900118`.|
|**resourceId** | ID de ressource du compte de stockage. Par exemple : `/subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/`<br>`myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/storageAccounts/blobServices/default`|
|**category** | Catégorie de l’opération demandée. Par exemple : `StorageRead`, `StorageWrite` ou `StorageDelete`.|
|**operationName** | Type de l’opération REST qui a été exécutée. <br> Pour obtenir la liste complète des opérations, consultez la rubrique [Opérations journalisées et messages d’état de Storage Analytics](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages). |
|**operationVersion** | Version du service de stockage qui a été spécifiée lorsque la demande a été effectuée. Elle équivaut à la valeur de l’en-tête **x-ms-version**. Par exemple : `2017-04-17`.|
|**schemaVersion** | Version de schéma du journal. Par exemple : `1.0`.|
|**statusCode** | Code d’état HTTP pour la demande. Si la demande est interrompue, cette valeur peut être définie sur `Unknown`. <br> Par exemple : `206` |
|**statusText** | Statut de l’opération demandée.  Pour obtenir la liste complète des messages d’état, consultez la rubrique [Opérations journalisées et messages d’état de Storage Analytics](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages). Dans la version 2017-04-17 et les versions ultérieures, le message d’état `ClientOtherError` n’est pas utilisé. À la place, ce champ contient un code d’erreur. Par exemple : `SASSuccess`  |
|**durationMs** | Durée totale, en millisecondes, pour exécuter l’opération demandée. Cette valeur inclut le temps nécessaire à la lecture de la demande entrante et à l’envoi de la réponse au demandeur. Par exemple : `12`.|
|**callerIpAddress** | Adresse IP du demandeur, numéro de port inclus. Par exemple : `192.100.0.102:4362`. |
|**correlationId** | ID utilisé pour corréler les journaux entre les ressources. Par exemple : `b99ba45e-a01e-0042-4ea6-772bbb000000`. |
|**location** | Emplacement du compte de stockage. Par exemple : `North Europe`. |
|**protocol**|Protocole utilisé dans l’opération. Par exemple : `HTTP`, `HTTPS`, `SMB` ou `NFS`|
| **uri** | URI (Uniform Resource Identifier) demandé. |
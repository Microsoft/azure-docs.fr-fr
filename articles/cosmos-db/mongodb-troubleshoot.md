---
title: Résoudre des erreurs courantes dans l’API Azure Cosmos DB pour MongoDB
description: Ce document détaille les façons de résoudre les problèmes couramment rencontrés dans l’API Azure Cosmos DB pour MongoDB.
author: christopheranderson
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: troubleshooting
ms.date: 07/15/2020
ms.author: chrande
ms.openlocfilehash: 9d76c3d9943300f88a146e82b862624d491cf546
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017812"
---
# <a name="troubleshoot-common-issues-in-azure-cosmos-dbs-api-for-mongodb"></a>Résoudre des problèmes courants dans l’API Azure Cosmos DB pour MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

L’article suivant décrit les erreurs courantes et les solutions relatives aux bases de données utilisant l’API Azure Cosmos DB pour MongoDB.

>[!Note]
> Azure Cosmos DB n’héberge pas le moteur MongoDB. Il fournit une implémentation de la [version 3.6 du protocole filaire](mongodb-feature-support-36.md) de MongoDB et la prise en charge héritée de la [version 3.2 du protocole filaire](mongodb-feature-support.md). Par conséquent, certaines de ces erreurs se trouvent uniquement dans l’API d’Azure Cosmos DB pour MongoDB. 

## <a name="common-errors-and-solutions"></a>Erreurs courantes et solutions

| Error               | Code  | Description  | Solution  |
|---------------------|-------|--------------|-----------|
| ExceededTimeLimit   | 50 | La requête a dépassé le délai d’exécution de 60 secondes. | Cette erreur peut avoir plusieurs causes. L’une d’elles est que la capacité actuelle d’unités de requête allouées n’est pas suffisante pour traiter la requête. Ce problème peut être résolu en augmentant le nombre d’unités de requête de cette collection ou base de données. Dans d’autres cas, il est possible de contourner cette erreur en fractionnant une requête volumineuse en plusieurs requêtes plus petites. |
| TooManyRequests     | 16500 | Le nombre total d’unités de requête consommées est supérieur au taux d’unités de requête provisionné pour la collection et a été limité. | Adaptez le débit assigné à un conteneur ou à un ensemble de conteneurs à partir du portail Azure ou renouvelez l’opération. |
| ExceededMemoryLimit | 16501 | En tant que service multi-locataire, l’opération a dépassé l’allocation de mémoire du client. | Réduisez l’étendue de l’opération en fixant un critère de requête plus restrictif, ou contactez le support technique via le [portail Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade). Exemple : `db.getCollection('users').aggregate([{$match: {name: "Andy"}}, {$sort: {age: -1}}]))` |
| Le chemin de l’index correspondant à l’élément order-by spécifié est exclu / La requête order by n’a pas d’index composite correspondant à partir duquel elle peut être traitée. | 2 | La requête demande un tri sur un champ qui n’est pas indexé. | Créez un index correspondant (ou un index composite) pour la requête de tri tentée. |
| Problèmes relatifs à la version câblée de MongoDB | - | Les anciennes versions des pilotes MongoDB ne sont pas en mesure de détecter le nom du compte Azure Cosmos dans les chaînes de connexion. | Ajoutez *appName=@**accountName**@* à la fin de la chaîne de connexion de votre API Cosmos DB pour MongoDB, où ***accountName*** correspond au nom de votre compte Cosmos DB. |

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [utiliser Studio 3T](mongodb-mongochef.md) avec l’API Azure Cosmos DB pour MongoDB.
- Découvrez comment [utiliser Robo 3T](mongodb-robomongo.md) avec l’API Azure Cosmos DB pour MongoDB.
- Explorez les [exemples](mongodb-samples.md) MongoDB avec l’API Azure Cosmos DB pour MongoDB.


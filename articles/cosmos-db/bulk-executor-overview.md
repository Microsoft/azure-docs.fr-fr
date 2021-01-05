---
title: Vue d’ensemble de la bibliothèque d’exécuteur en bloc Azure Cosmos DB
description: Effectuez des opérations en bloc dans Azure Cosmos DB par l’intermédiaire des API d’importation en bloc et de mise à jour en bloc proposées par la bibliothèque d’exécuteur en bloc.
author: tknandu
ms.service: cosmos-db
ms.topic: how-to
ms.date: 05/28/2019
ms.author: ramkris
ms.reviewer: sngun
ms.openlocfilehash: 211fc85f97069fcf3251048a074d625e777f8e7d
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93100471"
---
# <a name="azure-cosmos-db-bulk-executor-library-overview"></a>Vue d’ensemble de la bibliothèque d’exécuteur en bloc Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]
 
Azure Cosmos DB est un service de base de données rapide, flexible et distribué à l’échelle mondiale, conçu pour effectuer un scale-out élastique afin de prendre en charge : 

* Un important débit de lecture et d’écriture (plusieurs millions d’opérations par seconde).  
* Le stockage de volumes élevés (plusieurs centaines de téraoctets ou plus) de données transactionnelles et opérationnelles avec une latence prévisible de l’ordre de la milliseconde  

La bibliothèque d’exécuteur en bloc vous permet de tirer parti de ces stockage et débit conséquents. La bibliothèque d’exécuteur en bloc vous permet d’effectuer des opérations dans Azure Cosmos DB par l’intermédiaire les API d’importation en bloc et de mise à jour en bloc. Les sections qui suivent décrivent en détail les fonctionnalités de la bibliothèque d’exécuteur en bloc. 

> [!NOTE] 
> Actuellement, la bibliothèque d’exécuteur en bloc gère les opérations d’importation et de mise à jour. Seuls les comptes d’API SQL Azure Cosmos DB et les comptes d’API Gremlin la prennent en charge.

> [!IMPORTANT]
> La bibliothèque d’exécuteurs en bloc n’est actuellement pas prise en charge sur les comptes [serverless](serverless.md). Sur .NET, il est recommandé d’utiliser la [prise en charge de l’exécution en bloc](https://devblogs.microsoft.com/cosmosdb/introducing-bulk-support-in-the-net-sdk/) disponible dans la version V3 du kit SDK.
 
## <a name="key-features-of-the-bulk-executor-library"></a>Principales fonctionnalités de la bibliothèque d’exécuteur en bloc  
 
* Elle réduit considérablement la quantité de ressources de calcul côté client nécessaires pour saturer le débit alloué à un conteneur. Une application monothread qui écrit des données à l’aide de l’API d’importation en bloc atteint un débit d’écriture 10 fois supérieur à celui d’une application multithread qui écrit des données en parallèle tout en saturant l’UC de l’ordinateur client.  

* Elle élimine les tâches fastidieuses liées à l’écriture de la logique d’application qui gère la limitation du taux des requêtes, les délais d’attente des requêtes et autres exceptions temporaires en les traitant efficacement dans la bibliothèque.  

* Elle fournit un mécanisme simplifié pour le scale-out des applications effectuant des opérations en bloc. Une seule instance d’exécuteur en bloc sur une machine virtuelle Azure peut consommer plus de 500 000 RU/s. Vous pouvez atteindre un débit plus élevé en ajoutant des instances supplémentaires à chacune des machines virtuelles clientes.  
 
* Elle peut importer en bloc plus d’un téraoctet de données par heure en faisant appel à une architecture de montée en puissance.  

* Elle peut mettre à jour en bloc des données existantes dans des conteneurs Azure Cosmos sous forme de correctifs. 
 
## <a name="how-does-the-bulk-executor-operate"></a>Comment fonctionne l’exécuteur en bloc ? 

Quand une opération en bloc visant à importer ou mettre à jour des documents est déclenchée avec un lot d’entités, elles sont initialement mélangées dans des compartiments correspondant à leur plage de clés de partition Azure Cosmos DB. Dans chaque compartiment qui correspond à une plage de clés de partition, elles sont divisées en mini-lots jouant chacun le rôle de charge utile validée côté serveur. La bibliothèque d’exécuteur en bloc comporte des optimisations intégrées permettant l’exécution simultanée de ces mini-lots entre les groupes de clés de partition et au sein d’un même groupe. L’image suivante montre comment l’exécuteur en bloc traite les données par lots dans différentes clés de partition :  

:::image type="content" source="./media/bulk-executor-overview/bulk-executor-architecture.png" alt-text="Architecture de l’exécuteur en bloc" :::

La bibliothèque de l’exécuteur en bloc veille à utiliser au maximum le débit alloué à une collection. Elle utilise un  [mécanisme de contrôle de congestion de style AIMD](https://tools.ietf.org/html/rfc5681) pour chaque plage de clés de partition Azure Cosmos DB afin de gérer efficacement la limitation du taux et les délais d’attente. 

## <a name="next-steps"></a>Étapes suivantes 
  
* Pour en savoir plus, essayez les exemples d'applications utilisant la bibliothèque d'Exécuteurs en bloc en [.NET](bulk-executor-dot-net.md) et [Java](bulk-executor-java.md).  
* Consultez les notes de publication et les informations sur le SDK de l’exécuteur en bloc en [.NET](sql-api-sdk-bulk-executor-dot-net.md) et en [Java](sql-api-sdk-bulk-executor-java.md).
* La bibliothèque d'Exécuteurs en bloc est intégrée au connecteur Spark Cosmos DB. Pour plus d'informations, consultez [Connecteur Spark Azure Cosmos DB](spark-connector.md).  
* La bibliothèque d’exécuteur en bloc est également intégrée à une nouvelle version du [connecteur Azure Cosmos DB](../data-factory/connector-azure-cosmos-db.md) pour permettre à Azure Data Factory de copier des données.

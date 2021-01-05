---
title: Migrer des centaines de téraoctets de données dans Azure Cosmos DB
description: Ce document explique comment migrer des centaines de téraoctets de données vers Azure Cosmos DB.
author: SnehaGunda
ms.author: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 10/23/2019
ms.openlocfilehash: c45445415f3eaa7cb0f9069dd5f64b57c19e5836
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96437148"
---
# <a name="migrate-hundreds-of-terabytes-of-data-into-azure-cosmos-db"></a>Migrer des centaines de téraoctets de données dans Azure Cosmos DB 
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Azure Cosmos DB peut stocker plusieurs téraoctets de données. Vous pouvez procéder à une migration de données à grande échelle pour transférer votre charge de travail de production vers Azure Cosmos DB. Cet article décrit les défis liés au transfert de données à grande échelle vers Azure Cosmos DB et vous présente l'outil qui permet de relever ces défis pour migrer les données vers Azure Cosmos DB. Dans cette étude de cas, le client a utilisé l'API SQL Cosmos DB.  

Avant de migrer l'intégralité de la charge de travail vers Azure Cosmos DB, vous pouvez migrer un sous-ensemble de données pour valider certains aspects tels que le choix de la clé de partition, les performances des requêtes et la modélisation des données. Une fois la preuve de concept validée, vous pouvez transférer l'intégralité de la charge de travail vers Azure Cosmos DB.  

## <a name="tools-for-data-migration"></a>Outils de migration de données 

Les stratégies de migration Azure Cosmos DB varient en fonction du choix de l'API et de la taille des données. Pour migrer des jeux de données de petite taille (afin de valider la modélisation des données, les performances des requêtes, le choix de la clé de partition, etc.), vous pouvez choisir l'[Outil de migration de données](import-data.md) ou le [connecteur Azure Cosmos DB d'Azure Data Factory](../data-factory/connector-azure-cosmos-db.md). Si vous avez l'habitude de Spark, vous pouvez également choisir d'utiliser le [connecteur Spark Azure Cosmos DB](spark-connector.md) pour migrer les données.

## <a name="challenges-for-large-scale-migrations"></a>Défis liés aux migrations à grande échelle 

Les outils existants pour procéder à la migration des données vers Azure Cosmos DB présentent certaines limites qui deviennent particulièrement évidentes à grande échelle :

 * **Fonctionnalités de scale-out limitées** : pour migrer des téraoctets de données vers Azure Cosmos DB le plus rapidement possible, et utiliser efficacement l'intégralité du débit alloué, les clients de migration doivent avoir la possibilité d’effectuer un scale-out indéfiniment.  

* **Absence de suivi de l'avancement et de pointage de contrôle** : lors de la migration de jeux de données volumineux, il est important de suivre l'avancement de la migration et d'avoir recours au pointage de contrôle. À défaut, toute erreur survenant pendant la migration mettra fin à celle-ci, et vous devrez recommencer le processus à zéro. Il serait improductif de relancer l'intégralité du processus de migration alors que 99 % de celui-ci a déjà été effectué.  

* **Absence de file d'attente de lettres mortes** : dans les jeux de données volumineux, certaines parties des données sources posent parfois des problèmes. En outre, des problèmes temporaires de client ou de réseau peuvent être rencontrés. Aucun de ces cas ne doit entraîner l'échec complet de la migration. Même si la plupart des outils de migration disposent de fonctionnalités robustes permettant d'effectuer de nouvelles tentatives et de prévenir les problèmes intermittents, celles-ci ne sont pas toujours suffisantes. Par exemple, si moins de 0,01 % des documents de données sources font plus de 2 Mo, leur écriture dans Azure Cosmos DB échouera. Idéalement, il est préférable que l'outil de migration conserve ces documents « en échec » dans une autre file d'attente de lettres mortes, qui pourra être traitée après la migration. 

La plupart de ces limites sont en cours de correction pour les outils tels qu'Azure Data Factory et les services Azure Data Migration. 

## <a name="custom-tool-with-bulk-executor-library"></a>Outil personnalisé avec bibliothèque d'Exécuteurs en bloc 

Les défis décrits à la section précédente peuvent être résolus à l'aide d'un outil personnalisé qui peut facilement faire l'objet d'une montée en charge sur plusieurs instances et qui résiste aux défaillances temporaires. En outre, l'outil personnalisé peut suspendre et reprendre la migration à différents points de contrôle. Azure Cosmos DB fournit déjà la [bibliothèque d'Exécuteurs en bloc](./bulk-executor-overview.md) qui intègre certaines de ces fonctionnalités. Par exemple, la bibliothèque d'Exécuteurs en bloc dispose déjà de la fonctionnalité permettant de gérer les erreurs temporaires et peut effectuer un scale-out des threads d'un nœud individuel pour utiliser environ 500 K de RU par nœud. La bibliothèque d'Exécuteurs en bloc partitionne également le jeu de données source en micro-lots qui sont exploités indépendamment comme forme de pointage de contrôle.  

L'outil personnalisé utilise la bibliothèque d'Exécuteurs en bloc. En outre, il assure la montée en charge sur plusieurs clients et le suivi des erreurs pendant le processus d'ingestion. Pour utiliser cet outil, les données sources doivent être partitionnées en fichiers distincts dans Azure Data Lake Storage (ADLS) afin que différents agents de migration puissent récupérer chaque fichier et les ingérer dans Azure Cosmos DB. L'outil personnalisé utilise une collection distincte, qui stocke les métadonnées relatives à l'avancement de la migration de chaque fichier source dans ADLS et assure le suivi des erreurs associées.  

L'illustration suivante décrit le processus de migration à l'aide de cet outil personnalisé. L'outil s'exécute sur un ensemble de machines virtuelles, et chaque machine virtuelle interroge la collection de suivi d'Azure Cosmos DB pour acquérir un bail sur l'une des partitions de données source. Au terme de cette opération, la partition de données source est lue par l'outil et ingérée dans Azure Cosmos DB à l'aide de la bibliothèque d'Exécuteurs en bloc. La collection de suivi est ensuite mise à jour pour enregistrer l'avancement de l'ingestion des données et les éventuelles erreurs rencontrées. Après le traitement d'une partition de données, l'outil tente de rechercher la partition source disponible suivante. Il poursuit le traitement de la partition source suivante jusqu'à ce que toutes les données soient migrées. Le code source de l'outil est disponible [ici](https://github.com/Azure-Samples/azure-cosmosdb-bulkingestion).  

 
:::image type="content" source="./media/migrate-cosmosdb-data/migrationsetup.png" alt-text="Configuration de l’outil de migration" border="false":::
 

 

La collection de suivi contient des documents, comme illustré dans l'exemple suivant. Vous verrez un de ces documents pour chaque partition dans les données sources.  Chaque document contient les métadonnées de la partition de données source, comme son emplacement, son état de migration et les erreurs (le cas échéant) :  

```json
{ 
  "owner": "25812@bulkimporttest07", 
  "jsonStoreEntityImportResponse": { 
    "numberOfDocumentsReceived": 446688, 
    "isError": false, 
    "totalRequestUnitsConsumed": 3950252.2800000003, 
    "errorInfo": [], 
    "totalTimeTakenInSeconds": 188, 
    "numberOfDocumentsImported": 446688 
  }, 
  "storeType": "AZURE_BLOB", 
  "name": "sourceDataPartition", 
  "location": "sourceDataPartitionLocation", 
  "id": "sourceDataPartitionId", 
  "isInProgress": false, 
  "operation": "unpartitioned-writes", 
  "createDate": { 
    "seconds": 1561667225, 
    "nanos": 146000000 
  }, 
  "completeDate": { 
    "seconds": 1561667515, 
    "nanos": 180000000 
  }, 
  "isComplete": true 
} 
```
 

## <a name="prerequisites-for-data-migration"></a>Conditions préalables à la migration des données 

Avant d'entamer la migration des données, quelques conditions préalables doivent être prises en compte :  

#### <a name="estimate-the-data-size"></a>Estimez la taille des données :  

La taille des données sources peut être légèrement différente de celle des données contenues dans Azure Cosmos DB. Quelques échantillons de documents de la source peuvent être insérés pour vérifier la taille de leurs données dans Azure Cosmos DB. D'après la taille de l'échantillon, il est possible d'estimer la taille totale des données contenues dans Azure Cosmos DB. 

Par exemple, si la taille de chaque document est d'environ 1 Ko après la migration vers Azure Cosmos DB et que le jeu de données source contient environ 60 milliards de documents, la taille estimée des données contenues dans Azure Cosmos DB avoisine les 60 To. 

 

#### <a name="pre-create-containers-with-enough-rus"></a>Pré-créez des conteneurs avec suffisamment de RU : 

Bien qu'Azure Cosmos DB procède à une montée en charge automatique de l'espace de stockage, il est déconseillé de commencer avec la plus petite taille de conteneur. Les petits conteneurs ont un débit plus faible, ce qui ralentira la migration. Il est donc préférable de créer les conteneurs en tenant compte de la taille finale des données (telle qu'estimée à l'étape précédente) et de veiller à ce que la charge de travail de migration utilise pleinement le débit alloué.  

À l'étape précédente, la taille des données a été estimée à environ 60 To. Par conséquent, un conteneur d'au moins 2,4 millions de RU est nécessaire pour héberger l'intégralité du jeu de données.  

 

#### <a name="estimate-the-migration-speed"></a>Estimez la vitesse de migration : 

En supposant que la charge de travail de migration puisse utiliser la totalité du débit alloué, le débit alloué fournirait une estimation de la vitesse de migration. Si l'on reprend l'exemple précédent, 5 RU sont nécessaires pour écrire un document de 1 Ko sur le compte de l'API SQL Azure Cosmos DB.  2,4 millions de RU permettraient un transfert de 480 000 documents par seconde (ou 480 Mo/s). Cela signifie que la migration complète de 60 To prendra 125 000 secondes, soit environ 34 heures.  

Si vous souhaitez que la migration soit achevée dans la journée, vous devez augmenter le débit alloué en passant à 5 millions de RU. 

 

#### <a name="turn-off-the-indexing"></a>Désactivez l'indexation :  

Dans la mesure où la migration doit être achevée le plus tôt possible, il est conseillé de réduire le temps et les RU consacrés à la création d'index pour chacun des documents ingérés.  Azure Cosmos DB indexe automatiquement toutes les propriétés. Il peut donc être intéressant de réduire l'indexation à quelques termes sélectionnés ou de la désactiver complètement pendant la migration. Vous pouvez désactiver la stratégie d'indexation du conteneur en définissant indexingMode sur none, comme indiqué ci-dessous :  

 
```
  { 
        "indexingMode": "none" 
  } 
```
 

Au terme de la migration, vous pourrez procéder à une mise à jour de l'indexation.  

## <a name="migration-process"></a>Processus de migration 

Une fois les conditions préalables remplies, vous pouvez migrer les données en procédant comme suit :  

1. Commencez par importer les données de la source vers le Stockage Blob Azure. Pour accélérer la migration, n'hésitez pas à mettre en parallèle les partitions sources distinctes. Avant d'entamer la migration, le jeu de données source doit être partitionné en fichiers d'environ 200 Mo.   

2. La bibliothèque d'exécuteurs en bloc peut monter en puissance pour utiliser 500 000 RU sur une même machine virtuelle cliente. Puisque le débit disponible est de 5 millions de RU, 10 machines virtuelles Ubuntu 16.04 (Standard_D32_v3) doivent être approvisionnées dans la région de votre base de données Azure Cosmos. Vous devez préparer ces machines virtuelles avec l'outil de migration et son fichier de paramètres.  

3. Exécutez l'étape relative à la file d'attente sur l'une des machines virtuelles clientes. Cette étape crée la collection de suivi, qui analyse le conteneur ADLS et crée un document de suivi de l'avancement pour chacun des fichiers de partition du jeu de données source.  

4. Exécutez ensuite l'étape d'importation sur toutes les machines virtuelles clientes. Chacun des clients peut s'approprier une partition source et ingérer ses données dans Azure Cosmos DB. Une fois l'opération terminée et son état mis à jour dans la collection de suivi, les clients peuvent rechercher la prochaine partition source disponible dans la collection de suivi.  

5. Ce processus se poursuit jusqu'à ce que toutes les partitions sources aient été ingérées. Une fois que toutes les partitions sources ont été traitées, l'outil doit être réexécuté en mode correction d'erreurs sur la même collection de suivi. Cette étape est nécessaire pour identifier les partitions sources à retraiter en raison d'erreurs.  

6. Certaines de ces erreurs peuvent être dues à des documents incorrects dans les données sources. Vous devez les identifier et les corriger. Ensuite, vous devez réexécuter l'étape d'importation sur les partitions défaillantes pour les réingérer. 

Au terme de la migration, vous pouvez vérifier que le nombre de documents contenus dans Azure Cosmos DB et dans la base de données source est le même. Dans cet exemple, la taille totale des données contenues dans Azure Cosmos DB était de 65 téraoctets. Après la migration, l'indexation peut être activée de manière sélective et les RU peuvent être abaissées au niveau requis par les opérations de la charge de travail.

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus, essayez les exemples d'applications utilisant la bibliothèque d'Exécuteurs en bloc en [.NET](bulk-executor-dot-net.md) et [Java](bulk-executor-java.md). 
* La bibliothèque d'Exécuteurs en bloc est intégrée au connecteur Spark Cosmos DB. Pour plus d'informations, consultez [Connecteur Spark Azure Cosmos DB](spark-connector.md).  
* Pour contacter l’équipe Azure Cosmos DB en ouvrant un ticket de support sous le type de problème « Conseils généraux », sous-type de problème « Grandes (To+) migrations » pour obtenir une aide supplémentaire lors des migrations à grande échelle.
---
title: Architecture de SQL Synapse
description: Découvrez comment Azure Synapse SQL combine des capacités de traitement de requêtes distribuées avec Stockage Azure pour atteindre des performances et une scalabilité élevées.
services: synapse-analytics
author: mlee3gsd
manager: rothja
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: ''
ms.date: 04/15/2020
ms.author: martinle
ms.reviewer: igorstan
ms.openlocfilehash: da6c9f6df0e9e74de297cf6c8f655b62e3446bad
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462708"
---
# <a name="azure-synapse-sql-architecture"></a>Architecture Azure Synapse SQL 

Cet article décrit les composants d’architecture de SQL Synapse.

## <a name="synapse-sql-architecture-components"></a>Composants de l’architecture SQL Synapse

SQL Synapse tire parti d’une architecture scale-out pour répartir le traitement informatique des données sur plusieurs nœuds. Le calcul est séparé du stockage, ce qui permet d’adapter l’échelle du calcul indépendamment des données présentes dans le système. 

Pour un pool SQL dédié, l’unité d’échelle est une abstraction de la puissance de calcul connue sous le nom d’[unité DWU (Data Warehouse Unit)](resource-consumption-models.md). 

Pour un pool SQL serverless, la mise à l’échelle s’effectue automatiquement pour répondre aux besoins en ressources de la requête. À mesure que la topologie change au fil du temps avec l’ajout, la suppression de nœuds ou des basculements, elle s’adapte aux changements et vérifie que votre requête dispose de suffisamment de ressources et qu’elle se termine correctement. Par exemple, l’image ci-dessous montre un pool SQL serverless utilisant quatre nœuds de calcul pour exécuter une requête.

![Architecture de SQL Synapse](./media//overview-architecture/sql-architecture.png)

SQL Synapse utilise une architecture basée sur des nœuds. Les applications se connectent et envoient des commandes T-SQL à un nœud de contrôle qui est le seul point d’entrée pour SQL Synapse. 

Le nœud de contrôle Azure Synapse SQL utilise un moteur de requêtes distribuées, qui optimise les requêtes pour un traitement en parallèle, puis transmet les opérations aux nœuds de calcul qui accomplissent leur travail en parallèle. 

Le nœud de contrôle de pool SQL serverless utilise le moteur de traitement des requêtes distribuées (DQP) pour optimiser et orchestrer l’exécution distribuée de la requête utilisateur en la fractionnant en requêtes plus petites qui seront exécutées sur des nœuds de calcul. Chaque petite requête est appelée tâche et représente une unité d’exécution distribuée. Elle lit les fichiers à partir du stockage, joint les résultats à partir d’autres tâches, regroupe ou organise les données récupérées à partir d’autres tâches. 

Les nœuds de calcul stockent toutes les données utilisateur dans un stockage Azure et exécutent les requêtes parallèles. Le service de déplacement de données (DMS) est un service interne de niveau système, qui déplace les données entre les nœuds en fonction des besoins pour exécuter des requêtes en parallèle et retourner des résultats précis. 

Avec le stockage et le calcul découplés, lorsque vous utilisez SQL Synapse, vous pouvez tirer parti du dimensionnement indépendant de la puissance de calcul, quels que soient vos besoins en stockage. Pour un pool SQL serverless, la mise à l’échelle s’effectue automatiquement, tandis que pour un pool SQL dédié, vous pouvez :

* Augmenter ou réduire la puissance de calcul dans un pool SQL dédié sans déplacer les données.
* Suspendre la capacité de calcul tout en conservant les données intactes de façon à ce que vous payiez uniquement pour le stockage.
* Reprendre le calcul pendant les heures d’utilisation.

## <a name="azure-storage"></a>Stockage Azure

SQL Synapse met à profit Stockage Azure pour sécuriser vos données utilisateur. Étant donné que vos données sont stockées et gérées par Stockage Azure, votre consommation de stockage est facturée séparément. 

Un pool SQL serverless vous permet d’interroger les fichiers de votre lac de données en lecture seule, tandis qu’un pool SQL vous permet également d’ingérer des données. Lorsque des données sont ingérées dans le pool SQL dédié, elles sont partitionnées en **distributions** pour optimiser les performances du système. Vous pouvez choisir le modèle de partitionnement à utiliser pour distribuer les données lorsque vous définissez la table. Les modèles de partitionnement pris en charge sont les suivants :

* Hachage
* Tourniquet (round robin)
* Réplication

## <a name="control-node"></a>Nœud de contrôle

Le nœud de contrôle est le cerveau de l’architecture. Il s’agit du nœud frontal qui interagit avec toutes les applications et les connexions. 

Dans Synapse SQL, le moteur de requêtes distribuées s'exécute sur le nœud de contrôle pour optimiser et coordonner les requêtes parallèles. Lorsque vous envoyez une requête T-SQL à un pool SQL dédié, le nœud de contrôle la transforme en plusieurs requêtes distinctes qui s’exécutent sur chaque distribution en parallèle.

Dans un pool SQL serverless, le moteur de traitement des requêtes distribuées (DQP) s’exécute sur le nœud de contrôle pour optimiser et coordonner l’exécution distribuée de la requête utilisateur en la fractionnant en requêtes plus petites qui seront exécutées sur des nœuds de calcul. Il assigne également des ensembles de fichiers à traiter par chaque nœud.

## <a name="compute-nodes"></a>Nœuds de calcul

Les nœuds de calcul fournissent la puissance de calcul. 

Dans un pool SQL dédié, les distributions sont mappées aux nœuds de calcul pour traitement. À mesure que vous payez pour davantage de ressources de calcul, le pool re-mappe les distributions aux nœuds de calcul disponibles. Le nombre de nœuds calcul (entre 1 et 60) est déterminé par le niveau de service du pool SQL dédié. Chaque nœud de calcul a un ID de nœud visible dans les vues système. Vous pouvez voir l’ID de nœud de calcul en effectuant une recherche dans la colonne node_id des vues système dont le nom commence par sys.pdw_nodes. Pour obtenir la liste de ces vues système, consultez [Vues système Synapse SQL](/sql/relational-databases/system-catalog-views/sql-data-warehouse-and-parallel-data-warehouse-catalog-views?view=azure-sqldw-latest).

Dans un pool SQL serverless, chaque nœud de calcul reçoit une tâche et un ensemble de fichiers sur lesquels exécuter la tâche. La tâche est une unité d’exécution de requête distribuée, qui fait partie de la requête envoyée par l’utilisateur. La mise à l’échelle automatique est activée afin de s’assurer qu’un nombre suffisant de nœuds de calcul est utilisé pour exécuter une requête utilisateur.

## <a name="data-movement-service"></a>Service de déplacement de données

Le service de déplacement des données (DMS) est la technologie de transport de données dans le pool SQL dédié qui coordonne le déplacement des données entre les nœuds de calcul. Certaines requêtes nécessitent un déplacement de données pour garantir que les requêtes parallèles renvoient des résultats précis. Quand un déplacement de données est requis, le service de déplacement des données veille à ce que les bonnes données aillent au bon emplacement.

> [!VIDEO https://www.youtube.com/embed/PlyQ8yOb8kc]

## <a name="distributions"></a>Distributions

Une distribution est l’unité de base de stockage et de traitement pour des requêtes parallèles s’exécutant sur des données distribuées dans un pool SQL dédié. Quand le pool SQL dédié exécute une requête, le travail est divisé en 60 requêtes plus petites qui s’exécutent en parallèle. 

Chacune de ces 60 requêtes s’exécute sur l’une des distributions de données. Chaque nœud de calcul gère une ou plusieurs des 60 distributions. Un pool SQL dédié disposant des ressources de calcul maximales a une distribution par nœud de calcul. Un pool SQL dédié disposant des ressources de calcul minimales a toutes les distributions sur un nœud de calcul. 

## <a name="hash-distributed-tables"></a>Tables distribuées par hachage
Une table distribuée de hachage peut offrir les meilleures performances de requête pour des jointures et agrégations sur des tables volumineuses. 

Pour partitionner des données en table distribuée par hachage, le pool SQL dédié utilise une fonction de hachage pour attribuer de manière déterministe chaque ligne à une distribution. Dans la définition de la table, une des colonnes est désignée comme colonne de distribution. La fonction de hachage utilise les valeurs figurant dans la colonne de distribution pour assigner chaque ligne à une distribution.

Le schéma suivant illustre comment une table complète (non distribuée) est stockée sous la forme d’une table distribuée par hachage. 

![Table distribuée](media//overview-architecture/hash-distributed-table.png "Table distribuée") 

* Chaque ligne appartient à une distribution. 
* Un algorithme de hachage déterministe attribue chaque ligne à une distribution. 
* Le nombre de lignes de table par distribution varie selon la taille des tables.

Des critères de performances, tels que la distinction, l’asymétrie des données et les types des requêtes exécutées sur le système, conditionnent le choix de la colonne de distribution.

## <a name="round-robin-distributed-tables"></a>Tables distribuées par tourniquet

Une table de type tourniquet (round robin) est la table la plus simple à créer. Elle offre des performances rapides quand elle est utilisée en tant que table intermédiaire pour les chargements.

Une table distribuée de type tourniquet (round robin) distribue uniformément les données sur la table, mais sans optimisation. Une distribution est tout d’abord choisie au hasard. Ensuite, des tampons de lignes sont affectés aux distributions de manière séquentielle. Le chargement de données dans une table de type tourniquet (round robin) est rapide, mais les performances des requête peuvent souvent être meilleures avec des tables distribuées de hachage. Des jointures sur des tables de type tourniquet (round robin) nécessitent un remaniement des données, qui nécessite un temps supplémentaire.

## <a name="replicated-tables"></a>Tables répliquées
Une table répliquée offre les performances de requête les plus rapides pour des petites tables.

Une copie complète d’une table répliquée est mise en cache sur chaque nœud de calcul. Par conséquent, la réplication d’une table a pour effet d’éviter la nécessité de transférer des données entre nœuds de calcul avant une jointure ou une agrégation. Des tables répliquées sont utilisées de façon optimale avec des tables de petite taille. Un espace de stockage supplémentaire est nécessaire, et une surcharge additionnelle est engagée lors de l’écriture de données, ce qui rend peu pratique l’usage de tables volumineuses. 

Le diagramme ci-dessous présente une table répliquée qui est mise en cache sur la première distribution sur chaque nœud de calcul. 

![Table répliquée](media/overview-architecture/replicated-table.png "Table répliquée") 

## <a name="next-steps"></a>Étapes suivantes

À présent que vous en savez un peu plus sur Synapse SQL, découvrez comment [créer un pool SQL dédié](../quickstart-create-sql-pool-portal.md) et [charger des exemples de données](../sql-data-warehouse/sql-data-warehouse-load-from-azure-blob-storage-with-polybase.md) (./sql-data-warehouse-load-sample-databases.md) rapidement. Vous pouvez également commencer [à utiliser un pool SQL serverless](../quickstart-sql-on-demand.md). Si vous n’êtes pas encore familiarisé avec Azure, vous pouvez vous appuyer sur le [Glossaire Azure](../../azure-glossary-cloud-terminology.md) lorsque vous rencontrez de nouveaux termes. 

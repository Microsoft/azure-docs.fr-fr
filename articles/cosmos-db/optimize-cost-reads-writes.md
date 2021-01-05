---
title: Optimisation des coûts de vos demandes dans Azure Cosmos DB
description: Cet article explique comment optimiser les coûts lors de l’émission de demandes sur Azure Cosmos DB.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 10/14/2020
ms.openlocfilehash: 3f4c33a88d9a8fbf2c3d64135d93da54cf75fab3
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93097499"
---
# <a name="optimize-request-cost-in-azure-cosmos-db"></a>Optimiser le coût de requête dans Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Cet article explique comment les demandes de lecture et d’écriture se traduisent en [unités de requête](request-units.md) et comment optimiser le coût de ces requêtes. Les opérations de lecture incluent les requêtes et les lectures de points. Les opérations d’écriture incluent l’insertion, le remplacement, la suppression et l’upsert d’éléments.

Azure Cosmos DB propose un ensemble complet d’opérations de base de données qui opèrent sur les éléments au sein d’un conteneur. Le coût associé à chacune de ces opérations varie en fonction du processeur, des E/S et de la mémoire nécessaires à l’exécution de l’opération. Plutôt que de vous soucier de la gestion des ressources matérielles, vous pouvez considérer une unité de requête comme une mesure unique des ressources nécessaires à l’exécution des opérations de base de données pour répondre à une requête.

## <a name="measuring-the-ru-charge-of-a-request"></a>Mesure de la facturation d’unité de requête d’une demande

Il est important de mesurer le coût de la RU de vos demandes pour comprendre le coût réel et évaluer l’efficacité de vos optimisations. Vous pouvez extraire ce coût en utilisant le portail Azure ou en examinant la réponse renvoyée par Azure Cosmos DB via l’un des kits de développement logiciel (SDK). Pour obtenir des instructions détaillées sur la façon de procéder, consultez [Rechercher les frais d’unités de requête dans Azure Cosmos DB](find-request-unit-charge.md).

## <a name="reading-data-point-reads-and-queries"></a>Lecture des données : lectures et requêtes de points

Les opérations de lecture dans Azure Cosmos DB sont généralement triées de la plus à la moins rapide/efficace du point de vue de la consommation de RU :  

* Lectures de point (recherche de clé/valeur sur un ID d’élément unique et une clé de partition).
* Requête avec une clause de filtre au sein d’une clé de partition unique.
* Requête sans clause de filtre d’égalité ou de plage sur une propriété.
* Requête sans filtre.

### <a name="role-of-the-consistency-level"></a>Rôle du niveau de cohérence

Lors de l’utilisation des [niveaux de cohérence](consistency-levels.md) **fort** ou **obsolescence limitée** , le coût de la RU de toute opération de lecture (lecture de point ou requête) est doublé.

### <a name="point-reads"></a>Lectures de points

Le seul facteur affectant les frais de RU d’une lecture de point (en plus du niveau de cohérence utilisé) est la taille de l’élément récupéré. Le tableau suivant indique le coût de RU des lectures de points pour les éléments dont la taille est comprise entre 1 Ko et 100 Ko.

| **Taille de l’élément** | **Coût d’une lecture de point** |
| --- | --- |
| 1 Ko | 1 unité de requête |
| 100 Ko | 10 unités de requête |

Étant donné que les lectures de points (recherches de clé/valeur sur l’ID d’élément) constituent le type de lecture le plus efficace, vous devez vous assurer que votre ID d’élément a une valeur significative afin que vous puissiez extraire vos éléments avec une lecture de point (au lieu d’une requête) si possible.

### <a name="queries"></a>Requêtes

Les unités d’une requête dépendent de plusieurs facteurs. Par exemple, le nombre d’éléments Azure Cosmos chargés/renvoyés, le nombre de recherches basées sur l’index, les détails sur la durée de compilation de la requête, etc. Azure Cosmos DB veille à ce que la même requête, lorsqu’elle est exécutée sur les mêmes données, coûte toujours le même nombre d’unités de requête, même lorsque les exécutions se répètent. Le profil de requête basé sur les métriques d’exécution vous donne une idée assez précise de la façon dont les unités de requête sont utilisées.  

Dans certains cas, vous pouvez voir une séquence de 200 et 429 réponses, ainsi que des unités de requête variables dans une exécution de requête paginée. En effet, les requêtes s’exécutent aussi rapidement que possible selon les unités de requête disponibles. Une exécution de requête peut être répartie entre plusieurs pages/allers-retours serveur et client. Par exemple, 10 000 éléments peuvent être renvoyés sur plusieurs pages. Chacune de ces pages est facturée en fonction du calcul effectué pour cette page. Lorsque vous calculez la somme de toutes ces pages, vous devez obtenir le même nombre d’unités de requête que vous obtiendriez pour toute la requête.

#### <a name="metrics-for-troubleshooting-queries"></a>Métriques pour les requêtes de dépannage

Les performances et le débit consommés par les requêtes (ainsi que les fonctions définies par l’utilisateur) dépendent principalement du corps de la fonction. Pour déterminer facilement le temps d’exécution de la requête dans les UDF et le nombre d’unités de requête consommées, vous pouvez activer les métriques de requête. Si vous utilisez le SDK .net, voici quelques exemples de métriques de requête qu’il renvoie :

```bash
Retrieved Document Count                 :               1              
Retrieved Document Size                  :           9,963 bytes        
Output Document Count                    :               1              
Output Document Size                     :          10,012 bytes        
Index Utilization                        :          100.00 %            
Total Query Execution Time               :            0.48 milliseconds 
  Query Preparation Times 
    Query Compilation Time               :            0.07 milliseconds 
    Logical Plan Build Time              :            0.03 milliseconds 
    Physical Plan Build Time             :            0.05 milliseconds 
    Query Optimization Time              :            0.00 milliseconds 
  Index Lookup Time                      :            0.06 milliseconds 
  Document Load Time                     :            0.03 milliseconds 
  Runtime Execution Times 
    Query Engine Execution Time          :            0.03 milliseconds 
    System Function Execution Time       :            0.00 milliseconds 
    User-defined Function Execution Time :            0.00 milliseconds 
  Document Write Time                    :            0.00 milliseconds 
  Client Side Metrics 
    Retry Count                          :               1              
    Request Charge                       :            3.19 RUs  
```

#### <a name="best-practices-to-cost-optimize-queries"></a>Meilleures pratiques pour optimiser le coût des requêtes 

Gardez à l’esprit les meilleures pratiques suivantes lors de l’optimisation du coût des requêtes :

* **Rassembler plusieurs types d’entités**

   Essayez de rassembler plusieurs types d’entités au sein d’un conteneur unique ou d’un petit nombre de conteneurs. Cette méthode génère des avantages non seulement d’un point de vue tarifaire, mais également pour les transactions et l’exécution des requêtes. La portée des requêtes se limite à un seul conteneur. Par ailleurs, les transactions atomiques sur plusieurs enregistrements via des procédures stockées/déclencheurs sont limitées à une clé de partition dans un même conteneur. Le regroupement des entités dans le même conteneur peut réduire le nombre d’allers-retours réseau destinés à résoudre les relations entre les enregistrements. Par conséquent, les performances de bout en bout sont augmentées, les transactions atomiques sur plusieurs enregistrements pour un jeu de données plus volumineux sont possibles, et les coûts sont ainsi réduits. Si le regroupement de plusieurs types d’entités au sein d’un conteneur ou d’un groupe réduit de conteneurs est difficile pour votre scénario (c’est généralement le cas si vous migrez une application existante et ne souhaitez pas apporter des modifications de code), envisagez le provisionnement du débit au niveau de la base de données.  

* **Mesure et réglage pour réduire l’utilisation d’unités de requête par seconde**

   La complexité d’une requête a un impact sur le nombre d’unités de requête (RU) consommées pour une opération. Le nombre de prédicats, la nature des prédicats, le nombre de fonctions définies par l’utilisateur et la taille du jeu de données sources. Tous ces facteurs ont une influence sur le coût des opérations d’une requête. 

Azure Cosmos DB offre des performances prévisibles en termes de débit et de latence grâce à un modèle de débit configuré. Le débit configuré est représenté en termes d’[unités de requête](request-units.md) par seconde, ou RU/s. Une unité de requête (RU) désigne une abstraction logique sur des ressources de calcul telles que l’UC, la mémoire, les E/S, etc., qui sont nécessaires pour effectuer une requête. Le débit configuré (RU) est réservé et dédié à votre conteneur ou à votre base de données pour fournir une latence et un débit prévisibles. Le débit configuré permet à Azure Cosmos DB de fournir des performances prévisibles et cohérentes, une faible latence garantie et une haute disponibilité à n’importe quelle échelle. Les unités de requête représentent la devise normalisée qui simplifie le raisonnement lié au nombre de ressources requises pour une application.

Les frais renvoyés dans l’en-tête de requête indiquent le coût d’une requête donnée. Par exemple, si une requête renvoie 1 000 éléments de 1 Ko, le coût de l’opération est 1 000. Par conséquent, en une seconde, le serveur honore uniquement deux requêtes avant de limiter le taux de requêtes suivantes. Pour plus d’informations, accédez à l’article [Unités de requête](request-units.md) et à la calculatrice d’unités de requête.

## <a name="writing-data"></a>Écriture de données

Le coût de RU lié à l’écriture d’un élément dépend des points suivants :

- La taille de l’élément.
- Le nombre de propriétés couvertes par la [stratégie d’indexation](index-policy.md) et nécessaires pour l’indexation.

L’insertion d’un élément de 1 Ko avec moins de 5 propriétés à indexer coûte environ 5 RU. Le remplacement d’un élément coûte deux fois les frais requis pour insérer le même élément.

### <a name="optimizing-writes"></a>Optimisation des écritures

La meilleure façon d’optimiser le coût de RU des opérations d’écriture est de faire dimensionner correctement vos éléments et le nombre de propriétés indexées.

- Le stockage de très grands éléments dans Azure Cosmos DB entraîne des frais de RU élevés et peut être considéré comme un anti-modèle. En particulier, ne stockez pas de contenu binaire ou de gros blocs de texte que vous n’avez pas besoin d’interroger. Il est recommandé de placer ce type de données dans un [Stockage d’objets blob Azure](../storage/blobs/storage-blobs-introduction.md) et de stocker une référence (ou un lien) vers l’objet blob dans l’élément que vous écrivez dans Azure Cosmos DB.
- L’optimisation de votre stratégie d’indexation pour indexer uniquement les propriétés sur lesquelles vos requêtes sont filtrées peut faire une grande différence dans le nombre de RU consommées par vos opérations d’écriture. Lors de la création d’un conteneur, la stratégie d’indexation par défaut indexe chaque propriété trouvée dans vos éléments. Bien qu’il s’agit d’une bonne valeur par défaut pour les activités de développement, il est vivement recommandé de réévaluer et de [personnaliser votre stratégie d’indexation](how-to-manage-indexing-policy.md) lors de la mise en production ou lorsque votre charge de travail commence à recevoir un trafic significatif.

Lors de l’ingestion de données en bloc, il est également recommandé d’utiliser la [bibliothèque d’exécuteur en bloc Azure Cosmos DB](bulk-executor-overview.md), car il est conçu pour optimiser la consommation de RU par ces opérations. Si vous le souhaitez, vous pouvez également utiliser [Azure Data Factory](../data-factory/introduction.md) qui repose sur cette même bibliothèque.

## <a name="next-steps"></a>Étapes suivantes

Pour continuer à développer vos connaissances sur l’optimisation des coûts dans Azure Cosmos DB, consultez les articles suivants :

* En savoir plus sur l’[optimisation pour le développement et le test](optimize-dev-test.md)
* En savoir plus sur les [factures Azure Cosmos DB](understand-your-bill.md)
* En savoir plus sur l’[optimisation du coût du débit](optimize-cost-throughput.md)
* En savoir plus sur l’[optimisation du coût de stockage](optimize-cost-storage.md)
* En savoir plus sur l’[optimisation du coût des comptes Azure Cosmos sur plusieurs régions](optimize-cost-regions.md)
* En savoir plus sur la [capacité de réserve Azure Cosmos DB](cosmos-db-reserved-capacity.md)

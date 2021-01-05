---
title: Augmenter les performances de débit pour Azure SQL Database à partir d’Azure Stream Analytics
description: Découvrez comment charger des données de sortie d’Azure Stream Analytics dans SQL Azure et comment améliorer les débits d’écriture.
author: chetanmsft
ms.author: chetang
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 03/18/2019
ms.openlocfilehash: a902cbc8256e249a608a5da8436a1145ddb0d565
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94490931"
---
# <a name="increase-throughput-performance-to-azure-sql-database-from-azure-stream-analytics"></a>Augmenter les performances de débit pour Azure SQL Database à partir d’Azure Stream Analytics

Cet article donne des conseils pour améliorer les performances de débit d’écriture quand vous chargez des données dans Azure SQL Database à partir d’Azure Stream Analytics.

La sortie SQL dans Azure Stream Analytics prend en charge l’option d’écriture en parallèle. Cette option permet les topologies de travaux [massivement parallèles](stream-analytics-parallelization.md#embarrassingly-parallel-jobs), où plusieurs partitions de sortie sont écrites en parallèle dans la table de destination. Toutefois, l’activation de cette option dans Azure Stream Analytics ne permet pas toujours d’observer une hausse du débit, car le résultat dépend fortement de la configuration et du schéma de table de votre base de données. Les index, la clé de clustering, le facteur de remplissage d’index et la compression que vous choisissez ont un impact sur le temps de chargement des tables. Pour plus d’informations sur les moyens d’optimiser votre base de données afin d’améliorer les performances des requêtes et du chargement sur la base de tests d’évaluation internes, consultez les [recommandations relatives aux performances de SQL Database](../azure-sql/database/performance-guidance.md). La réorganisation des écritures n’est pas garantie lors de l’écriture en parallèle dans la base de données SQL.

Les configurations présentées ci-après pour chaque service peuvent vous aider à améliorer le débit global de votre solution.

## <a name="azure-stream-analytics"></a>Azure Stream Analytics

- **Héritage du partitionnement** – Cette option de configuration de la sortie SQL permet d’hériter le schéma de partitionnement de l’entrée ou l’étape de requête précédente. L’activation de cette option contribue à améliorer les débits lors de l’écriture dans une table sur disque et l’utilisation d’une topologie de travail [massivement parallèle](stream-analytics-parallelization.md#embarrassingly-parallel-jobs). Ce partitionnement est déjà automatiquement activé pour beaucoup d’autres [sorties](stream-analytics-parallelization.md#partitions-in-inputs-and-outputs). Le verrouillage de table (TABLOCK) est également désactivé pour les insertions en bloc effectuées avec cette option.

> [!NOTE] 
> Quand il y a plus de huit partitions d’entrée, l’héritage du schéma de partitionnement d’entrée n’est pas toujours une option appropriée. Cette limite supérieure a été observée sur une table contenant une seule colonne d’identité et un index cluster. Dans ce cas, envisagez d’utiliser [INTO](/stream-analytics-query/into-azure-stream-analytics#into-shard-count) 8 dans votre requête pour spécifier explicitement le nombre de générateurs de sortie. Les résultats observés peuvent varier en fonction du schéma et des index choisis.

- **Taille de lot** – Avec cette option de configuration de la sortie SQL, vous pouvez spécifier la taille de lot maximale dans une sortie SQL d’Azure Stream Analytics en fonction de la nature de votre table de destination/charge de travail. La taille de lot correspond au nombre maximal d’enregistrements qui sont envoyés avec chaque opération d’insertion en bloc. Dans les index cluster columnstore, une taille de lot d’environ [100 000](/sql/relational-databases/indexes/columnstore-indexes-data-loading-guidance) permet d’optimiser la parallélisation, la journalisation minimale et le verrouillage. Dans les tables sur disque, une taille de 10 000 (valeur par défaut) ou moins peut être optimale pour votre solution, car des tailles de lot plus élevées risquent de déclencher une escalade de verrous durant les insertions en bloc.

- **Paramétrage des messages d’entrée** – Si vous avez déjà optimisé les performances à l’aide de l’héritage du partitionnement et de la taille de lot, vous pouvez en plus augmenter le nombre d’événements d’entrée par message dans chaque partition pour améliorer encore le débit d’écriture. Le paramétrage des messages d’entrée permet d’augmenter les tailles de lot dans Azure Stream Analytics jusqu’à la taille de lot spécifié, ce qui améliore le débit. Cela est possible en utilisant la [compression](stream-analytics-define-inputs.md) ou en augmentant la taille des messages d’entrée dans EventHub ou Blob.

## <a name="sql-azure"></a>SQL Azure

- **Table et index partitionnés** – L’utilisation d’une table SQL [partitionnée](/sql/relational-databases/partitions/partitioned-tables-and-indexes) et d’index partitionnés sur la table ayant la même colonne que votre clé de partition (par exemple, PartitionId) peut réduire considérablement les conflits entre partitions durant les opérations d’écriture. Pour utiliser une table partitionnée, vous devez créer une [fonction de partition](/sql/t-sql/statements/create-partition-function-transact-sql) et un [schéma de partition](/sql/t-sql/statements/create-partition-scheme-transact-sql) dans le groupe de fichiers PRIMARY. Cela améliorera également la disponibilité des données existantes quand de nouvelles données sont chargées. La limite des E/S de journal peut être atteinte en fonction du nombre de partitions, mais vous pouvez l’augmenter en mettant à niveau la référence SKU.

- **Éviter les violations de clé unique** – Si vous obtenez [plusieurs messages d’avertissement de violation de clé](stream-analytics-troubleshoot-output.md#key-violation-warning-with-azure-sql-database-output) dans le journal d’activité Azure Stream Analytics, assurez-vous que votre travail n’est pas impacté par des violations de contrainte unique qui sont susceptibles de se produire dans des scénarios de reprise d’activité. Pour éviter ce problème, définissez l’option [IGNORE\_DUP\_KEY](stream-analytics-troubleshoot-output.md#key-violation-warning-with-azure-sql-database-output) sur vos index.

## <a name="azure-data-factory-and-in-memory-tables"></a>Azure Data Factory et tables en mémoire

- **Table en mémoire en tant que table temporaire** – Les [tables en mémoire](/sql/relational-databases/in-memory-oltp/in-memory-oltp-in-memory-optimization) permettent des chargements de données à très haut débit, mais les données doivent tenir dans la mémoire. Les tests d’évaluation montrent que le chargement en masse d’une table en mémoire vers une table sur disque est environ dix fois plus rapide que l’insertion en bloc directe à l’aide d’un seul writer dans la table sur disque ayant une colonne d’identité et un index cluster. Pour améliorer ces performances d’insertion en bloc, créez un [travail de copie avec Azure Data Factory](../data-factory/connector-azure-sql-database.md) qui copie les données de la table en mémoire vers la table sur disque.

## <a name="avoiding-performance-pitfalls"></a>Éviter l’écueil des performances
Insérer des données en bloc prend beaucoup moins de temps que de les charger par insertions multiples, car cela élimine les contraintes répétées liées au transfert des données, à l’analyse de l’instruction d’insertion, à l’exécution de l’instruction et à l’émission d’un enregistrement de transaction. En effet, le chemin emprunté par les données diffusées dans le moteur de stockage s’avère plus efficace. Le coût de configuration de ce chemin est cependant beaucoup plus élevé que celui d’une simple instruction d’insertion dans une table sur disque. Le point d’équilibre se situe généralement autour des 100 lignes. Au-delà, le chargement en masse est presque toujours plus efficace. 

Si le taux d’événements entrants est faible, des tailles de lot inférieures à 100 lignes peuvent alors être facilement créées, ce qui altère l’efficacité de l’insertion en masse et occupe trop d’espace disque. Pour contourner cette limitation, vous pouvez prendre une des mesures suivantes :
* Créer un [déclencheur](/sql/t-sql/statements/create-trigger-transact-sql) INSTEAD OF pour utiliser une instruction simple pour chaque ligne.
* Utilisez une table temporaire en mémoire, comme décrit dans la section précédente.

Un autre scénario de ce type se produit quand l’écriture se produit dans un index columnstore non cluster (NCCI), où des insertions en masse plus petites peuvent créer un nombre de segments trop important, ce qui peut bloquer l’index. Dans ce cas, il est recommandé d’utiliser plutôt un index columstore cluster.

## <a name="summary"></a>Résumé

En résumé, avec la fonctionnalité de sortie partitionnée dans Azure Stream Analytics pour la sortie SQL, la parallélisation alignée de votre travail avec une table partitionnée dans SQL Azure doit en principe vous apporter des améliorations de débit significatives. L’utilisation d’Azure Data Factory pour orchestrer le déplacement des données d’une table en mémoire vers des tables sur disque contribue aussi à améliorer le débit. Quand cela est possible, l’optimisation de la densité des messages peut également être un facteur majeur de l’amélioration du débit global.

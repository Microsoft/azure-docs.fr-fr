---
title: Niveau Premium pour Azure Data Lake Storage | Microsoft Docs
description: Utilisez le niveau de performance Premium avec Azure Data Lake Storage Gen2
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: conceptual
ms.custom: references_regions
ms.date: 10/30/2020
ms.author: normesta
ms.openlocfilehash: be440407fb6f4d9715ba80c584af023a5f662394
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93324220"
---
# <a name="premium-tier-for-azure-data-lake-storage"></a>Niveau Premium pour Azure Data Lake Storage

Azure Data Lake Storage Gen2 prend désormais en charge le [niveau de performance Premium](storage-blob-performance-tiers.md#premium-performance). Le niveau de performance Premium est idéal pour les applications d’analytique Big Data et les charges de travail qui nécessitent une latence faible et cohérente, et disposent d’un grand nombre de transactions.

## <a name="workloads-that-can-benefit-from-the-premium-performance-tier"></a>Charges de travail qui peuvent tirer parti du niveau de performance Premium

Les exemples de charges de travail incluent des charges de travail interactives, IoT, l’analytique streaming, l’intelligence artificielle et le machine learning. 

**Charges de travail interactives** 

Ces charges de travail requièrent des mises à jour instantanées et des commentaires de la part des utilisateurs, comme des applications d’e-commerce et de cartographie, des applications vidéo interactives, etc. Par exemple, dans une application d’e-commerce, les éléments les moins fréquemment consultés ne sont probablement pas mis en cache. Toutefois, ils doivent être affichés instantanément au client sur demande. Autre exemple : les scientifiques des données, les analystes et les développeurs peuvent dériver des insights sensibles au facteur temps encore plus rapidement en exécutant des requêtes sur des données stockées dans un compte qui utilise le niveau de performance Premium. 

**Analytique streaming/IoT** 

Dans un scénario IoT, un grand nombre d’opérations d’écriture plus petites peuvent être envoyées (push) chaque seconde dans le cloud. De grandes quantités de données peuvent être ingérées et agrégées à des fins d’analyse, puis supprimées presque immédiatement. Les capacités d’ingestion élevées du niveau de performance Premium le rendent efficace pour ce type de charge de travail. 

**Intelligence artificielle/machine learning (IA/ML)** 

IA/ML traite de la consommation et du traitement de différents types de données, tels que les visuels, la parole et le texte. Ce type de charge de travail de calcul haute performance gère des volumes importants de données nécessitant une réponse rapide et des temps d’ingestion efficaces pour l’analyse des données. 

## <a name="cost-effectiveness"></a>Rentabilité

Le niveau de performance Premium a un coût de stockage plus élevé, mais un coût de transaction moindre par rapport au niveau de performance standard. Si vos applications et charges de travail exécutent un grand nombre de transactions, le niveau de performance Premium peut être rentable.

Le tableau suivant illustre la rentabilité du niveau Premium pour Azure Data Lake Storage. Chaque colonne représente le nombre de transactions sur un mois.  Chaque ligne représente le pourcentage de transactions qui sont des transactions de lecture. Chaque cellule du tableau indique le pourcentage de réduction des coûts associé à un pourcentage de transactions de lecture et au nombre de transactions exécutées. 

Par exemple, en supposant que votre compte se trouve dans la région USA Est 2, le nombre de transactions avec votre compte dépasse 90 M et 70 % de ces transactions sont des transactions de lecture, le niveau de performance Premium est plus rentable.

> [!div class="mx-imgBorder"]
> ![emplacement de l’image](./media/premium-tier-for-data-lake-storage/premium-performance-data-lake-storage-cost-analysis-table.png)

> [!NOTE] 
> Si vous préférez évaluer la rentabilité en fonction du nombre de transactions par seconde pour chaque To de données, vous pouvez utiliser les en-têtes de colonne qui apparaissent en bas du tableau.

Pour plus d’informations sur les prix, consultez la page [Tarification d’Azure Data Lake Storage Gen2](https://azure.microsoft.com/pricing/details/storage/data-lake/).

## <a name="feature-availability"></a>Disponibilité des fonctionnalités 

Certaines fonctionnalités de stockage Blob ne sont peut-être pas disponibles ou ne prennent en charge que partiellement le niveau de performance Premium. Pour en obtenir la liste complète, consultez [Fonctionnalités de stockage Blob disponibles dans Azure Data Lake Storage Gen2](data-lake-storage-supported-blob-storage-features.md). Consultez ensuite la liste des [problèmes connus](data-lake-storage-known-issues.md) pour voir si certaines fonctionnalités sont manquantes.

## <a name="enabling-the-premium-performance-tier"></a>Activation du niveau de performance Premium 

Vous pouvez utiliser le niveau Premium pour Azure Data Lake Storage en créant un compte BlockBlobStorage avec le paramètre **Espace de noms hiérarchique** **activé**. Pour obtenir des instructions complètes, consultez [Créer un compte BlockBlobStorage](storage-blob-create-account-block-blob.md).

Quand vous créez le compte, veillez à choisir l’option de performance **Premium** et le type de compte **BlockBlobStorage**.

> [!div class="mx-imgBorder"]
> ![Créer un compte BlockBlobStorage](./media/premium-tier-for-data-lake-storage/create-block-blob-storage-account.png)

Activez le paramètre **Espace de noms hiérarchique** sous l’onglet **Avancé** de la page **Créer un compte de stockage**. Vous devez activer ce paramètre lorsque vous créez le compte. Vous ne pourrez pas l’activer par la suite.

L’illustration suivante montre ce paramètre dans la page **Créer un compte de stockage**.

> [!div class="mx-imgBorder"]
> ![Paramètre d’espace de noms hiérarchique](./media/create-data-lake-storage-account/hierarchical-namespace-feature.png)

## <a name="regional-availability"></a>Disponibilité régionale

Le niveau Premium pour Azure Data Lake Storage est disponible dans les régions suivantes.

|Région|Redondance|
|--|--|
|USA Est|LRS, ZRS|
|USA Est 2|LRS, ZRS|
|USA Centre|LRS|
|USA Ouest|LRS|
|USA Ouest 2|LRS, ZRS|
|Centre-USA Ouest|LRS|
|États-Unis - partie centrale méridionale|LRS|
|Centre du Canada|LRS|
|Est du Canada|LRS|
|Europe Nord|LRS, ZRS|
|Europe Ouest|LRS, ZRS|
|Sud du Royaume-Uni|LRS|
|Ouest du Royaume-Uni|LRS|
|France Centre|LRS|
|Asie de l’Est|LRS|
|Centre de la Corée|LRS|
|Corée du Sud|LRS|
|Inde centrale|LRS|
|Inde Ouest|LRS|
|Émirats arabes unis Nord|LRS|
|Japon Est|LRS|
|OuJapon Est|LRS|
|Asie Sud-Est|LRS, ZRS|
|Australie Est|LRS, ZRS|
|Sud-Est de l’Australie|LRS|
|Brésil Sud|LRS|

## <a name="next-steps"></a>Étapes suivantes

Utilisez le niveau Premium pour Azure Data Lake Storage avec votre service d’analytique favori, comme Azure Databricks, Azure HDInsight et Azure Synapse Analytics. 

- [Tutoriel : Azure Data Lake Storage Gen2, Azure Databricks et Spark](data-lake-storage-use-databricks-spark.md) 
- [Utilisez Azure Data Lake Storage Gen2 avec les clusters Azure HDInsight](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md) HDInsight prend actuellement en charge un compte qui utilise le niveau de performance Premium avec un cluster HBase sur lequel les écritures accélérées sont activées.
- [Démarrage rapide : Créer un espace de travail Synapse](../../synapse-analytics/quickstart-create-workspace.md)


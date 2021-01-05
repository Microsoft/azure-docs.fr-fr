---
title: Présentation d'Azure Data Lake Storage Gen2
description: Lisez une présentation d’Azure Data Lake Storage Gen2. Découvrez les fonctionnalités clés. Passez en revue les fonctionnalités de Stockage Blob, les intégrations de service Azure et les plateformes prises en charge.
author: normesta
ms.service: storage
ms.topic: overview
ms.date: 02/25/2020
ms.author: normesta
ms.reviewer: jamesbak
ms.subservice: data-lake-storage-gen2
ms.openlocfilehash: 5f2f3cfc5ccbdd6a3d3d3ede5bb39a3f6f548b19
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95913094"
---
# <a name="introduction-to-azure-data-lake-storage-gen2"></a>Présentation d'Azure Data Lake Storage Gen2

Azure Data Lake Storage Gen2 est un ensemble de fonctionnalités dédiées à l'analytique du Big Data et intégrées au service [Stockage Blob Azure](storage-blobs-introduction.md). 

Data Lake Storage Gen2 fait converger les fonctionnalités d’[Azure Data Lake Storage Gen1](../../data-lake-store/index.yml) vers celles du Stockage Blob Azure. Par exemple, Data Lake Storage Gen2 fournit une sémantique du système de fichiers, une sécurité au niveau des fichiers et la mise à l’échelle. Comme ces fonctionnalités sont basées sur le stockage Blob, vous bénéficiez également d’un stockage hiérarchisé à faible coût avec des fonctionnalités à haute disponibilité et de reprise d’activité.

## <a name="designed-for-enterprise-big-data-analytics"></a>Conçu pour l’analytique du Big Data d’entreprise

Data Lake Storage Gen2 fait du stockage Azure la base pour créer des dépôts Data Lake d’entreprise sur Azure. Conçu dès le départ pour traiter plusieurs téraoctets d’informations tout en assurant des centaines de gigaoctets de débit, Data Lake Storage Gen2 vous permet de facilement gérer d'importants volumes de données.

Une caractéristique fondamentale de Data Lake Storage Gen2 est l’ajout d’un [espace de noms hiérarchique](data-lake-storage-namespace.md) au stockage Blob. L’espace de noms hiérarchique organise les objets/fichiers dans une hiérarchie de répertoires pour offrir un accès efficace aux données. Une convention de nommage de magasin d’objets commune utilise des barres obliques dans le nom pour reproduire une structure hiérarchique de dossiers. Cette structure devient réelle avec Data Lake Storage Gen2. Des opérations telles que le renommage ou la suppression d’un répertoire deviennent des opérations individuelles de métadonnées atomiques sur le répertoire. Il n’est pas nécessaire d’énumérer et de traiter tous les objets qui partagent le préfixe du nom du répertoire.

Data Lake Storage Gen2 s’appuie sur le Stockage Blob. Il améliore les performances, la gestion et la sécurité de diverses façons :

-   Les **performances** sont optimisées, car vous n’avez pas besoin de copier ou transformer les données avant l’analyse. Par rapport à l’espace de noms plat du Stockage Blob, l’espace de noms hiérarchique améliore considérablement les performances des opérations de gestion de répertoires et donc les performances générales du travail.

-   La **gestion** est plus facile, car vous pouvez organiser et manipuler les fichiers dans des répertoires et des sous-répertoires.

-   La **sécurité** est applicable, car vous pouvez définir des autorisations POSIX sur des répertoires ou des fichiers spécifiques.

Par ailleurs, Data Lake Storage Gen2 est extrêmement rentable dans la mesure où il repose sur le [Stockage Blob Azure](storage-blobs-introduction.md), dont le coût est faible. Les fonctionnalités supplémentaires réduisent encore le coût total de possession de l’analytique du Big Data sur Azure.

## <a name="key-features-of-data-lake-storage-gen2"></a>Principales fonctionnalités de Data Lake Storage Gen2

-   **Accès compatible Hadoop** : Azure Data Lake Storage Gen2 vous permet de gérer les données et d’y accéder comme vous le feriez avec un [système de fichiers HDFS (Hadoop Distributed File System)](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html). Le nouveau [pilote ABFS](data-lake-storage-abfs-driver.md) (utilisé pour accéder aux données) est disponible dans tous les environnements Apache Hadoop. Ces environnements incluent [Azure HDInsight](../../hdinsight/index.yml) *,* [Azure Databricks](/azure/databricks/) et [Azure Synapse Analytics](/azure/synapse-analytics).

-   **Surensemble d’autorisations POSIX** : Le modèle de sécurité pour Data Lake Gen2 prend en charge les autorisations ACL et POSIX, ainsi que certaines granularités supplémentaires spécifiques à Data Lake Storage Gen2. Les paramètres peuvent être configurés via l’Explorateur Stockage ou des infrastructures telles que Hive et Spark.

-   **Rentabilité** : Data Lake Storage Gen2 offre une capacité de stockage et des transactions à bas coût. Des fonctionnalités telles que le [cycle de vie de Stockage Blob Azure](storage-lifecycle-management-concepts.md) optimisent les coûts tout au long du cycle de vie des données.

-   **Pilote optimisé** : Le pilote ABFS est [optimisé spécifiquement](data-lake-storage-abfs-driver.md) pour l’analytique Big Data. Les API REST correspondantes sont exposées par le biais du point de terminaison `dfs.core.windows.net`.

### <a name="scalability"></a>Extensibilité

Le Stockage Azure est nativement scalable, que vous y accédiez via les interfaces Data Lake Storage Gen2 ou de stockage Blob. Il peut stocker et traiter *plusieurs exaoctets de données*. Ce volume de stockage est disponible avec un débit mesuré en gigabits par seconde (Gbits/s) et des niveaux élevés d’opérations d’entrée/sortie par seconde (IOPS). Le traitement est exécuté à des latences par demande quasi constantes, qui sont mesurées au niveau du service, du compte et du fichier.

### <a name="cost-effectiveness"></a>Rentabilité

Comme Data Lake Storage Gen2 repose sur le stockage Blob Azure, la capacité de stockage et les coûts des transactions sont moins importants. Contrairement à d’autres services de stockage cloud, vous n’avez pas besoin de déplacer ni de transformer vos données pour pouvoir les analyser. Pour plus d’informations sur la tarification, consultez la page [Tarification Azure Storage](https://azure.microsoft.com/pricing/details/storage).

Par ailleurs, des fonctionnalités comme [l’espace de noms hiérarchique](data-lake-storage-namespace.md) améliorent considérablement les performances générales de nombreuses tâches analytiques. Cette amélioration des performances signifie que vous avez besoin d’une puissance de calcul moindre pour traiter la même quantité de données, ce qui contribue à la baisse du coût total de possession (TCO) pour le travail analytique de bout en bout.

### <a name="one-service-multiple-concepts"></a>Un seul service, plusieurs concepts

Comme Data Lake Storage Gen2 repose sur le stockage Blob Azure, plusieurs concepts peuvent décrire les mêmes éléments partagés.

Les éléments suivants sont des entités équivalentes, décrites par différents concepts. Sauf indication contraire, ces entités sont directement synonymes :

| Concept                                | Organisation de niveau supérieur | Organisation de niveau inférieur                                            | Conteneur de données |
|----------------------------------------|------------------------|---------------------------------------------------------------------|----------------|
| Objets BLOB – Stockage d’objets à usage général | Conteneur              | Répertoire virtuel (Kit de développement logiciel uniquement – ne fournit aucune manipulation atomique) | Objet blob           |
| Azure Data Lake Storage Gen2 – Analytics Storage          | Conteneur            | Répertoire                                                           | Fichier           |

## <a name="supported-blob-storage-features"></a>Fonctionnalités du stockage Blob prises en charge

Les fonctionnalités de stockage Blob, telles que la [journalisation des diagnostics](../common/storage-analytics-logging.md), les [niveaux d’accès](storage-blob-storage-tiers.md) et les [stratégies de gestion du cycle de vie du stockage Blob](storage-lifecycle-management-concepts.md) sont disponibles pour votre compte. 

Pour obtenir la liste des fonctionnalités du stockage Blob prises en charge, consultez [Fonctionnalités du stockage Blob disponibles dans Azure Data Lake Storage Gen2](data-lake-storage-supported-blob-storage-features.md).

## <a name="supported-azure-service-integrations"></a>Intégrations de service Azure prises en charge

Data Lake Storage Gen2 prend en charge plusieurs services Azure. Vous pouvez les utiliser pour ingérer des données, effectuer une analytique et créer des représentations visuelles. Pour obtenir la liste des services Azure pris en charge, consultez [Services Azure prenant en charge Azure Data Lake Storage Gen2](data-lake-storage-supported-azure-services.md).

## <a name="supported-open-source-platforms"></a>Plateformes open source prises en charge

Plusieurs plateformes open source prennent en charge le stockage Data Lake Gen2. Pour obtenir une liste complète, consultez [Plateformes open source prenant en charge Azure Data Lake Storage Gen2](data-lake-storage-supported-open-source-platforms.md).

## <a name="see-also"></a>Voir aussi

- [Problèmes connus avec Azure Data Lake Storage Gen2](data-lake-storage-known-issues.md)
- [Accès multiprotocole pour Azure Data Lake Storage](data-lake-storage-multi-protocol-access.md)
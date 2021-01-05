---
title: Scénarios de données impliquant Azure Data Lake Storage Gen2 | Microsoft Docs
description: Comprendre les différents scénarios et outils à l’aide desquels les données peuvent être ingérées, traitées, téléchargées et affichées dans Data Lake Storage Gen2 (anciennement Azure Data Lake Store)
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: conceptual
ms.date: 02/14/2020
ms.author: normesta
ms.reviewer: stewu
ms.openlocfilehash: eea7fb073cdf99ee1f4257f6824375a6502a4fad
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95913621"
---
# <a name="using-azure-data-lake-storage-gen2-for-big-data-requirements"></a>Utilisation d’Azure Data Lake Storage Gen2 pour le Big Data

Il existe quatre étapes principales dans traitement des données Big Data :

> [!div class="checklist"]
> * Réception de grandes quantités de données dans un magasin de données, en temps réel ou par lots
> * Traitement des données
> * Téléchargement des données
> * Visualisation des données

Cet article présente les options et outils pour chaque phase de traitement.

Pour obtenir la liste complète des services Azure que vous pouvez utiliser avec Azure Data Lake Storage Gen2, consultez [Intégrer Azure Data Lake Storage avec les services Azure](./data-lake-storage-supported-azure-services.md).

## <a name="ingest-the-data-into-data-lake-storage-gen2"></a>Ingérer les données dans Data Lake Storage Gen2

Cette section présente les différentes sources de données et les différentes manières d’ingérer ces données dans un compte Data Lake Storage Gen2.

![Ingérer des données dans Data Lake Storage Gen2](./media/data-lake-storage-data-scenarios/ingest-data.png "Ingérer des données dans Data Lake Storage Gen2")

### <a name="ad-hoc-data"></a>Données ad hoc

Ceci représente les petits jeux de données qui sont utilisés pour créer un prototype d’une application de Big Data. Il existe différentes façons de recevoir des données ad hoc en fonction de la source de données. 

Voici une liste d'outils que vous pouvez utiliser pour ingérer des données ad hoc.

| source de données | Réception avec |
| --- | --- |
| Ordinateur local |[Azure PowerShell](data-lake-storage-directory-file-acl-powershell.md)<br><br>[Azure CLI](data-lake-storage-directory-file-acl-cli.md)<br><br>[Explorateur Stockage](https://azure.microsoft.com/features/storage-explorer/)<br><br>[Outil AzCopy](../common/storage-use-azcopy-v10.md)|
| Azure Storage Blob |[Azure Data Factory](../../data-factory/connector-azure-data-lake-store.md).<br><br>[Outil AzCopy](../common/storage-use-azcopy-v10.md)<br><br>[DistCp en cours d’exécution sur un cluster HDInsight](data-lake-storage-use-distcp.md)|

### <a name="streamed-data"></a>Flux de données

Ceci représente les données qui peuvent être générées par diverses sources, telles que des applications, des appareils, des capteurs, etc. Ces données peuvent être ingérées dans Data Lake Storage Gen2 par des outils divers. En général, ces outils capturent et traitent les données sur la base de l’événement en temps réel, puis ils écrivent les événements par lots dans Data Lake Storage Gen2 afin qu’ils puissent être traités.

Voici une liste d'outils que vous pouvez utiliser pour ingérer des données diffusées en continu.

|Outil | Assistance |
|---|--|
|Azure Stream Analytics|[Démarrage rapide : Créer un travail Stream Analytics à l’aide du portail Azure](../../stream-analytics/stream-analytics-quick-create-portal.md) <br> [Sortie vers Azure Data Lake Gen2](../../stream-analytics/stream-analytics-define-outputs.md)|
|Azure HDInsight Storm | [Écrire dans Apache Hadoop HDFS à partir d'Apache Storm sur HDInsight](../../hdinsight/storm/apache-storm-write-data-lake-store.md) |

### <a name="relational-data"></a>Données relationnelles

Les bases de données relationnelles peuvent également être utilisées comme sources des données. Sur une période donnée, les bases de données relationnelles collectent de grandes quantités de données qui peuvent fournir des informations clés si elles sont traitées via un pipeline de Big Data. Vous pouvez utiliser les outils suivants pour déplacer ces données vers Data Lake Storage Gen2.

Voici une liste d'outils que vous pouvez utiliser pour ingérer des données relationnelles.

|Outil | Assistance |
|---|--|
|Azure Data Factory | [Activité Copy dans Azure Data Factory](../../data-factory/copy-activity-overview.md) |

### <a name="web-server-log-data-upload-using-custom-applications"></a>Données de journal de serveur web (téléchargement à l’aide d’applications personnalisées)

Ce type de jeu de données est spécifiquement indiqué, car l’analyse des données de journal de serveur web constitue un cas d’usage courant pour les applications de Big Data et nécessite le chargement d’importants volumes de fichiers journaux sur Data Lake Storage Gen2. Vous pouvez utiliser les outils suivants pour écrire vos propres scripts ou applications pour télécharger ces données.

Voici une liste d'outils que vous pouvez utiliser pour ingérer des données de journal de serveur web.

|Outil | Assistance |
|---|--|
|Azure Data Factory | [Activité Copy dans Azure Data Factory](../../data-factory/copy-activity-overview.md)  |
|Azure CLI|[Azure CLI](data-lake-storage-directory-file-acl-cli.md)|
|Azure PowerShell|[Azure PowerShell](data-lake-storage-directory-file-acl-powershell.md)|

Pour télécharger des données de journal de serveur web, et également pour télécharger d’autres types de données (par exemple, les données relatives aux sentiments sociaux), il est préférable d’écrire vos propres scripts/applications personnalisés, car cela vous donne la possibilité d’inclure votre composant de téléchargement de données dans le cadre de votre application de Big Data plus étendue. Dans certains cas, ce code peut prendre la forme d’un script ou d’un utilitaire de ligne de commande simple. Dans d’autres cas, le code peut être utilisé pour intégrer le traitement de Big Data dans une solution ou une application métier.

### <a name="data-associated-with-azure-hdinsight-clusters"></a>Données associées aux clusters Azure HDInsight

La plupart des types de clusters HDInsight (Hadoop, HBase, Storm) prennent en charge Data Lake Storage Gen2 comme référentiel de stockage des données. Les clusters HDInsight accèdent aux données à partir des objets blob d’Azure Storage (WASB). Pour optimiser les performances, vous pouvez copier les données à partir de WASB sur un compte Data Lake Storage Gen2 associé au cluster. Vous pouvez utiliser les outils suivants pour copier les données.

Voici une liste d'outils que vous pouvez utiliser pour ingérer des données associées à des clusters HDInsight.

|Outil | Assistance |
|---|--|
|Apache DistCp | [Utiliser DistCp pour copier des données entre Azure Storage Blob et Azure Data Lake Storage Gen2](./data-lake-storage-use-distcp.md) |
|Outil AzCopy | [Transférer des données avec AzCopy](../common/storage-use-azcopy-v10.md) |
|Azure Data Factory | [Copier des données vers ou depuis Azure Data Lake Storage Gen2 à l'aide d'Azure Data Factory](../../data-factory/load-azure-data-lake-storage-gen2.md) |

### <a name="data-stored-in-on-premises-or-iaas-hadoop-clusters"></a>Données stockées localement ou dans des clusters IaaS Hadoop

De grandes quantités de données peuvent être stockées dans des clusters Hadoop existants, localement sur les ordinateurs à l’aide de HDFS. Les clusters Hadoop peuvent être inclus dans un déploiement local ou au sein d’un cluster IaaS sur Azure. Il peut être nécessaire de copier ces données dans Azure Data Lake Storage Gen2 afin de bénéficier d’un accès unique ou périodique. Différentes options vous permettent d’y parvenir. Voici une liste de ces options et des compromis correspondants.

| Approche | Détails | Avantages | Considérations |
| --- | --- | --- | --- |
| Utiliser Azure Data Factory (ADF) pour copier des données directement à partir de clusters Hadoop dans Azure Data Lake Storage Gen2 |[ADF prend en charge HDFS comme source de données](../../data-factory/connector-hdfs.md) |ADF offre une prise en charge immédiate de HDFS ainsi qu’une gestion et une surveillance de bout en bout de premier ordre |Nécessite que la passerelle de gestion des données soit déployée localement ou dans le cluster IaaS |
| Utilisez Distcp pour copier les données de Hadoop dans Azure Storage. Copiez ensuite les fichiers de Stockage Azure vers Data Lake Storage Gen2 à l’aide du mécanisme approprié. |Vous pouvez copier les données de Stockage Azure vers Data Lake Storage Gen2 en utilisant : <ul><li>[Azure Data Factory](../../data-factory/copy-activity-overview.md).</li><li>[Outil AzCopy](../common/storage-use-azcopy-v10.md)</li><li>[pache DistCp en cours d’exécution sur des clusters HDInsight](data-lake-storage-use-distcp.md)</li></ul> |Vous pouvez utiliser des outils open source. |Processus en plusieurs étapes qui implique différentes technologies |

### <a name="really-large-datasets"></a>Jeux de données très volumineux

Pour télécharger des jeux de données qui comptent plusieurs téraoctets, l’utilisation des méthodes décrites ci-dessus peut parfois être lente et coûteuse. Dans ce cas, vous pouvez utiliser Azure ExpressRoute.  

Azure ExpressRoute vous permet de créer des connexions privées entre les infrastructures et les centres de données Azure dans votre environnement local. Ceci constitue une option fiable pour le transfert de grandes quantités de données. Pour en savoir plus, consultez la [Documentation Azure ExpressRoute](../../expressroute/expressroute-introduction.md).

## <a name="process-the-data"></a>Traiter les données

Une fois que les données sont disponibles dans Data Lake Storage Gen2, vous pouvez exécuter une analyse sur ces données à l’aide des applications de Big Data prises en charge. 

![Analyser des données dans Data Lake Storage Gen2](./media/data-lake-storage-data-scenarios/analyze-data.png "Analyser des données dans Data Lake Storage Gen2")

Voici une liste d'outils que vous pouvez utiliser pour exécuter des travaux d'analyse sur les données stockées dans Data Lake Storage Gen2.

|Outil | Assistance |
|---|--|
|Azure HDInsight | [Utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md) |
|Azure Databricks | [Azure Data Lake Storage Gen2](https://docs.azuredatabricks.net/spark/latest/data-sources/azure/azure-datalake-gen2.html)<br><br>[Démarrage rapide : Analyser des données dans Azure Data Lake Storage Gen2 à l'aide d'Azure Databricks](./data-lake-storage-quickstart-create-databricks-account.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)<br><br>[Tutoriel : Extraire, transformer et charger des données à l'aide d'Azure Databricks](/azure/databricks/scenarios/databricks-extract-load-sql-data-warehouse?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)|

## <a name="visualize-the-data"></a>Visualiser les données

Utilisez le connecteur Power BI pour créer des représentations visuelles des données stockées dans Data Lake Storage Gen2. Consultez [Analyser des données dans Azure Data Lake Storage Gen2 à l’aide de Power BI](/power-query/connectors/datalakestorage).

## <a name="download-the-data"></a>Télécharger les données

Vous pouvez également être amené à télécharger ou à déplacer les données à partir d’Azure Data Lake Storage Gen2 dans certains cas, tels que :

* Déplacer des données vers d’autres référentiels pour créer une interface avec vos pipelines de traitement des données existantes. Par exemple, vous pourriez vouloir déplacer des données de Data Lake Storage Gen2 vers Azure SQL Database ou une instance SQL Server.

* Télécharger des données sur votre ordinateur local pour le traitement dans des environnements IDE lors de la création de prototypes d’applications.

![Extraire des données de Data Lake Storage Gen2](./media/data-lake-storage-data-scenarios/egress-data.png "Extraire des données de Data Lake Storage Gen2")

Voici une liste d'outils que vous pouvez utiliser pour télécharger des données à partir de Data Lake Storage Gen2.

|Outil | Assistance |
|---|--|
|Azure Data Factory | [Activité Copy dans Azure Data Factory](../../data-factory/copy-activity-overview.md) |
|Apache DistCp | [Utiliser DistCp pour copier des données entre Azure Storage Blob et Azure Data Lake Storage Gen2](./data-lake-storage-use-distcp.md) |
|Explorateur de stockage Azure|[Utiliser l’Explorateur Stockage Azure pour gérer les répertoires, les fichiers et les listes de contrôle d’accès dans Azure Data Lake Storage Gen2](data-lake-storage-explorer.md)|
|Outil AzCopy|[Transférer des données avec AzCopy et le Stockage Blob](../common/storage-use-azcopy-blobs.md)|
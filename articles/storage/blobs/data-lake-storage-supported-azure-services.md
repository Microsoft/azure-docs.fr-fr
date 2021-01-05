---
title: Services Azure prenant en charge Azure Data Lake Storage Gen2 | Microsoft Docs
description: Apprenez-en davantage sur les services Azure qui s’intègrent avec Azure Data Lake Storage Gen2
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: conceptual
ms.date: 08/05/2020
ms.author: normesta
ms.reviewer: stewu
ms.openlocfilehash: cbbbf4ddc651c535cb1266257990f149c80b4742
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97562495"
---
# <a name="azure-services-that-support-azure-data-lake-storage-gen2"></a>Services Azure qui prennent en charge Azure Data Lake Storage Gen2

Vous pouvez utiliser les services Azure pour ingérer des données, effectuer une analytique et créer des représentations visuelles. Cet article fournit la liste des services Azure pris en charge, indique leur niveau de prise en charge, et fournit des liens vers des articles qui vous aident à utiliser ces services avec Azure Data Lake Storage Gen2.

## <a name="supported-azure-services"></a>Services Azure pris en charge

Ce tableau répertorie les services Azure que vous pouvez utiliser avec Azure Data Lake Storage Gen2. Les éléments figurant dans ces tableaux seront modifiés au fil du temps, car la prise en charge continue de s’étendre.

> [!NOTE]
> Le niveau de prise en charge fait référence uniquement à la façon dont le service est pris en charge avec Data Lake Storage Gen2.

|Service Azure |Niveau de support |Azure AD |Clé partagée| Articles connexes |
|---------------|-------------------|---|---|---|
|Azure Data Factory|Mise à la disposition générale|Oui|Oui|[Charger des données dans Azure Data Lake Storage Gen2 avec Azure Data Factory](../../data-factory/load-azure-data-lake-storage-gen2.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)|
|Azure Databricks|Mise à la disposition générale|Oui|Oui|[Utiliser avec Azure Databricks](https://docs.azuredatabricks.net/data/data-sources/azure/azure-datalake-gen2.html) <br> [Démarrage rapide : Analyser des données dans Azure Data Lake Storage Gen2 à l'aide d'Azure Databricks](data-lake-storage-quickstart-create-databricks-account.md) <br>[Tutoriel : Extraire, transformer et charger des données à l'aide d'Azure Databricks](/azure/databricks/scenarios/databricks-extract-load-sql-data-warehouse) <br>[Tutoriel : Accéder aux données Data Lake Storage Gen2 avec Azure Databricks à l’aide de Spark](data-lake-storage-use-databricks-spark.md)|
|Azure Event Hub|Mise à la disposition générale|Non|Oui|[Capturer des événements avec Azure Event Hubs dans Stockage Blob Azure ou Azure Data Lake Storage](../../event-hubs/event-hubs-capture-overview.md)|
|Azure Event Grid|Mise à la disposition générale|Oui|Oui|[Tutoriel : Implémenter le modèle de capture de lac de données pour mettre à jour une table Delta Databricks](data-lake-storage-events.md)|
|Azure Logic Apps|Mise à la disposition générale|Non|Oui|[Vue d’ensemble d’Azure Logic Apps](../../logic-apps/logic-apps-overview.md)|
|Azure Machine Learning|Mise à la disposition générale|Oui|Oui|[Accéder aux données dans les services de stockage Azure](../../machine-learning/how-to-access-data.md)|
|Azure Stream Analytics|Mise à la disposition générale|Oui|Oui|[Démarrage rapide : Créer un travail Stream Analytics à l’aide du portail Azure](../../stream-analytics/stream-analytics-quick-create-portal.md) <br> [Sortie vers Azure Data Lake Gen2](../../stream-analytics/stream-analytics-define-outputs.md)|
|Data Box|Mise à la disposition générale|Non|Oui|[Utiliser Azure Data Box pour migrer les données d’un magasin HDFS local vers Stockage Azure](data-lake-storage-migrate-on-premises-hdfs-cluster.md)|
|HDInsight |Mise à la disposition générale|Oui|Oui|[Utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)<br>[Utilisation de l’interface CLI HDFS avec Data Lake Storage Gen2](data-lake-storage-use-hdfs-data-lake-storage.md) <br>[Tutoriel : Extraire, transformer et charger des données à l’aide d’Apache Hive sur Azure HDInsight](data-lake-storage-tutorial-extract-transform-load-hive.md)|
|IoT Hub |Mise à la disposition générale|Oui|Oui|[Utiliser le routage des messages IoT Hub pour envoyer des messages appareil-à-cloud à différents points de terminaison](../../iot-hub/iot-hub-devguide-messages-d2c.md)|
|Power BI|Mise à la disposition générale|Oui|Oui|[Analyse des données dans Data Lake Storage Gen2 à l’aide de Power BI](/power-query/connectors/datalakestorage)|
|Azure Synapse Analytics (anciennement SQL Data Warehouse)|Mise à la disposition générale|Oui|Oui|[Analyser des données dans un compte de stockage](/azure/synapse-analytics/get-started-analyze-storage?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)|
|SQL Server Integration Services (SSIS)|Mise à la disposition générale|Oui|Oui|[Gestionnaire de connexions Stockage Azure](/sql/integration-services/connection-manager/azure-storage-connection-manager)|
|Explorateur de données Azure|Mise à la disposition générale|Oui|Oui|[Interroger des données dans Azure Data Lake à l’aide d’Azure Data Explorer](/azure/data-explorer/data-lake-query-data)|
|Recherche cognitive Azure|PRÉVERSION|Oui|Oui|[Indexer et parcourir des documents Azure Data Lake Storage Gen2 (préversion)](../../search/search-howto-index-azure-data-lake-storage.md)|
|Azure Content Delivery Network|Pas encore pris en charge|Non applicable|Non applicable|[Indexer et parcourir des documents Azure Data Lake Storage Gen2 (préversion)](../../cdn/cdn-overview.md)|
|Azure SQL Database|Pas encore pris en charge|Non applicable|Non applicable|[Qu’est-ce qu’Azure SQL Database ?](/azure/azure-sql/database/sql-database-paas-overview)|

## <a name="see-also"></a>Voir aussi

- [Problèmes connus avec Azure Data Lake Storage Gen2](data-lake-storage-known-issues.md)
- [Fonctionnalités de stockage blob disponibles dans Azure Data Lake Storage Gen2](data-lake-storage-supported-blob-storage-features.md)
- [Plateformes open source prenant en charge Azure Data Lake Storage Gen2](data-lake-storage-supported-open-source-platforms.md)
- [Accès multiprotocole pour Azure Data Lake Storage](data-lake-storage-multi-protocol-access.md)
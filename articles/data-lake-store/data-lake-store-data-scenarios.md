---
title: Scénarios de données impliquant Data Lake Storage Gen1 | Microsoft Docs
description: Comprendre les différents scénarios et outils à l’aide desquels les données peuvent être ingérées, traitées, téléchargées et affichées dans Data Lake Storage Gen1 (anciennement Azure Data Lake Store)
services: data-lake-store
documentationcenter: ''
author: twooley
manager: mtillman
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: twooley
ms.openlocfilehash: 702a59f768bfe978e1a7708d91c6b6b1bb9c92fe
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452232"
---
# <a name="using-azure-data-lake-storage-gen1-for-big-data-requirements"></a>Utilisation d’Azure Data Lake Storage Gen1 pour le Big Data

[!INCLUDE [data-lake-storage-gen1-rename-note.md](../../includes/data-lake-storage-gen1-rename-note.md)]

Il existe quatre étapes principales dans traitement des données Big Data :

* Réception de grandes quantités de données dans un magasin de données, en temps réel ou par lots
* Traitement des données
* Téléchargement des données
* Visualisation des données

Dans cet article, nous allons examiner ces étapes en ce qui concerne Azure Data Lake Storage Gen1 pour comprendre les options et les outils disponibles pour répondre à vos besoins en termes de Big Data.

## <a name="ingest-data-into-data-lake-storage-gen1"></a>Ingérer des données dans Data Lake Storage Gen1
Cette section présente les différentes sources de données et les différentes manières d’ingérer ces données dans un compte Data Lake Storage Gen1.

![Ingérer des données dans Data Lake Storage Gen1](./media/data-lake-store-data-scenarios/ingest-data.png "Ingérer des données dans Data Lake Storage Gen1")

### <a name="ad-hoc-data"></a>Données ad hoc
Ceci représente les petits jeux de données qui sont utilisés pour créer un prototype d’une application de Big Data. Il existe différentes façons de recevoir des données ad hoc en fonction de la source de données.

| source de données | Réception avec |
| --- | --- |
| Ordinateur local |<ul> <li>[Azure portal](data-lake-store-get-started-portal.md)</li> <li>[Azure PowerShell](data-lake-store-get-started-powershell.md)</li> <li>[Azure CLI](data-lake-store-get-started-cli-2.0.md)</li> <li>[Utilisation de Data Lake Tools pour Visual Studio](../data-lake-analytics/data-lake-analytics-data-lake-tools-get-started.md) </li></ul> |
| Azure Storage Blob |<ul> <li>[Azure Data Factory](../data-factory/connector-azure-data-lake-store.md).</li> <li>[l’outil AdlCopy](data-lake-store-copy-data-azure-storage-blob.md)</li><li>[DistCp en cours d’exécution sur un cluster HDInsight](data-lake-store-copy-data-wasb-distcp.md)</li> </ul> |

### <a name="streamed-data"></a>Flux de données
Ceci représente les données qui peuvent être générées par diverses sources, telles que des applications, des appareils, des capteurs, etc. Ces données peuvent être ingérées dans Data Lake Storage Gen1 par des outils divers. En général, ces outils capturent et traitent les données sur la base de l’événement en temps réel, puis ils écrivent les événements par lots dans Data Lake Storage Gen1 afin qu’ils puissent être traités.

Voici les outils que vous pouvez utiliser :

* [Azure Stream Analytics](../stream-analytics/stream-analytics-define-outputs.md) : les événements ingérés dans Event Hubs peuvent être écrits dans Azure Data Lake Storage Gen1 à l’aide d’une sortie Azure Data Lake Storage Gen1.
* [Azure HDInsight Storm](../hdinsight/storm/apache-storm-write-data-lake-store.md) : vous pouvez écrire des données directement dans Data Lake Storage Gen1 à partir du cluster Storm.
* [EventProcessorHost](../event-hubs/event-hubs-dotnet-standard-getstarted-send.md) : vous pouvez recevoir des événements à partir d’Event Hubs, puis les écrire dans Data Lake Storage Gen1 à l’aide du [SDK .NET Data Lake Storage Gen1](data-lake-store-get-started-net-sdk.md).

### <a name="relational-data"></a>Données relationnelles
Les bases de données relationnelles peuvent également être utilisées comme sources des données. Sur une période donnée, les bases de données relationnelles collectent de grandes quantités de données qui peuvent fournir des informations clés si elles sont traitées via un pipeline de Big Data. Vous pouvez utiliser les outils suivants pour déplacer ces données vers Data Lake Storage Gen1.

* [Apache Sqoop](data-lake-store-data-transfer-sql-sqoop.md)
* [Azure Data Factory](../data-factory/copy-activity-overview.md).

### <a name="web-server-log-data-upload-using-custom-applications"></a>Données de journal de serveur web (téléchargement à l’aide d’applications personnalisées)
Ce type de jeu de données est spécifiquement indiqué, car l’analyse des données de journal de serveur web constitue un cas d’usage courant pour les applications de Big Data et nécessite le chargement d’importants volumes de fichiers journaux sur Data Lake Storage Gen1. Vous pouvez utiliser les outils suivants pour écrire vos propres scripts ou applications pour télécharger ces données.

* [Azure CLI](data-lake-store-get-started-cli-2.0.md)
* [Azure PowerShell](data-lake-store-get-started-powershell.md)
* [SDK .NET Azure Data Lake Storage Gen1](data-lake-store-get-started-net-sdk.md)
* [Azure Data Factory](../data-factory/copy-activity-overview.md).

Pour télécharger des données de journal de serveur web, et également pour télécharger d’autres types de données (par exemple, les données relatives aux sentiments sociaux), il est préférable d’écrire vos propres scripts/applications personnalisés, car cela vous donne la possibilité d’inclure votre composant de téléchargement de données dans le cadre de votre application de Big Data plus étendue. Dans certains cas, ce code peut prendre la forme d’un script ou d’un utilitaire de ligne de commande simple. Dans d’autres cas, le code peut être utilisé pour intégrer le traitement de Big Data dans une solution ou une application métier.

### <a name="data-associated-with-azure-hdinsight-clusters"></a>Données associées aux clusters Azure HDInsight
La plupart des types de clusters HDInsight (Hadoop, HBase, Storm) prennent en charge Data Lake Storage Gen1 comme référentiel de stockage des données. Les clusters HDInsight accèdent aux données à partir des objets blob d’Azure Storage (WASB). Pour optimiser les performances, vous pouvez copier les données à partir de WASB sur un compte Data Lake Storage Gen1 associé au cluster. Vous pouvez utiliser les outils suivants pour copier les données.

* [Apache DistCp](data-lake-store-copy-data-wasb-distcp.md)
* [AdlCopy Service](data-lake-store-copy-data-azure-storage-blob.md)
* [Azure Data Factory](../data-factory/connector-azure-data-lake-store.md).

### <a name="data-stored-in-on-premises-or-iaas-hadoop-clusters"></a>Données stockées localement ou dans des clusters IaaS Hadoop
De grandes quantités de données peuvent être stockées dans des clusters Hadoop existants, localement sur les ordinateurs à l’aide de HDFS. Les clusters Hadoop peuvent être inclus dans un déploiement local ou au sein d’un cluster IaaS sur Azure. Il peut être nécessaire de copier ces données dans Azure Data Lake Storage Gen1 afin de bénéficier d’un accès unique ou périodique. Différentes options vous permettent d’y parvenir. Voici une liste de ces options et des compromis correspondants.

| Approche | Détails | Avantages | Considérations |
| --- | --- | --- | --- |
| Utiliser Azure Data Factory (ADF) pour copier des données directement à partir de clusters Hadoop dans Azure Data Lake Storage Gen1 |[ADF prend en charge HDFS comme source de données](../data-factory/connector-hdfs.md) |ADF offre une prise en charge immédiate de HDFS ainsi qu’une gestion et une surveillance de bout en bout de premier ordre |Nécessite que la passerelle de gestion des données soit déployée localement ou dans le cluster IaaS |
| Exportez les données depuis Hadoop sous forme de fichiers. Copiez ensuite les fichiers dans Azure Data Lake Storage Gen1 à l’aide du mécanisme approprié. |Vous pouvez copier des fichiers dans Azure Data Lake Storage Gen1 en utilisant : <ul><li>[Azure PowerShell pour système d'exploitation Windows](data-lake-store-get-started-powershell.md)</li><li>[Azure CLI](data-lake-store-get-started-cli-2.0.md)</li><li>Application personnalisée utilisant n’importe quel SDK Data Lake Storage Gen1</li></ul> |Pour commencer. Permet des téléchargements personnalisés |Processus en plusieurs étapes qui implique différentes technologies. La gestion et la surveillance deviendront de plus en plus exigeantes au fil du temps, étant donné le caractère personnalisé des outils |
| Utilisez Distcp pour copier les données de Hadoop dans Azure Storage. Copiez ensuite les fichiers de Stockage Azure vers Data Lake Storage Gen1 à l’aide du mécanisme approprié. |Vous pouvez copier les données de Stockage Azure vers Data Lake Storage Gen1 en utilisant : <ul><li>[Azure Data Factory](../data-factory/copy-activity-overview.md).</li><li>[l’outil AdlCopy](data-lake-store-copy-data-azure-storage-blob.md)</li><li>[pache DistCp en cours d’exécution sur des clusters HDInsight](data-lake-store-copy-data-wasb-distcp.md)</li></ul> |Vous pouvez utiliser des outils open source. |Processus en plusieurs étapes qui implique différentes technologies |

### <a name="really-large-datasets"></a>Jeux de données très volumineux
Pour télécharger des jeux de données qui comptent plusieurs téraoctets, l’utilisation des méthodes décrites ci-dessus peut parfois être lente et coûteuse. Dans ce cas, vous pouvez utiliser les options ci-dessous.

* **Utilisation d’Azure ExpressRoute**. Azure ExpressRoute vous permet de créer des connexions privées entre les infrastructures et les centres de données Azure dans votre environnement local. Ceci constitue une option fiable pour le transfert de grandes quantités de données. Pour plus d’informations, consultez la [Documentation Azure ExpressRoute](../expressroute/expressroute-introduction.md).
* **Téléchargement « hors connexion » des données**. Si vous ne pouvez pas utiliser Azure ExpressRoute pour une raison ou une autre, vous pouvez utiliser le [service Azure Import/Export](../storage/common/storage-import-export-service.md) pour expédier des disques durs contenant vos données à un centre de données Azure. Vos données sont alors téléchargées vers des objets blob Azure Storage. Vous pouvez ensuite utiliser [Azure Data Factory](../data-factory/connector-azure-data-lake-store.md) ou [l’outil AdlCopy](data-lake-store-copy-data-azure-storage-blob.md) pour copier des données des objets blob de stockage Azure vers Data Lake Storage Gen1.

  > [!NOTE]
  > Si vous utilisez le service Import/Export, la taille des fichiers sur les disques durs que vous envoyez au centre de données Azure ne doit pas être supérieure à 195 Go.
  >
  >

## <a name="process-data-stored-in-data-lake-storage-gen1"></a>Traiter des données stockées dans Data Lake Storage Gen1
Une fois que les données sont disponibles dans Data Lake Storage Gen1, vous pouvez exécuter une analyse sur ces données à l’aide des applications de Big Data prises en charge. Actuellement, vous pouvez utiliser Azure HDInsight et Azure Data Lake Analytics pour exécuter des tâches d’analyse des données sur les données stockées dans Data Lake Storage Gen1.

![Sécuriser les données dans Data Lake Storage Gen1](./media/data-lake-store-data-scenarios/analyze-data.png "Sécuriser les données dans Data Lake Storage Gen1")

Vous pouvez consulter les exemples suivants.

* [Créer un cluster HDInsight avec Data Lake Storage Gen1 comme stockage.](data-lake-store-hdinsight-hadoop-use-portal.md)
* [Utiliser Azure Data Lake Analytics avec Data Lake Storage Gen1](../data-lake-analytics/data-lake-analytics-get-started-portal.md)

## <a name="download-data-from-data-lake-storage-gen1"></a>Télécharger des données à partir de Data Lake Storage Gen1
Vous pouvez également être amené à télécharger ou à déplacer les données à partir d’Azure Data Lake Storage Gen1 dans certains cas, tels que :

* Déplacer des données vers d’autres référentiels pour créer une interface avec vos pipelines de traitement des données existantes. Par exemple, vous pourriez vouloir déplacer des données de Data Lake Storage Gen1 vers Azure SQL Database ou vers SQL Server.
* Télécharger des données sur votre ordinateur local pour le traitement dans des environnements IDE lors de la création de prototypes d’applications.

![Extraire des données de Data Lake Storage Gen1](./media/data-lake-store-data-scenarios/egress-data.png "Extraire des données de Data Lake Storage Gen1")

Dans ce cas, vous pouvez utiliser l’une des options suivantes :

* [Apache Sqoop](data-lake-store-data-transfer-sql-sqoop.md)
* [Azure Data Factory](../data-factory/copy-activity-overview.md).
* [Apache DistCp](data-lake-store-copy-data-wasb-distcp.md)

Vous pouvez également utiliser les méthodes suivantes pour écrire votre propre application/script pour télécharger des données à partir de Data Lake Storage Gen1.

* [Azure CLI](data-lake-store-get-started-cli-2.0.md)
* [Azure PowerShell](data-lake-store-get-started-powershell.md)
* [SDK .NET Azure Data Lake Storage Gen1](data-lake-store-get-started-net-sdk.md)

## <a name="visualize-data-in-data-lake-storage-gen1"></a>Visualiser des données dans Data Lake Storage Gen1
Vous pouvez utiliser une combinaison de services pour créer des représentations visuelles des données stockées dans Data Lake Storage Gen1.

![Visualiser des données dans Data Lake Storage Gen1](./media/data-lake-store-data-scenarios/visualize-data.png "Visualiser des données dans Data Lake Storage Gen1")

* Vous pouvez commencer par utiliser [Azure Data Factory pour déplacer les données de Data Lake Storage Gen1 vers Azure Synapse Analytics](../data-factory/copy-activity-overview.md).
* Ensuite, vous pouvez [intégrer Power BI à Azure Synapse Analytics](/power-bi/connect-data/service-azure-sql-data-warehouse-with-direct-connect) pour créer une représentation visuelle des données.
---
title: Déplacer des données de stockage d’objets blob avec des connecteurs SSIS - Team Data Science Process
description: Découvrez comment déplacer des données vers ou à partir du Stockage Blob Azure avec SQL Server Integration Services Feature Pack pour Azure.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 9572d612e7ee8e2fd72850ba14447e8449f0f371
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93322000"
---
# <a name="move-data-to-or-from-azure-blob-storage-using-ssis-connectors"></a>Déplacer des données vers ou depuis Stockage Blob Azure à l’aide de connecteurs SSIS
Le [Feature Pack SQL Server Integration Services pour Azure](/sql/integration-services/azure-feature-pack-for-integration-services-ssis) fournit des composants permettant de se connecter à Azure, de transférer des données entre des sources de données Azure et des sources de données sur site et de traiter les données stockées dans Azure.

[!INCLUDE [blob-storage-tool-selector](../../../includes/machine-learning-blob-storage-tool-selector.md)]

Une fois que les clients ont déplacé des données locales vers le cloud, ils peuvent y accéder à partir de tout service Azure pour tirer pleinement parti de la puissance de la suite de technologies Azure. Ils peuvent ensuite utiliser les données, par exemple, dans Azure Machine Learning ou sur un cluster HDInsight.

Vous trouverez des exemples d’utilisation de ces ressources Azure dans les procédures pas à pas [SQL](sql-walkthrough.md) et [HDInsight](hive-walkthrough.md).

Pour une présentation de scénarios canoniques utilisant SSIS pour répondre aux besoins de l’entreprise communs dans les scénarios d’intégration hybrides, reportez-vous à la section [En faire plus avec le Feature Pack SQL Server Integration Services pour Azure](https://techcommunity.microsoft.com/t5/sql-server-integration-services/doing-more-with-sql-server-integration-services-feature-pack-for/ba-p/388238) .

> [!NOTE]
> Pour une présentation complète du stockage d’objets blob Azure, consultez les articles [Fonctionnalités de base des objets blob Azure](../../storage/blobs/storage-quickstart-blobs-dotnet.md) et [Service Blob Azure](/rest/api/storageservices/Blob-Service-Concepts).
> 
> 

## <a name="prerequisites"></a>Prérequis
Pour exécuter les tâches décrites dans cet article, vous devez disposer d’un abonnement Azure et d’un compte de stockage Azure. Pour charger ou télécharger des données, vous devez connaître le nom et la clé du compte de stockage Azure.

* Pour configurer un **abonnement Azure** , consultez la section [Essai gratuit pendant un mois](https://azure.microsoft.com/pricing/free-trial/).
* Pour obtenir des instructions sur la création d’ **un compte de stockage** et des informations sur le compte et la clé, voir [À propos des comptes de stockage Azure](../../storage/common/storage-account-create.md).

Pour utiliser les **connecteurs SSIS** vous devez télécharger :

* **SQL Server 2014 ou 2016 Standard (ou version ultérieure)** : L’installation inclut SQL Server Integration Services.
* **Microsoft SQL Server 2014 ou 2016 Integration Services Feature Pack pour Azure** : Vous pouvez télécharger ces connecteurs respectivement depuis les pages [SQL Server 2014 Integration Services](https://www.microsoft.com/download/details.aspx?id=47366) et [SQL Server 2016 Integration Services](https://www.microsoft.com/download/details.aspx?id=49492).

> [!NOTE]
> SSIS est installé avec SQL Server, mais n’est pas inclus dans la version Express. Pour plus d'informations sur les applications incluses dans les différentes éditions de SQL Server, consultez [Éditions de SQL Server](https://www.microsoft.com/en-us/server-cloud/products/sql-server-editions/)
> 
> 

Pour les supports de formation sur SSIS, consultez la [formation pratique SSIS](https://www.microsoft.com/sql-server/training-certification)

Pour plus d’informations sur la mise en service à l'aide de SISS pour la création des packages d'extraction, de transformation et de chargement (ETL), consultez [Didacticiel SSIS : création d’un Package ETL Simple](/sql/integration-services/ssis-how-to-create-an-etl-package).

## <a name="download-nyc-taxi-dataset"></a>Télécharger l’ensemble de données Taxi NYC
L'exemple décrit ici utilise un groupe de données disponible au public (l'ensemble de données [Courses taxi NYC](https://www.andresmh.com/nyctaxitrips/) ). Le groupe de données se compose d’environ 173 millions de courses en taxi à New York dans l’année 2013. Il existe deux types de données : les données détaillées relatives aux voyages et celles relatives aux tarifs des courses. Comme il y a un fichier pour chaque mois, nous avons 24 fichiers, chacun d’environ 2 Go non compressé.

## <a name="upload-data-to-azure-blob-storage"></a>Téléchargement de données vers le stockage d’objets blob Azure
Pour déplacer des données à l’aide du pack de fonctionnalités du site local pour le stockage d’objets blobs Azure, nous utilisons une instance de la [**tâche de téléchargement d’objets blobs Azure**](/sql/integration-services/control-flow/azure-blob-upload-task), comme illustré ici :

![configure-data-science-vm](./media/move-data-to-azure-blob-using-ssis/ssis-azure-blob-upload-task.png)

Les paramètres que la tâche utilise sont décrits ici :

| Champ | Description |
| --- | --- |
| **AzureStorageConnection** |Spécifie un Gestionnaire de connexions de Stockage Azure ou en crée un faisant référence à un compte de stockage Azure pointant vers l’emplacement où les fichiers blob sont hébergés. |
| **BlobContainer** |Spécifie le nom du conteneur d’objets blobs dans lequel se trouvent les fichiers chargés en tant qu’objets blobs. |
| **BlobDirectory** |Spécifie le répertoire d’objets blob dans lequel le fichier chargé est stocké en tant qu’objet blob de bloc. Le répertoire d’objet blob est une structure hiérarchique virtuelle. Si l’objet blob existe déjà, il est remplacé. |
| **LocalDirectory** |Spécifie le répertoire local qui contient les fichiers à télécharger. |
| **FileName** |Spécifie un filtre de nom pour sélectionner des fichiers obéissant à un schéma de nom spécifié. Par exemple, MySheet\*.xls\* inclut des fichiers tels que MySheet001.xls et MySheetABC.xlsx |
| **TimeRangeFrom/TimeRangeTo** |Spécifie une plage de temps pour appliquer un filtre. Les fichiers modifiés après *TimeRangeFrom* et avant *TimeRangeTo* sont inclus. |

> [!NOTE]
> Les informations d’identification **AzureStorageConnection** doivent être correctes et **BlobContainer** doit exister avant que le transfert soit tenté.
> 
> 

## <a name="download-data-from-azure-blob-storage"></a>Télécharger les données depuis le stockage d’objets blobs Azure
Pour télécharger des données depuis le stockage d’objets blob Azure vers le stockage sur site avec SSIS, utilisez une instance de la [tâche de téléchargement d’objets blob Azure](/sql/integration-services/control-flow/azure-blob-download-task).

## <a name="more-advanced-ssis-azure-scenarios"></a>Scénarios SSIS-Azure plus élaborés
Le pack de fonctionnalités SSIS permet de gérer des flux de données plus complexes en regroupant plusieurs tâches. Par exemple, les données d’objets blobs peuvent alimenter directement un cluster HDInsight dont le résultat peut être de nouveau téléchargé vers un objet blob, puis dans un espace de stockage local. SSIS peut exécuter des tâches Hive et Pig sur un cluster HDInsight à l’aide de connecteurs SSIS supplémentaires :

* Pour exécuter un script Hive sur un cluster Azure HDInsight avec SSIS, utilisez la [tâche Azure HDInsight Hive](/sql/integration-services/control-flow/azure-hdinsight-hive-task).
* Pour exécuter un script Pig sur un cluster Azure HDInsight avec SSIS, utilisez la [tâche Azure HDInsight Pig](/sql/integration-services/control-flow/azure-hdinsight-pig-task).
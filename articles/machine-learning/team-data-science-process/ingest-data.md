---
title: Charger des données dans des environnements de Stockage Azure – Processus TDSP (Team Data Science Process)
description: Découvrez comment ingérer des données dans divers environnements cibles où les données sont stockées et traitées.
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
ms.openlocfilehash: 46e911ad6bdf5a478d46e425f8700740ece56c6e
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452662"
---
# <a name="load-data-into-storage-environments-for-analytics"></a>Charger des données dans des environnements de stockage à des fins d’analyse

Le processus TDSP (Team Data Science Process) exige que les données soient ingérées ou chargées de la manière la plus appropriée à chaque étape. Les destinations de données peuvent inclure le Stockage Blob Azure, les bases de données SQL Azure, SQL Server sur une machine virtuelle Azure, HDInsight (Hadoop), Azure Synapse Analytics et Azure Machine Learning. 

Les articles suivantes expliquent comment recevoir les données dans les environnements cibles où les données sont stockées et traitées.

* Vers/À partir d’[Azure Blob Storage](move-azure-blob.md)
* Vers [SQL Server dans les machines virtuelles Azure](move-sql-server-virtual-machine.md)
* Vers [Azure SQL Database](move-sql-azure.md)
* Vers les [tables Hive](move-hive-tables.md)
* Vers les [tables partitionnées SQL](parallel-load-sql-partitioned-tables.md)
* À partir du [Serveur SQL Server local](move-sql-azure-adf.md)

Les besoins techniques et métier, ainsi que l’emplacement initial, le format et la taille de vos données, déterminent le meilleur plan d’ingestion des données. Il n’est pas rare qu’un plan optimal comporte plusieurs étapes. Cette suite de tâches peut par exemple inclure l’exploration des données, leur prétraitement, leur nettoyage, l’échantillonnage et l’apprentissage du modèle.  Azure Data Factory est une ressource Azure recommandée pour orchestrer le déplacement et la transformation des données.

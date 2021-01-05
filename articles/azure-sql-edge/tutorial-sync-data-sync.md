---
title: Synchroniser des données à partir d'Azure SQL Edge à l'aide de SQL Data Sync
description: En savoir plus sur la synchronisation de données à partir d'Azure SQL Edge à l'aide d'Azure SQL Data Sync
keywords: SQL Edge, synchroniser des données à partir de SQL Edge, synchronisation de données SQL Edge
services: sql-edge
ms.service: sql-edge
ms.topic: tutorial
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2020
ms.openlocfilehash: 5659ce5fa6f99463f58a33662563d768248fd8cb
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93394898"
---
# <a name="tutorial-sync-data-from-sql-edge-to-azure-sql-database-by-using-sql-data-sync"></a>Tutoriel : Synchroniser des données de SQL Edge sur Azure SQL Database en utilisant SQL Data Sync

Ce tutoriel explique comment utiliser un *groupe de synchronisation* Azure SQL Data Sync pour synchroniser de façon incrémentielle des données d’Azure SQL Edge vers Azure SQL Database. SQL Data Sync est un service conçu sur Azure SQL Database qui vous permet de synchroniser les données choisies de manière bidirectionnelle sur plusieurs bases de données dans les instances Azure SQL Database et SQL Server. Pour plus d’informations sur SQL Data Sync, consultez [Azure SQL Data Sync](../azure-sql/database/sql-data-sync-data-sql-server-sql-database.md).

Azure SQL Edge étant construit sur les dernières versions du [moteur de base de données SQL Server](/sql/sql-server/sql-server-technical-documentation/), tout mécanisme de synchronisation de données applicable à une instance SQL Server peut également être utilisé pour synchroniser des données avec une instance SQL Edge exécutée sur un appareil périphérique.

## <a name="prerequisites"></a>Prérequis

Ce tutoriel nécessite un ordinateur Windows configuré avec [Data Sync Agent pour Azure SQL Data Sync](../azure-sql/database/sql-data-sync-agent-overview.md).

## <a name="before-you-begin"></a>Avant de commencer

* Créez une base de données dans Azure SQL Database. Pour plus d’informations sur la création d’une base de données à l’aide du portail Azure, consultez [Créer une base de données unique dans Azure SQL Database](../azure-sql/database/single-database-create-quickstart.md?tabs=azure-portal).

* Créez les tables et autres objets nécessaires dans votre déploiement Azure SQL Database.

* Créez les tables et objets nécessaires dans votre déploiement Azure SQL Edge. Pour plus d’informations, consultez [Utilisation de packages DAC SQL Database avec SQL Edge](deploy-dacpac.md).

* Inscrivez l’instance Azure SQL Edge auprès de Data Sync Agent pour Azure SQL Data Sync. Pour plus d’informations, consultez [Ajouter une base de données SQL Server](../azure-sql/database/sql-data-sync-sql-server-configure.md#add-on-prem).

## <a name="sync-data-between-a-database-in-azure-sql-database-and-sql-edge"></a>Synchroniser des données entre une base de données dans Azure SQL Database et SQL Edge

La configuration de la synchronisation entre une base de données dans Azure SQL Database et une instance SQL Edge à l’aide de SQL Data Sync implique trois étapes clés :  


1. Utilisez le portail Azure pour créer un groupe de synchronisation. Pour plus d’informations, consultez [Créer un groupe de synchronisation](../azure-sql/database/sql-data-sync-sql-server-configure.md#create-sync-group). Vous pouvez utiliser une seule base de données *Hub* pour créer plusieurs groupes de synchronisation afin de synchroniser les données de différentes instances SQL Edge avec une ou plusieurs bases de données dans Azure SQL Database. 

2. Ajoutez des membres de synchronisation au groupe de synchronisation. Pour plus d’informations, consultez [Ajouter des membres de synchronisation](../azure-sql/database/sql-data-sync-sql-server-configure.md#add-sync-members).

3. Configurez le groupe de synchronisation pour sélectionner les tables à ajouter à cette synchronisation. Pour plus d'informations, consultez [Configurer un groupe de synchronisation](../azure-sql/database/sql-data-sync-sql-server-configure.md#add-sync-members).

Une fois les étapes précédentes effectuées, vous disposez d’un groupe de synchronisation comprenant une base de données dans Azure SQL Database et une instance SQL Edge.

Pour plus d’informations sur SQL Data Sync, consultez ces articles :

* [Data Sync Agent pour Azure SQL Data Sync](../azure-sql/database/sql-data-sync-agent-overview.md)

* [Bonnes pratiques](../azure-sql/database/sql-data-sync-best-practices.md) et [Comment résoudre les problèmes liés à Azure SQL Data Sync](../azure-sql/database/sql-data-sync-troubleshoot.md)

* [Superviser SQL Data Sync avec des journaux Azure Monitor](../azure-sql/database/monitor-tune-overview.md)

* [Mettre à jour le schéma de synchronisation avec Transact-SQL](../azure-sql/database/sql-data-sync-update-sync-schema.md) ou [PowerShell](../azure-sql/database/scripts/update-sync-schema-in-sync-group.md)

## <a name="next-steps"></a>Étapes suivantes


* [Utiliser PowerShell pour synchroniser Azure SQL Database avec Azure SQL Edge](../azure-sql/database/scripts/sql-data-sync-sync-data-between-azure-onprem.md). Dans ce tutoriel, remplacez les détails de la base de données `OnPremiseServer` par ceux d’Azure SQL Edge.
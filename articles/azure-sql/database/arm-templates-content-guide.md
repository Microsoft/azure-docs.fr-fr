---
title: Modèles Azure Resource Manager - Azure SQL Database et Azure SQL Managed Instance
description: Utilisez des modèles Azure Resource Manager pour créer et configurer Azure SQL Database et Azure SQL Managed Instance.
services: sql-database
ms.service: sql-db-mi
ms.subservice: service
ms.custom: overview-samples sqldbrb=2
ms.devlang: ''
ms.topic: guide
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein
ms.date: 02/04/2019
ms.openlocfilehash: 5202145af6f60f34bdce57aa94f4fec92a8b227f
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92677267"
---
# <a name="azure-resource-manager-templates-for-azure-sql-database--sql-managed-instance"></a>Modèles Azure Resource Manager pour Azure SQL Database et Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Les modèles Azure Resource Manager vous permettent de définir votre infrastructure sous forme de code et de déployer vos solutions sur le cloud Azure pour Azure SQL Database et Azure SQL Managed Instance.

## <a name="azure-sql-database"></a>[Azure SQL Database](#tab/single-database)

Le tableau suivant inclut des liens vers des modèles Azure Resource Manager pour Azure SQL Database.

|Lien |Description|
|---|---|
| [Base de données SQL](https://github.com/Azure/azure-quickstart-templates/tree/master/201-sql-database-transparent-encryption-create) | Ce modèle Azure Resource Manager crée une base de données unique dans Azure SQL Database et configure des règles de pare-feu IP au niveau du serveur. |
| [Serveur](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sql-logical-server) | Ce modèle Azure Resource Manager crée un serveur pour Azure SQL Database. |
| [Pool élastique](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sql-elastic-pool-create) | Ce modèle vous permet de déployer un pool élastique et d’y affecter des bases de données. |
| [Groupes de basculement](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sql-with-failover-group) | Ce modèle crée deux serveurs, une base de données unique et un groupe de basculement dans Azure SQL Database.|
| [Détection de menaces](https://github.com/Azure/azure-quickstart-templates/tree/master/201-sql-threat-detection-db-policy-multiple-databases) | Ce modèle vous permet de déployer un serveur et un ensemble de bases de données avec la détection des menaces activée ainsi qu’une adresse e-mail pour les alertes de chaque base de données. La détection de menaces fait partie de l’offre SQL Advanced Threat Protection (ATP) qui fournit une couche de sécurité en réponse aux menaces potentielles qui pèsent sur les bases de données et les serveurs.|
| [Audit dans Stockage Blob Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/201-sql-auditing-server-policy-to-blob-storage) | Ce modèle vous permet de déployer un serveur avec audit activé pour écrire des journaux d’audit dans un stockage d’objets blob. L’audit pour Azure SQL Database suit les événements de base de données et les écrit dans un journal d’audit que vous pouvez placer dans votre compte de stockage Azure, votre espace de travail OMS ou Event Hubs.|
| [Audit pour Azure Event Hub](https://github.com/Azure/azure-quickstart-templates/tree/master/201-sql-auditing-server-policy-to-eventhub) | Ce modèle vous permet de déployer un serveur avec audit activé pour écrire des journaux d’audit dans un hub d’événements existant. Pour envoyer des événements d’audit à Event Hubs, définissez des paramètres d’audit avec `Enabled` `State`, et affectez à `IsAzureMonitorTargetEnabled` la valeur `true`. Configurez aussi des paramètres de diagnostic avec la catégorie de journal `SQLSecurityAuditEvents` sur la base de données `master` (pour l’audit au niveau du serveur). L’audit suit les événements de base de données et les écrit dans un journal d’audit que vous pouvez placer dans votre compte de stockage Azure, votre espace de travail OMS ou Event Hubs.|
| [Application web Azure avec SQL Database](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-sql-database) | Cet exemple crée une application web Azure gratuite et une base de données dans Azure SQL Database au niveau de service « De base ».|
| [Application web Azure et Cache Redis avec SQL Database](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-redis-cache-sql-database) | Ce modèle crée une application web, un cache Redis et une base de données dans le même groupe de ressources, puis il crée deux chaînes de connexion dans l’application web pour la base de données et le cache Redis.|
| [Importer des données à partir de Stockage Blob avec ADF V2](https://github.com/Azure/azure-quickstart-templates/tree/master/101-data-factory-v2-blob-to-sql-copy) | Ce modèle Azure Resource Manager crée une instance d’Azure Data Factory V2 qui copie des données de Stockage Blob Azure vers SQL Database.|
| [Cluster HDInsight avec une base de données](https://github.com/Azure/azure-quickstart-templates/tree/master/101-hdinsight-linux-with-sql-database) | Ce modèle vous permet de créer un cluster HDInsight, un serveur SQL logique, une base de données et deux tables. Il est utilisé dans l’article [Utiliser Sqoop avec Hadoop dans HDInsight](../../hdinsight/hadoop/hdinsight-use-sqoop.md). |
| [Application logique Azure qui exécute une procédure stockée SQL selon une planification](https://github.com/Azure/azure-quickstart-templates/tree/master/101-logic-app-sql-proc) | Ce modèle vous permet de créer une application logique qui exécute une procédure stockée SQL selon une planification. Tous les arguments relatifs à la procédure peuvent être placés dans la section du corps du modèle.|

## <a name="azure-sql-managed-instance"></a>[Azure SQL Managed Instance](#tab/managed-instance)

Le tableau suivant inclut des liens vers des modèles Azure Resource Manager pour Azure SQL Managed Instance.

|Lien|Description|
|---|---|
| [SQL Managed Instance dans un nouveau réseau virtuel](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sqlmi-new-vnet) | Ce modèle Azure Resource Manager crée un réseau virtuel Azure configuré et une instance managée sur ce réseau virtuel. |
| [Environnement réseau pour instance managée SQL](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sql-managed-instance-azure-environment) | Ce déploiement crée un réseau virtuel Azure configuré avec deux sous-réseaux : un dédié à vos instances managées et un autre sur lequel vous pouvez placer d’autres ressources (par exemple des machines virtuelles, des environnements App Service, et ainsi de suite). Ce modèle crée un environnement réseau correctement configuré, où vous pouvez déployer des instances managées. |
| [Instance managée SQL avec connexion P2S](https://github.com/Azure/azure-quickstart-templates/tree/master/201-sqlmi-new-vnet-w-point-to-site-vpn) | Ce déploiement crée un réseau virtuel Azure avec deux sous-réseaux, `ManagedInstance` et `GatewaySubnet`. SQL Managed Instance est déployée sur le sous-réseau ManagedInstance. Une passerelle de réseau virtuel est créée sur le sous-réseau `GatewaySubnet` et configurée pour la connexion VPN point à site. |
| [SQL Managed Instance avec une machine virtuelle](https://github.com/Azure/azure-quickstart-templates/tree/master/201-sqlmi-new-vnet-w-jumpbox) | Ce déploiement crée un réseau virtuel Azure avec deux sous-réseaux, `ManagedInstance` et `Management`. SQL Managed Instance est déployée sur le sous-réseau `ManagedInstance`. Une machine virtuelle dotée de la dernière version de SQL Server Management Studio (SSMS) est déployée sur le sous-réseau `Management`. |

---
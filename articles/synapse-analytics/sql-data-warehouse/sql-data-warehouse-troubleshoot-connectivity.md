---
title: Résoudre les problèmes de connectivité
description: Résolution des problèmes de connectivité dans le pool SQL dédié (anciennement SQL DW).
services: synapse-analytics
author: anumjs
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 03/27/2019
ms.author: anjangsh
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse, devx-track-csharp
ms.openlocfilehash: ea99c2ce1963ec58649fd4c2fbb4d98768da8c6f
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96447947"
---
# <a name="troubleshooting-connectivity-issues-in-dedicated-sql-pool-formerly-sql-dw"></a>Résolution des problèmes de connectivité dans le pool SQL dédié (anciennement SQL DW)

Cet article répertorie les techniques courantes de résolution des problèmes relatifs à la connexion à votre base de données du pool SQL dédié (anciennement SQL DW).

## <a name="check-service-availability"></a>Vérifier la disponibilité du service

Vérifiez si le service est disponible. Dans le portail Azure, accédez au pool SQL dédié (anciennement SQL DW) auquel vous essayez de vous connecter. Dans le panneau Table des matières à gauche, cliquez sur **Diagnostiquer et résoudre les problèmes**.

![Sélectionner l’intégrité des ressources](./media/sql-data-warehouse-troubleshoot-connectivity/diagnostics-link.png)

L’état de votre pool SQL dédié (anciennement SQL DW) s’affiche ici. Si le service ne s’affiche pas comme étant **disponible**, vérifiez d’autres étapes.

![Service disponible](./media/sql-data-warehouse-troubleshoot-connectivity/resource-health.png)

Si l’intégrité de vos ressources indique que votre instance de pool SQL dédié (anciennement SQL DW) est interrompue ou mise à l’échelle, suivez les instructions permettant de reprendre l’exécution de votre instance.

![Capture d’écran montrant une instance de pool SQL dédié interrompue ou mise à l’échelle.](./media/sql-data-warehouse-troubleshoot-connectivity/resource-health-pausing.png)
Des informations supplémentaires relatives à l’intégrité des ressources sont disponibles ici.

## <a name="check-for-paused-or-scaling-operation"></a>Vérifier les opérations de mise à l’échelle ou interrompues

Dans le portail, vérifiez si votre instance de pool SQL dédié (anciennement SQL DW) est interrompue ou mise à l’échelle.

![Capture d’écran montrant comment vérifier si un entrepôt de données est suspendu.](./media/sql-data-warehouse-troubleshoot-connectivity/overview-paused.png)

Si vous constatez que votre service est interrompu ou mis à l’échelle, vérifiez que ce n’est pas durant le programme de maintenance. Dans le portail, pour la *Vue d’ensemble* de votre pool SQL dédié (anciennement SQL DW), vous voyez la planification de maintenance sélectionnée.

![Vue d’ensemble du programme de maintenance](./media/sql-data-warehouse-troubleshoot-connectivity/overview-maintance-schedule.png)

Sinon, contactez votre administrateur informatique pour vérifier que cette opération de maintenance n’est pas un événement planifié. Pour reprendre l’exécution de l’instance du pool SQL dédié (anciennement SQL DW), suivez [ces étapes](pause-and-resume-compute-portal.md).

## <a name="check-your-firewall-settings"></a>Vérifier les paramètres de pare-feu

La base de données du pool SQL dédié (anciennement SQL DW) communique par le biais du port 1433.  Si vous essayez de vous connecter à partir d’un réseau d’entreprise, le trafic sortant sur le port 1433 peut être bloqué par le pare-feu de votre réseau. Dans ce cas, vous ne pouvez pas vous connecter à votre [serveur logique](../../azure-sql/database/logical-servers.md), sauf si votre service informatique ouvre le port 1433. Vous trouverez plus d’informations sur les configurations de pare-feu [ici](../../azure-sql/database/firewall-configure.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#create-and-manage-ip-firewall-rules).

## <a name="check-your-vnetservice-endpoint-settings"></a>Vérifiez les paramètres de point de terminaison de service/du réseau virtuel

Si vous recevez des erreurs 40914 et 40615, consultez la documentation [Résolution et description de l’erreur ici](../../azure-sql/database/vnet-service-endpoint-rule-overview.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#errors-40914-and-40615).

## <a name="check-for-the-latest-drivers"></a>Vérifier les derniers pilotes

### <a name="software"></a>Logiciel

Vérifiez que vous utilisez bien les outils les plus récents pour vous connecter à votre pool SQL dédié (anciennement SQL DW) :

- SSMS
- Azure Data Studio
- SQL Server Data Tools (Visual Studio)

### <a name="drivers"></a>Pilotes

Vérifiez que vous disposez des dernières versions des pilotes.  L’utilisation d’une version antérieure des pilotes peut entraîner des comportements inattendus car les anciens pilotes peuvent ne pas en charge les nouvelles fonctionnalités.

- [ODBC](/sql/connect/odbc/download-odbc-driver-for-sql-server?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [JDBC](/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [OLE DB](/sql/connect/oledb/download-oledb-driver-for-sql-server?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [PHP](/sql/connect/php/download-drivers-php-sql-server?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)

## <a name="check-your-connection-string"></a>Vérifier la chaîne de connexion

Vérifiez que vos chaînes de connexion sont correctement définies.  Vous trouverez quelques exemples ci-dessous.  Vous trouverez des informations supplémentaires concernant les [chaînes de connexion ici](sql-data-warehouse-connection-strings.md).

Chaîne de connexion ADO.NET

```csharp
Server=tcp:{your_server}.database.windows.net,1433;Database={your_database};User ID={your_user_name};Password={your_password_here};Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
```

Chaîne de connexion ODBC

```csharp
Driver={SQL Server Native Client 11.0};Server=tcp:{your_server}.database.windows.net,1433;Database={your_database};Uid={your_user_name};Pwd={your_password_here};Encrypt=yes;TrustServerCertificate=no;Connection Timeout=30;
```

Chaîne de connexion PHP

```PHP
Server: {your_server}.database.windows.net,1433 \r\nSQL Database: {your_database}\r\nUser Name: {your_user_name}\r\n\r\nPHP Data Objects(PDO) Sample Code:\r\n\r\ntry {\r\n   $conn = new PDO ( \"sqlsrv:server = tcp:{your_server}.database.windows.net,1433; Database = {your_database}\", \"{your_user_name}\", \"{your_password_here}\");\r\n    $conn->setAttribute( PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION );\r\n}\r\ncatch ( PDOException $e ) {\r\n   print( \"Error connecting to SQL Server.\" );\r\n   die(print_r($e));\r\n}\r\n\rSQL Server Extension Sample Code:\r\n\r\n$connectionInfo = array(\"UID\" => \"{your_user_name}\", \"pwd\" => \"{your_password_here}\", \"Database\" => \"{your_database}\", \"LoginTimeout\" => 30, \"Encrypt\" => 1, \"TrustServerCertificate\" => 0);\r\n$serverName = \"tcp:{your_server}.database.windows.net,1433\";\r\n$conn = sqlsrv_connect($serverName, $connectionInfo);
```

Chaîne de connexion JDBC

```Java
jdbc:sqlserver://yourserver.database.windows.net:1433;database=yourdatabase;user={your_user_name};password={your_password_here};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;
```

## <a name="intermittent-connection-issues"></a>Problèmes de connexion intermittente

Vérifiez si vous constatez une charge importante sur le serveur avec un grand nombre de requêtes en file d’attente. Vous devrez peut-être effectuer un scale-up de votre pool SQL dédié (anciennement SQL DW) pour obtenir des ressources supplémentaires.

## <a name="common-error-messages"></a>Messages d’erreur courants

Erreurs 40914 et 40615, consultez la documentation [Résolution et description de l’erreur ici](../../azure-sql/database/vnet-service-endpoint-rule-overview.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#errors-40914-and-40615).

## <a name="still-having-connectivity-issues"></a>Vous rencontrez toujours des problèmes de connectivité ?

Créez un [ticket de support](sql-data-warehouse-get-started-create-support-ticket.md) afin d’être assisté par l’équipe d’ingénierie.

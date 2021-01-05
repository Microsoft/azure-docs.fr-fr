---
title: Protection avancée contre les menaces
titleSuffix: Azure SQL Database, SQL Managed Instance, & Azure Synapse Analytics
description: Advanced Threat Protection détecte les activités anormales des bases de données révélant des menaces potentielles pour la sécurité dans Azure SQL Database, Azure SQL Managed Instance et Azure Synapse Analytics.
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.devlang: ''
ms.custom: sqldbrb=2
ms.topic: conceptual
author: monhaber
ms.author: ronmat
ms.reviewer: vanto, sstein
ms.date: 12/01/2020
tags: azure-synapse
ms.openlocfilehash: 931e914cd3c184136395a9bb9a7e148a90e9fb91
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96461926"
---
# <a name="advanced-threat-protection-for-azure-sql-database-sql-managed-instance-and-azure-synapse-analytics"></a>Advanced Threat Protection pour Azure SQL Database, SQL Managed Instance et Azure Synapse Analytics
[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

Advanced Threat Protection pour [Azure SQL Database](sql-database-paas-overview.md), [Azure SQL Managed Instance](../managed-instance/sql-managed-instance-paas-overview.md) et [Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md) détecte les activités anormales révélant des tentatives inhabituelles et potentiellement dangereuses d'accès ou d'exploitation des bases de données.

Advanced Threat Protection fait partie de l’offre [Azure Defender pour SQL](azure-defender-for-sql.md). Il s’agit d’un package unifié pour les fonctionnalités de sécurité SQL avancées. Advanced Threat Protection est accessible et peut être géré par le biais du portail Azure Defender pour SQL central.

## <a name="overview"></a>Vue d’ensemble

Advanced Threat Protection fournit une nouvelle couche de sécurité qui permet aux clients de détecter des menaces potentielles et d’y réagir en temps réel en déclenchant des alertes de sécurité en lien avec des activités anormales. Les utilisateurs reçoivent une alerte en cas d’activités de base de données suspectes, de vulnérabilités potentielles, d’attaques par injection de code SQL et de modèles d’accès et de requêtes anormaux à la base de données. Advanced Threat Protection intègre des alertes à [Azure Security Center](https://azure.microsoft.com/services/security-center/), qui incluent des détails sur les activités suspectes et recommandent l’action à entreprendre pour analyser et prévenir la menace. Advanced Threat Protection vous permet de réagir facilement aux menaces potentielles visant la base de données sans devenir un expert en sécurité ni gérer des systèmes avancés de supervision de la sécurité.

Pour une expérience d'analyse complète, il est recommandé d'activer l'audit, qui inscrit les événements de la base de données dans un journal d'audit sur votre compte de stockage Azure.  Pour activer l'audit, consultez [Audit pour Azure SQL Database et Azure Synapse](../../azure-sql/database/auditing-overview.md) ou [Audit pour Azure SQL Managed Instance](../managed-instance/auditing-configure.md).

## <a name="alerts"></a>Alertes

Advanced Threat Protection pour Azure SQL Database détecte les activités anormales indiquant des tentatives d’accès ou d’exploitation inhabituelles ou potentiellement dangereuses de bases de données. Pour obtenir la liste des alertes pour Azure SQL Database, consultez [Alertes pour SQL Database et Azure Synapse Analytics dans Azure Security Center](../../security-center/alerts-reference.md#alerts-sql-db-and-warehouse).

## <a name="explore-detection-of-a-suspicious-event"></a>Explorer la détection d'un événement suspect

Vous recevez une notification par e-mail quand des activités anormales sont détectées sur la base de données. L’e-mail contient des informations sur l’événement de sécurité suspect, notamment la nature des activités anormales, le nom de la base de données, le nom du serveur, le nom de l’application et l’heure de l’événement. Il fournit également des informations sur les causes possibles et les mesures recommandées pour examiner et atténuer la menace potentielle pesant sur la base de données.

![Rapports d’activités anormales](./media/threat-detection-overview/anomalous_activity_report.png)

1. Cliquez sur le lien **Afficher les alertes SQL récentes** contenu dans l'e-mail pour ouvrir le portail Azure et accéder à la page d'alertes d'Azure Security Center, qui fournit une vue d'ensemble des menaces actives détectées sur la base de données.

   ![Menaces d’activité](./media/threat-detection-overview/active_threats.png)

1. Cliquez sur une alerte spécifique pour obtenir des détails supplémentaires et des actions permettant d’examiner cette menace et d’atténuer les menaces futures.

   Par exemple, l’injection SQL représente l’un des problèmes de sécurité les plus courants auxquels sont le plus exposées les applications web, et est utilisée pour cibler les applications pilotées par des données. Les pirates exploitent les vulnérabilités des applications pour injecter des instructions SQL nuisibles dans les champs de saisie d’application afin de violer ou modifier les données contenues dans la base de données. Pour les alertes d’injection SQL, les détails de l’alerte incluent l’instruction SQL vulnérable qui a été exploitée.

   ![Alerte spécifique](./media/threat-detection-overview/specific_alert.png)

## <a name="explore-alerts-in-the-azure-portal"></a>Explorer les alertes dans le portail Azure

Advanced Threat Protection intègre ses alertes dans [Azure Security Center](https://azure.microsoft.com/services/security-center/). Des vignettes Advanced Threat Protection SQL dans les panneaux de base de données et SQL Azure Defender au sein du portail Azure effectuent le suivi de l’état des menaces actives.

Cliquez sur **Alerte Advanced Threat Protection** pour ouvrir la page des alertes d'Azure Security Center et obtenir une vue d'ensemble de toutes les menaces SQL actives détectées dans la base de données.

:::image type="content" source="media/azure-defender-for-sql/advanced-threat-protection-alerts.png" alt-text="Présentation de la protection avancée contre les menaces dans une base de données":::

:::image type="content" source="media/azure-defender-for-sql/advanced-threat-protection.png" alt-text="Protection avancée contre les menaces dans le Centre de sécurité":::

## <a name="next-steps"></a>Étapes suivantes

- Découvrez-en plus sur [Advanced Threat Protection dans Azure SQL Database et Azure Synapse](threat-detection-configure.md).
- Découvrez-en plus sur [Advanced Threat Protection dans Azure SQL Managed Instance](../managed-instance/threat-detection-configure.md).
- En savoir plus sur [Azure Defender pour SQL](azure-defender-for-sql.md).
- En savoir plus sur [Audit Azure SQL Database](../../azure-sql/database/auditing-overview.md)
- En savoir plus sur [Azure Security Center](../../security-center/security-center-introduction.md)
- Pour plus d'informations sur la tarification, consultez la [page de tarification Azure SQL Database](https://azure.microsoft.com/pricing/details/sql-database/)
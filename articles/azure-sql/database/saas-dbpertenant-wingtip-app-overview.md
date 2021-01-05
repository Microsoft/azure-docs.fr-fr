---
title: 'Exemple d’application multilocataire : Wingtip SaaS'
description: Apprendre à l’aide d’un exemple d’application multilocataire utilisant l’exemple Azure SQL Database - SaaS Wingtip
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 09/24/2018
ms.openlocfilehash: 09651ed5c7472c12d82108fdbd450cca6e84349f
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96447085"
---
# <a name="introduction-to-a-multitenant-saas-app-that-uses-the-database-per-tenant-pattern-with-azure-sql-database"></a>Présentation d’une application SaaS mutualisée qui utilise le modèle de base de données par locataire avec Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

L’application SaaS Wingtip est un exemple d’application mutualisée. L’application utilise le modèle d’application SaaS de base de données par locataire pour traiter plusieurs locataires. L’application présente les fonctionnalités Azure SQL Database qui permettent des scénarios SaaS à l’aide de plusieurs modèles de gestion et de conception SaaS. Pour que vous soyez rapidement opérationnel, l’application SaaS Wingtip se déploie en moins de cinq minutes.

Le code source de l’application et les scripts de gestion sont disponibles dans le référentiel GitHub [WingtipTicketsSaaS-DbPerTenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant). Avant de commencer, consultez les [conseils généraux](saas-tenancy-wingtip-app-guidance-tips.md) pour télécharger et débloquer les scripts de gestion Wingtip Tickets.

## <a name="application-architecture"></a>Architecture de l'application

L’application Wingtip SaaS utilise le modèle de base de données par locataire. Elle utilise les pools élastiques SQL pour optimiser l’efficacité. Pour l’approvisionnement et le mappage des clients à leurs données, une base de données catalogue est utilisée. L’application SaaS Wingtip centrale utilise un pool avec trois exemples de locataires, plus la base de données catalogue. Les serveurs du catalogue et du locataire ont été provisionnés avec des alias DNS. Ces alias sont utilisés pour conserver une référence aux ressources actives utilisées par l’application Wingtip. Ces alias sont mis à jour pour pointer vers les ressources de récupération dans les didacticiels consacrés à la récupération d’urgence. Suivre de nombreux didacticiels SaaS Wingtip permet de générer des modules complémentaires pour le déploiement initial. Des modules complémentaires comme les bases de données d’analyse et la gestion de schémas entre les bases de données sont présentés.


![Architecture de SaaS Wingtip](./media/saas-dbpertenant-wingtip-app-overview/app-architecture.png)


Lorsque vous suivez les didacticiels et utilisez l’application, concentrez-vous sur les modèles SaaS liés à la couche Données. En d’autres termes, concentrez-vous sur la couche Données et n’analysez pas trop l’application en elle-même. Comprendre l’implémentation de ces modèles SaaS est essentiel pour les mettre en place dans vos applications. Réfléchissez également aux modifications nécessaires pour satisfaire à vos exigences métier spécifiques.

## <a name="sql-database-wingtip-saas-tutorials"></a>Didacticiels de l’application SaaS Wingtip de SQL Database

Après avoir déployé l’application, explorez les didacticiels suivants qui se basent sur le déploiement initial. Ces tutoriels explorent les modèles SaaS courants qui tirent parti des fonctionnalités intégrées de SQL Database, d’Azure Synapse Analytics et d’autres services Azure. Les didacticiels incluent des scripts PowerShell avec des explications détaillées. Les explications simplifient la compréhension et l’implémentation des modèles de gestion SaaS dans vos applications.


| Didacticiel | Description |
|:--|:--|
| [Conseils généraux pour utiliser des exemples d’applications Wingtip Tickets SaaS](saas-tenancy-wingtip-app-guidance-tips.md) | Téléchargez et exécutez les scripts PowerShell permettant de préparer des parties de l’application. |
|[Déployer et explorer l’application SaaS Wingtip](./saas-dbpertenant-get-started-deploy.md)|  Déployez et explorez l’application SaaS Wingtip avec votre abonnement Azure. |
|[Approvisionner des clients et les inscrire dans le catalogue](./saas-dbpertenant-provision-and-catalog.md)| Découvrez comment l’application se connecte aux locataires à l’aide d’une base de données catalogue, et comment le catalogue mappe les locataires à leurs données. |
|[Surveiller et gérer les performances](./saas-dbpertenant-performance-monitoring.md)| Découvrez comment utiliser les fonctionnalités de surveillance de SQL Database, et définir des alertes qui se déclenchent en cas de dépassement des seuils de performances. |
|[Superviser avec les journaux d’activité Azure Monitor](./saas-dbpertenant-log-analytics.md) | Apprenez à utiliser les [journaux Azure Monitor](../../azure-monitor/log-query/log-query-overview.md) pour surveiller de grandes quantités de ressources dans plusieurs pools. |
|[Restaurer un client unique](./saas-dbpertenant-restore-single-tenant.md)| Découvrez comment restaurer une base de données client à un point antérieur dans le temps. Apprenez également à restaurer une base de données parallèle, qui laisse la base de données de locataire existante en ligne. |
|[Gérer le schéma de base de données client](saas-tenancy-schema-management.md)| Découvrez comment mettre à jour un schéma et des données de référence sur toutes les bases de données de locataire. |
|[Exécuter des requêtes distribuées entre locataires](saas-tenancy-cross-tenant-reporting.md) | Créez une base de données d’analyse ad hoc, puis exécutez des requêtes distribuées en temps réel sur tous les locataires.  |
|[Exécuter l’analytique sur les données client extraites](saas-tenancy-tenant-analytics.md) | Extrayez les données client dans une base de données analytique ou un entrepôt de données pour les requêtes analytiques en mode hors connexion. |


## <a name="next-steps"></a>Étapes suivantes

- [Conseils généraux pour utiliser des exemples d’applications Wingtip Tickets SaaS](saas-tenancy-wingtip-app-guidance-tips.md)
- [Déployer l’application SaaS Wingtip](./saas-dbpertenant-get-started-deploy.md)
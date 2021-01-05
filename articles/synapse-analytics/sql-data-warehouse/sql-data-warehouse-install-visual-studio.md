---
title: Installer Visual Studio 2019
description: Installer Visual Studio et SQL Server Data Tools (SSDT) pour Synapse SQL
services: synapse-analytics
ms.custom: vs-azure, azure-synapse
ms.workload: azure-vs
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 05/11/2020
ms.author: kevin
ms.reviewer: igorstan
ms.openlocfilehash: 456d7eb0cabd7ebede6415e3ce92d7403ecef7ef
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97368752"
---
# <a name="getting-started-with-visual-studio-2019"></a>Prise en main de Visual Studio 2019

Visual Studio **2019** SQL Server Data Tools (SSDT) est un outil unique qui vous permet d’effectuer les opérations suivantes :

- Connecter, interroger et développer des applications
- Utiliser un explorateur d’objets pour explorer visuellement tous les objets de votre modèle de données, notamment les tables, les vues, les procédures stockées, etc.
- Générer des scripts DDL (Data Definition Language) T-SQL pour vos objets
- Développer votre entrepôt de données à l’aide d’une approche basée sur l’état avec des projets de base de données SSDT
- Intégrer votre projet de base de données à des systèmes de contrôle de code source comme Git avec Azure Repos
- Configurer des pipelines d’intégration et de déploiement continus avec des serveurs d’automatisation comme Azure DevOps

## <a name="install-visual-studio-2019"></a>Installer Visual Studio 2019

Consultez [Télécharger Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) pour télécharger et installer Visual Studio **16.3 ou version ultérieure**. Lors de l’installation, sélectionnez la charge de travail de stockage et de traitement des données. L’installation SSDT autonome n’est plus obligatoire dans Visual Studio 2019.

## <a name="unsupported-features-in-ssdt"></a>Fonctionnalités non prises en charge sur SSDT

Certaines mises en production de fonctionnalités Synapse SQL peuvent ne pas prendre en charge SSDT. Actuellement, les fonctionnalités suivantes ne sont pas prises en charge :


- [Gestion des charges de travail](sql-data-warehouse-workload-management.md) : classifieurs et groupes de charge de travail
- [Sécurité au niveau des lignes](/sql/relational-databases/security/row-level-security?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) (y compris les fonctions table)
  - Envoyez un [ticket de support ou votez](https://feedback.azure.com/forums/307516-sql-data-warehouse/suggestions/39040057-ssdt-row-level-security) pour obtenir la fonctionnalité prise en charge.
  - Envoyez un [ticket de support ou votez](https://feedback.azure.com/forums/307516-sql-data-warehouse/suggestions/39040048-ssdt-support-dynamic-data-masking) pour obtenir la fonctionnalité prise en charge.
- Tables avec une [colonne d’identité](https://docs.microsoft.com/sql/t-sql/statements/create-table-transact-sql-identity-property?view=sql-server-ver15)
- Certaines fonctionnalités T-SQL, telles que :
   - La clause *WITHIN GROUP* de la fonction de chaîne [STRING_AGG](/sql/t-sql/functions/string-agg-transact-sql).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous disposez de la dernière version de SSDT, vous êtes prêt à vous [connecter](sql-data-warehouse-query-visual-studio.md) à votre pool SQL.

---
title: Résoudre les problèmes de pool SQL dédié (anciennement SQL DW)
description: Résolution des problèmes de pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics.
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 11/13/2020
ms.author: kevin
ms.reviewer: jrasnick
ms.custom: azure-synapse
ms.openlocfilehash: c69df88828a179cf1dce7bd35ac19c518b07903d
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96530712"
---
# <a name="troubleshooting-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>Résoudre les problèmes de pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics

Cet article répertorie les problèmes courants liés au pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics.

## <a name="connecting"></a>Connecting

| Problème                                                        | Résolution                                                   |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Échec de connexion pour l’utilisateur 'NT AUTHORITY\ANONYMOUS LOGON'. (Microsoft SQL Server, erreur : 18456) | Cette erreur se produit quand un utilisateur Azure AD tente de se connecter à la base de données MASTER alors qu’elle ne contient pas d’utilisateur.  Pour corriger ce problème, spécifiez le pool SQL dédié (anciennement SQL DW) auquel vous souhaitez vous connecter ou ajoutez l'utilisateur à la base de données MASTER.  Consultez l’article [Présentation de la sécurité](sql-data-warehouse-overview-manage-security.md) pour plus de détails. |
| Le serveur principal « MyUserName » n’est pas en mesure d’accéder à la base de données « master » dans le contexte de sécurité actuel. La base de données utilisateur par défaut ne peut pas être ouverte. Échec de la connexion. Échec de la connexion pour l'utilisateur 'MyUserName'. (Microsoft SQL Server, erreur : 916) | Cette erreur se produit quand un utilisateur Azure AD tente de se connecter à la base de données MASTER alors qu’elle ne contient pas d’utilisateur.  Pour corriger ce problème, spécifiez le pool SQL dédié (anciennement SQL DW) auquel vous souhaitez vous connecter ou ajoutez l'utilisateur à la base de données MASTER.  Consultez l’article [Présentation de la sécurité](sql-data-warehouse-overview-manage-security.md) pour plus de détails. |
| Erreur CTAIP                                                  | Cette erreur peut se produire lorsqu’un identifiant de connexion a été créé sur la base de données MASTER SQL Database, mais pas sur la base de données SQL Database spécifique.  Si vous rencontrez cette erreur, consultez l’article [Vue d’ensemble de la sécurité](sql-data-warehouse-overview-manage-security.md) .  Cet article explique comment créer un identifiant de connexion et un utilisateur sur la base de données MASTER, puis comment créer un utilisateur dans la base de données SQL. |
| Bloqué par le pare-feu                                          | Un pool SQL dédié (anciennement SQL DW) est protégé par des pare-feu pour que seules les adresses IP connues aient accès à une base de données. Les pare-feu sont sécurisés par défaut, ce qui signifie que vous devez activer explicitement une adresse IP ou une plage d’adresses avant de vous connecter.  Pour configurer votre pare-feu pour l’accès, suivez les étapes décrites dans la section [Configuration de l’accès au pare-feu du serveur pour l’adresse IP de votre client](create-data-warehouse-portal.md) dans les [Instructions d’approvisionnement](create-data-warehouse-portal.md). |
| Connexion impossible avec l’outil ou le pilote                           | Avec le pool SQL dédié (anciennement SQL DW), il est recommandé d'utiliser [SSMS](/sql/ssms/download-sql-server-management-studio-ssms?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest), [SSDT pour Visual Studio](sql-data-warehouse-install-visual-studio.md) ou [sqlcmd](sql-data-warehouse-get-started-connect-sqlcmd.md) pour interroger les données. Pour plus d’informations sur les pilotes et la connexion à Azure Synapse, consultez les articles [Pilotes pour Azure Synapse](sql-data-warehouse-connection-strings.md) et [Connexion à Azure Synapse](sql-data-warehouse-connect-overview.md). |

## <a name="tools"></a>Outils

| Problème                                                        | Résolution                                                   |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Des utilisateurs Azure AD sont manquants dans l’explorateur d’objets Visual Studio           | Il s'agit d'un problème connu.  Comme solution de contournement, vous pouvez afficher les utilisateurs dans [sys.database_principals](/sql/relational-databases/system-catalog-views/sys-database-principals-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  Pour plus d'informations sur l'utilisation d'Azure Active Directory avec le pool SQL dédié (anciennement SQL DW), consultez [Authentification à Azure Synapse](sql-data-warehouse-authentication.md). |
| Les scripts manuels, l’utilisation de l’Assistant Création de scripts ou la connexion via SSMS sont lents, ne répondent pas ou produisent des erreurs | Vérifiez que les utilisateurs ont été créés dans la base de données master. Dans les options de script, vérifiez aussi que l’édition du moteur est définie en tant que « Édition Microsoft Azure Synapse Analytics » et que le type de moteur est « Microsoft Azure SQL Database ». |
| La génération de scripts échoue dans SSMS                               | La génération d'un script pour le pool SQL dédié (anciennement SQL DW) échoue si l'option « Générer un script pour les objets dépendants » est définie sur « True ». Pour résoudre ce problème, les utilisateurs doivent accéder manuellement à **Outils -> Options -> Explorateur d’objets SQL Server -> Générer un script pour les options dépendants et affecter la valeur false à cette option**. |

## <a name="data-ingestion-and-preparation"></a>Ingestion et préparation des données

| Problème                                                        | Résolution                                                   |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| L’exportation de chaînes vides à l’aide de CETAS donnera des valeurs NULL dans les fichiers Parquet et ORC. Si vous exportez des chaînes vides à partir de colonnes avec des contraintes NOT NULL, CETAS entraîne le rejet d’enregistrements, et l’exportation risque d’échouer. | Supprimez les chaînes vides ou la colonne incriminée dans l’instruction SELECT de votre CETAS. |
| Le chargement d’une valeur en dehors de la plage 0-127 dans une colonne tinyint pour le format de fichier ORC et Parquet n’est pas pris en charge. | Spécifiez un plus grand type de données pour la colonne cible.           |

## <a name="performance"></a>Performances

| Problème                                                        | Résolution                                                   |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Résolution des problèmes de performances des requêtes                            | Si vous tentez de dépanner une requête spécifique, commencez avec l’article relatif à la [surveillance de vos requêtes](sql-data-warehouse-manage-monitor.md#monitor-query-execution). |
| Problèmes d’espaces TempDB | [Surveillez l’utilisation de l’espace TempDB](sql-data-warehouse-manage-monitor.md#monitor-tempdb).  Les causes courantes de l’insuffisance d’espace TempDB sont les suivantes :<br>- Les ressources allouées à la requête ne sont pas suffisantes et les données sont propagées dans TempDB.  Consultez [Gestion des charges de travail](resource-classes-for-workload-management.md) <br>- Les statistiques sont manquantes ou obsolètes, ce qui entraîne un déplacement excessif des données.  Consultez la section[Maintenance des statistiques de table](sql-data-warehouse-tables-statistics.md) pour plus d’informations sur la création de statistiques<br>- L’espace TempDB est alloué par niveau de service.  La [mise à l'échelle du pool SQL dédié (anciennement SQL DW)](sql-data-warehouse-manage-compute-overview.md#scaling-compute) sur un paramètre DWU plus élevé permet d'allouer plus d'espace TempDB.|
| Des performances des requêtes et des plans médiocres sont souvent le résultat de statistiques manquantes | La cause la plus courante de la médiocrité des performances est le manque de statistiques concernant vos tables.  Consultez la section[Maintenance des statistiques de table](sql-data-warehouse-tables-statistics.md) pour plus d’informations sur la création de statistiques et les raisons pour lesquelles elles sont essentielles à vos performances. |
| Concurrence faible / requêtes en file d’attente                             | Comprendre la [gestion des charges de travail](resource-classes-for-workload-management.md) est important pour comprendre comment équilibrer l’allocation de mémoire avec l’accès concurrentiel. |
| Comment mettre en œuvre les meilleures pratiques                              | Pour apprendre à améliorer le niveau de performance des requêtes, commencez par consulter l'article [Meilleures pratiques en matière de pool SQL dédié (anciennement SQL DW)](sql-data-warehouse-best-practices.md). |
| Comment améliorer les performances avec la mise à l’échelle                      | Pour améliorer le niveau de performance, il suffit parfois d'ajouter de la puissance de calcul aux requêtes en procédant à une [mise à l'échelle du pool SQL dédié (anciennement SQL DW)](sql-data-warehouse-manage-compute-overview.md). |
| Performances de requêtes médiocres en raison de la qualité médiocre de l’index     | Parfois, les requêtes peuvent ralentir en raison de la [qualité médiocre des index columnstore](sql-data-warehouse-tables-index.md#causes-of-poor-columnstore-index-quality).  Consultez cet article pour plus d’informations et pour découvrir comment [Reconstruire des index pour améliorer la qualité de segment](sql-data-warehouse-tables-index.md#rebuilding-indexes-to-improve-segment-quality). |

## <a name="system-management"></a>Gestion de systèmes

| Problème                                                        | Résolution                                                   |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Msg 40847 : Impossible d’effectuer l’opération car le serveur risque de dépasser le quota d’unité de transaction de base de données autorisé de 45 000. | Réduisez la [DWU](what-is-a-data-warehouse-unit-dwu-cdwu.md) de la base de données que vous essayez de créer ou [demandez une augmentation du quota](sql-data-warehouse-get-started-create-support-ticket.md). |
| Examen de l’utilisation de l’espace                              | Consultez la section relative aux [tailles des tables](sql-data-warehouse-tables-overview.md#table-size-queries) pour comprendre l’utilisation de l’espace de votre système. |
| Aide concernant la gestion des tables                                    | Consultez l’article [Vue d’ensemble des tables](sql-data-warehouse-tables-overview.md) pour obtenir de l’aide concernant la gestion de vos tables.  Cet article inclut également des liens vers des rubriques plus détaillées, notamment [Types de données de table](sql-data-warehouse-tables-data-types.md), [Distribution d’une table](sql-data-warehouse-tables-distribute.md), [Indexation d’une table](sql-data-warehouse-tables-index.md), [Partitionnement d’une table](sql-data-warehouse-tables-partition.md), [Maintenance des statistiques de table](sql-data-warehouse-tables-statistics.md) et [Tables temporaires](sql-data-warehouse-tables-temporary.md). |
| La barre de progression de TDE (Transparent Data Encryption) n’est pas mise à jour dans le portail Microsoft Azure | Vous pouvez afficher l’état de chiffrement de TDE via [PowerShell](/powershell/module/az.sql/get-azsqldatabasetransparentdataencryption?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). |

## <a name="differences-from-sql-database"></a>Différences par rapport à la base de données SQL

| Problème                                 | Résolution                                                   |
| :------------------------------------ | :----------------------------------------------------------- |
| Fonctionnalités de base de données SQL non prises en charge     | Voir [Fonctionnalités de tables non prises en charge](sql-data-warehouse-tables-overview.md#unsupported-table-features). |
| Types de données de base de données SQL non pris en charge   | Voir [Types de données non pris en charge](sql-data-warehouse-tables-data-types.md#identify-unsupported-data-types).        |
| Limitations des procédures stockées          | Consultez [Limitations des procédures stockées](sql-data-warehouse-develop-stored-procedures.md#limitations) pour comprendre certaines limitations des procédures stockées. |
| Les fonctions définies par l’utilisateur ne prennent pas en charge les instructions SELECT | Il s’agit d’une limitation actuelle de nos fonctions définies par l’utilisateur.  Consultez [CREATE FUNCTION](/sql/t-sql/statements/create-function-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) pour connaître la syntaxe que nous prenons en charge. |

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir une aide supplémentaire lors de la recherche d’une solution à votre problème, voici d’autres ressources auxquelles vous pouvez recourir.

* [Blogs](https://azure.microsoft.com/blog/tag/azure-sql-data-warehouse/)
* [Demandes de fonctionnalités](https://feedback.azure.com/forums/307516-sql-data-warehouse)
* [Vidéos](https://azure.microsoft.com/documentation/videos/index/?services=sql-data-warehouse)
* [Création d’un ticket de support](sql-data-warehouse-get-started-create-support-ticket.md)
* [Page de questions Microsoft Q&A](https://docs.microsoft.com/answers/topics/azure-synapse-analytics.html)
* [Forum Stack Overflow](https://stackoverflow.com/questions/tagged/azure-sqldw)
* [Twitter](https://twitter.com/hashtag/SQLDW)

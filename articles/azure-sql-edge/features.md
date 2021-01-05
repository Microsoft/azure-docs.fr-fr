---
title: Fonctionnalités prises en charge par Azure SQL Edge
description: Découvrez les fonctionnalités prises en charge par Azure SQL Edge.
keywords: présentation de SQL Edge, qu'est-ce que SQL Edge, vue d'ensemble de SQL Edge
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/03/2020
ms.openlocfilehash: 19dcbbf102a1d8d21f1b14780ea33816a1677c55
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93392025"
---
# <a name="supported-features-of-azure-sql-edge"></a>Fonctionnalités prises en charge par Azure SQL Edge 

Azure SQL Edge repose sur la dernière version du moteur de base de données SQL. Il prend en charge une partie des fonctionnalités prises en charge par SQL Server 2019 sur Linux, en plus de certaines fonctionnalités qui ne sont pas prises en charge ou disponibles dans SQL Server 2019 sur Linux (ou dans SQL Server sur Windows).

Pour obtenir la liste complète des fonctionnalités prises en charge dans SQL Server pour Linux, consultez [Éditions et fonctionnalités de SQL Server 2019 pour Linux prises en charge](/sql/linux/sql-server-linux-editions-and-components-2019). Pour obtenir la liste des éditions et fonctionnalités de SQL Server pour Windows prises en charge, consultez [Éditions et fonctionnalités de SQL Server 2019 (15.x) prises en charge](/sql/sql-server/editions-and-components-of-sql-server-version-15).

## <a name="azure-sql-edge-editions"></a>Éditions d’Azure SQL Edge

Deux éditions ou plans logiciels d'Azure SQL Edge sont disponibles. Ces éditions disposent des mêmes fonctionnalités et ne diffèrent que par leurs droits d’utilisation et par la capacité de mémoire et de cœurs à laquelle elles peuvent accéder sur le système hôte.

   |**Planification**  |**Description**  |
   |---------|---------|
   |Azure SQL Edge Développeur  |  Pour le développement uniquement. Chaque conteneur Azure SQL Edge Développeur est limité à 4 cœurs et à 32 Go de mémoire.  |
   |Azure SQL Edge    |  Pour la production. Chaque conteneur Azure SQL Edge est limité à 8 cœurs et à 64 Go de mémoire.  |

## <a name="operating-system"></a>Système d’exploitation

Les conteneurs Azure SQL Edge reposent sur Ubuntu 18.04 et, par conséquent, ne sont pris en charge que sur les hôtes Docker fonctionnant avec Ubuntu 18.04 LTS (recommandé) ou Ubuntu 20.04 LTS. Il est possible d’exécuter des conteneurs Azure SQL Edge sur d’autres hôtes de système d’exploitation, par exemple, sur d’autres distributions de Linux ou sur Windows (avec Docker CE ou Docker EE). Toutefois, Microsoft déconseille cette configuration, car elle risque de ne pas être rigoureusement testée.

La configuration recommandée pour exécuter Azure SQL Edge sur Windows consiste à configurer une machine virtuelle Ubuntu sur l’hôte Windows, puis à exécuter Azure SQL Edge sur la machine virtuelle Linux.

Le système de fichiers recommandé et pris en charge pour Azure SQL Edge est EXT4 et XFS. Si des volumes persistants sont utilisés pour sauvegarder le stockage de base de données Azure SQL Edge, le système de fichiers hôte sous-jacent doit être EXT4 et XFS.

## <a name="hardware-support"></a>Prise en charge matérielle

Un processeur 64 bits (x64 ou ARM64) est nécessaire pour Azure SQL Edge, avec au moins un processeur et 1 Go de RAM sur l’hôte. L’empreinte mémoire au démarrage d’Azure SQL Edge est proche de 450 Mo. Par conséquent, un supplément de mémoire est nécessaire pour les autres modules ou processus IoT Edge exécutés sur le périphérique de périmètre. Les besoins réels en mémoire et en processeur d’Azure SQL Edge varient en fonction de la complexité de la charge de travail et du volume de données traitées. Microsoft vous recommande, lorsque vous choisissez un matériel pour votre solution, d’exécuter des tests de performances rigoureux afin de vérifier que les caractéristiques de performances requises pour votre solution sont respectées.  

## <a name="azure-sql-edge-components"></a>Composants Azure SQL Edge

Azure SQL Edge ne prend en charge que le moteur de base de données. Il ne prend pas en charge les autres composants disponibles avec SQL Server 2019 pour Windows ou SQL Server 2019 pour Linux. Plus précisément, Azure SQL Edge ne prend pas en charge les composants SQL Server tels qu’Analysis Services, Reporting Services, Integration Services, Master Data Services, Machine Learning Services (dans la base de données) et Machine Learning Server (autonome).

## <a name="supported-features"></a>Fonctionnalités prises en charge

Outre la prise en charge d’un sous-ensemble de fonctionnalités de SQL Server sur Linux, Azure SQL Edge prend en charge les nouvelles fonctionnalités suivantes : 

- Le streaming SQL, basé sur le même moteur qu’Azure Stream Analytics, fournit des fonctionnalités de streaming de données en temps réel dans Azure SQL Edge. 
- La fonction T-SQL appelée `Date_Bucket` pour l’analytique données de série chronologique.
- Fonctionnalités de machine learning via le runtime ONNX inclus avec le moteur SQL.

## <a name="unsupported-features"></a>Fonctionnalités non prises en charge

La liste suivante présente les fonctionnalités de SQL Server 2019 pour Linux qui ne sont pas prises en charge par Azure SQL Edge.

| Domaine | Fonctionnalité ou service non pris en charge |
|-----|-----|
| **Création de bases de données** | OLTP en mémoire + commandes DDL et fonctions Transact-SQL associées, affichages catalogue et vues de gestion dynamique. |
| &nbsp; | Type de données `HierarchyID` + commandes DDL et fonctions Transact-SQL associées, affichages catalogue et vues de gestion dynamique. |
| &nbsp; | Type de données `Spatial` + commandes DDL et fonctions Transact-SQL associées, affichages catalogue et vues de gestion dynamique. |
| &nbsp; | Stretch DB + commandes DDL et fonctions Transact-SQL associées, affichages catalogue et vues de gestion dynamique. |
| &nbsp; | Index de recherche en texte intégral + commandes DDL et fonctions Transact-SQL associées, affichages catalogue et vues de gestion dynamique.|
| &nbsp; | `FileTable`, `FILESTREAM` + commandes DDL et fonctions Transact-SQL associées, affichages catalogue et vues de gestion dynamique.|
| **Moteur de base de données** | Réplication. Notez que vous pouvez configurer Azure SQL Edge en tant qu’abonné push d’une topologie de réplication. |
| &nbsp; | PolyBase. Notez que vous pouvez configurer Azure SQL Edge comme cible pour les tables externes dans Polybase. |
| &nbsp; | Extensibilité du langage avec Java et Spark. |
| &nbsp; | Intégration d’Active Directory. |
| &nbsp; | Réduction automatique de la base de données. Vous pouvez définir la propriété Réduction automatique pour une base de données à l’aide de la commande `ALTER DATABASE <database_name> SET AUTO_SHRINK ON`, mais cette modification n’a aucun effet. La tâche de réduction automatique ne s’exécutera pas sur la base de données. Les utilisateurs peuvent toujours réduire les fichiers de base de données à l’aide des commandes DBCC. |
| &nbsp; | Instantanés de base de données. |
| &nbsp; | Prise en charge de la mémoire persistante. |
| &nbsp; | Microsoft Distributed Transaction Coordinator. |
| &nbsp; | Gouverneur de ressources et gouvernance des ressources d’E/S. |
| &nbsp; | Extension du pool de mémoires tampons. |
| &nbsp; | Requête distribuée avec connexions tierces. |
| &nbsp; | Serveurs liés. |
| &nbsp; | Procédures stockées étendues système (par exemple, `XP_CMDSHELL`). |
| &nbsp; | Assemblys CLR + commandes DDL et fonctions Transact-SQL associées, affichages catalogue et vues de gestion dynamique. |
| &nbsp; | Fonctions T-SQL dépendant du CLR, comme `ASSEMBLYPROPERTY`, `FORMAT`, `PARSE` et `TRY_PARSE`. |
| &nbsp; | Fonctions, clauses de requête et affichages catalogue de dates et d’heures dépendant du CLR. |
| &nbsp; | Extension du pool de mémoires tampons. |
| &nbsp; | Messagerie de base de données. |
| &nbsp; | Service Broker |
| &nbsp; | Gestion basée sur des stratégies |
| &nbsp; | entrepôt de données de gestion |
| &nbsp; | Bases de données autonomes |
| **SQL Server Agent** |  Sous-systèmes : CmdExec, PowerShell, lecture de la file d’attente, SSIS, SSAS et SSRS. |
| &nbsp; | Alertes. |
| &nbsp; | Sauvegarde managée. |
| **Haute disponibilité** | Groupes de disponibilité Always On.  |
| &nbsp; | Groupes de disponibilité de base. |
| &nbsp; | Instance de cluster de basculement Always On. |
| &nbsp; | Mise en miroir de bases de données. |
| &nbsp; | Ajout de mémoire et de processeur à chaud. |
| **Sécurité** | Gestion de clés extensible. |
| &nbsp; | Intégration d’Active Directory.|
| &nbsp; | Prise en charge des enclaves sécurisées.|
| **Services** | SQL Server Browser. |
| &nbsp; | Machine Learning via R et Python. |
| &nbsp; | StreamInsight. |
| &nbsp; | Analysis Services. |
| &nbsp; | Reporting Services. |
| &nbsp; | Data Quality Services. |
| &nbsp; | Master Data Services. |
| &nbsp; | Distributed Replay. |
| **Facilité de gestion** | Point de contrôle de l’utilitaire SQL Server. |

## <a name="next-steps"></a>Étapes suivantes

- [Déployer Azure SQL Edge](deploy-portal.md)
- [Configurer Azure SQL Edge](configure.md)
- [Se connecter à une instance d'Azure SQL Edge à l'aide des outils du client SQL Server](connect.md)
- [Sauvegarder et restaurer des bases de données dans Azure SQL Edge](backup-restore.md)
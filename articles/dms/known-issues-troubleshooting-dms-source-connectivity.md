---
title: Problèmes de connexion aux bases de données sources
titleSuffix: Azure Database Migration Service
description: Découvrez comment résoudre des problèmes/erreurs connus associés à la connexion d’Azure Database Migration Service à des bases de données sources.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: troubleshooting
ms.date: 02/20/2020
ms.openlocfilehash: 93ac6ae3c8aed61557a239bb9c84d3587dce1daa
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94962329"
---
# <a name="troubleshoot-dms-errors-when-connecting-to-source-databases"></a>Résoudre les erreurs DMS lors de la connexion aux bases de données sources

L’article suivant fournit des informations sur la manière de traiter d’éventuels problèmes que vous pouvez rencontrer lors de la connexion d’Azure Database Migration Service (DMS) à votre base de données source. Chaque section ci-dessous concerne un type spécifique de base de données source, répertoriant l’erreur que vous pouvez rencontrer ainsi que des détails et des liens vers des informations indiquant comment résoudre les problèmes de connectivité.

## <a name="sql-server"></a>SQL Server

Le tableau suivant indique les éventuels problèmes liés à la connexion à une base de données SQL Server source et la façon d’y remédier.

| Error         | Cause du problème et informations pour sa résolution |
| ------------- | ------------- |
| La connexion SQL a échoué. Une erreur liée au réseau ou propre à une instance s’est produite lors de l’établissement d’une connexion à SQL Server. Le serveur est introuvable ou inaccessible. Vérifiez que le nom d’instance est correct et que SQL Server est configuré pour autoriser les connexions à distance.<br> | Cette erreur se produit si le service ne peut pas localiser le serveur source. Pour résoudre ce problème, consultez l’article [Erreur de connexion au serveur SQL Server source lors de l’utilisation d’un port dynamique ou d’une instance nommée](./known-issues-troubleshooting-dms.md#error-connecting-to-source-sql-server-when-using-dynamic-port-or-named-instance). |
| **Erreur 53** - La connexion SQL a échoué. (Également pour les codes d’erreur 1, 2, 5, 53, 233, 258, 1225, 11001)<br><br> | Cette erreur se produit si le service ne peut pas se connecter au serveur source. Pour résoudre le problème, consultez les ressources suivantes, puis retentez l’opération. <br><br>  [Guide de l’utilisateur interactif pour la résolution du problème de connectivité](https://support.microsoft.com/help/4009936/solving-connectivity-errors-to-sql-server)<br><br> [Prérequis pour la migration de SQL Server vers Azure SQL Database](./pre-reqs.md#prerequisites-for-migrating-sql-server-to-azure-sql-managed-instance) <br><br> [Conditions préalables à la migration de SQL Server vers une instance gérée SQL Azure](./pre-reqs.md#prerequisites-for-migrating-sql-server-to-azure-sql-managed-instance) |
| **Erreur 18456** - La connexion a échoué.<br> | Cette erreur se produit si le service ne peut pas se connecter à la base de données source avec les informations d’identification T-SQL fournies. Pour résoudre le problème, vérifiez les informations d’identification entrées. Vous pouvez également vous reporter à [MSSQLSERVER_18456](/sql/relational-databases/errors-events/mssqlserver-18456-database-engine-error?view=sql-server-2017) ou aux documents de résolution des problèmes listés dans la note sous ce tableau, puis retenter l’opération. |
| Valeur de nom de compte (AccountName) « {0} » fournie incorrecte. Le format attendu pour AccountName est Nom_domaine\Nom_utilisateur<br> | Cette erreur se produit si l’utilisateur sélectionne l’authentification Windows, mais qu’il fournit le nom d’utilisateur dans un format non valide. Pour résoudre le problème, fournissez le nom d’utilisateur dans le format correct pour l’authentification Windows ou sélectionnez **Authentification SQL**. |

## <a name="aws-rds-mysql"></a>AWS RDS MySQL

Le tableau suivant indique les éventuels problèmes liés à la connexion à une base de données AWS RDS MySQL source et la façon d’y remédier.

| Error         | Cause du problème et informations pour sa résolution |
| ------------- | ------------- |
| **Erreur [2003]** [HY000] - La connexion a échoué. ERREUR [HY000] [MySQL] [Pilote ODBC x.x(w)] Connexion au serveur MySQL sur « {server} » impossible (10060) | Cette erreur se produit si le pilote ODBC MySQL ne peut pas se connecter au serveur source. Pour résoudre le problème, consultez les documents de résolution des problèmes listés dans la note sous ce tableau, puis retentez l’opération.<br> |
| **Erreur [2005]** [HY000] - La connexion a échoué. ERREUR [HY000] [MySQL] [Pilote ODBC x.x(w)] Hôte du serveur MySQL « {server} » inconnu | Cette erreur se produit si le service ne parvient pas à trouver l’hôte source sur les services Bureau à distance (RDS). Le problème peut être dû au fait que la source indiquée n’existe pas ou qu’il existe un problème avec l’infrastructure des services Bureau à distance (RDS). Pour résoudre le problème, consultez les documents de résolution des problèmes listés dans la note sous ce tableau, puis retentez l’opération.<br> |
| **Erreur [1045]** [HY000] - La connexion a échoué. ERREUR [HY000] [MySQL] [Pilote ODBC x.x(w)] Accès refusé pour l’utilisateur '{user}'@'{server}' (utilisation d’un mot de passe : OUI) | Cette erreur se produit si le pilote ODBC MySQL ne peut pas se connecter au serveur source en raison d’informations d’identification non valides. Vérifiez les informations d’identification que vous avez entrées. Si le problème persiste, vérifiez que l’ordinateur source dispose des informations d’identification correctes. Vous devrez peut-être réinitialiser le mot de passe dans la console. Si vous rencontrez toujours le problème, consultez les documents de résolution des problèmes listés dans la note sous ce tableau, puis retentez l’opération.<br> |
| **Erreur [9002]** [HY000] - La connexion a échoué. ERREUR [HY000] [MySQL] [Pilote ODBC x.x(w)] La chaîne de connexion n’est peut-être pas correcte. Pour obtenir des références, visitez le portail.| Cette erreur se produit si la connexion échoue en raison d’un problème avec la chaîne de connexion. Vérifiez que la chaîne de connexion fournie est valide. Pour résoudre le problème, consultez les documents de résolution des problèmes listés dans la note sous ce tableau, puis retentez l’opération.<br> |
| **Erreur de journalisation binaire. La variable binlog_format a la valeur « {value} ». Remplacez-la par « row ».** | Cette erreur se produit s’il existe une erreur de journalisation binaire ; la valeur de la variable binlog_format est incorrecte. Pour résoudre le problème, remplacez la valeur de binlog_format dans le groupe de paramètres par « ROW », puis redémarrez l’instance. Pour plus d’informations, consultez les [Variables et options de journalisation binaire](https://dev.mysql.com/doc/refman/5.7/en/replication-options-binary-log.html) ou la [documentation Fichiers journaux de base de données AWS RDS MySQL](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.Concepts.MySQL.html).<br> |

> [!NOTE]
> Pour plus d’informations sur la résolution des problèmes liés à la connexion à une base de données AWS RDS MySQL source, consultez les ressources suivantes :
> * [Résolution des problèmes de connectivité d’Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_Troubleshooting.html#CHAP_Troubleshooting.Connecting)
> * [Comment résoudre les problèmes de connexion à une instance de base de données Amazon RDS ?](https://aws.amazon.com/premiumsupport/knowledge-center/rds-cannot-connect)

## <a name="aws-rds-postgresql"></a>AWS RDS PostgreSQL

Le tableau suivant indique les éventuels problèmes liés à la connexion à une base de données AWS RDS PostgreSQL source et la façon d’y remédier.

| Error         | Cause du problème et informations pour sa résolution |
| ------------- | ------------- |
| **Erreur [101]** [08001] - La connexion a échoué. ERREUR [08001] - Délai expiré | Cette erreur se produit si le pilote Postgres ne peut pas se connecter au serveur source. Pour résoudre le problème, consultez les documents de résolution des problèmes listés dans la note sous ce tableau, puis retentez l’opération. |
| **Erreur : Le paramètre wal_level a la valeur « {value} ». Remplacez-la par « logical » pour permettre la réplication.** | Cette erreur se produit si la valeur du paramètre wal_level est incorrecte. Pour résoudre le problème, remplacez la valeur de rds.logical_replication dans le groupe de paramètres par 1, puis redémarrez l’instance. Pour plus d’informations, consultez [Prérequis pour la migration vers Azure PostgreSQL à l’aide de DMS](./tutorial-postgresql-azure-postgresql-online.md#prerequisites) ou vers [PostgreSQL sur Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_PostgreSQL.html). |

> [!NOTE]
> Pour plus d’informations sur la résolution des problèmes liés à la connexion à une base de données AWS RDS PostgreSQL source, consultez les ressources suivantes :
> * [Résolution des problèmes de connectivité d’Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_Troubleshooting.html#CHAP_Troubleshooting.Connecting)
> * [Comment résoudre les problèmes de connexion à une instance de base de données Amazon RDS ?](https://aws.amazon.com/premiumsupport/knowledge-center/rds-cannot-connect)

## <a name="aws-rds-sql-server"></a>AWS RDS SQL Server

Le tableau suivant indique les éventuels problèmes liés à la connexion à une base de données AWS RDS SQL Server source et la façon d’y remédier.

| Error         | Cause du problème et informations pour sa résolution |
| ------------- | ------------- |
| **Erreur 53** - La connexion SQL a échoué. Une erreur liée au réseau ou propre à une instance s’est produite lors de l’établissement d’une connexion à SQL Server. Le serveur est introuvable ou inaccessible. Vérifiez que le nom d’instance est correct et que SQL Server est configuré pour autoriser les connexions à distance. (fournisseur : Fournisseur de canaux nommés, erreur : 40 - Impossible d'ouvrir une connexion à SQL Server | Cette erreur se produit si le service ne peut pas se connecter au serveur source. Pour résoudre le problème, consultez les documents de résolution des problèmes listés dans la note sous ce tableau, puis retentez l’opération. |
| **Erreur 18456** - La connexion a échoué. Échec de la connexion pour l’utilisateur « {user} » | Cette erreur se produit si le service ne peut pas se connecter à la base de données source avec les informations d’identification T-SQL fournies. Pour résoudre le problème, vérifiez les informations d’identification entrées. Vous pouvez également vous reporter à [MSSQLSERVER_18456](/sql/relational-databases/errors-events/mssqlserver-18456-database-engine-error?view=sql-server-2017) ou aux documents de résolution des problèmes listés dans la note sous ce tableau, puis retenter l’opération. |
| **Erreur 87**- Chaîne de connexion non valide. Une erreur liée au réseau ou propre à une instance s’est produite lors de l’établissement d’une connexion à SQL Server. Le serveur est introuvable ou inaccessible. Vérifiez que le nom d’instance est correct et que SQL Server est configuré pour autoriser les connexions à distance. (fournisseur : interfaces réseau SQL, erreur : 25 - Chaîne de connexion non valide) | Cette erreur se produit si le service ne peut pas se connecter au serveur source en raison d’une chaîne de connexion non valide. Pour résoudre ce problème, vérifiez la chaîne de connexion fournie. Si le problème persiste, consultez les documents de résolution des problèmes listés dans la note sous ce tableau, puis retentez l’opération. |
| **Erreur - Certificat de serveur non approuvé.** Une connexion a été établie avec le serveur, mais une erreur s’est ensuite produite pendant le processus de connexion. (fournisseur : Fournisseur SSL, erreur : 0 - La chaîne de certificats a été fournie par une autorité qui n’est pas approuvée.) | Cette erreur se produit si le certificat utilisé n’est pas approuvé. Pour résoudre ce problème, vous devez trouver un certificat qui peut être approuvé, puis l’activer sur le serveur. Vous pouvez également sélectionner l’option Approuver le certificat lors de la connexion. Effectuez cette action uniquement si vous connaissez le certificat utilisé et que vous l’approuvez. <br> Les connexions TLS chiffrées à l’aide d’un certificat auto-signé n’offrent pas de sécurité renforcée. Elles sont vulnérables aux attaques de l’intercepteur. Ne vous appuyez pas sur TLS à l’aide de certificats auto-signés dans un environnement de production ou sur des serveurs connectés à Internet. <br> Pour plus d’informations, consultez [Utilisation de SSL avec une instance de base de données Microsoft SQL Server](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/SQLServer.Concepts.General.SSL.Using.html) ou le [Tutoriel : Migrer RDS SQL Server vers Azure à l’aide de DMS](./tutorial-rds-sql-server-azure-sql-and-managed-instance-online.md#prerequisites). |
| **Erreur 300** - L’utilisateur n’a pas les autorisations nécessaires. L’autorisation VIEW SERVER STATE a été refusée sur l’objet « {server} », base de données « {database} » | Cette erreur se produit si l’utilisateur n’est pas autorisé à effectuer la migration. Pour résoudre ce problème, reportez-vous à [GRANT - Octroi d'autorisations de serveur (Transact-SQL)](/sql/t-sql/statements/grant-server-permissions-transact-sql?view=sql-server-2017) ou au [Tutoriel : Migrer RDS SQL Server vers Azure à l’aide de DMS](./tutorial-rds-sql-server-azure-sql-and-managed-instance-online.md#prerequisites) pour plus d’informations. |

> [!NOTE]
> Pour plus d’informations sur la résolution des problèmes liés à la connexion à une instance AWS RDS SQL Server source, consultez les ressources suivantes :
>
> * [Résolution des erreurs de connectivité à SQL Server](https://support.microsoft.com/help/4009936/solving-connectivity-errors-to-sql-server)
> * [Comment résoudre les problèmes de connexion à une instance de base de données Amazon RDS ?](https://aws.amazon.com/premiumsupport/knowledge-center/rds-cannot-connect)

## <a name="known-issues"></a>Problèmes connus

* [Problèmes connus/limitations de migration dans le cadre des migrations en ligne vers Azure SQL Database](./known-issues-azure-sql-online.md)
* [Problèmes connus/limitations de migration dans le cadre des migrations en ligne vers Azure Database pour MySQL](./known-issues-azure-mysql-online.md)
* [Problèmes connus/limitations de migration dans le cadre des migrations en ligne vers Azure Database pour PostgreSQL](./known-issues-azure-postgresql-online.md)

## <a name="next-steps"></a>Étapes suivantes

* Consultez l’article [PowerShell pour Azure Database Migration Service](/powershell/module/azurerm.datamigration/?view=azurermps-6.13.0#data_migration).
* Consultez l’article [Guide pratique pour configurer des paramètres de serveur dans Azure Database pour MySQL à l’aide du portail Azure](../mysql/howto-server-parameters.md).
* Consultez l’article [Vue d’ensemble des prérequis pour l’utilisation d’Azure Database Migration Service](./pre-reqs.md).
* Consultez les [questions fréquentes (FAQ) sur l’utilisation d’Azure Database Migration Service](./faq.md).

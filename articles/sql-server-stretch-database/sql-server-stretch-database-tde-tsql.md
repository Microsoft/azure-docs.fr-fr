---
title: Activer Transparent Data Encryption pour Stretch Database (T-SQL)
description: Activer le chiffrement transparent des données (TDE) pour SQL Server Stretch Database sur Azure TSQL
services: sql-server-stretch-database
documentationcenter: ''
ms.assetid: 27753d91-9ca2-4d47-b34d-b5e2c2f029bb
ms.service: sql-server-stretch-database
ms.workload: data-management
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 01/23/2017
author: blazem-msft
ms.author: blazem
ms.reviewer: jroth
manager: jroth
ms.custom: seo-lt-2019
ms.openlocfilehash: 772341d046186e46b79ad7b11170e1bad23a3a6f
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024192"
---
# <a name="enable-transparent-data-encryption-tde-for-stretch-database-on-azure-transact-sql"></a>Activer le chiffrement transparent des données (TDE) pour Stretch Database sur Azure (Transact-SQL)
> [!div class="op_single_selector"]
> * [Azure portal](sql-server-stretch-database-encryption-tde.md)
> * [TSQL](sql-server-stretch-database-tde-tsql.md)
>
>

Le chiffrement transparent des données (Transparent Data Encryption, TDE) protège le système contre toute menace d’activité malveillante, en effectuant un chiffrement et un déchiffrement en temps réel de la base de données, des sauvegardes associées et des fichiers journaux de transactions au repos, sans qu’il soit nécessaire de modifier l’application.

Le chiffrement transparent des données chiffre le stockage d’une base de données entière à l’aide d’une clé symétrique appelée clé de chiffrement de base de données. La clé de chiffrement de base de données est protégée par un certificat de serveur intégré. Le certificat de serveur intégré est unique pour chaque serveur Azure. Microsoft fait automatiquement alterner ces certificats au moins tous les 90 jours. Pour obtenir une description générale de TDE, consultez [Chiffrement transparent des données (TDE)].

## <a name="enabling-encryption"></a>Activation du chiffrement
Pour activer le chiffrement transparent des données pour une base de données Azure qui stocke les données migrées à partir d’une base de données SQL Server compatible avec Stretch, procédez comme suit :

1. Connectez-vous à la base de données *master* sur le serveur Azure hébergeant la base de données à l’aide d’identifiants de connexion administrateurs ou membres du rôle **dbmanager** dans la base de données master
2. Exécutez l'instruction suivante pour chiffrer la base de données.

```sql
ALTER DATABASE [database_name] SET ENCRYPTION ON;
```

## <a name="disabling-encryption"></a>Désactivation du chiffrement
Pour désactiver le chiffrement transparent des données pour une base de données Azure qui stocke les données migrées à partir d’une base de données SQL Server compatible avec Stretch, procédez comme suit :

1. Connectez-vous à la base de données *master* à l'aide d'identifiants de connexion administrateurs ou membres du rôle **dbmanager** dans la base de données master
2. Exécutez l'instruction suivante pour chiffrer la base de données.

```sql
ALTER DATABASE [database_name] SET ENCRYPTION OFF;
```

## <a name="verifying-encryption"></a>Vérification de chiffrement
Pour vérifier l’état de chiffrement d’une base de données Azure qui stocke les données migrées à partir d’une base de données SQL Server compatible avec Stretch, procédez comme suit :

1. Connectez-vous à la base de données *master* ou d’instance à l'aide d'identifiants de connexion administrateurs ou membres du rôle **dbmanager** dans la base de données master
2. Exécutez l'instruction suivante pour chiffrer la base de données.

```sql
SELECT
    [name],
    [is_encrypted]
FROM
    sys.databases;
```

Un résultat de ```1``` indique une base de données chiffrée, ```0``` indique une base de données non chiffrée.

<!--Anchors-->
[Chiffrement transparent des données (TDE)]: /sql/relational-databases/security/encryption/transparent-data-encryption


<!--Image references-->

<!--Link references-->
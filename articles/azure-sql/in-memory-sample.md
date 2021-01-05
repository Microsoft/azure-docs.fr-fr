---
title: Exemple en mémoire
description: Essayez les technologies en mémoire d’Azure SQL Database avec l’exemple columnstore et OLTP.
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: sample
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: ''
ms.date: 12/18/2018
ms.openlocfilehash: b5a1035f8a213a6ce02dd3252ff7d3ddea46faf7
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92786579"
---
# <a name="in-memory-sample"></a>Exemple en mémoire
[!INCLUDE[appliesto-sqldb-sqlmi](includes/appliesto-sqldb-sqlmi.md)]

Les technologies en mémoire d’Azure SQL Database vous permettent d’améliorer les performances de votre application, et potentiellement de réduire le coût de votre base de données. En utilisant les technologies en mémoire dans Azure SQL Database, vous pouvez réaliser des améliorations de performance avec différentes charges de travail.

Dans cet article, vous verrez deux exemples qui illustrent l’utilisation d’OLTP en mémoire, ainsi que des index columnstore dans Azure SQL Database.

Pour plus d'informations, consultez les pages suivantes :

- [Présentation de l’OLTP en mémoire et scénarios d’utilisation](/sql/relational-databases/in-memory-oltp/overview-and-usage-scenarios) (avec notamment des références à des études de cas client et des informations de prise en main)
- [Documentation pour l’OLTP In-Memory](/sql/relational-databases/in-memory-oltp/in-memory-oltp-in-memory-optimization)
- [Description des index columnstore](/sql/relational-databases/indexes/columnstore-indexes-overview)
- Traitement hybride transactionnel et analytique (HTAP), également appelé [analytique opérationnelle en temps réel](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics)

<a id="install_oltp_manuallink" name="install_oltp_manuallink"></a>

&nbsp;

## <a name="1-install-the-in-memory-oltp-sample"></a>1. Installer l’exemple In-Memory OLTP

Vous pouvez créer l’exemple de base de données AdventureWorksLT en quelques clics dans le [portail Azure](https://portal.azure.com/). Les étapes de cette section expliquent comment enrichir votre base de données AdventureWorksLT d’objets OLTP en mémoire et démontrent les avantages au niveau des performances.

Pour une démonstration plus simple mais intéressante des performances de l’OLTP en mémoire, consultez :

- Version : [in-memory-oltp-demo-v1.0](https://github.com/Microsoft/sql-server-samples/releases/tag/in-memory-oltp-demo-v1.0)
- Code source : [in-memory-oltp-demo-source-code](https://github.com/microsoft/sql-server-samples/tree/master/samples/features/in-memory-database)

### <a name="installation-steps"></a>Procédure d’installation :

1. Dans le [portail Azure](https://portal.azure.com/), créez une base de données Premium ou Critique pour l’entreprise sur un serveur. Définissez comme valeur **Source** l’exemple de base de données AdventureWorksLT. Pour obtenir des instructions détaillées, consultez [Créer votre première base de données dans Azure SQL Database](database/single-database-create-quickstart.md).

2. Connectez-vous à la base de données avec [SQL Server Management Studio (SSMS.exe)](/sql/ssms/download-sql-server-management-studio-ssms).

3. Copiez le [script In-Memory OLTP Transact-SQL](https://raw.githubusercontent.com/microsoft/sql-server-samples/master/samples/features/in-memory-database/in-memory-oltp/t-sql-scripts/sql_in-memory_oltp_sample.sql) dans le Presse-papiers. Le script T-SQL crée les objets en mémoire nécessaires dans l’exemple de base de données AdventureWorksLT créé à l’étape 1.

4. Collez le script T-SQL dans SSMS, puis exécutez-le. Les instructions CREATE TABLE de la clause `MEMORY_OPTIMIZED = ON` sont essentielles. Par exemple :

```sql
CREATE TABLE [SalesLT].[SalesOrderHeader_inmem](
    [SalesOrderID] int IDENTITY NOT NULL PRIMARY KEY NONCLUSTERED ...,
    ...
) WITH (MEMORY_OPTIMIZED = ON);
```

### <a name="error-40536"></a>Erreur 40536

Si vous obtenez une erreur 40536 lorsque vous exécutez le script T-SQL, exécutez le script T-SQL suivant pour vérifier que la base de données prend en charge In-Memory :

```sql
SELECT DatabasePropertyEx(DB_Name(), 'IsXTPSupported');
```

Un résultat de **0** signifie que la technologie en mémoire n’est pas prise en charge, et un résultat de **1** signifie qu’elle l’est. Pour diagnostiquer le problème, vérifiez que la base de données se trouve sur le niveau de service Premium.

### <a name="about-the-created-memory-optimized-items"></a>À propos des éléments créés à mémoire optimisée.

**Tables** : L’exemple contient les tables à mémoire optimisée suivantes :

- SalesLT.Product_inmem
- SalesLT.SalesOrderHeader_inmem
- SalesLT.SalesOrderDetail_inmem
- Demo.DemoSalesOrderHeaderSeed
- Demo.DemoSalesOrderDetailSeed

Vous pouvez inspecter les tables à mémoire optimisée via **l’Explorateur d’objets** dans SSMS. Cliquez avec le bouton droit sur **Tables** > **Filtre** > **Paramètres de filtre** > **Est à mémoire optimisée** . La valeur est égale à 1.

Vous pouvez aussi interroger les vues de catalogue, telles que :

```sql
SELECT is_memory_optimized, name, type_desc, durability_desc
    FROM sys.tables
    WHERE is_memory_optimized = 1;
```

**Procédure stockée compilée en mode natif**  : Vous pouvez inspecter SalesLT.usp_InsertSalesOrder_inmem via une requête de vue de catalogue :

```sql
SELECT uses_native_compilation, OBJECT_NAME(object_id), definition
    FROM sys.sql_modules
    WHERE uses_native_compilation = 1;
```

&nbsp;

### <a name="run-the-sample-oltp-workload"></a>Exécuter l’exemple de charge de travail OLTP

La seule différence entre les deux *procédures stockées* est que la première utilise les versions à mémoire optimisée des tables, tandis que la deuxième utilise les tables sur disque régulières :

- SalesLT **.** usp_InsertSalesOrder **_inmem**
- SalesLT **.** usp_InsertSalesOrder **_ondisk**

Dans cette section, vous apprendrez à utiliser l’utilitaire **ostress.exe** , pratique pour exécuter les deux procédures stockées à des niveaux de contrainte élevés. Vous pouvez comparer le temps d’exécution des deux contraintes.

Lorsque vous exécutez ostress.exe, nous vous recommandons de transmettre des valeurs de paramètre conçues pour :

- Exécuter un grand nombre de connexions simultanées, en utilisant -n100.
- Répéter chaque boucle de connexion une centaine de fois, en utilisant -r500.

Toutefois, vous pouvez commencer avec des valeurs plus petites, telles que -n10 et -r50 pour vous assurer que tout fonctionne.

### <a name="script-for-ostressexe"></a>Script pour ostress.exe

Cette section affiche le script T-SQL incorporé à la ligne de commande ostress.exe. Le script utilise des éléments créés par le script T-SQL installé précédemment.

Le script suivant insère un exemple de commande client avec cinq lignes dans les *tables* à mémoire optimisée suivantes :

- SalesLT.SalesOrderHeader_inmem
- SalesLT.SalesOrderDetail_inmem

```sql
DECLARE
    @i int = 0,
    @od SalesLT.SalesOrderDetailType_inmem,
    @SalesOrderID int,
    @DueDate datetime2 = sysdatetime(),
    @CustomerID int = rand() * 8000,
    @BillToAddressID int = rand() * 10000,
    @ShipToAddressID int = rand() * 10000;

INSERT INTO @od
    SELECT OrderQty, ProductID
    FROM Demo.DemoSalesOrderDetailSeed
    WHERE OrderID= cast((rand()*60) as int);

WHILE (@i < 20)
begin;
    EXECUTE SalesLT.usp_InsertSalesOrder_inmem @SalesOrderID OUTPUT,
        @DueDate, @CustomerID, @BillToAddressID, @ShipToAddressID, @od;
    SET @i = @i + 1;
end
```

Pour créer la version *_ondisk* du script T-SQL précédent pour ostress.exe, il suffit de remplacer les deux occurrences de la sous-chaîne *_inmem* par *_ondisk* . Ces remplacements affectent les noms des tables et des procédures stockées.

#### <a name="install-rml-utilities-and-ostress"></a>Installer les utilitaires RML et `ostress`

Dans l’idéal, vous devez prévoir d’exécuter ostress.exe sur une machine virtuelle Azure. Vous devez créer une [machine virtuelle Azure](https://azure.microsoft.com/documentation/services/virtual-machines/) dans la même région géographique Azure que celle où se trouve votre base de données AdventureWorksLT. Vous pouvez aussi exécuter ostress.exe sur votre ordinateur portable.

Sur la machine virtuelle (ou sur l’hôte que vous avez choisi d’utiliser), installez les utilitaires RML. Ceux-ci incluent ostress.exe.

Pour plus d'informations, consultez les pages suivantes :

- La discussion sur ostress.exe dans [Exemple de base de données pour OLTP en mémoire](/sql/relational-databases/in-memory-oltp/sample-database-for-in-memory-oltp).
- [Exemple de base de données pour OLTP en mémoire](/sql/relational-databases/in-memory-oltp/sample-database-for-in-memory-oltp).
- Le [blog pour l’installation de ostress.exe](https://techcommunity.microsoft.com/t5/sql-server-support/cumulative-update-2-to-the-rml-utilities-for-microsoft-sql/ba-p/317910).

<!--
dn511655.aspx is for SQL 2014,
[Extensions to AdventureWorks to Demonstrate In-Memory OLTP]
(https://msdn.microsoft.com/library/dn511655&#x28;v=sql.120&#x29;.aspx)

whereas for SQL 2016+
[Sample Database for In-Memory OLTP]
(https://msdn.microsoft.com/library/mt465764.aspx)
-->

#### <a name="run-the-_inmem-stress-workload-first"></a>Commencer par exécuter la charge de travail de contrainte *_inmem*

Vous pouvez utiliser une fenêtre d’ *invite de commande RML* pour exécuter la ligne de commande ostress.exe. Les paramètres de ligne de commande indiquent à `ostress` d’effectuer les tâches suivantes :

- Exécuter 100 connexions simultanément (-n100).
- Chaque connexion doit exécuter le script T-SQL 50 fois (-r50).

```
ostress.exe -n100 -r50 -S<servername>.database.windows.net -U<login> -P<password> -d<database> -q -Q"DECLARE @i int = 0, @od SalesLT.SalesOrderDetailType_inmem, @SalesOrderID int, @DueDate datetime2 = sysdatetime(), @CustomerID int = rand() * 8000, @BillToAddressID int = rand() * 10000, @ShipToAddressID int = rand()* 10000; INSERT INTO @od SELECT OrderQty, ProductID FROM Demo.DemoSalesOrderDetailSeed WHERE OrderID= cast((rand()*60) as int); WHILE (@i < 20) begin; EXECUTE SalesLT.usp_InsertSalesOrder_inmem @SalesOrderID OUTPUT, @DueDate, @CustomerID, @BillToAddressID, @ShipToAddressID, @od; set @i += 1; end"
```

Pour exécuter la ligne de commande ostress.exe précédente :

1. Réinitialisez le contenu de la base de données en exécutant la commande suivante dans SSMS, pour supprimer toutes les données insérées lors des exécutions précédentes :

    ``` tsql
    EXECUTE Demo.usp_DemoReset;
    ```

2. Copiez le texte de la ligne de commande ostress.exe qui précède dans le presse-papiers.

3. Remplacez le `<placeholders>` des paramètres -S - U -P -d par les valeurs réelles correctes.

4. Exécutez la ligne de commande que vous avez modifiée dans la fenêtre de commande RML.

#### <a name="result-is-a-duration"></a>Il en résulte une durée

Lorsque `ostress.exe` est terminé, la durée d’exécution est indiquée sur la dernière ligne de sortie dans la fenêtre de commande RML. Par exemple, une série de tests plus courte a duré environ 1,5 minute :

`11/12/15 00:35:00.873 [0x000030A8] OSTRESS exiting normally, elapsed time: 00:01:31.867`

#### <a name="reset-edit-for-_ondisk-then-rerun"></a>Réinitialisez, paramétrez *_ondisk* , puis procédez à une nouvelle exécution.

Une fois le résultat de l’exécution de *_inmem* obtenu, effectuez les opérations suivantes pour l’exécution de *_ondisk* :

1. Réinitialisez la base de données en exécutant la commande suivante dans SSMS pour supprimer toutes les données insérées lors de l’exécution précédente :

   ```sql
   EXECUTE Demo.usp_DemoReset;
   ```

2. Modifiez la ligne de commande ostress.exe pour remplacer toutes les occurrences de *_inmem* par *_ondisk* .

3. Réexécutez ostress.exe une deuxième fois, puis enregistrez le résultat de durée.

4. Réinitialisez à nouveau la base de données (pour supprimer une quantité de données de test qui peut s’avérer conséquente).

#### <a name="expected-comparison-results"></a>Résultats de la comparaison attendus

Nos tests en mémoire montrent une multiplication par **9** de l’amélioration des performances pour cette charge de travail simple, avec `ostress` s’exécutant sur une machine virtuelle Azure dans la même région Azure que la base de données.

<a id="install_analytics_manuallink" name="install_analytics_manuallink"></a>

&nbsp;

## <a name="2-install-the-in-memory-analytics-sample"></a>2. Installer l’exemple In-Memory Analytics

Dans cette section, vous comparez les résultats des statistiques et les résultats d’E/S lors de l’utilisation d’un index columnstore par rapport à un index d’arborescence B traditionnel.

Pour l’analyse en temps réel sur une charge de travail OLTP, il est souvent préférable d’utiliser un index columnstore sans cluster. Pour plus d’informations, consultez [Index columnstore décrits](/sql/relational-databases/indexes/columnstore-indexes-overview).

### <a name="prepare-the-columnstore-analytics-test"></a>Préparer le test d’analyse columnstore

1. Utilisez le portail Azure pour créer une base de données AdventureWorksLT à partir de l’exemple.
   - Utilisez ce nom exact.
   - Choisissez un niveau de service Premium.

2. Copiez [sql_in-memory_analytics_sample](https://raw.githubusercontent.com/microsoft/sql-server-samples/master/samples/features/in-memory-database/in-memory-oltp/t-sql-scripts/sql_in-memory_analytics_sample.sql) dans le Presse-papiers.
   - Le script T-SQL crée les objets en mémoire nécessaires dans l’exemple de base de données AdventureWorksLT créé à l’étape 1.
   - Le script crée la table Dimension et deux tables de faits. Les tables de faits comprennent 3,5 millions de lignes chacune.
   - Le script peut prendre 15 minutes pour s’exécuter.

3. Collez le script T-SQL dans SSMS, puis exécutez-le. Le mot clé **COLUMNSTORE** est essentiel dans l’instruction **CREATE INDEX** , comme dans l’exemple ci-dessous :<br/>`CREATE NONCLUSTERED COLUMNSTORE INDEX ...;`

4. Définissez AdventureWorksLT sur le niveau de compatibilité 130 :<br/>`ALTER DATABASE AdventureworksLT SET compatibility_level = 130;`

    Le niveau 130 n’est pas directement lié aux fonctionnalités In-Memory. Mais le niveau 130 offre généralement des performances de requêtes supérieures au niveau 120.

#### <a name="key-tables-and-columnstore-indexes"></a>Tables et index columnstore essentiels

- dbo.FactResellerSalesXL_CCI est une table contenant un index columnstore en cluster, ce qui permet la compression avancée au niveau des *données* .

- dbo.FactResellerSalesXL_PageCompressed est une table qui possède un index en cluster régulier équivalent, compressé uniquement au niveau de la *page* .

#### <a name="key-queries-to-compare-the-columnstore-index"></a>Requêtes essentielles pour comparer l’index columnstore

Il existe [plusieurs types de requête T-SQL que vous pouvez exécuter](https://raw.githubusercontent.com/microsoft/sql-server-samples/master/samples/features/in-memory-database/in-memory-oltp/t-sql-scripts/clustered_columnstore_sample_queries.sql) pour mettre en évidence les améliorations des performances. À l’étape 2 dans le script T-SQL, soyez attentif à ces deux requêtes. Elles diffèrent uniquement d’une ligne :

- `FROM FactResellerSalesXL_PageCompressed a`
- `FROM FactResellerSalesXL_CCI a`

Un index columnstore en cluster se trouve dans la table FactResellerSalesXL\_CCI.

L’extrait de script T-SQL suivant imprime des statistiques d’E/S et d’heure pour la requête de chaque table.

```sql
/*********************************************************************
Step 2 -- Overview
-- Page Compressed BTree table v/s Columnstore table performance differences
-- Enable actual Query Plan in order to see Plan differences when Executing
*/
-- Ensure Database is in 130 compatibility mode
ALTER DATABASE AdventureworksLT SET compatibility_level = 130
GO

-- Execute a typical query that joins the Fact Table with dimension tables
-- Note this query will run on the Page Compressed table, Note down the time
SET STATISTICS IO ON
SET STATISTICS TIME ON
GO

SELECT c.Year
    ,e.ProductCategoryKey
    ,FirstName + ' ' + LastName AS FullName
    ,count(SalesOrderNumber) AS NumSales
    ,sum(SalesAmount) AS TotalSalesAmt
    ,Avg(SalesAmount) AS AvgSalesAmt
    ,count(DISTINCT SalesOrderNumber) AS NumOrders
    ,count(DISTINCT a.CustomerKey) AS CountCustomers
FROM FactResellerSalesXL_PageCompressed a
INNER JOIN DimProduct b ON b.ProductKey = a.ProductKey
INNER JOIN DimCustomer d ON d.CustomerKey = a.CustomerKey
Inner JOIN DimProductSubCategory e on e.ProductSubcategoryKey = b.ProductSubcategoryKey
INNER JOIN DimDate c ON c.DateKey = a.OrderDateKey
GROUP BY e.ProductCategoryKey,c.Year,d.CustomerKey,d.FirstName,d.LastName
GO
SET STATISTICS IO OFF
SET STATISTICS TIME OFF
GO


-- This is the same Prior query on a table with a clustered columnstore index CCI
-- The comparison numbers are even more dramatic the larger the table is (this is an 11 million row table only)
SET STATISTICS IO ON
SET STATISTICS TIME ON
GO
SELECT c.Year
    ,e.ProductCategoryKey
    ,FirstName + ' ' + LastName AS FullName
    ,count(SalesOrderNumber) AS NumSales
    ,sum(SalesAmount) AS TotalSalesAmt
    ,Avg(SalesAmount) AS AvgSalesAmt
    ,count(DISTINCT SalesOrderNumber) AS NumOrders
    ,count(DISTINCT a.CustomerKey) AS CountCustomers
FROM FactResellerSalesXL_CCI a
INNER JOIN DimProduct b ON b.ProductKey = a.ProductKey
INNER JOIN DimCustomer d ON d.CustomerKey = a.CustomerKey
Inner JOIN DimProductSubCategory e on e.ProductSubcategoryKey = b.ProductSubcategoryKey
INNER JOIN DimDate c ON c.DateKey = a.OrderDateKey
GROUP BY e.ProductCategoryKey,c.Year,d.CustomerKey,d.FirstName,d.LastName
GO

SET STATISTICS IO OFF
SET STATISTICS TIME OFF
GO
```

Dans une base de données ayant le niveau tarifaire P2, vous pouvez attendre une multiplication par 9 des performances de cette requête avec l’index columnstore en cluster par rapport à l’index traditionnel. Avec P15, vous pouvez vous attendre à une multiplication des performances par 57 à l’aide de l’index columnstore.

## <a name="next-steps"></a>Étapes suivantes

- [Démarrage rapide 1 : Technologies OLTP en mémoire pour des performances T-SQL plus rapides](/sql/relational-databases/in-memory-oltp/survey-of-initial-areas-in-in-memory-oltp)

- [Utiliser OLTP en mémoire dans une application SQL Azure existante](in-memory-oltp-configure.md)

- [Surveiller le stockage OLTP en mémoire](in-memory-oltp-monitor-space.md) pour l’OLTP en mémoire

## <a name="additional-resources"></a>Ressources supplémentaires

### <a name="deeper-information"></a>Informations supplémentaires

- [Découvrez comment le Quorum double la charge de travail de la base de données clé tout en réduisant les DTU de 70 %, grâce à l’OLTP en mémoire dans Azure SQL Database](https://customers.microsoft.com/story/quorum-doubles-key-databases-workload-while-lowering-dtu-with-sql-database)

- [Billet de blog OLTP en mémoire dans Azure SQL Database](https://azure.microsoft.com/blog/in-memory-oltp-in-azure-sql-database/)

- [En savoir plus sur In-Memory OLTP](/sql/relational-databases/in-memory-oltp/in-memory-oltp-in-memory-optimization)

- [En savoir plus sur les index columnstore](/sql/relational-databases/indexes/columnstore-indexes-overview)

- [En savoir plus sur l’analytique opérationnelle en temps réel](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics)

- Consultez [Modèles de charge de travail courants et considérations relatives à la migration](/previous-versions/dn673538(v=msdn.10)) (qui décrit des modèles de charge de travail où l’OLTP en mémoire apporte généralement des gains de performances significatifs)

#### <a name="application-design"></a>Conception des applications

- [In-Memory OLTP (optimisation en mémoire)](/sql/relational-databases/in-memory-oltp/in-memory-oltp-in-memory-optimization)

- [Utiliser OLTP en mémoire dans une application SQL Azure existante](in-memory-oltp-configure.md)

#### <a name="tools"></a>Outils

- [Azure portal](https://portal.azure.com/)

- [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms)

- [Outils SQL Server Data Tools (SSDT)](/sql/ssdt/download-sql-server-data-tools-ssdt)
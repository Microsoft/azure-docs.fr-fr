---
title: Guide pratique pour utiliser le traitement par lot afin d’améliorer les performances des applications
description: Cette rubrique explique comment les opérations de traitement par lot appliquées aux bases de données contribuent à améliorer la rapidité et la scalabilité de vos applications Azure SQL Database et Azure SQL Managed Instance. Bien que ces techniques de traitement par lot fonctionnent pour toutes les bases de données SQL, cet article porte exclusivement sur Azure.
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: genemi
ms.date: 01/25/2019
ms.openlocfilehash: 07334d62cee94be8b5b8dd6188c1d6354c4d584b
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92792597"
---
# <a name="how-to-use-batching-to-improve-azure-sql-database-and-azure-sql-managed-instance-application-performance"></a>Utiliser le traitement par lot pour améliorer les performances des applications Azure SQL Database et Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](includes/appliesto-sqldb-sqlmi.md)]

Les opérations de traitement par lot améliorent considérablement les performances et la scalabilité de vos applications Azure SQL Database et Azure SQL Managed Instance. Pour en comprendre les avantages, la première partie de cet article présente des résultats de test qui comparent des demandes séquentielles à des demandes par lot exécutées sur une base de données Azure SQL Database ou Azure SQL Managed Instance. Le reste de cet article décrit des techniques, des scénarios et des remarques à prendre en compte pour vous aider à utiliser efficacement le traitement par lots dans vos applications Azure.

## <a name="why-is-batching-important-for-azure-sql-database-and-azure-sql-managed-instance"></a>En quoi le traitement par lot est-il important pour Azure SQL Database et Azure SQL Managed Instance ?

L’envoi d’appels de traitement par lots sur un service distant constitue une stratégie réputée pour améliorer les performances et l’évolutivité. Toute interaction avec un service distant, notamment la sérialisation, le transfert réseau et la désérialisation, entraîne des coûts de traitement fixes. L’empaquetage de plusieurs transactions distinctes dans un seul lot contribue à réduire ces coûts.

Dans cet article, nous allons examiner différents scénarios et stratégies de traitement par lot. Bien que ces stratégies soient également importantes pour les applications locales qui utilisent SQL Server, il convient d'insister sur le caractère essentiel de l'utilisation du traitement par lot pour Azure SQL Database et Azure SQL Managed Instance :

* L'accès à Azure SQL Database et Azure SQL Managed Instance présente un plus grand risque de latence du réseau, en particulier si vous accédez à Azure SQL Database ou Azure SQL Managed Instance depuis l'extérieur du même centre de données Microsoft Azure.
* Compte tenu des caractéristiques multilocataires d'Azure SQL Database et Azure SQL Managed Instance, l'efficacité de la couche d'accès aux données est en corrélation avec la scalabilité globale de la base de données. En réponse à une utilisation dépassant les quotas prédéfinis, Azure SQL Database and Azure SQL Managed Instance peuvent réduire le débit ou répondre par des exceptions de limitation. Les mesures d'efficacité telles que le traitement par lot vous permettent d'accomplir davantage avant d'atteindre ces limites.
* Le traitement par lots est également efficace pour les architectures qui utilisent plusieurs bases de données (partitionnement). L’efficacité de votre interaction avec chaque unité de base de données demeure un facteur clé pour votre évolutivité globale.

L'un des avantages associés à l'utilisation d'Azure SQL Database ou Azure SQL Managed Instance est que vous n'avez pas à gérer les serveurs qui hébergent la base de données. Toutefois, cette infrastructure gérée signifie également que vous devez adopter une approche différente pour l’optimisation de la base de données. Vous ne pouvez plus chercher à améliorer l’infrastructure matérielle ou réseau de la base de données. Ces environnements sont directement contrôlés par Microsoft Azure. Votre rayon de contrôle couvre essentiellement la manière dont votre application interagit avec Azure SQL Database et Azure SQL Managed Instance. Le traitement par lots constitue l’une de ces optimisations.

La première partie de cet article examine différentes techniques de traitement par lot pour les applications .NET qui utilisent Azure SQL Database ou Azure SQL Managed Instance. Les deux dernières sections décrivent des recommandations et des scénarios de traitement par lots.

## <a name="batching-strategies"></a>Stratégies de traitement par lots

### <a name="note-about-timing-results-in-this-article"></a>Remarque relative aux résultats de minutage fournis dans cet article

> [!NOTE]
> Les résultats ne représentent pas des valeurs de référence, mais des **performances relatives** . Les minutages reposent sur une moyenne calculée à partir d’au moins 10 séries de tests. Les opérations consistent en des insertions dans une table vide. Ces tests ont été mesurés avant la V12 et ne correspondent pas nécessairement au débit que vous pourriez obtenir avec une base de données V12 utilisant les nouveaux [niveaux de service DTU](database/service-tiers-dtu.md) ou [niveaux de service vCore](database/service-tiers-vcore.md). L’avantage relatif de la technique de traitement par lots doit être similaire.

### <a name="transactions"></a>Transactions

Il peut sembler étrange d’aborder la question du traitement par lots par la notion de transactions. Mais l’utilisation de transactions côté client a un effet subtil côté serveur subtil qui améliore les performances. Les transactions peuvent être ajoutées avec seulement quelques lignes de code, afin de fournir un moyen rapide d’améliorer les performances des opérations séquentielles.

Examinez le code C# suivant qui contient une séquence d’opérations d’insertion et de mise à jour sur une table simple.

```csharp
List<string> dbOperations = new List<string>();
dbOperations.Add("update MyTable set mytext = 'updated text' where id = 1");
dbOperations.Add("update MyTable set mytext = 'updated text' where id = 2");
dbOperations.Add("update MyTable set mytext = 'updated text' where id = 3");
dbOperations.Add("insert MyTable values ('new value',1)");
dbOperations.Add("insert MyTable values ('new value',2)");
dbOperations.Add("insert MyTable values ('new value',3)");
```

Le code ADO.NET suivant exécute ces opérations de manière séquentielle.

```csharp
using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
{
    conn.Open();

    foreach(string commandString in dbOperations)
    {
        SqlCommand cmd = new SqlCommand(commandString, conn);
        cmd.ExecuteNonQuery();
    }
}
```

La meilleure façon d’optimiser ce code consiste à implémenter une forme de traitement par lots de ces appels côté client. Mais il existe un moyen simple d’augmenter les performances de ce code en encapsulant simplement la séquence d’appels dans une transaction. Voici le même code qui utilise une transaction.

```csharp
using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
{
    conn.Open();
    SqlTransaction transaction = conn.BeginTransaction();

    foreach (string commandString in dbOperations)
    {
        SqlCommand cmd = new SqlCommand(commandString, conn, transaction);
        cmd.ExecuteNonQuery();
    }

    transaction.Commit();
}
```

Les transactions sont en fait utilisées dans ces deux exemples. Dans le premier exemple, chaque appel individuel est une transaction implicite. Dans le deuxième exemple, une transaction explicite encapsule tous les appels. Conformément à la documentation du [journal des transactions à écriture anticipée](/sql/relational-databases/sql-server-transaction-log-architecture-and-management-guide?view=sql-server-ver15#WAL), les enregistrements de journal sont vidés sur le disque lorsque la transaction est validée. Par conséquent, en incluant plusieurs appels dans une transaction, l’écriture dans le journal des transactions peut être retardée jusqu’à ce que la transaction soit validée. En effet, vous activez le traitement par lots pour les écritures effectuées dans le journal des transactions du serveur.

Le tableau suivant présente quelques résultats de tests ad hoc. Les tests ont consisté à exécuter les mêmes insertions séquentielles avec et sans transactions. Pour plus de perspective, la première série de tests a été exécutée à distance entre un ordinateur portable et la base de données dans Microsoft Azure. La deuxième série de tests a été exécutée depuis un service cloud et une base de données qui résidaient dans le même centre de données Microsoft Azure (USA Ouest). Le tableau suivant indique la durée en millisecondes des insertions séquentielles avec et sans transactions.

**Local vers Azure** :

| Opérations | Sans transaction (ms) | Avec transaction (ms) |
| --- | --- | --- |
| 1 |130 |402 |
| 10 |1208 |1226 |
| 100 |12 662 |10 395 |
| 1 000 |128 852 |102 917 |

**Azure vers Azure (même centre de données)** :

| Opérations | Sans transaction (ms) | Avec transaction (ms) |
| --- | --- | --- |
| 1 |21 |26 |
| 10 |220 |56 |
| 100 |2 145 |341 |
| 1 000 |21 479 |2756 |

> [!NOTE]
> Les résultats ne représentent pas des valeurs de référence. Voir la [remarque relative au minutage fournie dans cet article](#note-about-timing-results-in-this-article)

Compte tenu des résultats des tests précédents, l’encapsulation d’une seule opération dans une transaction a réellement pour effet de réduire les performances. Mais lorsque vous augmentez le nombre d’opérations dans une même transaction, vous obtenez une amélioration de performances plus marquée. La différence de performances est également plus manifeste lorsque toutes les opérations interviennent au sein du centre de données Microsoft Azure. L'augmentation du phénomène de latence associée à l'utilisation d'Azure SQL Database or Azure SQL Managed Instance à l'extérieur du centre de données Microsoft Azure masque en partie le gain de performances lié à l'utilisation de transactions.

Bien que l’utilisation de transactions puisse augmenter les performances, nous vous invitons à [respecter les meilleures pratiques en matière de connexions et de transactions](/previous-versions/sql/sql-server-2008-r2/ms187484(v=sql.105)). Utilisez la transaction la plus courte possible et fermez la connexion à la base de données une fois la tâche terminée. L’instruction using dans l’exemple précédent garantit la fermeture de la connexion à la fin de l’exécution du bloc de code suivant.

L’exemple précédent montre que vous pouvez ajouter une transaction locale au code ADO.NET avec deux lignes. Les transactions offrent un moyen rapide d’améliorer les performances du code qui génère les opérations d’insertion, de mise à jour et de suppression séquentielles. Toutefois, pour de meilleures performances, vous devriez apporter d’autres modifications au code afin de tirer parti des avantages du traitement par lots côté client, tels que les paramètres table.

Pour plus d’informations sur les transactions dans ADO.NET, consultez [Transactions locales dans ADO.NET](/dotnet/framework/data/adonet/local-transactions).

### <a name="table-valued-parameters"></a>Paramètres table

Les paramètres table prennent en charge les types de tables définis par l’utilisateur en tant que paramètres dans les instructions Transact-SQL, en tant que procédures stockées et en tant que fonctions. Cette technique de traitement par lots côté client vous permet d’envoyer plusieurs lignes de données dans le paramètre table. Pour utiliser les paramètres table, commencez par définir un type de table. L’instruction Transact-SQL suivante crée un type de table nommé **MyTableType** .

```sql
    CREATE TYPE MyTableType AS TABLE
    ( mytext TEXT,
      num INT );
```

Dans le code, vous créez un **DataTable** comportant exactement les mêmes noms et types que le type de table. Transférez ce **DataTable** dans un paramètre via une requête texte ou un appel de procédure stockée. L’exemple ci-après illustre cette technique :

```csharp
using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
{
    connection.Open();

    DataTable table = new DataTable();
    // Add columns and rows. The following is a simple example.
    table.Columns.Add("mytext", typeof(string));
    table.Columns.Add("num", typeof(int));
    for (var i = 0; i < 10; i++)
    {
        table.Rows.Add(DateTime.Now.ToString(), DateTime.Now.Millisecond);
    }

    SqlCommand cmd = new SqlCommand(
        "INSERT INTO MyTable(mytext, num) SELECT mytext, num FROM @TestTvp",
        connection);

    cmd.Parameters.Add(
        new SqlParameter()
        {
            ParameterName = "@TestTvp",
            SqlDbType = SqlDbType.Structured,
            TypeName = "MyTableType",
            Value = table,
        });

    cmd.ExecuteNonQuery();
}
```

Dans l’exemple précédent, l’objet **SqlCommand** insère des lignes à partir d’un paramètre table, **\@TestTvp** . L’objet **DataTable** créé précédemment est assigné à ce paramètre à l’aide de la méthode **SqlCommand.Parameters.Add** . Le traitement par lots des insertions dans un seul appel augmente considérablement les performances sur les insertions séquentielles.

Pour améliorer l’exemple précédent, utilisez une procédure stockée au lieu d’une commande de texte. La commande Transact-SQL suivante crée une procédure stockée qui utilise le paramètre table **SimpleTestTableType** .

```sql
CREATE PROCEDURE [dbo].[sp_InsertRows]
@TestTvp as MyTableType READONLY
AS
BEGIN
INSERT INTO MyTable(mytext, num)
SELECT mytext, num FROM @TestTvp
END
GO
```

Modifiez comme suit la déclaration d’objet **SqlCommand** dans l’exemple de code précédent.

```csharp
SqlCommand cmd = new SqlCommand("sp_InsertRows", connection);
cmd.CommandType = CommandType.StoredProcedure;
```

Dans la plupart des cas, les paramètres table présentent des performances équivalentes ou supérieures à celles des autres techniques de traitement par lots. Les paramètres table sont souvent préférables car ils apportent davantage de flexibilité que les autres options. Par exemple, les autres techniques, telles que la copie en bloc SQL, autorisent uniquement l’insertion de nouvelles lignes. Mais avec les paramètres table, vous pouvez utiliser une logique dans la procédure stockée pour déterminer quelles lignes sont des mises à jour et quelles lignes sont des insertions. Le type de table peut aussi être modifié pour contenir une colonne « Opération » qui indique si la ligne spécifiée doit être insérée, mise à jour ou supprimée.

Le tableau suivant présente les résultats de tests ad hoc pour l’utilisation des paramètres table en millisecondes.

| Opérations | Local vers Azure (ms) | Azure (même centre de données) |
| --- | --- | --- |
| 1 |124 |32 |
| 10 |131 |25 |
| 100 |338 |51 |
| 1 000 |2 615 |382 |
| 10000 |23 830 |3 586 |

> [!NOTE]
> Les résultats ne représentent pas des valeurs de référence. Voir la [remarque relative au minutage fournie dans cet article](#note-about-timing-results-in-this-article)

Le gain de performances associé au traitement par lots est immédiatement évident. Dans le test séquentiel précédent, 1 000 opérations prenaient 129 secondes à l’extérieur du centre de données et 21 secondes à l’intérieur du centre de données. Mais avec des paramètres table, ces 1 000 opérations ne prennent que 2,6 secondes à l’extérieur du centre de données et 0,4 seconde à l’intérieur du centre de données.

Pour plus d’informations sur les paramètres table, consultez [Paramètres table](/sql/relational-databases/tables/use-table-valued-parameters-database-engine).

### <a name="sql-bulk-copy"></a>Copie en bloc SQL

La copie en bloc SQL est une autre façon d’insérer de grandes quantités de données dans une base de données cible. Les applications .NET peuvent utiliser la classe **SqlBulkCopy** pour effectuer des opérations d’insertion en bloc. Le fonctionnement de la classe **SqlBulkCopy** est similaire à celui de l’outil en ligne de commande **Bcp.exe** ou de l’instruction Transact-SQL, **BULK INSERT** . L'exemple de code suivant montre comment copier en bloc les lignes de la table source **DataTable** dans la table de destination, MyTable.

```csharp
using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
{
    connection.Open();

    using (SqlBulkCopy bulkCopy = new SqlBulkCopy(connection))
    {
        bulkCopy.DestinationTableName = "MyTable";
        bulkCopy.ColumnMappings.Add("mytext", "mytext");
        bulkCopy.ColumnMappings.Add("num", "num");
        bulkCopy.WriteToServer(table);
    }
}
```

Il existe quelques cas où la copie en bloc est préférable à l’utilisation des paramètres table. Consultez le tableau de comparaison des paramètres table aux opérations BULK INSERT dans l’article [Paramètres table](/sql/relational-databases/tables/use-table-valued-parameters-database-engine).

Les résultats des tests ad hoc suivants montrent les performances du traitement par lots avec **SqlBulkCopy** en millisecondes.

| Opérations | Local vers Azure (ms) | Azure (même centre de données) |
| --- | --- | --- |
| 1 |433 |57 |
| 10 |441 |32 |
| 100 |636 |53 |
| 1 000 |2 535 |341 |
| 10000 |21605 |2 737 |

> [!NOTE]
> Les résultats ne représentent pas des valeurs de référence. Voir la [remarque relative au minutage fournie dans cet article](#note-about-timing-results-in-this-article)

Dans les lots plus petits, l’utilisation des paramètres table a permis d’obtenir de meilleures performances que la classe **SqlBulkCopy** . Pour les tests sur 1 000 et 10 000 lignes en revanche, l’utilisation de **SqlBulkCopy** a affiché des performances de 12 à 31 % plus rapides que les paramètres table. Comme les paramètres table, **SqlBulkCopy** est une bonne option pour les insertions par lots, en particulier comparativement aux performances des opérations non traitées par lots.

Pour plus d'informations sur la copie en bloc dans ADO.NET, consultez [Opérations de copie en bloc](/dotnet/framework/data/adonet/sql/bulk-copy-operations-in-sql-server).

### <a name="multiple-row-parameterized-insert-statements"></a>Instructions INSERT paramétrables sur plusieurs lignes

Une solution pour les petits lots consiste à construire une grande instruction INSERT paramétrable permettant d’insérer plusieurs lignes. L’exemple de code suivant illustre cette technique.

```csharp
using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
{
    connection.Open();

    string insertCommand = "INSERT INTO [MyTable] ( mytext, num ) " +
        "VALUES (@p1, @p2), (@p3, @p4), (@p5, @p6), (@p7, @p8), (@p9, @p10)";

    SqlCommand cmd = new SqlCommand(insertCommand, connection);

    for (int i = 1; i <= 10; i += 2)
    {
        cmd.Parameters.Add(new SqlParameter("@p" + i.ToString(), "test"));
        cmd.Parameters.Add(new SqlParameter("@p" + (i+1).ToString(), i));
    }

    cmd.ExecuteNonQuery();
}
```

Cet exemple vise à illustrer le concept de base. Un scénario plus réaliste effectuerait une boucle via les entités nécessaires à la construction simultanée de la chaîne de requête et des paramètres de commande. Vous êtes limité à un total de paramètres de 2 100 paramètres de requête, ce qui limite le nombre total de lignes pouvant être traitées de cette manière.

Les résultats des tests ad hoc suivants montrent les performances de ce type d’instruction d’insertion en millisecondes.

| Opérations | Paramètres table (ms) | Instruction INSERT unique (ms) |
| --- | --- | --- |
| 1 |32 |20 |
| 10 |30 |25 |
| 100 |33 |51 |

> [!NOTE]
> Les résultats ne représentent pas des valeurs de référence. Voir la [remarque relative au minutage fournie dans cet article](#note-about-timing-results-in-this-article)

Cette approche peut être légèrement plus rapide pour les lots comportant moins de 100 lignes. Bien que l’amélioration soit négligeable, cette technique constitue une autre solution potentiellement efficace dans votre scénario d’application spécifique.

### <a name="dataadapter"></a>DataAdapter

La classe **DataAdapter** vous permet de modifier un objet **DataSet** puis de soumettre les modifications en tant qu’opérations INSERT, UPDATE et DELETE. Si vous utilisez la classe **DataAdapter** de cette manière, il est important de noter que les appels distincts sont effectués pour chaque opération distincte. Pour améliorer les performances, utilisez la propriété **UpdateBatchSize** sur le nombre d’opérations devant être traitées par lot simultanément. Pour plus d’informations, consultez [Exécution de traitements par lots à l’aide de DataAdapters](/dotnet/framework/data/adonet/performing-batch-operations-using-dataadapters).

### <a name="entity-framework"></a>Entity Framework

[Entity Framework Core](/ef/efcore-and-ef6/#saving-data) prend désormais en charge le traitement par lot.

### <a name="xml"></a>XML

Par souci d’exhaustivité, nous estimons qu’il est important de considérer XML comme une stratégie de traitement par lots. Toutefois, l’utilisation du langage XML n’apporte aucun avantage par rapport aux autres méthodes et présente même plusieurs inconvénients. L’approche est similaire aux paramètres table, à ceci près qu’un fichier ou une chaîne XML, et non une table définie par l’utilisateur, est transféré vers une procédure stockée. La procédure stockée analyse les commandes dans la procédure stockée.

Cette approche présente plusieurs inconvénients :

* L’utilisation de XML peut s’avérer fastidieuse et sujette aux erreurs.
* L’analyse du fichier ou de la chaîne XML sur la base de données peut être gourmande en ressources processeur.
* Dans la plupart des cas, cette méthode est plus lente que celle des paramètres table.

Pour ces raisons, l’utilisation de XML pour les requêtes par lots n’est pas recommandée.

## <a name="batching-considerations"></a>Remarques relatives au traitement par lots

Les sections suivantes fournissent des conseils supplémentaires sur l'utilisation du traitement par lot dans les applications Azure SQL Database et Azure SQL Managed Instance.

### <a name="tradeoffs"></a>Compromis

En fonction de votre architecture, le traitement par lots peut vous obliger à faire un compromis entre performances et résilience. Par exemple, pensez à une situation où votre rôle rencontre une défaillance inattendue. Si vous perdez une ligne de données, l’impact sera moins important que celui associé à la perte d’un grand lot de lignes qui n’ont pas été envoyées. Le risque est d’autant plus élevé lorsque vous placez les lignes dans la mémoire tampon avant de les envoyer à la base de données dans un laps de temps spécifié.

Ce compromis doit vous amener à évaluer le type d’opérations que vous souhaitez traiter par lots. Optez pour une approche plus agressive (lots plus volumineux et intervalles plus longs) pour les données moins critiques.

### <a name="batch-size"></a>Taille du lot

Dans nos tests, il n’y avait généralement aucun avantage à fractionner les lots volumineux en plusieurs petits segments. En fait, cette sous-division s’est souvent traduite par des performances plus lentes que celles obtenues avec l’envoi d’un seul lot plus volumineux. Imaginez par exemple un scénario où vous souhaitez insérer 1 000 lignes. Le tableau suivant indique la durée nécessaire aux paramètres table pour insérer 1 000 lignes divisées en lots plus petits.

| Taille du lot | Itérations | Paramètres table (ms) |
| --- | --- | --- |
| 1 000 |1 |347 |
| 500 |2 |355 |
| 100 |10 |465 |
| 50 |20 |630 |

> [!NOTE]
> Les résultats ne représentent pas des valeurs de référence. Voir la [remarque relative au minutage fournie dans cet article](#note-about-timing-results-in-this-article)

Vous pouvez voir que, pour 1 000 lignes, on obtient les meilleures performances en les soumettant toutes en même temps. D’autres tests (qui ne sont pas présentés ici) ont révélé un faible gain de performances en divisant un lot de 10 000 lignes en deux lots de 5 000. Mais le schéma de table pour ces tests étant relativement simple, vous devriez effectuer les tests sur vos données et tailles de lot spécifiques afin de vérifier ces résultats.

Autre facteur à prendre en compte : si le lot total devient trop volumineux, Azure SQL Database ou Azure SQL Managed Instance risque de subir des limitations et de refuser de valider le lot. Pour de meilleurs résultats, testez votre scénario spécifique afin de déterminer s’il comporte une taille de lot idéale. Faites en sorte que la taille de lot puisse être configurée pendant l’exécution afin de permettre des modifications rapides compte tenu des performances ou des erreurs obtenues.

Enfin, équilibrez la taille du lot en fonction des risques liés au traitement par lots. Si vous obtenez des erreurs temporaires ou si le rôle échoue, tenez compte des conséquences associées à une nouvelle tentative de l’opération ou à la perte de données dans le lot.

### <a name="parallel-processing"></a>Traitement parallèle

Que se passe-t-il si vous avez adopté l’approche consistant à réduire la taille de lot mais que vous avez utilisé plusieurs threads pour exécuter la tâche ? Là encore, nos tests ont montré que plusieurs petits lots multithreads produisaient de moins bonnes performances que celles obtenues avec un seul lot plus volumineux. Le test suivant tente d’insérer 1 000 lignes dans un ou plusieurs lots parallèles. Il montre comment un plus grand nombre de lots simultanés affecte les performances.

| Taille de lot [itérations] | Deux threads (ms) | Quatre threads (ms) | Six threads (ms) |
| --- | --- | --- | --- |
| 1 000 [1] |277 |315 |266 |
| 500 [2] |548 |278 |256 |
| 250 [4] |405 |329 |265 |
| 100 [10] |488 |439 |391 |

> [!NOTE]
> Les résultats ne représentent pas des valeurs de référence. Voir la [remarque relative au minutage fournie dans cet article](#note-about-timing-results-in-this-article)

Plusieurs raisons peuvent potentiellement expliquer une dégradation des performances liées au parallélisme :

* Exécution de plusieurs appels réseau simultanés au lieu d’un seul.
* Plusieurs opérations sur une seule table peuvent entraîner une contention et un blocage.
* Surcharges associées multithreading.
* Avantage du traitement parallèle atténué par le coût associé à l’ouverture de plusieurs connexions.

Si vous ciblez différentes tables ou bases de données, vous pouvez constater un gain de performances grâce à cette stratégie. Le partitionnement ou les fédérations de bases de données constitue un scénario possible pour une telle approche. Le partitionnement utilise plusieurs bases de données et achemine des données différentes à chaque base de données. Si chaque petit lot est transmis à une base de données différente, l’exécution d’opérations en parallèle peut gagner en efficacité. Mais le gain de performances n’est pas assez important pour influer sur la décision d’utiliser ou non le partitionnement de base de données dans votre solution.

Dans certains modèles, l’exécution parallèle de lots plus petits peut entraîner une amélioration du débit des demandes dans un système sous charge. Dans ce cas, même s’il est plus rapide de traiter un seul lot plus volumineux, le traitement de plusieurs lots en parallèle peut se révéler plus efficace.

Si vous utilisez une exécution parallèle, veillez à contrôler le maximum de threads de travail. Un plus petit nombre peut réduire la contention et accélérer la durée d’exécution. Tenez également compte de la charge supplémentaire qui pèsera sur la base de données cible tant au niveau des connexions que des transactions.

### <a name="related-performance-factors"></a>Facteurs de performances associés

Les conseils classiques sur les performances de base de données s’appliquent également au traitement par lots. Par exemple, les performances d’insertion sont réduites pour les tables qui ont une grande clé primaire ou de nombreux index non ordonnés en clusters.

Si les paramètres table utilisent une procédure stockée, vous pouvez utiliser la commande **SET NOCOUNT ON** au début de la procédure. Cette instruction supprime le retour du nombre de lignes affectées dans la procédure. Toutefois, dans nos tests, l’utilisation de **SET NOCOUNT ON** n’avait aucun effet sur les performances, voire les dégradait. La procédure stockée de test reposait simplement sur l’exécution d’une seule commande **INSERT** à partir du paramètre table. Il est possible que les procédures stockées plus complexes puissent bénéficier de cette instruction. Mais ne partez pas du principe que l’ajout de la commande **SET NOCOUNT ON** à votre procédure stockée améliorera systématiquement les performances. Pour en comprendre l’impact, testez votre procédure stockée avec et sans l’instruction **SET NOCOUNT ON** .

## <a name="batching-scenarios"></a>Scénarios de traitement par lots

Les sections suivantes expliquent comment utiliser les paramètres table dans trois scénarios d’application. Le premier scénario montre comment utiliser une mise en mémoire tampon parallèlement à un traitement par lots. Le deuxième scénario améliore les performances en exécutant des opérations maître/détail dans un appel de procédure stockée unique. Le dernier scénario montre comment utiliser des paramètres table dans une opération « UPSERT ».

### <a name="buffering"></a>des réponses

Bien que certains scénarios apparaissent comme des candidats évidents pour le traitement par lots, il existe de nombreux scénarios qui peuvent tirer parti des avantages d’un traitement par lots différé. Ce type de traitement induit toutefois un risque plus élevé de perte des données en cas de défaillance inattendue. Il est important de comprendre ce risque et de prendre en compte ses conséquences.

Par exemple, considérez une application web qui assure le suivi de l’historique de navigation de chaque utilisateur. À chaque demande de page, l’application peut lancer un appel sur la base de données pour enregistrer la page consultée par l’utilisateur. Mais il est possible d’améliorer les performances et l’évolutivité en plaçant dans la mémoire tampon les activités de navigation des utilisateurs puis en envoyant ces données par lots à la base de données. Vous pouvez déclencher la mise à jour de la base de données par temps écoulé et/ou taille de mémoire tampon. Par exemple, une règle peut spécifier que le lot doit être traité après 20 secondes ou lorsque la mémoire tampon atteint 1 000 éléments.

L’exemple de code suivant utilise les [Extensions réactives - Rx](/previous-versions/dotnet/reactive-extensions/hh242985(v=vs.103)) pour traiter les événements mis en mémoire tampon déclenchés par une classe de surveillance. Lorsque la mémoire tampon est saturée ou qu’un délai d’attente est atteint, le lot de données utilisateur est envoyé à la base de données avec un paramètre table.

La classe NavHistoryData suivante modélise les détails de la navigation utilisateur. Elle contient des informations de base, telles que l’identifiant utilisateur, l’URL consultée et l’heure d’accès.

```csharp
public class NavHistoryData
{
    public NavHistoryData(int userId, string url, DateTime accessTime)
    { UserId = userId; URL = url; AccessTime = accessTime; }
    public int UserId { get; set; }
    public string URL { get; set; }
    public DateTime AccessTime { get; set; }
}
```

La classe NavHistoryDataMonitor est responsable de la mise en mémoire tampon des données de navigation utilisateur sur la base de données. Elle contient une méthode appelée RecordUserNavigationEntry qui répond en déclenchant un événement **OnAdded** . Le code suivant illustre la logique de constructeur qui utilise Rx pour créer une collection observable selon l’événement. Il souscrit ensuite à cette collection observable avec la méthode de la mémoire tampon. La surcharge spécifie que la mémoire tampon doit être envoyée toutes les 20 secondes ou toutes les 1 000 entrées.

```csharp
public NavHistoryDataMonitor()
{
    var observableData =
        Observable.FromEventPattern<NavHistoryDataEventArgs>(this, "OnAdded");

    observableData.Buffer(TimeSpan.FromSeconds(20), 1000).Subscribe(Handler);
}
```

Le gestionnaire convertit tous les éléments mis en mémoire tampon en type de paramètre table et transfère ensuite ce type à une procédure stockée chargée de traiter le lot. Le code suivant illustre la définition complète des classes NavHistoryDataEventArgs et NavHistoryDataMonitor.

```csharp
public class NavHistoryDataEventArgs : System.EventArgs
{
    public NavHistoryDataEventArgs(NavHistoryData data) { Data = data; }
    public NavHistoryData Data { get; set; }
}

public class NavHistoryDataMonitor
{
    public event EventHandler<NavHistoryDataEventArgs> OnAdded;

    public NavHistoryDataMonitor()
    {
        var observableData =
            Observable.FromEventPattern<NavHistoryDataEventArgs>(this, "OnAdded");

        observableData.Buffer(TimeSpan.FromSeconds(20), 1000).Subscribe(Handler);
    }
```

Le gestionnaire convertit tous les éléments mis en mémoire tampon en type de paramètre table et transfère ensuite ce type à une procédure stockée chargée de traiter le lot. Le code suivant illustre la définition complète des classes NavHistoryDataEventArgs et NavHistoryDataMonitor.

```csharp
    public class NavHistoryDataEventArgs : System.EventArgs
    {
        if (OnAdded != null)
            OnAdded(this, new NavHistoryDataEventArgs(data));
    }

    protected void Handler(IList<EventPattern<NavHistoryDataEventArgs>> items)
    {
        DataTable navHistoryBatch = new DataTable("NavigationHistoryBatch");
        navHistoryBatch.Columns.Add("UserId", typeof(int));
        navHistoryBatch.Columns.Add("URL", typeof(string));
        navHistoryBatch.Columns.Add("AccessTime", typeof(DateTime));
        foreach (EventPattern<NavHistoryDataEventArgs> item in items)
        {
            NavHistoryData data = item.EventArgs.Data;
            navHistoryBatch.Rows.Add(data.UserId, data.URL, data.AccessTime);
        }

        using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
        {
            connection.Open();

            SqlCommand cmd = new SqlCommand("sp_RecordUserNavigation", connection);
            cmd.CommandType = CommandType.StoredProcedure;

            cmd.Parameters.Add(
                new SqlParameter()
                {
                    ParameterName = "@NavHistoryBatch",
                    SqlDbType = SqlDbType.Structured,
                    TypeName = "NavigationHistoryTableType",
                    Value = navHistoryBatch,
                });

            cmd.ExecuteNonQuery();
        }
    }
}
```

Pour utiliser cette classe de mise en mémoire tampon, l’application crée un objet NavHistoryDataMonitor statique. À chaque fois qu’un utilisateur accède à une page, l’application appelle la méthode NavHistoryDataMonitor.RecordUserNavigationEntry. La logique de mise en mémoire tampon se poursuit pour envoyer ces entrées à la base de données sous forme de lots.

### <a name="master-detail"></a>Maître/détail

Les paramètres table sont utiles pour les scénarios INSERT simples. Il peut toutefois être plus difficile de traiter par lots des insertions qui impliquent plusieurs tables. Le scénario « maître/détail » en est un bon exemple. La table maître identifie l’entité principale. Une ou plusieurs tables détail stockent des données sur l’entité. Dans ce scénario, les relations de clés étrangères imposent la relation de détails à une seule entité maître. Imaginez une version simplifiée d’une table PurchaseOrder et de sa table OrderDetail associée. L’instruction Transact-SQL suivante crée la table PurchaseOrder avec quatre colonnes : OrderID, OrderDate, CustomerID et Status.

```sql
CREATE TABLE [dbo].[PurchaseOrder](
[OrderID] [int] IDENTITY(1,1) NOT NULL,
[OrderDate] [datetime] NOT NULL,
[CustomerID] [int] NOT NULL,
[Status] [nvarchar](50) NOT NULL,
CONSTRAINT [PrimaryKey_PurchaseOrder]
PRIMARY KEY CLUSTERED ( [OrderID] ASC ))
```

Chaque commande contient un ou plusieurs achats de produits. Ces informations sont recueillies dans la table PurchaseOrderDetail. L’instruction Transact-SQL suivante crée la table PurchaseOrderDetail avec cinq colonnes : OrderID, OrderDetailID, ProductID, UnitPrice et OrderQty.

```sql
CREATE TABLE [dbo].[PurchaseOrderDetail](
[OrderID] [int] NOT NULL,
[OrderDetailID] [int] IDENTITY(1,1) NOT NULL,
[ProductID] [int] NOT NULL,
[UnitPrice] [money] NULL,
[OrderQty] [smallint] NULL,
CONSTRAINT [PrimaryKey_PurchaseOrderDetail] PRIMARY KEY CLUSTERED
( [OrderID] ASC, [OrderDetailID] ASC ))
```

La colonne OrderID de la table PurchaseOrderDetail doit faire référence à une commande de la table PurchaseOrder. La définition suivante d’une clé étrangère applique cette contrainte.

```sql
ALTER TABLE [dbo].[PurchaseOrderDetail]  WITH CHECK ADD
CONSTRAINT [FK_OrderID_PurchaseOrder] FOREIGN KEY([OrderID])
REFERENCES [dbo].[PurchaseOrder] ([OrderID])
```

Pour utiliser des paramètres table, vous devez disposer d’un type de table défini par l’utilisateur pour chaque table cible.

```sql
CREATE TYPE PurchaseOrderTableType AS TABLE
( OrderID INT,
    OrderDate DATETIME,
    CustomerID INT,
    Status NVARCHAR(50) );
GO

CREATE TYPE PurchaseOrderDetailTableType AS TABLE
( OrderID INT,
    ProductID INT,
    UnitPrice MONEY,
    OrderQty SMALLINT );
GO
```

Vous devez ensuite définir ensuite une procédure stockée qui accepte les tables de ces types. Cette procédure permet à une application d’effectuer localement un traitement par lots sur un ensemble de commandes et de détails de commandes en un seul appel. L’instruction Transact-SQL suivante fournit la déclaration de procédure stockée complète pour cet exemple de bon de commande.

```sql
CREATE PROCEDURE sp_InsertOrdersBatch (
@orders as PurchaseOrderTableType READONLY,
@details as PurchaseOrderDetailTableType READONLY )
AS
SET NOCOUNT ON;

-- Table that connects the order identifiers in the @orders
-- table with the actual order identifiers in the PurchaseOrder table
DECLARE @IdentityLink AS TABLE (
SubmittedKey int,
ActualKey int,
RowNumber int identity(1,1)
);

-- Add new orders to the PurchaseOrder table, storing the actual
-- order identifiers in the @IdentityLink table
INSERT INTO PurchaseOrder ([OrderDate], [CustomerID], [Status])
OUTPUT inserted.OrderID INTO @IdentityLink (ActualKey)
SELECT [OrderDate], [CustomerID], [Status] FROM @orders ORDER BY OrderID;

-- Match the passed-in order identifiers with the actual identifiers
-- and complete the @IdentityLink table for use with inserting the details
WITH OrderedRows As (
SELECT OrderID, ROW_NUMBER () OVER (ORDER BY OrderID) As RowNumber
FROM @orders
)
UPDATE @IdentityLink SET SubmittedKey = M.OrderID
FROM @IdentityLink L JOIN OrderedRows M ON L.RowNumber = M.RowNumber;

-- Insert the order details into the PurchaseOrderDetail table,
-- using the actual order identifiers of the master table, PurchaseOrder
INSERT INTO PurchaseOrderDetail (
[OrderID],
[ProductID],
[UnitPrice],
[OrderQty] )
SELECT L.ActualKey, D.ProductID, D.UnitPrice, D.OrderQty
FROM @details D
JOIN @IdentityLink L ON L.SubmittedKey = D.OrderID;
GO
```

Dans cet exemple, la table @IdentityLink définie localement stocke les valeurs OrderID réelles provenant des lignes nouvellement insérées. Ces identificateurs de commande sont différents des valeurs OrderID temporaires contenues dans les paramètres table @orders et @details. Pour cette raison, la table @IdentityLink relie ensuite les valeurs OrderID du paramètre @orders aux valeurs OrderID réelles pour les nouvelles lignes de la table PurchaseOrder. Après cette étape, la table @IdentityLink peut faciliter l’insertion des détails de la commande avec la valeur OrderID réelle qui satisfait à la contrainte de clé étrangère.

Cette procédure stockée peut être utilisée à partir d’un code ou d’autres appels Transact-SQL. Consultez la section relative aux paramètres table de ce document pour obtenir un exemple de code. L’instruction Transact-SQL suivante explique comment appeler sp_InsertOrdersBatch.

```sql
declare @orders as PurchaseOrderTableType
declare @details as PurchaseOrderDetailTableType

INSERT @orders
([OrderID], [OrderDate], [CustomerID], [Status])
VALUES(1, '1/1/2013', 1125, 'Complete'),
(2, '1/13/2013', 348, 'Processing'),
(3, '1/12/2013', 2504, 'Shipped')

INSERT @details
([OrderID], [ProductID], [UnitPrice], [OrderQty])
VALUES(1, 10, $11.50, 1),
(1, 12, $1.58, 1),
(2, 23, $2.57, 2),
(3, 4, $10.00, 1)

exec sp_InsertOrdersBatch @orders, @details
```

Cette solution permet à chaque lot d’utiliser un ensemble de valeurs OrderID qui commencent à 1. Ces valeurs OrderID temporaires décrivent les relations dans le lot, mais les valeurs OrderID réelles sont déterminées au moment de l’opération d’insertion. Vous pouvez exécuter à plusieurs reprises les mêmes instructions de l’exemple précédent et générer des commandes uniques dans la base de données. Vous devez donc penser à ajouter plus de code ou de logique de base de données pour éviter les doublons lorsque vous utilisez cette technique de traitement par lots.

Cet exemple montre que les opérations de base de données encore plus complexes, telles que les opérations maître/détail, peuvent être traitées par lots à l’aide des paramètres table.

### <a name="upsert"></a>Opération UPSERT

Un autre scénario de traitement par lots implique la mise à jour simultanée de lignes existantes et l’insertion de nouvelles lignes. Cette opération est parfois appelée « UPSERT » (mise à jour + insertion). Plutôt que d’effectuer des appels distincts pour les opérations INSERT et UPDATE, l’instruction MERGE convient le mieux à cette tâche. L’instruction MERGE peut exécuter les deux opérations d’insertion et de mise à jour en un seul appel.

Les paramètres table peuvent être utilisés avec l’instruction MERGE pour effectuer des mises à jour et des insertions. Par exemple, considérez une table Employee simplifiée contenant les colonnes suivantes : EmployeeID, FirstName, LastName et SocialSecurityNumber :

```sql
CREATE TABLE [dbo].[Employee](
[EmployeeID] [int] IDENTITY(1,1) NOT NULL,
[FirstName] [nvarchar](50) NOT NULL,
[LastName] [nvarchar](50) NOT NULL,
[SocialSecurityNumber] [nvarchar](50) NOT NULL,
CONSTRAINT [PrimaryKey_Employee] PRIMARY KEY CLUSTERED
([EmployeeID] ASC ))
```

Dans cet exemple, vous pouvez utiliser le fait que les valeurs SocialSecurityNumber sont uniques pour fusionner plusieurs employés. Commencez par créer le type de table défini par l’utilisateur :

```sql
CREATE TYPE EmployeeTableType AS TABLE
( Employee_ID INT,
    FirstName NVARCHAR(50),
    LastName NVARCHAR(50),
    SocialSecurityNumber NVARCHAR(50) );
GO
```

Ensuite, créez une procédure stockée ou écrivez du code qui utilise l’instruction MERGE pour effectuer la mise à jour et l’insertion. L’exemple suivant utilise l’instruction MERGE sur un paramètre table, @employees, de type EmployeeTableType. Le contenu de la table @employees n’est pas indiqué ici.

```sql
MERGE Employee AS target
USING (SELECT [FirstName], [LastName], [SocialSecurityNumber] FROM @employees)
AS source ([FirstName], [LastName], [SocialSecurityNumber])
ON (target.[SocialSecurityNumber] = source.[SocialSecurityNumber])
WHEN MATCHED THEN
UPDATE SET
target.FirstName = source.FirstName,
target.LastName = source.LastName
WHEN NOT MATCHED THEN
    INSERT ([FirstName], [LastName], [SocialSecurityNumber])
    VALUES (source.[FirstName], source.[LastName], source.[SocialSecurityNumber]);
```

Pour plus d’informations, consultez la documentation et les exemples fournis pour l’instruction MERGE. Bien que la même tâche puisse être effectuée dans un appel de procédure stockée en plusieurs étapes avec des opérations INSERT et UPDATE distinctes, l’instruction MERGE est plus efficace. Le code de base de données peut également construire des appels Transact-SQL qui utilisent directement l’instruction MERGE sans nécessiter deux appels de base de données pour les opérations INSERT et UPDATE.

## <a name="recommendation-summary"></a>Résumé des recommandations

La liste suivante fournit un résumé des recommandations relatives au traitement par lots présentées dans cet article :

* Utilisez la mise en mémoire tampon et le traitement par lot pour améliorer les performances et la scalabilité des applications Azure SQL Database et Azure SQL Managed Instance.
* Tenez compte des compromis entre mise en mémoire tampon/traitement par lots et résilience. Lors de la défaillance d’un rôle, le risque de perte d’un lot non traité de données critiques peut contrebalancer le gain de performances du traitement par lots.
* Essayez de conserver tous les appels à la base de données dans un seul centre de données afin de réduire la latence.
* Si vous choisissez une technique de traitement par lots unique, les paramètres table offrent les meilleurs avantages en termes de performances et de flexibilité.
* Pour de meilleures performances d’insertion, suivez ces recommandations générales tout en prenant soin de tester votre scénario :
  * Pour moins de 100 lignes, utilisez une seule commande INSERT paramétrable.
  * Pour moins de 1 000 lignes, utilisez des paramètres table.
  * Au-delà de 1 000 lignes, utilisez SqlBulkCopy.
* Pour les opérations de mise à jour et de suppression, utilisez des paramètres table avec une logique de procédure stockée qui affecte l’opération correcte à chaque ligne du paramètre table.
* Conseils relatifs à la taille de lot :
  * Utilisez les plus grandes tailles de lot possibles en fonction de votre application et de vos besoins métiers.
  * Contrebalancez le gain de performances des lots volumineux avec les risques de défaillances temporaires ou catastrophiques. Quelle est la conséquence de nouvelles tentatives ou d’une perte de données dans le lot ?
  * Testez la plus grande taille de lot pour vérifier qu'elle est acceptée par Azure SQL Database ou Azure SQL Managed Instance.
  * Créez des paramètres de configuration qui contrôle le traitement par lots, telles que la taille du lot ou la fenêtre de temps de la mise en mémoire tampon. Ces paramètres garantissent la flexibilité. Vous pouvez modifier le comportement du traitement par lots en production sans avoir à redéployer le service cloud.
* Évitez l’exécution parallèle de lots qui s’exécutent sur une seule table dans une base de données unique. Si vous choisissez de diviser un seul lot sur plusieurs threads de travail, exécutez des tests pour déterminer le nombre idéal de threads. Après un seuil non spécifié, un plus grand nombre de threads aura pour effet de diminuer les performances au lieu de les augmenter.
* Envisagez la mise en mémoire tampon sur la taille et l’heure comme un moyen d’implémenter le traitement par lot pour d’autres scénarios.

## <a name="next-steps"></a>Étapes suivantes

Cet article se concentre sur la façon dont les techniques de conception et de codage de bases de données basées sur un traitement par lots peuvent améliorer les performances et l’évolutivité de votre application. Mais cet aspect ne représente qu’un facteur parmi d’autres dans votre stratégie globale. Pour accéder à d'autres méthodes d'amélioration des performances et de la scalabilité, consultez [Conseils sur les performances des bases de données](database/performance-guidance.md) et [Considérations sur les prix et performances pour un pool élastique](database/elastic-pool-overview.md).
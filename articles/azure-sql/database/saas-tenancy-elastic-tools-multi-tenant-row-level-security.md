---
title: Applications mutualisées avec une SNL et des outils de base de données élastique
description: Utilisez les outils de base de données élastique avec une sécurité au niveau des lignes pour générer une application avec une couche Données hautement évolutive.
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: VanMSFT
ms.author: vanto
ms.reviewer: sstein
ms.date: 12/18/2018
ms.openlocfilehash: 6d753a90f2a4cb19c9f3933d007fb3d378af6d81
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92793209"
---
# <a name="multi-tenant-applications-with-elastic-database-tools-and-row-level-security"></a>Applications multi-locataires avec des outils de base de données élastique et la sécurité au niveau des lignes
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Les [outils de base de données élastique](elastic-scale-get-started.md) et la fonction de [sécurité au niveau des lignes (SNL)][rls] coopèrent pour permettre la mise à l’échelle de la couche Données d’une application multilocataire au moyen d’Azure SQL Database. Ensemble, ces technologies vous aident à générer une application possédant une couche Données hautement évolutive. La couche Données prend en charge les partitions mutualisées et utilise **ADO.NET SqlClient** ou **Entity Framework** . Pour plus d’informations, consultez [Modèles de conception pour les applications SaaS multilocataires avec Azure SQL Database](./saas-tenancy-app-design-patterns.md).

- Les **outils de base de données élastique** permettent aux développeurs d’effectuer un scale-out de la couche Données avec des pratiques de partitionnement standard à l’aide de bibliothèques .NET et de modèles de service Microsoft Azure. En gérant les partitions à l’aide de la [bibliothèque cliente de bases de données élastiques][s-d-elastic-database-client-library], vous rationalisez et automatisez de nombreuses tâches de l’infrastructure portant généralement sur le partitionnement.
- La **sécurité au niveau des lignes** permet aux développeurs de stocker en toute sécurité des données pour plusieurs locataires dans la même base de données. Les stratégies de sécurité SNL filtrent les lignes qui n’appartiennent pas au locataire exécutant une requête. La centralisation de la logique de filtre à l’intérieur de la base de données simplifie la maintenance et réduit le risque d’une erreur de sécurité. L’alternative consistant à s’appuyer sur la totalité du code client pour appliquer la sécurité est risquée.

Grâce à l’utilisation conjointe de ces fonctionnalités, une application peut stocker les données de plusieurs locataires au sein de la base de données d’une seule et même partition. Le coût par locataire est inférieur lorsque les locataires partagent une base de données. Cependant, la même application peut également offrir à ses locataires premium une option de paiement pour leur propre partition locataire unique dédiée. L’un des avantages de l’isolation du locataire unique est la meilleure garantie de performances. Dans une base de données à locataire unique, il n’existe aucun autre locataire en concurrence pour les ressources.

L’objectif est d’utiliser les API de [routage dépendant des données](elastic-scale-data-dependent-routing.md) de la bibliothèque cliente de base de données élastique pour connecter automatiquement chaque locataire donné à la base de données de partition appropriée. Une seule partition contient la valeur TenantId particulière pour le locataire donné. Le TenantId est la *clé de partitionnement* . Une fois la connexion établie, une stratégie de sécurité SNL au sein de la base de données garantit que le locataire donné peut accéder uniquement aux lignes de données contenant son TenantId.

> [!NOTE]
> L’identificateur de locataire peut être constitué de plusieurs colonnes. Pour des raisons pratiques dans cette discussion, nous utilisons de manière informelle un TenantId à une seule colonne.

![Architecture d’application de création de blogs][1]

## <a name="download-the-sample-project"></a>Téléchargement de l’exemple de projet

### <a name="prerequisites"></a>Prérequis

- Exécuter Visual Studio version 2012 ou plus
- Créer trois bases de données dans Azure SQL Database
- Téléchargez l’exemple de projet : [Outils de base de données élastique pour SQL Azure - Partitions multilocataires](https://go.microsoft.com/?linkid=9888163)
  - Saisissez les informations sur vos bases de données au début du fichier **Program.cs**

Ce projet étend celui que décrit la section [Outils de base de données pour base de données SQL Microsoft Azure - Intégration d’Entity Framework](elastic-scale-use-entity-framework-applications-visual-studio.md) , en ajoutant la prise en charge des bases de données de partition multi-locataires. Le projet crée une application de console simple pour la création de blogs et de publications. Le projet inclut quatre locataires, ainsi que deux bases de données de partition mutualisée. Cette configuration est illustrée dans le précédent diagramme.

Générez et exécutez l’application. Cette exécution démarre le gestionnaire de mappage de la partition dédiée aux outils de base de données élastique et effectue les tests suivants :

1. À l’aide d’Entity Framework et de LINQ, créez un blog et affichez tous les blogs pour chaque client.
2. À l’aide de la fonction SqlClient ADO.NET, affichez tous les blogs d’un locataire.
3. Essayez d’insérer un blog associé à un locataire incorrect, afin de vérifier qu’une erreur est déclenchée.

Comme la fonction RLS n’a pas encore été activée sur les bases de données de la partition, vous pouvez voir que chacun de ces tests met en lumière un problème : les locataires peuvent afficher des blogs qui ne leur appartiennent pas et l’application est autorisée à insérer un blog associé à un locataire incorrect. Le reste de cet article explique comment résoudre ces problèmes en appliquant l’isolation des locataires avec la fonction RLS. En deux étapes :

1. **Couche Application**  : Modifiez le code de l’application pour toujours définir le TenantId actuel dans SESSION\_CONTEXT après l’ouverture d’une connexion. L’exemple de projet définit déjà le TenantId de cette manière.
2. **Couche données**  : Créez une stratégie de sécurité SNL dans chaque base de données de partition pour filtrer les lignes selon le TenantId stocké dans SESSION\_CONTEXT. Créez une stratégie pour chaque base de données de partition. Dans le cas contraire, les lignes de partitions mutualisées ne seront pas filtrées.

## <a name="1-application-tier-set-tenantid-in-the-session_context"></a>1. Couche Application : Définir le TenantId dans SESSION\_CONTEXT

Tout d’abord, connectez-vous à une base de données de partition à l’aide des API de routage dépendant des données de la bibliothèque cliente de base de données élastique. L’application doit toujours indiquer à la base de données quel TenantId utilise la connexion. Le TenantId indique à la stratégie de sécurité SNL les lignes qui doivent être éliminées par filtrage car appartenant à d’autres locataires. Stockez le TenantId actuel dans le [SESSION\_CONTEXT](/sql/t-sql/functions/session-context-transact-sql) de la connexion.

Une alternative à SESSION\_CONTEXT consiste à utiliser [CONTEXT\_INFO](/sql/t-sql/functions/context-info-transact-sql). Toutefois, SESSION\_CONTEXT constitue une meilleure option. SESSION\_CONTEXT est plus facile à utiliser, il renvoie la valeur NULL par défaut et prend en charge les paires clé-valeur.

### <a name="entity-framework"></a>Entity Framework

Pour les applications utilisant Entity Framework, l’approche la plus simple consiste à définir l’élément SESSION\_CONTEXT dans la substitution ElasticScaleContext décrite dans la section [Routage dépendant des données à l’aide de DbContext EF](elastic-scale-use-entity-framework-applications-visual-studio.md#data-dependent-routing-using-ef-dbcontext). Créez et exécutez une commande SqlCommand qui définit TenantId dans SESSION\_CONTEXT sur la valeur shardingKey spécifiée pour la connexion. Retournez ensuite la connexion répartie via le routage dépendant des données. De cette façon, vous n’avez à écrire le code qu’une seule fois pour définir l’élément SESSION\_CONTEXT.

```csharp
// ElasticScaleContext.cs
// Constructor for data-dependent routing.
// This call opens a validated connection that is routed to the
// proper shard by the shard map manager.
// Note that the base class constructor call fails for an open connection
// if migrations need to be done and SQL credentials are used.
// This is the reason for the separation of constructors.
// ...
public ElasticScaleContext(ShardMap shardMap, T shardingKey, string connectionStr)
    : base(
        OpenDDRConnection(shardMap, shardingKey, connectionStr),
        true)  // contextOwnsConnection
{
}

public static SqlConnection OpenDDRConnection(
    ShardMap shardMap,
    T shardingKey,
    string connectionStr)
{
    // No initialization.
    Database.SetInitializer<ElasticScaleContext<T>>(null);

    // Ask shard map to broker a validated connection for the given key.
    SqlConnection conn = null;
    try
    {
        conn = shardMap.OpenConnectionForKey(
            shardingKey,
            connectionStr,
            ConnectionOptions.Validate);

        // Set TenantId in SESSION_CONTEXT to shardingKey
        // to enable Row-Level Security filtering.
        SqlCommand cmd = conn.CreateCommand();
        cmd.CommandText =
            @"exec sp_set_session_context
                @key=N'TenantId', @value=@shardingKey";
        cmd.Parameters.AddWithValue("@shardingKey", shardingKey);
        cmd.ExecuteNonQuery();

        return conn;
    }
    catch (Exception)
    {
        if (conn != null)
        {
            conn.Dispose();
        }
        throw;
    }
}
// ...
```

Désormais, l’élément SESSION\_CONTEXT est automatiquement défini avec le TenantId spécifié chaque fois que le paramètre ElasticScaleContext est appelé :

```csharp
// Program.cs
SqlDatabaseUtils.SqlRetryPolicy.ExecuteAction(() =>
{
    using (var db = new ElasticScaleContext<int>(
        sharding.ShardMap, tenantId, connStrBldr.ConnectionString))
    {
        var query = from b in db.Blogs
                    orderby b.Name
                    select b;

        Console.WriteLine("All blogs for TenantId {0}:", tenantId);
        foreach (var item in query)
        {
            Console.WriteLine(item.Name);
        }
    }
});
```

### <a name="adonet-sqlclient"></a>SqlClient ADO.NET

Pour les applications utilisant ADO.NET SqlClient, créez une fonction wrapper autour de la méthode ShardMap.OpenConnectionForKey. Faites en sorte que le wrapper définisse automatiquement le TenantId dans SESSION\_CONTEXT sur le TenantId actuel avant de retourner une connexion. Pour garantir que SESSION\_CONTEXT est toujours défini, vous ne devez ouvrir des connexions qu’à l’aide de cette fonction wrapper.

```csharp
// Program.cs
// Wrapper function for ShardMap.OpenConnectionForKey() that
// automatically sets SESSION_CONTEXT with the correct
// tenantId before returning a connection.
// As a best practice, you should only open connections using this method
// to ensure that SESSION_CONTEXT is always set before executing a query.
// ...
public static SqlConnection OpenConnectionForTenant(
    ShardMap shardMap, int tenantId, string connectionStr)
{
    SqlConnection conn = null;
    try
    {
        // Ask shard map to broker a validated connection for the given key.
        conn = shardMap.OpenConnectionForKey(
            tenantId, connectionStr, ConnectionOptions.Validate);

        // Set TenantId in SESSION_CONTEXT to shardingKey
        // to enable Row-Level Security filtering.
        SqlCommand cmd = conn.CreateCommand();
        cmd.CommandText =
            @"exec sp_set_session_context
                @key=N'TenantId', @value=@shardingKey";
        cmd.Parameters.AddWithValue("@shardingKey", tenantId);
        cmd.ExecuteNonQuery();

        return conn;
    }
    catch (Exception)
    {
        if (conn != null)
        {
            conn.Dispose();
        }
        throw;
    }
}

// ...

// Example query via ADO.NET SqlClient.
// If row-level security is enabled, only Tenant 4's blogs are listed.
SqlDatabaseUtils.SqlRetryPolicy.ExecuteAction(() =>
{
    using (SqlConnection conn = OpenConnectionForTenant(
        sharding.ShardMap, tenantId4, connStrBldr.ConnectionString))
    {
        SqlCommand cmd = conn.CreateCommand();
        cmd.CommandText = @"SELECT * FROM Blogs";

        Console.WriteLine(@"--
All blogs for TenantId {0} (using ADO.NET SqlClient):", tenantId4);

        SqlDataReader reader = cmd.ExecuteReader();
        while (reader.Read())
        {
            Console.WriteLine("{0}", reader["Name"]);
        }
    }
});

```

## <a name="2-data-tier-create-row-level-security-policy"></a>2. Couche données : Créer une stratégie de sécurité au niveau des lignes

### <a name="create-a-security-policy-to-filter-the-rows-each-tenant-can-access"></a>Créez une stratégie de sécurité pour filtrer les lignes accessibles à chaque client.

Comme l’application définit désormais l’élément SESSION\_CONTEXT avec le TenantId actuel avant d’envoyer des requêtes, une stratégie de sécurité SNL peut filtrer les requêtes et exclure les lignes associées à un TenantId différent.

La sécurité SNL est implémentée dans Transact-SQL. Une fonction définie par l’utilisateur détermine la logique d’accès, et une stratégie de sécurité lie cette fonction à un nombre de tables quelconque. Pour ce projet :

1. La fonction vérifie que l’application est connectée à la base de données et que le TenantId stocké dans l’élément SESSION\_CONTEXT correspond au TenantId d’une ligne donnée.
    - C’est l’application qui est connectée, plutôt qu’un autre utilisateur SQL.

2. Un prédicat FILTER permet aux lignes qui répondent au filtre TenantId d’être transmises directement aux requêtes SELECT, UPDATE et DELETE.
    - Un prédicat BLOCK empêche les lignes qui ne répondent pas au filtre d’être insérées (INSERT) ou mises à jour (UPDATE).
    - Si l’élément SESSION\_CONTEXT n’a pas été défini, la fonction retourne la valeur NULL, et aucune ligne n’est visible ou ne peut être insérée.

Pour activer la sécurité SNL sur toutes les partitions, exécutez le code T-SQL suivant à l’aide de Visual Studio (SSDT), de SSMS ou du script PowerShell inclus dans le projet. Ou si vous utilisez des [tâches de base de données élastique](./elastic-jobs-overview.md), vous pouvez automatiser l’exécution de ce T-SQL sur toutes les partitions.

```sql
CREATE SCHEMA rls; -- Separate schema to organize RLS objects.
GO

CREATE FUNCTION rls.fn_tenantAccessPredicate(@TenantId int)
    RETURNS TABLE
    WITH SCHEMABINDING
AS
    RETURN SELECT 1 AS fn_accessResult
        -- Use the user in your application’s connection string.
        -- Here we use 'dbo' only for demo purposes!
        WHERE DATABASE_PRINCIPAL_ID() = DATABASE_PRINCIPAL_ID('dbo')
        AND CAST(SESSION_CONTEXT(N'TenantId') AS int) = @TenantId;
GO

CREATE SECURITY POLICY rls.tenantAccessPolicy
    ADD FILTER PREDICATE rls.fn_tenantAccessPredicate(TenantId) ON dbo.Blogs,
    ADD BLOCK  PREDICATE rls.fn_tenantAccessPredicate(TenantId) ON dbo.Blogs,
    ADD FILTER PREDICATE rls.fn_tenantAccessPredicate(TenantId) ON dbo.Posts,
    ADD BLOCK  PREDICATE rls.fn_tenantAccessPredicate(TenantId) ON dbo.Posts;
GO
```

> [!TIP]
> Dans un projet complexe, vous devrez peut-être ajouter le prédicat sur des centaines de tables, ce qui peut s’avérer fastidieux. Il existe une procédure stockée d’assistance qui génère automatiquement une stratégie de sécurité et ajoute un prédicat sur toutes les tables d’un schéma. Pour plus d’informations, consultez le billet de blog [Apply Row-Level Security to all tables - helper script (Appliquer la sécurité au niveau des lignes à toutes les tables - Script d’assistance)](https://techcommunity.microsoft.com/t5/sql-server/apply-row-level-security-to-all-tables-helper-script/ba-p/384360).

À présent, si vous exécutez l’exemple d’application une nouvelle fois, les locataires ne sont en mesure de voir que les lignes qui leur appartiennent. Par ailleurs, l’application ne peut pas insérer des lignes qui appartiennent à un locataire autre que celui qui est actuellement connecté à la base de données de partition. De plus, l’application ne peut mettre à jour le TenantId sur aucune ligne trouvée. Si elle tente d’effectuer l’une ou l’autre de ces opérations, une exception DbUpdateException est déclenchée.

Si vous ajoutez une nouvelle table par la suite, effectuez une opération ALTER sur la stratégie de sécurité pour ajouter des prédicats FILTER et BLOCK sur la nouvelle table.

```sql
ALTER SECURITY POLICY rls.tenantAccessPolicy
    ADD FILTER PREDICATE rls.fn_tenantAccessPredicate(TenantId) ON dbo.MyNewTable,
    ADD BLOCK  PREDICATE rls.fn_tenantAccessPredicate(TenantId) ON dbo.MyNewTable;
GO
```

### <a name="add-default-constraints-to-automatically-populate-tenantid-for-inserts"></a>Ajouter des contraintes par défaut afin d’indiquer automatiquement les ID de locataire pour les opérations INSERT

Vous pouvez placer une contrainte par défaut sur chaque table pour renseigner automatiquement le TenantId avec la valeur actuellement stockée dans l’élément SESSION\_CONTEXT lors de l’insertion de lignes. Un exemple suit.

```sql
-- Create default constraints to auto-populate TenantId with the
-- value of SESSION_CONTEXT for inserts.
ALTER TABLE Blogs
    ADD CONSTRAINT df_TenantId_Blogs
    DEFAULT CAST(SESSION_CONTEXT(N'TenantId') AS int) FOR TenantId;
GO

ALTER TABLE Posts
    ADD CONSTRAINT df_TenantId_Posts
    DEFAULT CAST(SESSION_CONTEXT(N'TenantId') AS int) FOR TenantId;
GO
```

À présent, l’application n’a pas besoin de spécifier un ID de locataire lors de l’insertion de lignes :

```csharp
SqlDatabaseUtils.SqlRetryPolicy.ExecuteAction(() =>
{
    using (var db = new ElasticScaleContext<int>(
        sharding.ShardMap, tenantId, connStrBldr.ConnectionString))
    {
        // The default constraint sets TenantId automatically!
        var blog = new Blog { Name = name };
        db.Blogs.Add(blog);
        db.SaveChanges();
    }
});
```

> [!NOTE]
> Si vous utilisez des contraintes par défaut pour un projet Entity Framework, il est recommandé de ne *PAS* inclure la colonne TenantId dans votre modèle de données Entity Framework. En effet, les requêtes Entity Framework fournissent automatiquement des valeurs par défaut qui remplacent les contraintes par défaut créées dans T-SQL et qui utilisent l’élément SESSION\_CONTEXT.
> Pour utiliser les contraintes par défaut dans l’exemple de projet, par exemple, vous devez supprimer l’ID de locataire dans le fichier DataClasses.cs (et exécuter l’élément Add-Migration dans la Console du gestionnaire de package), puis utiliser T-SQL pour vérifier que le champ existe uniquement dans les tables de base de données. De cette façon, Entity Framework fournit automatiquement des valeurs par défaut incorrectes lors de l’insertion de données.

### <a name="optional-enable-a-superuser-to-access-all-rows"></a>(Facultatif) Activer un *superutilisateur* pour accéder à toutes les lignes

Certaines applications souhaiteront peut-être créer un *superutilisateur* qui peut accéder à toutes les lignes. Un super utilisateur peut permettre la création de rapports pour tous les locataires sur toutes les partitions. Ou un super utilisateur peut effectuer des opérations de fusion et fractionnement sur les partitions qui impliquent le déplacement de lignes de locataires entre les bases de données.

Pour activer un superutilisateur, créez un nouvel utilisateur SQL (`superuser` dans cet exemple) dans chaque base de données de partition. Vous devez ensuite modifier la stratégie de sécurité en ajoutant une nouvelle fonction de prédicat permettant à cet utilisateur d’accéder à toutes les lignes. Une telle fonction est fournie plus loin.

```sql
-- New predicate function that adds superuser logic.
CREATE FUNCTION rls.fn_tenantAccessPredicateWithSuperUser(@TenantId int)
    RETURNS TABLE
    WITH SCHEMABINDING
AS
    RETURN SELECT 1 AS fn_accessResult
        WHERE
        (
            DATABASE_PRINCIPAL_ID() = DATABASE_PRINCIPAL_ID('dbo') -- Replace 'dbo'.
            AND CAST(SESSION_CONTEXT(N'TenantId') AS int) = @TenantId
        )
        OR
        (
            DATABASE_PRINCIPAL_ID() = DATABASE_PRINCIPAL_ID('superuser')
        );
GO

-- Atomically swap in the new predicate function on each table.
ALTER SECURITY POLICY rls.tenantAccessPolicy
    ALTER FILTER PREDICATE rls.fn_tenantAccessPredicateWithSuperUser(TenantId) ON dbo.Blogs,
    ALTER BLOCK  PREDICATE rls.fn_tenantAccessPredicateWithSuperUser(TenantId) ON dbo.Blogs,
    ALTER FILTER PREDICATE rls.fn_tenantAccessPredicateWithSuperUser(TenantId) ON dbo.Posts,
    ALTER BLOCK  PREDICATE rls.fn_tenantAccessPredicateWithSuperUser(TenantId) ON dbo.Posts;
GO
```


### <a name="maintenance"></a>Maintenance

- **Ajout de nouvelles partitions**  : Exécutez le script T-SQL pour activer SNL sur les nouvelles partitions, sinon, les requêtes portant sur ces partitions ne sont pas filtrées.
- **Ajout de nouvelles tables**  : Ajoutez un prédicat FILTER et BLOCK à la stratégie de sécurité sur toutes les partitions chaque fois qu’une table est créée. Dans le cas contraire, les requêtes portant sur la nouvelle table ne sont pas filtrées. Vous pouvez automatiser cet ajout par le biais d’un déclencheur DDL, comme décrit dans l’article [Apply Row-Level Security automatically to newly created tables (Appliquer automatiquement la sécurité au niveau des lignes aux nouvelles tables) (blog)](https://techcommunity.microsoft.com/t5/SQL-Server/Apply-Row-Level-Security-automatically-to-newly-created-tables/ba-p/384393).

## <a name="summary"></a>Résumé

Les outils de base de données élastique et la fonction de sécurité au niveau des lignes (RLS) peuvent être utilisés ensemble pour effectuer un scale-out de la couche Données d’une application prenant en charge les partitions multi-locataires ou à un seul locataire. Les partitions mutualisées peuvent être utilisées pour stocker les données plus efficacement. Cette efficacité est prononcée lorsqu’un grand nombre de locataires possède seulement quelques lignes de données. Les partitions de locataire unique peuvent prendre en charge les locataires premium aux exigences de performances et d’isolation plus strictes. Pour plus d’informations, consultez [Sécurité au niveau des lignes][rls].

## <a name="additional-resources"></a>Ressources supplémentaires

- [Qu’est-ce qu’un pool élastique Azure ?](elastic-pool-overview.md)
- [Scale-out avec Azure SQL Database](elastic-scale-introduction.md)
- [Modèles de conception pour les applications SaaS multilocataires avec Azure SQL Database](./saas-tenancy-app-design-patterns.md)
- [Authentification dans des applications multi-locataires avec Azure AD et OpenID Connect](/azure/architecture/multitenant-identity/authenticate)
- [Application Tailspin Surveys](/azure/architecture/multitenant-identity/tailspin)

## <a name="questions-and-feature-requests"></a>Questions et demandes de fonctionnalités

Pour toute question, contactez-nous sur la [page de questions Microsoft Q&R pour SQL Database](/answers/topics/azure-sql-database.html). Ajoutez également des demandes de fonctionnalités dans le [forum de commentaires de Base de données SQL](https://feedback.azure.com/forums/217321-sql-database/).

<!--Image references-->
[1]: ./media/saas-tenancy-elastic-tools-multi-tenant-row-level-security/blogging-app.png
<!--anchors-->
[rls]: /sql/relational-databases/security/row-level-security
[s-d-elastic-database-client-library]:elastic-database-client-library.md
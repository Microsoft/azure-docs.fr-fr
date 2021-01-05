---
title: Interroger des données d’Azure Cosmos DB à l’aide d’un pool SQL serverless dans Azure Synapse Link (préversion)
description: Cet article explique comment interroger Azure Cosmos DB à l’aide d’un pool SQL serverless dans Azure Synapse Link (préversion).
services: synapse analytics
author: jovanpop-msft
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 12/04/2020
ms.author: jovanpop
ms.reviewer: jrasnick
ms.openlocfilehash: 22103ad580fa474f44eaf42c696d19bbbd137c8e
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97095098"
---
# <a name="query-azure-cosmos-db-data-with-a-serverless-sql-pool-in-azure-synapse-link-preview"></a>Interroger des données d’Azure Cosmos DB avec un pool SQL serverless dans Azure Synapse Link (préversion)

> [!IMPORTANT]
> La prise en charge du pool SQL serverless pour Azure Synapse Link pour Azure Cosmos DB est actuellement en préversion. Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


Un pool SQL serverless vous permet d’analyser les données figurant dans vos conteneurs Azure Cosmos DB activés avec [Azure Synapse Link](../../cosmos-db/synapse-link.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) en quasi-temps réel, sans que cela affecte les performances de vos charges de travail transactionnelles. Il offre une syntaxe T-SQL familière pour interroger les données du [magasin analytique](../../cosmos-db/analytical-store-introduction.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json), et une connectivité intégrée à un vaste éventail d’outils décisionnels et d’interrogation ad hoc via l’interface T-SQL.

Pour l’interrogation d’Azure Cosmos DB, toute la surface d’exposition [SELECT](/sql/t-sql/queries/select-transact-sql?view=sql-server-ver15) est prise en charge via la fonction [OPENROWSET](develop-openrowset.md), y compris la majorité des [fonctions et opérateurs SQL](overview-features.md). Vous pouvez également stocker les résultats de la requête qui lit des données d’Azure Cosmos DB ainsi que des données du Stockage Blob Azure ou d’Azure Data Lake Storage à l’aide de la commande [create external table as select](develop-tables-cetas.md#cetas-in-serverless-sql-pool) (CETAS). Actuellement, vous ne pouvez pas stocker les résultats d’une requête de pool SQL serverless dans Azure Cosmos DB à l’aide de CETAS.

Cet article explique comment écrire une requête à l’aide d’un pool SQL serverless, qui interrogera les données de conteneurs Azure Cosmos DB pour lesquels la fonctionnalité Synapse Link est activée. [Ce tutoriel](./tutorial-data-analyst.md) fournit également des informations supplémentaires sur la création de vues de pool SQL serverless sur des conteneurs Azure Cosmos DB, et leur connexion à des modèles Power BI.

> [!IMPORTANT]
> Ce tutoriel utilise un conteneur avec un [schéma bien défini Azure Cosmos DB](../../cosmos-db/analytical-store-introduction.md#schema-representation). L’expérience de requête qu’offre le pool SQL serverless pour un [schéma de fidélité optimale Azure Cosmos DB](#full-fidelity-schema) est un comportement temporaire qui variera en fonction des commentaires sur la préversion. Ne vous fiez pas au schéma du jeu de résultats de la fonction `OPENROWSET` sans la clause `WITH` qui lit les données d’un conteneur avec un schéma de fidélité optimale, car l’expérience de requête pourrait être alignée et modifiée en fonction du schéma bien défini. Vous pouvez poster vos commentaires dans le [forum de commentaires Azure Synapse Analytics](https://feedback.azure.com/forums/307516-azure-synapse-analytics). Vous pouvez également contacter l’[équipe produit Synapse Link](mailto:cosmosdbsynapselink@microsoft.com) pour lui faire part de vos commentaires.

## <a name="overview"></a>Vue d’ensemble

Le pool SQL serverless vous permet d’interroger le stockage analytique d’Azure Cosmos DB à l’aide de la fonction `OPENROWSET`. 
- `OPENROWSET` avec clé inlined. Cette syntaxe peut être utilisée pour interroger les collections d’Azure Cosmos DB sans avoir besoin de préparer les informations d’identification.
- `OPENROWSET` qui a référencé les informations d’identification contenant la clé de compte Cosmos DB. Cette syntaxe peut être utilisée pour créer des vues sur des collections d’Azure Cosmos DB.

### <a name="openrowset-with-key"></a>[OPENROWSET avec clé](#tab/openrowset-key)

Pour prendre en charge l’interrogation et l’analyse des données d’un magasin analytique Azure Cosmos DB, un pool SQL serverless utilise la syntaxe `OPENROWSET` suivante :

```sql
OPENROWSET( 
       'CosmosDB',
       '<Azure Cosmos DB connection string>',
       <Container name>
    )  [ < with clause > ] AS alias
```

La chaîne de connexion Azure Cosmos DB spécifie le nom du compte Azure Cosmos DB, le nom de la base de données, la clé principale du compte de base de données et un nom de région facultatif pour la fonction `OPENROWSET`.

Le format de la chaîne de connexion est le suivant :
```sql
'account=<database account name>;database=<database name>;region=<region name>;key=<database account master key>'
```

Le nom du conteneur Azure Cosmos DB est spécifié sans guillemets dans la syntaxe `OPENROWSET`. Si le nom du conteneur contient des caractères spéciaux, par exemple un tiret (-), le nom doit être encapsulé entre crochets (`[]`) dans la syntaxe `OPENROWSET`.

### <a name="openrowset-with-credential"></a>[OPENROWSET avec informations d’identification](#tab/openrowset-credential)

Vous pouvez utiliser la syntaxe `OPENROWSET` qui référence les informations d’identification :

```sql
OPENROWSET( 
       PROVIDER = 'CosmosDB',
       CONNECTION = '<Azure Cosmos DB connection string without account key>',
       OBJECT = '<Container name>',
       [ CREDENTIAL | SERVER_CREDENTIAL ] = '<credential name>'
    )  [ < with clause > ] AS alias
```

Dans ce cas, la chaîne de connexion Azure Cosmos DB ne contient pas de clé. Le format de la chaîne de connexion est le suivant :
```sql
'account=<database account name>;database=<database name>;region=<region name>'
```

La clé principale du compte de base de données est placée dans les informations d’identification au niveau du serveur ou dans celles limitées à la base de données. 

---

> [!IMPORTANT]
> Assurez-vous que vous utilisez un classement de base de données UTF-8, par exemple `Latin1_General_100_CI_AS_SC_UTF8`, car les valeurs de chaîne dans le magasin analytique Azure Cosmos DB sont encodées sous forme de texte UTF-8.
> Une incompatibilité entre l’encodage de texte dans le fichier et le classement peut entraîner des erreurs de conversion de texte inattendues.
> Vous pouvez facilement modifier le classement par défaut de la base de données actuelle à l’aide de l’instruction T-SQL `alter database current collate Latin1_General_100_CI_AI_SC_UTF8`.

> [!NOTE]
> Un pool SQL serverless ne prend pas en charge l’interrogation d’un magasin transactionnel d’Azure Cosmos DB.

## <a name="sample-dataset"></a>Exemple de jeu de données

Les exemples de cet article sont basés sur des données relatives aux [Cas de COVID-19 du Centre européen pour la prévention et le contrôle des maladies (CEPCM)](https://azure.microsoft.com/services/open-datasets/catalog/ecdc-covid-19-cases/) et du [COVID-19 Open Research DataSet (CORD-19), doi:10.5281/zenodo.3715505](https://azure.microsoft.com/services/open-datasets/catalog/covid-19-open-research/).

Vous trouverez la licence et la structure des données dans ces pages. Vous pouvez également télécharger des exemples de données pour les jeux de données [ECDC](https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases/latest/ecdc_cases.json) et [CORD-19](https://azureopendatastorage.blob.core.windows.net/covid19temp/comm_use_subset/pdf_json/000b7d1517ceebb34e1e3e817695b6de03e2fa78.json) .

Pour suivre cet article montrant comment interroger les données d’Azure Cosmos DB avec un pool SQL serverless, veillez à créer les ressources suivantes :

* un compte de base de données Azure Cosmos DB [compatible Azure Synapse Link](../../cosmos-db/configure-synapse-link.md) ;
* une base de données Azure Cosmos DB nommée `covid` ;
* deux conteneurs Azure Cosmos DB nommés `Ecdc` et `Cord19` chargés avec les exemples de jeux de données ci-dessus.

Vous pouvez utiliser la chaîne de connexion suivante à des fins de test : `Account=synapselink-cosmosdb-sqlsample;Database=covid;Key=s5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==`. Notez que cette connexion ne garantit pas les performances, car ce compte peut se trouver dans une région distante par rapport à votre point de terminaison Synapse SQL.

## <a name="explore-azure-cosmos-db-data-with-automatic-schema-inference"></a>Explorer des données d’Azure Cosmos DB avec une inférence de schéma automatique

Le moyen le plus simple d’explorer des données dans Azure Cosmos DB consiste à utiliser la fonctionnalité d’inférence de schéma automatique. En omettant la clause `WITH` de l’instruction `OPENROWSET`, vous pouvez donner pour instruction au pool SQL serverless de détecter automatiquement (inférer) le schéma du magasin analytique du conteneur Azure Cosmos DB.

### <a name="openrowset-with-key"></a>[OPENROWSET avec clé](#tab/openrowset-key)

```sql
SELECT TOP 10 *
FROM OPENROWSET( 
       'CosmosDB',
       'Account=synapselink-cosmosdb-sqlsample;Database=covid;Key=s5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==',
       Ecdc) as documents
```

### <a name="openrowset-with-credential"></a>[OPENROWSET avec informations d’identification](#tab/openrowset-credential)

```sql
/*  Setup - create server-level or database scoped credential with Azure Cosmos DB account key:
    CREATE CREDENTIAL MyCosmosDbAccountCredential
    WITH IDENTITY = 'SHARED ACCESS SIGNATURE', SECRET = 's5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==';
*/
SELECT TOP 10 *
FROM OPENROWSET(
      PROVIDER = 'CosmosDB',
      CONNECTION = 'Account=synapselink-cosmosdb-sqlsample;Database=covid',
      OBJECT = 'Ecdc',
      SERVER_CREDENTIAL = 'MyCosmosDbAccountCredential'
    ) with ( date_rep varchar(20), cases bigint, geo_id varchar(6) ) as rows
```

---

Dans l’exemple ci-dessus, nous avons donné pour instruction à un pool SQL serverless de se connecter à la base de données `covid` dans un compte Azure Cosmos DB `MyCosmosDbAccount` authentifié à l’aide de la clé Azure Cosmos DB (la clé factice dans l’exemple ci-dessus). Nous avons accédé ensuite au magasin analytique `Ecdc` du conteneur dans la région `West US 2`. Étant donné qu’il n’y a aucune projection de propriétés spécifiques, la fonction `OPENROWSET` retourne toutes les propriétés des éléments d’Azure Cosmos DB.

En supposant que les éléments se trouvant dans le conteneur Azure Cosmos DB aient les propriétés `date_rep`, `cases` et `geo_id`, les résultats de cette requête figurent dans le tableau suivant :

| date_rep | cas | geo_id |
| --- | --- | --- |
| 2020-08-13 | 254 | RS |
| 2020-08-12 | 235 | RS |
| 2020-08-11 | 163 | RS |

Si vous devez explorer les données de l’autre conteneur dans la même base de données Azure Cosmos DB, vous pouvez utiliser la même chaîne de connexion et le conteneur de référence requis comme troisième paramètre :

```sql
SELECT TOP 10 *
FROM OPENROWSET( 
       'CosmosDB',
       'Account=synapselink-cosmosdb-sqlsample;Database=covid;Key=s5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==',
       Cord19) as cord19
```

## <a name="explicitly-specify-schema"></a>Spécifier explicitement le schéma

Bien que la fonctionnalité d’inférence de schéma automatique dans `OPENROWSET` offre une expérience de requête simple et facile à utiliser, vos scénarios d’entreprise peuvent vous obliger à spécifier explicitement le schéma pour lire uniquement les propriétés pertinentes à partir des données d’Azure Cosmos DB.

La fonction `OPENROWSET` vous permet de spécifier de manière explicite les propriétés que vous souhaitez lire à partir des données du conteneur, ainsi que de spécifier leurs types de données.

Supposons que nous avons importé des données du [jeu de données COVID du CEPCM](https://azure.microsoft.com/services/open-datasets/catalog/ecdc-covid-19-cases/) avec la structure suivante dans Azure Cosmos DB :

```json
{"date_rep":"2020-08-13","cases":254,"countries_and_territories":"Serbia","geo_id":"RS"}
{"date_rep":"2020-08-12","cases":235,"countries_and_territories":"Serbia","geo_id":"RS"}
{"date_rep":"2020-08-11","cases":163,"countries_and_territories":"Serbia","geo_id":"RS"}
```

Ces documents JSON plats dans Azure Cosmos DB peuvent être représentés sous la forme d’un ensemble de lignes et de colonnes dans Synapse SQL. La fonction `OPENROWSET` vous permet de spécifier un sous-ensemble de propriétés que vous souhaitez lire et les types de colonnes exacts dans la clause `WITH` :

### <a name="openrowset-with-key"></a>[OPENROWSET avec clé](#tab/openrowset-key)
```sql
SELECT TOP 10 *
FROM OPENROWSET(
      'CosmosDB',
      'Account=synapselink-cosmosdb-sqlsample;Database=covid;Key=s5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==',
       Ecdc
    ) with ( date_rep varchar(20), cases bigint, geo_id varchar(6) ) as rows
```
### <a name="openrowset-with-credential"></a>[OPENROWSET avec informations d’identification](#tab/openrowset-credential)
```sql
/*  Setup - create server-level or database scoped credential with Azure Cosmos DB account key:
    CREATE CREDENTIAL MyCosmosDbAccountCredential
    WITH IDENTITY = 'SHARED ACCESS SIGNATURE', SECRET = 's5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==';
*/
SELECT TOP 10 *
FROM OPENROWSET(
      PROVIDER = 'CosmosDB',
      CONNECTION = 'Account=synapselink-cosmosdb-sqlsample;Database=covid',
      OBJECT = 'Ecdc',
      SERVER_CREDENTIAL = 'MyCosmosDbAccountCredential'
    ) with ( date_rep varchar(20), cases bigint, geo_id varchar(6) ) as rows
```
---
Le résultat de cette requête peut ressembler au tableau suivant :

| date_rep | cas | geo_id |
| --- | --- | --- |
| 2020-08-13 | 254 | RS |
| 2020-08-12 | 235 | RS |
| 2020-08-11 | 163 | RS |

Pour plus d’informations sur les types SQL à utiliser pour les valeurs Azure Cosmos DB, consultez les [règles pour les mappages de type SQL](#azure-cosmos-db-to-sql-type-mappings) à la fin de l’article.

## <a name="create-view"></a>Créer une vue

Une fois que vous avez identifié le schéma, vous pouvez préparer une vue en plus de vos données Azure Cosmos DB. Vous devez placer votre clé de compte Azure Cosmos DB dans des informations d’identification distinctes et référencer ces informations d’identification à partir de la fonction `OPENROWSET`. Ne conservez pas votre clé de compte dans la définition de la vue.

```sql
CREATE CREDENTIAL MyCosmosDbAccountCredential
WITH IDENTITY = 'SHARED ACCESS SIGNATURE', SECRET = 's5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==';
GO
CREATE OR ALTER VIEW Ecdc
AS SELECT *
FROM OPENROWSET(
      PROVIDER = 'CosmosDB',
      CONNECTION = 'Account=synapselink-cosmosdb-sqlsample;Database=covid',
      OBJECT = 'Ecdc',
      SERVER_CREDENTIAL = 'MyCosmosDbAccountCredential'
    ) with ( date_rep varchar(20), cases bigint, geo_id varchar(6) ) as rows
```

N’utilisez pas `OPENROWSET` sans schéma explicitement défini, car cela peut avoir un impact sur vos performances. Veillez à utiliser les plus petites tailles possibles pour vos colonnes [par exemple VARCHAR(100) au lieu de VARCHAR(8000) par défaut]. Vous devriez utiliser un classement UTF-8 comme classement de base de données par défaut ou le définir comme classement de colonnes explicite pour éviter les [problèmes de conversion UTF-8](/azure/synapse-analytics/troubleshoot/reading-utf8-text). Le classement `Latin1_General_100_BIN2_UTF8` offre des performances optimales quand vous filtrez des données en utilisant des colonnes de chaînes.

## <a name="query-nested-objects-and-arrays"></a>Interroger des objets et des tableaux imbriqués

Azure Cosmos DB vous permet de représenter des modèles de données plus complexes en les composant en tant qu’objets ou tableaux imbriqués. La fonctionnalité de synchronisation automatique d’Azure Synapse Link pour Azure Cosmos DB gère la représentation de schéma dans le magasin analytique prêt à l’emploi qui comprend la gestion des types de données imbriqués, ce qui permet d’effectuer des requêtes enrichies à partir du pool SQL serverless.

Par exemple, le jeu de données [CORD-19](https://azure.microsoft.com/services/open-datasets/catalog/covid-19-open-research/) contient des documents JSON suivant cette structure :

```json
{
    "paper_id": <str>,                   # 40-character sha1 of the PDF
    "metadata": {
        "title": <str>,
        "authors": <array of objects>    # list of author dicts, in order
        ...
     }
     ...
}
```

Les objets et tableaux imbriqués dans Azure Cosmos DB sont représentés en tant que chaînes JSON dans le résultat de la requête lorsque la fonction `OPENROWSET` les lit. Vous pouvez spécifier les chemins d’accès aux valeurs imbriquées dans les objets lorsque vous utilisez la clause `WITH` :

```sql
SELECT TOP 10 *
FROM OPENROWSET( 
       'CosmosDB',
       'Account=synapselink-cosmosdb-sqlsample;Database=covid;Key=s5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==',
       Cord19)
WITH (  paper_id    varchar(8000),
        title        varchar(1000) '$.metadata.title',
        metadata     varchar(max),
        authors      varchar(max) '$.metadata.authors'
) AS docs;
```

Le résultat de cette requête peut ressembler au tableau suivant :

| paper_id | title | metadata | authors |
| --- | --- | --- |
| bb11206963e831f… | Informations supplémentaires An eco-epidemi… | `{"title":"Supplementary Informati…` | `[{"first":"Julien","last":"Mélade","suffix":"","af…`| 
| bb1206963e831f1… | The Use of Convalescent Sera in Immune-E… | `{"title":"The Use of Convalescent…` | `[{"first":"Antonio","last":"Lavazza","suffix":"", …` |
| bb378eca9aac649… | Tylosema esculentum (Marama) Tuber and B… | `{"title":"Tylosema esculentum (Ma…` | `[{"first":"Walter","last":"Chingwaru","suffix":"",…` | 

Apprenez-en davantage sur l’analyse des [types de données complexes dans Azure Synapse Link](../how-to-analyze-complex-schema.md) et les [structures imbriquées dans un pool SQL serverless](query-parquet-nested-types.md).

> [!IMPORTANT]
> Si vous voyez des caractères inattendus dans votre texte, par exemple `MÃƒÂ©lade` au lieu de `Mélade`, cela signifie que votre classement de base de données n’est pas défini sur le classement [UTF-8](https://docs.microsoft.com/sql/relational-databases/collations/collation-and-unicode-support#utf8).
> [Remplacez le classement de la base de données](https://docs.microsoft.com/sql/relational-databases/collations/set-or-change-the-database-collation#to-change-the-database-collation) par un classement UTF-8 à l’aide d’une instruction SQL telle que `ALTER DATABASE MyLdw COLLATE LATIN1_GENERAL_100_CI_AS_SC_UTF8`.

## <a name="flatten-nested-arrays"></a>Aplatir des tableaux imbriqués

Il se peut que les données Azure Cosmos DB comprennent des sous-tableaux imbriqués tels que le tableau d’auteurs du jeu de données [CORD-19](https://azure.microsoft.com/services/open-datasets/catalog/covid-19-open-research/) :

```json
{
    "paper_id": <str>,                      # 40-character sha1 of the PDF
    "metadata": {
        "title": <str>,
        "authors": [                        # list of author dicts, in order
            {
                "first": <str>,
                "middle": <list of str>,
                "last": <str>,
                "suffix": <str>,
                "affiliation": <dict>,
                "email": <str>
            },
            ...
        ],
        ...
}
```

Dans certains cas, il se peut que vous deviez « joindre » les propriétés de l’élément supérieur (métadonnées) à tous les éléments du tableau (authors). Un pool SQL serverless vous permet d’aplatir des structures imbriquées en appliquant la fonction `OPENJSON` au tableau imbriqué :

```sql
SELECT
    *
FROM
    OPENROWSET(
      'CosmosDB',
      'Account=synapselink-cosmosdb-sqlsample;Database=covid;Key=s5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==',
       Cord19
    ) WITH ( title varchar(1000) '$.metadata.title',
             authors varchar(max) '$.metadata.authors' ) AS docs
      CROSS APPLY OPENJSON ( authors )
                  WITH (
                       first varchar(50),
                       last varchar(50),
                       affiliation nvarchar(max) as json
                  ) AS a
```

Le résultat de cette requête peut ressembler au tableau suivant :

| title | authors | first | last | affiliation |
| --- | --- | --- | --- | --- |
| Informations supplémentaires An eco-epidemi… |   `[{"first":"Julien","last":"Mélade","suffix":"","affiliation":{"laboratory":"Centre de Recher…` | Julien | Mélade | `   {"laboratory":"Centre de Recher…` |
Informations supplémentaires An eco-epidemi… | `[{"first":"Nicolas","last":"4#","suffix":"","affiliation":{"laboratory":"","institution":"U…` | Nicolas | 4# |`{"laboratory":"","institution":"U…` | 
| Informations supplémentaires An eco-epidemi… |   `[{"first":"Beza","last":"Ramazindrazana","suffix":"","affiliation":{"laboratory":"Centre de Recher…` | Beza | Ramazindrazana | `{"laboratory":"Centre de Recher…` |
| Informations supplémentaires An eco-epidemi… |   `[{"first":"Olivier","last":"Flores","suffix":"","affiliation":{"laboratory":"UMR C53 CIRAD, …` | Olivier | Flores |`{"laboratory":"UMR C53 CIRAD, …` |     

> [!IMPORTANT]
> Si vous voyez des caractères inattendus dans votre texte, par exemple `MÃƒÂ©lade` au lieu de `Mélade`, cela signifie que votre classement de base de données n’est pas défini sur le classement [UTF-8](https://docs.microsoft.com/sql/relational-databases/collations/collation-and-unicode-support#utf8). [Remplacez le classement de la base de données](https://docs.microsoft.com/sql/relational-databases/collations/set-or-change-the-database-collation#to-change-the-database-collation) par un classement UTF-8 à l’aide d’une instruction SQL telle que `ALTER DATABASE MyLdw COLLATE LATIN1_GENERAL_100_CI_AS_SC_UTF8`.

## <a name="azure-cosmos-db-to-sql-type-mappings"></a>Mappages de type Azure Cosmos DB à SQL

Même si le magasin transactionnel Azure Cosmos DB est indépendant du schéma, le magasin analytique est schématisé afin de l’optimiser pour les performances de requête analytique. Avec la fonctionnalité de synchronisation automatique d’Azure Synapse Link, Azure Cosmos DB gère la représentation de schéma dans le magasin analytique prêt à l’emploi qui comprend la gestion des types de données imbriqués. Étant donné qu’un pool SQL serverless interroge le magasin analytique, il est important de comprendre comment mapper des types de données d’entrée Azure Cosmos DB à des types de données SQL.

Les comptes Azure Cosmos DB de l’API SQL (Core) prennent en charge les types de propriété JSON nombre, chaîne, booléen, null, objet imbriqué ou tableau. Vous devez choisir des types SQL correspondant à ces types JSON si vous utilisez `WITH` dans la clause `OPENROWSET`. Le tableau suivant montre les types de colonnes SQL à utiliser pour les différents types de propriétés dans Azure Cosmos DB.

| Type de propriété Azure Cosmos DB | Type de colonne SQL |
| --- | --- |
| Booléen | bit |
| Integer | bigint |
| Decimal | float |
| String | varchar (classement de base de données UTF-8) |
| Date et heure (chaîne au format ISO) | varchar(30) |
| Date et heure (horodatage UNIX) | bigint |
| Null | `any SQL type` 
| Objet ou tableau imbriqués | varchar(max) (classement de base de données UTF-8), sérialisé en tant que texte JSON |

## <a name="full-fidelity-schema"></a>Schéma de fidélité optimale

Le schéma de fidélité optimale Azure Cosmos DB enregistre les valeurs et leurs meilleurs types de correspondance pour chaque propriété au sein d’un conteneur. La fonction `OPENROWSET` sur un conteneur avec un schéma de fidélité optimale fournit le type et la valeur réelle dans chaque cellule. Supposons que la requête suivante lit les éléments d’un conteneur avec un schéma de fidélité optimale :

```sql
SELECT *
FROM OPENROWSET(
      'CosmosDB',
      'account=MyCosmosDbAccount;database=covid;region=westus2;key=C0Sm0sDbKey==',
       Ecdc
    ) as rows
```

Le résultat de cette requête renvoie des types et des valeurs au format de texte JSON :

| date_rep | cas | geo_id |
| --- | --- | --- |
| {"date":"2020-08-13"} | {"int32":"254"} | {"string":"RS"} |
| {"date":"2020-08-12"} | {"int32":"235"}| {"string":"RS"} |
| {"date":"2020-08-11"} | {"int32":"316"} | {"string":"RS"} |
| {"date":"2020-08-10"} | {"int32":"281"} | {"string":"RS"} |
| {"date":"2020-08-09"} | {"int32":"295"} | {"string":"RS"} |
| {"string":"2020/08/08"} | {"int32":"312"} | {"string":"RS"} |
| {"date":"2020-08-07"} | {"float64":"339.0"} | {"string":"RS"} |

Pour chaque valeur, vous pouvez voir le type identifié dans un élément de conteneur Azure Cosmos DB. La plupart des valeurs pour la propriété `date_rep` sont des valeurs `date`, mais certaines sont stockées de manière incorrecte en tant que chaînes (string) dans Azure Cosmos DB. Le schéma de fidélité optimale retourne les valeurs `date` correctement typées et les valeurs `string` mises en forme de manière incorrecte.
Le nombre de cas est une information stockée en tant que valeur `int32`, mais il existe une valeur entrée sous la forme d’un nombre décimal. Cette valeur a le type `float64`. Si certaines valeurs dépassent le plus grand nombre `int32`, elles sont stockées en tant que type `int64`. Toutes les valeurs `geo_id` de cet exemple sont stockées en tant que types `string`.

> [!IMPORTANT]
> La fonction `OPENROWSET` sans clause `WITH` expose les valeurs avec les types attendus et les valeurs dont les types sont incorrectement entrés. Cette fonction est conçue pour l’exploration de données, non pour la création de rapports. N’analysez pas les valeurs JSON retournées par cette fonction pour créer des rapports. Utilisez une [clause WITH](#query-items-with-full-fidelity-schema) explicite pour créer vos rapports. Vous devez nettoyer les valeurs dont les types sont incorrects dans le conteneur Azure Cosmos DB pour appliquer des corrections dans le magasin analytique de fidélité optimale.

Si vous devez interroger des comptes Azure Cosmos DB du type d’API Mongo DB, apprenez-en davantage sur la représentation du schéma de fidélité optimale dans le magasin analytique et sur les noms de propriétés étendues à utiliser dans [Qu’est-ce que le magasin analytique Azure Cosmos DB ?](../../cosmos-db/analytical-store-introduction.md#analytical-schema).

### <a name="query-items-with-full-fidelity-schema"></a>Interroger des éléments avec un schéma de fidélité optimale

Lors de l’interrogation du schéma de fidélité optimale, vous devez spécifier explicitement le type SQL et le type de propriété Azure Cosmos DB attendu dans la clause `WITH`. N’utilisez pas la fonction `OPENROWSET` sans clause `WITH` dans les rapports, car le format du jeu de résultats pourrait être modifié dans la préversion résultant des commentaires.

Dans l’exemple suivant, nous allons supposer que le type `string` est correct pour la propriété `geo_id`, et que le type `int32` est correct pour la propriété `cases` :

```sql
SELECT geo_id, cases = SUM(cases)
FROM OPENROWSET(
      'CosmosDB'
      'account=MyCosmosDbAccount;database=covid;region=westus2;key=C0Sm0sDbKey==',
       Ecdc
    ) WITH ( geo_id VARCHAR(50) '$.geo_id.string',
             cases INT '$.cases.int32'
    ) as rows
GROUP BY geo_id
```

Des valeurs `geo_id` et `cases` d’autres types sont retournées en tant que valeurs de `NULL`. Cette requête fait référence uniquement aux `cases` avec le type spécifié dans l’expression (`cases.int32`).

Si vous avez des valeurs avec d’autres types (`cases.int64`, `cases.float64`) qui ne peuvent pas être nettoyées dans le conteneur Azure Cosmos DB, vous devez les référencer explicitement dans une clause `WITH` et combiner les résultats. La requête suivante agrège les valeurs `int32`, `int64` et `float64` stockées dans la colonne `cases`:

```sql
SELECT geo_id, cases = SUM(cases_int) + SUM(cases_bigint) + SUM(cases_float)
FROM OPENROWSET(
      'CosmosDB',
      'account=MyCosmosDbAccount;database=covid;region=westus2;key=C0Sm0sDbKey==',
       Ecdc
    ) WITH ( geo_id VARCHAR(50) '$.geo_id.string', 
             cases_int INT '$.cases.int32',
             cases_bigint BIGINT '$.cases.int64',
             cases_float FLOAT '$.cases.float64'
    ) as rows
GROUP BY geo_id
```

Dans cet exemple, le nombre de cas est stocké en tant que valeurs `int32`, `int64` ou `float64`. Toutes les valeurs doivent être extraites pour calculer le nombre de cas par pays.

## <a name="known-issues"></a>Problèmes connus

- L’expérience de requête qu’offre un pool SQL serverless pour un [schéma de fidélité optimale Azure Cosmos DB](#full-fidelity-schema) est un comportement temporaire qui sera modifié en fonction des commentaires sur la préversion. Ne vous fiez pas au schéma que la fonction `OPENROWSET` sans clause `WITH` fournit pendant la période de préversion publique, car l’expérience de requête pourrait être alignée avec un schéma bien défini en fonction des commentaires des clients. Contactez l’[équipe produit Synapse Link](mailto:cosmosdbsynapselink@microsoft.com) pour lui faire part de vos commentaires.
- Un pool SQL serverless retourne un avertissement au moment de la compilation si le classement de la colonne `OPENROWSET` n’a pas d’encodage UTF-8. Vous pouvez facilement modifier le classement par défaut pour toutes les fonctions `OPENROWSET` en cours d’exécution dans la base de données actuelle à l’aide de l’instruction T-SQL `alter database current collate Latin1_General_100_CI_AS_SC_UTF8`.

Les erreurs possibles et les actions de résolution des problèmes sont répertoriées dans le tableau suivant.

| Erreur | Cause racine |
| --- | --- |
| Erreurs de syntaxe :<br/> - Syntaxe incorrecte près de `Openrowset`<br/> -  `...` n’est pas une option reconnue de fournisseur de `BULK OPENROWSET`.<br/> - Syntaxe incorrecte près de `...` | Causes principales possibles :<br/> - N’utilise pas CosmosDB comme premier paramètre,<br/> - Utilise un littéral de chaîne au lieu d’un identificateur dans le troisième paramètre,<br/> - Ne spécifie pas le troisième paramètre (nom de conteneur). |
| Une erreur s’est produite dans la chaîne de connexion CosmosDB. | -Le compte, la base de données ou la clé n’est pas spécifié(e), <br/> - Une option dans une chaîne de connexion n’est pas reconnue,<br/> - Un point-virgule `;` est placé à la fin d’une chaîne de connexion. |
| La résolution du chemin d’accès CosmosDB a échoué avec l’erreur « Nom de compte incorrect » ou « Nom de base de données incorrect ». | Le nom de compte, le nom de la base de données ou le conteneur spécifié est introuvable, ou le stockage analytique n’a pas été activé pour la collection spécifiée.|
| La résolution du chemin d’accès CosmosDB a échoué avec l’erreur « Valeur de secret incorrecte » ou « Secret null ou vide ». | La clé du compte n’est pas valide ou est manquante. |
| La colonne `column name` de type `type name` n’est pas compatible avec le type de données externe `type name`. | Le type de colonne spécifié dans la clause `WITH` ne correspond pas au type dans le conteneur Azure Cosmos DB. Essayez de modifier le type de colonne tel que décrit dans la section [Mappages de type Azure Cosmos DB à SQL](#azure-cosmos-db-to-sql-type-mappings), ou utilisez le type `VARCHAR`. |
| La colonne contient `NULL` valeurs dans toutes les cellules. | Possibilité d’une erreur de nom de colonne ou d’expression de chemin d’accès dans la clause `WITH`. Le nom de colonne (ou l’expression de chemin d’accès après le type de colonne) dans la clause `WITH` doit correspondre à un nom de propriété dans la collection Azure Cosmos DB. La comparaison *respecte la casse*. Par exemple, `productCode` et `ProductCode` sont des propriétés différentes. |

Vous pouvez nous faire part de vos suggestions et signaler des problèmes dans la [page de commentaires Azure Synapse Analytics](https://feedback.azure.com/forums/307516-azure-synapse-analytics?category_id=387862).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants :

- [Utiliser Power BI et un pool SQL serverless avec Azure Synapse Link](../../cosmos-db/synapse-link-power-bi.md)
- [Créer et utiliser des vues dans un pool SQL serverless](create-use-views.md)
- [Tutoriel sur la création de vues de pools SQL serverless sur Azure Cosmos DB et leur connexion à des modèles Power BI via DirectQuery](./tutorial-data-analyst.md)

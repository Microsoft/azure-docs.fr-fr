---
title: Analyser un schéma avec des tableaux et des structures imbriquées
description: Analyser des tableaux et des structures imbriquées avec Apache Spark et SQL
services: synapse-analytics
author: ArnoMicrosoft
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: spark
ms.date: 06/15/2020
ms.author: acomet
ms.reviewer: jrasnick
ms.openlocfilehash: 43e4027c9d88b209ed0606e045dd5b4d47d3ff6e
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92088514"
---
# <a name="analyze-complex-data-types-in-azure-synapse-analytics"></a>Analyser des types de données complexes dans Azure Synapse Analytics

Cet article s’applique aux fichiers et aux conteneurs Parquet de la section [Azure Synapse Link pour Azure Cosmos DB](.\synapse-link\how-to-connect-synapse-link-cosmos-db.md). Vous pouvez utiliser Spark ou SQL pour lire ou transformer des données avec des schémas complexes, comme des tableaux ou des structures imbriquées. L’exemple suivant n’utilise qu’un seul document, mais il peut facilement être mis à l’échelle pour inclure des milliards de documents avec Spark ou SQL. Le code inclus dans cet article utilise PySpark (Python).

## <a name="use-case"></a>Cas d’utilisation

Les types de données complexes sont de plus en plus courants et représentent un défi pour les ingénieurs Données. L’analyse des schémas et des tableaux imbriqués peut impliquer des requêtes SQL complexes et demandant un temps considérable. Il peut être également difficile de renommer ou de caster le type de données des colonnes imbriquées. En outre, quand vous utilisez des objets imbriqués profondément, vous pouvez rencontrer des problèmes de performances.

Les ingénieurs Données doivent comprendre comment traiter efficacement des types de données complexes et les rendre facilement accessibles à tous. Dans l’exemple suivant, vous utilisez Spark dans Azure Synapse Analytics pour lire et transformer des objets en une structure plate via des trames de données. Vous utilisez le modèle serverless de SQL dans Azure Synapse Analytics pour interroger directement ces objets et retourner ces résultats sous la forme d’un tableau standard.

## <a name="what-are-arrays-and-nested-structures"></a>Que sont les tableaux et les structures imbriquées ?

L’objet suivant provient d’[Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview). Cet objet contient des structures imbriquées et des tableaux incluant d’autres structures imbriquées.

```json
{
    "id": "66532691-ab20-11ea-8b1d-936b3ec64e54",
    "context": {
        "data": {
            "eventTime": "2020-06-10T13:43:34.553Z",
            "samplingRate": "100.0",
            "isSynthetic": "false"
        },
        "session": {
            "isFirst": "false",
            "id": "38619c14-7a23-4687-8268-95862c5326b1"
        },
        "custom": {
            "dimensions": [
                {
                    "customerInfo": {
                        "ProfileType": "ExpertUser",
                        "RoomName": "",
                        "CustomerName": "diamond",
                        "UserName": "XXXX@yahoo.com"
                    }
                },
                {
                    "customerInfo": {
                        "ProfileType": "Novice",
                        "RoomName": "",
                        "CustomerName": "topaz",
                        "UserName": "XXXX@outlook.com"
                    }
                }
            ]
        }
    }
}
```

### <a name="schema-example-of-arrays-and-nested-structures"></a>Exemple de schéma de tableaux et de structures imbriquées
Quand vous affichez le schéma de la trame de données (appelée **df**) de cet objet avec la commande `df.printschema`, vous voyez la représentation suivante :

* Le jaune représente les structure imbriquées.
* Le vert représente un tableau avec deux éléments.

[![Code avec mise en évidence en jaune et en vert, montrant l’origine du schéma](./media/how-to-complex-schema/schema-origin.png)](./media/how-to-complex-schema/schema-origin.png#lightbox)

`_rid`, `_ts` et `_etag` ont été ajoutés au système à mesure que le document était ingéré dans le magasin transactionnel d’Azure Cosmos DB.

Le nombre de trames de données ci-dessus a 5 colonnes et 1 ligne seulement. Après la transformation, la trame de données organisée aura 13 colonnes et 2 lignes, dans un format tabulaire.

## <a name="flatten-nested-structures-and-explode-arrays"></a>Aplatir des structures imbriquées et éclater des tableaux

Avec Spark dans Azure Synapse Analytics, vous pouvez facilement transformer des structures imbriquées en colonnes et des éléments de tableau en plusieurs lignes. Utilisez les étapes suivantes pour l’implémentation.

[![Organigramme montrant les étapes des transformations Spark](./media/how-to-complex-schema/spark-transform-steps.png)](./media/how-to-complex-schema/spark-transform-steps.png#lightbox)

### <a name="define-a-function-to-flatten-the-nested-schema"></a>Définir une fonction pour aplatir le schéma imbriqué

Vous pouvez utiliser cette fonction sans modification. Créez une cellule dans un [notebook PySpark](quickstart-apache-spark-notebook.md) à l’aide de la fonction suivante :

```python
from pyspark.sql.functions import col

def flatten_df(nested_df):
    stack = [((), nested_df)]
    columns = []

    while len(stack) > 0:
        parents, df = stack.pop()

        flat_cols = [
            col(".".join(parents + (c[0],))).alias("_".join(parents + (c[0],)))
            for c in df.dtypes
            if c[1][:6] != "struct"
        ]

        nested_cols = [
            c[0]
            for c in df.dtypes
            if c[1][:6] == "struct"
        ]

        columns.extend(flat_cols)

        for nested_col in nested_cols:
            projected_df = df.select(nested_col + ".*")
            stack.append((parents + (nested_col,), projected_df))

    return nested_df.select(columns)
```

### <a name="use-the-function-to-flatten-the-nested-schema"></a>Utiliser la fonction pour aplatir le schéma imbriqué

Dans cette étape, vous aplatissez le schéma imbriqué de la trame de données (**df**) dans une nouvelle trame de données (`df_flat`) :

```python
from pyspark.sql.types import StringType, StructField, StructType
df_flat = flatten_df(df)
display(df_flat.limit(10))
```

La fonction d’affichage doit retourner 10 colonnes et 1 ligne. Le tableau et ses éléments imbriqués sont toujours là.

### <a name="transform-the-array"></a>Transformer le tableau

Ici, vous transformez le tableau `context_custom_dimensions` de la trame de données `df_flat` en une nouvelle trame de données `df_flat_explode`. Dans le code suivant, vous définissez aussi la colonne à sélectionner :

```python
from pyspark.sql.functions import explode
from pyspark.sql.functions import flatten
from pyspark.sql.functions import arrays_zip
df_flat_explode = df_flat.select("_rid","_ts","id","_etag",explode(df_flat.context_custom_dimensions),"context_session_isFirst","context_session_id","context_data_eventTime","context_data_samplingRate","context_data_isSynthetic")\
.select("_rid","_ts","id","_etag","col.*","context_session_isFirst","context_session_id","context_data_eventTime","context_data_samplingRate","context_data_isSynthetic")
display(df_flat_explode.limit(10))

```

La fonction d’affichage doit retourner 10 colonnes et 2 lignes. L’étape suivante consiste à aplatir les schémas imbriqués avec la fonction définie à l’étape 1.

### <a name="use-the-function-to-flatten-the-nested-schema"></a>Utiliser la fonction pour aplatir le schéma imbriqué

Enfin, vous utilisez cette fonction pour aplatir le schéma imbriqué de la trame de données `df_flat_explode` en une nouvelle trame de données `df_flat_explode_flat` :
```python
df_flat_explode_flat = flatten_df(df_flat_explode)
display(df_flat_explode_flat.limit(10))
```

La fonction d’affichage doit retourner 13 colonnes et 2 lignes.

La fonction `printSchema` de la trame de données `df_flat_explode_flat` retourne le résultat suivant :

[![Code montrant le schéma final](./media/how-to-complex-schema/schema-final.png)](./media/how-to-complex-schema/schema-final.png#lightbox)

## <a name="read-arrays-and-nested-structures-directly"></a>Lire directement des tableaux et des structures imbriquées

Avec le modèle serverless de SQL, vous pouvez interroger et créer des vues et des tables sur ces objets.

Tout d’abord, en fonction du mode de stockage des données, les utilisateurs doivent utiliser la taxonomie suivante. Tout ce qui est indiqué en majuscules est spécifique à votre cas d’usage :

| Bloc | Format |
| ------ | ------ |
| 'https://ACCOUNTNAME.dfs.core.windows.net/FILESYSTEM/PATH/FINENAME.parquet' |'Parquet' (ADLSg2)|
| N'endpoint=https://ACCOUNTNAME.documents-staging.windows-ppe.net:443/;account=ACCOUNTNAME;database=DATABASENAME;collection=COLLECTIONNAME;region=REGIONTOQUERY', SECRET='YOURSECRET' |'CosmosDB' (Azure Synapse Link)|


Remplacez chaque champ comme suit :
* « YOUR BULK ABOVE » est la chaîne de connexion de la source de données à laquelle vous vous connectez.
* « YOUR TYPE ABOVE » est le format que vous utilisez pour vous connecter à la source.

```sql
select *
FROM
openrowset(
    BULK 'YOUR BULK ABOVE',
    FORMAT='YOUR TYPE ABOVE'
)
with (id varchar(50),
        contextdataeventTime varchar(50) '$.context.data.eventTime',
        contextdatasamplingRate varchar(50) '$.context.data.samplingRate',
        contextdataisSynthetic varchar(50) '$.context.data.isSynthetic',
        contextsessionisFirst varchar(50) '$.context.session.isFirst',
        contextsessionid varchar(50) '$.context.session.id',
        contextcustomdimensions varchar(max) '$.context.custom.dimensions'
) as q 
cross apply openjson (contextcustomdimensions) 
with ( ProfileType varchar(50) '$.customerInfo.ProfileType',
            RoomName varchar(50) '$.customerInfo.RoomName',
            CustomerName varchar(50) '$.customerInfo.CustomerName',
            UserName varchar(50) '$.customerInfo.UserName'
    )
```

Il existe deux types d’opérations :

- Le premier type d’opération est indiqué dans la ligne de code suivante, qui définit la colonne appelée `contextdataeventTime` qui fait référence à l’élément imbriqué `Context.Data.eventTime`. 
  ```sql
  contextdataeventTime varchar(50) '$.context.data.eventTime'
  ```

  Cette ligne définit la colonne appelée `contextdataeventTime` qui fait référence à l’élément imbriqué `Context>Data>eventTime`.

- Le deuxième type d’opération utilise `cross apply` pour créer de nouvelles lignes pour chaque élément sous le tableau. Ensuite, il définit chaque objet imbriqué. 
  ```sql
  cross apply openjson (contextcustomdimensions) 
  with ( ProfileType varchar(50) '$.customerInfo.ProfileType', 
  ```

  Si le tableau comporte 5 éléments avec 4 structures imbriquées, le modèle serverless de SQL retourne 5 lignes et 4 colonnes. Le modèle serverless de SQL peut interroger sur place, mapper le tableau dans 2 lignes, et afficher toutes les structures imbriquées dans des colonnes.

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment interroger Synapse Link pour Azure Cosmos DB à l’aide de Spark](./synapse-link/how-to-query-analytical-store-spark.md)
* [Interroger des types imbriqués Parquet](./sql/query-parquet-nested-types.md) 

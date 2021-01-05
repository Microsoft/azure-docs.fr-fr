---
title: Mappage de schéma et de type de données dans l’activité de copie
description: Découvrez comment l’activité de copie dans Azure Data Factory mappe les schémas et les types de données des données de la source aux données du récepteur.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: craigg
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 06/22/2020
ms.author: jingwang
ms.openlocfilehash: 2b54ee29b1b03bab5af8410a3fae06438180299d
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97507521"
---
# <a name="schema-and-data-type-mapping-in-copy-activity"></a>Mappage de schéma et de type de données dans l’activité de copie
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article décrit la manière dont l’activité de copie d’Azure Data Factory effectue un mappage de schéma et de type de données, des données de la source au données du récepteur.

## <a name="schema-mapping"></a>Mappage de schéma

### <a name="default-mapping"></a>Mappage par défaut

Par défaut, l’activité de copie mappe les données sources au récepteur **par noms de colonnes** de manière sensible à la casse. Si le récepteur n’existe pas, par exemple, en écrivant dans un ou plusieurs fichiers, les noms des champs sources sont rendus persistants en tant que noms de récepteurs. Ce mappage par défaut prend en charge des schémas flexibles et des dérives de schéma de la source au récepteur d’une exécution à l’autre. Toutes les données retournées par le magasin de données source peuvent être copiées dans le récepteur.

Si votre source est un fichier texte sans ligne d’en-tête, un [mappage explicite](#explicit-mapping) est requis, car la source ne contient pas de noms de colonne.

### <a name="explicit-mapping"></a>Mappage explicite

Vous pouvez également spécifier un mappage explicite pour personnaliser le mappage de colonne/de champ de la source au récepteur selon vos besoins. Avec le mappage explicite, vous pouvez copier uniquement des données sources partielles vers le récepteur, mapper les données sources au récepteur avec des noms différents ou remodeler les données tabulaires/hiérarchiques. Activité de copie :

1. Lit les données de la source et détermine le schéma source.
2. Applique votre mappage défini.
3. Écrit les données sur le récepteur.

Pour en savoir plus :

- [De la source tabulaire vers le récepteur tabulaire](#tabular-source-to-tabular-sink)
- [De la source hiérarchique vers le récepteur tabulaire](#hierarchical-source-to-tabular-sink)
- [De la source tabulaire/hiérarchique vers le récepteur hiérarchique](#tabularhierarchical-source-to-hierarchical-sink)

Vous pouvez configurer le mappage sur Interface utilisateur Data Factory -> activité de copie -> onglet Mappage, ou spécifier programmatiquement le mappage dans activité de copie -> propriété `translator`. Les propriétés suivantes sont prises en charge dans `translator` -> `mappings` tableau -> objets -> `source` et `sink`, qui pointe vers la colonne/le champ spécifique pour mapper les données.

| Propriété | Description                                                  | Obligatoire |
| -------- | ------------------------------------------------------------ | -------- |
| name     | Nom de la colonne/du champ source ou récepteur. Appliquer à la source tabulaire et au récepteur. | Oui      |
| ordinal  | Index de colonne. Démarrer à partir de 1. <br>À appliquer et requis lors de l’utilisation de texte sans ligne d’en-tête délimité. | Non       |
| path     | Expression de chemin JSON pour l’extraction ou le mappage de chaque champ. Appliquer à la source hiérarchique et au récepteur, par exemple, les connecteurs Cosmos DB, MongoDB ou REST.<br>Pour les champs situés sous l’objet racine, le chemin JSON commence par la racine `$` ; pour les champs qui se trouvent dans le tableau sélectionné par la propriété `collectionReference`, le chemin JSON commence par l’élément de tableau sans `$`. | Non       |
| type     | Type de données intermédiaires Data Factory de la colonne source ou récepteur. En général, vous n’avez pas besoin de spécifier ni de modifier cette propriété. En savoir plus sur le [mappage des types de données](#data-type-mapping). | Non       |
| culture  | Culture de la colonne source ou récepteur. À appliquer lorsque le type est `Datetime` ou `Datetimeoffset`. Par défaut, il s’agit de `en-us`.<br>En général, vous n’avez pas besoin de spécifier ni de modifier cette propriété. En savoir plus sur le [mappage des types de données](#data-type-mapping). | Non       |
| format   | Chaîne de format à utiliser lorsque le type est `Datetime` ou `Datetimeoffset`. Reportez-vous à [Chaînes de format Date et Heure personnalisées](/dotnet/standard/base-types/custom-date-and-time-format-strings) sur la mise en forme des date/heure. En général, vous n’avez pas besoin de spécifier ni de modifier cette propriété. En savoir plus sur le [mappage des types de données](#data-type-mapping). | Non       |

Les propriétés suivantes sont prises en charge sous `translator` en plus de `mappings` :

| Propriété            | Description                                                  | Obligatoire |
| ------------------- | ------------------------------------------------------------ | -------- |
| collectionReference | Appliquer lors de la copie de données de la source hiérarchique, par exemple, les connecteurs Cosmos DB, MongoDB ou REST.<br>Si vous souhaitez effectuer une itération et extraire des données à partir des objets situés **à l’intérieur d’un champ de tableau** présentant le même modèle et effectuer une conversion par ligne et par objet, spécifiez le chemin JSON de ce tableau afin d’effectuer une application croisée. | Non       |

#### <a name="tabular-source-to-tabular-sink"></a>De la source tabulaire vers le récepteur tabulaire

Par exemple, pour copier des données de Salesforce vers Azure SQL Database et mapper explicitement trois colonnes :

1. Sur activité de copie -> onglet de mappage, cliquez sur le bouton **Importer le schéma** pour importer les schémas source et récepteur.

2. Mappez les champs nécessaires et excluez/supprimez le reste.

![Mapper d’un tableau à un autre](media/copy-activity-schema-and-type-mapping/map-tabular-to-tabular.png)

Le même mappage peut être configuré comme suit dans la charge utile de l’activité de copie (voir `translator`) :

```json
{
    "name": "CopyActivityTabularToTabular",
    "type": "Copy",
    "typeProperties": {
        "source": { "type": "SalesforceSource" },
        "sink": { "type": "SqlSink" },
        "translator": {
            "type": "TabularTranslator",
            "mappings": [
                {
                    "source": { "name": "Id" },
                    "sink": { "name": "CustomerID" }
                },
                {
                    "source": { "name": "Name" },
                    "sink": { "name": "LastName" }
                },
                {
                    "source": { "name": "LastModifiedDate" },
                    "sink": { "name": "ModifiedDate" }
                }
            ]
        }
    },
    ...
}
```

Pour copier des données à partir d’un ou de plusieurs fichiers texte délimités sans ligne d’en-tête, les colonnes sont représentées par numéros plutôt que par noms. 

```json
{
    "name": "CopyActivityTabularToTabular",
    "type": "Copy",
    "typeProperties": {
        "source": { "type": "DelimitedTextSource" },
        "sink": { "type": "SqlSink" },
        "translator": {
            "type": "TabularTranslator",
            "mappings": [
                {
                    "source": { "ordinal": "1" },
                    "sink": { "name": "CustomerID" }
                }, 
                {
                    "source": { "ordinal": "2" },
                    "sink": { "name": "LastName" }
                }, 
                {
                    "source": { "ordinal": "3" },
                    "sink": { "name": "ModifiedDate" }
                }
            ]
        }
    },
    ...
}
```

#### <a name="hierarchical-source-to-tabular-sink"></a>De la source hiérarchique vers le récepteur tabulaire

Lors de la copie de données d’une source hiérarchique vers un récepteur tabulaire, l’activité de copie prend en charge les fonctionnalités suivantes :

- Extraire des données d’objets et de tableaux.
- Cross applique plusieurs objets avec le même modèle à partir d’un tableau, dans ce cas pour convertir un objet JSON en plusieurs enregistrements dans un résultat tabulaire.

Pour une transformation hiérarchique-à-tabulaire plus avancée, vous pouvez utiliser [Data Flow](concepts-data-flow-overview.md). 

Par exemple, si vous avez un document MongoDB source avec le contenu suivant :

```json
{
    "id": {
        "$oid": "592e07800000000000000000"
    },
    "number": "01",
    "date": "20170122",
    "orders": [
        {
            "prod": "p1",
            "price": 23
        },
        {
            "prod": "p2",
            "price": 13
        },
        {
            "prod": "p3",
            "price": 231
        }
    ],
    "city": [ { "name": "Seattle" } ]
}
```

Et que vous souhaitez copier ce fichier dans un fichier texte au format suivant avec une ligne d’en-tête, en mettant à plat les données se trouvant dans le tableau *(order_pd and order_price)* et en effectuant une jointure croisée avec les informations racines communes *(numéro, date et ville)*  :

| orderNumber | orderDate | order_pd | order_price | city    |
| ----------- | --------- | -------- | ----------- | ------- |
| 01          | 20170122  | P1       | 23          | Seattle |
| 01          | 20170122  | P2       | 13          | Seattle |
| 01          | 20170122  | P3       | 231         | Seattle |

Vous pouvez définir un mappage de ce type sur l’interface utilisateur de création Data Factory :

1. Sur activité de copie -> onglet de mappage, cliquez sur le bouton **Importer le schéma** pour importer les schémas source et récepteur. Comme Data Factory échantillonne les quelques objets les plus importants lors de l’importation d’un schéma, si un champ n’apparaît pas, vous pouvez l’ajouter à la couche appropriée de la hiérarchie : pointez sur un nom de champ existant et choisissez d’ajouter un nœud, un objet ou un tableau.

2. Sélectionnez le tableau à partir duquel vous souhaitez effectuer une itération et extraire des données. Il sera automatiquement renseigné en tant que **Référence de collection**. Notez qu’un seul un tableau est pris en charge pour une telle opération.

3. Mappez les champs nécessaires au récepteur. Data Factory détermine automatiquement les chemins JSON correspondants pour le côté hiérarchique.

![Mapper hiérarchiquement sous forme de tableau à l’aide de l’interface utilisateur](media/copy-activity-schema-and-type-mapping/map-hierarchical-to-tabular-ui.png)

Vous pouvez également basculer vers **Éditeur avancé**. Dans ce cas, vous pouvez afficher et modifier directement les chemins JSON des champs. Si vous choisissez d’ajouter un nouveau mappage dans cette vue, spécifiez le chemin JSON.

![Mapper hiérarchiquement sous forme de tableau à l’aide de l’éditeur avancé](media/copy-activity-schema-and-type-mapping/map-hierarchical-to-tabular-advanced-editor.png)

Le même mappage peut être configuré comme suit dans la charge utile de l’activité de copie (voir `translator`) :


```json
{
    "name": "CopyActivityHierarchicalToTabular",
    "type": "Copy",
    "typeProperties": {
        "source": { "type": "MongoDbV2Source" },
        "sink": { "type": "DelimitedTextSink" },
        "translator": {
            "type": "TabularTranslator",
            "mappings": [
                {
                    "source": { "path": "$['number']" },
                    "sink": { "name": "orderNumber" }
                },
                {
                    "source": { "path": "$['date']" },
                    "sink": { "name": "orderDate" }
                },
                {
                    "source": { "path": "['prod']" },
                    "sink": { "name": "order_pd" }
                },
                {
                    "source": { "path": "['price']" },
                    "sink": { "name": "order_price" }
                },
                {
                    "source": { "path": "$['city'][0]['name']" },
                    "sink": { "name": "city" }
                }
            ],
            "collectionReference": "$['orders']"
        }
    },
    ...
}
```

#### <a name="tabularhierarchical-source-to-hierarchical-sink"></a>De la source tabulaire/hiérarchique vers le récepteur hiérarchique

Le flux de l’expérience utilisateur est similaire à [De la source hiérarchique vers le récepteur tabulaire](#hierarchical-source-to-tabular-sink). 

Lors de la copie de données d’une source tabulaire vers un récepteur hiérarchique, l’écriture dans un tableau à l’intérieur de l’objet n’est pas prise en charge.

Lorsque vous copiez des données à partir d’une source hiérarchique vers un récepteur hiérarchique, vous pouvez également conserver la hiérarchie de la couche entière en sélectionnant l’objet/le tableau et mapper au récepteur sans toucher aux champs internes.

Pour une transformation de remodelage plus avancée des données, vous pouvez utiliser [Data Flow](concepts-data-flow-overview.md). 

### <a name="parameterize-mapping"></a>Paramétrer le mappage

Si vous souhaitez créer un pipeline modélisé pour copier dynamiquement un grand nombre d’objets, déterminez si vous pouvez tirer parti du [mappage par défaut](#default-mapping) ou si vous devez définir un [mappage explicite](#explicit-mapping) pour les objets respectifs.

Si vous avez besoin d’un mappage explicite, vous pouvez procéder comme suit :

1. Définir un paramètre avec le type d’objet au niveau du pipeline, par exemple, `mapping`.

2. Paramétrer le mappage : sur activité de copie -> onglet de mappage, choisissez d’ajouter du contenu dynamique et sélectionnez le paramètre ci-dessus. La charge utile d’activité serait la suivante :

    ```json
    {
        "name": "CopyActivityHierarchicalToTabular",
        "type": "Copy",
        "typeProperties": {
            "source": {...},
            "sink": {...},
            "translator": {
                "value": "@pipeline().parameters.mapping",
                "type": "Expression"
            },
            ...
        }
    }
    ```

3. Construisez la valeur à transférer dans le paramètre de mappage. Il doit s’agir de l’intégralité de l’objet de la définition de `translator`, reportez-vous aux échantillons dans la section de [mappage explicite](#explicit-mapping). Par exemple, pour la copie de la source tabulaire vers le récepteur tabulaire, la valeur doit être `{"type":"TabularTranslator","mappings":[{"source":{"name":"Id"},"sink":{"name":"CustomerID"}},{"source":{"name":"Name"},"sink":{"name":"LastName"}},{"source":{"name":"LastModifiedDate"},"sink":{"name":"ModifiedDate"}}]}`.

## <a name="data-type-mapping"></a>Mappage de types de données

L’activité de copie effectue un mappage des types de la source aux types du récepteur selon le flux suivant : 

1. Conversion de types de données sources natives en types de données intermédiaires Azure Data Factory.
2. Conversion automatique du type de données intermédiaire en fonction des besoins pour les adapter aux types de récepteurs correspondants. Ceci est applicable à la fois au [mappage par défaut](#default-mapping) et au [mappage explicite](#explicit-mapping).
3. Conversion de types de données sources Azure Data Factory intermédiaires en types de données de récepteur natives.

L’activité de copie prend actuellement en charge les types de données intermédiaires suivants : Boolean, Byte, Byte array, Datetime, DatetimeOffset, Decimal, Double, GUID, Int16, Int32, Int64, SByte, Single, String, Timespan, UInt16, UInt32 et UInt64.

Les conversions de types de données suivantes sont prises en charge entre les types intermédiaires de la source au récepteur.

| Source\Récepteur | Boolean | Tableau d’octets | Decimal | Date/Heure <small>(1)</small> | Virgule flottante <small>(2)</small> | GUID | Integer <small>(3)</small> | String | TimeSpan |
| ----------- | ------- | ---------- | ------- | ---------------------------- | ------------------------------ | ---- | -------------------------- | ------ | -------- |
| Boolean     | ✓       |            | ✓       |                              | ✓                              |      | ✓                          | ✓      |          |
| Tableau d’octets  |         | ✓          |         |                              |                                |      |                            | ✓      |          |
| Date/Heure   |         |            |         | ✓                            |                                |      |                            | ✓      |          |
| Decimal     | ✓       |            | ✓       |                              | ✓                              |      | ✓                          | ✓      |          |
| Virgule flottante | ✓       |            | ✓       |                              | ✓                              |      | ✓                          | ✓      |          |
| GUID        |         |            |         |                              |                                | ✓    |                            | ✓      |          |
| Integer     | ✓       |            | ✓       |                              | ✓                              |      | ✓                          | ✓      |          |
| String      | ✓       | ✓          | ✓       | ✓                            | ✓                              | ✓    | ✓                          | ✓      | ✓        |
| TimeSpan    |         |            |         |                              |                                |      |                            | ✓      | ✓        |

(1) Date/Heure inclut DateTime et DateTimeOffset.

(2) Virgule flottante inclut Simple et Double.

(3) Integer inclut SByte, Byte, Int16, UInt16, Int32, UInt32, Int64 et UInt64.

> [!NOTE]
> - Cette conversion de type de données est actuellement prise en charge lors de la copie entre des données tabulaires. Les sources/récepteurs hiérarchiques ne sont pas pris en charge, ce qui signifie qu’il n’existe aucune conversion de type de données définie par le système entre les types intermédiaires source et récepteur.
> - Cette fonctionnalité fonctionne avec le modèle de jeu de données le plus récent. Si vous ne voyez pas cette option dans l’interface utilisateur, essayez de créer un nouveau jeu de données.

Les propriétés suivantes sont prises en charge dans l’activité de copie pour la conversion de types de données (sous `translator` section pour la création programmatique) :

| Propriété                         | Description                                                  | Obligatoire |
| -------------------------------- | ------------------------------------------------------------ | -------- |
| typeConversion                   | Activez la nouvelle expérience de conversion de type de données. <br>La valeur par défaut est false en raison de la compatibilité descendante.<br><br>Pour les nouvelles activités de copie créées via l’interface utilisateur de création de Data Factory depuis la fin juin 2020, cette conversion de type de données est activée par défaut pour une expérience optimale. Vous pouvez voir les paramètres de conversion de type suivants sur activité de copie -> onglet de mappage pour les scénarios applicables. <br>Pour créer un pipeline programmatiquement, vous devez définir explicitement la propriété `typeConversion` sur true pour l’activer.<br>Pour les activités de copie existantes créées avant la publication de cette fonctionnalité, vous ne verrez pas les options de conversion de type sur l’interface utilisateur de création de Data Factory pour la compatibilité descendante. | Non       |
| typeConversionSettings           | Groupe de paramètres de conversion de type. Appliquer lorsque `typeConversion` a la valeur `true`. Les propriétés suivantes sont toutes sous ce groupe. | Non       |
| *Sous : `typeConversionSettings`* |                                                              |          |
| allowDataTruncation              | Autorisez la troncation des données lors de la conversion des données sources en récepteur avec un type différent pendant la copie, par exemple, de décimal à entier, de DatetimeOffset à DateTime. <br>La valeur par défaut est true. | Non       |
| treatBooleanAsNumber             | Traitez les valeurs booléennes comme des nombres, par exemple true comme 1.<br>La valeur par défaut est false. | Non       |
| dateTimeFormat                   | Chaîne de format lors de la conversion entre des dates sans décalage de fuseau horaire et des chaînes, par exemple, `yyyy-MM-dd HH:mm:ss.fff`.  Reportez-vous à [Chaînes de format Date et Heure personnalisées](/dotnet/standard/base-types/custom-date-and-time-format-strings) pour obtenir des informations détaillées. | Non       |
| dateTimeOffsetFormat             | Chaîne de format lors de la conversion entre des dates avec décalage de fuseau horaire et des chaînes, par exemple, `yyyy-MM-dd HH:mm:ss.fff zzz`.  Reportez-vous à [Chaînes de format Date et Heure personnalisées](/dotnet/standard/base-types/custom-date-and-time-format-strings) pour obtenir des informations détaillées. | Non       |
| timeSpanFormat                   | Chaîne de format lors de la conversion entre des périodes de temps et des chaînes, par exemple, `dd\.hh\:mm`. Reportez-vous à [Chaînes de format TimeSpan personnalisées](/dotnet/standard/base-types/custom-timespan-format-strings) pour obtenir des informations détaillées. | Non       |
| culture                          | Informations concernant la culture à utiliser lors de la conversion de types, par exemple, `en-us` ou `fr-fr`. | Non       |

**Exemple :**

```json
{
    "name": "CopyActivity",
    "type": "Copy",
    "typeProperties": {
        "source": {
            "type": "ParquetSource"
        },
        "sink": {
            "type": "SqlSink"
        },
        "translator": {
            "type": "TabularTranslator",
            "typeConversion": true,
            "typeConversionSettings": {
                "allowDataTruncation": true,
                "treatBooleanAsNumber": true,
                "dateTimeFormat": "yyyy-MM-dd HH:mm:ss.fff",
                "dateTimeOffsetFormat": "yyyy-MM-dd HH:mm:ss.fff zzz",
                "timeSpanFormat": "dd\.hh\:mm",
                "culture": "en-gb"
            }
        }
    },
    ...
}
```

## <a name="legacy-models"></a>Modèles hérités

> [!NOTE]
> Les modèles suivants pour mapper des colonnes/champs sources vers un récepteur sont toujours pris en charge tels quels à des fins de compatibilité descendante. Nous vous suggérons d’utiliser le nouveau modèle mentionné dans [mappage de schéma](#schema-mapping). L’interface utilisateur de création de Data Factory a basculé vers la génération du nouveau modèle.

### <a name="alternative-column-mapping-legacy-model"></a>Autre mappage de colonnes (modèle hérité)

Vous pouvez spécifier l’activité de copie -> `translator` -> `columnMappings` pour le mappage entre les données mises en forme tabulaire. Dans ce cas, la section « structure » est requise pour les jeux de données d’entrée et de sortie. Le mappage de colonnes prend en charge le **mappage de la totalité ou d’un sous-ensemble des colonnes de la « structure » du jeu de données de la source à toutes les colonnes de la « structure » du jeu de données du récepteur**. Voici une liste de conditions d’erreur qui entraînent la levée d’une exception :

- Le résultat de la requête de banque de données source n’a pas de nom de colonne spécifié dans la section « structure » du jeu de données d’entrée.
- La banque de données du récepteur (si un schéma est prédéfini) n’a pas de nom de colonne spécifié dans la section « structure » du jeu de données de sortie.
- La « structure » du jeu de données du récepteur contient un nombre de colonnes inférieur ou supérieur à celui spécifié par le mappage.
- Mappage en double.

Dans l’exemple suivant, le jeu de données d’entrée possède une structure et pointe vers une table dans une base de données Oracle locale.

```json
{
    "name": "OracleDataset",
    "properties": {
        "structure":
         [
            { "name": "UserId"},
            { "name": "Name"},
            { "name": "Group"}
         ],
        "type": "OracleTable",
        "linkedServiceName": {
            "referenceName": "OracleLinkedService",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "tableName": "SourceTable"
        }
    }
}
```

Dans cet exemple, le jeu de données de sortie possède une structure et pointe vers une table dans Salesfoce.

```json
{
    "name": "SalesforceDataset",
    "properties": {
        "structure":
        [
            { "name": "MyUserId"},
            { "name": "MyName" },
            { "name": "MyGroup"}
        ],
        "type": "SalesforceObject",
        "linkedServiceName": {
            "referenceName": "SalesforceLinkedService",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "tableName": "SinkTable"
        }
    }
}
```

Le JSON suivant définit une activité de copie dans un pipeline. Les colonnes de la source sont mappées aux colonnes dans le récepteur en utilisant la propriété **translator** -> **columnMappings**.

```json
{
    "name": "CopyActivity",
    "type": "Copy",
    "inputs": [
        {
            "referenceName": "OracleDataset",
            "type": "DatasetReference"
        }
    ],
    "outputs": [
        {
            "referenceName": "SalesforceDataset",
            "type": "DatasetReference"
        }
    ],
    "typeProperties":    {
        "source": { "type": "OracleSource" },
        "sink": { "type": "SalesforceSink" },
        "translator":
        {
            "type": "TabularTranslator",
            "columnMappings":
            {
                "UserId": "MyUserId",
                "Group": "MyGroup",
                "Name": "MyName"
            }
        }
    }
}
```

Si vous utilisiez la syntaxe de `"columnMappings": "UserId: MyUserId, Group: MyGroup, Name: MyName"` pour spécifier le mappage de colonnes, il est toujours pris en charge tel quel.

### <a name="alternative-schema-mapping-legacy-model"></a>Autre mappage de schéma (modèle hérité)

Vous pouvez spécifier l’activité de copie -> `translator` -> `schemaMapping` pour mapper entre des données au format hiérarchique et tabulaire, par exemple la copie à partir de MongoDB/REST vers un fichier texte et la copie à partir d’Oracle vers l'API Azure Cosmos DB pour MongoDB. Les propriétés suivantes sont prises en charge dans la section `translator` de l’activité de copie :

| Propriété            | Description                                                  | Obligatoire |
| :------------------ | :----------------------------------------------------------- | :------- |
| type                | La propriété type du traducteur d’activité de copie doit être définie sur : **TabularTranslator** | Oui      |
| schemaMapping       | Une collection de paires clé-valeur, qui représente la relation de mappage **du côté source au côté récepteur**.<br/>- **Clé :** représente la source. Pour une **source tabulaire**, spécifiez le nom de colonne tel que défini dans la structure du jeu de données ; pour une **source hiérarchique**, spécifiez l’expression de chemin JSON pour chaque champ à extraire et mapper.<br>- **Valeur :** représente le récepteur. Pour un **récepteur tabulaire**, spécifiez le nom de colonne tel que défini dans la structure du jeu de données ; pour un **récepteur hiérarchique**, spécifiez l’expression de chemin JSON pour chaque champ à extraire et mapper. <br>Dans le cas de données hiérarchiques, pour les champs situés sous l’objet racine, le chemin JSON commence par $ racine ; pour ceux qui se trouvent dans le tableau sélectionné par la propriété `collectionReference`, le chemin JSON commence par l’élément de tableau. | Oui      |
| collectionReference | Si vous souhaitez effectuer une itération et extraire des données à partir des objets situés **à l’intérieur d’un champ de tableau** présentant le même modèle et effectuer une conversion par ligne et par objet, spécifiez le chemin JSON de ce tableau afin d’effectuer une application croisée. Cette propriété est prise en charge uniquement quand des données hiérarchiques sont la source. | Non       |

**Exemple : copier à partir de MongoDB vers Oracle :**

Par exemple, si vous avez un document MongoDB avec le contenu suivant :

```json
{
    "id": {
        "$oid": "592e07800000000000000000"
    },
    "number": "01",
    "date": "20170122",
    "orders": [
        {
            "prod": "p1",
            "price": 23
        },
        {
            "prod": "p2",
            "price": 13
        },
        {
            "prod": "p3",
            "price": 231
        }
    ],
    "city": [ { "name": "Seattle" } ]
}
```

Vous souhaitez copier ce fichier dans une table SQL Azure au format suivant, en mettant à plat les données se trouvant dans le tableau *(order_pd and order_price)* et en effectuant une jointure croisée avec les informations racines communes *(numéro, date et ville)*  :

| orderNumber | orderDate | order_pd | order_price | city    |
| ----------- | --------- | -------- | ----------- | ------- |
| 01          | 20170122  | P1       | 23          | Seattle |
| 01          | 20170122  | P2       | 13          | Seattle |
| 01          | 20170122  | P3       | 231         | Seattle |

Configurez la règle de mappage de schéma comme l’exemple JSON d’activité de copie suivant :

```json
{
    "name": "CopyFromMongoDBToOracle",
    "type": "Copy",
    "typeProperties": {
        "source": {
            "type": "MongoDbV2Source"
        },
        "sink": {
            "type": "OracleSink"
        },
        "translator": {
            "type": "TabularTranslator",
            "schemaMapping": {
                "$.number": "orderNumber",
                "$.date": "orderDate",
                "prod": "order_pd",
                "price": "order_price",
                "$.city[0].name": "city"
            },
            "collectionReference":  "$.orders"
        }
    }
}
```

## <a name="next-steps"></a>Étapes suivantes
Voir les autres articles relatifs à l’activité de copie :

- [Vue d’ensemble des activités de copie](copy-activity-overview.md)
---
title: Concepts de projection
titleSuffix: Azure Cognitive Search
description: Enregistrez et mettez en forme vos données enrichies provenant du pipeline d’indexation d’enrichissement par IA dans une base de connaissances pour les utiliser dans des scénarios autres que ceux de recherche en texte intégral.
manager: nitinme
author: vkurpad
ms.author: vikurpad
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: 22db4f95bacd926208ac7edf3306cd136d81b00e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85565191"
---
# <a name="knowledge-store-projections-in-azure-cognitive-search"></a>« Projections » de base de connaissances dans Recherche cognitive Azure

Recherche cognitive Azure permet l’enrichissement de contenu via des compétences cognitives intégrées et personnalisées dans le cadre de l’indexation. Les enrichissements créent de nouvelles informations là où aucune n’existait précédemment : extraction d’informations à partir d’images, détection de sentiments, d’expressions clés et d’entités à partir de texte, pour n’en nommer que quelques-uns. Les enrichissements ajoutent également une structure à du texte non différencié. Tous ces processus produisent des documents qui rendent la recherche en texte intégral plus efficace. Dans de nombreux cas, les documents enrichis sont utiles pour des scénarios autres que la recherche, pour l’exploration de connaissances par exemple.

Les projections, un composant de la [base de connaissances](knowledge-store-concept-intro.md), sont des vues de documents enrichis qui peuvent être enregistrés dans un stockage physique à des fins d’exploration de connaissances. Une projection vous permet de « projeter » vos données dans une forme qui répond à vos besoins, en conservant les relations afin que les outils tels que Power BI puissent lire les données sans effort supplémentaire.

Les projections peuvent être tabulaires, avec des données stockées dans des lignes et des colonnes dans le stockage Table Azure, ou des objets JSON stockés dans le stockage Blob Azure. Vous pouvez définir plusieurs projections de vos données pendant leur enrichissement. Des projections multiples sont utile lorsque vous souhaitez que les mêmes données soient mises en forme différemment pour des cas d’utilisation individuels.

La base de connaissances prend en charge trois types de projections :

+ **Tables** : Pour les données qui sont mieux représentées sous forme de lignes et de colonnes, les projections de table vous permettent de définir une forme schématisée ou une projection dans le stockage Table. Seuls les objets JSON valides peuvent être projetés sous forme de tables, le document enrichi peut contenir des nœuds qui ne sont pas des objets JSON nommés. Par ailleurs, lors de la projection de ces objets, il peut créer un objet JSON valide avec une compétence de modélisateur ou une mise en forme incluse.

+ **Objets** : Lorsque vous avez besoin d’une représentation JSON de vos données et enrichissements, les projections d’objet sont enregistrées comme des objets Blob. Seuls les objets JSON valides peuvent être projetés sous forme d’objets, le document enrichi peut contenir des nœuds qui ne sont pas des objets JSON nommés. Par ailleurs, lors de la projection de ces objets, il peut créer un objet JSON valide avec une compétence de modélisateur ou une mise en forme incluse.

+ **Fichiers** : Quand vous devez enregistrer les images extraites des documents, les projections de fichiers vous permettent d’enregistrer les images normalisées dans un stockage d’objets blob.

Pour voir des projections définies en contexte, consultez [Créer une base de connaissances avec REST](knowledge-store-create-rest.md).

## <a name="projection-groups"></a>Groupes de projections

Dans certains cas, vous devrez projeter vos données enrichies dans différentes formes pour répondre à différents objectifs. La base de connaissances vous permet de définir plusieurs groupes de projections. Les groupes de projection disposent des principales caractéristiques d’exclusivité mutuelle et de parenté suivantes.

### <a name="mutual-exclusivity"></a>Exclusivité mutuelle

Tout le contenu projeté dans un même groupe est indépendant des données projetées dans d’autres groupes de projections.
Cette indépendance implique que les mêmes données peuvent être mises en forme différemment, mais répétées dans chaque groupe de projections.

### <a name="relatedness"></a>Parenté

Les groupes de projection vous permettent désormais de projeter vos documents dans différents types de projections tout en préservant les relations entre ceux-ci. Tout le contenu projeté dans un même groupe de projections conserve les relations entre les données dans les types de projections. À l’intérieur des tableaux, les relations sont basées sur une clé générée, et chaque nœud enfant conserve une référence au nœud parent. Parmi les types (tables, objets et fichiers), les relations sont préservées quand un seul nœud est projeté vers différents types. Par exemple, imaginez un scénario dans lequel vous avez un document contenant des images et du texte. Vous pouvez projeter le texte vers des tables ou des objets, et les images vers des fichiers dont l’URL figure dans une colonne/propriété dans ces tables ou objets.

## <a name="input-shaping"></a>Mise en forme d’entrée

Disposer de vos données dans la bonne forme ou structure est essentiel pour les utiliser efficacement, qu’il s’agisse de tables ou d’objets. La possibilité de mettre en forme ou de structurer vos données selon la méthode d’accès et d’utilisation souhaitées est une fonctionnalité clé présentée comme la compétence de **modélisateur** de l’ensemble de compétences.  

Les projections sont plus simples à définir lorsque vous avez un objet dans l’arborescence d’enrichissement qui correspond au schéma de la projection. La [compétence de modélisateur](cognitive-search-skill-shaper.md) mise à jour vous permet de composer un objet à partir de différents nœuds de l’arborescence d’enrichissement et de les apparenter sous un nouveau nœud. La compétence de **modélisateur** vous permet de définir des types complexes avec des objets imbriqués.

Lorsqu’une nouvelle forme définie contient tous les éléments que vous devez projeter, vous pouvez maintenant utiliser cette forme comme source pour vos projections ou comme entrée d’une autre compétence.

## <a name="projection-slicing"></a>Découpage de la projection

Quand vous définissez un groupe de projections, un nœud de l’arborescence d’enrichissements peut être divisé pour être projeté dans plusieurs tables ou objets associés. Si vous ajoutez une projection dont le chemin source est un nœud enfant d’une projection existante, le nœud enfant est divisé à partir du nœud parent et projeté dans la nouvelle table ou le nouvel objet associés. Cela vous permet de définir un nœud unique dans une compétence de modélisation qui peut être la source de toutes vos projections.

## <a name="table-projections"></a>Projections de table

Leur importation étant plus simple, nous vous recommandons les projections de table pour l’exploration des données avec Power BI. Les projections de table permettent également de modifier la cardinalité entre les relations de table. 

Vous pouvez projeter un document de votre index dans plusieurs tables, en conservant les relations. Lors de la projection dans plusieurs tables, la forme complète est projetée dans chaque table, sauf si un nœud enfant est la source d’une autre table du même groupe.

### <a name="defining-a-table-projection"></a>Définition d’une projection de table

Lorsque vous définissez une projection de la table dans l’élément `knowledgeStore` de votre ensemble de compétences, commencez par mapper un nœud dans l’arborescence d’enrichissement avec la source de table. Ce nœud est généralement la sortie d’une compétence de **modélisateur** que vous avez ajoutée à la liste des compétences pour produire une forme spécifique dont vous avez besoin pour projeter dans des tables. Le nœud que vous choisissez de projeter peut être divisé pour être projeté dans plusieurs tables. La définition de tables est une liste de tables que vous souhaitez projeter.

Chaque table requiert trois propriétés :

+ tableName : Nom de la table dans le Stockage Azure.

+ generatedKeyName : Nom de colonne de la clé qui identifie de façon unique cette ligne.

+ source : Nœud de l’arborescence d’enrichissement, source de vos enrichissements. Ce nœud est généralement la sortie d’un modélisateur, mais peut également être la sortie d’une des compétences.

Voici un exemple des projections de table.

```json
{
    "name": "your-skillset",
    "skills": [
      …your skills
    ],
"cognitiveServices": {
… your cognitive services key info
    },

    "knowledgeStore": {
      "storageConnectionString": "an Azure storage connection string",
      "projections" : [
        {
          "tables": [
            { "tableName": "MainTable", "generatedKeyName": "SomeId", "source": "/document/EnrichedShape" },
            { "tableName": "KeyPhrases", "generatedKeyName": "KeyPhraseId", "source": "/document/EnrichedShape/*/KeyPhrases/*" },
            { "tableName": "Entities", "generatedKeyName": "EntityId", "source": "/document/EnrichedShape/*/Entities/*" }
          ]
        },
        {
          "objects": [ ]
        },
        {
            "files": [ ]
        }
      ]
    }
}
```

Comme illustré dans cet exemple, les expressions et entités clés sont modélisées dans différentes tables et contiennent une référence au parent (MainTable) pour chaque ligne.

## <a name="object-projections"></a>Projections d’objet

Les projections d’objet sont des représentations JSON de l’arborescence d’enrichissement pouvant provenir de n’importe quel nœud. Dans de nombreux cas, la même compétence de **modélisateur** que celle qui crée une projection de table peut être utilisée pour générer une projection d’objet. 

```json
{
    "name": "your-skillset",
    "skills": [
      …your skills
    ],
"cognitiveServices": {
… your cognitive services key info
    },

    "knowledgeStore": {
      "storageConnectionString": "an Azure storage connection string",
      "projections" : [
        {
          "tables": [ ]
        },
        {
          "objects": [
            {
              "storageContainer": "hotelreviews", 
              "source": "/document/hotel"
            }
          ]
        },
        {
            "files": [ ]
        }
      ]
    }
}
```

La génération d’une projection d’objet nécessite quelques attributs spécifiques à un objet :

+ storageContainer : Conteneur de blobs dans lequel les objets seront enregistrés
+ source : Chemin d’accès au nœud de l’arborescence d’enrichissement qui est la racine de la projection

## <a name="file-projection"></a>Projection de fichier

Les projections de fichiers sont similaires aux projections d’objets et agissent uniquement sur la collection `normalized_images`. À l’instar des projections d’objets, les projections de fichiers sont enregistrées dans le conteneur d’objets blob dont le préfixe de dossier est la valeur encodée en base64 de l’ID de document. Les projections de fichiers ne peuvent pas partager le même conteneur que les projections d’objets. Elles doivent être projetées vers un conteneur distinct.

```json
{
    "name": "your-skillset",
    "skills": [
      …your skills
    ],
"cognitiveServices": {
… your cognitive services key info
    },

    "knowledgeStore": {
      "storageConnectionString": "an Azure storage connection string",
      "projections" : [
        {
          "tables": [ ]
        },
        {
          "objects": [ ]
        },
        {
            "files": [
                 {
                  "storageContainer": "ReviewImages",
                  "source": "/document/normalized_images/*"
                }
            ]
        }
      ]
    }
}
```

## <a name="projection-lifecycle"></a>Cycle de vie de projection

Vos projections ont un cycle de vie qui est lié à la source de données dans votre source de données. Lorsque vos données sont mises à jour et réindexées, vos projections sont mises à jour avec les résultats des enrichissements en s’assurant que vos projections sont cohérentes avec les données dans votre source de données. Les projections héritent de la stratégie de suppression que vous avez configurée pour votre index. Les projections ne sont pas supprimées lors de la suppression de l’indexeur ou du service de recherche.

## <a name="using-projections"></a>Utilisation de projections

Après l’exécution de l’indexeur, vous pouvez lire les données projetées dans les conteneurs ou les tables que vous avez spécifiés par le biais de projections.

Pour l’analyse, l’exploration dans Power BI est aussi simple que de définir le stockage Table Azure comme la source de données. Vous pouvez facilement créer un ensemble de visualisations sur vos données en utilisant les relations incluses.

Si vous devez utiliser les données enrichies dans un pipeline de science des données, vous pouvez également [charger les données d’objets Blob dans un dataframe Pandas](../machine-learning/team-data-science-process/explore-data-blob.md).

Enfin, si vous devez exporter vos données de la base de connaissances, Azure Data Factory comprend des connecteurs pour exporter les données et les placer dans la base de données de votre choix. 

## <a name="next-steps"></a>Étapes suivantes

À l’étape suivante, créez votre première base de connaissances à l’aide d’exemples de données et d’instructions.

> [!div class="nextstepaction"]
> [Créer une base de connaissances avec REST](knowledge-store-create-rest.md).

Pour un tutoriel sur les concepts de projections avancées, tels que le découpage, la mise en forme inline et les relations, commencez par [définir des projections dans une base de connaissances](knowledge-store-projections-examples.md).

> [!div class="nextstepaction"]
> [Définir des projections dans une base de connaissances](knowledge-store-projections-examples.md)

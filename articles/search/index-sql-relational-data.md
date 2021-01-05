---
title: Modéliser des données relationnelles SQL pour l’importation et l’indexation
titleSuffix: Azure Cognitive Search
description: Découvrez comment modéliser des données relationnelles, dénormalisées en un jeu de résultats plat, pour l’indexation et la recherche en texte intégral dans la Recherche cognitive Azure.
author: HeidiSteen
manager: nitinme
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 6c70b42e7d0f647a3b2b60d29b5098a791e4975f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88924518"
---
# <a name="how-to-model-relational-sql-data-for-import-and-indexing-in-azure-cognitive-search"></a>Comment modéliser des données relationnelles SQL pour l’importation et l’indexation dans la Recherche cognitive Azure

La Recherche cognitive Azure accepte un ensemble de lignes plat comme entrée dans le [pipeline d’indexation](search-what-is-an-index.md). Si vos données sources proviennent de tables jointes dans une base de données relationnelle SQL Server, lisez cet article pour savoir comment construire le jeu de résultats et comment modéliser une relation parent-enfant dans un index Recherche cognitive Azure.

Pour illustrer nos explications, nous faisons référence à une base de données Hotels fictive dont le contenu est tiré de ces [données de démonstration](https://github.com/Azure-Samples/azure-search-sample-data/tree/master/hotels). Supposons que la base de données se compose d’une table Hotels$ listant 50 hôtels et d’une table Rooms$ contenant les détails de 750 chambres au total (comme le type, l’équipement et le prix de chacune d’elles). Il y a une relation un-à-plusieurs entre les tables. Dans notre approche, nous utilisons une vue pour fournir la requête qui retourne 50 lignes (soit une ligne par hôtel), avec les détails des chambres associées incorporés dans chaque ligne.

   ![Tables et vue dans la base de données Hotels](media/index-sql-relational-data/hotels-database-tables-view.png "Tables et vue dans la base de données Hotels")


## <a name="the-problem-of-denormalized-data"></a>Le problème des données dénormalisées

L’un des problèmes que pose l’utilisation de relations un-à-plusieurs vient des requêtes standard créées sur des tables jointes qui retournent des données dénormalisées, un format qui n’est pas approprié dans un scénario Recherche cognitive Azure. Prenons l’exemple suivant qui joint les hôtels et les chambres.

```sql
SELECT * FROM Hotels$
INNER JOIN Rooms$
ON Rooms$.HotelID = Hotels$.HotelID
```
Les résultats de cette requête retournent tous les champs de la table Hotels, suivis de tous les champs de la table Rooms, avec des informations préliminaires sur l’hôtel qui sont répétées pour chaque chambre.

   ![Données dénormalisées, données d’hôtel redondantes lors de l’ajout de champs de chambre](media/index-sql-relational-data/denormalize-data-query.png "Données dénormalisées, données d’hôtel redondantes lors de l’ajout de champs de chambre")


Cette requête semble correcte de prime abord (elle fournit bien toutes les données dans un ensemble de lignes plat), mais en réalité, elle ne donne pas la structure de document appropriée pour l’expérience de recherche attendue. Durant l’indexation, la Recherche cognitive Azure crée un document de recherche pour chaque ligne ingérée. Si vos documents de recherche présentaient une structure similaire aux résultats ci-dessus, vous verriez des doublons, comme ici où nous avons sept documents distincts pour le seul hôtel Twin Dome. Une requête de recherche sur les hôtels en Floride renverrait sept résultats rien que pour l’hôtel Twin Dome, et les autres hôtels pertinents seraient affichés beaucoup plus loin dans les résultats de la recherche.

Pour obtenir l’expérience attendue d’un seul document par hôtel, vous devez fournir un ensemble de lignes à la granularité appropriée, mais avec des informations complètes. Heureusement, vous pouvez y parvenir facilement en adoptant les techniques décrites dans cet article.

## <a name="define-a-query-that-returns-embedded-json"></a>Définir une requête qui retourne une collection JSON incorporée

Pour offrir l’expérience de recherche attendue, votre jeu de données doit contenir une ligne pour chaque document de recherche dans la Recherche cognitive Azure. Dans notre exemple, nous voulons avoir une ligne distincte par hôtel, mais nous souhaitons également que les utilisateurs puissent faire des recherches sur d’autres champs descriptifs des chambres pour trouver les renseignements dont ils ont besoin, comme le prix par nuit, le nombre et la taille des lits ou une vue sur mer, etc.

La solution consiste à capturer les détails des chambres avec une requête JSON imbriquée, puis d’insérer la structure JSON dans un champ d’une vue, comme cela est montré à la deuxième étape. 

1. Supposons que vous avez deux tables Hotels$ et Rooms$ qui sont jointes sur le champ HotelID. Ces tables contiennent respectivement les détails de 50 hôtels et de leurs 750 chambres.

    ```sql
    CREATE TABLE [dbo].[Hotels$](
      [HotelID] [nchar](10) NOT NULL,
      [HotelName] [nvarchar](255) NULL,
      [Description] [nvarchar](max) NULL,
      [Description_fr] [nvarchar](max) NULL,
      [Category] [nvarchar](255) NULL,
      [Tags] [nvarchar](255) NULL,
      [ParkingIncluded] [float] NULL,
      [SmokingAllowed] [float] NULL,
      [LastRenovationDate] [smalldatetime] NULL,
      [Rating] [float] NULL,
      [StreetAddress] [nvarchar](255) NULL,
      [City] [nvarchar](255) NULL,
      [State] [nvarchar](255) NULL,
      [ZipCode] [nvarchar](255) NULL,
      [GeoCoordinates] [nvarchar](255) NULL
    ) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
    GO

    CREATE TABLE [dbo].[Rooms$](
      [HotelID] [nchar](10) NULL,
      [Description] [nvarchar](255) NULL,
      [Description_fr] [nvarchar](255) NULL,
      [Type] [nvarchar](255) NULL,
      [BaseRate] [float] NULL,
      [BedOptions] [nvarchar](255) NULL,
      [SleepsCount] [float] NULL,
      [SmokingAllowed] [float] NULL,
      [Tags] [nvarchar](255) NULL
    ) ON [PRIMARY]
    GO
    ```

2. Créez une vue composée de tous les champs de la table parente (`SELECT * from dbo.Hotels$`) et d’un nouveau champ *Rooms* qui contient la sortie d’une requête imbriquée. La clause **FOR JSON AUTO** spécifiée dans `SELECT * from dbo.Rooms$` structure la sortie au format JSON. 

     ```sql
   CREATE VIEW [dbo].[HotelRooms]
   AS
   SELECT *, (SELECT *
              FROM dbo.Rooms$
              WHERE dbo.Rooms$.HotelID = dbo.Hotels$.HotelID FOR JSON AUTO) AS Rooms
   FROM dbo.Hotels$
   GO
   ```

   La capture d’écran ci-dessous montre la vue obtenue, avec le champ *Rooms* nvarchar tout en bas. Le champ *Rooms* existe uniquement dans la vue HotelRooms.

   ![Vue HotelRooms](media/index-sql-relational-data/hotelsrooms-view.png "Vue HoteRooms")

1. Exécutez `SELECT * FROM dbo.HotelRooms` pour récupérer l’ensemble de lignes. Cette requête retourne 50 lignes (soit une par hôtel) ainsi que les détails des chambres associées sous forme de collection JSON. 

   ![Ensemble de lignes à partir de la vue HotelRooms](media/index-sql-relational-data/hotelrooms-rowset.png "Ensemble de lignes à partir de la vue HotelRooms")

Cet ensemble de lignes est maintenant prêt pour l’importation dans la Recherche cognitive Azure.

> [!NOTE]
> Cette approche suppose que la collection JSON incorporée ne dépasse pas les [limites de tailles de colonne maximales de SQL Server](/sql/sql-server/maximum-capacity-specifications-for-sql-server). 

 ## <a name="use-a-complex-collection-for-the-many-side-of-a-one-to-many-relationship"></a>Utiliser une collection complexe pour le côté « plusieurs » d’une relation un-à-plusieurs

Dans la Recherche cognitive Azure, créez un schéma d’index qui modélise la relation un-à-plusieurs à l’aide d’une requête JSON imbriquée. Le jeu de résultats que vous avez créé dans la section précédente correspond généralement au schéma d’index fourni ci-dessous (nous avons toutefois enlevé certains champs par souci de concision).

L’exemple suivant est similaire à l’exemple donné dans [Modélisation de types de données complexes](search-howto-complex-data-types.md#creating-complex-fields). La structure *Rooms*, qui a été le sujet principal de cet article, se trouve dans la collection fields d’un index nommé *hotels*. Cet exemple montre également un type complexe pour *Address*, qui diffère de *Rooms* dans la mesure où il est composé d’un ensemble fixe d’éléments, par opposition au nombre arbitraire d’éléments multiples autorisés dans une collection.

```json
{
  "name": "hotels",
  "fields": [
    { "name": "HotelId", "type": "Edm.String", "key": true, "filterable": true },
    { "name": "HotelName", "type": "Edm.String", "searchable": true, "filterable": false },
    { "name": "Description", "type": "Edm.String", "searchable": true, "analyzer": "en.lucene" },
    { "name": "Description_fr", "type": "Edm.String", "searchable": true, "analyzer": "fr.lucene" },
    { "name": "Category", "type": "Edm.String", "searchable": true, "filterable": false },
    { "name": "ParkingIncluded", "type": "Edm.Boolean", "filterable": true, "facetable": true },
    { "name": "Address", "type": "Edm.ComplexType",
      "fields": [
        { "name": "StreetAddress", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false, "searchable": true },
        { "name": "City", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true },
        { "name": "StateProvince", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true }
      ]
    },
    { "name": "Rooms", "type": "Collection(Edm.ComplexType)",
      "fields": [
        { "name": "Description", "type": "Edm.String", "searchable": true, "analyzer": "en.lucene" },
        { "name": "Description_fr", "type": "Edm.String", "searchable": true, "analyzer": "fr.lucene" },
        { "name": "Type", "type": "Edm.String", "searchable": true },
        { "name": "BaseRate", "type": "Edm.Double", "filterable": true, "facetable": true },
        { "name": "BedOptions", "type": "Edm.String", "searchable": true, "filterable": true, "facetable": true },
        { "name": "SleepsCount", "type": "Edm.Int32", "filterable": true, "facetable": true },
        { "name": "SmokingAllowed", "type": "Edm.Boolean", "filterable": true, "facetable": true },
        { "name": "Tags", "type": "Edm.Collection", "searchable": true }
      ]
    }
  ]
}
```

Avec le jeu de résultats précédent et le schéma d’index ci-dessus, vous avez tous les composants requis pour réussir l’indexation. Le jeu de données aplati répond aux exigences d’indexation, tout en préservant les informations détaillées nécessaires. Dans l’index de la Recherche cognitive Azure, les résultats de la recherche sont catégorisés par hôtel, tout en conservant le contexte de chacune des chambres et leurs caractéristiques.

## <a name="next-steps"></a>Étapes suivantes

Avec votre propre jeu de données, vous pouvez utiliser l’[Assistant Importer des données](search-import-data-portal.md) pour créer et charger l’index. L’Assistant détecte la collection JSON incorporée, telle que celle contenue dans *Rooms*, et déduit un schéma d’index qui contient une collection de types complexe. 

  ![Index déduit par l’Assistant Importation de données](media/index-sql-relational-data/search-index-rooms-complex-collection.png "Index déduit par l’Assistant Importation de données")

Essayez le guide de démarrage rapide suivant pour découvrir les étapes de base de l’Assistant Importer des données.

> [!div class="nextstepaction"]
> [Démarrage rapide : Créer un index de recherche à l’aide du portail Azure](search-get-started-portal.md)
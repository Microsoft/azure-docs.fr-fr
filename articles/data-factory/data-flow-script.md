---
title: Script du flux de données de mappage
description: Vue d’ensemble du langage code-behind du script de flux de données Azure Data Factory
author: kromerm
ms.author: nimoolen
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 12/03/2020
ms.openlocfilehash: 69b2713e928707479945df0bb242ac2fbc001c32
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96600657"
---
# <a name="data-flow-script-dfs"></a>Script de flux de données (DFS)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Le script de flux de données (DFS) est constitué des métadonnées sous-jacentes, similaires à un langage de codage, qui sont utilisées pour exécuter les transformations incluses dans un flux de données de mappage. Chaque transformation est représentée par une série de propriétés qui fournissent les informations nécessaires à l’exécution correcte du travail. Le script est visible et modifiable à partir de ADF en cliquant sur le bouton « script » sur le ruban supérieur de l’interface utilisateur du navigateur.

![Bouton script](media/data-flow/scriptbutton.png "Bouton Script")

Par exemple, dans une transformation source `allowSchemaDrift: true,` indique au service d’inclure toutes les colonnes du jeu de données source dans le flux de données, même s’ils ne sont pas inclus dans la projection de schéma.

## <a name="use-cases"></a>Cas d'utilisation
Le DFS est généré automatiquement par l’interface utilisateur. Vous pouvez cliquer sur le bouton Script pour afficher et personnaliser le script. Vous pouvez également générer des scripts en dehors de l’interface utilisateur ADF, puis les transmettre à la cmdlet PowerShell. Lors du débogage de flux de données complexes, il peut s’avérer plus facile d’analyser le code-behind de script au lieu d’analyser la représentation graphique d’interface utilisateur de vos flux.

Voici quelques exemples de cas d’usage :
- Générer par programme de nombreux flux de données qui sont assez similaires, par exemple des flux de données d’horodatage.
- Les expressions complexes qui sont difficiles à gérer dans l’interface utilisateur ou qui entraînent des problèmes de validation.
- Déboguer et mieux comprendre les différentes erreurs renvoyées lors de l’exécution.

Quand vous générez un script de flux de données à utiliser avec PowerShell ou une API, vous devez réduire le texte mis en forme pour ne former qu’une seule ligne. Vous pouvez conserver les tabulations et les nouvelles lignes comme des caractères d’échappement. Toutefois, le texte doit être mis en forme pour être intégré à une propriété JSON. Un bouton de l’interface utilisateur de l’éditeur de script situé en bas permet de mettre en forme le script sur une seule ligne.

![Bouton Copier](media/data-flow/copybutton.png "bouton Copier")

## <a name="how-to-add-transforms"></a>Comment ajouter des transformations
L’ajout de transformations requiert trois étapes de base : l’ajout des données de transformation principales, la redirection du flux d’entrée, puis celle du flux de sortie. Un exemple sera plus explicite.
Supposons que nous commençons avec une source simple pour recevoir le flux de données comme suit :

```
source(output(
        movieId as string,
        title as string,
        genres as string
    ),
    allowSchemaDrift: true,
    validateSchema: false) ~> source1
source1 sink(allowSchemaDrift: true,
    validateSchema: false) ~> sink1
```

Si nous décidons d’ajouter une transformation de dérivation, nous devons tout d’abord créer le texte de transformation principal comportant une expression simple permettant d’ajouter une nouvelle colonne en majuscules appelée `upperCaseTitle` :
```
derive(upperCaseTitle = upper(title)) ~> deriveTransformationName
```

Ensuite, nous prenons le DFS existant et ajoutons la transformation :
```
source(output(
        movieId as string,
        title as string,
        genres as string
    ),
    allowSchemaDrift: true,
    validateSchema: false) ~> source1
derive(upperCaseTitle = upper(title)) ~> deriveTransformationName
source1 sink(allowSchemaDrift: true,
    validateSchema: false) ~> sink1
```

Nous allons à présent rediriger le flux entrant en identifiant la transformation à la suite de laquelle nous voulons intégrer la nouvelle transformation (dans ce cas, `source1`) et en copiant le nom du flux dans la nouvelle transformation :
```
source(output(
        movieId as string,
        title as string,
        genres as string
    ),
    allowSchemaDrift: true,
    validateSchema: false) ~> source1
source1 derive(upperCaseTitle = upper(title)) ~> deriveTransformationName
source1 sink(allowSchemaDrift: true,
    validateSchema: false) ~> sink1
```

Enfin, nous identifions la transformation à intégrer à la suite de cette nouvelle transformation et remplaçons son flux d’entrée (dans ce cas, `sink1`) par le nom du flux de sortie de notre nouvelle transformation :
```
source(output(
        movieId as string,
        title as string,
        genres as string
    ),
    allowSchemaDrift: true,
    validateSchema: false) ~> source1
source1 derive(upperCaseTitle = upper(title)) ~> deriveTransformationName
deriveTransformationName sink(allowSchemaDrift: true,
    validateSchema: false) ~> sink1
```

## <a name="dfs-fundamentals"></a>Notions de base du DFS
Le DFS est constitué d’une série de transformations connectées, y compris de sources, de récepteurs et d’autres éléments permettant d’ajouter de nouvelles colonnes, de filtrer des données, de joindre des données, et plus encore. En règle générale, le script commence parc une ou plusieurs sources suivies par de nombreuses transformations et se termine par un ou plusieurs récepteurs.

Les sources ont toutes la même construction de base :
```
source(
  source properties
) ~> source_name
```

Par exemple, une source simple dotée de trois colonnes (movieId, title, genres) serait la suivante :
```
source(output(
        movieId as string,
        title as string,
        genres as string
    ),
    allowSchemaDrift: true,
    validateSchema: false) ~> source1
```

Toutes les transformations autres que les sources ont la même construction de base :
```
name_of_incoming_stream transformation_type(
  properties
) ~> new_stream_name
```

Par exemple, une transformation de dérivation simple qui prend une colonne (titre) et la remplace par une version en majuscules serait la suivante :
```
source1 derive(
  title = upper(title)
) ~> derive1
```

Et un récepteur sans schéma ressemblerait simplement à ce qui suit :
```
derive1 sink(allowSchemaDrift: true,
    validateSchema: false) ~> sink1
```

## <a name="script-snippets"></a>Extraits de script

Les extraits de script correspondent à du code partageable de script de flux de données que vous pouvez utiliser à des fins de partage entre les flux de données. La vidéo ci-dessous explique comment utiliser les extraits de script et le script de flux de données pour copier et coller des parties du script derrière vos graphes de flux données :

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4tA9b]


### <a name="aggregated-summary-stats"></a>Résumé agrégé des statistiques
Ajoutez une transformation d’agrégation à votre flux de données nommé « SummaryStats », puis collez le code ci-dessous pour la fonction d’agrégation dans votre script, en remplaçant le SummaryStats existant. Cela permet de fournir un modèle générique pour le résumé des statistiques du profil de données.

```
aggregate(each(match(true()), $$+'_NotNull' = countIf(!isNull($$)), $$ + '_Null' = countIf(isNull($$))),
        each(match(type=='double'||type=='integer'||type=='short'||type=='decimal'), $$+'_stddev' = round(stddev($$),2), $$ + '_min' = min ($$), $$ + '_max' = max($$), $$ + '_average' = round(avg($$),2), $$ + '_variance' = round(variance($$),2)),
        each(match(type=='string'), $$+'_maxLength' = max(length($$)))) ~> SummaryStats
```
Vous pouvez également utiliser l’exemple ci-dessous pour compter le nombre de lignes uniques et le nombre de lignes distinctes dans vos données. L’exemple ci-dessous peut être collé dans un flux de données avec la transformation d’agrégation appelée ValueDistAgg. Cet exemple utilise une colonne appelée « title ». Veillez à remplacer « title » par la colonne de chaîne de vos données que vous souhaitez utiliser pour connaître le nombre de valeurs.

```
aggregate(groupBy(title),
    countunique = count()) ~> ValueDistAgg
ValueDistAgg aggregate(numofunique = countIf(countunique==1),
        numofdistinct = countDistinct(title)) ~> UniqDist
```

### <a name="include-all-columns-in-an-aggregate"></a>Inclure toutes les colonnes dans un agrégat
Il s’agit d’un modèle d’agrégation générique qui montre comment conserver les colonnes restantes dans vos métadonnées de sortie lorsque vous générez des agrégats. Dans ce cas, nous utilisons la fonction ```first()``` pour choisir la première valeur de chaque colonne dont le nom n’est pas « movie ». Pour ce faire, créez une transformation d’agrégation appelée DistinctRows, puis collez-la dans votre script au-dessus du script d’agrégation DistinctRows existant.

```
aggregate(groupBy(movie),
    each(match(name!='movie'), $$ = first($$))) ~> DistinctRows
```

### <a name="create-row-hash-fingerprint"></a>Créer une empreinte digitale de hachage de ligne 
Utilisez ce code dans votre script de flux de données pour créer une colonne dérivée nommée ```DWhash``` qui produit un hachage ```sha1``` de trois colonnes.

```
derive(DWhash = sha1(Name,ProductNumber,Color)) ~> DWHash
```

Vous pouvez également utiliser le script ci-dessous pour générer un hachage de ligne à l’aide de toutes les colonnes présentes dans votre flux, sans avoir à nommer chacune d’elles :

```
derive(DWhash = sha1(columns())) ~> DWHash
```

### <a name="string_agg-equivalent"></a>Équivalent String_agg
Ce code agit comme la fonction ```string_agg()``` T-SQL et regroupe les valeurs de chaîne dans un tableau. Vous pouvez ensuite caster ce tableau en chaîne à utiliser avec des destinations SQL.

```
source1 aggregate(groupBy(year),
    string_agg = collect(title)) ~> Aggregate1
Aggregate1 derive(string_agg = toString(string_agg)) ~> StringAgg
```

### <a name="count-number-of-updates-upserts-inserts-deletes"></a>Nombre de mises à jour, upserts, insertions, suppressions
Lorsque vous utilisez une transformation de modification de ligne, vous pouvez compter le nombre de mises à jour, d’upserts, d’insertions, de suppressions résultant de vos stratégies de modification de ligne. Ajoutez une transformation d’agrégation après votre modification de ligne et collez ce script de Data Flow dans la définition de l’agrégat pour ces nombres.

```
aggregate(updates = countIf(isUpdate(), 1),
        inserts = countIf(isInsert(), 1),
        upserts = countIf(isUpsert(), 1),
        deletes = countIf(isDelete(),1)) ~> RowCount
```

### <a name="distinct-row-using-all-columns"></a>Ligne distincte utilisant toutes les colonnes
Cet extrait de code ajoute une nouvelle transformation d’agrégation à votre flux de données, qui prend toutes les colonnes entrantes, génère un hachage utilisé pour le regroupement afin d’éliminer les doublons, puis fournit la première occurrence de chaque doublon comme sortie. Vous n’avez pas besoin de nommer explicitement les colonnes, car elles seront générées automatiquement à partir de votre flux de données entrant.

```
aggregate(groupBy(mycols = sha2(256,columns())),
    each(match(true()), $$ = first($$))) ~> DistinctRows
```

### <a name="check-for-nulls-in-all-columns"></a>Rechercher les valeurs NULL dans toutes les colonnes
Il s’agit d’un extrait de code que vous pouvez coller dans votre flux de données pour rechercher de manière générique les valeurs NULL dans toutes vos colonnes. Cette technique s’appuie sur la dérive de schéma pour parcourir toutes les colonnes de toutes les lignes et utilise une opération de fractionnement conditionnel pour séparer les lignes contenant des valeurs NULL des lignes sans valeurs NULL. 

```
split(contains(array(columns()),isNull(#item)),
    disjoint: false) ~> LookForNULLs@(hasNULLs, noNULLs)
```

### <a name="automap-schema-drift-with-a-select"></a>Mapper automatiquement la dérive de schéma à l’aide d’une transformation de sélection
Lorsque vous devez charger un schéma de base de données existant à partir d’un ensemble inconnu ou dynamique de colonnes entrantes, vous devez mapper les colonnes de droite dans la transformation du récepteur. Cela est nécessaire uniquement lorsque vous chargez une table existante. Ajoutez cet extrait de code avant votre récepteur pour créer une transformation de sélection qui mappe automatiquement vos colonnes. Laissez le mappage de votre récepteur sur Mappage automatique.

```
select(mapColumn(
        each(match(true()))
    ),
    skipDuplicateMapInputs: true,
    skipDuplicateMapOutputs: true) ~> automap
```

### <a name="persist-column-data-types"></a>Rendre persistants les types de données de la colonne
Ajoutez ce script dans une définition de colonne dérivée pour stocker les noms de colonne et les types de données de votre flux de données dans un magasin persistant à l’aide d’un récepteur.

```
derive(each(match(type=='string'), $$ = 'string'),
    each(match(type=='integer'), $$ = 'integer'),
    each(match(type=='short'), $$ = 'short'),
    each(match(type=='complex'), $$ = 'complex'),
    each(match(type=='array'), $$ = 'array'),
    each(match(type=='float'), $$ = 'float'),
    each(match(type=='date'), $$ = 'date'),
    each(match(type=='timestamp'), $$ = 'timestamp'),
    each(match(type=='boolean'), $$ = 'boolean'),
    each(match(type=='double'), $$ = 'double')) ~> DerivedColumn1
```

## <a name="next-steps"></a>Étapes suivantes

Découvrez les flux de données en commençant par l’[article de présentation des flux de données](concepts-data-flow-overview.md)

---
title: Transformation de recherche dans le flux de données de mappage
description: Référencez des données à partir d’une autre source à l’aide de la transformation de recherche dans le flux de données de mappage.
author: kromerm
ms.reviewer: daperlov
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 10/30/2020
ms.openlocfilehash: 7ed1d9db09357b0702188c01a802600ff6350aff
ms.sourcegitcommit: 4b76c284eb3d2b81b103430371a10abb912a83f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2020
ms.locfileid: "93147264"
---
# <a name="lookup-transformation-in-mapping-data-flow"></a>Transformation de recherche dans le flux de données de mappage

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Utilisez la transformation de recherche pour référencer des données provenant d’une autre source dans un flux de données. La transformation de recherche ajoute des colonnes de données mises en correspondance à vos données sources.

Une transformation de recherche est similaire à une jointure externe gauche. Toutes les lignes du flux principal existent dans le flux de sortie avec des colonnes supplémentaires venant du flux de recherche.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4xsVT]

## <a name="configuration"></a>Configuration

![Capture d’écran affichant l’onglet Paramètres de recherche avec les étiquettes décrites dans le texte suivant.](media/data-flow/lookup1.png "Recherche")

**Flux principal :** Le flux de données entrant. Ce flux est équivalent au côté gauche d’une jointure.

**Flux de recherche :** Les données ajoutées au flux de données principal. Les données ajoutées sont déterminées par les conditions de recherche. Ce flux est équivalent au côté droit d’une jointure.

**Correspond à plusieurs lignes :** Si cette option est activée, une ligne avec plusieurs correspondances dans le flux principal retourne plusieurs lignes. Dans le cas contraire, une seule ligne est retournée en fonction de la condition ’Match on'.

**Correspond à :** Visible uniquement si l’option « Correspond à plusieurs lignes » n’est pas sélectionnée. Choisissez s’il faut faire correspondre sur n’importe quelle ligne, sur la première correspondance ou sur la dernière correspondance. Faire correspondre sur n’importe quelle ligne est recommandé, car c’est l’option qui s’exécute le plus rapidement. Si la première ligne ou la dernière ligne est sélectionnée, vous devez spécifier des conditions de tri.

**Conditions de recherche :** Choisissez les colonnes à faire correspondre. Si la condition d’égalité est remplie, les lignes sont considérées comme une correspondance. Pointez et sélectionnez « Colonne calculée » pour extraire une valeur à l’aide du [Langage d’expression des flux de données](data-flow-expression-functions.md).

La transformation de recherche prend uniquement en charge les correspondances d’égalité. Pour personnaliser l’expression de recherche afin d’inclure d’autres opérateurs tels que supérieur à, il est recommandé d’utiliser une [jointure croisée dans la transformation de jointure](data-flow-join.md#custom-cross-join). Avec une jointure croisée, vous éviterez ainsi les erreurs de produit cartésien lors de l’exécution.

Toutes les colonnes des deux flux sont incluses dans les données de sortie. Pour supprimer les colonnes en double ou indésirables, ajoutez une [transformation de sélection](data-flow-select.md) après votre transformation de recherche. Les colonnes peuvent également être supprimées ou renommées dans une transformation de récepteur.

### <a name="non-equi-joins"></a>Jointures différentes

Pour utiliser un opérateur conditionnel tel que « différent de » (!=) ou « supérieur à » (>) dans vos conditions de recherche, modifiez la liste déroulante des opérateurs entre les deux colonnes. Des jointures différentes nécessitent qu’au moins l’un des deux flux soit diffusés en utilisant la diffusion **fixe** dans l’onglet **Optimiser**.

![Non-equi lookup](media/data-flow/non-equi-lookup.png "Recherche de différence")

## <a name="analyzing-matched-rows"></a>Analyse des lignes correspondantes

Après la transformation de recherche, la fonction `isMatch()` peut être utilisée pour déterminer si la recherche correspond à des lignes individuelles.

![Modèle de recherche](media/data-flow/lookup111.png "Modèle de recherche")

Un exemple de ce modèle consiste à utiliser la transformation de fractionnement conditionnel pour fractionner sur la fonction `isMatch()`. Dans l’exemple ci-dessus, les lignes correspondantes passent par le flux principal, et les lignes sans correspondance transitent par le flux ```NoMatch```.

## <a name="testing-lookup-conditions"></a>Test des conditions de recherche

Lorsque vous testez la transformation de recherche avec l’aperçu des données en mode débogage, utilisez un jeu de données connues peu volumineux. Lors de l’échantillonnage de lignes à partir d’un jeu de données volumineux, vous ne pouvez pas prédire quelles lignes et quelles clés seront lues dans le cadre du test. Le résultat n’est pas déterministe, ce qui signifie que vos conditions de jointure peuvent ne renvoyer aucun résultat.

## <a name="broadcast-optimization"></a>Optimisation de la diffusion

![Jonction de diffusion](media/data-flow/broadcast.png "Jonction de diffusion")

Dans les transformations de jointure, de recherche et d’existence, si l’un des flux de données ou les deux tiennent dans la mémoire de nœud Worker, vous pouvez optimiser les performances en activant la **diffusion**. Par défaut, le moteur Spark détermine automatiquement s’il faut diffuser un côté. Pour choisir manuellement le côté à diffuser, sélectionnez **Fixe**.

Il n’est pas recommandé de désactiver la diffusion à l’aide de l’option **Désactivé** à moins que vos jointures ne rencontrent des erreurs de délai d’attente.

## <a name="cached-lookup"></a>Recherche mise en cache

Si vous effectuez plusieurs petites recherches sur la même source, un récepteur et une recherche en cache sont peut-être préférables à la transformation de recherche. La recherche d’une valeur maximale dans un magasin de données et la mise en correspondance des codes d’erreur avec une base de données de messages d’erreur sont des situations courantes dans lesquelles un récepteur de cache peut être préférable. Pour plus d’informations, apprenez-en plus sur les [récepteurs de cache](data-flow-sink.md#cache-sink) et les [recherches mises en cache](concepts-data-flow-expression-builder.md#cached-lookup).

## <a name="data-flow-script"></a>Script de flux de données

### <a name="syntax"></a>Syntaxe

```
<leftStream>, <rightStream>
    lookup(
        <lookupConditionExpression>,
        multiple: { true | false },
        pickup: { 'first' | 'last' | 'any' },  ## Only required if false is selected for multiple
        { desc | asc }( <sortColumn>, { true | false }), ## Only required if 'first' or 'last' is selected. true/false determines whether to put nulls first
        broadcast: { 'auto' | 'left' | 'right' | 'both' | 'off' }
    ) ~> <lookupTransformationName>
```
### <a name="example"></a>Exemple

![Capture d’écran représentant l’onglet Paramètres de recherche pour le code suivant.](media/data-flow/lookup-dsl-example.png "Recherche")

Le script de transmission de données pour la configuration de recherche ci-dessus se trouve dans l’extrait de code suivant.

```
SQLProducts, DimProd lookup(ProductID == ProductKey,
    multiple: false,
    pickup: 'first',
    asc(ProductKey, true),
    broadcast: 'auto')~> LookupKeys
```
## 
Étapes suivantes

* Les transformations [join](data-flow-join.md) et [exists](data-flow-exists.md) prennent toutes les deux plusieurs entrées de flux
* Utiliser une [transformation de fractionnement conditionnel](data-flow-conditional-split.md) avec ```isMatch()``` pour fractionner des lignes sur des valeurs correspondantes et non correspondantes

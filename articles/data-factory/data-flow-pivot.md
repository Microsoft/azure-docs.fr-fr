---
title: Transformation de tableau croisé dynamique dans le flux de données de mappage
description: Ajouter un tableau croisé dynamique à l’aide de la transformation d’ajout de tableau croisé dynamique de flux de données de mappage pour Azure Data Factory
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 07/17/2020
ms.openlocfilehash: e098182c000cbe05df533434a41c55b797ef876f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87086657"
---
# <a name="pivot-transformation-in-mapping-data-flow"></a>Transformation de tableau croisé dynamique dans le flux de données de mappage


[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Utilisez la transformation de tableau croisé dynamique pour créer plusieurs colonnes à partir des valeurs de lignes uniques d’une seule colonne. Un tableau croisé dynamique est une transformation d’agrégation dans laquelle vous sélectionnez l’option Regrouper par colonnes et générez des colonnes de tableau croisé dynamique à l’aide de [fonctions d’agrégation](data-flow-expression-functions.md#aggregate-functions).

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4C4YN]

## <a name="configuration"></a>Configuration

La transformation de tableau croisé dynamique requiert trois entrées différentes : regrouper par colonnes, la clé de tableau croisé dynamique et comment générer les colonnes ayant fait l’objet d’un ajout de tableau croisé dynamique

### <a name="group-by"></a>Regrouper par

![Options de regroupement](media/data-flow/pivot2.png "Options de regroupement")

Sélectionnez les colonnes sur lesquelles agréger les colonnes ayant fait l’objet d’un ajout de tableau croisé dynamique. Les données de sortie regroupent en une seule ligne toutes les lignes ayant les mêmes valeurs Regrouper par. L’agrégation effectuée dans la colonne ayant fait l’objet d’un ajout de tableau croisé dynamique est effectuée sur chaque groupe.

Cette section est facultative. Si aucune colonne Regrouper par n’est sélectionnée, l’ensemble du flux de données est agrégé et une seule ligne est sortie.

### <a name="pivot-key"></a>Clé de tableau croisé dynamique

![Clé de tableau croisé dynamique](media/data-flow/pivot3.png "Clé de tableau croisé dynamique")

La clé de tableau croisé dynamique est la colonne dont les valeurs de ligne ont fait l’objet d’un ajout de tableau croisé dynamique dans de nouvelles colonnes. Par défaut, la transformation de tableau croisé dynamique crée une colonne pour chaque valeur de ligne unique.

Dans la section intitulée **Valeur**, vous pouvez entrer des valeurs de ligne spécifiques devant faire l’objet d’un ajout de tableau croisé dynamique. Seules les valeurs de ligne entrées dans cette section font l’objet d’un ajout de tableau croisé dynamique. L’activation de la **valeur Null** crée une colonne ayant fait l’objet d’un ajout de tableau croisé dynamique pour les valeurs Null dans la colonne.

### <a name="pivoted-columns"></a>Colonnes ayant fait l’objet d’un ajout de tableau croisé dynamique

![Colonnes ayant fait l’objet d’un ajout de tableau croisé dynamique](media/data-flow/pivot4.png "Colonnes ayant fait l’objet d’un ajout de tableau croisé dynamique")

Pour chaque valeur de clé de tableau croisé dynamique unique qui devient une colonne, générez une valeur de ligne agrégée pour chaque groupe. Vous pouvez créer plusieurs colonnes par clé de tableau croisé dynamique. Chaque colonne de tableau croisé dynamique doit contenir au moins une [fonction d’agrégation](data-flow-expression-functions.md#aggregate-functions).

**Modèle de nom de colonne :** Sélectionnez le mode de mise en forme du nom de colonne de chaque colonne de tableau croisé dynamique. Le nom de colonne en sortie est une combinaison de la valeur de clé de tableau croisé dynamique, du préfixe de colonne et du préfixe facultatif, du suffixe, des caractères du milieu. 

**Disposition de colonne :** Si vous générez plusieurs colonnes de tableau croisé dynamique par clé de tableau croisé dynamique, choisissez la façon dont vous souhaitez trier les colonnes. 

**Préfixe de colonne :** Si vous générez plusieurs colonnes de tableau croisé dynamique par clé de tableau croisé dynamique, entrez un préfixe de colonne pour chaque colonne. Ce paramètre est facultatif si vous n’avez qu’une colonne ayant fait l’objet d’un ajout de tableau croisé dynamique.

## <a name="help-graphic"></a>Graphe d’aide

Le graphique d’aide ci-dessous montre comment les différents composants de tableau croisé dynamique interagissent les uns avec les autres

![Graphique d’aide sur le tableau croisé dynamique](media/data-flow/pivot5.png "Graphique d’aide sur le tableau croisé dynamique")

## <a name="pivot-metadata"></a>Métadonnées de tableau croisé dynamique

Si aucune valeur n’est spécifiée dans la configuration de la clé de tableau croisé dynamique, les colonnes ayant fait l’objet d’un ajout de tableau croisé dynamique sont générées dynamiquement au moment de l’exécution. Le nombre de colonnes ayant fait l’objet d’un ajout de tableau croisé dynamique est égal au nombre de valeurs de clés de tableau croisé dynamique, multiplié par le nombre de colonnes du tableau croisé dynamique. Comme il peut s’agir d’un nombre variable, l’expérience utilisateur n’affiche pas les métadonnées de colonne dans l’onglet **Inspecter** et il n’y a pas de propagation de colonne. Pour transformer ces colonnes, utilisez les fonctionnalités du [modèle de colonne](concepts-data-flow-column-pattern.md) du flux de données de mappage. 

Si des valeurs spécifiques de clés de tableau croisé dynamique sont définies, les colonnes ayant fait l’objet d’un ajout de tableau croisé dynamique s’affichent dans les métadonnées. Les noms de colonnes sont disponibles dans le mappage Inspecter et Récepteur.

### <a name="generate-metadata-from-drifted-columns"></a>Générer des métadonnées à partir de colonnes dérivées

Un tableau croisé dynamique génère de nouveaux noms de colonnes de manière dynamique en fonction des valeurs de ligne. Vous pouvez ajouter ces nouvelles colonnes dans les métadonnées qui peuvent ensuite être référencées dans votre flux de données. Pour ce faire, utilisez l’action rapide [Mapper les éléments dérivés](concepts-data-flow-schema-drift.md#map-drifted-columns-quick-action) dans l’aperçu des données. 

![Colonnes de tableau croisé dynamique](media/data-flow/newpivot1.png "Mappage de colonnes dérivées de tableau croisé dynamique")

### <a name="sinking-pivoted-columns"></a>Réception de colonnes ayant fait l’objet d’un ajout de tableau croisé dynamique

Bien que les colonnes ayant fait l’objet d’un ajout de tableau croisé dynamique soient dynamiques, elles peuvent toujours être écrites dans votre magasin de données de destination. Activez **Autoriser la dérive de schéma** dans vos paramètres de récepteur. Cela vous permet d’écrire des colonnes qui ne sont pas incluses dans les métadonnées. Vous ne verrez pas les nouveaux noms dynamiques dans vos métadonnées de colonnes, mais l’option de dérive de schéma vous permettra de charger les données.

### <a name="rejoin-original-fields"></a>Rejoindre des champs d’origine

La transformation de tableau croisé dynamique projette uniquement les colonnes Regrouper par et ayant fait l’objet d’un ajout de tableau croisé dynamique. Si vous souhaitez que vos données de sortie incluent d’autres colonnes d’entrée, utilisez un modèle de [jointure réflexive](data-flow-join.md#self-join).

## <a name="data-flow-script"></a>Script de flux de données

### <a name="syntax"></a>Syntaxe

```
<incomingStreamName>
    pivot(groupBy(Tm),
        pivotBy(<pivotKeyColumn, [<specifiedColumnName1>,...,<specifiedColumnNameN>]),
        <pivotColumnPrefix> = <pivotedColumnValue>,
        columnNaming: '< prefix >< $N | $V ><middle >< $N | $V >< suffix >',
        lateral: { 'true' | 'false'}
    ) ~> <pivotTransformationName
```
### <a name="example"></a>Exemple

Les écrans affichés dans la section de configuration comportent le script de transmission de données suivant :

```
BasketballPlayerStats pivot(groupBy(Tm),
    pivotBy(Pos),
    {} = count(),
    columnNaming: '$V$N count',
    lateral: true) ~> PivotExample

```

## <a name="next-steps"></a>Étapes suivantes

Essayez la [transformation Supprimer le tableau croisé dynamique](data-flow-unpivot.md) pour transformer des valeurs de colonnes en valeurs de lignes. 

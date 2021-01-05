---
title: Information de référence sur sur le module Fractionner les données
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Fractionner les données dans Azure Machine Learning pour diviser un jeu de données en deux jeux distincts.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 10/22/2019
ms.openlocfilehash: a4c93b12ad654e54a7f3c7ee0e75507d5cb45e90
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90907823"
---
# <a name="split-data-module"></a>Module Fractionner les données

Cet article décrit un module dans le concepteur Azure Machine Learning.

Utilisez le module Fractionner les données pour diviser un jeu de données en deux jeux distincts.

Ce module est utile quand vous devez séparer des données en jeux d’entraînement et de test. Vous pouvez également personnaliser la façon dont les données sont divisées. Certaines options prennent en charge la randomisation des données. D’autres sont adaptées à un type de données ou à un type de modèle spécifique.

## <a name="configure-the-module"></a>Configurer le module

> [!TIP]
> Avant de choisir le mode de fractionnement, prenez connaissance de toutes les options afin de déterminer le type de fractionnement adapté à vos besoins.
> Si vous changez le mode de fractionnement, toutes les autres options peuvent être réinitialisées.

1. Ajoutez le module **Fractionner les données** à votre pipeline dans le concepteur. Ce module est disponible sous **Transformation des données**, dans la catégorie **Échantillonner et fractionner**.

1. **Mode de fractionnement** : Choisissez l’un des modes suivants, selon le type de données dont vous disposez et la manière dont vous souhaitez les diviser. Chaque mode de fractionnement s'accompagne de différentes options.

   - **Fractionner les lignes** : Utilisez cette option pour diviser simplement les données en deux parties. Vous pouvez spécifier le pourcentage de données de chaque fractionnement. Par défaut, les données sont divisées à 50/50.

     Vous pouvez également rendre aléatoire la sélection des lignes de chaque groupe et utiliser l’échantillonnage stratifié. Dans l’échantillonnage stratifié, vous devez sélectionner une seule colonne de données pour laquelle vous souhaitez que les valeurs soient équitablement réparties entre les deux jeux de données de résultats.  

   - **Fractionnement Expression régulière** : choisissez cette option si vous souhaitez diviser votre jeu de données en testant une seule colonne pour une valeur.

     Par exemple, si vous analysez les sentiments, vous pouvez vérifier la présence d’un nom de produit particulier dans un champ de texte. Vous pouvez ensuite diviser le jeu de données en lignes avec le nom de produit cible et en lignes sans le nom de produit cible.

   - **Fractionnement Expression relative** : Utilisez cette option si vous souhaitez appliquer une condition à une colonne numérique. Le nombre peut correspondre à un champ de date/heure, une colonne qui contient un âge ou des montants en dollars, voire un pourcentage. Par exemple, vous souhaiterez peut-être diviser votre jeu de données en fonction du prix des éléments, grouper des personnes par tranches d’âge ou séparer les données par date.

### <a name="split-rows"></a>Fractionner les lignes

1. Ajoutez le module [Fractionner les données](./split-data.md) à votre pipeline dans le concepteur et connectez le jeu de données que vous souhaitez fractionner.
  
1. Pour **Mode de fractionnement**, sélectionnez **Lignes fractionnées**. 

1. **Fraction de lignes dans le premier jeu de données de sortie** : utilisez cette option pour déterminer le nombre de lignes de la première sortie (côté gauche). Toutes les autres lignes rejoindront la deuxième sortie (côté droit).

   Le rapport représente le pourcentage de lignes envoyées au premier jeu de données de sortie, et vous devez entrer un nombre décimal compris entre 0 et 1.
     
   Par exemple, si vous entrez **0.75** comme valeur, le jeu de données est fractionné à 75/25. Dans ce fractionnement, 75 % des lignes sont envoyées au premier jeu de données de sortie. Les 25 % restants sont envoyés au second jeu de données de sortie.
  
1. Sélectionnez l'option **Fractionnement aléatoire** si vous souhaitez rendre aléatoire la sélection de données dans les deux groupes. Cette option est recommandée quand vous créez des jeux de données d’entraînement et de test.

1. **Valeur de départ aléatoire** : entrez une valeur entière positive pour démarrer la séquence pseudoaléatoire d’instances à utiliser. Cette valeur de départ par défaut est utilisée dans tous les modules qui génèrent des nombres aléatoires. 

   La spécification d’une valeur de départ rend les résultats reproductibles. Pour répéter les résultats d’une opération de fractionnement, vous devez spécifier une valeur de départ pour le générateur de nombres aléatoires. Sinon, la valeur de départ aléatoire est définie sur **0** par défaut, ce qui signifie qu’elle est obtenue à partir de l’horloge système. Dès lors, la répartition des données peut légèrement varier chaque fois que vous effectuez un fractionnement. 

1. **Fractionnement stratifié** : Définissez cette option sur **True** pour vous assurer que les deux jeux de données de sortie contiennent un échantillon représentatif des valeurs dans la *colonne de strate* ou la *colonne clé de stratification*. 

   Avec un échantillonnage stratifié, les données sont divisées de manière à ce que chaque jeu de données de sortie obtienne approximativement le même pourcentage de chaque valeur cible. Par exemple, vous souhaiterez peut-être que vos jeux d’entraînement et de test soient globalement équilibrés en termes de résultat ou autres (comme le genre).

1. Envoyez le pipeline.


## <a name="select-a-regular-expression"></a>Sélectionner une expression régulière

1. Ajoutez le module [Fractionner les données](./split-data.md) à votre pipeline et connectez-le en tant qu’entrée au jeu de données que vous souhaitez fractionner.  
  
1. Pour **Mode de fractionnement**, sélectionnez **Fractionnement Expression régulière**.

1. Dans la zone **Expression régulière**, entrez une expression régulière valide. 
  
   L’expression régulière doit respecter la syntaxe Python des expressions régulières.

1. Envoyez le pipeline.

   En fonction de l’expression régulière entrée, le jeu de données est divisé en deux ensembles de lignes : les lignes avec des valeurs correspondant à l’expression et les autres. 

Les exemples suivants montrent comment diviser un jeu de données à l’aide de l’option **Expression régulière**. 

### <a name="single-whole-word"></a>Mot entier unique 

Cet exemple place dans le premier jeu de données toutes les lignes qui contiennent le texte `Gryphon` dans la colonne `Text`. Il place les autres lignes dans la deuxième sortie de **Fractionner les données**.

```text
    \"Text" Gryphon  
```

### <a name="substring"></a>Substring

Cet exemple recherche la chaîne spécifiée dans n’importe quelle position dans la deuxième colonne du jeu de données. La position est indiquée ici par la valeur d’index 1. La correspondance respecte la casse.

```text
(\1) ^[a-f]
```

Le premier jeu de données de résultats contient toutes les lignes où la colonne d’index commence par l’un des caractères suivants : `a`, `b`, `c`, `d`, `e`, `f`. Toutes les autres lignes sont dirigées vers la deuxième sortie.

## <a name="select-a-relative-expression"></a>Sélectionner une expression relative

1. Ajoutez le module [Fractionner les données](./split-data.md) à votre pipeline et connectez-le en tant qu’entrée au jeu de données que vous souhaitez fractionner.
  
1. Pour **Mode de fractionnement**, sélectionnez **Expression relative**.
  
1. Dans la zone **Expression relationnelle**, entrez une expression qui effectue une opération de comparaison sur une seule colonne.

   Pour **Colonne numérique** :
   - La colonne contient des nombres de n’importe quel type de données numérique, y compris les types de données date et heure.
   - L’expression peut faire référence à un nom de colonne.
   - Utilisez le caractère esperluette, `&`, pour l’opération ET. Utilisez le caractère de barre verticale, `|`, pour l’opération OU.
   - Les opérateurs suivants sont pris en charge : `<`, `>`, `<=`, `>=`, `==`, `!=`.
   - Vous ne pouvez pas regrouper les opérations en utilisant `(` et `)`.
   
   Pour **Colonne de chaîne** :
   - Les opérateurs suivants sont pris en charge : `==`, `!=`.

1. Envoyez le pipeline.

   L’expression divise le jeu de données en deux ensembles de lignes : les lignes avec des valeurs correspondant à la condition, et les autres.

Les exemples suivants montrent comment diviser un jeu de données à l’aide de l’option **Expression régulière** dans le module **Fractionner les données**.  

### <a name="calendar-year"></a>Année civile

Un scénario courant consiste à diviser un jeu de données par années. L’expression suivante sélectionne toutes les lignes où les valeurs dans la colonne `Year` sont supérieures à `2010`.

```text
\"Year" > 2010
```

L’expression de date doit tenir compte de toutes les parties de date incluses dans la colonne de données. Le format des dates dans la colonne de données doit être cohérent. 

Par exemple, dans une colonne de date qui utilise le format `mmddyyyy`, l’expression doit être semblable à ce qui suit :

```text
\"Date" > 1/1/2010
```

### <a name="column-index"></a>Index de colonne

L’expression suivante montre comment utiliser l’index de colonne pour sélectionner toutes les lignes de la première colonne du jeu de données qui contiennent des valeurs inférieures ou égales à 30, mais différentes de 20.

```text
(\0)<=30 & !=20
```


## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 

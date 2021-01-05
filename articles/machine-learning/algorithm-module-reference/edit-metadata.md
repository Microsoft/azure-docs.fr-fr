---
title: Informations de référence sur le module Modifier les métadonnées
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Modifier les métadonnées d’Azure Machine Learning pour changer les métadonnées associées aux colonnes d’un jeu de données.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 06/10/2020
ms.openlocfilehash: e279a7c9f6810ece482b043284f0be0719a3dafe
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90908040"
---
# <a name="edit-metadata-module"></a>Module Modifier les métadonnées

Cet article décrit un module inclus dans le concepteur Azure Machine Learning.

Utilisez le module Modifier les métadonnées pour modifier les métadonnées associées aux colonnes d’un jeu de données. La valeur et le type de données du jeu de données sont modifiés après l’utilisation du module Modifier les métadonnées.

Les modifications de métadonnées standard peuvent être les suivantes :
  
+ Traitement des valeurs booléennes ou numériques des colonnes en tant que valeurs catégorielles
  
+ Désignation de la colonne qui contient l’étiquette de **classe** ou les valeurs que vous souhaitez classer ou prédire.
  
+ Marquage des colonnes en tant que fonctionnalités.
  
+ Changement des valeurs de date/heure en valeurs numériques ou vice versa.
  
+ Changement de nom des colonnes.
  
 Utilisez le module Modifier les métadonnées à chaque fois que vous devez modifier la définition d’une colonne, généralement pour répondre aux besoins d’un module en aval. Par exemple, certains modules utilisent uniquement des types de données spécifiques ou nécessitent des indicateurs sur les colonnes, comme `IsFeature` ou `IsCategorical`.  
  
 Après avoir effectué l’opération requise, vous pouvez rétablir l’état d’origine des métadonnées.
  
## <a name="configure-edit-metadata"></a>Configurer le module Modifier les métadonnées
  
1. Dans le concepteur Azure Machine Learning, ajoutez le module Modifier les métadonnées à votre pipeline et connectez le jeu de données à mettre à jour. Vous pouvez rechercher le module dans la catégorie **Transformation des données**.
  
1. Cliquez sur **Modifier la colonne** dans le volet droit du module, et sélectionnez la colonne ou le jeu de colonnes à utiliser. Vous pouvez sélectionner des colonnes individuellement par nom ou index, ou vous pouvez choisir un groupe de colonnes par type.  
  
1. Sélectionnez l’option **Type de données** si vous devez attribuer un autre type de données aux colonnes sélectionnées. Vous devrez peut-être modifier le type de données pour certaines opérations. Par exemple, si votre jeu de données source comporte des nombres traités comme du texte, vous devez les remplacer par un type de données numériques avant d’utiliser des opérations mathématiques.

    + Types de données prises en charge : **Chaîne**, **Entier**, **Double**, **Booléen** et **DateHeure**.

    + Si vous sélectionnez plusieurs colonnes, vous devez appliquer les modifications des métadonnées à *toutes* les colonnes sélectionnées. Par exemple, supposons que vous choisissiez deux ou trois colonnes numériques. Vous pouvez remplacer toutes les données numériques par des données de type chaîne et les renommer en une seule opération. Toutefois, vous ne pouvez pas modifier une colonne en données de type chaîne et remplacer une autre colonne de données de type flottant en données de type entier.
  
    + Si vous ne spécifiez pas un nouveau type de données, les métadonnées de colonne restent inchangées.

    + Le type de colonne et les valeurs sont modifiés une fois que vous avez effectué l’opération Modifier les métadonnées. Vous pouvez récupérer le type de données d’origine à tout moment à l’aide de Modifier les métadonnées pour rétablir le type de données de colonne.  

    > [!NOTE]
    > Le **format DateHeure** suit le [format DateHeure intégré de Python](https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior).  
    > Si vous remplacez un type de nombre par le type **DateHeure**, laissez le champ **Format date et heure** vide. Actuellement, il n’est pas possible de spécifier le format cible des données.

1. Sélectionnez l’option **Catégorie** pour spécifier que les valeurs indiquées dans les colonnes sélectionnées doivent être considérées comme des catégories.

    Par exemple, une colonne peut contenir les nombres 0, 1 et 2, mais vous savez que ces nombres signifient en fait « Fumeur », « Non fumeur » et « Inconnu ». Dans ce cas, le fait de marquer la colonne comme catégorielle vous assure que les valeurs sont utilisées uniquement pour regrouper les données et non dans des calculs numériques.
  
1. Utilisez l’option **Champs** si vous souhaitez modifier la façon dont Azure Machine Learning utilise les données d’un modèle.

    + **Caractéristique** : Utilisez cette option pour marquer une colonne en tant que fonctionnalité dans les modules qui fonctionnent uniquement sur les colonnes de fonctionnalité. Par défaut, toutes les colonnes sont traitées initialement comme des fonctionnalités.  
  
    + **Étiquette** : Utilisez cette option pour marquer l’étiquette, qui est également appelée attribut prédictible ou variable cible. De nombreux modules nécessitent qu’une colonne d’étiquette précisément soit présente dans le jeu de données.

        Dans de nombreux cas, Azure Machine Learning peut déduire qu’une colonne contient une étiquette de classe. En définissant ces métadonnées, vous pouvez vous assurer que la colonne est identifiée correctement. La définition de cette option ne modifie pas les valeurs de données. Elle modifie uniquement la manière dont certains algorithmes d’apprentissage automatique gèrent les données.
  
    > [!TIP]
    > Disposez-vous de données qui ne s’inscrivent pas dans ces catégories ? Par exemple, votre jeu de données peut contenir des valeurs telles que des identificateurs uniques qui ne sont pas utiles en tant que variables. Parfois, ces ID peuvent entraîner des problèmes lorsqu’ils sont utilisés dans un modèle.
    >
    > Heureusement, Azure Machine Learning conserve toutes vos données, afin que vous n’ayez pas à supprimer ces colonnes du jeu de données. Lorsque vous devez effectuer des opérations sur un ensemble spécial de colonnes, supprimez simplement toutes les autres colonnes temporairement à l’aide du module [Sélectionner des colonnes dans un jeu de données](select-columns-in-dataset.md). Vous pouvez ultérieurement fusionner à nouveau les colonnes dans le jeu de données à l’aide du module [Ajouter des colonnes](add-columns.md).  
  
1. Utilisez les options suivantes pour effacer les sélections précédentes et rétablir les valeurs par défaut des métadonnées.  
  
    + **Effacer la fonctionnalité** : Utilisez cette option pour supprimer l’indicateur de fonctionnalité.  
  
         Toutes les colonnes sont traitées initialement comme des fonctionnalités. Pour les modules qui effectuent des opérations mathématiques, vous devrez peut-être utiliser cette option afin d’empêcher que les colonnes numériques ne soient considérées comme des variables.
  
    + **Effacer l’étiquette** : Utilisez cette option pour supprimer les métadonnées **d’étiquette** de la colonne spécifiée.  
  
    + **Effacer le score** : Utilisez cette option pour supprimer les métadonnées **de score** de la colonne spécifiée.  
  
         Pour le moment, vous ne pouvez pas marquer explicitement une colonne en tant que score dans Azure Machine Learning. Toutefois, certaines opérations entraînent le marquage d’une colonne en tant que score en interne. De même, un module R personnalisé peut générer des valeurs de score.

1. Pour le champ **Nouveaux noms de colonnes**, entrez le nouveau nom de la ou des colonnes sélectionnées.  
  
    + Les noms de colonnes ne peuvent comporter que des caractères pris en charge par l’encodage UTF-8. Les chaînes vides, les valeurs Null ou les noms composés uniquement d’espaces ne sont pas autorisés.  
  
    + Pour renommer plusieurs colonnes, entrez les noms sous forme d’une liste séparée par des virgules dans l’ordre des index de colonne.  
  
    + Toutes les colonnes sélectionnées doivent être renommées. Vous ne pouvez pas omettre ni ignorer des colonnes.  
  
1. Envoyez le pipeline.  

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning.

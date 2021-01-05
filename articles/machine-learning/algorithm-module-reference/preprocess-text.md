---
title: 'Pré-traiter le texte : Informations de référence sur les modules'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Pré-traiter le texte dans le concepteur Azure Machine Learning pour nettoyer et simplifier le texte.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 11/16/2020
ms.openlocfilehash: 366b30df677a5b74bc7d70e1aea60e05b4df0152
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94659281"
---
# <a name="preprocess-text"></a>Pré-traiter le texte

Cet article décrit un module dans le concepteur Azure Machine Learning.

Utilisez le module **Prétraiter le texte** pour nettoyer et simplifier le texte. Ce module prend en charge ces opérations courantes de traitement de texte :

* Suppression des mots vides
* Utilisation d’expressions régulières pour rechercher et remplacer des chaînes cibles spécifiques
* Lemmatisation, qui convertit plusieurs mots connexes en une seule forme canonique
* Normalisation de casse
* Suppression de certaines classes de caractères, comme les nombres, les caractères spéciaux et des séquences de caractères répétés comme « aaaa »
* Identification et suppression des adresses e-mail et URL

Actuellement, le module **Pré-traiter le texte** prend en charge uniquement l’anglais.

## <a name="configure-text-preprocessing"></a>Configurer le prétraitement du texte  

1.  Ajoutez le module **Pré-traiter le texte** à votre pipeline dans Azure Machine Learning. Ce module se trouve sous **Analyse de texte**.

1. Connectez un jeu de données qui comprend au moins une colonne contenant du texte.

1. Sélectionnez la langue dans la liste déroulante **Langue**.

1. **Colonne de texte à nettoyer** : Sélectionnez la colonne à prétraiter.

1. **Supprimer les mots vides** : Sélectionnez cette option si vous voulez appliquer une liste prédéfinie de mots vides à la colonne de texte. 

    Les listes de mots vides dépendent de la langue et sont personnalisables.

1. **Lemmatisation** : Sélectionnez cette option si vous voulez que les mots soient représentés sous leur forme canonique. Cette option s’avère utile pour réduire le nombre d’occurrences uniques des jetons de texte autrement similaires.

    Le processus de lemmatisation dépend fortement de la langue.

1. **Détecter les phrases** : Sélectionnez cette option si vous voulez que le module insère une marque de limite de phrase lors de l’exécution de l’analyse.

    Ce module utilise une série de trois barres verticales `|||` pour représenter la ponctuation de fin de phrase.

1. Effectuez des opérations de recherche et remplacement éventuelles à l’aide d’expressions régulières. L’expression régulière sera traitée en premier, avant toutes les autres options intégrées.

    * **Expression régulière personnalisée** : Définissez le texte que vous recherchez.
    * **Chaîne de remplacement personnalisée** : Définissez une valeur de remplacement unique.

1. **Normaliser la casse en minuscules** : Sélectionnez cette option si vous voulez convertir les caractères majuscules ASCII en minuscules.

    Si les caractères ne sont pas normalisés, le même mot en lettres majuscules et en lettres minuscules est considéré comme deux mots différents.

1. Vous pouvez également supprimer les types suivants de caractères ou de séquences de caractères du texte de sortie traité :

    * **Supprimer les chiffres** : Sélectionnez cette option pour supprimer tous les caractères numériques de la langue spécifiée. Les numéros d’identification dépendent du domaine et de la langue. Si les caractères numériques font partie intégrante d’un mot connu, ils ne peuvent pas être supprimés. Pour plus d’informations, consultez [Notes techniques](#technical-notes).
    
    * **Supprimer les caractères spéciaux** : Utilisez cette option pour supprimer tous les caractères spéciaux non alphanumériques.
    
    * **Supprimer les caractères en double** : Sélectionnez cette option pour supprimer les caractères supplémentaires dans toutes les séquences qui se répètent plus de deux fois. Par exemple, une séquence comme « aaaaa » serait réduite à « aa ».
    
    * **Supprimer les adresses e-mail** : Sélectionnez cette option pour supprimer toutes les séquences au format `<string>@<string>`.  
    * **Supprimer les URL** : Sélectionnez cette option pour supprimer toutes les séquences qui incluent les préfixes d’URL suivants : `http`, `https`, `ftp`, `www`
    
1. **Développer les contractions verbales** : Cette option s’applique uniquement aux langues qui utilisent les contractions verbales. Actuellement, il s’agit de l’anglais uniquement. 

    Par exemple, en sélectionnant cette option, vous pouvez remplacer la locution *« wouldn’t stay there »* par *« would not stay there »*.

1. **Normaliser les barres obliques inverses en barres obliques** : Sélectionnez cette option pour mapper toutes les instances de `\\` en `/`.

1. **Fractionner les jetons sur les caractères spéciaux** : Sélectionnez cette option si vous voulez couper des mots sur des caractères comme `&`, `-`, etc. Cette option permet également de réduire les caractères spéciaux quand ils se répètent plus de deux fois. 

    Par exemple, la chaîne `MS---WORD` est divisée en trois jetons, `MS`, `-` et `WORD`.

1. Envoyez le pipeline.

## <a name="technical-notes"></a>Notes techniques

Le module de **prétraitement du texte** dans Studio (Classique) et le concepteur utilisent des modèles de langage différents. Le concepteur utilise un modèle formé CNN à plusieurs tâches de [spaCy](https://spacy.io/models/en). Différents modèles offrent différents générateurs de jetons et différents générateurs de balises de catégorie grammaticale, ce qui aboutit à des résultats différents.

En voici quelques exemples :

| Configuration | Résultat de la sortie |
| --- | --- |
|Avec toutes les options sélectionnées </br> Explication : </br> Pour les cas comme « 3test » dans « WC-3 3test 4test », le concepteur supprime le mot entier « 3test » étant donné que, dans ce contexte, le générateur de balises de catégorie grammaticale spécifie le jeton « 3test » comme nombre et, en fonction de la catégorie grammaticale, le module le supprime.| :::image type="content" source="./media/module/preprocess-text-all-options-selected.png" alt-text="Avec toutes les options sélectionnées" border="True"::: |
|Avec uniquement `Removing number` sélectionnée </br> Explication : </br> Pour les cas tels que « 3test », « 4-EC », le générateur de jetons du concepteur ne les fractionnent pas et les traite comme des jetons entiers. Par conséquent, il ne supprime pas les nombres dans ces mots.| :::image type="content" source="./media/module/preprocess-text-removing-numbers-selected.png" alt-text="Avec uniquement l’option « Removing number » sélectionnée" border="True"::: |

Vous pouvez également utiliser une expression régulière pour produire des résultats personnalisés :

| Configuration | Résultat de la sortie |
| --- | --- |
|Avec toutes les options sélectionnées </br> Expression régulière personnalisée : `(\s+)*(-|\d+)(\s+)*` </br> Chaîne de remplacement personnalisée : `\1 \2 \3`| :::image type="content" source="./media/module/preprocess-text-regular-expression-all-options-selected.png" alt-text="Avec toutes les options sélectionnées et une expression régulière" border="True"::: |
|Avec uniquement `Removing number` sélectionnée </br> Expression régulière personnalisée : `(\s+)*(-|\d+)(\s+)*` </br> Chaîne de remplacement personnalisée : `\1 \2 \3`| :::image type="content" source="./media/module/preprocess-text-regular-expression-removing-numbers-selected.png" alt-text="Avec l’option « Removing number » sélectionnée et une expression régulière" border="True"::: |


## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 
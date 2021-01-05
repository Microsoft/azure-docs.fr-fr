---
title: Appliquer une opération mathématique
titleSuffix: Azure Machine Learning
description: Apprenez à utiliser le module Appliquer une opération mathématique dans Azure Machine Learning pour appliquer une opération mathématique aux valeurs de colonne dans un pipeline.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 09/09/2019
ms.openlocfilehash: 04ce45f428604275696d83938708bcee0c6c023f
ms.sourcegitcommit: f5580dd1d1799de15646e195f0120b9f9255617b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/29/2020
ms.locfileid: "91536764"
---
# <a name="apply-math-operation"></a>Appliquer une opération mathématique

Cet article décrit un module du concepteur Azure Machine Learning.

Utilisez le module Appliquer une opération mathématique pour créer des calculs et les appliquer aux colonnes numériques du jeu de données d’entrée. 

Parmi les opérations mathématiques figurent des fonctions arithmétiques, des fonctions trigonométriques, des fonctions d’arrondi et des fonctions spéciales utilisées en science des données, comme les fonctions gamma et les fonctions d’erreur.  

Une fois que vous avez défini une opération et exécuté le pipeline, les valeurs sont ajoutées à votre jeu de données. Selon la façon dont vous configurez le module, vous pouvez :

+ Ajoutez les résultats à votre jeu de données (utile pour vérifier le résultat de l’opération).
+ Remplacer les valeurs de colonne par les nouvelles valeurs calculées.
+ Générer une nouvelle colonne pour les résultats et ne pas afficher les données d’origine. 

Recherchez l’opération dont vous avez besoin dans les catégories suivantes :  

- [De base](#basic-math-operations)  
  
     Les fonctions de la catégorie **De base** peuvent être utilisées pour manipuler une seule valeur ou une colonne de valeurs. Par exemple, vous pouvez obtenir la valeur absolue de tous les nombres d’une colonne, ou calculer la racine carrée de chaque valeur dans une colonne.  
  
-   [Compare](#comparison-operations)  
  
      Les fonctions de la catégorie **Comparer** sont toutes utilisées pour la comparaison : vous pouvez comparer des paires de valeurs de deux colonnes, ou comparer chaque valeur d’une colonne à une constante spécifiée. Par exemple, vous pouvez comparer des colonnes pour déterminer si les valeurs étaient identiques dans deux jeux de données. Vous pouvez également utiliser une constante, par exemple, une valeur maximale autorisée, pour rechercher les valeurs hors norme dans une colonne numérique.  
  
-   [Opérations](#arithmetic-operations)  
  
     La catégorie **Opérations** comprend des fonctions mathématiques de base : addition, soustraction, multiplication et division. Vous pouvez utiliser des colonnes ou des constantes. Par exemple, vous pouvez ajouter la valeur de la colonne A à la valeur de la colonne B. Vous pouvez également soustraire une constante, par exemple, une moyenne calculée précédemment, de chaque valeur de la colonne A.  
  
-   [Arrondi](#rounding-operations)  
  
     La catégorie **Arrondi** comprend diverses fonctions pour effectuer diverses opérations, par exemple, arrondi, plafond, plancher et troncation à différents niveaux de précision. Vous pouvez spécifier le niveau de précision pour les nombres décimaux et entiers.  
  
-   [Spécial](#special-math-functions)  
  
     La catégorie **Spécial** comprend des fonctions mathématiques utilisées en particulier dans la science des données, comme les intégrales elliptiques et la fonction d’erreur gaussienne.  
  
-   [Trigonométrie](#trigonometric-functions)  
  
     La catégorie **Trigonométrie** comprend toutes les fonctions trigonométriques standard. Par exemple, vous pouvez convertir des radians en degrés, ou des fonctions de calcul comme la tangente en radians ou degrés.
     Ces fonctions sont unaires, ce qui signifie qu’elles prennent une seule colonne de valeurs comme entrée, appliquent la fonction trigonométrique et retournent une colonne de valeurs en résultat. Vérifiez que la colonne d’entrée est du bon type et qu’elle contient le type de valeur qui convient pour l’opération spécifiée.   

## <a name="how-to-configure-apply-math-operation"></a>Comment configurer le module Appliquer une opération mathématique  

Le module **Appliquer une opération mathématique** nécessite un jeu de données avec au moins une colonne contenant uniquement des nombres. Les nombres peuvent être discrets ou continus, mais doivent avoir un type de données numérique et non une chaîne.

Vous pouvez appliquer la même opération à plusieurs colonnes numériques, mais toutes les colonnes doivent se trouver dans le même jeu de données. 

Chaque instance de ce module peut exécuter un seul type d’opération à la fois. Pour effectuer des opérations mathématiques complexes, vous devez peut-être chaîner plusieurs instances du module **Appliquer une opération mathématique**.  
  
1.  Ajoutez le module **Appliquer une opération mathématique** à votre pipeline.

1. Connectez un jeu de données contenant au moins une colonne numérique.  

1.  Sélectionnez une ou plusieurs colonnes sources sur lesquelles effectuer le calcul.   
  
    - Toute colonne que vous choisissez doit avoir un type de données numérique. 
    - La plage de données doit être valide pour l’opération mathématique sélectionnée. Sinon, une erreur ou un résultat NaN (autre qu’un nombre) peut se produire. Par exemple, Ln(-1,0) est une opération non valide qui génère la valeur `NaN`.
  
1.  Sélectionnez **Catégorie** pour sélectionner le **type** de l’opération mathématique à effectuer.
    
1. Choisissez une opération spécifique dans la liste de cette catégorie.
  
1.  Définissez les paramètres supplémentaires nécessaires pour chaque type d’opération.  
  
1.  Utilisez l’option **Mode de sortie** pour indiquer comment vous voulez que l’opération mathématique soit générée : 

    - **Append**. Toutes les colonnes utilisées comme entrées sont incluses dans le jeu de données de sortie, une colonne supplémentaire est ajoutée pour contenir les résultats de l’opération mathématique.
    - **Inplace**. Les valeurs des colonnes utilisées comme entrées sont remplacées par les nouvelles valeurs calculées. 
    - **ResultOnly**. Une seule colonne est retournée avec les résultats de l’opération mathématique.
  
1.  Envoyez le pipeline.  
  
## <a name="results"></a>Résultats

Si vous générez les résultats à l’aide des options **Append** ou **ResultOnly**, les en-têtes de colonne du jeu de données retourné indiquent l’opération et les colonnes qui ont été utilisées. Par exemple, si vous comparez deux colonnes à l’aide de l’opérateur **Equals**, les résultats ressemblent à ceci :  
  
-   **Equals(Col2_Col1)**, qui indique que vous avez testé Col2 sur Col1.  
-   **Equals(Col2_$10)**, qui indique que vous avez comparé la colonne 2 à la constante 10.  

Même avec l’option **Inplace**, les données sources ne sont pas supprimées ni modifiées. La colonne du jeu de données d’origine est toujours disponible dans le concepteur. Pour voir les données d’origine, vous pouvez connecter le module [Ajouter des colonnes](add-columns.md) et le joindre à la sortie du module **Appliquer une opération mathématique**.  
    
## <a name="basic-math-operations"></a>Opérations mathématiques de base 

Les fonctions de la catégorie **De base** prennent généralement une seule valeur d’une colonne, effectuent l’opération prédéfinie et retournent une seule valeur. Pour certaines fonctions, vous pouvez spécifier une constante ou un jeu de colonnes comme deuxième argument.  
  
 Azure Machine Learning prend en charge les fonctions suivantes dans la catégorie **De base** :  

### <a name="abs"></a>Abs

Retourne la valeur absolue des colonnes sélectionnées.  
  
### <a name="atan2"></a>Atan2

Retourne une tangente inverse à quatre quadrants.  

Sélectionnez les colonnes qui contiennent les coordonnées de point. Pour le deuxième argument, qui correspond à la coordonnée x, vous pouvez également spécifier une constante.  

Correspond à la fonction ATAN2 en MATLAB.  

### <a name="conj"></a>Conj

Retourne le conjugué des valeurs de la colonne sélectionnée.  

### <a name="cuberoot"></a>CubeRoot

Retourne la racine cube des valeurs de la colonne sélectionnée.  

### <a name="doublefactorial"></a>DoubleFactorial  
 Calcule la factorielle double des valeurs de la colonne sélectionnée. La factorielle double est une extension de la fonction factorielle normale, dont la notation est `x!!`.  

### <a name="eps"></a>Eps

Retourne la taille de l’écart entre la valeur actuelle et le plus grand nombre suivant à double précision. Correspond à la fonction EPS en MATLAB.  
  
### <a name="exp"></a>Exp

Retourne e élevé à la puissance de la valeur de la colonne sélectionnée. C’est l’équivalent de la fonction Excel EXP.  

### <a name="exp2"></a>Exp2

Retourne la valeur exponentielle en base 2 des arguments, résolution de y = x * 2<sup>t</sup>, où t est une colonne de valeurs contenant des exposants.  

Dans le **jeu de colonnes**, sélectionnez la colonne qui contient les valeurs de l’exposant t.

Pour **Exp2**, vous pouvez spécifier un deuxième argument x, qui peut être une constante ou une autre colonne de valeurs. Dans **Deuxième type d’argument**, indiquez si vous fournissez le multiplicateur x comme une constante ou une valeur dans une colonne.  

Par exemple, si vous sélectionnez une colonne avec les valeurs {0,1,2,3,4,5} pour le multiplicateur et l’exposant, la fonction retourne {0, 2, 8, 24, 64 160).  

### <a name="expminus1"></a>ExpMinus1 

Retourne l’exposant négatif des valeurs de la colonne sélectionnée.  

### <a name="factorial"></a>Factorielle
Retourne la factorielle des valeurs de la colonne sélectionnée.  

### <a name="hypotenuse"></a>Hypotenuse
Calcule l’hypoténuse d’un triangle dans lequel la longueur d’un côté est une colonne de valeurs, et la longueur du deuxième côté est une constante ou deux colonnes.  

### <a name="ln"></a>Ln

Retourne le logarithme naturel des valeurs de la colonne sélectionnée.  

### <a name="lnplus1"></a>LnPlus1

Retourne le logarithme naturel plus un des valeurs de la colonne sélectionnée.  

### <a name="log"></a>Journal

Retourne le journal des valeurs de la colonne sélectionnée, en fonction de la base spécifiée.  

Vous pouvez spécifier la base (le deuxième argument) en choisissant une constante ou une autre colonne de valeurs.  

### <a name="log10"></a>Log10

Retourne les valeurs logarithmiques de base 10 pour la colonne sélectionnée.  

### <a name="log2"></a>Log2

Retourne les valeurs logarithmiques de base 2 pour la colonne sélectionnée.  

### <a name="nthroot"></a>NthRoot
Retourne la racine énième de la valeur, à l’aide d’une valeur n que vous spécifiez.  

Sélectionnez les colonnes pour lesquelles vous voulez calculer la racine, à l’aide de l’option **ColumnSet**.  

Dans **Deuxième type d’argument**, sélectionnez une autre colonne qui contient la racine ou spécifiez une constante à utiliser comme racine.  

Si le deuxième argument est une colonne, chaque valeur de la colonne est utilisée comme valeur de n pour la ligne correspondante. Si le deuxième argument est une constante, tapez la valeur de n dans la zone de texte **Deuxième argument**.
### <a name="pow"></a>Pow

Calcule X élevé à la puissance Y pour chacune des valeurs de la colonne sélectionnée.  

Commencez par sélectionner les colonnes qui contiennent la **base**, qui doit être une valeur float, à l’aide de l’option **ColumnSet**.  

Dans **Deuxième type d’argument**, sélectionnez la colonne qui contient l’exposant ou spécifiez une constante à utiliser comme exposant.  

Si le deuxième argument est une colonne, chaque valeur de la colonne est utilisée comme exposant de la ligne correspondante. Si le deuxième argument est une constante, tapez la valeur de l’exposant dans la zone de texte **Deuxième argument**.  

### <a name="sqrt"></a>Sqrt

Retourne la racine carrée des valeurs de la colonne sélectionnée.  

### <a name="sqrtpi"></a>SqrtPi

Pour chaque valeur de la colonne sélectionnée, multiplie la valeur par pi, puis retourne la racine carrée du résultat.  

### <a name="square"></a>Carré

Élève au carré les valeurs de la colonne sélectionnée.  

## <a name="comparison-operations"></a>Opérations de comparaison  

Utilisez les fonctions de comparaison dans le concepteur Azure Machine Learning chaque fois que vous devez tester deux ensembles de valeurs l’un par rapport à l’autre. Par exemple, dans un pipeline, vous pouvez être amené à effectuer ces opérations de comparaison :  

- Évaluer une colonne du modèle de score de probabilité par rapport à une valeur de seuil.
- Déterminez si deux jeux de résultats sont identiques. Pour chaque ligne différente, ajoutez un indicateur FALSE, utilisable par la suite (traitement ou filtrage).  

### <a name="equalto"></a>EqualTo

Retourne True si les valeurs sont identiques.  

### <a name="greaterthan"></a>GreaterThan

Retourne True si les valeurs du **Jeu de colonnes** sont supérieures à la constante spécifiée ou supérieures aux valeurs correspondantes dans la colonne de comparaison.  

### <a name="greaterthanorequalto"></a>GreaterThanOrEqualTo

Retourne True si les valeurs du **Jeu de colonnes** sont supérieures ou égales à la constante spécifiée ou supérieures ou égales aux valeurs correspondantes dans la colonne de comparaison.  

### <a name="lessthan"></a>LessThan

Retourne True si les valeurs du **Jeu de colonnes** sont inférieures à la constante spécifiée ou inférieures aux valeurs correspondantes dans la colonne de comparaison.  
  
### <a name="lessthanorequalto"></a>LessThanOrEqualTo

Retourne True si les valeurs du **Jeu de colonnes** sont inférieures ou égales à la constante spécifiée ou inférieures ou égales aux valeurs correspondantes dans la colonne de comparaison.  

### <a name="notequalto"></a>NotEqualTo

Renvoie True si les valeurs figurant dans **Jeu de colonnes** ne sont pas égales à la constante ou à la colonne de comparaison, et False dans le cas contraire.  

### <a name="pairmax"></a>PairMax

Renvoie la valeur supérieure entre celle figurant dans **Jeu de colonnes** ou celle figurant dans la colonne de constantes ou de comparaison.  

### <a name="pairmin"></a>PairMin

Retourne la valeur inférieure : la valeur du **Jeu de colonnes** ou la valeur de la constante ou de la colonne de comparaison  
  
##  <a name="arithmetic-operations"></a>Opérations arithmétiques   

Comprend les opérations arithmétiques de base : addition et soustraction, division et multiplication.  Comme la plupart des opérations sont binaires (nécessitent deux nombres), vous devez d’abord choisir l’opération, puis choisir la colonne ou les nombres à utiliser dans les premier et deuxième arguments.

L’ordre dans lequel vous choisissez les colonnes pour la division et la soustraction peut paraître contre-intuitif, toutefois, pour faciliter la compréhension des résultats, l’en-tête de colonne fournit le nom de l’opération ainsi que l’ordre dans lequel les colonnes ont été utilisées.

Opération|Num1|Num2|Colonne de résultats|Valeur de résultat|
----|----|----|----|----
|Addition|1|5|Add(Num2_Num1)| 4|
|Multiplication|1|5|Multiple(Num2_Num1)|5|
|Soustraction|1|5|Subtract(Num2_Num1)|4|
|Soustraction|0|1|Subtract(Num2_Num1)|0|
|Division|1|5|Divide(Num2_Num1)|5|
|Division|0|1|Divide(Num2_Num1)|Infini|

### <a name="add"></a>Ajouter

Spécifiez les colonnes sources à l’aide du **Jeu de colonnes**, puis ajoutez à ces valeurs un nombre spécifié dans **Deuxième argument**.  

Pour ajouter les valeurs dans deux colonnes, choisissez une ou plusieurs colonnes à l’aide du **Jeu de colonnes**, puis choisissez une deuxième colonne en utilisant **Deuxième argument**.  

### <a name="divide"></a>Diviser

Divise les valeurs du **Jeu de colonnes** par une constante ou par les valeurs de colonne définies dans **Deuxième argument**.  En d’autres termes, vous choisissez d’abord le diviseur, puis le dividende. La valeur de sortie est le quotient.

### <a name="multiply"></a>Multiplier

Multiplie les valeurs figurant dans le **Jeu de colonnes** par la constante ou les valeurs de colonne spécifiées.  

### <a name="subtract"></a>Soustraire

Spécifiez la colonne de valeurs sur laquelle effectuer l’opération (le *diminuende*), en choisissant une autre colonne, à l’aide de l’option **Jeu de colonnes**. Ensuite, spécifiez le nombre à soustraire (le *diminuteur*) à l’aide de la liste déroulante **Deuxième argument**. Vous pouvez choisir une constante ou une colonne de valeurs.

##  <a name="rounding-operations"></a>Opérations d’arrondi 

Le concepteur Azure Machine Learning prend en charge diverses opérations d’arrondi. Pour de nombreuses opérations, vous devez spécifier le degré de précision à utiliser pour l’arrondi. Vous pouvez utiliser un niveau de précision statique, spécifié comme une constante, ou appliquer une valeur de précision dynamique obtenue à partir d’une colonne de valeurs.  

- Si vous utilisez une constante, définissez **Type de précision** sur **Constante**, puis tapez le nombre de chiffres (un entier) dans la zone de texte **Précision de la constante**. Si vous tapez un nombre qui n’est pas un entier, le module ne génère pas d’erreur, mais les résultats peuvent être inattendus.  

- Pour utiliser une valeur de précision différente pour chaque ligne de votre jeu de données, définissez **Type de précision** sur **ColumnSet**, puis choisissez la colonne qui contient les valeurs de précision appropriées.  

### <a name="ceiling"></a>Ceiling

Renvoie la valeur plafond des valeurs figurant dans le **jeu de colonnes**.  

### <a name="ceilingpower2"></a>CeilingPower2

Renvoie la valeur plafond élevée au carré des valeurs figurant dans le **jeu de colonnes**.  

### <a name="floor"></a>Floor

Renvoie la valeur plancher des valeurs figurant dans le **jeu de colonnes**, selon la précision spécifiée.  

### <a name="mod"></a>Mod

Renvoie la partie fractionnaire des valeurs figurant dans le **jeu de colonnes**, selon la précision spécifiée.  

### <a name="quotient"></a>Quotient

Renvoie la partie fractionnaire des valeurs figurant dans le **jeu de colonnes**, selon la précision spécifiée.  

### <a name="remainder"></a>Reste

Renvoie le reste des valeurs figurant dans le **jeu de colonnes**.  

### <a name="rounddigits"></a>RoundDigits

Renvoie les valeurs figurant dans le **jeu de colonnes**, arrondies selon la règle des 4/5 au nombre de chiffres spécifié.  

### <a name="rounddown"></a>RoundDown

Renvoie les valeurs figurant dans le **jeu de colonnes**, arrondies à la valeur inférieure comportant le nombre de chiffres spécifié.  

### <a name="roundup"></a>RoundUp

Renvoie les valeurs figurant dans le **jeu de colonnes**, arrondies à la valeur supérieure comportant le nombre de chiffres spécifié.  

### <a name="toeven"></a>ToEven

Renvoie les valeurs figurant dans **Jeu de colonnes**, arrondies au nombre pair entier le plus proche.  

### <a name="toodd"></a>ToOdd

Renvoie les valeurs figurant dans **Jeu de colonnes**, arrondies au nombre impair entier le plus proche.  

### <a name="truncate"></a>Tronquer

Tronque les valeurs figurant dans **Jeu de colonnes** en supprimant tous les chiffres non autorisés par la précision spécifiée.  
  
## <a name="special-math-functions"></a>Fonctions mathématiques spéciales

Cette catégorie comprend des fonctions mathématiques spécialisées souvent utilisées dans la science des données. Sauf indication contraire, la fonction est unaire et retourne le calcul spécifié pour chaque valeur de la ou des colonnes sélectionnées.  

### <a name="beta"></a>Bêta

Retourne la valeur de la fonction bêta de Euler.  

### <a name="ellipticintegrale"></a>EllipticIntegralE
Retourne la valeur de l’intégrale elliptique incomplète.  
  

### <a name="ellipticintegralk"></a>EllipticIntegralK

Retourne la valeur de l’intégrale elliptique complète (K).  

### <a name="erf"></a>Erf

Retourne la valeur de la fonction d’erreur.  

La fonction d’erreur (également appelée fonction d’erreur de Gauss) est une fonction spéciale de la forme sigmoïde qui est utilisée en probabilités pour décrire la diffusion.  

### <a name="erfc"></a>Erfc

Retourne la valeur de la fonction d’erreur complémentaire.  

`Erfc` est défini par 1 – erf(x).  

### <a name="erfscaled"></a>ErfScaled

Retourne la valeur de la fonction d’erreur complémentaire normalisée.  

La version normalisée de la fonction d’erreur peut être utilisée pour éviter le soupassement arithmétique.  

### <a name="erfinverse"></a>ErfInverse

Retourne la valeur de la fonction `erf` inverse.  

### <a name="exponentialintegralein"></a>ExponentialIntegralEin

Retourne la valeur de l’intégrale exponentielle Ei.  

### <a name="gamma"></a>Gamma

Retourne la valeur de la fonction gamma.  

### <a name="gammaln"></a>GammaLn

Retourne la valeur du logarithme naturel de la fonction gamma.  

### <a name="gammaregularizedp"></a>GammaRegularizedP

Retourne la valeur de la fonction gamma incomplète régularisée.  

Cette fonction prend un deuxième argument, qui peut être une constante ou une colonne de valeurs.  

### <a name="gammaregularizedpinverse"></a>GammaRegularizedPInverse

Retourne la valeur de la fonction gamma incomplète régularisée inverse.  

Cette fonction prend un deuxième argument, qui peut être une constante ou une colonne de valeurs.  

### <a name="gammaregularizedq"></a>GammaRegularizedQ  

Retourne la valeur de la fonction gamma incomplète régularisée.  

Cette fonction prend un deuxième argument, qui peut être une constante ou une colonne de valeurs.  

### <a name="gammaregularizedqinverse"></a>GammaRegularizedQInverse

Retourne la valeur de la fonction gamma incomplète régularisée généralisée inverse.

Cette fonction prend un deuxième argument, qui peut être une constante ou une colonne de valeurs.  

### <a name="polygamma"></a>PolyGamma

Retourne la valeur de la fonction polygamma.  

Cette fonction prend un deuxième argument, qui peut être une constante ou une colonne de valeurs.

##  <a name="trigonometric-functions"></a>Fonctions trigonométriques 

Cette catégorie comprend la plupart des fonctions trigonométriques et trigonométriques inverses importantes. Toutes les fonctions trigonométriques sont unaires et ne nécessitent pas d’arguments supplémentaires.  

### <a name="acos"></a>Acos

Calcule l’arc cosinus des valeurs de colonne.  

### <a name="acosdegree"></a>AcosDegree

Calcule l’arc cosinus des valeurs de colonne, en degrés.  

### <a name="acosh"></a>Acosh

Calcule l’arc cosinus hyperbolique des valeurs de colonne.  

### <a name="acot"></a>Acot

Calcule l’arc cotangente des valeurs de colonne.  

### <a name="acotdegrees"></a>AcotDegrees

Calcule l’arc cotangente des valeurs de colonne, en degrés.  

### <a name="acoth"></a>Acoth

Calcule l’arc cotangente hyperbolique des valeurs de colonne.  

### <a name="acsc"></a>Acsc

Calcule l’arc cosécante des valeurs de colonne.  

### <a name="acscdegrees"></a>AcscDegrees

Calcule l’arc cosécante des valeurs de colonne, en degrés.  
### <a name="asec"></a>Asec

Calcule l’arc sécante des valeurs de colonne.  

### <a name="asecdegrees"></a>AsecDegrees

Calcule l’arc sécante des valeurs de colonne, en degrés.  

### <a name="asech"></a>Asech

Calcule l’arc sécante hyperbolique des valeurs de colonne.  

### <a name="asin"></a>Asin

Calcule l’arc sinus des valeurs de colonne.  

### <a name="asindegrees"></a>AsinDegrees

Calcule l’arc sinus des valeurs de colonne, en degrés.  

### <a name="asinh"></a>Asinh

Calcule l’arc sinus hyperbolique des valeurs de colonne.  

### <a name="atan"></a>Atan

Calcule l’arc tangente des valeurs de colonne.  

### <a name="atandegrees"></a>AtanDegrees

Calcule l’arc tangente des valeurs de colonne, en degrés.  

### <a name="atanh"></a>Atanh

Calcule l’arc tangente hyperbolique des valeurs de colonne.  

### <a name="cos"></a>Cos

Calcule le cosinus des valeurs de colonne.  

### <a name="cosdegrees"></a>CosDegrees

Calcule le cosinus des valeurs de colonne, en degrés.  

### <a name="cosh"></a>cosh

Calcule le cosinus hyperbolique des valeurs de colonne.  

### <a name="cot"></a>Cot

Calcule la cotangente des valeurs de colonne.  

### <a name="cotdegrees"></a>CotDegrees

Calcule la cotangente des valeurs de colonne, en degrés.  

### <a name="coth"></a>Coth
Calcule la cotangente hyperbolique des valeurs de colonne.  

### <a name="csc"></a>Csc

Calcule la cosécante des valeurs de colonne.  

### <a name="cscdegrees"></a>CscDegrees

Calcule la cosécante des valeurs de colonne, en degrés.  

### <a name="csch"></a>Csch

Calcule la cosécante hyperbolique des valeurs de colonne.  

### <a name="degreestoradians"></a>DegreesToRadians

Convertit les degrés en radians.  

### <a name="sec"></a>Sec

Calcule la sécante des valeurs de colonne.  

### <a name="asecdegrees"></a>aSecDegrees

Calcule la sécante des valeurs de colonne, en degrés.  

### <a name="asech"></a>aSech

Calcule la sécante hyperbolique des valeurs de colonne.  

### <a name="sign"></a>Signer

Retourne le signe des valeurs de colonne.  

### <a name="sin"></a>Sin

Calcule le sinus des valeurs de colonne.  

### <a name="sinc"></a>Sinc

Calcule la valeur sinus-cosinus des valeurs de colonne.  

### <a name="sindegrees"></a>SinDegrees

Calcule le sinus des valeurs de colonne, en degrés.  

### <a name="sinh"></a>Sinh

Calcule le sinus hyperbolique des valeurs de colonne.  

### <a name="tan"></a>Tan

Calcule la tangente des valeurs de colonne.  

### <a name="tandegrees"></a>TanDegrees

Calcule la tangente de l’argument, en degrés.  

### <a name="tanh"></a>Tanh

Calcule la tangente hyperbolique des valeurs de colonne.  
  
## <a name="technical-notes"></a>Notes techniques

Soyez prudent quand vous sélectionnez plusieurs colonnes comme deuxième opérateur. Les résultats sont faciles à comprendre si l’opération est simple, par exemple, ajouter une constante à toutes les colonnes. 

Supposons que votre jeu de données comporte plusieurs colonnes et que vous ajoutez le jeu de données à lui-même. Dans les résultats, chaque colonne est ajoutée à elle-même, comme suit :  
  
|Num1|Num2|Num3|Add(Num1_Num1)|Add(Num2_Num2)|Add(Num3_Num3)|
|----|----|----|----|----|----|
|1|5|2|2|10|4|
|2|3|-1|4|6|-2|
|0|1|-1|0|2|-2|

Si vous devez effectuer des calculs plus complexes, vous pouvez chaîner plusieurs instances du module **Appliquer une opération mathématique**. Par exemple, vous pouvez ajouter deux colonnes en utilisant une instance du module **Appliquer une opération mathématique**, puis utiliser une autre instance du module **Appliquer une opération mathématique** pour diviser la somme par une constante afin d’obtenir la moyenne.  
  
Vous pouvez aussi utiliser l’un des modules suivants pour effectuer tous les calculs à la fois, à l’aide d’un script SQL, R ou Python :
 
+ [Exécuter un script R](execute-r-script.md)
+ [Exécuter un script Python](execute-python-script.md)
+ [Appliquer une transformation SQL](apply-sql-transformation.md)   
  
## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 

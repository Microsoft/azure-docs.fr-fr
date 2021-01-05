---
title: 'ML Studio (classique) : Créer et déployer des modules R personnalisés - Azure'
description: Découvrez comment créer et déployer un module R personnalisé dans ML Studio (classique).
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18
ms.date: 11/29/2017
ms.openlocfilehash: ec6a3304ffe035e7ac206e96f7666e3ba1877d9e
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93322781"
---
# <a name="define-custom-r-modules-for-machine-learning-studio-classic"></a>Définir des modules R personnalisés pour Machine Learning Studio (classique)

**S’APPLIQUE À :**  ![S’applique à ](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Ne s’applique pas à ](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)

Cette rubrique explique comment créer et déployer un module R Studio personnalisé (classique). Elle explique ce qu’est un module R personnalisé, en détaillant les fichiers utilisés pour le définir. Par ailleurs, elle illustre la construction de ces fichiers et l’inscription du module à des fins de déploiement dans un espace de travail Machine Learning. Les éléments et attributs utilisés dans la définition du module personnalisé sont ensuite décrits plus en détail. Par ailleurs, nous allons découvrir comment utiliser les fichiers et la fonctionnalité auxiliaires, ainsi que les sorties multiples. 

Un **module personnalisé** est un module défini par l'utilisateur, qui peut être chargé dans votre espace de travail et exécuté dans le cadre d'une expérience Azure Machine Learning Studio (classique). Un **module R personnalisé** est un module exécutant une fonction R définie par l’utilisateur. **R** est un langage de programmation utilisé pour le traitement informatique des statistiques et les graphiques. Les chercheurs de données et les statisticiens s’en servent pour implémenter des algorithmes. Actuellement, R est le seul langage pris en charge dans les modules personnalisés, mais la prise en charge de langages supplémentaires est prévue pour les versions à venir.

Les modules personnalisés présentent un **état de première classe** dans Azure Machine Learning Studio (classique), en ce sens qu'ils peuvent être utilisés comme n'importe quel autre module. Ils peuvent être exécutés avec d’autres modules, inclus dans des expériences publiées ou dans des visualisations. Vous contrôlez l’algorithme implémenté par le module, les ports d’entrée et de sortie à utiliser, les paramètres de modélisation et divers autres comportements d’exécution. Une expérience qui contient des modules personnalisés peut également être publiée dans Azure AI Gallery pour faciliter le partage.

## <a name="files-in-a-custom-r-module"></a>Fichiers dans un module R personnalisé
Un module R personnalisé est défini par un fichier .zip contenant au moins deux fichiers :

* un **fichier source** qui implémente la fonction R exposée par le module ;
* un **fichier de définition XML** qui décrit l’interface du module personnalisé.

Des fichiers auxiliaires supplémentaires peuvent également être inclus dans le fichier .zip, qui fournit des fonctionnalités accessibles à partir du module personnalisé. Cette option est décrite dans la partie **Arguments** de la section de référence **Éléments dans le fichier de définition XML** suivant l’exemple de démarrage rapide.

## <a name="quickstart-example-define-package-and-register-a-custom-r-module"></a>Exemple de démarrage rapide : définir, mettre en package et enregistrer un module R personnalisé
Cet exemple explique comment créer les fichiers requis par un module R personnalisé, les empaqueter dans un fichier .zip, puis enregistrer le module dans l’espace de travail Machine Learning. Les exemples de fichiers et de package zip sont disponibles en téléchargement depuis le [fichier CustomAddRows.zip](https://go.microsoft.com/fwlink/?LinkID=524916&clcid=0x409).

## <a name="the-source-file"></a>Le fichier source
Prenons l’exemple d’un module **Custom Add Rows** , qui modifie l’implémentation standard du module **Add Rows** utilisé pour concaténer des lignes (observations) à partir de deux jeux de données (trames de données). Le module **Add Rows** standard ajoute les lignes du deuxième jeu de données d’entrée à la fin du premier, au moyen de l’algorithme `rbind`. De la même manière, la fonction `CustomAddRows` personnalisée accepte deux jeux de données, mais également un paramètre d’échange booléen en entrée supplémentaire. Si le paramètre d’échange a la valeur **FALSE** , il renvoie le même jeu de données que l’implémentation standard. Par contre, s’il a la valeur **TRUE** , la fonction ajoute des lignes du premier jeu de données d’entrée à la fin du deuxième jeu de données. Le fichier CustomAddRows.R qui contient l’implémentation de la fonction R `CustomAddRows` exposée par le module **Custom Add Rows** contient le code R suivant.

```r
CustomAddRows <- function(dataset1, dataset2, swap=FALSE) 
{
    if (swap)
    {
        return (rbind(dataset2, dataset1));
    }
    else
    {
        return (rbind(dataset1, dataset2));
    } 
} 
```

### <a name="the-xml-definition-file"></a>Le fichier de définition XML
Pour exposer la fonction `CustomAddRows` en tant que module Azure Machine Learning Studio (classique), vous devez créer un fichier de définition XML afin de définir l'apparence du module **Custom Add Rows** et son comportement. 

```xml
<!-- Defined a module using an R Script -->
<Module name="Custom Add Rows">
    <Owner>Microsoft Corporation</Owner>
    <Description>Appends one dataset to another. Dataset 2 is concatenated to Dataset 1 when Swap is FALSE, and vice versa when Swap is TRUE.</Description>

<!-- Specify the base language, script file and R function to use for this module. -->        
    <Language name="R" 
        sourceFile="CustomAddRows.R" 
        entryPoint="CustomAddRows" />  

<!-- Define module input and output ports -->
<!-- Note: The values of the id attributes in the Input and Arg elements must match the parameter names in the R Function CustomAddRows defined in CustomAddRows.R. -->
    <Ports>
        <Input id="dataset1" name="Dataset 1" type="DataTable">
            <Description>First input dataset</Description>
        </Input>
        <Input id="dataset2" name="Dataset 2" type="DataTable">
            <Description>Second input dataset</Description>
        </Input>
        <Output id="dataset" name="Dataset" type="DataTable">
            <Description>The combined dataset</Description>
        </Output>
    </Ports>

<!-- Define module parameters -->
    <Arguments>
        <Arg id="swap" name="Swap" type="bool" >
            <Description>Swap input datasets.</Description>
        </Arg>
    </Arguments>
</Module>
```

Il est essentiel de noter que la valeur des attributs **id** des éléments **Input** et **Arg** dans le fichier XML doit correspondre aux noms des paramètres de la fonction du code R dans le fichier CustomAddRows.R À L’EXACT ( *dataset1* , *dataset2* et *swap* dans l’exemple). De même, la valeur de l’attribut **entryPoint** de l’élément **Language** doit correspondre au nom de la fonction dans le script R À L’EXACT ( *CustomAddRows* dans l’exemple). 

En revanche, l’attribut **id** de l’élément **Output** ne correspond à aucune variable du script R. Lorsque plusieurs sorties sont requises, il suffit de renvoyer une liste à partir de la fonction R avec les résultats placés *dans le même ordre* que celui dans lequel les éléments **Sorties** sont déclarés dans le fichier XML.

### <a name="package-and-register-the-module"></a>Empaqueter et inscrire le module
Enregistrez ces deux fichiers sous *CustomAddRows.R* et *CustomAddRows.xml* , puis compressez les deux fichiers ensemble dans un fichier *CustomAddRows.zip*.

Pour les enregistrer dans votre espace de travail Machine Learning, accédez à votre espace de travail dans Azure Machine Learning Studio (classique), cliquez sur le bouton **+NOUVEAU** situé dans la partie inférieure de la fenêtre et choisissez **MODULE -> À PARTIR DU PACKAGE ZIP** pour charger le nouveau module **Custom Add Rows**.

![Charger les zip](./media/custom-r-modules/upload-from-zip-package.png)

Le module **Custom Add Rows** peut désormais être ouvert par vos expériences Machine Learning.

## <a name="elements-in-the-xml-definition-file"></a>Éléments dans le fichier de définition XML
### <a name="module-elements"></a>Éléments de module
L’élément **Module** est utilisé pour définir un module personnalisé dans le fichier XML. Plusieurs modules peuvent être définis dans un fichier XML à l’aide de plusieurs éléments **module** . Chaque module de votre espace de travail doit avoir un nom unique. Si vous enregistrez un module personnalisé avec le même nom qu’un module personnalisé existant, le module existant est remplacé par le nouveau module. Les modules personnalisés peuvent, toutefois, être inscrits avec le même nom qu’un module Azure Machine Learning Studio (classique) existant. Dans ce cas, ils apparaissent dans la catégorie **Personnalisé** de la palette de modules.

```xml
<Module name="Custom Add Rows" isDeterministic="false"> 
    <Owner>Microsoft Corporation</Owner>
    <Description>Appends one dataset to another...</Description>/> 
```

Dans l’élément **Module** , vous pouvez spécifier deux éléments facultatifs supplémentaires :

* un élément **Propriétaire** qui est incorporé dans le module  
* un élément **Description** qui contient le texte affiché dans l’aide rapide du module et lorsque vous passez la souris sur le module dans l’interface utilisateur Machine Learning.

Règles relatives aux limites de caractères dans les éléments Module :

* La valeur de l’attribut **name** dans l’élément **Module** ne doit pas dépasser 64 caractères. 
* Le contenu de l’élément **Description** ne doit pas dépasser 128 caractères.
* Le contenu de l’élément **Owner** ne doit pas dépasser 32 caractères.

Les résultats d’un module peuvent être déterministes ou non déterministes.** Par défaut, tous les modules sont considérés comme déterministes. Autrement dit, en présence d’un jeu de données et de paramètres d’entrée qui ne change pas, le module doit retourner les mêmes résultats chaque fois qu’il est exécuté. Étant donné ce comportement, Azure Machine Learning Studio (classique) ne réexécute les modules marqués comme étant déterministes que si un paramètre ou les données d'entrée ont été modifiés. Le renvoi des résultats mis en cache assure également une exécution bien plus rapide des expériences.

Il existe des fonctions non déterministes, notamment RAND ou une fonction qui retourne la date ou l’heure actuelle. Si votre module utilise une fonction non déterministe, vous pouvez spécifier que le module n’est pas déterministe en définissant l’attribut facultatif **isDeterministic** sur **FALSE**. Cela permet de s’assurer que le module est réexécuté à chaque exécution de l’expérience, même si le module d’entrée et les paramètres n’ont pas changé. 

### <a name="language-definition"></a>Définition de l’élément Language
L’élément **Language** dans votre fichier de définition XML est utilisé pour spécifier la langue du module personnalisé. Actuellement, R est le seul langage pris en charge. La valeur de l’attribut **sourceFile** doit être le nom du fichier R qui contient la fonction à appeler lorsque le module est exécuté. Ce fichier doit faire partie du package zip. La valeur de l’attribut **entryPoint** est le nom de la fonction appelée et doit correspondre à une fonction valide définie dans le fichier source.

```xml
<Language name="R" sourceFile="CustomAddRows.R" entryPoint="CustomAddRows" />
```

### <a name="ports"></a>Ports
Les ports d’entrée et de sortie d’un module personnalisé sont spécifiés dans les éléments enfants de la section **Ports** du fichier de définition XML. L’ordre de ces éléments détermine la disposition rencontrée par les utilisateurs (expérience utilisateur). Le premier enfant **input** ou **output** répertorié dans l’élément **Ports** du fichier XML devient le port d’entrée le plus à gauche dans l’expérience utilisateur Machine Learning.
Chaque port d’entrée et de sortie peut avoir un élément enfant **Description** facultatif qui spécifie le texte affiché lorsque vous passez le curseur de la souris sur le port dans l’interface utilisateur de Machine Learning.

**Règles relatives aux ports** :

* Le nombre maximum de **ports d’entrée et de sortie** est de 8 pour chacun.

### <a name="input-elements"></a>Éléments d'entrée
Les ports d’entrée vous permettent de transmettre des données à votre fonction R et à votre espace de travail. Les **types de données** pris en charge pour les ports d’entrée sont les suivants : 

**DataTable :** ce type est transmis à votre fonction R sous la forme suivante : data.frame. En réalité, tous les types (par exemple, des fichiers CSV ou des fichiers ARFF) pris en charge par Machine Learning et compatibles avec **DataTable** sont automatiquement convertis en data.frame. 

```xml
<Input id="dataset1" name="Input 1" type="DataTable" isOptional="false">
    <Description>Input Dataset 1</Description>
</Input>
```

L’attribut **id** associé à chaque port d’entrée **DataTable** doit avoir une valeur unique qui doit correspondre au paramètre nommé correspondant dans votre fonction R.
Pour les ports **DataTable** facultatifs qui ne sont pas transmis comme entrée d’une expérience, la valeur **NULL** est transmise à la fonction R et les ports zip facultatifs sont ignorés si l’entrée n’est pas connectée. L’attribut **isOptional** est facultatif pour les types **DataTable** et **Zip**  ; il a la valeur *false* par défaut.

**Zip :** modules personnalisés pouvant accepter un fichier .zip en entrée. Cette entrée est décompactée et placée dans le répertoire de travail R de votre fonction.

```xml
<Input id="zippedData" name="Zip Input" type="Zip" IsOptional="false">
    <Description>Zip files to be extracted to the R working directory.</Description>
</Input>
```

Pour les modules R personnalisés, l’ID d’un port Zip ne doit pas nécessairement correspondre aux paramètres de la fonction R. En effet, le fichier zip est automatiquement extrait dans le répertoire de travail R.

**Règles d'entrée :**

* La valeur de l’attribut **id** de l’élément **Input** doit être un nom de variable R valide.
* La valeur de l’attribut **id** de l’élément **Input** ne doit pas dépasser 64 caractères.
* La valeur de l’attribut **name** de l’élément **Input** ne doit pas dépasser 64 caractères.
* Le contenu de l’élément **Description** ne doit pas contenir plus de 128 caractères
* La valeur de l’attribut **type** de l’élément **Input** doit être *Zip* ou *DataTable*.
* La valeur de l’attribut **isOptional** de l’élément **Input** n’est pas obligatoire (et est définie sur *false* par défaut lorsqu’il n’est pas spécifié) ; mais s’il est spécifié, la valeur doit être *true* ou *false*.

### <a name="output-elements"></a>Éléments d’entrée
**Ports de sortie standard :** les ports de sortie sont mappés aux valeurs de retour à partir de votre fonction R, qui peut ensuite être utilisée par les modules suivants. *DataTable* est le seul type de port de sortie standard pris en charge actuellement. (Les types *Learners* et *Transformers* seront prochainement pris en charge.) Une sortie *DataTable* est définie comme suit :

```xml
<Output id="dataset" name="Dataset" type="DataTable">
    <Description>Combined dataset</Description>
</Output>
```

Pour les sorties dans des modules personnalisés R, la valeur de l’attribut **id** ne doit pas forcément correspondre à un élément du script R, mais elle doit être unique. Pour une sortie de module unique, la valeur de retour de la fonction R doit être un *data.frame*. Pour créer une sortie portant sur plusieurs objets qui présentent un type de données pris en charge, vous devez spécifier les ports de sortie appropriés dans le fichier de définition XML et les objets doivent être renvoyés dans une liste. Les objets de la sortie sont affectés à des ports de sortie, de gauche à droite, selon l’ordre dans lequel les objets sont placés dans la liste renvoyée.

Par exemple, si vous souhaitez modifier le module **Custom Add Rows** pour sortir les deux jeux de données d’origine, *dataset1* et *dataset2* , en plus du nouveau jeu de données joint dataset, *dataset* (dans l’ordre suivant, de gauche à droite : *dataset* , *dataset1* , *dataset2* ), vous devez définir les ports de sortie dans le fichier CustomAddRows.xml comme suit :

```xml
<Ports> 
    <Output id="dataset" name="Dataset Out" type="DataTable"> 
        <Description>New Dataset</Description> 
    </Output> 
    <Output id="dataset1_out" name="Dataset 1 Out" type="DataTable"> 
        <Description>First Dataset</Description> 
    </Output> 
    <Output id="dataset2_out" name="Dataset 2 Out" type="DataTable"> 
        <Description>Second Dataset</Description> 
    </Output> 
    <Input id="dataset1" name="Dataset 1" type="DataTable"> 
        <Description>First Input Table</Description>
    </Input> 
    <Input id="dataset2" name="Dataset 2" type="DataTable"> 
        <Description>Second Input Table</Description> 
    </Input> 
</Ports> 
```

Ensuite, retournez la liste des objets dans une liste respectant l’ordre adéquat dans « CustomAddRows » :

```r
CustomAddRows <- function(dataset1, dataset2, swap=FALSE) { 
    if (swap) { dataset <- rbind(dataset2, dataset1)) } 
    else { dataset <- rbind(dataset1, dataset2)) 
    } 
    return (list(dataset, dataset1, dataset2)) 
} 
```

**Sortie de visualisation :** vous pouvez également spécifier un port de sortie de type *Visualization* , qui affiche la sortie de la console et de l’appareil graphique R. Ce port ne fait pas partie de la sortie de la fonction R et n’interfère pas avec l’ordre des autres types de ports de sortie. Pour ajouter un port de visualisation pour les modules personnalisés, ajoutez un élément **Output** avec la valeur *Visualization* pour son attribut **type**  :

```xml
<Output id="deviceOutput" name="View Port" type="Visualization">
    <Description>View the R console graphics device output.</Description>
</Output>
```

**Règles de sortie :**

* La valeur de l’attribut **id** de l’élément **Output** doit être un nom de variable R valide.
* La valeur de l’attribut **id** de l’élément **Output** ne doit pas dépasser 32 caractères.
* La valeur de l’attribut **name** de l’élément **Output** ne doit pas dépasser 64 caractères.
* La valeur de l’attribut **type** de l’élément **Output** doit être *Visualization*.

### <a name="arguments"></a>Arguments
Des données supplémentaires peuvent être transmises à la fonction R via les paramètres de module qui sont définis dans l’élément **Arguments** . Ces paramètres apparaissent dans le volet des propriétés de l’interface utilisateur de Machine Learning le plus à droite lorsque le module est sélectionné. Les arguments peuvent être de n’importe quel type pris en charge, ou vous pouvez créer une énumération personnalisée lorsque cela s’avère nécessaire. Comme pour les éléments **Ports** , les éléments **Arguments** peuvent avoir un élément **Description** facultatif spécifiant le texte qui apparaît lorsque vous passez la souris sur le nom du paramètre.
Les propriétés facultatives pour un module, telles que defaultValue, minValue et maxValue, peuvent être ajoutées à n’importe quel argument en tant qu’attributs d’un élément **Properties** . Les propriétés valides pour l’élément **Properties** dépendent du type d’argument et sont décrites avec les types d’arguments pris en charge dans la section suivante. Pour les arguments dont la propriété **isOptional** est définie sur **« true »** , l’utilisateur n’a pas à entrer de valeur. Si aucune valeur n’est fournie à l’argument, celui-ci n’est pas transmis à la fonction de point d’entrée. Les arguments de la fonction de point d’entrée qui sont facultatifs doivent être gérés explicitement par la fonction, par exemple se voir attribuer la valeur NULL par défaut dans la définition de fonction de point d’entrée. Un argument facultatif applique uniquement les autres contraintes d’argument, par exemple min ou max, si une valeur est fournie par l’utilisateur.
Comme dans le cas des entrées et sorties, il est essentiel que chaque paramètre soit associé à une valeur d’ID unique. Dans notre exemple de démarrage rapide, l’ID/paramètre associé était *swap*.

### <a name="arg-element"></a>Élément Arg
Un paramètre de module est défini à l’aide de l’élément enfant **Arg** de la section **Arguments** du fichier de définition XML. Comme dans le cas des éléments enfants de la section **Ports** , l’ordre des paramètres de la section  **Arguments** définit la disposition rencontrée dans l’expérience utilisateur. Les paramètres apparaissent de haut en bas dans l’interface utilisateur dans le même ordre que celui dans lequel ils sont définis dans le fichier XML. Les types pris en charge par Machine Learning pour les paramètres sont répertoriés ici. 

**int** : paramètre de type entier (32 bits).

```xml
<Arg id="intValue1" name="Int Param" type="int">
    <Properties min="0" max="100" default="0" />
    <Description>Integer Parameter</Description>
</Arg>
```

* *Propriétés facultatives*  : **min** , **max** , **default** et **isOptional**

**double** : paramètre de type double.

```xml
<Arg id="doubleValue1" name="Double Param" type="double">
    <Properties min="0.000" max="0.999" default="0.3" />
    <Description>Double Parameter</Description>
</Arg>
```

* *Propriétés facultatives*  : **min** , **max** , **default** et **isOptional**

**bool** : paramètre booléen représenté par une case à cocher dans l’expérience utilisateur.

```xml
<Arg id="boolValue1" name="Boolean Param" type="bool">
    <Properties default="true" />
    <Description>Boolean Parameter</Description>
</Arg>
```

* *Propriétés facultatives*  : **default**. False si non défini

**string** : chaîne standard

```xml
<Arg id="stringValue1" name="My string Param" type="string">
    <Properties isOptional="true" />
    <Description>String Parameter 1</Description>
</Arg>    
```

* *Propriétés facultatives*  : **default** et **isOptional**

**ColumnPicker** : paramètre de sélection de colonne. Ce type est représenté sous la forme d’un sélecteur de colonne dans l’interface utilisateur. L’élément **Property** est utilisé ici pour spécifier l’ID du port à partir duquel les colonnes sont sélectionnées, où le type de port cible doit être *DataTable*. Le résultat de la sélection des colonnes est transmis à la fonction R sous forme d’une liste de chaînes contenant les noms des colonnes sélectionnées. 

```xml
<Arg id="colset" name="Column set" type="ColumnPicker">      
    <Properties portId="datasetIn1" allowedTypes="Numeric" default="NumericAll"/>
    <Description>Column set</Description>
</Arg>
```

* *Propriétés obligatoires*  : **portId**. Correspond à l’ID d’un élément Input de type *DataTable*.
* *Propriétés facultatives* :
  
  * **allowedTypes**. Filtre les types de colonnes que vous pouvez choisir. Les valeurs valides sont les suivantes : 
    
    * Numérique
    * Booléen
    * Par catégorie
    * String
    * Étiquette
    * Fonctionnalité
    * Score
    * Tous
  * **default** : les sélections par défaut valides pour le sélecteur de colonne sont les suivantes : 
    
    * None
    * NumericFeature
    * NumericLabel
    * NumericScore
    * NumericAll
    * BooleanFeature
    * BooleanLabel
    * BooleanScore
    * BooleanAll
    * CategoricalFeature
    * CategoricalLabel
    * CategoricalScore
    * CategoricalAll
    * StringFeature
    * StringLabel
    * StringScore
    * StringAll
    * AllLabel
    * AllFeature
    * AllScore
    * Tous

**DropDown** : liste (déroulante) énumérée spécifiée par l’utilisateur. Les éléments de liste déroulante sont spécifiés dans l’élément **Properties** à l’aide de l’élément **Item**. L’ **id** de chaque **Item** doit être unique et être une variable R valide. La valeur **name** d’un **Item** est à la fois le texte affiché et la valeur transmise à la fonction R.

```xml
<Arg id="color" name="Color" type="DropDown">
    <Properties default="red">
        <Item id="red" name="Red Value"/>
        <Item id="green" name="Green Value"/>
        <Item id="blue" name="Blue Value"/>
    </Properties>
    <Description>Select a color.</Description>
</Arg>    
```

* *Propriétés facultatives* :
  * **default**  : la valeur de la propriété par défaut doit correspondre à une valeur d’ID de l’un des éléments **Item**.

### <a name="auxiliary-files"></a>Fichiers auxiliaires
Tout fichier placé dans le fichier ZIP de votre module personnalisé sera disponible pour une utilisation au moment de l’exécution. Toutes les structures de répertoire présentes sont conservées. Cela signifie que l'approvisionnement du fichier fonctionne de la même façon en local et lors de l'exécution d'Azure Machine Learning Studio (classique). 

> [!NOTE]
> Notez que tous les fichiers sont extraits vers le répertoire « src ». Tous les chemins doivent donc comporter un préfixe « src/ ».
> 
> 

Supposons que vous souhaitiez supprimer du jeu de données toutes les lignes présentant la mention « NA » et toutes les lignes en double, avant de créer une sortie dans CustomAddRows, et que vous avez déjà créé une fonction R qui effectue cette opération dans un fichier, RemoveDupNARows.R :

```r
RemoveDupNARows <- function(dataFrame) {
    #Remove Duplicate Rows:
    dataFrame <- unique(dataFrame)
    #Remove Rows with NAs:
    finalDataFrame <- dataFrame[complete.cases(dataFrame),]
    return(finalDataFrame)
}
```

Vous pouvez approvisionner le fichier auxiliaire RemoveDupNARows.R dans la fonction myAddRows :

```r
CustomAddRows <- function(dataset1, dataset2, swap=FALSE) {
    source("src/RemoveDupNARows.R")
        if (swap) { 
            dataset <- rbind(dataset2, dataset1))
        } else { 
            dataset <- rbind(dataset1, dataset2)) 
        } 
    dataset <- removeDupNARows(dataset)
    return (dataset)
}
```

Ensuite, chargez un fichier zip contenant les éléments « CustomAddRows.R », « CustomAddRows.xml » et « RemoveDupNARows.R » en tant que module R personnalisé.

## <a name="execution-environment"></a>Environnement d’exécution
L’environnement d’exécution du script R utilise la même version de R que le module **Exécuter le script R** et vous pouvez utiliser les mêmes packages par défaut. Vous pouvez également ajouter des packages R supplémentaires à votre module personnalisé en les incluant dans le package zip du module personnalisé. Chargez-les simplement dans votre script R comme vous le feriez dans votre propre environnement R. 

**limitations de l’environnement d’exécution** sont les suivantes :

* Système de fichiers non persistant : les fichiers écrits lorsque le module personnalisé est exécuté ne sont pas persistants sur plusieurs exécutions du même module.
* Pas d’accès réseau.
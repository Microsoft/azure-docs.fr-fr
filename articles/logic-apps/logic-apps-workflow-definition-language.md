---
title: "Schéma du langage de définition du flux de travail - Azure Logic Apps | Microsoft Docs"
description: "Définir des flux de travail basés sur le schéma du langage de définition du flux de travail pour Azure Logic Apps"
services: logic-apps
author: jeffhollan
manager: anneta
editor: 
documentationcenter: 
ms.assetid: 26c94308-aa0d-4730-97b6-de848bffff91
ms.service: logic-apps
ms.workload: integration
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 03/21/2017
ms.author: jehollan
translationtype: Human Translation
ms.sourcegitcommit: 6e0ad6b5bec11c5197dd7bded64168a1b8cc2fdd
ms.openlocfilehash: 09ccdaa18d383569747612c33b63595c06a0131e
ms.lasthandoff: 03/28/2017

---

# <a name="workflow-definition-language-schema-for-azure-logic-apps"></a>Schéma du langage de définition du flux de travail pour Azure Logic Apps

Une définition de flux de travail contient la logique réelle qui s’exécute dans le cadre de votre application logique. Cette définition inclut un ou plusieurs déclencheurs qui démarrent l’application logique, et une ou plusieurs actions à entreprendre pour l’application logique.  
  
## <a name="basic-workflow-definition-structure"></a>Structure de la définition de flux de travail de base

Voici la structure de base d’une définition de flux de travail :  
  
```json
{
    "$schema": "<schema-of the-definition>",
    "contentVersion": "<version-number-of-definition>",
    "parameters": { <parameter-definitions-of-definition> },
    "triggers": [ { <definition-of-flow-triggers> } ],
    "actions": [ { <definition-of-flow-actions> } ],
    "outputs": { <output-of-definition> }
}
```
  
> [!NOTE]
> Le document sur [l’API REST de gestion du flux de travail](https://docs.microsoft.com/rest/api/logic/workflows) contient des informations sur la création et la gestion des flux de travail d’application logique.
  
|Nom de l'élément|Requis|Description|  
|------------------|--------------|-----------------|  
|$schema|Non|Spécifie l’emplacement du fichier de schéma JSON qui décrit la version du langage de définition. Cet emplacement est requis si vous référencez une définition en externe. Pour ce document, l’emplacement est le suivant : <p>`https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2015-08-01-preview/workflowdefinition.json#`|  
|contentVersion|Non|Spécifie la version de la définition. Lorsque vous déployez un flux de travail à l’aide de la définition, vous pouvez utiliser cette valeur pour vous assurer que la définition appropriée est utilisée.|  
|parameters|Non|Spécifie les paramètres utilisés pour entrer des données dans la définition. Il est possible de définir 50 paramètres maximum.|  
|Déclencheurs|Non|Spécifie des informations pour les déclencheurs qui lancent le flux de travail. Il est possible de définir 250 déclencheurs au maximum.|  
|actions|Non|Spécifie les actions qui sont effectuées pendant l’exécution du flux. Il est possible de définir 250 actions au maximum.|  
|outputs|Non|Spécifie des informations sur la ressource déployée. Il est possible de définir 10 sorties au maximum.|  
  
## <a name="parameters"></a>parameters

Cette section spécifie tous les paramètres utilisés dans la définition de flux de travail au moment du déploiement. Tous les paramètres doivent être déclarés dans cette section pour pouvoir être utilisés dans d’autres sections de la définition.  
  
L’exemple suivant illustre la structure d’une définition de sortie :  

```json
"parameters": {
    "<parameter-name>" : {
        "type" : "<type-of-parameter-value>",
        "defaultValue": <default-value-of-parameter>,
        "allowedValues": [ <array-of-allowed-values> ],
        "metadata" : { "key": { "name": "value"} }
    }
}
```

|Nom de l'élément|Requis|Description|  
|------------------|--------------|-----------------|  
|type|Oui|**Type** : string <p> **Déclaration** : `"parameters": {"parameter1": {"type": "string"}` <p> **Spécification** : `"parameters": {"parameter1": {"value": "myparamvalue1"}}` <p> **Type** : securestring <p> **Déclaration** : `"parameters": {"parameter1": {"type": "securestring"}}` <p> **Spécification** : `"parameters": {"parameter1": {"value": "myparamvalue1"}}` <p> **Type** : int <p> **Déclaration** : `"parameters": {"parameter1": {"type": "int"}}` <p> **Spécification** : `"parameters": {"parameter1": {"value" : 5}}` <p> **Type** : bool <p> **Déclaration** : `"parameters": {"parameter1": {"type": "array"}}` <p> **Spécification** : `"parameters": {"parameter1": { "value": true }}` <p> **Type** : array <p> **Déclaration** : `"parameters": {"parameter1": {"type": "array"}}` <p> **Spécification** : `"parameters": {"parameter1": { "value": [ array-of-values ]}}` <p> **Type** : object <p> **Déclaration** : `"parameters": {"parameter1": {"type": "object"}}` <p> **Spécification** : `"parameters": {"parameter1": { "value": { JSON-object } }}` <p> **Type** : secureobject <p> **Déclaration** : `"parameters": {"parameter1": {"type": "object"}}` <p> **Spécification** : `"parameters": {"parameter1": { "value": { JSON-object } }}` <p> **Remarque :** les types `securestring` et `secureobject` ne sont pas retournés dans les opérations `GET`. L’ensemble des mots de passe, clés et secrets doivent utiliser ce type.|  
|defaultValue|Non|Spécifie la valeur par défaut du paramètre lorsque aucune valeur n’est spécifiée à la création de la ressource.|  
|allowedValues|Non|Spécifie un tableau de valeurs autorisées pour le paramètre.|  
|métadonnées|Non|Spécifie des informations supplémentaires sur le paramètre, comme une description lisible ou des données design-time utilisées par Visual Studio ou d’autres outils.|  
  
Cet exemple montre comment vous pouvez utiliser un paramètre dans la section du corps d’une action :  
  
```json
"body" :
{
  "property1": "@parameters('parameter1')"
}
```

 Les paramètres peuvent également être utilisés dans les sorties.  
  
## <a name="triggers-and-actions"></a>Déclencheurs et actions  

Les déclencheurs et actions spécifient les appels qui peuvent participer à l’exécution de flux de travail. Pour plus d’informations sur cette section, consultez la page [Actions et déclencheurs de flux de travail](logic-apps-workflow-actions-triggers.md).
  
## <a name="outputs"></a>Sorties  

Les sorties spécifient les informations qui peuvent être retournées à partir de l’exécution d’un flux de travail. Par exemple, si vous souhaitez suivre un état ou une valeur spécifique pour chaque exécution, vous pouvez inclure ces données dans les sorties d’exécution. Les données apparaissent dans l’API REST de gestion pour cette exécution et dans l’interface utilisateur de gestion pour cette exécution dans le portail Azure. Vous pouvez également diriger ces sorties vers d’autres systèmes externes tels que Power BI pour créer des tableaux de bord. Les sorties ne sont *pas* utilisées pour répondre aux requêtes entrantes sur l’API REST du service. Voici un exemple pour répondre à une requête entrante à l’aide du type d’action `response` :
  
```json
"outputs": {  
  "key1": {  
    "value": "value1",  
    "type" : "<type-of-value>"  
  }  
} 
```

|Nom de l'élément|Requis|Description|  
|------------------|--------------|-----------------|  
|key1|Oui|Spécifie l’identificateur de clé pour la sortie. Remplacez **key1** par le nom que vous souhaitez utiliser pour identifier la sortie.|  
|value|Oui|Spécifie la valeur de la sortie.|  
|type|Oui|Spécifie le type de la valeur qui a été spécifiée. Types de valeur possibles : <p>- `string`<br />- `securestring`<br />- `int`<br />- `bool`<br />- `array`<br />- `object`|
  
## <a name="expressions"></a>Expressions  

Les valeurs JSON indiquées dans la définition peuvent être littérales. Il peut également s’agir d’expressions qui sont évaluées lorsque la définition est utilisée. Par exemple :  
  
```json
"name": "value"
```

 ou  
  
```json
"name": "@parameters('password') "
```

> [!NOTE]
> Certaines expressions obtiennent leurs valeurs à partir d’actions runtime qui peuvent ne pas exister au début de l’exécution. Vous pouvez utiliser des **fonctions** pour récupérer certaines de ces valeurs.  
  
Les expressions peuvent apparaître n’importe où dans une valeur de chaîne JSON, et entraînent toujours une autre valeur JSON. Si une valeur JSON a été déterminée pour être une expression, le corps de l’expression est extrait en supprimant l’arobase (@). Si une chaîne littérale devant commencer par @ est nécessaire, elle doit être placée dans une séquence d’échappement en utilisant @@. Les exemples suivants montrent comment les expressions sont évaluées.  
  
|Valeur JSON|Résultat|  
|----------------|------------|  
|« parameters »|Les caractères « parameters » sont retournés.|  
|« parameters[1] »|Les caractères « parameters[1] » sont retournés.|  
|« @@ »|Une chaîne de 1 caractère contenant « @ » est retournée.|  
|«  @ »|Une chaîne de 2 caractères contenant «  @ » est retournée.|  
  
Avec l’*interpolation de chaîne*, les expressions peuvent également apparaître dans des chaînes où les expressions sont encapsulées dans `@{ ... }`. Par exemple : <p>`"name" : "First Name: @{parameters('firstName')} Last Name: @{parameters('lastName')}"`

Le résultat est toujours une chaîne, ce qui rend cette fonctionnalité semblable à la fonction `concat`. Supposons que vous avez défini `myNumber` sur `42` et `myString` sur `sampleString` :  
  
|Valeur JSON|Résultat|  
|----------------|------------|  
|« @parameters('myString') »|Retourne `sampleString` en tant que chaîne.|  
|« @{parameters('myString')} »|Retourne `sampleString` en tant que chaîne.|  
|« @parameters('myNumber') »|Retourne `42` en tant que *nombre*.|  
|« @{parameters('myNumber')} »|Retourne `42` en tant que *chaîne*.|  
|« Answer is: @{parameters('myNumber')} »|Retourne la chaîne `Answer is: 42`.|  
|« @concat('Answer is: ', string(parameters('myNumber'))) »|Retourne la chaîne `Answer is: 42`.|  
|« Answer is: @@{parameters('myNumber')} »|Retourne la chaîne `Answer is: @{parameters('myNumber')}`.|  
  
## <a name="operators"></a>Opérateurs  

Les opérateurs sont les caractères que vous pouvez utiliser dans des expressions ou fonctions. 
  
|opérateur|Description|  
|--------------|-----------------|  
|.|L’opérateur point vous permet de référencer les propriétés d’un objet.|  
|?|L’opérateur point d’interrogation vous permet de référencer les propriétés Null d’un objet sans erreur d’exécution. Par exemple, vous pouvez utiliser cette expression pour gérer les sorties de déclencheur Null : <p>`@coalesce(trigger().outputs?.body?.property1, 'my default value')`|  
|'|Le guillemet simple est le seul moyen d’encapsuler des littéraux de chaîne. Vous ne pouvez pas utiliser de guillemets doubles dans les expressions, car cette ponctuation est en conflit avec l’apostrophe JSON qui encapsule l’expression entière.|  
|[]|Les crochets permettent d’obtenir une valeur d’un tableau avec un index spécifique. Par exemple, si l’une de vos actions transmet `range(0,10)` à la fonction `forEach`, vous pouvez utiliser cette fonction pour obtenir des éléments des tableaux :  <p>`myArray[item()]`|  
  
## <a name="functions"></a>Fonctions  

Vous pouvez également appeler des fonctions dans des expressions. Le tableau suivant présente les fonctions qui peuvent être utilisées dans une expression.  
  
|Expression|Évaluation|  
|----------------|----------------|  
|« @function('Hello') »|Appelle la fonction membre de la définition avec la chaîne littérale Hello comme premier paramètre.|  
|« @function('C’est cool !') »|Appelle la fonction membre de la définition avec la chaîne littérale C’est cool comme premier paramètre.|  
|« @function().prop1 »|Retourne la valeur de la propriété prop1 du membre `myfunction` de la définition.|  
|« @function('Hello').prop1 »|Appelle la fonction membre de la définition avec la chaîne littérale Hello comme premier paramètre, et retourne la propriété prop1 de l’objet.|  
|« @function(parameters('Hello')) »|Évalue le paramètre Hello et transmet la valeur à la fonction.|  
  
### <a name="referencing-functions"></a>Fonctions de référencement  

Vous pouvez utiliser les fonctions indiquées ci-dessous pour référencer les sorties des autres actions de l’application logique ou les valeurs transmises lors de la création de l’application logique. Par exemple, vous pouvez référencer les données d’une étape pour les utiliser dans une autre.  
  
|Nom de la fonction|Description|  
|-------------------|-----------------|  
|parameters|Retourne une valeur de paramètre définie dans la définition. <p>`parameters('password')` <p> **Numéro du paramètre** : 1 <p> **Nom** : paramètre <p> **Description** : obligatoire. Nom du paramètre dont vous voulez les valeurs.|  
|action|Permet à une expression de tirer sa valeur d’un autre nom JSON et de paires de valeurs ou de la sortie de l’action runtime actuelle. La propriété représentée par propertyPath dans l’exemple suivant est facultative. Si l’élément propertyPath n’est pas spécifié, la référence se rapporte à l’objet d’action complet. Cette fonction ne peut être utilisée que dans les conditions do-until d’une action. <p>`action().outputs.body.propertyPath`|  
|actions|Permet à une expression de tirer sa valeur d’un autre nom JSON et de paires de valeurs ou de la sortie de l’action runtime. Ces expressions déclarent explicitement qu’une action dépend d’une autre action. La propriété représentée par propertyPath dans l’exemple suivant est facultative. Si l’élément propertyPath n’est pas spécifié, la référence se rapporte à l’objet d’action complet. Vous pouvez utiliser cet élément ou l’élément conditions pour spécifier les dépendances, mais il est inutile d’utiliser les deux pour la même ressource dépendante. <p>`actions('myAction').outputs.body.propertyPath` <p> **Numéro du paramètre** : 1 <p> **Nom** : nom de l’action <p> **Description** : obligatoire. Nom de l’action dont vous voulez les valeurs.|  
|trigger|Permet à une expression de tirer sa valeur d’un autre nom JSON et de paires de valeurs ou de la sortie du déclencheur runtime. La propriété représentée par propertyPath dans l’exemple suivant est facultative. Si l’élément propertyPath n’est pas spécifié, la référence se rapporte à l’objet déclencheur complet. <p>`trigger().outputs.body.propertyPath` <p>Lorsqu’elle est utilisée dans les entrées d’un déclencheur, la fonction retourne les sorties de l’exécution précédente. Cependant, lorsqu’elle est utilisée dans la condition d’un déclencheur, la fonction `trigger` retourne les sorties de l’exécution actuelle.|  
|actionOutputs|Cette fonction est la version raccourcie de `actions('actionName').outputs`. <p> **Numéro du paramètre** : 1 <p> **Nom** : nom de l’action <p> **Description** : obligatoire. Nom de l’action dont vous voulez les valeurs.|  
|actionBody et body|Ces fonctions sont les versions raccourcies de `actions('actionName').outputs.body`. <p> **Numéro du paramètre** : 1 <p> **Nom** : nom de l’action <p> **Description** : obligatoire. Nom de l’action dont vous voulez les valeurs.|  
|triggerOutputs|Cette fonction est la version raccourcie de `trigger().outputs`.|  
|triggerBody|Cette fonction est la version raccourcie de `trigger().outputs.body`.|  
|item|Lorsqu’elle est utilisée dans une action répétée, cette fonction retourne l’élément qui se trouve dans le tableau correspondant à cette itération de l’action. Par exemple, si l’une de vos actions s’exécute pour chaque élément d’un tableau de messages, vous pouvez utiliser cette syntaxe : <p>`"input1" : "@item().subject"`|  
  
### <a name="collection-functions"></a>Fonctions de collection  

Ces fonctions s’opèrent sur les collections et s’appliquent généralement aux tableaux, chaînes et parfois aux dictionnaires.  
  
|Nom de la fonction|Description|  
|-------------------|-----------------|  
|contains|Retourne la valeur true si le dictionnaire contient une clé, si la liste contient une valeur ou si la chaîne contient une sous-chaîne. Par exemple, cette fonction retourne `true` : <p>`contains('abacaba','aca')` <p> **Numéro du paramètre** : 1 <p> **Nom** : au sein de la collection <p> **Description** : obligatoire. Collection dans laquelle effectuer une recherche. <p> **Numéro du paramètre** : 2 <p> **Nom** : rechercher un objet <p> **Description** : obligatoire. Objet à rechercher dans **Au sein de la collection**.|  
|length|Retourne le nombre d’éléments contenus dans un tableau ou une chaîne. Par exemple, cette fonction retourne `3` :  <p>`length('abc')` <p> **Numéro du paramètre** : 1 <p> **Nom** : collection <p> **Description** : obligatoire. Collection dont la longueur est obtenue.|  
|empty|Retourne la valeur true si l’objet, le tableau ou la chaîne est vide. Par exemple, cette fonction retourne `true` : <p>`empty('')` <p> **Numéro du paramètre** : 1 <p> **Nom** : collection <p> **Description** : obligatoire. Collection vérifiée pour voir si elle est vide.|  
|intersection|Retourne un tableau ou un objet unique qui contient des éléments communs transmis entre les tableaux ou les objets. Par exemple, cette fonction retourne `[1, 2]` : <p>`intersection([1, 2, 3], [101, 2, 1, 10],[6, 8, 1, 2])` <p>Les paramètres de la fonction peuvent être un ensemble d’objets ou un ensemble de tableaux (pas une combinaison des deux). Si deux objets portent le même nom, le dernier objet portant ce nom s’affiche dans l’objet final. <p> **Numéro du paramètre** : 1 ... *n* <p> **Nom** : collection *n* <p> **Description** : obligatoire. Collections à évaluer. Un objet doit figurer dans toutes les collections transmises pour apparaître dans le résultat.|  
|union|Retourne un tableau ou un objet unique avec tous les éléments contenus dans le tableau ou l’objet transmis à cette fonction. Par exemple, cette fonction retourne `[1, 2, 3, 10, 101]` : <p>`union([1, 2, 3], [101, 2, 1, 10])` <p>Les paramètres de la fonction peuvent être un ensemble d’objets ou un ensemble de tableaux (pas une combinaison de ceux-ci). Si deux objets portent le même nom dans la sortie finale, le dernier objet portant ce nom s’affiche dans l’objet final. <p> **Numéro du paramètre** : 1 ... *n* <p> **Nom** : collection *n* <p> **Description** : obligatoire. Collections à évaluer. Un objet affiché dans les collections apparaît également dans le résultat.|  
|first|Retourne le premier élément de la chaîne ou du tableau transmis. Par exemple, cette fonction retourne `0` : <p>`first([0,2,3])` <p> **Numéro du paramètre** : 1 <p> **Nom** : collection <p> **Description** : obligatoire. Collection dont il convient de prendre le premier objet.|  
|last|Retourne le dernier élément de la chaîne ou du tableau transmis. Par exemple, cette fonction retourne `3` : <p>`last('0123')` <p> **Numéro du paramètre** : 1 <p> **Nom** : collection <p> **Description** : obligatoire. Collection dans laquelle prendre le dernier objet.|  
|take|Retourne les premiers éléments **Nombre** de la chaîne ou du tableau transmis. Par exemple, cette fonction retourne `[1, 2]` :  <p>`take([1, 2, 3, 4], 2)` <p> **Numéro du paramètre** : 1 <p> **Nom** : collection <p> **Description** : obligatoire. Collection dans laquelle prendre les premiers objets **Nombre**. <p> **Numéro du paramètre** : 2 <p> **Nom** : nombre <p> **Description** : obligatoire. Nombre d’objets à prendre dans la **Collection**. Cette valeur doit être un entier positif.|  
|skip|Retourne les éléments du tableau commençant à l’index **Nombre**. Par exemple, cette fonction retourne `[3, 4]` : <p>`skip([1, 2 ,3 ,4], 2)` <p> **Numéro du paramètre** : 1 <p> **Nom** : collection <p> **Description** : obligatoire. Collection dans laquelle ignorer les premiers objets **Nombre**. <p> **Numéro du paramètre** : 2 <p> **Nom** : nombre <p> **Description** : obligatoire. Nombre d’objets à supprimer du début de la **Collection**. Cette valeur doit être un entier positif.|  
|join|Retourne une chaîne avec chaque élément d’un tableau joint par un délimiteur. Par exemple, cette fonction retourne `"1,2,3,4"` :<br /><br /> `join([1, 2, 3, 4], ',')`<br /><br /> **Numéro du paramètre** : 1<br /><br /> **Nom** : collection<br /><br /> **Description** : obligatoire. Collection dont des éléments sont à joindre.<br /><br /> **Numéro du paramètre** : 2<br /><br /> **Nom** : délimiteur<br /><br /> **Description** : obligatoire. Chaîne dans laquelle délimiter des éléments.|  
  
### <a name="string-functions"></a>Fonctions de chaîne

Les fonctions suivantes s’appliquent uniquement aux chaînes. Vous pouvez également appliquer certaines fonctions de collection aux chaînes.  
  
|Nom de la fonction|Description|  
|-------------------|-----------------|  
|concat|Combine plusieurs chaînes. Par exemple, si le paramètre 1 est `p1`, cette fonction retourne `somevalue-p1-somevalue` : <p>`concat('somevalue-',parameters('parameter1'),'-somevalue')` <p> **Numéro du paramètre** : 1 ... *n* <p> **Nom** : chaîne *n* <p> **Description** : obligatoire. Chaînes à combiner en une seule chaîne.|  
|substring|Retourne un sous-ensemble de caractères d’une chaîne. Par exemple, cette fonction retourne `abc` : <p>`substring('somevalue-abc-somevalue',10,3)` <p> **Numéro du paramètre** : 1 <p> **Nom** : chaîne <p> **Description** : obligatoire. Chaîne dont la sous-chaîne est extraite. <p> **Numéro du paramètre** : 2 <p> **Nom** : index de début <p> **Description** : obligatoire. Index de début de la sous-chaîne dans le paramètre 1. <p> **Numéro du paramètre** : 3 <p> **Nom** : longueur <p> **Description** : obligatoire. Longueur de la sous-chaîne.|  
|replace|Remplace une chaîne par une chaîne donnée. Par exemple, cette fonction retourne `the new string` : <p>`replace('the old string', 'old', 'new')` <p> **Numéro du paramètre** : 1 <p> **Nom** : chaîne <p> **Description** : obligatoire. Le paramètre 2 est recherché dans la chaîne, qui est mise à jour avec le paramètre 3 lorsque le paramètre 2 est trouvé dans le paramètre 1. <p> **Numéro du paramètre** : 2 <p> **Nom** : ancienne chaîne <p> **Description** : obligatoire. Chaîne à remplacer par le paramètre 3 lorsqu’une correspondance est trouvée dans le paramètre 1. <p> **Numéro du paramètre** : 3 <p> **Nom** : nouvelle chaîne <p> **Description** : obligatoire. Chaîne utilisée pour remplacer la chaîne du paramètre 2 lorsqu’une correspondance est trouvée dans le paramètre 1.|  
|guid|Cette fonction génère une chaîne globale unique (GUID). Par exemple, cette fonction peut générer ce GUID : `c2ecc88d-88c8-4096-912c-d6f2e2b138ce` <p>`guid()` <p> **Numéro du paramètre** : 1 <p> **Nom** : format <p> **Description** : facultatif. Spécificateur de format unique qui indique [comment mettre en forme la valeur de ce GUID](https://msdn.microsoft.com/library/97af8hh4%28v=vs.110%29.aspx). Le paramètre de format peut être « N », « D », « B », « P » ou « X ». Si aucun format n’est indiqué, « D » est utilisé.|  
|toLower|Convertit une chaîne en minuscules. Par exemple, cette fonction retourne `two by two is four` : <p>`toLower('Two by Two is Four')` <p> **Numéro du paramètre** : 1 <p> **Nom** : chaîne <p> **Description** : obligatoire. Chaîne à convertir en minuscules. Si un caractère de la chaîne ne possède pas d’équivalent minuscule, il est inclus tel quel dans la chaîne retournée.|  
|toUpper|Convertit une chaîne en majuscules. Par exemple, cette fonction retourne `TWO BY TWO IS FOUR` : <p>`toUpper('Two by Two is Four')` <p> **Numéro du paramètre** : 1 <p> **Nom** : chaîne <p> **Description** : obligatoire. Chaîne à convertir en majuscules. Si un caractère de la chaîne ne possède pas d’équivalent majuscule, il est inclus tel quel dans la chaîne retournée.|  
|indexof|Recherche l’index d’une valeur contenue dans une chaîne sans tenir compte de la casse. Par exemple, cette fonction retourne `7` : <p>`indexof('hello, world.', 'world')` <p> **Numéro du paramètre** : 1 <p> **Nom** : chaîne <p> **Description** : obligatoire. Chaîne pouvant contenir la valeur. <p> **Numéro du paramètre** : 2 <p> **Nom** : chaîne <p> **Description** : obligatoire. Valeur permettant d’en rechercher l’index.|  
|lastindexof|Recherche le dernier index d’une valeur contenue dans une chaîne sans tenir compte de la casse. Par exemple, cette fonction retourne `3` : <p>`lastindexof('foofoo', 'foo')` <p> **Numéro du paramètre** : 1 <p> **Nom** : chaîne <p> **Description** : obligatoire. Chaîne pouvant contenir la valeur. <p> **Numéro du paramètre** : 2 <p> **Nom** : chaîne <p> **Description** : obligatoire. Valeur permettant d’en rechercher l’index.|  
|startswith|Vérifie si la chaîne commence par une valeur sans tenir compte de la casse. Par exemple, cette fonction retourne `true` : <p>`startswith('hello, world', 'hello')` <p> **Numéro du paramètre** : 1 <p> **Nom** : chaîne <p> **Description** : obligatoire. Chaîne pouvant contenir la valeur. <p> **Numéro du paramètre** : 2 <p> **Nom** : chaîne <p> **Description** : obligatoire. Valeur de début susceptible de la chaîne.|  
|endswith|Vérifie si la chaîne se termine par une valeur sans tenir compte de la casse. Par exemple, cette fonction retourne `true` : <p>`endswith('hello, world', 'world')` <p> **Numéro du paramètre** : 1 <p> **Nom** : chaîne <p> **Description** : obligatoire. Chaîne pouvant contenir la valeur. <p> **Numéro du paramètre** : 2 <p> **Nom** : chaîne <p> **Description** : obligatoire. Valeur de fin susceptible de la chaîne.|  
|split|Fractionne la chaîne à l’aide d’un séparateur. Par exemple, cette fonction retourne `["a", "b", "c"]` : <p>`split('a;b;c',';')` <p> **Numéro du paramètre** : 1 <p> **Nom** : chaîne <p> **Description** : obligatoire. Chaîne qui est fractionnée. <p> **Numéro du paramètre** : 2 <p> **Nom** : chaîne <p> **Description** : obligatoire. Séparateur.|  
  
### <a name="logical-functions"></a>Fonctions logiques  

Ces fonctions sont utiles dans les conditions et permettent d’évaluer tout type de logique.  
  
|Nom de la fonction|Description|  
|-------------------|-----------------|  
|equals|Retourne la valeur true si deux valeurs sont égales. Par exemple, si parameter1 est égal à someValue, cette fonction retourne `true` : <p>`equals(parameters('parameter1'), 'someValue')` <p> **Numéro du paramètre** : 1 <p> **Nom** : objet 1 <p> **Description** : obligatoire. Objet à comparer à **Objet 2**. <p> **Numéro du paramètre** : 2 <p> **Nom** : objet 2 <p> **Description** : obligatoire. Objet à comparer à **Objet 1**.|  
|less|Retourne la valeur true si le premier argument est inférieur au second. Les valeurs ne peuvent être que du type entier, flottant ou chaîne. Par exemple, cette fonction retourne `true` : <p>`less(10,100)` <p> **Numéro du paramètre** : 1 <p> **Nom** : objet 1 <p> **Description** : obligatoire. Objet à vérifier pour voir s’il est inférieur à **Objet 2**. <p> **Numéro du paramètre** : 2 <p> **Nom** : objet 2 <p> **Description** : obligatoire. Objet à vérifier pour voir s’il est supérieur à **Objet 1**.|  
|lessOrEquals|Retourne la valeur true si le premier argument est inférieur ou égal au second. Les valeurs ne peuvent être que du type entier, flottant ou chaîne. Par exemple, cette fonction retourne `true` : <p>`lessOrEquals(10,10)` <p> **Numéro du paramètre** : 1 <p> **Nom** : objet 1 <p> **Description** : obligatoire. Objet à vérifier pour voir s’il est inférieur ou égal à **Objet 2**. <p> **Numéro du paramètre** : 2 <p> **Nom** : objet 2 <p> **Description** : obligatoire. Objet à vérifier pour voir s’il est supérieur ou égal à **Objet 1**.|  
|greater|Retourne la valeur true si le premier argument est supérieur au second. Les valeurs ne peuvent être que du type entier, flottant ou chaîne. Par exemple, cette fonction retourne `false` :  <p>`greater(10,10)` <p> **Numéro du paramètre** : 1 <p> **Nom** : objet 1 <p> **Description** : obligatoire. Objet à vérifier pour voir s’il est supérieur à **Objet 2**. <p> **Numéro du paramètre** : 2 <p> **Nom** : objet 2 <p> **Description** : obligatoire. Objet à vérifier pour voir s’il est inférieur à **Objet 1**.|  
|greaterOrEquals|Retourne la valeur true si le premier argument est supérieur ou égal au second. Les valeurs ne peuvent être que du type entier, flottant ou chaîne. Par exemple, cette fonction retourne `false` : <p>`greaterOrEquals(10,100)` <p> **Numéro du paramètre** : 1 <p> **Nom** : objet 1 <p> **Description** : obligatoire. Objet à vérifier pour voir s’il est supérieur ou égal à **Objet 2**. <p> **Numéro du paramètre** : 2 <p> **Nom** : objet 2 <p> **Description** : obligatoire. Objet à vérifier pour voir s’il est inférieur ou égal à **Objet 1**.|  
|and|Retourne la valeur true si les deux paramètres sont true. Les deux arguments doivent être des booléens. Par exemple, cette fonction retourne `false` : <p>`and(greater(1,10),equals(0,0))` <p> **Numéro du paramètre** : 1 <p> **Nom** : booléen 1 <p> **Description** : obligatoire. Premier argument qui doit être `true`. <p> **Numéro du paramètre** : 2 <p> **Nom** : booléen 2 <p> **Description** : obligatoire. Second argument qui doit être `true`.|  
|ou|Retourne la valeur true si l’un ou l’autre des paramètres est true. Les deux arguments doivent être des booléens. Par exemple, cette fonction retourne `true` : <p>`or(greater(1,10),equals(0,0))` <p> **Numéro du paramètre** : 1 <p> **Nom** : booléen 1 <p> **Description** : obligatoire. Premier argument pouvant être `true`. <p> **Numéro du paramètre** : 2 <p> **Nom** : booléen 2 <p> **Description** : obligatoire. Second argument pouvant être `true`.|  
|not|Retourne la valeur true si les paramètres sont `false`. Les deux arguments doivent être des booléens. Par exemple, cette fonction retourne `true` : <p>`not(contains('200 Success','Fail'))` <p> **Numéro du paramètre** : 1 <p> **Nom** : booléen <p> **Description** : retourne la valeur true si les paramètres sont `false`. Les deux arguments doivent être des booléens. Cette fonction retourne `true` : `not(contains('200 Success','Fail'))`|  
|if|Retourne une valeur spécifiée qui varie selon que le résultat de l’expression est `true` ou `false`.  Par exemple, cette fonction retourne `"yes"` : <p>`if(equals(1, 1), 'yes', 'no')` <p> **Numéro du paramètre** : 1 <p> **Nom** : expression <p> **Description** : obligatoire. Valeur booléenne qui détermine la valeur qui doit être retournée par l’expression. <p> **Numéro du paramètre** : 2 <p> **Nom** : true <p> **Description** : obligatoire. Valeur à retourner si l’expression est `true`. <p> **Numéro du paramètre** : 3 <p> **Nom** : false <p> **Description** : obligatoire. Valeur à retourner si l’expression est `false`.|  
  
### <a name="conversion-functions"></a>Fonctions de conversion  

Ces fonctions permettent de convertir chacun des types natifs du langage :  
  
- string  
  
- integer  
  
- float  
  
- booléenne  
  
- tableaux  
  
- dictionnaires  

-   formulaires  
  
|Nom de la fonction|Description|  
|-------------------|-----------------|  
|int|Convertit le paramètre en entier. Par exemple, cette fonction retourne 100 sous forme de nombre plutôt que sous forme de chaîne : <p>`int('100')` <p> **Numéro du paramètre** : 1 <p> **Nom** : valeur <p> **Description** : obligatoire. Valeur qui est convertie en entier.|  
|string|Convertit le paramètre en chaîne. Par exemple, cette fonction retourne `'10'` : <p>`string(10)` <p>Vous pouvez également convertir un objet en chaîne. Par exemple, si le paramètre `myPar` est un objet pourvu d’une propriété `abc : xyz`, cette fonction retourne `{"abc" : "xyz"}` : <p>`string(parameters('myPar'))` <p> **Numéro du paramètre** : 1 <p> **Nom** : valeur <p> **Description** : obligatoire. Valeur qui est convertie en chaîne.|  
|json|Convertit le paramètre en valeur de type JSON. Est l’opposé de `string()`. Par exemple, cette fonction retourne `[1,2,3]` sous forme de tableau plutôt que sous forme de chaîne : <p>`parse('[1,2,3]')` <p>Vous pouvez également convertir une chaîne en objet. Par exemple, cette fonction retourne `{ "abc" : "xyz" }` : <p>`json('{"abc" : "xyz"}')` <p> **Numéro du paramètre** : 1 <p> **Nom** : chaîne <p> **Description** : obligatoire. Chaîne qui est convertie en valeur de type natif. <p>La fonction `json()` prend également en charge les entrées XML. Par exemple, la valeur du paramètre : <p>`<?xml version="1.0"?> <root>   <person id='1'>     <name>Alan</name>     <occupation>Engineer</occupation>   </person> </root>` <p>est convertie en cette valeur JSON : <p>`{ "?xml": { "@version": "1.0" },   "root": {     "person": [     {       "@id": "1",       "name": "Alan",       "occupation": "Engineer"     }   ]   } }`|  
|float|Convertit l’argument de paramètre en nombre à virgule flottante. Par exemple, cette fonction retourne `10.333` : <p>`float('10.333')` <p> **Numéro du paramètre** : 1 <p> **Nom** : valeur <p> **Description** : obligatoire. Valeur qui est convertie en nombre à virgule flottante.|  
|valeur booléenne|Convertit le paramètre en booléen. Par exemple, cette fonction retourne `false` : <p>`bool(0)` <p> **Numéro du paramètre** : 1 <p> **Nom** : valeur <p> **Description** : obligatoire. Valeur qui est convertie en booléen.|  
|coalesce|Retourne le premier objet non Null dans les arguments transmis. **Remarque** : une chaîne vide n’a pas la valeur Null. Par exemple, si les paramètres 1 et 2 ne sont pas définis, cette fonction retourne `fallback` :  <p>`coalesce(parameters('parameter1'), parameters('parameter2') ,'fallback')` <p> **Numéro du paramètre** : 1 ... *n* <p> **Nom** : Objet *n* <p> **Description** : obligatoire. Objets dans lesquels rechercher la valeur Null.|  
|base64|Retourne la représentation en base 64 de la chaîne d'entrée. Par exemple, cette fonction retourne `c29tZSBzdHJpbmc=` : <p>`base64('some string')` <p> **Numéro du paramètre** : 1 <p> **Nom** : chaîne 1 <p> **Description** : obligatoire. Chaîne à encoder en représentation en base64.|  
|base64ToBinary|Retourne une représentation binaire d’une chaîne encodée en base64. Par exemple, cette fonction retourne la représentation binaire de `some string` : <p>`base64ToBinary('c29tZSBzdHJpbmc=')` <p> **Numéro du paramètre** : 1 <p> **Nom** : chaîne <p> **Description** : obligatoire. Chaîne encodée en Base64.|  
|base64ToString|Retourne une représentation de chaîne d’une chaîne encodée en base64. Par exemple, cette fonction retourne `some string` : <p>`base64ToString('c29tZSBzdHJpbmc=')` <p> **Numéro du paramètre** : 1 <p> **Nom** : chaîne <p> **Description** : obligatoire. Chaîne encodée en Base64.|  
|Fichier binaire|Retourne une représentation binaire d’une valeur.  Par exemple, cette fonction retourne une représentation binaire de `some string` : <p>`binary('some string')` <p> **Numéro du paramètre** : 1 <p> **Nom** : valeur <p> **Description** : obligatoire. Valeur qui est convertie au format binaire.|  
|dataUriToBinary|Retourne une représentation binaire d’un URI de données. Par exemple, cette fonction retourne la représentation binaire de `some string` : <p>`dataUriToBinary('data:;base64,c29tZSBzdHJpbmc=')` <p> **Numéro du paramètre** : 1 <p> **Nom** : chaîne <p> **Description** : obligatoire. URI de données à convertir en représentation binaire.|  
|dataUriToString|Retourne une représentation de chaîne d’un URI de données. Par exemple, cette fonction retourne `some string` : <p>`dataUriToString('data:;base64,c29tZSBzdHJpbmc=')` <p> **Numéro du paramètre** : 1 <p> **Nom** : chaîne<p> **Description** : obligatoire. URI de données à convertir en représentation de chaîne.|  
|dataUri|Retourne un URI de données d’une valeur. Par exemple, cette fonction retourne cet URI de données `text/plain;charset=utf8;base64,c29tZSBzdHJpbmc=` : <p>`dataUri('some string')` <p> **Numéro du paramètre** : 1<p> **Nom** : valeur<p> **Description** : obligatoire. Valeur à convertir en URI de données.|  
|decodeBase64|Retourne une représentation de chaîne d’une chaîne en base64 d’entrée. Par exemple, cette fonction retourne `some string` : <p>`decodeBase64('c29tZSBzdHJpbmc=')` <p> **Numéro du paramètre** : 1 <p> **Nom** : chaîne <p> **Description** : retourne une représentation de chaîne d’une chaîne en base64 d’entrée.|  
|encodeUriComponent|Place la chaîne transmise dans un échappement d’URL. Par exemple, cette fonction retourne `You+Are%3ACool%2FAwesome` : <p>`encodeUriComponent('You Are:Cool/Awesome')` <p> **Numéro du paramètre** : 1 <p> **Nom** : chaîne <p> **Description** : obligatoire. Chaîne à partir de laquelle placer les caractères non sécurisés pour les URL dans une séquence d’échappement.|  
|decodeUriComponent|Retire la chaîne transmise d’un échappement d’URL. Par exemple, cette fonction retourne `You Are:Cool/Awesome` : <p>`encodeUriComponent('You+Are%3ACool%2FAwesome')` <p> **Numéro du paramètre** : 1 <p> **Nom** : chaîne <p> **Description** : obligatoire. Chaîne à partir de laquelle décoder les caractères non sécurisés pour les URL.|  
|decodeDataUri|Retourne une représentation binaire d’une chaîne d’URI de données d’entrée. Par exemple, cette fonction retourne la représentation binaire de `some string` : <p>`decodeDataUri('data:;base64,c29tZSBzdHJpbmc=')` <p> **Numéro du paramètre** : 1 <p> **Nom** : chaîne <p> **Description** : obligatoire. DataURI à décoder en représentation binaire.|  
|uriComponent|Retourne une représentation encodée d’URI d’une valeur. Par exemple, cette fonction retourne `You+Are%3ACool%2FAwesome` : <p>`uriComponent('You Are:Cool/Awesome')` <p> **Numéro du paramètre** : 1<p> **Nom** : chaîne <p> **Description** : obligatoire. Chaîne à encoder en URI.|  
|uriComponentToBinary|Retourne une représentation binaire d’une chaîne encodée d’URI. Par exemple, cette fonction retourne une représentation binaire de `You Are:Cool/Awesome` : <p>`uriComponentToBinary('You+Are%3ACool%2FAwesome')` <p> **Numéro du paramètre** : 1 <p> **Nom** : chaîne<p> **Description** : obligatoire. Chaîne encodée d’URI.|  
|uriComponentToString|Retourne une représentation de chaîne d’une chaîne encodée d’URI. Par exemple, cette fonction retourne `You Are:Cool/Awesome` : <p>`uriComponentToBinary('You+Are%3ACool%2FAwesome')` <p> **Numéro du paramètre** : 1<p> **Nom** : chaîne<p> **Description** : obligatoire. Chaîne encodée d’URI.|  
|xml|Retourne une représentation XML de la valeur. Par exemple, cette fonction retourne du contenu XML représenté par `'\<name>Alan\</name>'` : <p>`xml('\<name>Alan\</name>')` <p>La fonction `xml()` prend également en charge les entrées d’objets JSON. Par exemple, le paramètre `{ "abc": "xyz" }` est converti en contenu XML : `\<abc>xyz\</abc>` <p> **Numéro du paramètre** : 1<p> **Nom** : valeur<p> **Description** : obligatoire. Valeur à convertir au format XML.|  
|xpath|Retourne un tableau de nœuds XML correspondant à l’expression xpath d’une valeur que prend l’expression xpath. <p> **Exemple 1** <p>Supposons que la valeur du paramètre `p1` soit une représentation de chaîne de ce contenu XML : <p>`<?xml version="1.0"?> <lab>   <robot>     <parts>5</parts>     <name>R1</name>   </robot>   <robot>     <parts>8</parts>     <name>R2</name>   </robot> </lab>` <p>Ce code : `xpath(xml(parameters('p1'), '/lab/robot/name')` <p>retourne <p>`[ <name>R1</name>, <name>R2</name> ]` <p>alors que ce code : <p>`xpath(xml(parameters('p1'), ' sum(/lab/robot/parts)')` <p>retourne <p>`13` <p> <p> **Exemple 2** <p>Étant donné le contenu XML suivant : <p>`<?xml version="1.0"?> <File xmlns="http://foo.com">   <Location>bar</Location> </File>` <p>Ce code : `@xpath(xml(body('Http')), '/*[name()=\"File\"]/*[name()=\"Location\"]')` <p>ou ce code : <p>`@xpath(xml(body('Http')), '/*[local-name()=\"File\" and namespace-uri()=\"http://foo.com\"]/*[local-name()=\"Location\" and namespace-uri()=\"\"]')` <p>retourne <p>`<Location xmlns="http://abc.com">xyz</Location>` <p>Et ce code : `@xpath(xml(body('Http')), 'string(/*[name()=\"File\"]/*[name()=\"Location\"])')` <p>retourne <p>``xyz`` <p> **Numéro du paramètre** : 1 <p> **Nom** : Xml <p> **Description** : obligatoire. Valeur XML que prend l’expression XPath. <p> **Numéro du paramètre** : 2 <p> **Nom** : XPath <p> **Description** : obligatoire. Expression XPath à évaluer.|  
|array|Convertit le paramètre en tableau. Par exemple, cette fonction retourne `["abc"]` : <p>`array('abc')` <p> **Numéro du paramètre** : 1 <p> **Nom** : valeur <p> **Description** : obligatoire. Valeur qui est convertie en tableau.|
|createArray|Crée un tableau à partir des paramètres. Par exemple, cette fonction retourne `["a", "c"]` : <p>`createArray('a', 'c')` <p> **Numéro du paramètre** : 1 ... *n* <p> **Nom** : tout *n* <p> **Description** : obligatoire. Valeurs à combiner en un tableau.|
|triggerFormDataValue|Retourne une valeur unique correspondant au nom de clé d’une sortie de déclencheur de données de formulaire ou encodée en formulaire.  Retourne une erreur en cas de correspondances multiples.  Par exemple, le contenu suivant retourne `bar` : `triggerFormDataValue('foo')`<br /><br />**Numéro du paramètre** : 1<br /><br />**Nom** : nom de la clé<br /><br />**Description** : obligatoire. Nom de la clé de la valeur des données de formulaire à retourner.|
|triggerFormDataMultiValues|Retourne un tableau de valeurs correspondant au nom de clé d’une sortie de déclencheur de données de formulaire ou encodée en formulaire.  Par exemple, le contenu suivant retourne `["bar"]` : `triggerFormDataValue('foo')`<br /><br />**Numéro du paramètre** : 1<br /><br />**Nom** : nom de la clé<br /><br />**Description** : obligatoire. Nom de la clé des valeurs des données de formulaire à retourner.|
|triggerMultipartBody|Retourne le corps d’une partie dans une sortie en plusieurs parties du déclencheur.<br /><br />**Numéro du paramètre** : 1<br /><br />**Nom** : index<br /><br />**Description** : obligatoire. Index de la partie à récupérer.|
|formDataValue|Retourne une valeur unique correspondant au nom de clé d’une sortie d’action de données de formulaire ou encodée en formulaire.  Retourne une erreur en cas de correspondances multiples.  Par exemple, le contenu suivant retourne `bar` : `formDataValue('someAction', 'foo')`<br /><br />**Numéro du paramètre** : 1<br /><br />**Nom** : nom de l’action<br /><br />**Description** : obligatoire. Nom de l’action avec une réponse de données de formulaire ou encodée en formulaire.<br /><br />**Numéro du paramètre** : 2<br /><br />**Nom** : nom de la clé<br /><br />**Description** : obligatoire. Nom de la clé de la valeur des données de formulaire à retourner.|
|formDataMultiValues|Retourne un tableau de valeurs correspondant au nom de clé d’une sortie d’action de données de formulaire ou encodée en formulaire.  Par exemple, le contenu suivant retourne `["bar"]` : `formDataMultiValues('someAction', 'foo')`<br /><br />**Numéro du paramètre** : 1<br /><br />**Nom** : nom de l’action<br /><br />**Description** : obligatoire. Nom de l’action avec une réponse de données de formulaire ou encodée en formulaire.<br /><br />**Numéro du paramètre** : 2<br /><br />**Nom** : nom de la clé<br /><br />**Description** : obligatoire. Nom de la clé des valeurs des données de formulaire à retourner.|
|multipartBody|Retourne le corps d’une partie dans une sortie en plusieurs parties d’une action.<br /><br />**Numéro du paramètre** : 1<br /><br />**Nom** : nom de l’action<br /><br />**Description** : obligatoire. Nom de l’action avec une réponse en plusieurs parties.<br /><br />**Numéro du paramètre** : 2<br /><br />**Nom** : index<br /><br />**Description** : obligatoire. Index de la partie à récupérer.|

### <a name="math-functions"></a>Fonctions mathématiques  

Ces fonctions peuvent être utilisées pour les deux types de nombre : **entiers** et **flottants**.  
  
|Nom de la fonction|Description|  
|-------------------|-----------------|  
|ajouter|Retourne le résultat de l’ajout de deux nombres. Par exemple, cette fonction retourne `20.333` : <p>`add(10,10.333)` <p> **Numéro du paramètre** : 1 <p> **Nom** : opérande 1 <p> **Description** : obligatoire. Nombre à ajouter à **Opérande 2**. <p> **Numéro du paramètre** : 2 <p> **Nom** : opérande 2 <p> **Description** : obligatoire. Nombre à ajouter à **Opérande 1**.|  
|sub|Retourne le résultat de la soustraction de deux nombres. Par exemple, cette fonction retourne `-0.333` : <p>`sub(10,10.333)` <p> **Numéro du paramètre** : 1 <p> **Nom** : diminuende <p> **Description** : obligatoire. Nombre duquel le **diminuteur** est soustrait. <p> **Numéro du paramètre** : 2 <p> **Nom** : diminuteur <p> **Description** : obligatoire. Nombre à soustraire du **diminuende**.|  
|mul|Retourne le résultat de la multiplication de deux nombres. Par exemple, cette fonction retourne `103.33` : <p>`mul(10,10.333)` <p> **Numéro du paramètre** : 1 <p> **Nom** : multiplicande 1 <p> **Description** : obligatoire. Nombre à multiplier avec le **multiplicande 2**. <p> **Numéro du paramètre** : 2 <p> **Nom** : multiplicande 2 <p> **Description** : obligatoire. Nombre à multiplier avec le **multiplicande 1**.|  
|div|Retourne le résultat de la division de deux nombres. Par exemple, cette fonction retourne `1.0333` : <p>`div(10.333,10)` <p> **Numéro du paramètre** : 1 <p> **Nom** : dividende <p> **Description** : obligatoire. Nombre à diviser par le **diviseur**. <p> **Numéro du paramètre** : 2 <p> **Nom** : diviseur <p> **Description** : obligatoire. Nombre par lequel diviser le **dividende**.|  
|mod|Retourne le reste après la division de deux nombres (modulo). Par exemple, cette fonction retourne `2` : <p>`mod(10,4)` <p> **Numéro du paramètre** : 1 <p> **Nom** : dividende <p> **Description** : obligatoire. Nombre à diviser par le **diviseur**. <p> **Numéro du paramètre** : 2 <p> **Nom** : diviseur <p> **Description** : obligatoire. Nombre par lequel diviser le **dividende**. Après la division, le reste est extrait.|  
|Min|Il existe deux modèles différents pour appeler cette fonction. <p>Ici, `min` extrait un tableau, et la fonction retourne `0` : <p>`min([0,1,2])` <p>Cette fonction peut également extraire une liste de valeurs séparées par des virgules, et elle retourne aussi `0` : <p>`min(0,1,2)` <p> **Remarque** : toutes les valeurs doivent être des nombres. Si le paramètre est un tableau, il ne doit contenir que des nombres. <p> **Numéro du paramètre** : 1 <p> **Nom** : collection ou valeur <p> **Description** : obligatoire. Tableau de valeurs pour y rechercher la valeur minimale ou première valeur d’un ensemble. <p> **Numéro du paramètre** : 2 ... *n* <p> **Nom** : valeur *n* <p> **Description** : facultatif. Si le premier paramètre est une valeur, vous pouvez transmettre des valeurs supplémentaires, et la valeur minimale de toutes les valeurs transmises est retournée.|  
|max|Il existe deux modèles différents pour appeler cette fonction. <p>Ici, `max` extrait un tableau, et la fonction retourne `2` : <p>`max([0,1,2])` <p>Cette fonction peut également extraire une liste de valeurs séparées par des virgules, et elle retourne aussi `2` : <p>`max(0,1,2)` <p> **Remarque** : toutes les valeurs doivent être des nombres. Si le paramètre est un tableau, il ne doit contenir que des nombres. <p> **Numéro du paramètre** : 1 <p> **Nom** : collection ou valeur <p> **Description** : obligatoire. Tableau de valeurs pour y rechercher la valeur maximale ou première valeur d’un ensemble. <p> **Numéro du paramètre** : 2 ... *n* <p> **Nom** : valeur *n* <p> **Description** : facultatif. Si le premier paramètre est une valeur, vous pouvez transmettre des valeurs supplémentaires, et la valeur maximale de toutes les valeurs transmises est retournée.|  
|range|Génère un tableau d’entiers à partir d’un certain nombre. Vous pouvez définir la longueur du tableau retourné. <p>Par exemple, cette fonction retourne `[3,4,5,6]` : <p>`range(3,4)` <p> **Numéro du paramètre** : 1 <p> **Nom** : index de début <p> **Description** : obligatoire. Premier entier du tableau. <p> **Numéro du paramètre** : 2 <p> **Nom** : nombre <p> **Description** : obligatoire. Cette valeur représente le nombre d’entiers qui se trouvent dans le tableau.|  
|rand|Génère un entier aléatoire compris dans la plage spécifiée (extrémités de plage incluses). Par exemple, cette fonction peut retourner `42` : <p>`rand(-1000,1000)` <p> **Numéro du paramètre** : 1 <p> **Nom** : minimum <p> **Description** : obligatoire. Entier inférieur pouvant être retourné. <p> **Numéro du paramètre** : 2 <p> **Nom** : maximum <p> **Description** : obligatoire. Cette valeur est l’entier le plus élevé pouvant être retourné.|  
  
### <a name="date-functions"></a>Fonctions de date  
  
|Nom de la fonction|Description|  
|-------------------|-----------------|  
|utcnow|Retourne l’horodateur actuel sous forme de chaîne, par exemple : `2017-03-15T13:27:36Z` : <p>`utcnow()` <p> **Numéro du paramètre** : 1 <p> **Nom** : format <p> **Description** : facultatif. [Caractère de spécificateur de format unique](https://msdn.microsoft.com/library/az4se3k1%28v=vs.110%29.aspx) ou [modèle de format personnalisé](https://msdn.microsoft.com/library/8kb3ddd4%28v=vs.110%29.aspx) qui indique comment mettre en forme la valeur de cet horodateur. Si aucun format n’est indiqué, le format ISO 8601 (« o ») est utilisé.|  
|addseconds|Ajoute un nombre entier de secondes à un horodateur de type chaîne transmis. Le nombre de secondes peut être positif ou négatif. Par défaut, le résultat est une chaîne au format ISO 8601 (« o »), sauf si un spécificateur de format est indiqué. Exemple : `2015-03-15T13:27:00Z` : <p>`addseconds('2015-03-15T13:27:36Z', -36)` <p> **Numéro du paramètre** : 1 <p> **Nom** : horodateur <p> **Description** : obligatoire. Chaîne qui contient l’heure. <p> **Numéro du paramètre** : 2 <p> **Nom** : secondes <p> **Description** : obligatoire. Nombre de secondes à ajouter. Peut être négatif pour soustraire des secondes. <p> **Numéro du paramètre** : 3 <p> **Nom** : format <p> **Description** : facultatif. [Caractère de spécificateur de format unique](https://msdn.microsoft.com/library/az4se3k1%28v=vs.110%29.aspx) ou [modèle de format personnalisé](https://msdn.microsoft.com/library/8kb3ddd4%28v=vs.110%29.aspx) qui indique comment mettre en forme la valeur de cet horodateur. Si aucun format n’est indiqué, le format ISO 8601 (« o ») est utilisé.|  
|addminutes|Ajoute un nombre entier de minutes à un horodateur de type chaîne transmis. Le nombre de minutes peut être positif ou négatif. Par défaut, le résultat est une chaîne au format ISO 8601 (« o »), sauf si un spécificateur de format est indiqué. Exemple : `2015-03-15T14:00:36Z` : <p>`addminutes('2015-03-15T13:27:36Z', 33)` <p> **Numéro du paramètre** : 1 <p> **Nom** : horodateur <p> **Description** : obligatoire. Chaîne qui contient l’heure. <p> **Numéro du paramètre** : 2 <p> **Nom** : minutes <p> **Description** : obligatoire. Nombre de minutes à ajouter. Peut être négatif pour soustraire des minutes. <p> **Numéro du paramètre** : 3 <p> **Nom** : format <p> **Description** : facultatif. [Caractère de spécificateur de format unique](https://msdn.microsoft.com/library/az4se3k1%28v=vs.110%29.aspx) ou [modèle de format personnalisé](https://msdn.microsoft.com/library/8kb3ddd4%28v=vs.110%29.aspx) qui indique comment mettre en forme la valeur de cet horodateur. Si aucun format n’est indiqué, le format ISO 8601 (« o ») est utilisé.|  
|addhours|Ajoute un nombre entier d’heures à un horodateur de type chaîne transmis. Le nombre d’heures peut être positif ou négatif. Par défaut, le résultat est une chaîne au format ISO 8601 (« o »), sauf si un spécificateur de format est indiqué. Exemple : `2015-03-16T01:27:36Z` : <p>`addhours('2015-03-15T13:27:36Z', 12)` <p> **Numéro du paramètre** : 1 <p> **Nom** : horodateur <p> **Description** : obligatoire. Chaîne qui contient l’heure. <p> **Numéro du paramètre** : 2 <p> **Nom** : heures <p> **Description** : obligatoire. Nombre d’heures à ajouter. Peut être négatif pour soustraire des heures. <p> **Numéro du paramètre** : 3 <p> **Nom** : format <p> **Description** : facultatif. [Caractère de spécificateur de format unique](https://msdn.microsoft.com/library/az4se3k1%28v=vs.110%29.aspx) ou [modèle de format personnalisé](https://msdn.microsoft.com/library/8kb3ddd4%28v=vs.110%29.aspx) qui indique comment mettre en forme la valeur de cet horodateur. Si aucun format n’est indiqué, le format ISO 8601 (« o ») est utilisé.|  
|adddays|Ajoute un nombre entier de jours à un horodateur de type chaîne transmis. Le nombre de jours peut être positif ou négatif. Par défaut, le résultat est une chaîne au format ISO 8601 (« o »), sauf si un spécificateur de format est indiqué. Exemple : `2015-02-23T13:27:36Z` : <p>`addseconds('2015-03-15T13:27:36Z', -20)` <p> **Numéro du paramètre** : 1 <p> **Nom** : horodateur <p> **Description** : obligatoire. Chaîne qui contient l’heure. <p> **Numéro du paramètre** : 2 <p> **Nom** : jours <p> **Description** : obligatoire. Nombre de jours à ajouter. Peut être négatif pour soustraire des jours. <p> **Numéro du paramètre** : 3 <p> **Nom** : format <p> **Description** : facultatif. [Caractère de spécificateur de format unique](https://msdn.microsoft.com/library/az4se3k1%28v=vs.110%29.aspx) ou [modèle de format personnalisé](https://msdn.microsoft.com/library/8kb3ddd4%28v=vs.110%29.aspx) qui indique comment mettre en forme la valeur de cet horodateur. Si aucun format n’est indiqué, le format ISO 8601 (« o ») est utilisé.|  
|formatDateTime|Retourne une chaîne au format de date. Par défaut, le résultat est une chaîne au format ISO 8601 (« o »), sauf si un spécificateur de format est indiqué. Exemple : `2015-02-23T13:27:36Z` : <p>`formatDateTime('2015-03-15T13:27:36Z', 'o')` <p> **Numéro du paramètre** : 1 <p> **Nom** : date <p> **Description** : obligatoire. Chaîne qui contient la date. <p> **Numéro du paramètre** : 2 <p> **Nom** : format <p> **Description** : facultatif. [Caractère de spécificateur de format unique](https://msdn.microsoft.com/library/az4se3k1%28v=vs.110%29.aspx) ou [modèle de format personnalisé](https://msdn.microsoft.com/library/8kb3ddd4%28v=vs.110%29.aspx) qui indique comment mettre en forme la valeur de cet horodateur. Si aucun format n’est indiqué, le format ISO 8601 (« o ») est utilisé.|  
|startOfHour|Retourne le début de l’heure dans un horodateur de type chaîne transmis. Exemple `2017-03-15T13:00:00Z` :<br /><br /> `startOfHour('2017-03-15T13:27:36Z')`<br /><br /> **Numéro du paramètre** : 1<br /><br /> **Nom** : horodateur<br /><br /> **Description** : obligatoire. Chaîne qui contient l’heure.<br /><br />**Numéro du paramètre** : 2<br /><br /> **Nom** : format<br /><br /> **Description** : facultatif. [Caractère de spécificateur de format unique](https://msdn.microsoft.com/library/az4se3k1%28v=vs.110%29.aspx) ou [modèle de format personnalisé](https://msdn.microsoft.com/library/8kb3ddd4%28v=vs.110%29.aspx) qui indique comment mettre en forme la valeur de cet horodateur. Si aucun format n’est indiqué, le format ISO 8601 (« o ») est utilisé.|  
|startOfDay|Retourne le début du jour dans un horodateur de type chaîne transmis. Exemple `2017-03-15T00:00:00Z` :<br /><br /> `startOfDay('2017-03-15T13:27:36Z')`<br /><br /> **Numéro du paramètre** : 1<br /><br /> **Nom** : horodateur<br /><br /> **Description** : obligatoire. Chaîne qui contient l’heure.<br /><br />**Numéro du paramètre** : 2<br /><br /> **Nom** : format<br /><br /> **Description** : facultatif. [Caractère de spécificateur de format unique](https://msdn.microsoft.com/library/az4se3k1%28v=vs.110%29.aspx) ou [modèle de format personnalisé](https://msdn.microsoft.com/library/8kb3ddd4%28v=vs.110%29.aspx) qui indique comment mettre en forme la valeur de cet horodateur. Si aucun format n’est indiqué, le format ISO 8601 (« o ») est utilisé.| 
|startOfMonth|Retourne le début du mois dans un horodateur de type chaîne transmis. Exemple `2017-03-01T00:00:00Z` :<br /><br /> `startOfMonth('2017-03-15T13:27:36Z')`<br /><br /> **Numéro du paramètre** : 1<br /><br /> **Nom** : horodateur<br /><br /> **Description** : obligatoire. Chaîne qui contient l’heure.<br /><br />**Numéro du paramètre** : 2<br /><br /> **Nom** : format<br /><br /> **Description** : facultatif. [Caractère de spécificateur de format unique](https://msdn.microsoft.com/library/az4se3k1%28v=vs.110%29.aspx) ou [modèle de format personnalisé](https://msdn.microsoft.com/library/8kb3ddd4%28v=vs.110%29.aspx) qui indique comment mettre en forme la valeur de cet horodateur. Si aucun format n’est indiqué, le format ISO 8601 (« o ») est utilisé.| 
|dayOfWeek|Retourne le jour de semaine d’un horodateur de type chaîne.  Le dimanche correspond à la valeur 0, le lundi à la valeur 1 et ainsi de suite. Exemple `3` :<br /><br /> `dayOfWeek('2017-03-15T13:27:36Z')`<br /><br /> **Numéro du paramètre** : 1<br /><br /> **Nom** : horodateur<br /><br /> **Description** : obligatoire. Chaîne qui contient l’heure.| 
|dayOfMonth|Retourne le jour de mois d’un horodateur de type chaîne. Exemple `15` :<br /><br /> `dayOfMonth('2017-03-15T13:27:36Z')`<br /><br /> **Numéro du paramètre** : 1<br /><br /> **Nom** : horodateur<br /><br /> **Description** : obligatoire. Chaîne qui contient l’heure.| 
|dayOfYear|Retourne le jour d’année d’un horodateur de type chaîne. Exemple `74` :<br /><br /> `dayOfYear('2017-03-15T13:27:36Z')`<br /><br /> **Numéro du paramètre** : 1<br /><br /> **Nom** : horodateur<br /><br /> **Description** : obligatoire. Chaîne qui contient l’heure.| 
|ticks|Retourne la propriété ticks d’un horodateur de type chaîne. Exemple `1489603019` :<br /><br /> `ticks('2017-03-15T18:36:59Z')`<br /><br /> **Numéro du paramètre** : 1<br /><br /> **Nom** : horodateur<br /><br /> **Description** : obligatoire. Chaîne qui contient l’heure.| 
  
### <a name="workflow-functions"></a>Fonctions de flux de travail  

Ces fonctions vous permettent d’obtenir des informations sur le flux de travail au moment de l’exécution.  
  
|Nom de la fonction|Description|  
|-------------------|-----------------|  
|listCallbackUrl|Retourne une chaîne à appeler pour appeler le déclencheur ou l’action. <p> **Remarque** : cette fonction peut être utilisée uniquement dans **httpWebhook** et **apiConnectionWebhook**, et pas dans **manuel**, **recurrence**, **http** ou **apiConnection**. <p>Par exemple, la fonction `listCallbackUrl()` retourne : <p>`https://prod-01.westus.logic.azure.com:443/workflows/1235...ABCD/triggers/manual/run?api-version=2015-08-01-preview&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=xxx...xxx` |  
|workflow|Cette fonction vous fournit tous les détails du flux de travail à l’exécution. Vous pouvez obtenir toutes les informations disponibles dans l’API de gestion, par exemple le nom, l’emplacement et l’ID de ressource. Pour en savoir plus sur ces propriétés, consultez l’article sur [l’API Rest](http://go.microsoft.com/fwlink/p/?LinkID=525617). Par exemple, cette fonction retourne un emplacement comme `westus` : <p>`workflow().location` <p> **Remarque** : `@workflow` est actuellement pris en charge dans les déclencheurs.|

## <a name="next-steps"></a>Étapes suivantes

[Actions et déclencheurs de flux de travail](logic-apps-workflow-actions-triggers.md)


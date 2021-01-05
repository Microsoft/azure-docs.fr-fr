---
title: Référence de schéma de langage de définition de flux de travail
description: Guide de référence du schéma JSON et de la syntaxe du langage de définition de flux de travail, qui décrit les flux de travail dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 05/13/2019
ms.openlocfilehash: 71929cd449f4a00b91cc6c8620b33b0e0c6d506c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87078139"
---
# <a name="schema-reference-guide-for-the-workflow-definition-language-in-azure-logic-apps"></a>Guide de référence du schéma du langage de définition de workflow dans Azure Logic Apps

Quand vous créez une application logique dans [Azure Logic Apps](../logic-apps/logic-apps-overview.md), elle dispose d’une définition de flux de travail sous-jacente qui décrit la logique réelle qui s’exécute dans votre application logique. Cette définition de flux de travail utilise [JSON](https://www.json.org/) et suit une structure validée par le schéma de langage de définition de flux de travail. Cette référence fournit une vue d’ensemble de cette structure et de la façon dont le schéma définit les attributs dans votre définition de flux de travail.

## <a name="workflow-definition-structure"></a>Structure d’une définition de flux de travail

Une définition de flux de travail inclut toujours un déclencheur afin d’instancier votre application logique, ainsi qu’une ou plusieurs actions qui s’exécutent après le déclencheur.

Voici la structure de haut niveau d’une définition de flux de travail :

```json
"definition": {
  "$schema": "<workflow-definition-language-schema-version>",
  "actions": { "<workflow-action-definitions>" },
  "contentVersion": "<workflow-definition-version-number>",
  "outputs": { "<workflow-output-definitions>" },
  "parameters": { "<workflow-parameter-definitions>" },
  "staticResults": { "<static-results-definitions>" },
  "triggers": { "<workflow-trigger-definitions>" }
}
```

| Attribut | Obligatoire | Description |
|-----------|----------|-------------|
| `definition` | Oui | Élément de départ de votre définition de flux de travail |
| `$schema` | Uniquement en cas de référence externe à une définition de flux de travail | Emplacement du fichier de schéma JSON qui décrit la version du langage de définition de flux de travail, que vous pouvez trouver ici : <p>`https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json`</p> |
| `actions` | Non | Définitions d’une ou plusieurs actions à exécuter lors de l’exécution du flux de travail. Pour plus d’informations, consultez [Déclencheurs et actions](#triggers-actions). <p><p>Nombre maximal d'actions : 250 |
| `contentVersion` | Non | Numéro de version de votre définition de flux de travail (1.0.0.0 par défaut). Pour identifier et vérifier plus facilement la définition correcte lors du déploiement d’un flux de travail, spécifiez une valeur à utiliser. |
| `outputs` | Non | Les définitions des sorties à renvoyer lors de l’exécution d’un flux de travail. Pour plus d’informations, consultez la section [Sorties](#outputs). <p><p>Nombre maximal de sorties : 10 |
| `parameters` | Non | Les définitions pour un ou plusieurs paramètres qui passent les valeurs à utiliser au moment de l’exécution de votre application logique. Pour plus d’informations, consultez [Paramètres](#parameters). <p><p>Nombre maximal de paramètres : 50 |
| `staticResults` | Non | Définitions d’un ou plusieurs résultats statiques renvoyés par les actions comme sorties fictives lorsque des résultats statiques sont activés sur ces actions. Dans chaque définition d’action, l’attribut `runtimeConfiguration.staticResult.name` fait référence à la définition correspondante dans `staticResults`. Pour plus d’informations, consultez [Résultats statiques](#static-results). |
| `triggers` | Non | Définitions d’un ou plusieurs déclencheurs qui instancient votre flux de travail. Vous pouvez définir plus d’un déclencheur, mais uniquement avec le langage de définition de flux de travail (vous ne pouvez pas le faire visuellement via le Concepteur Logic Apps). Pour plus d’informations, consultez [Déclencheurs et actions](#triggers-actions). <p><p>Nombre maximal de déclencheurs : 10 |
||||

<a name="triggers-actions"></a>

## <a name="triggers-and-actions"></a>Déclencheurs et actions

Dans une définition de flux de travail, les sections `triggers` et `actions` définissent les appels qui se produisent pendant l’exécution de votre flux de travail. Pour obtenir des informations supplémentaires sur ces sections et connaître la syntaxe à utiliser, consultez [Déclencheurs et actions du flux de travail](../logic-apps/logic-apps-workflow-actions-triggers.md).

<a name="parameters"></a>

## <a name="parameters"></a>Paramètres

Le cycle de vie du déploiement a généralement des environnements différents pour le développement, le test, la mise en lots et la production. Lorsque vous déployez des applications logiques dans différents environnements, vous souhaiterez probablement utiliser des valeurs différentes, telles que des chaînes de connexion, en fonction de vos besoins en matière de déploiement. Ou bien, vous pouvez avoir des valeurs que vous souhaitez réutiliser dans votre application logique sans codage en dur ou codes qui changent souvent. Dans la section `parameters` de votre définition de flux de travail, vous pouvez définir ou modifier des paramètres pour les valeurs que votre application logique utilise au moment de l’exécution. Vous devez d’abord définir ces paramètres avant de pouvoir référencer ces paramètres ailleurs dans votre définition de flux de travail.

Voici la structure générale d’une définition de paramètre :

```json
"parameters": {
   "<parameter-name>": {
      "type": "<parameter-type>",
      "defaultValue": <default-parameter-value>,
      "allowedValues": [ <array-with-permitted-parameter-values> ],
      "metadata": {
         "description": "<parameter-description>"
      }
   }
},
```

| Attribut | Obligatoire | Type | Description |
|-----------|----------|------|-------------|
| <*parameter-name*> | Oui | String | Le nom du paramètre que vous voulez définir |
| <*parameter-type*> | Oui | int, float, string, bool, array, object, securestring, secureobject <p><p>**Remarque** : Pour tous les mots de passe, les clés et les secrets, utilisez les types `securestring` et `secureobject` car l'opération `GET` ne renvoie pas ces types. Pour plus d’informations sur la sécurisation des paramètres, consultez [Recommandations de sécurité pour les paramètres d’action et d’entrée](../logic-apps/logic-apps-securing-a-logic-app.md#secure-action-parameters). | Type du paramètre |
| <*default-parameter-value*> | Oui | Identique à `type` | La valeur par défaut du paramètre quand aucune valeur n’est spécifiée lors de l’instanciation du flux de travail. L’attribut `defaultValue` est requis pour que le concepteur d’applications logiques puisse afficher correctement le paramètre, mais vous pouvez spécifier une valeur vide. |
| <*array-with-permitted-parameter-values*> | Non | Array | Tableau regroupant les valeurs que le paramètre peut accepter |
| <*parameter-description*> | Non | Objet JSON | Tous les autres détails des paramètres, tels que la description du paramètre |
||||

Ensuite, créez un [modèle Azure Resource Manager](../azure-resource-manager/templates/overview.md) pour votre définition de flux de travail, définissez les paramètres de modèle qui acceptent les valeurs que vous souhaitez au moment du déploiement, remplacez les valeurs codées en dur par des références au modèle ou à la définition du flux de travail et stockez les valeurs à utiliser au moment du déploiement dans un [fichier de paramètres](../azure-resource-manager/templates/parameter-files.md) distinct. De cette façon, vous pouvez modifier ces valeurs plus facilement par le biais du fichier de paramètres sans avoir à mettre à jour et à redéployer votre application logique. Pour les informations sensibles ou qui doivent être sécurisées, telles que des mots de passe et des secrets, vous pouvez les stocker dans Azure Key Vault et faire en sorte que votre fichier de paramètres récupère ces valeurs de votre coffre de clés. Pour plus d’informations et des exemples sur la définition des paramètres au niveau du modèle et de la définition du flux de travail, consultez [Vue d’ensemble : Automatiser le déploiement pour les applications logiques avec des modèles Azure Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md).

<a name="static-results"></a>

## <a name="static-results"></a>Résultats statiques

Dans l’attribut `staticResults`, définissez une sortie `outputs` et `status` fictives que l’action renvoie lorsque le paramètre de résultat statique de l’action est activé. Dans la définition de l’action, l’attribut `runtimeConfiguration.staticResult.name` fait référence au nom de la définition de résultat statique dans `staticResults`. Découvrez comment [Tester des applications logiques avec des données fictives en configurant des résultats statiques](../logic-apps/test-logic-apps-mock-data-static-results.md).

```json
"definition": {
   "$schema": "<...>",
   "actions": { "<...>" },
   "contentVersion": "<...>",
   "outputs": { "<...>" },
   "parameters": { "<...>" },
   "staticResults": {
      "<static-result-definition-name>": {
         "outputs": {
            <output-attributes-and-values-returned>,
            "headers": { <header-values> },
            "statusCode": "<status-code-returned>"
         },
         "status": "<action-status>"
      }
   },
   "triggers": { "<...>" }
}
```

| Attribut | Obligatoire | Type | Description |
|-----------|----------|------|-------------|
| <*static-result-definition-name*> | Oui | String | Nom de la définition de résultat statique qu’une définition d’action peut référencer via un objet `runtimeConfiguration.staticResult`. Pour plus d’informations, consultez [Paramètres de configuration d’exécution](../logic-apps/logic-apps-workflow-actions-triggers.md#runtime-config-options). <p>Vous pouvez utiliser n’importe quel nom unique. Par défaut, ce nom unique est ajouté avec un nombre, qui est incrémenté si nécessaire. |
| <*output-attributes-and-values-returned*> | Oui | Variable | Les configurations requises pour ces attributs varient selon différentes conditions. Par exemple, lorsque l’attribut `status` est `Succeeded`, l’attribut `outputs` inclut les attributs et les valeurs renvoyées comme sorties fictives par l’action. Si l’attribut `status` est `Failed`, l’attribut `outputs` inclut l’attribut `errors`, qui est un tableau avec un ou plusieurs objets `message` avec des informations erronées. |
| <*header-values*> | Non | JSON | Toute valeur renvoyée par l’action |
| <*status-code-returned*> | Oui | String | Code d’état retourné par l’action |
| <*action-status*> | Oui | String | État de l’action, par exemple, `Succeeded` ou `Failed` |
|||||

Par exemple, dans la définition de l’action HTTP, l’attribut `runtimeConfiguration.staticResult.name` fait référence à `HTTP0` dans l’attribut `staticResults`, où les sorties fictives de l’action sont définies. L’attribut `runtimeConfiguration.staticResult.staticResultOptions` spécifie le paramètre du résultat statique `Enabled` sur l’action HTTP.

```json
"actions": {
   "HTTP": {
      "inputs": {
         "method": "GET",
         "uri": "https://www.microsoft.com"
      },
      "runAfter": {},
      "runtimeConfiguration": {
         "staticResult": {
            "name": "HTTP0",
            "staticResultOptions": "Enabled"
         }
      },
      "type": "Http"
   }
},
```

L’action HTTP retourne les sorties dans la définition `HTTP0` dans `staticResults`. Dans cet exemple, pour le code d’état, la sortie fictive est `OK`. Pour les valeurs d’en-tête, la sortie fictive est `"Content-Type": "application/JSON"`. Pour l’état de l’action, la sortie fictive est `Succeeded`.

```json
"definition": {
   "$schema": "<...>",
   "actions": { "<...>" },
   "contentVersion": "<...>",
   "outputs": { "<...>" },
   "parameters": { "<...>" },
   "staticResults": {
      "HTTP0": {
         "outputs": {
            "headers": {
               "Content-Type": "application/JSON"
            },
            "statusCode": "OK"
         },
         "status": "Succeeded"
      }
   },
   "triggers": { "<...>" }
},
```

<a name="expressions"></a>

## <a name="expressions"></a>Expressions

Avec JSON, vous pouvez avoir des valeurs littérales qui existent au moment du design, par exemple :

```json
"customerName": "Sophia Owen",
"rainbowColors": ["red", "orange", "yellow", "green", "blue", "indigo", "violet"],
"rainbowColorsCount": 7
```

Vous pouvez également avoir des valeurs qui n’existent pas avant l’exécution. Pour représenter ces valeurs, vous pouvez utiliser des *expressions*, qui sont évaluées au moment de l’exécution. Une expression est une séquence qui peut contenir un ou plusieurs [opérateurs](#functions), [fonctions](#operators), [variables](./logic-apps-create-variables-store-values.md), valeurs explicites ou constantes. Dans votre définition de flux de travail, vous pouvez utiliser une expression n’importe où dans une valeur de chaîne JSON en faisant précéder l’expression d’une arobase (\@). Quand une expression qui représente une valeur JSON est évaluée, le corps de l’expression est extrait en supprimant le caractère \@, et une autre valeur JSON est systématiquement générée.

Par exemple, pour la propriété `customerName` définie précédemment, vous pouvez obtenir la valeur de la propriété en utilisant la fonction [parameters()](../logic-apps/workflow-definition-language-functions-reference.md#parameters) dans une expression de fonction et en assignant cette valeur à la propriété `accountName` :

```json
"customerName": "Sophia Owen",
"accountName": "@parameters('customerName')"
```

L’*interpolation de chaîne* permet également d’utiliser plusieurs expressions placées entre le caractère \@ et des accolades ({}) à l’intérieur d’une chaîne. Voici la syntaxe :

```json
@{ "<expression1>", "<expression2>" }
```

Le résultat est toujours une chaîne, ce qui rend cette fonctionnalité similaire à la fonction `concat()`, par exemple : 

```json
"customerName": "First name: @{parameters('firstName')} Last name: @{parameters('lastName')}"
```

Si vous avez une chaîne littérale qui commence par le caractère \@, faites précéder le caractère \@ d’un autre caractère \@ faisant office de caractère d’échappement : \@\@

Ces exemples montrent comment les expressions sont évaluées :

| Valeur JSON | Résultats |
|------------|--------|
| "Sophia Owen" | Renvoie les caractères suivants : 'Sophia Owen' |
| "array[1]" | Renvoie ces caractères : « array[1] » |
| "\@\@" | Renvoie ces caractères sous forme de chaîne de un caractère : « \@ » |
| " \@" | Renvoie ces caractères sous forme de chaîne de deux caractères : «  \@ » |
|||

Pour ces exemples, supposons que vous définissiez "January" pour "myBirthMonth" et le nombre 42 pour "myAge" :

```json
"myBirthMonth": "January",
"myAge": 42
```

Ces exemples montrent comment les expressions suivantes sont évaluées :

| Expression JSON | Résultats |
|-----------------|--------|
| "\@parameters('myBirthMonth')" | Renvoie la chaîne suivante : "January" |
| "\@{parameters('myBirthMonth')}" | Renvoie la chaîne suivante : "January" |
| "\@parameters('myAge')" | Renvoie le nombre suivant : 42 |
| "\@{parameters('myAge')}" | Renvoie le nombre suivant sous forme de chaîne : "42" |
| "My age is \@{parameters('myAge')}" | Renvoie la chaîne suivante : "My age is 42" |
| "\@concat('My age is ', string(parameters('myAge')))" | Renvoie la chaîne suivante : "My age is 42" |
| "My age is \@\@{parameters('myAge')}" | Renvoie la chaîne suivante, qui inclut l'expression : "My age is \@{parameters('myAge')}` |
|||

Quand vous travaillez visuellement dans le Concepteur Logic Apps, vous pouvez créer des expressions via le Générateur d’expressions, par exemple :

![Concepteur d’applications logiques > Générateur d’expressions](./media/logic-apps-workflow-definition-language/expression-builder.png)

Quand vous avez terminé, l’expression apparaît pour la propriété correspondante dans votre définition de flux de travail, par exemple, la propriété `searchQuery` ici :

```json
"Search_tweets": {
  "inputs": {
    "host": {
      "connection": {
        "name": "@parameters('$connections')['twitter']['connectionId']"
      }
    }
  },
  "method": "get",
  "path": "/searchtweets",
  "queries": {
    "maxResults": 20,
    "searchQuery": "Azure @{concat('firstName','', 'LastName')}"
  }
},
```

<a name="outputs"></a>

## <a name="outputs"></a>Outputs

Dans la section `outputs`, définissez les données que votre flux de travail peut renvoyer une fois son exécution terminée. Par exemple, pour suivre un état ou une valeur spécifique à chaque exécution, spécifiez le renvoi de cette donnée par la sortie du flux de travail.

> [!NOTE]
> Pour la réponse à des requêtes entrantes à partir de l’API REST d’un service, n’utilisez pas `outputs`. Utilisez le type d’action `Response` à la place. Pour plus d’informations, consultez [Déclencheurs et actions du flux de travail](../logic-apps/logic-apps-workflow-actions-triggers.md).

Voici la structure générale d’une définition de sortie :

```json
"outputs": {
  "<key-name>": {
    "type": "<key-type>",
    "value": "<key-value>"
  }
}
```

| Attribut | Obligatoire | Type | Description |
|-----------|----------|------|-------------|
| <*key-name*> | Oui | String | Nom de la clé de la valeur renvoyée pour la sortie |
| <*key-type*> | Oui | int, float, string, securestring, bool, array, objet JSON | Type de la valeur renvoyée pour la sortie |
| <*key-value*> | Oui | Identique à <*key-type*> | Valeur renvoyée pour la sortie |
|||||

Pour obtenir la sortie d’une exécution de flux de travail, examinez l’historique et les détails des exécutions de votre application logique dans le portail Azure ou utilisez l’[API REST de flux de travail](/rest/api/logic/workflows). Vous pouvez également transmettre la sortie à des systèmes externes, par exemple Power BI, afin de créer des tableaux de bord.

<a name="operators"></a>

## <a name="operators"></a>Opérateurs

Dans les [expressions](#expressions) et les [fonctions](#functions), les opérateurs effectuent des tâches spécifiques, telles que référencer une propriété ou une valeur dans un tableau.

| Opérateur | Tâche |
|----------|------|
| ' | Pour utiliser une chaîne littérale en tant qu’entrée ou dans des expressions et des fonctions, vous devez placer la chaîne uniquement entre des guillemets simples, par exemple, `'<myString>'`. N’utilisez pas de guillemets doubles (""), qui entrent en conflit avec le formatage JSON autour d’une expression entière. Par exemple : <p>**Oui** : length('Hello') </br>**Non** : length("Hello") <p>Quand vous transmettez des tableaux ou des nombres, vous n’avez pas besoin de les placer entre des signes de ponctuation. Par exemple : <p>**Oui** : length([1, 2, 3]) </br>**Non** : length("[1, 2, 3]") |
| [] | Pour référencer une valeur à une position spécifique (index) dans un tableau, utilisez des crochets. Par exemple, pour obtenir le deuxième élément d’un tableau : <p>`myArray[1]` |
| . | Pour référencer une propriété d’un objet, utilisez l’opérateur point. Par exemple, pour obtenir la propriété `name` d’un objet JSON `customer` : <p>`"@parameters('customer').name"` |
| ? | Pour référencer les propriétés Null d’un objet sans erreur d’exécution, utilisez l’opérateur point d’interrogation. Par exemple, pour gérer les sorties Null d’un déclencheur, vous pouvez utiliser cette expression : <p>`@coalesce(trigger().outputs?.body?.<someProperty>, '<property-default-value>')` |
|||

<a name="functions"></a>

## <a name="functions"></a>Fonctions

Certaines expressions obtiennent leurs valeurs à partir d’actions runtime qui peuvent ne pas encore exister au début de l’exécution de votre définition de flux de travail. Pour référencer ou utiliser ces valeurs dans des expressions, vous pouvez faire appel à des [*fonctions*](../logic-apps/workflow-definition-language-functions-reference.md) fournies par le Langage de définition de workflow.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [actions et déclencheurs du langage de définition de flux de travail](../logic-apps/logic-apps-workflow-actions-triggers.md)
* En savoir plus sur la création et la gestion par programmation des applications logiques avec l’[API REST de flux de travail](/rest/api/logic/workflows)

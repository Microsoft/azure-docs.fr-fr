---
title: Gérer les types de contenu
description: Découvrez comment gérer différents types de contenu dans les flux de travail au moment de la conception et au moment de l’exécution dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 07/20/2018
ms.openlocfilehash: 04642e69adba82cf33b933829d72c51109ad2ee1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91578177"
---
# <a name="handle-content-types-in-azure-logic-apps"></a>Gérer les types de contenu dans Azure Logic Apps

De nombreux types de contenu peuvent transiter par une application logique, notamment les données binaires, les fichiers plats, ainsi que les contenus XML et JSON. Pendant que Logic Apps prend en charge tous les types de contenu, certains ont une prise en charge native et ne nécessitent pas de transtypage ou de conversion dans vos applications logiques. D’autres types peuvent nécessiter un transtypage ou une conversion en fonction des besoins. Cet article décrit comment Logic Apps gère les types de contenu et comment vous pouvez caster ou convertir correctement ces types lorsque cela est nécessaire.

Pour déterminer la méthode appropriée pour la gestion des types de contenu, Logic Apps s’appuie sur la valeur de l’en-tête `Content-Type` dans les appels HTTP, par exemple :

* [application/json](#application-json) (type natif)
* [texte/brut](#text-plain) (type natif)
* [application/xml and application/octet-stream](#application-xml-octet-stream)
* [Autres types de contenu](#other-content-types)

<a name="application-json"></a>

## <a name="applicationjson"></a>application/json

Logic Apps stocke et gère n’importe quelle requête dont le type *application/json* en tant qu’objet JavaScript Notation (JSON). Par défaut, vous pouvez analyser le contenu JSON sans aucun transtypage. Pour analyser une requête comportant un en-tête avec le type de contenu « application/json », vous pouvez utiliser une expression. Cet exemple retourne la valeur `dog` à partir du tableau `animal-type` sans transtypage : 
 
`@body('myAction')['animal-type'][0]` 
  
  ```json
  {
    "client": {
       "name": "Fido",
       "animal-type": [ "dog", "cat", "rabbit", "snake" ]
    }
  }
  ```

Si vous travaillez avec des données JSON qui ne spécifient pas un en-tête, vous pouvez convertir manuellement ces données au format JSON à l’aide de la [fonction json()](../logic-apps/workflow-definition-language-functions-reference.md#json), par exemple : 
  
`@json(triggerBody())['animal-type']`

### <a name="create-tokens-for-json-properties"></a>Créer des jetons pour les propriétés JSON

Logic Apps offre la possibilité de générer des jetons conviviaux qui représentent les propriétés dans le contenu JSON afin de pouvoir référencer et utiliser ces propriétés plus facilement dans le flux de travail de votre application logique.

* **Déclencheur de requête**

  Lorsque vous utilisez ce déclencheur dans le Concepteur d’application logique, vous pouvez fournir un schéma JSON qui décrit la charge utile que vous comptez recevoir. 
  Le concepteur analyse le contenu JSON à l’aide de ce schéma et génère des jetons conviviaux qui représentent les propriétés dans votre contenu JSON. 
  Vous pouvez ensuite facilement référencer et utiliser ces propriétés dans l’ensemble de flux de travail de votre application logique. 
  
  Si vous n’avez pas un schéma, vous pouvez en générer un. 
  
  1. Dans le déclencheur de requête, sélectionnez **Utiliser l’exemple de charge utile pour générer le schéma**.  
  
  2. Sous **Entrer ou coller un exemple de charge utile JSON**, fournissez un exemple de charge utile, puis choisissez **Terminé**. Par exemple : 

     ![Capture d’écran montrant l’action « Lors de la réception d’une requête HTTP » avec un exemple de charge utile JSON.](./media/logic-apps-content-type/request-trigger.png)

     Le schéma généré apparaît désormais dans votre déclencheur.

     ![Fournir l’exemple de charge utile JSON](./media/logic-apps-content-type/generated-schema.png)

     Voici la définition sous-jacente pour votre déclencheur de requête dans l’éditeur en mode Code :

     ```json
     "triggers": { 
        "manual": {
           "type": "Request",
           "kind": "Http",
           "inputs": { 
              "schema": {
                 "type": "object",
                 "properties": {
                    "client": {
                       "type": "object",
                       "properties": {
                          "animal-type": {
                             "type": "array",
                             "items": {
                                "type": "string"
                             },
                          },
                          "name": {
                             "type": "string"
                          }
                       }
                    }
                 }
              }
           }
        }
     }
     ```

  3. Dans votre requête, veillez à inclure un en-tête `Content-Type` et à définir la valeur de l’en-tête sur `application/json`.

* **Action d’analyse de JSON**

  Lorsque vous utilisez cette action dans le Concepteur d’application logique, vous pouvez analyser la sortie JSON et générer des jetons conviviaux qui représentent les propriétés dans votre contenu JSON. 
  Vous pouvez ensuite facilement référencer et utiliser ces propriétés dans l’ensemble de flux de travail de votre application logique. À l’instar du déclencheur de requête, vous pouvez fournir ou générer un schéma JSON qui décrit le contenu JSON que vous souhaitez analyser. 
  De cette façon, vous pouvez utiliser plus facilement les données à partir d’Azure Service Bus, Azure Cosmos DB, et ainsi de suite.

  ![Analyse JSON](./media/logic-apps-content-type/parse-json.png)

<a name="text-plain"></a>

## <a name="textplain"></a>texte/brut

Lorsque votre application logique reçoit des messages HTTP qui ont l’en-tête `Content-Type` défini sur `text/plain`, votre application logique stocke ces messages au format brut. Si vous incluez ces messages dans les actions suivantes sans transtypage, ces demandes sortent avec l’en-tête `Content-Type` défini sur `text/plain`. 

Par exemple, lorsque vous travaillez avec un fichier plat, vous pouvez obtenir une requête HTTP avec l’ `Content-Type` en-tête défini sur `text/plain` type de contenu :

`Date,Name,Address`</br>
`Oct-1,Frank,123 Ave`

Si vous envoyez ensuite cette demande dans une action ultérieure dans le corps d’une autre demande, par exemple, `@body('flatfile')`, cette seconde requête possède également un en-tête `Content-Type` qui est défini sur `text/plain`. Si vous travaillez avec des données qui sont du texte brut mais sans en-tête spécifié, vous pouvez caster manuellement ces données au texte à l’aide de la [fonction string()](../logic-apps/workflow-definition-language-functions-reference.md#string) telle que cette expression : 

`@string(triggerBody())`

<a name="application-xml-octet-stream"></a>

## <a name="applicationxml-and-applicationoctet-stream"></a>application/xml et application/octet-stream

Logic Apps conserve toujours le `Content-Type` dans une requête HTTP ou une réponse reçues. Par conséquent, si votre application logique reçoit le contenu avec le `Content-Type` défini sur `application/octet-stream`, et que vous incluez le contenu dans une action ultérieure sans transtypage, la requête sortante a également `Content-Type` défini sur `application/octet-stream`. De cette façon, Logic Apps peut garantir que les données ne se perdent pas lors du déplacement dans le flux de travail. Toutefois, les états d’action, ou les entrées et sorties, sont parallèlement stockés dans un objet JSON tandis que l’état se déplace dans le flux de travail. 

## <a name="converter-functions"></a>Fonctions de conversion

Pour conserver certains types de données, Logic Apps convertit le contenu en une chaîne binaire encodée en base64, avec les métadonnées appropriées, qui conservent aussi bien la charge utile `$content` et le `$content-type`, qui sont automatiquement convertis. 

Cette liste décrit comment Logic Apps convertit le contenu lorsque vous utilisez ces [fonctions](../logic-apps/workflow-definition-language-functions-reference.md) :

* `json()`: caste les données en `application/json`
* `xml()`: caste les données en `application/xml`
* `binary()`: caste les données en `application/octet-stream`
* `string()`: caste les données en `text/plain`
* `base64()`: convertit le contenu en une chaîne encodée en base64
* `base64toString()`: convertit une chaîne encodée en base64 en `text/plain`
* `base64toBinary()`: convertit une chaîne encodée en base64 en `application/octet-stream`
* `dataUri()`: convertit une chaîne en un URI de données
* `dataUriToBinary()`: convertit un URI de données en une chaîne binaire
* `dataUriToString()`: convertit un URI de données en une chaîne

Par exemple, si vous recevez une requête HTTP où `Content-Type` est défini sur `application/xml`, tel que ce contenu :

```html
<?xml version="1.0" encoding="UTF-8" ?>
<CustomerName>Frank</CustomerName>
```

Vous pouvez caster ce contenu à l’aide de l’expression `@xml(triggerBody())` avec les fonctions `xml()` et `triggerBody()` pour ensuite utiliser ce contenu ultérieurement. Vous pouvez également utiliser l’expression `@xpath(xml(triggerBody()), '/CustomerName')` avec les fonctions `xpath()` et `xml()`. 

## <a name="other-content-types"></a>Autres types de contenu

Logic Apps fonctionne avec et prend en charge d’autres types de contenu, mais peut nécessiter que vous obteniez manuellement le corps du message en décodant la variable `$content`.

Par exemple, supposons que votre application logique est déclenchée par une requête avec le type de contenu `application/x-www-url-formencoded`. Pour conserver toutes les données, la variable `$content` dans le corps de la requête a une charge utile qui est encodée sous forme de chaîne en base64 :

`CustomerName=Frank&Address=123+Avenue`

Étant donné que la demande n’est pas en texte brut ou JSON, elle est stockée dans l’action comme suit :

```json
"body": {
   "$content-type": "application/x-www-url-formencoded",
   "$content": "AAB1241BACDFA=="
}
```

Logic Apps offre des fonctions natives pour la gestion des données de formulaire, par exemple : 

* [triggerFormDataValue()](../logic-apps/workflow-definition-language-functions-reference.md#triggerFormDataValue)
* [triggerFormDataMultiValues()](../logic-apps/workflow-definition-language-functions-reference.md#triggerFormDataMultiValues)
* [formDataValue()](../logic-apps/workflow-definition-language-functions-reference.md#formDataValue) 
* [formDataMultiValues()](../logic-apps/workflow-definition-language-functions-reference.md#formDataMultiValues)

Ou bien, vous pouvez manuellement accéder aux données à l’aide d’une expression telle que cet exemple :

`@string(body('formdataAction'))` 

Pour que la demande sortante ait également le même type de contenu d’en-tête `application/x-www-url-formencoded`, vous pouvez ajouter la requête au corps de l’action sans transtypage en utilisant une expression telle que `@body('formdataAction')`. Toutefois, cette méthode ne fonctionne que si le corps est le seul paramètre dans l’entrée `body`. Si vous essayez d’utiliser l’expression `@body('formdataAction')` dans une requête `application/json`, vous obtenez une erreur d’exécution, car le corps est envoyé codé.

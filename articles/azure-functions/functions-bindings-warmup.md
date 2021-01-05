---
title: Déclencheur de préchauffage Azure Functions
description: Découvrez comment utiliser le déclencheur de préchauffage dans Azure Functions.
documentationcenter: na
author: craigshoemaker
manager: gwallace
keywords: azure functions, fonctions, traitement des événements, préchauffage, démarrage à froid, premium, calcul dynamique, architecture serverless
ms.service: azure-functions
ms.topic: reference
ms.custom: devx-track-csharp
ms.date: 11/08/2019
ms.author: cshoe
ms.openlocfilehash: 6a909b246e7cb3695533a75294c523d7c9aeaeda
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94832940"
---
# <a name="azure-functions-warm-up-trigger"></a>Déclencheur de préchauffage Azure Functions

Cet article explique comment utiliser le déclencheur de préchauffage dans Azure Functions. Le déclencheur de préchauffage est pris en charge uniquement pour les applications de fonction qui s’exécutent dans un [plan Premium](functions-premium-plan.md). Un déclencheur de préchauffage est appelé lorsqu’une instance est ajoutée pour mettre à l’échelle une application de fonction en cours d’exécution. Vous pouvez utiliser un déclencheur de préchauffage pour précharger des dépendances personnalisées pendant le [processus de préchauffage](./functions-premium-plan.md#pre-warmed-instances) afin que vos fonctions soient prêtes à commencer le traitement des requêtes immédiatement. 

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

## <a name="packages---functions-2x-and-higher"></a>Packages – Functions 2.x et versions ultérieures

Le package NuGet [Microsoft.Azure.WebJobs.Extensions](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions), version **3.0.5 ou ultérieure**, est requis. Le code source du package se trouve dans le référentiel GitHub [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.Http/). 

[!INCLUDE [functions-package](../../includes/functions-package-auto.md)]

## <a name="trigger"></a>Déclencheur

Le déclencheur de préchauffage vous permet de définir une fonction qui est exécutée sur une nouvelle instance quand elle est ajoutée à votre application en cours d’exécution. Vous pouvez utiliser une fonction de préchauffage pour ouvrir des connexions, charger des dépendances ou exécuter toute autre logique personnalisée avant que votre application ne commence à recevoir le trafic. 

Le déclencheur de préchauffage est destiné à créer des dépendances partagées qui seront utilisées par les autres fonctions de application. [Consultez des exemples de dépendances partagées ici](./manage-connections.md#client-code-examples).

Notez que le déclencheur de préchauffage est uniquement appelé pendant les opérations de scale-out et non lors des redémarrages ou d’autres démarrages sans échelle. Vous devez vous assurer que votre logique peut charger toutes les dépendances nécessaires sans utiliser le déclencheur de préchauffage. Le chargement différé est un bon modèle pour y parvenir.

## <a name="trigger---example"></a>Déclencheur - exemple

# <a name="c"></a>[C#](#tab/csharp)

L’exemple suivant montre une [fonction C#](functions-dotnet-class-library.md) qui s’exécutera sur chaque nouvelle instance quand elle sera ajoutée à votre application. Aucun attribut de valeur renvoyée n’est requis.


* Votre fonction doit être nommée ```warmup``` (non sensible à la casse) et il ne peut y avoir qu’une seule fonction de préchauffage par application.
* Pour utiliser le préchauffage comme une fonction de bibliothèque de classes .NET, vérifiez que vous disposez d’une référence de package à **Microsoft.Azure.WebJobs.Extensions >= 3.0.5**
    * ```<PackageReference Include="Microsoft.Azure.WebJobs.Extensions" Version="3.0.5" />```


Les commentaires d’espace réservé indiquent les emplacements dans l’application afin de déclarer et d’initialiser les dépendances partagées. 
[En savoir plus sur les dépendances partagées ici](./manage-connections.md#client-code-examples).

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Extensions.Logging;
 
namespace WarmupSample
{

    //Declare shared dependencies here

    public static class Warmup
    {
        [FunctionName("Warmup")]
        public static void Run([WarmupTrigger()] WarmupContext context,
            ILogger log)
        {
            //Initialize shared dependencies here
            
            log.LogInformation("Function App instance is warm 🌞🌞🌞");
        }
    }
}
```
# <a name="c-script"></a>[Script C#](#tab/csharp-script)


L’exemple suivant montre un déclencheur de préchauffage dans un fichier *function.json* et une [fonction de script C#](functions-reference-csharp.md) qui s’exécutera sur chaque nouvelle instance quand elle sera ajoutée à votre application.

Votre fonction doit être nommée ```warmup``` (non sensible à la casse), et il ne peut y avoir qu’une seule fonction de préchauffage par application.

Voici le fichier *function.json* :

```json
{
    "bindings": [
        {
            "type": "warmupTrigger",
            "direction": "in",
            "name": "warmupContext"
        }
    ]
}
```

La section [configuration](#trigger---configuration) décrit ces propriétés.

```cs
public static void Run(WarmupContext warmupContext, ILogger log)
{
    log.LogInformation("Function App instance is warm 🌞🌞🌞");  
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

L’exemple suivant montre un déclencheur de préchauffage dans un fichier *function.json* et une [fonction JavaScript](functions-reference-node.md) qui s’exécutera sur chaque nouvelle instance quand elle sera ajoutée à votre application.

Votre fonction doit être nommée ```warmup``` (non sensible à la casse) et il ne peut y avoir qu’une seule fonction de préchauffage par application.

Voici le fichier *function.json* :

```json
{
    "bindings": [
        {
            "type": "warmupTrigger",
            "direction": "in",
            "name": "warmupContext"
        }
    ]
}
```

La section [configuration](#trigger---configuration) décrit ces propriétés.

Voici le code JavaScript :

```javascript
module.exports = async function (context, warmupContext) {
    context.log('Function App instance is warm 🌞🌞🌞');
};
```

# <a name="python"></a>[Python](#tab/python)

L’exemple suivant montre un déclencheur de préchauffage dans un fichier *function.json* et une [fonction Python](functions-reference-python.md) qui s’exécutera sur chaque nouvelle instance quand elle sera ajoutée à votre application.

Votre fonction doit être nommée ```warmup``` (non sensible à la casse) et il ne peut y avoir qu’une seule fonction de préchauffage par application.

Voici le fichier *function.json* :

```json
{
    "bindings": [
        {
            "type": "warmupTrigger",
            "direction": "in",
            "name": "warmupContext"
        }
    ]
}
```

La section [configuration](#trigger---configuration) décrit ces propriétés.

Voici le code Python :

```python
import logging
import azure.functions as func


def main(warmupContext: func.Context) -> None:
    logging.info('Function App instance is warm 🌞🌞🌞')
```

# <a name="java"></a>[Java](#tab/java)

L’exemple suivant montre un déclencheur de préchauffage qui s’exécute lors de l’ajout d’une nouvelle instance à votre application.

Votre fonction doit être nommée `warmup` (non sensible à la casse) et il ne peut y avoir qu’une seule fonction de préchauffage par application.

```java
@FunctionName("Warmup")
public void run( ExecutionContext context) {
       context.getLogger().info("Function App instance is warm 🌞🌞🌞");
}
```

---

## <a name="trigger---attributes"></a>Déclencheur - attributs

Dans les [bibliothèques de classes C#](functions-dotnet-class-library.md), l’attribut `WarmupTrigger` est disponible pour configurer la fonction.

# <a name="c"></a>[C#](#tab/csharp)

Cet exemple montre comment utiliser l’attribut [warmup](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/dev/src/WebJobs.Extensions/Extensions/Warmup/Trigger/WarmupTriggerAttribute.cs).

Notez que votre fonction doit être appelée ```Warmup``` et qu’il ne peut y avoir qu’une seule fonction de préchauffage par application.

```csharp
 [FunctionName("Warmup")]
        public static void Run(
            [WarmupTrigger()] WarmupContext context, ILogger log)
        {
            ...
        }
```

Pour obtenir un exemple complet, consultez l’[exemple de déclencheur](#trigger---example).

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Les attributs ne sont pas pris en charge par le script C#.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Les attributs ne sont pas pris en charge par JavaScript.

# <a name="python"></a>[Python](#tab/python)

Les attributs ne sont pas pris en charge par Python.

# <a name="java"></a>[Java](#tab/java)

Le déclencheur de préchauffage n’est pas pris en charge dans Java en tant qu’attribut.

---

## <a name="trigger---configuration"></a>Déclencheur - configuration

Le tableau suivant décrit les propriétés de configuration de liaison que vous définissez dans le fichier *function.json* et l’attribut `WarmupTrigger`.

|Propriété function.json | Propriété d’attribut |Description|
|---------|---------|----------------------|
| **type** | n/a| Obligatoire : doit être défini sur `warmupTrigger`. |
| **direction** | n/a| Obligatoire : doit être défini sur `in`. |
| **name** | n/a| Obligatoire : nom de variable utilisé dans le code de fonction.|

## <a name="trigger---usage"></a>Déclencheur - utilisation

Aucune information supplémentaire n’est fournie à une fonction de déclencheur de préchauffage quand elle est appelée.

## <a name="trigger---limits"></a>Déclencheur : limites

* Le déclencheur de préchauffage est uniquement disponible pour les applications qui s’exécutent dans le cadre d’un [plan Premium](./functions-premium-plan.md).
* Le déclencheur de préchauffage est uniquement appelé pendant les opérations de scale-out, et non lors des redémarrages ou d’autres démarrages sans mise à l’échelle. Vous devez vous assurer que votre logique peut charger toutes les dépendances nécessaires sans utiliser le déclencheur de préchauffage. Le chargement différé est un bon modèle pour y parvenir.
* Le déclencheur de préchauffage ne peut pas être appelé une fois qu’une instance est déjà en cours d’exécution.
* Il ne peut y avoir qu’une seule fonction de déclencheur de préchauffage par application de fonction.

## <a name="next-steps"></a>Étapes suivantes

[En savoir plus sur les déclencheurs et les liaisons Azure Functions](functions-triggers-bindings.md)

---
title: Contrôle de version dans l’extension Fonctions durables - Azure
description: Découvrez comment implémenter le contrôle de version dans l’extension Fonctions durables d’Azure Functions.
author: cgillum
ms.topic: conceptual
ms.date: 11/03/2019
ms.author: azfuncdf
ms.openlocfilehash: 87cbb94dbab241630dc7585bdf4314d858d5b4da
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74232758"
---
# <a name="versioning-in-durable-functions-azure-functions"></a>Contrôle de version dans l’extension Fonctions durables (Azure Functions)

Au cours du cycle de vie d’une application, des fonctions seront nécessairement ajoutées, retirées ou modifiées. L’extension [Fonctions durables](durable-functions-overview.md) permet le chaînage des fonctions selon des méthodes auparavant indisponibles. Ce chaînage affectera votre mode de gestion des versions.

## <a name="how-to-handle-breaking-changes"></a>Gestion des modifications avec rupture

Il existe plusieurs exemples de modifications avec rupture. Cet article aborde les plus fréquents. Elles partagent toutes un point commun : les orchestrations de fonctions, tant nouvelles qu’existantes, sont affectées par les modifications apportées au code de la fonction.

### <a name="changing-activity-or-entity-function-signatures"></a>Modification des signatures des fonctions d’activité ou d’entité

Une modification de signature est une opération consistant à changer le nom, l’entrée et la sortie d’une fonction. Si cette modification est apportée à une fonction d’activité ou d’entité, elle peut décomposer une fonction d’orchestrateur qui dépend d’elle. Si vous mettez à jour la fonction d’orchestrateur de manière à intégrer cette modification, vous risquez de décomposer les instances en cours.

Par exemple, supposons que vous disposiez de la fonction d’orchestrateur suivante.

```csharp
[FunctionName("FooBar")]
public static Task Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    bool result = await context.CallActivityAsync<bool>("Foo");
    await context.CallActivityAsync("Bar", result);
}
```

Cette fonction simpliste récupère les résultats de **Foo** et les passe à **Bar**. Partons du principe que nous devons modifier la valeur renvoyée par **Foo**, en remplaçant `bool` par `int`, afin de prendre en charge un plus grand éventail de valeurs de résultats. Le résultat ressemble à ceci :

```csharp
[FunctionName("FooBar")]
public static Task Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    int result = await context.CallActivityAsync<int>("Foo");
    await context.CallActivityAsync("Bar", result);
}
```

> [!NOTE]
> Les exemples C# précédents ciblent Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser `DurableOrchestrationContext` au lieu de `IDurableOrchestrationContext`. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

Cette modification fonctionne pour toutes les nouvelles instances de la fonction d’orchestrateur, mais décompose toutes les instances en cours. Envisageons le cas où une instance d’orchestration appelle une fonction nommée `Foo`, récupère une valeur booléenne, puis des points de contrôle. Si la modification de signature est déployée à ce stade, l’instance faisant l’objet de points de contrôle échoue immédiatement, puis reprend et exécute à nouveau l’appel à `context.CallActivityAsync<int>("Foo")`. Cette erreur se produit car le résultat de la table d’historique est `bool`, mais le nouveau code tente de le désérialiser en `int`.

Cet exemple est une des différentes méthodes selon lesquelles une modification de signature peut décomposer des instances existantes. En général, si un orchestrateur doit modifier le mode d’appel d’une fonction, la modification risque de créer des problèmes.

### <a name="changing-orchestrator-logic"></a>Modification de la logique de l’orchestrateur

Si vous modifiez le code de la fonction d’orchestrateur de sorte à confondre la logique de réexécution des instances en cours, vous créez l’autre classe de problèmes liés au contrôle de version.

Prenons l’exemple de fonction d’orchestrateur suivante :

```csharp
[FunctionName("FooBar")]
public static Task Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    bool result = await context.CallActivityAsync<bool>("Foo");
    await context.CallActivityAsync("Bar", result);
}
```

Supposons à présent que vous souhaitiez apporter une modification apparemment innocente, consistant à ajouter un autre appel de fonction.

```csharp
[FunctionName("FooBar")]
public static Task Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    bool result = await context.CallActivityAsync<bool>("Foo");
    if (result)
    {
        await context.CallActivityAsync("SendNotification");
    }

    await context.CallActivityAsync("Bar", result);
}
```

> [!NOTE]
> Les exemples C# précédents ciblent Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser `DurableOrchestrationContext` au lieu de `IDurableOrchestrationContext`. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

Cette modification ajoute un nouvel appel de fonction au paramètre **SendNotification**, entre **Foo** et **Bar**. Aucune modification de signature n’est effectuée. Le problème survient lorsqu’une instance existante reprend à partir de l’appel à **bar**. Lors de la réexécution, si l’appel d’origine à **Foo** renvoie `true`, la réexécution de l’orchestrateur effectue l’appel dans **SendNotification**, qui n’est pas dans son historique d’exécution. De ce fait, l’infrastructure de tâche durable échoue en générant une exception `NonDeterministicOrchestrationException`, car elle a rencontré un appel à **SendNotification** alors qu’elle attendait un appel envoyé à **Bar**. Le même type de problème peut survenir lors de l’ajout d’appels à des API « durables », y compris `CreateTimer`, `WaitForExternalEvent`, etc.

## <a name="mitigation-strategies"></a>Stratégies d’atténuation

Voici quelques stratégies permettant de gérer les défis associés au contrôle de version :

* Ne rien faire
* Arrêter toutes les instances en cours
* Effectuer des déploiements côte à côte

### <a name="do-nothing"></a>Ne rien faire

Le moyen le plus simple de gérer une modification avec rupture consiste à laisser échouer les instances d’orchestration en cours. Les nouvelles instances exécutent correctement le code modifié.

L’importance de vos instances en cours détermine si ce type de défaillance est un problème. Si le développement est actif et si vous ne vous souciez pas des instances en cours, cela peut s’avérer suffisant. Toutefois, vous devez traiter les exceptions et les erreurs dans votre pipeline de diagnostics. Si vous souhaitez éviter cette opération, envisagez les autres options de contrôle de version.

### <a name="stop-all-in-flight-instances"></a>Arrêter toutes les instances en cours

Une autre option consiste à arrêter toutes les instances en cours. Pour arrêter toutes les instances, effacez le contenu des files d’attente **control-queue** et **workitem-queue**. Les instances restent bloquées à leur niveau, mais n’encombrent pas vos journaux avec des messages signalant des échecs. Cette approche permet d’accélérer efficacement le développement de prototypes.

> [!WARNING]
> Les détails de ces files d’attente peuvent changer au fil du temps. Pour cette raison, ne vous fiez pas à cette technique dans le cas de charges de travail de production.

### <a name="side-by-side-deployments"></a>Effectuer des déploiements côte à côte

La méthode la plus sûre pour assurer un déploiement sécurisé des modifications consiste à les installer côte à côte avec les versions plus anciennes. Pour cela, utilisez l’une des techniques suivantes :

* Déployer toutes les mises à jour en tant que nouvelles fonctions, en laissant les fonctions existantes telles quelles. Cette solution peut s’avérer compliqué, car les appelants des nouvelles versions de la fonction doivent être aussi mis à jour en suivant les mêmes consignes.
* Déployez toutes les mises à jour en tant que nouvelle application de fonction, en utilisant un autre compte de stockage.
* Déployer une nouvelle copie de l’application de fonction avec le même compte de stockage, mais en utilisant un nom `taskHub` mis à jour. Les déploiements côte à côte sont la technique recommandée.

### <a name="how-to-change-task-hub-name"></a>Comment modifier le nom du hub de tâches

Le hub de tâches peut être configuré dans le fichier *host.json*, comme suit :

#### <a name="functions-1x"></a>Functions 1.x

```json
{
    "durableTask": {
        "hubName": "MyTaskHubV2"
    }
}
```

#### <a name="functions-20"></a>Functions 2.0

```json
{
    "extensions": {
        "durableTask": {
            "hubName": "MyTaskHubV2"
        }
    }
}
```

La valeur par défaut de Durable Functions v1.x est `DurableFunctionsHub`. À compter de Durable Functions v2.0, le nom du hub de tâches par défaut est identique au nom de l’application de fonction dans Azure, ou à `TestHubName` en cas d’exécution en dehors d’Azure.

Toutes les entités de stockage Azure sont nommées en fonction de la valeur de configuration de l’élément `hubName`. En donnant au hub de tâches un nouveau nom, vous vous assurez qu’une table d’historique et des files d’attente distinctes sont créées pour la nouvelle version de votre application. L’application de fonction, par contre, arrête le traitement des événements pour les orchestrations ou les entités créées sous le précédent nom du hub de tâches.

Nous vous recommandons de déployer la nouvelle version de l’application de fonction vers un nouvel [emplacement de déploiement](../functions-deployment-slots.md). Les emplacements de déploiement vous permettent d’exécuter plusieurs instances de votre application de fonction et ce, en parallèle avec l’une d’elles seulement, en tant qu’emplacement de *production*. Lorsque vous êtes prêt à exposer la nouvelle logique d’orchestration dans l’infrastructure existante, cette opération peut s’avérer aussi simple que l’échange d’une nouvelle version dans l’emplacement de production.

> [!NOTE]
> Cette stratégie est d’autant plus efficace lorsque vous utilisez des déclencheurs Webhook et HTTP pour les fonctions d’orchestrateur. Pour les déclencheurs autres que HTTP, tels que les files d’attente ou les hubs d’événements, la définition du déclencheur doit [dériver d’un paramètre d’application](../functions-bindings-expressions-patterns.md#binding-expressions---app-settings) mis à jour dans le cadre de l’opération d’échange.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur la gestion des problèmes liés à la mise à l’échelle et aux performances](durable-functions-perf-and-scale.md)

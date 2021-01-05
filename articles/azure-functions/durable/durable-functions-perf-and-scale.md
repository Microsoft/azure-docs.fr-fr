---
title: Performances et mise à l’échelle dans Fonctions durables - Azure
description: Découvrez les caractéristiques de mise à l’échelle uniques de l’extension de Durable Functions pour Azure Functions.
author: cgillum
ms.topic: conceptual
ms.date: 11/03/2019
ms.author: azfuncdf
ms.openlocfilehash: b9fc465b5e5f132264fd36e004fa3ee7623b87a5
ms.sourcegitcommit: 48cb2b7d4022a85175309cf3573e72c4e67288f5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96854986"
---
# <a name="performance-and-scale-in-durable-functions-azure-functions"></a>Performances et mise à l’échelle dans Fonctions durables (Azure Functions)

Pour optimiser les performances et l’évolutivité, il est important de comprendre les caractéristiques uniques de mise à l’échelle de [Fonctions durables](durable-functions-overview.md).

Pour comprendre le comportement lié à la mise à l’échelle, vous devez également comprendre certaines informations du fournisseur de stockage Azure sous-jacent.

## <a name="history-table"></a>Table d’historique

La table d’**historique** est une table de stockage Azure qui contient les événements d’historique de toutes les instances d’orchestration au sein d’un hub de tâches. Le nom de la table apparaît sous la forme *TaskHubName* History. Au fur et à mesure que des instances sont exécutées, de nouvelles lignes sont ajoutées à cette table. La clé de partition de la table provient de l’ID d’instance de l’orchestration. Un identifiant d’instance est aléatoire dans la plupart des cas, ce qui garantit une distribution optimale des partitions internes dans le stockage Azure.

Lorsqu’une instance d’orchestration doit s’exécuter, les lignes appropriées de la table d’historique sont chargées en mémoire. Ces *événements d’historique* sont ensuite relus dans le code de fonction d’orchestrateur pour revenir à l’état contrôlé précédemment. L’utilisation de l’historique d’exécution permettant de régénérer l’état de cette façon est influencée par le [modèle d’approvisionnement en événements](/azure/architecture/patterns/event-sourcing).

## <a name="instances-table"></a>Table d’instances

La table d’**instances** est une autre table de stockage Azure qui contient les états de toutes les instances d’orchestration et d’entité au sein d’un hub de tâches. Au fur et à mesure que des instances sont créées, des lignes sont ajoutées à cette table. La clé de partition de cette table est l’ID d’instance de l’orchestration ou la clé d’entité, et la clé de ligne est une chaîne vide. Il y a une ligne par instance d’orchestration ou d’entité.

Cette table est utilisée pour répondre aux demandes de requête d’instance provenant des API `GetStatusAsync` (.NET) et `getStatus` (JavaScript) et de l’API [HTTP de requête d’état](durable-functions-http-api.md#get-instance-status). Sa cohérence avec le contenu de la table d’**historique** mentionnée précédemment est conservée. L’utilisation d’une table de stockage Azure distincte pour satisfaire les opérations de requête d’instance de cette façon est influencée par le [modèle de séparation des responsabilités en matière de commande et de requête (CQRS)](/azure/architecture/patterns/cqrs).

## <a name="internal-queue-triggers"></a>Déclencheurs de file d’attente interne

Les fonctions d’orchestrateur et les fonctions d’activité sont déclenchées par des files d’attente internes dans le hub de tâches de l’application de fonction. Cette utilisation des files d’attente apporte des garanties de livraison de messages « au moins une fois » fiables. Il existe deux types de file d’attente dans Fonctions durables : la **file d’attente de contrôle** et la **file d’attente des éléments de travail**.

### <a name="the-work-item-queue"></a>File d’attente des éléments de travail

Il existe une file d’attente des éléments de travail par hub de tâches dans Fonctions durables. Il s’agit d’une file d’attente de base qui se comporte comme toute autre file d’attente `queueTrigger` dans Azure Functions. Cette file d’attente permet de déclencher des *fonctions d’activité* sans état en enlevant un message de la file d’attente à la fois. Chacun de ces messages contient des entrées de fonction d’activité et des métadonnées supplémentaires, par exemple la fonction à exécuter. Quand une application Fonctions durables est étendue à plusieurs machines virtuelles, ces dernières rivalisent entre elles pour acquérir le travail de la file d’attente des éléments de travail.

### <a name="control-queues"></a>File(s) d’attente de contrôle

Il existe plusieurs *files d’attente de contrôle* par hub de tâches dans Fonctions durables. Une *file d’attente de contrôle* est plus sophistiquée et complexe que la file d’attente des éléments de travail. Les files d’attente de contrôle servent à déclencher les fonctions d’orchestrateur ou d’entité avec état. Étant donné que les instances de fonction d’orchestrateur ou d’entité sont des singletons avec état, il est impossible d’utiliser un modèle concurrent de contrôle serveur consommateur pour distribuer la charge sur les machines virtuelles. À la place, la charge des messages d’orchestrateur est d’entité est équilibrée sur les files d’attente de contrôle. Vous trouverez plus d’informations sur ce comportement dans les sections suivantes.

Les files d’attente de contrôle contiennent différents types de message couvrant le cycle de vie de l’orchestration, tels que des [messages de contrôle d’orchestrateurs](durable-functions-instance-management.md), des messages de *réponse* de fonctions d’activité et des messages de minuteurs. Au maximum, 32 messages seront enlevés d’une file d’attente de contrôle lors d’une seule interrogation. Ces messages contiennent des données de charge utile ainsi que des métadonnées, et notamment l’instance d’orchestration de destination. Si plusieurs messages enlevés de la file d’attente sont prévus pour la même instance d’orchestration, ils seront traités en tant que lot.

### <a name="queue-polling"></a>Interrogation de file d'attente

L’extension Tâche durable implémente un algorithme d’interruption exponentiel et aléatoire pour réduire l’effet de l’interrogation de file d’attente inactive sur les coûts de transactions de stockage. Quand un message est détecté, le runtime vérifie s’il existe un autre message ; si aucun message n’est détecté, il attend quelques secondes avant de réessayer. Après plusieurs échecs de tentatives d’obtention d’un message de file d’attente, le temps d’attente continue à augmenter jusqu’à ce qu’il atteigne le délai d’attente maximal par défaut (30 secondes).

Le délai maximal d’interrogation est configurable via la propriété `maxQueuePollingInterval` dans le [fichier host.json](../functions-host-json.md#durabletask). Une valeur plus élevée de cette propriété peut entraîner une plus grande latence lors du traitement des messages. Les latences plus élevées ne devraient survenir qu'après des périodes d'inactivité. Une valeur plus faible de cette propriété peut entraîner une augmentation des coûts de stockage en raison d’un nombre plus important de transactions de stockage.

> [!NOTE]
> Lorsqu’il est exécuté dans les plans Consommation et Premium d’Azure Functions, le [contrôleur de mise à l'échelle Azure Functions](../functions-scale.md#how-the-consumption-and-premium-plans-work) interrogera chaque contrôle et chaque file d'attente des éléments de travail toutes les 10 secondes. Cette interrogation supplémentaire est nécessaire pour déterminer quand activer les instances d'application de fonction et pour prendre des décisions de mise à l'échelle. Au moment d’écrire ces lignes, cet intervalle de 10 secondes est constant et ne peut pas être configuré.

### <a name="orchestration-start-delays"></a>Retards de début de l’orchestration
Les instances d’orchestration sont démarrées en plaçant un message `ExecutionStarted` dans l’une des files d’attente de contrôle du hub de tâches. Dans certaines conditions, vous pouvez observer des délais de plusieurs secondes entre le moment où l’exécution d’une orchestration est planifiée et le moment où elle commence à s’exécuter. Pendant ce laps de temps, l’instance d’orchestration reste dans l’état `Pending`. Il existe deux causes possibles pour ce retard :

1. **Files d’attente de contrôle en backlog** : Si la file d’attente de contrôle de cette instance contient un grand nombre de messages, cela peut prendre du temps avant que le message `ExecutionStarted` ne soit reçu et traité par le runtime. Les backlogs de messages peuvent se produire lorsque des orchestrations traitent un grand nombre d’événements simultanément. Les événements qui se trouvent dans la file d’attente de contrôle incluent les événements de début d’orchestration, les saisies semi-automatiques d’activité, les minuteurs durables, l’achèvement et les événements externes. Si ce retard se produit dans des circonstances normales, envisagez de créer un hub de tâches avec un plus grand nombre de partitions. La configuration d’un plus grand nombre de partitions entraîne la création par le runtime d’autres files d’attente de contrôle pour la distribution de la charge.

2. **Retards d’interrogation de secours** : Une autre cause courante des retards d’orchestration est le [comportement d’interrogation de secours précédemment décrit pour les files d’attente de contrôle](#queue-polling). Toutefois, ce retard est attendu uniquement en cas de Scale-out d’une application sur plusieurs instances. S’il n’existe qu’une seule instance d’application ou si l’instance d’application qui démarre l’orchestration est également la même instance qui interroge la file d’attente de contrôle cible, il n’y aura pas de retard d’interrogation de la file d’attente. Les retards d’interrogation de secours peuvent être réduits en mettant à jour les paramètres **host.json**, comme décrit précédemment.

## <a name="storage-account-selection"></a>Sélection du compte de stockage

Les files d’attente, les tables et les objets blob utilisés par Durable Functions sont créés dans un compte de stockage Azure configuré. Le compte à utiliser peut être spécifié à l’aide du paramètre `durableTask/storageProvider/connectionStringName`(ou du paramètre `durableTask/azureStorageConnectionStringName` dans Durable Functions 1.x) dans le fichier **host.json**.

### <a name="durable-functions-2x"></a>Durable Functions 2.x

```json
{
  "extensions": {
    "durableTask": {
      "storageProvider": {
        "connectionStringName": "MyStorageAccountAppSetting"
      }
    }
  }
}
```

### <a name="durable-functions-1x"></a>Durable Functions 1.x

```json
{
  "extensions": {
    "durableTask": {
      "azureStorageConnectionStringName": "MyStorageAccountAppSetting"
    }
  }
}
```

S’il n’est pas spécifié, le compte de stockage `AzureWebJobsStorage` par défaut est utilisé. Pour les charges de travail sensibles aux performances, il est toutefois recommandé de configurer un compte de stockage non défini par défaut. Fonctions durables utilise beaucoup le stockage Azure, et l’utilisation d’un compte de stockage dédié distingue l’usage du stockage Fonctions durables et l’usage interne par l’hôte Azure Functions.

## <a name="orchestrator-scale-out"></a>Mise à l’échelle de l’orchestrateur

Les fonctions d’activité sont sans état et automatiquement mises à l’échelle par l’ajout de machines virtuelles. Les fonctions d’orchestrateur et les entités sont, quant à elles, *partitionnées* sur une ou plusieurs files d’attente de contrôle. Le nombre de files d’attente de contrôle est défini dans le fichier **host.json**. L’exemple suivant d’extrait de code host.json définit la propriété `durableTask/storageProvider/partitionCount` (ou `durableTask/partitionCount`dans Durable Functions 1.x) sur `3`.

### <a name="durable-functions-2x"></a>Durable Functions 2.x

```json
{
  "extensions": {
    "durableTask": {
      "storageProvider": {
          "partitionCount": 3
      }
    }
  }
}
```

### <a name="durable-functions-1x"></a>Durable Functions 1.x

```json
{
  "extensions": {
    "durableTask": {
      "partitionCount": 3
    }
  }
}
```

Un hub de tâches peut être configuré avec 1 à 16 partitions. Si ce paramètre n’est pas spécifié, le nombre de partitions par défaut s’élève à **4**.

Durant la mise à l’échelle sur plusieurs instances de l’hôte de fonction (généralement sur différentes machines virtuelles), chaque instance acquiert un verrou sur l’une des files d’attente de contrôle. Ces verrous sont implémentés en interne comme des baux de stockage blob et ils garantissent qu’une instance d’orchestration ou une entité s’exécute uniquement sur une seule instance d’hôte à la fois. Si un hub de tâches est configuré avec trois files d’attente de contrôle, la charge des instances d’orchestration et des entités peut être équilibrée sur trois machines virtuelles. Il est possible d’ajouter des machines virtuelles supplémentaires pour augmenter la capacité d’exécution de la fonction d’activité.

Le diagramme suivant illustre la façon dont l’hôte Azure Functions interagit avec les entités de stockage dans un environnement mis à l’échelle.

![Diagramme de mise à l’échelle](./media/durable-functions-perf-and-scale/scale-diagram.png)

Comme indiqué dans le diagramme précédent, toutes les machines virtuelles sont en concurrence pour les messages de la file d’attente des éléments de travail. Toutefois, seules trois machines virtuelles peuvent acquérir les messages des files d’attente de contrôle, et chacune d’elles verrouille une file d’attente de contrôle.

Les instances d’orchestration et les entités sont réparties sur toutes les instances de file d’attente de contrôle. La répartition s’effectue par hachage de l’ID d’instance de l’orchestration ou de la paire nom/clé de l’entité. Les ID d’instance d’orchestration par défaut sont des GUID aléatoires, ce qui garantit la bonne répartition des instances sur toutes les files d’attente de contrôle.

En règle générale, les fonctions d’orchestrateur sont conçues pour être légères et elles ne requièrent pas une grande puissance de calcul. Il n’est donc pas nécessaire de créer un grand nombre de partitions de file d’attente de contrôle pour obtenir un haut débit pour les orchestrations. Le travail lourd doit être principalement effectué dans les fonctions d’activité sans état, qui peuvent être mises à l’échelle à l’infini.

## <a name="auto-scale"></a>Mise à l’échelle automatique

Comme avec toutes les Azure Functions exécutées dans les plans Consommation et Élastique Premium, Durable Functions prend en charge la mise à l’échelle automatique par le biais du [contrôleur de mise à l’échelle Azure Functions](../functions-scale.md#runtime-scaling). Le contrôleur de mise à l’échelle surveille la latence de toutes les files d’attente en émettant régulièrement des commandes _peek_. Selon la latence des messages parcourus, le contrôleur de mise à l’échelle décide d’ajouter ou de supprimer des machines virtuelles.

Si le contrôleur de mise à l’échelle détermine que la latence des messages de file d’attente de contrôle est trop élevée, il ajoute des instances de machine virtuelle jusqu’à ce que la latence des messages diminue à un niveau acceptable ou qu’il atteigne le nombre de partitions de file d’attente de contrôle. De même, le contrôleur de mise à l’échelle ajoute continuellement des instances de machine virtuelle si le temps de latence de file d’attente des éléments de travail est élevé, quel que soit le nombre de partitions.

> [!NOTE]
> À partir de Durable Functions 2.0, les applications de fonction peuvent être configurées de sorte qu’elles s’exécutent au sein des points de terminaison de service protégés par VNET dans le plan Élastique Premium. Dans cette configuration, Durable Functions déclenche des demandes de mise à l’échelle initiées au lieu du contrôleur de mise à l’échelle.

## <a name="thread-usage"></a>Utilisation de threads

Les fonctions d’orchestrateur sont exécutées sur un seul thread pour garantir la possibilité d’une exécution déterministe entre les nombreuses relectures. Compte tenu de cette exécution monothread, il est important que les threads de fonction d’orchestrateur n’effectuent pas de tâches nécessitant une utilisation intensive du processeur, effectuent des opérations d’E/S ou se bloquent pour une raison quelconque. Tout travail susceptible de nécessiter un thread d’E/S, un blocage ou plusieurs threads doit être transféré vers les fonctions d’activité.

Les fonctions d’activité ont les mêmes comportements que les fonctions normales déclenchées par une file d’attente. Elles peuvent effectuer en toute sécurité des opérations d’E/S ou des opérations gourmandes en ressources de processeur et utiliser plusieurs threads. Étant donné que les déclencheurs d’activité sont sans état, ils peuvent librement effectuer un scale-out sur un nombre illimité de machines virtuelles.

Les fonctions d’entité sont également exécutées sur un thread unique et les opérations sont traitées une par une. Toutefois, les fonctions d’entité n’ont aucune restriction quant au type de code qui peut être exécuté.

## <a name="concurrency-throttles"></a>Limitations de la concurrence

Azure Functions prend en charge l’exécution simultanée de plusieurs fonctions dans une seule instance d’application. Cette exécution simultanée contribue à augmenter le parallélisme et réduit le nombre de démarrages à froid auxquels une application classique est confrontée au fil du temps. Toutefois, une concurrence élevée peut épuiser les ressources système de chaque machine virtuelle, telles que les connexions réseau ou la mémoire disponible. En fonction des besoins de l’application de fonction, il peut être nécessaire de limiter la concurrence par instance afin d’éviter le risque de manquer de mémoire dans les situations de charge importante.

Les limites de concurrence des fonctions d’activité, d’orchestrateur et d’entité peuvent être configurées dans le fichier **host.json**. Les paramètres pertinents sont `durableTask/maxConcurrentActivityFunctions` pour les fonctions d’activité et `durableTask/maxConcurrentOrchestratorFunctions` pour les fonctions d’orchestrateur et d’entité.

### <a name="functions-20"></a>Functions 2.0

```json
{
  "extensions": {
    "durableTask": {
      "maxConcurrentActivityFunctions": 10,
      "maxConcurrentOrchestratorFunctions": 10
    }
  }
}
```

### <a name="functions-1x"></a>Functions 1.x

```json
{
  "durableTask": {
    "maxConcurrentActivityFunctions": 10,
    "maxConcurrentOrchestratorFunctions": 10
  }
}
```

Dans l’exemple précédent, un maximum de 10 fonctions d’orchestrateur ou d’entité et 10 fonctions d’activité peut s’exécuter simultanément sur une seule machine virtuelle. S’il n’est pas spécifié, le nombre d’exécutions simultanées de fonction d’activité et de fonction d’orchestrateur ou d’entité est limité à 10 fois le nombre de cœurs sur la machine virtuelle.

> [!NOTE]
> Ces paramètres permettent de gérer l’utilisation de la mémoire et du processeur sur une seule machine virtuelle. Toutefois, lors de la montée en chaque sur plusieurs machines virtuelles, chaque machine virtuelle a son propre ensemble de limites défini. Ces paramètres ne peuvent pas être utilisés pour contrôler la concurrence au niveau global.

## <a name="extended-sessions"></a>Sessions étendues

Les sessions étendues sont un paramètre qui conserve les orchestrations et les entités en mémoire même après qu’elles aient terminé le traitement des messages. Généralement, l’activation des sessions étendues entraîne la baisse des E/S sur le compte de stockage Azure et un meilleur débit global.

Vous pouvez activer les sessions étendues en paramétrant `durableTask/extendedSessionsEnabled` sur `true` dans le fichier **host.json**. Le paramètre `durableTask/extendedSessionIdleTimeoutInSeconds` peut être utilisé pour contrôler la durée pendant laquelle une session inactive sera conservée en mémoire :

**Functions 2.0**
```json
{
  "extensions": {
    "durableTask": {
      "extendedSessionsEnabled": true,
      "extendedSessionIdleTimeoutInSeconds": 30
    }
  }
}
```

**Functions 1.0**
```json
{
  "durableTask": {
    "extendedSessionsEnabled": true,
    "extendedSessionIdleTimeoutInSeconds": 30
  }
}
```

Ce paramètre présente deux inconvénients potentiels à connaître :

1. L’utilisation de la mémoire des applications de fonction augmente de façon générale.
2. Le débit peut diminuer globalement s’il existe plusieurs exécutions simultanées et de courte durée de fonctions d’orchestrateur et d’entité.

Par exemple, si `durableTask/extendedSessionIdleTimeoutInSeconds` est défini sur 30 secondes, un épisode de fonction d’orchestrateur ou d’entité de courte durée qui s’exécute en moins de 1 seconde occupe tout de même la mémoire pendant 30 secondes. Il est également comptabilisé dans le quota `durableTask/maxConcurrentOrchestratorFunctions` mentionné précédemment et empêche potentiellement l’exécution d’autres fonctions d’orchestrateur ou d’entité.

Les effets spécifiques des sessions étendues sur les fonctions d’orchestrateur et d’entité sont décrits dans les sections suivantes.

> [!NOTE]
> Les sessions étendues ne sont actuellement prises en charge que dans les langages .NET, comme C# ou F#. La définition de `extendedSessionsEnabled` sur `true` pour d’autres plateformes peut entraîner des problèmes d’exécution, tels que l’exécution en mode silencieux de l’activité et des fonctions déclenchées par orchestration.


### <a name="orchestrator-function-replay"></a>Relecture de la fonction d’orchestrateur

Comme mentionné précédemment, les fonctions d’orchestrateur sont relues à l’aide du contenu de la table d’**historique**. Par défaut, le code de fonction d’orchestrateur est relu chaque fois qu’un lot de messages est enlevé d’une file d’attente de contrôle. Même si vous utilisez le modèle fan-out/fan-in et vous attendez à ce que toutes les tâches soient terminées (par exemple, en utilisant `Task.WhenAll` en .NET ou `context.df.Task.all` en JavaScript), des relectures se produisent à mesure que des lots de réponses de tâche sont traités. Lorsque des sessions étendues sont activées, les instances de fonction d’orchestrateur sont conservées en mémoire plus longtemps et les nouveaux messages peuvent être traités sans une relecture complète de l’historique.

L’amélioration des performances des sessions étendues est le plus souvent observée dans les situations suivantes :

* Lorsqu’il existe un nombre limité d’instances d’orchestration s’exécutant simultanément.
* Lorsque des orchestrations ont un grand nombre d’actions séquentielles (par exemple, des centaines d’appels de fonction d’activité) qui se terminent rapidement.
* Lorsque des orchestrations exécutent une distribution ramifiée (fan-out) et un traitement de réponse parallèle (fan-in) sur un grand nombre d’actions qui se terminent à peu près au même moment.
* Lorsque des fonctions d’orchestrateur doivent traiter des messages volumineux ou effectuer un traitement de données gourmand en ressources UC.

Dans toutes les autres situations, il n’y a généralement pas d’amélioration notable des performances pour les fonctions d’orchestrateur.

> [!NOTE]
> Ces paramètres doivent être utilisés uniquement après le développement et le test complets d’une fonction d’orchestrateur. Le comportement de réexécution agressif par défaut peut être utile pour détecter les violations des [contraintes du code de la fonction d’orchestrateur](durable-functions-code-constraints.md) lors du développement, et est donc désactivé par défaut.

### <a name="entity-function-unloading"></a>Déchargement des fonctions d’entité

Les fonctions d’entité traitent jusqu’à 20 opérations dans un lot unique. Dès qu’une entité termine le traitement d’un lot d’opérations, elle conserve son état et se décharge de la mémoire. Vous pouvez retarder le déchargement d’entités de la mémoire à l’aide du paramètre des sessions étendues. Les entités continuent à conserver leurs modifications d’état, comme auparavant, mais elles restent en mémoire pendant la durée configurée pour réduire le nombre de charges à partir de Stockage Azure. Cette réduction des charges à partir de Stockage Azure peut améliorer le débit global des entités fréquemment sollicitées.

## <a name="performance-targets"></a>Cibles de performance

Lors de la planification de l’utilisation de Fonctions durables pour une application de production, il est important de tenir compte très rapidement des exigences de performances dans le processus de planification. Cette section décrit certains scénarios d’utilisation de base et les débits maximaux attendus.

* **Exécution d’activité séquentielle** : ce scénario décrit une fonction d’orchestrateur qui exécute une série de fonctions d’activité, l’une après l’autre. Il ressemble beaucoup à l’exemple de [chaînage de fonctions](durable-functions-sequence.md).
* **Exécution d’activité parallèle** : ce scénario décrit une fonction d’orchestrateur qui exécute de nombreuses fonctions d’activité en parallèle à l’aide du modèle [fan-out/fan-in](durable-functions-cloud-backup.md).
* **Traitement de la réponse en parallèle** : ce scénario constitue la seconde moitié du modèle [fan-out/fan-in](durable-functions-cloud-backup.md). Il se concentre sur les performances du scénario fan-in. Contrairement au scénario fan-out, le scénario fan-in est réalisé par une instance de fonction d’orchestrateur unique et peut donc uniquement s’exécuter sur une seule machine virtuelle.
* **Traitement des événements externes** : ce scénario représente une instance unique de la fonction d’orchestrateur, qui attend les [événements externes](durable-functions-external-events.md), un par un.
* **Traitement des opérations d’entité** : ce scénario teste la rapidité avec laquelle une _seule_ [entité Compteur](durable-functions-entities.md) peut traiter un flux constant d’opérations.

> [!TIP]
> Contrairement aux opérations fan-out, les opérations fan-in sont limitées à une seule machine virtuelle. Si votre application utilise le modèle fan-out/fan-in et si vous vous préoccupez des performances de scénario fan-in, pensez à sous-diviser le scénario fan-out de fonction d’activité sur plusieurs [sous-orchestrations](durable-functions-sub-orchestrations.md).

Le tableau suivant présente les débits *maximaux* attendus pour les scénarios décrits précédemment. Le terme « instance » fait référence à l’instance unique d’une fonction d’orchestrateur en cours d’exécution sur une petite machine virtuelle ([A1](../../virtual-machines/sizes-previous-gen.md)) unique dans Azure App Service. Dans tous les cas, il est supposé que les [sessions étendues](#orchestrator-function-replay) sont activées. Les résultats réels peuvent varier selon le travail de l’UC ou des E/S effectué par le code de fonction.

| Scénario | Débit maximal |
|-|-|
| Exécution d’activité séquentielle | 5 activités par seconde et par instance |
| Exécution d’activité parallèle (fan-out) | 100 activités par seconde et par instance |
| Traitement de réponse parallèle (fan-in) | 150 réponses par seconde et par instance |
| Traitement d’événements externes | 50 événements par seconde et par instance |
| Traitement des opérations d’entité | 64 opérations par seconde |

> [!NOTE]
> Ces chiffres concernent la version 1.4.0 (GA) de l’extension Fonctions durables. Ces données peuvent changer au fil du temps, avec l’évolution de la fonctionnalité et les diverses optimisations.

Si vous n’obtenez pas les débits que vous attendiez et si l’utilisation de l’UC et de la mémoire semble correcte, vérifiez [l’intégrité de votre compte de stockage](../../storage/common/storage-monitoring-diagnosing-troubleshooting.md#troubleshooting-guidance). L’extension Fonctions durables peut placer une charge importante sur un compte de stockage Azure, et des charges suffisamment élevées peuvent entraîner la limitation du compte de stockage.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur la récupération d’urgence et la géo-distribution](durable-functions-disaster-recovery-geo-distribution.md)

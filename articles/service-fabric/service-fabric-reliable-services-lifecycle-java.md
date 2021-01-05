---
title: Cycle de vie des Reliable Services dans Azure Service Fabric
description: En savoir plus sur les événements de cycle de vie dans une application Azure Service Fabric Reliable Services à l’aide de Java pour les services avec et sans État.
author: PavanKunapareddyMSFT
ms.topic: conceptual
ms.date: 06/30/2017
ms.custom: devx-track-java
ms.author: pakunapa
ms.openlocfilehash: b22c78a0259e4430ac6bfae1c0a9379c4a832cd4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87324605"
---
# <a name="reliable-services-lifecycle"></a>Cycle de vie des Reliable Services
> [!div class="op_single_selector"]
> * [C# sur Windows](service-fabric-reliable-services-lifecycle.md)
> * [Java sur Linux](service-fabric-reliable-services-lifecycle-java.md)
>
>

Les Reliable Services sont l’un des modèles de programmation disponibles dans Azure Service Fabric. Ce qu’il est essentiel de comprendre sur le cycle de vie des Reliable Services, ce sont les événements de base qui y sont liés. L’ordre exact des événements dépend des détails de la configuration. 

En règle générale, le cycle de vie des Reliable Services inclut les événements suivants :

* Lors du démarrage :
  * Les services sont construits.
  * Ils ont l’opportunité de se construire et de renvoyer zéro ou plusieurs écouteurs.
  * Les écouteurs retournés sont ouverts, pour la communication avec le service.
  * La méthode `runAsync` du service est appelée, ce qui permet au service d’exécuter des tâches de longue durée ou d’arrière-plan.
* Lors de l’arrêt :
  * Le jeton d’annulation passé à `runAsync` est annulé et les écouteurs sont fermés.
  * L’objet de service lui-même est détruit.

L’ordre des événements des Reliable Services peut varier légèrement selon qu’il s’agit d’un service sans état ou avec état. 

En outre, pour les services avec état, vous devez gérer le scénario d’échange principal. Au cours de cette séquence, le rôle principal est transféré vers un autre réplica (ou revient) sans que le service ne s’arrête. 

Enfin, vous devez penser aux conditions d’erreur ou d’échec.

## <a name="stateless-service-startup"></a>Démarrage de service sans état
Le cycle de vie d’un service sans état est assez simple. Voici l’ordre des événements :

1. Le service est construit.
2. Ces événements se produisent en parallèle :
    - `StatelessService.createServiceInstanceListeners()` est appelé et les écouteurs retournés sont ouverts. `CommunicationListener.openAsync()` est appelé sur chaque écouteur.
    - La méthode `runAsync` du service (`StatelessService.runAsync()`) est appelée.
3. Si elle est présente, la propre méthode `onOpenAsync` du service est appelée. Plus précisément, `StatelessService.onOpenAsync()` est appelée. Il s’agit d’un remplacement rare, mais possible.

Il est important de noter qu’il n’existe pas d’ordre particulier entre l’appel pour créer et ouvrir les écouteurs, et l’appel de `runAsync`. Les écouteurs peuvent être ouverts avant le démarrage de `runAsync`. De même, `runAsync` peut être appelée avant que les écouteurs de communication ne soient ouverts ou même construits. Si une synchronisation est nécessaire, elle doit être effectuée par le responsable d’implémentation. Voici quelques solutions courantes :

* Parfois, les écouteurs ne peuvent pas fonctionner avant que d’autres informations ne soient créées ou qu’un travail ne soit effectué. Pour les services sans état, ce travail peut généralement être effectué dans le constructeur du service. Il peut être effectué pendant l’appel de `createServiceInstanceListeners()`, ou dans le cadre de la construction de l’écouteur lui-même.
* Parfois, le code de `runAsync` ne démarre pas tant que les écouteurs ne sont pas ouverts. Dans ce cas, une coordination supplémentaire est nécessaire. Une solution courante consiste à ajouter un indicateur dans les écouteurs. L’indicateur indique quand les écouteurs ont terminé. La méthode `runAsync` vérifie cet élément avant de poursuivre le travail réel.

## <a name="stateless-service-shutdown"></a>Arrêt de service sans état
Lorsque vous arrêtez un service sans état, le même modèle est suivi, mais dans l’ordre inverse :

1. Ces événements se produisent en parallèle :
    - Les écouteurs ouverts sont fermés. `CommunicationListener.closeAsync()` est appelé sur chaque écouteur.
    - Le jeton d’annulation passé à `runAsync()` est annulé. Vérification que la propriété `isCancelled` du jeton d’annulation retourne la valeur `true` et que, si elle est appelée, la méthode `throwIfCancellationRequested` du jeton lève une `CancellationException`.
2. Quand `closeAsync()` se termine sur chaque écouteur et que `runAsync()` se termine également, la méthode `StatelessService.onCloseAsync()` du service est appelée, le cas échéant. Une fois encore, ce remplacement n’est pas ordinaire, mais il peut être utilisé pour fermer des ressources, arrêter un traitement en arrière-plan, finaliser l’enregistrement de l’état externe, ou fermer des connexions existantes en toute sécurité.
3. Quand `StatelessService.onCloseAsync()` se termine, l’objet de service est détruit.

## <a name="stateful-service-startup"></a>Démarrage de service avec état
Les services avec état ont un modèle semblable aux services sans état, avec quelques modifications.  Voici l’ordre des événements de démarrage d’un service avec état :

1. Le service est construit.
2. `StatefulServiceBase.onOpenAsync()` est appelée. Il est rare de remplacer cet appel dans le service.
3. Ces événements se produisent en parallèle :
    - `StatefulServiceBase.createServiceReplicaListeners()` est appelé. 
      - Si le service est un service principal, tous les écouteurs retournés sont ouverts. `CommunicationListener.openAsync()` est appelé sur chaque écouteur.
      - Si le service est un service secondaire, seuls les écouteurs marqués comme `listenOnSecondary = true` sont ouverts. Il est plus rare d’avoir des écouteurs ouverts dans les services secondaires.
    - Si le service est un service principal, la méthode `StatefulServiceBase.runAsync()` du service est appelée.
4. Une fois que tous les appels `openAsync()` de l’écouteur de réplica sont terminés et que `runAsync()` est appelé, `StatefulServiceBase.onChangeRoleAsync()` est appelé. Il est rare de remplacer cet appel dans le service.

Comme pour les services sans état, il n’y a aucune coordination entre l’ordre dans lequel les écouteurs sont créés et ouverts, et le moment auquel `runAsync` est appelée. Si vous avez besoin d’une coordination, les solutions à utiliser sont similaires. Mais il existe un autre cas pour un service avec état. Supposons que les appels qui arrivent sur les écouteurs de communication nécessitent des informations conservées dans des [collections fiables](service-fabric-reliable-services-reliable-collections.md). Étant donné que les écouteurs de communication peuvent s’ouvrir avant que les Reliable Collections ne soient accessibles en lecture ou en écriture, et avant que `runAsync` ne démarre, une coordination supplémentaire est nécessaire. La solution la plus simple et la plus courante est que les écouteurs de communication retournent un code d’erreur. Le client utilise ce code d’erreur pour relancer la requête.

## <a name="stateful-service-shutdown"></a>Arrêt de service avec état
Comme pour les services sans état, les événements de cycle de vie lors de l’arrêt sont les mêmes que lors du démarrage, mais dans l’ordre inverse. Lorsqu’un service avec état est arrêté, les événements suivants se produisent :

1. Ces événements se produisent en parallèle :
    - Les écouteurs ouverts sont fermés. `CommunicationListener.closeAsync()` est appelé sur chaque écouteur.
    - Le jeton d’annulation passé à `runAsync()` est annulé. Un appel à la méthode `isCancelled()` du jeton d’annulation retourne la valeur `true` et, si elle est appelée, la méthode `throwIfCancellationRequested()` du jeton lève une `OperationCanceledException`.
2. Quand `closeAsync()` se termine sur chaque écouteur et que `runAsync()` se termine également, la méthode `StatefulServiceBase.onChangeRoleAsync()` du service est appelée. Il est rare de remplacer cet appel dans le service.

   > [!NOTE]  
   > Vous devez attendre que `runAsync` se termine uniquement si ce réplica est un réplica principal.

3. Une fois la méthode `StatefulServiceBase.onChangeRoleAsync()` terminée, la méthode `StatefulServiceBase.onCloseAsync()` est appelée. Il s’agit d’un remplacement rare, mais possible.
3. Quand `StatefulServiceBase.onCloseAsync()` se termine, l’objet de service est détruit.

## <a name="stateful-service-primary-swaps"></a>Échanges de réplica principal de service avec état
Pendant l’exécution d’un service avec état, les écouteurs de communication sont ouverts et la méthode `runAsync` est appelée seulement pour les réplicas principaux de ces services avec état. Les réplicas secondaires sont construits, mais ne reçoivent aucun autre appel. Pendant l’exécution d’un service avec état, le réplica principal peut changer. Les événements de cycle de vie que voit le réplica avec état varie selon qu’il s’agit du réplica qui est rétrogradé ou promu au cours de l’échange.

### <a name="for-the-demoted-primary"></a>Pour le réplica principal rétrogradé
Service Fabric nécessite que ce réplica arrête de traiter des messages et quitte tout travail en arrière-plan. Cette étape est similaire à l’arrêt du service. La différence est que le service n’est pas détruit ni fermé, car il est conservé en tant que réplica secondaire. Les événements suivants se produisent :

1. Ces événements se produisent en parallèle :
    - Les écouteurs ouverts sont fermés. `CommunicationListener.closeAsync()` est appelé sur chaque écouteur.
    - Le jeton d’annulation passé à `runAsync()` est annulé. La vérification de la méthode `isCancelled()` du jeton d’annulation renvoie la valeur `true`. Si elle est appelée, la méthode `throwIfCancellationRequested()` du jeton lève une `OperationCanceledException`.
2. Quand `closeAsync()` se termine sur chaque écouteur et que `runAsync()` se termine également, la méthode `StatefulServiceBase.onChangeRoleAsync()` du service est appelée. Il est rare de remplacer cet appel dans le service.

### <a name="for-the-promoted-secondary"></a>Pour le réplica secondaire promu
De même, Service Fabric nécessite que le réplica secondaire promu commence à écouter les messages sur le réseau et qu’il démarre les tâches en arrière-plan qui le concernent. Ce processus est similaire à la création du service. La différence est que le réplica lui-même existe déjà. Les événements suivants se produisent :

1. Ces événements se produisent en parallèle :
    - `StatefulServiceBase.createServiceReplicaListeners()` est appelé et les écouteurs retournés sont ouverts. `CommunicationListener.openAsync()` est appelé sur chaque écouteur.
    - La méthode `StatefulServiceBase.runAsync()` du service est appelée.
2. Une fois que tous les appels `openAsync()` de l’écouteur de réplica sont terminés et que `runAsync()` est appelé, `StatefulServiceBase.onChangeRoleAsync()` est appelé. Il est rare de remplacer cet appel dans le service.

### <a name="common-issues-during-stateful-service-shutdown-and-primary-demotion"></a>Problèmes courants se produisant pendant l’arrêt d’un service avec état et la rétrogradation du réplica principal
Service Fabric peut modifier le réplica principal d’un service avec état pour plusieurs raisons. Parmi les raisons les plus courantes figurent le [rééquilibrage des clusters](service-fabric-cluster-resource-manager-balancing.md) et la [mise à niveau des applications](service-fabric-application-upgrade.md). Pendant ces opérations, il est important que le service respecte le `cancellationToken`. Cela s’applique également lors de l’arrêt normal du service, par exemple si le service a été supprimé.

Les services qui ne gèrent pas l’annulation « proprement » peuvent être confrontés à plusieurs problèmes. Ces opérations sont lentes, car Service Fabric attend que les services s’arrêtent normalement. Cela peut entraîner l’échec des mises à niveau qui expirent et sont annulées. Si vous ne respectez pas le jeton d’annulation, vous pouvez également provoquer des clusters déséquilibrés. Les clusters sont déséquilibrés, car les nœuds deviennent actifs. Mais les services ne peuvent pas être rééquilibrés, car leur déplacement prend trop de temps. 

Comme les services sont avec état, il est également probable qu’ils utilisent des [Reliable Collections](service-fabric-reliable-services-reliable-collections.md). Dans Service Fabric, lorsqu’un réplica principal est rétrogradé, l’une des premières choses qui se produisent est que l’accès en écriture à l’état sous-jacent est révoqué. Cela conduit à un deuxième type de problème qui peut avoir un impact sur le cycle de vie du service. Les collections retournent des exceptions selon le délai et selon que le réplica est déplacé ou arrêté. Il est important de gérer ces exceptions correctement. 

Les exceptions levées par Service Fabric sont soit permanentes [(`FabricException`)](/java/api/system.fabric.exception), soit passagères [(`FabricTransientException`)](/java/api/system.fabric.exception.fabrictransientexception). Les exceptions permanentes doivent être journalisées et levées. Les exceptions passagères peuvent être retentées selon une logique de nouvelle tentative.

Le fait de gérer des exceptions issues de l’utilisation de `ReliableCollections` conjointement aux événements du cycle de vie du service constitue une partie importante du test et de la validation des Reliable Services. Il est recommandé de toujours exécuter votre service sous une charge. Vous devez également effectuer les mises à niveau et un [test CHAOS](service-fabric-controlled-chaos.md) avant le déploiement en production. Ces étapes de base vous permettent de vérifier que votre service est correctement implémenté et qu’il gère correctement les événements de cycle de vie.

## <a name="notes-on-service-lifecycle"></a>Remarques sur le cycle de vie du service
* La méthode `runAsync()` et l’appel `createServiceInstanceListeners/createServiceReplicaListeners` sont facultatifs. Un service peut utiliser l’un ou l’autre, ou aucun des deux. Par exemple, si le service effectue tout son travail en réponse aux appels d’utilisateur, il est inutile d’implémenter `runAsync()`. Seuls les écouteurs de communication et le code associé sont nécessaires.  De même, la création et le renvoi d’écouteurs de communication sont facultatifs. Le service peut n’avoir que du travail en arrière-plan à exécuter et n’a donc besoin d’implémenter que `runAsync()`.
* Il est valide pour un service de terminer `runAsync()` correctement et d’en revenir. Cela n’est pas considéré comme une condition d’échec. Cela représente le travail en arrière-plan de la fin du service. Pour les Reliable Services avec état, `runAsync()` est appelée à nouveau si le service est rétrogradé depuis la fonction principale, puis promu de nouveau à la fonction principale.
* Si un service quitte `runAsync()` en levant une exception inattendue, il s’agit d’un échec. L’objet de service est arrêté et une erreur d’intégrité est signalée.
* S’il n’existe pas de limite de temps sur le retour de ces méthodes, vous perdez immédiatement la possibilité d’écrire. Par conséquent, vous ne pouvez pas effectuer de travail réel. Il est recommandé de procéder au renvoi aussi rapidement que possible dès la réception de la demande d’annulation. Si votre service ne répond pas à ces appels d’API dans un délai raisonnable, Service Fabric peut mettre fin à votre service. En général, cela se produit uniquement lors de mises à niveau d’application ou lorsqu’un service est en cours de suppression. Par défaut, ce délai d’attente est de 15 minutes.
* Les échecs dans le chemin `onCloseAsync()` entraînent l’appel de `onAbort()`. Cet appel constitue une opportunité de dernière chance pour le service de nettoyer et de libérer les ressources demandées. Cette méthode est généralement appelée lorsqu'une erreur permanente est détectée sur le nœud, ou lorsque Service Fabric ne peut pas gérer de façon fiable le cycle de vie de l'instance du service en raison de défaillances internes.
* La méthode `OnChangeRoleAsync()` est appelée quand le réplica de service avec état change de rôle, par exemple, quand il devient un service principal ou secondaire. Les réplicas principaux se voient affecter un état d’écriture (ils sont autorisés à créer des collections fiables et à écrire dans celles-ci). Les réplicas secondaires se voient affecter un état de lecture (ils peuvent uniquement lire à partir de collections fiables existantes). La plupart du travail dans un service avec état est effectué sur le réplica principal. Les réplicas secondaires peuvent effectuer la validation en lecture seule, la génération de rapports, l’exploration de données ou d’autres tâches en lecture seule.

## <a name="next-steps"></a>Étapes suivantes
* [Présentation de Reliable Services](service-fabric-reliable-services-introduction.md)
* [Démarrage rapide de Reliable Services](service-fabric-reliable-services-quick-start-java.md)

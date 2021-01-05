---
title: Vue d’ensemble du cycle de vie de Reliable Services
description: Découvrez les événements de cycle de vie dans une application Azure Service Fabric Reliable Services pour les services avec et sans état.
author: masnider
ms.topic: conceptual
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: 162ad87f79109cf38d3d0013608812155c6988a7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86252247"
---
# <a name="reliable-services-lifecycle-overview"></a>Vue d’ensemble du cycle de vie de Reliable Services
> [!div class="op_single_selector"]
> * [C# sur Windows](service-fabric-reliable-services-lifecycle.md)
> * [Java sur Linux](service-fabric-reliable-services-lifecycle-java.md)
>
>

Quand vous pensez aux cycles de vie de Reliable Services dans Azure Service Fabric, les principes de base du cycle de vie sont les plus importants. En règle générale, le cycle de vie inclut les éléments suivants :

- Lors du démarrage :
  - Les services sont construits.
  - Ils ont l’opportunité de se construire et de renvoyer zéro ou plusieurs écouteurs.
  - Les écouteurs retournés sont ouverts, ce qui permet la communication avec le service.
  - La méthode **RunAsync** du service est appelée, ce qui permet au service d’exécuter des tâches de longue durée ou d’arrière-plan.
- Lors de l’arrêt :
  - Le jeton d’annulation passé à **RunAsync** est annulé et les écouteurs sont fermés.
  - Une fois que les écouteurs sont fermés, l’objet de service lui-même est détruit.

Il existe plus d’informations sur l’ordre exact de ces événements. L’ordre des événements peut varier légèrement selon que le service fiable est un service sans état ou avec état. En outre, pour les services avec état, nous devons gérer le scénario d’échange principal. Au cours de cette séquence, le rôle principal est transféré vers un autre réplica (ou revient) sans que le service ne s’arrête. Enfin, nous devons penser aux conditions d’erreur ou d’échec.

## <a name="stateless-service-startup"></a>Démarrage de service sans état
Le cycle de vie d’un service sans état est simple. Voici l’ordre des événements :

1. Le service est construit.
2. Ensuite, deux événements se produisent en parallèle :
    - `StatelessService.CreateServiceInstanceListeners()` est appelé et les écouteurs retournés sont ouverts. `ICommunicationListener.OpenAsync()` est appelé sur chaque écouteur.
    - La méthode `StatelessService.RunAsync()` du service est appelée.
3. Si elle est présente, la méthode `StatelessService.OnOpenAsync()` du service est appelée. Il s’agit d’un remplacement rare, mais possible. Les tâches d'initialisation de service étendues peuvent être démarrées à ce stade.

Gardez à l’esprit qu’il n’existe pas d’ordre particulier entre les appels pour créer et ouvrir les écouteurs, et **RunAsync**. Les écouteurs peuvent s’ouvrir avant que la méthode **RunAsync** ne soit démarrée. De même, vous pouvez appeler **RunAsync** avant que les écouteurs de communication ne soient ouverts ou même construits. Si une synchronisation est nécessaire, elle est considérée comme un exercice à exécuter par le responsable d’implémentation. Voici quelques solutions courantes :

  - Parfois, les écouteurs ne peuvent pas fonctionner avant que d’autres informations ne soient créées ou qu’un travail ne soit effectué. Pour les services sans état, ce travail peut généralement être effectué à d’autres emplacements, tels que les suivants : 
    - Dans le constructeur du service.
    - Pendant l’appel de `CreateServiceInstanceListeners()`.
    - Dans le cadre de la construction de l’écouteur.
  - Parfois, le code de **RunAsync** ne démarre pas tant que les écouteurs ne sont pas ouverts. Dans ce cas, une coordination supplémentaire est nécessaire. Une solution courante consiste à utiliser un indicateur dans les écouteurs, indiquant quand ceux-ci ont terminé. Cet indicateur est ensuite vérifié dans **RunAsync** avant de poursuivre le travail réel.

## <a name="stateless-service-shutdown"></a>Arrêt de service sans état
Pour arrêter un service sans état, le même modèle est suivi dans l’ordre inverse :

1. En parallèle :
    - Les écouteurs ouverts sont fermés. `ICommunicationListener.CloseAsync()` est appelé sur chaque écouteur.
    - Le jeton d’annulation passé à `RunAsync()` est annulé. Une vérification que la propriété `IsCancellationRequested` du jeton d’annulation retourne la valeur true et que, si elle est appelée, la méthode `ThrowIfCancellationRequested` du jeton lève une `OperationCanceledException`.
2. Quand `CloseAsync()` se termine sur chaque écouteur et que `RunAsync()` se termine également, la méthode `StatelessService.OnCloseAsync()` du service est appelée, le cas échéant.  La méthode OnCloseAsync est appelée quand l’instance de service sans état est sur le point d’être correctement arrêtée. Cela peut se produire lorsque le code du service est mis à niveau, lorsque l'instance du service est déplacée en raison d'un équilibrage de charge, ou lorsqu'une erreur transitoire est détectée. Il n’est pas fréquent de remplacer la méthode `StatelessService.OnCloseAsync()`, mais vous pouvez l’utiliser pour fermer les ressources, arrêter le traitement en arrière-plan, finaliser l’enregistrement de l’état externe, ou fermer des connexions existantes en toute sécurité.
3. Quand `StatelessService.OnCloseAsync()` se termine, l’objet de service est détruit.

## <a name="stateful-service-startup"></a>Démarrage de service avec état
Les services avec état ont un modèle semblable aux services sans état, avec quelques modifications. Pour démarrer un service avec état, l’ordre des événements est le suivant :

1. Le service est construit.
2. `StatefulServiceBase.OnOpenAsync()` est appelée. Il est rare de remplacer cet appel dans le service.
3. Les événements suivants se produisent en parallèle :
    - `StatefulServiceBase.CreateServiceReplicaListeners()` est appelé. 
      - Si le service est un service principal, tous les écouteurs retournés sont ouverts. `ICommunicationListener.OpenAsync()` est appelé sur chaque écouteur.
      - Si le service est un service secondaire, seuls les écouteurs marqués comme `ListenOnSecondary = true` sont ouverts. Il est plus rare d’avoir des écouteurs ouverts dans les services secondaires.
    - Si le service est un service principal, la méthode `StatefulServiceBase.RunAsync()` du service est appelée.
4. Une fois que tous les appels `OpenAsync()` de l’écouteur de réplica sont terminés et que `RunAsync()` est appelé, `StatefulServiceBase.OnChangeRoleAsync()` est appelé. Il est rare de remplacer cet appel dans le service.

Comme pour les services sans état, il n’y a aucune coordination entre l’ordre dans lequel les écouteurs sont créés et ouverts, et le moment auquel **RunAsync** est appelé. Si vous avez besoin d’une coordination, les solutions à utiliser sont similaires. Il existe un autre cas pour un service avec état. Supposons que les appels qui arrivent sur les écouteurs de communication nécessitent des informations conservées dans des [collections fiables](service-fabric-reliable-services-reliable-collections.md).

   > [!NOTE]  
   > Étant donné que les écouteurs de communication peuvent s’ouvrir avant que les collections fiables ne soient accessibles en lecture ou en écriture, et avant que **RunAsync** ne démarre, une coordination supplémentaire est nécessaire. La solution la plus simple et la plus courante est que les écouteurs de communication retournent un code d’erreur que le client utilise pour relancer la requête.

## <a name="stateful-service-shutdown"></a>Arrêt de service avec état
Comme pour les services sans état, les événements de cycle de vie lors de l’arrêt sont les mêmes que lors du démarrage, mais dans l’ordre inverse. Lorsqu’un service avec état est arrêté, les événements suivants se produisent :

1. En parallèle :
    - Les écouteurs ouverts sont fermés. `ICommunicationListener.CloseAsync()` est appelé sur chaque écouteur.
    - Le jeton d’annulation passé à `RunAsync()` est annulé. Une vérification que la propriété `IsCancellationRequested` du jeton d’annulation retourne la valeur true et que, si elle est appelée, la méthode `ThrowIfCancellationRequested` du jeton lève une `OperationCanceledException`.
2. Quand `CloseAsync()` se termine sur chaque écouteur et que `RunAsync()` se termine également, la méthode `StatefulServiceBase.OnChangeRoleAsync()` du service est appelée. Il est rare de remplacer cet appel dans le service.

   > [!NOTE]  
   > Ce n’est que si ce réplica est un réplica principal que vous devez attendre que **RunAsync** se termine.

3. Une fois la méthode `StatefulServiceBase.OnChangeRoleAsync()` terminée, la méthode `StatefulServiceBase.OnCloseAsync()` est appelée. Il s’agit d’un remplacement rare, mais possible.
3. Quand `StatefulServiceBase.OnCloseAsync()` se termine, l’objet de service est détruit.

## <a name="stateful-service-primary-swaps"></a>Échanges de réplica principal de service avec état
Pendant l’exécution d’un service avec état, seuls les réplicas principaux de ce service avec état ont leurs écouteurs de communication ouverts et leur méthode **RunAsync** appelée. Les réplicas secondaires sont construits, mais ne reçoivent aucun autre appel. Pendant l’exécution d’un service avec état, le réplica principal peut changer à la suite d’une panne ou d’une optimisation de l’équilibrage du cluster. Que cela signifie-t-il en termes des événements du cycle de vie qu’un réplica peut voir ? Le comportement que voit le réplica avec état varie selon qu’il s’agit du réplica qui est rétrogradé ou promu au cours de l’échange.

### <a name="for-the-primary-thats-demoted"></a>Pour le réplica principal rétrogradé
Pour le réplica principal rétrogradé, Service Fabric nécessite que ce réplica arrête de traiter des messages et quitte tout travail en arrière-plan qu’il exécute. Par conséquent, cette étape est similaire à l’arrêt du service. La différence est que le service n’est pas détruit ni fermé, car il est conservé en tant que réplica secondaire. Les API suivantes sont appelées :

1. En parallèle :
    - Les écouteurs ouverts sont fermés. `ICommunicationListener.CloseAsync()` est appelé sur chaque écouteur.
    - Le jeton d’annulation passé à `RunAsync()` est annulé. Une vérification que la propriété `IsCancellationRequested` du jeton d’annulation retourne la valeur true et que, si elle est appelée, la méthode `ThrowIfCancellationRequested` du jeton lève une `OperationCanceledException`.
2. Quand `CloseAsync()` se termine sur chaque écouteur et que `RunAsync()` se termine également, la méthode `StatefulServiceBase.OnChangeRoleAsync()` du service est appelée. Il est rare de remplacer cet appel dans le service.

### <a name="for-the-secondary-thats-promoted"></a>Pour le réplica secondaire promu
De même, Service Fabric nécessite que le réplica secondaire promu commence à écouter les messages sur le réseau et qu’il démarre les tâches en arrière-plan qui le concernent. Par conséquent, ce processus est similaire à la création du service, à ceci près que le réplica lui-même existe déjà. Les API suivantes sont appelées :

1. En parallèle :
    - `StatefulServiceBase.CreateServiceReplicaListeners()` est appelé et les écouteurs retournés sont ouverts. `ICommunicationListener.OpenAsync()` est appelé sur chaque écouteur.
    - La méthode `StatefulServiceBase.RunAsync()` du service est appelée.
2. Une fois que tous les appels `OpenAsync()` de l’écouteur de réplica sont terminés et que `RunAsync()` est appelé, `StatefulServiceBase.OnChangeRoleAsync()` est appelé. Il est rare de remplacer cet appel dans le service.

### <a name="common-issues-during-stateful-service-shutdown-and-primary-demotion"></a>Problèmes courants se produisant pendant l’arrêt d’un service avec état et la rétrogradation du réplica principal
Service Fabric peut modifier le réplica principal d’un service avec état pour diverses raisons. Parmi les raisons les plus courantes figurent le [rééquilibrage des clusters](service-fabric-cluster-resource-manager-balancing.md) et la [mise à niveau des applications](service-fabric-application-upgrade.md). Pendant ces opérations (ainsi que lors de l’arrêt normal du service, comme lorsque le service a été supprimé), il est important que le service respecte le `CancellationToken`. 

Les services qui ne gèrent pas l’annulation « proprement » peuvent être confrontés à plusieurs problèmes. Ces opérations sont lentes, car Service Fabric attend que les services s’arrêtent normalement. Cela peut entraîner l’échec des mises à jour qui expirent et sont annulées. Si vous ne respectez pas le jeton d’annulation, vous pouvez également provoquer des clusters déséquilibrés. Les clusters sont déséquilibrés, car les nœuds deviennent actifs, mais les services ne peuvent pas être rééquilibrés, car leur déplacement prend trop de temps. 

Comme les services sont avec état, il est également probable qu’ils utilisent des [collections fiables](service-fabric-reliable-services-reliable-collections.md). Dans Service Fabric, lorsqu’un réplica principal est rétrogradé, l’une des premières choses qui se produisent est que l’accès en écriture à l’état sous-jacent est révoqué. Cela conduit à un deuxième type de problème qui peut avoir un impact sur le cycle de vie du service. Les collections retournent des exceptions selon le délai et selon que le réplica est déplacé ou arrêté. Ces exceptions doivent être gérées correctement. Les exceptions levées par Service Fabric sont soit permanentes [(`FabricException`)](/dotnet/api/system.fabric.fabricexception?view=azure-dotnet), soit passagères [(`FabricTransientException`)](/dotnet/api/system.fabric.fabrictransientexception?view=azure-dotnet). Les exceptions permanentes doivent être journalisées et levées, alors que les exceptions passagères peuvent être retentées selon une logique de nouvelle tentative.

Le fait de gérer des exceptions issues de l’utilisation de `ReliableCollections` conjointement aux événements du cycle de vie du service constitue une partie importante du test et de la validation d’un service fiable. Il est recommandé de toujours exécuter votre service sous une charge quand vous effectuez une mise à niveau et des [tests de chaos](service-fabric-controlled-chaos.md) avant de le déployer sur un environnement de production. Ces étapes de base vous permettent de vérifier que votre service est correctement implémenté et qu’il gère correctement les événements de cycle de vie.


## <a name="notes-on-the-service-lifecycle"></a>Remarques sur le cycle de vie du service
  - La méthode `RunAsync()` et l’appel `CreateServiceReplicaListeners/CreateServiceInstanceListeners` sont facultatifs. Un service peut avoir l’un des deux, les deux ou aucun. Par exemple, si le service effectue tout son travail en réponse aux appels d’utilisateur, il est inutile d’implémenter `RunAsync()`. Seuls les écouteurs de communication et le code associé sont nécessaires. De même, la création et le renvoi d’écouteurs de communication sont facultatifs, car le service peut n’avoir que du travail en arrière-plan à exécuter et n’a donc besoin d’implémenter que `RunAsync()`.
  - Il est valide pour un service de terminer `RunAsync()` correctement et d’en revenir. Le fait qu’il se termine ne correspond pas à une condition d’échec. Le fait de terminer `RunAsync()` indique que le travail d’arrière-plan du service est terminé. Pour les services fiables avec état, `RunAsync()` est appelé à nouveau si le réplica principal est devenu un réplica secondaire, puis est repassé à l’état de réplica principal.
  - Si un service quitte `RunAsync()` en levant une exception inattendue, il s’agit d’un échec. L’objet de service est arrêté et une erreur d’intégrité est signalée.
  - Même s’il n’existe pas de limite de temps sur le renvoi de ces méthodes, vous perdez immédiatement la capacité d’écrire dans les collections fiables, et vous ne pouvez donc pas effectuer de travail réel. Il est recommandé de procéder au renvoi aussi rapidement que possible dès la réception de la demande d’annulation. Si votre service ne répond pas à ces appels d’API dans un délai raisonnable, Service Fabric peut mettre fin à votre service. En général, cela se produit uniquement lors de mises à niveau d’application ou lorsqu’un service est en cours de suppression. Par défaut, ce délai d’attente est de 15 minutes.
  - Les échecs dans le chemin `OnCloseAsync()` entraînent l’appel de `OnAbort()` qui constitue une opportunité de dernière chance pour le service de nettoyer et de libérer les ressources demandées. Cette méthode est généralement appelée lorsqu'une erreur permanente est détectée sur le nœud, ou lorsque Service Fabric ne peut pas gérer de façon fiable le cycle de vie de l'instance du service en raison de défaillances internes.
  - La méthode `OnChangeRoleAsync()` est appelée quand le réplica de service avec état change de rôle, par exemple, quand il devient un service principal ou secondaire. Les réplicas principaux se voient affecter un état d’écriture (ils sont autorisés à créer des collections fiables et à écrire dans celles-ci). Les réplicas secondaires se voient affecter un état de lecture (ils peuvent uniquement lire à partir de collections fiables existantes). La plupart du travail dans un service avec état est effectué sur le réplica principal. Les réplicas secondaires peuvent effectuer la validation en lecture seule, la génération de rapports, l’exploration de données ou d’autres tâches en lecture seule.

## <a name="next-steps"></a>Étapes suivantes
- [Présentation de Reliable Services](service-fabric-reliable-services-introduction.md)
- [Démarrage rapide de Reliable Services](service-fabric-reliable-services-quick-start.md)
- [Réplicas et instances](service-fabric-concepts-replica-lifecycle.md)

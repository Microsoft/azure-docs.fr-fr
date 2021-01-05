---
title: Test unitaire des services avec état dans Azure Service Fabric
description: Découvrez les concepts et pratiques de test unitaire des services avec état dans Service Fabric.
ms.topic: conceptual
ms.date: 09/04/2018
ms.openlocfilehash: 12e8a47d9685dee12594f4e2afaa848d9688d185
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75433907"
---
# <a name="unit-testing-stateful-services-in-service-fabric"></a>Test unitaire des services avec état dans Service Fabric

Cet article aborde les concepts et pratiques de test unitaire des services avec état dans Service Fabric. Les tests unitaires dans Service Fabric méritent leurs propres considérations, eu égard au fait que le code d’application s’exécute activement sous plusieurs contextes différents. Cet article décrit les pratiques permettant de garantir la couverture du code d’application dans chacun des contextes sous lesquels il peut s’exécuter.

## <a name="unit-testing-and-mocking"></a>Tests unitaires et simulation
Les tests unitaires dans le contexte de cet article sont des tests automatisés qui peuvent être exécutés dans le contexte d’un exécuteur de tests tel que MSTest ou NUnit. Les tests unitaires décrits dans cet article n’effectuent pas d’opérations sur une ressource distante telle qu’une base de données ou une API RESTFul. Ces ressources distantes doivent être simulées. La simulation dans le contexte de cet article simule, enregistre et contrôle les valeurs de retour pour les ressources distantes.

### <a name="service-fabric-considerations"></a>Considérations relatives à Service Fabric
Quand vous exécutez des tests unitaires sur un service avec état Service Fabric, vous devez prendre en considération plusieurs facteurs. Tout d’abord, le code de service s’exécute sur plusieurs nœuds, mais sous différents rôles. Les tests unitaires doivent évaluer le code sous chaque rôle afin d’obtenir une couverture complète. Les différents rôles sont Principal, Secondaire actif, Secondaire inactif et Inconnu. Le rôle Aucun n’a généralement pas besoin de couverture spéciale, car Service Fabric considère que ce rôle correspond à un service vide ou nul. Ensuite, chaque nœud change son rôle à un moment donné. Pour obtenir une couverture complète, le chemin d’exécution du code doit être testé avec des changements de rôle se produisant.

## <a name="why-unit-test-stateful-services"></a>Pourquoi exécuter des tests unitaires sur les services avec état ? 
Les tests unitaires sur les services avec état peuvent aider à identifier certaines erreurs courantes qui ne seraient pas forcément interceptées par les tests unitaires conventionnels propres à une application ou à un domaine. Par exemple, si le service avec état a un état en mémoire, ce type de test peut vérifier que cet état en mémoire est synchronisé sur chaque réplica. Ce type de test peut également vérifier qu’un service avec état répond correctement aux jetons d’annulation passés par l’orchestration Service Fabric. Quand des annulations sont déclenchées, le service doit arrêter toutes les opérations de longue durée et/ou asynchrones.  

## <a name="common-practices"></a>Pratiques courantes

La section suivante fournit des conseils sur les pratiques les plus courantes de test unitaire d’un service avec état. Elle fournit aussi des conseils sur ce que doit avoir une couche de simulation afin de s’aligner précisément avec l’orchestration et la gestion de l’état Service Fabric. À compter de la version 3.3.0 ou ultérieure, la bibliothèque [ServiceFabric.Mocks](https://www.nuget.org/packages/ServiceFabric.Mocks/) fournit la fonctionnalité de simulation recommandée et respecte les pratiques décrites ci-dessous.

### <a name="arrangement"></a>Disposition

#### <a name="use-multiple-service-instances"></a>Utiliser plusieurs instances de service
Les tests unitaires doivent exécuter plusieurs instances d’un service avec état. Cela simule ce qui se passe réellement sur le cluster où Service Fabric provisionne plusieurs réplicas exécutant votre service sur différents nœuds. Toutefois, chacune de ces instances s’exécutera sous un contexte différent. Lors de l’exécution du test, chaque instance doit être préparée avec la configuration de rôle attendue sur le cluster. Par exemple, si le service est censé avoir une taille de réplica cible de 3, Service Fabric doit provisionner trois réplicas sur différents nœuds. L’un d’entre eux sera le réplica principal, tandis que les deux autres seront des réplicas secondaires actifs.

Dans la plupart des cas, le chemin d’exécution du service varie légèrement pour chacun de ces rôles. Par exemple, si le service ne doit pas accepter de requête en provenance d’un réplica secondaire actif, il peut avoir une vérification pour ce cas afin de lever une exception informative signalant qu’une tentative de requête a eu lieu sur un réplica secondaire. Le fait d’avoir plusieurs instances permettra de tester cette situation.

Cela permettra également aux tests de permuter les rôles de chacune de ces instances pour vérifier que les réponses sont cohérentes malgré les changements de rôle.

#### <a name="mock-the-state-manager"></a>Simuler le gestionnaire d’état
Le gestionnaire d’état doit être considéré comme une ressource distante, et doit par conséquent être simulé. Lors de la simulation du gestionnaire d’état, il doit exister un stockage en mémoire sous-jacent pour effectuer le suivi de ce qui est enregistré dans le gestionnaire d’état, afin qu’il puisse être lu et vérifié. Un moyen simple d’y parvenir consiste à créer des instances fictives de chacun des types de collections fiables. Dans ces simulations, utilisez un type de données qui s’aligne étroitement avec les opérations effectuées sur cette collection. Voici quelques suggestions de types de données pour chaque collection fiable :

- IReliableDictionary<TKey, TValue> -> System.Collections.Concurrent.ConcurrentDictionary<TKey, TValue>
- IReliableQueue\<T> -> System.Collections.Generic.Queue\<T>
- IReliableConcurrentQueue\<T> -> System.Collections.Concurrent.ConcurrentQueue\<T>

#### <a name="many-state-manager-instances-single-storage"></a>Nombreuses instances de gestionnaire d’état, stockage unique
Comme mentionné plus haut, le gestionnaire d’état et les collections fiables doivent être traités comme des ressources distantes. Par conséquent, ces ressources doivent être simulées et seront simulées dans les tests unitaires. Toutefois, lors de l’exécution de plusieurs instances d’un service avec état, il est délicat de préserver la synchronisation de chaque gestionnaire d’état fictif parmi différentes instances de service avec état. Quand le service avec état s’exécute sur le cluster, Service Fabric se charge de maintenir la cohérence du gestionnaire d’état de chaque réplica secondaire avec le réplica principal. Ainsi, les tests doivent se comporter de la même manière, afin de pouvoir simuler les changements de rôle.

Pour obtenir cette synchronisation, une approche simple consiste à utiliser un modèle singleton pour l’objet sous-jacent qui stocke les données écrites dans chaque collection fiable. Par exemple, si un service avec état utilise un `IReliableDictionary<string, string>`, le gestionnaire d’état fictif doit retourner une simulation de `IReliableDictionary<string, string>`. Cette simulation peut utiliser un `ConcurrentDictionary<string, string>` pour effectuer le suivi des paires clé/valeur écrites. Le `ConcurrentDictionary<string, string>` doit être un singleton utilisé par toutes les instances du gestionnaire d’état passées au service.

#### <a name="keep-track-of-cancellation-tokens"></a>Effectuer le suivi des jetons d’annulation
Les jetons d’annulation sont un aspect important mais souvent négligé des services avec état. Quand Service Fabric démarre un réplica principal pour un service avec état, un jeton d’annulation est fourni. Ce jeton d’annulation est destiné à signaler au service quand il est supprimé ou rétrogradé à un rôle différent. Le service avec état doit arrêter toutes les opérations de longue durée ou asynchrones afin que Service Fabric puisse terminer le workflow de changement de rôle.

Lors de l’exécution de tests unitaires, tous les jetons d’annulation fournis à RunAsync, ChangeRoleAsync, OpenAsync et CloseAsync doivent être conservés pendant l’exécution du test. La conservation de ces jetons permettra au test de simuler un arrêt ou une rétrogradation du service, et de vérifier que le service répond de manière appropriée.

#### <a name="test-end-to-end-with-mocked-remote-resources"></a>Tester de bout en bout avec des ressources distantes simulées
Les tests unitaires doivent exécuter la plus grande partie possible du code d’application capable de changer l’état du service avec état. Nous vous recommandons d’implémenter des tests de bout en bout par nature. Les seules simulations existantes servent à enregistrer, à simuler et/ou à vérifier les interactions entre les ressources distantes, notamment celles avec le gestionnaire d’état et les collections fiables. L’extrait de code suivant illustre un test de bout en bout :

```
    Given stateful service named "fabric:/MyApp/MyService" is created
    And a new replica is created as "Primary" with id "111"
    And a new replica is created as "IdleSecondary" with id "222"
    And a new replica is created as "IdleSecondary" with id "333"
    And all idle secondary replicas are promoted to active secondary
    When a request is made to add the an employee "John Smith"
    And the active secondary replica "222" is promoted to primary
    And a request is made to get all employees
    Then the request should should return the "John Smith" employee
```

Ce test vérifie que les données capturées sur un réplica sont accessibles à un réplica secondaire quand il promu au rôle de réplica principal. En supposant qu’une collection fiable soit le magasin de stockage pour les données sur les employés, ce test pourrait par exemple intercepter une défaillance selon laquelle le code d’application n’aurait pas exécuté `CommitAsync` sur la transaction pour enregistrer le nouvel employé. Dans ce cas, la deuxième requête pour obtenir les employés ne retournerait pas l’employé ajouté par la première requête.

### <a name="acting"></a>Action
#### <a name="mimic-service-fabric-replica-orchestration"></a>Imiter l’orchestration de réplica Service Fabric
Lors de la gestion de plusieurs instances de service, les tests doivent initialiser et désactiver ces services de la même manière que l’orchestration Service Fabric. Par exemple, quand un service est créé sur un nouveau réplica principal, Service Fabric appelle CreateServiceReplicaListener, OpenAsync, ChangeRoleAsync et RunAsync. Les événements de cycle de vie sont documentés dans les articles suivants :

- [Démarrage de service avec état](service-fabric-reliable-services-lifecycle.md#stateful-service-startup)
- [Arrêt de service avec état](service-fabric-reliable-services-lifecycle.md#stateful-service-shutdown)
- [Échanges de réplica principal de service avec état](service-fabric-reliable-services-lifecycle.md#stateful-service-primary-swaps)

#### <a name="run-replica-role-changes"></a>Exécuter des changements de rôle de réplica
Les tests unitaires doivent changer les rôles des instances de service de la même manière que l’orchestration Service Fabric. Les rôles des réplicas sont documenté dans l’article suivant :

[Rôle du réplica](service-fabric-concepts-replica-lifecycle.md#replica-role)

La simulation des changements de rôle est l’un des aspects les plus critiques des tests, et elle peut révéler des problèmes comme l’incohérence entre les états des réplicas. Cette incohérence peut être due au stockage de l’état en mémoire dans des variables statiques ou d’instances de niveau classe (par exemple jetons d’annulation, énumérations et valeurs/objets de configuration). Cette simulation garantit également que le service respecte les jetons d’annulation fournis pendant RunAsync, afin que le changement de rôle puisse avoir lieu. Elle peut également déceler des problèmes qui peuvent survenir si le code n’est pas écrit pour autoriser un appel de RunAsync à plusieurs reprises.

#### <a name="cancel-cancellation-tokens"></a>Annuler les jetons d’annulation
Il doit exister des tests unitaires dans lesquels le jeton d’annulation fourni à RunAsync est annulé. Le test pourra ainsi vérifier que le service s’arrête de façon correcte. Durant cet arrêt, toutes les opérations asynchrones de longue durée doivent être arrêtées. Un service à l’écoute des messages sur une file d’attente fiable constitue un exemple de processus de longue durée sur un service. Il peut exister directement dans RunAsync ou un thread d’arrière-plan. L’implémentation doit inclure une logique permettant de quitter l’opération si ce jeton d’annulation est annulé.

Si les services avec état utilisent un état de cache ou en mémoire qui doit exister uniquement sur le réplica principal, il doit être supprimé à ce moment-là, ceci afin de garantir la cohérence de cet état si le nœud redevient plus tard un réplica principal. Le test d’annulation permettra au test de vérifier que cet état est supprimé correctement.

#### <a name="execute-requests-against-multiple-replicas"></a>Exécuter des requêtes sur plusieurs réplicas
Les tests d’assertion doivent exécuter la même requête sur différents réplicas. En cas d’association avec les changements de rôle, des problèmes de cohérence peuvent être détectés. Un exemple de test peut effectuer les étapes suivantes :
1. Exécution d’une requête d’écriture sur le réplica principal actuel
2. Exécution d’une requête de lecture qui retourne les données écrites à l’étape 1 sur le réplica principal actuel
3. Promotion d’un réplica secondaire en réplica principal. Le réplica principal actuel doit également être rétrogradé en réplica secondaire
4. Exécution de la même requête de lecture qu’à l’étape 2 sur le nouveau réplica secondaire

Lors de la dernière étape, le test peut vérifier que les données retournées sont cohérentes. L’un des problèmes susceptibles d’être détectés par ces tests serait que les données retournées par le service pourraient être en mémoire, mais secondées par une collection fiable. Ces données en mémoire risqueraient de ne pas être synchronisées correctement avec ce qui existe dans la collection fiable.

Les données en mémoire sont généralement utilisées pour créer des index secondaires ou des agrégations de données qui existent dans une collection fiable.

### <a name="asserting"></a>Assertion
#### <a name="ensure-responses-match-across-replicas"></a>Vérifier que les réponses correspondent d’un réplica à un autre
Les tests unitaires doivent vérifier qu’une réponse à une requête donnée est cohérente sur les différents réplicas après la transition en réplica principal. Cela peut permettre de détecter des problèmes potentiels, où les données fournies dans la réponse ne sont pas secondées par une collection fiable ou sont conservées en mémoire sans aucun mécanisme pour synchroniser ces données entre les réplicas. Cela permet de s’assurer que le service renvoie des réponses cohérentes une fois que Service Fabric a rééquilibré ou basculé vers un nouveau réplica principal.

#### <a name="verify-service-respects-cancellation"></a>Vérifier que le service respecte l’annulation
Vous devez vérifier que les processus asynchrones ou de longue durée qui doivent être arrêtés quand un jeton d’annulation est annulé sont bien arrêtés après l’annulation. Cela permet de s’assurer que malgré le changement de rôle de réplica, les processus dont l’exécution n’est pas censée continuer sur un réplica non principal s’arrêtent avant la fin de la transition. Cela peut également révéler des problèmes où un tel processus empêche une demande d’arrêt ou de changement de rôle envoyée par Service Fabric de se terminer.

#### <a name="verify-which-replicas-should-serve-requests"></a>Vérifier quels réplicas doivent traiter les requêtes
Les tests doivent vérifier le comportement attendu si une requête est routée vers un réplica autre qu’un réplica principal. Service Fabric permet de faire en sorte que les réplicas secondaires traitent les requêtes. Toutefois, les écritures dans les collections fiables peuvent se produire uniquement à partir du réplica principal. Si votre application souhaite que seuls les réplicas principaux répondent aux requêtes ou que seul un sous-ensemble de requêtes puissent être gérées par un réplica secondaire, les tests doivent vérifier le comportement attendu pour les cas positifs et négatifs. Le cas négatif est quand une requête est acheminée vers un réplica qui ne doit pas la gérer, et le cas positif est l’inverse.

## <a name="next-steps"></a>Étapes suivantes
Découvrez comment [créer des tests unitaires sur des services avec état](service-fabric-how-to-unit-test-stateful-services.md).

---
title: Présentation de Reliable Collections
description: Les services avec état de Service Fabric fournissent des collections fiables qui vous permettent d’écrire des applications cloud hautement disponibles, évolutives et à faible latence.
ms.topic: conceptual
ms.date: 3/10/2020
ms.openlocfilehash: 6e0f39b994087fe44038e62e85326945589c719a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86245124"
---
# <a name="introduction-to-reliable-collections-in-azure-service-fabric-stateful-services"></a>Introduction aux Collections fiables dans les services avec état d’Azure Service Fabric

Les Collections fiables vous permettent d’écrire des applications cloud hautement disponibles, évolutives et à faible latence comme si vous écriviez des applications informatiques uniques. Les classes dans l’espace de noms **Microsoft.ServiceFabric.Data.Collections** fournissent un ensemble de collections qui rendent automatiquement votre état hautement disponible. Les développeurs doivent programmer uniquement les API de Collection fiable et laisser les Collections fiables gérer l’état répliqué et local.

La principale différence entre les Collections fiables et d'autres technologies de haute disponibilité (comme Redis, le service Table Azure et le service File d'attente Azure) est que l'état est conservé localement dans l'instance de service tout en étant également rendu hautement disponible. Cela signifie que :

* Toutes les lectures sont locales, ce qui entraîne des lectures à faible latence et à débit élevé.
* Toutes les écritures induisent un nombre minimal d’entrées et sorties réseau, ce qui entraîne des écritures à latence faible et à débit élevé.

![Image de l'évolution des collections.](media/service-fabric-reliable-services-reliable-collections/ReliableCollectionsEvolution.png)

Les Collections fiables peuvent être considérées comme l’évolution naturelle des classes **System.Collections** : un nouveau jeu de collections qui sont conçues pour les applications cloud et les applications pour plusieurs ordinateurs sans accroître la complexité pour les développeurs. En tant que telles, les Collections fiables sont :

* Répliquées : les modifications d’état sont répliquées pour une haute disponibilité.
* Asynchrones : les API sont asynchrones afin que les threads ne soient pas bloqués en cas d’entrées/sorties.
* Transactionnelles : les API utilisent l’abstraction de transactions pour faciliter la gestion de plusieurs Collections fiables au sein d’un service.
* Persistantes ou volatiles : les données peuvent être conservées sur le disque afin de les protéger des pannes à grande échelle (par exemple, une panne de courant dans un centre de données). Certaines collections fiables prennent également en charge un mode volatile (avec [mises en garde](service-fabric-reliable-services-reliable-collections-guidelines.md#volatile-reliable-collections)) où toutes les données sont conservées en mémoire, à la manière d'un cache en mémoire répliqué.

Les Collections fiables fournissent des garanties de forte cohérence instantanée afin de faciliter le raisonnement sur l'état de l'application.
La forte cohérence est obtenue en s’assurant que la transaction n’est validée comme terminée que lorsque l’intégralité de la transaction a été enregistrée dans un quorum majoritaire de réplicas, y compris le réplica principal.
Pour obtenir une cohérence plus faible, les applications peuvent accuser réception au client/demandeur avant la validation asynchrone.

Les API de Collections fiables sont une évolution des API de collections simultanées (trouvées dans l’espace de noms **System.Collections.Concurrent** ) :

* Asynchrones : renvoie une tâche, car contrairement aux collections simultanées, les opérations sont répliquées et conservées.
* Aucun paramètre de sortie : utilise `ConditionalValue<T>` pour retourner un `bool` et une valeur au lieu des paramètres de sortie. `ConditionalValue<T>` est similaire à `Nullable<T>`, mais ne nécessite ne pas que T soit une structure.
* Transactions : utilise un objet de transaction pour permettre à l’utilisateur de regrouper des actions sur plusieurs Collections fiables dans une transaction.

Actuellement, **Microsoft.ServiceFabric.Data.Collections** contient trois collections :

* [Dictionnaire fiable](/dotnet/api/microsoft.servicefabric.data.collections.ireliabledictionary-2?view=azure-dotnet#microsoft_servicefabric_data_collections_ireliabledictionary_2) : représente une collection répliquée, transactionnelle et asynchrone de paires clé/valeur. Semblables à celles de **ConcurrentDictionary**, la clé et la valeur peuvent être de tout type.
* [File d’attente fiable](/dotnet/api/microsoft.servicefabric.data.collections.ireliablequeue-1?view=azure-dotnet#microsoft_servicefabric_data_collections_ireliablequeue_1) : représente une file d’attente FIFO stricte, répliquée, transactionnelle et asynchrone. Semblable à celle de **ConcurrentQueue**, la valeur peut être de tout type.
* [File d’attente simultanée fiable](service-fabric-reliable-services-reliable-concurrent-queue.md) : représente une file d’attente de classement de meilleur effort répliquée, transactionnelle et asynchrone, pour un débit élevé. Semblable à celle de **ConcurrentQueue**, la valeur peut être de tout type.

## <a name="next-steps"></a>Étapes suivantes

* [Instructions et recommandations relatives aux collections fiables](service-fabric-reliable-services-reliable-collections-guidelines.md)
* [Utilisation des collections fiables](service-fabric-work-with-reliable-collections.md)
* [Transactions et verrous](service-fabric-reliable-services-reliable-collections-transactions-locks.md)
* Gestion des données
  * [Sauvegarde et restauration](service-fabric-reliable-services-backup-restore.md)
  * [Notifications](service-fabric-reliable-services-notifications.md)
  * [Sérialisation de Collection fiable](service-fabric-reliable-services-reliable-collections-serialization.md)
  * [Sérialisation et mise à niveau](service-fabric-application-upgrade-data-serialization.md)
  * [Configuration du Gestionnaire d’état fiable](service-fabric-reliable-services-configuration.md)
* Autres
  * [Démarrage rapide de Reliable Services](service-fabric-reliable-services-quick-start.md)
  * [Référence du développeur pour les Collections fiables](/dotnet/api/microsoft.servicefabric.data.collections?view=azure-dotnet#microsoft_servicefabric_data_collections)

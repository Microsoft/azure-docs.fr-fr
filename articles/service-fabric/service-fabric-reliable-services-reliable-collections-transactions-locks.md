---
title: Transactions et modes de verrouillage dans les collections Reliable
description: Verrouillage et transactions des collections fiables et gestionnaire d’état fiable Azure Service Fabric.
ms.topic: conceptual
ms.date: 5/1/2017
ms.openlocfilehash: 57ca46047641b79d5e4c50ede4a27e16dcec5d89
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96576721"
---
# <a name="transactions-and-lock-modes-in-azure-service-fabric-reliable-collections"></a>Modes Transactions et Verrouillage dans les Collections fiables Azure Service Fabric

## <a name="transaction"></a>Transaction

Une transaction est une suite d'opérations effectuées comme une seule unité logique de travail. Elle présente les propriétés [ACID](https://en.wikipedia.org/wiki/ACID) (*atomicité*,  *cohérence*, *isolation* , *durabilité* ) communes des transactions de base de données :

* **Atomicité** : une transaction doit correspondre à une unité de travail atomique. En d’autres termes, soit toutes les modifications de données sont effectuées, soit aucune n’est effectuée.
* **Cohérence** : Lorsqu'elle est terminée, une transaction doit laisser les données dans un état cohérent. Toutes les structures de données internes doivent être correctes à la fin de la transaction.
* **Isolation** : Les modifications effectuées par des transactions concurrentes doivent être isolées transaction par transaction. Le niveau d'isolation utilisé pour une opération dans le cadre d'une [ITransaction](/dotnet/api/microsoft.servicefabric.data.itransaction?view=azure-dotnet) est déterminé par l'état [IReliableState](/dotnet/api/microsoft.servicefabric.data.ireliablestate?view=azure-dotnet) qui effectue l'opération.
* **Durabilité** : lorsqu’une transaction est terminée, ses effets sont définitivement en place dans le système. Les modifications sont conservées même en cas de défaillance du système.

### <a name="isolation-levels"></a>Niveaux d'isolement

Le niveau d’isolement définit le degré auquel la transaction doit être isolée des modifications apportées par d’autres transactions.
Il existe deux niveaux d'isolement pris en charge dans les Collections fiables :

* **Lecture renouvelée** : spécifie que les instructions ne peuvent pas lire les données qui ont été modifiées, mais pas encore validées par d’autres transactions et qu’aucune autre transaction ne peut modifier des données qui ont été lues par la transaction actuelle avant la fin de celle-ci.
* **Capture instantanée** : spécifie que les données lues par une instruction dans une transaction sont la version transactionnellement cohérente des données qui existaient au début de la transaction.
  La transaction ne peut reconnaître que les modifications de données qui ont été validées avant son démarrage.
  Autrement dit, les modifications de données effectuées par d'autres transactions après le début de la transaction active ne sont pas visibles pour les instructions qui s'exécutent dans le cadre de ladite transaction.
  C’est comme si les instructions d’une transaction obtenaient un instantané des données validées telles qu’elles existaient au début de la transaction.
  Les instantanés sont cohérents sur les Collections fiables.

Les Collections fiables choisissent automatiquement le niveau d'isolement à utiliser pour une opération de lecture donnée en fonction de l'opération et du rôle du réplica lors de la création de la transaction.
Voici le tableau qui décrit les valeurs par défaut du niveau d’isolement pour les opérations Dictionnaire fiable et File d’attente fiable.

| Fonctionnement \ Rôle | Principal | Secondary |
| --- |:--- |:--- |
| Lecture d'une seule entité |Lecture renouvelée |Instantané |
| Énumération, décompte |Instantané |Instantané |

> [!NOTE]
> `IReliableDictionary.TryGetValueAsync` et `IReliableQueue.TryPeekAsync` sont des exemples typiques d’opérations à une seule entité.
> 

Les opérations Dictionnaire fiable et File d'attente fiable prennent en charge le protocole *Read Your Writes*.
En d'autres termes, toute écriture dans une transaction sera visible pour une lecture suivante appartenant à la même transaction.

## <a name="locks"></a>Verrous

Dans les Collections fiables, toutes les transactions implémentent rigoureusement un verrouillage à deux phases : une transaction ne libère pas les verrous qu’elle a acquis tant qu’elle ne s’est pas terminée avec un abandon ou une validation.

Dictionnaire fiable utilise le verrou au niveau des lignes pour toutes les opérations à une seule entité.
File d’attente fiable compense la concurrence par une propriété FIFO transactionnelle stricte.
File d'attente fiable utilise des verrous au niveau des opérations qui permet une transaction avec `TryPeekAsync` et/ou `TryDequeueAsync`, et une transaction avec `EnqueueAsync` à la fois.
Notez que pour conserver FIFO, si une opération `TryPeekAsync` ou `TryDequeueAsync` constate que la file d’attente fiable est vide, elles verrouilleront également `EnqueueAsync`.

Les opérations d’écriture prennent toujours des verrous exclusifs.
Pour les opérations de lecture, le verrou dépend de deux facteurs :

- Les opérations de lecture effectuées à l'aide de l'isolement d'instantané ne sont pas verrouillées.
- Les opérations de lecture renouvelée utilisent par défaut des verrous partagés.
- Toutefois, pour toutes les opérations de lecture qui prennent en charge la lecture renouvelée, l’utilisateur peut demander un verrou de mise à jour au lieu du verrou partagé.
Un verrou de mise à jour est un verrou asymétrique utilisé pour éviter une forme courante de blocage qui se produit lorsque plusieurs transactions verrouillent les ressources pour des mises à jour éventuelles ultérieures.

La matrice de compatibilité de verrouillage est disponible dans la table ci-après :

| Requête \ Accordé | None | Partagé | Update | Exclusif |
| --- |:--- |:--- |:--- |:--- |
| Partagé |Aucun conflit |Aucun conflit |Conflit |Conflit |
| Update |Aucun conflit |Aucun conflit |Conflit |Conflit |
| Exclusif |Aucun conflit |Conflit |Conflit |Conflit |

Dans les API Collections fiables, l'argument timeout (délai d'expiration) est utilisé pour la détection des blocages.
Par exemple, deux transactions (T1 et T2) essayent de lire et de mettre à jour K1.
Elles peuvent être bloquées, car elles ont toutes deux le verrou partagé.
Dans ce cas, l’une ou les deux opérations arrivent à expiration. Dans ce scénario, un verrou de mise à jour peut empêcher ce type de blocage.

## <a name="next-steps"></a>Étapes suivantes

* [Utilisation des collections fiables](service-fabric-work-with-reliable-collections.md)
* [Notifications Reliable Services](service-fabric-reliable-services-notifications.md)
* [Sauvegarde et restauration de Reliable Services (récupération d’urgence)](service-fabric-reliable-services-backup-restore.md)
* [Configuration du Gestionnaire d’état fiable](service-fabric-reliable-services-configuration.md)
* [Référence du développeur pour les Collections fiables](/dotnet/api/microsoft.servicefabric.data.collections?view=azure-dotnet#microsoft_servicefabric_data_collections)

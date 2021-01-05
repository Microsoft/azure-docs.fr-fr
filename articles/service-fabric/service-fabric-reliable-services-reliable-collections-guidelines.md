---
title: Instructions pour les collections fiables
description: Instructions et recommandations relatives à l’utilisation de collections fiables Service Fabric dans une application Azure Service Fabric.
ms.topic: conceptual
ms.date: 03/10/2020
ms.openlocfilehash: 63e6de436bdaceed7f1d2a78e8385dd14bfc0ed6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86260920"
---
# <a name="guidelines-and-recommendations-for-reliable-collections-in-azure-service-fabric"></a>Instructions et recommandations pour les collections fiables dans Azure Service Fabric
Cette section fournit des instructions pour l’utilisation du gestionnaire d’état fiable et des collections fiables. L’objectif consiste à aider les utilisateurs à éviter les pièges courants.

Les directives sont organisées comme de simples recommandations avec les termes *Faire*, *Envisager*, *Éviter* et *Ne pas faire* en préfixe.

* Ne modifiez pas un objet de type personnalisé renvoyé par les opérations de lecture (par exemple, `TryPeekAsync` ou `TryGetValueAsync`). Les Collections fiables, comme les Collections simultanées, renvoient une référence aux objets et non une copie.
* Exécutez une copie complète de l’objet renvoyé de type personnalisé avant de le modifier. Étant donné que les structures et les types intégrés sont de passage par valeur, vous n’avez pas besoin d’y effectuer une copie complète, sauf s’ils contiennent des champs ou des propriétés saisis par référence que vous souhaitez modifier.
* N’utilisez pas `TimeSpan.MaxValue` pour les délais d’attente. Les délais d’expiration doivent être utilisés pour détecter des blocages.
* N’utilisez pas une transaction une fois qu’elle a été validée, abandonnée ou supprimée.
* N’utilisez pas une énumération en dehors de l’étendue de transaction dans laquelle elle a été créée.
* Ne créez pas de transaction au sein de l'instruction `using` d'une autre transaction car cela peut provoquer des blocages.
* Ne créez pas d’état fiable avec `IReliableStateManager.GetOrAddAsync` et utilisez l’état fiable dans la même transaction. Une exception InvalidOperationException en résulte.
* Assurez-vous que votre implémentation de `IComparable<TKey>` est correcte. Le système dépend de `IComparable<TKey>` pour la fusion des points de contrôle et des lignes.
* N’utilisez pas un verrou de mise à jour lors de la lecture d’un élément avec l’intention de le mettre à jour pour empêcher une certaine classe de blocages.
* Envisagez de conserver le nombre de Reliable Collections par partition sur un nombre inférieur à 1 000. Privilégiez les Reliable Collections avec plus d’éléments par rapport à plus de Reliable Collections avec moins d’éléments.
* Envisagez de conserver vos éléments (par ex. TKey + TValue pour le Dictionnaire fiable) sous 80 Ko : plus le volume est petit, mieux c’est. Cela réduit la quantité d’utilisation du Large Object Heap, ainsi que les exigences d’E/S relatives au disque et au réseau. Souvent, cela permet de réduire la réplication de données en double lorsque seule une petite partie de la valeur est mise à jour. La manière la plus courante pour y parvenir dans le Dictionnaire fiable consiste à diviser vos lignes en plusieurs lignes.
* Envisagez d’utiliser la fonctionnalité de sauvegarde et de restauration pour bénéficier de la récupération d’urgence.
* Évitez de combiner des opérations à une seule entité et des opérations à plusieurs entités (par exemple `GetCountAsync`, `CreateEnumerableAsync`) dans la même transaction en raison des différents niveaux d’isolement.
* Gérez l’exception InvalidOperationException. Les transactions des utilisateurs peuvent être annulées par le système pour diverses raisons. Par exemple, lorsque le Gestionnaire d’état fiable abandonne le rôle Principal ou qu’une transaction longue bloque la troncature du journal des transactions. Dans ce cas, l’utilisateur peut recevoir l’exception InvalidOperationException, indiquant que sa transaction a déjà été terminée. Dans l’hypothèse où l’arrêt de la transaction n’était pas demandé par l’utilisateur, la meilleure façon de gérer cette exception consiste à supprimer la transaction, vérifier si le jeton d’annulation a été signalé (ou si le rôle du réplica a été modifié) et, si ce n’est pas le cas, créer une nouvelle transaction, puis réessayer.  

Voici quelques points à retenir :

* Le délai d’expiration par défaut est de quatre secondes pour toutes les API de Collections fiables. La plupart des utilisateurs souhaiteront utiliser le délai d’attente par défaut.
* Le jeton d'annulation par défaut est `CancellationToken.None` dans toutes les API de Collections fiables.
* Le paramètre de type de clé (*TKey*) pour un Dictionnaire fiable doit implémenter correctement `GetHashCode()` et `Equals()`. Les clés doivent être immuables.
* Pour obtenir un haut niveau de disponibilité pour les Collections fiables, chaque service doit avoir au moins une taille de jeu de réplicas cible minimum égale à 3.
* Les opérations de lecture sur le secondaire peuvent lire des versions qui ne sont pas validées dans le quorum.
  Cela signifie qu’une version des données lue à partir d’un seul secondaire peut présenter une progression erronée.
  Les lectures à partir du principal sont toujours stables : la progression n’est jamais erronée.
* La sécurité et la confidentialité des données rendues persistantes par votre application dans une collection fiable dépendent de vous et font l’objet des protections fournies par votre gestion du stockage : par exemple, vous pouvez utiliser le chiffrement de disque de système d’exploitation pour protéger vos données au repos.
* L’énumération `ReliableDictionary` utilise une structure de données triée classée par clé. Pour rendre l’énumération efficace, des validations sont ajoutées à une table de hachage temporaire, puis déplacées vers le point de contrôle post de la structure de données triée principale. Les ajouts/mises à jour/suppressions ont le runtime de meilleur cas O(1) et du pire cas O(log n), dans le cas de contrôles de validation sur la présence de la clé. Les instructions get peuvent être O(1) ou O (log n) selon que vous lisez à partir d’une validation récente ou plus ancienne.

## <a name="volatile-reliable-collections"></a>Collections fiables volatiles
Lorsque vous décidez d'utiliser des collections fiables volatiles, tenez compte des éléments suivants :

* ```ReliableDictionary``` dispose d'une prise en charge volatile
* ```ReliableQueue``` dispose d'une prise en charge volatile
* ```ReliableConcurrentQueue``` NE dispose PAS d'une prise en charge volatile
* Les services persistants NE PEUVENT PAS devenir volatils. Pour remplacer l'indicateur ```HasPersistedState``` par ```false```, le service doit être entièrement recréé.
* Les services volatils NE PEUVENT PAS devenir persistants. Pour remplacer l'indicateur ```HasPersistedState``` par ```true```, le service doit être entièrement recréé.
* ```HasPersistedState``` est une configuration de niveau de service. Autrement dit, **TOUTES** les collections seront soit persistantes soit volatiles. Vous ne pouvez pas mélanger des collections volatiles et persistantes
* La perte de quorum d'une partition volatile entraîne une perte totale des données
* Les opérations de sauvegarde et de restauration ne sont PAS disponibles pour les services volatils

## <a name="next-steps"></a>Étapes suivantes
* [Utilisation des collections fiables](service-fabric-work-with-reliable-collections.md)
* [Transactions et verrous](service-fabric-reliable-services-reliable-collections-transactions-locks.md)
* Gestion des données
  * [Sauvegarde et restauration](service-fabric-reliable-services-backup-restore.md)
  * [Notifications](service-fabric-reliable-services-notifications.md)
  * [Sérialisation et mise à niveau](service-fabric-application-upgrade-data-serialization.md)
  * [Configuration du Gestionnaire d’état fiable](service-fabric-reliable-services-configuration.md)
* Autres
  * [Démarrage rapide de Reliable Services](service-fabric-reliable-services-quick-start.md)
  * [Référence du développeur pour les Collections fiables](/dotnet/api/microsoft.servicefabric.data.collections?view=azure-dotnet#microsoft_servicefabric_data_collections)

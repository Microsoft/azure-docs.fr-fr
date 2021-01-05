---
title: Forum aux questions sur le débit approvisionné en mode de mise à l’échelle automatique dans Azure Cosmos DB
description: Forum aux questions sur le débit provisionné en mode de mise à l’échelle automatique pour les bases de données et les conteneurs Azure Cosmos DB
author: deborahc
ms.author: dech
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 12/11/2020
ms.openlocfilehash: a740ad62dacc9a29cab1cc144f1789e125ec2e89
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97368577"
---
# <a name="frequently-asked-questions-about-autoscale-provisioned-throughput-in-azure-cosmos-db"></a>Forum aux questions sur le débit provisionné en mode de mise à l’échelle automatique dans Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Grâce au débit approvisionné par mise à l’échelle automatique, Azure Cosmos DB gère et met à l’échelle automatiquement l’unité de requête/s de votre base de données ou conteneur en fonction de l’utilisation. Cette page répond aux questions fréquentes à propos de la mise à l’échelle automatique.

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="what-is-the-difference-between-autopilot-and-autoscale-in-azure-cosmos-db"></a>Quelle est la différence entre l’« AutoPilot » et la « mise à l’échelle automatique » dans Azure Cosmos DB ?
« Mise à l’échelle automatique » ou « débit approvisionné par mise à l’échelle automatique » est le nom mis à jour pour la fonctionnalité, précédemment appelé « AutoPilot ». Avec la version actuelle de la mise à l’échelle automatique, nous avons ajouté de nouvelles fonctionnalités, notamment la possibilité de définir une unité de requête/s maximale personnalisée et un support programmatique. 

### <a name="what-happens-to-databases-or-containers-created-in-the-previous-autopilot-tier-model"></a>Qu’advient-il des bases de données ou des conteneurs créés dans le modèle de niveau AutoPilot précédent ?
Les ressources qui ont été créées avec le modèle de niveau précédent sont prises en charge automatiquement avec le nouveau modèle de RU/s personnalisée de mise à l’échelle automatique. La limite supérieure du niveau devient la nouvelle RU/s maximale, ce qui se traduit par la même plage d’échelle. 

Par exemple, si vous avez précédemment sélectionné le niveau de mise à l’échelle entre 400 et 4 000 RU/s, la base de données ou le conteneur s’affiche désormais avec un maximum de 4 000 RU/s, ce qui permet une mise à l’échelle de 400 à 4 000 RU/s. À partir de là, vous pouvez modifier la RU/s maximale en valeur personnalisée pour l’adapter à votre charge de travail. 

### <a name="how-quickly-will-autoscale-scale-up-based-on-spikes-in-traffic"></a>À quelle vitesse la mise à l’échelle automatique effectuera un scale-up en fonction des pics de trafic ?
Avec la mise à l’échelle automatique, le système met à l’échelle le débit (RU/s) `T` vers le haut ou vers le haut au sein du `0.1 * Tmax` et de la plage de `Tmax`, en fonction du trafic entrant. Étant donné que la mise à l’échelle est automatique et instantanée, à tout moment, vous pouvez consommer jusqu’à `Tmax` approvisionné sans retard. 

### <a name="how-do-i-determine-what-rus-the-system-is-currently-scaled-to"></a>Comment déterminer la RU/s sur laquelle le système est actuellement mis à l’échelle ?
Utilisez [Mesures Azure Monitor](how-to-choose-offer.md#measure-and-monitor-your-usage) pour analyser la RU/s maximal de mise à l’échelle automatique approvisionnée et le débit actuel (RU/s) sur lequel le système est mis à l’échelle. 

### <a name="what-is-the-pricing-for-autoscale"></a>Quels sont les tarifs de la mise à l’échelle automatique ?
Vous êtes facturé toutes les heures pour le débit le plus élevé `T` sur lequel le système est mis à l’échelle dans l’heure. Si votre ressource ne comporte aucune demande pendant l’heure ou n’est mise à l’échelle au-delà de `0.1 * Tmax`, vous serez facturé au minimum `0.1 * Tmax`. Pour plus d’informations, consultez la [page Tarification](https://azure.microsoft.com/pricing/details/cosmos-db/) d’Azure Cosmos DB. 

### <a name="how-does-autoscale-show-up-on-my-bill"></a>Comment la mise à l’échelle automatique s’affiche-t-elle sur ma facture ?
Dans les comptes de région d’écriture unique, le taux de mise à l’échelle automatique par 100 RU/s est de 1,5 x le taux de débit standard (manuel) approvisionné. Sur votre facture, vous verrez le compteur de débit standard approvisionné existant. La quantité de ce compteur est multipliée par 1,5. Par exemple, si la RU/s la plus élevée sur laquelle le système est mis à l’échelle en une heure est de 6 000 RU/s, vous êtes facturé 60 * 1,5 = 90 unités du compteur pour cette heure.

Dans les comptes couvrant plusieurs régions d’écritures, le taux de mise à l’échelle automatique par 100 RU/s est le même que le taux de débit standard (manuel) approvisionné couvrant plusieurs régions d’écriture. Sur votre facture, vous verrez le compteur d’écriture multirégion. Étant donné que les taux sont les mêmes, si vous utilisez la mise à l’échelle automatique, vous verrez la même quantité qu’avec le débit standard.

### <a name="does-autoscale-work-with-reserved-capacity"></a>La mise à l’échelle automatique fonctionne-t-elle avec la capacité de réserve ?
Oui. Lorsque vous achetez une capacité réservée pour des comptes couvrant des régions d’écriture unique, la remise de réservation pour ressources de mise à l’échelle automatique est appliquée à votre utilisation du compteur à un ratio de 1,5 * le [ratio de la région spécifique](../cost-management-billing/reservations/understand-cosmosdb-reservation-charges.md#reservation-discount-per-region). 

La capacité de réserve de régions d’écriture fonctionne de la même façon pour la mise à l’échelle automatique et le débit approvisionné standard (manuel). Consultez la rubrique [Capacité de réserve Azure Cosmos DB](cosmos-db-reserved-capacity.md)

### <a name="does-autoscale-work-with-free-tier"></a>La mise à l’échelle automatique fonctionne-t-elle avec le niveau gratuit ?
Oui. Dans le niveau gratuit, vous pouvez utiliser le débit de mise à l’échelle automatique sur un conteneur. Le support des bases de données de débit partagé de mise à l’échelle automatique avec une RU/s maximale personnalisée n’est pas encore disponible. Découvrez comment [fonctionne la facturation du niveau gratuit avec la mise à l’échelle automatique](understand-your-bill.md#billing-examples-with-free-tier-accounts).

### <a name="is-autoscale-supported-for-all-apis"></a>La mise à l’échelle automatique est-elle prise en charge pour toutes les API ?
Oui, la mise à l’échelle automatique est prise en charge pour toutes les API : Core (SQL), Gremlin, Table, Cassandra et API pour MongoDB.

### <a name="is-autoscale-supported-for-multi-region-write-accounts"></a>La mise à l’échelle automatique est-elle prise en charge pour les comptes d’écriture multirégion ?
Oui. L’unité de requête/s maximale est disponible dans chaque région ajoutée au compte Azure Cosmos DB. 

### <a name="how-do-i-enable-autoscale-on-new-databases-or-containers"></a>Comment faire pour activer la mise à l’échelle automatique sur de nouveaux conteneurs ou bases de données ?
Consultez cet article sur [l’activation de la mise à l’échelle automatique](how-to-provision-autoscale-throughput.md).

### <a name="can-i-enable-autoscale-on-an-existing-database-or-a-container"></a>Puis-je activer la mise à l’échelle automatique sur un conteneur ou une base de données existant ?
Oui. Vous pouvez basculer entre la mise à l’échelle automatique et le débit approvisionné standard (manuel) le cas échéant. Pour l’instant, pour toutes les API, vous pouvez uniquement utiliser le [Portail Azure](how-to-provision-autoscale-throughput.md#enable-autoscale-on-existing-database-or-container) pour effectuer ces opérations.

### <a name="how-does-the-migration-between-autoscale-and-standard-manual-provisioned-throughput-work"></a>Comment la migration entre la mise à l’échelle automatique et le débit approvisionné standard (manuel) fonctionne-t-elle ?
Conceptuellement, la modification du type de débit est un processus en deux étapes. Tout d’abord, vous envoyez une requête de modification des paramètres de débit pour utiliser la mise à l’échelle automatique ou le débit approvisionné manuellement. Dans les deux cas, le système détermine et définit automatiquement une valeur RU/s initiale, en fonction des paramètres de débit et du stockage actuels. Au cours de cette étape, aucune valeur RU/s fournie par l’utilisateur n’est acceptée. Ensuite, une fois la mise à jour terminée, vous pouvez [modifier la RU/s](#can-i-change-the-max-rus-on-the-database-or-container) pour l’adapter à votre charge de travail. 

**Migration du débit approvisionné standard (manuel) vers la mise à l’échelle automatique**

Pour un conteneur, utilisez la formule suivante qui vise à estimer la RU/s maximale de mise à l’échelle automatique : ``MAX(4000, current manual provisioned RU/s, maximum RU/s ever provisioned / 10, storage in GB * 100)``, arrondie aux 1 000 RU/s les plus proches. La RU/s maximale de mise à l’échelle automatique initiale réel peut varier en fonction de la configuration de votre compte.

Exemple 1 : Supposons que vous disposez d’un conteneur avec un débit approvisionné manuel de 10 000 RU/s et un stockage de 25 Go. Une fois la mise à l’échelle automatique activée, l’unité de requête/s maximale de mise à l’échelle automatique initiale est la suivante : 10 000 RU/s, qui seront mises à l’échelle entre 1 000 et10 000 RU/s. 

Exemple 2 : Supposons que vous disposez d’un conteneur avec un débit approvisionné manuel de 50 000 RU/s et un stockage de 2 500 Go. Une fois la mise à l’échelle automatique activée, l’unité de requête/s maximale de mise à l’échelle automatique initiale est la suivante : 250 000 RU/s, qui seront mises à l’échelle entre 25 000 et 250 000 RU/s. 

**Migration de la mise à l’échelle automatique vers le débit approvisionné standard (manuel)**

Le débit manuel approvisionné initial sera égal à l’unité de requête/s maximale de la mise à l’échelle automatique actuelle. 

Exemple : Supposons que vous disposez d’une base de données ou d’un conteneur de mise à l’échelle automatique avec une unité de requête/s maximale de 20 000 RU/s (mise à l’échelle entre 2 000 et 20 000 RU/s). Lors de ma mise à jour pour utiliser le débit approvisionné manuel, le débit initial est de 20 000 RU/s. 

### <a name="is-there-azure-cli-or-powershell-support-to-manage-databases-or-containers-with-autoscale"></a>Existe-t-il un support Azure CLI ou PowerShell pour gérer les bases de données ou les conteneurs avec la mise à l’échelle automatique ?
Actuellement, vous pouvez uniquement créer et gérer des ressources à l’aide de la mise à l’échelle automatique à partir du Portail Azure, du Kit de développement logiciel (SDK) .NET v3, du Kit de développement logiciel (SDK) Java v4 et d’Azure Resource Manager. La prise en charge dans Azure CLI, PowerShell et d’autres kits de développement logiciel (SDK) n’est pas encore disponible.

### <a name="is-autoscale-supported-for-shared-throughput-databases"></a>La mise à l’échelle automatique est-elle prise en charge pour les bases de données à débit partagé ?
Oui, la mise à l’échelle automatique est prise en charge pour les bases de données à débit partagé. Pour activer cette fonctionnalité, sélectionnez la mise à l’échelle automatique et l’option **Approvisionner le débit** lors de la création de la base de données. 

### <a name="what-is-the-number-of-allowed-containers-per-shared-throughput-database-when-autoscale-is-enabled"></a>Quel est le nombre de conteneurs autorisé par base de données à débit partagé quand la mise à l’échelle automatique est activée ?
Azure Cosmos DB applique un maximum de 25 conteneurs dans une base de données à débit partagé, qui s’applique aux bases de données avec mise à l’échelle automatique ou débit standard (manuel). 

### <a name="what-is-the-impact-of-autoscale-on-database-consistency-level"></a>Quel est l’impact de la mise à l’échelle automatique au niveau de la cohérence des bases de données ?
La mise à l’échelle automatique n’a aucun impact sur la cohérence des bases de données.
Pour plus d’informations sur les niveaux de cohérence disponibles, consultez l’article [Niveaux de cohérence](consistency-levels.md).

### <a name="what-is-the-storage-limit-associated-with-each-max-rus-option"></a>Quelle est la limite de stockage associée à chaque option RU/s max ?  
La limite de stockage en Go pour chaque RU/s max est de : RU/s max de base de données ou conteneur / 100. Par exemple, si le nombre maximal de RU/s est de 20 000, la ressource peut prendre en charge 200 Go de stockage. Pour connaître le nombre maximal de RU/s disponibles et les options de stockage, consultez l’article sur les [limites de la mise à l’échelle automatique](provision-throughput-autoscale.md#autoscale-limits). 

### <a name="what-happens-if-i-exceed-the-storage-limit-associated-with-my-max-throughput"></a>Que se passe-t-il si je dépasse la limite de stockage associée à mon débit maximal ?
Si la limite de stockage associée au débit maximal de la base de données ou du conteneur est dépassée, Azure Cosmos DB augmente automatiquement le débit maximal à l’unité de requête/s suivante la plus élevée pouvant prendre en charge ce niveau de stockage.

Par exemple, si vous démarrez avec une RU/s maximale de 50 000 RU/s (avec une mise à l’échelle de 5 000 à 50 000 RU/s), vous pouvez stocker jusqu’à 500 Go de données. Si vous dépassez 500 Go, par exemple, si le stockage est à présent de 600 Go, la nouvelle unité de requête/s maximale sera de 60 000 RU/s (les mises à l’échelle entre 6 000 et 60 000 RU/s).

### <a name="can-i-change-the-max-rus-on-the-database-or-container"></a>Puis-je modifier l’unité de requête/s maximale sur la base de données ou le conteneur ? 
Oui. Consultez cet [article](how-to-provision-autoscale-throughput.md) sur la façon de modifier la RU/s maximale. Lorsque vous modifiez l’unité de requête/s maximale, en fonction de la valeur demandée, il peut s’agir d’une opération asynchrone qui peut prendre un certain temps (pouvant atteindre jusqu’à 4 à 6 heures, en fonction de l’unité de requête/s sélectionnée)

#### <a name="increasing-the-max-rus"></a>Amélioration de l’unité de requête/s maximale
Lorsque vous envoyez une requête pour augmenter l’unité de requête/s maximale `Tmax`, en fonction de la RU/s sélectionnée, le service configure plus de ressources pour prendre en charge l’unité de requête/s maximale la plus élevée. Bien que cela se produise, vos charges de travail et opérations existantes ne seront pas affectées. Le système continue à mettre à l’échelle votre base de données ou votre conteneur entre `0.1*Tmax` et `Tmax` précédent, jusqu’à ce que la nouvelle plage de mise à l’échelle de `0.1*Tmax_new` à `Tmax_new` soit prête.

#### <a name="lowering-the-max-rus"></a>Réduction de l’unité de requête/s maximale
Lorsque vous réduisez l’unité de requête/s maximale, la valeur minimale que vous pouvez lui affecter est la suivante : `MAX(4000, highest max RU/s ever provisioned / 10, current storage in GB * 100)`, arrondie aux 1 000 RU/s les plus proches. 

Exemple 1 : Supposons que vous disposez d’un conteneur de mise à l’échelle automatique avec une unité de requête/s maximale de 20 000 RU/s (mise à l’échelle entre 2 000 et 20 000 RU/s) et d’un stockage de 50 Go. La valeur minimale la plus basse que vous pouvez définir pour la RU/s est : MAX (4 000, 20 000/10, **50 * 100**) = 5 000 RU/s (mise à l’échelle entre 500 et 5 000 RU/s).

Exemple 2 : Supposons que vous disposez d’un conteneur de mise à l’échelle automatique avec une unité de requête/s maximale de 100 000 RU/s et d’un stockage de 100 Go. Vous pouvez à présente mettre à l’échelle l’unité de requête/s maximale jusqu’à 150 000 RU/s (mise à l’échelle entre 15 000 et 150 000 RU/s). La valeur minimale la plus basse que vous pouvez à présent définir pour la RU/s est : MAX (4 000, **150 000/10**, 100 * 100) = 15 000 RU/s (mise à l’échelle entre 1 500 et 15 000 RU/s). 

Pour une base de données à débit partagé, lorsque vous réduisez l’unité de requête/s maximale, la valeur minimale que vous pouvez lui affecter est la suivante : `MAX(4000, highest max RU/s ever provisioned / 10, current storage in GB * 100,  4000 + (MAX(Container count - 25, 0) * 1000))`, arrondie aux 1 000 RU/s les plus proches.  

Les formules et exemples ci-dessus se rapportent à l’unité de requête/s maximale que vous pouvez définir pour la mise à l’échelle automatique minimale et sont différentes de la plage de `0.1 * Tmax` à `Tmax` que le système met automatiquement à l’échelle. Quel que soit la RU/s maximale, le système est toujours mis à l’échelle entre `0.1 * Tmax` et `Tmax`. 

### <a name="how-does-ttl-work-with-autoscale"></a>Comment fonctionne la durée de vie avec la mise à l’échelle automatique ?
Avec la mise à l’échelle automatique, les opérations de TTL n’affectent pas la mise à l’échelle de la RU/s. Toutes les unités de requête consommées en raison de la durée de vie ne font pas partie de la RU/s facturée pour le conteneur de mise à l’échelle automatique. 

Supposons, par exemple, que vous disposez d’un conteneur de mise à l’échelle automatique avec 400 à 4 000 RU/s :
- Heure 1 : T = 0 : Le conteneur n’a aucune utilisation (aucune durée de vie ni requête de charge de travail). L’unité de requête/s facturable est de 400 RU/s.
- Heure 1 : T = 1 : La durée de vie est activée.
- Heure 1 : T = 2 : Le conteneur commence à obtenir des requêtes, qui consomment 1 000 RU en 1 seconde. 200 unités de requête de valeur de durée de vie doivent également se produire. L’unité de requête/s facturable est toujours de 1 000 RU/s. Quel que soit le moment où les TTL se produisent, elles n’affectent pas la logique de mise à l’échelle de la mise à l'échelle automatique.

### <a name="what-is-the-mapping-between-the-max-rus-and-physical-partitions"></a>Quel est le mappage entre le nombre maximal de RU/s et les partitions physiques ?
Quand vous sélectionnez initialement le nombre maximal de RU/s, Azure Cosmos DB provisionne : RU/s max / 10 000 RU/s = nombre de partitions physiques. Chaque [partition physique](partitioning-overview.md#physical-partitions) peut prendre en charge jusqu’à 10 000 RU/s et 50 Go de stockage. À mesure que la taille de stockage augmente, Azure Cosmos DB fractionne automatiquement les partitions pour ajouter des partitions physiques supplémentaires afin de gérer l’augmentation du stockage ou il augmente le niveau de RU/s maximale si le stockage [dépasse la limite associée](#what-is-the-storage-limit-associated-with-each-max-rus-option). 

La RU/s maximale de la base de données ou du conteneur est répartie uniformément parmi toutes les partitions physiques. Ainsi, le débit total auquel une partition physique peut être mise à l’échelle est : RU/s max de base de données ou conteneur / nombre de partitions physiques. 

### <a name="what-happens-if-incoming-requests-exceed-the-max-rus-of-the-database-or-container"></a>Que se passe-t-il si les requêtes entrantes dépassent la valeur RU/s max de la base de données ou du conteneur ?
Si le nombre total de RU/s consommées dépasse le nombre maximal de RU/s de la base de données ou du conteneur, les requêtes qui dépassent le nombre maximal de RU/s sont limitées et retournent un code d’état 429. Les requêtes qui entraînent plus de 100 % d’utilisation normalisée sont également limitées. L’utilisation normalisée est définie comme la quantité maximale d’utilisation de RU/s parmi toutes les partitions physiques. Par exemple, supposez que votre débit maximal est de 20 000 RU/s et que vous disposez de deux partitions physiques, P_1 et P_2, chacune autorisant une mise à l’échelle à 10 000 RU/s. Durant une seconde donnée, si P_1 a utilisé 6 000 RU/s et P_2 8 000 RU/s, l’utilisation normalisée est MAX(6 000 RU / 10 000 RU, 8 000 RU / 10 000 RU) = 0,8.

> [!NOTE]
> Les kits SDK du client Azure Cosmos DB et les outils d’importation de données (Azure Data Factory, bibliothèque d’exécuteur en bloc) réessayent automatiquement en cas d’erreur 429. Des erreurs 429 occasionnelles ne posent donc pas de problème. Un nombre élevé et soutenu d’erreurs 429 peut indiquer que vous devez augmenter le nombre maximal de RU/s ou réexaminer votre stratégie de partitionnement pour une [partition à chaud](#autoscale-rate-limiting).

### <a name="is-it-still-possible-to-see-429s-throttlingrate-limiting-when-autoscale-is-enabled"></a><a id="autoscale-rate-limiting"></a> Est-il toujours possible d’observer des erreurs 429 (limitation du débit) quand la mise à l’échelle automatique est activée ? 
Oui. Il est possible d’observer des erreurs 429 dans deux scénarios. Tout d’abord, quand le nombre total de RU/s consommées dépasse le nombre maximal de RU/s de la base de données ou du conteneur, le service limite les requêtes en conséquence. 

Ensuite, s’il y a une partition à chaud, c’est-à-dire une valeur de clé de partition logique qui présente un nombre de requêtes beaucoup plus élevé que d’autres valeurs de clé de partition, il est possible que la partition physique sous-jacente dépasse son budget de RU/s. En guise de bonne pratique, pour éviter les partitions à chaud, [choisissez une clé de partition appropriée](partitioning-overview.md#choose-partitionkey) qui entraîne une répartition égale du stockage et du débit. 

Par exemple, si vous sélectionnez l’option de débit maximal de 20 000 RU/s et que vous avez 200 Go de stockage avec quatre partitions physiques, chaque partition physique peut être mise à l’échelle automatiquement jusqu’à 5 000 RU/s. S’il existe une partition à chaud sur une clé de partition logique spécifique, vous verrez des codes d’état 429 quand la partition physique sous-jacente dans laquelle elle se trouve dépasse 5 000 RU/s, c’est-à-dire 100 % de l’utilisation normalisée.


## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [activer la mise à l’échelle automatique sur une base de données ou un conteneur Azure Cosmos DB](how-to-provision-autoscale-throughput.md).
* En savoir plus sur les [avantages du débit approvisionné avec mise à l’échelle automatique](provision-throughput-autoscale.md#benefits-of-autoscale).
* Apprenez-en davantage sur les [partitions logiques et physiques](partitioning-overview.md).
                        

---
title: Réplicas en lecture - Azure Database pour PostgreSQL - Serveur unique
description: Cet article décrit la fonctionnalité de réplica en lecture dans Azure Database pour PostgreSQL, avec un serveur unique.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: conceptual
ms.date: 11/05/2020
ms.openlocfilehash: 8fabf8169270c3162604b6535a6cf2fb07cd9a9d
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93422142"
---
# <a name="read-replicas-in-azure-database-for-postgresql---single-server"></a>Réplicas en lecture dans Azure Database pour PostgreSQL - Serveur unique

La fonctionnalité de réplica en lecture vous permet de répliquer les données d’un serveur Azure Database pour PostgreSQL sur un serveur en lecture seule. Les réplicas sont mis à jour **de manière asynchrone** à l’aide de la technologie de réplication physique native du moteur PostgreSQL. Vous pouvez effectuer la réplication à partir du serveur principal vers cinq réplicas au maximum.

Les réplicas sont de nouveaux serveurs que vous gérez de manière similaire aux serveurs Azure Database pour PostgreSQL classiques. Pour chaque réplica en lecture, vous êtes facturé en fonction de la capacité de calcul provisionnée dans les vCores et du stockage provisionné en Go/mois.

Découvrez comment [créer et gérer des réplicas](howto-read-replicas-portal.md).

## <a name="when-to-use-a-read-replica"></a>Quand utiliser un réplica en lecture
La fonctionnalité de réplica en lecture contribue à améliorer les performances et la scalabilité des charges de travail nécessitant une lecture intensive. Les charges de travail de lecture peuvent être isolées sur les réplicas, tandis que les charges de travail d’écriture peuvent être dirigées vers le serveur principal. Les réplicas en lecture peuvent également être déployés sur une autre région et peuvent être promus en tant que serveurs en lecture/écriture en cas de récupération d’urgence.

Un scénario courant est d’avoir les charges de travail décisionnelles et analytiques qui utilisent le réplica en lecture comme source de données pour la création de rapports.

Dans la mesure où les réplicas sont en lecture seule, ils ne réduisent pas directement les charges relatives à la capacité d’écriture sur le serveur principal.

### <a name="considerations"></a>Considérations
Cette fonctionnalité est destinée aux scénarios où le décalage est acceptable et vise à décharger les requêtes. Elle n’est pas destinée aux scénarios de réplication synchrone où les données répliquées sont supposées être à jour. Il y aura un délai mesurable entre le serveur principal et le réplica. Il peut être de quelques minutes, voire de quelques heures, selon la charge de travail et le temps de latence entre la source et le réplica. Les données du réplica finissent par devenir cohérentes avec les données du serveur principal. Utilisez cette fonctionnalité pour les charges de travail pouvant s’adapter à ce délai. 

> [!NOTE]
> Pour la plupart des charges de travail, les réplicas en lecture offrent des mises à jour en quasi-temps réel à partir du serveur principal. Toutefois, avec des charges de travail principales gourmandes en écriture, intensives et persistantes, le retard de réplication peut continuer à s’allonger et ne jamais pouvoir se réaligner sur le principal. Cela peut également augmenter l’utilisation du stockage sur le serveur principal, car les fichiers WAL ne sont pas supprimés tant qu’ils ne sont pas reçus au niveau du réplica. Si cette situation persiste, la suppression et la recréation du réplica en lecture après la fin des charges de travail gourmandes en écriture est l’option qui permet de ramener le réplica à un bon état en ce qui concerne le décalage.
> Les réplicas en lecture asynchrone ne conviennent pas à de telles charges de travail d’écriture intensives. Lors de l’évaluation des réplicas en lecture pour votre application, surveillez le décalage sur le réplica pendant un cycle complet de charge de travail de l’application à travers ses heures de pointe et ses heures creuses afin d’accéder au décalage possible et au RTO/RPO attendu à différents points du cycle de charge de travail.
> 
## <a name="cross-region-replication"></a>Réplication entre régions
Vous pouvez créer un réplica en lecture dans une autre région à partir de votre serveur principal. La réplication entre régions peut être utile pour des scénarios tels que la planification de la récupération d’urgence ou le rapprochement des données de vos utilisateurs.

>[!NOTE]
> Les serveurs du niveau De base prennent uniquement en charge la réplication dans la même région.

Vous pouvez disposer d’un serveur principal dans toute [région Azure Database pour PostgreSQL](https://azure.microsoft.com/global-infrastructure/services/?products=postgresql). Un serveur principal peut avoir un réplica dans sa région jumelée ou dans les régions de réplica universelles. L’image ci-dessous montre les régions de réplica disponibles en fonction de votre région principale.

[ :::image type="content" source="media/concepts-read-replica/read-replica-regions.png" alt-text="Régions des réplicas en lecture":::](media/concepts-read-replica/read-replica-regions.png#lightbox)

### <a name="universal-replica-regions"></a>Régions de réplica universelles
Vous pouvez toujours créer un réplica en lecture dans les régions suivantes, quel que soit l’emplacement de votre serveur principal. Les régions de réplica universelles sont les suivantes :

Australie Est, Australie Sud-Est, Brésil Sud, Canada Centre, Canada Est, USA Centre, Asie Est, USA Est, USA Est 2, Japon Est, Japon Ouest, Corée Centre, Corée Sud, USA Centre Nord, Europe Nord, USA Centre Sud, Asie Sud-Est, Royaume-Uni Sud, Royaume-Uni Ouest, Europe Ouest, USA Ouest, USA Ouest 2, USA Centre-Ouest.

### <a name="paired-regions"></a>Régions jumelées
Outre les régions de réplica universelles, vous pouvez créer un réplica en lecture dans la région Azure jumelée de votre serveur principal. Si vous ne connaissez pas la région jumelée de votre région, vous trouverez plus d’informations dans l’[article sur les régions jumelées Azure](../best-practices-availability-paired-regions.md).

Si vous utilisez des réplicas entre régions pour la planification de la récupération d’urgence, nous vous recommandons de créer le réplica dans la région jumelée plutôt que dans une autre région. Les régions jumelées évitent les mises à jour simultanées et hiérarchisent l’isolement physique et la résidence des données.  

Il existe quelques limitations à prendre en compte : 

* Disponibilité régionale : Azure Database pour PostgreSQL est disponible dans les régions France Centre, Émirats arabes unis Nord et Allemagne Centre. Toutefois, leurs régions jumelées ne sont pas disponibles.
    
* Paires unidirectionnelles : Certaines régions Azure sont jumelées dans une seule direction. Ces régions incluent Inde Ouest, et Brésil Sud. 
   Cela signifie qu’un serveur principal dans la région Inde Ouest peut créer un réplica dans la région Inde Sud. Toutefois, un serveur principal dans la région Inde Sud ne peut pas créer de réplica dans la région Inde Ouest. En effet, la région secondaire de la région Inde Ouest est Inde Sud, mais la région secondaire de la région Inde Sud n’est pas Inde Ouest.


## <a name="create-a-replica"></a>Créer un réplica
Quand vous démarrez le workflow de création de réplica, un serveur Azure Database pour PostgreSQL vide est créé. Le nouveau serveur est rempli avec les données qui se trouvaient sur le serveur principal. Le temps de création dépend de la quantité de données présentes sur le serveur principal et du temps écoulé depuis la dernière sauvegarde complète hebdomadaire. Le temps nécessaire peut aller de quelques minutes à plusieurs heures.

Chaque réplica est activé pour la [croissance automatique](concepts-pricing-tiers.md#storage-auto-grow) du stockage. La fonctionnalité de croissance automatique permet au réplica de suivre les données qui sont répliquées sur lui et d’empêcher toute interruption à cause d’une erreur de saturation du stockage.

La fonctionnalité de réplica en lecture utilise la réplication physique PostgreSQL, et non la réplication logique. Le streaming de réplication à l’aide des slots de réplication est le mode de fonctionnement par défaut. Quand cela est nécessaire, la copie des journaux de transaction permet de rattraper le retard.

Découvrez comment [créer un réplica en lecture dans le portail Azure](howto-read-replicas-portal.md).

## <a name="connect-to-a-replica"></a>Se connecter à un réplica
Lorsque vous créez un réplica, il n’hérite pas des règles de pare-feu ni du point de terminaison de service VNet (réseau virtuel) du serveur principal. Ces règles doivent être configurés indépendamment pour le réplica.

Le réplica hérite du compte Administrateur du serveur principal. Tous les comptes d’utilisateur sur le serveur principal sont répliqués sur les réplicas en lecture. Vous pouvez uniquement vous connecter à un réplica en lecture à l’aide des comptes d’utilisateur disponibles sur le serveur principal.

Vous pouvez vous connecter au réplica à l’aide de son nom d’hôte et d’un compte d’utilisateur valide, comme vous le faites sur un serveur Azure Database pour PostgreSQL classique. Pour un serveur nommé **myreplica**, à l’aide du nom d’utilisateur administrateur **myadmin**, vous pouvez vous connecter au réplica via psql :

```bash
psql -h myreplica.postgres.database.azure.com -U myadmin@myreplica -d postgres
```

À l’invite, entrez le mot de passe du compte d’utilisateur.

## <a name="monitor-replication"></a>Superviser la réplication
Azure Database pour PostgreSQL fournit deux métriques de surveillance de la réplication, à savoir **Retard maximum entre réplicas** et **Retard du réplica**. Pour savoir comment afficher ces métriques, consultez la section **Superviser un réplica** section [Créer et gérer des réplicas en lecture dans Azure Database pour PostgreSQL - serveur unique à partir du Portail Azure](howto-read-replicas-portal.md).

La métrique **Retard maximum entre réplicas** représente le retard entre le serveur principal et le réplica le plus en retard, en octets. Cette métrique est applicable et disponible uniquement sur le serveur principal et sera disponible uniquement si au moins l’un des réplicas en lecture est connecté au serveur principal et que le serveur principal est en mode de réplication en continu. Les informations relatives au décalage n’indiquent pas les détails lorsque le réplica est en train de rattraper le réplica principal à l’aide des journaux archivés du serveur principal en mode de réplication d’envoi de fichiers.

La métrique **Retard du réplica** indique le temps écoulé depuis la dernière transaction réexécutée. S’il n’existe aucune transaction sur votre serveur principal, la métrique reflète ce retard. Cette métrique est applicable et disponible pour les serveurs réplicas uniquement. Le retard du réplica est calculé à partir de la vue `pg_stat_wal_receiver` :

```SQL
SELECT EXTRACT (EPOCH FROM now() - pg_last_xact_replay_timestamp());
```

Définissez une alerte qui vous informe quand le retard du réplica atteint une valeur inacceptable pour votre charge de travail. 

Pour plus d’informations, interrogez directement le serveur principal pour connaître le délai de réplication en octets sur tous les réplicas.

> [!NOTE]
> Si un serveur principal ou un réplica en lecture redémarre, le temps nécessaire pour redémarrer et rattraper le temps perdu est reflété par la métrique Retard du réplica.

## <a name="stop-replication--promote-replica"></a>Arrêter la réplication/promouvoir le réplica
Vous pouvez arrêter la réplication entre un serveur principal et un réplica à tout moment. L’action d’arrêt provoque le redémarrage du réplica et promeut le réplica en tant que serveur accessible en lecture indépendant et autonome. Les données du serveur autonome sont les données disponibles sur le serveur réplica au moment où la réplica est arrêtée. Toutes les mises à jour ultérieures sur le serveur principal ne sont pas propagées sur le réplica. Toutefois, le serveur réplica peut avoir accumulé des journaux qui ne sont pas encore appliqués. Dans le cadre du processus de redémarrage, le réplica applique tous les journaux en attente avant d’accepter les connexions clientes.  

### <a name="considerations"></a>Considérations
- Avant d’arrêter la réplication sur un réplica en lecture, vérifiez le retard de la réplication pour vous assurer que le réplica contient toutes les données nécessaires. 
- Comme le réplica en lecture doit appliquer tous les journaux en attente avant de pouvoir devenir un serveur autonome, le RTO peut être plus élevé pour les charges de travail volumineuses en écriture lorsque l’arrêt de la réplication se produit, car le réplica peut avoir un retard important. Veuillez y prêter attention lorsque vous envisagez de promouvoir un réplica.
- Le serveur réplica promu ne peut pas être à nouveau transformé en réplica.
- Si vous promouvez un réplica en serveur principal, vous ne pouvez pas réactiver la réplication vers l’ancien serveur principal. Si vous souhaitez revenir à l’ancienne région primaire, vous pouvez soit établir un nouveau serveur réplica avec un nouveau nom (ou) supprimer l’ancien serveur principal et créer un réplica en utilisant le nom de l’ancien serveur principal.
- Si vous avez plusieurs réplicas en lecture et que vous promouvez l’un d’eux en serveur principal, les autres serveurs réplicas sont toujours connectés à l’ancien serveur principal. Vous devrez peut-être recréer les réplicas à partir du nouveau serveur promu.

Lorsque vous arrêtez la réplication, le réplica perd tous les liens avec son serveur principal précédent et d’autres réplicas.

Découvrez comment [arrêter la réplication sur un réplica](howto-read-replicas-portal.md).

## <a name="failover-to-replica"></a>Basculement vers le réplica

En cas de défaillance d’un serveur principal, il n’est **pas** automatiquement basculé sur le réplica en lecture. 

Étant donné que la réplication est asynchrone, il peut y avoir un décalage considérable entre le serveur principal et le réplica. Le niveau de décalage est influencé par un certain nombre de facteurs tels que le type de charge de travail exécutée sur le serveur principal et la latence entre le serveur principal et le serveur réplica. Dans les cas typiques de charge de travail d’écriture nominale, on s’attend à un retard du réplica de quelques secondes à quelques minutes. Toutefois, dans les cas où le serveur principal exécute des charges de travail gourmandes en écriture et volumineuses et que le réplica n’est pas suffisamment rapide, le décalage peut être beaucoup plus important. Pour suivre le retard de chaque réplica, consultez la métrique *Retard du réplica*. La métrique indique le temps écoulé depuis la dernière transaction réexécutée au niveau du réplica. Il est recommandé d’observer le réplica sur une période donnée afin de déterminer le décalage moyen. Vous pouvez configurer une alerte afin d’être averti lorsque le retard d’un réplica dépasse la plage définie et de prendre les mesures nécessaires.

> [!Tip]
> Si vous basculez vers le réplica, le décalage qui existe au moment où vous supprimez la liaison entre le réplica et le serveur principal indiquera la quantité de données perdues.

Une fois que vous avez décidé de basculer vers un réplica : 

1. Arrêtez la réplication vers le réplica<br/>
   Cette étape est nécessaire pour que le serveur réplica devienne un serveur autonome et puisse accepter les écritures. Dans le cadre de ce processus, le serveur réplica est redémarré, puis il est dissocié du serveur principal. Une fois que vous initiez l’arrêt de la réplication, le processus principal prend généralement quelques minutes pour appliquer tous les journaux résiduels qui n’ont pas encore été appliqués et pour ouvrir la base de données en tant que serveur accessible en lecture et en écriture. Pour comprendre les implications d’une telle action, consultez la section [Arrêter la réplication](#stop-replication--promote-replica).
    
2. Faites pointer votre application vers l’ancien réplica<br/>
   Chaque serveur est associé à une chaîne de connexion unique. Mettez à jour la chaîne de connexion de votre application pour qu’elle pointe vers l’ancien réplica au lieu du serveur principal.
    
Lorsque votre application est en mesure de traiter les lectures et les écritures, le basculement est terminé. Le temps d’arrêt de votre application dépend du moment où vous détectez le problème et où vous effectuez les étapes 1 et 2 ci-dessus.

### <a name="disaster-recovery"></a>Récupération d'urgence

En cas de sinistre majeur, tel que des défaillances au niveau de la zone de disponibilité ou des zones régionales, vous pouvez effectuer une opération de récupération d’urgence en promouvant votre réplica en lecture. À partir du portail de l’interface utilisateur, vous pouvez accéder au serveur réplica en lecture. Cliquez ensuite sur l’onglet de réplication. Vous pouvez alors arrêter le réplica pour le promouvoir en tant que serveur indépendant. Vous pouvez également utiliser [Azure CLI](/cli/azure/postgres/server/replica#az_postgres_server_replica_stop) pour arrêter et promouvoir le serveur réplica.

## <a name="considerations"></a>Considérations

Cette section résume les considérations relatives à la fonctionnalité de réplica en lecture.

### <a name="prerequisites"></a>Prérequis
Les réplicas en lecture et le [décodage logique](concepts-logical.md) dépendent du journal WAL de Postgres pour obtenir des informations. Ces deux fonctionnalités ont besoin de différents niveaux de journalisation à partir de Postgres. Le décodage logique nécessite un niveau de journalisation plus élevé que les réplicas en lecture.

Pour configurer le niveau de journalisation approprié, utilisez le paramètre de prise en charge de la réplication Azure. La prise en charge de la réplication Azure propose trois options de paramétrage :

* **Désactivé** : place le moins d’informations dans le journal WAL. Ce paramètre n’est pas disponible sur la plupart des serveurs Azure Database pour PostgreSQL.  
* **Réplica** : plus de détails que l’option **Désactivé**. Il s’agit du niveau minimal de journalisation nécessaire pour que les [réplicas en lecture](concepts-read-replicas.md) fonctionnent. Il s’agit du paramètre par défaut sur la plupart des serveurs.
* **Logique** : plus de détails que l’option **Réplica**. Il s’agit du niveau minimal de journalisation pour que le décodage logique fonctionne. Les réplicas en lecture fonctionnent également avec ce paramètre.


### <a name="new-replicas"></a>Nouveaux réplicas
Un réplica en lecture est créé en tant que serveur Azure Database pour PostgreSQL. Un serveur existant ne peut pas être transformé en réplica. Vous ne pouvez pas créer un réplica d’un autre réplica en lecture.

### <a name="replica-configuration"></a>Configuration du réplica
Le réplica doit être créé en utilisant les mêmes paramètres de calcul et de stockage que le serveur principal. Après la création d’un réplica, plusieurs paramètres sont modifiables, y compris la période de rétention du stockage et de la sauvegarde.

Les règles de pare-feu, les règles de réseau virtuel et les paramètres de paramétrage sont hérités du serveur principal au réplica lorsque le réplica est créé ou après sa création.

### <a name="scaling"></a>Mise à l'échelle
Mise à l’échelle de vCores ou entre Usage général et À mémoire optimisée :
* PostgreSQL exige que le paramètre `max_connections` sur un serveur secondaire soit [supérieur ou égal au paramètre sur le serveur principal](https://www.postgresql.org/docs/current/hot-standby.html). Sinon, le serveur secondaire ne démarrera pas.
* Dans Azure Database pour PostgreSQL, le nombre maximal de connexions autorisées pour chaque serveur est fixé à la référence SKU de calcul, car les connexions occupent de la mémoire. Vous pouvez en savoir plus sur le [mappage entre max_connections et les références SKU de calcul](concepts-limits.md).
* **Scale-up** : Effectuez tout d’abord un scale-up du calcul d’un réplica, puis du serveur principal. Cet ordre empêchera des erreurs de violer la spécification de `max_connections`.
* **Scale-down** : Effectuez tout d’abord un scale-down du calcul du serveur principal, puis du réplica. Si vous essayez de mettre le réplica à une échelle inférieure à celle du serveur principal, une erreur se produira, car cela ne respecte pas la spécification de `max_connections`.

Mise à l’échelle du stockage :
* La croissance automatique du stockage est activée pour tous les réplicas afin d’éviter les problèmes de réplication dus à un réplica plein. Ce paramètre ne peut pas être désactivé.
* Vous pouvez également mettre à l’échelle manuellement le stockage, comme vous le feriez sur n’importe quel autre serveur.


### <a name="basic-tier"></a>Niveau de base
Les serveurs du niveau De base prennent uniquement en charge la réplication dans la même région.

### <a name="max_prepared_transactions"></a>max_prepared_transactions
Avec [PostgreSQL](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-MAX-PREPARED-TRANSACTIONS), la valeur du paramètre `max_prepared_transactions` sur le réplica en lecture doit être supérieure ou égale à la valeur du serveur principal. Sinon, le réplica ne démarre pas. Si vous souhaitez modifier `max_prepared_transactions` sur le serveur principal, modifiez-le d’abord sur les réplicas.

### <a name="stopped-replicas"></a>Réplicas arrêtés
Si vous arrêtez la réplication entre un serveur principal et un réplica en lecture, le réplica redémarre pour permettre l’application du changement. Le réplica arrêté devient un serveur autonome qui accepte les lectures et les écritures. Le serveur autonome ne peut pas être retransformé en réplica.

### <a name="deleted-primary-and-standalone-servers"></a>Serveurs principaux et autonomes supprimés
Lorsqu’un serveur principal est supprimé, tous ses réplicas en lecture deviennent des serveurs autonomes. Les réplicas redémarrent pour permettre l’application de ce changement.

## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment [créer et gérer des réplicas en lecture dans le portail Azure](howto-read-replicas-portal.md).
* Découvrez comment [créer et gérer des réplicas en lecture avec l’interface de ligne de commande Azure et l’API REST](howto-read-replicas-cli.md).
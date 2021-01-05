---
title: Réplicas en lecture - Azure Database pour MySQL - Serveur flexible
description: 'Découvrez en quoi consistent les réplicas en lecture dans Azure Database pour serveur flexible MySQL : création de réplicas, connexion à des réplicas, supervision de la réplication et arrêt de la réplication.'
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.topic: conceptual
ms.date: 10/26/2020
ms.openlocfilehash: 3fe63deb8115c0043023301c6d0dc3731e97743f
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96492623"
---
# <a name="read-replicas-in-azure-database-for-mysql---flexible-server"></a>Réplicas en lecture dans Azure Database pour MySQL - Serveur flexible

> [!IMPORTANT]
> La fonctionnalité Réplicas en lecture dans Azure Database pour MySQL – Serveur flexible est disponible en préversion.

MySQL est l’un des moteurs de base de données couramment utilisés pour exécuter des applications web et mobiles à l’échelle d’Internet. La plupart de nos clients s’en servent pour leurs services de formation en ligne, services de diffusion vidéo, solutions de paiement numérique, plateformes de commerce électronique, services de jeux, portails d’actualité, administrations publiques et sites web de santé. Ces services sont requis à des fins de mise à l’échelle à mesure que le trafic sur l’application web ou mobile augmente.

Du côté des applications, l’application est généralement développée en Java ou PHP et migrée pour s’exécuter sur des groupes de machines virtuelles identiques Azure, Azure App Services ou en conteneur sur Azure Kubernetes Service (AKS). Avec un groupe de machines virtuelles identiques, App Service ou AKS en tant qu’infrastructure sous-jacente, la mise à l’échelle des applications est simplifiée grâce à l’approvisionnement instantané de nouvelles machines virtuelles et à la réplication des composants sans état des applications pour répondre aux demandes, mais souvent, la base de données finit par constituer un goulot d’étranglement comme composant avec état centralisé.

La fonctionnalité de réplica en lecture vous permet de répliquer les données d’un serveur flexible Azure Database pour MySQL sur un serveur en lecture seule. Vous pouvez effectuer la réplication à partir du serveur source vers **10** réplicas au maximum. Les réplicas sont mis à jour de manière asynchrone à l’aide de la technologie de réplication selon la position du fichier journal binaire (binlog) native au moteur MySQL. Pour en savoir plus sur la réplication binlog, consultez la [vue d’ensemble de la réplication binlog MySQL](https://dev.mysql.com/doc/refman/5.7/en/binlog-replication-configuration-overview.html).

Les réplicas sont de nouveaux serveurs que vous gérez de manière similaire à votre source Azure Database pour serveurs flexibles MySQL. Vous encourez des frais de facturation pour chaque réplica en lecture, en fonction de la capacité de calcul provisionnée dans les vCores et du stockage provisionné en Go/mois. Pour plus d’informations, consultez [Tarification](./concepts-compute-storage.md#pricing).

Pour découvrir plus en détail les fonctionnalités de réplication MySQL et les problèmes associés, consultez la [documentation sur la réplication MySQL](https://dev.mysql.com/doc/refman/5.7/en/replication-features.html).

> [!NOTE]
> Communication sans biais
>
> La diversité et l’inclusion sont au cœur des valeurs de Microsoft. Cet article contient des références au mot _esclave_. Le [guide de style de Microsoft sur la communication sans stéréotype](https://github.com/MicrosoftDocs/microsoft-style-guide/blob/master/styleguide/bias-free-communication.md) le reconnaît comme un mot à exclure. Le mot est utilisé dans cet article pour des raisons de cohérence, car il s’agit du mot qui figure dans le logiciel. Une fois que le mot aura été supprimé du logiciel, cet article sera mis à jour en conséquence.
>

## <a name="common-use-cases-for-read-replica"></a>Cas d’usage courants du réplica en lecture

La fonctionnalité de réplica en lecture contribue à améliorer les performances et la scalabilité des charges de travail nécessitant une lecture intensive. Les charges de travail de lecture peuvent être isolées sur les réplicas, tandis que les charges de travail d’écriture peuvent être dirigées vers la source.

Voici des cas d’utilisation courants :

* Mise à l’échelle des charges de travail de lecture provenant de l’application à l’aide d’un proxy de connexion léger comme [ProxySQL](https://aka.ms/ProxySQLLoadBalanceReplica) ou d’un modèle basé sur les microservices à des fins de scale-out de vos requêtes de lecture en réplicas en lecture.
* Les charges de travail décisionnelles et analytiques peuvent utiliser des réplicas en lecture en tant que source de données pour la création de rapports.
* Pour les scénarios IoT ou de fabrication dans lesquels les informations de télémétrie sont ingérées dans le moteur de base de données MySQL alors que plusieurs réplicas en lecture sont utilisés pour la création de rapports de données.

Dans la mesure où les réplicas sont en lecture seule, ils ne réduisent pas directement les charges relatives à la capacité d’écriture sur la source. Cette fonctionnalité ne cible pas les charges de travail nécessitant une écriture intensive.

La fonctionnalité de réplica en lecture utilise la réplication asynchrone MySQL. La fonctionnalité n’est pas destinée aux scénarios de réplication synchrone. Il y aura un délai mesurable entre la source et le réplica. Les données du réplica finissent par devenir cohérentes avec les données du serveur source. Utilisez cette fonctionnalité pour les charges de travail pouvant s’adapter à ce délai.

## <a name="create-a-replica"></a>Créer un réplica

Si un serveur source ne dispose d’aucun serveur réplica, le serveur source redémarre tout d’abord afin de se préparer pour la réplication.

Quand vous démarrez le workflow de création de réplica, un serveur Azure Database pour MySQL vide est créé. Le nouveau serveur est rempli avec les données qui se trouvaient sur le serveur source. Le temps de création dépend de la quantité de données présentes sur le serveur source et du temps écoulé depuis la dernière sauvegarde complète hebdomadaire. Le temps nécessaire peut aller de quelques minutes à plusieurs heures.

> [!NOTE]
> Les réplicas en lecture sont créés avec la même configuration de serveur que le serveur source. La configuration du serveur réplica peut être modifiée après la création de ce dernier. Le serveur réplica est toujours créé dans le même groupe de ressources, le même emplacement et le même abonnement que le serveur source. Si vous souhaitez créer un serveur réplica dans un autre groupe de ressources ou un autre abonnement, vous pouvez [déplacer le serveur réplica](../../azure-resource-manager/management/move-resource-group-and-subscription.md) après sa création. Il est recommandé de maintenir la configuration du serveur réplica à des valeurs égales ou supérieures à celles du serveur source pour garantir que le réplica sera à la hauteur du serveur source.

Découvrez comment [créer un réplica en lecture dans le portail Azure](how-to-read-replicas-portal.md).

## <a name="connect-to-a-replica"></a>Se connecter à un réplica

Au moment de sa création, un réplica hérite de la méthode de connectivité du serveur source. Vous ne pouvez pas modifier la méthode de connectivité du réplica. Par exemple, si le serveur source dispose d’un **accès privé (intégration au réseau virtuel)** , le réplica ne peut pas être en **accès public (adresses IP autorisées)** .

Le réplica hérite du compte Administrateur du serveur source. Tous les comptes d’utilisateur sur le serveur source sont répliqués sur les réplicas en lecture. Vous pouvez uniquement vous connecter à un réplica en lecture à l’aide des comptes d’utilisateur disponibles sur le serveur source.

Vous pouvez vous connecter au réplica à l’aide de son nom d’hôte et d’un compte d’utilisateur valide, comme vous le faites sur un serveur Azure Database pour serveur flexible MySQL. Sur un serveur nommé **myreplica**, à l’aide du nom d’utilisateur administrateur **myadmin**, vous pouvez vous connecter au réplica via l’interface de ligne de commande mysql :

```bash
mysql -h myreplica.mysql.database.azure.com -u myadmin -p
```

À l’invite, entrez le mot de passe du compte d’utilisateur.

## <a name="monitor-replication"></a>Superviser la réplication

Azure Database pour serveur flexible MySQL fournit la métrique **Décalage de la réplication en secondes** dans Azure Monitor. Cette métrique est disponible pour les réplicas uniquement. Cette métrique est calculée à l’aide de la métrique `seconds_behind_master` disponible dans la commande `SHOW SLAVE STATUS` de MySQL. Définissez une alerte qui vous informe quand le décalage de la réplication atteint une valeur inacceptable pour votre charge de travail.

Si vous constatez un décalage de la réplication supérieur, consultez [Résoudre les problèmes de latence de réplication](./../howto-troubleshoot-replication-latency.md) pour dépanner et comprendre les causes possibles.

## <a name="stop-replication"></a>Arrêter la réplication

Vous pouvez arrêter la réplication entre un serveur source et un réplica. Une fois la réplication arrêtée entre un serveur source et un réplica en lecture, celui-ci devient un serveur autonome. Les données du serveur autonome sont les données disponibles sur le réplica au lancement de la commande d’arrêt de la réplication. Le serveur autonome ne se met pas au même niveau que le serveur source.

Lorsque vous décidez d’arrêter la réplication pour un réplica, celui-ci perd tous les liens avec son serveur source précédent et d’autres réplicas. Il n’existe aucun basculement automatique entre un serveur source et son réplica.

> [!IMPORTANT]
> Le serveur autonome ne peut pas être retransformé en réplica.
> Avant d’arrêter la réplication sur un réplica en lecture, vérifiez que celui-ci contient toutes les données nécessaires.

Découvrez comment [arrêter la réplication sur un réplica](how-to-read-replicas-portal.md).

## <a name="failover"></a>Basculement

Il n’existe aucun basculement automatique entre les serveurs source et réplica. 

Les réplicas en lecture ont vocation à mettre à l’échelle les charges de travail intensives en lecture, et non à répondre aux besoins de haute disponibilité d’un serveur. Il n’existe aucun basculement automatique entre les serveurs source et réplica. L’arrêt de la réplication sur le réplica en lecture pour le mettre en ligne en mode lecture-écriture permet d’effectuer ce basculement manuel.

Étant donné que la réplication est asynchrone, il existe un décalage entre le serveur source et le réplica. Le niveau de décalage dépend d’un certain nombre de facteurs, comme la charge de travail exécutée sur le serveur source et la latence qui existe entre les centres de données. Dans la plupart des cas, le décalage du réplica va de quelques secondes à quelques minutes. Pour connaître le décalage d’un réplica, consultez la métrique *Décalage de la réplication*, qui est disponible pour chaque réplica. Cette métrique indique le temps écoulé depuis la dernière transaction réexécutée. Il est recommandé d’observer votre réplica sur une période donnée afin de déterminer le décalage moyen. Vous pouvez configurer une alerte afin d’être averti lorsque le décalage d’un réplica sort de la plage définie et prendre les mesures nécessaires.

> [!Tip]
> Si vous basculez vers le réplica, le décalage qui existe au moment où vous supprimez la liaison entre le réplica et le serveur source indiquera la quantité de données perdues.

Une fois que vous avez décidé de basculer vers un réplica : 

1. Arrêtez la réplication vers le réplica<br/>
   Cette étape est nécessaire pour que le serveur de réplication puisse accepter les écritures. Dans le cadre de ce processus, le serveur de réplication est dissocié du serveur source. En général, une fois que le processus d’arrêt de la réplication est lancé, le processus back-end prend environ 2 minutes. Pour comprendre les implications d’une telle action, consultez la section [Arrêter la réplication](#stop-replication).
    
2. Faites pointer votre application vers l’ancien réplica<br/>
   Chaque serveur est associé à une chaîne de connexion unique. Mettez à jour votre application pour qu’elle pointe vers l’ancien réplica plutôt que le serveur source.
    
Lorsque votre application est en mesure de traiter les lectures et les écritures, le basculement est terminé. Le temps d’arrêt de votre application dépend du moment où vous détectez le problème et où vous effectuez les étapes 1 et 2 ci-dessus.

## <a name="considerations-and-limitations"></a>Observations et limitations

| Scénario | Limitation/Considération |
|:-|:-|
| Réplica sur serveur avec haute disponibilité redondante interzone | Non pris en charge |
| Réplication en lecture inter-régions | Non pris en charge |
| Tarifs | Le coût d’exécution du serveur de réplication est basé sur la région où le serveur de réplication est en cours d’exécution. |
| Redémarrage du serveur source | Lorsque vous créez un réplica pour un serveur source qui n’en a pas, ce dernier commence par redémarrer afin de se préparer à la réplication. Tenez-en compte et effectuez ces opérations en période creuse. |
| Nouveaux réplicas | Un réplica en lecture est créé en tant que serveur flexible Azure Database pour MySQL. Un serveur existant ne peut pas être transformé en réplica. Vous ne pouvez pas créer un réplica d’un autre réplica en lecture. |
| Configuration du réplica | Un réplica est créé à partir de la même configuration que celle du serveur source. Une fois le réplica créé, vous pouvez changer plusieurs paramètres indépendamment du serveur source : génération de calcul, vCores, stockage et période de conservation de la sauvegarde. Le niveau de calcul peut également être modifié indépendamment.<br> <br> **IMPORTANT**  <br> - Avant de mettre à jour une configuration de serveur source avec de nouvelles valeurs, mettez à jour la configuration du réplica avec des valeurs égales ou supérieures. Ainsi, vous aurez la garantie que le réplica peut suivre les changements apportés au serveur source. <br/> La méthode de connectivité et les paramètres sont transmis du serveur source au réplica lorsque le réplica est créé. Par la suite, les règles du réplica sont indépendantes. |
| Réplicas arrêtés | Si vous arrêtez la réplication entre un serveur source et un réplica en lecture, le réplica arrêté devient un serveur autonome qui accepte aussi bien les lectures que les écritures. Le serveur autonome ne peut pas être retransformé en réplica. |
| Serveurs sources et autonomes supprimés | Lorsqu’un serveur source est supprimé, la réplication est arrêtée sur tous les réplicas en lecture. Ces réplicas deviennent automatiquement des serveurs autonomes pouvant accepter des lectures et des écritures. Le serveur source lui-même est supprimé. |
| Comptes d'utilisateurs | Les utilisateurs sur le serveur source sont répliqués sur les réplicas en lecture. Vous ne pouvez vous connecter à un réplica en lecture qu’avec des comptes d’utilisateur disponibles sur le serveur source. |
| Paramètres de serveur | Pour empêcher les données de se désynchroniser et pour éviter leur perte ou leur endommagement potentiels, certains paramètres de serveur sont verrouillés et ne peuvent pas être mis à jour lors de l’utilisation des réplicas en lecture. <br> Les paramètres de serveur suivants sont verrouillés sur les serveurs sources et réplicas :<br> - [`innodb_file_per_table`](https://dev.mysql.com/doc/refman/8.0/en/innodb-file-per-table-tablespaces.html) <br> - [`log_bin_trust_function_creators`](https://dev.mysql.com/doc/refman/5.7/en/replication-options-binary-log.html#sysvar_log_bin_trust_function_creators) <br> Le paramètre [`event_scheduler`](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_event_scheduler) est verrouillé sur les serveurs réplicas. <br> Pour mettre à jour l’un des paramètres ci-dessus sur le serveur source, supprimez les serveurs réplicas, mettez à jour la valeur du paramètre sur le serveur source, puis recréez les réplicas. |
| Autre | - La création d’un réplica d’un réplica n’est pas prise en charge. <br> - Les tables en mémoire peuvent entraîner la désynchronisation des réplicas. Il s’agit d’une limitation de la technologie de réplication MySQL. Pour plus d’informations, lisez la [documentation de référence MySQL](https://dev.mysql.com/doc/refman/5.7/en/replication-features-memory.html). <br>- Vérifiez que les tables du serveur source possèdent des clés primaires. L’absence de clés primaires peut entraîner une latence de réplication entre la source et les réplicas.<br>- Passer en revue la liste complète des limitations relatives à la réplication MySQL dans la [documentation MySQL](https://dev.mysql.com/doc/refman/5.7/en/replication-features.html) |

## <a name="next-steps"></a>Étapes suivantes

- Découvrir comment [créer et gérer des réplicas en lecture clusters avec le portail Azure](how-to-read-replicas-portal.md)
- Découvrir comment [créer et gérer des réplicas en lecture avec l’interface de ligne de commande Azure](how-to-read-replicas-cli.md)
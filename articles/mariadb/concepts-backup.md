---
title: Sauvegarde et restauration - Azure Database for MariaDB
description: Découvrez plus en détail les sauvegardes automatiques et la restauration de votre serveur Azure Database for MariaDB.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: conceptual
ms.date: 8/13/2020
ms.openlocfilehash: 68605a22dd0d0b2b716b148399c8406a1ea8d89e
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94541740"
---
# <a name="backup-and-restore-in-azure-database-for-mariadb"></a>Sauvegarde et restauration dans Azure Database for MariaDB

Azure Database for MariaDB crée automatiquement des sauvegardes de serveur et les conserve dans un stockage géoredondant ou redondant localement configuré par l’utilisateur. Les sauvegardes peuvent être utilisées pour restaurer votre serveur à un point dans le temps. La sauvegarde et la restauration sont une partie essentielle de toute stratégie de continuité d’activité, dans la mesure où elles protègent vos données des corruptions et des suppressions accidentelles.

## <a name="backups"></a>Sauvegardes

Azure Database for MariaDB accepte les sauvegardes complètes, différentielles et de journal des transactions. Celles-ci vous permettent de restaurer un serveur à n’importe quel point dans le temps au sein de votre période de rétention de sauvegarde configurée. La période de rétention de sauvegarde par défaut est de sept jours. Vous pouvez éventuellement la configurer sur 35 jours maximum. Toutes les sauvegardes sont chiffrées à l’aide du chiffrement AES de 256 bits.

Ces fichiers de sauvegarde ne sont pas exposés à l’utilisateur et ne peuvent pas être exportés. Ces sauvegardes ne peuvent être utilisées que pour des opérations de restauration dans Azure Database for MariaDB. Vous pouvez utiliser [mysqldump](howto-migrate-dump-restore.md) pour copier une base de données.

### <a name="backup-frequency"></a>Fréquence de sauvegarde

#### <a name="servers-with-up-to-4-tb-storage"></a>Serveurs avec jusqu’à 4 To de stockage

Pour les serveurs qui prennent en charge jusqu’à 4 To de stockage, les sauvegardes complètes se produisent une fois par semaine. Les sauvegardes différentielles se produisent deux fois par jour. Les sauvegardes des journaux des transactions se produisent toutes les cinq minutes.

#### <a name="servers-with-up-to-16-tb-storage"></a>Serveurs avec jusqu’à 16 To de stockage
Dans un sous-ensemble de [régions Azure](concepts-pricing-tiers.md#storage), tous les serveurs nouvellement approvisionnés peuvent prendre en charge un stockage jusqu’à 16 To de stockage. Les sauvegardes sur ces serveurs de stockage volumineux sont basées sur des captures instantanées. La première sauvegarde de capture instantanée complète est planifiée immédiatement après la création d’un serveur. La première sauvegarde complète de capture instantanée est conservée en tant que sauvegarde de base du serveur. Les sauvegardes de captures instantanées suivantes sont des sauvegardes différentielles uniquement. 

Les sauvegardes de captures instantanées différentielles se produisent au moins une fois par jour. Les sauvegardes de captures instantanées différentielles ne se produisent pas selon une planification fixe. Les sauvegardes de captures instantanées différentielles ont lieu toutes les 24 heures, à moins que le journal des transactions (binlog dans MariaDB) dépasse 50 Go depuis la dernière sauvegarde différentielle. Au cours d’une journée, six captures instantanées différentielles maximum sont autorisées. 

Les sauvegardes des journaux des transactions se produisent toutes les cinq minutes. 

### <a name="backup-retention"></a>Rétention des sauvegardes

Les sauvegardes sont conservées en fonction du paramètre de période de rétention de sauvegarde sur le serveur. Vous pouvez sélectionner une période de rétention comprise entre 7 et 35 jours. La période de conservation par défaut est 7 jours. Vous pouvez définir la période de rétention lors de la création du serveur ou ultérieurement en mettant à jour la configuration de la sauvegarde à l’aide du [portail Azure](howto-restore-server-portal.md#set-backup-configuration) ou d’[Azure CLI](howto-restore-server-cli.md#set-backup-configuration). 

La période de rétention de sauvegarde détermine jusqu’à quelle date une restauration à un point dans le temps peut être récupérée, dans la mesure où elle est basée sur les sauvegardes disponibles. La période de rétention de sauvegarde peut également être traitée comme une fenêtre de récupération du point de vue de la restauration. Toutes les sauvegardes requises pour effectuer une restauration à un instant dans le passé au cours de la période de rétention de sauvegarde sont conservées dans le stockage de sauvegarde. Par exemple, si la période de conservation des sauvegardes est définie sur 7 jours, la fenêtre de récupération est considérée comme les 7 derniers jours. Dans ce scénario, toutes les sauvegardes nécessaires à la restauration du serveur au cours des 7 derniers jours sont conservées. Avec une fenêtre de rétention de sauvegarde de sept jours :
- Les serveurs avec un stockage jusqu’à 4 To peuvent conserver jusqu’à 2 sauvegardes complètes de base de données, toutes les sauvegardes différentielles et les sauvegardes du journal des transactions effectuées depuis la première sauvegarde complète de la base de données.
-   Les serveurs avec un stockage jusqu’à 16 To conservent la capture instantanée complète de base de données, toutes les captures instantanées différentielles et les sauvegardes du journal des transactions au cours des 8 derniers jours.

#### <a name="long-term-retention-of-backups"></a>Rétention à long terme des sauvegardes
La rétention à long terme des sauvegardes au-delà de 35 jours n’est pas encore prise en charge de manière native par le service. Vous avez la possibilité d’utiliser mysqldump pour effectuer des sauvegardes et les stocker pour une conservation à long terme. Notre équipe de support technique a publié un [article pas à pas](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/automate-backups-of-your-azure-database-for-mysql-server-to/ba-p/1791157) pour vous permettre de savoir comment y parvenir. 

### <a name="backup-redundancy-options"></a>Options de redondance de sauvegarde

Azure Database for MariaDB offre la possibilité de choisir entre le stockage de sauvegarde géoredondant ou redondant localement dans les niveaux Usage général et Mémoire optimisée. Lorsque les sauvegardes sont conservées dans le stockage de sauvegarde géoredondant, elles ne sont pas uniquement conservées dans la région d’hébergement de votre serveur, mais sont également répliquées dans un [centre de données jumelé](../best-practices-availability-paired-regions.md). Cela permet de bénéficier d’une meilleure protection et de la possibilité de restaurer votre serveur dans une région différente en cas de sinistre. Le niveau De base propose uniquement un stockage de sauvegarde redondant localement.

#### <a name="moving-from-locally-redundant-to-geo-redundant-backup-storage"></a>Passage du stockage localement redondant au stockage géoredondant
La configuration du stockage géoredondant ou redondant localement pour la sauvegarde est uniquement possible lors de la création du serveur. Une fois que le serveur est approvisionné, vous ne pouvez pas modifier l’option de redondance du stockage de sauvegarde. Pour déplacer votre stockage de sauvegarde d’un stockage localement redondant vers un stockage géo-redondant, la création d’un nouveau serveur et la migration des données à l’aide du [vidage et de la restauration](howto-migrate-dump-restore.md) est la seule option prise en charge.

### <a name="backup-storage-cost"></a>Coût du stockage de sauvegarde

Azure Database for MariaDB fournit jusqu’à 100 % du stockage de votre serveur provisionné en stockage de sauvegarde sans coût supplémentaire. Tous les stockages de sauvegarde supplémentaires utilisés sont facturés en Go par mois. Par exemple, si vous avez configuré un serveur avec 250 Go de stockage, vous disposez de 250 Go de stockage supplémentaire pour les sauvegardes de serveur sans frais supplémentaires. Le stockage utilisé pour les sauvegardes de plus de 250 Go est facturé conformément au [modèle de tarification](https://azure.microsoft.com/pricing/details/mariadb/). 

Vous pouvez utiliser la métrique [Stockage de sauvegarde utilisé](concepts-monitoring.md) dans Azure Monitor, disponible via le portail Azure, pour superviser le stockage de sauvegarde consommé par un serveur. La métrique Stockage de sauvegarde utilisé représente le total du stockage consommé par l’ensemble des sauvegardes de base de données complètes, sauvegardes différentielles et sauvegardes de journaux conservées en fonction de la période de rétention de sauvegarde définie pour le serveur. La fréquence des sauvegardes est gérée par le service et expliquée plus haut. Une activité transactionnelle importante sur le serveur peut entraîner une augmentation de l’utilisation du stockage de sauvegarde, quelle que soit la taille totale de la base de données. Pour le stockage géo-redondant, l’utilisation du stockage de sauvegarde est le double de celle du stockage localement redondant. 

Le principal moyen de contrôler le coût fu stockage de sauvegarde consiste à définir la période de rétention de sauvegarde appropriée et à choisir les options de redondance de sauvegarde appropriées pour atteindre les objectifs de récupération souhaités. Vous pouvez sélectionner une période de conservation comprise entre 7 et 35 jours. Les serveurs à usage général et à mémoire optimisée peuvent disposer d’un stockage géoredondant pour les sauvegardes.

## <a name="restore"></a>Restaurer

Dans Azure Database for MariaDB, l’exécution d’une restauration crée un serveur à partir de sauvegardes du serveur d’origine, et restaure toutes les bases de données contenues dans le serveur.

Deux types de restauration sont disponibles :

- La **restauration à un point dans le temps** est disponible avec l’option de redondance de sauvegarde, et crée un serveur dans la même région que votre serveur d’origine en utilisant la combinaison de sauvegarde complète et de sauvegarde du journal des transactions.
- La **géorestauration** est disponible uniquement si vous avez configuré votre serveur pour le stockage géoredondant. Elle vous permet de restaurer votre serveur dans une autre région en utilisant la sauvegarde la plus récente.

Le délai estimé de récupération dépend de plusieurs facteurs, notamment du nombre total de bases de données à récupérer dans la même région au même moment, de la taille des bases de données, de la taille du journal des transactions et de la bande passante réseau. Le délai de récupération est généralement inférieur à 12 heures.

> [!IMPORTANT]
> Il n’est **pas** possible de restaurer des serveurs supprimés. Si vous supprimez le serveur, toutes les bases de données qui appartiennent au serveur sont également supprimées, sans pouvoir être restaurées.Pour protéger les ressources du serveur, après le déploiement, contre une suppression accidentelle ou des changements inattendus, les administrateurs peuvent utiliser [les verrous de gestion](../azure-resource-manager/management/lock-resources.md).

### <a name="point-in-time-restore"></a>Restauration dans le temps

Quelle que soit l’option de redondance de sauvegarde, vous pouvez effectuer une restauration à n’importe quel point dans le temps au sein de votre période de rétention de sauvegarde. Un serveur est créé dans la même région Azure que le serveur d’origine. Il est créé avec la configuration du serveur d’origine pour le niveau de tarification, la génération de calcul, le nombre de vCores, la taille de stockage, la période de rétention de sauvegarde et l’option de redondance de sauvegarde.

La restauration à un point dans le temps est utile dans plusieurs scénarios. Par exemple, lorsqu’un utilisateur supprime accidentellement des données, perd une base de données ou une table importante ou si une application remplace accidentellement des données correctes par des données erronées en raison d’un défaut de l’application.

Vous devez peut-être attendre la prochaine sauvegarde du journal des transactions avant de pouvoir restaurer à un point dans le temps dans les cinq dernières minutes.

### <a name="geo-restore"></a>La géorestauration

Vous pouvez restaurer un serveur dans une autre région Azure où le service est disponible si vous avez configuré votre serveur pour les sauvegardes géoredondantes. Les serveurs qui prennent en charge jusqu’à 4 To de stockage peuvent être restaurés dans la région géographiquement associée ou dans n’importe quelle région qui prend en charge jusqu’à 16 To de stockage. Pour les serveurs prenant en charge jusqu’à 16 To de stockage, les géo-sauvegardes peuvent être restaurées dans n’importe quelle région qui prend également en charge les serveurs de 16 To. Passez en revue [Niveaux tarifaires d’Azure Database for MariaDB](concepts-pricing-tiers.md) pour obtenir la liste des régions prises en charge.

La géorestauration constitue l’option de récupération par défaut lorsque votre serveur est indisponible en raison d’un incident dans la région où il est hébergé. Si un incident à grande échelle dans une région entraîne l’indisponibilité de votre application de base de données, vous pouvez restaurer un serveur à partir des sauvegardes géoredondantes sur un serveur situé dans n’importe quelle autre région. La géorestauration utilise la sauvegarde la plus récente du serveur. Il peut y avoir un délai entre le moment où une sauvegarde est effectuée et celui où elle est répliquée dans une autre région. Ce délai peut atteindre une heure. En cas d’incident, il peut donc y avoir jusqu’à une heure de pertes de données.

> [!IMPORTANT]
>Si une géorestauration est effectuée pour un serveur nouvellement créé, la synchronisation de la sauvegarde initiale peut prendre plus de 24 heures en fonction de la taille des données, car la durée de la sauvegarde initiale d’un instantané complet est bien supérieure. Les sauvegardes d’instantanés ultérieures sont des copies incrémentielles. Par conséquent, les restaurations sont plus rapides 24 heures après la création du serveur. Si vous évaluez des géorestaurations pour définir votre RTO, nous vous recommandons d’attendre et de les évaluer **uniquement 24 heures après** la création du serveur pour obtenir de meilleures estimations.

Pendant la géorestauration, les configurations de serveur qui peuvent être changées incluent la génération de calcul, les vCores, la période de conservation des sauvegardes et les options de redondance de sauvegarde. Le changement de niveau tarifaire (De base, Usage général ou Mémoire optimisée) ou de la taille du stockage pendant la géorestauration n’est pas pris en charge.

Le délai estimé de récupération dépend de plusieurs facteurs, notamment du nombre total de bases de données à récupérer dans la même région au même moment, de la taille des bases de données, de la taille du journal des transactions et de la bande passante réseau. Le délai de récupération est généralement inférieur à 12 heures.

### <a name="perform-post-restore-tasks"></a>Effectuer des tâches de post-restauration

Après une restauration à l’aide d’un de ces mécanismes de récupération, vous devez effectuer les tâches suivantes afin que les utilisateurs et les applications soient de nouveau opérationnels :

- Si le nouveau serveur est destiné à remplacer le serveur d’origine, redirigez les clients et les applications clientes vers le nouveau serveur
- Vérifiez que les règles de réseau virtuel appropriées sont en place pour permettre aux utilisateurs de se connecter. Ces règles ne sont pas copiées à partir du serveur d’origine.
- Assurez-vous que les connexions et les autorisations appropriées au niveau de la base de données sont en place
- Configurer les alertes, selon les besoins

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur la continuité d’activité, consultez la  [vue d’ensemble de la continuité d’activité](concepts-business-continuity.md).
- Pour effectuer une restauration à un point dans le temps à l’aide du portail Azure, consultez  [Restaurer un serveur à un point dans le temps à l’aide du portail Azure](howto-restore-server-portal.md).
- Pour effectuer une restauration à un point dans le temps à l’aide d’Azure CLI, consultez  [Restaurer un serveur à un point dans le temps à l’aide d’Azure CLI](howto-restore-server-cli.md).

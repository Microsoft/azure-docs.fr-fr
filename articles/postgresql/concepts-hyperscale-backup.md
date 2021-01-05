---
title: Sauvegarde et restauration - Hyperscale (Citus) - Azure Database pour PostgreSQL
description: Protection des données contre une altération ou une suppression accidentelle
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 04/28/2020
ms.openlocfilehash: 9656e6ab608665e325ac5f772e06df9ac296c521
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92485373"
---
# <a name="backup-and-restore-in-azure-database-for-postgresql---hyperscale-citus"></a>Sauvegarde et restauration dans Azure Database pour PostgreSQL - Hyperscale (Citus)

Azure Database pour PostgreSQL – Hyperscale (Citus) crée automatiquement des sauvegardes de chaque nœud et les conserve dans un stockage localement redondant. Les sauvegardes peuvent être utilisées pour restaurer votre cluster Hyperscale (Citus) à une heure spécifiée. La sauvegarde et la restauration sont une partie essentielle de toute stratégie de continuité d’activité, dans la mesure où elles protègent vos données des corruptions et des suppressions accidentelles.

## <a name="backups"></a>Sauvegardes

Au moins une fois par jour, Azure Database pour PostgreSQL effectue des sauvegardes instantanées des fichiers de données et du journal des transactions de base de données. Les sauvegardes vous permettent de restaurer un serveur à n’importe quel point dans le temps au sein de la période de conservation. (La période de conservation est de 35 jours pour tous les clusters.) Toutes les sauvegardes sont chiffrées à l’aide du chiffrement AES de 256 bits.

Dans les régions Azure qui prennent en charge les zones de disponibilité, les instantanés de sauvegarde sont stockés dans trois zones de disponibilité. Tant qu’au moins une zone de disponibilité est en ligne, le cluster Hyperscale (Citus) peut être restauré.

Les fichiers de sauvegarde ne peuvent pas être exportés. Ils sont utilisables uniquement pour les opérations de restauration dans Azure Database pour PostgreSQL.

### <a name="backup-storage-cost"></a>Coût du stockage de sauvegarde

Pour connaître les prix du stockage de sauvegarde, consultez la [page des tarifs](https://azure.microsoft.com/pricing/details/postgresql/hyperscale-citus/) d’Azure Database pour PostgreSQL - Hyperscale (Citus).

## <a name="restore"></a>Restaurer

Dans Azure Database pour PostgreSQL, la restauration d’un cluster Hyperscale (Citus) crée un cluster à partir des sauvegardes des nœuds d’origine. 

> [!IMPORTANT]
>Vous pouvez uniquement restaurer le cluster Hyperscale (Citus) au sein du même abonnement et du même groupe de ressources, et avec un nom de cluster différent.


> [!IMPORTANT]
> Les clusters Hyperscale (Citus) supprimés ne peuvent pas être restaurés. Si vous supprimez le cluster, tous les nœuds qui appartiennent au cluster sont supprimés et ne peuvent pas être récupérés. À l’issue du déploiement, pour protéger les ressources du cluster d’une suppression accidentelle ou de changements inattendus, les administrateurs peuvent utiliser des [verrous de gestion](../azure-resource-manager/management/lock-resources.md).

### <a name="point-in-time-restore-pitr"></a>Restauration à un point dans le temps

Vous pouvez restaurer un cluster à n’importe quel point dans le temps au cours des 35 derniers jours.
La restauration à un point dans le temps est utile dans plusieurs scénarios. Par exemple, quand un utilisateur supprime accidentellement des données, perd une base de données ou une table importante ou si une application remplace accidentellement des données correctes par des données erronées.

Le processus de restauration crée un cluster dans les mêmes région, abonnement et groupe de ressources Azure que le cluster d’origine. Le cluster a la configuration d’origine : mêmes nombre de nœuds, nombre de vCores, taille de stockage, rôles d’utilisateur, version PostgreSQL et version de l’extension Citus.

Les paramètres de pare-feu et les paramètres de serveur PostgreSQL ne sont pas conservés à partir du groupe de serveurs d’origine ; ils sont réinitialisés aux valeurs par défaut. Le pare-feu empêche toute connexion. Vous devez ajuster manuellement ces paramètres après la restauration.

> [!IMPORTANT]
> Vous devez ouvrir une demande de support pour effectuer une restauration à un point dans le temps de votre cluster Hyperscale (Citus).

### <a name="post-restore-tasks"></a>Tâches post-restauration

Après une restauration à l’aide d’un de ces mécanismes de récupération, vous devez effectuer les opérations suivantes afin que les utilisateurs et les applications soient de nouveau opérationnels :

* Si le nouveau serveur est destiné à remplacer le serveur d’origine, redirigez les clients et les applications clientes vers le nouveau serveur
* Vérifiez que les règles de pare-feu et de réseau virtuel appropriées sont en place au niveau du serveur pour permettre aux utilisateurs de se connecter. Ces règles ne sont pas copiées à partir du groupe de serveurs d’origine.
* Ajustez les paramètres du serveur PostgreSQL, si nécessaire. Les paramètres ne sont pas copiés à partir du groupe de serveurs d’origine.
* Assurez-vous que les connexions et les autorisations appropriées au niveau de la base de données sont en place
* Configurer les alertes, selon les besoins

## <a name="next-steps"></a>Étapes suivantes

* Découvrez-en plus sur les  [zones de disponibilité Azure](../availability-zones/az-overview.md).
* Définissez des  [alertes suggérées](./howto-hyperscale-alert-on-metric.md#suggested-alerts) sur des groupes de serveurs Hyperscale (Citus).
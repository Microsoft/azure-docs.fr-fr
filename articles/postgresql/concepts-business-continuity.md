---
title: Continuité de l’activité - Azure Database pour PostgreSQL - Serveur unique
description: Cet article décrit la continuité d’activité (limite de restauration dans le temps, interruption de centre de données, géo-restauration, réplicas) quand vous utilisez Azure Database pour PostgreSQL.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: conceptual
ms.date: 08/07/2020
ms.openlocfilehash: cf3c07f32f15ff176974219bd8143a1ea315c945
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93423043"
---
# <a name="overview-of-business-continuity-with-azure-database-for-postgresql---single-server"></a>Vue d’ensemble de la continuité d’activité avec Azure Database pour PostgreSQL - Serveur unique

Cette vue d’ensemble décrit les fonctionnalités de continuité d’activité et de reprise après sinistre que fournit Azure Database pour PostgreSQL. Découvrez les options de reprise à la suite d’événements d’interruption susceptibles d’entraîner une perte de données ou une indisponibilité de votre base de données et de votre application. Connaissez la procédure à suivre lorsqu’un utilisateur ou qu’une erreur d’application affecte l’intégrité des données, lorsqu’une région Azure subit une panne ou que votre application nécessite une maintenance.

## <a name="features-that-you-can-use-to-provide-business-continuity"></a>Fonctionnalités que vous pouvez utiliser pour garantir la continuité d’activité

Au moment d’élaborer votre plan de continuité d’activité, vous devez comprendre le délai maximal acceptable nécessaire à la récupération complète de l’application après l’événement d’interruption, c’est-à-dire votre objectif de délai de récupération (RTO). Vous devez aussi comprendre la quantité maximale des récentes mises à jour de données (intervalle) que l’application peut accepter de perdre lors de la reprise après l’événement d’interruption, c’est-à-dire votre objectif de point de récupération (RPO).

Azure Database pour PostgreSQL propose des fonctionnalités de continuité d’activité, notamment des sauvegardes géoredondantes offrant la possibilité de lancer une géorestauration et le déploiement de réplicas en lecture dans une autre région. Chacune de ces fonctionnalités présente des caractéristiques spécifiques concernant le temps de récupération et le risque de perte de données. Avec la fonctionnalité de [Géorestauration](concepts-backup.md), un nouveau serveur est créé à l’aide des données de sauvegarde répliquées à partir d’une autre région. Le temps total nécessaire à la restauration et à la récupération dépend de la taille de la base de données et de la quantité de journaux à récupérer. La durée totale d’établissement du serveur varie entre quelques minutes et quelques heures. Avec les [réplicas en lecture](concepts-read-replicas.md), les journaux des transactions du serveur principal sont diffusés de façon asynchrone vers le réplica. En cas de panne de la base de données primaire en raison d’une défaillance au niveau de la zone ou de la région, le basculement vers le réplica permet de raccourcir le RTO et de réduire les pertes de données.

> [!NOTE]
> Le décalage entre le serveur principal et le réplica dépend de la latence entre les sites, de la quantité de données à transmettre et, plus important, de la charge de travail d’écriture du serveur principal. Des charges de travail d’écriture intensives peuvent entraîner un décalage significatif. 
>
> En raison de la nature asynchrone de la réplication utilisée pour les réplicas en lecture, ils **ne doivent pas** être considérés comme une solution à haute disponibilité (HA), car des décalages plus importants peuvent signifier des RTO et RPO plus élevés. Les réplicas en lecture ne peuvent être considérés comme une solution de haute disponibilité que pour les charges de travail pour lesquelles le décalage reste plus faible pendant les périodes de pointe et les périodes creuses de la charge de travail. Dans le cas contraire, les réplicas en lecture sont destinés à une véritable échelle de lecture pour les charges de travail intensives prêtes et pour les scénarios de récupération d’urgence.

Le tableau suivant compare le RTO et le RPO dans un scénario de **charge de travail classique** :

| **Fonctionnalité** | **De base** | **Usage général** | **Mémoire optimisée** |
| :------------: | :-------: | :-----------------: | :------------------: |
| Limite de restauration dans le temps à partir de la sauvegarde | N’importe quel point de restauration dans la période de rétention | N’importe quel point de restauration dans la période de rétention | N’importe quel point de restauration dans la période de rétention |
| Géo-restauration à partir de sauvegardes répliquées géographiquement | Non pris en charge | RTO – Variable <br/>RPO < 1 h | RTO – Variable <br/>RPO < 1 h |
| Réplicas en lecture | RTO – Quelques minutes* <br/>RPO < 5 min* | RTO – Quelques minutes* <br/>RPO < 5 min*| RTO – Quelques minutes* <br/>RPO < 5 min*|

 \* Le RTO et le RPO **peuvent être beaucoup plus élevés** dans certains cas en fonction de différents facteurs, notamment la latence entre les sites, la quantité de données à transmettre et, surtout, la charge de travail d’écriture de la base de données primaire. 

## <a name="recover-a-server-after-a-user-or-application-error"></a>Récupérer un serveur après une erreur d’utilisateur ou d’application

Vous pouvez vous servir des sauvegardes du service pour récupérer un serveur à la suite de plusieurs événements d’interruption. Un utilisateur peut supprimer par inadvertance des données, une table importante, voire une base de données tout entière. Une application peut accidentellement remplacer des données correctes par des données incorrectes en raison d’une défaillance d’application, etc.

Vous pouvez procéder à une **restauration à un point dans le temps** afin de créer une copie de votre serveur à un point dans le temps valide connu. Ce moment donné doit se situer dans la période de rétention de sauvegarde que vous avez configurée pour votre serveur. Une fois les données restaurées sur le nouveau serveur, vous pouvez remplacer le serveur d’origine par le serveur nouvellement restauré ou copier les données nécessaires du serveur restauré sur le serveur d’origine.

> [!IMPORTANT]
> Il n’est **pas** possible de restaurer des serveurs supprimés. Si vous supprimez le serveur, toutes les bases de données qui appartiennent au serveur sont également supprimées, sans pouvoir être restaurées. Utilisez le [verrouillage des ressources Azure](../azure-resource-manager/management/lock-resources.md) pour éviter la suppression accidentelle de votre serveur.

## <a name="recover-from-an-azure-data-center-outage"></a>Récupérer après une panne du centre de données Azure

Bien que le fait soit rare, un centre de données Azure peut subir une panne. En cas de panne, il s’ensuit une interruption d’activité dont la durée peut se limiter à quelques minutes, mais aussi se compter en heures.

Vous pouvez attendre que votre serveur redevienne disponible une fois la panne réparée au niveau du centre de données. Cette solution vaut pour les applications qui peuvent s’accommoder d’un certain temps d’indisponibilité du serveur, par exemple dans un environnement de développement. Quand un centre de données connaît une panne, vous ne savez pas combien de temps cela peut durer. Cette solution n’est donc valable que si vous n’avez pas besoin du serveur pendant un certain temps.

## <a name="geo-restore"></a>La géorestauration

La fonctionnalité de géorestauration permet de restaurer le serveur à l’aide de sauvegardes géoredondantes. Les sauvegardes sont hébergées dans la [région appairée](../best-practices-availability-paired-regions.md) de votre serveur. Vous pouvez effectuer une restauration à partir de ces sauvegardes dans n’importe quelle autre région. La géorestauration crée un serveur avec les données issues des sauvegardes. Pour plus d’informations sur la géorestauration, consultez cet [article sur les concepts de sauvegarde et de restauration](concepts-backup.md).

> [!IMPORTANT]
> La géorestauration n’est possible que si vous avez provisionné le serveur avec le stockage de sauvegardes géoredondantes. Si vous souhaitez basculer des sauvegardes redondantes localement aux sauvegardes géoredondantes pour un serveur existant, vous devez effectuer une copie de sauvegarde de votre serveur existant en utilisant pg_dump et la restaurer vers un serveur nouvellement créé configuré avec des sauvegardes géoredondantes.

## <a name="cross-region-read-replicas"></a>Réplicas en lecture inter-régions
Vous pouvez utiliser des réplicas en lecture inter-régions pour améliorer la planification de la continuité d’activité et de la reprise d’activité. Les réplicas en lecture sont mis à jour de manière asynchrone à l’aide de la technologie de réplication physique de PostgreSQL et peuvent présenter un décalage avec le serveur principal. Pour plus d’informations sur les réplicas en lecture, les régions disponibles et le basculement, consultez cet [article sur les concepts relatifs aux réplicas en lecture](concepts-read-replicas.md). 

## <a name="faq"></a>Questions fréquentes (FAQ)
### <a name="where-does-azure-database-for-postgresql-store-customer-data"></a>Où Azure Database pour PostgreSQL stocke-t-il les données client ?
Par défaut, Azure Database pour PostgreSQL ne déplace pas ou ne stocke pas les données client en dehors de la région dans laquelle elles sont déployées. Toutefois, les clients peuvent éventuellement choisir d’activer les [sauvegardes géoredondantes](concepts-backup.md#backup-redundancy-options) ou de créer un [réplica en lecture sur plusieurs régions](concepts-read-replicas.md#cross-region-replication) pour le stockage des données dans une autre région.


## <a name="next-steps"></a>Étapes suivantes
- Pour plus d’informations sur les sauvegardes automatisées, consultez [Sauvegardes dans Azure Database pour PostgreSQL](concepts-backup.md). 
- Découvrez comment effectuer une restauration à l’aide du [portail Azure](howto-restore-server-portal.md) ou de l’interface [Azure CLI](howto-restore-server-cli.md).
- Apprenez-en davantage sur les [réplicas en lecture dans Azure Database pour PostgreSQL](concepts-read-replicas.md).

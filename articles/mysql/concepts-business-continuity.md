---
title: Continuité de l’activité - Azure Database pour MySQL
description: Découvrez en quoi consiste la continuité d’activité (limite de restauration dans le temps, interruption de centre de données, géo-restauration) quand vous utilisez le service Azure Database pour MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 7/7/2020
ms.openlocfilehash: 15fde6e7558c685537d36f45bcc7e3ff341544ff
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94542491"
---
# <a name="understand-business-continuity-in-azure-database-for-mysql"></a>Comprendre la continuité d’activité dans Azure Database pour MySQL

Cet article décrit les fonctionnalités d’Azure Database pour MySQL en matière de continuité d’activité et de récupération d’urgence. Découvrez les options de reprise à la suite d’événements d’interruption susceptibles d’entraîner une perte de données ou une indisponibilité de votre base de données et de votre application. Connaissez la procédure à suivre lorsqu’un utilisateur ou qu’une erreur d’application affecte l’intégrité des données, lorsqu’une région Azure subit une panne ou que votre application nécessite une maintenance.

## <a name="features-that-you-can-use-to-provide-business-continuity"></a>Fonctionnalités que vous pouvez utiliser pour garantir la continuité d’activité

Azure Database pour MySQL propose des fonctionnalités de continuité d’activité, notamment des sauvegardes automatisées et la possibilité pour les utilisateurs de lancer une géorestauration. Chacune de ces fonctionnalités présente des caractéristiques spécifiques concernant le temps de récupération estimé (ERT) et le risque de perte de données. Le temps de récupération estimé (ERT, Estimated Recovery Time) est le temps estimé pour que la base de données soit pleinement opérationnelle après une demande de restauration ou de basculement. Une fois que vous avez compris ces options, vous pouvez choisir celles qui vous conviennent et les utiliser ensemble dans différents scénarios. Au moment d’élaborer votre plan de continuité d’activité, vous devez comprendre le délai maximal acceptable nécessaire à la récupération complète de l’application après l’événement d’interruption, c’est-à-dire votre objectif de délai de récupération (RTO). Vous devez aussi comprendre la quantité maximale des récentes mises à jour de données (intervalle) que l’application peut accepter de perdre lors de la reprise après l’événement d’interruption, c’est-à-dire votre objectif de point de récupération (RPO).

Le tableau suivant compare l’ERT et le RPO pour les fonctionnalités disponibles :

| **Fonctionnalité** | **De base** | **Usage général** | **Mémoire optimisée** |
| :------------: | :-------: | :-----------------: | :------------------: |
| Limite de restauration dans le temps à partir de la sauvegarde | N’importe quel point de restauration dans la période de rétention | N’importe quel point de restauration dans la période de rétention | N’importe quel point de restauration dans la période de rétention |
| Géo-restauration à partir de sauvegardes répliquées géographiquement | Non pris en charge | ERT < 12 h<br/>RPO < 1 h | ERT < 12 h<br/>RPO < 1 h |

> [!IMPORTANT]
> Il n’est **pas** possible de restaurer des serveurs supprimés. Si vous supprimez le serveur, toutes les bases de données qui appartiennent au serveur sont également supprimées, sans pouvoir être restaurées.

## <a name="recover-a-server-after-a-user-or-application-error"></a>Récupérer un serveur après une erreur d’utilisateur ou d’application

Vous pouvez vous servir des sauvegardes du service pour récupérer un serveur à la suite de plusieurs événements d’interruption. Un utilisateur peut supprimer par inadvertance des données, une table importante, voire une base de données tout entière. Une application peut accidentellement remplacer des données correctes par des données incorrectes en raison d’une défaillance d’application, etc.

Vous pouvez procéder à une restauration jusqu’à une date et heure pour créer une copie de votre serveur à un moment donné adéquat et connu. Ce moment donné doit se situer dans la période de rétention de sauvegarde que vous avez configurée pour votre serveur. Une fois les données restaurées sur le nouveau serveur, vous pouvez remplacer le serveur d’origine par le serveur nouvellement restauré ou copier les données nécessaires du serveur restauré sur le serveur d’origine.

## <a name="recover-from-an-azure-regional-data-center-outage"></a>Récupérer à la suite d’une panne du centre de données régional Azure

Bien que le fait soit rare, un centre de données Azure peut subir une panne. En cas de panne, il s’ensuit une interruption d’activité dont la durée peut se limiter à quelques minutes, mais aussi se compter en heures.

Vous pouvez attendre que votre serveur redevienne disponible une fois la panne réparée au niveau du centre de données. Cette solution vaut pour les applications qui peuvent s’accommoder d’un certain temps d’indisponibilité du serveur, par exemple dans un environnement de développement. Quand un centre de données subit une panne, vous ne savez pas combien de temps cela peut durer. Cette solution n’est donc valable que si vous n’avez pas besoin du serveur pendant un certain temps.

L’autre solution consiste à utiliser la fonctionnalité de géorestauration d’Azure Database pour MySQL qui restaure le serveur à partir de sauvegardes géoredondantes. Ces sauvegardes sont accessibles même si la région dans laquelle votre serveur est hébergé n’est pas disponible. Vous pouvez effectuer une restauration à partir de ces sauvegardes dans n’importe quelle autre région et remettre votre serveur en ligne.

> [!IMPORTANT]
> La géorestauration n’est possible que si vous avez provisionné le serveur avec le stockage de sauvegardes géoredondantes. Si vous souhaitez basculer des sauvegardes redondantes localement aux sauvegardes géoredondantes pour un serveur existant, vous devez effectuer une image mémoire de votre serveur existant en utilisant mysqldump et la restaurer vers un serveur nouvellement créé configuré avec des sauvegardes géoredondantes.

## <a name="cross-region-read-replicas"></a>Réplicas en lecture inter-régions

Vous pouvez utiliser des réplicas en lecture inter-régions pour améliorer la planification de la continuité d’activité et de la reprise d’activité. Les réplicas en lecture sont mis à jour de manière asynchrone à l’aide de la technologie de réplication des journaux des transactions de MySQL. Pour plus d’informations sur les réplicas en lecture, les régions disponibles et le basculement, consultez cet [article sur les concepts relatifs aux réplicas en lecture](concepts-read-replicas.md). 

## <a name="faq"></a>Questions fréquentes (FAQ)
### <a name="where-does-azure-database-for-mysql-store-customer-data"></a>Où Azure Database pour MySQL stocke-t-il les données client ?
Par défaut, Azure Database pour MySQL ne déplace pas ni ne stocke les données client en dehors de la région dans laquelle il est déployé. Toutefois, les clients peuvent s’ils le souhaitent choisir d’activer les [sauvegardes géoredondantes](concepts-backup.md#backup-redundancy-options) ou de créer un [réplica en lecture sur plusieurs régions](concepts-read-replicas.md#cross-region-replication) pour stocker les données dans une autre région.

## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur les [sauvegardes automatisées dans Azure Database pour MySQL](concepts-backup.md).
- Découvrez comment effectuer une restauration à l’aide du [portail Azure](howto-restore-server-portal.md) ou de l’interface [Azure CLI](howto-restore-server-cli.md).
- Apprenez-en davantage sur les [réplicas en lecture dans Azure Database pour MySQL](concepts-read-replicas.md).

---
title: Comment une remise sur réservation Azure est-elle appliquée ?
description: Cet article vous permet de comprendre comment les remises sur instances réservées sont généralement appliquées.
author: bandersmsft
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 03/22/2020
ms.author: banders
ms.openlocfilehash: 1c038e896a9f9fc65c3b2201b3e763e966285c56
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88684675"
---
# <a name="how-a-reservation-discount-is-applied"></a>Comment une remise sur réservation est-elle appliquée ?

Cet article vous permet de comprendre comment les remises sur instances réservées sont généralement appliquées. La remise sur réservation s’applique à l’utilisation de ressource correspondant aux attributs que vous sélectionnez lorsque vous achetez la réservation. Les attributs englobent l’étendue dans laquelle les machines virtuelles, les bases de données SQL, Azure Cosmos DB ou d’autres ressources correspondantes s’exécutent. Par exemple, si vous voulez une remise sur réservation pour quatre machines virtuelles Standard D2 de la région USA Ouest, sélectionnez l’abonnement dans le cadre duquel ces machines virtuelles s’exécutent.

Une remise de réservation repose sur le principe de « *capacité utilisée ou perdue* ». Si vous ne disposez pas des ressources correspondantes pour une heure donnée, vous perdez une quantité de réservation pour cette heure. Vous ne pouvez pas reporter les heures réservées inutilisées.

Lorsque vous arrêtez une ressource, la remise de réservation s'applique automatiquement à une autre ressource correspondante dans l'étendue spécifiée. Si aucune ressource correspondante n'est trouvée dans l'étendue spécifiée, les heures réservées sont *perdues*.

Par exemple, vous pourrez plus tard créer une ressource et disposer d'une réservation correspondante sous-utilisée. La remise de réservation s’applique automatiquement à la nouvelle ressource correspondante.

Si elles s’exécutent dans différents abonnements au sein de votre compte/inscription, sélectionnez l’étendue alors définie comme partagée. Cette étendue partagée permet d’appliquer la remise de réservation à différents abonnements. Vous pouvez modifier l’étendue après l’achat d’une réservation. Pour plus d’informations, voir [Gérer les réservations Azure](manage-reserved-vm-instance.md).

Une remise de réservation s’applique uniquement aux ressources associées aux types d’abonnements Entreprise, Contrat Client Microsoft, CSP, ou avec paiement à l’utilisation. Les ressources opérant dans le cadre d’un abonnement relevant d’autres types d’offres ne bénéficient d’aucune remise sur réservation.

## <a name="when-the-reservation-term-expires"></a>Expiration du terme de la réservation

À l’issue de la période de réservation, la remise sur facturation cesse de s’appliquer et les ressources sont facturées au tarif du paiement à l’utilisation. Par défaut, les réservations ne sont pas configurées pour se renouveler automatiquement. Vous pouvez choisir d’activer le renouvellement automatique d’une réservation en sélectionnant l’option dans les paramètres de renouvellement. En cas de renouvellement automatique, une réservation de remplacement est achetée à l’expiration de la réservation existante. Par défaut, la réservation de remplacement a les mêmes attributs que la réservation arrivant à expiration. Vous pouvez éventuellement changer la périodicité de facturation, le terme ou la quantité dans les paramètres de renouvellement. Tout utilisateur disposant d’un accès propriétaire sur la réservation et l’abonnement utilisé pour la facturation peut configurer le renouvellement.  

## <a name="discount-applies-to-different-sizes"></a>La remise s'applique à différentes tailles

Quand vous achetez une réservation, la remise peut être appliquée à d’autres instances avec des attributs qui se trouvent dans le même groupe de tailles. Cette fonctionnalité est appelée flexibilité de la taille d'instance. La flexibilité de la couverture de remise varie selon le type de réservation et les attributs que vous choisissez quand vous achetez la réservation.

Plans de service :

- Instances de machines virtuelles réservées : Lorsque vous achetez la réservation et sélectionnez **Optimisé pour flexibilité de la taille d’instance**, la couverture de remise dépend de la taille de machine virtuelle que vous sélectionnez. La réservation peut s’appliquer aux tailles des machines virtuelles dans le même groupe de gammes de tailles. Pour plus d’informations, consultez [Flexibilité en termes de taille de machine virtuelle avec des instances de machines virtuelles réservées](../../virtual-machines/reserved-vm-instance-size-flexibility.md).
- Capacité réservée de stockage Azure : Vous pouvez acheter une capacité réservée pour les comptes de stockage Azure standard, en unités de 100 Tio ou 1 PiB par mois. Pour plus d’informations sur les régions qui prennent en charge la capacité de réserve de Stockage Azure, consultez [Tarification des objets blob de blocs](https://azure.microsoft.com/pricing/details/storage/blobs/). La capacité de réserve de Stockage Azure est disponible pour tous les niveaux d’accès (chaud, froid ou archive) et pour toute configuration de réplication (LRS, GRS ou ZRS).
- Capacité réservée SQL Database : la couverture de remise dépend du niveau de performance que vous choisissez. Pour plus d’informations, consultez [Comprendre comment une remise de réservation Azure est appliquée aux bases de données SQL](understand-reservation-charges.md).
- Capacité réservée Azure Cosmos DB : la couverture de remise dépend du débit provisionné. Pour plus d’informations, consultez [Comprendre comment une remise de réservation Azure Cosmos DB est appliquée](understand-cosmosdb-reservation-charges.md).

## <a name="how-discounts-apply-to-specific-azure-services"></a>Comment les remises s’appliquent-elles à des services Azure spécifiques ?

Lisez les articles suivants s’appliquant à votre situation pour savoir comment les remises s’appliquent à un service Azure spécifique :

- [App Service](reservation-discount-app-service-isolated-stamp.md)
- [Cache Azure pour Redis](understand-azure-cache-for-redis-reservation-charges.md)
- [Cosmos DB](understand-cosmosdb-reservation-charges.md)
- [Database for MariaDB](understand-reservation-charges-mariadb.md)
- [Database pour MySQL](understand-reservation-charges-mysql.md)
- [Database pour PostgreSQL](understand-reservation-charges-postgresql.md)
- [Databricks](reservation-discount-databricks.md)
- [Explorateur de données](understand-azure-data-explorer-reservation-charges.md)
- [Hôtes dédiés](billing-understand-dedicated-hosts-reservation-charges.md)
- [Stockage sur disque](understand-disk-reservations.md)
- [Red Hat Enterprise Linux](understand-rhel-reservation-charges.md)
- [Abonnements logiciels](understand-suse-reservation-charges.md)
- [Stockage](understand-storage-charges.md)
- [Base de données SQL](understand-reservation-charges.md)
- [Azure Synapse Analytics](reservation-discount-azure-sql-dw.md)
- [Machines virtuelles](../manage/understand-vm-reservation-charges.md)


## <a name="next-steps"></a>Étapes suivantes

- [Gérer les réservations Azure](manage-reserved-vm-instance.md)
- [Comprendre l’utilisation d’une réservation pour votre abonnement avec paiement à l’utilisation](understand-reserved-instance-usage.md)
- [Comprendre l’utilisation d’une réservation pour votre Accord de Mise en Œuvre Entreprise](understand-reserved-instance-usage-ea.md)
- [Coûts des logiciels Windows non inclus dans les réservations](reserved-instance-windows-software-costs.md)
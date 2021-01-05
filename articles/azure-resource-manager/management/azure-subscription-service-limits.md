---
title: Quotas et limites d’abonnement Azure
description: Fournit une liste des abonnements Azure et des limites, quotas et contraintes de service habituels. Cet article inclut des informations sur la façon d’augmenter les limites ainsi que les valeurs maximales.
ms.topic: conceptual
ms.date: 09/02/2020
ms.openlocfilehash: 4d56c5c3fd33d1cda05e23d5bc53e7540428671b
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96547846"
---
# <a name="azure-subscription-and-service-limits-quotas-and-constraints"></a>Abonnement Azure et limites, quotas et contraintes de service

Ce document répertorie quelques-unes des limites de Microsoft Azure les plus courantes, parfois appelées des quotas.

Pour en savoir plus sur la tarification Azure, consultez [Tarification Azure](https://azure.microsoft.com/pricing/). Elle vous permet d'estimer vos coûts à l’aide de la [calculatrice de prix](https://azure.microsoft.com/pricing/calculator/). Vous pouvez également accéder à la page de détails de tarification pour un service particulier, par exemple, [Machines virtuelles Windows](https://azure.microsoft.com/pricing/details/virtual-machines/#Windows). Pour des conseils sur la gestion des coûts, consultez [Éviter les coûts inattendus avec la gestion de la facturation et des coûts dans Azure](../../cost-management-billing/cost-management-billing-overview.md).

## <a name="managing-limits"></a>Gestion des limites

> [!NOTE]
> Certains services ont des limites ajustables.
>
> Lorsqu’un service n’a pas de limites ajustables, les tables suivantes utilisent l’en-tête **Limite**. Dans de tels cas, la valeur par défaut et les limites maximales sont les mêmes.
>
> Lorsque la limite peut être ajustée, les tables incluent des en-têtes **Limite par défaut** et **Limite maximale**. La limite peut être augmentée au-dessus de la limite par défaut, mais pas au-delà de la limite maximale.
>
> Si vous souhaitez élever la limite ou le quota au-dessus de la limite par défaut, [ouvrez une demande de service clientèle en ligne gratuitement](../templates/error-resource-quota.md).
>
> Les termes *limite non stricte* et *limite inconditionnelle* sont souvent utilisés de manière informelle pour décrire la limite actuelle, la limite ajustable (non stricte) et la limite maximale (inconditionnelle). Si une limite n’est pas réglable, il n’y a pas de limite non stricte, uniquement une limite inconditionnelle.
>

Les [abonnements d’essai gratuit](https://azure.microsoft.com/offers/ms-azr-0044p) ne permettent pas de bénéficier d’augmentations de la limite ou du quota. Si vous disposez d’un [abonnement d’essai gratuit](https://azure.microsoft.com/offers/ms-azr-0044p), vous pouvez le mettre à niveau vers un abonnement avec [paiement à l’utilisation](https://azure.microsoft.com/offers/ms-azr-0003p/). Pour en savoir plus, consultez [Mettre à niveau votre abonnement d'essai gratuit Azure vers un abonnement avec paiement à l’utilisation](../../cost-management-billing/manage/upgrade-azure-subscription.md) et [FAQ sur l’abonnement d’essai gratuit](https://azure.microsoft.com/free/free-account-faq).

Certaines limites sont gérées au niveau régional.

Nous allons utiliser des quotas de processeurs virtuels pour l’exemple. Pour demander une augmentation des quotas de prise en charge des processeurs virtuels, vous devez déterminer le nombre de processeurs virtuels souhaités et les régions concernées. Effectuez ensuite une demande spécifique de quotas de processeurs virtuels de groupes de ressources Azure pour la quantité et les régions de votre choix. Si vous avez besoin de 30 processeurs virtuels dans la région Europe Ouest pour y exécuter votre application, demandez spécifiquement 30 processeurs virtuels dans cette région. Votre quota de processeurs virtuels n’est pas augmenté dans les autres régions ; seule la région Europe Ouest dispose du quota de 30 processeurs virtuels.

Par conséquent, déterminez les quotas de groupes de ressources Azure pour votre charge de travail dans une région. Puis, demandez ce volume dans chaque région où vous envisagez d’effectuer un déploiement. Pour vous aider à déterminer vos quotas actuels dans des régions spécifiques, consultez [Résoudre les erreurs liées aux quotas de ressources](../templates/error-resource-quota.md).

## <a name="general-limits"></a>Limites générales

Pour connaître les restrictions relatives aux noms de ressources, voir [Règles de nommage et restrictions pour les ressources Azure](resource-name-rules.md).

Pour plus d’informations sur l’API Resource Manager et les limites d’écriture, voir [Limitation des requêtes de Resource Manager](request-limits-and-throttling.md).

### <a name="management-group-limits"></a>Limites de groupe d'administration

Les limites suivantes s’appliquent aux [groupes d’administration](../../governance/management-groups/overview.md).

[!INCLUDE [management-group-limits](../../../includes/management-group-limits.md)]

### <a name="subscription-limits"></a>Limites d’abonnement

Les limites suivantes s'appliquent lorsque vous utilisez Azure Resource Manager et les groupes de ressources Azure.

[!INCLUDE [azure-subscription-limits-azure-resource-manager](../../../includes/azure-subscription-limits-azure-resource-manager.md)]

### <a name="resource-group-limits"></a>Limites de groupe de ressources

[!INCLUDE [azure-resource-groups-limits](../../../includes/azure-resource-groups-limits.md)]

## <a name="active-directory-limits"></a>Limites d'Active Directory

[!INCLUDE [AAD-service-limits](../../../includes/active-directory-service-limits-include.md)]

## <a name="api-management-limits"></a>Limites Gestion des API

[!INCLUDE [api-management-service-limits](../../../includes/api-management-service-limits.md)]

## <a name="app-service-limits"></a>Limites App Service

Les limites App Service suivantes incluent celles de Web Apps, Mobile Apps et API Apps.

[!INCLUDE [azure-websites-limits](../../../includes/azure-websites-limits.md)]

## <a name="automation-limits"></a>Limites du service Automation

[!INCLUDE [automation-limits](../../../includes/azure-automation-service-limits.md)]

## <a name="azure-app-configuration"></a>Azure App Configuration

[!INCLUDE [app-configuration-limits](../../../includes/app-configuration-limits.md)]

## <a name="azure-cache-for-redis-limits"></a>Limites du Cache Azure pour Redis

[!INCLUDE [redis-cache-service-limits](../../../includes/redis-cache-service-limits.md)]

## <a name="azure-cloud-services-limits"></a>Limites d'Azure Cloud Services

[!INCLUDE [azure-cloud-services-limits](../../../includes/azure-cloud-services-limits.md)]

## <a name="azure-cognitive-search-limits"></a>Limites de Recherche cognitive Azure

Les niveaux tarifaires déterminent la capacité et les limites de votre service de recherche. Les niveaux sont les suivants :

* **Gratuit** : service multilocataire, partagé avec d’autres abonnés Azure, destiné à l’évaluation et à des petits projets de développement.
* **De base** : fournit des ressources de calcul dédiées pour des charges de travail de production à plus petite échelle, avec jusqu’à trois réplicas pour les charges de travail de requête hautement disponible.
* **Standard**, qui inclut S1, S2, S3 et S3 haute densité, est approprié pour des charges de travail de production de plus grande taille. Plusieurs niveaux existent dans le niveau Standard pour vous permettre de choisir une configuration de ressources parfaitement adaptée à votre profil de charge de travail.

**Limites par abonnement**

[!INCLUDE [azure-search-limits-per-subscription](../../../includes/azure-search-limits-per-subscription.md)]

**Limites par service de recherche**

[!INCLUDE [azure-search-limits-per-service](../../../includes/azure-search-limits-per-service.md)]

Pour plus d’informations sur les limites à un niveau plus précis, notamment la taille des documents, les requêtes par seconde, les clés, les requêtes et les réponses, consultez [Limites du service dans Recherche cognitive Azure](../../search/search-limits-quotas-capacity.md).

## <a name="azure-cognitive-services-limits"></a>Limites d’Azure Cognitive Services

[!INCLUDE [azure-cognitive-services-limits](../../../includes/azure-cognitive-services-limits.md)]

## <a name="azure-cosmos-db-limits"></a>Limites d’Azure Cosmos DB

Pour les limites d’Azure Cosmos DB, consultez [Limites dans Azure Cosmos DB](../../cosmos-db/concepts-limits.md).

## <a name="azure-data-explorer-limits"></a>Limites d'Azure Data Explorer

[!INCLUDE [azure-data-explorer-limits](../../../includes/data-explorer-limits.md)]

## <a name="azure-database-for-mysql"></a>Azure Database pour MySQL

Pour les limites d’Azure Database pour MySQL, voir [Limitations dans Azure Database pour MySQL](../../mysql/concepts-limits.md).

## <a name="azure-database-for-postgresql"></a>Azure Database pour PostgreSQL

Pour les limites d’Azure Database pour PostgreSQL, voir [Limitations dans Azure Database pour PostgreSQL](../../postgresql/concepts-limits.md).

## <a name="azure-functions-limits"></a>Limites d'Azure Functions

[!INCLUDE [functions-limits](../../../includes/functions-limits.md)]

Pour plus d’informations, consultez [Comparaison des plans d’hébergement Functions](../../azure-functions/functions-scale.md#hosting-plans-comparison).

## <a name="azure-kubernetes-service-limits"></a>Limites d'Azure Kubernetes Service

[!INCLUDE [container-service-limits](../../../includes/container-service-limits.md)]

## <a name="azure-machine-learning-limits"></a>Limites d’Azure Machine Learning

Les valeurs les plus récentes des quotas de calcul Azure Machine Learning sont disponibles dans la [page des quotas Azure Machine Learning](../../machine-learning/how-to-manage-quotas.md)

## <a name="azure-maps-limits"></a>Limites d’Azure Maps

[!INCLUDE [maps-limits](../../../includes/maps-limits.md)]

## <a name="azure-monitor-limits"></a>Limites d'Azure Monitor

### <a name="alerts"></a>Alertes

[!INCLUDE [monitoring-limits](../../../includes/azure-monitor-limits-alerts.md)]

### <a name="action-groups"></a>Groupes d’actions

[!INCLUDE [monitoring-limits](../../../includes/azure-monitor-limits-action-groups.md)]

### <a name="autoscale"></a>Mise à l’échelle automatique

[!INCLUDE [monitoring-limits](../../../includes/azure-monitor-limits-autoscale.md)]

### <a name="log-queries-and-language"></a>Requêtes dans les journaux et langage

[!INCLUDE [monitoring-limits](../../../includes/azure-monitor-limits-log-queries.md)]

### <a name="log-analytics-workspaces"></a>Espaces de travail Log Analytics

[!INCLUDE [monitoring-limits](../../../includes/azure-monitor-limits-workspaces.md)]

### <a name="application-insights"></a>Application Insights

[!INCLUDE [monitoring-limits](../../../includes/azure-monitor-limits-app-insights.md)]

## <a name="azure-policy-limits"></a>Limites d’Azure Policy

[!INCLUDE [policy-limits](../../../includes/azure-policy-limits.md)]

## <a name="azure-signalr-service-limits"></a>Limites d'Azure SignalR Service

[!INCLUDE [signalr-service-limits](../../../includes/signalr-service-limits.md)]

## <a name="backup-limits"></a>Limites Azure Backup

[!INCLUDE [azure-backup-limits](../../../includes/azure-backup-limits.md)]

## <a name="batch-limits"></a>Limites Azure Batch

[!INCLUDE [azure-batch-limits](../../../includes/azure-batch-limits.md)]

## <a name="classic-deployment-model-limits"></a>Limites du modèle de déploiement classique

Si vous utilisez le modèle de déploiement classique plutôt que le modèle de déploiement Azure Resource Manager, les limites suivantes s'appliquent.

[!INCLUDE [azure-subscription-limits](../../../includes/azure-subscription-limits.md)]

## <a name="container-instances-limits"></a>Limites de Container Instances

[!INCLUDE [container-instances-limits](../../../includes/container-instances-limits.md)]

## <a name="container-registry-limits"></a>Limites du registre de conteneurs

Le tableau suivant détaille les fonctionnalités et les limites des [niveaux de service](../../container-registry/container-registry-skus.md) De base, Standard et Premium.

[!INCLUDE [container-registry-limits](../../../includes/container-registry-limits.md)]

## <a name="content-delivery-network-limits"></a>Limite du réseau de distribution de contenu

[!INCLUDE [cdn-limits](../../../includes/cdn-limits.md)]

## <a name="data-factory-limits"></a>Limites de Data Factory

[!INCLUDE [azure-data-factory-limits](../../../includes/azure-data-factory-limits.md)]

## <a name="data-lake-analytics-limits"></a>Limites Data Lake Analytics

[!INCLUDE [azure-data-lake-analytics-limits](../../../includes/azure-data-lake-analytics-limits.md)]

## <a name="data-lake-storage-limits"></a>Limites de Data Lake Storage

[!INCLUDE [azure-data-lake-store-limits](../../../includes/azure-data-lake-store-limits.md)]

## <a name="data-share-limits"></a>Limites de Data Share

[!INCLUDE [azure-data-share-limits](../../../includes/azure-data-share-limits.md)]

## <a name="database-migration-service-limits"></a>Limites Database Migration Service

[!INCLUDE [database-migration-service-limits](../../../includes/database-migration-service-limits.md)]

## <a name="digital-twins-limits"></a>Limites Digital Twins

> [!NOTE]
> Certaines zones de ce service ont des limites réglables, et d’autres pas. Cela est représenté dans les tableaux ci-dessous avec la colonne *Réglable ?* . Quand la limite peut être ajustée, la valeur *Réglable ?* est *Oui*.

[!INCLUDE [digital-twins-limits](../../../includes/digital-twins-limits.md)]

## <a name="event-grid-limits"></a>Limites d’Event Grid

[!INCLUDE [event-grid-limits](../../../includes/event-grid-limits.md)]

## <a name="event-hubs-limits"></a>Limites de concentrateurs d’événements

[!INCLUDE [azure-servicebus-limits](../../../includes/event-hubs-limits.md)]

## <a name="iot-central-limits"></a>Limites d’IoT Central
[!INCLUDE [iot-central-limits](../../../includes/iot-central-limits.md)]

## <a name="iot-hub-limits"></a>Limites de hub IoT (IoT Hub)

[!INCLUDE [azure-iothub-limits](../../../includes/iot-hub-limits.md)]

## <a name="iot-hub-device-provisioning-service-limits"></a>Limites du service IoT Hub Device Provisioning

[!INCLUDE [azure-iotdps-limits](../../../includes/iot-dps-limits.md)]

## <a name="key-vault-limits"></a>Limites du coffre de clés

[!INCLUDE [key-vault-limits](../../../includes/key-vault-limits.md)]

## <a name="managed-identity-limits"></a>Limites d’identité gérée

[!INCLUDE [Managed-Identity-Limits](../../../includes/managed-identity-limits.md)]


## <a name="media-services-limits"></a>Limites de Media Services

[!INCLUDE [azure-mediaservices-limits](../../../includes/media-servieces-limits-quotas-constraints.md)]

### <a name="media-services-v2-legacy"></a>Media Services v2 (hérité)

Pour connaître les limites spécifiques à Media Services v2 (hérité), consultez [Media Services v2 (hérité)](../../media-services/previous/media-services-quotas-and-limitations.md)

## <a name="mobile-services-limits"></a>Limites Mobile Services

[!INCLUDE [mobile-services-limits](../../../includes/mobile-services-limits.md)]

## <a name="multi-factor-authentication-limits"></a>Limites de Multi-Factor Authentication

[!INCLUDE [azure-mfa-service-limits](../../../includes/azure-mfa-service-limits.md)]

## <a name="networking-limits"></a>Limites de mise en réseau

[!INCLUDE [azure-virtual-network-limits](../../../includes/azure-virtual-network-limits.md)]

### <a name="expressroute-limits"></a>Limites d’ExpressRoute

[!INCLUDE [expressroute-limits](../../../includes/expressroute-limits.md)]

### <a name="virtual-network-gateway-limits"></a>Limites de la passerelle de réseau virtuel

[!INCLUDE [virtual-network-gateway-limits](../../../includes/azure-virtual-network-gateway-limits.md)]

### <a name="nat-gateway-limits"></a>Limites de NAT Gateway

[!INCLUDE [nat-gateway-limits](../../../includes/azure-nat-gateway-limits.md)]

### <a name="virtual-wan-limits"></a>Limites de Virtual WAN

[!INCLUDE [virtual-wan-limits](../../../includes/virtual-wan-limits.md)]

### <a name="application-gateway-limits"></a>Limites d’Application Gateway

Sauf mention contraire, le tableau suivant s’applique aux références SKU v1, v2, Standard et WAF.
[!INCLUDE [application-gateway-limits](../../../includes/application-gateway-limits.md)]

### <a name="network-watcher-limits"></a>Limites de Network Watcher

[!INCLUDE [network-watcher-limits](../../../includes/network-watcher-limits.md)]

### <a name="private-link-limits"></a>Limites de Liaison privée

[!INCLUDE [private-link-limits](../../../includes/private-link-limits.md)]

## <a name="purview-limits"></a>Limites de Purview

Les valeurs les plus récentes des quotas Azure Purview se trouvent dans la page des [quotas Azure Purview](../../purview/how-to-manage-quotas.md)

### <a name="traffic-manager-limits"></a>Limites de Traffic Manager

[!INCLUDE [traffic-manager-limits](../../../includes/traffic-manager-limits.md)]

### <a name="azure-bastion-limits"></a>Limites d’Azure Bastion

[!INCLUDE [Azure Bastion limits](../../../includes/bastion-limits.md)]

### <a name="azure-dns-limits"></a>Limites d'Azure DNS

[!INCLUDE [dns-limits](../../../includes/dns-limits.md)]

### <a name="azure-firewall-limits"></a>Limites du Pare-feu Azure

[!INCLUDE [azure-firewall-limits](../../../includes/firewall-limits.md)]

### <a name="azure-front-door-service-limits"></a>Limites d’Azure Front Door Service

[!INCLUDE [azure-front-door-service-limits](../../../includes/front-door-limits.md)]

## <a name="notification-hubs-limits"></a>Limites de Notification Hubs

[!INCLUDE [notification-hub-limits](../../../includes/notification-hub-limits.md)]

## <a name="azure-role-based-access-control-limits"></a>Limites de contrôle d’accès en fonction du rôle Azure

[!INCLUDE [role-based-access-control-limits](../../../includes/role-based-access-control-limits.md)]

## <a name="service-bus-limits"></a>Limites de Service Bus

[!INCLUDE [azure-servicebus-limits](../../../includes/service-bus-quotas-table.md)]

## <a name="site-recovery-limits"></a>Limites Azure Site Recovery

[!INCLUDE [site-recovery-limits](../../../includes/site-recovery-limits.md)]

## <a name="sql-database-limits"></a>Limites de base de données SQL

Pour connaître les limites de SQL Database, consultez [Limites de ressources de SQL Database pour les bases de données uniques](../../azure-sql/database/resource-limits-vcore-single-databases.md), [Limites de ressources de SQL Database pour les pools élastiques et les bases de données mises en pool](../../azure-sql/database/resource-limits-vcore-elastic-pools.md) et [Limites de ressources de SQL Database pour SQL Managed Instance](../../azure-sql/managed-instance/resource-limits.md).

## <a name="azure-synapse-analytics-limits"></a>Limites Azure Synapse Analytics

Pour connaître les limites d’Azure Synapse Analytics, consultez les [limites des ressources Azure Synapse](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-service-capacity-limits.md).

## <a name="storage-limits"></a>Limites de stockage

<!--like # storage accts -->
[!INCLUDE [azure-storage-account-limits-standard](../../../includes/azure-storage-account-limits-standard.md)]

Pour plus d’informations sur les limites des comptes de stockage standard, voir [Objectifs d’extensibilité pour les comptes de stockage Standard](../../storage/common/scalability-targets-standard-account.md).

### <a name="storage-resource-provider-limits"></a>Limites de fournisseur de ressources de stockage

[!INCLUDE [azure-storage-limits-azure-resource-manager](../../../includes/azure-storage-limits-azure-resource-manager.md)]

### <a name="azure-blob-storage-limits"></a>Limites de Stockage Blob Azure

[!INCLUDE [storage-blob-scale-targets](../../../includes/storage-blob-scale-targets.md)]

### <a name="azure-files-limits"></a>Limite d’Azure Files

Pour plus d’informations sur les limites d'Azure Files, consultez la page [Objectifs de performance et évolutivité d'Azure Files](../../storage/files/storage-files-scale-targets.md).

[!INCLUDE [storage-files-scale-targets](../../../includes/storage-files-scale-targets.md)]

### <a name="azure-file-sync-limits"></a>Limites d’Azure File Sync

[!INCLUDE [storage-sync-files-scale-targets](../../../includes/storage-sync-files-scale-targets.md)]

### <a name="azure-queue-storage-limits"></a>Limites de Stockage File d’attente Azure

[!INCLUDE [storage-queues-scale-targets](../../../includes/storage-queues-scale-targets.md)]

### <a name="azure-table-storage-limits"></a>Limites de Stockage Table Azure

[!INCLUDE [storage-tables-scale-targets](../../../includes/storage-tables-scale-targets.md)]

<!-- conceptual info about disk limits -- applies to unmanaged and managed -->
### <a name="virtual-machine-disk-limits"></a>Limites du nombre de disques de machine virtuelle

[!INCLUDE [azure-storage-limits-vm-disks](../../../includes/azure-storage-limits-vm-disks.md)]

Pour plus d’informations, consultez [Tailles de machines virtuelles](../../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

#### <a name="disk-encryption-sets"></a>Jeux de chiffrement Azure Data Box Disk

Il y a une limite de 50 jeux de chiffrements de disques par région et par abonnement. Pour plus d’informations, consultez la documentation relative aux chiffrement pour les machines virtuelles [Linux](../../virtual-machines/disk-encryption.md#restrictions) ou [Windows](../../virtual-machines/disk-encryption.md#restrictions). Pour augmenter ce quota, contactez le support Azure.

### <a name="managed-virtual-machine-disks"></a>Disques de machines virtuelles gérées

[!INCLUDE [azure-storage-limits-vm-disks-managed](../../../includes/azure-storage-limits-vm-disks-managed.md)]

### <a name="unmanaged-virtual-machine-disks"></a>Disques de machines virtuelles non gérées

[!INCLUDE [azure-storage-limits-vm-disks-standard](../../../includes/azure-storage-limits-vm-disks-standard.md)]

[!INCLUDE [azure-storage-limits-vm-disks-premium](../../../includes/azure-storage-limits-vm-disks-premium.md)]

## <a name="storsimple-system-limits"></a>Limites du système StorSimple

[!INCLUDE [storsimple-limits-table](../../../includes/storsimple-limits-table.md)]

## <a name="stream-analytics-limits"></a>Limites Stream Analytics

[!INCLUDE [stream-analytics-limits-table](../../../includes/stream-analytics-limits-table.md)]

## <a name="virtual-machines-limits"></a>Limites de machines virtuelles

### <a name="virtual-machines-limits"></a>Limites de machines virtuelles

[!INCLUDE [azure-virtual-machines-limits](../../../includes/azure-virtual-machines-limits.md)]

### <a name="virtual-machines-limits---azure-resource-manager"></a>Limites de machines virtuelles - Azure Resource Manager

Les limites suivantes s'appliquent lorsque vous utilisez Azure Resource Manager et les groupes de ressources Azure.

[!INCLUDE [azure-virtual-machines-limits-azure-resource-manager](../../../includes/azure-virtual-machines-limits-azure-resource-manager.md)]

### <a name="shared-image-gallery-limits"></a>Limites de Shared Image Gallery

Certaines limites par abonnement ont été définies pour le déploiement de ressources à l’aide des galeries d’images partagées :

- 100 galeries d’images partagées par abonnement et par région
- 1 000 définitions d’images par abonnement et par région
- 10 000 versions d’image par abonnement et par région

## <a name="virtual-machine-scale-sets-limits"></a>Limites des groupes de machines virtuelles identiques

[!INCLUDE [virtual-machine-scale-sets-limits](../../../includes/azure-virtual-machine-scale-sets-limits.md)]

## <a name="see-also"></a>Voir aussi

* [Comprendre les limites et les augmentations Azure](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)
* [Tailles de machine virtuelle et de service cloud pour Azure](../../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Tailles pour Services cloud Azure](../../cloud-services/cloud-services-sizes-specs.md)
* [Règles de nommage et restrictions pour les ressources Azure](resource-name-rules.md)
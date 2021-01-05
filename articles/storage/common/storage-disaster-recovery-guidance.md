---
title: Reprise d’activité après sinistre et basculement de compte de stockage
titleSuffix: Azure Storage
description: Stockage Azure prend en charge le basculement de compte pour les comptes de stockage géoredondants. Avec le basculement de compte, vous pouvez lancer le processus de basculement pour votre compte de stockage si le point de terminaison principal devient indisponible.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 05/05/2020
ms.author: tamram
ms.reviewer: artek
ms.subservice: common
ms.openlocfilehash: f7d7bff1bc85e0dec78a69422d126b86f61b7704
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92783978"
---
# <a name="disaster-recovery-and-storage-account-failover"></a>Reprise d’activité après sinistre et basculement de compte de stockage

Microsoft s’efforce de faire en sorte que les services Azure soient toujours disponibles. Toutefois, des interruptions de service non planifiées peuvent se produire. Si votre application nécessite la résilience, Microsoft vous recommande d’utiliser le stockage géoredondant, afin que vos données soient copiées vers une seconde région. De plus, les clients doivent disposer d’un plan de reprise d’activité après sinistre afin de gérer toute interruption de service régionale. Une partie importante du plan de reprise d’activité consiste à préparer le basculement vers le point de terminaison secondaire, au cas où le point de terminaison principal deviendrait indisponible.

Stockage Azure prend en charge le basculement de compte pour les comptes de stockage géoredondants. Avec le basculement de compte, vous pouvez lancer le processus de basculement pour votre compte de stockage si le point de terminaison principal devient indisponible. Le basculement met à jour le point de terminaison secondaire pour qu’il devienne le point de terminaison principal pour votre compte de stockage. Une fois le basculement terminé, les clients peuvent commencer à écrire dans le nouveau point de terminaison principal.

Le basculement de compte est disponible pour les types de comptes GPv1, GPv2 et de stockage d’objets BLOB avec des déploiements d’Azure Resource Manager. Le basculement de compte est pris en charge pour toutes les régions publiques, mais n’est pas disponible dans les clouds souverains ou nationaux à l’heure actuelle.

Cet article décrit les concepts et les processus impliqués dans un basculement de compte, et explique comment préparer votre compte de stockage pour la reprise avec le moins d’impact possible sur le client. Pour découvrir comment lancer un basculement de compte dans le Portail Azure ou PowerShell, consultez [Lancer un basculement de compte](storage-initiate-account-failover.md).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="choose-the-right-redundancy-option"></a>Choisir l’option de redondance appropriée

Stockage Azure conserve plusieurs copies de votre compte de stockage afin de garantir la durabilité et la haute disponibilité. L’option de redondance que vous choisirez pour votre compte dépendra du degré de résilience dont vous avez besoin. Pour la protection contre les pannes régionales, configurez votre compte pour le stockage géoredondant, avec ou sans l’option d’accès en lecture à partir de la région secondaire :  

Le **stockage géoredondant (GRS) ou stockage géoredondant interzone (GZRS)** copie vos données de façon asynchrone dans deux régions géographiques distantes d’au moins plusieurs centaines de kilomètres. Si la région primaire subit une panne, la région secondaire sert de source redondante pour vos données. Vous pouvez lancer un basculement pour transformer le point de terminaison secondaire en point de terminaison principal.

Le **stockage géoredondant avec accès en lecture (RA-GRS) ou stockage géoredondant interzone avec accès en lecture (RA-GZRS)** offre un stockage géoredondant avec en plus un accès en lecture au point de terminaison secondaire. Si une panne se produit au niveau du point de terminaison principal, les applications configurées pour l’accès en lecture à la région secondaire et conçues pour la haute disponibilité peuvent continuer à lire à partir du point de terminaison secondaire. Microsoft recommande RA-GZRS pour une disponibilité et une durabilité maximales pour vos applications.

Pour plus d’informations sur la redondance dans Stockage Azure, consultez [Redondance de Stockage Azure](storage-redundancy.md).

> [!WARNING]
> Le stockage géoredondant comporte un risque de perte de données. Les données sont copiées vers la région secondaire de façon asynchrone, ce qui signifie qu’il existe un délai entre l’écriture des données dans la région primaire et dans la région secondaire. En cas de panne, les opérations d’écriture sur le point de terminaison principal qui n’ont pas encore été copiées vers le point de terminaison secondaire seront perdues.

## <a name="design-for-high-availability"></a>Concevoir pour la haute disponibilité

Il est important de concevoir votre application à des fins de haute disponibilité dès le départ. Pour obtenir des conseils sur la conception de votre application et la planification de la reprise d’activité, consultez ces ressources Azure :

- [Conception d’applications résilientes pour Azure](/azure/architecture/framework/resiliency/app-design) : vue d’ensemble des concepts clés de l’architecture des applications hautement disponibles dans Azure.
- [Liste de vérification de résilience](/azure/architecture/checklist/resiliency-per-service) : liste de contrôle pour vérifier que votre application implémente les bonnes pratiques de conception pour la haute disponibilité.
- [Utilisez la géo-redondance pour concevoir des applications hautement disponibles](geo-redundant-design.md) : guide de conception pour créer des applications tirant parti du stockage géoredondant.
- [Tutoriel : Générer une application hautement disponible avec le stockage d’objets Blob](../blobs/storage-create-geo-redundant-storage.md) : tutoriel qui montre comment créer une application hautement disponible qui bascule automatiquement entre des points de terminaison lors de la simulation de pannes et de récupérations. 

Gardez également à l’esprit ces bonnes pratiques pour maintenir une haute disponibilité pour vos données de Stockage Azure :

- **Disques :** utilisez [Sauvegarde Azure](https://azure.microsoft.com/services/backup/) pour sauvegarder les disques de machine virtuelle utilisés par vos machines virtuelles Azure. Vous pouvez aussi utiliser [Azure Site Recovery](https://azure.microsoft.com/services/site-recovery/) pour protéger vos machines virtuelles en cas de sinistre régional.
- **Objets blob de blocs :** activez la [suppression réversible](../blobs/soft-delete-blob-overview.md) pour protéger contre les suppressions et remplacements au niveau objet, ou copiez les objets blob de blocs vers un autre compte de stockage dans une région différente à l’aide d’ [AzCopy](./storage-use-azcopy-v10.md), d’ [Azure PowerShell](/powershell/module/az.storage/) ou de la [bibliothèque de déplacement de données Azure](storage-use-data-movement-library.md).
- **Fichiers :** utilisez [AzCopy](./storage-use-azcopy-v10.md) ou [Azure PowerShell](/powershell/module/az.storage/) pour copier vos fichiers vers un autre compte de stockage dans une région différente.
- [Tables :](./storage-use-azcopy-v10.md) utilisez **AzCopy** pour exporter les données de table vers un autre compte de stockage dans une région différente.

## <a name="track-outages"></a>Effectuer le suivi des pannes

Les clients peuvent s’abonner au [tableau de bord Azure Service Health](https://azure.microsoft.com/status/) pour effectuer le suivi de l’intégrité et de l’état de Stockage Azure et d’autres services Azure.

Microsoft vous recommande également de concevoir votre application pour l’éventualité d’échecs d’écriture. Votre application doit exposer les échecs d’écriture d’une manière qui vous avertit de la possibilité d’une panne dans la région primaire.

## <a name="understand-the-account-failover-process"></a>Comprendre le processus de basculement de compte

Le basculement de compte géré par le client vous permet de basculer votre compte de stockage entier vers la région secondaire si la région primaire devient indisponible pour une raison quelconque. Quand vous forcez un basculement vers la région secondaire, les clients peuvent commencer à écrire des données sur le point de terminaison secondaire une fois le basculement terminé. Le basculement prend généralement environ une heure.

### <a name="how-an-account-failover-works"></a>Fonctionnement d’un basculement de compte

Dans des circonstances normales, un client écrit des données dans un compte Stockage Azure dans la région primaire, et ces données sont copiées de manière asynchrone vers la région secondaire. L’illustration suivante montre le scénario quand la région primaire est disponible :

![Les clients écrivent des données dans le compte de stockage dans la région primaire](media/storage-disaster-recovery-guidance/primary-available.png)

Si le point de terminaison principal devient indisponible pour une raison quelconque, le client ne peut plus écrire dans le compte de stockage. L’illustration suivante montre le scénario dans lequel la région primaire n’est plus disponible, mais aucune reprise n’a encore été effectuée :

![La région primaire n’étant pas disponible, les clients ne peuvent pas écrire de données](media/storage-disaster-recovery-guidance/primary-unavailable-before-failover.png)

Le client lance le basculement de compte vers le point de terminaison secondaire. Le processus de basculement met à jour l’entrée DNS fournie par Stockage Azure afin que le point de terminaison secondaire devienne le nouveau point de terminaison principal pour votre compte de stockage, comme illustré dans l’image suivante :

![Le client lance le basculement de compte vers le point de terminaison secondaire](media/storage-disaster-recovery-guidance/failover-to-secondary.png)

L’accès en écriture est restauré pour les comptes géoredondants une fois que l’entrée DNS a été mise à jour et que les requêtes sont dirigées vers le nouveau point de terminaison principal. Les points de terminaison de service de stockage existants pour les objets blob, les tables, les files d’attente et les fichiers restent les mêmes après le basculement.

> [!IMPORTANT]
> Une fois le basculement terminé, le compte de stockage est configuré pour être localement redondant dans le nouveau point de terminaison principal. Pour reprendre la réplication vers la nouvelle région secondaire, configurez à nouveau le compte pour la géoredondance.
>
> N’oubliez pas que la conversion d’un compte LRS pour utiliser la géo-redondance entraîne un coût. Ce coût s’applique à la mise à jour du compte de stockage dans la nouvelle région primaire après un basculement.  

### <a name="anticipate-data-loss"></a>Anticiper la perte de données

> [!CAUTION]
> Un basculement de compte entraîne généralement une certaine perte de données. Il est important de bien comprendre les implications d’un basculement de compte.  

Les données étant écrites de façon asynchrone de la région primaire vers la région secondaire, il y a toujours un délai avant qu’une écriture dans la région primaire soit copiée vers la région secondaire. Si la région primaire devient indisponible, il se peut que les écritures les plus récentes n’aient pas encore été copiées vers la région secondaire.

Quand vous forcez un basculement, toutes les données dans la région primaire sont perdues car la région secondaire devient la nouvelle région primaire et le compte de stockage est configuré pour être localement redondant. Toutes les données déjà copiées vers la région secondaire sont conservées quand le basculement se produit. En revanche, les données écrites dans la région primaire mais qui n’ont pas encore été copiées vers la région secondaire sont définitivement perdues.

La propriété **Dernière heure de synchronisation** indique l’heure la plus récente à laquelle il est garanti que les données de la région primaire ont été écrites dans la région secondaire. Toutes les données écrites avant la dernière heure de synchronisation sont disponibles dans la région secondaire. Quant aux données écrites après la dernière heure de synchronisation, il y a un risque qu’elles n’aient pas été écrites dans la région secondaire et qu’elles soient perdues. Utilisez cette propriété en cas de panne pour estimer la perte de données que peut entraîner un basculement de compte.

En guise de bonne pratique, concevez votre application afin de pouvoir utiliser la dernière heure de synchronisation pour évaluer la perte de données attendue. Par exemple, si vous enregistrez dans le journal toutes les opérations d’écriture, vous pouvez comparer l’heure de vos dernières opérations d’écriture à la dernière heure de synchronisation pour identifier les écritures qui n’ont pas été synchronisées dans la région secondaire.

Pour plus d’informations sur la vérification de la propriété **Heure de la dernière synchronisation** , consultez [Vérification de la propriété Heure de la dernière synchronisation d’un compte de stockage](last-sync-time-get.md).

### <a name="use-caution-when-failing-back-to-the-original-primary"></a>Faire attention lors de la restauration automatique vers la région primaire

Après le basculement de la région primaire vers la région secondaire, votre compte de stockage est configuré pour être localement redondant dans la nouvelle région primaire. Vous pouvez ensuite configurer à nouveau le compte pour la géo-redondance. Quand le compte est reconfiguré pour la géoredondance après un basculement, la nouvelle région primaire commence immédiatement la copie des données vers la nouvelle région secondaire, qui était la région primaire avant le basculement d’origine. Toutefois, il peut s’écouler un certain temps avant que les données existantes dans la région primaire soient entièrement copiées vers la nouvelle région secondaire.

Une fois le compte de stockage reconfiguré pour la géoredondance, il est possible de lancer un autre basculement de la nouvelle région primaire vers la nouvelle région secondaire. Dans ce cas, la région primaire d’origine avant le basculement redevient la région primaire, et est configurée pour être localement redondante. Toutes les données dans la région primaire post-basculement (la région secondaire d’origine) sont alors perdues. Si la plupart des données dans le compte de stockage n’ont pas été copiées vers la nouvelle région secondaire avant la restauration automatique, vous risquez de subir une perte de données majeure.

Pour éviter toute perte de données majeure, vérifiez la valeur de la propriété **Dernière heure de synchronisation** avant de procéder à la restauration automatique. Comparez la dernière heure de synchronisation aux dernières heures où ces données ont été écrites dans la nouvelle région primaire afin d’évaluer la perte de données attendue. 

## <a name="initiate-an-account-failover"></a>Initier un basculement de compte

Vous pouvez lancer un basculement de compte à partir du portail Azure, de PowerShell, d’Azure CLI ou de l’API du fournisseur de ressources Stockage Azure. Pour plus d’informations sur la façon de lancer un basculement, consultez [Lancer un basculement de compte](storage-initiate-account-failover.md).

## <a name="additional-considerations"></a>Considérations supplémentaires

Passez en revue les considérations supplémentaires décrites dans cette section pour comprendre comment vos applications et services peuvent être affectés quand vous forcez un basculement.

### <a name="storage-account-containing-archived-blobs"></a>Compte de stockage contenant des objets blob archivés

Les comptes de stockage contenant des objets blob archivés peuvent faire l’objet d’un basculement. Une fois le basculement terminé, tous les objets BLOB archivés doivent être réalimentés à un niveau en ligne pour que le compte puisse être configuré pour la géo-redondance.

### <a name="storage-resource-provider"></a>Fournisseur de ressources de stockage

Une fois le basculement terminé, les clients peuvent à nouveau lire et écrire des données Stockage Azure dans la nouvelle région primaire. Toutefois, le fournisseur de ressources Stockage Azure ne bascule pas ; les opérations de gestion des ressources doivent donc toujours avoir lieu dans la région primaire. Si la région primaire n’est pas disponible, vous ne pourrez pas effectuer d’opérations de gestion sur le compte de stockage.

Étant donné que le fournisseur de ressources Stockage Azure ne bascule pas, la propriété [Emplacement](/dotnet/api/microsoft.azure.management.storage.models.trackedresource.location) retourne l’emplacement primaire d’origine une fois le basculement terminé.

### <a name="azure-virtual-machines"></a>Machines virtuelles Azure

Les machines virtuelles Azure ne basculent pas dans le cadre d’un basculement de compte. Si la région primaire devient indisponible et que vous basculez vers la région secondaire, vous devez recréer toutes les machines virtuelles après le basculement. En outre, il existe un risque de perte de données lié au basculement de compte. Microsoft vous recommande de suivre les recommandations suivantes en matière de [haute disponibilité](../../virtual-machines/manage-availability.md) et de [reprise d’activité après sinistre](../../virtual-machines/backup-recovery.md) propres aux machines virtuelles dans Azure.

### <a name="azure-unmanaged-disks"></a>Disques non managés Azure

En guise de bonne pratique, Microsoft vous recommande de convertir les disques non managés en disques managés. Toutefois, si vous avez besoin d’effectuer le basculement d’un compte qui contient des disques non managés attachés à des machines virtuelles Azure, vous devez arrêter la machine virtuelle avant de lancer le basculement.

Les disques non managés sont stockés en tant qu’objets blob de pages dans Stockage Azure. Quand une machine virtuelle s’exécute dans Azure, les disques non managés attachés à la machine virtuelle sont loués. Un basculement de compte ne peut pas continuer quand il existe un bail sur un objet blob. Pour effectuer le basculement, effectuez les étapes suivantes :

1. Avant de commencer, notez les noms de tous les disques non managés, leurs numéros d’unité logique (LUN) et la machine virtuelle à laquelle ils sont attachés. Cela facilitera le réattachement des disques après le basculement.
2. Arrêtez la machine virtuelle.
3. Supprimez la machine virtuelle, mais conservez les fichiers de disque dur virtuel pour les disques non managés. Notez l’heure à laquelle vous avez supprimé la machine virtuelle.
4. Attendez que la **Dernière heure de synchronisation** ait été mise à jour et soit postérieure à l’heure à laquelle vous avez supprimé la machine virtuelle. Cette étape est importante, car si le point de terminaison secondaire n’a pas été totalement mis à jour avec les fichiers de disques durs virtuels quand le basculement se produit, la machine virtuelle risque de ne pas fonctionner correctement dans la nouvelle région primaire.
5. Lancez le basculement de compte.
6. Attendez que le basculement de compte soit terminé et que la région secondaire soit devenue la nouvelle région primaire.
7. Créez une machine virtuelle dans la nouvelle région primaire et réattachez les disques durs virtuels.
8. Démarrez la nouvelle machine virtuelle.

N’oubliez pas que toutes les données stockées dans un disque temporaire sont perdues quand la machine virtuelle est arrêtée.

## <a name="unsupported-features-and-services"></a>Fonctionnalités et services non pris en charge

Les fonctionnalités et services suivants ne sont pas pris en charge pour le basculement de compte :

- Azure File Sync ne prend pas en charge le basculement de compte de stockage. Les comptes de stockage contenant des partages de fichiers Azure utilisés en tant que points de terminaison cloud dans Azure File Sync ne doivent pas être basculés. Cela provoquera en effet un arrêt de la synchronisation et pourra entraîner une perte inattendue de données dans le cas de fichiers nouvellement hiérarchisés.
- Les comptes de stockage ADLS Gen2 (comptes dans lesquels est activé un espace de noms hiérarchique) ne sont pas pris en charge pour l’instant.
- Un compte de stockage contenant des objets blob de blocs premium ne peut pas être basculé. Les comptes de stockage qui prennent en charge les objets blob de blocs premium ne prennent pas en charge la géoredondance.
- Un compte de stockage incluant des conteneurs avec une [stratégie d’immuabilité WORM](../blobs/storage-blob-immutable-storage.md) ne peut pas être basculé. Les stratégies de rétention temporelles ou les stratégies de conservation légale déverrouillées/verrouillées empêchent le basculement afin de maintenir la conformité.

## <a name="copying-data-as-an-alternative-to-failover"></a>Copie de données comme alternative au basculement

Si votre compte de stockage est configuré pour l’accès en lecture à la région secondaire, vous pouvez concevoir votre application pour la lecture à partir du point de terminaison secondaire. Si vous préférez ne pas effectuer de basculement en cas de panne dans la région primaire, vous pouvez utiliser des outils tels que [AzCopy](./storage-use-azcopy-v10.md), [Azure PowerShell](/powershell/module/az.storage/) ou la [bibliothèque de déplacement de données Azure](../common/storage-use-data-movement-library.md) pour copier des données de votre compte de stockage dans la région secondaire vers un autre compte de stockage dans une région non affectée. Vous pouvez ensuite faire en sorte que vos applications pointent vers ce compte de stockage pour la disponibilité en lecture et en écriture.

> [!CAUTION]
> Un basculement de compte ne doit pas être utilisé dans le cadre de votre stratégie de migration des données.

## <a name="microsoft-managed-failover"></a>Basculement géré par Microsoft

Dans des circonstances extrêmes où une région est perdue suite à un sinistre majeur, Microsoft peut lancer un basculement régional. Dans ce cas, aucune action n’est requise de votre part. Tant que le basculement géré par Microsoft ne sera pas terminé, vous n’aurez pas d’accès en écriture à votre compte de stockage. Vos applications peuvent lire à partir de la région secondaire si votre compte de stockage est configuré pour RA-GRS ou RA-GZRS.

## <a name="see-also"></a>Voir aussi

- [Utilisez la géo-redondance pour concevoir des applications hautement disponibles](geo-redundant-design.md)
- [Initier un basculement de compte](storage-initiate-account-failover.md)
- [Vérifier la propriété Heure de la dernière synchronisation pour un compte de stockage](last-sync-time-get.md)
- [Tutoriel : Générer une application hautement disponible avec le stockage Blob](../blobs/storage-create-geo-redundant-storage.md)
---
title: Choisir des tailles et des images de machines virtuelles pour les pools
description: Quelle taille de machine virtuelle et quelle version de système d’exploitation choisir parmi celles disponibles pour les nœuds de calcul dans des pools Azure Batch
ms.topic: conceptual
ms.date: 11/24/2020
ms.custom: seodec18
ms.openlocfilehash: 8bb54a4db62f56f442f7cec81e6768241a05ffee
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95895228"
---
# <a name="choose-a-vm-size-and-image-for-compute-nodes-in-an-azure-batch-pool"></a>Choisir une taille et une image de machine virtuelle pour des nœuds de calcul dans un pool Azure Batch

Lorsque vous sélectionnez une taille de nœud pour un pool Azure Batch, vous avez le choix entre quasiment toutes les tailles de machine virtuelle disponibles dans Azure. Azure propose une gamme de tailles pour les machines virtuelles Windows et Linux pour différentes charges de travail.

## <a name="supported-vm-series-and-sizes"></a>Tailles et séries de machine virtuelle prises en charge

Quelques exceptions et limites s’appliquent quant au choix d’une taille de machine virtuelle pour votre pool Batch :

- Certaines séries ou tailles de machine virtuelle ne sont pas prises en charge dans Batch.
- Certaines tailles de machine virtuelle sont limitées et doivent être activées explicitement avant de pouvoir être allouées.

### <a name="pools-in-virtual-machine-configuration"></a>Pools dans la configuration de la machine virtuelle

Les pools Batch dans la configuration de la machine virtuelle prennent en charge presque toutes les [tailles de machine virtuelle](../virtual-machines/sizes.md). Consultez le tableau suivant pour en savoir plus sur les tailles prises en charge et les restrictions.

| Série de la machine virtuelle  | Tailles prises en charge |
|------------|---------|
| De base A | Toutes les tailles *sauf* Basic_A0 (A0) |
| Un | Toutes les tailles *sauf* Standard_A0 |
| Av2 | Toutes les tailles |
| B | Non pris en charge |
| DC | Non pris en charge |
| Dv2, DSv2 | Toutes les tailles |
| Dv3, Dsv3 | Toutes les tailles |
| Dav4<sup>1</sup> | Toutes les tailles |
| Dasv4<sup>1</sup> | Toutes les tailles |
| Ddv4, Ddsv4 |  Toutes les tailles |
| Dv4, Dsv4 | Non pris en charge |
| Ev3, Esv3 | Toutes les tailles, à l’exception de E64is_v3 |
| Eav4<sup>1</sup> | Toutes les tailles |
| Easv4<sup>1</sup> | Toutes les tailles |
| Edv4, Edsv4 |  Toutes les tailles |
| Ev4, Esv4 | Non pris en charge |
| F, Fs | Toutes les tailles |
| Fsv2 | Toutes les tailles |
| G, Gs | Toutes les tailles |
| H | Toutes les tailles |
| HB<sup>1</sup> | Toutes les tailles |
| HBv2<sup>1</sup> | Toutes les tailles |
| HC<sup>1</sup> | Toutes les tailles |
| Ls | Toutes les tailles |
| Lsv2<sup>1</sup> | Toutes les tailles |
| M<sup>1</sup> | Toutes les tailles |
| Mv2<sup>1, 2</sup> | Toutes les tailles |
| NC | Toutes les tailles |
| NCv2<sup>1</sup> | Toutes les tailles |
| NCv3<sup>1</sup> | Toutes les tailles |
| NCasT4_v3 | Aucune – pas encore disponible |
| ND<sup>1</sup> | Toutes les tailles |
| NDv2<sup>1</sup> | Aucune – pas encore disponible |
| NV | Toutes les tailles |
| NVv3<sup>1</sup> | Toutes les tailles |
| NVv4<sup>1</sup> | Toutes les tailles |
| SAP HANA | Non pris en charge |

<sup>1</sup> Ces séries de machines virtuelles peuvent être allouées dans des pools Batch dans la configuration de la machine virtuelle, mais vous devez créer un compte Batch et demander une [augmentation de quota](batch-quota-limit.md#increase-a-quota) spécifique. Cette limitation sera supprimée une fois que le quota de processeurs virtuels par série de machines virtuelles sera entièrement pris en charge pour les comptes Batch.

<sup>2</sup> Ces séries de machines virtuelles peuvent uniquement être utilisées avec les images de machine virtuelle de 2e génération.

### <a name="using-generation-2-vm-images"></a>Utilisation d’images de machine virtuelle de 2e génération

Certaines séries de machines virtuelles, telles que [Mv2](../virtual-machines/mv2-series.md), peuvent être utilisées uniquement avec des [images de machine virtuelle de 2e génération](../virtual-machines/generation-2.md). Les images de machine virtuelle de 2e génération sont spécifiées comme n’importe quelle image de machine virtuelle à l’aide de la propriété « SKU » de la configuration [« imageReference »](/rest/api/batchservice/pool/add#imagereference) ; les chaînes « SKU » ont un suffixe tel que « -g2 » ou « -gen2 ». Pour obtenir la liste des images de machines virtuelles prises en charge par Batch, notamment les images de 2e génération, utilisez l’API [« Répertorier les images prises en charge »](/rest/api/batchservice/account/listsupportedimages), [PowerShell](/powershell/module/az.batch/get-azbatchsupportedimage) ou [Azure CLI](/cli/azure/batch/pool/supported-images).

### <a name="pools-in-cloud-service-configuration"></a>Pools dans la configuration de service cloud

Les pools Batch dans la configuration de service cloud prennent en charge toutes les [tailles de machine virtuelle pour Services cloud](../cloud-services/cloud-services-sizes-specs.md), **sauf** les suivantes :

| Série de la machine virtuelle  | Tailles non prises en charge |
|------------|-------------------|
| Série A   | Très petite       |
| Série Av2 | Standard_A1_v2, Standard_A2_v2, Standard_A2m_v2 |

## <a name="size-considerations"></a>Considérations en matière de taille

- **Exigences pour l’application** : tenez compte des caractéristiques et des exigences de l’application que vous allez exécuter sur les nœuds. Des aspects tels que la nature multithread de l’application et le volume de mémoire utilisé vous aideront à déterminer la taille de nœud la mieux adaptée et la plus rentable. Pour les applications CUDA ou [charges de travail MPI](batch-mpi.md) avec plusieurs instances, pensez aux tailles de machine virtuelles [compatibles GPU](../virtual-machines/sizes-hpc.md) ou [HPC](../virtual-machines/sizes-gpu.md), respectivement. Pour plus d’informations, consultez [Utiliser des instances compatibles RDMA ou GPU dans les pools Batch](batch-pool-compute-intensive-sizes.md).

- **Tâches par nœud** : la taille du nœud est souvent sélectionnée en supposant qu’une tâche s’exécute sur un nœud à la fois. Cependant, plusieurs tâches (et par conséquent, plusieurs instances d’application) peuvent [s’exécuter en parallèle](batch-parallel-node-tasks.md) sur les nœuds de calcul lors de l’exécution du travail. Dans ce cas, il est courant de choisir une taille de nœud multicœur pour prendre en charge la demande accrue de l’exécution parallèle des tâches.

- **Charger des niveaux pour différentes tâches** : tous les nœuds dans un pool ont la même taille. Si vous prévoyez d’exécuter des applications dont la configuration système requise et/ou les niveaux de charge diffèrent, nous vous recommandons d’utiliser des pools distincts.

- **Disponibilité de la région** : une série ou une taille de machines virtuelles peut ne pas être disponible dans les régions où vous créez vos comptes Batch. Pour vérifier qu’une taille est disponible, consultez [Disponibilité des produits par région](https://azure.microsoft.com/regions/services/).

- **Quotas** : les [quotas de cœurs](batch-quota-limit.md#resource-quotas) dans votre compte Batch peuvent limiter le nombre de nœuds d’une taille donnée que vous pouvez ajouter à un pool Batch. Si nécessaire, vous pouvez [demander une augmentation du quota](batch-quota-limit.md#increase-a-quota).

- **Configuration du pool** : en général, vous avez plusieurs options de taille de machine virtuelle lorsque vous créez un pool dans la configuration de machine virtuelle, en comparaison avec la configuration de service cloud.

## <a name="supported-vm-images"></a>Images de machine virtuelle prises en charge

Utilisez l’une des API suivantes pour renvoyer une liste d’images de machine virtuelle Windows et Linux actuellement prises en charge par Batch, y compris les ID de référence SKU des agents de nœuds pour chaque image :

- API REST du service Batch : [List Supported Images](/rest/api/batchservice/account/listsupportedimages)
- PowerShell : [Get-AzBatchSupportedImage](/powershell/module/az.batch/get-azbatchsupportedimage)
- Azure CLI : [az batch pool supported-images](/cli/azure/batch/pool/supported-images)

## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur le [workflow et les ressources principales du service Batch](batch-service-workflow-features.md) telles que les pools, les nœuds, les travaux et les tâches.
- Pour plus d’informations sur l’utilisation de tailles de machine virtuelle nécessitant beaucoup de ressources système, consultez [Utiliser des instances compatibles RDMA ou GPU dans les pools Batch](batch-pool-compute-intensive-sizes.md).

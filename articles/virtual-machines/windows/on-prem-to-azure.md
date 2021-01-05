---
title: Migrer à partir d’AWS et d’autres plateformes vers la fonctionnalité Disques managés dans Azure
description: Créez des machines virtuelles dans Azure à l’aide de disques durs virtuels chargés à partir d’autres clouds comme AWS ou d’autres plateformes de virtualisation, et tirez parti d’Azure Disques managés.
author: roygara
manager: twooley
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: conceptual
ms.date: 10/07/2017
ms.author: rogarana
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 32cbfbcc8feeff66101ab5e2c95f476a4a4215e9
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91973901"
---
# <a name="migrate-from-amazon-web-services-aws-and-other-platforms-to-managed-disks-in-azure"></a>Migrer à partir d’Amazon Web Services (AWS) et d’autres plateformes vers la fonctionnalité Disques managés dans Azure

Vous pouvez charger des fichiers VHD sur Azure à partir d’AWS ou de solutions de virtualisation locales afin de créer des machines virtuelles qui tirent parti de la fonctionnalité Disques managés. Avec Azure Disques managés, il n’est plus nécessaire de gérer des comptes de stockage pour les machines virtuelles IaaS Azure. Vous devez seulement spécifier le type (Premium ou Standard) et la taille du disque dont vous avez besoin, et Azure crée et gère le disque pour vous. 

Vous pouvez charger des disques durs virtuels généralisés et spécialisés. 
- **Disque dur virtuel généralisé** : supprime toutes les informations de votre compte personnel avec Sysprep. 
- **Disque dur virtuel spécialisé** : gère les comptes d’utilisateur, les applications et les autres données d’état à partir de votre machine virtuelle d’origine. 

> [!IMPORTANT]
> Avant de charger un disque dur virtuel dans Azure, vous devez apprendre à [préparer un disque dur virtuel Windows ou un disque VHDX à charger sur Azure](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
>
>


| Scénario                                                                                                                         | Documentation                                                                                                                       |
|----------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| Vous avez une instance AWS EC2 existante que vous voulez migrer vers des machines virtuelles Azure à l’aide de disques managés.                              | [Migrer une machine virtuelle d’Amazon Web Services (AWS) vers Azure](aws-to-azure.md)                           |
| Vous avez une machine virtuelle sur une autre plateforme de virtualisation que vous aimeriez utiliser en tant qu’image pour créer plusieurs machines virtuelles Azure. | [Charger un disque dur virtuel généralisé et l’utiliser pour créer une machine virtuelle dans Azure](upload-generalized-managed.md) |
| Vous avez une machine virtuelle unique personnalisée que vous souhaitez recréer dans Azure.                                                      | [Charger un disque dur virtuel spécialisé dans Azure et créer une machine virtuelle](create-vm-specialized.md)         |


## <a name="overview-of-managed-disks"></a>Vue d’ensemble de la fonctionnalité Disques managés

Azure Disques managés simplifie la gestion des machines virtuelles en éliminant la nécessité de gérer des comptes de stockage. La fonctionnalité Disques managés bénéficie également d’une meilleure fiabilité des machines virtuelles dans un groupe à haute disponibilité. Cela permet de s’assurer que les disques des différentes machines virtuelles d’un groupe à haute disponibilité sont suffisamment isolés les uns des autres pour éviter un point de défaillance unique. Les disques des différentes machines virtuelles d’un groupe à haute disponibilité sont automatiquement placés dans des unités d’échelle (tampons) de stockage distinctes, ce qui limite l’impact des défaillances d’unités d’échelle de stockage uniques dues à des défaillances matérielles et logicielles.
Selon vos besoins, vous avez le choix entre quatre types d’options de stockage. Pour en savoir plus sur les types de disques disponibles, consultez l'article [Sélectionner un type de disque](../disks-types.md).

## <a name="plan-for-the-migration-to-managed-disks"></a>Planification de la migration vers la fonctionnalité Disques managés

Cette section vous aide à prendre la meilleure décision concernant les types de machines virtuelles et de disques.

Si vous envisagez de migrer à partir de disques non managés vers des disques managés, prenez en compte le fait que les utilisateurs avec le rôle [Contributeur de machine virtuelle](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) ne sera pas en mesure de modifier la taille de la machine virtuelle (comme avant la conversion). Cela s’explique par le fait que les machines virtuelles avec disques managés obligent l’utilisateur à détenir une autorisation Microsoft.Compute/disks/write sur les disques du système d’exploitation.

### <a name="location"></a>Location

Choisissez un emplacement où Azure Disques managés est disponible. Si vous effectuez une migration vers la fonctionnalité Disques managés Premium, assurez-vous également que le stockage Premium est disponible dans la région où vous prévoyez la migration. Pour obtenir des informations à jour sur les emplacements disponibles, consultez [Services Azure par région](https://azure.microsoft.com/regions/#services) .

### <a name="vm-sizes"></a>Tailles de machine virtuelle

Si vous effectuez une migration vers la fonctionnalité Disques managés Premium, vous devez mettre à jour la taille de la machine virtuelle par rapport à la capacité de stockage Premium disponible dans la région où se trouve la machine virtuelle. Passez en revue les tailles de machine virtuelle compatibles avec le stockage Premium. Les spécifications des tailles des machines virtuelles Azure sont répertoriées dans la section [Tailles des machines virtuelles](../sizes.md).
Passez en revue les caractéristiques de performances des machines virtuelles fonctionnant avec Premium Storage et choisissez la taille de machine virtuelle la mieux adaptée à votre charge de travail. Assurez-vous que la bande passante disponible est suffisante sur votre machine virtuelle pour gérer le trafic du disque.

### <a name="disk-sizes"></a>Tailles du disque

**Disques managés Premium**

Il existe sept types de disques managés Premium qui peuvent être utilisés avec votre machine virtuelle, chacun d’eux présentant des limites d’E/S par seconde et de débit spécifiques. Prenez en compte ces limites lors de la sélection du type de disque Premium pour votre machine virtuelle en fonction des besoins en capacité, en performances, en extensibilité et en charges maximales de votre application.

| Type de disque Premium  | P4    | P6    | P10   | P15   | P20   | P30   | P40   | P50   | 
|---------------------|-------|-------|-------|-------|-------|-------|-------|-------|
| Taille du disque           | 32 Go| 64 Go| 128 Go| 256 Go|512 Go | 1024 Go (1 To)    | 2 048 Go (2 To)    | 4 095 Go (4 To)    | 
| IOPS par disque       | 120   | 240   | 500   | 1100  |2300              | 5 000              | 7500              | 7500              | 
| Débit par disque | 25 Mo par seconde  | 50 Mo par seconde  | 100 Mo par seconde | 125 Mo par seconde |150 Mo par seconde | 200 Mo par seconde | 250 Mo par seconde | 250 Mo par seconde |

**Disques managés Standard**

Il existe sept types de disques managés Standard qui peuvent être utilisés avec votre machine virtuelle. Chacun d’eux dispose d’une capacité différente, mais ils partagent les mêmes limites d’E/S par seconde et de débit. Choisissez le type de disque managé Standard selon les besoins en capacité de votre application.

| Type de disque Standard  | S4               | S6               | S10              | S15              | S20              | S30              | S40              | S50              | 
|---------------------|------------------|------------------|------------------|------------------|------------------|------------------|------------------|------------------| 
| Taille du disque           | 30 Go            | 64 Go            | 128 Go           | 256 Go           |512 Go           | 1024 Go (1 To)   | 2 048 Go (2 To)    | 4 095 Go (4 To)   | 
| IOPS par disque       | 500              | 500              | 500              | 500              |500              | 500              | 500             | 500              | 
| Débit par disque | 60 Mo par seconde | 60 Mo par seconde | 60 Mo par seconde | 60 Mo par seconde |60 Mo par seconde | 60 Mo par seconde | 60 Mo par seconde | 60 Mo par seconde | 

### <a name="disk-caching-policy"></a>Stratégie de mise en cache du disque 

**Disques managés Premium**

Par défaut, la stratégie de mise en cache est *Lecture seule* pour tous les disques de données Premium et *Lecture-écriture* pour le disque du système d’exploitation Premium attaché à la machine virtuelle. Ce paramètre de configuration est recommandé pour optimiser les performances des E/S de votre application. Pour les disques de données en écriture seule ou avec d'importantes opérations d'écriture (par ex., les fichiers journaux de SQL Server), désactivez la mise en cache du disque pour de meilleures performances de l'application.

### <a name="pricing"></a>Tarifs

Consultez la [tarification de la fonctionnalité Disques managés](https://azure.microsoft.com/pricing/details/managed-disks/). La tarification des disques managés Premium est identique à celle des disques non managés Premium. En revanche, la tarification de la fonctionnalité Disques managés Standard est différente de celle des disques non managés Standard.


## <a name="next-steps"></a>Étapes suivantes

- Avant de charger un disque dur virtuel dans Azure, vous devez apprendre à [préparer un disque dur virtuel Windows ou un disque VHDX à charger sur Azure](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
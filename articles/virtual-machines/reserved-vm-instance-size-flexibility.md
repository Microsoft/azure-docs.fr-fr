---
title: Flexibilité des tailles de machines virtuelles – Azure Reserved VM Instances
description: Découvrez les gammes de tailles auxquelles s’applique une remise de réservation lorsque vous achetez une instance de machine virtuelle réservée.
author: manish-shukla01
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 08/03/2018
ms.author: manshuk
ms.openlocfilehash: 381023a0650b078a74c01ff7b698aebacb256147
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88855504"
---
# <a name="virtual-machine-size-flexibility-with-reserved-vm-instances"></a>Flexibilité en termes de taille de machine virtuelle avec des instances de machines virtuelles réservées

Quand vous achetez une instance de machine virtuelle réservée, vous pouvez choisir d’optimiser la flexibilité de la taille ou la priorité de capacité de l’instance. Pour plus d’informations sur la définition ou la modification du paramètre d’optimisation pour des instances de machine virtuelle réservées, consultez [Modifier le paramètre d’optimisation pour des instances de machine virtuelle réservées](../cost-management-billing/reservations/manage-reserved-vm-instance.md#change-optimize-setting-for-reserved-vm-instances).

Avec une instance de machine virtuelle réservée optimisée pour la flexibilité de la taille de l’instance, la réservation que vous achetez peut s’appliquer aux tailles de machine virtuelle d’un même groupe de flexibilité de la taille d'instance. Par exemple, si vous achetez une réservation pour une taille de machine virtuelle qui est répertoriée dans la série DSv2, comme Standard_DS5_v2, la remise sur réservation peut s’appliquer aux quatre autres tailles répertoriées dans ce même groupe :

- Standard_DS1_v2
- Standard_DS2_v2
- Standard_DS3_v2
- Standard_DS4_v2

Mais cette remise sur réservation ne s’applique pas aux tailles de machine virtuelle qui figurent dans des groupes de flexibilité de la taille d'instance différents, comme des références SKU dans la table de mémoire haute de la série DSv2 : Standard_DS11_v2, Standard_DS12_v2, etc.

Dans le groupe de flexibilité de la taille d'instance, le nombre de machines virtuelles auxquelles la remise sur réservation s’applique dépend de la taille de machine virtuelle que vous choisissez lorsque vous achetez une réservation. Ce nombre dépend également des tailles des machines virtuelles que vous utilisez. La colonne de ratio compare l’empreinte relative de chaque taille de machine virtuelle de ce groupe de flexibilité de la taille d'instance. Utilisez la valeur de ratio pour calculer la façon dont la remise sur réservation s’applique aux machines virtuelles vous utilisez.

## <a name="examples"></a>Exemples

Les exemples suivants utilisent les tailles et ratios de la table de la série DSv2.

Vous achetez une instance de machine virtuelle réservée avec la taille Standard_DS4_v2, où le ratio ou l’empreinte relative par rapport aux autres tailles de cette série correspond à la valeur 8.

- Scénario 1 : utiliser huit machines virtuelles de taille Standard_DS1_v2 avec un ratio de valeur 1. Votre remise sur réservation s’applique à ces huit machines virtuelles.
- Scénario 2 : utiliser deux machines virtuelles de taille Standard_DS2_v2 avec chacune un ratio de valeur 2. Utiliser également une machine virtuelle de taille Standard_DS3_v2 avec un ratio de valeur 4. Empreinte totale : 2 + 2 + 4 = 8. Votre remise sur réservation s’applique à ces trois machines virtuelles.
- Scénario 3 : utiliser une machine virtuelle de taille Standard_DS5_v2 avec un ratio de valeur 16. Votre remise sur réservation s’applique à la moitié du coût du calcul de cette machine virtuelle.

Les sections suivantes présentent les tailles qui figurent dans le même groupe de séries de taille lorsque vous achetez une instance de machine virtuelle réservée optimisée pour la flexibilité de taille d’instance.

## <a name="instance-size-flexibility-ratio-for-vms"></a>Ratio de flexibilité de la taille d’instance pour les machines virtuelles 

Le fichier CSV ci-dessous contient les groupes de flexibilité de la taille d'instance, ArmSkuName et les ratios.  

[Ratios de flexibilité de la taille d'instance](https://isfratio.blob.core.windows.net/isfratio/ISFRatio.csv)

Nous conservons l’URL du fichier et le schéma fixes pour que vous puissiez utiliser le fichier par programmation. Les données seront également disponibles prochainement via l’API.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez [Qu’est-ce qu’une réservation Azure ?](../cost-management-billing/reservations/save-compute-costs-reservations.md).

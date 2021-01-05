---
title: Quotas de processeurs virtuels pour Azure
description: Découvrez plus en détail les quotas de processeurs virtuels pour les machines virtuelles Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: sizes
ms.workload: infrastructure-services
ms.topic: conceptual
ms.date: 05/31/2018
ms.author: cynthn
ms.openlocfilehash: b00d84e1c2dc8f104046fe24360d79e6952dfacc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88950287"
---
# <a name="check-vcpu-quotas-using-azure-powershell"></a>Vérifier les quotas de processeurs virtuels à l’aide d’Azure PowerShell

Les quotas de processeurs virtuels pour les machines virtuelles et les groupes de machines virtuelles identiques sont organisés en deux niveaux pour chaque abonnement, dans chaque région. Le premier niveau est le total des processeurs virtuels régionaux. Le deuxième niveau se compose des cœurs des différentes familles de tailles de machine virtuelle, comme les processeurs virtuels de la série D. Chaque fois qu’une nouvelle machine virtuelle est déployée, le nombre de processeurs virtuels de cette machine virtuelle ne doit pas dépasser le quota de processeurs virtuels pour la famille de tailles de machine virtuelle ou le quota du total des processeurs virtuels régionaux. Si l’un de ces quotas est dépassé, le déploiement des machines virtuelles n’est pas autorisé. Il existe également un quota pour le nombre total de machines virtuelles dans la région. Pour plus d’informations sur chacun de ces quotas, consultez la section **Utilisation + quotas** de la page **Abonnement** dans le [portail Azure](https://portal.azure.com) ou interrogez les valeurs à l’aide de PowerShell.

> [!NOTE]
> Le quota se calcule en fonction du nombre total de cœurs en cours d’utilisation, à la fois alloués et libérés. Si vous avez besoin de cœurs supplémentaires, [demandez une augmentation de quota](../../azure-portal/supportability/resource-manager-core-quotas-request.md) ou supprimez des machines virtuelles devenues inutiles. 
 
## <a name="check-usage"></a>Vérifier l’utilisation

Vous pouvez utiliser la cmdlet [Get-AzVMUsage](/powershell/module/az.compute/get-azvmusage) pour vérifier votre utilisation du quota.

```azurepowershell-interactive
Get-AzVMUsage -Location "East US"
```

Le résultat ressemble à ce qui suit :

```
Name                             Current Value Limit  Unit
----                             ------------- -----  ----
Availability Sets                            0  2000 Count
Total Regional vCPUs                         4   260 Count
Virtual Machines                             4 10000 Count
Virtual Machine Scale Sets                   1  2000 Count
Standard B Family vCPUs                      1    10 Count
Standard DSv2 Family vCPUs                   1   100 Count
Standard Dv2 Family vCPUs                    2   100 Count
Basic A Family vCPUs                         0   100 Count
Standard A0-A7 Family vCPUs                  0   250 Count
Standard A8-A11 Family vCPUs                 0   100 Count
Standard D Family vCPUs                      0   100 Count
Standard G Family vCPUs                      0   100 Count
Standard DS Family vCPUs                     0   100 Count
Standard GS Family vCPUs                     0   100 Count
Standard F Family vCPUs                      0   100 Count
Standard FS Family vCPUs                     0   100 Count
Standard NV Family vCPUs                     0    24 Count
Standard NC Family vCPUs                     0    48 Count
Standard H Family vCPUs                      0     8 Count
Standard Av2 Family vCPUs                    0   100 Count
Standard LS Family vCPUs                     0   100 Count
Standard Dv2 Promo Family vCPUs              0   100 Count
Standard DSv2 Promo Family vCPUs             0   100 Count
Standard MS Family vCPUs                     0     0 Count
Standard Dv3 Family vCPUs                    0   100 Count
Standard DSv3 Family vCPUs                   0   100 Count
Standard Ev3 Family vCPUs                    0   100 Count
Standard ESv3 Family vCPUs                   0   100 Count
Standard FSv2 Family vCPUs                   0   100 Count
Standard ND Family vCPUs                     0     0 Count
Standard NCv2 Family vCPUs                   0     0 Count
Standard NCv3 Family vCPUs                   0     0 Count
Standard LSv2 Family vCPUs                   0     0 Count
Standard Storage Managed Disks               2 10000 Count
Premium Storage Managed Disks                1 10000 Count
```


## <a name="reserved-vm-instances"></a>Instances de machines virtuelles réservées
Les instances de machines virtuelles réservées, dont l’étendue se limite à un seul abonnement sans flexibilité de la taille des machines virtuelles, ajoutent un nouvel aspect aux quotas de processeurs virtuels. Ces valeurs décrivent le nombre d’instances de la taille indiquée qui doivent être déployables dans l’abonnement. Elles jouent le rôle d’espace réservé dans le système de quota, pour garantir que le quota est réservé et que les instances de machines virtuelles sont ainsi déployables dans l’abonnement. Par exemple, si un abonnement spécifique dispose de 10 instances de machines virtuelles réservées Standard_D1, la limite d’utilisation des instances de machines virtuelles réservées Standard_D1 est de 10. De cette façon, Azure veille à ce qu’il y ait toujours au moins 10 processeurs virtuels disponibles dans le quota Total des processeurs virtuels régionaux pour les instances Standard_D1 et au moins 10 processeurs virtuels disponibles dans le quota de processeurs virtuels Famille D Standard pour les instances Standard_D1.

Si une augmentation de quota est nécessaire pour acheter une instance réservée à abonnement unique, vous pouvez [demander une augmentation de quota](../../azure-portal/supportability/resource-manager-core-quotas-request.md) sur votre abonnement.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la facturation et les quotas, consultez [Abonnement Azure et limites, quotas et contraintes du service](../../azure-resource-manager/management/azure-subscription-service-limits.md?toc=/azure/billing/TOC.json).

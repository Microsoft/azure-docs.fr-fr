---
title: Codes d’erreur pour les instances de groupes de machines virtuelles identiques et les machines virtuelles Azure Spot
description: Découvrez les codes d’erreur que vous pouvez voir quand vous utilisez des instances de groupes de machines virtuelles identiques et des machines virtuelles Spot.
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: troubleshooting
ms.date: 03/25/2020
ms.author: cynthn
ms.openlocfilehash: 3e151145bd3c4d1db87ef5d1b0d553555a580279
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88816743"
---
# <a name="error-messages-for-spot-vms-and-scale-sets"></a>Messages d’erreur pour les groupes de machines virtuelles identiques et les machines virtuelles Spot

Voici quelques codes d’erreur que vous pouvez recevoir quand vous utilisez des groupes de machines virtuelles identiques et des machines virtuelles Spot.


| Clé | Message | Description |
|-----|---------|-------------|
| SkuNotAvailable | Le niveau demandé pour la ressource « \<resource\> » n’est pas disponible dans l’emplacement « \<location\> » pour l’abonnement « \<subscriptionID\> ». Essayez un autre niveau ou déployez sur un autre emplacement. | Il n’y a pas assez de capacité Azure Spot sur cet emplacement pour créer votre machine virtuelle ou votre instance de groupe de machines virtuelles identiques. |
| EvictionPolicyCanBeSetOnlyOnAzureSpotVirtualMachines  |  Une stratégie d’éviction peut être définie seulement sur des machines virtuelles Azure Spot. | Cette machine virtuelle n’étant pas une machine virtuelle Spot, vous ne pouvez pas définir la stratégie d’éviction. |
| AzureSpotVMNotSupportedInAvailabilitySet  |  Les machines virtuelles Azure Spot ne sont pas prises en charge dans les groupes à haute disponibilité. | Vous devez choisir d’utiliser une machine virtuelle Spot, ou bien une machine virtuelle dans un groupe à haute disponibilité ; vous ne pouvez pas choisir les deux. |
| AzureSpotFeatureNotEnabledForSubscription  |  L’abonnement n’est pas activé avec la fonctionnalité Azure Spot. | Utilisez un abonnement qui prend en charge les machines virtuelles Spot. |
| VMPriorityCannotBeApplied  |  Impossible d’appliquer la valeur de priorité spécifiée « {0} » à la machine virtuelle « {1} », car aucune priorité n’a été spécifiée à la création de la machine virtuelle. | Spécifiez la priorité lors de la création de la machine virtuelle. |
| SpotPriceGreaterThanProvidedMaxPrice  |  Impossible d’effectuer l’opération « {0} », car le prix maximal (« {1} USD ») fourni est inférieur au prix Spot actuel (« {2} USD ») pour la taille de machine virtuelle Azure Spot (« {3} »). | Sélectionnez un prix maximal supérieur. Pour plus d’informations, consultez les informations sur les prix pour [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) ou [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/).|
| MaxPriceValueInvalid  |  Valeur du prix maximal non valide. Les seules valeurs prises en charge pour le prix maximal sont -1 ou un chiffre décimal supérieur à zéro. Une valeur de prix maximal égale à -1 indique que la machine virtuelle Azure Spot n’est pas supprimée pour des raisons de prix. | Entrez un prix maximal valide. Pour plus d’informations, consultez les prix pour [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) ou [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/). |
| MaxPriceChangeNotAllowedForAllocatedVMs | Il n’est pas autorisé de changer le prix maximal quand la machine virtuelle « {0} » est allouée. Désallouez-la et réessayez. | Arrêtez\désallouez la machine virtuelle afin que vous puissiez changer de prix maximal. |
| MaxPriceChangeNotAllowed | Il n’est pas autorisé de changer le prix maximal. | Vous ne pouvez pas changer de prix maximal pour cette machine virtuelle. |
| AzureSpotIsNotSupportedForThisAPIVersion  |  Azure Spot n’est pas pris en charge pour cette version d’API. | La version d’API doit être 2019-03-01. |
| AzureSpotIsNotSupportedForThisVMSize  |  Azure Spot n’est pas pris en charge pour cette taille de machine virtuelle {0}. | Sélectionnez une autre taille de machine virtuelle. Pour plus d’informations, consultez [Machines virtuelles Spot](./spot-vms.md). |
| MaxPriceIsSupportedOnlyForAzureSpotVirtualMachines  |  Le prix maximal est pris en charge uniquement pour les machines virtuelles Azure Spot. | Pour plus d’informations, consultez [Machines virtuelles Spot](./spot-vms.md). |
| MoveResourcesWithAzureSpotVMNotSupported  |  La demande de déplacement de ressources contient une machine virtuelle Azure Spot. Non pris en charge actuellement. Consultez les détails de l’erreur pour connaître les ID des machines virtuelles. | Vous ne pouvez pas déplacer des machines virtuelles Spot. |
| MoveResourcesWithAzureSpotVmssNotSupported  |  La demande de déplacement de ressources contient un groupe de machines virtuelles identiques Azure Spot. Non pris en charge actuellement. Consultez les détails de l’erreur pour connaître les ID des groupes de machines virtuelles identiques. | Vous ne pouvez pas déplacer des instances de groupes de machines virtuelles identiques Spot. |
| AzureSpotVMNotSupportedInVmssWithVMOrchestrationMode | Les machines virtuelles Azure Spot ne sont pas prises en charge dans les groupes de machines virtuelles identiques avec le mode d’orchestration de machine virtuelle. | Définissez le mode d’orchestration sur le groupe de machines virtuelles identiques pour utiliser des instances Spot. |


**Étapes suivantes** Pour plus d’informations, consultez [Machines virtuelles Spot](./spot-vms.md).
---
title: Mettre à niveau des adresses IP publiques
titleSuffix: Azure Virtual Network
description: Mettre à niveau des adresses IP publiques du niveau De base à Standard.
services: virtual-network
documentationcenter: na
author: blehr
editor: ''
ms.assetid: bb71abaf-b2d9-4147-b607-38067a10caf6
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/08/2020
ms.author: blehr
ms.custom: references_regions
ms.openlocfilehash: 3e2905019244279129528c177a76291cb7d75e11
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97825769"
---
# <a name="upgrade-public-ip-addresses"></a>Mettre à niveau des adresses IP publiques

Les adresses IP publiques Azure sont créées avec une référence SKU (De base ou Standard) qui détermine les aspects de leurs fonctionnalités (notamment la méthode d’allocation, l’utilisation dans les zones de disponibilité et les ressources auxquelles elles peuvent être associées). 

Les scénarios abordés dans cet article sont les suivants :
* Mise à niveau d’une adresse IP publique de référence SKU De base vers une adresse IP publique de référence SKU Standard (à l’aide du portail, de PowerShell ou d’Azure CLI)
* Migration d’une adresse IP réservée classique Azure vers une adresse IP publique de référence SKU De base Azure Resource Manager

## <a name="upgrade-public-ip-address-from-basic-to-standard-sku"></a>Mettre à niveau une adresse IP publique d’une référence SKU De base à Standard

Pour pouvoir mettre à niveau une adresse IP publique, celle-ci ne doit être associée à aucune ressource (consultez [cette page](https://docs.microsoft.com/azure/virtual-network/virtual-network-public-ip-address#view-modify-settings-for-or-delete-a-public-ip-address) pour en savoir plus sur la façon de dissocier les adresses IP publiques).

>[!IMPORTANT]
>Les adresses IP publiques mises à niveau d’une référence SKU De base à Standard n’ont toujours aucune [zone de disponibilité](https://docs.microsoft.com/azure/availability-zones/az-overview?toc=/azure/virtual-network/toc.json#availability-zones).  Cela signifie qu’elles ne peuvent pas être associées à une ressource Azure redondante interzone ou liée à une zone prédéfinie dans les régions où cela est proposé.

---
# <a name="basic-to-standard---powershell"></a>[**De base à Standard - PowerShell**](#tab/option-upgrade-powershell)

L’exemple suivant suppose qu’une adresse IP publique de référence SKU De base a été créée auparavant, à l’aide de l’exemple fourni sur [cette page](https://docs.microsoft.com/azure/virtual-network/create-public-ip-powershell?tabs=option-create-public-ip-basic) avec une adresse IP publique De base **myBasicPublicIP** dans **myResourceGroup**.

Pour procéder à la mise à niveau de l’adresse IP, exécutez simplement les commandes ci-dessous à l’aide de PowerShell.  Notez que si l’adresse IP est déjà allouée de manière statique, vous pouvez passer cette section.

```azurepowershell-interactive
## Variables for the command ##
$rg = 'myResourceGroup'
$name = 'myBasicPublicIP'
$newsku = 'Standard'
$pubIP = Get-AzPublicIpAddress -name $name -ResourceGroupName $rg

## This section is only needed if the Basic IP is not already set to Static ##
$pubIP.PublicIpAllocationMethod = 'Static'
Set-AzPublicIpAddress -PublicIpAddress $pubIP

## This section is for conversion to Standard ##
$pubIP.Sku.Name = $newsku
Set-AzPublicIpAddress -PublicIpAddress $pubIP
```

# <a name="basic-to-standard---cli"></a>[**De base à Standard - CLI**](#tab/option-upgrade-cli)

L’exemple suivant suppose qu’une adresse IP publique de référence SKU De base a été créée auparavant, à l’aide de l’exemple fourni sur [cette page](https://docs.microsoft.com/azure/virtual-network/create-public-ip-cli?tabs=option-create-public-ip-basic) avec une adresse IP publique De base **myBasicPublicIP** dans **myResourceGroup**.

Pour procéder à la mise à niveau de l’adresse IP, exécutez simplement les commandes ci-dessous à l’aide d’Azure CLI.  Notez que si l’adresse IP est déjà allouée de manière statique, vous pouvez passer cette section.

```azurecli-interactive
## Variables for the command ##
$rg = 'myResourceGroup'
$name = 'myBasicPublicIP'

## This section is only needed if the Basic IP is not already set to Static ##
az network public-ip update \
--resource-group $rg \
--name $name \
--allocation-method Static 

## This section is for conversion to Standard ##
az network public-ip update \
--resource-group $rg \
--name $name \
--sku Standard
```
---

## <a name="upgrade-migrate-a-classic-reserved-ip-to-a-static-public-ip"></a>Mettre à niveau (migrer) une adresse IP réservée classique vers une adresse IP publique statique

Pour tirer parti des nouvelles fonctionnalités d’Azure Resource Manager, vous pouvez migrer les adresses IP statiques publiques existantes (appelées adresses IP réservées) du modèle classique vers le modèle Azure Resource Manager moderne.  L’adresse IP publique migrée aura le type de référence SKU De base.


---

# <a name="reserved-to-basic---powershell"></a>[**Réservé à De base - PowerShell**](#tab/option-migrate-powershell)

L’exemple suivant suppose qu’une adresse IP réservée Azure classique **myReservedIP** a été créée auparavant dans **myResourceGroup**. Autre prérequis pour la migration, l’abonnement Azure Resource Manager doit être inscrit pour la migration. Ce sujet est traité en détail dans les étapes 3 et 4 de [cette page](https://docs.microsoft.com/azure/virtual-machines/windows/migration-classic-resource-manager-ps).

Pour procéder à la migration de l’adresse IP réservée, exécutez les commandes ci-dessous à l’aide de PowerShell.  Notez que si l’adresse IP n’est associée à aucun service (dans l’exemple ci-dessous il y en a un, appelé **myService**), vous pouvez passer cette étape.

```azurepowershell-interactive
## Variables for the command ##
$name = 'myReservedIP'

## This section is only needed if the Reserved IP is not already disassociated from any Cloud Services ##
$serviceName = 'myService'
Remove-AzureReservedIPAssociation -ReservedIPName $name -ServiceName $service

$validate = Move-AzureReservedIP -ReservedIPName $name -Validate
$validate.ValidationMessages
```
La commande précédente affiche les avertissements et erreurs qui bloquent la migration. Si la validation se déroule correctement, vous pouvez exécuter les étapes suivantes pour préparer et valider la migration :
```azurepowershell-interactive
Move-AzureReservedIP -ReservedIPName $name -Prepare
Move-AzureReservedIP -ReservedIPName $name -Commit
```
Un nouveau groupe de ressources dans Azure Resource Manager est créé à l’aide du nom de l’adresse IP réservée qui a été migrée (dans l’exemple ci-dessus, il s’agit du groupe de ressources **myReservedIP-Migrated**).

# <a name="reserved-to-basic---cli"></a>[**Réservé à De base - CLI**](#tab/option-migrate-cli)

L’exemple suivant suppose qu’une adresse IP réservée Azure classique **myReservedIP** a été créée auparavant dans **myResourceGroup**. Autre prérequis pour la migration, l’abonnement Azure Resource Manager doit être inscrit pour la migration. Ce sujet est traité en détail dans les étapes 3 et 4 de [cette page](https://docs.microsoft.com/azure/virtual-machines/linux/migration-classic-resource-manager-cli).

Pour procéder à la migration de l’adresse IP réservée, exécutez les commandes ci-dessous à l’aide d’Azure CLI.  Notez que si l’adresse IP n’est associée à aucun service (dans l’exemple ci-dessous il y a un service, appelé **myService**, et un déploiement, **myDeployment**), vous pouvez passer cette étape.

```azurecli-interactive
## Variables for the command ##
$name = 'myReservedIP'

## This section is only needed if the Reserved IP is not already disassociated from any Cloud Services ##
$service = 'myService'
$deployment = 'myDeployment'
azure network reserved-ip disassociate $name $service $deployment

azure network reserved-ip validate-migration $name
```
La commande précédente affiche les avertissements et erreurs qui bloquent la migration. Si la validation se déroule correctement, vous pouvez exécuter les étapes suivantes pour préparer et valider la migration :
```azurecli-interactive
azure network reserved-ip prepare-migration $name
azure network reserved-ip commit-migration $name
```
Un nouveau groupe de ressources dans Azure Resource Manager est créé à l’aide du nom de l’adresse IP réservée qui a été migrée (dans l’exemple ci-dessus, il s’agit du groupe de ressources **myReservedIP-Migrated**).

---

## <a name="limitations"></a>Limites

* Pour pouvoir mettre à niveau une adresse IP publique De base, celle-ci ne doit être associée à aucune ressource Azure.  Consultez [cette page](https://docs.microsoft.com/azure/virtual-network/virtual-network-public-ip-address#view-modify-settings-for-or-delete-a-public-ip-address) pour en savoir plus sur la façon de dissocier des adresses IP publiques.  De même, pour pouvoir migrer une adresse IP réservée, celle-ci ne doit être associée à aucun service cloud.  Consultez [cette page](https://docs.microsoft.com/azure/virtual-network/remove-public-ip-address-vm) pour en savoir plus sur la façon de dissocier des adresses IP réservées.  
* Les adresses IP publiques mises à niveau d’une référence SKU De base à Standard n’ont toujours aucune [zone de disponibilité](https://docs.microsoft.com/azure/availability-zones/az-overview?toc=/azure/virtual-network/toc.json#availability-zones) et, par conséquent, ne peuvent pas être associées à une ressource Azure redondante interzone ou zonale.  Notez que cela s’applique uniquement aux régions qui proposent des zones de disponibilité.
* Vous ne pouvez pas passer du niveau Standard au niveau De base.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [adresses IP publiques](virtual-network-ip-addresses-overview-arm.md#public-ip-addresses) dans Azure, notamment sur la différence entre les types de référence SKU, ainsi que sur les [paramètres d’adresse IP publique](virtual-network-public-ip-address.md#create-a-public-ip-address).
- Découvrez comment [mettre à niveau Azure Public Load Balancer du niveau De base à Standard](https://docs.microsoft.com/azure/load-balancer/upgrade-basic-standard).
- Découvrez les [adresses IP réservées Azure classiques](https://docs.microsoft.com/previous-versions/azure/virtual-network/virtual-networks-reserved-public-ip) et la [migration de ressources classiques vers Azure Resource Manager](https://docs.microsoft.com/azure/virtual-machines/windows/migration-classic-resource-manager-overview).

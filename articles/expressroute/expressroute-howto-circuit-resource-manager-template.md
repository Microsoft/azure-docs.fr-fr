---
title: 'Modèle Azure ExpressRoute : Création d’un circuit ExpressRoute'
description: Découvrez comment créer un circuit Azure ExpressRoute en déployant un modèle Azure Resource Manager à l’aide d’Azure PowerShell.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 11/13/2019
ms.author: duau
ms.openlocfilehash: 2e9b6ddc9da4467590946af12a47f1473a4ea494
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92202053"
---
# <a name="create-an-expressroute-circuit-by-using-azure-resource-manager-template"></a>Création d’un circuit ExpressRoute en utilisant le modèle Azure Resource Manager

> [!div class="op_single_selector"]
> * [Azure portal](expressroute-howto-circuit-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-circuit-arm.md)
> * [Azure CLI](howto-circuit-cli.md)
> * [Modèle Azure Resource Manager](expressroute-howto-circuit-resource-manager-template.md)
> * [Vidéo - portail Azure](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-an-expressroute-circuit)
> * [PowerShell (classique)](expressroute-howto-circuit-classic.md)
>

Découvrez comment créer un circuit ExpressRoute en déployant un modèle Azure Resource Manager à l’aide d’Azure PowerShell. Pour plus d’informations sur le développement de modèles Resource Manager, consultez la [documentation Resource Manager](../azure-resource-manager/index.yml) et les [informations de référence sur les modèles](/azure/templates/microsoft.network/expressroutecircuits).

## <a name="before-you-begin"></a>Avant de commencer

* Examinez les [conditions préalables](expressroute-prerequisites.md) et les [flux de travail](expressroute-workflows.md) avant de commencer la configuration.
* Assurez-vous que vous disposez des autorisations nécessaires pour créer des ressources réseau. Contactez votre administrateur de compte si vous n'avez pas les autorisations appropriées.
* Vous pouvez [visualiser une vidéo](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-an-expressroute-circuit) avant de commencer afin de mieux comprendre les étapes.

## <a name="create-and-provision-an-expressroute-circuit"></a><a name="create"></a>Créer et approvisionner un circuit ExpressRoute

[Les modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/) comprennent une bonne collection de modèles Resource Manager. Vous utilisez un des [modèles existants](https://azure.microsoft.com/resources/templates/101-expressroute-circuit-create/) pour créer un circuit ExpressRoute.

[!code-json[create-azure-expressroute-circuit](~/quickstart-templates/101-expressroute-circuit-create/azuredeploy.json)]

Pour afficher plusieurs modèles associés, cliquez[ici](https://azure.microsoft.com/resources/templates/?term=expressroute).

Pour créer un circuit ExpressRoute en déployant un modèle :

1. Sélectionnez **Essayer** à partir du bloc de code suivant, puis suivez les instructions permettant de vous connecter à Azure Cloud Shell.

    ```azurepowershell-interactive
    $circuitName = Read-Host -Prompt "Enter a circuit name"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $resourceGroupName = "${circuitName}rg"
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-expressroute-circuit-create/azuredeploy.json"

    $serviceProviderName = "Equinix"
    $peeringLocation = "Silicon Valley"
    $bandwidthInMbps = 500
    $sku_tier = "Premium"
    $sku_family = "MeteredData"

    New-AzResourceGroup -Name $resourceGroupName -Location $location
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri -circuitName $circuitName -serviceProviderName $serviceProviderName -peeringLocation $peeringLocation -bandwidthInMbps $bandwidthInMbps -sku_tier $sku_tier -sku_family $sku_family

    Write-Host "Press [ENTER] to continue ..."
    ```

   * Le **niveau SKU** détermine si un circuit ExpressRoute est [Local](expressroute-faqs.md#expressroute-local), Standard ou [Premium](expressroute-faqs.md#expressroute-premium). Vous pouvez spécifier *Local*, *Standard ou *Premium*. Vous ne pouvez pas remplacer la référence SKU de *Standard/Premium* par celle de *Local*.
   * La **famille de références** détermine le type de facturation. Vous pouvez spécifier *Metereddata* pour définir un forfait de données limité et *Unlimiteddata* pour un forfait de données illimité. Vous pouvez changer le type de facturation de *Metereddata* en *Unlimiteddata*, mais que vous ne pouvez pas le changer de *Unlimiteddata* en *Metereddata*. Un circuit *local* est uniquement *Unlimiteddata*.
   * L’**Emplacement de peering** est l’emplacement physique où vous effectuez le peering auprès de Microsoft.

     > [!IMPORTANT]
     > L’emplacement de peering indique [l’emplacement physique](expressroute-locations.md) où vous effectuez le peering auprès de Microsoft. Cet emplacement n’est **pas** lié à la propriété « Emplacement », qui fait référence à la zone géographique où se trouve le fournisseur de ressources réseau Azure. Bien que ces éléments ne soient pas liés, nous vous conseillons de choisir un fournisseur de ressources réseau géographiquement proche de l’emplacement de peering du circuit.

    Le nom du groupe de ressources correspond au nom du Service Bus auquel **rg** a été ajouté.

2. Sélectionnez **Copier** pour copier le script PowerShell.
3. Cliquez avec le bouton droit sur la console d’interpréteur de commandes, puis sélectionnez **Coller**.

Il faut quelques instants pour créer un Event Hub.

Dans ce tutoriel, Azure PowerShell permet de déployer le modèle. Pour d’autres méthodes de déploiement de modèle, consultez :

* [En utilisant le Portail Azure](../azure-resource-manager/templates/deploy-portal.md).
* [En utilisant Azure CLI](../azure-resource-manager/templates/deploy-cli.md).
* [En utilisant l’API REST](../azure-resource-manager/templates/deploy-rest.md).

## <a name="deprovisioning-and-deleting-an-expressroute-circuit"></a><a name="delete"></a>Annulation de l’approvisionnement et suppression d’un circuit ExpressRoute

Vous pouvez supprimer votre circuit ExpressRoute en sélectionnant l’icône **Supprimer** . Notez les informations suivantes :

* Vous devez dissocier tous les réseaux virtuels du circuit ExpressRoute. Si cette opération échoue, vérifiez si certains de vos réseaux virtuels sont liés au circuit.
* Si l’état d’approvisionnement du fournisseur de services du circuit ExpressRoute est **En cours d’approvisionnement** ou **Approvisionné**, vous devez vous mettre en relation avec votre fournisseur de services pour annuler l’approvisionnement du circuit de son côté. Nous continuons à réserver des ressources et à vous facturer jusqu’à ce que le fournisseur de services termine le désapprovisionnement du circuit et nous en avertisse.
* Si le fournisseur de services a annulé l’approvisionnement du circuit (l’état d’approvisionnement du fournisseur de services affiche la valeur **Non approvisionné**), vous pouvez supprimer le circuit. Cette opération arrête la facturation du circuit.

Vous pouvez supprimer votre circuit ExpressRoute en exécutant la commande PowerShell suivante :

```azurepowershell-interactive
$circuitName = Read-Host -Prompt "Enter the same circuit name that you used earlier"
$resourceGroupName = "${circuitName}rg"

Remove-AzExpressRouteCircuit -ResourceGroupName $resourceGroupName -Name $circuitName
```

## <a name="next-steps"></a>Étapes suivantes

Après avoir créé votre circuit, passez aux étapes suivantes :

* [Créer et modifier le routage le routage pour votre circuit ExpressRoute](expressroute-howto-routing-portal-resource-manager.md)
* [Lier votre réseau virtuel à votre circuit ExpressRoute](expressroute-howto-linkvnet-arm.md)
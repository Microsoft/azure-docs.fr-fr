---
title: 'Démarrage rapide : Créer et modifier un circuit avec ExpressRoute – Azure PowerShell'
description: Ce démarrage rapide montre comment créer, approvisionner, vérifier, mettre à jour, supprimer et déprovisionner un circuit ExpressRoute.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: quickstart
ms.date: 10/12/2020
ms.author: duau
ms.openlocfilehash: 5397dd2745a0d4e61804cf631014846ae15ec4e1
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91971538"
---
# <a name="quickstart-create-and-modify-an-expressroute-circuit-using-azure-powershell"></a>Démarrage rapide : Créer et modifier un circuit ExpressRoute à l’aide d’Azure PowerShell

Ce démarrage rapide vous montre comment créer un circuit ExpressRoute à l’aide de cmdlets PowerShell et du modèle de déploiement Azure Resource Manager. Vous pouvez également vérifier l’état d’un circuit, le mettre à jour, le supprimer ou le déprovisionner.

## <a name="prerequisites"></a>Prérequis

* Examinez les [conditions préalables](expressroute-prerequisites.md) et les [flux de travail](expressroute-workflows.md) avant de commencer la configuration.
* Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Azure PowerShell installé localement ou Azure Cloud Shell

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-and-provision-an-expressroute-circuit"></a><a name="create"></a>Créer et approvisionner un circuit ExpressRoute
### <a name="sign-in-to-your-azure-account-and-select-your-subscription"></a>Vous connecter à votre compte Azure et sélectionner votre abonnement

[!INCLUDE [sign in](../../includes/expressroute-cloud-shell-connect.md)]

### <a name="get-the-list-of-supported-providers-locations-and-bandwidths"></a>Récupérer la liste des fournisseurs, des emplacements et des bandes passantes pris en charge
Avant de créer un circuit ExpressRoute, vous avez besoin d’une liste des fournisseurs de services, des emplacements et des options de bande passante pris en charge.

L’applet de commande PowerShell **Get-AzExpressRouteServiceProvider** retourne ces informations que vous utilisez dans les étapes ultérieures :

```azurepowershell-interactive
Get-AzExpressRouteServiceProvider
```

Vérifiez si votre fournisseur de connectivité y est référencé. Prenez note des éléments suivants, car vous en aurez besoin lors de la création d’un circuit :

* Nom
* PeeringLocations
* BandwidthsOffered

Vous êtes maintenant prêt à créer un circuit ExpressRoute.

### <a name="create-an-expressroute-circuit"></a>Création d’un circuit ExpressRoute
Si vous n’avez pas déjà un groupe de ressources, vous devez en créer un avant de créer votre circuit ExpressRoute. Pour cela, vous pouvez exécuter la commande suivante :

```azurepowershell-interactive
New-AzResourceGroup -Name "ExpressRouteResourceGroup" -Location "West US"
```

L’exemple suivant montre comment créer un circuit ExpressRoute de 200 Mb/s par le biais d’Equinix dans la Silicon Valley. Si vous utilisez un autre fournisseur et des paramètres différents, remplacez ces informations quand vous créez votre requête. Utilisez l’exemple suivant pour demander une nouvelle clé de service :

```azurepowershell-interactive
New-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup" -Location "West US" -SkuTier Standard -SkuFamily MeteredData -ServiceProviderName "Equinix" -PeeringLocation "Silicon Valley" -BandwidthInMbps 200
```

Assurez-vous que vous spécifiez le niveau de référence (SKU) et la famille de références corrects :

* Le niveau de référence (SKU) détermine si un circuit ExpressRoute est [Local](expressroute-faqs.md#expressroute-local), Standard ou [Premium](expressroute-faqs.md#expressroute-premium). Vous pouvez spécifier *Local*, *Standard ou *Premium*. Vous ne pouvez pas modifier la référence (SKU) de *Standard/Premium* en *Local*.
* La famille de références détermine le type de facturation. Vous pouvez spécifier *Metereddata* pour un forfait de données limité, et *Unlimiteddata* pour un forfait de données illimitées. Vous pouvez modifier le type de facturation *Metereddata* en *Unlimiteddata*, mais ne pouvez pas le modifier de *Unlimiteddata* en *Metereddata*. Un circuit *Local* est toujours *Unlimiteddata*.

> [!IMPORTANT]
> Votre circuit ExpressRoute est facturé à partir de l’émission d'une clé de service. Effectuez cette opération seulement quand le fournisseur de connectivité prêt à approvisionner le circuit.
>

La réponse contient la clé de service. Vous pouvez obtenir une description détaillée de tous les paramètres en exécutant la commande suivante :

```azurepowershell-interactive
get-help New-AzExpressRouteCircuit -detailed
```


### <a name="list-all-expressroute-circuits"></a>Répertorier tous les circuits ExpressRoute
Pour obtenir la liste de tous les circuits ExpressRoute que vous avez créés, exécutez la commande **Get-AzExpressRouteCircuit** :

```azurepowershell-interactive
Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
```

La réponse ressemble à ce qui suit :

```azurepowershell
Name                             : ExpressRouteARMCircuit
ResourceGroupName                : ExpressRouteResourceGroup
Location                         : westus
Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
Etag                             : W/"################################"
ProvisioningState                : Succeeded
Sku                              : {
                                    "Name": "Standard_MeteredData",
                                    "Tier": "Standard",
                                    "Family": "MeteredData"
                                    }
CircuitProvisioningState          : Enabled
ServiceProviderProvisioningState  : NotProvisioned
ServiceProviderNotes              :
ServiceProviderProperties         : {
                                    "ServiceProviderName": "Equinix",
                                    "PeeringLocation": "Silicon Valley",
                                    "BandwidthInMbps": 200
                                    }
ServiceKey                        : **************************************
Peerings                          : []
```

Vous pouvez récupérer ces informations à tout moment à l’aide de l’applet de commande `Get-AzExpressRouteCircuit` . Un appel effectué sans paramètre répertorie tous les circuits. Votre clé de service apparaît dans le champ *ServiceKey* :

```azurepowershell-interactive
Get-AzExpressRouteCircuit
```

La réponse ressemble à ce qui suit :

```azurepowershell
Name                             : ExpressRouteARMCircuit
ResourceGroupName                : ExpressRouteResourceGroup
Location                         : westus
Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
Etag                             : W/"################################"
ProvisioningState                : Succeeded
Sku                              : {
                                    "Name": "Standard_MeteredData",
                                    "Tier": "Standard",
                                    "Family": "MeteredData"
                                    }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : NotProvisioned
ServiceProviderNotes             :
ServiceProviderProperties        : {
                                    "ServiceProviderName": "Equinix",
                                    "PeeringLocation": "Silicon Valley",
                                    "BandwidthInMbps": 200
                                    }
ServiceKey                       : **************************************
Peerings                         : []
```

### <a name="send-the-service-key-to-your-connectivity-provider-for-provisioning"></a>Envoyer la clé de service à votre fournisseur de connectivité pour l’approvisionnement
*ServiceProviderProvisioningState* fournit des informations sur l’état actuel de l’approvisionnement côté fournisseur de services. Satus indique l’état côté Microsoft. Pour plus d’informations sur les états d’approvisionnement des circuits, consultez [Flux de travail](expressroute-workflows.md#expressroute-circuit-provisioning-states).

Quand vous créez un circuit ExpressRoute, ce circuit affiche l’état suivant :

```azurepowershell
ServiceProviderProvisioningState : NotProvisioned
CircuitProvisioningState         : Enabled
```

Le circuit passe à l’état suivant quand le fournisseur de connectivité l’active pour vous :

```azurepowershell
ServiceProviderProvisioningState : Provisioning
Status                           : Enabled
```

Pour être utilisé, le circuit ExpressRoute doit être dans l’état suivant :

```azurepowershell
ServiceProviderProvisioningState : Provisioned
CircuitProvisioningState         : Enabled
```

### <a name="periodically-check-the-status-and-the-state-of-the-circuit-key"></a>Vérifier régulièrement le statut et l’état de la clé du circuit
La vérification du statut et de l’état de la clé du service vous permet de savoir quand votre fournisseur a approvisionné votre circuit. Une fois le circuit configuré, *ServiceProviderProvisioningState* a la valeur *Approvisionné*, comme le montre l’exemple suivant :

```azurepowershell-interactive
Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
```

La réponse ressemble à ce qui suit :

```azurepowershell
Name                             : ExpressRouteARMCircuit
ResourceGroupName                : ExpressRouteResourceGroup
Location                         : westus
Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
Etag                             : W/"################################"
ProvisioningState                : Succeeded
Sku                              : {
                                    "Name": "Standard_MeteredData",
                                    "Tier": "Standard",
                                    "Family": "MeteredData"
                                    }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : Provisioned
ServiceProviderNotes             :
ServiceProviderProperties        : {
                                    "ServiceProviderName": "Equinix",
                                    "PeeringLocation": "Silicon Valley",
                                    "BandwidthInMbps": 200
                                    }
ServiceKey                       : **************************************
Peerings                         : []
```

### <a name="create-your-routing-configuration"></a>Créer votre configuration de routage
Pour obtenir des instructions pas à pas, consultez l’article [Configuration du routage des circuits ExpressRoute](expressroute-howto-routing-arm.md) pour créer et modifier des peerings de circuit.

> [!IMPORTANT]
> Ces instructions s’appliquent seulement aux circuits créés avec des fournisseurs de services proposant des services de connectivité de couche 2. Si vous utilisez un fournisseur de services proposant des services gérés de couche 3 (généralement un VPN IP, comme MPLS), votre fournisseur de connectivité configure et gère le routage pour vous.
>

### <a name="link-a-virtual-network-to-an-expressroute-circuit"></a>Lier un réseau virtuel à un circuit ExpressRoute
Maintenant, vous devez lier un réseau virtuel à votre circuit ExpressRoute. Utilisez l’article [Liaison de réseaux virtuels à des circuits ExpressRoute](expressroute-howto-linkvnet-arm.md) quand vous travaillez avec le modèle de déploiement Resource Manager.

## <a name="getting-the-status-of-an-expressroute-circuit"></a>Récupération de l’état d’un circuit ExpressRoute
Vous pouvez récupérer ces informations à tout moment à l’aide de l’applet de commande **Get-AzExpressRouteCircuit**. Un appel effectué sans paramètre répertorie tous les circuits.

```azurepowershell-interactive
Get-AzExpressRouteCircuit
```

La réponse ressemble à ce qui suit :

```azurepowershell
Name                             : ExpressRouteARMCircuit
ResourceGroupName                : ExpressRouteResourceGroup
Location                         : westus
Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
Etag                             : W/"################################"
ProvisioningState                : Succeeded
Sku                              : {
                                    "Name": "Standard_MeteredData",
                                    "Tier": "Standard",
                                    "Family": "MeteredData"
                                    }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : Provisioned
ServiceProviderNotes             :
ServiceProviderProperties        : {
                                        "ServiceProviderName": "Equinix",
                                        "PeeringLocation": "Silicon Valley",
                                        "BandwidthInMbps": 200
                                    }
ServiceKey                       : **************************************
Peerings                         : []
```

Vous pouvez obtenir des informations sur un circuit ExpressRoute spécifique en transmettant à l’appel le nom du groupe de ressources et le nom du circuit comme paramètres :

```azurepowershell-interactive
Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
```

La réponse ressemble à ce qui suit :

```azurepowershell
Name                             : ExpressRouteARMCircuit
ResourceGroupName                : ExpressRouteResourceGroup
Location                         : westus
Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
Etag                             : W/"################################"
ProvisioningState                : Succeeded
Sku                              : {
                                        "Name": "Standard_MeteredData",
                                        "Tier": "Standard",
                                        "Family": "MeteredData"
                                    }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : Provisioned
ServiceProviderNotes             :
ServiceProviderProperties        : {
                                        "ServiceProviderName": "Equinix",
                                        "PeeringLocation": "Silicon Valley",
                                        "BandwidthInMbps": 200
                                    }
ServiceKey                       : **************************************
Peerings                         : []
```

Vous pouvez obtenir une description détaillée de tous les paramètres en exécutant la commande suivante :

```azurepowershell-interactive
get-help Get-AzExpressRouteCircuit -detailed
```

## <a name="modifying-an-expressroute-circuit"></a><a name="modify"></a>Modification d’un circuit ExpressRoute
Vous pouvez modifier certaines propriétés d'un circuit ExpressRoute sans affecter la connectivité.

Vous pouvez effectuer les tâches suivantes sans entraîner de temps d’arrêt :

* Activer ou désactiver le module complémentaire ExpressRoute Premium pour votre circuit ExpressRoute. La modification de la référence SKU de *Standard/Premium* en *Local* n’est pas prise en charge.
* Augmenter la bande passante de votre circuit ExpressRoute à condition que la capacité disponible sur le port le permette. La rétrogradation de la bande passante d’un circuit n’est pas prise en charge.
* Modifiez le plan de mesure de Données limitées à Données illimitées. La modification du plan de limitation de Données illimitées en Données limitées n’est pas prise en charge.
* Vous pouvez activer et désactiver *Autoriser les opérations classiques*.

Pour plus d’informations sur les limites et les limitations, reportez-vous au [FAQ ExpressRoute](expressroute-faqs.md).

### <a name="to-enable-the-expressroute-premium-add-on"></a>Pour activer le module complémentaire ExpressRoute Premium
Vous pouvez activer le module complémentaire ExpressRoute Premium pour votre circuit existant à l’aide de l’extrait de code PowerShell suivant :

```azurepowershell-interactive
$ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

$ckt.Sku.Tier = "Premium"
$ckt.sku.Name = "Premium_MeteredData"

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

Les fonctionnalités du module complémentaire ExpressRoute Premium sont ainsi activées pour votre circuit. Nous commençons à vous facturer la fonctionnalité du module complémentaire Premium dès que la commande a été exécutée avec succès.

### <a name="to-disable-the-expressroute-premium-add-on"></a>Pour désactiver le module complémentaire ExpressRoute Premium
> [!IMPORTANT]
> Cette opération peut échouer si vous utilisez des ressources supérieures à ce qui est autorisé pour le circuit standard.
>

Notez les informations suivantes :

* Avant de passer du niveau Premium au niveau Standard, vous devez vérifier que le nombre de réseaux virtuels liés au circuit est inférieur à 10. Si vous ne le faites pas, votre demande de mise à jour échoue et nous appliquons les tarifs Premium.
* Tous les réseaux virtuels dans d’autres régions géopolitiques doivent préalablement être dissociés. Si vous ne supprimez pas le lien, votre demande de mise à jour échoue et nous continuons de vous facturer aux tarifs Premium.
* Pour le peering privé, votre table de routage doit comporter moins de 4 000 routages. Si elle contient plus de 4 000 routages, la session BGP est abandonnée. Cette session ne sera réactivée qu’une fois le nombre de préfixes publiés repassé sous la barre de 4 000.

Vous pouvez désactiver le module complémentaire ExpressRoute Premium dans votre circuit existant à l’aide de l’applet de commande PowerShell suivante :

```azurepowershell-interactive
$ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

$ckt.Sku.Tier = "Standard"
$ckt.sku.Name = "Standard_MeteredData"

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

### <a name="to-update-the-expressroute-circuit-bandwidth"></a>Pour mettre à jour la bande passante d’un circuit ExpressRoute
Pour connaître les options de bande passante prises en charge par votre fournisseur, consultez le [FAQ ExpressRoute](expressroute-faqs.md). Vous pouvez choisir n'importe quelle taille supérieure à la taille de votre circuit existant.

> [!IMPORTANT]
> Vous devrez peut-être recréer le circuit ExpressRoute si la capacité sur le port existant est inappropriée. Vous ne pouvez pas mettre le circuit à niveau si aucune capacité supplémentaire n’est disponible à cet emplacement.
>
> Vous ne pouvez pas réduire la bande passante d’un circuit ExpressRoute sans interrompre le service. Le fait de passer à un niveau inférieur de bande passante vous oblige à annuler l’approvisionnement du circuit ExpressRoute, puis à réapprovisionner un nouveau circuit ExpressRoute.
>

Une fois que vous avez décidé de la taille dont vous avez besoin, utilisez la commande suivante pour redimensionner votre circuit :

```azurepowershell-interactive
$ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

$ckt.ServiceProviderProperties.BandwidthInMbps = 1000

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

Votre circuit sera mis à niveau côté Microsoft. Vous devrez ensuite contacter votre fournisseur de connectivité pour qu’il mette à jour les configurations de son côté afin de refléter cette modification. Une fois cette modification apportée, nous commencerons à vous facturer pour l’option de bande passante mise à jour.

### <a name="to-move-the-sku-from-metered-to-unlimited"></a>Pour modifier la référence de limité à illimité
Vous pouvez modifier la référence d'un circuit ExpressRoute à l'aide de l'extrait de code PowerShell suivant :

```azurepowershell-interactive
$ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

$ckt.Sku.Family = "UnlimitedData"
$ckt.sku.Name = "Premium_UnlimitedData"

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

### <a name="to-control-access-to-the-classic-and-resource-manager-environments"></a>Pour contrôler l'accès aux environnements classique et Resource Manager
Consultez les instructions dans [Transférer des circuits ExpressRoute du modèle de déploiement classique vers le modèle de déploiement Resource Manager](expressroute-howto-move-arm.md).

## <a name="deprovisioning-an-expressroute-circuit"></a><a name="delete"></a>Déprovisionnement d’un circuit ExpressRoute
Notez les informations suivantes :

* Vous devez dissocier tous les réseaux virtuels du circuit ExpressRoute. Si cette opération échoue, vérifiez si certains de vos réseaux virtuels sont liés au circuit.
* Si l’état d’approvisionnement du fournisseur de services du circuit ExpressRoute est **En cours d’approvisionnement** ou **Approvisionné**, vous devez vous mettre en relation avec votre fournisseur de services pour annuler l’approvisionnement du circuit de son côté. Nous continuons à réserver des ressources et à vous facturer jusqu’à ce que le fournisseur de services termine le désapprovisionnement du circuit et nous en avertisse.
* Si le fournisseur de services a déprovisionné le circuit, ce qui signifie que l’état d’approvisionnement du fournisseur de services est défini sur **Non approvisionné**, vous pouvez supprimer le circuit. La facturation du circuit s’arrêtera.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez supprimer votre circuit ExpressRoute en exécutant la commande suivante :

```azurepowershell-interactive
Remove-AzExpressRouteCircuit -ResourceGroupName "ExpressRouteResourceGroup" -Name "ExpressRouteARMCircuit"
```

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez créé votre circuit et l’avez approvisionné avec votre fournisseur, passez à l’étape suivante pour configurer l’homologation :

> [!div class="nextstepaction"]
> [Créer et modifier le routage le routage pour votre circuit ExpressRoute](expressroute-howto-routing-arm.md)

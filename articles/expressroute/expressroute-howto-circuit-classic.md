---
title: 'Azure ExpressRoute : Modifier un circuit : PowerShell : classique'
description: Cet article vous montre les étapes nécessaires à la vérification de l’état, à la mise à jour, à la suppression et au déprovisionnement du circuit de votre modèle de déploiement classique ExpressRoute.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 11/05/2019
ms.author: duau
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: be45d49d3f445810c7ac6a38e3e12abe178a4bed
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018153"
---
# <a name="modify-an-expressroute-circuit-using-powershell-classic"></a>Modifier un circuit ExpressRoute à l’aide de PowerShell (Azure Classic)

> [!div class="op_single_selector"]
> * [Azure portal](expressroute-howto-circuit-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-circuit-arm.md)
> * [Azure CLI](howto-circuit-cli.md)
> * [Modèle Azure Resource Manager](expressroute-howto-circuit-resource-manager-template.md)
> * [Vidéo - portail Azure](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-an-expressroute-circuit)
> * [PowerShell (classique)](expressroute-howto-circuit-classic.md)
>

Cet article vous montre les étapes nécessaires à la vérification de l’état, à la mise à jour, à la suppression et au déprovisionnement du circuit de votre modèle de déploiement classique ExpressRoute. Cet article s’applique au modèle de déploiement classique.

[!INCLUDE [expressroute-classic-end-include](../../includes/expressroute-classic-end-include.md)]

**À propos des modèles de déploiement Azure**

[!INCLUDE [vpn-gateway-classic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

## <a name="before-you-begin"></a>Avant de commencer

[!INCLUDE [classic powershell install instructions](../../includes/expressroute-poweshell-classic-install-include.md)]

## <a name="get-the-status-of-a-circuit"></a>Récupérer l’état d’un circuit

Vous pouvez récupérer ces informations à tout moment à l’aide de l’applet de commande `Get-AzureCircuit` . Un appel effectué sans aucun paramètre répertorie tous les circuits.

```powershell
Get-AzureDedicatedCircuit

Bandwidth                        : 200
CircuitName                      : MyTestCircuit
Location                         : Silicon Valley
ServiceKey                       : *********************************
ServiceProviderName              : equinix
ServiceProviderProvisioningState : Provisioned
Sku                              : Standard
Status                           : Enabled

Bandwidth                        : 1000
CircuitName                      : MyAsiaCircuit
Location                         : Singapore
ServiceKey                       : #################################
ServiceProviderName              : equinix
ServiceProviderProvisioningState : Provisioned
Sku                              : Standard
Status                           : Enabled
```

Vous pouvez obtenir des informations sur un circuit ExpressRoute spécifique en passant, en tant que paramètre, la clé de service à l’appel.

```powershell
Get-AzureDedicatedCircuit -ServiceKey "*********************************"

Bandwidth                        : 200
CircuitName                      : MyTestCircuit
Location                         : Silicon Valley
ServiceKey                       : *********************************
ServiceProviderName              : equinix
ServiceProviderProvisioningState : Provisioned
Sku                              : Standard
Status                           : Enabled
```

Vous pouvez obtenir une description détaillée de tous les paramètres en exécutant l’exemple suivant :

```powershell
get-help get-azurededicatedcircuit -detailed
```

## <a name="modify-a-circuit"></a>Modifier un circuit

Vous pouvez modifier certaines propriétés d'un circuit ExpressRoute sans affecter la connectivité.

Vous pouvez effectuer les tâches suivantes sans entraîner de temps d’arrêt :

* Activer ou désactiver le module complémentaire ExpressRoute Premium pour votre circuit ExpressRoute.
* Augmenter la bande passante de votre circuit ExpressRoute à condition que la capacité disponible sur le port le permette. La rétrogradation de la bande passante d'un circuit n'est pas prise en charge.
* Modifiez le plan de mesure de Données limitées à Données illimitées. La modification du plan de limitation de Données illimitées à Données limitées n’est pas prise en charge.
* Vous pouvez activer et désactiver *Autoriser les opérations classiques*.

Pour plus d’informations sur les limites et les limitations, reportez-vous au [FAQ ExpressRoute](expressroute-faqs.md) .

### <a name="enable-the-expressroute-premium-add-on"></a>Activer le module complémentaire ExpressRoute Premium

Vous pouvez activer le module complémentaire ExpressRoute Premium pour votre circuit existant à l’aide de l’applet de commande PowerShell suivante :

```powershell
Set-AzureDedicatedCircuitProperties -ServiceKey "*********************************" -Sku Premium

Bandwidth                        : 1000
CircuitName                      : TestCircuit
Location                         : Silicon Valley
ServiceKey                       : *********************************
ServiceProviderName              : equinix
ServiceProviderProvisioningState : Provisioned
Sku                              : Premium
Status                           : Enabled
```

Les fonctionnalités du module complémentaire ExpressRoute premium seront activées sur votre circuit. La facturation du module complémentaire Premium démarre dès que la commande est exécutée avec succès.

### <a name="disable-the-expressroute-premium-add-on"></a>Désactiver le module complémentaire ExpressRoute Premium

> [!IMPORTANT]
> Cette opération peut échouer si vous utilisez des ressources supérieures à ce qui est autorisé pour le circuit standard.
>
>

#### <a name="considerations"></a>Considérations

* Vérifiez que le nombre de réseaux virtuels liés au circuit est inférieur à 10 avant de rétrograder du niveau Premium au niveau Standard. Si vous ne le faites pas, votre demande de mise à jour échoue et les tarifs Premium continuent d’être appliqués.
* Vous devez dissocier tous les réseaux virtuels dans d'autres régions géopolitiques. Si vous ne le faites pas, votre demande de mise à jour échoue et les tarifs Premium continuent d’être appliqués.
* Pour le peering privé, votre table de routage doit comporter moins de 4 000 routages. Si la table de routage comporte plus de 4 000 routages, la session BGP s’arrête et ne sera pas réactivée tant que le nombre de préfixes publiés ne sera pas inférieur à 4 000.

#### <a name="to-disable-the-premium-add-on"></a>Pour désactiver le module complémentaire Premium

Vous pouvez désactiver le module complémentaire ExpressRoute Premium pour votre circuit existant à l’aide de l’applet de commande PowerShell suivante :

```powershell

Set-AzureDedicatedCircuitProperties -ServiceKey "*********************************" -Sku Standard

Bandwidth                        : 1000
CircuitName                      : TestCircuit
Location                         : Silicon Valley
ServiceKey                       : *********************************
ServiceProviderName              : equinix
ServiceProviderProvisioningState : Provisioned
Sku                              : Standard
Status                           : Enabled
```

### <a name="update-the-expressroute-circuit-bandwidth"></a>Mettre à jour la bande passante d’un circuit ExpressRoute

Pour connaître les options de bande passante prises en charge par votre fournisseur, consultez le [FAQ ExpressRoute](expressroute-faqs.md) . Vous pouvez choisir toute taille supérieure à celle de votre circuit existant, pour autant que le port physique (sur lequel votre circuit est créé) le permette.

> [!IMPORTANT]
> Vous devrez peut-être recréer le circuit ExpressRoute si la capacité sur le port existant est inappropriée. Vous ne pouvez pas mettre le circuit à niveau si aucune capacité supplémentaire n’est disponible à cet emplacement.
>
> Vous ne pouvez pas réduire la bande passante d’un circuit ExpressRoute sans interrompre le service. Le fait de passer à un niveau inférieur de bande passante vous oblige à annuler l’approvisionnement du circuit ExpressRoute, puis à réapprovisionner un nouveau circuit ExpressRoute.
>
>

#### <a name="resize-a-circuit"></a>Redimensionner un circuit

Une fois que vous avez décidé de la taille dont vous avez besoin, vous pouvez utiliser la commande suivante pour redimensionner votre circuit :

```powershell
Set-AzureDedicatedCircuitProperties -ServiceKey ********************************* -Bandwidth 1000

Bandwidth                        : 1000
CircuitName                      : TestCircuit
Location                         : Silicon Valley
ServiceKey                       : *********************************
ServiceProviderName              : equinix
ServiceProviderProvisioningState : Provisioned
Sku                              : Standard
Status                           : Enabled
```

Une fois le circuit redimensionné par Microsoft, vous devez contacter votre fournisseur de connectivité pour qu’il mette à jour les configurations de son côté afin de refléter ce changement. C’est à partir de là que démarre la facturation de l’option de bande passante mise à jour.

Si l’erreur suivante s’affiche lors de l’augmentation de la bande passante du circuit, cela signifie qu’il ne reste pas suffisamment de bande passante sur le port physique sur lequel votre circuit est créé. Vous devez supprimer ce circuit et en créer un nouveau de la taille nécessaire.

```powershell
Set-AzureDedicatedCircuitProperties : InvalidOperation : Insufficient bandwidth available to perform this circuit
update operation
At line:1 char:1
+ Set-AzureDedicatedCircuitProperties -ServiceKey ********************* ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  + CategoryInfo          : CloseError: (:) [Set-AzureDedicatedCircuitProperties], CloudException
  + FullyQualifiedErrorId : Microsoft.WindowsAzure.Commands.ExpressRoute.SetAzureDedicatedCircuitPropertiesCommand
```

## <a name="deprovision-and-delete-a-circuit"></a>Déprovisionner et supprimer un circuit

### <a name="considerations"></a>Considérations

* Vous devez annuler la liaison de tous les réseaux virtuels du circuit ExpressRoute pour que cette opération réussisse. Si cette opération échoue, vérifiez si des réseaux virtuels sont liés au circuit.
* Si l’état d’approvisionnement du fournisseur de services du circuit ExpressRoute est **En cours d’approvisionnement** ou **Approvisionné**, vous devez vous mettre en relation avec votre fournisseur de services pour annuler l’approvisionnement du circuit de son côté. Nous continuons à réserver des ressources et à vous facturer jusqu’à ce que le fournisseur de services termine le désapprovisionnement du circuit et nous en avertisse.
* Si le fournisseur de services a déprovisionné le circuit (l’état d’approvisionnement du fournisseur de services doit afficher la valeur **Non approvisionné**), vous pouvez supprimer le circuit. Cette opération arrête la facturation du circuit.

#### <a name="delete-a-circuit"></a>Supprimer un circuit

Vous pouvez supprimer votre circuit ExpressRoute en exécutant la commande suivante :

```powershell
Remove-AzureDedicatedCircuit -ServiceKey "*********************************"
```

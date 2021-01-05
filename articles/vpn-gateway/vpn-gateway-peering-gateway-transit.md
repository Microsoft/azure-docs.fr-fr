---
title: Configurer le transit par passerelle VPN pour le peering de réseaux virtuels
description: Configurez le transit par passerelle pour l’appairage de réseaux virtuels, afin de connecter en toute transparence deux réseaux virtuels Azure en un seul à des fins de connectivité.
services: vpn-gateway
titleSuffix: Azure VPN Gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 11/30/2020
ms.author: cherylmc
ms.openlocfilehash: 2fc12385c78135269b6a73038fd0ad810ebaedd6
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96576168"
---
# <a name="configure-vpn-gateway-transit-for-virtual-network-peering"></a>Configurer le transit par passerelle VPN pour le peering de réseaux virtuels

Cet article explique comment configurer le transit par passerelle pour le peering de réseaux virtuels.Cet article explique comment configurer le transit par passerelle pour le peering de réseaux virtuels. Le [peering de réseaux virtuels](../virtual-network/virtual-network-peering-overview.md) connecte en toute transparence deux réseaux virtuels Azure de manière à les fusionner en un seul réseau virtuel à des fins de connectivité. Le [transit par passerelle](../virtual-network/virtual-network-peering-overview.md#gateways-and-on-premises-connectivity) est une propriété de peering qui permet à un réseau virtuel d'utiliser la passerelle VPN du réseau virtuel appairé pour la mise en œuvre d'une connectivité intersite ou de réseau virtuel à réseau virtuel. Le diagramme suivant illustre le fonctionnement du transit par passerelle avec le peering de réseaux virtuels.

![Diagramme : Transit par passerelle](./media/vpn-gateway-peering-gateway-transit/gatewaytransit.png)

Dans le diagramme, le transit par passerelle permet aux réseaux virtuels homologués d’utiliser la passerelle VPN du réseau virtuel Hub-RM. La connectivité disponible sur la passerelle VPN, notamment les connexions S2S, P2S et de réseau virtuel à réseau virtuel, s’applique aux trois réseaux virtuels. L'option de transit est disponible pour le peering entre des modèles de déploiement identiques ou différents. Si vous configurez le transit entre des modèles de déploiement différents, le réseau virtuel hub et la passerelle de réseau virtuel doivent se trouver dans le modèle de déploiement Resource Manager, et non dans le modèle de déploiement classique.
>

Dans l’architecture réseau hub-and-spoke, le transit par passerelle permet aux réseaux virtuels spoke d’exploiter la passerelle VPN du hub, évitant ainsi d’avoir à déployer des passerelles VPN dans chaque réseau virtuel spoke. Les itinéraires vers les réseaux locaux ou les réseaux virtuels connectés à la passerelle sont propagés aux tables de routage pour les réseaux virtuels homologués à l’aide du transit par passerelle. Vous pouvez désactiver la propagation automatique des itinéraires à partir de la passerelle VPN. Créez une table de routage avec l’option **Désactiver la propagation des itinéraires BGP** et associez la table de routage aux sous-réseaux afin d’empêcher la distribution des itinéraires à ces derniers. Pour plus d’informations, consultez [Virtual network routing table](../virtual-network/manage-route-table.md) (Table de routage de réseau virtuel).

Cet article présente deux scénarios :

* **Modèle de déploiement identique** : les deux réseaux virtuels sont créés dans le modèle de déploiement Resource Manager.
* **Modèles de déploiement différents** : le réseau virtuel spoke est créé dans le modèle de déploiement classique tandis que le réseau virtuel hub et la passerelle se trouvent dans le modèle de déploiement Resource Manager.

>[!NOTE]
> Si vous apportez un changement à la topologie de votre réseau et que vous avez des clients VPN Windows, vous devez retélécharger et réinstaller le package client VPN pour les clients Windows afin d’appliquer ce changement au client.
>

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous disposez des réseaux virtuels et des autorisations ci-dessous :

### <a name="virtual-networks"></a><a name="vnet"></a>Réseaux virtuels

|Réseau virtuel|Modèle de déploiement| Passerelle de réseau virtuel|
|---|---|---|---|
| Hub-RM| [Resource Manager](vpn-gateway-howto-site-to-site-resource-manager-portal.md)| [Oui](tutorial-create-gateway-portal.md)|
| Spoke-RM | [Resource Manager](vpn-gateway-howto-site-to-site-resource-manager-portal.md)| No |
| Spoke-Classic | [Classique](vpn-gateway-howto-site-to-site-classic-portal.md#CreatVNet) | No |

### <a name="permissions"></a><a name="permissions"></a>Autorisations

Les comptes que vous utilisez pour créer un peering de réseaux virtuels doivent être dotés des autorisations ou des rôles nécessaires. Dans l'exemple ci-dessous, si vous avez effectué un peering entre les deux réseaux virtuels nommés **Hub-RM** et **Spoke-Classic**, votre compte doit disposer des autorisations ou des rôles suivants pour chaque réseau virtuel :

|Réseau virtuel|Modèle de déploiement|Role|Autorisations|
|---|---|---|---|
|Hub-RM|Gestionnaire de ressources|[Contributeur de réseau](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)|Microsoft.Network/virtualNetworks/virtualNetworkPeerings/write|
| |Classique|[Contributeur de réseau classique](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#classic-network-contributor)|N/A|
|Spoke-Classic|Gestionnaire de ressources|[Contributeur de réseau](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)|Microsoft.Network/virtualNetworks/peer|
||Classique|[Contributeur de réseau classique](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#classic-network-contributor)|Microsoft.ClassicNetwork/virtualNetworks/peer|

Apprenez-en davantage sur les [rôles intégrés](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) et l’affectation d’autorisations spécifiques aux [rôles personnalisés](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (Gestionnaire des ressources uniquement).

## <a name="same-deployment-model"></a><a name="same"></a>Modèle de déploiement identique

Dans ce scénario, les réseaux virtuels se trouvent tous les deux dans le modèle de déploiement Resource Manager. Procédez comme suit pour créer ou mettre à jour les peerings de réseaux virtuels de manière à activer le transit par passerelle.

### <a name="to-add-a-peering-and-enable-transit"></a>Pour ajouter un peering et activer le transit

1. Sur le [portail Azure](https://portal.azure.com), créez ou mettez à jour le peering de réseaux virtuels à partir du réseau virtuel Hub-RM. Accédez au réseau virtuel **Hub-RM**. Sélectionnez **Peerings**, puis **+Ajouter** pour ouvrir **Ajouter un peering**.
1. Sur la page **Ajouter un peering**, configurez les valeurs de **Ce réseau virtuel**.

   * Nom du lien de peering : nommez le lien. Exemple : **HubRMToSpokeRM**
   * Trafic vers le réseau virtuel distant : **Autoriser**
   * Trafic transféré à partir du réseau virtuel distant : **Autoriser**
   * Passerelle de réseau virtuel : **Utiliser la passerelle de ce réseau virtuel**

     :::image type="content" source="./media/vpn-gateway-peering-gateway-transit/peering-vnet.png" alt-text="Capture d'écran illustrant l'ajout d'un peering.":::

1. Sur la même page, configurez les valeurs du **réseau virtuel distant**.

   * Nom du lien de peering : nommez le lien. Exemple : **SpokeRMtoHubRM**
   * Modèle de déploiement : **Resource Manager**
   * Réseau virtuel : **Spoke-RM**
   * Trafic vers le réseau virtuel distant : **Autoriser**
   * Trafic transféré à partir du réseau virtuel distant : **Autoriser**
   * Passerelle de réseau virtuel : **Utiliser la passerelle du réseau virtuel distant**

     :::image type="content" source="./media/vpn-gateway-peering-gateway-transit/peering-remote.png" alt-text="Capture d'écran présentant les valeurs du réseau virtuel distant.":::

1. Sélectionnez **Ajouter** pour créer le peering.
1. Vérifiez que l'état du peering est défini sur **Connecté** pour les deux réseaux virtuels.

### <a name="to-modify-an-existing-peering-for-transit"></a>Pour modifier un peering existant en vue d'un transit

Si le peering a déjà été créé, vous pouvez le modifier pour le transit.

1. Accédez au réseau virtuel. Sélectionnez **Peerings** et choisissez le peering que vous souhaitez modifier.

   :::image type="content" source="./media/vpn-gateway-peering-gateway-transit/peering-modify.png" alt-text="Capture d'écran illustrant une sélection de peerings.":::

1. Mettez à jour le peering de réseaux virtuels.

   * Trafic vers le réseau virtuel distant : **Autoriser**
   * Trafic transféré vers le réseau virtuel ; **Autoriser**
   * Passerelle de réseau virtuel : **Utiliser la passerelle du réseau virtuel distant**

     :::image type="content" source="./media/vpn-gateway-peering-gateway-transit/modify-peering-settings.png" alt-text="Capture d'écran illustrant la modification de la passerelle de peering.":::

1. **Enregistrez** les paramètres de peering.

### <a name="powershell-sample"></a><a name="ps-same"></a>Exemple de code PowerShell

Vous pouvez également utiliser PowerShell pour créer ou mettre à jour le peering avec l’exemple ci-dessous. Remplacez les variables par le nom de vos réseaux virtuels et de vos groupes de ressources.

```azurepowershell-interactive
$SpokeRG = "SpokeRG1"
$SpokeRM = "Spoke-RM"
$HubRG   = "HubRG1"
$HubRM   = "Hub-RM"

$spokermvnet = Get-AzVirtualNetwork -Name $SpokeRM -ResourceGroup $SpokeRG
$hubrmvnet   = Get-AzVirtualNetwork -Name $HubRM -ResourceGroup $HubRG

Add-AzVirtualNetworkPeering `
  -Name SpokeRMtoHubRM `
  -VirtualNetwork $spokermvnet `
  -RemoteVirtualNetworkId $hubrmvnet.Id `
  -UseRemoteGateways

Add-AzVirtualNetworkPeering `
  -Name HubRMToSpokeRM `
  -VirtualNetwork $hubrmvnet `
  -RemoteVirtualNetworkId $spokermvnet.Id `
  -AllowGatewayTransit
```

## <a name="different-deployment-models"></a><a name="different"></a>Modèles de déploiement différents

Dans cette configuration, le réseau virtuel spoke **Spoke-Classic** se trouve dans le modèle de déploiement classique tandis que le réseau virtuel hub **Hub-RM** se trouve dans le modèle de déploiement Resource Manager. Lors de la configuration du transit entre les modèles de déploiement, la passerelle de réseau virtuel doit être configurée pour le réseau virtuel Resource Manager, et non pour le réseau virtuel classique.

Pour cette configuration, il vous suffit de configurer le réseau virtuel **​​Hub-RM**. Vous n'avez rien à configurer sur le réseau virtuel **​​Spoke-Classic**.

1. Sur le portail Azure, accédez au réseau virtuel **​​Hub-RM**, sélectionnez **Peerings**, puis sélectionnez **+Ajouter**.
1. Sur la page **Ajouter un peering**, configurez les valeurs suivantes :

   * Nom du lien de peering : nommez le lien. Exemple : **HubRMToClassic**
   * Trafic vers le réseau virtuel distant : **Autoriser**
   * Trafic transféré à partir du réseau virtuel distant : **Autoriser**
   * Passerelle de réseau virtuel : **Utiliser la passerelle de ce réseau virtuel**
   * Réseau virtuel distant : **Classique**

     :::image type="content" source="./media/vpn-gateway-peering-gateway-transit/peering-classic.png" alt-text="Ajouter une page de peering pour Spoke-Classic":::

1. Vérifiez que l'abonnement est correct, puis sélectionnez le réseau virtuel dans la liste déroulante.
1. Sélectionnez **Ajouter** pour ajouter le peering.
1. Vérifiez que l'état de peering est défini sur **Connecté** pour le réseau virtuel Hub-RM. 

Pour cette configuration, vous n'avez rien à configurer sur le réseau virtuel **​​Spoke-Classic**. Une fois que l'état indique **Connecté**, le réseau virtuel spoke peut utiliser la connectivité via la passerelle VPN du réseau virtuel hub.

### <a name="powershell-sample"></a><a name="ps-different"></a>Exemple de code PowerShell

Vous pouvez également utiliser PowerShell pour créer ou mettre à jour le peering avec l’exemple ci-dessous. Remplacez les variables par les valeurs de votre réseau virtuel et de votre groupe de ressources, et « subscription ID » par votre ID d’abonnement. Vous devez uniquement créer le peering de réseaux virtuels sur le réseau virtuel hub.

```azurepowershell-interactive
$HubRG   = "HubRG1"
$HubRM   = "Hub-RM"

$hubrmvnet   = Get-AzVirtualNetwork -Name $HubRM -ResourceGroup $HubRG

Add-AzVirtualNetworkPeering `
  -Name HubRMToClassic `
  -VirtualNetwork $hubrmvnet `
  -RemoteVirtualNetworkId "/subscriptions/<subscription Id>/resourceGroups/Default-Networking/providers/Microsoft.ClassicNetwork/virtualNetworks/Spoke-Classic" `
  -AllowGatewayTransit
```

## <a name="next-steps"></a>Étapes suivantes

* Prenez connaissance des [comportements et contraintes importants du peering de réseaux virtuels](../virtual-network/virtual-network-manage-peering.md#requirements-and-constraints) et les [paramètres de peering de réseaux virtuels](../virtual-network/virtual-network-manage-peering.md#create-a-peering) avant d’en créer un pour une utilisation en production.
* Découvrez comment [créer une topologie de réseau de type hub et spoke](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke#virtual-network-peering) avec le peering de réseaux virtuels et le transit par passerelle.
* [Créez un peering de réseaux virtuels avec le même modèle de déploiement](../virtual-network/tutorial-connect-virtual-networks-portal.md).
* [Créez un peering de réseaux virtuels avec des modèles de déploiement différents](../virtual-network/create-peering-different-deployment-models.md).
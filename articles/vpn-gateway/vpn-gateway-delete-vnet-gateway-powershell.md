---
title: 'Passerelle VPN Azure : Supprimer une passerelle : PowerShell'
description: Supprimez une passerelle de réseau virtuel avec PowerShell dans le modèle de déploiement Resource Manager.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.date: 09/03/2020
ms.author: cherylmc
ms.topic: how-to
ms.openlocfilehash: 48e0998719ea19c0b360b50d8c9171d2b448ad19
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89440760"
---
# <a name="delete-a-virtual-network-gateway-using-powershell"></a>Supprimer une passerelle de réseau virtuel avec PowerShell
> [!div class="op_single_selector"]
> * [Azure portal](vpn-gateway-delete-vnet-gateway-portal.md)
> * [PowerShell](vpn-gateway-delete-vnet-gateway-powershell.md)
> * [PowerShell (classique)](vpn-gateway-delete-vnet-gateway-classic-powershell.md)
>
>

Deux approches sont possibles afin de supprimer une passerelle de réseau virtuel pour une configuration de passerelle VPN.

- Si vous voulez tout supprimer et recommencer, comme dans le cas d’un environnement de test, vous pouvez supprimer le groupe de ressources. Supprimer un groupe de ressources supprime toutes les ressources du groupe. Cette méthode est recommandée seulement si vous ne voulez conserver aucune des ressources du groupe de ressources. Vous ne pouvez pas choisir de supprimer uniquement certaines ressources avec cette approche.

- Si vous souhaitez conserver certaines ressources de votre groupe de ressources, la suppression d’une passerelle de réseau virtuel devient légèrement plus complexe. Avant de supprimer la passerelle de réseau virtuel, vous devez commencer par supprimer toutes les ressources qui dépendent de la passerelle. Les étapes à suivre dépendent du type de connexions que vous avez créées et des ressources dépendantes de chaque connexion.

## <a name="before-beginning"></a>Avant tout chose



### <a name="1-download-the-latest-azure-resource-manager-powershell-cmdlets"></a>1. Téléchargez la dernière version des applets de commande PowerShell Azure Resource Manager.

Téléchargez et installez la dernière version des applets de commande PowerShell Azure Resource Manager. Pour plus d’informations sur le téléchargement et l’installation des applets de commande PowerShell, consultez la page [Guide pratique d’installation et de configuration d’Azure PowerShell](/powershell/azure/).

### <a name="2-connect-to-your-azure-account"></a>2. Connectez-vous à votre compte Azure.

Ouvrez la console PowerShell et connectez-vous à votre compte. Utilisez l’exemple suivant pour faciliter votre connexion :

```powershell
Connect-AzAccount
```

Vérifiez les abonnements associés au compte.

```powershell
Get-AzSubscription
```

Si vous avez plusieurs abonnements, spécifiez celui que vous souhaitez utiliser.

```powershell
Select-AzSubscription -SubscriptionName "Replace_with_your_subscription_name"
```

## <a name="delete-a-site-to-site-vpn-gateway"></a><a name="S2S"></a>Supprimer une passerelle VPN de site à site

Pour supprimer une passerelle de réseau virtuel pour une configuration S2S, vous devez d’abord supprimer chaque ressource liée à la passerelle de réseau virtuel. Les ressources doivent être supprimées dans un certain ordre en raison des dépendances. Quand vous travaillez avec les exemples ci-dessous, certaines valeurs doivent être spécifiées, tandis que d’autres sont des résultats en sortie. Nous utilisons les valeurs suivantes dans les exemples à des fins de démonstration :

Nom du réseau virtuel : VNet1<br>
Nom du groupe de ressources : RG1<br>
Nom de la passerelle de réseau virtuel : GW1<br>

Les étapes suivantes s’appliquent au modèle de déploiement Resource Manager.

### <a name="1-get-the-virtual-network-gateway-that-you-want-to-delete"></a>1. Récupérez la passerelle de réseau virtuel que vous souhaitez supprimer.

```powershell
$GW=get-Azvirtualnetworkgateway -Name "GW1" -ResourceGroupName "RG1"
```

### <a name="2-check-to-see-if-the-virtual-network-gateway-has-any-connections"></a>2. Vérifiez si la passerelle de réseau virtuel possède des connexions.

```powershell
get-Azvirtualnetworkgatewayconnection -ResourceGroupName "RG1" | where-object {$_.VirtualNetworkGateway1.Id -eq $GW.Id}
$Conns=get-Azvirtualnetworkgatewayconnection -ResourceGroupName "RG1" | where-object {$_.VirtualNetworkGateway1.Id -eq $GW.Id}
```

### <a name="3-delete-all-connections"></a>3. Supprimez toutes les connexions.

Vous serez peut-être invité à confirmer la suppression de chacune des connexions.

```powershell
$Conns | ForEach-Object {Remove-AzVirtualNetworkGatewayConnection -Name $_.name -ResourceGroupName $_.ResourceGroupName}
```

### <a name="4-delete-the-virtual-network-gateway"></a>4. Supprimez la passerelle de réseau virtuel.

Vous serez peut-être invité à confirmer la suppression de la passerelle. Si vous avez une configuration P2S sur ce réseau virtuel en plus de votre configuration S2S, la suppression de la passerelle de réseau virtuel déconnecte automatiquement tous les clients P2S sans avertissement.


```powershell
Remove-AzVirtualNetworkGateway -Name "GW1" -ResourceGroupName "RG1"
```

À ce stade, votre passerelle de réseau virtuel a été supprimée. Vous pouvez vous conformer aux étapes suivantes pour supprimer toutes les ressources qui ne sont plus utilisées.

### <a name="5-delete-the-local-network-gateways"></a>5\. Supprimez les passerelles de réseau local.

Récupérez la liste des passerelles de réseau local correspondantes.

```powershell
$LNG=Get-AzLocalNetworkGateway -ResourceGroupName "RG1" | where-object {$_.Id -In $Conns.LocalNetworkGateway2.Id}
```

Supprimez les passerelles de réseau local. Vous serez peut-être invité à confirmer la suppression de chacune des passerelles de réseau local.

```powershell
$LNG | ForEach-Object {Remove-AzLocalNetworkGateway -Name $_.Name -ResourceGroupName $_.ResourceGroupName}
```

### <a name="6-delete-the-public-ip-address-resources"></a>6. Supprimez les ressources de l’adresse IP publique.

Récupérez les configurations IP de la passerelle de réseau virtuel.

```powershell
$GWIpConfigs = $Gateway.IpConfigurations
```

Récupérez la liste des ressources d’adresse IP publique utilisées pour cette passerelle de réseau virtuel. Si la passerelle de réseau virtuel était active-active, vous verrez deux adresses IP publiques.

```powershell
$PubIP=Get-AzPublicIpAddress | where-object {$_.Id -In $GWIpConfigs.PublicIpAddress.Id}
```

Supprimez les ressources de l’adresse IP publique.

```powershell
$PubIP | foreach-object {remove-AzpublicIpAddress -Name $_.Name -ResourceGroupName "RG1"}
```

### <a name="7-delete-the-gateway-subnet-and-set-the-configuration"></a>7. Supprimez le sous-réseau de passerelle et définissez la configuration.

```powershell
$GWSub = Get-AzVirtualNetwork -ResourceGroupName "RG1" -Name "VNet1" | Remove-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet"
Set-AzVirtualNetwork -VirtualNetwork $GWSub
```

## <a name="delete-a-vnet-to-vnet-vpn-gateway"></a><a name="v2v"></a>Supprimer une passerelle VPN de réseau virtuel à réseau virtuel

Si vous souhaitez supprimer une passerelle de réseau virtuel pour une configuration V2V, vous devez d’abord supprimer chaque ressource liée à la passerelle de réseau virtuel. Les ressources doivent être supprimées dans un certain ordre en raison des dépendances. Quand vous travaillez avec les exemples ci-dessous, certaines valeurs doivent être spécifiées, tandis que d’autres sont des résultats en sortie. Nous utilisons les valeurs suivantes dans les exemples à des fins de démonstration :

Nom du réseau virtuel : VNet1<br>
Nom du groupe de ressources : RG1<br>
Nom de la passerelle de réseau virtuel : GW1<br>

Les étapes suivantes s’appliquent au modèle de déploiement Resource Manager.

### <a name="1-get-the-virtual-network-gateway-that-you-want-to-delete"></a>1. Récupérez la passerelle de réseau virtuel que vous souhaitez supprimer.

```powershell
$GW=get-Azvirtualnetworkgateway -Name "GW1" -ResourceGroupName "RG1"
```

### <a name="2-check-to-see-if-the-virtual-network-gateway-has-any-connections"></a>2. Vérifiez si la passerelle de réseau virtuel possède des connexions.

```powershell
get-Azvirtualnetworkgatewayconnection -ResourceGroupName "RG1" | where-object {$_.VirtualNetworkGateway1.Id -eq $GW.Id}
```
 
Il peut y avoir d’autres connexions à la passerelle de réseau virtuel qui font partie d’un autre groupe de ressources. Recherchez les connexions supplémentaires dans chacun des autres groupes de ressources. Dans cet exemple, nous vérifions les connexions à partir de RG2. Exécutez cette commande pour chacun de vos groupes de ressources qui sont susceptibles d’avoir une connexion à la passerelle de réseau virtuel.

```powershell
get-Azvirtualnetworkgatewayconnection -ResourceGroupName "RG2" | where-object {$_.VirtualNetworkGateway2.Id -eq $GW.Id}
```

### <a name="3-get-the-list-of-connections-in-both-directions"></a>3. Récupérez la liste des connexions dans les deux sens.

Dans la mesure où il s’agit d’une configuration de réseau virtuel à réseau virtuel, vous avez besoin de la liste des connexions dans les deux sens.

```powershell
$ConnsL=get-Azvirtualnetworkgatewayconnection -ResourceGroupName "RG1" | where-object {$_.VirtualNetworkGateway1.Id -eq $GW.Id}
```
 
Dans cet exemple, nous vérifions les connexions à partir de RG2. Exécutez cette commande pour chacun de vos groupes de ressources qui sont susceptibles d’avoir une connexion à la passerelle de réseau virtuel.

```powershell
 $ConnsR=get-Azvirtualnetworkgatewayconnection -ResourceGroupName "<NameOfResourceGroup2>" | where-object {$_.VirtualNetworkGateway2.Id -eq $GW.Id}
 ```

### <a name="4-delete-all-connections"></a>4. Supprimez toutes les connexions.

Vous serez peut-être invité à confirmer la suppression de chacune des connexions.

```powershell
$ConnsL | ForEach-Object {Remove-AzVirtualNetworkGatewayConnection -Name $_.name -ResourceGroupName $_.ResourceGroupName}
$ConnsR | ForEach-Object {Remove-AzVirtualNetworkGatewayConnection -Name $_.name -ResourceGroupName $_.ResourceGroupName}
```

### <a name="5-delete-the-virtual-network-gateway"></a>5. Supprimez la passerelle de réseau virtuel.

Vous serez peut-être invité à confirmer la suppression de la passerelle de réseau virtuel. Si vous avez des configurations P2S sur vos réseaux virtuels en plus de votre configuration V2V, la suppression des passerelles de réseau virtuel déconnecte automatiquement tous les clients P2S sans avertissement.

```powershell
Remove-AzVirtualNetworkGateway -Name "GW1" -ResourceGroupName "RG1"
```

À ce stade, votre passerelle de réseau virtuel a été supprimée. Vous pouvez vous conformer aux étapes suivantes pour supprimer toutes les ressources qui ne sont plus utilisées.

### <a name="6-delete-the-public-ip-address-resources"></a>6. Supprimez les ressources de l’adresse IP publique

Récupérez les configurations IP de la passerelle de réseau virtuel.

```powershell
$GWIpConfigs = $Gateway.IpConfigurations
```

Récupérez la liste des ressources d’adresse IP publique utilisées pour cette passerelle de réseau virtuel. Si la passerelle de réseau virtuel était active-active, vous verrez deux adresses IP publiques.

```powershell
$PubIP=Get-AzPublicIpAddress | where-object {$_.Id -In $GWIpConfigs.PublicIpAddress.Id}
```

Supprimez les ressources de l’adresse IP publique. Vous serez peut-être invité à confirmer la suppression de l’adresse IP publique.

```powershell
$PubIP | foreach-object {remove-AzpublicIpAddress -Name $_.Name -ResourceGroupName "<NameOfResourceGroup1>"}
```

### <a name="7-delete-the-gateway-subnet-and-set-the-configuration"></a>7. Supprimez le sous-réseau de passerelle et définissez la configuration.

```powershell
$GWSub = Get-AzVirtualNetwork -ResourceGroupName "RG1" -Name "VNet1" | Remove-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet"
Set-AzVirtualNetwork -VirtualNetwork $GWSub
```

## <a name="delete-a-point-to-site-vpn-gateway"></a><a name="deletep2s"></a>Suppression d’une passerelle VPN de point à site

Pour supprimer une passerelle de réseau virtuel pour une configuration P2S, vous devez d’abord supprimer chaque ressource liée à la passerelle de réseau virtuel. Les ressources doivent être supprimées dans un certain ordre en raison des dépendances. Quand vous travaillez avec les exemples ci-dessous, certaines valeurs doivent être spécifiées, tandis que d’autres sont des résultats en sortie. Nous utilisons les valeurs suivantes dans les exemples à des fins de démonstration :

Nom du réseau virtuel : VNet1<br>
Nom du groupe de ressources : RG1<br>
Nom de la passerelle de réseau virtuel : GW1<br>

Les étapes suivantes s’appliquent au modèle de déploiement Resource Manager.


>[!NOTE]
> Lorsque vous supprimez la passerelle VPN, tous les clients connectés sont déconnectés du réseau virtuel sans avertissement.
>
>

### <a name="1-get-the-virtual-network-gateway-that-you-want-to-delete"></a>1. Récupérez la passerelle de réseau virtuel que vous souhaitez supprimer.

```powershell
$GW=get-Azvirtualnetworkgateway -Name "GW1" -ResourceGroupName "RG1"
```

### <a name="2-delete-the-virtual-network-gateway"></a>2. Supprimez la passerelle de réseau virtuel.

Vous serez peut-être invité à confirmer la suppression de la passerelle de réseau virtuel.

```powershell
Remove-AzVirtualNetworkGateway -Name "GW1" -ResourceGroupName "RG1"
```

À ce stade, votre passerelle de réseau virtuel a été supprimée. Vous pouvez vous conformer aux étapes suivantes pour supprimer toutes les ressources qui ne sont plus utilisées.

### <a name="3-delete-the-public-ip-address-resources"></a>3. Supprimez les ressources de l’adresse IP publique

Récupérez les configurations IP de la passerelle de réseau virtuel.

```powershell
$GWIpConfigs = $Gateway.IpConfigurations
```

Récupérez la liste des adresses IP publiques utilisées pour cette passerelle de réseau virtuel. Si la passerelle de réseau virtuel était active-active, vous verrez deux adresses IP publiques.

```powershell
$PubIP=Get-AzPublicIpAddress | where-object {$_.Id -In $GWIpConfigs.PublicIpAddress.Id}
```

Supprimez les adresses IP publiques. Vous serez peut-être invité à confirmer la suppression de l’adresse IP publique.

```powershell
$PubIP | foreach-object {remove-AzpublicIpAddress -Name $_.Name -ResourceGroupName "<NameOfResourceGroup1>"}
```

### <a name="4-delete-the-gateway-subnet-and-set-the-configuration"></a>4. Supprimez le sous-réseau de passerelle et définissez la configuration.

```powershell
$GWSub = Get-AzVirtualNetwork -ResourceGroupName "RG1" -Name "VNet1" | Remove-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet"
Set-AzVirtualNetwork -VirtualNetwork $GWSub
```

## <a name="delete-a-vpn-gateway-by-deleting-the-resource-group"></a><a name="delete"></a>Supprimer une passerelle VPN en supprimant le groupe de ressources

Si vous n’avez pas besoin de conserver de ressources dans le groupe de ressources et que vous voulez simplement recommencer à zéro, vous pouvez supprimer tout un groupe de ressources. Il s’agit d’un moyen rapide de tout supprimer. Les étapes suivantes s’appliquent uniquement au modèle de déploiement Resource Manager.

### <a name="1-get-a-list-of-all-the-resource-groups-in-your-subscription"></a>1. Récupérez la liste des groupes de ressources de votre abonnement.

```powershell
Get-AzResourceGroup
```

### <a name="2-locate-the-resource-group-that-you-want-to-delete"></a>2. Recherchez le groupe de ressources que vous souhaitez supprimer.

Localisez le groupe de ressources que vous souhaitez supprimer et affichez la liste des ressources qu’il contient. Dans l’exemple, le nom du groupe de ressources est RG1. Modifiez l’exemple pour récupérer la liste de toutes les ressources.

```powershell
Find-AzResource -ResourceGroupNameContains RG1
```

### <a name="3-verify-the-resources-in-the-list"></a>3. Vérifiez les ressources de la liste.

Lorsque la liste est renvoyée, passez-la en revue afin de vérifier que vous souhaitez supprimer toutes les ressources du groupe de ressources, ainsi que le groupe de ressources lui-même. Si vous voulez conserver certaines des ressources dans le groupe de ressources, utilisez les étapes des sections précédentes de cet article pour supprimer votre passerelle.

### <a name="4-delete-the-resource-group-and-resources"></a>4. Supprimez le groupe de ressources et les ressources.

Pour supprimer le groupe de ressources et toutes les ressources qu’il contient, modifiez l’exemple et exécutez-le.

```powershell
Remove-AzResourceGroup -Name RG1
```

### <a name="5-check-the-status"></a>5. Vérifiez l’état.

Azure met quelque temps à supprimer toutes les ressources. Vous pouvez vérifier l’état de votre groupe de ressources avec cette applet de commande.

```powershell
Get-AzResourceGroup -ResourceGroupName RG1
```

Le résultat retourné indique « Réussi ».

```
ResourceGroupName : RG1
Location          : eastus
ProvisioningState : Succeeded
```

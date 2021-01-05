---
title: Fichier Include
description: Fichier Include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 02/14/2019
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 76a602ae722bd975e634631819ebc703e8896c98
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96028269"
---
### <a name="to-modify-local-network-gateway-ip-address-prefixes---no-gateway-connection"></a><a name="noconnection"></a>Pour modifier des préfixes d’adresses IP de passerelle de réseau local - sans connexion de passerelle

Pour ajouter des préfixes d’adresses :

1. Définissez la variable pour LocalNetworkGateway.

   ```azurepowershell-interactive
   $local = Get-AzLocalNetworkGateway -Name Site1 -ResourceGroupName TestRG1
   ```
2. Modifiez les préfixes.

   ```azurepowershell-interactive
   Set-AzLocalNetworkGateway -LocalNetworkGateway $local `
   -AddressPrefix @('10.101.0.0/24','10.101.1.0/24','10.101.2.0/24')
   ```

Pour supprimer des préfixes d’adresses :

  Abandonnez les préfixes dont vous n'avez plus besoin. Dans cet exemple, le préfixe 10.101.2.0/24 (de l’exemple précédent) n’est plus nécessaire. Nous allons donc modifier la passerelle de réseau local et exclure ce préfixe.

1. Définissez la variable pour LocalNetworkGateway.

   ```azurepowershell-interactive
   $local = Get-AzLocalNetworkGateway -Name Site1 -ResourceGroupName TestRG1
   ```
2. Définissez la passerelle avec les préfixes mis à jour.

   ```azurepowershell-interactive
   Set-AzLocalNetworkGateway -LocalNetworkGateway $local `
   -AddressPrefix @('10.101.0.0/24','10.101.1.0/24')
   ```

### <a name="to-modify-local-network-gateway-ip-address-prefixes---existing-gateway-connection"></a><a name="withconnection"></a>Pour modifier des préfixes d’adresses IP de passerelle de réseau local - avec une connexion de passerelle existante

Si vous disposez d’une connexion de passerelle et que vous souhaitez ajouter ou supprimer des préfixes d’adresses IP contenues dans votre passerelle de réseau local, vous devez suivre les étapes suivantes dans l’ordre. Cela entraînera une interruption de votre connexion VPN. Lorsque vous modifiez des préfixes d’adresses IP, vous n’avez pas besoin de supprimer la passerelle VPN. Vous devez uniquement supprimer la connexion.

1. Supprimez la connexion.

   ```azurepowershell-interactive
   Remove-AzVirtualNetworkGatewayConnection -Name VNet1toSite1 -ResourceGroupName TestRG1
   ```
2. Définissez la passerelle de réseau local avec les préfixes d’adresse modifiés.
   
   Définissez la variable pour LocalNetworkGateway.

   ```azurepowershell-interactive
   $local = Get-AzLocalNetworkGateway -Name Site1 -ResourceGroupName TestRG1
   ```
   
   Modifiez les préfixes.
   
   ```azurepowershell-interactive
   Set-AzLocalNetworkGateway -LocalNetworkGateway $local `
   -AddressPrefix @('10.101.0.0/24','10.101.1.0/24')
   ```
3. Créez la connexion. Dans cet exemple, nous allons configurer un type de connexion IPsec. Lorsque vous recréez votre connexion, utilisez le type de connexion spécifié pour votre configuration. Pour les autres types de connexion, consultez la page sur [les applets de commande PowerShell](/powershell/module/Azurerm.Network/New-AzureRmVirtualNetworkGatewayConnection) .
   
   Définissez la variable pour VirtualNetworkGateway.

   ```azurepowershell-interactive
   $gateway1 = Get-AzVirtualNetworkGateway -Name VNet1GW  -ResourceGroupName TestRG1
   ```
   
   Créez la connexion. Cet exemple utilise la variable $local que vous avez définie à l’étape 2.

   ```azurepowershell-interactive
   New-AzVirtualNetworkGatewayConnection -Name VNet1toSite1 `
   -ResourceGroupName TestRG1 -Location 'East US' `
   -VirtualNetworkGateway1 $gateway1 -LocalNetworkGateway2 $local `
   -ConnectionType IPsec `
   -RoutingWeight 10 -SharedKey 'abc123'
   ```
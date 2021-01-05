---
title: 'Azure ExpressRoute : Configurer MACsec'
description: Cet article vous aide à configurer MACsec pour sécuriser les connexions entre vos routeurs de périphérie et les routeurs de périphérie de Microsoft.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 10/22/2019
ms.author: duau
ms.openlocfilehash: 8ee7f42a4b05def7c2239c7a1e5bcef54f6f7e50
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89395772"
---
# <a name="configure-macsec-on-expressroute-direct-ports"></a>Configurer MACsec sur les ports ExpressRoute Direct

Cet article vous aide à configurer MACsec pour sécuriser les connexions entre vos routeurs de périphérie et les routeurs de périphérie de Microsoft à l’aide de PowerShell.

## <a name="before-you-begin"></a>Avant de commencer

Avant de commencer la configuration, vérifiez les éléments suivants :

* Vous comprenez les [workflows de provisionnement d’ExpressRoute Direct](expressroute-erdirect-about.md).
* Vous avez créé une [ressource de port ExpressRoute Direct](expressroute-howto-erdirect.md).
* Si vous souhaitez exécuter PowerShell localement, vérifiez que la dernière version d’Azure PowerShell est installée sur votre ordinateur.

### <a name="working-with-azure-powershell"></a>Utilisation d’Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]

[!INCLUDE [expressroute-cloudshell](../../includes/expressroute-cloudshell-powershell-about.md)]

### <a name="sign-in-and-select-the-right-subscription"></a>Se connecter et sélectionner l’abonnement approprié

Pour commencer la configuration, connectez-vous à votre compte Azure et sélectionnez l’abonnement que vous souhaitez utiliser.

   [!INCLUDE [sign in](../../includes/expressroute-cloud-shell-connect.md)]

## <a name="1-create-azure-key-vault-macsec-secrets-and-user-identity"></a>1. Créer une instance Azure Key Vault, les secrets MACsec et une identité d’utilisateur

1. Créez une instance Key Vault pour stocker les secrets MACsec dans un nouveau groupe de ressources.

    ```azurepowershell-interactive
    New-AzResourceGroup -Name "your_resource_group" -Location "resource_location"
    $keyVault = New-AzKeyVault -Name "your_key_vault_name" -ResourceGroupName "your_resource_group" -Location "resource_location" -EnableSoftDelete 
    ```

    Si vous avez déjà un coffre de clés ou un groupe de ressources, vous pouvez les réutiliser. Dans ce cas, vous devez impérativement activer la [fonctionnalité de **suppression réversible**](../key-vault/general/soft-delete-overview.md) sur votre coffre de clés existant. Si la suppression réversible n’est pas activée, vous pouvez l’activer à l’aide des commandes suivantes :

    ```azurepowershell-interactive
    ($resource = Get-AzResource -ResourceId (Get-AzKeyVault -VaultName "your_existing_keyvault").ResourceId).Properties | Add-Member -MemberType "NoteProperty" -Name "enableSoftDelete" -Value "true"
    Set-AzResource -resourceid $resource.ResourceId -Properties $resource.Properties
    ```
2. Créez une identité d’utilisateur.

    ```azurepowershell-interactive
    $identity = New-AzUserAssignedIdentity  -Name "identity_name" -Location "resource_location" -ResourceGroupName "your_resource_group"
    ```

    Si New-AzUserAssignedIdentity n’est pas reconnu comme une applet de commande PowerShell valide, installez le module suivant (en mode administrateur) et réexécutez la commande ci-dessus.

    ```azurepowershell-interactive
    Install-Module -Name Az.ManagedServiceIdentity
    ```
3. Créez une clé d’association de connectivité (CAK) et un nom de clé d’association de connectivité (CKN), puis stockez-les dans le coffre de clés.

    ```azurepowershell-interactive
    $CAK = ConvertTo-SecureString "your_key" -AsPlainText -Force
    $CKN = ConvertTo-SecureString "your_key_name" -AsPlainText -Force
    $MACsecCAKSecret = Set-AzKeyVaultSecret -VaultName "your_key_vault_name" -Name "CAK_name" -SecretValue $CAK
    $MACsecCKNSecret = Set-AzKeyVaultSecret -VaultName "your_key_vault_name" -Name "CKN_name" -SecretValue $CKN
    ```
4. Accordez l’autorisation GET à l’identité d’utilisateur.

    ```azurepowershell-interactive
    Set-AzKeyVaultAccessPolicy -VaultName "your_key_vault_name" -PermissionsToSecrets get -ObjectId $identity.PrincipalId
    ```

   Cette identité peut maintenant obtenir les secrets, par exemple CAK et CKN, à partir du coffre de clés.
5. Définissez cette identité d’utilisateur à utiliser par ExpressRoute.

    ```azurepowershell-interactive
    $erIdentity = New-AzExpressRoutePortIdentity -UserAssignedIdentityId $identity.Id
    ```
 
## <a name="2-configure-macsec-on-expressroute-direct-ports"></a>2. Configurer MACsec sur les ports ExpressRoute Direct

### <a name="to-enable-macsec"></a>Pour activer MACsec

Chaque instance ExpressRoute Direct a deux ports physiques. Vous pouvez choisir d’activer MACsec sur les deux ports en même temps ou sur un port à la fois. L’activation d’un port à la fois (en basculant le trafic sur un port actif pendant la maintenance de l’autre port) contribue à réduire le temps d’arrêt si ExpressRoute Direct est déjà en marche.

1. Définissez le chiffrement et les secrets MACsec, et associez l’identité d’utilisateur au port afin que le code de gestion d’ExpressRoute puisse accéder aux secrets MACsec dont il a besoin.

    ```azurepowershell-interactive
    $erDirect = Get-AzExpressRoutePort -ResourceGroupName "your_resource_group" -Name "your_direct_port_name"
    $erDirect.Links[0]. MacSecConfig.CknSecretIdentifier = $MacSecCKNSecret.Id
    $erDirect.Links[0]. MacSecConfig.CakSecretIdentifier = $MacSecCAKSecret.Id
    $erDirect.Links[0]. MacSecConfig.Cipher = "GcmAes256"
    $erDirect.Links[1]. MacSecConfig.CknSecretIdentifier = $MacSecCKNSecret.Id
    $erDirect.Links[1]. MacSecConfig.CakSecretIdentifier = $MacSecCAKSecret.Id
    $erDirect.Links[1]. MacSecConfig.Cipher = "GcmAes256"
    $erDirect.identity = $erIdentity
    Set-AzExpressRoutePort -ExpressRoutePort $erDirect
    ```
2. (Facultatif) Si les ports sont dans l’état d’administration Arrêté, vous pouvez exécuter les commandes suivantes pour redémarrer les ports.

    ```azurepowershell-interactive
    $erDirect = Get-AzExpressRoutePort -ResourceGroupName "your_resource_group" -Name "your_direct_port_name"
    $erDirect.Links[0].AdminState = "Enabled"
    $erDirect.Links[1].AdminState = "Enabled"
    Set-AzExpressRoutePort -ExpressRoutePort $erDirect
    ```

    À ce stade, MACsec est activé sur les ports ExpressRoute Direct côté Microsoft. Si vous ne l’avez pas configuré sur vos appareils de périphérie, vous pouvez le faire en utilisant les mêmes chiffrement et secrets MACsec.

### <a name="to-disable-macsec"></a>Pour désactiver MACsec

Si vous n’avez plus besoin de MACsec sur votre instance ExpressRoute Direct, vous pouvez le désactiver à l’aide des commandes suivantes.

```azurepowershell-interactive
$erDirect = Get-AzExpressRoutePort -ResourceGroupName "your_resource_group" -Name "your_direct_port_name"
$erDirect.Links[0]. MacSecConfig.CknSecretIdentifier = $null
$erDirect.Links[0]. MacSecConfig.CakSecretIdentifier = $null
$erDirect.Links[1]. MacSecConfig.CknSecretIdentifier = $null
$erDirect.Links[1]. MacSecConfig.CakSecretIdentifier = $null
$erDirect.identity = $null
Set-AzExpressRoutePort -ExpressRoutePort $erDirect
```

MACsec est maintenant désactivé sur les ports ExpressRoute Direct côté Microsoft.

### <a name="test-connectivity"></a>Tester la connectivité
Une fois que vous avez configuré MACsec (y compris la mise à jour de la clé MACsec) sur vos ports ExpressRoute Direct, [vérifiez](expressroute-troubleshooting-expressroute-overview.md) que les sessions BGP des circuits sont opérationnelles et actives. Si vous n’avez pas encore de circuit sur les ports, commencez par en créer un, puis configurez le peering privé Azure ou le peering Microsoft du circuit. Si MACsec n’est pas correctement configuré, notamment si les clés MACsec ne correspondent pas, entre vos appareils réseau et ceux de Microsoft, vous ne verrez pas la résolution ARP au niveau de la couche 2 ni l’établissement de session BGP au niveau de la couche 3. Si tout est configuré comme il faut, vous voyez les routes BGP publiées correctement dans les deux directions et vos données d’application circuler sur ExpressRoute.

## <a name="next-steps"></a>Étapes suivantes
1. [Créer un circuit ExpressRoute sur ExpressRoute Direct](expressroute-howto-erdirect.md)
2. [Lier un circuit ExpressRoute à un réseau virtuel Azure](expressroute-howto-linkvnet-arm.md)
3. [Vérifier la connectivité ExpressRoute](expressroute-troubleshooting-expressroute-overview.md)

---
title: 'Démarrage rapide : Diriger le trafic web à l’aide de PowerShell'
titleSuffix: Azure Application Gateway
description: Dans ce guide de démarrage rapide, découvrez comment utiliser Azure PowerShell pour créer une passerelle d’application Azure qui dirige le trafic web vers les machines virtuelles d’un pool de back-ends.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: quickstart
ms.date: 08/27/2020
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: 3f64086ed97594416b5964cf648c857c2f271480
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91331095"
---
# <a name="quickstart-direct-web-traffic-with-azure-application-gateway-using-azure-powershell"></a>Démarrage rapide : Diriger le trafic web avec Azure Application Gateway à l’aide d’Azure PowerShell

Dans le cadre de ce guide de démarrage rapide, vous allez utiliser Azure PowerShell pour créer une passerelle d’application. Puis, vous la testerez pour vous assurer qu’elle fonctionne correctement. 

La passerelle d’application dirige le trafic web des applications vers des ressources spécifiques d’un pool de back-ends. Vous attribuez des écouteurs aux ports, créez des règles et ajoutez des ressources à un pool de back-ends. Par souci de simplicité, cet article utilise une configuration simple avec une adresse IP front-end publique, un écouteur de base pour héberger un site unique sur cette passerelle d’application, une règle de routage des requêtes simple et deux machines virtuelles dans le pool de back-ends.

Vous pouvez également suivre ce guide de démarrage rapide en utilisant [Azure CLI](quick-create-cli.md) ou le [portail Azure](quick-create-portal.md).

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Azure PowerShell version 1.0.0 ou ultérieure](/powershell/azure/install-az-ps) (si vous exécutez Azure PowerShell localement).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="connect-to-azure"></a>Connexion à Azure

Pour vous connecter à Azure, exécutez `Connect-AzAccount`.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Dans Azure, vous allouez les ressources associées à un groupe de ressources. Vous pouvez utiliser un groupe de ressources existant ou en créer un.

Pour créer un groupe de ressources, utilisez l’applet de commande `New-AzResourceGroup` : 

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroupAG -Location eastus
```
## <a name="create-network-resources"></a>Créer des ressources réseau

Azure a besoin d’un réseau virtuel pour communiquer avec les différentes ressources que vous créez.  Le sous-réseau de passerelle d’application peut contenir uniquement des passerelles d’application. Aucune autre ressource n’est autorisée.  Vous pouvez créer un sous-réseau pour Application Gateway ou en utiliser qui existe déjà. Vous créez deux sous-réseaux dans cet exemple : un pour la passerelle d’application et un autre pour les serveurs back-end. Vous pouvez l’adresse IP frontale d’Application Gateway pour qu’elle soit publique ou privée conformément à votre cas d’utilisation. Dans cet exemple, vous allez choisir une adresse IP front-end publique.

1. Créez les configurations de sous-réseau en utilisant `New-AzVirtualNetworkSubnetConfig`.
2. Créez le réseau virtuel à l’aide des configurations de sous-réseau en utilisant `New-AzVirtualNetwork`. 
3. Créez l’adresse IP publique en utilisant `New-AzPublicIpAddress`. 
> [!NOTE]
> Les [stratégies de points de terminaison de service de réseau virtuel](../virtual-network/virtual-network-service-endpoint-policies-overview.md) ne sont pas prises en charge dans un sous-réseau Application Gateway.

```azurepowershell-interactive
$agSubnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name myAGSubnet `
  -AddressPrefix 10.0.1.0/24
$backendSubnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name myBackendSubnet `
  -AddressPrefix 10.0.2.0/24
New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -Name myVNet `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $agSubnetConfig, $backendSubnetConfig
New-AzPublicIpAddress `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -Name myAGPublicIPAddress `
  -AllocationMethod Static `
  -Sku Standard
```
## <a name="create-an-application-gateway"></a>Créer une passerelle Application Gateway

### <a name="create-the-ip-configurations-and-frontend-port"></a>Créer les configurations IP et le port frontal

1. Utilisez `New-AzApplicationGatewayIPConfiguration` pour créer la configuration qui associe le sous-réseau que vous avez créé à la passerelle d’application. 
2. Utilisez `New-AzApplicationGatewayFrontendIPConfig` pour créer la configuration qui attribue l’adresse IP publique que vous avez créée précédemment à la passerelle d’application. 
3. Utilisez `New-AzApplicationGatewayFrontendPort` pour configurer le port 80 comme port d’accès à la passerelle d’application.

```azurepowershell-interactive
$vnet   = Get-AzVirtualNetwork -ResourceGroupName myResourceGroupAG -Name myVNet
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name myAGSubnet
$pip    = Get-AzPublicIPAddress -ResourceGroupName myResourceGroupAG -Name myAGPublicIPAddress 
$gipconfig = New-AzApplicationGatewayIPConfiguration `
  -Name myAGIPConfig `
  -Subnet $subnet
$fipconfig = New-AzApplicationGatewayFrontendIPConfig `
  -Name myAGFrontendIPConfig `
  -PublicIPAddress $pip
$frontendport = New-AzApplicationGatewayFrontendPort `
  -Name myFrontendPort `
  -Port 80
```

### <a name="create-the-backend-pool"></a>Créer le pool principal

1. Utilisez `New-AzApplicationGatewayBackendAddressPool` pour créer le pool de back-ends pour la passerelle d’application. Le pool de back-ends est vide pour le moment. Quand vous allez créer les cartes réseau du serveur back-end dans la section suivante, vous les ajouterez au pool de back-ends.
2. Configurez les paramètres du pool de back-ends avec `New-AzApplicationGatewayBackendHttpSetting`.

```azurepowershell-interactive
$backendPool = New-AzApplicationGatewayBackendAddressPool `
  -Name myAGBackendPool
$poolSettings = New-AzApplicationGatewayBackendHttpSetting `
  -Name myPoolSettings `
  -Port 80 `
  -Protocol Http `
  -CookieBasedAffinity Enabled `
  -RequestTimeout 30
```

### <a name="create-the-listener-and-add-a-rule"></a>Créer l’écouteur et ajouter une règle

Azure nécessite un écouteur pour permettre à la passerelle d’application d’acheminer le trafic de manière appropriée vers le pool de back-ends. Azure nécessite également une règle pour que l’écouteur sache quel pool de back-ends utiliser pour le trafic entrant. 

1. Créez un écouteur à l’aide de `New-AzApplicationGatewayHttpListener` avec la configuration front-end et le port front-end que vous avez créés précédemment. 
2. Utilisez `New-AzApplicationGatewayRequestRoutingRule` pour créer une règle nommée *rule1*. 

```azurepowershell-interactive
$defaultlistener = New-AzApplicationGatewayHttpListener `
  -Name myAGListener `
  -Protocol Http `
  -FrontendIPConfiguration $fipconfig `
  -FrontendPort $frontendport
$frontendRule = New-AzApplicationGatewayRequestRoutingRule `
  -Name rule1 `
  -RuleType Basic `
  -HttpListener $defaultlistener `
  -BackendAddressPool $backendPool `
  -BackendHttpSettings $poolSettings
```

### <a name="create-the-application-gateway"></a>Créer la passerelle Application Gateway

Maintenant que vous avez créé les ressources de prise en charge nécessaires, créez la passerelle d’application :

1. Utilisez `New-AzApplicationGatewaySku` pour spécifier les paramètres de la passerelle d’application.
2. Utilisez `New-AzApplicationGateway` pour créer la passerelle d’application.

```azurepowershell-interactive
$sku = New-AzApplicationGatewaySku `
  -Name Standard_v2 `
  -Tier Standard_v2 `
  -Capacity 2
New-AzApplicationGateway `
  -Name myAppGateway `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -BackendAddressPools $backendPool `
  -BackendHttpSettingsCollection $poolSettings `
  -FrontendIpConfigurations $fipconfig `
  -GatewayIpConfigurations $gipconfig `
  -FrontendPorts $frontendport `
  -HttpListeners $defaultlistener `
  -RequestRoutingRules $frontendRule `
  -Sku $sku
```

### <a name="backend-servers"></a>Serveurs principaux

Maintenant que vous avez créé la passerelle d’application, créez les machines virtuelles back-end qui vont héberger les sites web. La partie principale peut se composer de cartes d’interface réseau, de groupes de machines virtuelles identiques, d’adresses IP publiques, d’adresses IP internes, de noms de domaine complets et de serveurs back-end multi-locataires comme Azure App Service. Dans cet exemple, vous créez deux machines virtuelles Azure à utiliser comme serveurs back-end pour la passerelle d’application. Vous installez également IIS sur les machines virtuelles pour vérifier qu’Azure a bien créé la passerelle d’application.

#### <a name="create-two-virtual-machines"></a>Créer deux machines virtuelles

1. Procurez-vous la configuration du pool de back-ends Application Gateway récemment créé avec `Get-AzApplicationGatewayBackendAddressPool`.
2. Créez une interface réseau avec `New-AzNetworkInterface`.
3. Créez une configuration de machine virtuelle avec `New-AzVMConfig`.
4. Créez la machine virtuelle avec `New-AzVM`.

Quand vous exécutez l’exemple de code suivant pour créer les machines virtuelles, Azure vous invite à entrer les informations d’identification. Entrez *azureuser* pour le nom d’utilisateur et un mot de passe :
    
```azurepowershell-interactive
$appgw = Get-AzApplicationGateway -ResourceGroupName myResourceGroupAG -Name myAppGateway
$backendPool = Get-AzApplicationGatewayBackendAddressPool -Name myAGBackendPool -ApplicationGateway $appgw
$vnet   = Get-AzVirtualNetwork -ResourceGroupName myResourceGroupAG -Name myVNet
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name myBackendSubnet
$cred = Get-Credential
for ($i=1; $i -le 2; $i++)
{
  $nic = New-AzNetworkInterface `
    -Name myNic$i `
    -ResourceGroupName myResourceGroupAG `
    -Location EastUS `
    -Subnet $subnet `
    -ApplicationGatewayBackendAddressPool $backendpool
  $vm = New-AzVMConfig `
    -VMName myVM$i `
    -VMSize Standard_DS2_v2
  Set-AzVMOperatingSystem `
    -VM $vm `
    -Windows `
    -ComputerName myVM$i `
    -Credential $cred
  Set-AzVMSourceImage `
    -VM $vm `
    -PublisherName MicrosoftWindowsServer `
    -Offer WindowsServer `
    -Skus 2016-Datacenter `
    -Version latest
  Add-AzVMNetworkInterface `
    -VM $vm `
    -Id $nic.Id
  Set-AzVMBootDiagnostic `
    -VM $vm `
    -Disable
  New-AzVM -ResourceGroupName myResourceGroupAG -Location EastUS -VM $vm
  Set-AzVMExtension `
    -ResourceGroupName myResourceGroupAG `
    -ExtensionName IIS `
    -VMName myVM$i `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.4 `
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}' `
    -Location EastUS
}
```

## <a name="test-the-application-gateway"></a>Tester la passerelle d’application

IIS n’est pas nécessaire pour créer la passerelle d’application, mais vous l’avez installé dans ce guide de démarrage rapide pour vérifier qu’Azure avait bien créé la passerelle d’application. Utilisez IIS pour tester la passerelle d’application :

1. Exécutez `Get-AzPublicIPAddress` pour récupérer l’adresse IP publique de la passerelle d’application. 
2. Copiez et collez l’adresse IP publique dans la barre d’adresse de votre navigateur. Quand vous actualisez le navigateur, vous devez voir s’afficher le nom de la machine virtuelle. Une réponse valide vérifie que la passerelle d’application a bien été créée avec succès et qu’elle est capable de se connecter au back-end.

```azurepowershell-interactive
Get-AzPublicIPAddress -ResourceGroupName myResourceGroupAG -Name myAGPublicIPAddress
```

![Tester la passerelle d’application](./media/quick-create-powershell/application-gateway-iistest.png)


## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin des ressources que vous avez créées avec la passerelle d’application, supprimez le groupe de ressources. Quand vous supprimez le groupe de ressources, vous supprimez aussi la passerelle d’application et toutes ses ressources associées. 

Pour supprimer le groupe de ressources, appelez l’applet de commande `Remove-AzResourceGroup` :

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroupAG
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Gérer le trafic web avec une passerelle d’application en utilisant Azure PowerShell](./tutorial-manage-web-traffic-powershell.md)


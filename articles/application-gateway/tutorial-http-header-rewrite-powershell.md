---
title: Créer une passerelle d’application Azure Application Gateway et réécrire des en-têtes HTTP
description: Cet article fournit des informations sur la façon de créer une passerelle d’application Azure Application Gateway et de réécrire les en-têtes HTTP à l’aide d’Azure PowerShell
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/19/2019
ms.author: absha
ms.openlocfilehash: 4a1a122eb7b5b0abcc47cd321c74267a1a4aecda
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93396853"
---
# <a name="create-an-application-gateway-and-rewrite-http-headers"></a>Créer une passerelle d’application et réécrire des en-têtes HTTP

Vous pouvez utiliser Azure PowerShell pour configurer des [règles de réécriture d’en-têtes de requête et de réponse HTTP](rewrite-http-headers.md) lorsque vous créez une [référence SKU de passerelle d’application se mettant à l’échelle automatiquement et redondante dans la zone](./application-gateway-autoscaling-zone-redundant.md)

Dans cet article, vous apprendrez comment :

* Créer un réseau virtuel avec mise à l'échelle automatique
* Créer une adresse IP publique réservée
* Configurer l'infrastructure de votre passerelle d'application
* Spécifier la configuration de règle de réécriture de votre en-tête HTTP
* Spécifier la mise à l’échelle automatique
* Créer la passerelle Application Gateway
* Tester la passerelle d’application

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour cet article, vous devez exécuter Azure PowerShell localement. Vous devez avoir installé la version du module Az 1.0.0 ou version ultérieure. Exécutez `Import-Module Az`, puis`Get-Module Az` pour trouver la version. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Après avoir vérifié la version PowerShell, exécutez `Login-AzAccount` pour créer une connexion avec Azure.

## <a name="sign-in-to-azure"></a>Connexion à Azure

```azurepowershell
Connect-AzAccount
Select-AzSubscription -Subscription "<sub name>"
```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources
Créer un groupe de ressources dans l’un des emplacements disponibles.

```azurepowershell
$location = "East US 2"
$rg = "<rg name>"

#Create a new Resource Group
New-AzResourceGroup -Name $rg -Location $location
```

## <a name="create-a-virtual-network"></a>Créez un réseau virtuel

Créez un réseau virtuel composé d'un sous-réseau dédié pour une passerelle d'application avec mise à l'échelle automatique. Actuellement, une seule passerelle d’application de mise à l’échelle automatique peut être déployée dans chaque sous-réseau dédié.

```azurepowershell
#Create VNet with two subnets
$sub1 = New-AzVirtualNetworkSubnetConfig -Name "AppGwSubnet" -AddressPrefix "10.0.0.0/24"
$sub2 = New-AzVirtualNetworkSubnetConfig -Name "BackendSubnet" -AddressPrefix "10.0.1.0/24"
$vnet = New-AzvirtualNetwork -Name "AutoscaleVNet" -ResourceGroupName $rg `
       -Location $location -AddressPrefix "10.0.0.0/16" -Subnet $sub1, $sub2
```

## <a name="create-a-reserved-public-ip"></a>Créer une adresse IP publique réservée

Définissez la méthode d'allocation d'adresses PublicIPAddress sur **Statique**. Une adresse IP virtuelle de passerelle d’application de mise à l’échelle automatique peut être uniquement statique. Les adresses IP dynamiques ne sont pas prises en charge. Seule la référence SKU PublicIpAddress standard est prise en charge.

```azurepowershell
#Create static public IP
$pip = New-AzPublicIpAddress -ResourceGroupName $rg -name "AppGwVIP" `
       -location $location -AllocationMethod Static -Sku Standard
```

## <a name="retrieve-details"></a>Récupérer les détails

Récupérez les détails du groupe de ressources, du sous-réseau et de l'adresse IP dans un objet local afin de créer des informations de configuration d'adresse IP pour la passerelle d'application.

```azurepowershell
$resourceGroup = Get-AzResourceGroup -Name $rg
$publicip = Get-AzPublicIpAddress -ResourceGroupName $rg -name "AppGwVIP"
$vnet = Get-AzvirtualNetwork -Name "AutoscaleVNet" -ResourceGroupName $rg
$gwSubnet = Get-AzVirtualNetworkSubnetConfig -Name "AppGwSubnet" -VirtualNetwork $vnet
```

## <a name="configure-the-infrastructure"></a>Configurer l’infrastructure

Configurez l’adresse IP, l’adresse IP frontend, le pool principal, les paramètres HTTP, le certificat, le port et l’écouteur dans un format identique à celui de la passerelle d’application standard existante. La nouvelle référence SKU suit le même modèle d’objet que la référence SKU Standard.

```azurepowershell
$ipconfig = New-AzApplicationGatewayIPConfiguration -Name "IPConfig" -Subnet $gwSubnet
$fip = New-AzApplicationGatewayFrontendIPConfig -Name "FrontendIPCOnfig" -PublicIPAddress $publicip
$pool = New-AzApplicationGatewayBackendAddressPool -Name "Pool1" `
       -BackendIPAddresses testbackend1.westus.cloudapp.azure.com, testbackend2.westus.cloudapp.azure.com
$fp01 = New-AzApplicationGatewayFrontendPort -Name "HTTPPort" -Port 80

$listener01 = New-AzApplicationGatewayHttpListener -Name "HTTPListener" `
             -Protocol Http -FrontendIPConfiguration $fip -FrontendPort $fp01

$setting = New-AzApplicationGatewayBackendHttpSettings -Name "BackendHttpSetting1" `
          -Port 80 -Protocol Http -CookieBasedAffinity Disabled
```

## <a name="specify-your-http-header-rewrite-rule-configuration"></a>Spécifier la configuration de règle de réécriture de votre en-tête HTTP

Configurez les nouveaux objets requis pour réécrire les en-têtes HTTP :

- **RequestHeaderConfiguration**  : cet objet est utilisé pour spécifier les champs d’en-tête de requête que vous souhaitez réécrire et la nouvelle valeur de réécriture des en-têtes d’origine.
- **ResponseHeaderConfiguration**  : cet objet est utilisé pour spécifier les champs d’en-tête de réponse que vous souhaitez réécrire et la nouvelle valeur de réécriture des en-têtes d’origine.
- **ActionSet**  : cet objet contient les configurations des en-têtes de requête et de réponse spécifiés ci-dessus. 
- **RewriteRule**  : cet objet contient tous les *actionSets* spécifiés ci-dessus. 
- **RewriteRuleSet**  : cet objet contient toutes les *rewriteRules* et devra être attaché à une règle d’acheminement des requêtes, qu’elles soient basiques ou basées sur un chemin d’accès.

   ```azurepowershell
   $requestHeaderConfiguration = New-AzApplicationGatewayRewriteRuleHeaderConfiguration -HeaderName "X-isThroughProxy" -HeaderValue "True"
   $responseHeaderConfiguration = New-AzApplicationGatewayRewriteRuleHeaderConfiguration -HeaderName "Strict-Transport-Security" -HeaderValue "max-age=31536000"
   $actionSet = New-AzApplicationGatewayRewriteRuleActionSet -RequestHeaderConfiguration $requestHeaderConfiguration -ResponseHeaderConfiguration $responseHeaderConfiguration    
   $rewriteRule = New-AzApplicationGatewayRewriteRule -Name rewriteRule1 -ActionSet $actionSet    
   $rewriteRuleSet = New-AzApplicationGatewayRewriteRuleSet -Name rewriteRuleSet1 -RewriteRule $rewriteRule
   ```

## <a name="specify-the-routing-rule"></a>Spécifier la règle d’acheminement

Créer une règle d’acheminement de requête. Une fois créée, cette configuration de réécriture est attachée à l’écouteur source via la règle d’acheminement. Lorsque vous utilisez une règle d’acheminement de base, la configuration de réécriture de l’en-tête est associée à un écouteur de la source et est une réécriture de l’en-tête global. Lorsqu’une règle d’acheminement basée sur le chemin d’accès est utilisée, la configuration de réécriture de l’en-tête est définie sur le mappage de chemin d’accès d’URL. Elle s’applique donc exclusivement à la zone de chemin d’accès spécifique d’un site. L’exemple ci-dessous illustre la création d’une règle d’acheminement de base et l’attachement de l’ensemble des règles de réécriture.

```azurepowershell
$rule01 = New-AzApplicationGatewayRequestRoutingRule -Name "Rule1" -RuleType basic `
         -BackendHttpSettings $setting -HttpListener $listener01 -BackendAddressPool $pool -RewriteRuleSet $rewriteRuleSet
```

## <a name="specify-autoscale"></a>Spécifier la mise à l’échelle automatique

Vous pouvez maintenant spécifier la configuration de mise à l'échelle automatique pour la passerelle d'application. Deux types de configuration de mise à l’échelle automatique sont pris en charge :

* **Mode de capacité fixe**. Dans ce mode, la passerelle d’application n’effectue pas de mise à l’échelle automatique et fonctionne à une capacité d’unité d’échelle fixe.

   ```azurepowershell
   $sku = New-AzApplicationGatewaySku -Name Standard_v2 -Tier Standard_v2 -Capacity 2
   ```

* **Mode de mise à l’échelle automatique**. Dans ce mode, la passerelle d’application se met automatiquement à l’échelle en fonction du modèle de trafic d’application.

   ```azurepowershell
   $autoscaleConfig = New-AzApplicationGatewayAutoscaleConfiguration -MinCapacity 2
   $sku = New-AzApplicationGatewaySku -Name Standard_v2 -Tier Standard_v2
   ```

## <a name="create-the-application-gateway"></a>Créer la passerelle Application Gateway

Créez la passerelle d'application et incluez les zones de redondance et la configuration de mise à l'échelle automatique.

```azurepowershell
$appgw = New-AzApplicationGateway -Name "AutoscalingAppGw" -Zone 1,2,3 -ResourceGroupName $rg -Location $location -BackendAddressPools $pool -BackendHttpSettingsCollection $setting -GatewayIpConfigurations $ipconfig -FrontendIpConfigurations $fip -FrontendPorts $fp01 -HttpListeners $listener01 -RequestRoutingRules $rule01 -Sku $sku -AutoscaleConfiguration $autoscaleConfig -RewriteRuleSet $rewriteRuleSet
```

## <a name="test-the-application-gateway"></a>Tester la passerelle d’application

Utilisez Get-AzPublicIPAddress pour obtenir l’adresse IP publique de la passerelle d’application. Copiez l’adresse IP publique ou le nom DNS, puis collez cette donnée dans la barre d’adresses de votre navigateur.

```azurepowershell
Get-AzPublicIPAddress -ResourceGroupName $rg -Name AppGwVIP
```



## <a name="clean-up-resources"></a>Nettoyer les ressources

Commencez par explorer les ressources créées avec la passerelle d'application. Puis, lorsque vous n'en aurez plus besoin, vous pourrez utiliser la commande `Remove-AzResourceGroup` pour supprimer le groupe de ressources, la passerelle d'application et toutes les ressources associées.

`Remove-AzResourceGroup -Name $rg`

## <a name="next-steps"></a>Étapes suivantes

- [Créer une passerelle d’application avec des règles d’acheminement par chemin d’URL](./tutorial-url-route-powershell.md)
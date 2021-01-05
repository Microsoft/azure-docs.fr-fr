---
title: 'Tutoriel : Améliorer l’accès aux applications web – Azure Application Gateway'
description: Dans ce tutoriel, apprenez à créer une passerelle d'application redondante dans une zone, avec une mise à l'échelle automatique et avec une adresse IP réservée à l’aide d'Azure PowerShell.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: tutorial
ms.date: 11/13/2019
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: 5731b65892877e5c363220d84a0bddeb5f958cee
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93396870"
---
# <a name="tutorial-create-an-application-gateway-that-improves-web-application-access"></a>Tutoriel : Créer une passerelle d'application qui améliore l'accès aux applications web

Si vous êtes un administrateur informatique soucieux d'améliorer l'accès aux applications web, vous pouvez optimiser votre passerelle d'application pour qu'elle s'adapte à la demande des clients et s'étende sur plusieurs zones de disponibilité. Ce tutoriel vous aide à configurer les fonctionnalités Azure Application Gateway qui gèrent la mise à l'échelle automatique, la redondance de zone et les adresses IP virtuelles réservées (adresses IP statiques). Vous allez utiliser des cmdlets Azure PowerShell et le modèle de déploiement Azure Resource Manager pour résoudre le problème.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un certificat auto-signé
> * Créer un réseau virtuel avec mise à l'échelle automatique
> * Créer une adresse IP publique réservée
> * Configurer l'infrastructure de votre passerelle d'application
> * Spécifier la mise à l’échelle automatique
> * Créer la passerelle Application Gateway
> * Tester la passerelle d’application

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Pour ce didacticiel, vous devez exécuter Azure PowerShell localement. Le module Azure PowerShell 1.0.0 ou version ultérieure doit être installé. Exécutez `Get-Module -ListAvailable Az` pour trouver la version. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Après avoir vérifié la version PowerShell, exécutez `Connect-AzAccount` pour créer une connexion avec Azure.

## <a name="sign-in-to-azure"></a>Connexion à Azure

```azurepowershell
Connect-AzAccount
Select-AzSubscription -Subscription "<sub name>"
```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources
Créer un groupe de ressources dans l’un des emplacements disponibles.

```azurepowershell
$location = "East US 2"
$rg = "AppGW-rg"

#Create a new Resource Group
New-AzResourceGroup -Name $rg -Location $location
```

## <a name="create-a-self-signed-certificate"></a>Créer un certificat auto-signé

Dans un environnement de production, vous devez importer un certificat valide signé par un fournisseur approuvé. Pour ce didacticiel, vous créez un certificat auto-signé à l’aide de [New-SelfSignedCertificate](/powershell/module/pkiclient/new-selfsignedcertificate). Vous pouvez utiliser [Export-PfxCertificate](/powershell/module/pkiclient/export-pfxcertificate) avec l’empreinte numérique qui a été retournée pour exporter un fichier pfx du certificat.

```powershell
New-SelfSignedCertificate `
  -certstorelocation cert:\localmachine\my `
  -dnsname www.contoso.com
```

Un résultat comme l’exemple suivant devrait s’afficher :

```
PSParentPath: Microsoft.PowerShell.Security\Certificate::LocalMachine\my

Thumbprint                                Subject
----------                                -------
E1E81C23B3AD33F9B4D1717B20AB65DBB91AC630  CN=www.contoso.com
```

Utilisez l’empreinte numérique pour créer le fichier pfx :

```powershell
$pwd = ConvertTo-SecureString -String "Azure123456!" -Force -AsPlainText

Export-PfxCertificate `
  -cert cert:\localMachine\my\E1E81C23B3AD33F9B4D1717B20AB65DBB91AC630 `
  -FilePath c:\appgwcert.pfx `
  -Password $pwd
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

Configurez l'adresse IP, l'adresse IP frontend, le pool principal, les paramètres HTTP, le certificat, le port, l'écouteur et la règle dans un format identique à celui de la passerelle d'application standard existante. La nouvelle référence SKU suit le même modèle d’objet que la référence SKU Standard.

```azurepowershell
$ipconfig = New-AzApplicationGatewayIPConfiguration -Name "IPConfig" -Subnet $gwSubnet
$fip = New-AzApplicationGatewayFrontendIPConfig -Name "FrontendIPCOnfig" -PublicIPAddress $publicip
$pool = New-AzApplicationGatewayBackendAddressPool -Name "Pool1" `
       -BackendIPAddresses testbackend1.westus.cloudapp.azure.com, testbackend2.westus.cloudapp.azure.com
$fp01 = New-AzApplicationGatewayFrontendPort -Name "SSLPort" -Port 443
$fp02 = New-AzApplicationGatewayFrontendPort -Name "HTTPPort" -Port 80

$securepfxpwd = ConvertTo-SecureString -String "Azure123456!" -AsPlainText -Force
$sslCert01 = New-AzApplicationGatewaySslCertificate -Name "SSLCert" -Password $securepfxpwd `
            -CertificateFile "c:\appgwcert.pfx"
$listener01 = New-AzApplicationGatewayHttpListener -Name "SSLListener" `
             -Protocol Https -FrontendIPConfiguration $fip -FrontendPort $fp01 -SslCertificate $sslCert01
$listener02 = New-AzApplicationGatewayHttpListener -Name "HTTPListener" `
             -Protocol Http -FrontendIPConfiguration $fip -FrontendPort $fp02

$setting = New-AzApplicationGatewayBackendHttpSettings -Name "BackendHttpSetting1" `
          -Port 80 -Protocol Http -CookieBasedAffinity Disabled
$rule01 = New-AzApplicationGatewayRequestRoutingRule -Name "Rule1" -RuleType basic `
         -BackendHttpSettings $setting -HttpListener $listener01 -BackendAddressPool $pool
$rule02 = New-AzApplicationGatewayRequestRoutingRule -Name "Rule2" -RuleType basic `
         -BackendHttpSettings $setting -HttpListener $listener02 -BackendAddressPool $pool
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
$appgw = New-AzApplicationGateway -Name "AutoscalingAppGw" -Zone 1,2,3 `
  -ResourceGroupName $rg -Location $location -BackendAddressPools $pool `
  -BackendHttpSettingsCollection $setting -GatewayIpConfigurations $ipconfig `
  -FrontendIpConfigurations $fip -FrontendPorts $fp01, $fp02 `
  -HttpListeners $listener01, $listener02 -RequestRoutingRules $rule01, $rule02 `
  -Sku $sku -sslCertificates $sslCert01 -AutoscaleConfiguration $autoscaleConfig
```

## <a name="test-the-application-gateway"></a>Tester la passerelle d’application

Utilisez Get-AzPublicIPAddress pour obtenir l’adresse IP publique de la passerelle d’application. Copiez l’adresse IP publique ou le nom DNS, puis collez cette donnée dans la barre d’adresses de votre navigateur.

`Get-AzPublicIPAddress -ResourceGroupName $rg -Name AppGwVIP`

## <a name="clean-up-resources"></a>Nettoyer les ressources

Commencez par explorer les ressources créées avec la passerelle d'application. Puis, lorsque vous n'en aurez plus besoin, vous pourrez utiliser la commande `Remove-AzResourceGroup` pour supprimer le groupe de ressources, la passerelle d'application et toutes les ressources associées.

`Remove-AzResourceGroup -Name $rg`

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer une passerelle d’application avec des règles d’acheminement par chemin d’URL](./tutorial-url-route-powershell.md)
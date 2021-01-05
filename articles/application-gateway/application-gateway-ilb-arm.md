---
title: Utiliser un équilibreur de charge interne – Azure Application Gateway
description: Cette page fournit des instructions pour la création, la configuration, le démarrage et la suppression d’une passerelle Application Gateway Azure avec un équilibrage de charge interne (ILB) pour Azure Resource Manager
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/13/2019
ms.author: victorh
ms.openlocfilehash: 3d663dc4e2bd860ec9494785ecbf6dbf10a4c5b5
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397754"
---
# <a name="create-an-application-gateway-with-an-internal-load-balancer-ilb"></a>Créer une passerelle d’application avec un équilibreur de charge interne (ILB)

Vous pouvez configurer une passerelle Azure Application Gateway avec une adresse IP virtuelle côté Internet ou avec un point de terminaison interne non exposé à Internet, également appelé point de terminaison d’équilibrage de charge interne (ILB). La configuration de la passerelle avec un équilibrage de charge interne est utile pour les applications métier internes non exposées à Internet. Elle est également utile pour les services et niveaux au sein d’une application multiniveau qui se trouve dans une limite de sécurité non exposée à Internet, mais qui requiert tout de même une distribution de charge par tourniquet, une adhérence de session ou une terminaison TLS (Transport Layer Security), anciennement SSL (Secure Sockets Layer).

Cet article vous guidera au cours des étapes de configuration d’une passerelle Application Gateway avec un équilibrage de charge interne.

## <a name="before-you-begin"></a>Avant de commencer

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

1. Installez la dernière version du module Azure PowerShell en suivant les [instructions d’installation](/powershell/azure/install-az-ps).
2. Vous créez un réseau virtuel et un sous-réseau pour la passerelle Application Gateway. Assurez-vous qu’aucun ordinateur virtuel ou déploiement cloud n’utilise le sous-réseau. La passerelle Application Gateway doit être seule sur un sous-réseau virtuel.
3. Les serveurs que vous configurez pour utiliser la passerelle Application Gateway doivent exister ou vous devez créer leurs points de terminaison sur le réseau virtuel ou avec une adresse IP/VIP publique affectée.

## <a name="what-is-required-to-create-an-application-gateway"></a>Quels sont les éléments nécessaires pour créer une passerelle Application Gateway ?

* **Pool de serveurs back-end :** Liste des adresses IP des serveurs principaux. Les adresses IP répertoriées doivent appartenir au réseau virtuel, mais à un sous-réseau différent de la plateforme d’application ou elles doivent correspondre à une adresse IP/VIP publique.
* **Paramètres de pool de serveurs back-end :** Chaque pool dispose de paramètres tels que le port, le protocole et l’affinité en fonction des cookies. Ces paramètres sont liés à un pool et sont appliqués à tous les serveurs du pool.
* **Port front-end :** Il s’agit du port public ouvert sur la passerelle d’application. Le trafic atteint ce port, puis il est redirigé vers l’un des serveurs principaux.
* **Écouteur :** L’écouteur a un port frontal, un protocole (Http ou Https, avec respect de la casse) et le nom du certificat SSL (en cas de configuration du déchargement SSL).
* **Règle :** la règle lie l’écouteur et le pool de serveurs back-end, et définit vers quel pool de serveurs back-end le trafic doit être dirigé quand il atteint un écouteur spécifique. Actuellement, seule la règle *de base* est prise en charge. La règle de *base* est la distribution de charge par tourniquet.

## <a name="create-an-application-gateway"></a>Créer une passerelle Application Gateway

La différence entre l’utilisation d’Azure Classic et celle d’Azure Resource Manager réside dans l’ordre de création de la passerelle Application Gateway et des éléments à configurer.
Avec Resource Manager, tous les éléments constitutifs d’une passerelle Application Gateway sont configurés individuellement, puis regroupés pour créer la ressource Application Gateway.

Procédure de création d’une passerelle Application Gateway :

1. Créer un groupe de ressources pour Resource Manager
2. Créer un réseau virtuel et un sous-réseau pour la passerelle Application Gateway
3. Créer un objet de configuration de passerelle Application Gateway
4. Créer une ressource de passerelle d’application

## <a name="create-a-resource-group-for-resource-manager"></a>Créer un groupe de ressources pour Resource Manager

Veillez à passer en mode PowerShell pour utiliser les applets de commande d’Azure Resource Manager. Pour plus d’informations, voir [Utilisation de Windows PowerShell avec Azure Resource Manager](../azure-resource-manager/management/manage-resources-powershell.md).

### <a name="step-1"></a>Étape 1

```powershell
Connect-AzAccount
```

### <a name="step-2"></a>Étape 2

Vérifiez les abonnements associés au compte.

```powershell
Get-AzSubscription
```

Vous êtes invité à vous authentifier à l’aide de vos informations d’identification.

### <a name="step-3"></a>Étape 3 :

Parmi vos abonnements Azure, choisissez celui que vous souhaitez utiliser.

```powershell
Select-AzSubscription -Subscriptionid "GUID of subscription"
```

### <a name="step-4"></a>Étape 4

Créez un groupe de ressources (ignorez cette étape si vous utilisez un groupe de ressources existant)

```powershell
New-AzResourceGroup -Name appgw-rg -location "West US"
```

Azure Resource Manager requiert que tous les groupes de ressources spécifient un emplacement. Ce dernier est utilisé comme emplacement par défaut des ressources de ce groupe. Assurez-vous que toutes les commandes pour la création d’une passerelle Application Gateway utilisent le même groupe de ressources.

Dans l’exemple précédent, nous avons créé un groupe de ressources appelé « appgw-rg », ainsi que l’emplacement « USA Ouest ».

## <a name="create-a-virtual-network-and-a-subnet-for-the-application-gateway"></a>Créer un réseau virtuel et un sous-réseau pour la passerelle Application Gateway

L’exemple ci-après indique comment créer un réseau virtuel à l’aide de Resource Manager :

### <a name="step-1"></a>Étape 1

```powershell
$subnetconfig = New-AzVirtualNetworkSubnetConfig -Name subnet01 -AddressPrefix 10.0.0.0/24
```

Attribue la plage d’adresses 10.0.0.0/24 à une variable subnet à utiliser pour créer un réseau virtuel.

### <a name="step-2"></a>Étape 2

```powershell
$vnet = New-AzVirtualNetwork -Name appgwvnet -ResourceGroupName appgw-rg -Location "West US" -AddressPrefix 10.0.0.0/16 -Subnet $subnetconfig
```

Crée un réseau virtuel nommé « appgwvnet » dans le groupe de ressources « appw-rg » pour la région « USA Ouest » à l’aide du préfixe 10.0.0.0/16 avec le sous-réseau 10.0.0.0/24.

### <a name="step-3"></a>Étape 3 :

```powershell
$subnet = $vnet.subnets[0]
```

Assigne l’objet de sous-réseau à la variable $subnet pour les étapes suivantes.

## <a name="create-an-application-gateway-configuration-object"></a>Créer un objet de configuration de passerelle Application Gateway

### <a name="step-1"></a>Étape 1

```powershell
$gipconfig = New-AzApplicationGatewayIPConfiguration -Name gatewayIP01 -Subnet $subnet
```

Crée une configuration IP de passerelle Application Gateway nommée « gatewayIP01 ». Lorsque la passerelle Application Gateway démarre, elle sélectionne une adresse IP à partir du sous-réseau configuré et achemine le trafic réseau vers les adresses IP du pool IP principal. Gardez à l’esprit que chaque instance utilise une adresse IP unique.

### <a name="step-2"></a>Étape 2

```powershell
$pool = New-AzApplicationGatewayBackendAddressPool -Name pool01 -BackendIPAddresses 10.1.1.8,10.1.1.9,10.1.1.10
```

Configure le pool d’adresses IP principal nommé « pool01 » avec les adresses IP « 10.1.1.8, 10.1.1.9, 10.1.1.10 ». Il s’agit des adresses IP qui recevront le trafic réseau provenant du point de terminaison IP frontal. Remplacez les adresses IP précédentes pour ajouter vos propres points de terminaison d’adresse IP d’application.

### <a name="step-3"></a>Étape 3 :

```powershell
$poolSetting = New-AzApplicationGatewayBackendHttpSettings -Name poolsetting01 -Port 80 -Protocol Http -CookieBasedAffinity Disabled
```

Cette étape permet de configurer les paramètres de passerelle Application Gateway « poolsetting01 » pour le trafic réseau à charge équilibrée dans le pool principal.

### <a name="step-4"></a>Étape 4

```powershell
$fp = New-AzApplicationGatewayFrontendPort -Name frontendport01  -Port 80
```

Cette étape permet de configurer le port d’adresses IP frontal nommé « frontendport01 » pour l’équilibrage de charge interne.

### <a name="step-5"></a>Étape 5

```powershell
$fipconfig = New-AzApplicationGatewayFrontendIPConfig -Name fipconfig01 -Subnet $subnet
```

Cette étape permet de créer la configuration IP frontale nommée « fipconfig01 » et lui associe une adresse IP privée à partir du sous-réseau du réseau virtuel actuel.

### <a name="step-6"></a>Étape 6

```powershell
$listener = New-AzApplicationGatewayHttpListener -Name listener01  -Protocol Http -FrontendIPConfiguration $fipconfig -FrontendPort $fp
```

Cette étape permet de créer l’écouteur nommé « listener01 » et associe le port frontal à la configuration IP frontale.

### <a name="step-7"></a>Étape 7

```powershell
$rule = New-AzApplicationGatewayRequestRoutingRule -Name rule01 -RuleType Basic -BackendHttpSettings $poolSetting -HttpListener $listener -BackendAddressPool $pool
```

Cette étape permet de créer la règle d’acheminement d’équilibrage de charge nommée « rule01 » qui configure le comportement d’équilibrage de charge.

### <a name="step-8"></a>Étape 8

```powershell
$sku = New-AzApplicationGatewaySku -Name Standard_Small -Tier Standard -Capacity 2
```

Cette étape permet de configurer la taille d’instance de la passerelle Application Gateway.

> [!NOTE]
> La valeur par défaut pour la capacité est 2. Pour le nom de la référence (SKU), vous pouvez choisir entre Standard_Small, Standard_Medium et Standard_Large.

## <a name="create-an-application-gateway-by-using-new-azureapplicationgateway"></a>Créer une passerelle Application Gateway avec New-AzureApplicationGateway

Créez une passerelle Application Gateway avec tous les éléments de configuration de la procédure précédente. Dans notre exemple, la passerelle Application Gateway est appelée « appgwtest ».

```powershell
$appgw = New-AzApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg -Location "West US" -BackendAddressPools $pool -BackendHttpSettingsCollection $poolSetting -FrontendIpConfigurations $fipconfig  -GatewayIpConfigurations $gipconfig -FrontendPorts $fp -HttpListeners $listener -RequestRoutingRules $rule -Sku $sku
```

Cette étape permet de créer une passerelle Application Gateway avec tous les éléments de configuration de la procédure précédente. Dans notre exemple, la passerelle Application Gateway est appelée « appgwtest ».

## <a name="delete-an-application-gateway"></a>Supprimer une passerelle Application Gateway

Pour supprimer une passerelle Application Gateway, vous devez effectuer les opérations suivantes dans l’ordre :

1. Utilisez l’applet de commande `Stop-AzApplicationGateway` pour arrêter la passerelle.
2. Utilisez l’applet de commande `Remove-AzApplicationGateway` pour supprimer la passerelle.
3. Vérifiez que la passerelle a été supprimée à l’aide de l’applet de commande `Get-AzureApplicationGateway`.

### <a name="step-1"></a>Étape 1

Obtenez l’objet de passerelle Application Gateway et associez-le à une variable « $getgw ».

```powershell
$getgw =  Get-AzApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg
```

### <a name="step-2"></a>Étape 2

Utilisez `Stop-AzApplicationGateway` pour arrêter la passerelle Application Gateway. Cet exemple montre l'applet de commande `Stop-AzApplicationGateway` sur la première ligne, suivie de la sortie.

```powershell
Stop-AzApplicationGateway -ApplicationGateway $getgw  
```

```
VERBOSE: 9:49:34 PM - Begin Operation: Stop-AzureApplicationGateway
VERBOSE: 10:10:06 PM - Completed Operation: Stop-AzureApplicationGateway
Name       HTTP Status Code     Operation ID                             Error
----       ----------------     ------------                             ----
Successful OK                   ce6c6c95-77b4-2118-9d65-e29defadffb8
```

Une fois la passerelle Application Gateway dans un état arrêté, utilisez l’applet de commande `Remove-AzApplicationGateway` pour supprimer le service.

```powershell
Remove-AzApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg -Force
```

```
VERBOSE: 10:49:34 PM - Begin Operation: Remove-AzureApplicationGateway
VERBOSE: 10:50:36 PM - Completed Operation: Remove-AzureApplicationGateway
Name       HTTP Status Code     Operation ID                             Error
----       ----------------     ------------                             ----
Successful OK                   055f3a96-8681-2094-a304-8d9a11ad8301
```

> [!NOTE]
> Il est possible d’utiliser le commutateur **-force** pour supprimer le message de confirmation de suppression.

Pour vérifier que le service a été supprimé, vous pouvez utiliser l’applet de commande `Get-AzApplicationGateway`. Cette étape n'est pas requise.

```powershell
Get-AzApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg
```

```
VERBOSE: 10:52:46 PM - Begin Operation: Get-AzureApplicationGateway

Get-AzureApplicationGateway : ResourceNotFound: The gateway does not exist.
```

## <a name="next-steps"></a>Étapes suivantes

Si vous souhaitez configurer le déchargement SSL, consultez [Configuration d’une passerelle Application Gateway pour le déchargement SSL](./tutorial-ssl-powershell.md).

Si vous souhaitez plus d'informations sur les options d'équilibrage de charge en général, consultez :

* [Équilibrage de charge Azure](https://azure.microsoft.com/documentation/services/load-balancer/)
* [Azure Traffic Manager](https://azure.microsoft.com/documentation/services/traffic-manager/)
---
title: Utilisation du service Gestion des API dans un réseau virtuel avec Application Gateway
titleSuffix: Azure API Management
description: Découvrez comment installer et configurer le service Gestion des API Azure dans un réseau virtuel interne avec Application Gateway (WAF) en tant que FrontEnd
services: api-management
documentationcenter: ''
author: solankisamir
manager: kjoshi
editor: vlvinogr
ms.assetid: a8c982b2-bca5-4312-9367-4a0bbc1082b1
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/04/2019
ms.author: sasolank
ms.openlocfilehash: 3db1c8bfc3a11151342589af0873d88e3d90c6a1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91825627"
---
# <a name="integrate-api-management-in-an-internal-vnet-with-application-gateway"></a>Intégrer le service Gestion des API dans un réseau virtuel interne avec Application Gateway

## <a name="overview"></a><a name="overview"> </a> Vue d’ensemble

Le service Gestion des API peut être configuré dans un réseau virtuel en mode interne, ce qui le rend uniquement accessible à partir du réseau virtuel. Azure Application Gateway est un service PAAS qui propose un équilibreur de charge de couche 7. Il agit comme un service proxy inverse et fournit dans son offre un pare-feu d’applications web (WAF).

Combiner la gestion des API configurée dans un réseau virtuel interne avec le frontal Application Gateway permet les scénarios suivants :

* Utilisez la même ressource de gestion des API pour la consommation à la fois par les consommateurs internes et externes.
* Utilisez une seule ressource de gestion des API et mettez à disposition un sous-ensemble d’API défini dans la gestion des API pour les consommateurs externes.
* Fournissez un moyen clé en main d’activer et désactiver l’accès à la gestion des API à partir de l’Internet public.

[!INCLUDE [premium-dev.md](../../includes/api-management-availability-premium-dev.md)]

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Pour suivre les étapes décrites dans cet article, vous devez disposer des éléments suivants :

* Un abonnement Azure actif.

    [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

* Des certificats : pfx et cer pour le nom d’hôte de l’API, et pfx pour le nom d’hôte du portail des développeurs.

## <a name="scenario"></a><a name="scenario"> </a> Scénario

Dans cet article, nous allons étudier comment utiliser un seul et même service Gestion des API pour les consommateurs internes et externes, et l’utiliser comme serveur frontal sur les API locales et cloud. Vous allez également voir comment exposer uniquement un sous-ensemble de vos API (dans cet exemple, elles sont mises en surbrillance en vert) pour une consommation externe, à l’aide de la fonctionnalité disponible dans Application Gateway.

Dans le premier exemple de configuration, toutes vos API sont gérées uniquement à partir de votre réseau virtuel. Les consommateurs internes (mis en surbrillance en orange) peuvent accéder à toutes vos API internes et externes. Le trafic ne sort jamais vers Internet. Une connectivité à haute performance est fournie via des circuits Express Route.

![itinéraire d’URL](./media/api-management-howto-integrate-internal-vnet-appgateway/api-management-howto-integrate-internal-vnet-appgateway.png)

## <a name="before-you-begin"></a><a name="before-you-begin"> </a> Avant de commencer

* Assurez-vous que vous disposez de la version la plus récente d’Azure PowerShell. Consultez les instructions d’installation sur [Installer Azure PowerShell](/powershell/azure/install-az-ps). 

## <a name="what-is-required-to-create-an-integration-between-api-management-and-application-gateway"></a>Qu’est-ce qui est nécessaire pour créer une intégration entre le service Gestion des API et Application Gateway ?

* **Pool de serveurs back-end :** Adresse IP virtuelle interne du service Gestion des API.
* **Paramètres de pool de serveurs back-end :** Chaque pool dispose de paramètres tels que le port, le protocole et l’affinité en fonction des cookies. Ces paramètres sont appliqués à tous les serveurs du pool.
* **Port front-end :** Il s’agit du port public ouvert sur la passerelle d’application. Le trafic l’atteignant est redirigé vers l’un des serveurs principaux.
* **Écouteur :** L’écouteur dispose d’un port front-end, d’un protocole (HTTP ou HTTPS ; valeurs sensibles à la casse) et du nom du certificat TLS/SSL (en cas de configuration du déchargement TLS).
* **Règle :** La règle relie un écouteur à un pool de serveurs principaux.
* **Sonde d’intégrité personnalisée :** Application Gateway, par défaut, utilise des sondes basées sur des adresses IP pour déterminer les serveurs actifs dans le BackendAddressPool. Le service Gestion des API répond uniquement aux requêtes avec l’en-tête d’hôte est correct. C’est pourquoi les sondes par défaut échouent. Une sonde d’intégrité personnalisée doit être définie pour aider Application Gateway à déterminer que le service est actif et qu’il doit transférer les demandes.
* **Certificat de domaine personnalisé :** Pour accéder au service Gestion des API à partir d’Internet, vous devez créer un mappage CNAME de son nom d’hôte au nom DNS frontal d’Application Gateway. Cela garantit que l’en-tête de nom d’hôte et le certificat envoyé à Application Gateway qui est transféré au service Gestion des API peuvent être reconnus comme valides par l’APIM. Dans cet exemple, nous allons utiliser deux certificats : pour le serveur backend et pour le portail des développeurs.  

## <a name="steps-required-for-integrating-api-management-and-application-gateway"></a><a name="overview-steps"> </a> Étapes requises pour l’intégration du service Gestion des API et d’Application Gateway

1. Créer un groupe de ressources pour Resource Manager
2. Créer un réseau virtuel, un sous-réseau et une adresse IP publique pour la passerelle Application Gateway Créer un autre sous-réseau pour le service Gestion des API
3. Créer un service Gestion des API dans le sous-réseau de réseau virtuel créé ci-dessus et veiller à l’utiliser en mode interne
4. Configurer un nom de domaine personnalisé dans le service Gestion des API
5. Créer un objet de configuration de passerelle Application Gateway
6. Créer une ressource Application Gateway
7. Créer un CNAME à partir du nom DNS public de la passerelle Application Gateway pour le nom d’hôte proxy du service Gestion des API

## <a name="exposing-the-developer-portal-externally-through-application-gateway"></a>Exposition du portail des développeurs en externe à l’aide d’Application Gateway

Dans ce guide, nous allons également exposer le **portail des développeurs** à un public extérieur à l’aide d’Application Gateway. Cela nécessite des étapes supplémentaires pour créer un écouteur, une sonde, des paramètres et des règles pour le portail des développeurs. Tous les détails sont fournis dans les étapes respectives.

> [!WARNING]
> Si vous utilisez Azure AD ou une authentification tierce partie, veuillez activer la fonctionnalité [Affinité de session basée sur les cookies](../application-gateway/features.md#session-affinity) dans Application Gateway.

> [!WARNING]
> Pour empêcher le pare-feu d’applications web Application Gateway de rompre le téléchargement de la spécification OpenAPI dans le portail des développeurs, vous devez désactiver la règle de pare-feu `942200 - "Detects MySQL comment-/space-obfuscated injections and backtick termination"`.
> 
> Les règles WAF d’Application Gateway, qui peuvent nuire aux fonctionnalités du portail, notamment :
> 
> - `920300`, `920330`, `931130`, `942100`, `942110`, `942180`, `942200`, `942260`, `942340`, `942370` pour le mode administratif
> - `942200`, `942260`, `942370`, `942430`, `942440` pour le portail publié

## <a name="create-a-resource-group-for-resource-manager"></a>Créer un groupe de ressources pour Resource Manager

### <a name="step-1"></a>Étape 1

Connexion à Azure

```powershell
Connect-AzAccount
```

Authentifiez-vous à l’aide de vos informations d’identification.

### <a name="step-2"></a>Étape 2

Sélectionnez l’abonnement souhaité.

```powershell
$subscriptionId = "00000000-0000-0000-0000-000000000000" # GUID of your Azure subscription
Get-AzSubscription -Subscriptionid $subscriptionId | Select-AzSubscription
```

### <a name="step-3"></a>Étape 3 :

Créez un groupe de ressources (ignorez cette étape si vous utilisez un groupe de ressources existant).

```powershell
$resGroupName = "apim-appGw-RG" # resource group name
$location = "West US"           # Azure region
New-AzResourceGroup -Name $resGroupName -Location $location
```

Azure Resource Manager requiert que tous les groupes de ressources spécifient un emplacement. Ce dernier est utilisé comme emplacement par défaut des ressources de ce groupe. Assurez-vous que toutes les commandes pour la création d'une passerelle Application Gateway utiliseront le même groupe de ressources.

## <a name="create-a-virtual-network-and-a-subnet-for-the-application-gateway"></a>Créer un réseau virtuel et un sous-réseau pour la passerelle Application Gateway

L’exemple suivant montre comment créer un réseau virtuel avec Resource Manager.

### <a name="step-1"></a>Étape 1

Affectez la plage d’adresses 10.0.0.0/24 à la variable subnet à utiliser pour la passerelle Application Gateway lors de la création d’un réseau virtuel.

```powershell
$appgatewaysubnet = New-AzVirtualNetworkSubnetConfig -Name "apim01" -AddressPrefix "10.0.0.0/24"
```

### <a name="step-2"></a>Étape 2

Affectez la plage d’adresses 10.0.1.0/24 à la variable subnet à utiliser pour le service Gestion des API lors de la création d’un réseau virtuel.

```powershell
$apimsubnet = New-AzVirtualNetworkSubnetConfig -Name "apim02" -AddressPrefix "10.0.1.0/24"
```

### <a name="step-3"></a>Étape 3 :

Créez un réseau virtuel nommé **appgwvnet** dans le groupe de ressources **apim-appGw-RG** pour la région USA Ouest. Utilisez le préfixe 10.0.0.0/16 avec les sous-réseaux 10.0.0.0/24 et 10.0.1.0/24.

```powershell
$vnet = New-AzVirtualNetwork -Name "appgwvnet" -ResourceGroupName $resGroupName -Location $location -AddressPrefix "10.0.0.0/16" -Subnet $appgatewaysubnet,$apimsubnet
```

### <a name="step-4"></a>Étape 4

Attribution d’une variable de sous-réseau pour les étapes suivantes

```powershell
$appgatewaysubnetdata = $vnet.Subnets[0]
$apimsubnetdata = $vnet.Subnets[1]
```

## <a name="create-an-api-management-service-inside-a-vnet-configured-in-internal-mode"></a>Créer un service Gestion des API dans un réseau virtuel configuré en mode interne

L’exemple ci-dessous montre comment créer un service Gestion des API dans un réseau virtuel configuré pour un accès interne uniquement.

### <a name="step-1"></a>Étape 1

Créez un objet de réseau virtuel du service Gestion des API via le sous-réseau $apimsubnetdata créé ci-dessus.

```powershell
$apimVirtualNetwork = New-AzApiManagementVirtualNetwork -SubnetResourceId $apimsubnetdata.Id
```

### <a name="step-2"></a>Étape 2

Créez un service Gestion des API dans le réseau virtuel.

```powershell
$apimServiceName = "ContosoApi"       # API Management service instance name
$apimOrganization = "Contoso"         # organization name
$apimAdminEmail = "admin@contoso.com" # administrator's email address
$apimService = New-AzApiManagement -ResourceGroupName $resGroupName -Location $location -Name $apimServiceName -Organization $apimOrganization -AdminEmail $apimAdminEmail -VirtualNetwork $apimVirtualNetwork -VpnType "Internal" -Sku "Developer"
```

Après la réussite de la commande ci-dessus, consultez la [configuration DNS requise pour accéder au service Gestion des API du réseau virtuel interne](api-management-using-with-internal-vnet.md#apim-dns-configuration) pour y accéder. Cette étape peut prendre plus d’une demi-heure.

## <a name="set-up-a-custom-domain-name-in-api-management"></a>Configurer un nom de domaine personnalisé dans le service Gestion des API

> [!IMPORTANT]
> Le [nouveau portail des développeurs](api-management-howto-developer-portal.md) nécessite également l’activation de la connectivité au point de terminaison de gestion de Gestion des API en plus des étapes ci-dessous.

### <a name="step-1"></a>Étape 1

Initialisez les variables suivantes avec les détails des certificats avec clés privées pour les domaines. Dans cet exemple, nous utiliserons `api.contoso.net` et `portal.contoso.net`.  

```powershell
$gatewayHostname = "api.contoso.net"                 # API gateway host
$portalHostname = "portal.contoso.net"               # API developer portal host
$gatewayCertCerPath = "C:\Users\Contoso\gateway.cer" # full path to api.contoso.net .cer file
$gatewayCertPfxPath = "C:\Users\Contoso\gateway.pfx" # full path to api.contoso.net .pfx file
$portalCertPfxPath = "C:\Users\Contoso\portal.pfx"   # full path to portal.contoso.net .pfx file
$gatewayCertPfxPassword = "certificatePassword123"   # password for api.contoso.net pfx certificate
$portalCertPfxPassword = "certificatePassword123"    # password for portal.contoso.net pfx certificate

$certPwd = ConvertTo-SecureString -String $gatewayCertPfxPassword -AsPlainText -Force
$certPortalPwd = ConvertTo-SecureString -String $portalCertPfxPassword -AsPlainText -Force
```

### <a name="step-2"></a>Étape 2

Créez et définissez des objets de configuration de nom d’hôte pour le proxy et pour le portail.  

```powershell
$proxyHostnameConfig = New-AzApiManagementCustomHostnameConfiguration -Hostname $gatewayHostname -HostnameType Proxy -PfxPath $gatewayCertPfxPath -PfxPassword $certPwd
$portalHostnameConfig = New-AzApiManagementCustomHostnameConfiguration -Hostname $portalHostname -HostnameType DeveloperPortal -PfxPath $portalCertPfxPath -PfxPassword $certPortalPwd

$apimService.ProxyCustomHostnameConfiguration = $proxyHostnameConfig
$apimService.PortalCustomHostnameConfiguration = $portalHostnameConfig
Set-AzApiManagement -InputObject $apimService
```

> [!NOTE]
> Pour configurer la connectivité de l’ancien portail des développeurs, vous devez remplacer `-HostnameType DeveloperPortal` par `-HostnameType Portal`.

## <a name="create-a-public-ip-address-for-the-front-end-configuration"></a>Création d'une adresse IP publique pour la configuration frontale

Créez une ressource IP publique **publicIP01** dans le groupe de ressources.

```powershell
$publicip = New-AzPublicIpAddress -ResourceGroupName $resGroupName -name "publicIP01" -location $location -AllocationMethod Dynamic
```

Une adresse IP est affectée à la passerelle Application Gateway au démarrage du service.

## <a name="create-application-gateway-configuration"></a>Créer une configuration de passerelle Application Gateway

Tous les éléments de configuration doivent être installés avant de créer la passerelle Application Gateway. Les étapes suivantes permettent de créer les éléments de configuration nécessaires à une ressource Application Gateway.

### <a name="step-1"></a>Étape 1

Créez une configuration IP de passerelle Application Gateway nommée **gatewayIP01**. Lorsque la passerelle Application Gateway démarre, elle sélectionne une adresse IP à partir du sous-réseau configuré et achemine le trafic réseau vers les adresses IP du pool IP principal. Gardez à l’esprit que chaque instance utilise une adresse IP unique.

```powershell
$gipconfig = New-AzApplicationGatewayIPConfiguration -Name "gatewayIP01" -Subnet $appgatewaysubnetdata
```

### <a name="step-2"></a>Étape 2

Configurez le port IP frontal pour le point de terminaison IP public. Ce port est le port auquel les utilisateurs finaux se connectent.

```powershell
$fp01 = New-AzApplicationGatewayFrontendPort -Name "port01"  -Port 443
```

### <a name="step-3"></a>Étape 3 :

Configurez l’adresse IP frontale avec un point de terminaison IP public.

```powershell
$fipconfig01 = New-AzApplicationGatewayFrontendIPConfig -Name "frontend1" -PublicIPAddress $publicip
```

### <a name="step-4"></a>Étape 4

Configurez le certificat pour la passerelle Application Gateway, laquelle sera utilisée pour déchiffrer et rechiffrer le trafic transitant par celle-ci.

```powershell
$cert = New-AzApplicationGatewaySslCertificate -Name "cert01" -CertificateFile $gatewayCertPfxPath -Password $certPwd
$certPortal = New-AzApplicationGatewaySslCertificate -Name "cert02" -CertificateFile $portalCertPfxPath -Password $certPortalPwd
```

### <a name="step-5"></a>Étape 5

Créez les écouteurs HTTP pour la passerelle Application Gateway. Affectez-leur la configuration IP front-end, le port et les certificats TLS/SSL.

```powershell
$listener = New-AzApplicationGatewayHttpListener -Name "listener01" -Protocol "Https" -FrontendIPConfiguration $fipconfig01 -FrontendPort $fp01 -SslCertificate $cert -HostName $gatewayHostname -RequireServerNameIndication true
$portalListener = New-AzApplicationGatewayHttpListener -Name "listener02" -Protocol "Https" -FrontendIPConfiguration $fipconfig01 -FrontendPort $fp01 -SslCertificate $certPortal -HostName $portalHostname -RequireServerNameIndication true
```

### <a name="step-6"></a>Étape 6

Créez des sondes personnalisées pour le point de terminaison de domaine de proxy `ContosoApi` du service Gestion des API. Le chemin d’accès `/status-0123456789abcdef` est un point de terminaison d’intégrité par défaut hébergé sur tous les services de gestion des API. Définissez `api.contoso.net` comme nom d’hôte de sonde personnalisée pour la sécuriser à l’aide du certificat TLS/SSL.

> [!NOTE]
> Le nom d’hôte `contosoapi.azure-api.net` est le nom d’hôte du proxy par défaut configuré lorsqu’un service nommé `contosoapi` est créé dans la version publique d’Azure.
>

```powershell
$apimprobe = New-AzApplicationGatewayProbeConfig -Name "apimproxyprobe" -Protocol "Https" -HostName $gatewayHostname -Path "/status-0123456789abcdef" -Interval 30 -Timeout 120 -UnhealthyThreshold 8
$apimPortalProbe = New-AzApplicationGatewayProbeConfig -Name "apimportalprobe" -Protocol "Https" -HostName $portalHostname -Path "/internal-status-0123456789abcdef" -Interval 60 -Timeout 300 -UnhealthyThreshold 8
```

### <a name="step-7"></a>Étape 7

Chargez le certificat à utiliser sur les ressources du pool de back-ends pour lequel TLS est activé. Il s’agit du certificat que vous avez fourni à l’étape 4 ci-dessus.

```powershell
$authcert = New-AzApplicationGatewayAuthenticationCertificate -Name "whitelistcert1" -CertificateFile $gatewayCertCerPath
```

### <a name="step-8"></a>Étape 8

Configurez les paramètres de serveur principal HTTP de la passerelle Application Gateway. Définissez notamment une limite de délai d’expiration pour les requêtes de serveur backend, après laquelle elles sont annulées. Cette valeur est différente du délai d’expiration de la sonde.

```powershell
$apimPoolSetting = New-AzApplicationGatewayBackendHttpSettings -Name "apimPoolSetting" -Port 443 -Protocol "Https" -CookieBasedAffinity "Disabled" -Probe $apimprobe -AuthenticationCertificates $authcert -RequestTimeout 180
$apimPoolPortalSetting = New-AzApplicationGatewayBackendHttpSettings -Name "apimPoolPortalSetting" -Port 443 -Protocol "Https" -CookieBasedAffinity "Disabled" -Probe $apimPortalProbe -AuthenticationCertificates $authcert -RequestTimeout 180
```

### <a name="step-9"></a>Étape 9

Configurez un pool d’adresses IP du serveur principal nommé **apimbackend** avec l’adresse IP virtuelle interne du service Gestion des API créé ci-dessus.

```powershell
$apimProxyBackendPool = New-AzApplicationGatewayBackendAddressPool -Name "apimbackend" -BackendIPAddresses $apimService.PrivateIPAddresses[0]
```

### <a name="step-10"></a>Étape 10

Créez des règles pour la passerelle Application Gateway pour utiliser le routage de base.

```powershell
$rule01 = New-AzApplicationGatewayRequestRoutingRule -Name "rule1" -RuleType Basic -HttpListener $listener -BackendAddressPool $apimProxyBackendPool -BackendHttpSettings $apimPoolSetting
$rule02 = New-AzApplicationGatewayRequestRoutingRule -Name "rule2" -RuleType Basic -HttpListener $portalListener -BackendAddressPool $apimProxyBackendPool -BackendHttpSettings $apimPoolPortalSetting
```

> [!TIP]
> Changez la valeur -RuleType et le routage afin de limiter l’accès à certaines pages du portail des développeurs.

### <a name="step-11"></a>Étape 11

Configurez le nombre d’instances et la taille de la passerelle Application Gateway. Dans cet exemple, nous utilisons la [référence (SKU) WAF](../web-application-firewall/ag/ag-overview.md) pour renforcer la sécurité de la ressource du service Gestion des API.

```powershell
$sku = New-AzApplicationGatewaySku -Name "WAF_Medium" -Tier "WAF" -Capacity 2
```

### <a name="step-12"></a>Étape 12

Configurez WAF pour passer en mode Prévention.

```powershell
$config = New-AzApplicationGatewayWebApplicationFirewallConfiguration -Enabled $true -FirewallMode "Prevention"
```

## <a name="create-application-gateway"></a>Créer une passerelle Application Gateway

Créez une passerelle Application Gateway avec tous les objets de configuration des étapes précédentes.

```powershell
$appgwName = "apim-app-gw"
$appgw = New-AzApplicationGateway -Name $appgwName -ResourceGroupName $resGroupName -Location $location -BackendAddressPools $apimProxyBackendPool -BackendHttpSettingsCollection $apimPoolSetting, $apimPoolPortalSetting  -FrontendIpConfigurations $fipconfig01 -GatewayIpConfigurations $gipconfig -FrontendPorts $fp01 -HttpListeners $listener, $portalListener -RequestRoutingRules $rule01, $rule02 -Sku $sku -WebApplicationFirewallConfig $config -SslCertificates $cert, $certPortal -AuthenticationCertificates $authcert -Probes $apimprobe, $apimPortalProbe
```

## <a name="cname-the-api-management-proxy-hostname-to-the-public-dns-name-of-the-application-gateway-resource"></a>Définition du CNAME du nom d’hôte du proxy du service Gestion des API sur le nom DNS public de la ressource Application Gateway

Une fois la passerelle créée, l’étape suivante consiste à configurer le serveur frontal pour la communication. Lorsque vous utilisez une adresse IP publique, la passerelle Application Gateway requiert un nom DNS attribué dynamiquement, qui risque de ne pas être facile à utiliser.

Le nom DNS de la passerelle Application Gateway doit être utilisé pour créer un enregistrement CNAME qui pointe le nom d’hôte proxy APIM (`api.contoso.net` dans les exemples ci-dessus) vers ce nom DNS. Pour configurer l’enregistrement CNAME d’adresses IP frontales, récupérez les détails de la passerelle Application Gateway et de son nom IP/DNS associé à l’aide de l’élément PublicIPAddress. L’utilisation de A-records n’est pas recommandée étant donné que l’adresse IP virtuelle peut changer lors du redémarrage de la passerelle.

```powershell
Get-AzPublicIpAddress -ResourceGroupName $resGroupName -Name "publicIP01"
```

## <a name="summary"></a><a name="summary"> </a> Récapitulatif
Le service Gestion des API Azure configuré dans un réseau virtuel fournit une interface de passerelle unique pour l’ensemble des API configurées, qu’elles soient hébergées en local ou dans le cloud. L’intégration d’Application Gateway au service Gestion des API vous permet d’activer facilement l’accessibilité d’API particulières sur Internet, tout en fournissant un pare-feu d’applications web en tant que pare-feu frontal pour votre instance de service Gestion des API.

## <a name="next-steps"></a><a name="next-steps"> </a>Étapes suivantes
* En savoir plus sur Azure Application Gateway
  * [Vue d’ensemble d’Application Gateway](../application-gateway/overview.md)
  * [Pare-feu d’applications web sur Application Gateway](../web-application-firewall/ag/ag-overview.md)
  * [Application Gateway à l’aide du routage basé sur le chemin](../application-gateway/tutorial-url-route-powershell.md)
* En savoir plus sur le service Gestion des API et les réseaux virtuels
  * [Utilisation de Gestion des API disponible uniquement dans le réseau virtuel](api-management-using-with-internal-vnet.md)
  * [Avec la gestion des API dans le réseau virtuel](api-management-using-with-vnet.md)

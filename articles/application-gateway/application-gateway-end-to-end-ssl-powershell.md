---
title: Configurer le protocole TLS de bout en bout avec Azure Application Gateway
description: Cet article explique comment configurer le protocole TLS de bout en bout avec Azure Application Gateway en utilisant PowerShell
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 06/09/2020
ms.author: victorh
ms.openlocfilehash: 47891dfa7fc0c9b30ccdbf2ed7710125eb36e4a3
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397805"
---
# <a name="configure-end-to-end-tls-by-using-application-gateway-with-powershell"></a>Configurer le protocole TLS de bout en bout avec Application Gateway en utilisant PowerShell

## <a name="overview"></a>Vue d’ensemble

Azure Application Gateway prend en charge le chiffrement de bout en bout du trafic. Application Gateway arrête la connexion TLS/SSL au niveau de la passerelle d’application. La passerelle applique ensuite les règles d'acheminement au trafic, rechiffre le paquet, puis transfère celui-ci au serveur principal approprié selon les règles d'acheminement définies. Toute réponse du serveur web passe par le même processus vers l’utilisateur final.

Application Gateway prend en charge la définition d’options TLS personnalisées. La passerelle prend également en charge la désactivation des versions suivantes du protocole : **TLSv1.0** , **TLSv1.1** et **TLSv1.2** , ainsi que la définition des suites de chiffrement à utiliser et leur ordre de préférence. Pour en savoir plus sur les options TLS configurables, consultez cette [vue d’ensemble de la stratégie TLS](application-gateway-SSL-policy-overview.md).

> [!NOTE]
> SSL 2.0 et SSL 3.0 sont désactivés par défaut et ne peuvent pas être activés. Ces protocoles sont considérés comme non sécurisés et ne peuvent pas être utilisés avec Application Gateway.

![image du scénario][scenario]

## <a name="scenario"></a>Scénario

Dans ce scénario, vous allez apprendre à créer une passerelle d’application en utilisant un chiffrement TLS de bout en bout avec PowerShell.

Ce scénario va :

* créer un groupe de ressources nommé **appgw-rg**  ;
* créer un réseau virtuel nommé **appgwvnet** avec un espace d’adressage de **10.0.0.0/16**  ;
* créer deux sous-réseaux appelés **appgwsubnet** et **appsubnet** ;
* créer une petite passerelle d’application prenant en charge le chiffrement TLS de bout en bout qui limite les versions de protocole TLS et les suites de chiffrement.

## <a name="before-you-begin"></a>Avant de commencer

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Pour configurer un chiffrement TLS de bout en bout avec une passerelle d’application, un certificat est requis pour la passerelle et des certificats sont requis pour les serveurs back-end. Le certificat de passerelle est utilisé pour dériver une clé symétrique conformément aux spécifications du protocole TLS. La clé symétrique est ensuite utilisée pour chiffrer et déchiffrer le trafic envoyé à la passerelle. Le certificat de passerelle doit être partagé au format Personal Information Exchange (PFX). Ce format de fichier permet d’exporter la clé privée requise par la passerelle d’application pour effectuer le chiffrement et le déchiffrement du trafic.

Pour le chiffrement TLS de bout en bout, le serveur principal doit être expressément approuvé par la passerelle d’application. Chargez le certificat public des serveurs principaux sur la passerelle d'application. L’ajout du certificat permet à la passerelle d’application de communiquer uniquement avec des instances de serveur principal connues. Il sécurise la communication de bout en bout.

Ce processus de configuration est décrit dans les sections suivantes.

## <a name="create-the-resource-group"></a>Créer le groupe de ressources

Cette section vous guide lors de la création d’un groupe de ressources contenant la passerelle d’application.

1. Connectez-vous à votre compte Azure.

   ```powershell
   Connect-AzAccount
   ```

2. Sélectionnez l’abonnement à utiliser pour ce scénario.

   ```powershell
   Select-Azsubscription -SubscriptionName "<Subscription name>"
   ```

3. Créez un groupe de ressources. (Ignorez cette étape si vous utilisez un groupe de ressources existant.)

   ```powershell
   New-AzResourceGroup -Name appgw-rg -Location "West US"
   ```

## <a name="create-a-virtual-network-and-a-subnet-for-the-application-gateway"></a>Créer un réseau virtuel et un sous-réseau pour la passerelle Application Gateway

L’exemple suivant crée un réseau virtuel et deux sous-réseaux. Un sous-réseau sert à héberger la passerelle d’application. Le second sous-réseau est utilisé pour les serveurs principaux hébergeant l’application web.

1. Affectez une plage d’adresses au sous-réseau à utiliser pour la passerelle d’application.

   ```powershell
   $gwSubnet = New-AzVirtualNetworkSubnetConfig -Name 'appgwsubnet' -AddressPrefix 10.0.0.0/24
   ```

   > [!NOTE]
   > Les sous-réseaux configurés pour une passerelle d’application doivent être correctement dimensionnés. Une passerelle d’application peut être configurée pour 10 instances maximum. Chaque instance prend une adresse IP du sous-réseau. Un sous-réseau trop petit peut nuire à la montée en charge d’une passerelle d’application.
   >

2. Affectez une plage d’adresses au pool d’adresses principales.

   ```powershell
   $nicSubnet = New-AzVirtualNetworkSubnetConfig  -Name 'appsubnet' -AddressPrefix 10.0.2.0/24
   ```

3. Créez un réseau virtuel avec les sous-réseaux définis dans les étapes précédentes.

   ```powershell
   $vnet = New-AzvirtualNetwork -Name 'appgwvnet' -ResourceGroupName appgw-rg -Location "West US" -AddressPrefix 10.0.0.0/16 -Subnet $gwSubnet, $nicSubnet
   ```

4. Récupérez les ressources de réseau virtuel et les ressources de sous-réseau à utiliser dans les étapes qui suivent.

   ```powershell
   $vnet = Get-AzvirtualNetwork -Name 'appgwvnet' -ResourceGroupName appgw-rg
   $gwSubnet = Get-AzVirtualNetworkSubnetConfig -Name 'appgwsubnet' -VirtualNetwork $vnet
   $nicSubnet = Get-AzVirtualNetworkSubnetConfig -Name 'appsubnet' -VirtualNetwork $vnet
   ```

## <a name="create-a-public-ip-address-for-the-front-end-configuration"></a>Création d'une adresse IP publique pour la configuration frontale

Créez une ressource IP publique à utiliser pour la passerelle d’application. Cette adresse IP publique est utilisée dans une des étapes qui suivent.

```powershell
$publicip = New-AzPublicIpAddress -ResourceGroupName appgw-rg -Name 'publicIP01' -Location "West US" -AllocationMethod Dynamic
```

> [!IMPORTANT]
> Application Gateway ne prend pas en charge l’utilisation d’une adresse IP publique créée avec un nom de domaine défini. Seule une adresse IP publique avec un nom de domaine créé dynamiquement est prise en charge. Si vous avez besoin d’un nom DNS convivial pour la passerelle d’application, nous vous recommandons d’utiliser un enregistrement CNAME en tant qu’alias.

## <a name="create-an-application-gateway-configuration-object"></a>Créer un objet de configuration de passerelle Application Gateway

Tous les éléments de configuration sont définis avant la création de la passerelle Application Gateway. Les étapes suivantes permettent de créer les éléments de configuration nécessaires à une ressource Application Gateway.

1. Créez une configuration IP de passerelle d’application. Ce paramètre indique quel sous-réseau la passerelle d’application utilise. Au démarrage, la passerelle d’application sélectionne une adresse IP du sous-réseau configuré et achemine le trafic réseau vers les adresses IP du pool IP principal. Gardez à l’esprit que chaque instance utilise une adresse IP unique.

   ```powershell
   $gipconfig = New-AzApplicationGatewayIPConfiguration -Name 'gwconfig' -Subnet $gwSubnet
   ```

2. Créez une configuration IP frontale. Ce paramètre mappe une adresse IP privée ou publique au composant frontal de la passerelle d’application. L’étape suivante associe l’adresse IP publique à l’étape précédente à la configuration IP frontale.

   ```powershell
   $fipconfig = New-AzApplicationGatewayFrontendIPConfig -Name 'fip01' -PublicIPAddress $publicip
   ```

3. Configurez le pool d’adresses IP principales avec les adresses IP des serveurs web principaux. Il s’agit des adresses IP qui recevront le trafic réseau provenant du point de terminaison IP frontal. Remplacez les adresses IP de l’exemple par vos propres points de terminaison d’adresse IP d’application.

   ```powershell
   $pool = New-AzApplicationGatewayBackendAddressPool -Name 'pool01' -BackendIPAddresses 1.1.1.1, 2.2.2.2, 3.3.3.3
   ```

   > [!NOTE]
   > Un nom de domaine complet (FQDN) est également une valeur valide à utiliser pour remplacer une adresse IP dans les serveurs principaux. Pour l’activer, utilisez le commutateur **-BackendFqdns**. 

4. Configurez le port IP frontal pour le point de terminaison IP public. Ce port est le port auquel les utilisateurs finaux se connectent.

   ```powershell
   $fp = New-AzApplicationGatewayFrontendPort -Name 'port01'  -Port 443
   ```

5. Configurez le certificat pour la passerelle d’application. Ce certificat sert à chiffrer et à rechiffrer le trafic sur la passerelle d’application.

   ```powershell
   $passwd = ConvertTo-SecureString  <certificate file password> -AsPlainText -Force 
   $cert = New-AzApplicationGatewaySSLCertificate -Name cert01 -CertificateFile <full path to .pfx file> -Password $passwd 
   ```

   > [!NOTE]
   > Cet exemple configure le certificat utilisé pour la connexion TLS. Le certificat doit être au format .pfx et le mot de passe contenir entre 4 et 12 caractères.

6. Créez l’écouteur HTTP pour la passerelle d’application. Affectez la configuration IP frontale, le port et le certificat TLS/SSL à utiliser.

   ```powershell
   $listener = New-AzApplicationGatewayHttpListener -Name listener01 -Protocol Https -FrontendIPConfiguration $fipconfig -FrontendPort $fp -SSLCertificate $cert
   ```

7. Chargez le certificat à utiliser sur les ressources du pool de back-ends pour lequel TLS est activé.

   > [!NOTE]
   > La sonde par défaut obtient la clé publique de la liaison TLS *par défaut* sur l’adresse IP du serveur principal et compare la valeur de clé publique reçue à celle que vous fournissez ici. 
   > 
   > Si vous utilisez des en-têtes d’hôte et une indication du nom du serveur (SNI) sur le serveur principal, la clé publique récupérée n’est pas nécessairement le site vers lequel vous souhaitez que le trafic soit dirigé. En cas de doute, visitez https://127.0.0.1/ sur les serveurs principaux pour confirmer le certificat utilisé pour la liaison TLS *par défaut*. Utilisez la clé publique de cette demande dans cette section. Si vous utilisez des en-têtes d’hôte et une indication du nom du serveur (SNI) sur les liaisons HTTPS et que vous ne recevez pas une réponse et un certificat à partir d’une demande de navigateur manuelle vers https://127.0.0.1/ sur les serveurs principaux, vous devez configurer une liaison TLS par défaut sur ceux-ci. Si vous ne le faites pas, les sondes échouent et le serveur principal n’est pas approuvé.
   
   Pour plus d’informations sur SNI dans Application Gateway, consultez [Présentation de la terminaison TLS et du chiffrement TLS de bout en bout avec Application Gateway](ssl-overview.md).

   ```powershell
   $authcert = New-AzApplicationGatewayAuthenticationCertificate -Name 'allowlistcert1' -CertificateFile C:\cert.cer
   ```

   > [!NOTE]
   > Le certificat fourni dans l’étape précédente doit être la clé publique du certificat .pfx présent sur le serveur principal. Exportez le certificat (pas le certificat racine) installé sur le serveur principal au format .CER (Claim, Evidence, and Reasoning) et utilisez-le dans cette étape. Cette étape autorise le serveur principal avec la passerelle d’application.

   Si vous utilisez la référence SKU Application Gateway v2, créez un certificat racine approuvé au lieu d’un certificat d’authentification. Pour plus d’informations, consultez [Présentation du chiffrement TLS de bout en bout sur la passerelle Application Gateway](ssl-overview.md#end-to-end-tls-with-the-v2-sku) :

   ```powershell
   $trustedRootCert01 = New-AzApplicationGatewayTrustedRootCertificate -Name "test1" -CertificateFile  <path to root cert file>
   ```

8. Configurez les paramètres HTTP du serveur principal de la passerelle d’application. Affectez le certificat chargé à l’étape précédente aux paramètres HTTP.

   ```powershell
   $poolSetting = New-AzApplicationGatewayBackendHttpSettings -Name 'setting01' -Port 443 -Protocol Https -CookieBasedAffinity Enabled -AuthenticationCertificates $authcert
   ```

   Pour la référence SKU Application Gateway v2, utilisez la commande suivante :

   ```powershell
   $poolSetting01 = New-AzApplicationGatewayBackendHttpSettings -Name “setting01” -Port 443 -Protocol Https -CookieBasedAffinity Disabled -TrustedRootCertificate $trustedRootCert01 -HostName "test1"
   ```

9. Créez une règle d’acheminement d’équilibreur de charge qui configure le comportement de l’équilibreur de charge. Dans cet exemple, une simple règle de type tourniquet (round robin) est créée.

   ```powershell
   $rule = New-AzApplicationGatewayRequestRoutingRule -Name 'rule01' -RuleType basic -BackendHttpSettings $poolSetting -HttpListener $listener -BackendAddressPool $pool
   ```

10. Configurez la taille d’instance de la passerelle Application Gateway. Les tailles disponibles sont **Standard\_Small** , **Standard\_Medium** et **Standard\_Large**.  Pour la capacité, les valeurs disponibles vont de **1** à **10**.

    ```powershell
    $sku = New-AzApplicationGatewaySku -Name Standard_Small -Tier Standard -Capacity 2
    ```

    > [!NOTE]
    > Vous pouvez choisir un nombre d’instances de 1 à des fins de test. Il est important de savoir que n’importe quel nombre d’instances inférieur à 2 n’est pas couvert par le contrat SLA et n’est donc pas recommandé. Les petites passerelles doivent être utilisées pour les tests de développement et non à des fins de production.

11. Configurez la stratégie TLS à utiliser sur la passerelle d’application. La passerelle Application Gateway prend en charge la possibilité de définir une version minimale pour les versions du protocole TLS.

    Les valeurs suivantes représentent la liste des versions de protocole pouvant être définies :

    - **TLSV1_0**
    - **TLSV1_1**
    - **TLSV1_2**
    
    L’exemple suivant définit la version de protocole minimale sur **TLSv1_2** et active **TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_GCM\_SHA256** , **TLS\_ECDHE\_ECDSA\_WITH\_AES\_256\_GCM\_SHA384** et **TLS\_RSA\_WITH\_AES\_128\_GCM\_SHA256** uniquement.

    ```powershell
    $SSLPolicy = New-AzApplicationGatewaySSLPolicy -MinProtocolVersion TLSv1_2 -CipherSuite "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256", "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384", "TLS_RSA_WITH_AES_128_GCM_SHA256" -PolicyType Custom
    ```

## <a name="create-the-application-gateway"></a>Créer la passerelle Application Gateway

À l’aide des étapes précédentes, créez la passerelle d’application. La création de la passerelle est un processus dont la durée d’exécution est longue.

Pour la référence SKU v1, utilisez la commande ci-dessous
```powershell
$appgw = New-AzApplicationGateway -Name appgateway -SSLCertificates $cert -ResourceGroupName "appgw-rg" -Location "West US" -BackendAddressPools $pool -BackendHttpSettingsCollection $poolSetting -FrontendIpConfigurations $fipconfig -GatewayIpConfigurations $gipconfig -FrontendPorts $fp -HttpListeners $listener -RequestRoutingRules $rule -Sku $sku -SSLPolicy $SSLPolicy -AuthenticationCertificates $authcert -Verbose
```

Pour la référence SKU v2, utilisez la commande ci-dessous
```powershell
$appgw = New-AzApplicationGateway -Name appgateway -SSLCertificates $cert -ResourceGroupName "appgw-rg" -Location "West US" -BackendAddressPools $pool -BackendHttpSettingsCollection $poolSetting01 -FrontendIpConfigurations $fipconfig -GatewayIpConfigurations $gipconfig -FrontendPorts $fp -HttpListeners $listener -RequestRoutingRules $rule -Sku $sku -SSLPolicy $SSLPolicy -TrustedRootCertificate $trustedRootCert01 -Verbose
```

## <a name="apply-a-new-certificate-if-the-back-end-certificate-is-expired"></a>Appliquer un nouveau certificat si le certificat de serveur principal a expiré

Utiliser cette procédure pour appliquer un nouveau certificat si le certificat de serveur principal a expiré

1. Récupérez la passerelle d’application à mettre à jour.

   ```powershell
   $gw = Get-AzApplicationGateway -Name AdatumAppGateway -ResourceGroupName AdatumAppGatewayRG
   ```
   
2. Ajouter la nouvelle ressource de certificat à partir du fichier .cer qui contient la clé publique du certificat. Il peut également s’agir du même certificat que celui ajouté à l’écouteur pour la terminaison TLS au niveau de la passerelle d’application.

   ```powershell
   Add-AzApplicationGatewayAuthenticationCertificate -ApplicationGateway $gw -Name 'NewCert' -CertificateFile "appgw_NewCert.cer" 
   ```
    
3. Obtenez le nouvel objet de certificat d’authentification dans une variable (TypeName : Microsoft.Azure.Commands.Network.Models.PSApplicationGatewayAuthenticationCertificate).

   ```powershell
   $AuthCert = Get-AzApplicationGatewayAuthenticationCertificate -ApplicationGateway $gw -Name NewCert
   ```
 
 4. Affecter le nouveau certificat au paramètre **BackendHttp** et faites-y référence avec la variable $AuthCert. Spécifiez le nom du paramètre HTTP que vous souhaitez modifier.
 
   ```powershell
   $out= Set-AzApplicationGatewayBackendHttpSetting -ApplicationGateway $gw -Name "HTTP1" -Port 443 -Protocol "Https" -CookieBasedAffinity Disabled -AuthenticationCertificates $Authcert
   ```
    
 5. Valider la modification dans la passerelle d’application et transmettre la nouvelle configuration contenue dans la variable $out.
 
   ```powershell
   Set-AzApplicationGateway -ApplicationGateway $gw  
   ```

## <a name="remove-an-unused-expired-certificate-from-http-settings"></a>Supprimer un certificat expiré inutilisé à partir des paramètres HTTP

Utilisez cette procédure pour supprimer un certificat expiré inutilisé à partir des paramètres HTTP.

1. Récupérez la passerelle d’application à mettre à jour.

   ```powershell
   $gw = Get-AzApplicationGateway -Name AdatumAppGateway -ResourceGroupName AdatumAppGatewayRG
   ```
   
2. Répertoriez le nom du certificat d’authentification que vous souhaitez supprimer.

   ```powershell
   Get-AzApplicationGatewayAuthenticationCertificate -ApplicationGateway $gw | select name
   ```
    
3. Supprimez le certificat d’authentification trouvé d’une passerelle d’application.

   ```powershell
   $gw=Remove-AzApplicationGatewayAuthenticationCertificate -ApplicationGateway $gw -Name ExpiredCert
   ```
 
 4. Validez la modification.
 
   ```powershell
   Set-AzApplicationGateway -ApplicationGateway $gw
   ```

   
## <a name="limit-tls-protocol-versions-on-an-existing-application-gateway"></a>Limiter les versions du protocole TLS sur une passerelle d’application existante

Les étapes précédentes vous ont guidé dans la création d’une application en utilisant un chiffrement TLS de bout en bout et en désactivant certaines versions du protocole TLS. L’exemple suivant désactive certaines stratégies TLS sur une passerelle d’application existante.

1. Récupérez la passerelle d’application à mettre à jour.

   ```powershell
   $gw = Get-AzApplicationGateway -Name AdatumAppGateway -ResourceGroupName AdatumAppGatewayRG
   ```

2. Définissez une stratégie TLS. Dans l’exemple suivant, **TLSv1.0** et **TLSv1.1** sont désactivées et les suites de chiffrement **TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_GCM\_SHA256** , **TLS\_ECDHE\_ECDSA\_WITH\_AES\_256\_GCM\_SHA384** et **TLS\_RSA\_WITH\_AES\_128\_GCM\_SHA256** sont les seules autorisées.

   ```powershell
   Set-AzApplicationGatewaySSLPolicy -MinProtocolVersion TLSv1_2 -PolicyType Custom -CipherSuite "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256", "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384", "TLS_RSA_WITH_AES_128_GCM_SHA256" -ApplicationGateway $gw

   ```

3. Pour finir, mettez à jour la passerelle. Cette dernière étape est une tâche de longue haleine. Une fois l’opération terminée, le chiffrement TLS de bout en bout est configuré sur la passerelle d’application.

   ```powershell
   $gw | Set-AzApplicationGateway
   ```

## <a name="get-an-application-gateway-dns-name"></a>Obtenir le nom DNS d’une passerelle d’application

Une fois la passerelle créée, l’étape suivante consiste à configurer le serveur frontal pour la communication. La passerelle Application Gateway requiert un nom DNS attribué dynamiquement lorsque vous utilisez une adresse IP publique, ce qui n’est pas convivial. Pour s’assurer que les utilisateurs finaux peuvent atteindre la passerelle Application Gateway, vous pouvez utiliser un enregistrement CNAME pour pointer vers le point de terminaison public de la passerelle d’application. Pour plus d’informations, consultez la page [Configuration d’un nom de domaine personnalisé dans Azure](../cloud-services/cloud-services-custom-domain-name-portal.md). 

Pour configurer un alias, récupérez les détails de la passerelle d’application et son nom IP/DNS associé à l’aide de l’élément **PublicIPAddress** qui lui est attaché. Utilisez le nom DNS de la passerelle d’application pour créer un enregistrement CNAME qui pointe les deux applications web vers ce nom DNS. L’utilisation de A-records n’est pas recommandée étant donné que l’adresse IP virtuelle peut changer lors du redémarrage de la passerelle d’application.

```powershell
Get-AzPublicIpAddress -ResourceGroupName appgw-RG -Name publicIP01
```

```
Name                     : publicIP01
ResourceGroupName        : appgw-RG
Location                 : westus
Id                       : /subscriptions/<subscription_id>/resourceGroups/appgw-RG/providers/Microsoft.Network/publicIPAddresses/publicIP01
Etag                     : W/"00000d5b-54ed-4907-bae8-99bd5766d0e5"
ResourceGuid             : 00000000-0000-0000-0000-000000000000
ProvisioningState        : Succeeded
Tags                     : 
PublicIpAllocationMethod : Dynamic
IpAddress                : xx.xx.xxx.xx
PublicIpAddressVersion   : IPv4
IdleTimeoutInMinutes     : 4
IpConfiguration          : {
                                "Id": "/subscriptions/<subscription_id>/resourceGroups/appgw-RG/providers/Microsoft.Network/applicationGateways/appgwtest/frontendIP
                            Configurations/frontend1"
                            }
DnsSettings              : {
                                "Fqdn": "00000000-0000-xxxx-xxxx-xxxxxxxxxxxx.cloudapp.net"
                            }
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le renforcement de la sécurité de vos applications web avec un pare-feu d’applications web via Application Gateway, consultez [Vue d’ensemble du pare-feu d’applications web](../web-application-firewall/ag/ag-overview.md).

[scenario]: ./media/application-gateway-end-to-end-SSL-powershell/scenario.png
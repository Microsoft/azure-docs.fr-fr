---
title: Créer une sonde personnalisée à l’aide du modèle de déploiement personnalisé – Azure Application Gateway
description: Apprendre à créer une sonde personnalisée pour Application Gateway en utilisant PowerShell dans le modèle de déploiement classique
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/13/2019
ms.author: victorh
ms.openlocfilehash: 13441899eeb5ca2b7c60977ab2858fe40a398d1a
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397856"
---
# <a name="create-a-custom-probe-for-azure-application-gateway-classic-by-using-powershell"></a>Créer une sonde personnalisée pour Azure Application Gateway (classique) en utilisant PowerShell

> [!div class="op_single_selector"]
> * [Azure portal](application-gateway-create-probe-portal.md)
> * [Commandes PowerShell pour Azure Resource Manager](application-gateway-create-probe-ps.md)
> * [Azure Classic PowerShell](application-gateway-create-probe-classic-ps.md)

Dans cet article, une sonde personnalisée est ajoutée à une passerelle d’application existante à l’aide de PowerShell. Les sondes personnalisées sont utiles pour les applications qui ont une page de contrôle d’intégrité spécifique ou pour les applications qui ne fournissent pas de réponse correcte dans l’application web par défaut.

> [!IMPORTANT]
> Azure a deux modèles de déploiement différents pour créer et utiliser des ressources : [Resource Manager et classique](../azure-resource-manager/management/deployment-models.md). Cet article traite du modèle de déploiement classique. Pour la plupart des nouveaux déploiements, Microsoft recommande d’utiliser le modèle Resource Manager. Découvrez comment [effectuer ces étapes à l’aide du modèle Resource Manager](application-gateway-create-probe-ps.md).

[!INCLUDE [azure-ps-prerequisites-include.md](../../includes/azure-ps-prerequisites-include.md)]

## <a name="create-an-application-gateway"></a>Créer une passerelle Application Gateway

Pour créer une passerelle d’application :

1. Créez une ressource Application Gateway.
2. Créez un fichier XML de configuration ou un objet de configuration.
3. Validez la configuration de la ressource Application Gateway nouvellement créée.

### <a name="create-an-application-gateway-resource-with-a-custom-probe"></a>Créer une ressource de passerelle d’application avec une sonde personnalisée

Pour créer la passerelle, utilisez l’applet de commande `New-AzureApplicationGateway` en remplaçant les valeurs par les vôtres. La facturation de la passerelle ne démarre pas à ce stade. La facturation commence à une étape ultérieure, lorsque la passerelle a démarré correctement.

L’exemple suivant illustre la création d’une nouvelle passerelle d’application avec un réseau virtuel appelé « testvnet1 » et un sous-réseau appelé « subnet-1 ».

```powershell
New-AzureApplicationGateway -Name AppGwTest -VnetName testvnet1 -Subnets @("Subnet-1")
```

Pour valider la création de la passerelle, vous pouvez utiliser l’applet de commande `Get-AzureApplicationGateway`.

```powershell
Get-AzureApplicationGateway AppGwTest
```

> [!NOTE]
> La valeur par défaut du paramètre *InstanceCount* est de 2, avec une valeur maximale de 10. La valeur par défaut du paramètre *GatewaySize* est Medium. Vous avez le choix entre Small, Medium et Large.
> 
> 

Les paramètres *VirtualIPs* et *DnsName* sont sans valeur, car la passerelle n’a pas encore démarré. Ces valeurs seront créées une fois la passerelle en cours d’exécution.

### <a name="configure-an-application-gateway-by-using-xml"></a>Configurer une passerelle d’application à l’aide de XML

Dans l’exemple ci-dessous, vous allez utiliser un fichier XML pour configurer tous les paramètres de la passerelle d’application et les valider dans la ressource de passerelle d’application.  

Copiez le texte suivant dans le Bloc-notes.

```xml
<ApplicationGatewayConfiguration xmlns:i="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/windowsazure">
<FrontendIPConfigurations>
    <FrontendIPConfiguration>
        <Name>fip1</Name>
        <Type>Private</Type>
    </FrontendIPConfiguration>
</FrontendIPConfigurations>
<FrontendPorts>
    <FrontendPort>
        <Name>port1</Name>
        <Port>80</Port>
    </FrontendPort>
</FrontendPorts>
<Probes>
    <Probe>
        <Name>Probe01</Name>
        <Protocol>Http</Protocol>
        <Host>contoso.com</Host>
        <Path>/path/custompath.htm</Path>
        <Interval>15</Interval>
        <Timeout>15</Timeout>
        <UnhealthyThreshold>5</UnhealthyThreshold>
    </Probe>
    </Probes>
    <BackendAddressPools>
    <BackendAddressPool>
        <Name>pool1</Name>
        <IPAddresses>
            <IPAddress>1.1.1.1</IPAddress>
            <IPAddress>2.2.2.2</IPAddress>
        </IPAddresses>
    </BackendAddressPool>
</BackendAddressPools>
<BackendHttpSettingsList>
    <BackendHttpSettings>
        <Name>setting1</Name>
        <Port>80</Port>
        <Protocol>Http</Protocol>
        <CookieBasedAffinity>Enabled</CookieBasedAffinity>
        <RequestTimeout>120</RequestTimeout>
        <Probe>Probe01</Probe>
    </BackendHttpSettings>
</BackendHttpSettingsList>
<HttpListeners>
    <HttpListener>
        <Name>listener1</Name>
        <FrontendIP>fip1</FrontendIP>
    <FrontendPort>port1</FrontendPort>
        <Protocol>Http</Protocol>
    </HttpListener>
</HttpListeners>
<HttpLoadBalancingRules>
    <HttpLoadBalancingRule>
        <Name>lbrule1</Name>
        <Type>basic</Type>
        <BackendHttpSettings>setting1</BackendHttpSettings>
        <Listener>listener1</Listener>
        <BackendAddressPool>pool1</BackendAddressPool>
    </HttpLoadBalancingRule>
</HttpLoadBalancingRules>
</ApplicationGatewayConfiguration>
```

Modifiez les valeurs entre parenthèses pour les éléments de configuration. Enregistrez le fichier avec l’extension .xml.

L’exemple suivant montre comment utiliser un fichier de configuration pour configurer la passerelle d’application en vue d’équilibrer la charge du trafic HTTP sur le port public 80 et d’orienter le trafic réseau vers le port 80 principal entre deux adresses IP en utilisant une sonde personnalisée.

> [!IMPORTANT]
> L’élément de protocole Http ou Https est sensible à la casse.

Un nouvel élément de configuration \<Probe\> est ajouté pour configurer les sondes personnalisées.

Les paramètres de configuration sont :

|Paramètre|Description|
|---|---|
|**Nom** |Nom de référence de la sonde personnalisée. |
| **Protocole** | Protocole utilisé (les valeurs possibles sont HTTP ou HTTPS).|
| **Hôte** et **Chemin** | Chemin complet de l’URL qui est appelé par la passerelle d’application pour déterminer l’intégrité de l’instance. Par exemple : avec un site web http:\//contoso.com/, la sonde personnalisée peut être configurée pour « http:\//contoso.com/path/custompath.htm » afin que les contrôles de sonde renvoient une réponse HTTP réussie.|
| **Intervalle** | Configure les vérifications d’intervalle de sonde en secondes.|
| **Délai d'expiration** | Définit le délai d’expiration d’un contrôle de réponse HTTP.|
| **Seuil de défaillance sur le plan de l’intégrité** | Le nombre d’échecs de réponses HTTP nécessaires pour marquer l’instance de serveur principal comme *défectueuse*.|

Le nom de la sonde est référencé dans la configuration \<BackendHttpSettings\> pour affecter le pool principal qui utilise les paramètres de sonde personnalisée.

## <a name="add-a-custom-probe-to-an-existing-application-gateway"></a>Ajoute une sonde personnalisée à une passerelle d’application existante

La modification de la configuration actuelle d’une passerelle d’application se fait en trois étapes : obtenez le fichier de configuration XML actuel, modifiez-le de façon à avoir une probe personnalisée et configurez la passerelle d’application avec les nouveaux paramètres XML.

1. Obtenir le fichier XML à l’aide de `Get-AzureApplicationGatewayConfig`. L’applet de commande exporte le fichier XML de configuration, afin d’être modifié pour y ajouter un paramètre de sonde.

   ```powershell
   Get-AzureApplicationGatewayConfig -Name "<application gateway name>" -Exporttofile "<path to file>"
   ```

1. Ouvrez le fichier XML dans un éditeur de texte. Ajoutez une section `<probe>` après `<frontendport>`.

   ```xml
   <Probes>
    <Probe>
        <Name>Probe01</Name>
        <Protocol>Http</Protocol>
        <Host>contoso.com</Host>
        <Path>/path/custompath.htm</Path>
        <Interval>15</Interval>
        <Timeout>15</Timeout>
        <UnhealthyThreshold>5</UnhealthyThreshold>
    </Probe>
   </Probes>
   ```

   Dans la section backendHttpSettings du fichier XML, ajoutez le nom de la sonde comme dans l’exemple suivant :

   ```xml
    <BackendHttpSettings>
        <Name>setting1</Name>
        <Port>80</Port>
        <Protocol>Http</Protocol>
        <CookieBasedAffinity>Enabled</CookieBasedAffinity>
        <RequestTimeout>120</RequestTimeout>
        <Probe>Probe01</Probe>
    </BackendHttpSettings>
   ```

   Enregistrez le fichier XML.

1. Mettez à jour la configuration de la passerelle d’application avec le nouveau fichier XML avec `Set-AzureApplicationGatewayConfig`. Cette applet de commande met à jour votre passerelle d’application avec cette nouvelle configuration.

```powershell
Set-AzureApplicationGatewayConfig -Name "<application gateway name>" -Configfile "<path to file>"
```

## <a name="next-steps"></a>Étapes suivantes

Si vous souhaitez configurer TLS (Transport Layer Security), précédemment connue sous le nom de déchargement SSL (Secure Sockets Layer), consultez [Configurer une passerelle d'application pour le déchargement TLS](./tutorial-ssl-powershell.md).

Si vous voulez configurer une passerelle Application Gateway à utiliser avec l’équilibreur de charge interne, consultez [Création d’une passerelle Application Gateway avec un équilibrage de charge interne (ILB)](./application-gateway-ilb-arm.md).
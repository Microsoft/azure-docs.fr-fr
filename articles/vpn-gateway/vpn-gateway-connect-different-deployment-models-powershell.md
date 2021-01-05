---
title: 'Connecter des réseaux virtuels classiques à des réseaux virtuels Resource Manager : PowerShell'
description: Créez une connexion VPN entre des réseaux virtuels classiques et des réseaux virtuels Resource Manager à l’aide d’une passerelle VPN et de PowerShell.
services: vpn-gateway
titleSuffix: Azure VPN Gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 10/17/2018
ms.author: cherylmc
ms.openlocfilehash: 63505f470410234f720dd28c29e87c4a2a6d123f
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94661135"
---
# <a name="connect-virtual-networks-from-different-deployment-models-using-powershell"></a>Connecter des réseaux virtuels utilisant des modèles de déploiement différents à l’aide de PowerShell

Cet article vous permet de connecter des réseaux virtuels classiques à des réseaux virtuels Resource Manager, afin d’autoriser les ressources situées dans les modèles de déploiement distincts à communiquer entre elles. Les étapes décrites dans cet article utilisent PowerShell, mais vous pouvez également créer cette configuration à l’aide du portail Azure en sélectionnant l’article dans cette liste.

> [!div class="op_single_selector"]
> * [Portail](vpn-gateway-connect-different-deployment-models-portal.md)
> * [PowerShell](vpn-gateway-connect-different-deployment-models-powershell.md)
> 
> 

La connexion d’un réseau virtuel classique à un réseau virtuel Resource Manager est semblable à la connexion d’un réseau virtuel à un emplacement de site local. Les deux types de connectivité font appel à une passerelle VPN pour offrir un tunnel sécurisé utilisant Ipsec/IKE. Vous pouvez créer une connexion entre des réseaux virtuels situés dans des abonnements différents et des régions différentes. Vous pouvez également connecter des réseaux virtuels qui disposent déjà de connexions à des réseaux locaux, à condition que la passerelle avec laquelle ils ont été configurés soit dynamique ou basée sur un itinéraire. Pour plus d’informations sur les connexions de réseau virtuel à réseau virtuel, consultez le [Forum Aux Questions sur l’interconnexion de réseaux virtuels](#faq) à la fin de cet article. 

Si vous n’avez pas encore de passerelle de réseau virtuel et que vous ne souhaitez pas en créer une, vous voudrez peut-être connecter vos réseaux virtuels en utilisant le peering de réseaux virtuels. L’homologation de réseaux virtuels (ou VNet Peering) n’utilise pas de passerelle VPN. Pour plus d’informations, consultez l’article [Peering de réseaux virtuels](../virtual-network/virtual-network-peering-overview.md).

## <a name="before-you-begin"></a><a name="before"></a>Avant de commencer

Les étapes suivantes vous guident à travers les paramétrages nécessaires pour configurer une passerelle dynamique ou basée sur un itinéraire pour chaque réseau virtuel et créer une connexion VPN entre les passerelles. Cette configuration ne prend pas en charge les passerelles statiques ou basées sur des stratégies.

### <a name="prerequisites"></a><a name="pre"></a>Configuration requise

* Les deux réseaux virtuels ont déjà été créés. Pour créer un réseau virtuel Azure Resource Manager, consultez [Créer un groupe de ressources et un réseau virtuel](../virtual-network/quick-create-powershell.md#create-a-resource-group-and-a-virtual-network). Pour créer un réseau virtuel classique, consultez [Créer un réseau virtuel (Classic)](/previous-versions/azure/virtual-network/create-virtual-network-classic).
* Les plages d’adresses des réseaux virtuels ne se chevauchent pas ou ne chevauchent aucune des plages des autres connexions susceptibles d’être utilisées par les passerelles.
* Vous avez installé les dernières applets de commande PowerShell. Pour plus d’informations, consultez [Installation et configuration d’Azure PowerShell](/powershell/azure/) . Veillez à installer à la fois les applets de commande de gestion des services et Resource Manager (RM). 

### <a name="example-settings"></a><a name="exampleref"></a>Exemples de paramètres

Vous pouvez utiliser ces valeurs pour créer un environnement de test ou vous y référer pour mieux comprendre les exemples de cet article.

**Paramètres de réseau virtuel classique**

Nom du réseau virtuel = ClassicVNet <br>
Emplacement = USA Ouest <br>
Espaces d’adressage du réseau virtuel = 10.0.0.0/24 <br>
Sous-réseau-1 = 10.0.0.0/27 <br>
Sous-réseau de passerelle : 10.0.0.32/29 <br>
Nom du réseau local = RMVNetLocal <br>
Type de passerelle = DynamicRouting

**Paramètres de réseau virtuel Resource Manager**

Nom du réseau virtuel = RMVNet <br>
Groupe de ressources = RG1 <br>
Espaces d’adressage IP du réseau virtuel = 192.168.0.0/16 <br>
Sous-réseau-1 = 192.168.1.0/24 <br>
Sous-réseau de passerelle = 192.168.0.0/26 <br>
Emplacement = USA Est <br>
Nom d’adresse IP publique de passerelle = gwpip <br>
Passerelle de réseau local = ClassicVNetLocal <br>
Nom de passerelle de réseau virtuel = RMGateway <br>
Configuration d’adressage IP de la passerelle = gwipconfig

## <a name="section-1---configure-the-classic-vnet"></a><a name="createsmgw"></a>Section 1 : Configurer le réseau virtuel classique
### <a name="1-download-your-network-configuration-file"></a>1. Télécharger votre fichier de configuration réseau
1. Dans la console PowerShell avec des droits élevés, connectez-vous à votre compte Azure. Les applets de commande suivantes vous invitent à entrer les informations d’identification de connexion pour votre compte Azure. Une fois que vous êtes connecté, l'applet de commande télécharge vos paramètres de compte pour qu'ils soient reconnus par Azure PowerShell. Les applets de commande Azure PowerShell de gestion de service classique sont utilisées dans cette section.

   ```azurepowershell
   Add-AzureAccount
   ```

   Récupérez votre abonnement Azure.

   ```azurepowershell
   Get-AzureSubscription
   ```

   Si vous avez plusieurs abonnements, sélectionnez celui que vous souhaitez utiliser.

   ```azurepowershell
   Select-AzureSubscription -SubscriptionName "Name of subscription"
   ```
2. Exportez votre fichier de configuration réseau Azure en exécutant la commande suivante. Vous pouvez modifier l’emplacement d’exportation du fichier si nécessaire.

   ```azurepowershell
   Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml
   ```
3. Ouvrez le fichier .xml que vous avez téléchargé pour le modifier. Pour obtenir un exemple de fichier de configuration réseau, consultez le [schéma de configuration réseau](/previous-versions/azure/reference/jj157100(v=azure.100)).

### <a name="2-verify-the-gateway-subnet"></a>2. Vérifier le sous-réseau de passerelle
Dans l’élément **VirtualNetworkSites**, ajoutez un sous-réseau de passerelle à votre réseau virtuel si vous n’en avez pas déjà créé un. Lorsque vous travaillez avec le fichier de configuration réseau, le sous-réseau de passerelle DOIT être nommé « GatewaySubnet ». Sinon, Azure ne peut pas le reconnaître et l’utiliser comme sous-réseau de passerelle.

[!INCLUDE [vpn-gateway-no-nsg-include](../../includes/vpn-gateway-no-nsg-include.md)]

**Exemple :**

```xml
<VirtualNetworkSites>
  <VirtualNetworkSite name="ClassicVNet" Location="West US">
    <AddressSpace>
      <AddressPrefix>10.0.0.0/24</AddressPrefix>
    </AddressSpace>
    <Subnets>
      <Subnet name="Subnet-1">
        <AddressPrefix>10.0.0.0/27</AddressPrefix>
      </Subnet>
      <Subnet name="GatewaySubnet">
        <AddressPrefix>10.0.0.32/29</AddressPrefix>
      </Subnet>
    </Subnets>
  </VirtualNetworkSite>
</VirtualNetworkSites>
```

### <a name="3-add-the-local-network-site"></a>3. Ajouter le site de réseau local
Le site de réseau local que vous ajoutez représente le réseau virtuel RM auquel vous souhaitez vous connecter. Ajoutez un élément **LocalNetworkSites** au fichier s’il n’en existe pas déjà un. À ce stade de la configuration, n’importe quelle adresse IP publique valide peut être utilisée pour l’élément VPNGatewayAddress, car nous n’avons pas encore créé la passerelle pour le réseau virtuel Resource Manager. Une fois la passerelle créée, nous remplacerons cette adresse IP temporaire par l’adresse IP publique qui a été affectée à la passerelle RM.

```xml
<LocalNetworkSites>
  <LocalNetworkSite name="RMVNetLocal">
    <AddressSpace>
      <AddressPrefix>192.168.0.0/16</AddressPrefix>
    </AddressSpace>
    <VPNGatewayAddress>13.68.210.16</VPNGatewayAddress>
  </LocalNetworkSite>
</LocalNetworkSites>
```

### <a name="4-associate-the-vnet-with-the-local-network-site"></a>4. Associer le réseau virtuel au site de réseau local
Dans cette section, nous spécifions le site de réseau local auquel vous souhaitez connecter le réseau virtuel. Dans ce cas, il s’agit du réseau virtuel Resource Manager que vous avez référencé plus tôt. Assurez-vous que les noms correspondent. Cette étape ne vise pas à créer une passerelle. Elle spécifie le réseau local auquel la passerelle se connectera.

```xml
<Gateway>
  <ConnectionsToLocalNetwork>
    <LocalNetworkSiteRef name="RMVNetLocal">
      <Connection type="IPsec" />
    </LocalNetworkSiteRef>
  </ConnectionsToLocalNetwork>
</Gateway>
```

### <a name="5-save-the-file-and-upload"></a>5. Enregistrer et charger le fichier
Enregistrez le fichier, puis importez-le dans Azure en exécutant la commande ci-dessous. Assurez-vous que vous modifiez le chemin d'accès selon les besoins de votre environnement.

```azurepowershell
Set-AzureVNetConfig -ConfigurationPath C:\AzureNet\NetworkConfig.xml
```

Vous verrez un résultat similaire indiquant que l’importation a réussi.

```output
OperationDescription        OperationId                      OperationStatus                                                
--------------------        -----------                      ---------------                                                
Set-AzureVNetConfig        e0ee6e66-9167-cfa7-a746-7casb9    Succeeded 
```

### <a name="6-create-the-gateway"></a>6. Créer la passerelle

Avant d’exécuter cet exemple, consultez le fichier de configuration de réseau que vous avez téléchargé pour les noms exacts qu’Azure s’attend à voir. Ce fichier de configuration réseau contient les valeurs de vos réseaux virtuels classiques. Parfois, les noms de réseau virtuel classique sont modifiés dans le fichier de configuration réseau lors de la création des paramètres du réseau virtuel classique dans le portail Azure en raison de différences dans les modèles de déploiement. Par exemple, si vous avez utilisé le portail Azure pour créer un réseau virtuel nommé « Classic VNet » dans un groupe de ressources nommé « ClassicRG », le nom figurant dans le fichier de configuration du réseau est converti en « Group ClassicRG Classic VNetl ». Lorsque vous spécifiez le nom d’un réseau virtuel qui contient des espaces, entourez la valeur avec des guillemets.


L’exemple suivant montre comment créer une passerelle de routage dynamique :

```azurepowershell
New-AzureVNetGateway -VNetName ClassicVNet -GatewayType DynamicRouting
```

Vous pouvez vérifier l’état de la passerelle à l’aide de l’applet de commande **Get-AzureVNetGateway**.

## <a name="section-2---configure-the-rm-vnet-gateway"></a><a name="creatermgw"></a>Section 2 - Configurer la passerelle du réseau virtuel Resource Manager



Les conditions préalables supposent que vous ayez déjà créé un réseau virtuel RM. Dans cette étape, vous créez une passerelle VPN pour le réseau virtuel RM. Ne commencez pas cette procédure avant d’avoir récupéré l’adresse IP publique de la passerelle du réseau virtuel classique. 

1. Connectez-vous à votre compte Azure dans la console PowerShell. Les applets de commande suivantes vous invitent à entrer les informations d’identification de connexion pour votre compte Azure. Une fois que vous êtes connecté, vos paramètres de compte sont téléchargés et transmis à Azure PowerShell. Vous pouvez éventuellement utiliser la fonctionnalité de test pour lancer Azure Cloud Shell dans le navigateur.

   Si vous utilisez Azure Cloud Shell, ignorez la cmdlet suivante :

   ```azurepowershell
   Connect-AzAccount
   ``` 
   Pour vérifier que vous utilisez le bon abonnement, exécutez la cmdlet suivante :  

   ```azurepowershell-interactive
   Get-AzSubscription
   ```
   
   Si vous avez plusieurs abonnements, spécifiez celui que vous souhaitez utiliser.

   ```azurepowershell-interactive
   Select-AzSubscription -SubscriptionName "Name of subscription"
   ```
2. Créer une passerelle de réseau local. Dans un réseau virtuel, la passerelle de réseau local fait généralement référence à votre emplacement local. Dans ce cas, la passerelle de réseau local fait référence à votre réseau virtuel classique. Donnez à cet emplacement un nom auquel Azure pourra se référer, puis spécifiez le préfixe de l’espace d’adressage. Azure utilise le préfixe d’adresse IP que vous spécifiez pour identifier le trafic à envoyer vers votre emplacement local. Si vous avez besoin d’ajuster ces informations ultérieurement, avant de créer votre passerelle, vous pouvez modifier les valeurs et exécuter à nouveau l’exemple.
   
   **-Name** est le nom à affecter pour faire référence à la passerelle de réseau local.<br>
   **-AddressPrefix** est l’espace d’adressage de votre réseau virtuel classique.<br>
   **-GatewayIpAddress** est l’adresse IP publique de la passerelle du réseau virtuel classique. Veillez à remplacer la séquence « n.n.n.n » par l’adresse IP correcte.<br>

   ```azurepowershell-interactive
   New-AzLocalNetworkGateway -Name ClassicVNetLocal `
   -Location "West US" -AddressPrefix "10.0.0.0/24" `
   -GatewayIpAddress "n.n.n.n" -ResourceGroupName RG1
   ```
3. Demandez qu’une adresse IP publique soit allouée à la passerelle de réseau virtuel pour le réseau virtuel Resource Manager. Vous ne pouvez pas spécifier l’adresse IP que vous souhaitez utiliser. L’adresse IP est allouée de façon dynamique à la passerelle de réseau virtuel. Néanmoins, cela ne signifie pas que l’adresse IP est modifiée. L’adresse IP de la passerelle de réseau virtuel change uniquement lorsque la passerelle est supprimée, puis recréée. Elle n’est pas modifiée lors du redimensionnement, de la réinitialisation ou des autres opérations de maintenance/mise à niveau internes de la passerelle.

   Dans cette étape, nous définissons également une variable qui sera utilisée à une étape ultérieure.

   ```azurepowershell-interactive
   $ipaddress = New-AzPublicIpAddress -Name gwpip `
   -ResourceGroupName RG1 -Location 'EastUS' `
   -AllocationMethod Dynamic
   ```

4. Vérifiez que votre réseau virtuel possède un sous-réseau de passerelle. S’il n’existe aucun sous-réseau de passerelle, ajoutez-en un. Veillez à nommer le sous-réseau de passerelle *GatewaySubnet*.
5. Récupérez le sous-réseau utilisé pour la passerelle en exécutant la commande suivante. À cette étape, nous définissons également une variable qui sera utilisée à l’étape suivante.
   
   **-Name** est le nom de votre réseau virtuel Resource Manager.<br>
   **-ResourceGroupName** est le groupe de ressources auquel le réseau virtuel est associé. Pour fonctionner correctement, le sous-réseau de passerelle doit être nommé *GatewaySubnet* .<br>

   ```azurepowershell-interactive
   $subnet = Get-AzVirtualNetworkSubnetConfig -Name GatewaySubnet `
   -VirtualNetwork (Get-AzVirtualNetwork -Name RMVNet -ResourceGroupName RG1)
   ``` 

6. Créez la configuration de l’adressage IP de la passerelle. La configuration de la passerelle définit le sous-réseau et l’adresse IP publique à utiliser. Utilisez l’exemple suivant pour créer la configuration de votre passerelle.

   Dans cette étape, la propriété ID du sous-réseau et des objets d’adresse IP doit être transmise aux paramètres **-SubnetId** et **-PublicIpAddressId**. Vous ne pouvez pas utiliser une chaîne simple. Ces variables sont définies à l’étape de demande d’une adresse IP publique et à l’étape de récupération du sous-réseau.

   ```azurepowershell-interactive
   $gwipconfig = New-AzVirtualNetworkGatewayIpConfig `
   -Name gwipconfig -SubnetId $subnet.id `
   -PublicIpAddressId $ipaddress.id
   ```
7. Créez la passerelle de réseau virtuel Resource Manager en exécutant la commande suivante. `-VpnType` doit être défini sur *RouteBased*. La création de la passerelle peut prendre 45 minutes ou davantage.

   ```azurepowershell-interactive
   New-AzVirtualNetworkGateway -Name RMGateway -ResourceGroupName RG1 `
   -Location "EastUS" -GatewaySKU Standard -GatewayType Vpn `
   -IpConfigurations $gwipconfig `
   -EnableBgp $false -VpnType RouteBased
   ```
8. Copiez l’adresse IP publique une fois que la passerelle VPN a été créée. Elle vous servira lors de la configuration des paramètres de réseau local pour votre réseau virtuel classique. Vous pouvez utiliser l’applet de commande suivante pour récupérer l’adresse IP publique. L’adresse IP publique est répertoriée dans la réponse en tant que *IpAddress*.

   ```azurepowershell-interactive
   Get-AzPublicIpAddress -Name gwpip -ResourceGroupName RG1
   ```

## <a name="section-3---modify-the-classic-vnet-local-site-settings"></a><a name="localsite"></a>Section 3 - Modifier les paramètres de site local du réseau virtuel classique

Dans cette section, vous travaillez avec le réseau virtuel classique. Vous remplacez l’adresse IP avec espace réservé que vous avez utilisée lorsque vous avez spécifié les paramètres du site local qui vous permettront de vous connecter à la passerelle de réseau virtuel Resource Manager. Comme vous travaillez avec le réseau virtuel classique, utilisez le PowerShell installé sur votre ordinateur au lieu d’Azure Cloud Shell TryIt.

1. Exportez le fichier de configuration réseau.

   ```azurepowershell
   Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml
   ```
2. À l’aide d’un éditeur de texte, modifiez la valeur de l’élément VPNGatewayAddress. Remplacez l’adresse IP avec espace réservé par l’adresse IP publique de la passerelle Resource Manager, puis enregistrez les modifications.

   ```
   <VPNGatewayAddress>13.68.210.16</VPNGatewayAddress>
   ```
3. Importez le fichier de configuration réseau modifié dans Azure.

   ```azurepowershell
   Set-AzureVNetConfig -ConfigurationPath C:\AzureNet\NetworkConfig.xml
   ```

## <a name="section-4---create-a-connection-between-the-gateways"></a><a name="connect"></a>Section 4 - Créer une connexion entre les passerelles
La création d’une connexion entre les passerelles nécessite PowerShell. Vous devrez peut-être ajouter votre compte Azure pour utiliser la version classique des cmdlets PowerShell. Pour ce faire, utilisez **Add-AzureAccount**.

1. Dans la console PowerShell, définissez votre clé partagée. Avant d’exécuter les applets de commande, recherchez dans le fichier de configuration réseau que vous avez téléchargé les noms exacts qu’Azure s’attend à voir. Lorsque vous spécifiez le nom d’un réseau virtuel qui contient des espaces, encadrez la valeur avec des guillemets.<br><br>Dans cet exemple, **-VNetName** est le nom du réseau virtuel classique et **-LocalNetworkSiteName** est le nom que vous avez spécifié pour le site du réseau local. **-SharedKey** est une valeur que vous pouvez générer et spécifier. Dans l’exemple, nous avons utilisé « abc123 », mais vous pouvez générer et utiliser quelque chose de plus complexe. L’important, c’est que la valeur que vous spécifiez ici doit être identique à celle spécifiée à l’étape suivante lors de la création de votre connexion. La réponse doit afficher **État : Réussi**.

   ```azurepowershell
   Set-AzureVNetGatewayKey -VNetName ClassicVNet `
   -LocalNetworkSiteName RMVNetLocal -SharedKey abc123
   ```
2. Créez la connexion VPN en exécutant les commandes suivantes :
   
   Définissez les variables.

   ```azurepowershell-interactive
   $vnet01gateway = Get-AzLocalNetworkGateway -Name ClassicVNetLocal -ResourceGroupName RG1
   $vnet02gateway = Get-AzVirtualNetworkGateway -Name RMGateway -ResourceGroupName RG1
   ```
   
   Créez la connexion. Notez que le type de connexion **-ConnectionType** est IPsec et pas Vnet2Vnet.

   ```azurepowershell-interactive
   New-AzVirtualNetworkGatewayConnection -Name RM-Classic -ResourceGroupName RG1 `
   -Location "East US" -VirtualNetworkGateway1 `
   $vnet02gateway -LocalNetworkGateway2 `
   $vnet01gateway -ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'
   ```

## <a name="section-5---verify-your-connections"></a><a name="verify"></a>Section 5 - Vérifier vos connexions

### <a name="to-verify-the-connection-from-your-classic-vnet-to-your-resource-manager-vnet"></a>Pour vérifier la connexion de votre réseau virtuel classique à votre réseau virtuel Resource Manager

#### <a name="powershell"></a>PowerShell

[!INCLUDE [vpn-gateway-verify-connection-ps-classic](../../includes/vpn-gateway-verify-connection-ps-classic-include.md)]

#### <a name="azure-portal"></a>Portail Azure

[!INCLUDE [vpn-gateway-verify-connection-azureportal-classic](../../includes/vpn-gateway-verify-connection-azureportal-classic-include.md)]


### <a name="to-verify-the-connection-from-your-resource-manager-vnet-to-your-classic-vnet"></a>Pour vérifier la connexion de votre réseau virtuel Resource Manager à votre réseau virtuel classique

#### <a name="powershell"></a>PowerShell

[!INCLUDE [vpn-gateway-verify-ps-rm](../../includes/vpn-gateway-verify-connection-ps-rm-include.md)]

#### <a name="azure-portal"></a>Portail Azure

[!INCLUDE [vpn-gateway-verify-connection-portal-rm](../../includes/vpn-gateway-verify-connection-portal-rm-include.md)]

## <a name="vnet-to-vnet-faq"></a><a name="faq"></a>Forum Aux Questions sur l’interconnexion de réseaux virtuels

[!INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-faq-vnet-vnet-include.md)]
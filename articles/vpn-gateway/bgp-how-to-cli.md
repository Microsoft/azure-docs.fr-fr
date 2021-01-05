---
title: 'Configurer le protocole BGP sur des passerelles VPN Azure : Interface CLI'
description: Cet article vous guide dans la configuration de BGP avec une passerelle VPN Azure à l’aide d’Azure Resource Manager et de CLI.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: yushwang
ms.openlocfilehash: a69ce0592b79be0868dd7c15ac054910eee75fc7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89393596"
---
# <a name="how-to-configure-bgp-on-an-azure-vpn-gateway-by-using-cli"></a>Configurer BGP sur une passerelle VPN Azure à l’aide de CLI

Cet article vous permet d’activer BGP sur une connexion VPN de site à site (S2S) intersite et une connexion de réseau virtuel à réseau virtuel en utilisant le modèle de déploiement Azure Resource Manager et Azure CLI.

## <a name="about-bgp"></a>À propos du protocole BGP

BGP est le protocole de routage standard couramment utilisé sur Internet pour échanger des informations de routage et d’accessibilité entre plusieurs réseaux. BGP permet aux passerelles VPN et à vos périphériques VPN locaux, appelés homologues ou voisins BGP d’échanger des itinéraires. Les itinéraires informent les deux passerelles sur la disponibilité et l’accessibilité des préfixes pour le passage à travers les passerelles ou les routeurs impliqués. Le protocole BGP assure également le routage de transit entre plusieurs réseaux en propageant les itinéraires qu’une passerelle BGP obtient d’un homologue BGP à tous les autres homologues BGP.

Pour plus d’informations sur les avantages de BGP et les exigences techniques et considérations d’utilisation de BGP, consultez [Vue d’ensemble du protocole BGP avec les passerelles VPN Azure](vpn-gateway-bgp-overview.md).

Cet article vous aidera à exécuter les tâches suivantes :

* [Activer BGP pour la passerelle VPN](#enablebgp) (requis)

  Vous pouvez ensuite exécuter les sections suivantes, ou les deux :

* [Établir une connexion intersite avec BGP](#crossprembgp)
* [Établir une connexion de réseau virtuel à réseau virtuel avec BGP](#v2vbgp)

Chaque partie de ces trois parties constitue un bloc de base pour activer BGP dans votre connectivité réseau. Si vous terminez ces trois parties, vous générez la topologie comme sur le diagramme suivant :

![Topologie BGP](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crosspremv2v.png)

Vous pouvez les combiner pour créer un réseau de transit plus complexe, à plusieurs tronçons, qui répond à vos besoins.

## <a name="enable-bgp-for-your-vpn-gateway"></a><a name ="enablebgp"></a>Activer BGP pour la passerelle VPN

Cette partie est obligatoire avant d’effectuer des étapes des autres sections de configuration. Les étapes suivantes configurent les paramètres BGP de la passerelle VPN Azure comme indiqué dans le diagramme ci-dessous :

![Passerelle BGP](./media/vpn-gateway-bgp-resource-manager-ps/bgp-gateway.png)

### <a name="before-you-begin"></a>Avant de commencer

Installez la dernière version des commandes CLI (version 2.0 ou ultérieure). Pour plus d’informations sur l’installation des commandes CLI, consultez [Installer l’interface de ligne de commande Microsoft Azure](/cli/azure/install-azure-cli) et [Prise en main d’Azure CLI](/cli/azure/get-started-with-azure-cli).

### <a name="step-1-create-and-configure-testvnet1"></a>Étape 1 : Créer et configurer TestVNet1

#### <a name="1-connect-to-your-subscription"></a><a name="Login"></a>1. Connexion à votre abonnement

[!INCLUDE [CLI login](../../includes/vpn-gateway-cli-login-include.md)]

#### <a name="2-create-a-resource-group"></a>2. Créer un groupe de ressources

L’exemple suivant crée un groupe de ressources nommé TestRG1 à l’emplacement « eastus ». Si vous disposez déjà d’un groupe de ressources dans la région où vous souhaitez créer votre réseau virtuel, vous pouvez l’utiliser à la place de l’exemple donné.

```azurecli
az group create --name TestBGPRG1 --location eastus
```

#### <a name="3-create-testvnet1"></a>3. Créer TestVNet1

L’exemple suivant permet de créer un réseau virtuel nommé TestVNet1 et trois sous-réseaux : GatewaySubnet, FrontEnd et BackEnd. Lorsque vous remplacez les valeurs, pensez à toujours nommer votre sous-réseau de passerelle GatewaySubnet. Si vous le nommez autrement, la création de votre passerelle échoue.

La première commande crée l’espace d’adressage de serveur frontal et le sous-réseau frontal. La deuxième commande crée un espace d’adressage supplémentaire pour le sous-réseau principal. Les troisième et quatrième commandes créent le sous-réseau principal et GatewaySubnet.

```azurecli
az network vnet create -n TestVNet1 -g TestBGPRG1 --address-prefix 10.11.0.0/16 -l eastus --subnet-name FrontEnd --subnet-prefix 10.11.0.0/24 
 
az network vnet update -n TestVNet1 --address-prefixes 10.11.0.0/16 10.12.0.0/16 -g TestBGPRG1 
 
az network vnet subnet create --vnet-name TestVNet1 -n BackEnd -g TestBGPRG1 --address-prefix 10.12.0.0/24 
 
az network vnet subnet create --vnet-name TestVNet1 -n GatewaySubnet -g TestBGPRG1 --address-prefix 10.12.255.0/27 
```

### <a name="step-2-create-the-vpn-gateway-for-testvnet1-with-bgp-parameters"></a>Étape 2 : créer la passerelle VPN de TestVNet1 avec les paramètres BGP

#### <a name="1-create-the-public-ip-address"></a>1. Créer une adresse IP publique

Demandez une adresse IP publique L’adresse IP publique sera affectée à la passerelle VPN que vous créez pour votre réseau virtuel.

```azurecli
az network public-ip create -n GWPubIP -g TestBGPRG1 --allocation-method Dynamic 
```

#### <a name="2-create-the-vpn-gateway-with-the-as-number"></a>2. Créer la passerelle VPN avec le numéro AS

Créez la passerelle de réseau virtuel pour TestVNet1. BGP requiert l’utilisation d’une passerelle VPN basée sur un itinéraire. Vous avez également besoin du paramètre supplémentaire `-Asn` pour définir le numéro de système autonome (NSA) de TestVNet1. La création d’une passerelle peut prendre un certain temps (45 minutes ou plus). 

Si vous exécutez cette commande à l’aide du paramètre `--no-wait`, vous ne voyez aucun commentaire ni sortie. Le paramètre `--no-wait` permet à la passerelle d’être créée à l’arrière-plan. Cela ne signifie pas que la passerelle VPN est immédiatement créée.

```azurecli
az network vnet-gateway create -n VNet1GW -l eastus --public-ip-address GWPubIP -g TestBGPRG1 --vnet TestVNet1 --gateway-type Vpn --sku HighPerformance --vpn-type RouteBased --asn 65010 --no-wait
```

#### <a name="3-obtain-the-azure-bgp-peer-ip-address"></a>3. Obtenir l’adresse IP de l’homologue BGP Azure

Une fois la passerelle créée, vous devez obtenir l’adresse IP de l’homologue BGP sur la passerelle VPN Azure. Cette adresse est nécessaire pour configurer la passerelle VPN comme un homologue BGP pour vos périphériques VPN locaux.

Exécutez la commande suivante et vérifiez la section `bgpSettings` en haut de la sortie :

```azurecli
az network vnet-gateway list -g TestBGPRG1 
 
  
"bgpSettings": { 
      "asn": 65010, 
      "bgpPeeringAddress": "10.12.255.30", 
      "peerWeight": 0 
    }
```

Une fois la passerelle créée, vous pouvez l’utiliser pour établir une connexion intersite ou de réseau virtuel à réseau virtuel avec BGP.

## <a name="establish-a-cross-premises-connection-with-bgp"></a><a name ="crossprembgp"></a>Établir une connexion intersite avec BGP

Pour établir une connexion intersite, vous devez créer une passerelle de réseau local pour représenter votre périphérique VPN local. Ensuite, vous connectez la passerelle VPN Azure à la passerelle de réseau local. Alors que ces étapes sont semblables à la création d’autres connexions, ils incluent les propriétés supplémentaires nécessaires pour spécifier les paramètres de configuration BGP.

![BGP pour une connexion intersite](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crossprem.png)


### <a name="step-1-create-and-configure-the-local-network-gateway"></a>Étape 1 : créer et configurer la passerelle de réseau local

Cet exercice continue à générer la configuration représentée dans le diagramme. Veillez à remplacer les valeurs par celles que vous souhaitez utiliser pour votre configuration. Lorsque vous travaillez avec des passerelles de réseau local, gardez à l’esprit ce qui suit :

* La passerelle de réseau local peut se trouver dans les mêmes emplacement et groupe de ressources que la passerelle VPN, ou son emplacement et son groupe de ressources peuvent être différents. Cet exemple montre les passerelles dans différents groupes de ressources situés dans différents emplacements.
* Le préfixe minimum à déclarer pour la passerelle de réseau local est l’adresse IP hôte de votre homologue BGP sur votre périphérique VPN. Dans ce cas, c’est un préfixe /32 de 10.51.255.254/32.
* À titre de rappel, vous devez utiliser différents numéros de système autonome BGP entre vos réseaux locaux et le réseau virtuel Azure. S’ils sont identiques, vous devez modifier l’ASN de votre réseau si votre périphérique VPN local utilise déjà cet ASN pour se connecter à d’autres voisins BGP.

Avant de continuer, assurez-vous d’avoir terminé la section [Activer BGP pour la passerelle VPN](#enablebgp) de cet exercice et d’être toujours connecté à Subscription 1. Notez que dans cet exemple, vous créez un nouveau groupe de ressources. Remarquez également les deux paramètres supplémentaires pour la passerelle de réseau local : `Asn` et `BgpPeerAddress`.

```azurecli
az group create -n TestBGPRG5 -l eastus2 
 
az network local-gateway create --gateway-ip-address 23.99.221.164 -n Site5 -g TestBGPRG5 --local-address-prefixes 10.51.255.254/32 --asn 65050 --bgp-peering-address 10.51.255.254
```

### <a name="step-2-connect-the-vnet-gateway-and-local-network-gateway"></a>Étape 2 : connecter la passerelle de réseau virtuel et la passerelle de réseau local

Dans cette étape, vous allez créer la connexion entre TestVNet1 et Site5. Vous devez spécifier le paramètre `--enable-bgp` pour activer BGP sur cette connexion. 

Dans cet exemple, la passerelle de réseau virtuel et la passerelle de réseau local se trouvent dans des groupes de ressources différents. Lorsque les passerelles se trouvent dans différents groupes de ressources, vous devez spécifier l’ID de ressource complet des deux passerelles pour configurer une connexion entre les réseaux virtuels.

#### <a name="1-get-the-resource-id-of-vnet1gw"></a>1. Obtenir l’ID de ressource de VNet1GW

Obtenez l’ID de ressource de VNet1GW à partir de la sortie de la commande suivante :

```azurecli
az network vnet-gateway show -n VNet1GW -g TestBGPRG1
```

Dans la sortie, recherchez la ligne `"id":`. Vous avez besoin des valeurs entre guillemets pour créer la connexion dans la section suivante.

Exemple de sortie :

```
{ 
  "activeActive": false, 
  "bgpSettings": { 
    "asn": 65010, 
    "bgpPeeringAddress": "10.12.255.30", 
    "peerWeight": 0 
  }, 
  "enableBgp": true, 
  "etag": "W/\"<your etag number>\"", 
  "gatewayDefaultSite": null, 
  "gatewayType": "Vpn", 
  "id": "/subscriptions/<subscription ID>/resourceGroups/TestBGPRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW",
```

Copiez ces valeurs après `"id":` dans un éditeur de texte, tel que Bloc-notes, pour pouvoir les coller facilement lors de la création de la connexion. 

```
"id": "/subscriptions/<subscription ID>/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW"
```

#### <a name="2-get-the-resource-id-of-site5"></a>2. Obtenir l’ID de ressource de Site5

Obtenez l’ID de ressource de Site5 à partir de la sortie de la commande suivante :

```azurecli
az network local-gateway show -n Site5 -g TestBGPRG5
```

#### <a name="3-create-the-testvnet1-to-site5-connection"></a>3. Créer la connexion entre TestVNet1 et Site5

Dans cette étape, vous allez créer la connexion entre TestVNet1 et Site5. Comme nous l’avons vu, il est possible d’avoir des connexions BGP et non BGP sur la même passerelle VPN Azure. À moins que BGP ne soit activé dans la propriété de connexion, Azure n’active pas BGP sur cette connexion même si les paramètres BGP sont déjà configurés sur les deux passerelles. Remplacez les ID d’abonnement par les vôtres.

```azurecli
az network vpn-connection create -n VNet1ToSite5 -g TestBGPRG1 --vnet-gateway1 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW --enable-bgp -l eastus --shared-key "abc123" --local-gateway2 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG5/providers/Microsoft.Network/localNetworkGateways/Site5 --no-wait
```

L’exemple ci-dessous répertorie les paramètres que vous devez saisir dans la section de configuration de BGP de votre périphérique VPN local pour cet exercice :

```
Site5 ASN            : 65050
Site5 BGP IP         : 10.52.255.254
Prefixes to announce : (for example) 10.51.0.0/16 and 10.52.0.0/16
Azure VNet ASN       : 65010
Azure VNet BGP IP    : 10.12.255.30
Static route         : Add a route for 10.12.255.30/32, with nexthop being the VPN tunnel interface on your device
eBGP Multihop        : Ensure the "multihop" option for eBGP is enabled on your device if needed
```

Après quelques minutes, la connexion doit être établie. La session de peering BGP démarre une fois la connexion IPsec établie.

## <a name="establish-a-vnet-to-vnet-connection-with-bgp"></a><a name ="v2vbgp"></a>Établir une connexion de réseau virtuel à réseau virtuel avec BGP

Cette section ajoute une connexion de réseau virtuel à réseau virtuel avec le protocole BGP, comme illustré dans le diagramme ci-dessous : 

![BGP pour une connexion de réseau virtuel à réseau virtuel](./media/vpn-gateway-bgp-resource-manager-ps/bgp-vnet2vnet.png)

Les instructions suivantes sont la suite des étapes des précédentes sections. Pour créer et configurer TestVNet1 et la passerelle VPN avec le protocole BGP, vous devez terminer la section [Activer BGP pour la passerelle VPN](#enablebgp).

### <a name="step-1-create-testvnet2-and-the-vpn-gateway"></a>Étape 1 : créer TestVNet2 et la passerelle VPN

Il est important de s’assurer que l’espace d’adressage IP du nouveau réseau virtuel, TestVNet2, n’empiète sur aucune de vos plages de réseau virtuel.

Dans cet exemple, les réseaux virtuels appartiennent au même abonnement. Vous pouvez configurer les connexions de réseau virtuel à réseau virtuel entre différents abonnements. Pour en savoir plus, consultez [Configurer une connexion de réseau virtuel à réseau virtuel](vpn-gateway-howto-vnet-vnet-cli.md). Assurez-vous d’ajouter `-EnableBgp $True` lors de la création des connexions pour activer BGP.

#### <a name="1-create-a-new-resource-group"></a>1. Création d’un groupe de ressources

```azurecli
az group create -n TestBGPRG2 -l westus
```

#### <a name="2-create-testvnet2-in-the-new-resource-group"></a>2. Créer TestVNet2 dans le nouveau groupe de ressources

La première commande crée l’espace d’adressage de serveur frontal et le sous-réseau frontal. La deuxième commande crée un espace d’adressage supplémentaire pour le sous-réseau principal. Les troisième et quatrième commandes créent le sous-réseau principal et GatewaySubnet.

```azurecli
az network vnet create -n TestVNet2 -g TestBGPRG2 --address-prefix 10.21.0.0/16 -l westus --subnet-name FrontEnd --subnet-prefix 10.21.0.0/24 
 
az network vnet update -n TestVNet2 --address-prefixes 10.21.0.0/16 10.22.0.0/16 -g TestBGPRG2 
 
az network vnet subnet create --vnet-name TestVNet2 -n BackEnd -g TestBGPRG2 --address-prefix 10.22.0.0/24 
 
az network vnet subnet create --vnet-name TestVNet2 -n GatewaySubnet -g TestBGPRG2 --address-prefix 10.22.255.0/27
```

#### <a name="3-create-the-public-ip-address"></a>3. Créer une adresse IP publique

Demandez une adresse IP publique L’adresse IP publique sera affectée à la passerelle VPN que vous créez pour votre réseau virtuel.

```azurecli
az network public-ip create -n GWPubIP2 -g TestBGPRG2 --allocation-method Dynamic
```

#### <a name="4-create-the-vpn-gateway-with-the-as-number"></a>4. Créer la passerelle VPN avec le numéro AS

Créez la passerelle de réseau virtuel pour TestVNet2. Vous devez substituer la valeur par défaut de l’ASN sur vos passerelles VPN Azure. Les numéros de système autonome des réseaux virtuels connectés doivent être différents pour activer BGP et le routage de transit.
 
```azurecli
az network vnet-gateway create -n VNet2GW -l westus --public-ip-address GWPubIP2 -g TestBGPRG2 --vnet TestVNet2 --gateway-type Vpn --sku Standard --vpn-type RouteBased --asn 65020 --no-wait
```

### <a name="step-2-connect-the-testvnet1-and-testvnet2-gateways"></a>Étape 2 : connecter les passerelles TestVNet1 et TestVNet2

Dans cette étape, vous allez créer la connexion entre TestVNet1 et Site5. Pour activer BGP sur cette connexion, vous devez spécifier le paramètre `--enable-bgp`.

Dans l’exemple suivant, la passerelle de réseau virtuel et la passerelle de réseau local se trouvent dans des groupes de ressources différents. Lorsque les passerelles se trouvent dans différents groupes de ressources, vous devez spécifier l’ID de ressource complet des deux passerelles pour configurer une connexion entre les réseaux virtuels. 

#### <a name="1-get-the-resource-id-of-vnet1gw"></a>1. Obtenir l’ID de ressource de VNet1GW 

Obtenez l’ID de ressource de VNet1GW à partir de la sortie de la commande suivante :

```azurecli
az network vnet-gateway show -n VNet1GW -g TestBGPRG1
```

#### <a name="2-get-the-resource-id-of-vnet2gw"></a>2. Obtenir l’ID de ressource de VNet2GW

Obtenez l’ID de ressource de VNet2GW à partir de la sortie de la commande suivante :

```azurecli
az network vnet-gateway show -n VNet2GW -g TestBGPRG2
```

#### <a name="3-create-the-connections"></a>3. Créer les connexions

Créez la connexion TestVNet1 à TestVNet2 et de TestVNet2 à TestVNet1. Remplacez les ID d’abonnement par les vôtres.

```azurecli
az network vpn-connection create -n VNet1ToVNet2 -g TestBGPRG1 --vnet-gateway1 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW --enable-bgp -l eastus --shared-key "efg456" --vnet-gateway2 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG2/providers/Microsoft.Network/virtualNetworkGateways/VNet2GW
```

```azurecli
az network vpn-connection create -n VNet2ToVNet1 -g TestBGPRG2 --vnet-gateway1 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG2/providers/Microsoft.Network/virtualNetworkGateways/VNet2GW --enable-bgp -l westus --shared-key "efg456" --vnet-gateway2 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW
```

> [!IMPORTANT]
> Activez BGP pour *les deux* connexions.
> 
> 

Une fois ces étapes effectuées, la connexion est établie au bout de quelques minutes. La session de peering BGP démarre une fois la connexion de réseau virtuel à réseau virtuel établie.

## <a name="next-steps"></a>Étapes suivantes

Une fois la connexion achevée, vous pouvez ajouter des machines virtuelles à vos réseaux virtuels. Pour connaître les différentes étapes, consultez [Créer une machine virtuelle](../virtual-machines/windows/quick-create-portal.md).

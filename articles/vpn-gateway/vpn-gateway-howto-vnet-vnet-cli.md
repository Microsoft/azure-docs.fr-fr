---
title: 'Connecter un réseau virtuel à un réseau virtuel avec une connexion de réseau virtuel à réseau virtuel : Azure CLI'
description: Connectez des réseaux virtuels avec une connexion de réseau virtuel à réseau virtuel et Azure CLI.
services: vpn-gateway
titleSuffix: Azure VPN Gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: cherylmc
ms.custom: devx-track-azurecli
ms.openlocfilehash: b9502f3fbd50aad756e15daa4db1badda2abf9ab
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94660064"
---
# <a name="configure-a-vnet-to-vnet-vpn-gateway-connection-using-azure-cli"></a>Configurer une connexion de passerelle VPN de réseau virtuel à réseau virtuel à l’aide d’Azure CLI

Cet article vous explique comment connecter des réseaux virtuels avec une connexion de réseau virtuel à réseau virtuel. Les réseaux virtuels peuvent être situés dans des régions identiques ou différentes et appartenir à des abonnements identiques ou différents. Lors de la connexion de réseaux virtuels provenant de différents abonnements, les abonnements ne sont pas tenus d’être associés au même locataire Active Directory.

Les étapes mentionnées dans cet article s’appliquent au modèle de déploiement Resource Manager et utilisent Azure CLI. Vous pouvez également créer cette configuration à l’aide d’un autre outil ou modèle de déploiement en sélectionnant une option différente dans la liste suivante :

> [!div class="op_single_selector"]
> * [Azure portal](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md)
> * [Azure CLI](vpn-gateway-howto-vnet-vnet-cli.md)
> * [Portail Azure (classique)](vpn-gateway-howto-vnet-vnet-portal-classic.md)
> * [Connexions entre différents modèles de déploiement - Portail Azure](vpn-gateway-connect-different-deployment-models-portal.md)
> * [Connexions entre différents modèles de déploiement - PowerShell](vpn-gateway-connect-different-deployment-models-powershell.md)
>
>

## <a name="about-connecting-vnets"></a><a name="about"></a>À propos de la connexion de réseaux virtuels

Il existe plusieurs manières de se connecter à des réseaux virtuels. Les sections ci-dessous décrivent les différentes manières de se connecter à des réseaux virtuels.

### <a name="vnet-to-vnet"></a>Connexion entre deux réseaux virtuels

La configuration d’une connexion de réseau virtuel à réseau virtuel est un bon moyen de se connecter facilement à des réseaux virtuels. La connexion entre deux réseaux virtuels est semblable à la création d’une connexion IPsec de site à site dans un emplacement local. Ces deux types de connectivité font appel à une passerelle VPN pour offrir un tunnel sécurisé utilisant Ipsec/IKE et communiquent de la même façon. La différence entre ces types de connexion se trouve dans la configuration de la passerelle réseau locale. Lorsque vous créez une connexion de réseau virtuel à réseau virtuel, vous ne voyez pas l’espace d’adressage de la passerelle réseau locale. L’espace est créé et rempli automatiquement. Si vous mettez à jour l’espace d’adressage pour un réseau virtuel, l’autre réseau virtuel achemine automatiquement le trafic vers le nouvel espace d’adressage. La création d’une connexion de réseau virtuel à réseau virtuel est généralement plus rapide et plus simple que la création d’une connexion de site à site entre des réseaux virtuels.

### <a name="connecting-vnets-using-site-to-site-ipsec-steps"></a>Connexion de réseaux virtuels à l’aide des étapes de site à site (IPsec)

Si vous travaillez avec une configuration réseau complexe, vous pouvez connecter vos réseaux virtuels à l’aide des étapes [site à site](vpn-gateway-howto-site-to-site-resource-manager-cli.md), à la place des étapes de réseau virtuel à réseau virtuel. Lorsque vous utilisez les étapes de site à site, vous créez et configurez manuellement les passerelles réseau locales. La passerelle de réseau local pour chaque réseau virtuel traite l’autre réseau virtuel comme un site local. Ainsi, vous pourrez spécifier un espace d’adressage supplémentaire pour la passerelle réseau locale afin d’acheminer le trafic. Si l’espace d’adressage pour un réseau virtuel est modifié, vous devez mettre à jour manuellement la passerelle réseau local correspondante pour le refléter. Elle n’est pas automatiquement mise à jour.

### <a name="vnet-peering"></a>Peering de réseaux virtuels

Vous envisagerez probablement de connecter vos réseaux virtuels à l’aide de VNet Peering. VNet Peering n’utilise pas une passerelle VPN et possède d’autres contraintes. En outre, la [tarification de VNet Peering](https://azure.microsoft.com/pricing/details/virtual-network) est différente de la [tarification de la passerelle VPN de réseau virtuel à réseau virtuel](https://azure.microsoft.com/pricing/details/vpn-gateway). Pour plus d’informations, consultez l’article [Peering de réseaux virtuels](../virtual-network/virtual-network-peering-overview.md).

## <a name="why-create-a-vnet-to-vnet-connection"></a><a name="why"></a>Pourquoi créer une connexion de réseau virtuel à réseau virtuel ?

Vous pouvez décider de connecter des réseaux virtuels à l’aide d’une connexion de réseau virtuel à réseau virtuel pour les raisons suivantes :

* **Géo-redondance et présence géographique dans plusieurs régions**

  * Vous pouvez configurer la géo-réplication ou la synchronisation avec une connectivité sécurisée sans passer par les points de terminaison accessibles sur Internet.
  * Avec Traffic Manager et l’équilibreur de charge Azure, vous pouvez configurer une charge de travail hautement disponible avec la géo-redondance dans plusieurs régions Azure. Vous pouvez par exemple configurer SQL Always On avec des groupes de disponibilité répartis dans différentes régions Azure.
* **Applications multiniveaux régionales avec une limite d’isolement ou administrative**

  * Dans la même région, vous pouvez configurer des applications multiniveaux avec plusieurs réseaux virtuels interconnectés pour des besoins d’isolement ou d’administration.

Vous pouvez combiner une communication de réseau virtuel à réseau virtuel avec des configurations multisites. Vous établissez ainsi des topologies réseau qui combinent une connectivité entre différents locaux et une connectivité entre différents réseaux virtuels.

## <a name="which-vnet-to-vnet-steps-should-i-use"></a><a name="steps"></a>Quelles étapes de réseau virtuel à réseau virtuel dois-je utiliser ?

Cet article inclut deux ensembles distincts d’étapes de connexion de réseau virtuel à réseau virtuel. L’un des ensembles est destiné aux [réseaux virtuels qui se trouvent dans le même abonnement](#samesub), et l’autre aux [réseaux virtuels qui se trouvent dans des abonnements différents](#difsub). 

Pour cet exercice, vous pouvez combiner des configurations ou choisir simplement celle que vous voulez utiliser. Toutes les configurations utilisent la connexion de réseau virtuel à réseau virtuel. Le trafic circule entre les réseaux virtuels connectés directement entre eux. Dans cet exercice, le trafic du réseau TestVNet4 n’est pas acheminé jusqu’au réseau TestVNet5.

* [Réseaux virtuels situés dans le même abonnement :](#samesub) Les étapes à suivre pour cette configuration utilisent les réseaux virtuels TestVNet1 et TestVNet4.

  ![Diagramme représentant les étapes de réseau virtuel à réseau virtuel pour les réseaux virtuels associés au même abonnement.](./media/vpn-gateway-howto-vnet-vnet-cli/v2vrmps.png)

* [Réseaux virtuels situés dans des abonnements différents :](#difsub) Les étapes à suivre pour cette configuration utilisent les réseaux virtuels TestVNet1 et TestVNet5.

  ![Diagramme v2v](./media/vpn-gateway-howto-vnet-vnet-cli/v2vdiffsub.png)


## <a name="connect-vnets-that-are-in-the-same-subscription"></a><a name="samesub"></a>Connecter des réseaux virtuels situés dans le même abonnement

### <a name="before-you-begin"></a>Avant de commencer

Avant de commencer, installez la dernière version des commandes CLI (version 2.0 ou ultérieure). Pour plus d’informations sur l’installation des commandes CLI, consultez l’article [Installer l’interface de ligne de commande Azure](/cli/azure/install-azure-cli).

### <a name="plan-your-ip-address-ranges"></a><a name="Plan"></a>Panifier vos plages d’adresses IP

Dans les étapes suivantes, vous allez créer deux réseaux virtuels avec leurs sous-réseaux de passerelle respectifs et leur configuration. Vous allez ensuite créer une connexion VPN entre les deux réseaux virtuels. Il est important de planifier les plages d’adresses IP pour votre configuration réseau. N’oubliez pas que vous devez vous assurer qu’aucune plage de réseaux virtuels ou de réseaux locaux ne se chevauche. Dans ces exemples, nous n’incluons pas de serveur DNS. Si vous souhaitez une résolution de noms pour vos réseaux virtuels, consultez la page [Résolution de noms](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md).

Nous utilisons les valeurs suivantes dans les exemples :

**Valeurs pour TestVNet1 :**

* Nom du réseau virtuel : TestVNet1
* Groupe de ressources : TestRG1
* Localisation : USA Est
* TestVNet1 : 10.11.0.0/16 et 10.12.0.0/16
* Front-end : 10.11.0.0/24
* BackEnd : 10.12.0.0/24
* GatewaySubnet : 10.12.255.0/27
* GatewayName : VNet1GW
* Adresse IP publique : VNet1GWIP
* VPNType : RouteBased
* Connection(1to4) : VNet1toVNet4
* Connection(1to5) : VNet1toVNet5 (pour les réseaux virtuels se trouvant dans des abonnements différents)

**Valeurs pour TestVNet4 :**

* Nom du réseau virtuel : TestVNet4
* TestVNet2 : 10.41.0.0/16 et 10.42.0.0/16
* Front-end : 10.41.0.0/24
* BackEnd : 10.42.0.0/24
* GatewaySubnet : 10.42.255.0/27
* Groupe de ressources : TestRG4
* Localisation : USA Ouest
* GatewayName : VNet4GW
* Adresse IP publique : VNet4GWIP
* VPNType : RouteBased
* Connexion : VNet4toVNet1

### <a name="step-1---connect-to-your-subscription"></a><a name="Connect"></a>Étape 1 : connectez-vous à votre abonnement

[!INCLUDE [CLI login](../../includes/vpn-gateway-cli-login-numbers-include.md)]

### <a name="step-2---create-and-configure-testvnet1"></a><a name="TestVNet1"></a>Étape 2 : créez et configurez TestVNet1

1. Créez un groupe de ressources.

   ```azurecli
   az group create -n TestRG1  -l eastus
   ```
2. Créez TestVNet1 et les sous-réseaux de TestVNet1. L’exemple suivant permet de créer un réseau virtuel nommé TestVNet1 et un sous-réseau nommé FrontEnd.

   ```azurecli
   az network vnet create -n TestVNet1 -g TestRG1 --address-prefix 10.11.0.0/16 -l eastus --subnet-name FrontEnd --subnet-prefix 10.11.0.0/24
   ```
3. Créez un espace d’adressage supplémentaire pour le sous-réseau principal. Notez que dans cette étape, nous spécifions l’espace d’adressage créé précédemment et l’espace d’adressage supplémentaires que nous souhaitons ajouter. Cela vient du fait que la [mise à jour du réseau virtuel du réseau az](/cli/azure/network/vnet) écrase les paramètres précédents. Veillez à spécifier tous les préfixes d’adresse lors de l’utilisation de cette commande.

   ```azurecli
   az network vnet update -n TestVNet1 --address-prefixes 10.11.0.0/16 10.12.0.0/16 -g TestRG1
   ```
4. Créez le sous-réseau principal.
  
   ```azurecli
   az network vnet subnet create --vnet-name TestVNet1 -n BackEnd -g TestRG1 --address-prefix 10.12.0.0/24 
   ```
5. Créez le sous-réseau de passerelle. Notez que le sous-réseau de passerelle est nommé GatewaySubnet. Ce nom est obligatoire. Dans cet exemple, le sous-réseau de passerelle utilise /27. Bien qu’il soit possible de créer un sous-réseau de passerelle aussi petit que /29, nous vous recommandons de créer un sous-réseau plus vaste qui inclut un plus grand nombre d’adresses en sélectionnant au moins /28 ou /27. Cela permettra à un nombre suffisant d’adresses de s’adapter à de possibles configurations supplémentaires possibles que vous êtes susceptible de souhaiter par la suite.

   ```azurecli 
   az network vnet subnet create --vnet-name TestVNet1 -n GatewaySubnet -g TestRG1 --address-prefix 10.12.255.0/27
   ```
6. Demandez l’allocation d’une adresse IP publique à la passerelle que vous allez créer pour votre réseau virtuel. Notez que la valeur AllocationMethod est dynamique. Vous ne pouvez pas spécifier l’adresse IP que vous souhaitez utiliser. Elle est allouée à votre passerelle de façon dynamique.

   ```azurecli
   az network public-ip create -n VNet1GWIP -g TestRG1 --allocation-method Dynamic
   ```
7. Créez la passerelle de réseau virtuel pour TestVNet1. Les configurations de réseau virtuel à réseau virtuel requièrent un VPN de type RouteBased. Si vous exécutez cette commande à l’aide du paramètre « --no-wait », vous ne voyez aucun commentaire ni sortie. Le paramètre ’--no-wait’ permet à la passerelle d’être créée en arrière-plan. Cela ne signifie pas que la passerelle VPN termine la création immédiatement. La création d’une passerelle nécessite généralement au moins 45 minutes, selon la référence SKU de passerelle utilisée.

   ```azurecli
   az network vnet-gateway create -n VNet1GW -l eastus --public-ip-address VNet1GWIP -g TestRG1 --vnet TestVNet1 --gateway-type Vpn --sku VpnGw1 --vpn-type RouteBased --no-wait
   ```

### <a name="step-3---create-and-configure-testvnet4"></a><a name="TestVNet4"></a>Étape 3 : créez et configurez TestVNet4

1. Créez un groupe de ressources.

   ```azurecli
   az group create -n TestRG4  -l westus
   ```
2. Créez TestVNet4.

   ```azurecli
   az network vnet create -n TestVNet4 -g TestRG4 --address-prefix 10.41.0.0/16 -l westus --subnet-name FrontEnd --subnet-prefix 10.41.0.0/24
   ```

3. Créez des sous-réseaux supplémentaires pour TestVNet4.

   ```azurecli
   az network vnet update -n TestVNet4 --address-prefixes 10.41.0.0/16 10.42.0.0/16 -g TestRG4 
   az network vnet subnet create --vnet-name TestVNet4 -n BackEnd -g TestRG4 --address-prefix 10.42.0.0/24 
   ```
4. Créez le sous-réseau de passerelle.

   ```azurecli
   az network vnet subnet create --vnet-name TestVNet4 -n GatewaySubnet -g TestRG4 --address-prefix 10.42.255.0/27
   ```
5. Demandez une adresse IP publique.

   ```azurecli
   az network public-ip create -n VNet4GWIP -g TestRG4 --allocation-method Dynamic
   ```
6. Créez la passerelle de réseau virtuel TestVNet4.

   ```azurecli
   az network vnet-gateway create -n VNet4GW -l westus --public-ip-address VNet4GWIP -g TestRG4 --vnet TestVNet4 --gateway-type Vpn --sku VpnGw1 --vpn-type RouteBased --no-wait
   ```

### <a name="step-4---create-the-connections"></a><a name="createconnect"></a>Étape 4 : créez les connexions

Vous avez maintenant deux réseaux virtuels avec des passerelles VPN. L’étape suivante consiste à créer des connexions à la passerelle VPN entre les passerelles de réseau virtuel. Si vous avez utilisé les exemples ci-dessus, vos passerelles de réseau virtuel se trouvent dans différents groupes de ressources. Lorsque les passerelles se trouvent dans différents groupes de ressources, vous devez identifier et spécifier l’ID de ressource pour chaque passerelle lors de la connexion. Si vos réseaux virtuels sont dans le même groupe de ressources, vous pouvez utiliser le [deuxième ensemble d’instructions](#samerg) , dans la mesure où il n’est pas nécessaire de spécifier les ID de ressource.

### <a name="to-connect-vnets-that-reside-in-different-resource-groups"></a><a name="diffrg"></a>Connexion de réseaux virtuels qui se trouvent dans différents groupes de ressources

1. Obtenez l’ID de ressource de VNet1GW à partir de la sortie de la commande suivante :

   ```azurecli
   az network vnet-gateway show -n VNet1GW -g TestRG1
   ```

   Dans la sortie, recherchez la ligne « id: ». Les valeurs entre guillemets sont nécessaires pour créer la connexion dans la section suivante. Copiez ces valeurs dans un éditeur de texte, tel que Bloc-notes, afin de pouvoir les coller facilement lors de la création de la connexion.

   Exemple de sortie :

   ```
   "activeActive": false, 
   "bgpSettings": { 
    "asn": 65515, 
    "bgpPeeringAddress": "10.12.255.30", 
    "peerWeight": 0 
   }, 
   "enableBgp": false, 
   "etag": "W/\"ecb42bc5-c176-44e1-802f-b0ce2962ac04\"", 
   "gatewayDefaultSite": null, 
   "gatewayType": "Vpn", 
   "id": "/subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW", 
   "ipConfigurations":
   ```

   Copiez les valeurs situées après **« id » :** entre les guillemets.

   ```
   "id": "/subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW"
   ```

2. Obtenez l’ID de ressource de VNet4GW et copiez les valeurs dans un éditeur de texte.

   ```azurecli
   az network vnet-gateway show -n VNet4GW -g TestRG4
   ```

3. Créez la connexion de TestVNet1 à TestVNet4. Lors de cette étape, vous créez la connexion de TestVNet1 à TestVNet4. Une clé partagée est référencée dans les exemples. Vous pouvez utiliser vos propres valeurs pour cette clé partagée. Il est important que la clé partagée corresponde aux deux connexions. La création d’une connexion prend quelques instants.

   ```azurecli
   az network vpn-connection create -n VNet1ToVNet4 -g TestRG1 --vnet-gateway1 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW -l eastus --shared-key "aabbcc" --vnet-gateway2 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG4/providers/Microsoft.Network/virtualNetworkGateways/VNet4GW 
   ```
4. Créez la connexion de TestVNet4 à TestVNet1. Cette étape est similaire à celle présentée ci-dessus, sauf que vous créez la connexion de TestVNet4 à TestVNet1. Vérifiez que les clés partagées correspondent. L’établissement de la connexion prend quelques minutes.

   ```azurecli
   az network vpn-connection create -n VNet4ToVNet1 -g TestRG4 --vnet-gateway1 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG4/providers/Microsoft.Network/virtualNetworkGateways/VNet4GW -l westus --shared-key "aabbcc" --vnet-gateway2 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1G
   ```
5. Vérifiez vos connexions. Consultez [Vérifier votre connexion](#verify).

### <a name="to-connect-vnets-that-reside-in-the-same-resource-group"></a><a name="samerg"></a>Pour connecter des réseaux virtuels qui se trouvent dans le même groupe de ressources

1. Créez la connexion de TestVNet1 à TestVNet4. Lors de cette étape, vous créez la connexion de TestVNet1 à TestVNet4. Notez que les groupes de ressources sont les mêmes dans les exemples. Une clé partagée est également référencée dans les exemples. Vous pouvez utiliser vos propres valeurs pour la clé partagée, toutefois, cette dernière doit correspondre aux deux connexions. La création d’une connexion prend quelques instants.

   ```azurecli
   az network vpn-connection create -n VNet1ToVNet4 -g TestRG1 --vnet-gateway1 VNet1GW -l eastus --shared-key "eeffgg" --vnet-gateway2 VNet4GW
   ```
2. Créez la connexion de TestVNet4 à TestVNet1. Cette étape est similaire à celle présentée ci-dessus, sauf que vous créez la connexion de TestVNet4 à TestVNet1. Vérifiez que les clés partagées correspondent. L’établissement de la connexion prend quelques minutes.

   ```azurecli
   az network vpn-connection create -n VNet4ToVNet1 -g TestRG1 --vnet-gateway1 VNet4GW -l eastus --shared-key "eeffgg" --vnet-gateway2 VNet1GW
   ```
3. Vérifiez vos connexions. Consultez [Vérifier votre connexion](#verify).

## <a name="connect-vnets-that-are-in-different-subscriptions"></a><a name="difsub"></a>Connecter des réseaux virtuels situés dans différents abonnements

Dans ce scénario, vous connectez TestVNet1 et TestVNet5. Les réseaux virtuels se trouvent dans différents abonnements. Les abonnements ne sont pas tenus d’être associés au même locataire Active Directory. Cette configuration nécessite l’ajout d’une connexion supplémentaire entre réseaux virtuels pour connecter TestVNet1 à TestVNet5.

### <a name="step-5---create-and-configure-testvnet1"></a><a name="TestVNet1diff"></a>Étape 5 : créez et configurez TestVNet1

Ces instructions sont la suite des étapes décrites dans les sections précédentes. Vous devez terminer les étapes [1](#Connect) et [2](#TestVNet1) pour créer et configurer TestVNet1 et la passerelle VPN pour TestVNet1. Pour cette configuration, il n’est pas nécessaire de créer TestVNet4 à partir de la section précédente. Néanmoins, si vous la créez, elle n’entrera pas en conflit avec ces étapes. Une fois les étapes 1 et 2 effectuées, passez à l’étape 6 (ci-dessous).

### <a name="step-6---verify-the-ip-address-ranges"></a><a name="verifyranges"></a>Étape 6 : vérifiez les plages d’adresses IP

Lors de la création de connexions supplémentaires, il est important de s’assurer que l’espace d’adressage IP du nouveau réseau virtuel ne chevauche aucune de vos autres plages de réseaux virtuels ou de passerelles de réseau locales. Pour cet exercice, vous pouvez utiliser les valeurs suivantes pour TestVNet5 :

**Valeurs pour TestVNet5 :**

* Nom du réseau virtuel : TestVNet5
* Groupe de ressources : TestRG5
* Localisation : Japon Est
* TestVNet5 : 10.51.0.0/16 et 10.52.0.0/16
* Front-end : 10.51.0.0/24
* BackEnd : 10.52.0.0/24
* GatewaySubnet : 10.52.255.0.0/27
* GatewayName : VNet5GW
* Adresse IP publique : VNet5GWIP
* VPNType : RouteBased
* Connexion : VNet5toVNet1
* ConnectionType : VNet2VNet

### <a name="step-7---create-and-configure-testvnet5"></a><a name="TestVNet5"></a>Étape 7 : créez et configurez TestVNet5

Cette étape doit être effectuée dans le cadre du nouvel abonnement, Abonnement 5. Cette partie peut être effectuée par l’administrateur dans une organisation différente qui possède l’abonnement. Pour basculer entre les abonnements, utilisez `az account list --all` pour répertorier les abonnements disponibles pour votre compte, puis `az account set --subscription <subscriptionID>` pour basculer vers l’abonnement que vous souhaitez utiliser.

1. Assurez-vous d’être connecté à Abonnement 5, puis créez un groupe de ressources.

   ```azurecli
   az group create -n TestRG5  -l japaneast
   ```
2. Créez TestVNet5.

   ```azurecli
   az network vnet create -n TestVNet5 -g TestRG5 --address-prefix 10.51.0.0/16 -l japaneast --subnet-name FrontEnd --subnet-prefix 10.51.0.0/24
   ```

3. Ajoutez des sous-réseaux.

   ```azurecli
   az network vnet update -n TestVNet5 --address-prefixes 10.51.0.0/16 10.52.0.0/16 -g TestRG5
   az network vnet subnet create --vnet-name TestVNet5 -n BackEnd -g TestRG5 --address-prefix 10.52.0.0/24
   ```

4. Ajoutez le sous-réseau de passerelle.

   ```azurecli
   az network vnet subnet create --vnet-name TestVNet5 -n GatewaySubnet -g TestRG5 --address-prefix 10.52.255.0/27
   ```

5. Demandez une adresse IP publique

   ```azurecli
   az network public-ip create -n VNet5GWIP -g TestRG5 --allocation-method Dynamic
   ```
6. Créer la passerelle TestVNet5

   ```azurecli
   az network vnet-gateway create -n VNet5GW -l japaneast --public-ip-address VNet5GWIP -g TestRG5 --vnet TestVNet5 --gateway-type Vpn --sku VpnGw1 --vpn-type RouteBased --no-wait
   ```

### <a name="step-8---create-the-connections"></a><a name="connections5"></a>Étape 8 : créez les connexions

Étant donné que les passerelles se trouvent dans différents abonnements, cette étape est divisée en deux sessions CLI notées **[Abonnement 1]** et **[Abonnement 5]** . Pour basculer entre les abonnements, utilisez `az account list --all` pour répertorier les abonnements disponibles pour votre compte, puis `az account set --subscription <subscriptionID>` pour basculer vers l’abonnement que vous souhaitez utiliser.

1. **[Abonnement 1]** Ouvrez une session et connectez-vous à Abonnement 1. Exécutez la commande suivante pour obtenir le nom et l’ID de la passerelle à partir de la sortie :

   ```azurecli
   az network vnet-gateway show -n VNet1GW -g TestRG1
   ```

   Copiez la sortie pour « id: ». Envoyez l’ID et le nom de la passerelle de réseau virtuel (VNet1GW) à l’administrateur de Abonnement 5 via e-mail ou une autre méthode.

   Exemple de sortie :

   ```
   "id": "/subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW"
   ```

2. **[Abonnement 5]** Ouvrez une session et connectez-vous à Abonnement 5. Exécutez la commande suivante pour obtenir le nom et l’ID de la passerelle à partir de la sortie :

   ```azurecli
   az network vnet-gateway show -n VNet5GW -g TestRG5
   ```

   Copiez la sortie pour « id: ». Envoyez l’ID et le nom de la passerelle de réseau virtuel (VNet5GW) à l’administrateur de Abonnement 1 via e-mail ou une autre méthode.

3. **[Abonnement 1]** Dans cette étape, vous créez la connexion de TestVNet1 à TestVNet5. Vous pouvez utiliser vos propres valeurs pour la clé partagée, toutefois, cette dernière doit correspondre aux deux connexions. La création d’une connexion peut prendre quelques instants.  Veillez à vous connecter à Abonnement 1.

   ```azurecli
   az network vpn-connection create -n VNet1ToVNet5 -g TestRG1 --vnet-gateway1 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW -l eastus --shared-key "eeffgg" --vnet-gateway2 /subscriptions/e7e33b39-fe28-4822-b65c-a4db8bbff7cb/resourceGroups/TestRG5/providers/Microsoft.Network/virtualNetworkGateways/VNet5GW
   ```

4. **[Abonnement 5]** Cette étape est similaire à celle présentée ci-dessus, sauf que vous créez la connexion de TestVNet5 à TestVNet1. Assurez-vous que les clés partagées correspondent et que vous vous connectez à Abonnement 5.

   ```azurecli
   az network vpn-connection create -n VNet5ToVNet1 -g TestRG5 --vnet-gateway1 /subscriptions/e7e33b39-fe28-4822-b65c-a4db8bbff7cb/resourceGroups/TestRG5/providers/Microsoft.Network/virtualNetworkGateways/VNet5GW -l japaneast --shared-key "eeffgg" --vnet-gateway2 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW
   ```

## <a name="verify-the-connections"></a><a name="verify"></a>Vérifier les connexions
[!INCLUDE [vpn-gateway-no-nsg-include](../../includes/vpn-gateway-no-nsg-include.md)]

[!INCLUDE [verify connections](../../includes/vpn-gateway-verify-connection-cli-rm-include.md)]

## <a name="vnet-to-vnet-faq"></a><a name="faq"></a>Forum Aux Questions sur l’interconnexion de réseaux virtuels
[!INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-faq-vnet-vnet-include.md)]

## <a name="next-steps"></a>Étapes suivantes

* Une fois la connexion achevée, vous pouvez ajouter des machines virtuelles à vos réseaux virtuels. Pour plus d’informations, consultez la [documentation relative aux machines virtuelles](../index.yml).
* Pour plus d’informations sur le protocole BGP, consultez les articles [Vue d’ensemble du protocole BGP](vpn-gateway-bgp-overview.md) et [Comment configurer BGP](vpn-gateway-bgp-resource-manager-ps.md).
---
title: Configurer une adresse IP sortante statique
description: Configurer le pare-feu Azure et les itinéraires définis par l’utilisateur pour les charges de travail Azure Container Instances qui utilisent l’IP publique du pare-feu pour l’entrée et la sortie
ms.topic: article
ms.date: 07/16/2020
ms.openlocfilehash: 497645b9fe7f908cc9b8b4d7ed0ba5e201570160
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89566566"
---
# <a name="configure-a-single-public-ip-address-for-outbound-and-inbound-traffic-to-a-container-group"></a>Configurer une IP publique unique pour le trafic sortant et entrant d’un groupe de conteneurs

La configuration d’un [groupe de conteneurs](container-instances-container-groups.md) avec une adresse IP externe permet aux clients externes d’utiliser l’adresse IP pour accéder à un conteneur du groupe. Par exemple, un navigateur peut accéder à une application web exécutée dans un conteneur. Toutefois, actuellement, un groupe de conteneurs utilise une adresse IP différente pour le trafic sortant. Cette adresse IP de sortie n’est pas exposée par programmation, ce qui rend plus complexe la surveillance et la configuration des groupes de conteneurs pour les règles de pare-feu du client.

Cet article décrit les étapes permettant de configurer un groupe de conteneurs dans un [réseau virtuel](container-instances-virtual-network-concepts.md) intégré à [Pare-feu Azure](../firewall/overview.md). En configurant un itinéraire défini par l’utilisateur vers le groupe de conteneurs et les règles de pare-feu, vous pouvez acheminer et identifier le trafic vers et depuis le groupe de conteneurs. L’entrée et la sortie des groupes de conteneurs utilisent l’IP publique du pare-feu. Une seule adresse IP de sortie peut être utilisée par plusieurs groupes de conteneurs déployés dans le sous-réseau du réseau virtuel délégué à Azure Container Instances.

Dans cet article, vous utilisez Azure CLI pour créer les ressources de ce scénario :

* Des groupes de conteneurs déployés sur un sous-réseau délégué [dans le réseau virtuel](container-instances-vnet.md) 
* Un pare-feu Azure déployé sur le réseau avec une IP publique statique
* Un itinéraire défini par l’utilisateur sur le sous-réseau des groupes de conteneurs
* Une règle NAT pour l’entrée de pare-feu et une règle d’application pour la sortie

Vous validez ensuite l’entrée et la sortie à partir d’exemples de groupes de conteneurs via le pare-feu.

## <a name="deploy-aci-in-a-virtual-network"></a>Déployer ACI dans un réseau virtuel

Dans un cas classique, vous disposez peut-être déjà d’un réseau virtuel Azure dans lequel déployer un groupe de conteneurs. À des fins de démonstration, les commandes suivantes créent un réseau virtuel et un sous-réseau lors de la création du groupe de conteneurs. Le sous-réseau est délégué à Azure Container Instances. 

Le groupe de conteneurs exécute une petite application web à partir de l’image `aci-helloworld`. Comme indiqué dans d’autres articles de la documentation, cette image contient une petite application web écrite en Node.js qui sert une page HTML statique.

Si vous en avez besoin, commencez par créer un groupe de ressources Azure avec la commande [az group create][az-group-create]. Par exemple :

```azurecli
az group create --name myResourceGroup --location eastus
```

Pour simplifier les exemples de commandes suivants, utilisez une variable d’environnement pour le nom du groupe de ressources :

```console
export RESOURCE_GROUP_NAME=myResourceGroup
```

Créez le groupe de conteneurs à l’aide de la commande [az container create][az-container-create] :

```azurecli
az container create \
  --name appcontainer \
  --resource-group $RESOURCE_GROUP_NAME \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --vnet aci-vnet \
  --vnet-address-prefix 10.0.0.0/16 \
  --subnet aci-subnet \
  --subnet-address-prefix 10.0.0.0/24
```

> [!TIP]
> Ajustez la valeur de `--subnet address-prefix` pour l’espace d’adressage IP dont vous avez besoin dans votre sous-réseau. Le plus petit sous-réseau pris en charge est /29, lequel fournit 8 adresses IP. Certaines adresses IP sont réservées à Azure.

Pour une utilisation ultérieure, vous devez obtenir l’adresse IP privée du groupe de conteneurs en exécutant la commande [az container show][az-container-show] :

```azurecli
ACI_PRIVATE_IP="$(az container show --name appcontainer \
  --resource-group $RESOURCE_GROUP_NAME \
  --query ipAddress.ip --output tsv)"
```

## <a name="deploy-azure-firewall-in-network"></a>Déployer Pare-feu Azure dans le réseau

Dans les sections suivantes, utilisez Azure CLI pour déployer un pare-feu Azure dans le réseau virtuel. Pour plus d’informations, consultez [Didacticiel : Déployer et configurer un pare-feu Azure à l’aide du portail Azure](../firewall/deploy-cli.md)

Tout d’abord, utilisez la commande [az network vnet subnet create][az-network-vnet-subnet-create] pour ajouter un sous-réseau nommé AzureFirewallSubnet pour le pare-feu. AzureFirewallSubnet est le nom *obligatoire* de ce sous-réseau.

```azurecli
az network vnet subnet create \
  --name AzureFirewallSubnet \
  --resource-group $RESOURCE_GROUP_NAME \
  --vnet-name aci-vnet   \
  --address-prefix 10.0.1.0/26
```

Utilisez les [commandes Azure CLI](../firewall/deploy-cli.md) suivantes pour créer un pare-feu dans le sous-réseau.

Si ce n’est déjà fait, ajoutez l’extension de pare-feu à Azure CLI à l’aide de la commande [az extension add][az-extension-add] :

```azurecli
az extension add --name azure-firewall
```

Créez les ressources de pare-feu :

```azurecli
az network firewall create \
  --name myFirewall \
  --resource-group $RESOURCE_GROUP_NAME \
  --location eastus

az network public-ip create \
  --name fw-pip \
  --resource-group $RESOURCE_GROUP_NAME \
  --location eastus \
  --allocation-method static \
  --sku standard
    
az network firewall ip-config create \
  --firewall-name myFirewall \
  --name FW-config \
  --public-ip-address fw-pip \
  --resource-group $RESOURCE_GROUP_NAME \
  --vnet-name aci-vnet
```

Mettez à jour la configuration du pare-feu à l’aide de la commande [az network firewall update][az-network-firewall-update] :

```azurecli
az network firewall update \
  --name myFirewall \
  --resource-group $RESOURCE_GROUP_NAME
```

Récupérez l’adresse IP privée du pare-feu à l’aide de la commande [az network firewall ip-config list][az-network-firewall-ip-config-list]. Cette adresse IP privée est utilisée dans une commande ultérieure.


```azurecli
FW_PRIVATE_IP="$(az network firewall ip-config list \
  --resource-group $RESOURCE_GROUP_NAME \
  --firewall-name myFirewall \
  --query "[].privateIpAddress" --output tsv)"
```
Récupérez l’IP publique du pare-feu à l’aide de la commande [az network public-ip show][az-network-public-ip-show]. Cette IP publique est utilisée dans une commande ultérieure.

```azurecli
FW_PUBLIC_IP="$(az network public-ip show \
  --name fw-pip \
  --resource-group $RESOURCE_GROUP_NAME \
  --query ipAddress --output tsv)"
```

## <a name="define-user-defined-route-on-aci-subnet"></a>Définir un itinéraire défini par l’utilisateur sur le sous-réseau ACI

Définissez un itinéraire défini par l’utilisation sur le sous-réseau ACI afin de détourner le trafic vers le pare-feu Azure. Pour plus d’informations, consultez [Acheminer le trafic](../virtual-network/tutorial-create-route-table-cli.md). 

### <a name="create-route-table"></a>Créer une table de routage

Tout d’abord, exécutez la commande [az network route-table create][az-network-route-table-create] suivante pour créer la table de routage. Créez la table de routage dans la même région que le réseau virtuel.

```azurecli
az network route-table create \
  --name Firewall-rt-table \
  --resource-group $RESOURCE_GROUP_NAME \
  --location eastus \
  --disable-bgp-route-propagation true
```

### <a name="create-route"></a>Créer un itinéraire

Exécutez [az network-route-table route create][az-network-route-table-route-create] pour créer un itinéraire dans la table de routage. Pour acheminer le trafic vers le pare-feu, définissez le type de tronçon suivant sur `VirtualAppliance` puis transmettez l’adresse IP privée du pare-feu comme adresse de tronçon suivant.

```azurecli
az network route-table route create \
  --resource-group $RESOURCE_GROUP_NAME \
  --name DG-Route \
  --route-table-name Firewall-rt-table \
  --address-prefix 0.0.0.0/0 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address $FW_PRIVATE_IP
```

### <a name="associate-route-table-to-aci-subnet"></a>Associer la table de routage au sous-réseau ACI

Exécutez la commande [az network vnet subnet update][az-network-vnet-subnet-update] pour associer la table de routage au sous-réseau délégué à Azure Container Instances.

```azurecli
az network vnet subnet update \
  --name aci-subnet \
  --resource-group $RESOURCE_GROUP_NAME \
  --vnet-name aci-vnet \
  --address-prefixes 10.0.0.0/24 \
  --route-table Firewall-rt-table
```

## <a name="configure-rules-on-firewall"></a>Configurer des règles sur le pare-feu

Par défaut, Pare-feu Azure refuse (bloque) le trafic entrant et sortant. 

### <a name="configure-nat-rule-on-firewall-to-aci-subnet"></a>Configurer une règle NAT sur le pare-feu vers le sous-réseau ACI

Créez une [règle NAT](../firewall/rule-processing.md) sur le pare-feu pour traduire et filtrer le trafic Internet entrant vers le conteneur d’applications que vous avez démarré précédemment dans le réseau. Pour plus d’informations, consultez [Filtrer le trafic Internet entrant à l’aide de la DNAT Pare-feu Azure](../firewall/tutorial-firewall-dnat.md).

Créez une règle et collection NAT à l’aide de la commande [az network firewall nat-rule create][az-network-firewall-nat-rule-create] :

```azurecli
az network firewall nat-rule create \
  --firewall-name myFirewall \
  --collection-name myNATCollection \
  --action dnat \
  --name myRule \
  --protocols TCP \
  --source-addresses '*' \
  --destination-addresses $FW_PUBLIC_IP \
  --destination-ports 80 \
  --resource-group $RESOURCE_GROUP_NAME \
  --translated-address $ACI_PRIVATE_IP \
  --translated-port 80 \
  --priority 200
```

Ajoutez des règles NAT si nécessaire pour filtrer le trafic vers d’autres adresses IP dans le sous-réseau. Par exemple, d’autres groupes de conteneurs du sous-réseau peuvent exposer des adresses IP pour le trafic entrant, ou d’autres adresses IP internes peuvent être attribuées au groupe de conteneurs après un redémarrage.

### <a name="create-outbound-application-rule-on-the-firewall"></a>Créer une règle d’application de trafic sortant sur le pare-feu

Exécutez la commande [az network firewall application-rule create][az-network-firewall-application-rule-create] suivante pour créer une règle de trafic sortant sur le pare-feu. Cet exemple de règle accorde au nom de domaine complet `checkip.dyndns.org` l’accès à Azure Container Instances à partir du sous-réseau délégué. L’accès HTTP au site est utilisé dans une étape ultérieure pour confirmer l’adresse IP de sortie d’Azure Container Instances.

```azurecli
az network firewall application-rule create \
  --collection-name myAppCollection \
  --firewall-name myFirewall \
  --name Allow-CheckIP \
  --protocols Http=80 Https=443 \
  --resource-group $RESOURCE_GROUP_NAME \
  --target-fqdns checkip.dyndns.org \
  --source-addresses 10.0.0.0/24 \
  --priority 200 \
  --action Allow
```

## <a name="test-container-group-access-through-the-firewall"></a>Tester l’accès au groupe de conteneurs via le pare-feu

Les sections suivantes vérifient que le sous-réseau délégué à Azure Container Instances est correctement configuré derrière le pare-feu Azure. Les étapes précédentes ont permis d’acheminer le trafic entrant vers le sous-réseau et le trafic sortant du sous-réseau via le pare-feu.

### <a name="test-ingress-to-a-container-group"></a>Tester l’entrée dans un groupe de conteneurs

Testez l’accès entrant au *appcontainer* s’exécutant dans le réseau virtuel en accédant à l’IP publique du pare-feu. Précédemment, vous avez stocké l’IP publique dans la variable $FW_PUBLIC_IP :

```bash
echo $FW_PUBLIC_IP
```

Le résultat se présente ainsi :

```console
52.142.18.133
```

Si la règle NAT sur le pare-feu est configurée correctement, les éléments suivants s’affichent lorsque vous entrez l’IP publique du pare-feu dans votre navigateur :

:::image type="content" source="media/container-instances-egress-ip-address/aci-ingress-ip-address.png" alt-text="Accéder à l’IP publique du pare-feu":::

### <a name="test-egress-from-a-container-group"></a>Tester la sortie d’un groupe de conteneurs


Déployez l’exemple de conteneur suivant dans le réseau virtuel. Lorsqu’il s’exécute, il envoie une requête HTTP unique à `http://checkip.dyndns.org`, qui affiche l’adresse IP de l’expéditeur (l’adresse IP de sortie). Si la règle d’application sur le pare-feu est configurée correctement, l’IP publique du pare-feu est retournée.

```azurecli
az container create \
  --resource-group $RESOURCE_GROUP_NAME \
  --name testegress \
  --image mcr.microsoft.com/azuredocs/aci-tutorial-sidecar \
  --command-line "curl -s http://checkip.dyndns.org" \
  --restart-policy OnFailure \
  --vnet aci-vnet \
  --subnet aci-subnet
```

Affichez les journaux de conteneur pour confirmer que l’adresse IP est identique à l’IP publique du pare-feu.

```azurecli
az container logs \
  --resource-group $RESOURCE_GROUP_NAME \
  --name testegress 
```

Le résultat se présente ainsi :

```console
<html><head><title>Current IP Check</title></head><body>Current IP Address: 52.142.18.133</body></html>
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez configuré des groupes de conteneurs dans un réseau virtuel situé derrière un pare-feu Azure. Vous avez configuré un itinéraire défini par l’utilisateur et des règles d’application et NAT sur le pare-feu. À l’aide de cette configuration, vous avez configuré une adresse IP statique unique pour l’entrée et la sortie d’Azure Container Instances.

Pour plus d’informations sur la gestion du trafic et la protection des ressources Azure, consultez la documentation relative à [Pare-feu Azure](../firewall/index.yml).



[az-group-create]: /cli/azure/group#az-group-create
[az-container-create]: /cli/azure/container#az-container-create
[az-network-vnet-subnet-create]: /cli/azure/network/vnet/subnet#az-network-vnet-subnet-create
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-network-firewall-update]: /cli/azure/ext/azure-firewall/network/firewall#ext-azure-firewall-az-network-firewall-update
[az-network-public-ip-show]: /cli/azure/network/public-ip/#az-network-public-ip-show
[az-network-route-table-create]:/cli/azure/network/route-table/#az-network-route-table-create
[az-network-route-table-route-create]: /cli/azure/network/route-table/route#az-network-route-table-route-create
[az-network-firewall-ip-config-list]: /cli/azure/ext/azure-firewall/network/firewall/ip-config#ext-azure-firewall-az-network-firewall-ip-config-list
[az-network-vnet-subnet-update]: /cli/azure/network/vnet/subnet#az-network-vnet-subnet-update
[az-container-exec]: /cli/azure/container#az-container-exec
[az-vm-create]: /cli/azure/vm#az-vm-create
[az-vm-open-port]: /cli/azure/vm#az-vm-open-port
[az-vm-list-ip-addresses]: /cli/azure/vm#az-vm-list-ip-addresses
[az-network-firewall-application-rule-create]: /cli/azure/ext/azure-firewall/network/firewall/application-rule#ext-azure-firewall-az-network-firewall-application-rule-create
[az-network-firewall-nat-rule-create]: /cli/azure/ext/azure-firewall/network/firewall/nat-rule#ext-azure-firewall-az-network-firewall-nat-rule-create







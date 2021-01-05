---
title: Adresse IP statique pour un groupe de conteneurs
description: Créer un groupe de conteneurs dans un réseau virtuel et utiliser une passerelle d’application Azure pour exposer une adresse IP front-end statique à une application web conteneurisée
ms.topic: article
ms.date: 03/16/2020
ms.openlocfilehash: bc128da0f4c2e92af98781cef45f48f9e8aeab31
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86260788"
---
# <a name="expose-a-static-ip-address-for-a-container-group"></a>Exposer une adresse IP statique pour un groupe de conteneurs

Cet article illustre une façon d’exposer une adresse IP publique statique pour un [groupe de conteneurs](container-instances-container-groups.md) à l’aide d’une [passerelle d’application](../application-gateway/overview.md) Azure. Procédez ainsi quand vous avez besoin d’un point d’entrée statique pour une application conteneurisée externe qui s’exécute dans Azure Container Instances. 

Dans cet article, vous utilisez Azure CLI pour créer les ressources de ce scénario :

* Un réseau virtuel Azure.
* Un groupe de conteneurs déployé [sur le réseau virtuel](container-instances-vnet.md) qui héberge une petite application web
* Une passerelle d’application avec une adresse IP front-end publique, un écouteur pour héberger un site web sur la passerelle et une route vers le groupe de conteneurs back-end

Tant que la passerelle d’application s’exécute et que le groupe de conteneurs expose une adresse IP privée stable dans le sous-réseau délégué du réseau, le groupe de conteneurs est accessible à cette adresse IP publique.

> [!NOTE]
> Azure facture une passerelle d’application en fonction de la durée pendant laquelle elle est provisionnée et disponible ainsi que de la quantité de données qu’elle traite. Consultez les [tarifs](https://azure.microsoft.com/pricing/details/application-gateway/).

## <a name="create-virtual-network"></a>Création d’un réseau virtuel

Dans un cas classique, vous disposez peut-être déjà d’un réseau virtuel Azure. Si vous n’en avez pas, créez-en un comme indiqué dans les exemples de commandes suivants. Le réseau virtuel a besoin de sous-réseaux distincts pour la passerelle d’application et le groupe de conteneurs.

Si vous en avez besoin, créez un groupe de ressources Azure. Par exemple :

```azureci
az group create --name myResourceGroup --location eastus
```

Utilisez la commande [az network vnet create][az-network-vnet-create] pour créer un réseau virtuel. Cette commande crée le sous-réseau *myAGSubnet* dans le réseau.

```azurecli
az network vnet create \
  --name myVNet \
  --resource-group myResourceGroup \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name myAGSubnet \
  --subnet-prefix 10.0.1.0/24
```

Utilisez la commande [az network vnet subnet create][az-network-vnet-subnet-create] afin de créer un sous-réseau pour le groupe de conteneurs back-end. Ici, il est nommé *myACISubnet*.

```azurecli
az network vnet subnet create \
  --name myACISubnet \
  --resource-group myResourceGroup \
  --vnet-name myVNet   \
  --address-prefix 10.0.2.0/24
```

Utilisez la commande [az network public-ip create][az-network-public-ip-create] pour créer une ressource IP publique statique. Dans une étape ultérieure, cette adresse est configurée en tant que front-end de la passerelle d’application.

```azurecli
az network public-ip create \
  --resource-group myResourceGroup \
  --name myAGPublicIPAddress \
  --allocation-method Static \
  --sku Standard
```

## <a name="create-container-group"></a>Créer un groupe de conteneurs

Exécutez la commande [az container create][az-container-create] suivante pour créer un groupe de conteneurs dans le réseau virtuel que vous avez configuré à l’étape précédente. 

Le groupe est déployé sur le sous-réseau *myACISubnet* et contient une instance unique nommée *appcontainer* qui extrait l’image `aci-helloworld`. Comme indiqué dans d’autres articles de la documentation, cette image contient une petite application web écrite en Node.js qui sert une page HTML statique. 

```azurecli
az container create \
  --name appcontainer \
  --resource-group myResourceGroup \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --vnet myVNet \
  --subnet myACISubnet
```

Une fois le déploiement correctement effectué, le groupe de conteneurs reçoit une adresse IP privée dans le réseau virtuel. Par exemple, exécutez la commande [az container show][az-container-show] suivante pour récupérer l’adresse IP du groupe :

```azurecli
az container show \
  --name appcontainer --resource-group myResourceGroup \
  --query ipAddress.ip --output tsv
```

Le résultat ressemble à ce qui suit : `10.0.2.4`.

En vue d’une utilisation ultérieure, enregistrez l’adresse IP dans une variable d’environnement :

```azurecli
ACI_IP=$(az container show \
  --name appcontainer \
  --resource-group myResourceGroup \
  --query ipAddress.ip --output tsv)
```

## <a name="create-application-gateway"></a>Créer une passerelle Application Gateway

Créez une passerelle d’application dans le réseau virtuel, en suivant les étapes décrites dans le [guide de démarrage rapide consacré aux passerelles d’application](../application-gateway/quick-create-cli.md). La commande [az network application-gateway create][az-network-application-gateway-create] suivante crée une passerelle avec une adresse IP front-end publique et une route vers le groupe de conteneurs back-end. Pour plus d’informations sur les paramètres de passerelle, consultez la [documentation Application Gateway](../application-gateway/index.yml).

```azurecli
az network application-gateway create \
  --name myAppGateway \
  --location eastus \
  --resource-group myResourceGroup \
  --capacity 2 \
  --sku Standard_v2 \
  --http-settings-protocol http \
  --public-ip-address myAGPublicIPAddress \
  --vnet-name myVNet \
  --subnet myAGSubnet \
  --servers "$ACI_IP" 
```


La création de la passerelle d’application par Azure peut prendre jusqu’à 15 minutes. 

## <a name="test-public-ip-address"></a>Tester l’adresse IP publique
  
Vous pouvez maintenant tester l’accès à l’application web qui s’exécute dans le groupe de conteneurs derrière la passerelle d’application.

Exécutez la commande [az network public-ip show][az-network-public-ip-show] pour récupérer l’adresse IP publique front-end de la passerelle :

```azurecli
az network public-ip show \
--resource-group myresourcegroup \
--name myAGPublicIPAddress \
--query [ipAddress] \
--output tsv
```

La sortie est une adresse IP publique, semblable à celle-ci : `52.142.18.133`.

Si la configuration est correcte, vous pouvez afficher l’application web en cours d’exécution en accédant à l’adresse IP publique de la passerelle dans votre navigateur. Si l’accès réussit, vous devez obtenir une page semblable à celle-ci :

![Capture d’écran du navigateur représentant une application exécutée dans une instance de conteneur Azure](./media/container-instances-application-gateway/aci-app-app-gateway.png)

## <a name="next-steps"></a>Étapes suivantes

* Consultez un [modèle de démarrage rapide](https://github.com/Azure/azure-quickstart-templates/tree/master/201-aci-wordpress-vnet) pour créer un groupe de conteneurs avec une instance de conteneur WordPress en tant que serveur back-end derrière une passerelle d’application.
* Vous pouvez également configurer une passerelle d’application avec un certificat pour la terminaison SSL. Consultez la [vue d’ensemble](../application-gateway/ssl-overview.md) et le [tutoriel](../application-gateway/create-ssl-portal.md).
* Selon votre scénario, envisagez d’utiliser d’autres solutions Azure d’équilibrage de charge avec Azure Container Instances. Par exemple, utilisez [Azure Traffic Manager](../traffic-manager/traffic-manager-overview.md) pour répartir le trafic entre plusieurs instances de conteneur et entre plusieurs régions. Consultez ce [billet de blog](https://aaronmsft.com/posts/azure-container-instances/).

[az-network-vnet-create]:  /cli/azure/network/vnet#az-network-vnet-create
[az-network-vnet-subnet-create]: /cli/azure/network/vnet/subnet#az-network-vnet-subnet-create
[az-network-public-ip-create]: /cli/azure/network/public-ip#az-network-public-ip-create
[az-network-public-ip-show]: /cli/azure/network/public-ip#az-network-public-ip-show
[az-network-application-gateway-create]: /cli/azure/network/application-gateway#az-network-application-gateway-create
[az-container-create]: /cli/azure/container#az-container-create
[az-container-show]: /cli/azure/container#az-container-show

---
title: Acheminer le trafic web selon l’URL - Azure CLI
description: Avec cet article, découvrez comment acheminer le trafic web selon l’URL vers des pools scalables spécifiques de serveurs à l’aide d’Azure CLI.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 08/01/2019
ms.author: victorh
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 8e8fed99fe0b1de52d2e2d0018dfd8867b54b63b
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94566518"
---
# <a name="route-web-traffic-based-on-the-url-using-the-azure-cli"></a>Acheminer le trafic web selon l’URL à l’aide d’Azure CLI

En tant qu’administrateur gérant le trafic web, vous souhaitez aider vos clients ou utilisateurs à obtenir les informations nécessaires aussi vite que possible. Pour optimiser leur expérience, vous pouvez acheminer différents types de trafic web vers différentes ressources de serveur. Cet article vous montre comment utiliser Azure CLI pour installer et configurer l’acheminement Application Gateway pour différents types de trafic depuis votre application. L’acheminement dirige le trafic vers différents pools de serveurs basé sur l’URL.

![Exemple d’acheminement d’URL](./media/tutorial-url-route-cli/scenario.png)

Dans cet article, vous apprendrez comment :

* Créer un groupe de ressources pour les ressources réseau dont vous aurez besoin
* Créer les ressources réseau
* Créer une passerelle d’application pour le trafic provenant de votre application
* Spécifier des pools de serveurs et des règles d’acheminement pour les différents types de trafic
* Créer un groupe identique pour chaque pool afin que ce dernier puisse effectuer une mise à l'échelle
* Exécuter un test pour vérifier que les différents types de trafic sont dirigés vers le bon pool

Si vous préférez, vous pouvez suivre cette procédure en utilisant [Azure PowerShell](tutorial-url-route-powershell.md) ou le [portail Azure](create-url-route-portal.md).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

 - Cet tutoriel nécessite la version 2.0.4 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. Créez un groupe de ressources avec `az group create`.

L’exemple suivant crée un groupe de ressources nommé *myResourceGroupAG* à l’emplacement *eastus*.

```azurecli-interactive
az group create --name myResourceGroupAG --location eastus
```

## <a name="create-network-resources"></a>Créer des ressources réseau

Créez le réseau virtuel nommé *myVNet* et le sous-réseau nommé *myAGSubnet* à l’aide de la commande `az network vnet create`. Ajoutez ensuite le sous-réseau nommé *myBackendSubnet* nécessaire aux serveurs backend à l’aide de la commande `az network vnet subnet create`. Créez l’adresse IP publique nommée *myAGPublicIPAddress* à l’aide de la commande `az network public-ip create`.

```azurecli-interactive
az network vnet create \
  --name myVNet \
  --resource-group myResourceGroupAG \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name myAGSubnet \
  --subnet-prefix 10.0.1.0/24

az network vnet subnet create \
  --name myBackendSubnet \
  --resource-group myResourceGroupAG \
  --vnet-name myVNet \
  --address-prefix 10.0.2.0/24

az network public-ip create \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --allocation-method Static \
  --sku Standard
```

## <a name="create-the-app-gateway-with-a-url-map"></a>Créer la passerelle d’application avec une carte URL

Utilisez `az network application-gateway create` pour créer une passerelle d’application nommée *myAppGateway*. Quand vous créez une passerelle d’application avec Azure CLI, vous spécifiez des informations de configuration, telles que la capacité, la référence SKU et les paramètres HTTP. La passerelle d’application est affectée à *myAGSubnet* et à *myAGPublicIPAddress*.

```azurecli-interactive
az network application-gateway create \
  --name myAppGateway \
  --location eastus \
  --resource-group myResourceGroupAG \
  --vnet-name myVNet \
  --subnet myAGsubnet \
  --capacity 2 \
  --sku Standard_v2 \
  --http-settings-cookie-based-affinity Disabled \
  --frontend-port 80 \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --public-ip-address myAGPublicIPAddress
```

 La création de la passerelle d’application peut prendre plusieurs minutes. Une fois la passerelle d’application créée, vous pouvez voir ces nouvelles fonctionnalités :


|Fonctionnalité  |Description  |
|---------|---------|
|appGatewayBackendPool     |Une passerelle d’application doit avoir au moins un pool d’adresses backend.|
|appGatewayBackendHttpSettings     |Spécifie que le port 80 et le protocole HTTP sont utilisés pour la communication.|
|appGatewayHttpListener     |Écouteur par défaut associé à appGatewayBackendPool|
|appGatewayFrontendIP     |Assigne myAGPublicIPAddress à appGatewayHttpListener.|
|rule1     |Règle de routage par défaut associée à appGatewayHttpListener.|

### <a name="add-image-and-video-backend-pools-and-a-port"></a>Ajouter un port et les pools principaux image et vidéo

Ajoutez des pools principaux nommés *imagesBackendPool* et *videoBackendPool* à votre passerelle d’application à l’aide de la commande `az network application-gateway address-pool create`. Ajoutez le port frontal pour les pools à l’aide de la commande `az network application-gateway frontend-port create`.

```azurecli-interactive
az network application-gateway address-pool create \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name imagesBackendPool

az network application-gateway address-pool create \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name videoBackendPool

az network application-gateway frontend-port create \
  --port 8080 \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name port8080
```

### <a name="add-a-backend-listener"></a>Ajouter un écouteur principal

Ajoutez l’écouteur principal nommé *backendListener*, nécessaire pour acheminer le trafic, à l’aide de la commande`az network application-gateway http-listener create`.


```azurecli-interactive
az network application-gateway http-listener create \
  --name backendListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port port8080 \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway
```

### <a name="add-a-url-path-map"></a>Ajouter une carte de chemin d’accès URL

Les cartes de chemin d’accès URL assurent que certaines URL sont acheminées vers des pools principaux spécifiques. Créer des cartes de chemin d’accès nommés *imagePathRule* et *videoPathRule* à l’aide des commandes `az network application-gateway url-path-map create` et `az network application-gateway url-path-map rule create`.

```azurecli-interactive
az network application-gateway url-path-map create \
  --gateway-name myAppGateway \
  --name myPathMap \
  --paths /images/* \
  --resource-group myResourceGroupAG \
  --address-pool imagesBackendPool \
  --default-address-pool appGatewayBackendPool \
  --default-http-settings appGatewayBackendHttpSettings \
  --http-settings appGatewayBackendHttpSettings \
  --rule-name imagePathRule

az network application-gateway url-path-map rule create \
  --gateway-name myAppGateway \
  --name videoPathRule \
  --resource-group myResourceGroupAG \
  --path-map-name myPathMap \
  --paths /video/* \
  --address-pool videoBackendPool
```

### <a name="add-a-routing-rule"></a>Ajouter une règle de routage

La règle d’acheminement associe les cartes d’URL à l’écouteur créé. Ajoutez une règle nommée *rule2* à l’aide de la commande `az network application-gateway rule create`.

```azurecli-interactive
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name rule2 \
  --resource-group myResourceGroupAG \
  --http-listener backendListener \
  --rule-type PathBasedRouting \
  --url-path-map myPathMap \
  --address-pool appGatewayBackendPool
```

## <a name="create-virtual-machine-scale-sets"></a>Créer des groupes de machines virtuelles identiques

Cet article crée trois groupes de machines virtuelles identiques prenant en charge les trois pools backend que vous avez créés. Ils sont nommés *myvmss1*, *myvmss2* et *myvmss3*. Chacun contient deux instances de machines virtuelles sur lesquelles NGINX sera installé.

```azurecli-interactive
for i in `seq 1 3`; do

  if [ $i -eq 1 ]
  then
    poolName="appGatewayBackendPool" 
  fi

  if [ $i -eq 2 ]
  then
    poolName="imagesBackendPool"
  fi

  if [ $i -eq 3 ]
  then
    poolName="videoBackendPool"
  fi

  az vmss create \
    --name myvmss$i \
    --resource-group myResourceGroupAG \
    --image UbuntuLTS \
    --admin-username azureuser \
    --admin-password Azure123456! \
    --instance-count 2 \
    --vnet-name myVNet \
    --subnet myBackendSubnet \
    --vm-sku Standard_DS2 \
    --upgrade-policy-mode Automatic \
    --app-gateway myAppGateway \
    --backend-pool-name $poolName
done
```

### <a name="install-nginx"></a>Installer NGINX

```azurecli-interactive
for i in `seq 1 3`; do
  az vmss extension set \
    --publisher Microsoft.Azure.Extensions \
    --version 2.0 \
    --name CustomScript \
    --resource-group myResourceGroupAG \
    --vmss-name myvmss$i \
    --settings '{ "fileUris": ["https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/install_nginx.sh"], "commandToExecute": "./install_nginx.sh" }'
done
```

## <a name="test-the-application-gateway"></a>Tester la passerelle d’application

Pour obtenir l’adresse IP publique de la passerelle d’application, utilisez la commande az network public-ip show. Copiez l’adresse IP publique, puis collez-la dans la barre d’adresses de votre navigateur. Par exemple, `http://40.121.222.19`, `http://40.121.222.19:8080/images/test.htm` ou `http://40.121.222.19:8080/video/test.htm`.

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [ipAddress] \
  --output tsv
```

![Tester l’URL de base dans la passerelle d’application](./media/tutorial-url-route-cli/application-gateway-nginx.png)

Modifiez l’URL http://&lt;ip-address&gt;:8080/images/test.htm, en remplaçant votre adresse IP par &lt;ip-address&gt;. Vous devez ensuite voir quelque chose ressemblant à ceci :

![Tester l’URL images dans la passerelle d’application](./media/tutorial-url-route-cli/application-gateway-nginx-images.png)

Modifiez l’URL http://&lt;ip-address&gt;:8080/video/test.htm, en remplaçant votre adresse IP par &lt;ip-address&gt;. Vous devez ensuite voir quelque chose ressemblant à ceci.

![Tester l’URL vidéo dans la passerelle d’application](./media/tutorial-url-route-cli/application-gateway-nginx-video.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources, la passerelle d’application et toutes les ressources associées.

```azurecli-interactive
az group delete --name myResourceGroupAG
```

## <a name="next-steps"></a>Étapes suivantes

[Créer une passerelle d’application avec réacheminement par chemin d’URL](./tutorial-url-redirect-cli.md)

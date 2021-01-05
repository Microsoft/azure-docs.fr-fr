---
title: 'Tutoriel : Redirection basée sur un chemin utilisant l’interface de ligne de commande'
titleSuffix: Azure Application Gateway
description: Dans ce tutoriel, vous découvrez comment créer une passerelle d’application avec un trafic redirigé en fonction d’un chemin d’URL, à l’aide d’Azure CLI.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: tutorial
ms.date: 08/27/2020
ms.author: victorh
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 36ba593a1d8cd2e50293eaf77dc9ec864245df4c
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94566586"
---
# <a name="tutorial-create-an-application-gateway-with-url-path-based-redirection-using-the-azure-cli"></a>Tutoriel : Créer une passerelle d’application avec une redirection basée sur un chemin d’accès d’URL à l’aide d’Azure CLI

Vous pouvez utiliser Azure CLI pour configurer des [règles d’acheminement par chemin d’accès URL](tutorial-url-route-cli.md) lors de la création d’une [passerelle d’application](./overview.md). Ce didacticiel montre comment créer des pools principaux à l’aide de [groupes de machines virtuelles identiques](../virtual-machine-scale-sets/overview.md). Vous créez ensuite des règles de routage d’URL garantissant que le trafic web est redirigé vers le pool principal approprié.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Configurer le réseau
> * Créer une passerelle Application Gateway
> * Créer des écouteurs et des règles de routage
> * Créer des groupes de machines virtuelles identiques pour les pools principaux

L’exemple suivant montre le trafic du site en provenance des ports 8080 et 8081, et sa redirection vers les même pools principaux :

![Exemple d’acheminement d’URL](./media/tutorial-url-redirect-cli/scenario.png)

Si vous préférez, vous pouvez effectuer ce didacticiel en utilisant [Azure PowerShell](tutorial-url-redirect-powershell.md).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

 - Ce tutoriel nécessite l’interface Azure CLI version 2.0.4 ou ultérieure. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. Créez un groupe de ressources à l’aide de la commande [az group create](/cli/azure/group).

L’exemple suivant crée un groupe de ressources nommé *myResourceGroupAG* à l’emplacement *eastus*.

```azurecli-interactive 
az group create --name myResourceGroupAG --location eastus
```

## <a name="create-network-resources"></a>Créer des ressources réseau 

Créez le réseau virtuel nommé *myVNet* et le sous-réseau nommé *myAGSubnet* à l’aide de la commande [az network vnet create](/cli/azure/network/vnet). Vous pouvez ensuite ajouter le sous-réseau nommé *myBackendSubnet* nécessaire aux serveurs backend à l’aide de la commande [az network vnet subnet create](/cli/azure/network/vnet/subnet). Créez l’adresse IP publique nommée *myAGPublicIPAddress* à l’aide de la commande [az network public-ip create](/cli/azure/network/public-ip).

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

## <a name="create-an-application-gateway"></a>Créer une passerelle Application Gateway

Utilisez la commande [az network application-gateway create](/cli/azure/network/application-gateway) pour créer la passerelle d’application nommée myAppGateway. Quand vous créez une passerelle d’application avec Azure CLI, vous spécifiez des informations de configuration, telles que la capacité, la référence SKU et les paramètres HTTP. La passerelle d’application est assignée aux *myAGSubnet* et *myPublicIPAddress* que vous avez créés.

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

- *appGatewayBackendPool* : une passerelle d’application doit avoir au moins un pool d’adresses backend.
- *appGatewayBackendHttpSettings* : spécifie que le port 80 et le protocole HTTP sont utilisés pour la communication.
- *appGatewayHttpListener* : écouteur par défaut associé à *appGatewayBackendPool*.
- *appGatewayFrontendIP* - assigne *myAGPublicIPAddress* à *appGatewayHttpListener*.
- *rule1* : règle de routage par défaut associée à *appGatewayHttpListener*.


### <a name="add-backend-pools-and-ports"></a>Ajouter des ports et des pools principaux

Vous pouvez ajouter des pools d’adresses principaux nommés *imagesBackendPool* et *videoBackendPool* à votre passerelle d’application en utilisant l’applet de commande [az network application-gateway address-pool create](/cli/azure/network/application-gateway/address-pool). Pour ajouter les ports frontaux des pools, utilisez la commande [az network application-gateway frontend-port create](/cli/azure/network/application-gateway/frontend-port). 

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
  --name bport

az network application-gateway frontend-port create \
  --port 8081 \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name rport
```

## <a name="add-listeners-and-rules"></a>Ajouter des écouteurs et des règles

### <a name="add-listeners"></a>Ajouter des écouteurs

Ajoutez les écouteurs principaux nommés *backendListener* et *redirectedListener*, nécessaires pour router le trafic en utilisant l’applet de commande [az network application-gateway http-listener create](/cli/azure/network/application-gateway/http-listener).


```azurecli-interactive
az network application-gateway http-listener create \
  --name backendListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port bport \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway

az network application-gateway http-listener create \
  --name redirectedListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port rport \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway
```

### <a name="add-the-default-url-path-map"></a>Ajouter le mappage de chemin d’accès d’URL par défaut

Les cartes de chemin d’accès URL spécifient que certaines URL sont acheminées vers des pools backend spécifiques. Vous pouvez créer des cartes de chemins d’accès URL nommées *imagePathRule* et *videoPathRule* à l’aide des commandes [az network application-gateway url-path-map create](/cli/azure/network/application-gateway/url-path-map) et [az network application-gateway url-path-map rule create](/cli/azure/network/application-gateway/url-path-map/rule).

```azurecli-interactive
az network application-gateway url-path-map create \
  --gateway-name myAppGateway \
  --name urlpathmap \
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
  --path-map-name urlpathmap \
  --paths /video/* \
  --address-pool videoBackendPool
```

### <a name="add-redirection-configuration"></a>Ajouter une configuration de redirection

Vous pouvez configurer la redirection de l’écouteur en utilisant l’applet de commande [az network application-gateway redirect-config create](/cli/azure/network/application-gateway/redirect-config).

```azurecli-interactive
az network application-gateway redirect-config create \
  --gateway-name myAppGateway \
  --name redirectConfig \
  --resource-group myResourceGroupAG \
  --type Found \
  --include-path true \
  --include-query-string true \
  --target-listener backendListener
```

### <a name="add-the-redirection-url-path-map"></a>Ajouter le mappage de chemin d’accès d’URL de redirection

```azurecli-interactive
az network application-gateway url-path-map create \
  --gateway-name myAppGateway \
  --name redirectpathmap \
  --paths /images/* \
  --resource-group myResourceGroupAG \
  --redirect-config redirectConfig \
  --rule-name redirectPathRule
```

### <a name="add-routing-rules"></a>Ajouter des règles de routage

Les règles de routage associent les mappages de chemin d’accès d’URL aux écouteurs que vous avez créés. Vous pouvez ajouter les règles nommées *defaultRule* et *redirectedRule* en utilisant l’applet de commande [az network application-gateway rule create](/cli/azure/network/application-gateway/rule).

```azurecli-interactive
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name defaultRule \
  --resource-group myResourceGroupAG \
  --http-listener backendListener \
  --rule-type PathBasedRouting \
  --url-path-map urlpathmap \
  --address-pool appGatewayBackendPool

az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name redirectedRule \
  --resource-group myResourceGroupAG \
  --http-listener redirectedListener \
  --rule-type PathBasedRouting \
  --url-path-map redirectpathmap \
  --address-pool appGatewayBackendPool
```

## <a name="create-virtual-machine-scale-sets"></a>Créer des groupes de machines virtuelles identiques

Cet exemple crée trois groupes de machines virtuelles identiques prenant en charge les trois pools principaux qui ont été créés. Ils sont nommés *myvmss1*, *myvmss2* et *myvmss3*. Chacun contient deux instances de machines virtuelles sur lesquelles NGINX sera installé.

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

Pour obtenir l’adresse IP publique de la passerelle d’application, utilisez la commande [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show). Copiez l’adresse IP publique, puis collez-la dans la barre d’adresses de votre navigateur. Par exemple, `http://40.121.222.19`, `http://40.121.222.19:8080/images/test.htm`, `http://40.121.222.19:8080/video/test.htm` ou `http://40.121.222.19:8081/images/test.htm`.

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [ipAddress] \
  --output tsv
```

![Tester l’URL de base dans la passerelle d’application](./media/tutorial-url-redirect-cli/application-gateway-nginx.png)

Modifiez l’URL http://&lt;ip-address&gt;:8080/images/test.htm, en remplaçant votre adresse IP par &lt;ip-address&gt;. Vous devez ensuite voir quelque chose ressemblant à ceci :

![Tester l’URL images dans la passerelle d’application](./media/tutorial-url-redirect-cli/application-gateway-nginx-images.png)

Modifiez l’URL http://&lt;ip-address&gt;:8080/video/test.htm, en remplaçant votre adresse IP par &lt;ip-address&gt;. Vous devez ensuite voir quelque chose ressemblant à ceci :

![Tester l’URL vidéo dans la passerelle d’application](./media/tutorial-url-redirect-cli/application-gateway-nginx-video.png)

À présent, changez l’URL en http://&lt;ip-address&gt;:8081/images/test.htm, en remplaçant &lt;ip-address&gt; par votre adresse IP, de façon à voir le trafic redirigé vers le pool backend d’images à l’adresse http://&lt;ip-address&gt;:8080/images.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources, la passerelle d’application et toutes les ressources associées.

```azurecli-interactive
az group delete --name myResourceGroupAG
```
## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur ce que la passerelle d’application vous permet de faire](./overview.md)
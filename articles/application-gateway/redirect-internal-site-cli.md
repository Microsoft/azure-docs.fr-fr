---
title: Redirection interne avec l’interface CLI
titleSuffix: Azure Application Gateway
description: Découvrez comment créer une passerelle d’application qui redirige le trafic web interne vers le pool approprié à l’aide de l’interface CLI d’Azure.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/14/2019
ms.author: victorh
ms.openlocfilehash: b443fa7c2d6c644fc1173295f89813c18657d160
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94566722"
---
# <a name="create-an-application-gateway-with-internal-redirection-using-the-azure-cli"></a>Créer une passerelle d’application avec redirection interne à l’aide d’Azure CLI

Vous pouvez utiliser l’interface CLI Azure pour configurer une [redirection du trafic web](multiple-site-overview.md) lors de la création d’une [passerelle d’application](overview.md). Dans ce didacticiel, vous définissez un pool backend à l’aide d’un groupe de machines virtuelles identiques. Vous configurez ensuite des écouteurs et des règles en fonction de domaines qui vous appartiennent pour vérifier que le trafic web arrive au pool approprié. Ce didacticiel, qui part du principe que vous possédez plusieurs domaines, utilise *www\.contoso.com* and *www\.contoso.org* en guise d’exemples.

Dans cet article, vous apprendrez comment :

* Configurer le réseau
* Créer une passerelle Application Gateway
* Ajouter des écouteurs et une règle de redirection
* Créer un groupe de machines virtuelles identiques avec le pool backend
* Créer un enregistrement CNAME dans votre domaine

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

 - Cet tutoriel nécessite la version 2.0.4 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. Créez un groupe de ressources à l’aide de la commande [az group create](/cli/azure/group).

L’exemple suivant crée un groupe de ressources nommé *myResourceGroupAG* à l’emplacement *eastus*.

```azurecli-interactive 
az group create --name myResourceGroupAG --location eastus
```

## <a name="create-network-resources"></a>Créer des ressources réseau 

Créez le réseau virtuel nommé *myVNet* et le sous-réseau nommé *myAGSubnet* à l’aide de la commande [az network vnet create](/cli/azure/network/vnet). Vous pouvez ensuite ajouter le sous-réseau nommé *myBackendSubnet* nécessaire au pool backend de serveurs à l’aide de la commande [az network vnet subnet create](/cli/azure/network/vnet/subnet). Créez l’adresse IP publique nommée *myAGPublicIPAddress* à l’aide de la commande [az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create).

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
  --name myAGPublicIPAddress
```

## <a name="create-an-application-gateway"></a>Créer une passerelle Application Gateway

Vous pouvez utiliser la commande [az network application-gateway create](/cli/azure/network/application-gateway) pour créer la passerelle d’application nommée *myAppGateway*. Quand vous créez une passerelle d’application avec Azure CLI, vous spécifiez des informations de configuration, telles que la capacité, la référence SKU et les paramètres HTTP. La passerelle d’application est affectée à *myAGSubnet* et à *myAGPublicIPAddress*, que vous avez créés. 

```azurecli-interactive
az network application-gateway create \
  --name myAppGateway \
  --location eastus \
  --resource-group myResourceGroupAG \
  --vnet-name myVNet \
  --subnet myAGsubnet \
  --capacity 2 \
  --sku Standard_Medium \
  --http-settings-cookie-based-affinity Disabled \
  --frontend-port 80 \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --public-ip-address myAGPublicIPAddress
```

La création de la passerelle d’application peut prendre plusieurs minutes. Une fois la passerelle d’application créée, vous pouvez voir ses nouvelles fonctionnalités suivantes :

- *appGatewayBackendPool* : une passerelle d’application doit avoir au moins un pool d’adresses backend.
- *appGatewayBackendHttpSettings* : spécifie que le port 80 et le protocole HTTP sont utilisés pour la communication.
- *appGatewayHttpListener* : écouteur par défaut associé à *appGatewayBackendPool*.
- *appGatewayFrontendIP* - assigne *myAGPublicIPAddress* à *appGatewayHttpListener*.
- *rule1* : règle de routage par défaut associée à *appGatewayHttpListener*.


## <a name="add-listeners-and-rules"></a>Ajouter des écouteurs et des règles 

Un écouteur est requis pour permettre à la passerelle d’application d’acheminer le trafic de manière appropriée vers le pool principal. Ce didacticiel vous montre comment créer deux écouteurs pour vos deux domaines. Dans cet exemple, des écouteurs sont créés pour les domaines *www\.contoso.com* et *www\.contoso.org*.

Ajoutez les écouteurs backend nécessaires pour acheminer le trafic à l’aide de la commande [az network application-gateway http-listener create](/cli/azure/network/application-gateway/http-listener#az-network-application-gateway-http-listener-create).

```azurecli-interactive
az network application-gateway http-listener create \
  --name contosoComListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port appGatewayFrontendPort \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway \
  --host-name www.contoso.com
az network application-gateway http-listener create \
  --name contosoOrgListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port appGatewayFrontendPort \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway \
  --host-name www.contoso.org   
  ```

### <a name="add-the-redirection-configuration"></a>Ajouter la configuration de redirection

Ajoutez la configuration de redirection qui envoie le trafic de *www\.consoto.org* à l’écouteur pour *www\.contoso.com* dans la passerelle d’application à l’aide de la commande [az network application-gateway redirect-config create](/cli/azure/network/application-gateway/redirect-config#az-network-application-gateway-redirect-config-create).

```azurecli-interactive
az network application-gateway redirect-config create \
  --name orgToCom \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --type Permanent \
  --target-listener contosoListener \
  --include-path true \
  --include-query-string true
```

### <a name="add-routing-rules"></a>Ajouter des règles de routage

Les règles sont traitées dans l’ordre dans lequel elles sont créées, et le trafic est dirigé à l’aide de la première règle qui correspond à l’URL envoyée à la passerelle d’application. Par exemple, si une règle utilise un écouteur de base et qu’une autre utilise un écouteur multisite sur le même port, la règle avec l’écouteur multisite doit être répertoriée avant la règle avec l’écouteur de base pour que la règle multisite fonctionne comme prévu. 

Cet exemple crée deux règles et supprime la règle par défaut qui avait été créée.  Vous pouvez ajouter la règle à l’aide de la commande [az network application-gateway rule create](/cli/azure/network/application-gateway/rule#az-network-application-gateway-rule-create).

```azurecli-interactive
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name contosoComRule \
  --resource-group myResourceGroupAG \
  --http-listener contosoComListener \
  --rule-type Basic \
  --address-pool appGatewayBackendPool
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name contosoOrgRule \
  --resource-group myResourceGroupAG \
  --http-listener contosoOrgListener \
  --rule-type Basic \
  --redirect-config orgToCom
az network application-gateway rule delete \
  --gateway-name myAppGateway \
  --name rule1 \
  --resource-group myResourceGroupAG
```

## <a name="create-virtual-machine-scale-sets"></a>Créer des groupes de machines virtuelles identiques

Dans cet exemple, vous créez un groupe de machines virtuelles identiques qui prend en charge le pool backend que vous avez créé. Le groupe identique que vous créez est nommé *myvmss* et contient deux instances de machine virtuelle sur lesquelles NGINX est installé.

```azurecli-interactive
az vmss create \
  --name myvmss \
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
  --backend-pool-name appGatewayBackendPool
```

### <a name="install-nginx"></a>Installer NGINX

Exécutez la commande suivante dans la fenêtre de l’interpréteur de commandes :

```azurecli-interactive
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroupAG \
  --vmss-name myvmss \
  --settings '{ "fileUris": ["https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/install_nginx.sh"],
  "commandToExecute": "./install_nginx.sh" }'
```

## <a name="create-cname-record-in-your-domain"></a>Créer un enregistrement CNAME dans votre domaine

Une fois la passerelle d’application créée avec son adresse IP publique, vous pouvez obtenir l’adresse DNS et l’utiliser pour créer un enregistrement CNAME dans votre domaine. Vous pouvez utiliser la commande [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show) pour obtenir l’adresse DNS de la passerelle d’application. Copiez la valeur *fqdn* de DNSSettings et utilisez-la comme valeur de l’enregistrement CNAME que vous créez. L’utilisation d’enregistrements A n’est pas recommandée étant donné que l’adresse IP virtuelle peut changer au moment du redémarrage de la passerelle d’application.

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [dnsSettings.fqdn] \
  --output tsv
```

## <a name="test-the-application-gateway"></a>Tester la passerelle d’application

Entrez votre nom de domaine dans la barre d’adresse de votre navigateur. Par exemple : http:\//www.contoso.com.

![Tester le site contoso dans la passerelle d’application](./media/redirect-internal-site-cli/application-gateway-nginxtest.png)

Remplacez l’adresse par celle de votre autre domaine, par exemple http:\//www.contoso.org. Vous devriez alors remarquer que le trafic est redirigé vers l’écouteur pour www\.contoso.com.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> * Configurer le réseau
> * Créer une passerelle Application Gateway
> * Ajouter des écouteurs et une règle de redirection
> * Créer un groupe de machines virtuelles identiques avec le pool backend
> * Créer un enregistrement CNAME dans votre domaine

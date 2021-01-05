---
title: Héberger plusieurs sites web avec l’interface CLI
titleSuffix: Azure Application Gateway
description: Découvrez comment créer une passerelle d’application qui héberge plusieurs sites web à l’aide d’Azure CLI.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/13/2019
ms.author: victorh
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 350962aed89d04c5508e7b2c50e8a838cd5a7174
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94566144"
---
# <a name="create-an-application-gateway-that-hosts-multiple-web-sites-using-the-azure-cli"></a>Créer une passerelle d’application qui héberge plusieurs sites web à l’aide d’Azure CLI

Vous pouvez utiliser Azure CLI pour [configurer l’hébergement de plusieurs sites web](multiple-site-overview.md) quand vous créez une [passerelle d’application](overview.md). Dans cet article, vous allez définir des pools d’adresses backend à l’aide de groupes de machines virtuelles identiques. Vous configurez ensuite des écouteurs et des règles en fonction des domaines qui vous appartiennent pour vérifier que le trafic web arrive sur les serveurs appropriés dans les pools. Cet article, qui part du principe que vous possédez plusieurs domaines, utilise *www\.contoso.com* et *www\.fabrikam.com* en guise d’exemples.

Dans cet article, vous apprendrez comment :

* Configurer le réseau
* Créer une passerelle Application Gateway
* Créer des écouteurs principaux
* Créer des règles d’acheminement
* Créer des groupes de machines virtuelles identiques avec les pools principaux
* Créer un enregistrement CNAME dans votre domaine

:::image type="content" source="./media/tutorial-multiple-sites-cli/scenario.png" alt-text="Passerelle d’application multisite":::

Si vous préférez, vous pouvez suivre cette procédure en utilisant [Azure PowerShell](tutorial-multiple-sites-powershell.md).

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

Créez le réseau virtuel et le sous-réseau nommé *myAGSubnet* à l’aide de la commande [az network vnet create](/cli/azure/network/vnet). Vous pouvez ensuite ajouter le sous-réseau nécessaire aux serveurs backend à l’aide de la commande [az network vnet subnet create](/cli/azure/network/vnet/subnet). Créez l’adresse IP publique nommée *myAGPublicIPAddress* à l’aide de la commande [az network public-ip create](/cli/azure/network/public-ip).

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

## <a name="create-the-application-gateway"></a>Créer la passerelle Application Gateway

Vous pouvez utiliser la commande [az network application-gateway create](/cli/azure/network/application-gateway#az-network-application-gateway-create) pour créer la passerelle d’application. Quand vous créez une passerelle d’application avec Azure CLI, vous spécifiez des informations de configuration, telles que la capacité, la référence SKU et les paramètres HTTP. La passerelle d’application est affectée à *myAGSubnet* et à *myAGPublicIPAddress*, que vous avez créés. 

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

La création de la passerelle d’application peut prendre plusieurs minutes. Une fois la passerelle d’application créée, vous pouvez voir ses nouvelles fonctionnalités suivantes :

- *appGatewayBackendPool* : une passerelle d’application doit avoir au moins un pool d’adresses backend.
- *appGatewayBackendHttpSettings* : spécifie que le port 80 et le protocole HTTP sont utilisés pour la communication.
- *appGatewayHttpListener* : écouteur par défaut associé à *appGatewayBackendPool*.
- *appGatewayFrontendIP* - assigne *myAGPublicIPAddress* à *appGatewayHttpListener*.
- *rule1* : règle de routage par défaut associée à *appGatewayHttpListener*.

### <a name="add-the-backend-pools"></a>Ajouter les pools backend

Ajoutez les pools back-end qui doivent contenir les serveurs back-end à l’aide de la commande [az network application-gateway address-pool create](/cli/azure/network/application-gateway/address-pool#az-network-application-gateway-address-pool-create)
```azurecli-interactive
az network application-gateway address-pool create \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name contosoPool

az network application-gateway address-pool create \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name fabrikamPool
```

### <a name="add-listeners"></a>Ajouter des écouteurs

Ajoutez les écouteurs nécessaires pour acheminer le trafic à l’aide de la commande [az network application-gateway http-listener create](/cli/azure/network/application-gateway/http-listener#az-network-application-gateway-http-listener-create).

>[!NOTE]
> À l’aide du niveau tarifaire Application Gateway ou WAF v2, vous pouvez également configurer jusqu’à 5 noms d’hôte par écouteur et utiliser des caractères génériques dans le nom d’hôte. Pour plus d’informations, consultez les informations relatives aux [noms d’hôtes comportant des caractères génériques dans l’écouteur](multiple-site-overview.md#wildcard-host-names-in-listener-preview).
>Pour utiliser plusieurs noms d’hôte et caractères génériques dans un écouteur à l’aide d’Azure CLI, vous devez utiliser `--host-names` au lieu de `--host-name`. Grâce au préfixe « host-names », vous pouvez spécifier jusqu’à cinq noms d’hôte comme valeurs séparées par une espace. Par exemple : `--host-names "*.contoso.com *.fabrikam.com"`

```azurecli-interactive
az network application-gateway http-listener create \
  --name contosoListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port appGatewayFrontendPort \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway \
  --host-name www.contoso.com

az network application-gateway http-listener create \
  --name fabrikamListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port appGatewayFrontendPort \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway \
  --host-name www.fabrikam.com   
  ```

### <a name="add-routing-rules"></a>Ajouter des règles de routage

Les règles sont traitées dans par ordre de création. Le trafic est dirigé à l’aide de la première règle qui correspond, quelle que soit la particularité. Par exemple, si une règle utilise un écouteur de base et qu’une autre utilise un écouteur multisite sur le même port, la règle avec l’écouteur multisite doit être répertoriée avant la règle avec l’écouteur de base pour que la règle multisite fonctionne comme prévu. 

Dans cet exemple, vous créez deux nouvelles règles et supprimez la règle par défaut qui a été créée lorsque vous avez déployé la passerelle d’application. Vous pouvez ajouter la règle à l’aide de la commande [az network application-gateway rule create](/cli/azure/network/application-gateway/rule#az-network-application-gateway-rule-create).

```azurecli-interactive
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name contosoRule \
  --resource-group myResourceGroupAG \
  --http-listener contosoListener \
  --rule-type Basic \
  --address-pool contosoPool

az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name fabrikamRule \
  --resource-group myResourceGroupAG \
  --http-listener fabrikamListener \
  --rule-type Basic \
  --address-pool fabrikamPool

az network application-gateway rule delete \
  --gateway-name myAppGateway \
  --name rule1 \
  --resource-group myResourceGroupAG
```

## <a name="create-virtual-machine-scale-sets"></a>Créer des groupes de machines virtuelles identiques

Cet exemple crée trois groupes de machines virtuelles identiques prenant en charge les trois pools backend dans la passerelle d’application. Ils sont nommés *myvmss1*, *myvmss2* et *myvmss3*. Chacun contient deux instances de machines virtuelles sur lesquelles IIS sera installé.

```azurecli-interactive
for i in `seq 1 2`; do

  if [ $i -eq 1 ]
  then
    poolName="contosoPool"
  fi

  if [ $i -eq 2 ]
  then
    poolName="fabrikamPool"
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
for i in `seq 1 2`; do

  az vmss extension set \
    --publisher Microsoft.Azure.Extensions \
    --version 2.0 \
    --name CustomScript \
    --resource-group myResourceGroupAG \
    --vmss-name myvmss$i \
    --settings '{ "fileUris": ["https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/install_nginx.sh"],
  "commandToExecute": "./install_nginx.sh" }'

done
```

## <a name="create-a-cname-record-in-your-domain"></a>Créer un enregistrement CNAME dans votre domaine

Une fois la passerelle d’application créée avec son adresse IP publique, vous pouvez obtenir l’adresse DNS et l’utiliser pour créer un enregistrement CNAME dans votre domaine. Vous pouvez utiliser la commande [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show) pour obtenir l’adresse DNS de la passerelle d’application. Copiez la valeur *fqdn* de DNSSettings et utilisez-la comme valeur de l’enregistrement CNAME que vous créez. 

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [dnsSettings.fqdn] \
  --output tsv
```

L’utilisation d’enregistrements A n’est pas recommandée puisque l’adresse IP virtuelle peut changer lors du redémarrage de la passerelle d’application.

## <a name="test-the-application-gateway"></a>Tester la passerelle d’application

Entrez votre nom de domaine dans la barre d’adresse de votre navigateur. Par exemple : http:\//www.contoso.com.

![Tester le site contoso dans la passerelle d’application](./media/tutorial-multiple-sites-cli/application-gateway-nginxtest1.png)

Remplacez l’adresse par celle de votre autre domaine. Voici ce qui doit apparaître :

![Tester le site fabrikam dans la passerelle d’application](./media/tutorial-multiple-sites-cli/application-gateway-nginxtest2.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources, la passerelle d’application et toutes les ressources associées.

```azurecli-interactive
az group delete --name myResourceGroupAG
```

## <a name="next-steps"></a>Étapes suivantes

[Créer une passerelle d’application avec des règles d’acheminement par chemin d’URL](./tutorial-url-route-cli.md)

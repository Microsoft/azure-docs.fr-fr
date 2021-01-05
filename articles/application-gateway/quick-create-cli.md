---
title: 'Démarrage rapide : Diriger le trafic web à l’aide d’une interface de ligne de commande (CLI)'
titleSuffix: Azure Application Gateway
description: Dans ce guide de démarrage rapide, découvrez comment utiliser Azure CLI pour créer une passerelle d’application Azure qui dirige le trafic web vers les machines virtuelles d’un pool de back-ends.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: quickstart
ms.date: 08/27/2020
ms.author: victorh
ms.custom: mvc, devx-track-js, devx-track-azurecli
ms.openlocfilehash: d4b6fc296ae41667d271e243e9aca9b594e4a5b6
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94886700"
---
# <a name="quickstart-direct-web-traffic-with-azure-application-gateway---azure-cli"></a>Démarrage rapide : Diriger le trafic web avec Azure Application Gateway - Azure CLI

Dans le cadre de ce guide de démarrage rapide, vous allez utiliser Azure CLI pour créer une passerelle d’application. Puis, vous la testerez pour vous assurer qu’elle fonctionne correctement. 

La passerelle d’application dirige le trafic web des applications vers des ressources spécifiques d’un pool de back-ends. Vous attribuez des écouteurs aux ports, créez des règles et ajoutez des ressources à un pool de back-ends. Par souci de simplicité, cet article utilise une configuration simple avec une adresse IP front-end publique, un écouteur de base pour héberger un site unique sur cette passerelle d’application, une règle de routage des requêtes simple et deux machines virtuelles dans le pool de back-ends.

Vous pouvez également suivre ce guide de démarrage rapide en utilisant [Azure PowerShell](quick-create-powershell.md) ou le [portail Azure](quick-create-portal.md).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article demande la version 2.0.4 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-resource-group"></a>Créer un groupe de ressources

Dans Azure, vous allouez les ressources associées à un groupe de ressources. Créez un groupe de ressources avec `az group create`. 

L’exemple suivant crée un groupe de ressources nommé *myResourceGroupAG* à l’emplacement *eastus*.

```azurecli-interactive 
az group create --name myResourceGroupAG --location eastus
```

## <a name="create-network-resources"></a>Créer des ressources réseau 

Azure a besoin d’un réseau virtuel pour communiquer avec les différentes ressources que vous créez.  Le sous-réseau de passerelle d’application peut contenir uniquement des passerelles d’application. Aucune autre ressource n’est autorisée.  Vous pouvez créer un sous-réseau pour Application Gateway ou en utiliser qui existe déjà. Dans cet exemple, vous créez deux sous-réseaux : un pour la passerelle d’application et un autre pour les serveurs back-end. Vous pouvez l’adresse IP frontale d’Application Gateway pour qu’elle soit publique ou privée conformément à votre cas d’utilisation. Dans cet exemple, vous allez choisir une adresse IP front-end publique.

Pour créer le réseau virtuel et le sous-réseau, utilisez `az network vnet create`. Exécutez `az network public-ip create` pour créer l’adresse IP publique.

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
  --vnet-name myVNet   \
  --address-prefix 10.0.2.0/24
az network public-ip create \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --allocation-method Static \
  --sku Standard
```

## <a name="create-the-backend-servers"></a>Créer les serveurs back-end

Un back-end peut être composé de cartes d’interface réseau, de groupes de machines virtuelles identiques, d’adresses IP publiques, d’adresses IP internes, de noms de domaine complets et de serveurs back-end multilocataires comme Azure App Service. Dans cet exemple, vous allez créer deux machines virtuelles à utiliser comme serveurs back-end pour la passerelle d’application. Vous installerez également IIS sur les machines virtuelles pour tester la passerelle d’application.

#### <a name="create-two-virtual-machines"></a>Créer deux machines virtuelles

Installez le serveur web NGINX sur les machines virtuelles pour vérifier que la passerelle d’application a bien été créée. Vous pouvez utiliser un fichier de configuration cloud-init pour installer NGINX et exécuter une application Node.js « Hello World » sur une machine virtuelle Linux. Pour plus d’informations sur cloud-init, consultez [Prise en charge de cloud-init pour les machines virtuelles dans Azure](../virtual-machines/linux/using-cloud-init.md).

Dans Azure Cloud Shell, copiez et collez la configuration suivante dans un fichier appelé *cloud-init.txt*. Entrez *editor cloud-init.txt* pour créer le fichier.

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
  - nodejs
  - npm
write_files:
  - owner: www-data:www-data
  - path: /etc/nginx/sites-available/default
    content: |
      server {
        listen 80;
        location / {
          proxy_pass http://localhost:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection keep-alive;
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
        }
      }
  - owner: azureuser:azureuser
  - path: /home/azureuser/myapp/index.js
    content: |
      var express = require('express')
      var app = express()
      var os = require('os');
      app.get('/', function (req, res) {
        res.send('Hello World from host ' + os.hostname() + '!')
      })
      app.listen(3000, function () {
        console.log('Hello world app listening on port 3000!')
      })
runcmd:
  - service nginx restart
  - cd "/home/azureuser/myapp"
  - npm init
  - npm install express -y
  - nodejs index.js
```

Créez les interfaces réseau avec `az network nic create`. Pour créer les machines virtuelles, vous utilisez `az vm create`.

```azurecli-interactive
for i in `seq 1 2`; do
  az network nic create \
    --resource-group myResourceGroupAG \
    --name myNic$i \
    --vnet-name myVNet \
    --subnet myBackendSubnet
  az vm create \
    --resource-group myResourceGroupAG \
    --name myVM$i \
    --nics myNic$i \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init.txt
done
```

## <a name="create-the-application-gateway"></a>Créer la passerelle Application Gateway

Créez une passerelle d’application en utilisant `az network application-gateway create`. Quand vous créez une passerelle d’application avec Azure CLI, vous spécifiez des informations de configuration, telles que la capacité, la référence SKU et les paramètres HTTP. Azure ajoute ensuite les adresses IP privées des interfaces réseau en tant que serveurs dans le pool de back-ends de la passerelle d’application.

```azurecli-interactive
address1=$(az network nic show --name myNic1 --resource-group myResourceGroupAG | grep "\"privateIpAddress\":" | grep -oE '[^ ]+$' | tr -d '",')
address2=$(az network nic show --name myNic2 --resource-group myResourceGroupAG | grep "\"privateIpAddress\":" | grep -oE '[^ ]+$' | tr -d '",')
az network application-gateway create \
  --name myAppGateway \
  --location eastus \
  --resource-group myResourceGroupAG \
  --capacity 2 \
  --sku Standard_v2 \
  --http-settings-cookie-based-affinity Enabled \
  --public-ip-address myAGPublicIPAddress \
  --vnet-name myVNet \
  --subnet myAGSubnet \
  --servers "$address1" "$address2"
```

La création de la passerelle d’application par Azure peut prendre jusqu’à 30 minutes. Une fois la passerelle d’application créée, vous pouvez afficher les paramètres suivants dans la section **Paramètres** de la page **Application Gateway** :

- **appGatewayBackendPool** : situé dans la page **Pools principaux**, ce paramètre spécifie le pool de back-ends nécessaire.
- **appGatewayBackendHttpSettings** : situé dans la page **Paramètres HTTP**, ce paramètre spécifie que la passerelle d’application utilise le port 80 et le protocole HTTP pour la communication.
- **appGatewayHttpListener** : situé dans la page **Écouteurs**, ce paramètre spécifie l’écouteur par défaut associé à **appGatewayBackendPool**.
- **appGatewayFrontendIP** : situé dans la page **Configurations d’adresses IP frontales**, ce paramètre assigne *myAGPublicIPAddress* à **appGatewayHttpListener**.
- **rule1** : situé dans la page **Règles**, ce paramètre spécifie la règle de routage par défaut associée à **appGatewayHttpListener**.

## <a name="test-the-application-gateway"></a>Tester la passerelle d’application

Azure ne nécessite pas de serveur web NGINX pour créer la passerelle d’application, mais vous avez installé ce serveur dans ce guide de démarrage rapide pour vérifier qu’Azure avait bien créé la passerelle d’application. Pour récupérer l’adresse IP publique de la nouvelle passerelle d’application, utilisez `az network public-ip show`. 

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [ipAddress] \
  --output tsv
```

Copiez et collez l’adresse IP publique dans la barre d’adresse de votre navigateur.
    
![Tester la passerelle d’application](./media/quick-create-cli/application-gateway-nginxtest.png)

Quand vous actualisez le navigateur, vous devez voir s’afficher le nom de la deuxième machine virtuelle. Cela indique que la passerelle d’application a été correctement créée et peut se connecter au back-end.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin des ressources que vous avez créées avec la passerelle d’application, utilisez la commande `az group delete` pour supprimer le groupe de ressources. Quand vous supprimez le groupe de ressources, vous supprimez aussi la passerelle d’application et toutes ses ressources associées.

```azurecli-interactive 
az group delete --name myResourceGroupAG
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Gérer le trafic web avec une passerelle d’application à l’aide d’Azure CLI](./tutorial-manage-web-traffic-cli.md)


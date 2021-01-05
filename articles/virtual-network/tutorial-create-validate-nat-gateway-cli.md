---
title: Créer et tester une passerelle NAT - Azure CLI
titlesuffix: Azure Virtual Network NAT
description: Ce tutoriel montre comment créer une passerelle NAT avec l’interface Azure CLI, et tester le service NAT.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
Customer intent: I want to test a NAT gateway for outbound connectivity for my virtual network.
ms.service: virtual-network
ms.subservice: nat
ms.devlang: na
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 06/11/2020
ms.author: allensu
ms.custom: devx-track-azurecli
ms.openlocfilehash: 0ec054d55432ad2680314b4ff91a067d37b629d4
ms.sourcegitcommit: c2dd51aeaec24cd18f2e4e77d268de5bcc89e4a7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94734325"
---
# <a name="tutorial-create-a-nat-gateway-using-azure-cli-and-test-the-nat-service"></a>Tutoriel : Créer une passerelle NAT avec Azure CLI et tester le service NAT

Dans ce tutoriel, vous allez créer une passerelle NAT pour fournir une connectivité sortante à des machines virtuelles dans Azure. Pour tester la passerelle NAT, vous déployez une machine virtuelle source et une machine virtuelle de destination. Vous allez tester la passerelle NAT en établissant des connexions sortantes à une adresse IP publique. Ces connexions iront de la machine virtuelle source vers la machine de destination. Ce tutoriel déploie la source et la destination sur deux réseaux virtuels différents dans le même groupe de ressources dans l’unique but de rester simple.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite l’interface Azure CLI version 2.0.71 ou ultérieure. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources avec la commande [az group create](https://docs.microsoft.com/cli/azure/group). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.

L’exemple suivant crée un groupe de ressources nommé **myResourceGroupNAT** à l’emplacement **eastus2** :

```azurecli-interactive
  az group create \
    --name myResourceGroupNAT \
    --location eastus2
    
```

## <a name="create-the-nat-gateway"></a>Créer la passerelle NAT

### <a name="create-a-public-ip-address"></a>Créer une adresse IP publique

Pour accéder à l’Internet public, vous avez besoin d’une ou de plusieurs adresses IP publiques pour la passerelle NAT. Utilisez la commande [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip) pour créer une ressource d’adresse IP publique nommée **myPublicIPsource** dans **myResourceGroupNAT**.

```azurecli-interactive
  az network public-ip create \
  --resource-group myResourceGroupNAT \
  --name myPublicIPsource \
  --sku standard
  
```

### <a name="create-a-public-ip-prefix"></a>Créer un préfixe d’adresse IP publique

Vous pouvez utiliser une ou plusieurs ressources d’adresse IP publique, des préfixes d’adresse IP publique, ou ces deux options à la fois avec la passerelle NAT. Nous allons ajouter une ressource de préfixe d’adresse IP publique à ce scénario pour le démontrer.   Utilisez la commande [az network public-ip prefix create](/cli/azure/network/public-ip/prefix?view=azure-cli-latest#az-network-public-ip-prefix-create) pour créer une ressource de préfixe d’adresse IP publique nommée **myPublicIPprefixsource** dans **myResourceGroupNAT**.

```azurecli-interactive
  az network public-ip prefix create \
  --resource-group myResourceGroupNAT \
  --name myPublicIPprefixsource \
  --length 31
  
```

### <a name="create-a-nat-gateway-resource"></a>Créer une ressource de passerelle NAT

Cette section explique en détail comment vous pouvez créer et configurer les composants suivants du service NAT à l’aide de la ressource de passerelle NAT :
  - Un pool d’adresses IP publiques et un préfixe d’adresse IP publique à utiliser pour les flux sortants qui sont traduits par la ressource de passerelle NAT.
  - Passer la valeur par défaut du délai d’inactivité de 4 minutes à 10 minutes.

Créez une passerelle NAT Azure globale avec la commande [az network nat gateway create](https://docs.microsoft.com/cli/azure/network/nat?view=azure-cli-latest) nommée **myNATgateway**. La commande utilise à la fois l’adresse IP publique **myPublicIP** et le préfixe d’adresse IP publique **myPublicIPprefix**. La commande passe également le délai d’inactivité à 10 minutes.

```azurecli-interactive
  az network nat gateway create \
    --resource-group myResourceGroupNAT \
    --name myNATgateway \
    --public-ip-addresses myPublicIPsource \
    --public-ip-prefixes myPublicIPprefixsource \
    --idle-timeout 10       
    
  ```

À ce stade, la passerelle NAT est fonctionnelle et il ne reste plus qu’à définir quels sous-réseaux d’un réseau virtuel doivent l’utiliser.

## <a name="prepare-the-source-for-outbound-traffic"></a>Préparer la source du trafic sortant

Nous allons vous guider tout au long de la configuration d’un environnement de test complet. Vous allez configurer un test à l’aide d’outils open source pour vérifier la passerelle NAT. Nous allons commencer par la source, qui utilisera la passerelle NAT que nous avons créée précédemment.

### <a name="configure-virtual-network-for-source"></a>Configurer un réseau virtuel pour la source

Avant de déployer une machine virtuelle et de pouvoir tester votre passerelle NAT, il nous faut créer le réseau virtuel.

Créez un réseau virtuel nommé **myVnetsource** avec un sous-réseau nommé **mySubnetsource** dans **myResourceGroupNAT** à l’aide de la commande [az network Microsoft Azure Virtual Network create](https://docs.microsoft.com/cli/azure/network/vnet).  L’espace d’adressage IP pour le réseau virtuel est **192.168.0.0/16**. Le sous-réseau au sein du réseau virtuel est **192.168.0.0/24**.

```azurecli-interactive
  az network vnet create \
    --resource-group myResourceGroupNAT \
    --name myVnetsource \
    --address-prefix 192.168.0.0/16 \
    --subnet-name mySubnetsource \
    --subnet-prefix 192.168.0.0/24
    
```

### <a name="configure-nat-service-for-source-subnet"></a>Configurer le service NAT pour le sous-réseau source

Configurez le sous-réseau source **mySubnetsource** dans le réseau virtuel **myVnetsource** pour utiliser une ressource de passerelle NAT spécifique **myNATgateway** avec la commande [az network Microsoft Azure Virtual Network subnet update](https://docs.microsoft.com/cli/azure/network/vnet/subnet). Cette commande active le service NAT sur le sous-réseau précisé.

```azurecli-interactive
    az network vnet subnet update \
    --resource-group myResourceGroupNAT \
    --vnet-name myVnetsource \
    --name mySubnetsource \
    --nat-gateway myNATgateway
    
```

Tout le trafic sortant vers les destinations Internet utilise désormais le service NAT.  Il n’est pas nécessaire de configurer un UDR.

Avant de pouvoir tester la passerelle NAT, nous devons créer une machine virtuelle source.  Nous allons attribuer une ressource d’adresse IP publique en tant qu’adresse IP publique au niveau de l’instance pour accéder à cette machine virtuelle de l’extérieur. Cette adresse est uniquement utilisée pour y accéder dans le cadre du test.  Nous allons vous montrer comment le service NAT est prioritaire sur les autres options sortantes.

Vous pouvez également créer cette machine virtuelle sans adresse IP publique, et créer une autre machine virtuelle à utiliser comme jumpbox sans adresse IP publique dans le cadre d’un exercice.

### <a name="create-public-ip-for-source-vm"></a>Créer une adresse IP publique pour la machine virtuelle source

Nous créons une adresse IP publique à utiliser pour accéder à la machine virtuelle source. Utilisez la commande [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip) pour créer une ressource d’adresse IP publique nommée **myPublicIPsourceVM** dans **myResourceGroupNAT**.

```azurecli-interactive
  az network public-ip create \
    --resource-group myResourceGroupNAT \
    --name myPublicIPsourceVM \
    --sku standard
    
```

### <a name="create-an-nsg-for-source-vm"></a>Créer un groupe de sécurité réseau pour une machine virtuelle source

Les adresses IP publiques standard étant « sécurisées par défaut », nous devons créer un groupe de sécurité réseau pour autoriser l’accès entrant à l’accès SSH.  Le service NAT Azure reconnaît la direction du flux. Ce groupe de sécurité réseau n’est pas utilisé pour le trafic sortant dès lors que la passerelle NAT est configurée sur le même sous-réseau. Utilisez [az network nsg create](https://docs.microsoft.com/cli/azure/network/nsg?view=azure-cli-latest#az-network-nsg-create) pour créer une ressource de groupe de sécurité réseau nommée **myNSGsource** dans **myResourceGroupNAT**.

```azurecli-interactive
  az network nsg create \
    --resource-group myResourceGroupNAT \
    --name myNSGsource 
    
```

### <a name="expose-ssh-endpoint-on-source-vm"></a>Exposer le point de terminaison SSH sur la machine virtuelle source

Nous créons une règle dans le groupe de sécurité réseau pour l’accès SSH à la machine virtuelle source. Utilisez [az network nsg rule create](https://docs.microsoft.com/cli/azure/network/nsg/rule?view=azure-cli-latest#az-network-nsg-rule-create) pour créer une règle NSG nommée **ssh**. Cette règle est créée dans le groupe de sécurité réseau nommé **myNSGsource**, dans le groupe de ressources **myResourceGroupNAT**.

```azurecli-interactive
  az network nsg rule create \
    --resource-group myResourceGroupNAT \
    --nsg-name myNSGsource \
    --priority 100 \
    --name ssh \
    --description "SSH access" \
    --access allow \
    --protocol tcp \
    --direction inbound \
    --destination-port-ranges 22
    
```

### <a name="create-nic-for-source-vm"></a>Créer une carte réseau pour la machine virtuelle source

Créez une interface réseau à l’aide de la commande [az network nic create](/cli/azure/network/nic#az-network-nic-create) et associez-la à l’adresse IP publique et au groupe de sécurité réseau. 

```azurecli-interactive
  az network nic create \
    --resource-group myResourceGroupNAT \
    --name myNicsource \
    --vnet-name myVnetsource \
    --subnet mySubnetsource \
    --public-ip-address myPublicIPSourceVM \
    --network-security-group myNSGsource
    
```

### <a name="create-a-source-vm"></a>Créer une machine virtuelle source

Créez la machine virtuelle avec la commande [az vm create](/cli/azure/vm#az-vm-create).  Nous générons des clés SSH pour cette machine virtuelle, et stockons la clé privée à utiliser ultérieurement.

```azurecli-interactive
  az vm create \
    --resource-group myResourceGroupNAT \
    --name myVMsource \
    --nics myNicsource \
    --image UbuntuLTS \
    --generate-ssh-keys \
    --no-wait
    
```

Bien que la commande soit retournée immédiatement, le déploiement de la machine virtuelle peut prendre quelques minutes.

## <a name="prepare-destination-for-outbound-traffic"></a>Préparer la destination pour le trafic sortant

Nous allons maintenant créer une destination pour le trafic sortant qui est traduit par le service NAT, et ainsi vous permettre de le tester.

### <a name="configure-virtual-network-for-destination"></a>Configurer un réseau virtuel pour la destination

 Nous devons créer un réseau virtuel sur lequel se trouvera la machine virtuelle de destination.  Ces commandes sont les mêmes que celles des étapes de la machine virtuelle source, avec des modifications mineures pour exposer le point de terminaison de destination.

Créez un réseau virtuel nommé **myVnetdestination** avec un sous-réseau nommé **mySubnetdestination** dans **myResourceGroupNAT** à l’aide de la commande [az network Microsoft Azure Virtual Network create](https://docs.microsoft.com/cli/azure/network/vnet).  L’espace d’adressage IP pour le réseau virtuel est **192.168.0.0/16**. Le sous-réseau au sein du réseau virtuel est **192.168.0.0/24**.

```azurecli-interactive
  az network vnet create \
    --resource-group myResourceGroupNAT \
    --name myVnetdestination \
    --address-prefix 192.168.0.0/16 \
    --subnet-name mySubnetdestination \
    --subnet-prefix 192.168.0.0/24
    
```

### <a name="create-public-ip-for-destination-vm"></a>Créer une adresse IP publique pour la machine virtuelle de destination

Nous créons une adresse IP publique à utiliser pour accéder à la machine virtuelle source. Utilisez la commande [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip) pour créer une ressource d’adresse IP publique nommée **myPublicIPdestinationVM** dans **myResourceGroupNAT**. 

```azurecli-interactive
  az network public-ip create \
  --resource-group myResourceGroupNAT \
  --name myPublicIPdestinationVM \
  --sku standard
  
```

### <a name="create-an-nsg-for-destination-vm"></a>Créer un groupe de sécurité réseau pour la machine virtuelle de destination

Les adresses IP publiques standard étant « sécurisées par défaut », nous devons créer un groupe de sécurité réseau pour autoriser SSH à l’accès entrant. Le service NAT Azure reconnaît la direction du flux. Ce groupe de sécurité réseau n’est pas utilisé pour le trafic sortant dès lors que la passerelle NAT est configurée sur le même sous-réseau. Utilisez [az network nsg create](https://docs.microsoft.com/cli/azure/network/nsg?view=azure-cli-latest#az-network-nsg-create) pour créer une ressource de groupe de sécurité réseau nommée **myNSGdestination** dans **myResourceGroupNAT**.

```azurecli-interactive
    az network nsg create \
    --resource-group myResourceGroupNAT \
    --name myNSGdestination
    
```

### <a name="expose-ssh-endpoint-on-destination-vm"></a>Exposer le point de terminaison SSH sur la machine virtuelle de destination

Nous créons une règle dans le groupe de sécurité réseau pour l’accès SSH à la machine virtuelle de destination. Utilisez [az network nsg rule create](https://docs.microsoft.com/cli/azure/network/nsg/rule?view=azure-cli-latest#az-network-nsg-rule-create) pour créer une règle NSG nommée **ssh**. Cette règle est créée dans le groupe de sécurité réseau nommé **myNSGdestination**, dans le groupe de ressources **myResourceGroupNAT**.

```azurecli-interactive
    az network nsg rule create \
    --resource-group myResourceGroupNAT \
    --nsg-name myNSGdestination \
    --priority 100 \
    --name ssh \
    --description "SSH access" \
    --access allow \
    --protocol tcp \
    --direction inbound \
    --destination-port-ranges 22
    
```

### <a name="expose-http-endpoint-on-destination-vm"></a>Exposer le point de terminaison HTTP sur la machine virtuelle de destination

Nous créons une règle dans le groupe de sécurité réseau pour l’accès HTTP à la machine virtuelle de destination. Utilisez la commande [az network nsg rule create](https://docs.microsoft.com/cli/azure/network/nsg/rule?view=azure-cli-latest#az-network-nsg-rule-create) pour créer une règle de groupe de sécurité réseau nommée **http** dans le groupe de sécurité réseau **myNSGdestination** de **myResourceGroupNAT**.

```azurecli-interactive
    az network nsg rule create \
    --resource-group myResourceGroupNAT \
    --nsg-name myNSGdestination \
    --priority 101 \
    --name http \
    --description "HTTP access" \
    --access allow \
    --protocol tcp \
    --direction inbound \
    --destination-port-ranges 80
    
```

### <a name="create-nic-for-destination-vm"></a>Créer une carte réseau pour la machine virtuelle de destination

Créez une interface réseau à l’aide de la commande [az network nic create](/cli/azure/network/nic#az-network-nic-create) et associez-la à l’adresse IP publique **myPublicIPdestinationVM** et au groupe de sécurité réseau **myNSGdestination**. 

```azurecli-interactive
    az network nic create \
    --resource-group myResourceGroupNAT \
    --name myNicdestination \
    --vnet-name myVnetdestination \
    --subnet mySubnetdestination \
    --public-ip-address myPublicIPdestinationVM \
    --network-security-group myNSGdestination
    
```

### <a name="create-a-destination-vm"></a>Créer une machine virtuelle de destination

Créez la machine virtuelle avec la commande [az vm create](/cli/azure/vm#az-vm-create).  Nous générons des clés SSH pour cette machine virtuelle, et stockons la clé privée à utiliser ultérieurement.

 ```azurecli-interactive
    az vm create \
    --resource-group myResourceGroupNAT \
    --name myVMdestination \
    --nics myNicdestination \
    --image UbuntuLTS \
    --generate-ssh-keys \
    --no-wait
    
```
Bien que la commande soit retournée immédiatement, le déploiement de la machine virtuelle peut prendre quelques minutes.

## <a name="prepare-a-web-server-and-test-payload-on-destination-vm"></a>Préparer un serveur web et une charge utile de test sur la machine virtuelle de destination

Tout d’abord, nous avons besoin de découvrir l’adresse IP de la machine virtuelle de destination.  Pour obtenir l’adresse IP publique de la machine virtuelle de destination, utilisez la commande [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show). 

```azurecli-interactive
  az network public-ip show \
    --resource-group myResourceGroupNAT \
    --name myPublicIPdestinationVM \
    --query [ipAddress] \
    --output tsv
    
``` 

>[!IMPORTANT]
>Copiez l’adresse IP publique, puis collez-la dans un bloc-notes afin de pouvoir l’utiliser dans les étapes suivantes. Indiquez qu’il s’agit de la machine virtuelle de destination.

### <a name="sign-in-to-destination-vm"></a>Se connecter à la machine virtuelle de destination

Les informations d’identification SSH doivent être stockées dans votre instance Cloud Shell depuis l’opération précédente.  Ouvrez une session [Azure Cloud Shell](https://shell.azure.com) dans votre navigateur. Utilisez l’adresse IP récupérée à l’étape précédente pour établir une connexion SSH à la machine virtuelle. 

```bash
ssh <ip-address-destination>
```

Copiez et collez les commandes suivantes une fois que vous êtes connecté.  

```bash
sudo apt -y update && \
sudo apt -y upgrade && \
sudo apt -y install nginx && \
sudo ln -sf /dev/null /var/log/nginx/access.log && \
sudo touch /var/www/html/index.html && \
sudo rm /var/www/html/index.nginx-debian.html && \
sudo dd if=/dev/zero of=/var/www/html/100k bs=1024 count=100

```

Ces commandes mettent à jour votre machine virtuelle, installent Nginx et créent un fichier de 100 Ko. Ce fichier sera récupéré à partir de la machine virtuelle source par le biais du service NAT.

Fermez la session SSH avec la machine virtuelle de destination.

## <a name="prepare-test-on-source-vm"></a>Préparer le test sur la machine virtuelle source

Tout d’abord, nous avons besoin de découvrir l’adresse IP de la machine virtuelle source.  Pour obtenir l’adresse IP publique de la machine virtuelle source, utilisez la commande [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show). 

```azurecli-interactive
  az network public-ip show \
    --resource-group myResourceGroupNAT \
    --name myPublicIPsourceVM \
    --query [ipAddress] \
    --output tsv
    
``` 

>[!IMPORTANT]
>Copiez l’adresse IP publique, puis collez-la dans un bloc-notes afin de pouvoir l’utiliser dans les étapes suivantes. Indiquez qu’il s’agit de la machine virtuelle source.

### <a name="sign-in-to-source-vm"></a>Se connecter à la machine virtuelle source

Là encore, les informations d’identification SSH sont stockées dans Cloud Shell. Ouvrez un nouvel onglet pour [Azure Cloud Shell](https://shell.azure.com) dans votre navigateur.  Utilisez l’adresse IP récupérée à l’étape précédente pour établir une connexion SSH à la machine virtuelle. 

```bash
ssh <ip-address-source>
```

Copiez et collez les commandes suivantes pour préparer le test du service NAT.

```bash
sudo apt -y update && \
sudo apt -y upgrade && \
sudo apt install -y nload golang && \
echo 'export GOPATH=${HOME}/go' >> .bashrc && \
echo 'export PATH=${PATH}:${GOPATH}/bin' >> .bashrc && \
. ~/.bashrc &&
go get -u github.com/rakyll/hey

```

Cette commande met à jour votre machine virtuelle, installe Go, installe [hey](https://github.com/rakyll/hey) à partir de GitHub, et met à jour votre environnement d’interpréteur de commandes.

Vous êtes désormais prêt à tester le service NAT.

## <a name="validate-nat-service"></a>Valider le service NAT

Quand vous êtes connecté à la machine virtuelle source, vous pouvez utiliser **curl** et **hey** pour générer des requêtes vers l’adresse IP de destination.

Utilisez curl pour récupérer le fichier de 100 Ko.  Dans l’exemple ci-dessous, remplacez **\<ip-address-destination>** par l’adresse IP de destination que vous avez copiée précédemment.  Le paramètre **--output** indique que le fichier récupéré sera ignoré.

```bash
curl http://<ip-address-destination>/100k --output /dev/null
```

Vous pouvez également générer une série de requêtes à l’aide de **hey**. Là encore, remplacez **\<ip-address-destination>** par l’adresse IP de destination que vous avez copiée précédemment.

```bash
hey -n 100 -c 10 -t 30 --disable-keepalive http://<ip-address-destination>/100k
```

Cette commande génère 100 requêtes, 10 simultanément, avec un délai d’expiration de 30 secondes. La connexion TCP ne sera pas réutilisée.  Chaque demande récupère 100 Ko.  À la fin de l’exécution, **hey** fournit quelques statistiques sur le fonctionnement du service NAT.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, utilisez la commande [az group delete](/cli/azure/group#az-group-delete) pour supprimer le groupe de ressources et toutes les ressources qu’il contient.

```azurecli-interactive 
  az group delete --name myResourceGroupNAT
  
```

## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez créé une passerelle NAT, créé une machine virtuelle source et une machine virtuelle de destination, puis testé la passerelle NAT.

Consultez les métriques dans Azure Monitor pour découvrir le fonctionnement de votre service NAT. Diagnostiquez des problèmes, tels que l’épuisement des ressources des ports SNAT disponibles.  L’épuisement des ressources de ports SNAT est résolu facilement en ajoutant des ressources supplémentaires, d’adresse IP publique ou de préfixe d’adresse IP publique, ou des deux à la fois.

- Apprenez-en davantage sur le service [Nat de Réseau virtuel](./nat-overview.md).
- Apprenez-en davantage sur la [ressource de passerelle NAT](./nat-gateway-resource.md).
- Guide de démarrage rapide du déploiement d’une [ressource de passerelle NAT avec Azure CLI](./quickstart-create-nat-gateway-cli.md).
- Guide de démarrage rapide du déploiement d’une [ressource de passerelle NAT avec Azure PowerShell](./quickstart-create-nat-gateway-powershell.md).
- Guide de démarrage rapide du déploiement d’une [ressource de passerelle NAT avec le portail Azure](./quickstart-create-nat-gateway-portal.md).

> [!div class="nextstepaction"]


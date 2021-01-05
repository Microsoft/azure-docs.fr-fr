---
title: 'Démarrage rapide : Créer un équilibreur de charge interne - Azure CLI'
titleSuffix: Azure Load Balancer
description: Ce guide de démarrage rapide montre comment créer un équilibreur de charge interne en utilisant Azure CLI.
services: load-balancer
documentationcenter: na
author: asudbring
manager: KumudD
Customer intent: I want to create a load balancer so that I can load balance internal traffic to VMs.
ms.service: load-balancer
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/23/2020
ms.author: allensu
ms.custom: mvc, devx-track-js, devx-track-azurecli
ms.openlocfilehash: 834b5c3651a7fff085dc53096f66d5e3f4bf27b4
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94700407"
---
# <a name="quickstart-create-an-internal-load-balancer-to-load-balance-vms-using-azure-cli"></a>Démarrage rapide : Créer un équilibreur de charge interne pour équilibrer la charge de machines virtuelles avec Azure CLI

Commencez avec Azure Load Balancer en utilisant le Azure CLI pour créer un équilibreur de charge public et trois machines virtuelles.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)] 

- Ce guide de démarrage rapide nécessite la version 2.0.28 ou ultérieure de l’interface Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group?view=azure-cli-latest#az-group-create) :

* Nommé **CreateIntLBQS-rg**. 
* Dans l’emplacement **eastus**.

```azurecli-interactive
  az group create \
    --name CreateIntLBQS-rg \
    --location eastus
```
---

# <a name="standard-sku"></a>[**Référence Standard**](#tab/option-1-create-load-balancer-standard)

>[!NOTE]
>Il est recommandé de disposer d’un équilibreur de charge de référence SKU Standard pour les charges de travail de production. Pour plus d’informations sur les références (SKU), consultez **[Références SKU Azure Load Balancer](skus.md)** .

## <a name="configure-virtual-network"></a>Configurer un réseau virtuel

Avant de déployer des machines virtuelles et votre équilibreur de charge, créez les ressources de réseau virtuel associées.

### <a name="create-a-virtual-network"></a>Créez un réseau virtuel

Créez un réseau virtuel avec la commande [az network vnet create](/cli/azure/network/vnet?view=azure-cli-latest#az-network-vnet-createt) :

* Nommé **myVNet**.
* Préfixe d’adresse **10.1.0.0/16**.
* Sous-réseau nommé **myBackendSubnet**.
* Préfixe du sous-réseau **10.1.0.0/24**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.
* Emplacement **eastus**.

```azurecli-interactive
  az network vnet create \
    --resource-group CreateIntLBQS-rg \
    --location eastus \
    --name myVNet \
    --address-prefixes 10.1.0.0/16 \
    --subnet-name myBackendSubnet \
    --subnet-prefixes 10.1.0.0/24
```
### <a name="create-a-network-security-group"></a>Créer un groupe de sécurité réseau

Pour un équilibreur de charge standard, les machines virtuelles à l’adresse principale doivent disposer d’interfaces réseau appartenant à un groupe de sécurité réseau. 

Créez un Groupe de sécurité réseau avec la commande [az network nsg create](/cli/azure/network/nsg?view=azure-cli-latest#az-network-nsg-create) :

* Nommé **myNSG**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.

```azurecli-interactive
  az network nsg create \
    --resource-group CreateIntLBQS-rg \
    --name myNSG
```

### <a name="create-a-network-security-group-rule"></a>Créer une règle de groupe de sécurité réseau

Créez une règle de groupe de sécurité réseau avec la commande [az network nsg rule create](/cli/azure/network/nsg/rule?view=azure-cli-latest#az-network-nsg-rule-create) :

* Nommé **myNSGRuleHTTP**.
* Dans le groupe de sécurité réseau que vous avez créé à l’étape précédente, **myNSG**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.
* Protocole **(*)** .
* Direction **entrante**.
* Source **(*)** .
* Destination **(*)** .
* Port de destination **Port 80**.
* Accès **Autoriser**.
* Priorité **200**.

```azurecli-interactive
  az network nsg rule create \
    --resource-group CreateIntLBQS-rg \
    --nsg-name myNSG \
    --name myNSGRuleHTTP \
    --protocol '*' \
    --direction inbound \
    --source-address-prefix '*' \
    --source-port-range '*' \
    --destination-address-prefix '*' \
    --destination-port-range 80 \
    --access allow \
    --priority 200
```

## <a name="create-backend-servers"></a>Créer des serveurs principaux

Dans cette section, vous allez créer :

* Interfaces réseau pour les serveurs back-end.
* Un fichier de configuration cloud nommé **cloud-init.txt** pour la configuration du serveur.
* Deux machines virtuelles à utiliser en tant que serveurs back-ends pour l’équilibreur de charge.

### <a name="create-network-interfaces-for-the-virtual-machines"></a>Créer des interfaces réseau pour les machines virtuelles

Créez deux interfaces réseau avec la commande [az network nic create](/cli/azure/network/nic?view=azure-cli-latest#az-network-nic-create) :

#### <a name="vm1"></a>MV1

* Nommée **myNicVM1**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.
* Dans le réseau virtuel **myVNet**.
* Dans le sous-réseau **myBackendSubnet**.
* Dans le groupe de sécurité réseau **myNSG**.

```azurecli-interactive
  az network nic create \
    --resource-group CreateIntLBQS-rg \
    --name myNicVM1 \
    --vnet-name myVNet \
    --subnet myBackEndSubnet \
    --network-security-group myNSG
```
#### <a name="vm2"></a>MV2

* Nommé **myNicVM2**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.
* Dans le réseau virtuel **myVNet**.
* Dans le sous-réseau **myBackendSubnet**.
* Dans le groupe de sécurité réseau **myNSG**.

```azurecli-interactive
  az network nic create \
    --resource-group CreateIntLBQS-rg \
    --name myNicVM2 \
    --vnet-name myVnet \
    --subnet myBackEndSubnet \
    --network-security-group myNSG
```

### <a name="create-cloud-init-configuration-file"></a>Créer un fichier de configuration cloud-init

Utilisez un fichier de configuration cloud-init pour installer NGINX et exécuter une application Node.js « Hello World » sur une machine virtuelle Linux. 

Dans votre interpréteur de commandes actuel, créez un fichier nommé cloud-init.txt. Copiez et collez l’extrait de code suivant dans l’interpréteur de commandes. Vérifiez que vous copiez bien le fichier cloud-init entier, en particulier la première ligne :

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
### <a name="create-virtual-machines"></a>Créer des machines virtuelles

Créez les machines virtuelles avec la commande [az vm create](/cli/azure/vm?view=azure-cli-latest#az-vm-create) :

#### <a name="vm1"></a>MV1
* Nommée **myVM1**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.
* Attachée à l’interface réseau **myNicVM1**.
* Image de machine virtuelle **UbuntuLTS**.
* Fichier de configuration **cloud-init.txt** que vous avez créé à l’étape précédente.
* Dans **Zone 1**.

```azurecli-interactive
  az vm create \
    --resource-group CreateIntLBQS-rg \
    --name myVM1 \
    --nics myNicVM1 \
    --image UbuntuLTS \
    --admin-user azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init.txt \
    --zone 1 \
    --no-wait
    
```
#### <a name="vm2"></a>MV2
* Nommée **myVM2**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.
* Attachée à l’interface réseau **myNicVM2**.
* Image de machine virtuelle **UbuntuLTS**.
* Fichier de configuration **cloud-init.txt** que vous avez créé à l’étape précédente.
* Dans **Zone 2**.

```azurecli-interactive
  az vm create \
    --resource-group CreateIntLBQS-rg \
    --name myVM2 \
    --nics myNicVM2 \
    --image UbuntuLTS \
    --admin-user azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init.txt \
    --zone 2 \
    --no-wait
```

Le déploiement des machines virtuelles peut prendre quelques minutes.

## <a name="create-standard-load-balancer"></a>Créer un équilibreur de charge standard

Cette section explique en détail comment vous pouvez créer et configurer les composants suivants de l’équilibreur de charge :

  * Un pool d’adresses IP frontal qui reçoit le trafic réseau entrant sur l’équilibreur de charge.
  * Un pool d’adresses IP principal auquel le pool frontal envoie le trafic réseau dont la charge est équilibrée.
  * Une sonde d’intégrité qui détermine l’intégrité des instances de machine virtuelle backend.
  * Une règle d’équilibreur de charge qui définit la distribution du trafic aux machines virtuelles.

### <a name="create-the-load-balancer-resource"></a>Créer la ressource d’équilibreur de charge

Créez un équilibreur de charge public avec la commande [az network lb create](/cli/azure/network/lb?view=azure-cli-latest#az-network-lb-create) :

* Nommé **myLoadBalancer**.
* Un pool frontal nommé **myFrontEnd**.
* Un pool principal nommé **myBackEndPool**.
* Associé au réseau virtuel **myVNet**.
* Associé au sous-réseau de back-end **myBackendSubnet**.

```azurecli-interactive
  az network lb create \
    --resource-group CreateIntLBQS-rg \
    --name myLoadBalancer \
    --sku Standard \
    --vnet-name myVnet \
    --subnet myBackendSubnet \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool       
```

### <a name="create-the-health-probe"></a>Créer la sonde d’intégrité

Une sonde d’intégrité vérifie toutes les instances de machine virtuelle pour s’assurer qu’elles peuvent transmettre du trafic réseau. 

Une machine virtuelle dont le contrôle de sonde a échoué est supprimée de l’équilibreur de charge. La machine virtuelle est rajoutée à l’équilibreur de charge une fois l’échec résolu.

Créez une sonde d’intégrité avec la commande [az network lb probe create](/cli/azure/network/lb/probe?view=azure-cli-latest#az-network-lb-probe-create) :

* Analyse l’intégrité des machines virtuelles.
* Nommée **myHealthProbe.** .
* Protocole **TCP**.
* Surveillant le **Port 80**.

```azurecli-interactive
  az network lb probe create \
    --resource-group CreateIntLBQS-rg \
    --lb-name myLoadBalancer \
    --name myHealthProbe \
    --protocol tcp \
    --port 80   
```

### <a name="create-the-load-balancer-rule"></a>Créer la règle d’équilibreur de charge

Une règle d’équilibreur de charge définit les éléments suivants :

* La configuration IP frontale pour le trafic entrant.
* Le pool d’adresses IP principal qui reçoit le trafic.
* Les ports source et de destination requis. 

Créez une règle d’équilibreur de charge avec la commande [az network lb rule create](/cli/azure/network/lb/rule?view=azure-cli-latest#az-network-lb-rule-create) :

* Nommée **myHTTPRule**.
* Écoutant le **Port 80** dans le pool frontal **myFrontEnd**.
* Envoyant le trafic réseau dont la charge est équilibrée au pool d’adresses principal **myBackEndPool** à l’aide du **Port 80**. 
* Utilisant la sonde d’intégrité **myHealthProbe**.
* Protocole **TCP**.
* Délai d’inactivité de **15 minutes**.
* Activez la réinitialisation TCP.

```azurecli-interactive
  az network lb rule create \
    --resource-group CreateIntLBQS-rg \
    --lb-name myLoadBalancer \
    --name myHTTPRule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe \
    --disable-outbound-snat true \
    --idle-timeout 15 \
    --enable-tcp-reset true
```
>[!NOTE]
>Avec cette configuration, les machines virtuelles du pool principal ne disposent pas d’une connectivité Internet sortante. </br> Pour plus d’informations sur la façon de fournir une connectivité sortante, consultez : </br> **[Connexions sortantes dans Azure](load-balancer-outbound-connections.md)**</br> Options pour fournir la connectivité : </br> **[Configuration de l’équilibreur de charge en sortie uniquement](egress-only.md)** </br> **[Qu’est-ce que le service NAT de Réseau virtuel ?](../virtual-network/nat-overview.md)**

### <a name="add-virtual-machines-to-load-balancer-backend-pool"></a>Ajouter des machines virtuelles au pool de back-ends de l’équilibreur de charge

Ajoutez les machines virtuelles au pool de back-ends avec la commande [az network nic ip-config address-pool add](/cli/azure/network/nic/ip-config/address-pool?view=azure-cli-latest#az-network-nic-ip-config-address-pool-add) :


#### <a name="vm1"></a>MV1
* Dans le pool d’adresses de back-end **myBackEndPool**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.
* Associée à l’interface réseau **myNicVM1** et **ipconfig1**.
* Associée à l’équilibreur de charge **myLoadBalancer**.

```azurecli-interactive
  az network nic ip-config address-pool add \
   --address-pool myBackendPool \
   --ip-config-name ipconfig1 \
   --nic-name myNicVM1 \
   --resource-group CreateIntLBQS-rg \
   --lb-name myLoadBalancer
```

#### <a name="vm2"></a>MV2
* Dans le pool d’adresses de back-end **myBackEndPool**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.
* Associée à l’interface réseau **myNicVM2** et **ipconfig1**.
* Associée à l’équilibreur de charge **myLoadBalancer**.

```azurecli-interactive
  az network nic ip-config address-pool add \
   --address-pool myBackendPool \
   --ip-config-name ipconfig1 \
   --nic-name myNicVM2 \
   --resource-group CreateIntLBQS-rg \
   --lb-name myLoadBalancer
```

# <a name="basic-sku"></a>[**Référence De base**](#tab/option-1-create-load-balancer-basic)

>[!NOTE]
>Il est recommandé de disposer d’un équilibreur de charge de référence SKU Standard pour les charges de travail de production. Pour plus d’informations sur les références (SKU), consultez **[Références SKU d’Azure Load Balancer](skus.md)** .

## <a name="configure-virtual-network"></a>Configurer un réseau virtuel

Avant de déployer des machines virtuelles et votre équilibreur de charge, créez les ressources de réseau virtuel associées.

### <a name="create-a-virtual-network"></a>Créez un réseau virtuel

Créez un réseau virtuel avec la commande [az network vnet create](/cli/azure/network/vnet?view=azure-cli-latest#az-network-vnet-createt) :

* Nommé **myVNet**.
* Préfixe d’adresse **10.1.0.0/16**.
* Sous-réseau nommé **myBackendSubnet**.
* Préfixe du sous-réseau **10.1.0.0/24**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.
* Emplacement **eastus**.

```azurecli-interactive
  az network vnet create \
    --resource-group CreateIntLBQS-rg \
    --location eastus \
    --name myVNet \
    --address-prefixes 10.1.0.0/16 \
    --subnet-name myBackendSubnet \
    --subnet-prefixes 10.1.0.0/24
```
### <a name="create-a-network-security-group"></a>Créer un groupe de sécurité réseau

Pour un équilibreur de charge standard, les machines virtuelles à l’adresse principale doivent disposer d’interfaces réseau appartenant à un groupe de sécurité réseau. 

Créez un Groupe de sécurité réseau avec la commande [az network nsg create](/cli/azure/network/nsg?view=azure-cli-latest#az-network-nsg-create) :

* Nommé **myNSG**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.

```azurecli-interactive
  az network nsg create \
    --resource-group CreateIntLBQS-rg \
    --name myNSG
```

### <a name="create-a-network-security-group-rule"></a>Créer une règle de groupe de sécurité réseau

Créez une règle de groupe de sécurité réseau avec la commande [az network nsg rule create](/cli/azure/network/nsg/rule?view=azure-cli-latest#az-network-nsg-rule-create) :

* Nommé **myNSGRuleHTTP**.
* Dans le groupe de sécurité réseau que vous avez créé à l’étape précédente, **myNSG**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.
* Protocole **(*)** .
* Direction **entrante**.
* Source **(*)** .
* Destination **(*)** .
* Port de destination **Port 80**.
* Accès **Autoriser**.
* Priorité **200**.

```azurecli-interactive
  az network nsg rule create \
    --resource-group CreateIntLBQS-rg \
    --nsg-name myNSG \
    --name myNSGRuleHTTP \
    --protocol '*' \
    --direction inbound \
    --source-address-prefix '*' \
    --source-port-range '*' \
    --destination-address-prefix '*' \
    --destination-port-range 80 \
    --access allow \
    --priority 200
```

### <a name="create-network-interfaces-for-the-virtual-machines"></a>Créer des interfaces réseau pour les machines virtuelles

Créez deux interfaces réseau avec la commande [az network nic create](/cli/azure/network/nic?view=azure-cli-latest#az-network-nic-create) :

#### <a name="vm1"></a>MV1

* Nommée **myNicVM1**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.
* Dans le réseau virtuel **myVNet**.
* Dans le sous-réseau **myBackendSubnet**.
* Dans le groupe de sécurité réseau **myNSG**.

```azurecli-interactive

  az network nic create \
    --resource-group CreateIntLBQS-rg \
    --name myNicVM1 \
    --vnet-name myVNet \
    --subnet myBackEndSubnet \
    --network-security-group myNSG
```
#### <a name="vm2"></a>MV2

* Nommé **myNicVM2**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.
* Dans le réseau virtuel **myVNet**.
* Dans le sous-réseau **myBackendSubnet**.

```azurecli-interactive
  az network nic create \
    --resource-group CreateIntLBQS-rg \
    --name myNicVM2 \
    --vnet-name myVnet \
    --subnet myBackEndSubnet \
    --network-security-group myNSG
```

## <a name="create-backend-servers"></a>Créer des serveurs principaux

Dans cette section, vous allez créer :

* Un fichier de configuration cloud nommé **cloud-init.txt** pour la configuration du serveur. 
* Un groupe à haute disponibilité pour les machines virtuelles.
* Deux machines virtuelles à utiliser en tant que serveurs back-ends pour l’équilibreur de charge.

Pour vérifier que l’équilibreur de charge a bien été créé, vous installez NGINX sur les machines virtuelles.

### <a name="create-cloud-init-configuration-file"></a>Créer un fichier de configuration cloud-init

Utilisez un fichier de configuration cloud-init pour installer NGINX et exécuter une application Node.js « Hello World » sur une machine virtuelle Linux. 

Dans votre interpréteur de commandes actuel, créez un fichier nommé cloud-init.txt. Copiez et collez l’extrait de code suivant dans l’interpréteur de commandes. Vérifiez que vous copiez bien le fichier cloud-init entier, en particulier la première ligne :

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

### <a name="create-availability-set-for-virtual-machines"></a>Créer un groupe à haute disponibilité pour des machines virtuelles

Créez le groupe à haute disponibilité avec la commande [az vm availability-set create](/cli/azure/vm/availability-set?view=azure-cli-latest#az-vm-availability-set-create) :

* Nommé **myAvSet**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.
* Emplacement **eastus**.

```azurecli-interactive
  az vm availability-set create \
    --name myAvSet \
    --resource-group CreateIntLBQS-rg \
    --location eastus 
    
```

### <a name="create-virtual-machines"></a>Créer des machines virtuelles

Créez les machines virtuelles avec la commande [az vm create](/cli/azure/vm?view=azure-cli-latest#az-vm-create) :

#### <a name="vm1"></a>MV1
* Nommée **myVM1**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.
* Attachée à l’interface réseau **myNicVM1**.
* Image de machine virtuelle **UbuntuLTS**.
* Fichier de configuration **cloud-init.txt** que vous avez créé à l’étape précédente.
* Dans le groupe à haute disponibilité **myAvSet**.

```azurecli-interactive
  az vm create \
    --resource-group CreateIntLBQS-rg \
    --name myVM1 \
    --nics myNicVM1 \
    --image UbuntuLTS \
    --admin-user azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init.txt \
    --availability-set myAvSet \
    --no-wait
    
```
#### <a name="vm2"></a>MV2
* Nommée **myVM2**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.
* Attachée à l’interface réseau **myNicVM2**.
* Image de machine virtuelle **UbuntuLTS**.
* Fichier de configuration **cloud-init.txt** que vous avez créé à l’étape précédente.
* Dans **Zone 2**.

```azurecli-interactive
  az vm create \
    --resource-group CreateIntLBQS-rg \
    --name myVM2 \
    --nics myNicVM2 \
    --image UbuntuLTS \
    --admin-user azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init.txt \
    --availability-set myAvSet  \
    --no-wait
```

Le déploiement des machines virtuelles peut prendre quelques minutes.

## <a name="create-basic-load-balancer"></a>Créer un équilibreur de charge de base

Cette section explique en détail comment vous pouvez créer et configurer les composants suivants de l’équilibreur de charge :

  * Un pool d’adresses IP frontal qui reçoit le trafic réseau entrant sur l’équilibreur de charge.
  * Un pool d’adresses IP principal auquel le pool frontal envoie le trafic réseau dont la charge est équilibrée.
  * Une sonde d’intégrité qui détermine l’intégrité des instances de machine virtuelle backend.
  * Une règle d’équilibreur de charge qui définit la distribution du trafic aux machines virtuelles.

### <a name="create-the-load-balancer-resource"></a>Créer la ressource d’équilibreur de charge

Créez un équilibreur de charge public avec la commande [az network lb create](/cli/azure/network/lb?view=azure-cli-latest#az-network-lb-create) :

* Nommé **myLoadBalancer**.
* Un pool frontal nommé **myFrontEnd**.
* Un pool principal nommé **myBackEndPool**.
* Associé au réseau virtuel **myVNet**.
* Associé au sous-réseau de back-end **myBackendSubnet**.

```azurecli-interactive
  az network lb create \
    --resource-group CreateIntLBQS-rg \
    --name myLoadBalancer \
    --sku Basic \
    --vnet-name myVNet \
    --subnet myBackendSubnet \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool       
```

### <a name="create-the-health-probe"></a>Créer la sonde d’intégrité

Une sonde d’intégrité vérifie toutes les instances de machine virtuelle pour s’assurer qu’elles peuvent transmettre du trafic réseau. 

Une machine virtuelle dont le contrôle de sonde a échoué est supprimée de l’équilibreur de charge. La machine virtuelle est rajoutée à l’équilibreur de charge une fois l’échec résolu.

Créez une sonde d’intégrité avec la commande [az network lb probe create](/cli/azure/network/lb/probe?view=azure-cli-latest#az-network-lb-probe-create) :

* Analyse l’intégrité des machines virtuelles.
* Nommée **myHealthProbe.** .
* Protocole **TCP**.
* Surveillant le **Port 80**.

```azurecli-interactive
  az network lb probe create \
    --resource-group CreateIntLBQS-rg \
    --lb-name myLoadBalancer \
    --name myHealthProbe \
    --protocol tcp \
    --port 80   
```

### <a name="create-the-load-balancer-rule"></a>Créer la règle d’équilibreur de charge

Une règle d’équilibreur de charge définit les éléments suivants :

* La configuration IP frontale pour le trafic entrant.
* Le pool d’adresses IP principal qui reçoit le trafic.
* Les ports source et de destination requis. 

Créez une règle d’équilibreur de charge avec la commande [az network lb rule create](/cli/azure/network/lb/rule?view=azure-cli-latest#az-network-lb-rule-create) :

* Nommée **myHTTPRule**.
* Écoutant le **Port 80** dans le pool frontal **myFrontEnd**.
* Envoyant le trafic réseau dont la charge est équilibrée au pool d’adresses principal **myBackEndPool** à l’aide du **Port 80**. 
* Utilisant la sonde d’intégrité **myHealthProbe**.
* Protocole **TCP**.
* Délai d’inactivité de **15 minutes**.

```azurecli-interactive
  az network lb rule create \
    --resource-group CreateIntLBQS-rg \
    --lb-name myLoadBalancer \
    --name myHTTPRule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe \
    --idle-timeout 15 
```
### <a name="add-virtual-machines-to-load-balancer-backend-pool"></a>Ajouter des machines virtuelles au pool de back-ends de l’équilibreur de charge

Ajoutez les machines virtuelles au pool de back-ends avec la commande [az network nic ip-config address-pool add](/cli/azure/network/nic/ip-config/address-pool?view=azure-cli-latest#az-network-nic-ip-config-address-pool-add) :


#### <a name="vm1"></a>MV1
* Dans le pool d’adresses de back-end **myBackEndPool**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.
* Associée à l’interface réseau **myNicVM1** et **ipconfig1**.
* Associée à l’équilibreur de charge **myLoadBalancer**.

```azurecli-interactive
  az network nic ip-config address-pool add \
   --address-pool myBackendPool \
   --ip-config-name ipconfig1 \
   --nic-name myNicVM1 \
   --resource-group CreateIntLBQS-rg \
   --lb-name myLoadBalancer
```

#### <a name="vm2"></a>MV2
* Dans le pool d’adresses de back-end **myBackEndPool**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.
* Associée à l’interface réseau **myNicVM2** et **ipconfig1**.
* Associée à l’équilibreur de charge **myLoadBalancer**.

```azurecli-interactive
  az network nic ip-config address-pool add \
   --address-pool myBackendPool \
   --ip-config-name ipconfig1 \
   --nic-name myNicVM2 \
   --resource-group CreateIntLBQS-rg \
   --lb-name myLoadBalancer
```

---

## <a name="test-the-load-balancer"></a>Tester l’équilibreur de charge

### <a name="create-azure-bastion-public-ip"></a>Créer une adresse IP publique Azure Bastion

Utilisez [az network public-ip create](/cli/azure/network/public-ip?view=azure-cli-latest#az-network-public-ip-create) pour créer une adresse IP publique pour l’hôte bastion :

* Créez une adresse IP publique redondante de zone standard nommée **myBastionIP**.
* Dans **CreateIntLBQS-rg**.

```azurecli-interactive
  az network public-ip create \
    --resource-group CreateIntLBQS-rg \
    --name myBastionIP \
    --sku Standard
```

### <a name="create-azure-bastion-subnet"></a>Créer un sous-réseau Azure Bastion

Utilisez [az network vnet subnet create](/cli/azure/network/vnet/subnet?view=azure-cli-latest#az-network-vnet-subnet-create) pour créer un sous-réseau :

* Nommé **AzureBastionSubnet**.
* Préfixe d’adresse **10.1.1.0/24**.
* Dans le réseau virtuel **myVNet**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.

```azurecli-interactive
  az network vnet subnet create \
    --resource-group CreateIntLBQS-rg \
    --name AzureBastionSubnet \
    --vnet-name myVNet \
    --address-prefixes 10.1.1.0/24
```

### <a name="create-azure-bastion-host"></a>Créer un hôte Azure Bastion
Utilisez [az network bastion create](/cli/azure/network/bastion?view=azure-cli-latest#az-network-bastion-create) pour créer un hôte bastion :

* Nommé **myBastionHost**
* Dans **CreateIntLBQS-rg**
* Associé à l’adresse IP publique **myBastionIP**.
* Associé au réseau virtuel **myVNet**.
* À l’emplacement **eastus**.

```azurecli-interactive
  az network bastion create \
    --resource-group CreateIntLBQS-rg \
    --name myBastionHost \
    --public-ip-address myBastionIP \
    --vnet-name myVNet \
    --location eastus
```
Le déploiement de l’hôte bastion va prendre quelques minutes.

### <a name="create-test-virtual-machine"></a>Créer une machine virtuelle de test

Créez l’interface réseau avec la commande [az network nic create](/cli/azure/network/nic?view=azure-cli-latest#az-network-nic-create) :

* Nommée **myNicTestVM**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.
* Dans le réseau virtuel **myVNet**.
* Dans le sous-réseau **myBackendSubnet**.
* Dans le groupe de sécurité réseau **myNSG**.

```azurecli-interactive
  az network nic create \
    --resource-group CreateIntLBQS-rg \
    --name myNicTestVM \
    --vnet-name myVNet \
    --subnet myBackEndSubnet \
    --network-security-group myNSG
```
Créez la machine virtuelle avec la commande [az vm create](/cli/azure/vm?view=azure-cli-latest#az-vm-create) :

* Nommé **myTestVM**.
* Dans le groupe de ressources **CreateIntLBQS-rg**.
* Attachée à l’interface réseau **myNicTestVM**.
* Image de machine virtuelle **Win2019Datacenter**.
* Choisissez des valeurs pour **\<adminpass>** et **\<adminuser>** .
  

```azurecli-interactive
  az vm create \
    --resource-group CreateIntLBQS-rg \
    --name myTestVM \
    --nics myNicTestVM \
    --image Win2019Datacenter \
    --admin-username <adminuser> \
    --admin-password <adminpass> \
    --no-wait
```
Le déploiement de la machine virtuelle peut prendre quelques minutes.

### <a name="test"></a>Test

1. [Connectez-vous](https://portal.azure.com) au portail Azure.

1. Recherchez l’adresse IP privée de l’équilibreur de charge sur l’écran **Vue d’ensemble**. Sélectionnez **Tous les services** dans le menu de gauche, sélectionnez **Toutes les ressources**, puis sélectionnez **myLoadBalancer**.

2. Notez ou copiez l’adresse en regard de **Adresse IP privée** dans la **Vue d’ensemble** de **myLoadBalancer**.

3. Sélectionnez **Tous les services** dans le menu de gauche, sélectionnez **Toutes les ressources** puis, dans la liste des ressources, sélectionnez **myTestVM** qui se trouve dans le groupe de ressources **CreateIntLBQS-rg**.

4. Dans la page **Vue d’ensemble**, sélectionnez **Se connecter**, puis **Bastion**.

6. Entrez le nom d’utilisateur et le mot de passe saisis pendant la création de la machine virtuelle.

7. Ouvrez **Internet Explorer** sur **myTestVM**.

8. Entrez l’adresse IP de l’étape précédente dans la barre d’adresse du navigateur. La page par défaut du serveur Web IIS s’affiche sur le navigateur.

    :::image type="content" source="./media/quickstart-load-balancer-standard-internal-portal/load-balancer-test.png" alt-text="Créer un équilibreur de charge interne standard" border="true":::
   
Pour voir l’équilibreur de charge répartir le trafic entre les trois machines virtuelles, vous pouvez personnaliser la page par défaut du serveur web IIS de chaque machine virtuelle, puis forcer votre navigateur à s’actualiser à partir de la machine cliente.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, utilisez la commande [az group delete](/cli/azure/group?view=azure-cli-latest#az-group-delete) pour supprimer le groupe de ressources, l’équilibreur de charge et toutes les ressources associées.

```azurecli-interactive
  az group delete \
    --name CreateIntLBQS-rg
```

## <a name="next-steps"></a>Étapes suivantes
Dans ce guide de démarrage rapide

* Vous avez créé un équilibreur de charge standard ou public
* Attaché des machines virtuelles. 
* Vous avez configuré la règle de trafic et la sonde d’intégrité de l’équilibreur de charge.
* Vous avez testé l’équilibreur de charge.

Pour en savoir plus sur Azure Load Balancer, consultez 
> [!div class="nextstepaction"]
> [Qu’est-ce que Azure Load Balancer ?](load-balancer-overview.md)
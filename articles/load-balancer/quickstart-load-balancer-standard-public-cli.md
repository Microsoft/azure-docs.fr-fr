---
title: 'Démarrage rapide : Créer un équilibreur de charge public – Azure CLI'
titleSuffix: Azure Load Balancer
description: Ce démarrage rapide montre comment créer un équilibreur de charge public à l’aide d’Azure CLI
services: load-balancer
documentationcenter: na
author: asudbring
manager: KumudD
tags: azure-resource-manager
Customer intent: I want to create a load balancer so that I can load balance internet traffic to VMs.
ms.service: load-balancer
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/23/2020
ms.author: allensu
ms.custom: mvc, devx-track-js, devx-track-azurecli
ms.openlocfilehash: 2b22c00845b38d2edea2d78497fb4b46a51896d4
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97587126"
---
# <a name="quickstart-create-a-public-load-balancer-to-load-balance-vms-using-azure-cli"></a>Démarrage rapide : Créer un équilibreur de charge public pour équilibrer la charge des machines virtuelles à l’aide d’Azure CLI

Commencez avec Azure Load Balancer en utilisant le Azure CLI pour créer un équilibreur de charge public et trois machines virtuelles.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Ce guide de démarrage rapide nécessite la version 2.0.28 ou ultérieure de l’interface Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group#az-group-create) :

* Nommé **CreatePubLBQS-rg**. 
* Dans l’emplacement **eastus**.

```azurecli-interactive
  az group create \
    --name CreatePubLBQS-rg \
    --location eastus
```
---

# <a name="standard-sku"></a>[**Référence Standard**](#tab/option-1-create-load-balancer-standard)

>[!NOTE]
>Il est recommandé de disposer d’un équilibreur de charge de référence SKU Standard pour les charges de travail de production. Pour plus d’informations sur les références SKU, consultez **[Références SKU Azure Load Balancer](skus.md)** .

:::image type="content" source="./media/quickstart-load-balancer-standard-public-portal/resources-diagram.png" alt-text="Ressources Standard Load Balancer créées pour le guide de démarrage rapide." border="false":::

## <a name="configure-virtual-network---standard"></a>Configurer le réseau virtuel - Standard

Avant de déployer des machines virtuelles et de tester votre équilibreur de charge, créez les ressources de réseau virtuel associées.

### <a name="create-a-virtual-network"></a>Créez un réseau virtuel

Créez un réseau virtuel avec la commande [az network vnet create](/cli/azure/network/vnet#az-network-vnet-createt) :

* Nommé **myVNet**.
* Préfixe d’adresse **10.1.0.0/16**.
* Sous-réseau nommé **myBackendSubnet**.
* Préfixe du sous-réseau **10.1.0.0/24**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.
* Emplacement **eastus**.

```azurecli-interactive
  az network vnet create \
    --resource-group CreatePubLBQS-rg \
    --location eastus \
    --name myVNet \
    --address-prefixes 10.1.0.0/16 \
    --subnet-name myBackendSubnet \
    --subnet-prefixes 10.1.0.0/24
```
### <a name="create-a-public-ip-address"></a>Créer une adresse IP publique

Utilisez [az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create) pour créer une adresse IP publique pour l’hôte bastion :

* Créez une adresse IP publique redondante de zone standard nommée **myBastionIP**.
* Dans **CreatePubLBQS.rg**.

```azurecli-interactive
az network public-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myBastionIP \
    --sku Standard
```
### <a name="create-a-bastion-subnet"></a>Créer un sous-réseau bastion

Utilisez [az network vnet subnet create](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-create) pour créer un sous-réseau bastion :

* Nommé **AzureBastionSubnet**.
* Préfixe d’adresse **10.1.1.0/24**.
* Dans le réseau virtuel **myVNet**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.

```azurecli-interactive
az network vnet subnet create \
    --resource-group CreatePubLBQS-rg \
    --name AzureBastionSubnet \
    --vnet-name myVNet \
    --address-prefixes 10.1.1.0/24
```

### <a name="create-bastion-host"></a>Créer un hôte bastion

Utilisez [az network bastion create](/cli/azure/network/bastion#az-network-bastion-create) pour créer un hôte bastion :

* Nom : **myBastionHost**
* Dans **CreatePubLBQS.rg**.
* Associé à l’adresse IP publique **myBastionIP**.
* Associé au réseau virtuel **myVNet**.
* À l’emplacement **eastus**.

```azurecli-interactive
az network bastion create \
    --resource-group CreatePubLBQS-rg \
    --name myBastionHost \
    --public-ip-address myBastionIP \
    --vnet-name myVNet \
    --location eastus
```

Le déploiement de l’hôte Azure Bastion peut prendre quelques minutes.

### <a name="create-a-network-security-group"></a>Créer un groupe de sécurité réseau

Pour un équilibreur de charge standard, les machines virtuelles à l’adresse principale doivent disposer d’interfaces réseau appartenant à un groupe de sécurité réseau. 

Créez un Groupe de sécurité réseau avec la commande [az network nsg create](/cli/azure/network/nsg#az-network-nsg-create) :

* Nommé **myNSG**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.

```azurecli-interactive
  az network nsg create \
    --resource-group CreatePubLBQS-rg \
    --name myNSG
```

### <a name="create-a-network-security-group-rule"></a>Créer une règle de groupe de sécurité réseau

Créez une règle de groupe de sécurité réseau avec la commande [az network nsg rule create](/cli/azure/network/nsg/rule#az-network-nsg-rule-create) :

* Nommé **myNSGRuleHTTP**.
* Dans le groupe de sécurité réseau que vous avez créé à l’étape précédente, **myNSG**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.
* Protocole **(*)** .
* Direction **entrante**.
* Source **(*)** .
* Destination **(*)** .
* Port de destination **Port 80**.
* Accès **Autoriser**.
* Priorité **200**.

```azurecli-interactive
  az network nsg rule create \
    --resource-group CreatePubLBQS-rg \
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

## <a name="create-backend-servers---standard"></a>Créer des serveurs back-end - Standard

Dans cette section, vous allez créer :

* Trois interfaces réseau pour les machines virtuelles.
* Trois machines virtuelles à utiliser en tant que serveurs principaux pour l’équilibreur de charge.

### <a name="create-network-interfaces-for-the-virtual-machines"></a>Créer des interfaces réseau pour les machines virtuelles

Créez les interfaces réseau avec la commande [az network nic create](/cli/azure/network/nic#az-network-nic-create) :

* Nommées **myNicVM1**, **myNicVM2** et **myNicVM3**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.
* Dans le réseau virtuel **myVNet**.
* Dans le sous-réseau **myBackendSubnet**.
* Dans le groupe de sécurité réseau **myNSG**.

```azurecli-interactive
  array=(myNicVM1 myNicVM2 myNicVM3)
  for vmnic in "${array[@]}"
  do
    az network nic create \
        --resource-group CreatePubLBQS-rg \
        --name $vmnic \
        --vnet-name myVNet \
        --subnet myBackEndSubnet \
        --network-security-group myNSG
  done
```

### <a name="create-virtual-machines"></a>Créer des machines virtuelles

Créez les machines virtuelles avec la commande [az vm create](/cli/azure/vm#az-vm-create) :

### <a name="vm1"></a>MV1
* Nommée **myVM1**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.
* Attachée à l’interface réseau **myNicVM1**.
* Image de machine virtuelle **win2019datacenter**.
* Dans **Zone 1**.

```azurecli-interactive
  az vm create \
    --resource-group CreatePubLBQS-rg \
    --name myVM1 \
    --nics myNicVM1 \
    --image win2019datacenter \
    --admin-username azureuser \
    --zone 1 \
    --no-wait
```
#### <a name="vm2"></a>MV2
* Nommée **myVM2**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.
* Attachée à l’interface réseau **myNicVM2**.
* Image de machine virtuelle **win2019datacenter**.
* Dans **Zone 2**.

```azurecli-interactive
  az vm create \
    --resource-group CreatePubLBQS-rg \
    --name myVM2 \
    --nics myNicVM2 \
    --image win2019datacenter \
    --admin-username azureuser \
    --zone 2 \
    --no-wait
```

#### <a name="vm3"></a>MV3
* Nommée **myVM3**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.
* Attachée à l’interface réseau **myNicVM3**.
* Image de machine virtuelle **win2019datacenter**.
* Dans **Zone 3**.

```azurecli-interactive
   az vm create \
    --resource-group CreatePubLBQS-rg \
    --name myVM3 \
    --nics myNicVM3 \
    --image win2019datacenter \
    --admin-username azureuser \
    --zone 3 \
    --no-wait
```
Le déploiement des machines virtuelles peut prendre quelques minutes.

## <a name="create-a-public-ip-address---standard"></a>Créer une adresse IP publique - Standard

Pour accéder à votre application web sur Internet, vous avez besoin d’une adresse IP publique pour l’équilibreur de charge. 

Utilisez la commande [az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create) pour effectuer les opérations suivantes :

* Créer une adresse IP publique redondante dans une zone standard nommée **myPublicIP**.
* Dans **CreatePubLBQS.rg**.

```azurecli-interactive
  az network public-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIP \
    --sku Standard
```

Pour créer une adresse IP publique redondante zonale dans la Zone 1, procédez comme suit :

```azurecli-interactive
  az network public-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIP \
    --sku Standard \
    --zone 1
```

## <a name="create-standard-load-balancer"></a>Créer un équilibreur de charge standard

Cette section explique en détail comment vous pouvez créer et configurer les composants suivants de l’équilibreur de charge :

  * Un pool d’adresses IP frontal qui reçoit le trafic réseau entrant sur l’équilibreur de charge.
  * Un pool d’adresses IP principal auquel le pool frontal envoie le trafic réseau dont la charge est équilibrée.
  * Une sonde d’intégrité qui détermine l’intégrité des instances de machine virtuelle backend.
  * Une règle d’équilibreur de charge qui définit la distribution du trafic aux machines virtuelles.

### <a name="create-the-load-balancer-resource"></a>Créer la ressource d’équilibreur de charge

Créez un équilibreur de charge public avec la commande [az network lb create](/cli/azure/network/lb#az-network-lb-create) :

* Nommé **myLoadBalancer**.
* Un pool frontal nommé **myFrontEnd**.
* Un pool principal nommé **myBackEndPool**.
* Associé à l’adresse IP publique **myPublicIP** que vous avez créé à l’étape précédente. 

```azurecli-interactive
  az network lb create \
    --resource-group CreatePubLBQS-rg \
    --name myLoadBalancer \
    --sku Standard \
    --public-ip-address myPublicIP \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool       
```

### <a name="create-the-health-probe"></a>Créer la sonde d’intégrité

Une sonde d’intégrité vérifie toutes les instances de machine virtuelle pour s’assurer qu’elles peuvent transmettre du trafic réseau. 

Une machine virtuelle dont le contrôle de sonde a échoué est supprimée de l’équilibreur de charge. La machine virtuelle est rajoutée à l’équilibreur de charge une fois l’échec résolu.

Créez une sonde d’intégrité avec la commande [az network lb probe create](/cli/azure/network/lb/probe#az-network-lb-probe-create) :

* Analyse l’intégrité des machines virtuelles.
* Nommée **myHealthProbe.** .
* Protocole **TCP**.
* Surveillant le **Port 80**.

```azurecli-interactive
  az network lb probe create \
    --resource-group CreatePubLBQS-rg \
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

Créez une règle d’équilibreur de charge avec la commande [az network lb rule create](/cli/azure/network/lb/rule#az-network-lb-rule-create) :

* Nommée **myHTTPRule**.
* Écoutant le **Port 80** dans le pool frontal **myFrontEnd**.
* Envoyant le trafic réseau dont la charge est équilibrée au pool d’adresses principal **myBackEndPool** à l’aide du **Port 80**. 
* Utilisant la sonde d’intégrité **myHealthProbe**.
* Protocole **TCP**.
* Délai d’inactivité de **15 minutes**.
* Activez la réinitialisation TCP.


```azurecli-interactive
  az network lb rule create \
    --resource-group CreatePubLBQS-rg \
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
### <a name="add-virtual-machines-to-load-balancer-backend-pool"></a>Ajouter des machines virtuelles au pool de back-ends de l’équilibreur de charge

Ajoutez les machines virtuelles au pool de back-ends avec la commande [az network nic ip-config address-pool add](/cli/azure/network/nic/ip-config/address-pool#az-network-nic-ip-config-address-pool-add) :

* Dans le pool d’adresses de back-end **myBackEndPool**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.
* Associée à l’équilibreur de charge **myLoadBalancer**.

```azurecli-interactive
  array=(myNicVM1 myNicVM2 myNicVM3)
  for vmnic in "${array[@]}"
  do
    az network nic ip-config address-pool add \
     --address-pool myBackendPool \
     --ip-config-name ipconfig1 \
     --nic-name $vmnic \
     --resource-group CreatePubLBQS-rg \
     --lb-name myLoadBalancer
  done
```

## <a name="create-outbound-rule-configuration"></a>Créer une configuration de règle de trafic sortant
Les règles de trafic sortant de l’équilibreur de charge configurent la SNAT sortante pour les machines virtuelles dans le pool principal. 

Pour plus d’informations sur les connexions sortantes, consultez [Connexions sortantes dans Azure](load-balancer-outbound-connections.md).

Vous pouvez utiliser un préfixe ou une adresse IP publique pour la configuration sortante.

### <a name="public-ip"></a>Adresse IP publique

Utilisez la commande [az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create) pour créer une adresse IP unique pour la connectivité sortante.  

* Nommée **myPublicIPOutbound**.
* Dans **CreatePubLBQS.rg**.

```azurecli-interactive
  az network public-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIPOutbound \
    --sku Standard
```

Pour créer une adresse IP publique redondante zonale dans la Zone 1, procédez comme suit :

```azurecli-interactive
  az network public-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIPOutbound \
    --sku Standard \
    --zone 1
```

### <a name="public-ip-prefix"></a>Préfixe d’adresse IP publique

Utilisez la commande [az network public-ip prefix create](/cli/azure/network/public-ip/prefix#az-network-public-ip-prefix-create) pour créer un préfixe d’adresse IP publique pour la connectivité sortante.

* Nommé **myPublicIPPrefixOutbound**.
* Dans **CreatePubLBQS.rg**.
* Longueur de préfixe de **28**.

```azurecli-interactive
  az network public-ip prefix create \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIPPrefixOutbound \
    --length 28
```
Pour créer un préfixe d’adresse IP publique redondante zonale dans la Zone 1 :

```azurecli-interactive
  az network public-ip prefix create \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIPPrefixOutbound \
    --length 28 \
    --zone 1
```

Pour plus d’informations sur la mise à l’échelle d’une traduction d’adresses réseau (NAT) sortante et d’une connectivité sortante, consultez [Mettre à l’échelle un NAT sortante avec plusieurs adresses IP](load-balancer-outbound-connections.md).

### <a name="create-outbound-frontend-ip-configuration"></a>Créer une configuration d’adresse IP frontale sortante

Créez une configuration d’adresse IP frontale avec la commande [az network lb frontend-ip create](/cli/azure/network/lb/frontend-ip#az-network-lb-frontend-ip-create) :

Sélectionnez les commandes d’adresse IP publique ou de préfixe d’adresse IP publique en fonction de la décision prise à l’étape précédente.

#### <a name="public-ip"></a>Adresse IP publique

* Nommée **myFrontEndOutbound**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.
* Associé à une adresse IP publique **myPublicIPOutbound**.
* Associée avec l’équilibreur de charge **myLoadBalancer**.

```azurecli-interactive
  az network lb frontend-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myFrontEndOutbound \
    --lb-name myLoadBalancer \
    --public-ip-address myPublicIPOutbound 
```

#### <a name="public-ip-prefix"></a>Préfixe d’adresse IP publique

* Nommée **myFrontEndOutbound**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.
* Associée avec le préfixe d’adresse IP publique **myPublicIPPrefixOutbound**.
* Associée avec l’équilibreur de charge **myLoadBalancer**.

```azurecli-interactive
  az network lb frontend-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myFrontEndOutbound \
    --lb-name myLoadBalancer \
    --public-ip-prefix myPublicIPPrefixOutbound 
```

### <a name="create-outbound-pool"></a>Créer un pool sortant

Créez un pool sortant avec la commande [az network lb address-pool create](/cli/azure/network/lb/address-pool#az-network-lb-address-pool-create) :

* Nommé **myBackendPoolOutbound**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.
* Associée avec l’équilibreur de charge **myLoadBalancer**.

```azurecli-interactive
  az network lb address-pool create \
    --resource-group CreatePubLBQS-rg \
    --lb-name myLoadBalancer \
    --name myBackendPoolOutbound
```
### <a name="create-outbound-rule"></a>Créer une règle de trafic sortant

Créez une règle de trafic sortant pour le pool principal sortant avec la commande [az network lb outbound-rule create](/cli/azure/network/lb/outbound-rule#az-network-lb-outbound-rule-create) :

* Nommée **myOutboundRule**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.
* Associée à l’équilibreur de charge **myLoadBalancer**.
* Associée au serveur frontal **myFrontEndOutbound**.
* Protocole **Tous**.
* Délai d’inactivité de **15**.
* **10000** ports sortants.
* Associée au pool principal **myBackEndPoolOutbound**.

```azurecli-interactive
  az network lb outbound-rule create \
    --resource-group CreatePubLBQS-rg \
    --lb-name myLoadBalancer \
    --name myOutboundRule \
    --frontend-ip-configs myFrontEndOutbound \
    --protocol All \
    --idle-timeout 15 \
    --outbound-ports 10000 \
    --address-pool myBackEndPoolOutbound
```
### <a name="add-virtual-machines-to-outbound-pool"></a>Ajouter des machines virtuelles à un pool sortant

Ajoutez les machines virtuelles au pool sortant avec la commande [az network nic ip-config address-pool add](/cli/azure/network/nic/ip-config/address-pool#az-network-nic-ip-config-address-pool-add) :


* Dans le pool d’adresses principal **myBackEndPoolOutbound**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.
* Associée à l’équilibreur de charge **myLoadBalancer**.

```azurecli-interactive
  array=(myNicVM1 myNicVM2 myNicVM3)
  for vmnic in "${array[@]}"
  do
    az network nic ip-config address-pool add \
     --address-pool myBackendPoolOutbound \
     --ip-config-name ipconfig1 \
     --nic-name $vmnic \
     --resource-group CreatePubLBQS-rg \
     --lb-name myLoadBalancer
  done
```

# <a name="basic-sku"></a>[**Référence De base**](#tab/option-1-create-load-balancer-basic)

>[!NOTE]
>Il est recommandé de disposer d’un équilibreur de charge de référence SKU Standard pour les charges de travail de production. Pour plus d’informations sur les références SKU, consultez **[Références SKU Azure Load Balancer](skus.md)** .

:::image type="content" source="./media/quickstart-load-balancer-standard-public-portal/resources-diagram-basic.png" alt-text="Ressources de l’équilibreur de charge de base créées dans le guide de démarrage rapide." border="false":::

## <a name="configure-virtual-network---basic"></a>Configurer un réseau virtuel - De base

Avant de déployer des machines virtuelles et de tester votre équilibreur de charge, créez les ressources de réseau virtuel associées.

### <a name="create-a-virtual-network"></a>Créez un réseau virtuel

Créez un réseau virtuel avec la commande [az network vnet create](/cli/azure/network/vnet#az-network-vnet-create) :

* Nommé **myVNet**.
* Préfixe d’adresse **10.1.0.0/16**.
* Sous-réseau nommé **myBackendSubnet**.
* Préfixe du sous-réseau **10.1.0.0/24**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.
* Emplacement **eastus**.

```azurecli-interactive
  az network vnet create \
    --resource-group CreatePubLBQS-rg \
    --location eastus \
    --name myVNet \
    --address-prefixes 10.1.0.0/16 \
    --subnet-name myBackendSubnet \
    --subnet-prefixes 10.1.0.0/24
```

### <a name="create-a-public-ip-address"></a>Créer une adresse IP publique

Utilisez [az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create) pour créer une adresse IP publique pour l’hôte bastion :

* Créez une adresse IP publique redondante de zone standard nommée **myBastionIP**.
* Dans **CreatePubLBQS.rg**.

```azurecli-interactive
az network public-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myBastionIP \
    --sku Standard
```
### <a name="create-a-bastion-subnet"></a>Créer un sous-réseau bastion

Utilisez [az network vnet subnet create](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-create) pour créer un sous-réseau bastion :

* Nommé **AzureBastionSubnet**.
* Préfixe d’adresse **10.1.1.0/24**.
* Dans le réseau virtuel **myVNet**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.

```azurecli-interactive
az network vnet subnet create \
    --resource-group CreatePubLBQS-rg \
    --name AzureBastionSubnet \
    --vnet-name myVNet \
    --address-prefixes 10.1.1.0/24
```

### <a name="create-bastion-host"></a>Créer un hôte bastion

Utilisez [az network bastion create](/cli/azure/network/bastion#az-network-bastion-create) pour créer un hôte bastion :

* Nom : **myBastionHost**
* Dans **CreatePubLBQS.rg**.
* Associé à l’adresse IP publique **myBastionIP**.
* Associé au réseau virtuel **myVNet**.
* À l’emplacement **eastus**.

```azurecli-interactive
az network bastion create \
    --resource-group CreatePubLBQS-rg \
    --name myBastionHost \
    --public-ip-address myBastionIP \
    --vnet-name myVNet \
    --location eastus
```

Le déploiement de l’hôte Azure Bastion peut prendre quelques minutes.

### <a name="create-a-network-security-group"></a>Créer un groupe de sécurité réseau

Pour un équilibreur de charge standard, les machines virtuelles à l’adresse principale doivent disposer d’interfaces réseau appartenant à un groupe de sécurité réseau. 

Créez un Groupe de sécurité réseau avec la commande [az network nsg create](/cli/azure/network/nsg#az-network-nsg-create) :

* Nommé **myNSG**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.

```azurecli-interactive
  az network nsg create \
    --resource-group CreatePubLBQS-rg \
    --name myNSG
```

### <a name="create-a-network-security-group-rule"></a>Créer une règle de groupe de sécurité réseau

Créez une règle de groupe de sécurité réseau avec la commande [az network nsg rule create](/cli/azure/network/nsg/rule#az-network-nsg-rule-create) :

* Nommé **myNSGRuleHTTP**.
* Dans le groupe de sécurité réseau que vous avez créé à l’étape précédente, **myNSG**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.
* Protocole **(*)** .
* Direction **entrante**.
* Source **(*)** .
* Destination **(*)** .
* Port de destination **Port 80**.
* Accès **Autoriser**.
* Priorité **200**.

```azurecli-interactive
  az network nsg rule create \
    --resource-group CreatePubLBQS-rg \
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

## <a name="create-backend-servers---basic"></a>Créer des serveurs back-end - De base

Dans cette section, vous allez créer :

* Trois interfaces réseau pour les machines virtuelles.
* Un groupe à haute disponibilité pour les machines virtuelles.
* Trois machines virtuelles à utiliser en tant que serveurs principaux pour l’équilibreur de charge.


### <a name="create-network-interfaces-for-the-virtual-machines"></a>Créer des interfaces réseau pour les machines virtuelles

Créez les interfaces réseau avec la commande [az network nic create](/cli/azure/network/nic#az-network-nic-create) :


* Nommées **myNicVM1**, **myNicVM2** et **myNicVM3**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.
* Dans le réseau virtuel **myVNet**.
* Dans le sous-réseau **myBackendSubnet**.
* Dans le groupe de sécurité réseau **myNSG**.

```azurecli-interactive
  array=(myNicVM1 myNicVM2 myNicVM3)
  for vmnic in "${array[@]}"
  do
    az network nic create \
        --resource-group CreatePubLBQS-rg \
        --name $vmnic \
        --vnet-name myVNet \
        --subnet myBackEndSubnet \
        --network-security-group myNSG
  done
```
### <a name="create-availability-set-for-virtual-machines"></a>Créer un groupe à haute disponibilité pour des machines virtuelles

Créez le groupe à haute disponibilité avec la commande [az vm availability-set create](/cli/azure/vm/availability-set#az-vm-availability-set-create) :

* Nommé **myAvSet**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.
* Emplacement **eastus**.

```azurecli-interactive
  az vm availability-set create \
    --name myAvSet \
    --resource-group CreatePubLBQS-rg \
    --location eastus 
    
```

### <a name="create-virtual-machines"></a>Créer des machines virtuelles

Créez les machines virtuelles avec la commande [az vm create](/cli/azure/vm#az-vm-create) :

### <a name="vm1"></a>MV1
* Nommée **myVM1**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.
* Attachée à l’interface réseau **myNicVM1**.
* Image de machine virtuelle **win2019datacenter**.
* Dans **Zone 1**.

```azurecli-interactive
  az vm create \
    --resource-group CreatePubLBQS-rg \
    --name myVM1 \
    --nics myNicVM1 \
    --image win2019datacenter \
    --admin-username azureuser \
    --availability-set myAvSet \
    --no-wait
```
#### <a name="vm2"></a>MV2
* Nommée **myVM2**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.
* Attachée à l’interface réseau **myNicVM2**.
* Image de machine virtuelle **win2019datacenter**.
* Dans **Zone 2**.

```azurecli-interactive
  az vm create \
    --resource-group CreatePubLBQS-rg \
    --name myVM2 \
    --nics myNicVM2 \
    --image win2019datacenter \
    --admin-username azureuser \
    --availability-set myAvSet \
    --no-wait
```

#### <a name="vm3"></a>MV3
* Nommée **myVM3**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.
* Attachée à l’interface réseau **myNicVM3**.
* Image de machine virtuelle **win2019datacenter**.
* Dans **Zone 3**.

```azurecli-interactive
   az vm create \
    --resource-group CreatePubLBQS-rg \
    --name myVM3 \
    --nics myNicVM3 \
    --image win2019datacenter \
    --admin-username azureuser \
    --availability-set myAvSet \
    --no-wait
```
Le déploiement des machines virtuelles peut prendre quelques minutes.

## <a name="create-a-public-ip-address---basic"></a>Créer une adresse IP publique - De base

Pour accéder à votre application web sur Internet, vous avez besoin d’une adresse IP publique pour l’équilibreur de charge. 

Utilisez la commande [az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create) pour effectuer les opérations suivantes :

* Créer une adresse IP publique redondante dans une zone standard nommée **myPublicIP**.
* Dans **CreatePubLBQS.rg**.

```azurecli-interactive
  az network public-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIP \
    --sku Basic
```

## <a name="create-basic-load-balancer"></a>Créer un équilibreur de charge de base

Cette section explique en détail comment vous pouvez créer et configurer les composants suivants de l’équilibreur de charge :

  * Un pool d’adresses IP frontal qui reçoit le trafic réseau entrant sur l’équilibreur de charge.
  * Un pool d’adresses IP principal auquel le pool frontal envoie le trafic réseau dont la charge est équilibrée.
  * Une sonde d’intégrité qui détermine l’intégrité des instances de machine virtuelle backend.
  * Une règle d’équilibreur de charge qui définit la distribution du trafic aux machines virtuelles.

### <a name="create-the-load-balancer-resource"></a>Créer la ressource d’équilibreur de charge

Créez un équilibreur de charge public avec la commande [az network lb create](/cli/azure/network/lb#az-network-lb-create) :

* Nommé **myLoadBalancer**.
* Un pool frontal nommé **myFrontEnd**.
* Un pool principal nommé **myBackEndPool**.
* Associé à l’adresse IP publique **myPublicIP** que vous avez créé à l’étape précédente. 

```azurecli-interactive
  az network lb create \
    --resource-group CreatePubLBQS-rg \
    --name myLoadBalancer \
    --sku Basic \
    --public-ip-address myPublicIP \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool       
```

### <a name="create-the-health-probe"></a>Créer la sonde d’intégrité

Une sonde d’intégrité vérifie toutes les instances de machine virtuelle pour s’assurer qu’elles peuvent transmettre du trafic réseau. 

Une machine virtuelle dont le contrôle de sonde a échoué est supprimée de l’équilibreur de charge. La machine virtuelle est rajoutée à l’équilibreur de charge une fois l’échec résolu.

Créez une sonde d’intégrité avec la commande [az network lb probe create](/cli/azure/network/lb/probe#az-network-lb-probe-create) :

* Analyse l’intégrité des machines virtuelles.
* Nommée **myHealthProbe.** .
* Protocole **TCP**.
* Surveillant le **Port 80**.

```azurecli-interactive
  az network lb probe create \
    --resource-group CreatePubLBQS-rg \
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

Créez une règle d’équilibreur de charge avec la commande [az network lb rule create](/cli/azure/network/lb/rule#az-network-lb-rule-create) :

* Nommée **myHTTPRule**.
* Écoutant le **Port 80** dans le pool frontal **myFrontEnd**.
* Envoyant le trafic réseau dont la charge est équilibrée au pool d’adresses principal **myBackEndPool** à l’aide du **Port 80**. 
* Utilisant la sonde d’intégrité **myHealthProbe**.
* Protocole **TCP**.
* Délai d’inactivité de **15 minutes**.

```azurecli-interactive
  az network lb rule create \
    --resource-group CreatePubLBQS-rg \
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

Ajoutez les machines virtuelles au pool de back-ends avec la commande [az network nic ip-config address-pool add](/cli/azure/network/nic/ip-config/address-pool#az-network-nic-ip-config-address-pool-add) :

* Dans le pool d’adresses de back-end **myBackEndPool**.
* Dans le groupe de ressources **CreatePubLBQS-rg**.
* Associée avec l’équilibreur de charge **myLoadBalancer**.

```azurecli-interactive
  array=(myNicVM1 myNicVM2 myNicVM3)
  for vmnic in "${array[@]}"
  do
    az network nic ip-config address-pool add \
     --address-pool myBackendPool \
     --ip-config-name ipconfig1 \
     --nic-name $vmnic \
     --resource-group CreatePubLBQS-rg \
     --lb-name myLoadBalancer
  done
```

---

## <a name="install-iis"></a>Installer IIS

Utilisez [AZ VM Extension Set](/cli/azure/vm/extension#az_vm_extension_set) pour installer IIS sur les machines virtuelles et définir le nom de l’ordinateur comme site web par défaut.

```azurecli-interactive
  array=(myVM1 myVM2 myVM3)
    for vm in "${array[@]}"
    do
     az vm extension set \
       --publisher Microsoft.Compute \
       --version 1.8 \
       --name CustomScriptExtension \
       --vm-name $vm \
       --resource-group CreatePubLBQS-rg \
       --settings '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}'
  done

```

## <a name="test-the-load-balancer"></a>Tester l’équilibreur de charge

Pour obtenir l’adresse IP publique de l’équilibreur de charge, utilisez la commande [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show). 

Copiez l’adresse IP publique, puis collez-la dans la barre d’adresses de votre navigateur.

```azurecli-interactive
  az network public-ip show \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIP \
    --query ipAddress \
    --output tsv
```
:::image type="content" source="./media/load-balancer-standard-public-cli/running-nodejs-app.png" alt-text="Tester l’équilibreur de charge" border="true":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, utilisez la commande [az group delete](/cli/azure/group#az-group-delete) pour supprimer le groupe de ressources, l’équilibreur de charge et toutes les ressources associées.

```azurecli-interactive
  az group delete \
    --name CreatePubLBQS-rg
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide

* Vous avez créé un équilibreur de charge standard ou public
* Attaché des machines virtuelles. 
* Vous avez configuré la règle de trafic et la sonde d’intégrité de l’équilibreur de charge.
* Vous avez testé l’équilibreur de charge.

Pour en savoir plus sur Azure Load Balancer, consultez :
> [!div class="nextstepaction"]
> [Qu’est-ce que Azure Load Balancer ?](load-balancer-overview.md)
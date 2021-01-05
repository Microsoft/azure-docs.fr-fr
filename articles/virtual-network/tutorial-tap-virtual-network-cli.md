---
title: Créer, modifier ou supprimer un point d’accès terminal de réseau virtuel – Azure CLI
description: Découvrez comment créer, changer ou supprimer un TAP de réseau virtuel à l’aide d’Azure CLI.
services: virtual-network
documentationcenter: na
author: karthikananth
manager: ganesr
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/18/2018
ms.author: kaanan
ms.custom: devx-track-azurecli
ms.openlocfilehash: 8e3a56e4a6eb1fb6eb633021178ef78f8ac7287d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96014786"
---
# <a name="work-with-a-virtual-network-tap-using-the-azure-cli"></a>Utiliser un TAP de réseau virtuel à l’aide d’Azure CLI

Le TAP (point d’accès terminal) de réseau virtuel Azure vous permet de diffuser en continu votre trafic réseau de machine virtuelle vers un collecteur de paquets réseau ou un outil analytique. Le collecteur ou l’outil analytique est fourni par une [appliance virtuelle réseau](https://azure.microsoft.com/solutions/network-appliances/) partenaire. Pour obtenir la liste des solutions de partenaires qui fonctionnent avec un TAP de réseau virtuel, consultez les [solutions de partenaires](virtual-network-tap-overview.md#virtual-network-tap-partner-solutions). 

## <a name="create-a-virtual-network-tap-resource"></a>Créer une ressource TAP de réseau virtuel

Lisez les [prérequis](virtual-network-tap-overview.md#prerequisites) avant de créer une ressource TAP de réseau virtuel. Vous pouvez exécuter les commandes qui suivent dans [Azure Cloud Shell](https://shell.azure.com/bash) ou en exécutant l’interface de ligne de commande (CLI) Azure à partir de votre ordinateur. Azure Cloud Shell est un interpréteur de commandes interactif gratuit qui ne nécessite pas l’installation d’Azure CLI sur votre ordinateur. Vous devez vous connecter à Azure avec un compte qui a les [autorisations](virtual-network-tap-overview.md#permissions) appropriées. Cet article nécessite la version 2.0.46 ou ultérieure d’Azure CLI. Exécutez `az --version` pour rechercher la version installée. Si vous devez installer ou mettre à niveau, consultez [Installation d’Azure CLI 2.0](/cli/azure/install-azure-cli). Le point d’accès terminal (TAP) de réseau virtuel est actuellement disponible sous forme d’extension. Pour installer l’extension, vous devez exécuter `az extension add -n virtual-network-tap`. Si vous exécutez Azure CLI localement, vous devez également exécuter `az login` pour créer une connexion avec Azure.

1. Récupérer l’ID de votre abonnement dans une variable, utilisée dans une étape ultérieure :

   ```azurecli-interactive
   subscriptionId=$(az account show \
   --query id \
   --out tsv)
   ```

2. Définissez l’ID d’abonnement à utiliser pour créer une ressource TAP de réseau virtuel.

   ```azurecli-interactive
   az account set --subscription $subscriptionId
   ```

3. Réinscrivez l’ID d’abonnement à utiliser pour créer une ressource TAP de réseau virtuel. Si vous obtenez une erreur d’inscription quand vous créez une ressource TAP, exécutez la commande suivante :

   ```azurecli-interactive
   az provider register --namespace Microsoft.Network --subscription $subscriptionId
   ```

4. Si la destination du TAP de réseau virtuel est l’interface réseau sur l’appliance virtuelle réseau d’un collecteur ou d’un outil analytique :

   - Récupérez la configuration IP de l’interface réseau de l’appliance virtuelle réseau dans une variable, utilisée dans une étape ultérieure. L’ID est le point de terminaison qui va agréger le trafic TAP. L’exemple suivant récupère l’ID de la configuration IP *ipconfig1* pour une interface réseau nommée *myNetworkInterface*, dans un groupe de ressources nommé *myResourceGroup* :

      ```azurecli-interactive
       IpConfigId=$(az network nic ip-config show \
       --name ipconfig1 \
       --nic-name myNetworkInterface \
       --resource-group myResourceGroup \
       --query id \
       --out tsv)
      ```

   - Créez le TAP de réseau virtuel dans la région Azure westcentralus à l’aide de l’ID de la configuration IP de destination et d’une propriété de port facultative. Le port spécifie le port de destination du trafic TAP sur la configuration IP d’interface réseau :  

      ```azurecli-interactive
       az network vnet tap create \
       --resource-group myResourceGroup \
       --name myTap \
       --destination $IpConfigId \
       --port 4789 \
       --location westcentralus
      ```

5. Si la destination du TAP de réseau virtuel est un équilibreur de charge interne Azure :
  
   - Récupérez la configuration IP frontend de l’équilibreur de charge interne Azure dans une variable, utilisée dans une étape ultérieure. L’ID est le point de terminaison qui va agréger le trafic TAP. L’exemple suivant récupère l’ID de la configuration IP frontend *frontendipconfig1* pour un équilibreur de charge nommé *myInternalLoadBalancer*, dans un groupe de ressources nommé  *myResourceGroup* :

      ```azurecli-interactive
      FrontendIpConfigId=$(az network lb frontend-ip show \
      --name frontendipconfig1 \
      --lb-name myInternalLoadBalancer \
      --resource-group myResourceGroup \
      --query id \
      --out tsv)
      ```

   - Créez le TAP de réseau virtuel à l’aide de l’ID de la configuration IP de destination et d’une propriété de port facultative. Le port spécifie le port de destination du trafic TAP sur la configuration IP frontend :  

      ```azurecli-interactive
      az network vnet tap create \
      --resource-group myResourceGroup \
      --name myTap \
      --destination $FrontendIpConfigId \
      --port 4789 \
     --location westcentralus
     ```

6. Confirmez la création du TAP de réseau virtuel :

   ```azurecli-interactive
   az network vnet tap show \
   --resource-group myResourceGroup
   --name myTap
   ```

## <a name="add-a-tap-configuration-to-a-network-interface"></a>Ajouter une configuration TAP à une interface réseau

1. Récupérez l’ID d’une ressource TAP de réseau virtuel existante. L’exemple suivant récupère un TAP de réseau virtuel nommé *myTap* dans un groupe de ressources nommé *myResourceGroup* :

   ```azurecli-interactive
   tapId=$(az network vnet tap show \
   --name myTap \
   --resource-group myResourceGroup \
   --query id \
   --out tsv)
   ```

2. Créez une configuration TAP sur l’interface réseau de la machine virtuelle supervisée. L’exemple suivant crée une configuration TAP pour une interface réseau nommée *myNetworkInterface* :

   ```azurecli-interactive
   az network nic vtap-config create \
   --resource-group myResourceGroup \
   --nic myNetworkInterface \
   --vnet-tap $tapId \
   --name mytapconfig \
   --subscription subscriptionId
   ```

3. Confirmez la création de la configuration TAP :

   ```azurecli-interactive
   az network nic vtap-config show \
   --resource-group myResourceGroup \
   --nic-name myNetworkInterface \
   --name mytapconfig \
   --subscription subscriptionId
   ```

## <a name="delete-the-tap-configuration-on-a-network-interface"></a>Supprimer la configuration TAP sur une interface réseau

   ```azurecli-interactive
   az network nic vtap-config delete \
   --resource-group myResourceGroup \
   --nic myNetworkInterface \
   --name myTapConfig \
   --subscription subscriptionId
   ```

## <a name="list-virtual-network-taps-in-a-subscription"></a>Lister les TAP de réseau virtuel dans un abonnement

   ```azurecli-interactive
   az network vnet tap list
   ```

## <a name="delete-a-virtual-network-tap-in-a-resource-group"></a>Supprimer un TAP de réseau virtuel dans un groupe de ressources

   ```azurecli-interactive
   az network vnet tap delete \
   --resource-group myResourceGroup \
   --name myTap
   ```

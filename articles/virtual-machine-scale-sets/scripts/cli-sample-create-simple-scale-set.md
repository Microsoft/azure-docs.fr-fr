---
title: Exemples Azure CLI - Créer un groupe de machines virtuelles identiques
description: Ce script permet de créer un groupe de machines virtuelles identiques Azure avec un système d’exploitation Ubuntu et les ressources réseau associées, y compris un équilibreur de charge.
author: mimckitt
ms.author: mimckitt
ms.topic: sample
ms.service: virtual-machine-scale-sets
ms.subservice: cli
ms.date: 06/25/2020
ms.reviewer: jushiman
ms.custom: mimckitt, devx-track-azurecli
ms.openlocfilehash: 31c90879d6f80f598ba71846ba68da814f254677
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91825181"
---
# <a name="create-a-virtual-machine-scale-set-with-the-azure-cli"></a>Créer un groupe de machines virtuelles identiques avec Azure CLI
Ce script permet de créer un groupe de machines virtuelles identiques Azure avec un système d’exploitation Ubuntu et les ressources réseau associées, y compris un équilibreur de charge. Une fois que vous avez exécuté le script, vous pouvez accéder aux instances de la machine virtuelle sur SSH.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Exemple de script
```azurecli-interactive
#!/bin/bash

# Create a resource group
az group create --name myResourceGroup --location eastus

# Create a Network Security Group and allow access to port 22
az network nsg create --resource-group MyResourceGroup --name MyNsg
az network nsg rule create --resource-group MyResourceGroup --name AllowSsh --nsg-name MyNsg --priority 100 --destination-port-ranges 22

# Create a scale set
# Network resources such as an Azure load balancer are automatically created
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --admin-username azureuser \
  --generate-ssh-keys
  --nsg MyNsg
```

## <a name="clean-up-deployment"></a>Nettoyer le déploiement
Exécutez la commande suivante pour supprimer le groupe de ressources, le groupe identique et toutes les ressources associées.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>Explication du script
Ce script utilise les commandes suivantes pour créer un groupe de ressources, un groupe de machines virtuelles identiques, ainsi que toutes les ressources associées. Chaque commande du tableau renvoie à une documentation spécifique.

| Commande | Notes |
|---|---|
| [az group create](/cli/azure/ad/group) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [az vmss create](/cli/azure/vmss) | Crée le groupe de machines virtuelles identiques et le connecte au réseau virtuel, au sous-réseau et au groupe de sécurité réseau. Un équilibreur de charge est également créé pour distribuer le trafic vers les différentes instances de machine virtuelle. Cette commande spécifie aussi l’image de machine virtuelle à utiliser ainsi que les informations d’identification d’administration.  |
| [az group delete](/cli/azure/ad/group) | Supprime un groupe de ressources, y compris toutes les ressources imbriquées. |

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur l’interface Azure CLI, consultez la [documentation relative à l’interface Azure CLI](/cli/azure/overview).

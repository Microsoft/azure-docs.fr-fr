---
title: Créer et configurer un plan Azure DDoS Protection à l’aide d’Azure CLI
description: Découvrez comment créer un plan de protection DDoS à l’aide d’Azure CLI
services: ddos-protection
documentationcenter: na
author: yitoh
ms.service: ddos-protection
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/28/2020
ms.author: yitoh
ms.openlocfilehash: 6c628d93c112a770c85a10d0eff958614a7cf4cb
ms.sourcegitcommit: 1140ff2b0424633e6e10797f6654359947038b8d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97814157"
---
# <a name="quickstart-create-and-configure-azure-ddos-protection-standard-using-azure-cli"></a>Démarrage rapide : Créer et configurer Azure DDoS Protection Standard à l’aide d’Azure CLI

Prise en main d’Azure DDoS Protection Standard à l’aide d’Azure CLI. 

Un plan de protection DDoS définit un ensemble de réseaux virtuels pour lesquels la protection DDoS standard est activée par le biais des abonnements. Vous pouvez configurer un plan de protection DDoS pour votre organisation et lier des réseaux virtuels à partir de plusieurs abonnements au même plan. 

Dans ce démarrage rapide, vous allez créer un plan de protection DDoS et le lier à un réseau virtuel. 

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Azure CLI installé localement ou Azure Cloud Shell

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si vous choisissez d’installer et d’utiliser l’interface CLI en local, ce guide de démarrage nécessite Azure CLI version 2.0.28 ou ultérieure. Pour connaître la version de l’interface, exécutez `az --version`. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI]( /cli/azure/install-azure-cli).

## <a name="create-a-ddos-protection-plan"></a>Créer un plan de protection DDoS

Dans Azure, vous allouez les ressources associées à un groupe de ressources. Vous pouvez utiliser un groupe de ressources existant ou en créer un.

Pour créer un groupe de ressources, utilisez la commande [az group create](/cli/azure/group?preserve-view=true&view=azure-cli-latest#az-group-create). Dans cet exemple, nous allons nommer notre groupe de ressources _MyResourceGroup_ et utiliser la région _USA Est_ :

```azurecli-interactive
az group create \
    --name MyResourceGroup \
    --location eastus
```

Créez maintenant un plan de protection DDoS nommé _MyDdosProtectionPlan_ :

```azurecli-interactive
az network ddos-protection create \
    --resource-group MyResourceGroup \
    --name MyDdosProtectionPlan
```

## <a name="enable-ddos-protection-for-a-virtual-network"></a>Activer la protection DDoS pour un réseau virtuel

### <a name="enable-ddos-protection-for-a-new-virtual-network"></a>Activer la protection DDoS pour un nouveau réseau virtuel

Vous pouvez activer la protection DDoS lors de la création d’un réseau virtuel. Dans cet exemple, nous allons nommer notre réseau virtuel _MyVnet_ : 

```azurecli-interactive
az network vnet create \
    --resource-group MyResourceGroup \
    --name MyVnet \
    --location eastus \
    --ddos-protection true
```

Vous ne pouvez pas déplacer un réseau virtuel vers un autre groupe de ressources ou abonnement quand la protection DDoS Standard est activée pour le réseau virtuel. Si vous devez déplacer un réseau virtuel pour lequel la protection DDoS Standard est activée, désactivez d’abord celle-ci, déplacez le réseau virtuel, puis activez la protection DDoS Standard. Après le déplacement, les seuils de stratégie réglée automatiquement pour toutes les adresses IP publiques protégées dans le réseau virtuel sont réinitialisés.

### <a name="enable-ddos-protection-for-an-existing-virtual-network"></a>Activer la protection DDoS pour un réseau virtuel existant

Lors de [la création d’un plan de protection DDoS](#create-a-ddos-protection-plan), vous pouvez associer un ou plusieurs réseaux virtuels au plan. Pour ajouter plusieurs réseaux virtuels, il suffit de répertorier les noms ou ID, séparés par une espace. Dans cet exemple, nous ajouterons _MyVnet_ :

```azurecli-interactive
az group create \
    --name MyResourceGroup \
    --location eastus

az network ddos-protection create \
    --resource-group MyResourceGroup \
    --name MyDdosProtectionPlan
    --vnet MyVnet
```

Vous pouvez également activer la protection DDoS pour un réseau virtuel donné :

```azurecli-interactive
az network vnet update \
    --resource-group MyResourceGroup \
    --vnet MyVnet \
    --ddos-protection true
```

## <a name="validate-and-test"></a>Valider et tester

Tout d’abord, vérifiez les détails de votre plan de protection DDoS :

```azurecli-interactive
az network ddos-protection show \
    --resource-group MyResourceGroup \
    --name MyDdosProtectionPlan
```

Vérifiez que la commande retourne les détails corrects de votre plan de protection DDoS.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez conserver vos ressources pour le tutoriel suivant. Si vous n’en avez plus besoin, supprimez le groupe de ressources _MyResourceGroup_. Quand vous supprimez le groupe de ressources, vous supprimez aussi le plan de protection DDoS et toutes ses ressources associées. 

Pour supprimer le groupe de ressources, utilisez la commande [az group delete](/cli/azure/group?preserve-view=true&view=azure-cli-latest#az_group_delete) :

```azurecli-interactive
az group delete \
--name MyResourceGroup 
```

Mettez à jour un réseau virtuel donné pour désactiver la protection DDoS :

```azurecli-interactive
az network vnet update \
    --resource-group MyResourceGroup \
    --vnet MyVnet \
    --ddos-protection false
```

Si vous souhaitez supprimer un plan de protection DDoS, vous devez d’abord dissocier tous les réseaux virtuels de celui-ci. 

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment consulter et configurer la télémétrie pour votre plan de protection DDoS, passez aux tutoriels.

> [!div class="nextstepaction"]
> [Consulter et configurer la télémétrie de la protection DDoS](telemetry.md)
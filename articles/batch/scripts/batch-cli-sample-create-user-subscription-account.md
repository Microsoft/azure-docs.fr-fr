---
title: Exemple de script Azure CLI - Créer un compte Batch - abonnement utilisateur
description: Ce script crée un compte Azure Batch en mode abonnement utilisateur. Ce compte alloue des nœuds de calcul dans votre abonnement.
ms.topic: sample
ms.date: 01/29/2018
ms.custom: devx-track-azurecli
ms.openlocfilehash: c9b8ba2ef782dcdc99cb18698175b8b53a53f0dd
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93076773"
---
# <a name="cli-example-create-a-batch-account-in-user-subscription-mode"></a>Exemple CLI : créer un compte Batch en mode abonnement utilisateur

Ce script crée un compte Azure Batch en mode abonnement utilisateur. Un compte qui alloue des nœuds de calcul dans votre abonnement doit être authentifié via un jeton Azure Active Directory. Les nœuds de calcul alloués sont pris en compte dans le quota de processeurs virtuels (cœur) de votre abonnement. 

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Ce tutoriel nécessite Azure CLI version 2.0.20 ou ultérieure. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.  

## <a name="example-script"></a>Exemple de script

[!code-azurecli-interactive[main](../../../cli_scripts/batch/create-account/create-account-user-subscription.sh "Create Account using user subscription")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement

Exécutez la commande suivante pour supprimer le groupe de ressources et toutes les ressources qui lui sont associées.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>Explication du script

Ce script utilise les commandes suivantes. Chaque commande du tableau renvoie à une documentation spécifique.

| Commande | Notes |
|---|---|
| [az role assignment create](/cli/azure/role) | Crée une nouvelle attribution de rôle pour un utilisateur, un groupe ou un principal de service. |
| [az group create](/cli/azure/group#az-group-create) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [az keyvault create](/cli/azure/keyvault#az-keyvault-create) | Crée un coffre de clés. |
| [az keyvault set-policy](/cli/azure/keyvault#az-keyvault-set-policy) | Met à jour la stratégie de sécurité du coffre de clés spécifié. |
| [az batch account create](/cli/azure/batch/account#az-batch-account-create) | Crée le compte Batch.  |
| [az batch account login](/cli/azure/batch/account#az-batch-account-login) | Effectue l’authentification par rapport au compte Batch spécifié pour renforcer les interactions avec la CLI.  |
| [az group delete](/cli/azure/group#az-group-delete) | Supprime un groupe de ressources, y compris toutes les ressources imbriquées. |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’interface Azure CLI, consultez la [documentation relative à l’interface Azure CLI](/cli/azure).

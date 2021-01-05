---
title: Créer une application de fonction avec déploiement de DevOps - Azure CLI
description: Créer une application de fonction Function App et déployer le code de fonction à partir d’Azure DevOps
ms.date: 07/03/2018
ms.topic: sample
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 25e0f43a851bbf7658847aa52cc9ee778941f718
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94565209"
---
# <a name="create-a-function-in-azure-that-is-deployed-from-azure-devops"></a>Créer une fonction dans Azure déployée à partir d’Azure DevOps

Cette rubrique vous explique comment utiliser Azure Functions pour créer une application de fonction [serverless](https://azure.microsoft.com/solutions/serverless/) à l’aide du [plan Consommation](../functions-scale.md#consumption-plan). L’application de fonction, qui est un conteneur pour vos fonctions, est déployée en continu à partir d’un dépôt Azure DevOps. 

Dans le cadre de cette rubrique, vous avez besoin des éléments suivants :

* Un dépôt Azure DevOps qui contient votre projet d’application de fonction, et pour lequel vous disposez d’autorisations d’administration.
* Un [PAT (jeton d’accès personnel)](/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate) qui permet d’accéder au dépôt Azure DevOps.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Ce tutoriel nécessite l’interface Azure CLI version 2.0 ou ultérieure. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée. 

## <a name="sample-script"></a>Exemple de script

Cet exemple crée une application de fonction Azure et déploie le code de fonction à partir d’Azure DevOps.

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/deploy-function-app-with-function-vsts/deploy-function-app-with-function-vsts.sh?highlight=3-4 "Azure Service")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explication du script

Ce script utilise les commandes suivantes pour créer un groupe de ressources, un compte de stockage, une application de fonction et toutes les ressources associées. Chaque commande du tableau renvoie à une documentation spécifique.

| Commande | Notes |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [az storage account create](/cli/azure/storage/account#az-storage-account-create) | Crée le compte de stockage requis par l’application de fonction. |
| [az functionapp create](/cli/azure/functionapp#az-functionapp-create) | Crée une application de fonction dans le [plan Consommation](../functions-scale.md#consumption-plan) serverless. |
| [az functionapp deployment source config](/cli/azure/functionapp/deployment/source#az-functionapp-deployment-source-config) | Associe une Function App à un référentiel Git ou Mercurial. |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’interface Azure CLI, consultez la [documentation relative à l’interface Azure CLI](/cli/azure).

Vous trouverez des exemples supplémentaires de scripts CLI Azure Functions dans la [documentation d’Azure Functions](../functions-cli-samples.md).

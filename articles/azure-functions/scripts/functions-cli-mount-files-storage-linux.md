---
title: Monter un partage de fichiers dans une application de fonction Python - Azure CLI
description: Créez une application de fonction Python serverless et montez un partage de fichiers existant à l’aide d’Azure CLI.
ms.topic: sample
ms.date: 03/01/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: bdcaeaca7c063f0532167077bba63f7e52a3d491
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94565056"
---
# <a name="mount-a-file-share-to-a-python-function-app-using-azure-cli"></a>Monter un partage de fichiers dans une application de fonction Python à l’aide d’Azure CLI

Cet exemple de script Azure Functions crée une application de fonction, ainsi qu’un partage dans Azure Files. Il monte ensuite le partage afin que les fonctions soient accessibles par vos fonctions.  

>[!NOTE]
>L’application de fonction créée s’exécute sur Python version 3.7. Azure Functions [prend également en charge les versions 3.6 et 3.8 de Python](../functions-reference-python.md#python-version).

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Ce tutoriel nécessite l’interface Azure CLI version 2.0 ou ultérieure. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée. 

## <a name="sample-script"></a>Exemple de script

Ce script crée une application de fonction Azure à l’aide du [plan Consommation](../functions-scale.md#consumption-plan).

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/functions-cli-mount-files-storage-linux/functions-cli-mount-files-storage-linux.sh "Create an Azure Function on a Consumption plan")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explication du script

Chaque commande du tableau renvoie à une documentation spécifique. Ce script utilise les commandes suivantes :

| Commande | Notes |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [az storage account create](/cli/azure/storage/account#az-storage-account-create) | Crée un compte de stockage Azure. |
| [az functionapp create](/cli/azure/functionapp#az-functionapp-create) | Crée une Function App. |
| [az storage share create](/cli/azure/storage/share#az-storage-share-create) | Crée un partage Azure Files dans le compte de stockage. | 
| [az storage directory create](/cli/azure/storage/directory#az-storage-directory-create) | Créez un répertoire dans le partage. |
| [az webapp config storage-account add](/cli/azure/webapp/config/storage-account#az-webapp-config-storage-account-add) | Monte le partage dans l’application de fonction. |
| [az webapp config storage-account list](/cli/azure/webapp/config/storage-account#az-webapp-config-storage-account-list) | Affiche les partages de fichiers montés dans l’application de fonction. | 

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’interface Azure CLI, consultez la [documentation relative à l’interface Azure CLI](/cli/azure).

Vous trouverez des exemples supplémentaires de scripts CLI Azure Functions dans la [documentation d’Azure Functions](../functions-cli-samples.md).

---
title: Exemple de script Azure CLI - Exécuter une tâche Batch
description: Ce script crée une tâche Batch et ajoute une série de tâches au travail. Il explique également comment surveiller un travail et ses tâches.
ms.topic: sample
ms.date: 12/12/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 8e51fa96370b46c9a5a5d5ed6fd7593a06e45430
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/20/2021
ms.locfileid: "107768158"
---
# <a name="cli-example-run-a-job-and-tasks-with-azure-batch"></a>Exemple CLI : Exécution d’un travail et de tâches avec Azure Batch

Ce script crée une tâche Batch et ajoute une série de tâches au travail. Il explique également comment surveiller un travail et ses tâches. 

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Ce tutoriel nécessite Azure CLI version 2.0.20 ou ultérieure. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée. 

## <a name="example-script"></a>Exemple de script

[!code-azurecli-interactive[main](../../../cli_scripts/batch/run-job/run-job.sh "Run Job")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement

Exécutez la commande suivante pour supprimer le groupe de ressources et toutes les ressources qui lui sont associées.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>Explication du script

Ce script utilise les commandes suivantes. Chaque commande du tableau renvoie à une documentation spécifique.

| Commande | Notes |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [az batch account create](/cli/azure/batch/account#az_batch_account_create) | Crée le compte Batch. |
| [az batch account login](/cli/azure/batch/account#az_batch_account_login) | Effectue l’authentification par rapport au compte Batch spécifié pour renforcer les interactions avec la CLI.  |
| [az batch pool create](/cli/azure/batch/pool#az_batch_pool_create) | Crée un pool de nœuds de calcul.  |
| [az batch job create](/cli/azure/batch/job#az_batch_job_create) | Crée un travail Batch.  |
| [az batch task create](/cli/azure/batch/task#az_batch_task_create) | Ajoute une tâche au travail Batch spécifié.  |
| [az batch job set](/cli/azure/batch/job#az_batch_job_set) | Met à jour les propriétés d’un travail Batch.  |
| [az batch job show](/cli/azure/batch/job#az_batch_job_show) | Récupère les détails relatifs au travail Batch spécifié.  |
| [az batch task show](/cli/azure/batch/task#az_batch_task_show) | Récupère les détails relatifs à une tâche du travail Batch spécifié.  |
| [az group delete](/cli/azure/group#az_group_delete) | Supprime un groupe de ressources, y compris toutes les ressources imbriquées. |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’interface Azure CLI, consultez la [documentation relative à l’interface Azure CLI](/cli/azure).

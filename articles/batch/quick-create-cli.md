---
title: Démarrage rapide - exécution de votre premier travail Batch avec l’interface de ligne de commande (CLI) Azure
description: Ce guide de démarrage rapide montre comment créer un compte Batch et exécuter un travail Batch avec l’interface Azure CLI.
ms.topic: quickstart
ms.date: 08/13/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 297af47b6280381646e654eaededfe8b71a5d874
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97106680"
---
# <a name="quickstart-run-your-first-batch-job-with-the-azure-cli"></a>Démarrage rapide : Exécuter votre premier travail Batch avec Azure CLI

Démarrez avec Azure Batch à l’aide de l’interface de ligne de commande Azure pour créer un compte Batch, un pool de nœuds de calcul (machines virtuelles) et un travail qui exécute des tâches sur le pool. Chaque exemple de tâche exécute une commande de base sur un des nœuds du pool.

L’interface de ligne de commande (CLI) Azure permet de créer et gérer des ressources Azure à partir de la ligne de commande ou dans les scripts. À l’issue de ce démarrage rapide, vous maîtriserez les concepts clés du service Batch et serez prêt à essayer Azure Batch avec des charges de travail plus réalistes à plus grande échelle.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Pour ce guide de démarrage rapide, vous devez disposer de la version 2.0.20 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group#az-group-create). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.

L’exemple suivant crée un groupe de ressources nommé *QuickstartBatch-rg* à l’emplacement *eastus2*.

```azurecli-interactive
az group create \
    --name QuickstartBatch-rg \
    --location eastus2
```

## <a name="create-a-storage-account"></a>Créez un compte de stockage.

Vous pouvez lier un compte Stockage Azure à votre compte Batch. Bien que ce ne soit pas obligatoire pour ce démarrage rapide, le compte de stockage est utile pour déployer des applications et stocker des données d’entrée et de sortie pour la plupart des charges de travail réelles. Créez un compte de stockage dans votre groupe de ressources avec la commande [az storage account create](/cli/azure/storage/account#az-storage-account-create).

```azurecli-interactive
az storage account create \
    --resource-group QuickstartBatch-rg \
    --name mystorageaccount \
    --location eastus2 \
    --sku Standard_LRS
```

## <a name="create-a-batch-account"></a>Création d’un compte Batch

Créez un compte Batch avec la commande [az batch account create](/cli/azure/batch/account#az-batch-account-create). Vous avez besoin d’un compte pour créer des ressources de calcul (pools de nœuds de calcul) et des travaux Batch.

L’exemple suivant crée un compte Batch nommé *mybatchaccount* dans *QuickstartBatch-rg* et lie le compte de stockage que vous avez créé.  

```azurecli-interactive
az batch account create \
    --name mybatchaccount \
    --storage-account mystorageaccount \
    --resource-group QuickstartBatch-rg \
    --location eastus2
```

Pour créer et gérer des pools de calcul et des travaux, vous devez vous authentifier avec Azure Batch. Connectez-vous au compte avec la commande [az batch account login](/cli/azure/batch/account#az-batch-account-login). Une fois que vous êtes connecté, vos commandes `az batch` utilisent ce contexte de compte.

```azurecli-interactive
az batch account login \
    --name mybatchaccount \
    --resource-group QuickstartBatch-rg \
    --shared-key-auth
```

## <a name="create-a-pool-of-compute-nodes"></a>Créer un pool de nœuds de calcul

Maintenant que vous avez un compte Batch, créez un pool d’exemple de nœuds de calcul Linux à l’aide de la commande [az batch pool create](/cli/azure/batch/pool#az-batch-pool-create). L’exemple suivant crée un pool appelé *mypool* avec 2 nœuds de taille *Standard_A1_v2* exécutant Ubuntu 16.04 LTS. La taille de nœud suggérée offre un bon compromis entre performances et coûts pour cet exemple rapide.
 
```azurecli-interactive
az batch pool create \
    --id mypool --vm-size Standard_A1_v2 \
    --target-dedicated-nodes 2 \
    --image canonical:ubuntuserver:16.04-LTS \
    --node-agent-sku-id "batch.node.ubuntu 16.04"
```

Azure Batch crée le pool immédiatement, mais prend quelques minutes pour allouer et démarrer les nœuds de calcul. Pendant ce temps, le pool est à l’état `resizing`. Pour afficher l’état du pool, exécutez la commande [az batch pool show](/cli/azure/batch/pool#az-batch-pool-show). Cette commande affiche toutes les propriétés du pool, vous pouvez également rechercher des propriétés spécifiques. La commande suivante obtient l’état de l’allocation du pool :

```azurecli-interactive
az batch pool show --pool-id mypool \
    --query "allocationState"
```

Continuez les étapes suivantes pour créer un travail et des tâches durant la modification de l’état du pool. Le pool est prêt à exécuter des tâches lorsque l’état de l’allocation est `steady` et que tous les nœuds sont en cours d’exécution.

## <a name="create-a-job"></a>Créer un travail

Maintenant que vous disposez d’un pool, créez un travail à exécuter sur celui-ci. Un travail Batch est un groupe logique d’une ou de plusieurs tâches. Un travail inclut les paramètres communs aux tâches, tels que la priorité et le pool pour exécuter des tâches. Créez un travail Batch à l’aide de la commande [az batch job create](/cli/azure/batch/job#az-batch-job-create). L’exemple suivant crée un travail *myjob* sur le pool *mypool*. Dans un premier temps, le travail n’a aucune tâche.

```azurecli-interactive
az batch job create \
    --id myjob \
    --pool-id mypool
```

## <a name="create-tasks"></a>Créer des tâches

Utilisez maintenant la commande [az batch task create](/cli/azure/batch/task#az-batch-task-create) pour créer des tâches à exécuter dans le travail. Dans cet exemple, vous créez quatre tâches identiques. Chaque tâche exécute une `command-line` pour afficher les variables d’environnement Azure Batch sur un nœud de calcul, puis attend 90 secondes. Lorsque vous utilisez Azure Batch, cette ligne de commande se trouve là où vous spécifiez votre application ou votre script. Azure Batch fournit plusieurs façons de déployer des applications et des scripts sur des nœuds de calcul.

Le script Bash suivant crée 4 tâches parallèles (*mytask1* à *mytask4*).

```azurecli-interactive
for i in {1..4}
do
   az batch task create \
    --task-id mytask$i \
    --job-id myjob \
    --command-line "/bin/bash -c 'printenv | grep AZ_BATCH; sleep 90s'"
done
```

La sortie de commande affiche les paramètres pour chaque tâche. Azure Batch distribue les tâches pour les nœuds de calcul.

## <a name="view-task-status"></a>Affichage de l’état de la tâche

Après avoir créé une tâche, Azure Batch la met en file d’attente pour l’exécuter sur le pool. Une fois qu’un nœud est disponible pour l’exécuter, la tâche s’exécute.

Utilisez la commande [az batch task show](/cli/azure/batch/task#az-batch-task-show) pour afficher l’état des tâches Azure Batch. L’exemple suivant affiche des détails sur *mytask1* en cours d’exécution sur un des nœuds du pool.

```azurecli-interactive
az batch task show \
    --job-id myjob \
    --task-id mytask1
```

La sortie de la commande inclut de nombreux détails, mais notez le `exitCode` de la ligne de commande de la tâche et le `nodeId`. Un `exitCode` de 0 indique que la ligne de commande de la tâche s’est correctement effectuée. Le `nodeId` indique l’ID du nœud de pool sur lequel la tâche a été exécutée.

## <a name="view-task-output"></a>Afficher la sortie des tâches

Pour répertorier les fichiers créés par une tâche sur un nœud de calcul, utilisez la commande [az batch task file list](/cli/azure/batch/task). La commande suivante répertorie les fichiers créés par *mytask1* :

```azurecli-interactive
az batch task file list \
    --job-id myjob \
    --task-id mytask1 \
    --output table
```

Le résultat ressemble à ce qui suit :

```
Name        URL                                                                                         Is Directory      Content Length
----------  ------------------------------------------------------------------------------------------  --------------  ----------------
stdout.txt  https://mybatchaccount.eastus2.batch.azure.com/jobs/myjob/tasks/mytask1/files/stdout.txt  False                  695
certs       https://mybatchaccount.eastus2.batch.azure.com/jobs/myjob/tasks/mytask1/files/certs       True
wd          https://mybatchaccount.eastus2.batch.azure.com/jobs/myjob/tasks/mytask1/files/wd          True
stderr.txt  https://mybatchaccount.eastus2.batch.azure.com/jobs/myjob/tasks/mytask1/files/stderr.txt  False                     0

```

Pour télécharger un des fichiers de sortie dans un répertoire local, utilisez la commande [az batch task file download](/cli/azure/batch/task). Dans cet exemple, la sortie de la tâche est au format `stdout.txt`.

```azurecli-interactive
az batch task file download \
    --job-id myjob \
    --task-id mytask1 \
    --file-path stdout.txt \
    --destination ./stdout.txt
```

Vous pouvez afficher le contenu de `stdout.txt` dans un éditeur de texte. Le contenu affiche les variables d’environnement Azure Batch qui sont définies sur le nœud. Lorsque vous créez vos propres travaux Batch, vous pouvez référencer ces variables d’environnement dans des lignes de commande de tâche, ainsi que dans les applications et les scripts exécutés par les lignes de commande. Par exemple :

```
AZ_BATCH_TASK_DIR=/mnt/batch/tasks/workitems/myjob/job-1/mytask1
AZ_BATCH_NODE_STARTUP_DIR=/mnt/batch/tasks/startup
AZ_BATCH_CERTIFICATES_DIR=/mnt/batch/tasks/workitems/myjob/job-1/mytask1/certs
AZ_BATCH_ACCOUNT_URL=https://mybatchaccount.eastus2.batch.azure.com/
AZ_BATCH_TASK_WORKING_DIR=/mnt/batch/tasks/workitems/myjob/job-1/mytask1/wd
AZ_BATCH_NODE_SHARED_DIR=/mnt/batch/tasks/shared
AZ_BATCH_TASK_USER=_azbatch
AZ_BATCH_NODE_ROOT_DIR=/mnt/batch/tasks
AZ_BATCH_JOB_ID=myjobl
AZ_BATCH_NODE_IS_DEDICATED=true
AZ_BATCH_NODE_ID=tvm-257509324_2-20180703t215033z
AZ_BATCH_POOL_ID=mypool
AZ_BATCH_TASK_ID=mytask1
AZ_BATCH_ACCOUNT_NAME=mybatchaccount
AZ_BATCH_TASK_USER_IDENTITY=PoolNonAdmin
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez poursuivre les exemples et didacticiels Azure Batch, utilisez le compte Batch et le compte de stockage lié créés dans ce démarrage rapide. Il n’existe aucun frais pour le compte Batch proprement dit.

Vous êtes facturé pour les pools pendant que les nœuds sont en cours d’exécution, même si aucun travail n’est planifié. Lorsque vous n’avez plus besoin d’un pool, supprimez-le avec la commande [az batch pool delete](/cli/azure/batch/pool#az-batch-pool-delete). Lorsque vous supprimez le pool, toutes les sorties de tâche sur les nœuds sont supprimées.

```azurecli-interactive
az batch pool delete --pool-id mypool
```

Quand vous n’en avez plus besoin, vous pouvez utiliser la commande [az group delete](/cli/azure/group#az-group-delete) pour supprimer le groupe de ressources, le compte Batch, les pools et toutes les ressources associées. Supprimez les ressources comme suit :

```azurecli-interactive
az group delete --name QuickstartBatch-rg
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un compte Batch, un pool Batch et un travail Batch. Le travail a exécuté des tâches d’exemple et vous avez affiché des sorties créées sur un des nœuds. Maintenant que vous maîtriserez les concepts clés du service Batch, vous êtres prêt à essayer Azure Batch avec des charges de travail plus réalistes à plus grande échelle. Pour plus d’informations sur Microsoft Azure Batch, consultez les didacticiels Azure Batch.

> [!div class="nextstepaction"]
> [Didacticiels Azure Batch](./tutorial-parallel-dotnet.md)

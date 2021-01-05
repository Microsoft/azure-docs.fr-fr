---
title: Tutoriel - Effectuer le rendu d’une scène dans le cloud
description: 'Didacticiel : comment créer le rendu d’une scène Autodesk 3ds Max avec Arnold à l’aide du service Azure Batch Rendering et de l’interface de ligne de commande Azure'
ms.topic: tutorial
ms.date: 03/05/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: e0858e838ba73862ef7f15040915c5f5cd3c751b
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97106340"
---
# <a name="tutorial-render-a-scene-with-azure-batch"></a>Tutoriel : Créer le rendu d’une scène avec Azure Batch 

Azure Batch propose des fonctionnalités de création de rendus à l’échelle du cloud, sur une base de paiement à l’utilisation. Azure Batch prend en charge les applications de rendu, notamment Autodesk Maya, 3ds Max, Arnold et V-Ray. Ce didacticiel vous montre les étapes permettant de créer le rendu d’une scène avec Batch à l’aide de l’interface de ligne de commande de Azure. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Charger une scène sur le Stockage Azure
> * Créer un pool Batch pour créer le rendu
> * Créer le rendu d’une scène à image unique
> * Mettre à l’échelle le pool et créer le rendu d’une scène comportant plusieurs images
> * Télécharger la sortie rendue

Dans ce didacticiel, vous créer le rendu d’une scène 3ds Max avec Batch à l’aide du convertisseur [Arnold](https://www.autodesk.com/products/arnold/overview) à lancer de rayon. Le pool Batch utilise une image de la Place de marché Azure avec des graphiques préinstallés et des applications de rendu qui fournissent des licences de paiement à l’utilisation.

## <a name="prerequisites"></a>Prérequis

 - Vous avez besoin d’un abonnement de paiement à l’utilisation ou autre option d’achat Azure pour utiliser les applications de rendu dans Batch sur une base de paiement à l’utilisation. **Les licences de paiement à l’utilisation ne sont pas prises en charge si vous utilisez une offre Azure gratuite qui propose un crédit monétaire.**

 - La scène 3ds Max pour ce didacticiel se trouve sur [GitHub](https://github.com/Azure/azure-docs-cli-python-samples/tree/master/batch/render-scene), ainsi qu’un exemple de script Batch et les fichiers de configuration JSON. La scène 3ds Max est issue des [exemples de fichiers Autodesk 3ds Max](https://download.autodesk.com/us/support/files/3dsmax_sample_files/2017/Autodesk_3ds_Max_2017_English_Win_Samples_Files.exe). (Les exemples de fichier Autodesk 3ds Max sont disponibles sous licence Commons Attribution-NonCommercial-Share Alike. Copyright &copy; Autodesk, Inc.)

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- Ce tutoriel nécessite Azure CLI version 2.0.20 ou ultérieure. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

> [!TIP]
> Vous pouvez afficher les [modèles de travail Arnold](https://github.com/Azure/batch-extension-templates/tree/master/templates/arnold/render-windows-frames) dans le dépôt GitHub de modèles d’extension Azure Batch.
## <a name="create-a-batch-account"></a>Création d’un compte Batch

Si vous ne l’avez pas encore fait, créez un groupe de ressources, un compte Batch et un compte de stockage lié dans votre abonnement. 

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group#az-group-create). L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus2*.

```azurecli-interactive 
az group create \
    --name myResourceGroup \
    --location eastus2
```

Créez un compte Azure Storage dans votre groupe de ressources avec la commande [az storage account create](/cli/azure/storage/account#az-storage-account-create). Pour ce didacticiel, vous utilisez le compte de stockage pour stocker une scène 3ds Max d’entrée et la sortie rendue.

```azurecli-interactive
az storage account create \
    --resource-group myResourceGroup \
    --name mystorageaccount \
    --location eastus2 \
    --sku Standard_LRS
```
Créez un compte Batch avec la commande [az batch account create](/cli/azure/batch/account#az-batch-account-create). L’exemple suivant crée un compte Batch nommé *mybatchaccount* dans *myResourceGroup* et lie le compte de stockage que vous avez créé.  

```azurecli-interactive 
az batch account create \
    --name mybatchaccount \
    --storage-account mystorageaccount \
    --resource-group myResourceGroup \
    --location eastus2
```

Pour créer et gérer des pools de calcul et des travaux, vous devez vous authentifier avec Azure Batch. Connectez-vous au compte avec la commande [az batch account login](/cli/azure/batch/account#az-batch-account-login). Une fois que vous êtes connecté, vos commandes `az batch` utilisent ce contexte de compte. L’exemple suivant utilise l’authentification par clé partagée, basée sur le nom et la clé du compte Batch. Batch prend également en charge l’authentification via [Azure Active Directory](batch-aad-auth.md) pour authentifier des utilisateurs individuels ou une application sans assistance.

```azurecli-interactive 
az batch account login \
    --name mybatchaccount \
    --resource-group myResourceGroup \
    --shared-key-auth
```
## <a name="upload-a-scene-to-storage"></a>Charger une scène sur le stockage

Pour charger la séquence d’entrée sur le stockage, vous devez d’abord accéder au compte de stockage et créer un conteneur de destination pour les objets Blob. Pour accéder au compte de stockage Azure, exportez les variables d’environnement `AZURE_STORAGE_KEY` et `AZURE_STORAGE_ACCOUNT`. La première commande de l’interpréteur de commandes Bash utilise la commande [az storage account keys list](/cli/azure/storage/account/keys#az-storage-account-keys-list) pour obtenir la première clé de compte. Après avoir défini ces variables d’environnement, vos commandes de stockage utilisent ce contexte de compte.

```azurecli-interactive
export AZURE_STORAGE_KEY=$(az storage account keys list --account-name mystorageaccount --resource-group myResourceGroup -o tsv --query [0].value)

export AZURE_STORAGE_ACCOUNT=mystorageaccount
```

Créez maintenant un conteneur d’objets Blob dans le compte de stockage pour les fichiers de scène. L’exemple suivant utilise la commande [az storage container create](/cli/azure/storage/container#az-storage-container-create) pour créer un conteneur d’objets blob nommé *scenefiles* qui permet l’accès en lecture public.

```azurecli-interactive
az storage container create \
    --public-access blob \
    --name scenefiles
```

Téléchargez la scène `MotionBlur-Dragon-Flying.max` depuis [GitHub](https://github.com/Azure/azure-docs-cli-python-samples/raw/master/batch/render-scene/MotionBlur-DragonFlying.max) vers un répertoire de travail local. Par exemple :

```azurecli-interactive
wget -O MotionBlur-DragonFlying.max https://github.com/Azure/azure-docs-cli-python-samples/raw/master/batch/render-scene/MotionBlur-DragonFlying.max
```

Chargez le fichier de scène à partir de votre répertoire de travail local vers le conteneur d’objets blob. L’exemple suivant utilise la commande [az storage blob upload-batch](/cli/azure/storage/blob#az-storage-blob-upload-batch), qui permet de charger plusieurs fichiers :

```azurecli-interactive
az storage blob upload-batch \
    --destination scenefiles \
    --source ./
```

## <a name="create-a-rendering-pool"></a>Créer un pool de rendu

Créez un pool Batch pour le rendu à l’aide de la commande [az batch pool create](/cli/azure/batch/pool#az-batch-pool-create). Dans cet exemple, vous spécifiez les paramètres du pool dans un fichier JSON. Dans l’interpréteur de commandes actuel, créez un fichier nommé *mypool.json* et copiez-collez le contenu suivant. Vérifiez que tout le texte est correctement copié. Vous pouvez télécharger le fichier à partir de [GitHub](https://raw.githubusercontent.com/Azure/azure-docs-cli-python-samples/master/batch/render-scene/json/mypool.json).


```json
{
  "id": "myrenderpool",
  "vmSize": "standard_d2_v2",
  "virtualMachineConfiguration": {
    "imageReference": {
      "publisher": "batch",
      "offer": "rendering-windows2016",
      "sku": "rendering",
      "version": "1.3.8"
    },
    "nodeAgentSKUId": "batch.node.windows amd64"
  },
  "targetDedicatedNodes": 0,
  "targetLowPriorityNodes": 1,
  "enableAutoScale": false,
  "applicationLicenses":[
         "3dsmax",
         "arnold"
      ],
  "enableInterNodeCommunication": false 
}
```
Azure Batch prend en charge les nœuds dédiés et les [nœuds de faible priorité](batch-low-pri-vms.md) que vous pouvez utiliser dans vos pools. Les nœuds dédiés sont réservés à votre pool. Les nœuds de faible priorité sont proposés à prix réduit à partir de la capacité de machine virtuelle excédentaire dans Azure. Les nœuds de faible priorité deviennent indisponibles si la capacité d’Azure est insuffisante. 

Le pool spécifié contient un seul nœud de faible priorité exécutant une image Windows Server avec le logiciel pour le service Batch Rendering. Ce pool est concédé sous licence pour créer le rendu avec 3ds Max et Arnold. Au cours d’une étape ultérieure, vous élargissez le pool à un plus grand nombre de nœuds.

Créez le pool en transmettant le fichier JSON à la commande `az batch pool create` :

```azurecli-interactive
az batch pool create \
    --json-file mypool.json
``` 
L’approvisionnement du pool prend quelques minutes. Pour afficher l’état du pool, exécutez la commande [az batch pool show](/cli/azure/batch/pool#az-batch-pool-show). La commande suivante obtient l’état de l’allocation du pool :

```azurecli-interactive
az batch pool show \
    --pool-id myrenderpool \
    --query "allocationState"
```

Continuez les étapes suivantes pour créer un travail et des tâches durant la modification de l’état du pool. Le pool est totalement approvisionné lorsque l’état de l’allocation est `steady` et que les nœuds sont en cours d’exécution.  

## <a name="create-a-blob-container-for-output"></a>Créer un conteneur d’objets blob pour la sortie

Dans les exemples de ce didacticiel, chaque tâche dans le travail de rendu crée un fichier de sortie. Avant de planifier le travail, créez un conteneur d’objets blob dans votre compte de stockage comme destination pour les fichiers de sortie. L’exemple suivant utilise la commande [az storage container create](/cli/azure/storage/container#az-storage-container-create) pour créer un conteneur *job-myrenderjob* qui permet l’accès en lecture public. 

```azurecli-interactive
az storage container create \
    --public-access blob \
    --name job-myrenderjob
```

Pour écrire des fichiers de sortie dans le conteneur, Batch doit utiliser un jeton de signature d’accès partagé (SAS). Créez le jeton avec la commande [az storage account generate-sas](/cli/azure/storage/account#az-storage-account-generate-sas). Cet exemple crée un jeton à écrire dans un conteneur d’objets blob du compte, et ce jeton expire le 15 novembre 2020 :

```azurecli-interactive
az storage account generate-sas \
    --permissions w \
    --resource-types co \
    --services b \
    --expiry 2020-11-15
```

Prenez note du jeton renvoyé par la commande, qui ressemble à ce qui suit. Vous utilisez ce jeton ultérieurement.

```
se=2020-11-15&sp=rw&sv=2019-09-24&ss=b&srt=co&sig=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## <a name="render-a-single-frame-scene"></a>Créer le rendu d’une scène à image unique

### <a name="create-a-job"></a>Créer un travail

Créer un travail de rendu à exécuter sur le pool à l’aide de la commande [az batch job create](/cli/azure/batch/job#az-batch-job-create). Dans un premier temps, le travail n’a aucune tâche.

```azurecli-interactive
az batch job create \
    --id myrenderjob \
    --pool-id myrenderpool
```

### <a name="create-a-task"></a>Créer une tâche

Utilisez la commande [az batch task create](/cli/azure/batch/task#az-batch-task-create) pour créer une tâche à exécuter dans le travail. Dans cet exemple, vous spécifiez les paramètres de tâche pool dans un fichier JSON. Dans l’interpréteur de commandes actuel, créez un fichier nommé *myrendertask.json* et copiez-collez le contenu suivant. Vérifiez que tout le texte est correctement copié. Vous pouvez télécharger le fichier à partir de [GitHub](https://raw.githubusercontent.com/Azure/azure-docs-cli-python-samples/master/batch/render-scene/json/myrendertask.json).

La tâche spécifie une commande 3ds Max pour créer le rendu d’une seule image de la scène *MotionBlur-DragonFlying.max*.

Modifiez les éléments `blobSource` et `containerURL` du JSON fichier afin qu’ils contiennent le nom de votre compte de stockage et de votre jeton SAP. 

> [!TIP]
> Votre `containerURL` se termine par votre jeton SAP et ressemble à cela :
> 
> ```
> https://mystorageaccount.blob.core.windows.net/job-myrenderjob/$TaskOutput?se=2018-11-15&sp=rw&sv=2017-04-17&ss=b&srt=co&sig=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
> ```

```json
{
  "id": "myrendertask",
  "commandLine": "cmd /c \"%3DSMAX_2018%3dsmaxcmdio.exe -secure off -v:5 -rfw:0 -start:1 -end:1 -outputName:\"dragon.jpg\" -w 400 -h 300 MotionBlur-DragonFlying.max\"",
  "resourceFiles": [
    {
        "httpUrl": "https://mystorageaccount.blob.core.windows.net/scenefiles/MotionBlur-DragonFlying.max",
        "filePath": "MotionBlur-DragonFlying.max"
    }
  ],
    "outputFiles": [
        {
            "filePattern": "dragon*.jpg",
            "destination": {
                "container": {
                    "containerUrl": "https://mystorageaccount.blob.core.windows.net/job-myrenderjob/myrendertask/$TaskOutput?Add_Your_SAS_Token_Here"
                }
            },
            "uploadOptions": {
                "uploadCondition": "TaskSuccess"
            }
        }
    ],
  "userIdentity": {
    "autoUser": {
      "scope": "task",
      "elevationLevel": "nonAdmin"
    }
  }
}
```

Ajoutez la tâche au travail avec la commande suivante :

```azurecli-interactive
az batch task create \
    --job-id myrenderjob \
    --json-file myrendertask.json
```

Batch planifie la tâche et celle-ci s’exécute dès qu’un nœud du pool est disponible.


### <a name="view-task-output"></a>Afficher la sortie des tâches

L’exécution d’une tâche prend quelques minutes. Utilisez la commande [az batch task show](/cli/azure/batch/task#az-batch-task-show) pour afficher des détails sur la tâche.

```azurecli-interactive
az batch task show \
    --job-id myrenderjob \
    --task-id myrendertask
```

La tâche génère *dragon0001.jpg* sur le nœud de calcul et le charge dans le conteneur *job-myrenderjob* dans votre compte de stockage. Pour afficher la sortie, téléchargez le fichier à partir du stockage se trouvant sur l’ordinateur local à l’aide de la commande [az storage blob download](/cli/azure/storage/blob#az-storage-blob-download).

```azurecli-interactive
az storage blob download \
    --container-name job-myrenderjob \
    --file dragon.jpg \
    --name dragon0001.jpg

```

Ouvrez *dragon.jpg* sur votre ordinateur. L’image rendue ressemble à ce qui suit :

![Rendu de l’image de dragon 1](./media/tutorial-rendering-cli/dragon-frame.png) 


## <a name="scale-the-pool"></a>Mettre le pool à l’échelle

À présent modifiez le pool pour vous préparer à une tâche de rendu plus vaste à plusieurs images. Batch propose plusieurs modes de mise à l’échelle des ressources de calcul, y compris la [mise à l’échelle automatique](batch-automatic-scaling.md) qui ajoute ou supprime des nœuds à mesure que les demandes de tâche évoluent. Pour cet exemple de base, utilisez la commande [az batch pool resize](/cli/azure/batch/pool#az-batch-pool-resize) pour faire passer le nombre de nœuds de faible priorité dans le pool à *6* :

```azurecli-interactive
az batch pool resize --pool-id myrenderpool --target-dedicated-nodes 0 --target-low-priority-nodes 6
```

Le redimensionnement du pool prend quelques minutes. Pendant ce processus, définissez les tâches suivantes à exécuter dans le travail de rendu existant.

## <a name="render-a-multiframe-scene"></a>Créer le rendu d’une scène à plusieurs images

Comme dans l’exemple à image unique, utilisez la commande [az batch task create](/cli/azure/batch/task#az-batch-task-create) pour créer des tâches de rendu dans le travail nommé *myrenderjob*. Dans ce cas, spécifiez les paramètres de tâche dans un fichier JSON appelé *myrendertask_multi.json*. Vous pouvez télécharger le fichier à partir de [GitHub](https://raw.githubusercontent.com/Azure/azure-docs-cli-python-samples/master/batch/render-scene/json/myrendertask_multi.json). Chacune des six tâches spécifie une ligne de commande Arnold pour créer le rendu d’une image de la scène 3ds Max *MotionBlur-DragonFlying.max*.

Créez un fichier dans votre interpréteur de commandes actuel nommé *myrendertask_multi.json*, puis copiez et collez le contenu à partir du fichier téléchargé. Modifiez les éléments `blobSource` et `containerURL` du JSON fichier afin qu’ils contiennent le nom de votre compte de stockage et de votre jeton SAP. Veillez à modifier les paramètres pour chacune des six tâches. Enregistrez le fichier et exécutez la commande suivante pour mettre en file d’attente les tâches :

```azurecli-interactive
az batch task create --job-id myrenderjob --json-file myrendertask_multi.json
```

### <a name="view-task-output"></a>Afficher la sortie des tâches

L’exécution d’une tâche prend quelques minutes. Utilisez la commande [az batch task list](/cli/azure/batch/task#az-batch-task-list) pour afficher l’état des tâches. Par exemple :

```azurecli-interactive
az batch task list \
    --job-id myrenderjob \
    --output table
```

Utilisez la commande [az batch task show](/cli/azure/batch/task#az-batch-task-show) pour afficher des détails de chaque tâche. Par exemple :

```azurecli-interactive
az batch task show \
    --job-id myrenderjob \
    --task-id mymultitask1
```
 
Les tâches génèrent des fichiers de sortie nommés *dragon0002.jpg* - *dragon0007.jpg* sur les nœuds de calcul et les chargent dans le conteneur *job-myrenderjob* de votre compte de stockage. Pour afficher la sortie, téléchargez les fichiers dans un dossier de votre ordinateur local à l’aide de la commande [az storage blob download-batch](/cli/azure/storage/blob). Par exemple :

```azurecli-interactive
az storage blob download-batch \
    --source job-myrenderjob \
    --destination .
```

Ouvrez un des fichiers sur votre ordinateur. L’image rendue 6 ressemble à ce qui suit :

![Rendu de l’image de dragon 6](./media/tutorial-rendering-cli/dragon-frame6.png) 


## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, vous pouvez utiliser la commande [az group delete](/cli/azure/group#az-group-delete) pour supprimer le groupe de ressources, le compte Batch, les pools et toutes les ressources associées. Supprimez les ressources comme suit :

```azurecli-interactive 
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Télécharger des images dans le stockage Azure
> * Créer un pool Batch pour créer le rendu
> * Créer le rendu d’une scène à image unique avec Arnold
> * Mettre à l’échelle le pool et créer le rendu d’une scène comportant plusieurs images
> * Télécharger la sortie rendue

Pour en savoir plus sur le rendu à l’échelle du cloud, consultez la documentation relative au rendu Batch.

> [!div class="nextstepaction"]
> [Service de rendu Batch](batch-rendering-service.md)

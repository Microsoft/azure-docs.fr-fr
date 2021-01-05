---
title: Tutoriel - Exécuter une charge de travail parallèle à l’aide de l’API Python
description: Didacticiel - Traiter des fichiers multimédias en parallèle avec ffmpeg dans Azure Batch à l’aide de la bibliothèque cliente Python Batch
ms.devlang: python
ms.topic: tutorial
ms.date: 11/29/2018
ms.custom: mvc, devx-track-python
ms.openlocfilehash: 7013ef1e9b6dda4554b665fa8199edb86a8a9adb
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97106510"
---
# <a name="tutorial-run-a-parallel-workload-with-azure-batch-using-the-python-api"></a>Tutoriel : Exécuter une charge de travail parallèle avec Azure Batch à l’aide de l’API Python

Utilisez Azure Batch pour exécuter des programmes de traitement par lots de calcul haute performance (HPC) en parallèle, efficacement et à grande échelle dans Azure. Ce didacticiel vous permet de découvrir un exemple d’exécution Python d’une charge de travail parallèle utilisant Batch. Vous découvrez un workflow d’application Batch courant et comment interagir par programme avec les ressources de stockage et Batch. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * S’authentifier avec des comptes de stockage et Batch
> * Charger des fichiers d’entrée sur le stockage
> * Créer un pool de nœuds de calcul pour exécuter une application
> * Créer un travail et des tâches pour traiter les fichiers d’entrée
> * Surveiller l’exécution d’une tâche
> * Récupérer les fichiers de sortie

Dans ce didacticiel, vous convertissez des fichiers de multimédia MP4 en parallèle au format MP3 à l’aide de l’outil open-source [ffmpeg](https://ffmpeg.org/). 

[!INCLUDE [quickstarts-free-trial-note.md](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

* [Python 2.7 ou 3.3 ou version ultérieure](https://www.python.org/downloads/)

* Gestionnaire de package [pip](https://pip.pypa.io/en/stable/installing/)

* Un compte Azure Batch et un compte Stockage Azure lié. Pour créer ces comptes, consultez les démarrages rapides Azure Batch à l’aide du [portail Azure](quick-create-portal.md) ou de l’[interface de ligne de commande Azure](quick-create-cli.md).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

[!INCLUDE [batch-common-credentials](../../includes/batch-common-credentials.md)] 

## <a name="download-and-run-the-sample"></a>Télécharger et exécuter l’exemple

### <a name="download-the-sample"></a>Télécharger l’exemple

[Téléchargez ou clonez l’exemple d’application](https://github.com/Azure-Samples/batch-python-ffmpeg-tutorial) à partir de GitHub. Pour cloner le référentiel d’exemple d’application avec un client Git, utilisez la commande suivante :

```
git clone https://github.com/Azure-Samples/batch-python-ffmpeg-tutorial.git
```

Naviguez vers le répertoire qui contient le fichier `batch_python_tutorial_ffmpeg.py`.

Dans votre environnement Python, installez les packages requis à l’aide de `pip`.

```bash
pip install -r requirements.txt
```

Ouvrez le fichier `config.py`. Mettez à jour les chaînes d’identification de compte Batch et de stockage avec les valeurs propres à vos comptes. Par exemple :


```Python
_BATCH_ACCOUNT_NAME = 'mybatchaccount'
_BATCH_ACCOUNT_KEY = 'xxxxxxxxxxxxxxxxE+yXrRvJAqT9BlXwwo1CwF+SwAYOxxxxxxxxxxxxxxxx43pXi/gdiATkvbpLRl3x14pcEQ=='
_BATCH_ACCOUNT_URL = 'https://mybatchaccount.mybatchregion.batch.azure.com'
_STORAGE_ACCOUNT_NAME = 'mystorageaccount'
_STORAGE_ACCOUNT_KEY = 'xxxxxxxxxxxxxxxxy4/xxxxxxxxxxxxxxxxfwpbIC5aAWA8wDu+AFXZB827Mt9lybZB1nUcQbQiUrkPtilK5BQ=='
```

### <a name="run-the-app"></a>Exécuter l’application

Pour exécuter le script :

```
python batch_python_tutorial_ffmpeg.py
```

Lorsque vous exécutez l’exemple d’application, la sortie de la console est identique à ce qui suit. Pendant l’exécution, l’étape `Monitoring all tasks for 'Completed' state, timeout in 00:30:00...` fait l’objet d’une pause correspondant au démarrage des nœuds de calcul du pool. 
   
```
Sample start: 11/28/2018 3:20:21 PM

Container [input] created.
Container [output] created.
Uploading file LowPriVMs-1.mp4 to container [input]...
Uploading file LowPriVMs-2.mp4 to container [input]...
Uploading file LowPriVMs-3.mp4 to container [input]...
Uploading file LowPriVMs-4.mp4 to container [input]...
Uploading file LowPriVMs-5.mp4 to container [input]...
Creating pool [LinuxFFmpegPool]...
Creating job [LinuxFFmpegJob]...
Adding 5 tasks to job [LinuxFFmpegJob]...
Monitoring all tasks for 'Completed' state, timeout in 00:30:00...
Success! All tasks completed successfully within the specified timeout period.
Deleting container [input]....

Sample end: 11/28/2018 3:29:36 PM
Elapsed time: 00:09:14.3418742
```

Accédez à votre compte Batch dans le portail Azure pour surveiller le pool, les nœuds de calcul, les travaux et les tâches. Par exemple, pour voir une carte thermique des nœuds de calcul de votre pool, cliquez sur **Pools** > *LinuxFFmpegPool*.

Lorsque les tâches sont en cours d’exécution, la carte thermique est similaire à ce qui suit :

![Carte thermique des pools](./media/tutorial-parallel-python/pool.png)

Le temps d’exécution standard est **d’environ 5 minutes** lorsque l’application fonctionne dans sa configuration par défaut. La création d’un pool est l’opération la plus longue. 

[!INCLUDE [batch-common-tutorial-download](../../includes/batch-common-tutorial-download.md)]

## <a name="review-the-code"></a>Vérifier le code

Dans les sections suivantes, nous examinons l’exemple d’application en nous servant des opérations qu’elle effectue pour traiter une charge de travail dans le service Batch. Référez-vous au code Python lorsque vous lisez le reste de cet article, car certaines lignes de code de cet exemple ne sont pas expliquées ici.

### <a name="authenticate-blob-and-batch-clients"></a>Authentifier les clients Blob et Batch

Pour interagir avec un compte de stockage, l’application utilise le package [azure-storage-blob](https://pypi.python.org/pypi/azure-storage-blob) pour créer un objet [BlockBlobService](/python/api/azure-storage-blob/azure.storage.blob.blockblobservice.blockblobservice).

```python
blob_client = azureblob.BlockBlobService(
    account_name=_STORAGE_ACCOUNT_NAME,
    account_key=_STORAGE_ACCOUNT_KEY)
```

L’application crée un objet [BatchServiceClient](/python/api/azure.batch.batchserviceclient) pour créer et gérer des pools, des travaux et des tâches dans le service Batch. Le client Batch dans l’exemple utilise l’authentification de la clé partagée. Batch prend également en charge l’authentification via [Azure Active Directory](batch-aad-auth.md) pour authentifier des utilisateurs individuels ou une application sans assistance.

```python
credentials = batchauth.SharedKeyCredentials(_BATCH_ACCOUNT_NAME,
                                             _BATCH_ACCOUNT_KEY)

batch_client = batch.BatchServiceClient(
    credentials,
    base_url=_BATCH_ACCOUNT_URL)
```

### <a name="upload-input-files"></a>Charger des fichiers d’entrée

L’application utilise la référence `blob_client` pour créer un conteneur de stockage pour les fichiers MP4 d’entrée et un conteneur pour la sortie de la tâche. Ensuite, il appelle la fonction `upload_file_to_container` pour charger des fichiers MP4 du répertoire `InputFiles` local vers le conteneur. Les fichiers de stockage sont définis en tant qu’objets Batch [ResourceFile](/python/api/azure-batch/azure.batch.models.resourcefile) que Batch peut télécharger ultérieurement sur les nœuds de calcul.

```python
blob_client.create_container(input_container_name, fail_on_exist=False)
blob_client.create_container(output_container_name, fail_on_exist=False)
input_file_paths = []

for folder, subs, files in os.walk(os.path.join(sys.path[0], './InputFiles/')):
    for filename in files:
        if filename.endswith(".mp4"):
            input_file_paths.append(os.path.abspath(
                os.path.join(folder, filename)))

# Upload the input files. This is the collection of files that are to be processed by the tasks.
input_files = [
    upload_file_to_container(blob_client, input_container_name, file_path)
    for file_path in input_file_paths]
```

### <a name="create-a-pool-of-compute-nodes"></a>Créer un pool de nœuds de calcul

Ensuite, l’exemple crée un pool de nœuds de traitement dans le compte Batch, avec un appel à `create_pool`. Cette fonction définie utilise la classe [PoolAddParameter](/python/api/azure-batch/azure.batch.models.pooladdparameter) Batch pour définir le nombre de nœuds, la taille de machine virtuelle et une configuration de pool. Ici, un objet [VirtualMachineConfiguration](/python/api/azure-batch/azure.batch.models.virtualmachineconfiguration) spécifie une référence [ImageReference](/python/api/azure-batch/azure.batch.models.imagereference) sur une image Ubuntu Server 18.04 LTS publiée dans la Place de marché Azure. Azure Batch prend en charge une large plage de machine virtuelle dans la Place de marché Microsoft Azure, ainsi que des images de machines virtuelles personnalisées.

Le nombre de nœuds et la taille de machine virtuelle sont définis à l’aide de constantes définies. Azure Batch prend en charge les nœuds dédiés et les [nœuds de faible priorité](batch-low-pri-vms.md) que vous pouvez utiliser dans vos pools. Les nœuds dédiés sont réservés à votre pool. Les nœuds de faible priorité sont proposés à prix réduit à partir de la capacité de machine virtuelle excédentaire dans Azure. Les nœuds de faible priorité deviennent indisponibles si la capacité d’Azure est insuffisante. L’exemple par défaut crée un pool contenant seulement 5 nœuds de faible priorité taille *Standard_A1_v2*. 

En plus des propriétés du nœud physique, cette configuration de pool inclut un objet [StartTask](/python/api/azure-batch/azure.batch.models.starttask). La tâche StartTask s’exécute sur chacun des nœuds rejoignant le pool, ainsi qu’à chaque redémarrage d’un nœud. Dans cet exemple, StartTask exécute les commandes de l’interpréteur de commandes Bash pour installer le package ffmpeg et les dépendances sur les nœuds.

La méthode [pool.add](/python/api/azure-batch/azure.batch.operations.pooloperations) soumet le pool au service Batch.

```python
new_pool = batch.models.PoolAddParameter(
    id=pool_id,
    virtual_machine_configuration=batchmodels.VirtualMachineConfiguration(
        image_reference=batchmodels.ImageReference(
            publisher="Canonical",
            offer="UbuntuServer",
            sku="18.04-LTS",
            version="latest"
        ),
        node_agent_sku_id="batch.node.ubuntu 18.04"),
    vm_size=_POOL_VM_SIZE,
    target_dedicated_nodes=_DEDICATED_POOL_NODE_COUNT,
    target_low_priority_nodes=_LOW_PRIORITY_POOL_NODE_COUNT,
    start_task=batchmodels.StartTask(
        command_line="/bin/bash -c \"apt-get update && apt-get install -y ffmpeg\"",
        wait_for_success=True,
        user_identity=batchmodels.UserIdentity(
            auto_user=batchmodels.AutoUserSpecification(
                scope=batchmodels.AutoUserScope.pool,
                elevation_level=batchmodels.ElevationLevel.admin)),
    )
)
batch_service_client.pool.add(new_pool)
```

### <a name="create-a-job"></a>Créer un travail

Un programme de traitement par lots spécifie un pool pour exécuter des tâches et des paramètres facultatifs tels qu’une priorité et un calendrier pour le travail. L’exemple crée un travail avec un appel à `create_job`. Cette fonction définie utilise la classe [JobAddParameter](/python/api/azure-batch/azure.batch.models.jobaddparameter) pour créer un travail sur votre pool. La méthode [job.add](/python/api/azure-batch/azure.batch.operations.joboperations) soumet le pool au service Batch. Dans un premier temps, le travail n’a aucune tâche.

```python
job = batch.models.JobAddParameter(
    id=job_id,
    pool_info=batch.models.PoolInformation(pool_id=pool_id))

batch_service_client.job.add(job)
```

### <a name="create-tasks"></a>Créer des tâches

L’application crée des tâches dans le travail avec un appel à `add_tasks`. Cette fonction définie crée une liste d’objets de tâches à l’aide de la classe [TaskAddParameter](/python/api/azure-batch/azure.batch.models.taskaddparameter). Chaque tâche exécute ffmpeg pour traiter un objet `resource_files` d’entrée à l’aide d’un paramètre `command_line`. ffmpeg a été précédemment installé sur chaque nœud lors de la création du pool. Ici, la ligne de commande exécute ffmpeg pour convertir chaque fichier (vidéo) MP4 en fichier MP3 (audio).

L’exemple crée un objet [OutputFile](/python/api/azure-batch/azure.batch.models.outputfile) pour le fichier MP3 après l’exécution de la ligne de commande. Les fichiers de sortie de chaque tâche (un, dans ce cas) sont chargés sur un conteneur dans le compte de stockage lié, à l’aide de la propriété `output_files` de la tâche.

Ensuite, l’application ajoute des tâches au travail avec la méthode [task.add_collection](/python/api/azure-batch/azure.batch.operations.taskoperations), qui les met en file d’attente afin de les exécuter sur les nœuds de calcul. 

```python
tasks = list()

for idx, input_file in enumerate(input_files):
    input_file_path = input_file.file_path
    output_file_path = "".join((input_file_path).split('.')[:-1]) + '.mp3'
    command = "/bin/bash -c \"ffmpeg -i {} {} \"".format(
        input_file_path, output_file_path)
    tasks.append(batch.models.TaskAddParameter(
        id='Task{}'.format(idx),
        command_line=command,
        resource_files=[input_file],
        output_files=[batchmodels.OutputFile(
            file_pattern=output_file_path,
            destination=batchmodels.OutputFileDestination(
                container=batchmodels.OutputFileBlobContainerDestination(
                    container_url=output_container_sas_url)),
            upload_options=batchmodels.OutputFileUploadOptions(
                upload_condition=batchmodels.OutputFileUploadCondition.task_success))]
    )
    )
batch_service_client.task.add_collection(job_id, tasks)
```

### <a name="monitor-tasks"></a>Surveiller les tâches

Lorsque les tâches sont ajoutées à un travail, le service les met automatiquement en file d’attente et planifie leur exécution sur les nœuds de calcul dans le pool associé. Selon les paramètres que vous spécifiez, Batch gère l’ensemble des opérations de mise en file d’attente, de planification, de ré-exécution et d’administration des tâches. 

Il existe plusieurs approches pour l’exécution de la tâche d’analyse. La fonction `wait_for_tasks_to_complete` de cet exemple utilise l’objet [TaskState](/python/api/azure-batch/azure.batch.models.taskstate) pour surveiller les tâches relatives à un état spécifique. Dans ce cas, il s’agit de l’état terminé, dans un délai imparti.

```python
while datetime.datetime.now() < timeout_expiration:
    print('.', end='')
    sys.stdout.flush()
    tasks = batch_service_client.task.list(job_id)

    incomplete_tasks = [task for task in tasks if
                        task.state != batchmodels.TaskState.completed]
    if not incomplete_tasks:
        print()
        return True
    else:
        time.sleep(1)
...
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Après avoir exécuté les tâches, l’application supprime automatiquement le conteneur de stockage d’entrée créé et vous donne la possibilité de supprimer le travail et le pool Azure Batch. Les classes [JobOperations](/python/api/azure-batch/azure.batch.operations.joboperations) et [PoolOperations](/python/api/azure-batch/azure.batch.operations.pooloperations) de BatchServiceClient disposent toutes deux de méthodes de suppression, appelées si l’utilisateur confirme la suppression. Bien que vous ne soyez pas facturé pour les travaux et les tâches à proprement parler, les nœuds de calcul vous sont facturés. Par conséquent, nous vous conseillons d’affecter les pools uniquement en fonction des besoins. Lorsque vous supprimez le pool, toutes les sorties de tâche sur les nœuds sont supprimées. Toutefois, les fichiers d’entrée et de sortie restent dans le compte de stockage.

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources, le compte Batch et le compte de stockage. Pour ce faire, dans le portail Azure, sélectionnez le groupe de ressources pour le compte Batch, puis cliquez sur **Supprimer le groupe de ressources**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * S’authentifier avec des comptes de stockage et Batch
> * Charger des fichiers d’entrée sur le stockage
> * Créer un pool de nœuds de calcul pour exécuter une application
> * Créer un travail et des tâches pour traiter les fichiers d’entrée
> * Surveiller l’exécution d’une tâche
> * Récupérer les fichiers de sortie

Pour plus d’exemples d’utilisation de l’API Python pour planifier et traiter les charges de travail Batch, consultez les exemples sur GitHub.

> [!div class="nextstepaction"]
> [Exemples Python Batch](https://github.com/Azure/azure-batch-samples/tree/master/Python/Batch)


---
title: Exécuter des travaux de bout en bout à l’aide de modèles
description: En utilisant juste des commandes CLI, vous pouvez créer un pool, charger des données d’entrée, créer des travaux et des tâches associées, et télécharger les données de sortie produites.
ms.topic: how-to
ms.date: 10/08/2020
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 845a32c2feda5a5a3b8d44d237c62db94cae1779
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91848719"
---
# <a name="use-azure-batch-cli-templates-and-file-transfer"></a>Utiliser des modèles d’interface de ligne de commande Azure Batch et le transfert de fichiers

Vous pouvez utiliser une extension Batch de l’interface Azure CLI pour exécuter des travaux Batch sans écrire de code.

Créez et utilisez des modèles de fichier JSON avec Azure CLI pour créer des pools, travaux et tâches Batch. Utilisez des commandes d’extension LCI pour charger facilement les fichiers d’entrée des travaux dans le compte de stockage associé au compte Batch, et télécharger les fichiers de sortie de travaux.

> [!NOTE]
> Les fichiers JSON ne prennent pas en charge les mêmes fonctionnalités que [les modèles Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md). Ils sont conçus pour être mis en forme comme le corps de la demande REST brute. L’extension CLI ne modifie pas les commandes existantes, mais elle possède une option de modèle similaire qui ajoute une fonctionnalité partielle de modèle Azure Resource Manager. Voir [Extensions CLI d’Azure Batch pour Windows, Mac et Linux](https://github.com/Azure/azure-batch-cli-extensions).

## <a name="overview"></a>Vue d’ensemble

Une extension de l’interface Azure CLI permet aux utilisateurs qui ne sont pas des développeurs d’utiliser Batch de bout en bout. En utilisant juste des commandes CLI, vous pouvez créer un pool, charger des données d’entrée, créer des travaux et des tâches associées, et télécharger les données de sortie produites. Aucun code supplémentaire n’est nécessaire. Exécutez les commandes CLI directement, ou intégrez-les dans des scripts.

Les modèles Batch s’appuient sur la prise en charge actuelle de Batch dans [Azure CLI](batch-cli-get-started.md#json-files-for-resource-creation) pour les fichiers JSON, afin de spécifier des valeurs de propriété lors de la création notamment de pools, de travaux et de tâches. Les modèles Batch ajoutent les fonctionnalités suivantes :

-   Des paramètres peuvent être définis. Lorsque le modèle est utilisé, seules les valeurs de paramètre sont spécifiées pour créer l’élément et les autres valeurs de propriété d’élément sont spécifiées dans le corps du modèle. Un utilisateur qui comprend Batch et les applications exécutées par Batch peut créer des modèles, en spécifiant les valeurs de propriété des pools, des travaux et des tâches. Un utilisateur moins familiarisé avec Batch et/ou les applications doit uniquement spécifier les valeurs des paramètres définis.

-   Les fabriques de tâches de travail créent une ou plusieurs tâches associées à un travail sans avoir à créer des définitions de tâches, simplifiant considérablement les soumissions de travaux.


En général, les travaux utilisent des fichiers de données d’entrée et produisent des fichiers de données de sortie. Par défaut, un compte de stockage est associé à chaque compte Batch. Transférez des fichiers vers et depuis ce compte de stockage à l’aide de l’interface CLI, sans codage et sans information d’identification de stockage.

Par exemple, [ffmpeg](https://ffmpeg.org/) est une application courante qui traite les fichiers audio et vidéo. Voici les étapes à suivre pour appeler ffmpeg par l’intermédiaire de l’interface CLI d’Azure Batch et transcoder les fichiers vidéo sources dans différentes résolutions.

-   Créez un modèle de pool. L’utilisateur qui crée le modèle sait comment appeler l’application ffmpeg et connaît les prérequis. Il spécifie le système d’exploitation approprié, la taille de machine virtuelle, la façon dont ffmpeg est installé (à partir d’un package d’application ou à l’aide d’un gestionnaire de package, par exemple) et d’autres valeurs de propriété du pool. Les paramètres sont créés de façon à ce que, lorsque le modèle est utilisé, seul l’ID du pool et le nombre de machines virtuelles doivent être spécifiés.

-   Créez un modèle de travail. L’utilisateur qui crée le modèle sait comment appeler ffmpeg pour transcoder la vidéo source dans une résolution différente et spécifie la ligne de commande de la tâche. Il sait également qu’il y a un dossier contenant les fichiers vidéo sources, avec une tâche requise pour chaque fichier d’entrée.

-   Un utilisateur final avec un ensemble de fichiers vidéo à transcoder crée d’abord un pool à l’aide du modèle de pool, en spécifiant uniquement l’ID du pool et le nombre de machines virtuelles requises. Il peut ensuite charger les fichiers sources à transcoder. Un travail peut ensuite être soumis à l’aide du modèle de travail, en spécifiant uniquement l’ID du pool et l’emplacement des fichiers source chargés. Le travail Batch est créé, avec une tâche par fichier d’entrée généré. Enfin, les fichiers de sortie transcodés peuvent être téléchargés.

## <a name="installation"></a>Installation

Pour installer l’extension de l’interface CLI Azure Batch, [installez Azure CLI 2.0](/cli/azure/install-azure-cli) ou exécutez l’interface de ligne de commande Azure dans [Azure Cloud Shell](../cloud-shell/overview.md).

Installez la dernière version de l’extension Batch en exécutant la commande de l’interface de ligne de commande Azure suivante :

```azurecli
az extension add --name azure-batch-cli-extensions
```

Pour plus d’informations sur l’extension Batch de l’interface CLI et les options d’installation supplémentaires, consultez le [référentiel GitHub](https://github.com/Azure/azure-batch-cli-extensions).


Pour utiliser les fonctionnalités d’extension CLI, vous avez besoin d’un compte Azure Batch et, pour les commandes permettant de transférer des fichiers vers et depuis le stockage, d’un compte de stockage lié.

Pour vous connecter à un compte Batch avec Azure CLI, consultez [Gérer les ressources Batch avec Azure CLI](batch-cli-get-started.md).

## <a name="templates"></a>Modèles

Les modèles Azure Batch sont semblables aux modèles Azure Resource Manager en termes de syntaxe et de fonctionnalités. Il s’agit de fichiers JSON qui contiennent des valeurs et noms de propriétés, mais qui intègrent en plus les concepts principaux suivants :

-   **Paramètres**

    -   Ils autorisent la spécification des valeurs de propriété dans une section Corps, seules les valeurs de paramètre devant être fournies lorsque le modèle est utilisé. Par exemple, la définition complète d’un pool peut être placée dans le corps et un seul paramètre défini pour `poolId`. Par conséquent, seule la chaîne d’ID du pool doit être fournie pour créer un pool.

    -   Le corps du modèle peut être créé par une personne connaissant Batch et les applications à exécuter dans celui-ci. Seules les valeurs des paramètres définis par l’auteur doivent être fournis lorsque le modèle est utilisé. Un utilisateur sans connaissance poussée de Batch ou des applications peut par conséquent utiliser les modèles.

-   **Variables**

    -   Ils autorisent la spécification de valeurs de paramètre simples ou complexes dans un seul emplacement et leur utilisation dans un ou plusieurs emplacements dans le corps du modèle. Des variables peuvent simplifier et réduire la taille du modèle, mais aussi faciliter sa gestion en ayant un emplacement où changer les propriétés.

-   **Constructions de niveau supérieur**

    -   Certaines constructions de niveau supérieur sont disponibles dans le modèle, mais ne sont pas encore disponibles dans les API Batch. Par exemple, une fabrique de tâches peut être définie dans un modèle de travail qui crée plusieurs tâches pour le travail à l’aide d’une définition de tâche commune. Ces constructions évitent de devoir écrire du code pour créer dynamiquement plusieurs fichiers JSON, par exemple un fichier par tâche, mais aussi créer des fichiers de script pour installer des applications via un gestionnaire de package.

    -   À un moment donné, ces constructions peuvent être ajoutées au service Batch et être disponibles dans les interfaces utilisateur, les API Batch, etc.

### <a name="pool-templates"></a>Modèles de pool

Les modèles de pools prennent en charge les fonctionnalités du modèle standard des paramètres et des variables. Ils prennent également en charge la construction de niveau supérieur suivante :

-   **Références de package**

    -   Autorise éventuellement la copie du logiciel sur les nœuds du pool à l’aide de gestionnaires de packages. Le gestionnaire de package et l’ID du package sont spécifiés. En déclarant un ou plusieurs packages, vous évitez la création d’un script pour obtenir les packages nécessaires, l’installation de ce script et son exécution sur chaque nœud du pool.

Voici l’exemple d’un modèle qui crée un pool de machines virtuelles Linux avec ffmpeg installé. Pour l’utiliser, fournissez seulement une chaîne d’ID de pool et le nombre de machines virtuelles dans le pool :

```json
{
    "parameters": {
        "nodeCount": {
            "type": "int",
            "metadata": {
                "description": "The number of pool nodes"
            }
        },
        "poolId": {
            "type": "string",
            "metadata": {
                "description": "The pool ID "
            }
        }
    },
    "pool": {
        "type": "Microsoft.Batch/batchAccounts/pools",
        "apiVersion": "2016-12-01",
        "properties": {
            "id": "[parameters('poolId')]",
            "virtualMachineConfiguration": {
                "imageReference": {
                    "publisher": "Canonical",
                    "offer": "UbuntuServer",
                    "sku": "16.04-LTS",
                    "version": "latest"
                },
                "nodeAgentSKUId": "batch.node.ubuntu 16.04"
            },
            "vmSize": "STANDARD_D3_V2",
            "targetDedicatedNodes": "[parameters('nodeCount')]",
            "enableAutoScale": false,
            "taskSlotsPerNode": 1,
            "packageReferences": [
                {
                    "type": "aptPackage",
                    "id": "ffmpeg"
                }
            ]
        }
    }
}
```

Si le fichier de modèle est nommé _pool-ffmpeg.json_, appelez le modèle de la façon suivante :

```azurecli
az batch pool create --template pool-ffmpeg.json
```

L’interface CLI vous invite à fournir des valeurs pour les paramètres `poolId` et `nodeCount`. Vous pouvez également définir les paramètres dans un fichier JSON. Par exemple :

```json
{
  "poolId": {
    "value": "mypool"
  },
  "nodeCount": {
    "value": 2
  }
}
```

Si le fichier JSON de paramètre est nommé *pool-parameters.json*, appelez le modèle de la façon suivante :

```azurecli
az batch pool create --template pool-ffmpeg.json --parameters pool-parameters.json
```

### <a name="job-templates"></a>Modèles de travail

Les modèles de travail prennent en charge les fonctionnalités du modèle standard des paramètres et des variables. Ils prennent également en charge la construction de niveau supérieur suivante :

-   **Fabrique de tâches**

    -   Crée plusieurs tâches pour un travail à partir d’une définition de tâche. Trois types de fabrique de tâches sont pris en charge : balayage paramétrique, tâche par fichier et collection de tâches.

Voici un exemple de modèle qui crée un travail de transcodage de fichiers vidéo MP4 avec ffmpeg dans une des deux résolutions inférieures. Il crée une tâche par fichier vidéo source. Consultez [Groupes de fichiers et transfert de fichiers](#file-groups-and-file-transfer) pour en savoir plus sur les groupes de fichiers pour l’entrée et la sortie du travail.

```json
{
    "parameters": {
        "poolId": {
            "type": "string",
            "metadata": {
                "description": "The name of Azure Batch pool which runs the job"
            }
        },
        "jobId": {
            "type": "string",
            "metadata": {
                "description": "The name of Azure Batch job"
            }
        },
        "resolution": {
            "type": "string",
            "defaultValue": "428x240",
            "allowedValues": [
                "428x240",
                "854x480"
            ],
            "metadata": {
                "description": "Target video resolution"
            }
        }
    },
    "job": {
        "type": "Microsoft.Batch/batchAccounts/jobs",
        "apiVersion": "2016-12-01",
        "properties": {
            "id": "[parameters('jobId')]",
            "constraints": {
                "maxWallClockTime": "PT5H",
                "maxTaskRetryCount": 1
            },
            "poolInfo": {
                "poolId": "[parameters('poolId')]"
            },
            "taskFactory": {
                "type": "taskPerFile",
                "source": {
                    "fileGroup": "ffmpeg-input"
                },
                "repeatTask": {
                    "commandLine": "ffmpeg -i {fileName} -y -s [parameters('resolution')] -strict -2 {fileNameWithoutExtension}_[parameters('resolution')].mp4",
                    "resourceFiles": [
                        {
                            "blobSource": "{url}",
                            "filePath": "{fileName}"
                        }
                    ],
                    "outputFiles": [
                        {
                            "filePattern": "{fileNameWithoutExtension}_[parameters('resolution')].mp4",
                            "destination": {
                                "autoStorage": {
                                    "path": "{fileNameWithoutExtension}_[parameters('resolution')].mp4",
                                    "fileGroup": "ffmpeg-output"
                                }
                            },
                            "uploadOptions": {
                                "uploadCondition": "TaskSuccess"
                            }
                        }
                    ]
                }
            },
            "onAllTasksComplete": "terminatejob"
        }
    }
}
```

Si le fichier de modèle est nommé _job-ffmpeg.json_, appelez le modèle de la façon suivante :

```azurecli
az batch job create --template job-ffmpeg.json
```

Comme précédemment, l’interface CLI vous invite à fournir des valeurs pour les paramètres. Vous pouvez également définir les paramètres dans un fichier JSON.

### <a name="use-templates-in-batch-explorer"></a>Utiliser des modèles dans Batch Explorer

Vous pouvez charger un modèle d’interface CLI Batch pour l’application de bureau [Batch Explorer](https://github.com/Azure/BatchExplorer) (anciennement appelée BatchLabs) afin de créer un pool ou un travail Batch. Vous pouvez également sélectionner des modèles de pool et de travail prédéfinis dans la galerie de Batch Explorer.

Pour charger un modèle :

1. Dans Batch Explorer, sélectionnez **Galerie** > **Local templates** (Modèles locaux).

2. Sélectionnez, ou faites glisser et déplacez, un modèle de pool ou de travail local.

3. Sélectionnez **Utiliser ce modèle** et suivez les invites qui s’affichent à l’écran.

## <a name="file-groups-and-file-transfer"></a>Groupes de fichiers et transfert de fichiers

La plupart des travaux et des tâches requièrent les fichiers d’entrée et produisent des fichiers de sortie. Les fichiers d’entrée et de sortie sont généralement transférés, soit du client vers le nœud, soit du nœud vers le client. L’extension CLI d’Azure Batch met de côté le transfert de fichiers et utilise le compte de stockage que vous avez associé à chaque compte Batch.

Un groupe de fichiers correspond à un conteneur créé dans le compte de stockage Azure. Le groupe de fichiers peut contenir des sous-dossiers.

L’extension Batch CLI fournit des commandes pour charger les fichiers du client vers un groupe de fichiers spécifié, et pour télécharger les fichiers du groupe de fichiers spécifié vers le client.

```azurecli
az batch file upload --local-path c:\source_videos\*.mp4
    --file-group ffmpeg-input

az batch file download --file-group ffmpeg-output --local-path
    c:\output_lowres_videos
```

Les modèles de pool et de travail permettent de spécifier les fichiers stockés dans des groupes de fichiers pour la copie sur les nœuds de pool ou le renvoi des nœuds de pool vers un groupe de fichiers. Par exemple, dans le modèle de travail indiqué précédemment, le groupe de fichiers *ffmpeg-input* est spécifié pour la fabrique de tâches comme l’emplacement des fichiers vidéo sources copiés dans le nœud pour le transcodage. Le groupe de fichiers *ffmpeg-output* est l’emplacement où les fichiers de sortie transcodés sont copiés à partir du nœud exécutant chaque tâche.

## <a name="summary"></a>Résumé

La prise en charge du transfert de fichiers et des modèles a seulement été ajoutée à l’interface Azure CLI pour l’instant. L’objectif est d’étendre le public pouvant utiliser Batch aux utilisateurs qui n’ont pas besoin de développer du code à l’aide des API Batch, tels que les chercheurs et les utilisateurs. Sans effectuer de codage, les utilisateurs qui connaissent bien Azure, Batch et les applications peuvent créer des modèles pour la création de pools et de travaux. Avec les paramètres de modèle, les utilisateurs sans connaissance approfondie de Batch et des applications d’utiliser les modèles.

Testez l’extension Batch pour l’interface de ligne de commande Azure et faites-nous part de vos commentaires et de vos suggestions dans les commentaires de cet article ou via le [référentiel de la communauté Batch](https://github.com/Azure/Batch).

## <a name="next-steps"></a>Étapes suivantes

- Une documentation détaillée sur l’installation et l’utilisation, des exemples et du code source sont disponibles dans le [dépôt GitHub Azure](https://github.com/Azure/azure-batch-cli-extensions).

- En savoir plus sur l’utilisation de [Batch Explorer](https://github.com/Azure/BatchExplorer) pour créer et gérer les ressources Batch.

---
title: Exécuter les conteneurs init
description: Exécutez des conteneurs init dans Azure Container Instances pour effectuer des tâches d’installation dans un groupe de conteneurs avant l’exécution des conteneurs de l’application.
ms.topic: article
ms.date: 06/01/2020
ms.openlocfilehash: 5a729263ee632eb9227694ec8684eb6889c6324b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85954279"
---
# <a name="run-an-init-container-for-setup-tasks-in-a-container-group"></a>Exécuter un conteneur init pour les tâches d’installation dans un groupe de conteneurs

Azure Container Instances prend en charge les *conteneurs init* dans un groupe de conteneurs. Les conteneurs init sont exécutés jusqu’à la fin avant le démarrage du ou des conteneurs de l’application. Comme pour les [conteneurs init Kubernetes](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/), utilisez un ou plusieurs conteneurs init pour effectuer la logique d’initialisation de vos conteneurs d’application, tels que la définition de comptes, l’exécution de scripts d’installation ou la configuration de bases de données.

Cet article explique comment utiliser un modèle Azure Resource Manager pour configurer un groupe de conteneurs avec un conteneur init.

## <a name="things-to-know"></a>À savoir

* **Version de l’API** : vous avez au moins besoin de la version d’API Azure Container Instances API 2019-12-01 pour déployer des conteneurs init. Déployez à l’aide d’une propriété `initContainers` dans un [fichier YAML](container-instances-multi-container-yaml.md) ou d’un modèle [Resource Manager](container-instances-multi-container-group.md).
* **Ordre d’exécution** : les conteneurs-init sont exécutés dans l’ordre spécifié dans le modèle et avant les autres conteneurs. Par défaut, vous pouvez spécifier un maximum de 59 conteneurs init par groupe de conteneurs. Le groupe doit contenir au moins un conteneur non init.
* **Environnement hôte** : les conteneurs init s’exécutent sur le même matériel que le reste des conteneurs du groupe.
* **Ressources** : vous ne spécifiez pas de ressources pour les conteneurs init. Toutes les ressources, telles que les processeurs et la mémoire disponible pour le groupe de conteneurs, leur sont octroyées. Lorsqu’un conteneur init s’exécute, aucun autre conteneur du groupe n’est exécuté.
* **Propriétés prises en charge** : les conteneurs init peuvent utiliser des propriétés de groupe telles que des volumes, des secrets et des identités managées. Toutefois, ils ne peuvent utiliser ni des ports ou ni une adresse IP s’ils sont configurés pour le groupe de conteneurs. 
* **Stratégie de redémarrage** : chaque conteneur init doit se fermer correctement avant le démarrage du conteneur suivant dans le groupe. Si un conteneur init ne s’arrête pas correctement, son action de redémarrage dépend de la [stratégie de redémarrage](container-instances-restart-policy.md) configurée pour le groupe :

    |Stratégie dans le groupe  |Stratégie dans init  |
    |---------|---------|
    |Toujours     |OnFailure         |
    |OnFailure     |OnFailure         |
    |Jamais     |Jamais         |
* **Frais** : le groupe de conteneurs génère des frais dès le premier déploiement d’un conteneur init.

## <a name="resource-manager-template-example"></a>Exemple de modèle Resource Manager

Commencez par copier le JSON suivant dans un nouveau fichier nommé `azuredeploy.json`. Le modèle configure un groupe de conteneurs avec un conteneur init et deux conteneurs d’application :

* Le conteneur *init1* exécute l’image [busybox](https://hub.docker.com/_/busybox) à partir de Docker Hub. Il se met en veille pendant 60 secondes, puis écrit une chaîne de ligne de commande dans un fichier dans un [volume emptyDir](container-instances-volume-emptydir.md).
* Les deux conteneurs d’application exécutent l’image de conteneur Microsoft `aci-wordcount` :
    * Le conteneur *hamlet* exécute l’application wordcount dans sa configuration par défaut, en comptant les fréquences des mots dans la pièce *Hamlet* de Shakespeare.
    * Le conteneur d’application *juliet* lit la chaîne de ligne de commande à partir du volume emptDir pour exécuter l’application wordcount dans *Roméo et Juliette* de Shakespeare à la place.

Pour plus d’informations et d’exemples avec l’image `aci-wordcount`, consultez [Définir des variables d’environnement dans des instances de conteneur](container-instances-environment-variables.md).

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "containerGroupName": {
        "type": "string",
        "defaultValue": "myContainerGroup",
        "metadata": {
          "description": "Container Group name."
        }
      }
    },
    "resources": [
        {
            "name": "[parameters('containerGroupName')]",
            "type": "Microsoft.ContainerInstance/containerGroups",
            "apiVersion": "2019-12-01",
            "location": "[resourceGroup().location]",
            "properties": {
                "sku": "Standard",
                "initContainers": [
                {
                    "name": "init1",
                    "properties": {
                        "image": "busybox",
                        "environmentVariables": [],
                        "volumeMounts": [
                            {
                                "name": "emptydir1",
                                "mountPath": "/mnt/emptydir"
                            }
                        ],
                         "command": [
                            "/bin/sh",
                            "-c",
                            "sleep 60; echo python wordcount.py http://shakespeare.mit.edu/romeo_juliet/full.html > /mnt/emptydir/command_line.txt"
                        ]
                    }
                }
            ], 
                "containers": [
                    {
                        "name": "hamlet",
                        "properties": {
                            "image": "mcr.microsoft.com/azuredocs/aci-wordcount",
                            "volumeMounts": [
                            {
                                "name": "emptydir1",
                                "mountPath": "/mnt/emptydir"
                            }
                        ],
                            "environmentVariables": [
                             {
                                "name": "NumWords",
                                "value": "3"
                             },
                             {  "name": "MinLength",
                                "value": "5"
                             }
                            ],
                            "resources": {
                                "requests": {
                                    "memoryInGB": 1.0,
                                    "cpu": 1
                                }
                            }
                        }
                    },
                    {
                        "name": "juliet",
                        "properties": {
                            "image": "mcr.microsoft.com/azuredocs/aci-wordcount",
                            "volumeMounts": [
                            {
                                "name": "emptydir1",
                                "mountPath": "/mnt/emptydir"
                            }
                            ],
                            "command": [
                                "/bin/sh",
                                "-c",
                                "$(cat /mnt/emptydir/command_line.txt)"
                            ],
                            "environmentVariables": [
                             {
                                "name": "NumWords",
                                "value": "3"
                             },
                             {  "name": "MinLength",
                                "value": "5"
                             }
                            ],
                            "resources": {
                                "requests": {
                                    "memoryInGB": 1.0,
                                    "cpu": 1
                                }
                            }
                        }
                    }
                ],
                "restartPolicy": "OnFailure",
                "osType": "Linux",
                "volumes": [
                    {
                        "name": "emptydir1",
                        "emptyDir": {}
                    }
                ]           
            },
            "tags": {}
        }
    ]
}
```

## <a name="deploy-the-template"></a>Déployer le modèle

Créez un groupe de ressources avec la commande [az group create][az-group-create].

```azurecli
az group create --name myResourceGroup --location eastus
```

Déployez ensuite le modèle avec la commande [az deployment group create][az-deployment-group-create].

```azurecli
az deployment group create \
  --resource-group myResourceGroup \
  --template-file azuredeploy.json
```

Dans un groupe avec un conteneur init, le temps de déploiement est augmenté en raison du temps nécessaire pour que le ou les conteneurs init se terminent.


## <a name="view-container-logs"></a>Afficher les journaux d’activité du conteneur

Pour vérifier que le conteneur init s’est exécuté correctement, affichez la sortie du journal des conteneurs d’application à l’aide de la commande [journaux de conteneurs az][az-container-logs]. L’argument `--container-name` spécifie le conteneur à partir duquel extraire les journaux d’activité. Dans cet exemple, extrayez les journaux des conteneurs *hamlet* et *juliet*, qui affichent une sortie de commande différente :

```azurecli
az container logs \
  --resource-group myResourceGroup \
  --name myContainerGroup \
  --container-name hamlet
```

Sortie :

```console
[('HAMLET', 386), ('HORATIO', 127), ('CLAUDIUS', 120)]
```

```azurecli
az container logs \
  --resource-group myResourceGroup \
  --name myContainerGroup \
  --container-name juliet
```

Sortie :

```console
[('ROMEO', 177), ('JULIET', 134), ('CAPULET', 119)]
```

## <a name="next-steps"></a>Étapes suivantes

Les conteneurs init vous aident à effectuer des tâches d’installation et d’initialisation pour vos conteneurs d’application. Pour plus d’informations sur l’exécution des conteneurs basés sur des tâches, consultez l’article [Exécuter des tâches conteneurisées avec des stratégies de redémarrage](container-instances-restart-policy.md).

Azure Container Instances fournit d’autres options pour modifier le comportement des conteneurs d’application. Voici quelques exemples :

* [Définir des variables d’environnement dans des instances de conteneur](container-instances-environment-variables.md)
* [Définir la ligne de commande dans une instance de conteneur pour remplacer l’opération de ligne de commande par défaut](container-instances-start-command.md)


[az-group-create]: /cli/azure/group#az-group-create
[az-deployment-group-create]: /cli/azure/deployment/group#az-deployment-group-create
[az-container-logs]: /cli/azure/container#az-container-logs

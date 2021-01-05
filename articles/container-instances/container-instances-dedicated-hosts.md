---
title: Déployer sur un hôte dédié
description: Utiliser un hôte dédié à des fins d'isolation au niveau de l’hôte pour vos charges de travail Azure Container Instances
ms.topic: article
ms.date: 01/17/2020
author: macolso
ms.author: macolso
ms.openlocfilehash: 967d2da61ffdfa9d1723bcab589deb2277d4041e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91825604"
---
# <a name="deploy-on-dedicated-hosts"></a>Déployer sur des hôtes dédiés

« Dédié » est une SKU Azure Container Instances (ACI) qui fournit un environnement Compute isolé et dédié pour l’exécution sécurisée des conteneurs. L’utilisation de la référence SKU dédiée donne à chaque groupe de conteneurs un serveur physique dédié dans un centre de données Azure, ce qui garantit l’isolation complète de la charge de travail afin de répondre aux exigences de conformité et de sécurité de votre organisation. 

La référence SKU dédiée est appropriée pour les charges de travail de conteneur qui requièrent l’isolation de la charge de travail du point de vue du serveur physique.

## <a name="prerequisites"></a>Prérequis

* La limite par défaut d’un abonnement pour utiliser la référence SKU dédiée est 0. Si vous souhaitez utiliser cette référence SKU pour vos déploiements de conteneur de production, créez une [demande de support Azure][azure-support] afin d'augmenter la limite.

## <a name="use-the-dedicated-sku"></a>Utiliser la référence SKU dédiée

> [!IMPORTANT]
> L’utilisation de la référence SKU dédiée est uniquement disponible dans la dernière version de l’API (2019-12-01) qui est en cours de déploiement. Spécifiez cette version d’API dans votre modèle de déploiement.
>

À partir de la version de l’API 2019-12-01, il existe une propriété `sku` dans la section Propriétés du groupe de conteneurs d’un modèle de déploiement qui est nécessaire pour un déploiement ACI. Actuellement, vous pouvez utiliser cette propriété dans le cadre d’un modèle de déploiement Azure Resource Manager pour ACI. Apprenez-en davantage sur le déploiement de ressources ACI à l’aide d’un modèle dans le [Tutoriel : Déployer un groupe de plusieurs conteneurs avec un modèle Resource Manager](./container-instances-multi-container-group.md). 

La propriété `sku` peut présenter l'une des valeurs suivantes :
* `Standard` : le choix de déploiement ACI standard, qui garantit toujours la sécurité au niveau de l’hyperviseur 
* `Dedicated` : utilisé pour l’isolation au niveau de la charge de travail avec des hôtes physiques dédiés pour le groupe de conteneurs

## <a name="modify-your-json-deployment-template"></a>Modifier votre modèle de déploiement JSON

Dans votre modèle de déploiement, modifiez ou ajoutez les propriétés suivantes :
* Sous `resources`, définissez `apiVersion` sur `2019-12-01`.
* Sous les propriétés du groupe de conteneurs, ajoutez une propriété `sku` avec la valeur `Dedicated`.

Voici un exemple d’extrait de code pour la section Ressources d’un modèle de déploiement pour groupe de conteneurs qui utilise la référence SKU dédiée :

```json
[...]
"resources": [
    {
        "name": "[parameters('containerGroupName')]",
        "type": "Microsoft.ContainerInstance/containerGroups",
        "apiVersion": "2019-12-01",
        "location": "[resourceGroup().location]",    
        "properties": {
            "sku": "Dedicated",
            "containers": {
                [...]
            }
        }
    }
]
```

Voici un modèle complet qui déploie un exemple de groupe de conteneurs exécutant une instance de conteneur unique :

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
                "sku": "Dedicated",
                "containers": [
                    {
                        "name": "container1",
                        "properties": {
                            "image": "nginx",
                            "command": [
                                "/bin/sh",
                                "-c",
                                "while true; do echo `date`; sleep 1000000; done"
                            ],
                            "ports": [
                                {
                                    "protocol": "TCP",
                                    "port": 80
                                }
                            ],
                            "environmentVariables": [],
                            "resources": {
                                "requests": {
                                    "memoryInGB": 1.0,
                                    "cpu": 1
                                }
                            }
                        }
                    }
                ],
                "restartPolicy": "Always",
                "ipAddress": {
                    "ports": [
                        {
                            "protocol": "TCP",
                            "port": 80
                        }
                    ],
                    "type": "Public"
                },
                "osType": "Linux"
            },
            "tags": {}
        }
    ]
}
```

## <a name="deploy-your-container-group"></a>Déployer votre groupe de conteneurs

Si vous avez créé et modifié le fichier de modèle de déploiement sur votre Bureau, vous pouvez le télécharger dans votre répertoire Cloud Shell en y faisant glisser le fichier. 

Créez un groupe de ressources avec la commande [az group create][az-group-create].

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Déployez ensuite le modèle avec la commande [az deployment group create][az-deployment-group-create].

```azurecli-interactive
az deployment group create --resource-group myResourceGroup --template-file deployment-template.json
```

Après quelques secondes, vous devriez recevoir une réponse initiale d’Azure. Un déploiement réussi intervient sur un hôte dédié.

<!-- LINKS - Internal -->
[az-group-create]: /cli/azure/group#az-group-create
[az-deployment-group-create]: /cli/azure/deployment/group#az-deployment-group-create

<!-- LINKS - External -->
[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest

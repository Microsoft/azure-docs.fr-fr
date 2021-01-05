---
title: Déployer des modèles sur Azure Container Instances
titleSuffix: Azure Machine Learning
description: Découvrez comment déployer vos modèles Azure Machine Learning en tant que service web à l’aide d’Azure Container Instances.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, deploy
ms.author: jordane
author: jpe316
ms.reviewer: larryfr
ms.date: 06/12/2020
ms.openlocfilehash: a0e9401842284cad29b297be5ce572fa53cfe774
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96188086"
---
# <a name="deploy-a-model-to-azure-container-instances"></a>Déployer un modèle sur Azure Container Instances

Découvrez comment utiliser Azure Machine Learning pour déployer un modèle en tant que service web sur Azure Container Instances (ACI). Utilisez Azure Container Instances si l’une des conditions suivantes est vraie :

- Vous avez besoin de déployer et de valider rapidement votre modèle. Vous n’avez pas besoin de créer des conteneurs ACI à l’avance. Ils sont créés lors du processus de déploiement.
- Vous testez un modèle en cours de développement. 

Pour plus d’informations sur les quotas et la disponibilité d’ACI en fonction des régions, consultez l’article [Quotas et disponibilité dans les régions pour Azure Container Instances](../container-instances/container-instances-quotas.md).

> [!IMPORTANT]
> Il est fortement recommandé d’effectuer un débogage local avant de procéder au déploiement sur le service web. Pour plus d’informations, consultez [Débogage local](./how-to-troubleshoot-deployment-local.md).
>
> Vous pouvez également vous reporter à Azure Machine Learning – [Déploiement sur un notebook local](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/deployment/deploy-to-local).

## <a name="prerequisites"></a>Prérequis

- Un espace de travail Azure Machine Learning. Pour plus d’informations, voir la page [Créer un espace de travail Azure Machine Learning](how-to-manage-workspace.md).

- Un modèle Machine Learning inscrit dans votre espace de travail. Si vous n’avez pas de modèle inscrit, consultez la section [Comment et où déployer des modèles](how-to-deploy-and-where.md).

- L’[extension Azure CLI pour Machine Learning service](reference-azure-machine-learning-cli.md), le [SDK Azure Machine Learning pour Python](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py) ou l’[extension Azure Machine Learning pour Visual Studio Code](tutorial-setup-vscode-extension.md).

- Les extraits de code __Python__ de cet article partent du principe que les variables suivantes sont définies :

    * `ws` – Sur votre espace de travail.
    * `model` – Sur votre modèle inscrit.
    * `inference_config` – Sur la configuration d’inférence pour le modèle.

    Pour plus d’informations sur le paramétrage de ces variables, consultez la section [Comment et où déployer des modèles ?](how-to-deploy-and-where.md)

- Les extraits de code __CLI__ de cet article partent du principe que vous avez créé un document `inferenceconfig.json`. Pour plus d’informations sur la création de ce document, consultez la section [Comment et où déployer des modèles ?](how-to-deploy-and-where.md)

## <a name="limitations"></a>Limites

* Lorsque vous utilisez Azure Container Instances dans un réseau virtuel, ce dernier doit se trouver dans le même groupe de ressources que votre espace de travail Azure Machine Learning.
* Lorsque vous utilisez Azure Container Instances à l’intérieur du réseau virtuel, l’instance Azure Container Registry (ACR) de votre espace de travail ne doit pas se trouver dans le réseau virtuel.

Pour plus d’informations, consultez [Sécurisation de l’inférence avec des réseaux virtuels](how-to-secure-inferencing-vnet.md#enable-azure-container-instances-aci).

## <a name="deploy-to-aci"></a>Déployer sur ACI

Pour déployer un modèle sur Azure Container instances, créez une __configuration de déploiement__ décrivant les ressources de calcul nécessaires. Par exemple, le nombre de cœurs et la mémoire. Vous avez également besoin d’une __configuration d’inférence__ décrivant l’environnement nécessaire pour héberger le modèle et le service Web. Pour plus d’informations sur la création de la configuration d’inférence, consultez la section [Comment et où déployer des modèles ?](how-to-deploy-and-where.md)

> [!NOTE]
> * ACI convient uniquement pour les petits modèles de taille inférieure à 1 Go. 
> * Nous vous recommandons d’utiliser AKS mononœud pour développer/tester des modèles plus volumineux.
> * Le nombre de modèles à déployer est limité à 1 000 modèles par déploiement (par conteneur). 

### <a name="using-the-sdk"></a>Utilisation du kit de développement logiciel

```python
from azureml.core.webservice import AciWebservice, Webservice
from azureml.core.model import Model

deployment_config = AciWebservice.deploy_configuration(cpu_cores = 1, memory_gb = 1)
service = Model.deploy(ws, "aciservice", [model], inference_config, deployment_config)
service.wait_for_deployment(show_output = True)
print(service.state)
```

Pour plus d’informations sur les classes, les méthodes et les paramètres utilisés dans cet exemple, consultez les documents de référence suivants :

* [AciWebservice.deploy_configuration](/python/api/azureml-core/azureml.core.webservice.aciwebservice?preserve-view=true&view=azure-ml-py#&preserve-view=truedeploy-configuration-cpu-cores-none--memory-gb-none--tags-none--properties-none--description-none--location-none--auth-enabled-none--ssl-enabled-none--enable-app-insights-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none--dns-name-label-none--primary-key-none--secondary-key-none--collect-model-data-none--cmk-vault-base-url-none--cmk-key-name-none--cmk-key-version-none-)
* [Model.deploy](/python/api/azureml-core/azureml.core.model.model?preserve-view=true&view=azure-ml-py#&preserve-view=truedeploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-)
* [Webservice.wait_for_deployment](/python/api/azureml-core/azureml.core.webservice%28class%29?preserve-view=true&view=azure-ml-py#&preserve-view=truewait-for-deployment-show-output-false-)

### <a name="using-the-cli"></a>Utilisation de l’interface CLI

Pour déployer à l’aide de la CLI, utilisez la commande suivante. Remplacez `mymodel:1` par le nom et la version du modèle inscrit. Remplacez `myservice` par le nom à attribuer à ce service :

```azurecli-interactive
az ml model deploy -m mymodel:1 -n myservice -ic inferenceconfig.json -dc deploymentconfig.json
```

[!INCLUDE [deploymentconfig](../../includes/machine-learning-service-aci-deploy-config.md)]

Pour plus d’informations, consultez les informations de référence sur [az ml model deploy](/cli/azure/ext/azure-cli-ml/ml/model?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-model-deploy). 

## <a name="using-vs-code"></a>Avec VS Code

Consultez la section[Déployer vos modèles avec VS Code](tutorial-train-deploy-image-classification-model-vscode.md#deploy-the-model).

> [!IMPORTANT]
> Vous n’avez pas besoin de créer un conteneur ACI à tester à l’avance. Les conteneurs ACI sont créés en fonction des besoins.

> [!IMPORTANT]
> Nous ajoutons l’ID de l’espace de travail haché à toutes les ressources ACI sous-jacentes qui sont créées ; tous les noms ACI du même espace de travail auront le même suffixe. Le nom du service Azure Machine Learning est toujours le même « service_name » fourni par le client, et l’ensemble des API du kit SDK Azure Machine Learning orienté utilisateur ne nécessite aucune modification. Nous ne garantissons pas les noms des ressources sous-jacentes en cours de création.

## <a name="next-steps"></a>Étapes suivantes

* [Guide pratique pour déployer un modèle à l’aide d’une image Docker personnalisée](how-to-deploy-custom-docker-image.md)
* [Résolution des problèmes liés au déploiement](how-to-troubleshoot-deployment.md)
* [Mettre à jour le service web](how-to-deploy-update-web-service.md)
* [Utiliser TLS pour sécuriser un service web par le biais d’Azure Machine Learning](how-to-secure-web-service.md)
* [Utiliser un modèle ML déployé en tant que service web](how-to-consume-web-service.md)
* [Superviser vos modèles Azure Machine Learning avec Application Insights](how-to-enable-app-insights.md)
* [Collecter des données pour des modèles en production](how-to-enable-data-collection.md)
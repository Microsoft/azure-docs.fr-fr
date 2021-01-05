---
title: " Mettre à jour les services web"
titleSuffix: Azure Machine Learning
description: Découvrez comment actualiser un service web déjà déployé dans Azure Machine Learning. Vous pouvez mettre à jour des paramètres tels que le modèle, l’environnement et le script d’entrée.
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: larryfr
ms.author: gopalv
author: gvashishtha
ms.date: 07/31/2020
ms.custom: deploy
ms.openlocfilehash: f955ee3f8bc150982c19107271fe7e7bdb1368cb
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97824567"
---
# <a name="update-a-deployed-web-service"></a>Mettre à jour un service web déployé

Dans cet article, vous allez apprendre à mettre à jour un service web déployé avec Azure Machine Learning.

## <a name="prerequisites"></a>Prérequis

Ce tutoriel part du principe que vous avez déjà déployé un service Web avec Azure Machine Learning. Si vous avez besoin d’apprendre à déployer un service Web, [procédez comme suit](how-to-deploy-and-where.md).

## <a name="update-web-service"></a>Mettre à jour un service web

Pour mettre à jour un service web, utilisez la méthode `update`. Vous pouvez mettre à jour le service web pour utiliser un nouveau modèle, un nouveau script d’entrée ou de nouvelles dépendances qui peuvent être spécifiées dans une configuration d’inférence. Pour plus d’informations, consultez la documentation sur [Webservice.update](/python/api/azureml-core/azureml.core.webservice.webservice.webservice?preserve-view=true&view=azure-ml-py#&preserve-view=trueupdate--args-).

Consultez [Méthode de mise à jour du service AKS.](/python/api/azureml-core/azureml.core.webservice.akswebservice?preserve-view=true&view=azure-ml-py#&preserve-view=trueupdate-image-none--autoscale-enabled-none--autoscale-min-replicas-none--autoscale-max-replicas-none--autoscale-refresh-seconds-none--autoscale-target-utilization-none--collect-model-data-none--auth-enabled-none--cpu-cores-none--memory-gb-none--enable-app-insights-none--scoring-timeout-ms-none--replica-max-concurrent-requests-none--max-request-wait-time-none--num-replicas-none--tags-none--properties-none--description-none--models-none--inference-config-none--gpu-cores-none--period-seconds-none--initial-delay-seconds-none--timeout-seconds-none--success-threshold-none--failure-threshold-none--namespace-none--token-auth-enabled-none-)

Consultez [Méthode de mise à jour du service ACI.](/python/api/azureml-core/azureml.core.webservice.aci.aciwebservice?preserve-view=true&view=azure-ml-py#&preserve-view=trueupdate-image-none--tags-none--properties-none--description-none--auth-enabled-none--ssl-enabled-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none--enable-app-insights-none--models-none--inference-config-none-)

> [!IMPORTANT]
> Lorsque vous créez une nouvelle version d’un modèle, vous devez mettre à jour manuellement chacun des services pour lesquels vous souhaitez l’utiliser.
>
> Vous ne pouvez pas utiliser le kit de développement logiciel (SDK) pour mettre à jour un service web publié à partir du concepteur Azure Machine Learning.

**Avec le kit SDK**

Le code suivant montre comment utiliser le SDK pour mettre à jour le modèle, l’environnement et le script d’entrée d’un service web :

```python
from azureml.core import Environment
from azureml.core.webservice import Webservice
from azureml.core.model import Model, InferenceConfig

# Register new model.
new_model = Model.register(model_path="outputs/sklearn_mnist_model.pkl",
                           model_name="sklearn_mnist",
                           tags={"key": "0.1"},
                           description="test",
                           workspace=ws)

# Use version 3 of the environment.
deploy_env = Environment.get(workspace=ws,name="myenv",version="3")
inference_config = InferenceConfig(entry_script="score.py",
                                   environment=deploy_env)

service_name = 'myservice'
# Retrieve existing service.
service = Webservice(name=service_name, workspace=ws)



# Update to new model(s).
service.update(models=[new_model], inference_config=inference_config)
service.wait_for_deployment(show_output=True)
print(service.state)
print(service.get_logs())
```

**Avec l’interface CLI**

Vous pouvez également mettre à jour un service web en utilisant ML CLI. L’exemple suivant explique comment enregistrer un nouveau modèle, puis mettre à jour le service web pour utiliser ce nouveau modèle :

```azurecli
az ml model register -n sklearn_mnist  --asset-path outputs/sklearn_mnist_model.pkl  --experiment-name myexperiment --output-metadata-file modelinfo.json
az ml service update -n myservice --model-metadata-file modelinfo.json
```

> [!TIP]
> Dans cet exemple, un document JSON est utilisé pour transférer les informations de modèle à partir de la commande d’inscription vers la commande de mise à jour.
>
> Pour mettre à jour le service afin d’utiliser un nouveau script ou environnement d’entrée, créez un [fichier de configuration d’inférence](./reference-azure-machine-learning-cli.md#inference-configuration-schema) et spécifiez-le avec le paramètre `ic`.

Pour plus d’informations, consultez la documentation [az ml service update](/cli/azure/ext/azure-cli-ml/ml/service?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-service-update).

## <a name="next-steps"></a>Étapes suivantes

* [Résoudre des problèmes d’échec de déploiement](how-to-troubleshoot-deployment.md)
* [Déployer dans Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md)
* [Créer des applications clientes pour utiliser des services web](how-to-consume-web-service.md)
* [Guide pratique pour déployer un modèle à l’aide d’une image Docker personnalisée](how-to-deploy-custom-docker-image.md)
* [Utiliser TLS pour sécuriser un service web par le biais d’Azure Machine Learning](how-to-secure-web-service.md)
* [Superviser vos modèles Azure Machine Learning avec Application Insights](how-to-enable-app-insights.md)
* [Collecter des données pour des modèles en production](how-to-enable-data-collection.md)
* [Créer des alertes d’événement et des déclencheurs pour les déploiements de modèle](how-to-use-event-grid.md)

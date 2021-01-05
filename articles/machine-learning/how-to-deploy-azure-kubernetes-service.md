---
title: Déployer des modèles Machine Learning dans le service Kubernetes
titleSuffix: Azure Machine Learning
description: Découvrez comment déployer vos modèles Azure Machine Learning en tant que service web à l’aide d’Azure Kubernetes Service.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, contperf-fy21q1, deploy, devx-track-azurecli
ms.author: jordane
author: jpe316
ms.reviewer: larryfr
ms.date: 09/01/2020
ms.openlocfilehash: d7540066ccc0d3a62dbd4012eee100d8e8aea98f
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97505084"
---
# <a name="deploy-a-model-to-an-azure-kubernetes-service-cluster"></a>Déployer un modèle sur un cluster Azure Kubernetes Service

Découvrez comment utiliser Azure Machine Learning pour déployer un modèle en tant que service web sur Azure Kubernetes service (AKS). Azure Kubernetes Service est idéal pour les déploiements de production à grande échelle. Utilisez Azure Kubernetes Service si vous avez besoin d’une ou de plusieurs des fonctionnalités suivantes :

- __Temps de réponse rapide__
- __Mise à l’échelle automatique__ du service déployé
- __Logging__
- __Collection de données de modèle__
- __Authentification__
- __Arrêt TLS__
- Options d’__accélération matérielle__ telles que le GPU et les FPGA (Field-Programmable Gate Array)

Lors d’un déploiement sur Azure Kubernetes Service, vous déployez sur un cluster AKS qui est __connecté à votre espace de travail__. Pour en savoir plus sur la connexion d’un cluster AKS à votre espace de travail, consultez [Créer et attacher un cluster Azure Kubernetes Service](how-to-create-attach-kubernetes.md).

> [!IMPORTANT]
> Nous vous recommandons de procéder à un débogage local avant le déploiement sur le service web. Pour plus d’informations, consultez [Déboguer localement](./how-to-troubleshoot-deployment-local.md).
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

- Un cluster Azure Kubernetes Service connecté à votre espace de travail. Pour plus d’informations, consultez [Créer et attacher un cluster Azure Kubernetes Service](how-to-create-attach-kubernetes.md).

    - Si vous souhaitez déployer des modèles sur des nœuds GPU ou FPGA (ou sur une référence SKU spécifique), vous devez créer un cluster de la référence SKU en question. Il n’est pas possible de créer un pool de nœuds secondaire dans un cluster existant et de déployer des modèles dans le pool de nœuds secondaire.

## <a name="understand-the-deployment-processes"></a>Comprendre le processus de déploiement

Le mot « déploiement » est utilisé à la fois dans Kubernetes et Azure Machine Learning. « Déploiement » a des significations différentes dans ces deux contextes. Dans Kubernetes, une `Deployment` est une entité concrète, spécifiée avec un fichier YAML déclaratif. Une `Deployment` Kubernetes a un cycle de vie défini et des relations concrètes avec d’autres entités Kubernetes, telles que `Pods` et `ReplicaSets`. Vous pouvez en savoir plus sur Kubernetes à partir des documents et des vidéos sur [Qu’est-ce que Kubernetes ?](https://aka.ms/k8slearning).

Dans Azure Machine Learning, le « déploiement » est utilisé dans le sens le plus général pour mettre à disposition et nettoyer vos ressources de projet. Les étapes qu’Azure Machine Learning prend en compte dans le cadre du déploiement sont les suivantes :

1. Compression des fichiers dans votre dossier de projet, en ignorant ceux spécifiés dans .amlignore ou .gitignore
1. Mise à l’échelle de votre cluster de calcul (en relation avec Kubernetes)
1. Création ou téléchargement du dockerfile sur le nœud de calcul (en relation avec Kubernetes)
    1. Le système calcule un code de hachage pour : 
        - l’image de base ; 
        - les étapes Docker personnalisées (voir [Déployer un modèle à l’aide d’une image de base Docker personnalisée](./how-to-deploy-custom-docker-image.md)) ;
        - la définition Conda YAML (voir [Créer et utiliser des environnements logiciels dans Azure Machine Learning](./how-to-use-environments.md)).
    1. Le système utilise ce code de hachage comme clé pour rechercher le Dockerfile dans l’espace de travail Azure Container Registry (ACR).
    1. Si le Dockerfile est introuvable, il recherche une correspondance dans l’ensemble d’ACR.
    1. Si le Dockerfile est introuvable, le système génère une nouvelle image (qui sera mise en cache et envoyée à l’espace de travail ACR).
1. Téléchargement de votre Fichier projet compressé vers le stockage temporaire sur le nœud de calcul
1. Décompression du Fichier projet
1. Nœud de calcul exécutant `python <entry script> <arguments>`
1. Enregistrement des journaux, des fichiers de modèle et des autres fichiers écrits dans `./outputs` dans le compte de stockage associé à l’espace de travail
1. Scale down du calcul, notamment la suppression du stockage temporaire (en relation avec Kubernetes)

### <a name="azure-ml-router"></a>Routeur Azure ML

Le composant frontal (azureml-fe) qui achemine les demandes d’inférence entrantes vers les services déployés se met à l’échelle automatiquement selon les besoins. La mise à l’échelle du composant azureml-fe se fait en fonction de l’objet et de la taille (nombre de nœuds) du cluster AKS. L’objet et les nœuds du cluster sont configurés lorsque vous [créez ou attachez un cluster AKS](how-to-create-attach-kubernetes.md). Il existe un service azureml-fe par cluster, susceptible de s’exécuter sur plusieurs pods.

> [!IMPORTANT]
> Lorsque vous utilisez un cluster configuré comme __dev-test__, le processus de mise à l’échelle automatique est **désactivé**.

Azureml-fe met à l’échelle aussi bien verticalement, de façon à utiliser plus de cœurs, qu’horizontalement, de façon à utiliser plus de pods. En cas de choix d’un scale-up, on tient compte du temps nécessaire pour acheminer les demandes d’inférence entrantes. Si cette durée dépasse le seuil, un scale-up est effectué. Si le temps nécessaire pour acheminer les demandes entrantes continue de dépasser le seuil, un scale-out est effectué.

En cas de scale-down et de scale-in, on tient compte de l’utilisation du processeur. Si le seuil d’utilisation du processeur est atteint, c’est le serveur frontal qui est mis à l’échelle en premier lieu. Si l’utilisation du processeur tombe au seuil du scale-in, une opération de scale-in est effectuée. Les opérations de scale-up et de scale-out se produisent uniquement si le cluster dispose de ressources suffisantes.

## <a name="deploy-to-aks"></a>Déployer sur AKS

Pour déployer un modèle sur Azure Kubernetes Service, créez une __configuration de déploiement__ décrivant les ressources de calcul nécessaires. Par exemple, le nombre de cœurs et la mémoire. Vous avez également besoin d’une __configuration d’inférence__ décrivant l’environnement nécessaire pour héberger le modèle et le service Web. Pour plus d’informations sur la création de la configuration d’inférence, consultez la section [Comment et où déployer des modèles ?](how-to-deploy-and-where.md)

> [!NOTE]
> Le nombre de modèles à déployer est limité à 1 000 modèles par déploiement (par conteneur).

<a id="using-the-cli"></a>

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core.webservice import AksWebservice, Webservice
from azureml.core.model import Model

aks_target = AksCompute(ws,"myaks")
# If deploying to a cluster configured for dev/test, ensure that it was created with enough
# cores and memory to handle this deployment configuration. Note that memory is also used by
# things such as dependencies and AML components.
deployment_config = AksWebservice.deploy_configuration(cpu_cores = 1, memory_gb = 1)
service = Model.deploy(ws, "myservice", [model], inference_config, deployment_config, aks_target)
service.wait_for_deployment(show_output = True)
print(service.state)
print(service.get_logs())
```

Pour plus d’informations sur les classes, les méthodes et les paramètres utilisés dans cet exemple, consultez les documents de référence suivants :

* [AksCompute](/python/api/azureml-core/azureml.core.compute.aks.akscompute?preserve-view=true&view=azure-ml-py)
* [AksWebservice.deploy_configuration](/python/api/azureml-core/azureml.core.webservice.aks.aksservicedeploymentconfiguration?preserve-view=true&view=azure-ml-py)
* [Model.deploy](/python/api/azureml-core/azureml.core.model.model?preserve-view=true&view=azure-ml-py#&preserve-view=truedeploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-)
* [Webservice.wait_for_deployment](/python/api/azureml-core/azureml.core.webservice%28class%29?preserve-view=true&view=azure-ml-py#&preserve-view=truewait-for-deployment-show-output-false-)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour déployer à l’aide de la CLI, utilisez la commande suivante. Remplacez `myaks` par le nom de la cible de calcul AKS. Remplacez `mymodel:1` par le nom et la version du modèle inscrit. Remplacez `myservice` par le nom à attribuer à ce service :

```azurecli-interactive
az ml model deploy -ct myaks -m mymodel:1 -n myservice -ic inferenceconfig.json -dc deploymentconfig.json
```

[!INCLUDE [deploymentconfig](../../includes/machine-learning-service-aks-deploy-config.md)]

Pour plus d’informations, consultez les informations de référence sur [az ml model deploy](/cli/azure/ext/azure-cli-ml/ml/model?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-model-deploy).

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Pour plus d’informations sur l’utilisation de VS Code, consultez [déployer sur AKS via l’extension VS Code](tutorial-train-deploy-image-classification-model-vscode.md#deploy-the-model).

> [!IMPORTANT]
> Le déploiement via VS Code nécessite que le cluster AKS soit créé ou attaché à votre espace de travail à l’avance.

---

### <a name="autoscaling"></a>Mise à l’échelle automatique

Le composant qui gère la mise à l’échelle automatique pour les déploiements de modèles Azure ML est azureml-fe, qui est un routeur de demande intelligent. Étant donné que toutes les demandes d’inférence passent par lui, il possède les données nécessaires pour mettre à l’échelle automatiquement les modèles déployés.

> [!IMPORTANT]
> * **N’activez pas l’Autoscaler de pods horizontaux (HPA) Kubernetes pour les déploiements de modèles**. Cela mettrait en concurrence les deux composants de mise à l’échelle automatique. Azureml-fe est conçu pour mettre à l’échelle automatiquement les modèles déployés par Azure ML, dans lesquels HPA devrait deviner ou estimer l’utilisation du modèle à partir d’une mesure générique telle que l’utilisation du processeur ou une configuration de métrique personnalisée.
> 
> * **Azureml-fe ne met pas à l’échelle le nombre de nœuds d’un cluster AKS**, car cela pourrait entraîner une augmentation inattendue du coût. Au lieu de cela, **il met à l’échelle le nombre de réplicas du modèle** dans les limites du cluster physique. Si vous devez mettre à l’échelle le nombre de nœuds au sein du cluster, vous pouvez mettre à l’échelle le cluster manuellement ou [configurer le programme de mise à l’échelle automatique du cluster AKS](../aks/cluster-autoscaler.md).

La mise à l’échelle automatique peut être contrôlée en définissant les paramètres `autoscale_target_utilization`, `autoscale_min_replicas` et `autoscale_max_replicas` pour le service web AKS. L’exemple suivant montre comment activer la mise à l’échelle automatique :

```python
aks_config = AksWebservice.deploy_configuration(autoscale_enabled=True, 
                                                autoscale_target_utilization=30,
                                                autoscale_min_replicas=1,
                                                autoscale_max_replicas=4)
```

La décision d’effectuer un scale-up/down est basée sur l’utilisation des réplicas de conteneur actuels. Le nombre de réplicas occupés (qui traitent une demande) divisé par le nombre total de réplicas actuels donne l’utilisation en cours. Si ce nombre dépasse `autoscale_target_utilization`, d’autres réplicas sont créés. S’il est inférieur, des réplicas sont supprimés. Par défaut, le pourcentage d’utilisation ciblé est 70 %.

La décision d’ajouter des réplicas est hâtive et rapide (environ 1 seconde). La décision de supprimer des réplicas est prudente (environ 1 minute).

Vous pouvez calculer le nombre de réplicas nécessaires à l’aide du code suivant :

```python
from math import ceil
# target requests per second
targetRps = 20
# time to process the request (in seconds)
reqTime = 10
# Maximum requests per container
maxReqPerContainer = 1
# target_utilization. 70% in this example
targetUtilization = .7

concurrentRequests = targetRps * reqTime / targetUtilization

# Number of container replicas
replicas = ceil(concurrentRequests / maxReqPerContainer)
```

Pour plus d’informations sur la configuration de `autoscale_target_utilization`, `autoscale_max_replicas` et `autoscale_min_replicas`, consultez les informations de référence sur le module [AksWebservice](/python/api/azureml-core/azureml.core.webservice.akswebservice?preserve-view=true&view=azure-ml-py).

## <a name="deploy-models-to-aks-using-controlled-rollout-preview"></a>Déployer des modèles sur AKS à l’aide d’un déploiement contrôlé (préversion)

Analyser et promouvoir des versions de modèle de manière contrôlée à l’aide de points de terminaison. Vous pouvez déployer jusqu’à six versions derrière un seul point de terminaison. Les points de terminaison offrent les capacités suivantes :

* Configurez le __pourcentage du trafic de scoring envoyé à chaque point de terminaison__. Par exemple, acheminez 20 % du trafic vers le point de terminaison « test » et 80 % vers « production ».

    > [!NOTE]
    > Si 100 % du trafic n’est pas utilisé, tout pourcentage restant est acheminé vers la version __par défaut__ du point de terminaison. Par exemple, si vous configurez que 10 % du trafic ira à la version du point de terminaison « test » et 30 % à « prod », les 60 % restants sont envoyés à la version par défaut du point de terminaison.
    >
    > La première version créée du point de terminaison est automatiquement configurée comme version par défaut. Vous pouvez modifier cela en définissant `is_default=True` lors de la création ou de la mise à jour d’une version de point de terminaison.
     
* Marquez une version de point de terminaison comme __contrôle__ ou __traitement__. Par exemple, la version actuelle du point de terminaison de production peut être le contrôle, tandis que les éventuels nouveaux modèles sont déployés en tant que versions de traitement. Après évaluation des performances des versions de traitement, si l’une d’elles obtient de meilleures performances que le contrôle actuel, elle peut être promue comme nouvelle version de production/contrôle.

    > [!NOTE]
    > Vous ne pouvez avoir qu’__un seul__ contrôle. Vous pouvez avoir plusieurs traitements.

Vous pouvez activer App Insights pour afficher les métriques opérationnelles des points de terminaison et des versions déployées.

### <a name="create-an-endpoint"></a>Créer un point de terminaison
Une fois que vous êtes prêt à déployer vos modèles, créez un point de terminaison de notation et déployez votre première version. L’exemple suivant montre comment déployer et créer le point de terminaison à l’aide du Kit de développement logiciel (SDK). Le premier déploiement sera défini comme la version par défaut, ce qui signifie que tout percentile de trafic non spécifié dans l’ensemble des versions ira à la version par défaut.  

> [!TIP]
> Dans l’exemple suivant, la configuration définit que la version initiale du point de terminaison traite 20 % du trafic. Étant donné qu’il s’agit du premier point de terminaison, il s’agit également de la version par défaut. Et puisque nous n’avons pas d’autres versions pour recevoir les 80 % restants du trafic, ce trafic sera également acheminé vers la version par défaut. Tant que d’autres versions recevant un pourcentage du trafic n’ont pas été déployées, celle-ci reçoit de fait 100 % du trafic.

```python
import azureml.core,
from azureml.core.webservice import AksEndpoint
from azureml.core.compute import AksCompute
from azureml.core.compute import ComputeTarget
# select a created compute
compute = ComputeTarget(ws, 'myaks')

# define the endpoint and version name
endpoint_name = "mynewendpoint"
version_name= "versiona"
# create the deployment config and define the scoring traffic percentile for the first deployment
endpoint_deployment_config = AksEndpoint.deploy_configuration(cpu_cores = 0.1, memory_gb = 0.2,
                                                              enable_app_insights = True,
                                                              tags = {'sckitlearn':'demo'},
                                                              description = "testing versions",
                                                              version_name = version_name,
                                                              traffic_percentile = 20)
 # deploy the model and endpoint
 endpoint = Model.deploy(ws, endpoint_name, [model], inference_config, endpoint_deployment_config, compute)
 # Wait for he process to complete
 endpoint.wait_for_deployment(True)
 ```

### <a name="update-and-add-versions-to-an-endpoint"></a>Mettre à jour un point de terminaison et y ajouter des versions

Ajoutez une autre version à votre point de terminaison et configurez le centile de trafic de notation passant à la version. Il existe deux types de versions : version de contrôle et version de traitement. Il peut y avoir plusieurs versions de traitement pour faciliter la comparaison avec une version de contrôle unique.

> [!TIP]
> La deuxième version, créée par l’extrait de code suivant, accepte 10 % du trafic. La première version est configurée pour 20 %, donc seulement 30 % du trafic est configuré pour des versions spécifiques. Les 70 % restants sont envoyés à la première version du point de terminaison, car il s’agit également de la version par défaut.

 ```python
from azureml.core.webservice import AksEndpoint

# add another model deployment to the same endpoint as above
version_name_add = "versionb"
endpoint.create_version(version_name = version_name_add,
                        inference_config=inference_config,
                        models=[model],
                        tags = {'modelVersion':'b'},
                        description = "my second version",
                        traffic_percentile = 10)
endpoint.wait_for_deployment(True)
```

Mettez à jour les versions existantes ou supprimez-les dans un point de terminaison. Vous pouvez modifier le type par défaut, le type de contrôle et le centile de trafic de la version. Dans l’exemple suivant, la deuxième version augmente son part de trafic à 40 % et est désormais la valeur par défaut.

> [!TIP]
> Après l’extrait de code suivant, la deuxième version est désormais la version par défaut. Elle est maintenant configurée pour 40 %, tandis que la version d’origine est toujours configurée pour 20 %. Cela signifie que 40 % du trafic n’est pas utilisé pour les configurations de version. Le trafic restant est acheminé vers la deuxième version, car il s’agit désormais de la version par défaut. Elle reçoit de fait 80 % du trafic.

 ```python
from azureml.core.webservice import AksEndpoint

# update the version's scoring traffic percentage and if it is a default or control type
endpoint.update_version(version_name=endpoint.versions["versionb"].name,
                        description="my second version update",
                        traffic_percentile=40,
                        is_default=True,
                        is_control_version_type=True)
# Wait for the process to complete before deleting
endpoint.wait_for_deployment(true)
# delete a version in an endpoint
endpoint.delete_version(version_name="versionb")

```

## <a name="web-service-authentication"></a>Authentification de service web

Lors du déploiement sur Azure Kubernetes Service, l’authentification __basée sur les clés__ est activée par défaut. Vous pouvez également activer l'authentification __par jeton__. L'authentification par jeton exige que les clients utilisent un compte Azure Active Directory pour demander un jeton d'authentification, qui est utilisé pour adresser des requêtes au service déployé.

Pour __désactiver__ l’authentification, définissez le paramètre `auth_enabled=False` lors de la création de la configuration de déploiement. L’exemple suivant désactive l’authentification à l’aide du kit de développement logiciel (SDK) :

```python
deployment_config = AksWebservice.deploy_configuration(cpu_cores=1, memory_gb=1, auth_enabled=False)
```

Pour plus d’informations sur l'authentification à partir d’une application cliente, consultez [Utiliser un modèle Azure Machine Learning déployé en tant que service web](how-to-consume-web-service.md).

### <a name="authentication-with-keys"></a>Authentification avec des clés

Si l’authentification par clé est activée, vous pouvez utiliser la méthode `get_keys` pour récupérer des clés d’authentification primaire et secondaire :

```python
primary, secondary = service.get_keys()
print(primary)
```

> [!IMPORTANT]
> Si vous devez regénérer une clé, utilisez [`service.regen_key`](/python/api/azureml-core/azureml.core.webservice%28class%29?preserve-view=true&view=azure-ml-py)

### <a name="authentication-with-tokens"></a>Authentification avec des jetons

Pour activer l’authentification par jeton, utilisez le paramètre `token_auth_enabled=True` lorsque vous créez ou mettez à jour un déploiement. L’exemple suivant active l’authentification par jeton à l’aide du kit de développement logiciel (SDK) :

```python
deployment_config = AksWebservice.deploy_configuration(cpu_cores=1, memory_gb=1, token_auth_enabled=True)
```

Si l’authentification par jeton est activée, vous pouvez utiliser la méthode `get_token` pour récupérer un jeton JWT et le délai d’expiration du jeton :

```python
token, refresh_by = service.get_token()
print(token)
```

> [!IMPORTANT]
> Vous devrez demander un nouveau jeton après l’heure de `refresh_by` du jeton.
>
> Microsoft recommande vivement de créer votre espace de travail Azure Machine Learning dans la même région que celle de votre Azure Kubernetes Service. Pour s’authentifier avec un jeton, le service web appelle la région dans laquelle votre espace de travail Azure Machine Learning est créé. Si la région de votre espace de travail est indisponible, vous ne pouvez pas extraire de jeton pour votre service web, même si votre cluster se trouve dans une région différente de celle de votre espace de travail. Cela a pour effet d'empêcher l'authentification par jeton tant que la région de votre espace de travail n'est pas disponible. Par ailleurs, plus la distance entre la région de votre cluster et celle de votre espace de travail est élevée, plus l’extraction de jeton prend de temps.
>
> Pour récupérer un jeton, vous devez utiliser le Kit de développement logiciel (SDK) Azure Machine Learning ou la commande [az ml service obten-access-token](/cli/azure/ext/azure-cli-ml/ml/service?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-service-get-access-token).


### <a name="vulnerability-scanning"></a>Analyse des vulnérabilités

Azure Security Center fournit des fonctionnalités unifiées de gestion de la sécurité et de protection avancée contre les menaces sur l’ensemble des charges de travail cloud hybrides. Vous devez autoriser Azure Security Center à analyser vos ressources et suivre ses recommandations. Pour plus d’informations, consultez [Intégration d’Azure Kubernetes Service à Security Center](../security-center/defender-for-kubernetes-introduction.md).

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser Azure RBAC pour l’autorisation Kubernetes](../aks/manage-azure-rbac.md)
* [Sécuriser l’environnement d’inférence avec un réseau virtuel Microsoft Azure](how-to-secure-inferencing-vnet.md)
* [Guide pratique pour déployer un modèle à l’aide d’une image Docker personnalisée](how-to-deploy-custom-docker-image.md)
* [Résolution des problèmes liés au déploiement](how-to-troubleshoot-deployment.md)
* [Mettre à jour un service web](how-to-deploy-update-web-service.md)
* [Utiliser TLS pour sécuriser un service web par le biais d’Azure Machine Learning](how-to-secure-web-service.md)
* [Utiliser un modèle ML déployé en tant que service web](how-to-consume-web-service.md)
* [Superviser vos modèles Azure Machine Learning avec Application Insights](how-to-enable-app-insights.md)
* [Collecter des données pour des modèles en production](how-to-enable-data-collection.md)

---
title: Déployer un modèle à utiliser avec Recherche cognitive
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser Azure Machine Learning afin de déployer un modèle pour l’utiliser avec Recherche cognitive. Le modèle est utilisé comme une compétence personnalisée pour enrichir l’expérience de recherche.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: cgronlun
author: cjgronlund
ms.reviewer: larryfr
ms.date: 06/11/2020
ms.custom: deploy
ms.openlocfilehash: e1eebf88b72c87ce9db02760c5c44a0aa25c57cc
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93305912"
---
# <a name="deploy-a-model-for-use-with-cognitive-search"></a>Déployer un modèle à utiliser avec Recherche cognitive


Cet article vous explique comment utiliser Azure Machine Learning pour déployer un modèle pour l’utiliser avec [Recherche cognitive Azure](../search/search-what-is-azure-search.md).

Recherche cognitive effectue un traitement de contenu sur du contenu hétérogène afin de le rendre interrogeable par des humains ou des applications. Ce processus peut être amélioré en utilisant un modèle déployé depuis Azure Machine Learning.

Azure Machine Learning peut déployer un modèle entraîné en tant que service web. Le service web est ensuite incorporé dans une _compétence_ Recherche cognitive, qui devient partie intégrante du pipeline de traitement.

> [!IMPORTANT]
> Les informations présentes dans cet article sont spécifiques au déploiement du modèle. Il fournit des informations sur les configurations de déploiement prises en charge qui permettent au modèle d’être utilisé par Recherche cognitive.
>
> Pour plus d’informations sur la configuration de Recherche cognitive pour utiliser le modèle déployé, consultez le tutoriel [Créer et déployer une compétence personnalisée avec Azure Machine Learning](../search/cognitive-search-tutorial-aml-custom-skill.md).
>
> Pour l’exemple sur lequel est basé le tutoriel, consultez [https://github.com/Azure-Samples/azure-search-python-samples/tree/master/AzureML-Custom-Skill](https://github.com/Azure-Samples/azure-search-python-samples/tree/master/AzureML-Custom-Skill).

Lors du déploiement d’un modèle à utiliser avec Recherche cognitive Azure, le déploiement doit remplir les conditions suivantes :

* Utiliser Azure Kubernetes Service pour héberger le modèle pour l’inférence.
* Activer le protocole TLS (Transport Layer Security) pour Azure Kubernetes Service. TLS est utilisé pour sécuriser les communications HTTPS entre Recherche cognitive et le modèle déployé.
* Le script d’entrée doit utiliser le package `inference_schema` pour générer un schéma OpenAPI (Swagger) pour le service.
* Le script d’entrée doit également accepter les données JSON en entrée et générer du JSON en sortie.


## <a name="prerequisites"></a>Prérequis

* Un espace de travail Azure Machine Learning. Pour plus d’informations, voir la page [Créer un espace de travail Azure Machine Learning](how-to-manage-workspace.md).

* Un environnement de développement Python dans lequel le SDK Azure Machine Learning est installé. Pour plus d’informations, consultez [Kit SDK Azure Machine Learning](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py).  

* Un modèle inscrit. Si vous n’avez pas de modèle, utilisez l’exemple de notebook sur [https://github.com/Azure-Samples/azure-search-python-samples/tree/master/AzureML-Custom-Skill](https://github.com/Azure-Samples/azure-search-python-samples/tree/master/AzureML-Custom-Skill).

* Compréhension générale de [comment et où déployer des modèles](how-to-deploy-and-where.md).

## <a name="connect-to-your-workspace"></a>Se connecter à un espace de travail

Un espace de travail Azure Machine Learning fournit un emplacement centralisé pour travailler avec tous les artefacts que vous créez quand vous utilisez Azure Machine Learning. L’espace de travail conserve un historique de toutes les exécutions d’entraînement, y compris les journaux d’activité, les métriques, les sorties et un instantané de vos scripts.

Pour vous connecter à un espace de travail existant, utilisez le code suivant :

> [!IMPORTANT]
> Cet extrait de code s’attend à ce que la configuration de l’espace de travail soit enregistrée dans le répertoire actif ou son parent. Pour plus d’informations, consultez [Créer et gérer des espaces de travail Azure Machine Learning](how-to-manage-workspace.md). Pour plus d’informations sur l’enregistrement de la configuration dans un fichier, consultez [Créer un fichier de configuration d’espace de travail](how-to-configure-environment.md#workspace).

```python
from azureml.core import Workspace

try:
    # Load the workspace configuration from local cached inffo
    ws = Workspace.from_config()
    print(ws.name, ws.location, ws.resource_group, ws.location, sep='\t')
    print('Library configuration succeeded')
except:
    print('Workspace not found')
```

## <a name="create-a-kubernetes-cluster"></a>Créer un cluster Kubernetes

**Durée estimée** : environ 20 minutes.

Un cluster Kubernetes est un ensemble d’instances de machines virtuelles (appelées « nœuds ») utilisées pour exécuter des applications conteneurisées.

Quand vous déployez un modèle depuis Azure Machine Learning vers Azure Kubernetes Service, le modèle et toutes les ressources nécessaires pour l’héberger en tant que service web sont empaquetés dans un conteneur Docker. Ce conteneur est ensuite déployé sur le cluster.

Le code suivant montre comment créer un cluster Azure Kubernetes Service (AKS) pour votre espace de travail :

> [!TIP]
> Vous pouvez également attacher un service Azure Kubernetes existant à votre espace de travail Azure Machine Learning. Pour plus d’informations, consultez [Guide pratique pour déployer des modèles sur Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md).

> [!IMPORTANT]
> Notez que le code utilise la méthode `enable_ssl()` pour activer le protocole TLS (Transport Layer Security) pour le cluster. Ceci est nécessaire quand vous prévoyez d’utiliser le modèle déployé depuis Cognitive Services.

```python
from azureml.core.compute import AksCompute, ComputeTarget
# Create or attach to an AKS inferencing cluster

# Create the provisioning configuration with defaults
prov_config = AksCompute.provisioning_configuration()

# Enable TLS (sometimes called SSL) communications
# Leaf domain label generates a name using the formula
#  "<leaf-domain-label>######.<azure-region>.cloudapp.azure.net"
#  where "######" is a random series of characters
prov_config.enable_ssl(leaf_domain_label = "contoso")

cluster_name = 'amlskills'
# Try to use an existing compute target by that name.
# If one doesn't exist, create one.
try:
    
    aks_target = ComputeTarget(ws, cluster_name)
    print("Attaching to existing cluster")
except Exception as e:
    print("Creating new cluster")
    aks_target = ComputeTarget.create(workspace = ws, 
                                  name = cluster_name, 
                                  provisioning_configuration = prov_config)
    # Wait for the create process to complete
    aks_target.wait_for_completion(show_output = True)
```

> [!IMPORTANT]
> Azure vous facture tant que le cluster AKS existe. Veillez à supprimer votre cluster AKS lorsque vous n’en avec plus besoin.

Pour plus d’informations sur l’utilisation d’AKS avec Azure Machine Learning, consultez [Guide pratique pour déployer sur Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md).

## <a name="write-the-entry-script"></a>Écrire le script d’entrée

Le script d’entrée reçoit les données envoyées au service web, les transmet au modèle et retourne les résultats du scoring. Le script suivant charge le modèle au démarrage, puis l’utilise pour effectuer le scoring des données. Ce fichier est parfois appelé `score.py`.

> [!TIP]
> Le script d’entrée est spécifique de votre modèle. Par exemple, le script doit connaître le framework à utiliser avec votre modèle, les formats de données, etc.

> [!IMPORTANT]
> Quand vous prévoyez d’utiliser le modèle déployé à partir d’Azure Cognitive Services, vous devez utiliser le package `inference_schema` pour activer la génération de schéma pour le déploiement. Ce package fournit des décorateurs qui vous permettent de définir le format des données d’entrée et de sortie pour le service web qui effectue l’inférence à l’aide du modèle.

```python
from azureml.core.model import Model
from nlp_architect.models.absa.inference.inference import SentimentInference
from spacy.cli.download import download as spacy_download
import traceback
import json
# Inference schema for schema discovery
from inference_schema.schema_decorators import input_schema, output_schema
from inference_schema.parameter_types.numpy_parameter_type import NumpyParameterType
from inference_schema.parameter_types.standard_py_parameter_type import StandardPythonParameterType

def init():
    """
    Set up the ABSA model for Inference  
    """
    global SentInference
    spacy_download('en')
    aspect_lex = Model.get_model_path('hotel_aspect_lex')
    opinion_lex = Model.get_model_path('hotel_opinion_lex') 
    SentInference = SentimentInference(aspect_lex, opinion_lex)

# Use inference schema decorators and sample input/output to
# build the OpenAPI (Swagger) schema for the deployment
standard_sample_input = {'text': 'a sample input record containing some text' }
standard_sample_output = {"sentiment": {"sentence": "This place makes false booking prices, when you get there, they say they do not have the reservation for that day.", 
                                        "terms": [{"text": "hotels", "type": "AS", "polarity": "POS", "score": 1.0, "start": 300, "len": 6}, 
                                                  {"text": "nice", "type": "OP", "polarity": "POS", "score": 1.0, "start": 295, "len": 4}]}}
@input_schema('raw_data', StandardPythonParameterType(standard_sample_input))
@output_schema(StandardPythonParameterType(standard_sample_output))    
def run(raw_data):
    try:
        # Get the value of the 'text' field from the JSON input and perform inference
        input_txt = raw_data["text"]
        doc = SentInference.run(doc=input_txt)
        if doc is None:
            return None
        sentences = doc._sentences
        result = {"sentence": doc._doc_text}
        terms = []
        for sentence in sentences:
            for event in sentence._events:
                for x in event:
                    term = {"text": x._text, "type":x._type.value, "polarity": x._polarity.value, "score": x._score,"start": x._start,"len": x._len }
                    terms.append(term)
        result["terms"] = terms
        print("Success!")
        # Return the results to the client as a JSON document
        return {"sentiment": result}
    except Exception as e:
        result = str(e)
        # return error message back to the client
        print("Failure!")
        print(traceback.format_exc())
        return json.dumps({"error": result, "tb": traceback.format_exc()})
```

Pour plus d’informations sur les scripts d’entrée, consultez la page [Comment et où déployer](how-to-deploy-and-where.md).

## <a name="define-the-software-environment"></a>Définir l’environnement logiciel

La classe d’environnement est utilisée pour définir les dépendances Python pour le service. Il inclut les dépendances requises aussi bien par le modèle que par le script d’entrée. Dans cet exemple, elle installe les packages à partir de l’index pypi normal ainsi qu’à partir d’un dépôt GitHub. 

```python
from azureml.core.conda_dependencies import CondaDependencies 
from azureml.core import Environment

conda = None
pip = ["azureml-defaults", "azureml-monitoring", 
       "git+https://github.com/NervanaSystems/nlp-architect.git@absa", 'nlp-architect', 'inference-schema',
       "spacy==2.0.18"]

conda_deps = CondaDependencies.create(conda_packages=None, pip_packages=pip)

myenv = Environment(name='myenv')
myenv.python.conda_dependencies = conda_deps
```

Pour plus d’informations sur les environnements , consultez [Créer et gérer des environnements pour la formation et le déploiement](how-to-use-environments.md).

## <a name="define-the-deployment-configuration"></a>Définir la configuration du déploiement

La configuration du déploiement définit l’environnement d’hébergement Azure Kubernetes Service utilisé pour exécuter le service web.

> [!TIP]
> Si vous n’êtes pas sûr des besoins en mémoire, en processeur ou en GPU de votre déploiement, vous pouvez utiliser le profilage pour les découvrir. Pour plus d’informations, consultez [Comment et où déployer un modèle](how-to-deploy-and-where.md).

```python
from azureml.core.model import Model
from azureml.core.webservice import Webservice
from azureml.core.image import ContainerImage
from azureml.core.webservice import AksWebservice, Webservice

# If deploying to a cluster configured for dev/test, ensure that it was created with enough
# cores and memory to handle this deployment configuration. Note that memory is also used by
# things such as dependencies and AML components.

aks_config = AksWebservice.deploy_configuration(autoscale_enabled=True, 
                                                       autoscale_min_replicas=1, 
                                                       autoscale_max_replicas=3, 
                                                       autoscale_refresh_seconds=10, 
                                                       autoscale_target_utilization=70,
                                                       auth_enabled=True, 
                                                       cpu_cores=1, memory_gb=2, 
                                                       scoring_timeout_ms=5000, 
                                                       replica_max_concurrent_requests=2, 
                                                       max_request_wait_time=5000)
```

Pour plus d’informations, consultez la documentation de référence pour [AksService.deploy_configuration](/python/api/azureml-core/azureml.core.webservice.akswebservice?view=azure-ml-py&preserve-view=true#&preserve-view=truedeploy-configuration-autoscale-enabled-none--autoscale-min-replicas-none--autoscale-max-replicas-none--autoscale-refresh-seconds-none--autoscale-target-utilization-none--collect-model-data-none--auth-enabled-none--cpu-cores-none--memory-gb-none--enable-app-insights-none--scoring-timeout-ms-none--replica-max-concurrent-requests-none--max-request-wait-time-none--num-replicas-none--primary-key-none--secondary-key-none--tags-none--properties-none--description-none--gpu-cores-none--period-seconds-none--initial-delay-seconds-none--timeout-seconds-none--success-threshold-none--failure-threshold-none--namespace-none--token-auth-enabled-none--compute-target-name-none-).

## <a name="define-the-inference-configuration"></a>Définir la configuration de l’inférence

La configuration de l’inférence pointe vers le script d’entrée et l’objet d’environnement :

```python
from azureml.core.model import InferenceConfig
inf_config = InferenceConfig(entry_script='score.py', environment=myenv)
```

Pour plus d’informations, consultez la documentation de référence pour [InferenceConfig](/python/api/azureml-core/azureml.core.model.inferenceconfig?preserve-view=true&view=azure-ml-py).

## <a name="deploy-the-model"></a>Déployer le modèle

Déployez le modèle sur votre cluster AKS et attendez qu’il crée votre service. Dans cet exemple, deux modèles inscrits sont chargés à partir du Registre et déployés sur AKS. Après le déploiement, le fichier `score.py` dans le déploiement charge ces modèles et les utilise pour effectuer l’inférence.

```python
from azureml.core.webservice import AksWebservice, Webservice

c_aspect_lex = Model(ws, 'hotel_aspect_lex')
c_opinion_lex = Model(ws, 'hotel_opinion_lex') 
service_name = "hotel-absa-v2"

aks_service = Model.deploy(workspace=ws,
                           name=service_name,
                           models=[c_aspect_lex, c_opinion_lex],
                           inference_config=inf_config,
                           deployment_config=aks_config,
                           deployment_target=aks_target,
                           overwrite=True)

aks_service.wait_for_deployment(show_output = True)
print(aks_service.state)
```

Pour plus d’informations, consultez la documentation de référence sur [le modèle](/python/api/azureml-core/azureml.core.model.model?preserve-view=true&view=azure-ml-py).

## <a name="issue-a-sample-query-to-your-service"></a>Envoyer un exemple de requête à votre service

L’exemple suivant utilise les informations de déploiement stockées dans la variable `aks_service` par la section de code précédente. Il utilise cette variable pour récupérer l’URL du scoring et le jeton d’authentification nécessaires pour communiquer avec le service :

```python
import requests
import json

primary, secondary = aks_service.get_keys()

# Test data
input_data = '{"raw_data": {"text": "This is a nice place for a relaxing evening out with friends. The owners seem pretty nice, too. I have been there a few times including last night. Recommend."}}'

# Since authentication was enabled for the deployment, set the authorization header.
headers = {'Content-Type':'application/json',  'Authorization':('Bearer '+ primary)} 

# Send the request and display the results
resp = requests.post(aks_service.scoring_uri, input_data, headers=headers)
print(resp.text)
```

Le résultat retourné par le service est similaire au code JSON suivant :

```json
{"sentiment": {"sentence": "This is a nice place for a relaxing evening out with friends. The owners seem pretty nice, too. I have been there a few times including last night. Recommend.", "terms": [{"text": "place", "type": "AS", "polarity": "POS", "score": 1.0, "start": 15, "len": 5}, {"text": "nice", "type": "OP", "polarity": "POS", "score": 1.0, "start": 10, "len": 4}]}}
```

## <a name="connect-to-cognitive-search"></a>Se connecter à Recherche cognitive

Pour plus d’informations sur l’utilisation de ce modèle à partir de Recherche cognitive, consultez le tutoriel [Créer et déployer une compétence personnalisée avec Azure Machine Learning](../search/cognitive-search-tutorial-aml-custom-skill.md).

## <a name="clean-up-the-resources"></a>Nettoyer les ressources

Si vous avez créé le cluster AKS spécifiquement pour cet exemple, supprimez vos ressources une fois que vous l’avez testé avec Recherche cognitive.

> [!IMPORTANT]
> Azure vous facture en fonction de la durée de déploiement du cluster AKS. Veillez à le nettoyer lorsque vous n’en avez plus besoin.

```python
aks_service.delete()
aks_target.delete()
```

## <a name="next-steps"></a>Étapes suivantes

* [Créer et déployer une compétence personnalisée avec Azure Machine Learning](../search/cognitive-search-tutorial-aml-custom-skill.md)
---
title: Profiler l’utilisation du processeur et de la mémoire d’un modèle
titleSuffix: Azure Machine Learning
description: Découvrez comment profiler votre modèle avant son déploiement. Le profilage détermine l’utilisation de la mémoire et du processeur de votre modèle.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: gopalv
author: gvashishtha
ms.date: 07/31/2020
ms.topic: conceptual
zone_pivot_groups: aml-control-methods
ms.reviewer: larryfr
ms.custom: deploy
ms.openlocfilehash: b9ae40b3d2673961f9b84ed702f18b25b79b6d0c
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93320400"
---
# <a name="profile-your-model-to-determine-resource-utilization"></a>Profiler votre modèle pour déterminer l’utilisation des ressources

Cet article explique comment profiler un modèle Machine Learning afin de déterminer la quantité de processeur et de mémoire que vous devrez allouer pour le modèle lors de son déploiement en tant que service web.

## <a name="prerequisites"></a>Prérequis

Cet article suppose que vous avez entraîné et inscrit un modèle à l’aide d’Azure Machine Learning. Consultez [cet exemple de tutoriel](how-to-train-scikit-learn.md) pour obtenir un exemple d’entraînement et d’inscription d’un modèle scikit-learn avec Azure Machine Learning.

## <a name="limitations"></a>Limites

* Le profilage ne fonctionnera pas lorsque le compte Azure Container Registry (ACR) de votre espace de travail se trouve derrière un réseau virtuel.

## <a name="run-the-profiler"></a>Exécuter le profileur

Une fois que vous avez inscrit votre modèle et préparé les autres composants nécessaires à son déploiement, vous pouvez déterminer l’UC et la mémoire dont le service déployé aura besoin. Le profilage teste le service qui exécute votre modèle et retourne des informations telles que l’utilisation de l’UC, l’utilisation de la mémoire et la latence de la réponse. Il fournit également une recommandation pour l’UC et la mémoire en fonction de l’utilisation des ressources.

Pour profiler votre modèle, vous aurez besoin des éléments suivants :
* Un modèle inscrit.
* Une configuration d’inférence basée sur votre script d’entrée et la définition de l’environnement d’inférence.
* Un jeu de données tabulaires à une seule colonne, où chaque ligne contient une chaîne représentant des exemples de données de requête.

> [!IMPORTANT]
> À ce stade, nous ne prenons en charge que le profilage des services qui s’attendent à ce que leurs données de requête soient une chaîne, par exemple : chaîne JSON sérialisée, texte, image sérialisée de chaîne, etc. Le contenu de chaque ligne du jeu de données (chaîne) est placé dans le corps de la requête HTTP et envoyé au service qui encapsule le modèle pour le scoring.

> [!IMPORTANT]
> Nous prenons uniquement en charge le profilage jusqu’à 2 processeurs dans les régions ChinaEast2 et USGovArizona.

Vous trouverez ci-dessous un exemple de création d’un jeu de données d’entrée pour profiler un service qui s’attend à ce que ses données de requête entrantes contiennent une chaîne JSON sérialisée. Dans ce cas, nous avons créé un jeu de données basé sur 100 instances du même contenu de données de requête. Dans les scénarios réels, nous vous suggérons d’utiliser des jeux de données plus volumineux contenant différentes entrées, en particulier si votre utilisation ou le comportement des ressources du modèle sont dépendants de l’entrée.

::: zone pivot="py-sdk"

```python
import json
from azureml.core import Datastore
from azureml.core.dataset import Dataset
from azureml.data import dataset_type_definitions

input_json = {'data': [[1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
                       [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]]}
# create a string that can be utf-8 encoded and
# put in the body of the request
serialized_input_json = json.dumps(input_json)
dataset_content = []
for i in range(100):
    dataset_content.append(serialized_input_json)
dataset_content = '\n'.join(dataset_content)
file_name = 'sample_request_data.txt'
f = open(file_name, 'w')
f.write(dataset_content)
f.close()

# upload the txt file created above to the Datastore and create a dataset from it
data_store = Datastore.get_default(ws)
data_store.upload_files(['./' + file_name], target_path='sample_request_data')
datastore_path = [(data_store, 'sample_request_data' +'/' + file_name)]
sample_request_data = Dataset.Tabular.from_delimited_files(
    datastore_path, separator='\n',
    infer_column_types=True,
    header=dataset_type_definitions.PromoteHeadersBehavior.NO_HEADERS)
sample_request_data = sample_request_data.register(workspace=ws,
                                                   name='sample_request_data',
                                                   create_new_version=True)
```

Une fois que le jeu de données contenant des exemples de données de requête est prêt, créez une configuration d’inférence. La configuration de l’inférence est basée sur le fichier score.py et la définition de l’environnement. L’exemple suivant montre comment créer une configuration d’inférence et exécuter le profilage :

```python
from azureml.core.model import InferenceConfig, Model
from azureml.core.dataset import Dataset


model = Model(ws, id=model_id)
inference_config = InferenceConfig(entry_script='path-to-score.py',
                                   environment=myenv)
input_dataset = Dataset.get_by_name(workspace=ws, name='sample_request_data')
profile = Model.profile(ws,
            'unique_name',
            [model],
            inference_config,
            input_dataset=input_dataset)

profile.wait_for_completion(True)

# see the result
details = profile.get_details()
```

::: zone-end

::: zone pivot="cli"


La commande suivante montre comment profiler un modèle à l’aide de l’interface CLI :

```azurecli-interactive
az ml model profile -g <resource-group-name> -w <workspace-name> --inference-config-file <path-to-inf-config.json> -m <model-id> --idi <input-dataset-id> -n <unique-name>
```

> [!TIP]
> Pour conserver les informations retournées par le profilage, utilisez des balises ou des propriétés pour le modèle. L’utilisation de balises ou de propriétés stocke les données avec le modèle dans le registre du modèle. Les exemples suivants illustrent l’ajout d’une nouvelle balise contenant les informations `requestedCpu` et `requestedMemoryInGb` :
>
> ```python
> model.add_tags({'requestedCpu': details['requestedCpu'],
>                 'requestedMemoryInGb': details['requestedMemoryInGb']})
> ```
>
> ```azurecli-interactive
> az ml model profile -g <resource-group-name> -w <workspace-name> --i <model-id> --add-tag requestedCpu=1 --add-tag requestedMemoryInGb=0.5
> ```

::: zone-end

## <a name="next-steps"></a>Étapes suivantes

* [Résoudre des problèmes d’échec de déploiement](how-to-troubleshoot-deployment.md)
* [Déployer dans Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md)
* [Créer des applications clientes pour utiliser des services web](how-to-consume-web-service.md)
* [Mettre à jour un service web](how-to-deploy-update-web-service.md)
* [Guide pratique pour déployer un modèle à l’aide d’une image Docker personnalisée](how-to-deploy-custom-docker-image.md)
* [Utiliser TLS pour sécuriser un service web par le biais d’Azure Machine Learning](how-to-secure-web-service.md)
* [Superviser vos modèles Azure Machine Learning avec Application Insights](how-to-enable-app-insights.md)
* [Collecter des données pour des modèles en production](how-to-enable-data-collection.md)
* [Créer des alertes d’événement et des déclencheurs pour les déploiements de modèle](how-to-use-event-grid.md)


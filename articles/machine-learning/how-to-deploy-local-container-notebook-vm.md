---
title: Déployer des modèles sur des instances de calcul
titleSuffix: Azure Machine Learning
description: Découvrez comment déployer vos modèles Azure Machine Learning en tant que service web en utilisant des instances de calcul.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, deploy
ms.author: mnark
author: MrudulaN
ms.reviewer: larryfr
ms.date: 03/05/2020
ms.openlocfilehash: 17b2d717ac6f244cb3ddddfdbc2f29581aa1f59f
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95527307"
---
# <a name="deploy-a-model-to-azure-machine-learning-compute-instances"></a>Déployer un modèle sur des instances de calcul Azure Machine Learning



Découvrez comment utiliser Azure Machine Learning pour déployer un modèle en tant que service web sur l’instance de calcul Azure Machine Learning. Utilisez des instances de calcul si l’une des conditions suivantes est vraie :

- Vous avez besoin de déployer et de valider rapidement votre modèle.
- Vous testez un modèle en cours de développement.

> [!TIP]
> Le déploiement d’un modèle à partir d’un Jupyter Notebook sur une instance de calcul vers un service web sur la même machine virtuelle est un _déploiement local_. Dans ce cas, l’ordinateur « local » est l’instance de calcul. Pour plus d’informations sur les déploiement, consultez [Déployer des modèles avec Azure Machine Learning](how-to-deploy-and-where.md).

## <a name="prerequisites"></a>Prérequis

- Un espace de travail Azure Machine Learning avec une instance de calcul en cours d’exécution. Pour plus d’informations, consultez [Configuration de l’environnement et de l’espace de travail](tutorial-1st-experiment-sdk-setup.md).

## <a name="deploy-to-the-compute-instances"></a>Déploiement sur des instances de calcul

Un exemple de notebook illustrant les déploiements locaux est inclus sur votre instance de calcul. Utilisez les étapes suivantes pour charger le notebook et déployer le modèle en tant que service web sur la machine virtuelle :

1. Dans [Azure Machine Learning Studio](https://ml.azure.com), sélectionnez vos instances de calcul Azure Machine Learning.

1. Ouvrez le sous-répertoire `samples-*`, puis ouvrez `how-to-use-azureml/deployment/deploy-to-local/register-model-deploy-local.ipynb`. Une fois ouvert, exécutez le notebook.

    ![Capture d’écran du service local en cours d’exécution sur le notebook](./media/how-to-deploy-local-container-notebook-vm/deploy-local-service.png)

1. Le notebook affiche l’URL et le port sur lequel le service s’exécute. Par exemple : `https://localhost:6789`. Vous pouvez également exécuter la cellule contenant `print('Local service port: {}'.format(local_service.port))` pour afficher ce port.

    ![Capture d’écran du port du service local en cours d’exécution](./media/how-to-deploy-local-container-notebook-vm/deploy-local-service-port.png)

1. Pour tester le service à partir d’une instance de calcul, utilisez l’URL `https://localhost:<local_service.port>`. Pour effectuer un test à partir d’un client distant, récupérez l’URL publique du service en cours d’exécution sur l’instance de calcul. L’URL publique peut être déterminée à l’aide de la formule suivante : 
    * Machine virtuelle de notebooks : `https://<vm_name>-<local_service_port>.<azure_region_of_workspace>.notebooks.azureml.net/score`. 
    * Instance de calcul : `https://<vm_name>-<local_service_port>.<azure_region_of_workspace>.instances.azureml.net/score`. 

    Par exemple, 
    * Machine virtuelle de notebooks : `https://vm-name-6789.northcentralus.notebooks.azureml.net/score` 
    * Instance de calcul : `https://vm-name-6789.northcentralus.instances.azureml.net/score`

## <a name="test-the-service"></a>Testez le service

Pour soumettre des exemples de données au service en cours d’exécution, utilisez le code suivant. Remplacez la valeur de `service_url` par l’URL de l’étape précédente :

> [!NOTE]
> Lors de l’authentification auprès d’un déploiement sur l’instance de calcul, l’authentification est effectuée à l’aide d’Azure Active Directory. L’appel à `interactive_auth.get_authentication_header()` dans l’exemple de code vous authentifie à l’aide d’AAD et retourne un en-tête qui peut ensuite être utilisé pour s’authentifier auprès du service sur l’instance de calcul. Pour plus d’informations, consultez [Configurer l’authentification pour des ressources et workflows Azure Machine Learning](how-to-setup-authentication.md#interactive-authentication).
>
> Lors de l’authentification auprès d’un déploiement sur Azure Kubernetes Service ou Azure Container Instances, une autre méthode d’authentification est utilisée. Pour plus d’informations sur le sujet, consultez [Configurer l’authentification pour les modèles Machine Learning déployés en tant que services web](how-to-authenticate-web-service.md).

```python
import requests
import json
from azureml.core.authentication import InteractiveLoginAuthentication

# Get a token to authenticate to the compute instance from remote
interactive_auth = InteractiveLoginAuthentication()
auth_header = interactive_auth.get_authentication_header()

# Create and submit a request using the auth header
headers = auth_header
# Add content type header
headers.update({'Content-Type':'application/json'})

# Sample data to send to the service
test_sample = json.dumps({'data': [
    [1,2,3,4,5,6,7,8,9,10],
    [10,9,8,7,6,5,4,3,2,1]
]})
test_sample = bytes(test_sample,encoding = 'utf8')

# Replace with the URL for your compute instance, as determined from the previous section
service_url = "https://vm-name-6789.northcentralus.notebooks.azureml.net/score"
# for a compute instance, the url would be https://vm-name-6789.northcentralus.instances.azureml.net/score
resp = requests.post(service_url, test_sample, headers=headers)
print("prediction:", resp.text)
```

## <a name="next-steps"></a>Étapes suivantes

* [Guide pratique pour déployer un modèle à l’aide d’une image Docker personnalisée](how-to-deploy-custom-docker-image.md)
* [Résolution des problèmes liés au déploiement](how-to-troubleshoot-deployment.md)
* [Utiliser TLS pour sécuriser un service web par le biais d’Azure Machine Learning](how-to-secure-web-service.md)
* [Utiliser un modèle ML déployé en tant que service web](how-to-consume-web-service.md)
* [Superviser vos modèles Azure Machine Learning avec Application Insights](how-to-enable-app-insights.md)
* [Collecter des données pour des modèles en production](how-to-enable-data-collection.md)

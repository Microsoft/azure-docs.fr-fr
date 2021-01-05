---
title: Déployer le service d’inférence chiffrée (version préliminaire)
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser Microsoft SEAL pour déployer un service de prédiction chiffrée pour la classification des images
author: luisquintanilla
ms.author: luquinta
ms.date: 07/09/2020
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, devx-track-python, deploy
ms.openlocfilehash: 3509530994b07a16fb1f2780fffc6fd27cf8aa7c
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93325514"
---
# <a name="how-to-deploy-an-encrypted-inferencing-web-service-preview"></a>Comment déployer un service web d’inférence chiffrée (version préliminaire)

Découvrez comment déployer un modèle de classification d’images en tant que service web d’inférence chiffrée dans [Azure Container Instances](../container-instances/index.yml) (ACI). Le service web est une image conteneur Docker qui contient le modèle et la logique de scoring.

Dans ce guide, vous utilisez Azure Machine Learning service pour :

> [!div class="checklist"]
> * Configurer vos environnements
> * Déployer le service web d’inférence chiffrée
> * Préparer les données de test
> * Faire des prédictions chiffrées
> * Nettoyer les ressources

ACI est une excellente solution pour comprendre et tester le workflow du modèle de déploiement. Pour les déploiements de production évolutifs, envisagez d’utiliser Azure Kubernetes Service. Pour plus d’informations, consultez [Comment et où déployer](./how-to-deploy-and-where.md).

La méthode de chiffrement utilisée dans cet exemple est [le chiffrement homomorphe](https://github.com/Microsoft/SEAL#homomorphic-encryption). Le chiffrement homomorphe permet d’effectuer des calculs sur des données chiffrées sans avoir à accéder à une clé secrète (déchiffrement). Les résultats des calculs sont chiffrés et peuvent être révélés uniquement par le propriétaire de la clé secrète. 

## <a name="prerequisites"></a>Prérequis

Ce guide suppose que vous disposez d’un modèle de classification d’images enregistré dans Azure Machine Learning. Si ce n’est pas le cas, enregistrez le modèle à l’aide d’un [modèle préformé](https://github.com/Azure/MachineLearningNotebooks/raw/master/tutorials/image-classification-mnist-data/sklearn_mnist_model.pkl) ou créez le vôtre en suivant le [didacticiel Effectuer l’apprentissage d’un modèle de classification d’images avec Azure Machine Learning](tutorial-train-models-with-aml.md).

## <a name="configure-local-environment"></a>Configurer l’environnement local

Dans un notebook Jupyter :

1. Importez les packages Python nécessaires pour cet exemple.

    ```python
    %matplotlib inline
    import numpy as np
    import matplotlib.pyplot as plt

    import azureml.core

    # display the core SDK version number
    print("Azure ML SDK Version: ", azureml.core.VERSION)
    ```

2. Installez la bibliothèque de chiffrement homomorphe pour une inférence sécurisée.

    > [!NOTE]
    > Le package `encrypted-inference` est actuellement en préversion.

    [`encrypted-inference`](https://pypi.org/project/encrypted-inference) est une bibliothèque qui contient des liaisons pour l’inférence chiffrée basée sur [Microsoft SEAL](https://github.com/Microsoft/SEAL).

    ```python
    !pip install encrypted-inference==0.9
    ```

## <a name="configure-the-inferencing-environment"></a>Configurer l’environnement d’inférence

Créez un environnement pour l’inférence et ajoutez le package `encrypted-inference` en tant que dépendance Conda.

```python
from azureml.core.environment import Environment
from azureml.core.conda_dependencies import CondaDependencies

# to install required packages
env = Environment('tutorial-env')
cd = CondaDependencies.create(pip_packages=['azureml-dataprep[pandas,fuse]>=1.1.14', 'azureml-defaults', 'azure-storage-blob', 'encrypted-inference==0.9'], conda_packages = ['scikit-learn==0.22.1'])

env.python.conda_dependencies = cd

# Register environment to re-use later
env.register(workspace = ws)
```

## <a name="deploy-encrypted-inferencing-web-service"></a>Déployer le service web d’inférence chiffrée

Déployez le modèle en tant que service web hébergé dans ACI.

Pour créer l’environnement approprié pour ACI, fournissez les éléments suivants :

* Un script de scoring pour montrer comment utiliser le modèle
* Un fichier de configuration pour créer l’ACI
* Un modèle formé

### <a name="create-scoring-script"></a>Créer le script de scoring

Créez le script de scoring `score.py` utilisé par le service web pour l’inférence.

Vous devez inclure deux fonctions nécessaires dans le script de scoring :

* La fonction `init()`, qui charge en général le modèle dans un objet global. Cette fonction est exécutée une seule fois lors du démarrage du conteneur Docker.
* La fonction `run(input_data)` utilise le modèle pour prédire une valeur sur la base des données d’entrée. Les entrées et les sorties de l’exécution utilisent en général JSON pour la sérialisation et la désérialisation, mais d’autres formats sont pris en charge. La fonction extrait les clés publiques basées sur le chiffrement homomorphe qui sont téléchargées par l’appelant du service.

```python
%%writefile score.py
import json
import os
import pickle
import joblib
from azure.storage.blob import BlobServiceClient, BlobClient, ContainerClient, PublicAccess
from encrypted.inference.eiserver import EIServer

def init():
    global model
    # AZUREML_MODEL_DIR is an environment variable created during deployment.
    # It is the path to the model folder (./azureml-models/$MODEL_NAME/$VERSION)
    # For multiple models, it points to the folder containing all deployed models (./azureml-models)
    model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'sklearn_mnist_model.pkl')
    model = joblib.load(model_path)

    global server
    server = EIServer(model.coef_, model.intercept_, verbose=True)

def run(raw_data):

    json_properties = json.loads(raw_data)

    key_id = json_properties['key_id']
    conn_str = json_properties['conn_str']
    container = json_properties['container']
    data = json_properties['data']

    # download the public keys from blob storage
    blob_service_client = BlobServiceClient.from_connection_string(conn_str=conn_str)
    blob_client = blob_service_client.get_blob_client(container=container, blob=key_id)
    public_keys = blob_client.download_blob().readall()

    result = {}
    # make prediction
    result = server.predict(data, public_keys)

    # you can return any data type as long as it is JSON-serializable
    return result
```

### <a name="create-configuration-file"></a>Création d’un fichier de configuration

Créez un fichier de configuration de déploiement, et spécifiez le nombre de processeurs et de gigaoctets de RAM nécessaires pour votre conteneur ACI. Bien que cela dépende de votre modèle, les valeurs par défaut de 1 cœur et de 1 gigaoctet de RAM sont généralement suffisantes pour de nombreux modèles. Si vous pensez plus tard que des valeurs supérieures sont nécessaires, vous devrez recréer l’image et redéployer le service.

```python
from azureml.core.webservice import AciWebservice

aciconfig = AciWebservice.deploy_configuration(cpu_cores=1, 
                                               memory_gb=1, 
                                               tags={"data": "MNIST",  "method" : "sklearn"}, 
                                               description='Encrypted Predict MNIST with sklearn + SEAL')
```

### <a name="deploy-to-azure-container-instances"></a>Déployer vers Azure Container Instances

Durée estimée : **environ 2 à 5 minutes**

Configurez l’image et déployez. Le code suivant effectue ces étapes :

1. Création d’un objet d’environnement contenant les dépendances nécessaires au modèle utilisant le fichier d’environnement (`myenv.yml`)
1. Création de la configuration d’inférence nécessaire au déploiement du modèle en tant que service web en utilisant :
   * Le fichier de scoring (`score.py`)
   * L’objet d’environnement créé à l’étape précédente
1. Déploiement du modèle dans le conteneur ACI.
1. Obtenir le point de terminaison HTTP du service web.

```python
%%time
from azureml.core.webservice import Webservice
from azureml.core.model import InferenceConfig
from azureml.core.environment import Environment
from azureml.core import Workspace
from azureml.core.model import Model

ws = Workspace.from_config()
model = Model(ws, 'sklearn_mnist')

myenv = Environment.get(workspace=ws, name="tutorial-env")
inference_config = InferenceConfig(entry_script="score.py", environment=myenv)

service = Model.deploy(workspace=ws,
                       name='sklearn-encrypted-mnist-svc',
                       models=[model],
                       inference_config=inference_config,
                       deployment_config=aciconfig)

service.wait_for_deployment(show_output=True)
```

Obtenir le point de terminaison HTTP du service web de scoring qui accepte les appels du client REST. Ce point de terminaison peut être partagé avec toute personne souhaitant tester le service web ou l’intégrer dans une application.

```python
print(service.scoring_uri)
```

## <a name="prepare-test-data"></a>Préparer les données de test

1. Téléchargez les données de test. Dans ce cas, elles sont enregistrées dans un répertoire appelé *data*.

    ```python
    import os
    from azureml.core import Dataset
    from azureml.opendatasets import MNIST
    
    data_folder = os.path.join(os.getcwd(), 'data')
    os.makedirs(data_folder, exist_ok=True)
    
    mnist_file_dataset = MNIST.get_file_dataset()
    mnist_file_dataset.download(data_folder, overwrite=True)
    ```

1. Chargez les données de test à partir du répertoire *data*.

    ```python
    from utils import load_data
    import os
    import glob
    
    data_folder = os.path.join(os.getcwd(), 'data')
    # note we also shrink the intensity values (X) from 0-255 to 0-1. This helps the neural network converge faster
    X_test = load_data(glob.glob(os.path.join(data_folder,"**/t10k-images-idx3-ubyte.gz"), recursive=True)[0], False) / 255.0
    y_test = load_data(glob.glob(os.path.join(data_folder,"**/t10k-labels-idx1-ubyte.gz"), recursive=True)[0], True).reshape(-1)
    ```

## <a name="make-encrypted-predictions"></a>Faire des prédictions chiffrées

Utilisez le jeu de données de test avec le modèle pour obtenir des prédictions.

Pour faire des prédictions chiffrées :

1. Créez un `EILinearRegressionClient`, un client basé sur le chiffrement homomorphe, et des clés publiques.

    ```python
    from encrypted.inference.eiclient import EILinearRegressionClient

    # Create a new Encrypted inference client and a new secret key.
    edp = EILinearRegressionClient(verbose=True)

    public_keys_blob, public_keys_data = edp.get_public_keys()
    ```

1. Chargez les clés publiques générées par le chiffrement homomorphe dans le magasin d’objets blob par défaut de l’espace de travail. Cela vous permettra de partager les clés avec le serveur d’inférence.

    ```python
    import azureml.core
    from azureml.core import Workspace, Datastore
    import os

    ws = Workspace.from_config()

    datastore = ws.get_default_datastore()
    container_name=datastore.container_name

    # Create a local file and write the keys to it
    public_keys = open(public_keys_blob, "wb")
    public_keys.write(public_keys_data)
    public_keys.close()

    # Upload the file to blob store
    datastore.upload_files([public_keys_blob])

    # Delete the local file
    os.remove(public_keys_blob)
    ```

1. Chiffrez les données de test.

    ```python
    #choose any one sample from the test data 
    sample_index = 1

    #encrypt the data
    raw_data = edp.encrypt(X_test[sample_index])

    ```

1. Utilisez l’API `run` du Kit de développement logiciel (SDK) pour appeler le service et fournir le jeu de données de test au modèle afin d’obtenir des prédictions. Nous devrons envoyer la chaîne de connexion au stockage de blobs où les clés publiques ont été chargées.

    ```python
    import json
    from azureml.core import Webservice

    service = Webservice(ws, 'sklearn-encrypted-mnist-svc')

    #pass the connection string for blob storage to give the server access to the uploaded public keys 
    conn_str_template = 'DefaultEndpointsProtocol={};AccountName={};AccountKey={};EndpointSuffix=core.windows.net'
    conn_str = conn_str_template.format(datastore.protocol, datastore.account_name, datastore.account_key)

    #build the json 
    data = json.dumps({"data": raw_data, "key_id" : public_keys_blob, "conn_str" : conn_str, "container" : container_name })
    data = bytes(data, encoding='ASCII')

    print ('Making an encrypted inference web service call ')
    eresult = service.run(input_data=data)

    print ('Received encrypted inference results')
    ```

1. Utilisez le client pour déchiffrer les résultats.

    ```python
    import numpy as np

    results = edp.decrypt(eresult)

    print ('Decrypted the results ', results)

    #Apply argmax to identify the prediction result
    prediction = np.argmax(results)

    print ( ' Prediction : ', prediction)
    print ( ' Actual Label : ', y_test[sample_index])
    ```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Supprimez le service web créé dans cet exemple :

```python
service.delete()
```

Si vous n’envisagez plus d’utiliser les ressources Azure que vous avez créées, supprimez-les. Cela vous évite d’être facturé pour des ressources inutilisées qui sont toujours en cours d’exécution. Pour en savoir plus, consultez ce guide pratique sur le [nettoyage des ressources](how-to-manage-workspace.md#clean-up-resources).
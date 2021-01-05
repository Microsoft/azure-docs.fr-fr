---
title: 'Tutoriel sur la classification d’images : Déployer des modèles'
titleSuffix: Azure Machine Learning
description: Ce tutoriel montre comment utiliser Azure Machine Learning pour déployer un modèle de classification d’images avec scikit-learn dans un notebook Jupyter Python.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: sdgilley
ms.author: sgilley
ms.date: 03/18/2020
ms.custom: seodec18
ms.openlocfilehash: ee6529f36c302cba1dc7b5a4b53aaf96ce58e4a5
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94532781"
---
# <a name="tutorial-deploy-an-image-classification-model-in-azure-container-instances"></a>Tutoriel : Déployer un modèle de classification d’images dans Azure Container Instances


Ce tutoriel est le **deuxième d’une série de deux**. Dans le [tutoriel précédent](tutorial-train-models-with-aml.md), vous avez entraîné des modèles Machine Learning, puis vous avez inscrit un modèle dans votre espace de travail sur le cloud.  Vous êtes maintenant prêt à déployer le modèle en tant que service web. Un service web est une image, dans le cas présent une image Docker. Elle encapsule la logique de scoring et le modèle proprement dit. 

Dans cette partie du tutoriel, vous utilisez Azure Machine Learning pour effectuer les tâches suivantes :

> [!div class="checklist"]
> * Configurer votre environnement de test
> * Récupérer le modèle à partir de votre espace de travail
> * Déployer le modèle sur Azure Container Instances
> * Tester le modèle déployé

Container Instances est une excellente solution pour comprendre et tester le workflow. Pour les déploiements de production évolutifs, envisagez d’utiliser Azure Kubernetes Service. Pour plus d’informations, consultez [Comment et où déployer](how-to-deploy-and-where.md).

>[!NOTE]
> Le code présenté dans cet article a été testé avec le kit SDK Azure Machine Learning version 1.0.83.

## <a name="prerequisites"></a>Prérequis

Pour exécuter le notebook, effectuez tout d’abord l’entraînement du modèle dans le [Tutoriel (partie 1) : Entraîner un modèle de classification d’images](tutorial-train-models-with-aml.md).   Ouvrez ensuite le notebook *img-classification-part2-deploy.ipynb* dans le dossier *tutorials/image-classification-mnist-data* cloné.

Vous trouverez également ce tutoriel sur [GitHub](https://github.com/Azure/MachineLearningNotebooks/tree/master/tutorials) si vous souhaitez l’utiliser dans votre propre [environnement local](how-to-configure-environment.md#local).  Veillez à installer `matplotlib` et `scikit-learn` dans votre environnement. 

> [!Important]
> Le reste de cet article contient le même contenu que ce que vous voyez dans le notebook.  
>
> Basculez maintenant vers le notebook Jupyter si vous voulez lire le code à mesure que vous l’exécutez.
> Pour exécuter une seule cellule de code dans un notebook, cliquez sur celle-ci et appuyez sur **Maj + Entrée**. Sinon, exécutez l’intégralité du notebook en choisissant **Run all** (Tout exécuter) dans la barre d’outils supérieure.

## <a name="set-up-the-environment"></a><a name="start"></a>Configurer l’environnement

Commencez par configurer un environnement de test.

### <a name="import-packages"></a>Importer des packages

Importer les packages Python nécessaires pour ce tutoriel.


```python
%matplotlib inline
import numpy as np
import matplotlib.pyplot as plt
 
import azureml.core

# Display the core SDK version number
print("Azure ML SDK Version: ", azureml.core.VERSION)
```

## <a name="deploy-as-web-service"></a>Déployer en tant que service web

Déployez le modèle en tant que service web hébergé dans ACI. 

Pour créer l’environnement approprié pour ACI, fournissez les éléments suivants :
* Un script de scoring pour montrer comment utiliser le modèle
* Un fichier de configuration pour créer l’ACI
* Le modèle que vous avez précédemment entraîné

### <a name="create-scoring-script"></a>Créer le script de scoring

Créez le script de scoring, nommé score.py, utilisé par l’appel du service web pour montrer comment utiliser le modèle.

Vous devez inclure deux fonctions nécessaires dans le script de scoring :
* La fonction `init()`, qui charge en général le modèle dans un objet global. Cette fonction est exécutée une seule fois lors du démarrage du conteneur Docker. 

* La fonction `run(input_data)` utilise le modèle pour prédire une valeur sur la base des données d’entrée. Les entrées et les sorties de l’exécution utilisent en général JSON pour la sérialisation et la désérialisation, mais d’autres formats sont pris en charge.

```python
%%writefile score.py
import json
import numpy as np
import os
import pickle
import joblib

def init():
    global model
    # AZUREML_MODEL_DIR is an environment variable created during deployment.
    # It is the path to the model folder (./azureml-models/$MODEL_NAME/$VERSION)
    # For multiple models, it points to the folder containing all deployed models (./azureml-models)
    model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'sklearn_mnist_model.pkl')
    model = joblib.load(model_path)

def run(raw_data):
    data = np.array(json.loads(raw_data)['data'])
    # make prediction
    y_hat = model.predict(data)
    # you can return any data type as long as it is JSON-serializable
    return y_hat.tolist()
```

### <a name="create-configuration-file"></a>Création d’un fichier de configuration

Créez un fichier de configuration de déploiement, et spécifiez le nombre de processeurs et de gigaoctets de RAM nécessaires pour votre conteneur ACI. Bien que cela dépende de votre modèle, les valeurs par défaut de 1 cœur et de 1 gigaoctet de RAM sont généralement suffisantes pour de nombreux modèles. Si vous pensez plus tard que des valeurs supérieures sont nécessaires, vous devrez recréer l’image et redéployer le service.


```python
from azureml.core.webservice import AciWebservice

aciconfig = AciWebservice.deploy_configuration(cpu_cores=1, 
                                               memory_gb=1, 
                                               tags={"data": "MNIST",  "method" : "sklearn"}, 
                                               description='Predict MNIST with sklearn')
```

### <a name="deploy-in-aci"></a>Déployer dans ACI
Durée estimée : **environ 2 à 5 minutes**

Configurez l’image et déployez. Le code suivant effectue ces étapes :

1. Création d’un objet d’environnement contenant les dépendances nécessaires au modèle utilisant l’environnement (`tutorial-env`) enregistré pendant l’entraînement.
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


myenv = Environment.get(workspace=ws, name="tutorial-env", version="1")
inference_config = InferenceConfig(entry_script="score.py", environment=myenv)

service = Model.deploy(workspace=ws, 
                       name='sklearn-mnist-svc3', 
                       models=[model], 
                       inference_config=inference_config, 
                       deployment_config=aciconfig)

service.wait_for_deployment(show_output=True)
```

Obtenir le point de terminaison HTTP du service web de scoring qui accepte les appels du client REST. Ce point de terminaison peut être partagé avec toute personne souhaitant tester le service web ou l’intégrer dans une application.


```python
print(service.scoring_uri)
```

## <a name="test-the-model"></a>Tester le modèle


### <a name="download-test-data"></a>Télécharger les données de test
Téléchargez les données de test dans le répertoire **./data/** .


```python
import os
from azureml.core import Dataset
from azureml.opendatasets import MNIST

data_folder = os.path.join(os.getcwd(), 'data')
os.makedirs(data_folder, exist_ok=True)

mnist_file_dataset = MNIST.get_file_dataset()
mnist_file_dataset.download(data_folder, overwrite=True)
```

### <a name="load-test-data"></a>Charger les données de test

Chargez les données de test à partir du répertoire **./data/** créé au cours du tutoriel consacré à l’entraînement.


```python
from utils import load_data
import os
import glob

data_folder = os.path.join(os.getcwd(), 'data')
# note we also shrink the intensity values (X) from 0-255 to 0-1. This helps the neural network converge faster
X_test = load_data(glob.glob(os.path.join(data_folder,"**/t10k-images-idx3-ubyte.gz"), recursive=True)[0], False) / 255.0
y_test = load_data(glob.glob(os.path.join(data_folder,"**/t10k-labels-idx1-ubyte.gz"), recursive=True)[0], True).reshape(-1)
```

### <a name="predict-test-data"></a>Effectuant des prédictions avec les données de test

Alimentez le jeu de données de test du modèle pour obtenir des prédictions.


Le code suivant effectue ces étapes :
1. Envoyer les données sous forme de tableau JSON au service web hébergé dans ACI. 

1. Utiliser l’API `run` du SDK pour appeler le service. Vous pouvez également effectuer des appels directs avec n’importe quel outil HTTP, comme curl.


```python
import json
test = json.dumps({"data": X_test.tolist()})
test = bytes(test, encoding='utf8')
y_hat = service.run(input_data=test)
```

###  <a name="examine-the-confusion-matrix"></a>Examiner la matrice de confusion

Générez une matrice de confusion pour voir combien d’exemples du jeu de test sont classifiés correctement. Notez la valeur indiquant la mauvaise classification pour les prédictions incorrectes.


```python
from sklearn.metrics import confusion_matrix

conf_mx = confusion_matrix(y_test, y_hat)
print(conf_mx)
print('Overall accuracy:', np.average(y_hat == y_test))
```

La sortie montre la matrice de confusion :

```output
[[ 960    0    1    2    1    5    6    3    1    1]
 [   0 1112    3    1    0    1    5    1   12    0]
 [   9    8  920   20   10    4   10   11   37    3]
 [   4    0   17  921    2   21    4   12   20    9]
 [   1    2    5    3  915    0   10    2    6   38]
 [  10    2    0   41   10  770   17    7   28    7]
 [   9    3    7    2    6   20  907    1    3    0]
 [   2    7   22    5    8    1    1  950    5   27]
 [  10   15    5   21   15   27    7   11  851   12]
 [   7    8    2   13   32   13    0   24   12  898]]
Overall accuracy: 0.9204
```

Utilisez `matplotlib` pour afficher la matrice de confusion sous forme de graphe. Dans ce graphe, l’axe X représente les valeurs réelles, et l’axe Y représente les valeurs prédites. La couleur dans chaque grille représente le taux d’erreur. Plus la couleur est claire, plus le taux d’erreurs est élevé. Par exemple, beaucoup de 5 sont incorrectement classifiés en 3. Par conséquent, vous voyez une grille claire en (5,3).

```python
# normalize the diagonal cells so that they don't overpower the rest of the cells when visualized
row_sums = conf_mx.sum(axis=1, keepdims=True)
norm_conf_mx = conf_mx / row_sums
np.fill_diagonal(norm_conf_mx, 0)

fig = plt.figure(figsize=(8, 5))
ax = fig.add_subplot(111)
cax = ax.matshow(norm_conf_mx, cmap=plt.cm.bone)
ticks = np.arange(0, 10, 1)
ax.set_xticks(ticks)
ax.set_yticks(ticks)
ax.set_xticklabels(ticks)
ax.set_yticklabels(ticks)
fig.colorbar(cax)
plt.ylabel('true labels', fontsize=14)
plt.xlabel('predicted values', fontsize=14)
plt.savefig('conf.png')
plt.show()
```

![Schéma montrant la matrice de confusion](./media/tutorial-deploy-models-with-aml/confusion.png)


## <a name="show-predictions"></a>Afficher les prédictions

Testez le modèle déployé avec un échantillon aléatoire de 30 images tirées des données de test.  


1. Affichez les prédictions retournées et tracez-les avec les images d’entrée. Une police rouge et une image inverse (blanc sur fond noir) sont utilisées pour mettre en évidence les exemples mal classifiés. 

 Comme la précision du modèle est élevée, il peut être nécessaire d’exécuter le code suivant plusieurs fois avant de voir un exemple mal classifié.



```python
import json

# find 30 random samples from test set
n = 30
sample_indices = np.random.permutation(X_test.shape[0])[0:n]

test_samples = json.dumps({"data": X_test[sample_indices].tolist()})
test_samples = bytes(test_samples, encoding='utf8')

# predict using the deployed model
result = service.run(input_data=test_samples)

# compare actual value vs. the predicted values:
i = 0
plt.figure(figsize = (20, 1))

for s in sample_indices:
    plt.subplot(1, n, i + 1)
    plt.axhline('')
    plt.axvline('')
    
    # use different color for misclassified sample
    font_color = 'red' if y_test[s] != result[i] else 'black'
    clr_map = plt.cm.gray if y_test[s] != result[i] else plt.cm.Greys
    
    plt.text(x=10, y=-10, s=result[i], fontsize=18, color=font_color)
    plt.imshow(X_test[s].reshape(28, 28), cmap=clr_map)
    
    i = i + 1
plt.show()
```

Vous pouvez également envoyer une demande HTTP directe pour tester le service web.


```python
import requests

# send a random row from the test set to score
random_index = np.random.randint(0, len(X_test)-1)
input_data = "{\"data\": [" + str(list(X_test[random_index])) + "]}"

headers = {'Content-Type': 'application/json'}

# for AKS deployment you'd need to the service key in the header as well
# api_key = service.get_key()
# headers = {'Content-Type':'application/json',  'Authorization':('Bearer '+ api_key)} 

resp = requests.post(service.scoring_uri, input_data, headers=headers)

print("POST to url", service.scoring_uri)
#print("input data:", input_data)
print("label:", y_test[random_index])
print("prediction:", resp.text)
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour conserver le groupe de ressources et l’espace de travail pour d’autres tutoriels et d’autres explorations, vous pouvez supprimer uniquement le déploiement Azure Container Instances avec cet appel d’API :

```python
service.delete()
```

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]


## <a name="next-steps"></a>Étapes suivantes

+ Découvrez toutes les [options de déploiement d’Azure Machine Learning](how-to-deploy-and-where.md).
+ Découvrez comment [créer des clients pour le service web](how-to-consume-web-service.md).
+  [Effectuez des prédictions sur de grandes quantités de données](./tutorial-pipeline-batch-scoring-classification.md) de façon asynchrone.
+ Supervisez vos modèles Azure Machine Learning avec [Application Insights](how-to-enable-app-insights.md).
+ Suivez le tutoriel [Sélection automatique d’un algorithme](tutorial-auto-train-models.md).
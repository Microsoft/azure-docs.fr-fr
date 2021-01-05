---
title: Déployer des modèles ML sur des FPGA
titleSuffix: Azure Machine Learning
description: En savoir plus sur les réseaux de portes programmables in situ (FGPA, field-programmable gate array). Vous pouvez déployer un service web sur un FPGA avec Azure Machine Learning pour que la latence de l’inférence soit très faible.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: larryfr
ms.author: jordane
author: jpe316
ms.date: 09/24/2020
ms.topic: conceptual
ms.custom: how-to, contperf-fy21q2, devx-track-python, deploy
ms.openlocfilehash: 9e43291325510f92f2e5fd6c07cdb9257fdede9d
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033066"
---
# <a name="deploy-ml-models-to-field-programmable-gate-arrays-fpgas-with-azure-machine-learning"></a>Déployer des modèles ML sur des réseaux de portes programmables in situ (FGPA, field-programmable gate array) avec Azure Machine Learning 

Dans cet article, vous allez en apprendre davantage sur les FPGA et le déploiement de vos modèles ML sur un FPGA Azure à l’aide du [package Python de modèles accélérés matériellement](/python/api/azureml-accel-models/azureml.accel?preserve-view=true&view=azure-ml-py) d’[Azure Machine Learning](overview-what-is-azure-ml.md).

## <a name="what-are-fpgas"></a>Présentation des FPGA
Les FPGA contiennent un tableau de blocs logiques programmables et une hiérarchie d’interconnexions reconfigurables. Les interconnexions permettent la configuration de ces blocs de différentes façons après leur fabrication. Par rapport aux autres circuits intégrés, les FPGA offrent une combinaison de programmabilité et de performances. 

Les FPGA permettent d’obtenir une latence faible pour les requêtes d’inférence en temps réel (ou notation de modèle). Les requêtes asynchrones (traitement par lot) ne sont pas nécessaires. Le traitement par lots peut entraîner une latence, car davantage de données doivent être traitées. Les implémentations des unités de traitement neuronal ne nécessitent pas de traitement par lots. Ainsi, la latence peut être plusieurs fois inférieure à celle des UC et des GPU.

Vous pouvez reconfigurer les FGPA pour différents types de modèles Machine Learning. Cette souplesse permet d’accélérer les applications basées sur la précision numérique la plus optimale et le modèle de mémoire utilisé. Étant donné que les FPGA sont reconfigurables, vous pouvez suivre l’évolution des besoins des algorithmes d’intelligence artificielle.

![Diagramme de comparaison des FPGA Azure Machine Learning](./media/how-to-deploy-fpga-web-service/azure-machine-learning-fpga-comparison.png)

|Processeur| Abréviation |Description|
|---|:-------:|------|
|Application Specific Integrated Circuit (Circuits intégrés spécifiques à une application)|ASIC|Les circuits personnalisés, comme les TPU (Tensor Processor Units) de Google, offrent la meilleure efficacité. Ils ne peuvent pas être reconfigurés selon vos besoins.|
|Field-Programmable Gate Array|FPGA|Les FPGA, comme ceux disponibles sur Azure, offrent des performances approchant celles des ASIC. Ils sont flexibles et reconfigurables, afin de pouvoir implémenter de nouvelles logiques.|
|GPU (Graphics Processing Unit, Unité de traitement graphique)|GPU|Un choix courant pour les calculs en intelligence artificielle. Les GPU offrent des fonctionnalités de traitement parallèle qui les rendent plus rapides que les UC pour le rendu des images.|
|UC (Unité centrale)|UC|Processeurs à usage général dont les performances ne sont pas idéales pour le traitement des images et de la vidéo.|

## <a name="fpga-support-in-azure"></a>Prise en charge des FPGA dans Azure

Microsoft Azure constitue le plus important investissement cloud au monde en matière de FPGA. Microsoft utilise des FPGA pour l’évaluation des réseaux neuronaux profonds, le classement des recherches Bing et l’accélération réseau définie par logiciel pour réduire la latence, tout en libérant les UC pour d’autres tâches.

Les FPGA sur Azure sont basés sur les FPGA d’Intel, que les scientifiques des données et les développeurs utilisent pour accélérer les calculs d’intelligence artificielle en temps réel. Cette architecture compatible avec les FPGA offre performances, flexibilité et scalabilité, et est disponible sur Azure.

Les FPGA Azure ne sont pas intégrés à Azure Machine Learning. Azure peut mettre en parallèle des réseaux neuronaux profonds préformés sur des FPGA pour effectuer un scale-out de votre service. Les réseaux neuronaux profonds peuvent être préentraînés, en tant que caractériseurs profonds pour l’apprentissage du transfert, ou optimisés avec des pondérations mises à jour.

|Scénarios et configurations sur Azure|Modèles de réseaux neuronaux profonds pris en charge|Support régional|
|--------------------------|--------------------|----------------|
|+ Scénarios de classification et de reconnaissance d’images<br/>+ Déploiement de TensorFlow (requiert Tensorflow 1.x)<br/>+ Matériel FPGA d’Intel|- ResNet-50<br/>- ResNet-152<br/>- DenseNet-121<br/>- VGG-16<br/>- SSD-VGG|- USA Est<br/>- Asie Sud-Est<br/> - Europe Ouest<br/>- USA Ouest 2|

Pour optimiser la latence et le débit, votre client qui envoie des données au modèle FGPA doit se trouver dans une des régions ci-dessus (celle dans laquelle vous avez déployé le modèle).

La **gamme de machines virtuelles Azure PBS** contient des FPGA Intel Arria 10. Elle apparaît comme « Processeurs virtuels de gamme PBS Standard » lorsque vous vérifiez votre allocation de quota Azure. La machine virtuelle PB6 dispose de six processeurs virtuels et d’un FPGA. La machine virtuelle PB6 est automatiquement approvisionnée par Azure Machine Learning lors du déploiement d’un modèle sur un FPGA. Elle est uniquement utilisée avec Azure Machine Learning, et ne peut exécuter de flux binaires arbitraires. Par exemple, vous ne pouvez pas flasher le FPGA avec des flux binaires pour effectuer du chiffrement, de l’encodage, etc.

## <a name="deploy-models-on-fpgas"></a>Déployer des modèles sur des FPGA

Vous pouvez déployer un modèle en tant que service web sur FGPA avec des [Modèles accélérés matériellement Azure Machine Learning](/python/api/azureml-accel-models/azureml.accel?preserve-view=true&view=azure-ml-py). L’utilisation de FGPA assure une inférence à très faible latence, même avec une taille de lot unique. 

Dans cet exemple, vous créez un graphe TensorFlow pour prétraiter l’image en entrée, en faire un caractériseur à l’aide de ResNet-50 sur un FPGA et exécuter les fonctionnalités via un classifieur formé sur le jeu de données ImageNet. Ensuite, le modèle est déployé sur un cluster AKS.

### <a name="prerequisites"></a>Prérequis

- Un abonnement Azure. Le cas échéant, créez un compte [avec option de paiement à l’utilisation](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go) (les comptes Azure gratuits ne sont pas éligibles pour le quota de FPGA).

- Un espace de travail Azure Machine Learning doté du Kit de développement logiciel (SDK) Azure Machine Learning pour Python, comme décrit dans [Créer un espace de travail](how-to-manage-workspace.md).
 
- Le package de modèles accélérés matériellement : `pip install --upgrade azureml-accel-models[cpu]`.    
    
- [Azure CLI](/cli/azure/install-azure-cli?preserve-view=true&view=azure-cli-latest)

- Quota FPGA. Soumettez une [demande de quota](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR2nac9-PZhBDnNSV2ITz0LNUN0U5S0hXRkNITk85QURTWk9ZUUFUWkkyTC4u), ou exécutez cette commande CLI pour vérifier le quota : 

    ```azurecli-interactive
    az vm list-usage --location "eastus" -o table --query "[?localName=='Standard PBS Family vCPUs']"
    ```

   Vérifiez que vous avez au moins six processeurs virtuels sous la valeur __CurrentValue__ retournée.  

### <a name="define-the-tensorflow-model"></a>Définir le modèle TensorFlow

Commencez par utiliser le [Kit de développement logiciel (SDK) Azure Machine Learning pour Python](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py) afin de créer une définition de service. Une définition de service est un fichier décrivant un pipeline de graphes (entrée, caractériseur et classifieur) basé sur TensorFlow. La commande de déploiement compresse la définition et les graphes dans un fichier ZIP, puis charge ce fichier dans Stockage Blob Azure. Le réseau neuronal profond est déjà déployé pour s’exécuter sur le FPGA.

1. Charger un espace de travail Azure Machine Learning

   ```python
   import os
   import tensorflow as tf
   
   from azureml.core import Workspace
  
   ws = Workspace.from_config()
   print(ws.name, ws.resource_group, ws.location, ws.subscription_id, sep='\n')
   ```

1. Prétraitez l’image. L’entrée du service web est une image JPEG.  La première étape consiste à décoder l’image JPEG et la prétraiter.  Les images JPEG sont traitées comme des chaînes, ce qui donne des tenseurs qui constituent l’entrée du modèle ResNet 50.

   ```python
   # Input images as a two-dimensional tensor containing an arbitrary number of images represented a strings
   import azureml.accel.models.utils as utils
   tf.reset_default_graph()
   
   in_images = tf.placeholder(tf.string)
   image_tensors = utils.preprocess_array(in_images)
   print(image_tensors.shape)
   ```

1. Chargez un caractériseur. Initialisez le modèle et téléchargez un point de contrôle TensorFlow de la version quantifiée de ResNet50 à utiliser comme caractériseur.  Remplacez « QuantizedResnet50 » dans l’extrait de code pour importer d’autres réseaux neuronaux profonds :

   - QuantizedResnet152
   - QuantizedVgg16
   - Densenet121

   ```python
   from azureml.accel.models import QuantizedResnet50
   save_path = os.path.expanduser('~/models')
   model_graph = QuantizedResnet50(save_path, is_frozen=True)
   feature_tensor = model_graph.import_graph_def(image_tensors)
   print(model_graph.version)
   print(feature_tensor.name)
   print(feature_tensor.shape)
   ```

1. Ajoutez un classifieur. Ce classifieur a été formé sur le jeu de données ImageNet.

   ```python
   classifier_output = model_graph.get_default_classifier(feature_tensor)
   print(classifier_output)
   ```

1. Enregistrez le modèle. Maintenant le préprocesseur, le caractériseur ResNet 50 et le classifieur ont été chargés, enregistrez le graph et les variables associées en tant que modèle.

   ```python
   model_name = "resnet50"
   model_save_path = os.path.join(save_path, model_name)
   print("Saving model in {}".format(model_save_path))
   
   with tf.Session() as sess:
       model_graph.restore_weights(sess)
       tf.saved_model.simple_save(sess, model_save_path,
                                  inputs={'images': in_images},
                                  outputs={'output_alias': classifier_output})
   ```

1. Enregistrez les tenseurs d’entrée et de sortie, **vous les utiliserez pour toute demande d’inférence ou de conversion de modèle**. 

   ```python
   input_tensors = in_images.name
   output_tensors = classifier_output.name

   print(input_tensors)
   print(output_tensors)
   ```

   Les modèles suivants sont disponibles, répertoriés avec leurs tenseurs de sortie de classifieur pour l’inférence si vous avez utilisé le classifieur par défaut.

   + Resnet50, QuantizedResnet50
     ```python
     output_tensors = "classifier_1/resnet_v1_50/predictions/Softmax:0"
     ```
   + Resnet152, QuantizedResnet152
     ```python
     output_tensors = "classifier/resnet_v1_152/predictions/Softmax:0"
     ```
   + Densenet121, QuantizedDensenet121
     ```python
     output_tensors = "classifier/densenet121/predictions/Softmax:0"
     ```
   + Vgg16, QuantizedVgg16
     ```python
     output_tensors = "classifier/vgg_16/fc8/squeezed:0"
     ```
   + SsdVgg, QuantizedSsdVgg
     ```python
     output_tensors = ['ssd_300_vgg/block4_box/Reshape_1:0', 'ssd_300_vgg/block7_box/Reshape_1:0', 'ssd_300_vgg/block8_box/Reshape_1:0', 'ssd_300_vgg/block9_box/Reshape_1:0', 'ssd_300_vgg/block10_box/Reshape_1:0', 'ssd_300_vgg/block11_box/Reshape_1:0', 'ssd_300_vgg/block4_box/Reshape:0', 'ssd_300_vgg/block7_box/Reshape:0', 'ssd_300_vgg/block8_box/Reshape:0', 'ssd_300_vgg/block9_box/Reshape:0', 'ssd_300_vgg/block10_box/Reshape:0', 'ssd_300_vgg/block11_box/Reshape:0']
     ```

### <a name="convert-the-model-to-the-open-neural-network-exchange-format-onnx"></a>Convertir le modèle au format Open Neural Network Exchange (ONNX)

Avant de pouvoir déployer le modèle sur des FPGA, convertissez-le au format [ONNX](https://onnx.ai/).

1. [Inscrivez](concept-model-management-and-deployment.md) le modèle en utilisant le SDK avec le fichier ZIP dans Stockage Blob Azure. L’ajout de balises et autres modèles vous aide à suivre vos modèles formés.

   ```python
   from azureml.core.model import Model

   registered_model = Model.register(workspace=ws,
                                     model_path=model_save_path,
                                     model_name=model_name)

   print("Successfully registered: ", registered_model.name,
         registered_model.description, registered_model.version, sep='\t')
   ```

   Si vous avez déjà inscrit un modèle et que vous souhaitez le charger, vous pouvez le récupérer.

   ```python
   from azureml.core.model import Model
   model_name = "resnet50"
   # By default, the latest version is retrieved. You can specify the version, i.e. version=1
   registered_model = Model(ws, name="resnet50")
   print(registered_model.name, registered_model.description,
         registered_model.version, sep='\t')
   ```

1. Convertissez le graphe TensorFlow au format ONNX.  Vous devez renseigner les noms des tenseurs d’entrée et de sortie afin que votre client puisse les utiliser lorsque vous consommez le service web.

   ```python
   from azureml.accel import AccelOnnxConverter

   convert_request = AccelOnnxConverter.convert_tf_model(
       ws, registered_model, input_tensors, output_tensors)

   # If it fails, you can run wait_for_completion again with show_output=True.
   convert_request.wait_for_completion(show_output=False)

   # If the above call succeeded, get the converted model
   converted_model = convert_request.result
   print("\nSuccessfully converted: ", converted_model.name, converted_model.url, converted_model.version,
         converted_model.id, converted_model.created_time, '\n')
   ```

### <a name="containerize-and-deploy-the-model"></a>Conteneuriser et déployer le modèle

Créez ensuite une image Docker à partir du modèle converti et de toutes les dépendances.  Cette image Docker peut ensuite être déployée et instanciée.  Les cibles de déploiement prises en charge comprennent Azure Kubernetes Service (AKS) dans le cloud ou un périphérique tel qu’[Azure Data Box Edge](../databox-online/azure-stack-edge-overview.md).  Vous pouvez aussi ajouter des balises et des descriptions à votre image Docker inscrite.

   ```python
   from azureml.core.image import Image
   from azureml.accel import AccelContainerImage

   image_config = AccelContainerImage.image_configuration()
   # Image name must be lowercase
   image_name = "{}-image".format(model_name)

   image = Image.create(name=image_name,
                        models=[converted_model],
                        image_config=image_config,
                        workspace=ws)
   image.wait_for_creation(show_output=False)
   ```

   Répertoriez les images par balise et obtenez des journaux détaillés en cas de débogage.

   ```python
   for i in Image.list(workspace=ws):
       print('{}(v.{} [{}]) stored at {} with build log {}'.format(
           i.name, i.version, i.creation_state, i.image_location, i.image_build_log_uri))
   ```

#### <a name="deploy-to-an-azure-kubernetes-service-cluster"></a>Déployer sur un cluster Azure Kubernetes Service

1. Pour déployer votre modèle en tant que service web de production à grande échelle, utilisez AKS. Vous pouvez en créer un en utilisant le Kit de développement logiciel (SDK) Azure Machine Learning, l’interface CLI ou [Azure Machine Learning Studio](https://ml.azure.com).

    ```python
    from azureml.core.compute import AksCompute, ComputeTarget
    
    # Specify the Standard_PB6s Azure VM and location. Values for location may be "eastus", "southeastasia", "westeurope", or "westus2". If no value is specified, the default is "eastus".
    prov_config = AksCompute.provisioning_configuration(vm_size = "Standard_PB6s",
                                                        agent_count = 1,
                                                        location = "eastus")
    
    aks_name = 'my-aks-cluster'
    # Create the cluster
    aks_target = ComputeTarget.create(workspace=ws,
                                      name=aks_name,
                                      provisioning_configuration=prov_config)
    ```

    Le déploiement AKS peut prendre environ 15 minutes.  Vérifiez que le déploiement a réussi.

    ```python
    aks_target.wait_for_completion(show_output=True)
    print(aks_target.provisioning_state)
    print(aks_target.provisioning_errors)
    ```

1. Déployez le conteneur dans le cluster AKS.

    ```python
    from azureml.core.webservice import Webservice, AksWebservice
    
    # For this deployment, set the web service configuration without enabling auto-scaling or authentication for testing
    aks_config = AksWebservice.deploy_configuration(autoscale_enabled=False,
                                                    num_replicas=1,
                                                    auth_enabled=False)
    
    aks_service_name = 'my-aks-service'
    
    aks_service = Webservice.deploy_from_image(workspace=ws,
                                               name=aks_service_name,
                                               image=image,
                                               deployment_config=aks_config,
                                               deployment_target=aks_target)
    aks_service.wait_for_deployment(show_output=True)
    ```

#### <a name="deploy-to-a-local-edge-server"></a>Déployer dans un serveur edge local

Tous les [appareils Azure Data Box Edge](../databox-online/azure-stack-edge-overview.md
) contiennent un FPGA pour l’exécution du modèle.  Un seul modèle peut être exécuté à la fois sur le FGPA.  Pour exécuter un modèle différent, déployez simplement un nouveau conteneur. Vous trouverez des instructions et des exemples de code dans [cet exemple Azure](https://github.com/Azure-Samples/aml-hardware-accelerated-models).

### <a name="consume-the-deployed-model"></a>Utiliser le modèle déployé

Pour finir, utilisez l’exemple de client pour appeler l’image Docker afin d’obtenir des prédictions du modèle.  L’exemple de code client est disponible :
- [Python](https://github.com/Azure/aml-real-time-ai/blob/master/pythonlib/amlrealtimeai/client.py)
- [C#](https://github.com/Azure/aml-real-time-ai/blob/master/sample-clients/csharp)

L’image Docker prend en charge gRPC et l’API de prédiction TensorFlow Serving.

Vous pouvez également télécharger un exemple de client pour TensorFlow Serving.

```python
# Using the grpc client in Azure ML Accelerated Models SDK package
from azureml.accel import PredictionClient

address = aks_service.scoring_uri
ssl_enabled = address.startswith("https")
address = address[address.find('/')+2:].strip('/')
port = 443 if ssl_enabled else 80

# Initialize Azure ML Accelerated Models client
client = PredictionClient(address=address,
                          port=port,
                          use_ssl=ssl_enabled,
                          service_name=aks_service.name)
```

Comme ce classifieur a été formé sur un ensemble de données [ImageNet](http://www.image-net.org/), mappez les classes à des étiquettes pouvant être lues par des humains.

```python
import requests
classes_entries = requests.get(
    "https://raw.githubusercontent.com/Lasagne/Recipes/master/examples/resnet50/imagenet_classes.txt").text.splitlines()

# Score image with input and output tensor names
results = client.score_file(path="./snowleopardgaze.jpg",
                            input_name=input_tensors,
                            outputs=output_tensors)

# map results [class_id] => [confidence]
results = enumerate(results)
# sort results by confidence
sorted_results = sorted(results, key=lambda x: x[1], reverse=True)
# print top 5 results
for top in sorted_results[:5]:
    print(classes_entries[top[0]], 'confidence:', top[1])
```

### <a name="clean-up-resources"></a>Nettoyer les ressources

Pour éviter les coûts inutiles, nettoyez vos ressources **dans cet ordre** : service web, puis l’image, et enfin le modèle.

```python
aks_service.delete()
aks_target.delete()
image.delete()
registered_model.delete()
converted_model.delete()
```

## <a name="next-steps"></a>Étapes suivantes

+ Découvrez comment [sécuriser vos services web](how-to-secure-web-service.md).

+ En savoir plus sur FPGA ainsi que sur les [tarifs et les coûts d’Azure Machine Learning](https://azure.microsoft.com/pricing/details/machine-learning/).

+ [Matériel Hyperscale : ML à grande échelle sur Azure + FPGA : Build 2018 (vidéo)](https://channel9.msdn.com/events/Build/2018/BRK3202)

+ [Cloud configurable basé sur les FPGA de Microsoft (vidéo)](https://channel9.msdn.com/Events/Build/2017/B8063)

+ [Project Brainwave pour l’IA en temps réel](https://www.microsoft.com/research/project/project-brainwave/)

+ [Système d’inspection optique automatisé](https://blogs.microsoft.com/ai/build-2018-project-brainwave/)

---
title: 'Exécuter un script Python : sur le module Modifier les métadonnées'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Exécuter un script Python dans le concepteur Azure Machine Learning pour exécuter du code Python.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
ms.custom: devx-track-python
author: likebupt
ms.author: keli19
ms.date: 12/02/2020
ms.openlocfilehash: 360f0ce60a35bc96c6dd8e46d636f07124d01255
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96511914"
---
# <a name="execute-python-script-module"></a>Module Exécuter un script Python

Cet article décrit le module Exécuter un script Python dans le concepteur Azure Machine Learning.

Utilisez ce module pour exécuter du code Python. Pour plus d’informations sur les principes de conception et d’architecture de Python, consultez [Exécuter du code python dans le concepteur Azure Machine Learning](../how-to-designer-python.md).

Python vous permet d’exécuter des tâches qui ne sont pas prises en charge par les modules existants, telles que :

+ Visualisation des données à l’aide de `matplotlib`.
+ Utilisation des bibliothèques Python pour énumérer les jeux de données et les modèles de votre espace de travail.
+ Lecture, chargement et manipulation de données à partir de sources que le module [Importer des données](./import-data.md) ne prend pas en charge.
+ Exécution de votre propre code de deep learning. 

## <a name="supported-python-packages"></a>Packages Python pris en charge

Azure Machine Learning utilise la distribution Anaconda de Python, qui inclut de nombreux utilitaires courants pour le traitement des données. La version d’Anaconda est mise à jour automatiquement. La version actuelle est :
 -  distribution Anaconda 4.5+ pour Python 3.6 

Pour obtenir la liste complète, consultez la section [Packages Python préinstallés](#preinstalled-python-packages).

Pour installer des packages qui ne figurent pas dans la liste des packages préinstallés (par exemple, *scikit-misc*), ajoutez le code suivant à votre script : 

```python
import os
os.system(f"pip install scikit-misc")
```

Utilisez le code suivant pour installer des packages pour de meilleures performances, en particulier pour l’inférence :
```python
import importlib.util
package_name = 'scikit-misc'
spec = importlib.util.find_spec(package_name)
if spec is None:
    import os
    os.system(f"pip install scikit-misc")
```

> [!NOTE]
> Si votre pipeline contient plusieurs modules Exécuter un script Python qui nécessitent des packages qui ne figurent pas dans la liste des packages préinstallés, installez les packages dans chaque module.

> [!WARNING]
> Le module Exécuter un script Python ne prend pas en charge l’installation de packages qui dépendent de bibliothèques natives supplémentaires avec une commande telle que « apt-obten », par exemple, Java, PyODBC, etc. Cela est dû au fait que ce module est exécuté dans un environnement simple avec uniquement Python préinstallé et une autorisation non administrateur.  

## <a name="access-to-registered-datasets"></a>Accès aux jeux de données inscrits

Vous pouvez vous référer à l’exemple de code suivant pour accéder aux [jeux de données inscrits](../how-to-create-register-datasets.md) dans votre espace de travail :

```Python
def azureml_main(dataframe1 = None, dataframe2 = None):

    # Execution logic goes here
    print(f'Input pandas.DataFrame #1: {dataframe1}')
    from azureml.core import Run
    run = Run.get_context(allow_offline=True)
    ws = run.experiment.workspace

    from azureml.core import Dataset
    dataset = Dataset.get_by_name(ws, name='test-register-tabular-in-designer')
    dataframe1 = dataset.to_pandas_dataframe()
     
    # If a zip file is connected to the third input port,
    # it is unzipped under "./Script Bundle". This directory is added
    # to sys.path. Therefore, if your zip file contains a Python file
    # mymodule.py you can import it using:
    # import mymodule

    # Return value must be of a sequence of pandas.DataFrame
    # E.g.
    #   -  Single return value: return dataframe1,
    #   -  Two return values: return dataframe1, dataframe2
    return dataframe1,
```

## <a name="upload-files"></a>Charger des fichiers
Le module Exécuter un script Python prend en charge le chargement de fichiers à l’aide du [kit SDK Azure Machine Learning Python](/python/api/azureml-core/azureml.core.run%28class%29?preserve-view=true&view=azure-ml-py#upload-file-name--path-or-stream-).

L’exemple suivant montre comment charger un fichier image dans le module Exécuter un script Python :

```Python

# The script MUST contain a function named azureml_main,
# which is the entry point for this module.

# Imports up here can be used to
import pandas as pd

# The entry point function must have two input arguments:
#   Param<dataframe1>: a pandas.DataFrame
#   Param<dataframe2>: a pandas.DataFrame
def azureml_main(dataframe1 = None, dataframe2 = None):

    # Execution logic goes here
    print(f'Input pandas.DataFrame #1: {dataframe1}')

    from matplotlib import pyplot as plt
    plt.plot([1, 2, 3, 4])
    plt.ylabel('some numbers')
    img_file = "line.png"
    plt.savefig(img_file)

    from azureml.core import Run
    run = Run.get_context(allow_offline=True)
    run.upload_file(f"graphics/{img_file}", img_file)

    # Return value must be of a sequence of pandas.DataFrame
    # For example:
    #   -  Single return value: return dataframe1,
    #   -  Two return values: return dataframe1, dataframe2
    return dataframe1,
}
```

Une fois l’exécution du pipeline terminée, vous pouvez voir un aperçu de l’image dans le panneau droit du module.

> [!div class="mx-imgBorder"]
> ![Aperçu de l’image chargée](media/module/upload-image-in-python-script.png)

## <a name="how-to-configure-execute-python-script"></a>Comment configurer le module Exécuter un script Python

Le module Exécuter un script Python contient un exemple de code Python que vous pouvez utiliser comme point de départ. Pour configurer le module Exécuter un script Python, fournissez un ensemble d’entrées et de code Python à exécuter dans la zone de texte **Script Python**.

1. Ajoutez le module **Exécuter un script Python** à votre pipeline.

2. À partir du concepteur, ajoutez et connectez dans **Jeu de données 1** tous les jeux de données que vous souhaitez utiliser pour l’entrée. Référencez ce jeu de données dans votre script Python sous le nom **DataFrame1**.

    L’utilisation d’un jeu de données est facultative. Utilisez-en un si vous souhaitez générer des données à l’aide de Python ou utiliser du code Python pour importer les données directement dans le module.

    Ce module prend en charge l’ajout d’un second jeu de données, **Dataset2**. Référencez ce second jeu de données dans votre script Python sous le nom **DataFrame2**.

    Les jeux de données stockés dans Azure Machine Learning sont automatiquement convertis en dataframes pandas lorsqu’ils sont chargés avec ce module.

    ![Mappage des entrées de l’exécution d’un script Python](media/module/python-module.png)

4. Pour inclure du code ou de nouveaux packages Python, connectez le fichier compressé qui contient ces ressources personnalisées au port **Script bundle**. Si la taille de votre script est supérieure à 16 Ko, vous pouvez également utiliser le port **Script Bundle** pour éviter des erreurs comme *La ligne de commande dépasse la limite de 16597 caractères*. 

    
    1. Regroupez le script et d'autres ressources personnalisées dans un fichier zip.
    1. Chargez le fichier zip en tant que **jeu de données** dans Studio. 
    1. Faites glisser le module du jeu de données de la liste *Jeux de données* vers le volet de module de gauche sur la page de création du concepteur. 
    1. Connectez le module de jeu de données au port **Script Bundle** du module **Exécuter le script R**.
    
    Tous les fichiers qui figurent dans l’archive zip chargée sont utilisables lors de l’exécution du pipeline. Si l’archive est contenue dans une structure de répertoires, la structure est conservée.
 
    > [!WARNING]
    > **N’utilisez pas** **app** comme nom de dossier ou de script, car **app** est un mot réservé pour les services intégrés. Toutefois, vous pouvez utiliser d’autres espaces de noms comme `app123`.
   
    Voici un exemple de regroupement de scripts, qui contient un fichier de script Python et un fichier txt :
      
    > [!div class="mx-imgBorder"]
    > ![Exemple de regroupement de scripts](media/module/python-script-bundle.png)  

    Voici le contenu de `my_script.py` :

    ```python
    def my_func(dataframe1):
    return dataframe1
    ```
    Voici un exemple de code montrant comment utiliser les fichiers dans le regroupement de scripts :    

    ```python
    import pandas as pd
    from my_script import my_func
 
    def azureml_main(dataframe1 = None, dataframe2 = None):
 
        # Execution logic goes here
        print(f'Input pandas.DataFrame #1: {dataframe1}')
 
        # Test the custom defined python function
        dataframe1 = my_func(dataframe1)
 
        # Test to read custom uploaded files by relative path
        with open('./Script Bundle/my_sample.txt', 'r') as text_file:
            sample = text_file.read()
    
        return dataframe1, pd.DataFrame(columns=["Sample"], data=[[sample]])
    ```

5. Dans la zone de texte **Script Python**, saisissez ou collez un script Python valide.

    > [!NOTE]
    >  Faites preuve de prudence lors de l’écriture de votre script. Assurez-vous qu’il n’existe pas d’erreurs de syntaxe, comme l’utilisation de variables non déclarées ou de modules ou fonctions non importés. Portez une attention particulière à la liste des modules préinstallés. Pour importer des modules qui ne sont pas listés, installez les packages correspondants dans votre script, tels que :
    >  ``` Python
    > import os
    > os.system(f"pip install scikit-misc")
    > ```
    
    La zone de texte **Script Python** est préremplie avec certaines instructions en commentaires, ainsi qu’avec un exemple de code pour l’accès aux données et la sortie. Vous devez modifier ou remplacer ce code. Suivez les conventions Python concernant la mise en retrait et la casse :

    + Le script doit contenir une fonction nommée `azureml_main` comme point d’entrée pour ce module.
    + La fonction de point d’entrée doit avoir deux arguments d’entrée, `Param<dataframe1>` et `Param<dataframe2>`, même lorsque ces arguments ne sont pas utilisés dans votre script.
    + Les fichiers compressés connectés au troisième port d’entrée sont décompressés et stockés dans le répertoire `.\Script Bundle`, qui est également ajouté à l’élément `sys.path` Python. 

    Si votre fichier .zip contient `mymodule.py`, importez-le à l’aide de `import mymodule`.

    Deux jeux de données peuvent être renvoyés au concepteur, ce qui doit constituer une séquence de type `pandas.DataFrame`. Vous pouvez créer d’autres sorties dans votre code Python et les écrire directement dans le service Stockage Azure.

    > [!WARNING]
    > Il n’est **pas** recommandé de se connecter à une base de données ou à d’autres stockages externes dans le **Module Exécuter un script Python**. Vous pouvez utiliser le [Module Importer des données](./import-data.md) et le [Module Exporter les données](./export-data.md)     

6. Envoyez le pipeline.

    La totalité des données et du code sont chargés sur une machine virtuelle et s’exécutent à l’aide de l’environnement Python spécifié.

## <a name="results"></a>Résultats

Les résultats des calculs effectués par le code Python incorporé doivent être fournis en tant que `pandas.DataFrame`, qui est automatiquement converti au format de jeu de données Azure Machine Learning. Vous pouvez ensuite utiliser ces résultats avec d’autres modules dans le pipeline.

Le module renvoie deux jeux de données :  
  
+ **Jeu de données de résultats 1**, défini par la première trame de données pandas renvoyée dans un script Python.

+ **Jeu de données de résultats 2**, défini par le second dataframe pandas retourné dans un script Python.

## <a name="preinstalled-python-packages"></a>Packages Python préinstallés
Les packages préinstallés sont les suivants :
-    adal==1.2.2
-    applicationinsights==0.11.9
-    attrs==19.3.0
-    azure-common==1.1.25
-    azure-core==1.3.0
-    azure-graphrbac==0.61.1
-    azure-identity==1.3.0
-    azure-mgmt-authorization==0.60.0
-    azure-mgmt-containerregistry==2.8.0
-    azure-mgmt-keyvault==2.2.0
-    azure-mgmt-resource==8.0.1
-    azure-mgmt-storage==8.0.0
-    azure-storage-blob==1.5.0
-    azure-storage-common==1.4.2
-    azureml-core==1.1.5.5
-    azureml-dataprep-native==14.1.0
-    azureml-dataprep==1.3.5
-    azureml-defaults==1.1.5.1
-    azureml-designer-classic-modules==0.0.118
-    azureml-designer-core==0.0.31
-    azureml-designer-internal==0.0.18
-    azureml-model-management-sdk==1.0.1b6.post1
-    azureml-pipeline-core==1.1.5
-    azureml-telemetry==1.1.5.3
-    backports.tempfile==1.0
-    backports.weakref==1.0.post1
-    boto3==1.12.29
-    botocore==1.15.29
-    cachetools==4.0.0
-    certifi==2019.11.28
-    cffi==1.12.3
-    chardet==3.0.4
-    click==7.1.1
-    cloudpickle==1.3.0
-    configparser==3.7.4
-    contextlib2==0.6.0.post1
-    cryptography==2.8
-    cycler==0.10.0
-    dill==0.3.1.1
-    distro==1.4.0
-    docker==4.2.0
-    docutils==0.15.2
-    dotnetcore2==2.1.13
-    flask==1.0.3
-    fusepy==3.0.1
-    gensim==3.8.1
-    google-api-core==1.16.0
-    google-auth==1.12.0
-    google-cloud-core==1.3.0
-    google-cloud-storage==1.26.0
-    google-resumable-media==0.5.0
-    googleapis-common-protos==1.51.0
-    gunicorn==19.9.0
-    idna==2.9
-    imbalanced-learn==0.4.3
-    isodate==0.6.0
-    itsdangerous==1.1.0
-    jeepney==0.4.3
-    jinja2==2.11.1
-    jmespath==0.9.5
-    joblib==0.14.0
-    json-logging-py==0.2
-    jsonpickle==1.3
-    jsonschema==3.0.1
-    kiwisolver==1.1.0
-    liac-arff==2.4.0
-    lightgbm==2.2.3
-    markupsafe==1.1.1
-    matplotlib==3.1.3
-    more-itertools==6.0.0
-    msal-extensions==0.1.3
-    msal==1.1.0
-    msrest==0.6.11
-    msrestazure==0.6.3
-    ndg-httpsclient==0.5.1
-    nimbusml==1.6.1
-    numpy==1.18.2
-    oauthlib==3.1.0
-    pandas==0.25.3
-    pathspec==0.7.0
-    pip==20.0.2
-    portalocker==1.6.0
-    protobuf==3.11.3
-    pyarrow==0.16.0
-    pyasn1-modules==0.2.8
-    pyasn1==0.4.8
-    pycparser==2.20
-    pycryptodomex==3.7.3
-    pyjwt==1.7.1
-    pyopenssl==19.1.0
-    pyparsing==2.4.6
-    pyrsistent==0.16.0
-    python-dateutil==2.8.1
-    pytz==2019.3
-    requests-oauthlib==1.3.0
-    requests==2.23.0
-    rsa==4.0
-    ruamel.yaml==0.15.89
-    s3transfer==0.3.3
-    scikit-learn==0.22.2
-    scipy==1.4.1
-    secretstorage==3.1.2
-    setuptools==46.1.1.post20200323
-    six==1.14.0
-    smart-open==1.10.0
-    urllib3==1.25.8
-    websocket-client==0.57.0
-    werkzeug==0.16.1
-    wheel==0.34.2

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning.
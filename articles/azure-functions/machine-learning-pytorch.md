---
title: Déployer un modèle PyTorch en tant qu’application Azure Functions
description: Utilisez un réseau neuronal profond ResNet 18 PyTorch préentraîné avec Azure Functions pour attribuer à une image l’une des 1 000 étiquettes ImageNet.
author: gvashishtha
ms.topic: tutorial
ms.date: 02/28/2020
ms.author: gopalv
ms.custom: devx-track-python, devx-track-azurepowershell
ms.openlocfilehash: 8891c29e5d8d06df6292d06ec06e5e57fb9880e7
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93422839"
---
# <a name="tutorial-deploy-a-pre-trained-image-classification-model-to-azure-functions-with-pytorch"></a>Tutoriel : Déployer un modèle de classification d’image préentraîné dans Azure Functions avec PyTorch

Dans cet article, vous allez découvrir comment utiliser Python, PyTorch et Azure Functions pour charger un modèle préentraîné en vue de classifier une image en fonction de son contenu. Comme vous effectuez tout le travail localement et que vous ne créez aucune ressource Azure dans le cloud, aucun coût n’est à prévoir pour effectuer ce tutoriel.

> [!div class="checklist"]
> * Initialisez un environnement local pour le développement de fonctions Azure en Python.
> * Importez un modèle Machine Learning PyTorch préentraîné dans une application de fonction.
> * Créez une API HTTP serverless pour classifier une image comme l’une des 1 000 [classes](https://gist.github.com/yrevar/942d3a0ac09ec9e5eb3a) ImageNet.
> * Consommez l’API à partir d’une application web.

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- [Python 3.7.4 ou version ultérieure](https://www.python.org/downloads/release/python-374/) (Python 3.8.x et Python 3.6.x sont également vérifiés avec Azure Functions).
- [Azure Functions Core Tools](functions-run-local.md#install-the-azure-functions-core-tools)
- Un éditeur de code tel que [Visual Studio Code](https://code.visualstudio.com/).

### <a name="prerequisite-check"></a>Vérification du prérequis

1. Dans un terminal ou une fenêtre de commande, exécutez `func --version` pour vérifier que la version d’Azure Functions Core Tools est bien 2.7.1846 ou ultérieure.
1. Exécutez `python --version` (Linux/MacOS) ou `py --version` (Windows) pour vérifier que votre version de Python est bien 3.7.x.

## <a name="clone-the-tutorial-repository"></a>Cloner le dépôt du tutoriel

1. Dans un fenêtre de terminal ou de commande, clonez le dépôt suivant en utilisant Git :

    ```
    git clone https://github.com/Azure-Samples/functions-python-pytorch-tutorial.git
    ```

1. Accédez au dossier et examinez son contenu.

    ```
    cd functions-python-pytorch-tutorial
    ```

    - *start* est votre dossier de travail pour le tutoriel.
    - *end* est le résultat final et l’implémentation complète à titre de référence.
    - *resources* contient le modèle Machine Learning et les bibliothèques de helpers.
    - *frontend* est un site web qui appelle l’application de fonction.

## <a name="create-and-activate-a-python-virtual-environment"></a>Créer et activer un environnement virtuel Python

Accédez au dossier *start* , et exécutez les commandes suivantes pour créer et activer un environnement virtuel nommé `.venv`.


# <a name="bash"></a>[bash](#tab/bash)

```bash
cd start
python -m venv .venv
source .venv/bin/activate
```

Si Python n’a pas installé le package venv sur votre distribution Linux, exécutez la commande suivante :

```bash
sudo apt-get install python3-venv
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
cd start
py -m venv .venv
.venv\scripts\activate
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
cd start
py -m venv .venv
.venv\scripts\activate
```

---

Vous devez exécuter toutes les commandes suivantes dans cet environnement virtuel activé. (Pour quitter l’environnement virtuel, exécutez `deactivate`.)


## <a name="create-a-local-functions-project"></a>Créer un projet de fonctions local

Dans Azure Functions, un projet de fonction est un conteneur pour une ou plusieurs fonctions individuelles qui répond chacune à un déclencheur spécifique. Toutes les fonctions d’un projet partagent les mêmes configurations locales et d’hébergement. Dans cette section, vous créez un projet de fonction qui contient une seule fonction réutilisable nommée `classify`, qui fournit un point de terminaison HTTP. Vous y ajoutez du code spécifique dans une section ultérieure.

1. Dans le dossier *start* , utilisez Azure Functions Core Tools pour initialiser une application de fonction Python :

    ```
    func init --worker-runtime python
    ```

    Après l’initialisation, le dossier *start* contient différents fichiers pour le projet, dont des fichiers de configuration nommés [local.settings.json](functions-run-local.md#local-settings-file) et [host.json](functions-host-json.md). Comme *local.settings.json* peut contenir des secrets téléchargés depuis Azure, le fichier est exclu par défaut du contrôle de code source dans le fichier *.gitignore*.

    > [!TIP]
    > Comme un projet de fonction est lié à un runtime spécifique, toutes les fonctions du projet doivent être écrites dans le même langage.

1. Ajoutez une fonction à votre projet avec la commande suivante, où l’argument `--name` est le nom unique de votre fonction et où l’argument `--template` spécifie le déclencheur de la fonction. `func new` crée un sous-dossier correspondant au nom de la fonction qui contient un fichier de code approprié au langage choisi du projet et un fichier de configuration nommé *function.json*.

    ```
    func new --name classify --template "HTTP trigger"
    ```

    Cette commande crée un dossier correspondant au nom de la fonction, *classify*. Dans ce dossier se trouvent deux fichiers : *\_\_init\_\_.py* , qui contient le code de la fonction, et *function.json* , qui décrit le déclencheur de la fonction, et ses liaisons d’entrée et de sortie. Pour plus d’informations sur le contenu de ces fichiers, consultez la section [Examiner le contenu du fichier](./create-first-function-cli-python.md#optional-examine-the-file-contents) du guide de démarrage rapide Python.


## <a name="run-the-function-locally"></a>Exécuter la fonction localement

1. Démarrez la fonction en démarrant l’hôte du runtime Azure Functions local dans le dossier *start*  :

    ```
    func start
    ```

1. Une fois que le point de terminaison `classify` apparaît dans la sortie, accédez à l’URL ```http://localhost:7071/api/classify?name=Azure```. Le message « Hello Azure! » doit apparaître dans la sortie.

1. Utilisez **Ctrl**-**C** pour arrêter l’hôte.


## <a name="import-the-pytorch-model-and-add-helper-code"></a>Importer le modèle PyTorch et ajouter le code d’assistance

Pour modifier la fonction `classify` en vue de classifier une image en fonction de son contenu, utilisez un modèle [ResNet](https://arxiv.org/abs/1512.03385) préentraîné. Le modèle préentraîné, fourni par [PyTorch](https://pytorch.org/hub/pytorch_vision_resnet/), classifie une image comme faisant partie de l’une des 1 000 [classes ImageNet](https://gist.github.com/yrevar/942d3a0ac09ec9e5eb3a). Vous ajoutez ensuite du code helper et des dépendances à votre projet.

1. Dans le dossier *start* , exécutez la commande suivante pour copier le code de prédiction et les étiquettes dans le dossier *classify*.

    # <a name="bash"></a>[bash](#tab/bash)

    ```bash
    cp ../resources/predict.py classify
    cp ../resources/labels.txt classify
    ```

    # <a name="powershell"></a>[PowerShell](#tab/powershell)

    ```powershell
    copy ..\resources\predict.py classify
    copy ..\resources\labels.txt classify
    ```

    # <a name="cmd"></a>[Cmd](#tab/cmd)

    ```cmd
    copy ..\resources\predict.py classify
    copy ..\resources\labels.txt classify
    ```

    ---

1. Vérifiez que le dossier *classify* contient les fichiers nommés *predict.py* et *labels.txt*. Si ce n’est pas le cas, vérifiez que vous avez exécuté la commande dans le dossier *start*.

1. Ouvrez *start/requirements.txt* dans un éditeur de texte et ajoutez les dépendances exigées par le code d’assistance, qui doit ressembler à ceci :

    ```txt
    azure-functions
    requests
    -f https://download.pytorch.org/whl/torch_stable.html
    torch==1.5.0+cpu
    torchvision==0.6.0+cpu
    ```

1. Enregistrez *requirements.txt* , puis exécutez la commande suivante à partir du dossier *start* pour installer les dépendances.


    ```
    pip install --no-cache-dir -r requirements.txt
    ```

L’installation peut prendre quelques minutes, pendant lesquelles vous pouvez modifier la fonction dans la section suivante.
> [!TIP]
> >Sur Windows, vous pouvez rencontrer l’erreur « Impossible d’installer les packages en raison d’une erreur EnvironmentError : [errno 2]. Pas de fichier ou de répertoire correspondant : », suivi d’un long chemin vers un fichier, comme *sharded_mutable_dense_hashtable.cpython-37.pyc*. En général, cette erreur se produit parce que la profondeur du chemin du dossier devient trop longue. Dans ce cas, définissez la clé de Registre `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem@LongPathsEnabled` sur `1` pour activer les chemins longs. Vous pouvez également vérifier où votre interpréteur Python est installé. Si cet emplacement a un chemin long, essayez en le réinstallant dans un dossier avec un chemin plus court.

## <a name="update-the-function-to-run-predictions"></a>Mettre à jour la fonction pour effectuer des prédictions

1. Ouvrez *classify/\_\_init\_\_.py* dans un éditeur de texte et ajoutez les lignes suivantes après les instructions `import` existantes pour importer la bibliothèque JSON standard et les helpers *predict*  :

    :::code language="python" source="~/functions-pytorch/end/classify/__init__.py" range="1-6" highlight="5-6":::

1. Remplacez tout le contenu de la fonction `main` par le code suivant :

    :::code language="python" source="~/functions-pytorch/end/classify/__init__.py" range="8-19":::

    Cette fonction reçoit une URL d’image dans un paramètre de chaîne de requête nommé `img`. Elle appelle ensuite `predict_image_from_url` à partir de la bibliothèque de helpers pour télécharger et classifier l’image en utilisant le modèle PyTorch. La fonction retourne ensuite une réponse HTTP avec les résultats.

    > [!IMPORTANT]
    > Comme ce point de terminaison HTTP est appelé par une page web hébergée sur un autre domaine, la réponse comprend un en-tête `Access-Control-Allow-Origin` pour satisfaire aux exigences CORS (Cross-Origin Resource Sharing) du navigateur.
    >
    > Dans une application de production, remplacez `*` par l’origine spécifique de la page web pour renforcer la sécurité.

1. Enregistrez vos modifications puis, en supposant que l’installation des dépendances est terminée, redémarrez l’hôte de fonction local avec `func start`. Veillez à exécuter l’hôte dans le dossier *start* avec l’environnement virtuel activé. Sinon, l’hôte démarrera, mais vous verrez des erreurs lors de l’appel de la fonction.

    ```
    func start
    ```

1. Dans un navigateur, ouvrez l’URL suivante pour appeler la fonction avec l’URL d’une image de bouvier bernois, puis vérifiez que le code JSON retourné classifie l’image comme représentant un bouvier bernois.

    ```
    http://localhost:7071/api/classify?img=https://raw.githubusercontent.com/Azure-Samples/functions-python-pytorch-tutorial/master/resources/assets/Bernese-Mountain-Dog-Temperament-long.jpg
    ```

1. Conservez l’hôte en cours d’exécution, car vous allez l’utiliser à l’étape suivante.

### <a name="run-the-local-web-app-front-end-to-test-the-function"></a>Exécuter le front-end de l’application web locale pour tester la fonction

Pour tester l’appel du point de terminaison de fonction à partir d’une autre application web, il existe une application simple dans le dossier *frontend* du référentiel.

1. Ouvrez un nouveau terminal ou une invite de commandes, puis activez l’environnement virtuel (comme décrit précédemment sous [Créer et activer un environnement virtuel Python](#create-and-activate-a-python-virtual-environment)).

1. Accédez au dossier *frontend* du référentiel.

1. Démarrez un serveur HTTP avec Python :

    # <a name="bash"></a>[bash](#tab/bash)

    ```bash
    python -m http.server
    ```

    # <a name="powershell"></a>[PowerShell](#tab/powershell)

    ```powershell
    py -m http.server
    ```

    # <a name="cmd"></a>[Cmd](#tab/cmd)

    ```cmd
    py -m http.server
    ```

1. Dans un navigateur, accédez à `localhost:8000`, puis entrez une des URL de photo suivantes dans la zone de texte ou utilisez l’URL de n’importe quelle image accessible publiquement.

    - `https://raw.githubusercontent.com/Azure-Samples/functions-python-pytorch-tutorial/master/resources/assets/Bernese-Mountain-Dog-Temperament-long.jpg`
    - `https://github.com/Azure-Samples/functions-python-pytorch-tutorial/blob/master/resources/assets/bald-eagle.jpg?raw=true`
    - `https://raw.githubusercontent.com/Azure-Samples/functions-python-pytorch-tutorial/master/resources/assets/penguin.jpg`

1. Sélectionnez **Submit** (Envoyer) pour appeler le point de terminaison de la fonction afin de classifier l’image.

    ![Capture d’écran du projet terminé](media/machine-learning-pytorch/screenshot.png)

    Si le navigateur signale une erreur quand vous envoyez l’URL de l’image, vérifiez le terminal dans lequel vous exécutez l’application de fonction. Si vous voyez une erreur comme « Aucun module "PIL" trouvé », c’est que vous avez peut-être démarré l’application de fonction dans le dossier *start* sans activer au préalable l’environnement virtuel que vous avez créé précédemment. Si vous voyez encore des erreurs, réexécutez `pip install -r requirements.txt` avec l’environnement virtuel activé et regardez s’il y a des erreurs.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Comme l’intégralité de ce tutoriel s’exécute localement sur votre machine, il n’y a pas de ressources ni de services Azure à nettoyer.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez vu comment créer et personnaliser un point de terminaison d’API HTTP avec Azure Functions pour classifier des images à l’aide d’un modèle PyTorch. Vous avez également découvert comment appeler l’API depuis une application web. Vous pouvez utiliser les techniques de ce tutoriel pour élaborer des API plus complexes, qui s’exécutent sur le modèle de calcul serverless fourni par Azure Functions.

Voir aussi :

- [Déployer la fonction Azure en utilisant Visual Studio Code](https://code.visualstudio.com/docs/python/tutorial-azure-functions).
- [Guide des développeurs Python sur Azure Functions](./functions-reference-python.md)


> [!div class="nextstepaction"]
> [Déployer la fonction sur Azure Functions en utilisant le guide Azure CLI](./functions-run-local.md#publish)

---
title: Utiliser Python et TensorFlow pour le Machine Learning dans Azure
description: Utilisez Python, TensorFlow et Azure Functions avec un modèle Machine Learning pour classifier une image en fonction de son contenu.
author: anthonychu
ms.topic: tutorial
ms.date: 01/15/2020
ms.author: antchu
ms.custom: mvc, devx-track-python, devx-track-azurepowershell
ms.openlocfilehash: ca9ce27583168dfee1a597fce559afad38a3a8c7
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95994602"
---
# <a name="tutorial-apply-machine-learning-models-in-azure-functions-with-python-and-tensorflow"></a>Tutoriel : Appliquer des modèles Machine Learning dans Azure Functions avec Python et TensorFlow

Dans cet article, vous allez découvrir comment utiliser Python, TensorFlow et Azure Functions avec un modèle Machine Learning pour classifier une image en fonction de son contenu. Comme vous effectuez tout le travail localement et que vous ne créez aucune ressource Azure dans le cloud, aucun coût n’est à prévoir pour effectuer ce tutoriel.

> [!div class="checklist"]
> * Initialisez un environnement local pour le développement de fonctions Azure en Python.
> * Importez un modèle Machine Learning TensorFlow personnalisé dans une application de fonction.
> * Créez une API HTTP serverless pour classifier une image comme contenant un chien ou un chat.
> * Consommez l’API à partir d’une application web.

## <a name="prerequisites"></a>Prérequis 

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- [Python 3.7.4](https://www.python.org/downloads/release/python-374/). (Le fonctionnement de Python 3.7.4 et Python 3.6.x avec Azure Functions a été vérifié ; Python 3.8 et ultérieur ne sont pas encore pris en charge.)
- [Azure Functions Core Tools](functions-run-local.md#install-the-azure-functions-core-tools)
- Un éditeur de code tel que [Visual Studio Code](https://code.visualstudio.com/).

### <a name="prerequisite-check"></a>Vérification du prérequis

1. Dans un terminal ou une fenêtre de commande, exécutez `func --version` pour vérifier que la version d’Azure Functions Core Tools est bien 2.7.1846 ou ultérieure.
1. Exécutez `python --version` (Linux/MacOS) ou `py --version` (Windows) pour vérifier que votre version de Python est bien 3.7.x.

## <a name="clone-the-tutorial-repository"></a>Cloner le dépôt du tutoriel

1. Dans un fenêtre de terminal ou de commande, clonez le dépôt suivant en utilisant Git :

    ```
    git clone https://github.com/Azure-Samples/functions-python-tensorflow-tutorial.git
    ```

1. Accédez au dossier et examinez son contenu.

    ```
    cd functions-python-tensorflow-tutorial
    ```

    - *start* est votre dossier de travail pour le tutoriel.
    - *end* est le résultat final et l’implémentation complète à titre de référence.
    - *resources* contient le modèle Machine Learning et les bibliothèques de helpers.
    - *frontend* est un site web qui appelle l’application de fonction.
    
## <a name="create-and-activate-a-python-virtual-environment"></a>Créer et activer un environnement virtuel Python

Accédez au dossier *start*, et exécutez les commandes suivantes pour créer et activer un environnement virtuel nommé `.venv`. Veillez à utiliser Python 3.7, qui est pris en charge par Azure Functions.


# <a name="bash"></a>[bash](#tab/bash)

```bash
cd start
```

```bash
python -m venv .venv
```

```bash
source .venv/bin/activate
```

Si Python n’a pas installé le package venv sur votre distribution Linux, exécutez la commande suivante :

```bash
sudo apt-get install python3-venv
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
cd start
```

```powershell
py -3.7 -m venv .venv
```

```powershell
.venv\scripts\activate
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
cd start
```

```cmd
py -3.7 -m venv .venv
```

```cmd
.venv\scripts\activate
```

---

Vous devez exécuter toutes les commandes suivantes dans cet environnement virtuel activé. (Pour quitter l’environnement virtuel, exécutez `deactivate`.)


## <a name="create-a-local-functions-project"></a>Créer un projet de fonctions local

Dans Azure Functions, un projet de fonction est un conteneur pour une ou plusieurs fonctions individuelles qui répond chacune à un déclencheur spécifique. Toutes les fonctions d’un projet partagent les mêmes configurations locales et d’hébergement. Dans cette section, vous créez un projet de fonction qui contient une seule fonction réutilisable nommée `classify`, qui fournit un point de terminaison HTTP. Vous y ajoutez du code spécifique dans une section ultérieure.

1. Dans le dossier *start*, utilisez Azure Functions Core Tools pour initialiser une application de fonction Python :

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

    Cette commande crée un dossier correspondant au nom de la fonction, *classify*. Dans ce dossier se trouvent deux fichiers : *\_\_init\_\_.py*, qui contient le code de la fonction, et *function.json*, qui décrit le déclencheur de la fonction, et ses liaisons d’entrée et de sortie. Pour plus d’informations sur le contenu de ces fichiers, consultez la section [Examiner le contenu du fichier](./create-first-function-cli-python.md#optional-examine-the-file-contents) du guide de démarrage rapide Python.


## <a name="run-the-function-locally"></a>Exécuter la fonction localement

1. Démarrez la fonction en démarrant l’hôte du runtime Azure Functions local dans le dossier *start* :

    ```
    func start
    ```
    
1. Une fois que le point de terminaison `classify` apparaît dans la sortie, accédez à l’URL ```http://localhost:7071/api/classify?name=Azure```. Le message « Hello Azure! » doit apparaître dans la sortie.

1. Utilisez **Ctrl**-**C** pour arrêter l’hôte.


## <a name="import-the-tensorflow-model-and-add-helper-code"></a>Importer le modèle TensorFlow et ajouter le code helper

Pour modifier la fonction `classify` de façon à classifier une image en fonction de son contenu, vous utilisez un modèle TensorFlow prédéfini qui a été entraîné avec le service Azure Custom Vision et exporté depuis celui-ci. Le modèle, qui est contenu dans le dossier *resources* de l’exemple que vous avez cloné précédemment, classifie une image selon qu’elle contient un chien ou un chat. Vous ajoutez ensuite du code helper et des dépendances à votre projet.

Pour générer votre propre modèle en utilisant le niveau gratuit du service Custom Vision, suivez les instructions de l’[exemple de dépôt de projet](https://github.com/Azure-Samples/functions-python-tensorflow-tutorial/blob/master/train-custom-vision-model.md).

> [!TIP]
> Si vous voulez héberger votre modèle TensorFlow indépendant de l’application de fonction, vous pouvez monter à la place un partage de fichiers contenant votre modèle dans votre application de fonction Linux. Pour en savoir plus, consultez [Monter un partage de fichiers dans une application de fonction Python à l’aide d’Azure CLI](./scripts/functions-cli-mount-files-storage-linux.md).

1. Dans le dossier *start*, exécutez la commande suivante pour copier les fichiers modèle dans le dossier *classify*. Veillez à inclure `\*` dans la commande. 

    # <a name="bash"></a>[bash](#tab/bash)
    
    ```bash
    cp ../resources/model/* classify
    ```
    
    # <a name="powershell"></a>[PowerShell](#tab/powershell)
    
    ```powershell
    copy ..\resources\model\* classify
    ```
    
    # <a name="cmd"></a>[Cmd](#tab/cmd)
    
    ```cmd
    copy ..\resources\model\* classify
    ```
    
    ---
    
1. Vérifiez que le dossier *classify* contient les fichiers nommés *model.pb* et *labels.txt*. Si ce n’est pas le cas, vérifiez que vous avez exécuté la commande dans le dossier *start*.

1. Dans le dossier *start*, exécutez la commande suivante pour copier un fichier avec le code helper dans le dossier *classify* :

    # <a name="bash"></a>[bash](#tab/bash)
    
    ```bash
    cp ../resources/predict.py classify
    ```
    
    # <a name="powershell"></a>[PowerShell](#tab/powershell)
    
    ```powershell
    copy ..\resources\predict.py classify
    ```
    
    # <a name="cmd"></a>[Cmd](#tab/cmd)
    
    ```cmd
    copy ..\resources\predict.py classify
    ```
    
    ---

1. Vérifiez que le dossier *classify* contient à présent un fichier nommé *predict.py*.

1. Ouvrez *start/requirements.txt* dans un éditeur de texte et ajoutez les dépendances suivantes nécessaires au code helper :

    ```txt
    tensorflow==1.14
    Pillow
    requests
    ```
    
1. Enregistrez *requirements.txt*.

1. Installez les dépendances en exécutant la commande suivante dans le dossier *start*. L’installation peut prendre quelques minutes, pendant lesquelles vous pouvez modifier la fonction dans la section suivante.

    ```
    pip install --no-cache-dir -r requirements.txt
    ```
    
    Sur Windows, vous pouvez rencontrer l’erreur « Impossible d’installer les packages en raison d’une erreur EnvironmentError : [errno 2]. Pas de fichier ou de répertoire correspondant : », suivi d’un long chemin vers un fichier, comme *sharded_mutable_dense_hashtable.cpython-37.pyc*. En général, cette erreur se produit parce que la profondeur du chemin du dossier devient trop longue. Dans ce cas, définissez la clé de Registre `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem@LongPathsEnabled` sur `1` pour activer les chemins longs. Vous pouvez également vérifier où votre interpréteur Python est installé. Si cet emplacement a un chemin long, essayez en le réinstallant dans un dossier avec un chemin plus court.

> [!TIP]
> Lors de l’appel de *predict.py* pour effectuer sa première prédiction, une fonction nommée `_initialize` charge le modèle TensorFlow à partir du disque et le met en cache dans des variables globales. Cette mise en cache accélère les prédictions suivantes. Pour plus d’informations sur l’utilisation de variables globales, reportez-vous au [Guide des développeurs Python sur Azure Functions](functions-reference-python.md#global-variables).

## <a name="update-the-function-to-run-predictions"></a>Mettre à jour la fonction pour effectuer des prédictions

1. Ouvrez *classify/\_\_init\_\_.py* dans un éditeur de texte et ajoutez les lignes suivantes après les instructions `import` existantes pour importer la bibliothèque JSON standard et les helpers *predict* :

    :::code language="python" source="~/functions-python-tensorflow-tutorial/end/classify/__init__.py" range="1-6" highlight="5-6":::

1. Remplacez tout le contenu de la fonction `main` par le code suivant :

    :::code language="python" source="~/functions-python-tensorflow-tutorial/end/classify/__init__.py" range="8-19":::

    Cette fonction reçoit une URL d’image dans un paramètre de chaîne de requête nommé `img`. Elle appelle ensuite `predict_image_from_url` à partir de la bibliothèque de helpers pour télécharger et classifier l’image en utilisant le modèle TensorFlow. La fonction retourne ensuite une réponse HTTP avec les résultats. 

    > [!IMPORTANT]
    > Comme ce point de terminaison HTTP est appelé par une page web hébergée sur un autre domaine, la réponse comprend un en-tête `Access-Control-Allow-Origin` pour satisfaire aux exigences CORS (Cross-Origin Resource Sharing) du navigateur.
    >
    > Dans une application de production, remplacez `*` par l’origine spécifique de la page web pour renforcer la sécurité.

1. Enregistrez vos modifications puis, en supposant que l’installation des dépendances est terminée, redémarrez l’hôte de fonction local avec `func start`. Veillez à exécuter l’hôte dans le dossier *start* avec l’environnement virtuel activé. Sinon, l’hôte démarrera, mais vous verrez des erreurs lors de l’appel de la fonction.

    ```
    func start
    ```

1. Dans un navigateur, ouvrez l’URL suivante pour appeler la fonction avec l’URL d’une image de chat et vérifiez que le JSON retourné classifie l’image comme étant un chat.

    ```
    http://localhost:7071/api/classify?img=https://raw.githubusercontent.com/Azure-Samples/functions-python-tensorflow-tutorial/master/resources/assets/samples/cat1.png
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

    - `https://raw.githubusercontent.com/Azure-Samples/functions-python-tensorflow-tutorial/master/resources/assets/samples/cat1.png`
    - `https://raw.githubusercontent.com/Azure-Samples/functions-python-tensorflow-tutorial/master/resources/assets/samples/cat2.png`
    - `https://raw.githubusercontent.com/Azure-Samples/functions-python-tensorflow-tutorial/master/resources/assets/samples/dog1.png`
    - `https://raw.githubusercontent.com/Azure-Samples/functions-python-tensorflow-tutorial/master/resources/assets/samples/dog2.png`
    
1. Sélectionnez **Submit** (Envoyer) pour appeler le point de terminaison de la fonction afin de classifier l’image.

    ![Capture d’écran du projet terminé](media/functions-machine-learning-tensorflow/functions-machine-learning-tensorflow-screenshot.png)

    Si le navigateur signale une erreur quand vous envoyez l’URL de l’image, vérifiez le terminal dans lequel vous exécutez l’application de fonction. Si vous voyez une erreur comme « Aucun module "PIL" trouvé », c’est que vous avez peut-être démarré l’application de fonction dans le dossier *start* sans activer au préalable l’environnement virtuel que vous avez créé précédemment. Si vous voyez encore des erreurs, réexécutez `pip install -r requirements.txt` avec l’environnement virtuel activé et regardez s’il y a des erreurs.

> [!NOTE]
> Le modèle classifie toujours le contenu de l’image en tant que chat ou chien, que l’image contienne l’un ou l’autre, en classifiant par défaut en tant que chien. Par exemple, les images de tigres et de panthères sont généralement classifiées en tant que chat, mais les images d’éléphants, de carottes ou d’avions sont classifiées en tant que chien.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Comme l’intégralité de ce tutoriel s’exécute localement sur votre machine, il n’y a pas de ressources ni de services Azure à nettoyer.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez découvert comment créer et personnaliser un point de terminaison d’API HTTP avec Azure Functions pour classifier des images en utilisant un modèle TensorFlow. Vous avez également découvert comment appeler l’API depuis une application web. Vous pouvez utiliser les techniques de ce tutoriel pour élaborer des API plus complexes, qui s’exécutent sur le modèle de calcul serverless fourni par Azure Functions.

> [!div class="nextstepaction"]
> [Déployer la fonction sur Azure Functions en utilisant le guide Azure CLI](./functions-run-local.md#publish)

Voir aussi :

- [Déployer la fonction Azure en utilisant Visual Studio Code](https://code.visualstudio.com/docs/python/tutorial-azure-functions).
- [Guide des développeurs Python sur Azure Functions](./functions-reference-python.md)
- [Monter un partage de fichiers dans une application de fonction Python à l’aide d’Azure CLI](./scripts/functions-cli-mount-files-storage-linux.md)

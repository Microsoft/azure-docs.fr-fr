---
title: 'Tutoriel : Configurer l’extension Visual Studio Code (préversion)'
titleSuffix: Azure Machine Learning
description: Découvrez comment installer et exécuter des scripts d’entraînement à l’aide de l’extension Azure Machine Learning de Visual Studio Code.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: luisquintanilla
ms.author: luquinta
ms.date: 11/16/2020
ms.openlocfilehash: f485c12bf82dde798a77175a3fab4d76488150c3
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94701199"
---
# <a name="set-up-azure-machine-learning-visual-studio-code-extension-preview"></a>Configurer l’extension Azure Machine Learning de Visual Studio Code (préversion)

Découvrez comment installer et exécuter des scripts à l’aide de l’extension Azure Machine Learning de Visual Studio Code.

Dans ce tutoriel, vous allez apprendre à effectuer les tâches suivantes :

> [!div class="checklist"]
> * Installer l’extension Azure Machine Learning de Visual Studio Code
> * Vous connecter à votre compte Azure à partir de Visual Studio Code
> * Utiliser l’extension Azure Machine Learning pour exécuter un exemple de script

## <a name="prerequisites"></a>Prérequis

- Abonnement Azure. Si vous n’en avez pas, inscrivez-vous ou essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree)
- Visual Studio Code. [Installez-le](https://code.visualstudio.com/docs/setup/setup-overview) si ce n’est déjà fait.
- [Python 3](https://www.python.org/downloads/)

## <a name="install-the-extension"></a>Installer l’extension

1. Ouvrez Visual Studio Code.
1. Sélectionnez l’icône **Extensions** dans la **Barre d’activité** pour ouvrir la vue Extensions.
1. Dans la vue Extensions, recherchez « Azure Machine Learning ».
1. Sélectionnez **Installer**.

    > [!div class="mx-imgBorder"]
    > ![Installer l’extension Azure Machine Learning VS Code](./media/tutorial-setup-vscode-extension/install-aml-vscode-extension.PNG)

> [!NOTE]
> Vous pouvez également installer l’extension Azure Machine Learning par le biais de Visual Studio Marketplace en [téléchargeant directement le programme d’installation](https://aka.ms/vscodetoolsforai). 

Les autres étapes de ce tutoriel ont été testées avec la **version 0.6.8** de l’extension.

## <a name="sign-in-to-your-azure-account"></a>Vous connecter à votre compte Azure

Pour provisionner des ressources et exécuter des charges de travail sur Azure, vous devez vous connecter avec vos informations d’identification de compte Azure. Pour faciliter la gestion des comptes, Azure Machine Learning installe automatiquement l’extension Azure Account. Visitez le site suivant pour [en savoir plus sur l’extension Azure Account](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account).

1. Ouvrez la palette de commandes en sélectionnant **Affichage > Palette de commandes** dans la barre de menus. 
1. Entrez la commande « Azure: Sign In » dans la palette de commandes pour démarrer le processus de connexion.

## <a name="run-a-machine-learning-model-training-script-in-azure"></a>Exécuter un script d’entraînement de modèle Machine Learning dans Azure

Maintenant que vous êtes connecté à Azure avec vos informations d’identification de compte, effectuez les étapes de cette section afin d’apprendre à utiliser l’extension pour entraîner un modèle Machine Learning.

1. Téléchargez et décompressez le [dépôt VS Code Tools for AI](https://github.com/microsoft/vscode-tools-for-ai/archive/master.zip) n’importe où sur votre ordinateur.
1. Ouvrez le répertoire `mnist-vscode-docs-sample` dans Visual Studio Code.
1. Sélectionnez l’icône **Azure** dans la barre d’activité.
1. Sélectionnez l’icône **Exécuter l’expérience** en haut de la vue Azure Machine Learning.

    > [!div class="mx-imgBorder"]
    > ![Exécuter une expérience](./media/tutorial-setup-vscode-extension/run-experiment.PNG)

1. Une fois la palette de commandes développée, suivez les invites.

    > [!NOTE]
    > Si vous avez déjà provisionné des ressources Azure Machine Learning, consultez [Exécuter des expériences dans le guide VS Code](./how-to-manage-resources-vscode.md#run-experiment).

    1. Sélectionnez votre abonnement Azure.
    1. Dans la liste des environnements, sélectionnez **Fichier de dépendances Conda**.
    1. Appuyez sur **Entrée** pour parcourir le fichier de dépendances Conda. Ce fichier contient les dépendances nécessaires pour exécuter votre script. En l’occurrence, le fichier de dépendances est le fichier `env.yml` dans le répertoire `mnist-vscode-docs-sample`.
    1. Appuyez sur **Entrée** pour parcourir le fichier de script d’entraînement. Il s’agit du fichier qui contient le code d’un modèle Machine Learning qui classifie des images de chiffres manuscrits. En l’occurrence, le script pour l’entraînement du modèle est le fichier `train.py` dans le répertoire `mnist-vscode-docs-sample`.

1. À ce stade, un fichier de configuration semblable à celui ci-dessous apparaît dans l’éditeur de texte. La configuration contient les informations requises pour exécuter le travail d’entraînement, comme le fichier qui contient le code pour entraîner le modèle et les dépendances Python spécifiées à l’étape précédente.

    ```json
    {
        "workspace": "WS06271500",
        "resourceGroup": "WS06271500-rg2",
        "location": "South Central US",
        "experiment": "WS06271500-exp1",
        "compute": {
            "name": "WS06271500-com1",
            "vmSize": "Standard_D1_v2, Cores: 1; RAM: 3.5GB;"
        },
        "runConfiguration": {
            "filename": "WS06271500-com1-rc1",
            "environment": {
                "name": "WS06271500-env1",
                "conda_dependencies": [
                    "python=3.6.2",
                    "tensorflow=1.15.0",
                    "pip"
                ],
                "pip_dependencies": [
                    "azureml-defaults"
                ],
                "environment_variables": {}
            }
        }
    }
    ```

1. Quand vous êtes satisfait de votre configuration, lancez l’expérience en ouvrant la palette de commandes et en entrant la commande suivante :

    ```text
    Azure ML: Submit Experiment
    ```

    Le fichier `train.py` et le fichier de configuration sont alors envoyés vers votre espace de travail Azure Machine Learning. Le travail d’entraînement est ensuite démarré sur une ressource de calcul dans Azure.

### <a name="track-the-progress-of-the-training-script"></a>Suivre la progression du script d’entraînement

L’exécution du script peut prendre plusieurs minutes. Pour suivre sa progression :

1. Sélectionnez l’icône **Azure** dans la barre d’activité.
1. Développez votre nœud d’abonnement.
1. Développez le nœud de l’expérience en cours d’exécution. Il s’agit du nœud `{workspace}/Experiments/{experiment}` où les valeurs de l’espace de travail et de l’expérience correspondent aux propriétés définies dans le fichier de configuration.
1. Toutes les exécutions de l’expérience sont listées ainsi que leur état. Pour obtenir l’état le plus récent, cliquez sur l’icône d’actualisation en haut de la vue Azure Machine Learning.

    > [!div class="mx-imgBorder"]
    > ![Suivre la progression d’une expérience](./media/tutorial-setup-vscode-extension/track-experiment-progress.PNG)

### <a name="download-the-trained-model"></a>Télécharger le modèle entraîné

Une fois l’expérience exécutée, la sortie est un modèle entraîné. Pour télécharger les sorties localement :

1. Cliquez avec le bouton droit sur l’exécution la plus récente, puis sélectionnez **Télécharger les sorties**.

    > [!div class="mx-imgBorder"]
    > ![Télécharger un modèle entraîné](./media/tutorial-setup-vscode-extension/download-trained-model.PNG)

1. Sélectionnez l’emplacement d’enregistrement des sorties.
1. Un dossier portant le nom de votre exécution est téléchargé localement. Naviguez vers cet emplacement.
1. Les fichiers de modèle se trouvent dans le répertoire `outputs/outputs/model`.

## <a name="next-steps"></a>Étapes suivantes

* [Tutoriel : Entraîner et déployer un modèle de classification d’images TensorFlow à l’aide de l’extension Azure Machine Learning de Visual Studio Code](tutorial-train-deploy-image-classification-model-vscode.md).

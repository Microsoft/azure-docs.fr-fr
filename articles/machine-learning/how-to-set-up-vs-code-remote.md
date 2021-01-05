---
title: Se connecter à l’instance de calcul dans Visual Studio Code (préversion)
titleSuffix: Azure Machine Learning
description: Découvrez comment vous connecter à une instance de calcul Azure Machine Learning dans Visual Studio Code pour exécuter des charges de travail de développement à distance et Jupyter Notebook interactives.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.author: jmartens
author: j-martens
ms.date: 11/16/2020
ms.openlocfilehash: 0afbf32f80a3d19f6db457f398a213a634cfa3d4
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96447058"
---
# <a name="connect-to-an-azure-machine-learning-compute-instance-in-visual-studio-code-preview"></a>Se connecter à une instance de calcul Azure Machine Learning dans Visual Studio Code (préversion)

Dans cet article, vous allez apprendre à vous connecter à une instance de calcul Azure Machine Learning à l’aide de Visual Studio Code.

Une [Instance de calcul Azure Machine Learning](concept-compute-instance.md) est une station de travail Cloud complètement managée pour les scientifiques des données qui fournit des fonctionnalités de gestion et de préparation de l’entreprise pour les administrateurs informatiques.

Il existe deux façons de se connecter à une instance de calcul à partir de Visual Studio Code :

* Serveur Jupyter Notebook distant. Cette option vous permet de définir une instance de calcul en tant que serveur Jupyter Notebook distant.
* [Développement à distance de Visual Studio Code](https://code.visualstudio.com/docs/remote/remote-overview). Le développement à distance de Visual Studio Code vous permet d’utiliser un conteneur, un ordinateur distant ou Sous-système Windows pour Linux (WSL) en tant qu’environnement de développement complet.

## <a name="configure-compute-instance-as-remote-notebook-server"></a>Configurer une instance de calcul en tant que serveur de notebook distant

Pour configurer une instance de calcul en tant que serveur Jupyter Notebook distant, vous avez besoin de respecter quelques conditions préalables :

* Extension Azure Machine Learning de Visual Studio Code. Pour plus d’informations, consultez le [guide d’installation de l’extension Azure Machine Learning de Visual Studio Code](tutorial-setup-vscode-extension.md).
* Espace de travail Azure Machine Learning. [Utilisez l’extension Azure Machine Learning de Visual Studio Code pour créer un espace de travail](how-to-manage-resources-vscode.md#create-a-workspace) si vous n’en avez pas déjà un.

Pour se connecter à une instance de calcul :

1. Ouvrez un notebook Jupyter dans Visual Studio Code.
1. Lors du chargement de l’expérience intégrée du notebook, sélectionnez **Serveur Jupyter**.

    > [!div class="mx-imgBorder"]
    > ![Menu déroulant Lancer le serveur Jupyter Notebook distant d'Azure Machine Learning](media/how-to-set-up-vs-code-remote/launch-server-selection-dropdown.png)

    Vous pouvez également utiliser la palette de commandes :

    1. Ouvrez la palette de commandes en sélectionnant **Affichage > Palette de commandes** dans la barre de menus.
    1. Entrez `Azure ML: Connect to Compute instance Jupyter server` dans la zone de texte.

1. Choisissez `Azure ML Compute Instances` dans la liste des options du serveur Jupyter.
1. Sélectionnez votre abonnement dans la liste des abonnements. Si vous avez déjà configuré votre espace de travail Azure Machine Learning par défaut, cette étape est ignorée.
1. Sélectionnez votre espace de travail.
1. Sélectionnez votre instance de calcul dans la liste. Si vous n’en avez pas, sélectionnez **Créer une instance Capacité de calcul ML** et suivez les instructions pour en créer une.
1. Pour que les modifications prennent effet, vous devez recharger Visual Studio Code.
1. Ouvrez un notebook Jupyter et exécutez une cellule.

> [!IMPORTANT]
> Vous **DEVEZ** exécuter une cellule afin d’établir la connexion.

À ce stade, vous pouvez continuer à exécuter des cellules dans votre instance de Jupyter Notebook.

> [!TIP]
> Vous pouvez également utiliser des fichiers de script Python (.py) contenant des cellules de code de type Jupyter. Pour plus d’informations, consultez la [documentation interactive de Visual Studio Code relative à Python](https://code.visualstudio.com/docs/python/jupyter-support-py).

## <a name="configure-compute-instance-remote-development"></a>Configurer le développement à distance de l’instance de calcul

Pour bénéficier d’une expérience de développement à distance complète, vous devez respecter quelques conditions préalables :

* [Extension Remote SSH de Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh).
* Instance de calcul compatible SSH. Pour plus d’informations, consultez le guide [Créer une instance de calcul](how-to-create-manage-compute-instance.md).

> [!NOTE]
> Sur les plateformes Windows, vous devez [installer un client SSH compatible OpenSSH](https://code.visualstudio.com/docs/remote/troubleshooting#_installing-a-supported-ssh-client) s’il n’en existe pas déjà un. PuTTY n’est pas pris en charge sur Windows, car la commande SSH doit se trouver dans le chemin d’accès.

### <a name="get-the-ip-and-ssh-port-for-your-compute-instance"></a>Obtenir le port IP et SSH pour votre instance de calcul

1. Accédez à Azure Machine Learning Studio à l’adresse https://ml.azure.com/.
2. Sélectionnez votre [espace de travail](concept-workspace.md).
1. Cliquez sur l’onglet **Instances de calcul**.
1. Dans la colonne **URI de l’application**, cliquez sur le lien **SSH** de l’instance de calcul que vous souhaitez utiliser comme calcul distant. 
1. Dans la boîte de dialogue, notez l’adresse IP et le port SSH. 
1. Enregistrez votre clé privée dans le répertoire ~/.ssh/ de votre ordinateur local ; par exemple, ouvrez un éditeur pour un nouveau fichier et collez la clé dans : 

   **Linux** :

   ```sh
   vi ~/.ssh/id_azmlcitest_rsa  
   ```

   **Windows** :

   ```cmd
   notepad C:\Users\<username>\.ssh\id_azmlcitest_rsa
   ```

   La clé privée ressemble à ce qui suit :

   ```text
   -----BEGIN RSA PRIVATE KEY-----

   MIIEpAIBAAKCAQEAr99EPm0P4CaTPT2KtBt+kpN3rmsNNE5dS0vmGWxIXq4vAWXD
   ..... 
   ewMtLnDgXWYJo0IyQ91ynOdxbFoVOuuGNdDoBykUZPQfeHDONy2Raw==

   -----END RSA PRIVATE KEY-----
   ```

1. Modifiez les autorisations sur le fichier pour vous assurer que vous êtes le seul à pouvoir lire le fichier.  

   ```sh
   chmod 600 ~/.ssh/id_azmlcitest_rsa
   ```

### <a name="add-instance-as-a-host"></a>Ajouter une instance en tant qu’hôte

Ouvrez le fichier `~/.ssh/config` (Linux) ou `C:\Users<username>.ssh\config` (Windows) dans un éditeur et ajoutez une entrée semblable au contenu ci-dessous :

```
Host azmlci1 

    HostName 13.69.56.51 

    Port 50000 

    User azureuser 

    IdentityFile ~/.ssh/id_azmlcitest_rsa
```

Voici quelques détails sur les champs :

|Champ|Description|
|----|---------|
|Host|Utilisez le raccourci de votre choix pour l’instance de calcul |
|HostName|Il s’agit de l’adresse IP de l’instance de calcul |
|Port|Il s’agit du port affiché dans la boîte de dialogue SSH ci-dessus |
|Utilisateur|Il doit être défini sur `azureuser` |
|IdentityFile|Doit pointer vers le fichier où vous avez enregistré la clé privée |

À présent, vous devez être en mesure d’utiliser SSH pour votre instance de calcul, à l’aide de la syntaxe abrégée que vous avez utilisée précédemment, `ssh azmlci1`.

### <a name="connect-vs-code-to-the-instance"></a>Connecter VS Code à l’instance

1. Cliquez sur l’icône Remote-SSH dans la barre d’activité Visual Studio Code pour afficher vos configurations SSH.

1. Cliquez avec le bouton droit sur la configuration d’hôte SSH que vous venez de créer.

1. Sélectionnez **Se connecter à l’hôte dans la fenêtre active**. 

À partir de là, vous travaillez entièrement sur l’instance de calcul et vous pouvez modifier, déboguer, utiliser Git, utiliser des extensions, etc., tout comme vous pouvez le faire avec votre Visual Studio Code local.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez configuré Visual Studio Code Remote, vous pouvez utiliser une instance de calcul comme calcul à distance à partir de Visual Studio Code pour [déboguer votre code de manière interactive](how-to-debug-visual-studio-code.md).

[Tutoriel : Effectuer l’apprentissage de votre premier modèle ML](tutorial-1st-experiment-sdk-train.md) montre comment utiliser une instance de calcul avec un notebook intégré.

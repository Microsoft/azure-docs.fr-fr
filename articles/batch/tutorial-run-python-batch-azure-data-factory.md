---
title: 'Tutoriel : Exécuter des scripts Python par le biais de Data Factory'
description: Découvrez comment exécuter des scripts Python dans le cadre d’un pipeline par le biais d’Azure Data Factory avec Azure Batch.
author: pkshultz
ms.devlang: python
ms.topic: tutorial
ms.date: 08/12/2020
ms.author: peshultz
ms.custom: mvc, devx-track-python
ms.openlocfilehash: 6cc6e6a9739b8b06ab3c48dd3fd75f19de8d0787
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97106272"
---
# <a name="tutorial-run-python-scripts-through-azure-data-factory-using-azure-batch"></a>Tutoriel : Exécuter des scripts Python par le biais d’Azure Data Factory avec Azure Batch

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * S’authentifier avec des comptes de stockage et Batch
> * Développer et exécuter un script en Python
> * Créer un pool de nœuds de calcul pour exécuter une application
> * Planifier des charges de travail Python
> * Superviser un pipeline d’analytique
> * Accéder aux fichiers journaux

L’exemple ci-dessous exécute un script Python qui reçoit une entrée CSV à partir d’un conteneur de stockage d’objets blob, exécute un processus de manipulation de données et écrit la sortie dans un conteneur de stockage d’objets blob distinct.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

* Une distribution [Python](https://www.python.org/downloads/) installée pour le test local.
* Package `pip` [azure-storage-blob](https://pypi.org/project/azure-storage-blob/).
* Le [jeu de données iris.csv](https://www.kaggle.com/uciml/iris/version/2#Iris.csv).
* Un compte Azure Batch et un compte Stockage Azure lié. Consultez [Créer un compte Batch](quick-create-portal.md#create-a-batch-account) pour plus d’informations sur la façon de créer des comptes Batch et de les lier à des comptes de stockage.
* Un compte Azure Data Factory. Pour plus d’informations sur la création d’une fabrique de données par le biais du portail Azure, consultez [Créer une fabrique de données](../data-factory/quickstart-create-data-factory-portal.md#create-a-data-factory).
* [Batch Explorer](https://azure.github.io/BatchExplorer/).
* [Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

[!INCLUDE [batch-common-credentials](../../includes/batch-common-credentials.md)]

## <a name="create-a-batch-pool-using-batch-explorer"></a>Créer un pool Batch avec Batch Explorer

Dans cette section, vous allez utiliser Batch Explorer pour créer le pool Batch qui sera utilisé par votre pipeline Azure Data Factory. 

1. Connectez-vous à Batch Explorer à l’aide de vos informations d’identification Azure.
1. Sélectionnez votre compte Batch.
1. Créez un pool en sélectionnant **Pools** dans la barre de gauche, puis le bouton **Ajouter** au-dessus du formulaire de recherche. 
    1. Choisissez un ID et un nom d’affichage. Pour cet exemple, nous allons utiliser `custom-activity-pool`.
    1. Définissez le type de mise à l’échelle sur **Taille fixe** et définissez le nombre de nœuds dédiés sur 2.
    1. Sous **Science des données**, sélectionnez **Windows DSVM** comme système d’exploitation.
    1. Choisissez `Standard_f2s_v2` comme taille de machine virtuelle.
    1. Activez la tâche de démarrage et ajoutez la commande `cmd /c "pip install azure-storage-blob pandas"`. L’**utilisateur de pool** par défaut peut être conservé comme identité d’utilisateur.
    1. Sélectionnez **OK**.

## <a name="create-blob-containers"></a>Créer des conteneurs d’objets blob

Ici, vous allez créer des conteneurs d’objets blob pour stocker vos fichiers d’entrée et de sortie pour la tâche Batch de reconnaissance optique de caractères (OCR).

1. Connectez-vous à l’Explorateur Stockage à l’aide de vos informations d’identification Azure.
1. À l’aide du compte de stockage lié à votre compte Batch, créez deux conteneurs d’objets blob (un pour les fichiers d’entrée, un pour les fichiers de sortie) en suivant les étapes indiquées dans [Créer un conteneur d’objets blob](../vs-azure-tools-storage-explorer-blobs.md#create-a-blob-container).
    * Dans cet exemple, nous allons appeler notre conteneur d’entrée `input`et notre conteneur de sortie `output`.
1. Chargez [`iris.csv`](https://www.kaggle.com/uciml/iris/version/2#Iris.csv) dans votre conteneur d’entrée `input` à l’aide de l’Explorateur Stockage en suivant les étapes de la section [Gestion des objets blob dans un conteneur d’objets blob](../vs-azure-tools-storage-explorer-blobs.md#managing-blobs-in-a-blob-container).

## <a name="develop-a-script-in-python"></a>Développer un script en Python

Le script Python suivant charge le jeu de données `iris.csv` à partir de votre conteneur `input`, exécute un processus de manipulation de données et enregistre les résultats dans le conteneur `output`.

``` python
# Load libraries
from azure.storage.blob import BlobServiceClient
import pandas as pd

# Define parameters
storageAccountURL = "<storage-account-url>"
storageKey         = "<storage-account-key>"
containerName      = "output"

# Establish connection with the blob storage account
blob_service_client = BlockBlobService(account_url=storageAccountURL,
                               credential=storageKey
                               )

# Load iris dataset from the task node
df = pd.read_csv("iris.csv")

# Subset records
df = df[df['Species'] == "setosa"]

# Save the subset of the iris dataframe locally in task node
df.to_csv("iris_setosa.csv", index = False)

# Upload iris dataset
container_client = blob_service_client.get_container_client(containerName)
with open("iris_setosa.csv", "rb") as data:
    blob_client = container_client.upload_blob(name="iris_setosa.csv", data=data)
```

Enregistrez le script sous `main.py` et chargez-le vers le conteneur **Stockage Azure** `input`. Veillez à tester et vérifier son fonctionnement localement avant de le charger dans votre conteneur d’objets blob :

``` bash
python main.py
```

## <a name="set-up-an-azure-data-factory-pipeline"></a>Configurer un pipeline Azure Data Factory

Dans cette section, vous allez créer et vérifier un pipeline à l’aide d’un script Python.

1. Suivez la procédure de la section « Créer une fabrique de données » de [cet article](../data-factory/quickstart-create-data-factory-portal.md#create-a-data-factory) pour créer une fabrique de données.
1. Dans la zone **Ressources de fabrique** sélectionnez le bouton + (plus), puis sélectionnez **Pipeline**.
1. Sous l’onglet **Général**, nommez le pipeline « Run Python ».

    ![Sous l’onglet Général, nommer le pipeline « Run Python »](./media/run-python-batch-azure-data-factory/create-pipeline.png)

1. Dans la zone **Activités**, développez **Service Batch**. Faites glisser l’activité personnalisée de la boîte à outils **Activités** vers la surface du concepteur de pipeline.
1. Dans l’onglet **Général**, spécifiez **testPipeline** pour le Nom.

    ![Sous l’onglet Général, spécifier testPipeline comme Nom](./media/run-python-batch-azure-data-factory/create-custom-task.png)
1. Sous l’onglet **Azure Batch**, ajoutez le **compte Batch** créé au cours des étapes précédentes et vérifiez que la connexion a réussi en sélectionnant **Tester la connexion**.

    ![Sous l’onglet Azure Batch, ajouter le compte Batch créé au cours des étapes précédentes, puis tester la connexion](./media/run-python-batch-azure-data-factory/integrate-pipeline-with-azure-batch.png)

1. Dans l’onglet **Paramètres**, entrez la commande `python main.py`.
1. Sous **Service lié de ressource**, ajoutez le compte de stockage créé au cours des étapes précédentes. Testez la connexion pour vous assurer qu’elle a réussi.
1. Sous **Chemin du dossier**, sélectionnez le nom du conteneur **Stockage Blob Azure** qui contient le script Python et les entrées associées. Les fichiers sélectionnés seront ainsi téléchargés à partir du conteneur vers les instances de nœud du pool avant l’exécution du script Python.

    ![Dans le chemin du dossier, sélectionner le nom du conteneur du Stockage Blob Azure](./media/run-python-batch-azure-data-factory/create-custom-task-py-script-command.png)
1. Pour valider les paramètres du pipeline, cliquez sur **Valider** sur la barre d’outils pour le pipeline au-dessus du canevas. Vérifiez que le pipeline a été validé avec succès. Pour fermer la sortie de validation, sélectionnez le bouton &gt;&gt; (flèche droite).
1. Cliquez sur **Déboguer** pour tester le pipeline et vérifier qu’il fonctionne correctement.
1. Cliquez sur **Publier** pour publier le pipeline.
1. Cliquez sur **Déclencher** pour exécuter le script Python dans le cadre d’un processus de traitement par lots.

    ![Cliquer sur Déclencher pour exécuter le script Python dans le cadre d’un processus de traitement par lots](./media/run-python-batch-azure-data-factory/create-custom-task-py-success-run.png)

### <a name="monitor-the-log-files"></a>Superviser les fichiers journaux

Si l’exécution du script génère des avertissements ou des erreurs, vous pouvez consulter `stdout.txt` ou `stderr.txt` pour obtenir plus d’informations sur la sortie journalisée.

1. Sélectionnez **Travaux** dans la partie gauche de Batch Explorer.
1. Choisissez le travail créé par votre fabrique de données. En supposant que vous avez nommé votre pool `custom-activity-pool`, sélectionnez `adfv2-custom-activity-pool`.
1. Cliquez sur la tâche pour laquelle un code de sortie d’échec a été retourné.
1. Consultez `stdout.txt` et `stderr.txt` pour examiner et diagnostiquer le problème.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Bien que vous ne soyez pas facturé pour les travaux et les tâches à proprement parler, les nœuds de calcul vous sont facturés. Par conséquent, nous vous conseillons d’affecter les pools uniquement en fonction des besoins. Lorsque vous supprimez le pool, toutes les sorties de tâche sur les nœuds sont supprimées. Toutefois, les fichiers d’entrée et de sortie restent dans le compte de stockage. Quand vous n’en avez plus besoin, vous pouvez également supprimer le compte Batch et le compte de stockage.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * S’authentifier avec des comptes de stockage et Batch
> * Développer et exécuter un script en Python
> * Créer un pool de nœuds de calcul pour exécuter une application
> * Planifier des charges de travail Python
> * Superviser un pipeline d’analytique
> * Accéder aux fichiers journaux

Pour en savoir plus sur Azure Data Factory, consultez :

> [!div class="nextstepaction"]
> [Aperçu d’Azure Data Factory](../data-factory/introduction.md)
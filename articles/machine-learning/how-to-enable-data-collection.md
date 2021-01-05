---
title: Collecter des données relatives à vos modèles de production
titleSuffix: Azure Machine Learning
description: Découvrez comment collecter des données à partir d’un modèle d’Azure Machine Learning déployé sur un cluster Azure Kubernetes service (AKS).
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: sgilley
ms.author: copeters
author: lostmygithubaccount
ms.date: 07/14/2020
ms.topic: conceptual
ms.custom: how-to, data4ml
ms.openlocfilehash: c6b9dc95e1d50481ac5353460910032ca1711ab1
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000451"
---
# <a name="collect-data-from-models-in-production"></a>Collecter des données pour des modèles en production

Cet article montre comment collecter des données à partir d’un modèle d’Azure Machine Learning déployé sur un cluster Azure Kubernetes service (AKS). Les données collectées sont alors stockées dans le Azure Blob.

Une fois la collecte activée, les données que vous collectez vous permettent d’effectuer les opérations suivantes :

* [Surveiller les dérives de données](how-to-monitor-datasets.md) sur les données de production que vous recueillez.

* Analyser les données collectées à l’aide de [Power BI](#powerbi) ou [Azure Databricks](#databricks)

* Prendre de meilleures décisions concernant le réentraînement ou l’optimisation de votre modèle

* Réentraîner votre modèle avec les données collectées

## <a name="what-is-collected-and-where-it-goes"></a>Types et destination des données collectées

Les données suivantes peuvent être collectées :

* Données d’entrée de modèle issues des services web déployés dans un cluster AKS. La voix audio, les images et les vidéos *ne sont pas* collectés.
  
* Prédictions de modèle utilisant des données d’entrée de production.

>[!NOTE]
> La préagrégation et les précalculs de ces données ne font pas partie du service de collecte.

La sortie est enregistrée dans le stockage d’objets blob. Étant donné que les données sont ajoutées au stockage d’objets blob, vous pouvez choisir votre outil favori pour exécuter l’analyse.

Le chemin des données de sortie dans l’objet blob respecte cette syntaxe :

```
/modeldata/<subscriptionid>/<resourcegroup>/<workspace>/<webservice>/<model>/<version>/<designation>/<year>/<month>/<day>/data.csv
# example: /modeldata/1a2b3c4d-5e6f-7g8h-9i10-j11k12l13m14/myresourcegrp/myWorkspace/aks-w-collv9/best_model/10/inputs/2018/12/31/data.csv
```

>[!NOTE]
> Dans les versions du SDK Azure Machine Learning pour Python qui sont antérieures à la version 0.1.0a16, l’argument `designation` se nomme `identifier`. Si votre code a été développé avec une version antérieure, vous devrez le mettre à jour.

## <a name="prerequisites"></a>Prérequis

- Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://aka.ms/AMLFree) avant de commencer.

- Vous devez installer un espace de travail Azure Machine Learning, un répertoire local contenant vos scripts, et le SDK Azure Machine Learning pour Python. Pour savoir comment les installer, consultez [Guide pratique pour configurer un environnement de développement](how-to-configure-environment.md).

- Vous devez déployer un modèle Machine Learning entraîné sur AKS. Si vous n’avez pas de modèle, consultez le tutoriel [Entraîner un modèle de classification des images](tutorial-train-models-with-aml.md).

- Vous avez besoin d’un cluster AKS. Pour plus d’informations sur la création et le déploiement d’un cluster AKS, consultez [Où et comment effectuer un déploiement](how-to-deploy-and-where.md)

- [Configurez votre environnement](how-to-configure-environment.md) et installez le [SDK de supervision Azure Machine Learning](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py).

## <a name="enable-data-collection"></a>Activer la collecte des données

Vous pouvez activer la [collecte de données](/python/api/azureml-monitoring/azureml.monitoring.modeldatacollector.modeldatacollector?preserve-view=true&view=azure-ml-py), quel que soit le modèle que vous déployez par le biais d’Azure Machine Learning ou d’autres outils.

Pour activer la collecte de données, vous devez :

1. Ouvrir le fichier de scoring.

1. Ajoutez le code suivant au début du fichier :

   ```python 
   from azureml.monitoring import ModelDataCollector
   ```

1. Déclarez vos variables de collecte de données dans votre fonction `init` :

    ```python
    global inputs_dc, prediction_dc
    inputs_dc = ModelDataCollector("best_model", designation="inputs", feature_names=["feat1", "feat2", "feat3", "feat4", "feat5", "feat6"])
    prediction_dc = ModelDataCollector("best_model", designation="predictions", feature_names=["prediction1", "prediction2"])
    ```

    *CorrelationId* est un paramètre facultatif. Il n’est pas obligatoire si votre modèle n’en a pas besoin. L’utilisation de *CorrelationId* vous permet un mappage plus facile avec d’autres données, telles que *LoanNumber* ou *CustomerId*.
    
    Le paramètre *Identifier* est utilisé ultérieurement pour créer la structure de dossiers dans votre objet blob. Vous pouvez l’utiliser pour différencier les données brutes des données traitées.

1. Ajoutez les lignes de code suivantes à la fonction `run(input_df)` :

    ```python
    data = np.array(data)
    result = model.predict(data)
    inputs_dc.collect(data) #this call is saving our input data into Azure Blob
    prediction_dc.collect(result) #this call is saving our input data into Azure Blob
    ```

1. La collecte de données *n’est pas* automatiquement définie sur **true** quand vous déployez un service dans AKS. Mettez à jour votre fichier de configuration, comme dans l’exemple suivant :

    ```python
    aks_config = AksWebservice.deploy_configuration(collect_model_data=True)
    ```

    Vous pouvez également activer Application Insights pour la supervision de service en modifiant cette configuration :

    ```python
    aks_config = AksWebservice.deploy_configuration(collect_model_data=True, enable_app_insights=True)
    ```

1. Pour créer une image et déployer le modèle Machine Learning, consultez [Où et comment effectuer un déploiement](how-to-deploy-and-where.md).


## <a name="disable-data-collection"></a>Désactiver la collecte de données

Vous pouvez arrêter la collecte de données à tout moment. Utilisez le code Python pour désactiver la collecte de données.

  ```python 
  ## replace <service_name> with the name of the web service
  <service_name>.update(collect_model_data=False)
  ```

## <a name="validate-and-analyze-your-data"></a>Valider et analyser vos données

Vous pouvez choisir l’outil de votre choix pour analyser les données collectées dans votre stockage d’objets blob.

### <a name="quickly-access-your-blob-data"></a>Accéder rapidement à vos données d’objets blob

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Ouvrez votre espace de travail.

1. Sélectionnez **Stockage**.

    [![Sélection de l’option de stockage](./media/how-to-enable-data-collection/StorageLocation.png)](././media/how-to-enable-data-collection/StorageLocation.png#lightbox)

1. Suivez le chemin des données de sortie de l’objet blob avec cette syntaxe :

   ```
   /modeldata/<subscriptionid>/<resourcegroup>/<workspace>/<webservice>/<model>/<version>/<designation>/<year>/<month>/<day>/data.csv
   # example: /modeldata/1a2b3c4d-5e6f-7g8h-9i10-j11k12l13m14/myresourcegrp/myWorkspace/aks-w-collv9/best_model/10/inputs/2018/12/31/data.csv
   ```

### <a name="analyze-model-data-using-power-bi"></a><a id="powerbi"></a>Analyser des données de modèle à l’aide de Power BI

1. Téléchargez et ouvrez [Power BI Desktop](https://www.powerbi.com).

1. Sélectionnez **Obtenir des données**, puis sélectionnez [**Stockage Blob Azure**](/power-bi/desktop-data-sources).

    [![Configuration de l’objet blob Power BI](./media/how-to-enable-data-collection/PBIBlob.png)](././media/how-to-enable-data-collection/PBIBlob.png#lightbox)

1. Ajoutez le nom de votre compte de stockage et entrez votre clé de stockage. Vous pouvez accéder à ces informations en sélectionnant **Paramètres** > **Clés d’accès** dans votre objet blob.

1. Sélectionnez le conteneur de **données de modèle**, puis sélectionnez **Modifier**.

    [![Navigateur - Power BI](./media/how-to-enable-data-collection/pbiNavigator.png)](././media/how-to-enable-data-collection/pbiNavigator.png#lightbox)

1. Dans l’éditeur de requête, cliquez sous la colonne **Name** et ajoutez votre compte de stockage.

1. Entrez le chemin du modèle dans le filtre. Si vous ne voulez examiner que les fichiers d’une année ou d’un mois spécifique, développez simplement le chemin du filtre. Par exemple, pour rechercher uniquement les données du mois de mars, utilisez ce chemin de filtre :

   /modeldata/\<subscriptionid>/\<resourcegroupname>/\<workspacename>/\<webservicename>/\<modelname>/\<modelversion>/\<designation>/\<year>/3

1. Filtrez les données qui vous concernent en fonction des valeurs de **Nom**. Si vous avez stocké des prédictions et des entrées, vous devrez créer une requête pour chacune d’elles.

1. Sélectionnez les flèches doubles vers le bas en regard de l’en-tête de colonne **Contenu** pour combiner les fichiers.

    [![Contenu - Power BI](./media/how-to-enable-data-collection/pbiContent.png)](././media/how-to-enable-data-collection/pbiContent.png#lightbox)

1. Sélectionnez **OK**. Les données sont préchargées.

    [![Combiner les fichiers - Power BI](./media/how-to-enable-data-collection/pbiCombine.png)](././media/how-to-enable-data-collection/pbiCombine.png#lightbox)

1. Sélectionnez **Fermer et appliquer**.

1. Si vous avez ajouté des entrées et des prédictions, vos tables sont automatiquement classées en fonction des valeurs **RequestId**.

1. Commencez à créer vos rapports personnalisés à partir des données de votre modèle.

### <a name="analyze-model-data-using-azure-databricks"></a><a id="databricks"></a>Analyser des données de modèle à l’aide d’Azure Databricks

1. Créez un [espace de travail Azure Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal).

1. Accédez à votre espace de travail Databricks.

1. Dans votre espace de travail Databricks, sélectionnez **Charger des données**.

    [![Sélection de l’option Charger des données - Databricks](./media/how-to-enable-data-collection/dbupload.png)](././media/how-to-enable-data-collection/dbupload.png#lightbox)

1. Sélectionnez **Créer une table**, puis sélectionnez **Autres sources de données** > **Stockage Blob Azure** > **Créer une table dans Notebook**.

    [![Création d’une table - Databricks](./media/how-to-enable-data-collection/dbtable.PNG)](././media/how-to-enable-data-collection/dbtable.PNG#lightbox)

1. Mettez à jour l’emplacement de vos données. Voici un exemple :

    ```
    file_location = "wasbs://mycontainer@storageaccountname.blob.core.windows.net/modeldata/1a2b3c4d-5e6f-7g8h-9i10-j11k12l13m14/myresourcegrp/myWorkspace/aks-w-collv9/best_model/10/inputs/2018/*/*/data.csv" 
    file_type = "csv"
    ```

    [![Configuration Databricks](./media/how-to-enable-data-collection/dbsetup.png)](././media/how-to-enable-data-collection/dbsetup.png#lightbox)

1. Suivez les étapes du modèle afin d’afficher et d’analyser vos données.

## <a name="next-steps"></a>Étapes suivantes

[Détectez la dérive des données](how-to-monitor-datasets.md) sur les données que vous avez collectées.
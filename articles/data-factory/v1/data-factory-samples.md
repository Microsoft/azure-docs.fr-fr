---
title: Azure Data Factory - Exemples
description: Fournit des détails sur les exemples fournis avec le service Azure Data Factory.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: b373b5c440bf2ef2f50bf323baa2488981cdc775
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96495189"
---
# <a name="azure-data-factory---samples"></a>Azure Data Factory - Exemples
> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Exemples PowerShell dans Data Factory](../samples-powershell.md) et [Exemples de code dans la galerie d’exemples de code Azure](https://azure.microsoft.com/resources/samples/?service=data-factory).


## <a name="samples-on-github"></a>Exemples sur GitHub
Le [dépôt GitHub Azure-DataFactory](https://github.com/azure/azure-datafactory) contient plusieurs exemples qui vous aideront à prendre en main rapidement le service Azure Data Factory (ou) à modifier les scripts et à utiliser ce service dans votre propre application. Le dossier Samples\JSON contient des extraits de code JSON pour les scénarios courants.

| Exemple | Description |
|:--- |:--- |
| [Procédure pas à pas Azure Data Factory](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/ADFWalkthrough) |Cet exemple fournit une procédure pas à pas de traitement de journaux à l’aide d’Azure Data Factory pour transformer les données des fichiers journaux en informations. <br/><br/>Dans cette procédure, le pipeline Data Factory collecte des exemples de journaux d’activité, traite et enrichit les données des journaux d’activité avec des données de référence, puis transforme les données pour évaluer l’efficacité d’une campagne marketing lancée dernièrement. |
| [Exemples JSON](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/JSON) |Cet exemple fournit des exemples JSON pour des scénarios courants. |
| [Exemple de téléchargeur de données HTTP](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/HttpDataDownloaderSample) |Cet exemple montre le téléchargement des données à partir d’un point de terminaison HTTP vers Azure Blob Storage, à l’aide d’une activité .NET personnalisée. |
| [Exemple d’activité .NET entre AppDomains](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/CrossAppDomainDotNetActivitySample) |Cet exemple vous montre comment créer une activité .NET personnalisée qui ne soit pas limitée aux versions d’assembly utilisées par le lanceur d’Azure Data Factory (par exemple, WindowsAzure.Storage v4.3.0, Newtonsoft.Json v6.0.x, etc.). |
| [Exécuter un script R](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/RunRScriptUsingADFSample) |Cet exemple inclut l’activité personnalisée Data Factory qui peut être utilisée pour appeler RScript.exe. Cet exemple fonctionne uniquement avec votre propre cluster HDInsight (et non celui à la demande) sur lequel est déjà installé le script R. |
| [Appeler des travaux Spark sur un cluster HDInsight Hadoop](../tutorial-transform-data-spark-portal.md) |Cet exemple montre comment utiliser l’activité MapReduce pour appeler un programme Spark. Le programme Spark copie simplement les données d'un conteneur d'objets blob Azure vers un autre. |
| [Analyse de Twitter à l’aide d’une activité de scoring par lots Azure Machine Learning studio (classique)](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/TwitterAnalysisSample-AzureMLBatchScoringActivity) |Cet exemple montre comment utiliser AzureMLBatchScoringActivity pour appeler un modèle Azure Machine Learning qui effectue une analyse de sentiments, une notation, des prédictions, etc. pour Twitter. |
| [Analyse de Twitter à l’aide d’une activité personnalisée](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/TwitterAnalysisSample-CustomC%23Activity) |Cet exemple montre comment utiliser une activité .NET personnalisée pour appeler un modèle Azure Machine Learning qui effectue une analyse de sentiments, une notation, des prédictions, etc. pour Twitter. |
| [Pipelines paramétrés pour Azure Machine Learning](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/ParameterizedPipelinesForAzureML) |Cet exemple fournit du code C# de bout en bout permettant de déployer des pipelines N, puis d’en effectuer la notation et le recyclage avec un paramètre de région différent pour chacun. La liste des régions provient du fichier parameters.txt qui est inclus avec cet exemple. |
| [Actualisation des données de référence pour les travaux Azure Stream Analytics](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/ReferenceDataRefreshForASAJobs) |Cet exemple montre comment utiliser Azure Data Factory et Azure Stream Analytics pour exécuter des requêtes avec des données de référence et planifier l’actualisation des données de référence. |
| [Pipeline hybride avec une instance locale d’Hadoop Hortonworks](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/HybridPipelineWithOnPremisesHortonworksHadoop) |Cet exemple utilise un cluster Hadoop local comme cible de calcul pour exécuter des travaux dans Data Factory. La procédure est la même que pour ajouter d’autres cibles de calcul, telles qu’un cluster Hadoop HDInsight, dans le cloud. |
| [Outil de conversion JSON](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/JSONConversionTool) |Cet outil vous permet de convertir des fichiers JSON depuis les versions antérieures à 2015-07-01-preview vers la version 2015-07-01-preview, qui est la plus récente (par défaut). |
| [Exemple de fichier d’entrée U-SQL](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/U-SQL%20Sample%20Input%20File) |Il s’agit d’un exemple de fichier utilisé par une activité U-SQL. |
| [Supprimer le fichier blob](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/DeleteBlobFileFolderCustomActivity) | Cet exemple présente un fichier C# qui peut être utilisé dans le cadre de l’activité ADF .net personnalisée pour supprimer des fichiers de l’emplacement d’objets Blob Azure source une fois que les fichiers ont été copiés.|

## <a name="azure-resource-manager-templates"></a>Modèles Microsoft Azure Resource Manager
Vous trouverez les modèles Azure Resource Manager suivants pour Data Factory sur GitHub.

| Modèle | Description |
| --- | --- |
| [Copier à partir du Stockage Blob Azure vers une base de données Azure SQL](https://github.com/Azure/azure-quickstart-templates/tree/master/101-data-factory-blob-to-sql-copy) |Le déploiement de ce modèle crée une fabrique de données Azure avec un pipeline qui copie les données à partir du Stockage Blob Azure spécifié vers Azure SQL Database |
| [Copier à partir de Salesforce vers le Stockage Blob Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/101-data-factory-salesforce-to-blob-copy) |Le déploiement de ce modèle crée une fabrique de données Azure avec un pipeline qui copie les données à partir du compte Salesforce spécifié vers le Stockage Blob Azure. |
| [Transformer les données en exécutant le script Hive sur un cluster Azure HDInsight](https://github.com/Azure/azure-quickstart-templates/tree/master/101-data-factory-hive-transformation) |Le déploiement de ce modèle crée une fabrique de données Azure avec un pipeline qui transforme les données en exécutant le script Hive sur un cluster Azure HDInsight Hadoop. |

## <a name="samples-in-azure-portal"></a>Exemples dans le portail Azure
Vous pouvez utiliser la vignette **Exemples de pipelines** figurant dans la page d’accueil de votre fabrique de données pour déployer des exemples de pipelines et les entités associées (ensembles de données et services liés) dans votre fabrique de données.

1. Créez une fabrique de données ou ouvrez une fabrique de données existante. Pour connaître les étapes de création d’une fabrique de données, consultez [Copie de données Blob Storage vers une base de données SQL à l’aide de Data Factory](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
2. Dans le panneau **DATA FACTORY** de la fabrique de données, cliquez sur la vignette **Exemples de pipelines**.

    ![Vignette Exemples de pipelines](./media/data-factory-samples/SamplePipelinesTile.png)
3. Dans le panneau **Exemples de pipelines**, cliquez sur **l’exemple** que vous souhaitez déployer.

    ![Panneau Exemples de pipelines](./media/data-factory-samples/SampleTile.png)
4. Spécifiez les paramètres de configuration de l’exemple. Par exemple, votre clé de compte et votre nom de compte de stockage Azure, le nom du serveur SQL logique, la base de données, l’ID d’utilisateur, le mot de passe, etc.

    ![Panneau Exemple](./media/data-factory-samples/SampleBlade.png)
5. Une fois que vous avez fini de spécifier les paramètres de configuration, cliquez sur **Créer** pour créer/déployer les exemples de pipelines et les tables/services liés utilisés par les pipelines.
6. L’état du déploiement s’affiche sur l’exemple de mosaïque sur laquelle vous avez cliqué dans le panneau **Exemples de pipelines**.

    ![état du déploiement](./media/data-factory-samples/DeploymentStatus.png)
7. Lorsque vous voyez le message **Déploiement réussi** sur la mosaïque de l’exemple, fermez le panneau **Exemples de pipelines**.  
8. Dans le panneau **DATA FACTORY**, vous pouvez voir que les services liés, les jeux de données et les pipelines sont ajoutés à votre fabrique de données.  

    ![Panneau Data Factory](./media/data-factory-samples/DataFactoryBladeAfter.png)

## <a name="samples-in-visual-studio"></a>Exemples dans Visual Studio
### <a name="prerequisites"></a>Prérequis
Les composants suivants doivent être installés sur votre ordinateur :

* Visual Studio 2013 ou Visual Studio 2015
* Téléchargez le Kit de développement logiciel (SDK) Azure pour Visual Studio 2013 ou Visual Studio 2015. Accédez à la [page de téléchargement d’Azure](https://azure.microsoft.com/downloads/), puis cliquez sur **VS 2013** ou **VS 2015** dans la section  **.NET**.
* Téléchargez le dernier plug-in Azure Data Factory pour Visual Studio : [VS 2013](https://visualstudiogallery.msdn.microsoft.com/754d998c-8f92-4aa7-835b-e89c8c954aa5) ou [VS 2015](https://visualstudiogallery.msdn.microsoft.com/371a4cf9-0093-40fa-b7dd-be3c74f49005). Si vous utilisez Visual Studio 2013, vous pouvez également mettre à jour le plug-in en effectuant les étapes suivantes : Dans le menu, cliquez sur **Outils** -> **Extensions et mises à jour** -> **En ligne** -> **Galerie Visual Studio** -> **Outils Microsoft Azure Data Factory pour Visual Studio** -> **Mettre à jour**.

### <a name="use-data-factory-templates"></a>Utilisation de modèles de fabrique de données
1. Cliquez sur **Fichier** dans le menu, pointez sur **Nouveau**, puis cliquez sur **Projet**.
2. Dans la boîte de dialogue **Nouveau projet**, procédez comme suit :

   1. Sous **Modèles**, sélectionnez **DataFactory**.
   2. Dans le volet droit, sélectionnez **Modèles de fabrique de données** .
   3. Entrez un **nom** pour le projet.
   4. Sélectionnez un **emplacement** pour le projet.
   5. Cliquez sur **OK**.

      ![Boîte de dialogue Nouveau projet](./media/data-factory-samples/vs-new-project-adf-templates.png)
3. Dans la boîte de dialogue **Modèles de fabrique de données**, sélectionnez l’exemple de modèle dans la section **Modèles de cas d’utilisation**, puis cliquez sur **Suivant**. La procédure suivante décrit l’utilisation du modèle **Analyse des profils des clients** . Les étapes sont similaires pour les autres exemples.

    ![Boîte de dialogue Modèles de fabrique de données](./media/data-factory-samples/vs-data-factory-templates-dialog.png)
4. Dans la boîte de dialogue **Configuration de Data Factory**, cliquez sur **Suivant** dans la page **Principes de base de Data Factory**.
5. Dans la page **Configurer une fabrique de données**, procédez comme suit :
   1. Sélectionnez **Créer une fabrique de données** . Vous pouvez également sélectionner **Utiliser une fabrique de données existante**.
   2. Entrez un **nom** pour la fabrique de données.
   3. Sélectionnez l’ **abonnement Azure** dans lequel vous voulez créer la fabrique de données.
   4. Sélectionnez le **groupe de ressources** pour la fabrique de données.
   5. Sélectionnez **USA Ouest**, **USA Est** ou **Europe Nord** pour la **région**.
   6. Cliquez sur **Suivant**.
6. Dans la page **Configurer des banques de données**, spécifiez une **base de données dans Azure SQL Database** et un **compte de stockage Azure** existants, ou créez-en des nouveaux, puis cliquez sur Suivant.
7. Dans la page **Configurer le calcul**, sélectionnez les valeurs par défaut, puis cliquez sur **Suivant**.
8. Dans la page **Résumé**, passez en revue tous les paramètres, puis cliquez sur **Suivant**.
9. Dans la page **État du déploiement**, patientez jusqu’à ce que le déploiement soit terminé, puis cliquez sur **Terminer**.
10. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet, puis cliquez sur **Publier**.
11. Si la boîte de dialogue **Connectez-vous à votre compte Microsoft** s’affiche, saisissez vos informations d’identification pour le compte associé à l’abonnement Azure, puis cliquez sur **Se connecter**.
12. La boîte de dialogue suivante doit s’afficher :

    ![Boîte de dialogue Publier](./media/data-factory-build-your-first-pipeline-using-vs/publish.png)
13. Dans la page **Configurer une fabrique de données**, procédez comme suit :

    1. Confirmez l’option **Utiliser une fabrique de données existante** .
    2. Sélectionnez la **fabrique de données** que vous aviez sélectionnée en utilisant le modèle.
    3. Cliquez sur **Suivant** pour basculer vers la page **Publier des éléments**. (Utilisez la touche **TABULATION** pour passer au champ Nom si le bouton **Suivant** est désactivé.)
14. Dans la page **Publier des éléments**, vérifiez que toutes les entités de fabriques de données sont sélectionnées, puis cliquez sur **Suivant** pour basculer vers la page **Résumé**.     
15. Passez en revue le résumé, puis cliquez sur **Suivant** pour démarrer le processus de déploiement et afficher l’**état du déploiement**.
16. Dans la page **État du déploiement** , vous devez voir l’état du processus de déploiement. Une fois le déploiement terminé, cliquez sur Terminer.

Pour plus d’informations sur l’utilisation de Visual Studio pour créer des entités Data Factory et les publier sur Azure, consultez [Créer votre première fabrique de données (Visual Studio)](data-factory-build-your-first-pipeline-using-vs.md) .
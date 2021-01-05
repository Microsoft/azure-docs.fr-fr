---
title: 'ML Studio (classique) : reformer un service web – Azure'
description: Apprenez à mettre à jour un service web de façon à utiliser un modèle Machine Learning récemment entraîné dans Azure Machine Learning Studio (classique).
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18, devx-track-csharp
ms.date: 02/14/2019
ms.openlocfilehash: ff0378871139a038f096a44b9ee0c6af2cb67d73
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93325821"
---
# <a name="retrain-and-deploy-a-machine-learning-model"></a>Recycler et déployer un modèle Machine Learning

**S’APPLIQUE À :**  ![S’applique à ](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Ne s’applique pas à ](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)


Le recyclage représente un moyen de garantir que les modèles Machine Learning restent précis et qu’ils reposent sur les données les plus pertinentes. Cet article montre comment recycler et déployer un modèle Machine Learning sous la forme d’un nouveau service web dans Studio (classique). Pour recycler un service web classique, voir cet article [guide pratique](retrain-classic-web-service.md).

Cet article suppose qu’un service web prédictif est déjà déployé. Si vous n’en disposez pas, [découvrez comment déployer un service web Studio (classique)](deploy-a-machine-learning-web-service.md).

Pour recycler et déployer un nouveau service web Machine Learning, vous allez suivre ces étapes :

1. Déployer un **service web de recyclage**.
1. Effectuer l’apprentissage d’un nouveau modèle à l’aide du **service web de recyclage**.
1. Mettre à jour **l’expérience prédictive** de façon à utiliser le nouveau modèle.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="deploy-the-retraining-web-service"></a>Déployez le service web de reformation

Un service web de recyclage permet de recycler un modèle avec un nouvel ensemble de paramètres, par exemple de nouvelles données, et de l’enregistrer pour plus tard. Lorsqu’une **Sortie de service web** est connectée à un module **Effectuer l’apprentissage du modèle** , l’expérience de formation génère un nouveau modèle utilisable.

Suivez les étapes ci-dessous pour déployer un service web de recyclage :

1. Connectez un module **Entrée de service web** à votre entrée de données. En règle générale, vous souhaitez vous assurer que vos données d’entrée sont traitées de la même manière que vos données de formation d’origine.
1. Connectez un module **Sortie de service web** à la sortie de votre module **Effectuer l’apprentissage du modèle**.
1. Si vous disposez d’un module **Évaluer le modèle** , vous pouvez connecter un module **Sortie de service web** pour obtenir les résultats de l’évaluation en sortie.
1. Exécutez votre expérience.

    Après exécution de l’expérience, le workflow obtenu devrait se présenter ainsi :

    ![Workflow obtenu](media/retrain-machine-learning/machine-learning-retrain-models-programmatically-IMAGE04.png)

    Vous allez maintenant déployer l’expérience de formation en tant que service web de recyclage produisant un modèle entraîné et les résultats de son évaluation.

1. En bas du canevas de l’expérience, cliquez sur **Configurer le service web**.
1. Sélectionnez **Déployer le service web [Nouveau]**. Le portail des services web Azure Machine Learning s’ouvre sur la page **Déployer le service web**.
1. Tapez le nom de votre service web et choisissez un plan de paiement.
1. Sélectionnez **Déployer**.

## <a name="retrain-the-model"></a>Recycler le modèle

Pour cet exemple, nous utilisons le langage C# pour créer l’application de reformation. Pour accomplir cette tâche, vous pouvez également utiliser un code Python ou R.

Suivez les étapes ci-dessous pour appeler les API de recyclage :

1. Créez une application console C# dans Visual Studio : **Nouveau** > **Projet** > **Visual C#**  > **Bureau classique Windows** > **Console App (.NET Framework)** .
1. Connectez-vous au portail des services web Azure Machine Learning.
1. Cliquez sur le service web que vous utilisez.
1. Cliquez sur **Consommer**.
1. En bas de la page **Utiliser** , dans la section **Exemple de code** , cliquez sur **Lot**.
1. Copiez l’exemple de code C# pour l’exécution par lot et collez-le dans le fichier Program.cs. Assurez-vous que l’espace de noms reste intact.

Ajoutez le package NuGet Microsoft.AspNet.WebApi.Client comme indiqué dans les commentaires. Pour ajouter la référence à Microsoft.WindowsAzure.Storage.dll, il peut se révéler nécessaire d’installer la [bibliothèque de client pour les services de Stockage Azure](https://www.nuget.org/packages/WindowsAzure.Storage).

La capture d’écran suivante montre la page **Consommer** du portail des services web Azure Machine Learning.

![Page Consommer](media/retrain-machine-learning/machine-learning-retrain-models-consume-page.png)

### <a name="update-the-apikey-declaration"></a>Mettre à jour la déclaration apiKey

Localisez la déclaration **apikey** :

```csharp
const string apiKey = "abc123"; // Replace this with the API key for the web service
```

Dans la section **Informations de base sur la consommation** de la page **Consommer** , recherchez la clé primaire et copiez-la dans la déclaration **apiKey**.

### <a name="update-the-azure-storage-information"></a>Mettre à jour les informations Azure Storage

L’exemple de code BES charge un fichier d’un lecteur local (par exemple, « C:\temp\CensusInput.csv ») vers le Stockage Azure, le traite et réécrit les résultats dans le Stockage Azure.

1. Se connecter au portail Azure
1. Dans la colonne de navigation de gauche, cliquez sur **Autres services** , recherchez **Comptes de stockage** et sélectionnez-le.
1. Dans la liste des comptes de stockage, sélectionnez-en un pour stocker le modèle reformé.
1. Dans la colonne de navigation de gauche, cliquez sur **Clés d’accès**.
1. Copiez et enregistrez la **Clé d’accès primaire**.
1. Dans la colonne de navigation de gauche, cliquez sur **Blobs**.
1. Sélectionnez un conteneur existant ou créez-en un et enregistrez le nom.

Localisez les déclarations *StorageAccountName* , *StorageAccountKey* et *StorageContainerName* , puis mettez à jour les valeurs que vous avez enregistrées à partir du portail.

```csharp
const string StorageAccountName = "mystorageacct"; // Replace this with your Azure storage account name
const string StorageAccountKey = "a_storage_account_key"; // Replace this with your Azure Storage key
const string StorageContainerName = "mycontainer"; // Replace this with your Azure Storage container name
```

Vous devez également vous assurer que le fichier d’entrée est disponible à l’emplacement spécifié dans le code.

### <a name="specify-the-output-location"></a>Spécifier l’emplacement de sortie

Lorsque vous spécifiez l’emplacement de sortie dans la Charge utile des demandes, l’extension du fichier spécifiée dans *RelativeLocation* doit être spécifiée en tant que valeur `ilearner`.

```csharp
Outputs = new Dictionary<string, AzureBlobDataReference>() {
    {
        "output1",
        new AzureBlobDataReference()
        {
            ConnectionString = storageConnectionString,
            RelativeLocation = string.Format("{0}/output1results.ilearner", StorageContainerName) /*Replace this with the location you want to use for your output file and a valid file extension (usually .csv for scoring results or .ilearner for trained models)*/
        }
    },
```

Voici un exemple de sortie de recyclage :

![Sortie du nouvel apprentissage.](media/retrain-machine-learning/machine-learning-retrain-models-programmatically-IMAGE06.png)

### <a name="evaluate-the-retraining-results"></a>Évaluer les résultats de la reformation

Lorsque vous exécutez l’application, la sortie inclut l’URL et le jeton de signature d’accès partagé (SAP) nécessaires pour accéder aux résultats de l’évaluation.

Pour consulter les résultats des performances du modèle recyclé, combinez *BaseLocation* , *RelativeLocation* et *SasBlobToken* dans les résultats de sortie de *output2* et collez l’URL complète dans la barre d’adresse du navigateur.

Examinez les résultats pour déterminer si le nouveau modèle entraîné est plus performant que l’actuel.

Enregistrez *BaseLocation* , *RelativeLocation* et *SasBlobToken* dans les résultats de sortie.

## <a name="update-the-predictive-experiment"></a>Mettre à jour l’expérience prédictive

### <a name="sign-in-to-azure-resource-manager"></a>Se connecter à Azure Resource Manager

Tout d’abord, connectez-vous à votre compte Azure dans l’environnement PowerShell avec l’applet de commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

### <a name="get-the-web-service-definition-object"></a>Obtenir l’objet Définition du service web

Ensuite, obtenez l’objet Définition du service web en appelant l’applet de commande [Get-AzMlWebService](/powershell/module/az.machinelearning/get-azmlwebservice).

```azurepowershell
$wsd = Get-AzMlWebService -Name 'RetrainSamplePre.2016.8.17.0.3.51.237' -ResourceGroupName 'Default-MachineLearning-SouthCentralUS'
```

Pour déterminer le nom du groupe de ressources d’un service web existant, exécutez l’applet de commande Get-AzMlWebService sans paramètres pour afficher les services web dans votre abonnement. Recherchez le service web et examinez son ID de service web. Le nom du groupe de ressources est le quatrième élément de l’ID, juste après l’élément *resourceGroups* . Dans l’exemple suivant, le nom du groupe de ressources est Default-MachineLearning-SouthCentralUS.

```azurepowershell
Properties : Microsoft.Azure.Management.MachineLearning.WebServices.Models.WebServicePropertiesForGraph
Id : /subscriptions/<subscription ID>/resourceGroups/Default-MachineLearning-SouthCentralUS/providers/Microsoft.MachineLearning/webServices/RetrainSamplePre.2016.8.17.0.3.51.237
Name : RetrainSamplePre.2016.8.17.0.3.51.237
Location : South Central US
Type : Microsoft.MachineLearning/webServices
Tags : {}
```

Pour déterminer le nom du groupe de ressources d’un service web existant, vous pouvez également vous connecter au portail des services web Azure Machine Learning. Sélectionnez le service web. Le nom de groupe de ressources est le cinquième élément de l’URL du service web, juste après l’élément *resourceGroups* . Dans l’exemple suivant, le nom du groupe de ressources est Default-MachineLearning-SouthCentralUS.

`https://services.azureml.net/subscriptions/<subscription ID>/resourceGroups/Default-MachineLearning-SouthCentralUS/providers/Microsoft.MachineLearning/webServices/RetrainSamplePre.2016.8.17.0.3.51.237`

### <a name="export-the-web-service-definition-object-as-json"></a>Exporter l’objet Définition du service web en tant que JSON

Pour modifier la définition du modèle formé de manière à utiliser le modèle nouvellement formé, vous devez d’abord utiliser l’applet de commande [Export-AzMlWebService](/powershell/module/az.machinelearning/export-azmlwebservice) pour l’exporter vers un fichier au format JSON.

```azurepowershell
Export-AzMlWebService -WebService $wsd -OutputFile "C:\temp\mlservice_export.json"
```

### <a name="update-the-reference-to-the-ilearner-blob"></a>Mettre à jour la référence à l’objet blob ilearner

Dans les ressources, recherchez le [modèle formé], mettez à jour la valeur *uri* dans le nœud *locationInfo* avec l’URI de l’objet blob ilearner. L’URI est générée en combinant les valeurs *BaseLocation* et *RelativeLocation* de la sortie de l’appel de reformation BES.

```json
"asset3": {
    "name": "Retrain Sample [trained model]",
    "type": "Resource",
    "locationInfo": {
        "uri": "https://mltestaccount.blob.core.windows.net/azuremlassetscontainer/baca7bca650f46218633552c0bcbba0e.ilearner"
    },
    "outputPorts": {
        "Results dataset": {
            "type": "Dataset"
        }
    }
},
```

### <a name="import-the-json-into-a-web-service-definition-object"></a>Importer le JSON dans un objet Définition du service web

Utilisez l’applet de commande [Import-AzMlWebService](/powershell/module/az.machinelearning/import-azmlwebservice) pour reconvertir le fichier JSON modifié en un objet de définition de service web permettant de mettre à jour l’expérience prédictive.

```azurepowershell
$wsd = Import-AzMlWebService -InputFile "C:\temp\mlservice_export.json"
```

### <a name="update-the-web-service"></a>Mise à jour du service web

Enfin, utilisez l’applet de commande [Update-AzMlWebService](/powershell/module/az.machinelearning/update-azmlwebservice) pour mettre à jour l’expérience prédictive.

```azurepowershell
Update-AzMlWebService -Name 'RetrainSamplePre.2016.8.17.0.3.51.237' -ResourceGroupName 'Default-MachineLearning-SouthCentralUS'
```

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment gérer les services web ou effectuer le suivi de plusieurs exécutions d’expériences, voir les articles suivants :

* [Explorer le portail Services web](manage-new-webservice.md)
* [Gérer des itérations d'expériences](manage-experiment-iterations.md)
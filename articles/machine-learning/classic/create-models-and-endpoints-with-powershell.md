---
title: 'ML Studio (classique) : Création de plusieurs modèles et points de terminaison – Azure'
description: Utilisez PowerShell pour créer plusieurs modèles de formation et points de terminaison de service web Machine Learning avec le même algorithme, mais différents jeux de données de formation.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18
ms.date: 04/04/2017
ms.openlocfilehash: ef9ea055f437b53313dc9ee11b0b91f095664f5e
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93322851"
---
# <a name="create-multiple-web-service-endpoints-from-one-experiment-with-ml-studio-classic-and-powershell"></a>Créer plusieurs points de terminaison de service web à partir d’une expérience avec ML Studio (classique) et PowerShell

**S’APPLIQUE À :**  ![S’applique à ](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Ne s’applique pas à ](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)

Voici un problème d’apprentissage automatique courant : vous souhaitez créer un grand nombre de modèles ayant le même flux de travail d’apprentissage et utilisant le même algorithme. Mais vous souhaitez qu’ils aient différents jeux de données d’apprentissage comme entrée. Cet article montre comment procéder dans Azure Machine Learning Studio (classique) à l’aide d’une expérience unique.

Supposez par exemple que vous avez une franchise de location de vélos à l’échelle internationale. Vous souhaitez créer un modèle de régression pour prédire la demande en location sur la base de données historiques. Vous avez 1 000 emplacements de location dans le monde et vous avez collecté un jeu de données pour chaque emplacement. Ils comprennent des caractéristiques importantes telles que la date, l’heure, la météo et le trafic qui sont propres à chaque emplacement.

Vous pourriez former votre modèle une fois à l’aide d’une version fusionnée de tous les jeux de données et de tous les emplacements. Toutefois, chacun de vos emplacements a un environnement unique. Une meilleure approche consiste donc à former le modèle de régression séparément à l’aide du jeu de données de chacun. Ainsi, chaque modèle formé peut prendre en compte les différences en termes de taille de magasin, de volume, de géographie, de population, de qualité de l’environnement de circulation pour les vélos, et ainsi de suite.

Cela pourrait être la meilleure approche, mais vous ne souhaitez pas créer 1 000 expériences d’apprentissage dans Azure Machine Learning Studio (classique) représentant chacune un emplacement unique. Cette tâche serait non seulement intensive mais également inefficace, dans la mesure où chaque expérience aurait les mêmes composants, à l’exception du jeu de données d’apprentissage.

Heureusement, vous pouvez obtenir le même résultat en utilisant l’[API de reformation Azure Machine Learning Studio (classique)](./retrain-machine-learning-model.md) et en automatisant la tâche avec [Azure Machine Learning Studio (classique) PowerShell](powershell-module.md).

> [!NOTE]
> Pour accélérer l’exécution de notre exemple, nous allons réduire le nombre d’emplacements de 1000 à 10, mais les mêmes principes et procédures sont valables pour 1 000 emplacements. Toutefois, si vous ne souhaitez pas effectuer l’apprentissage à partir de 1000 jeux de données, vous pouvez exécuter les scripts PowerShell suivants en parallèle. Cette opération sort du cadre de cet article, mais vous trouverez des exemples de multi-threading PowerShell sur Internet.  
> 
> 

## <a name="set-up-the-training-experiment"></a>Configurer l’expérience de formation
Utilisez l’exemple [d’expérience de formation](https://gallery.azure.ai/Experiment/Bike-Rental-Training-Experiment-1) qui se trouve dans la [Cortana Intelligence Gallery](https://gallery.azure.ai). Ouvrez cette expérience dans votre espace de travail [Azure Machine Learning Studio (classique)](https://studio.azureml.net) .

> [!NOTE]
> Pour suivre cet exemple, il est préférable d’utiliser un espace de travail standard plutôt qu’un espace de travail gratuit. Vous créez un point de terminaison pour chaque client (soit 10 points de terminaison en tout), ce qui nécessite un espace de travail standard car un espace de travail gratuit est limité à trois points de terminaison.
> 
> 

L’expérience utilise un module **Import Data** pour importer le jeu de données de formation *customer001.csv* à partir d’un compte de stockage Azure. Supposez que vous avez recueilli des jeux de données de formation à partir de tous les emplacements de location de vélos et que vous les avez stockés dans le même emplacement de stockage d’objets blob avec des noms de fichiers allant de *rentalloc001.csv* à *rentalloc10.csv*.

![Le module lecteur importe des données à partir d’un objet blob Azure](./media/create-models-and-endpoints-with-powershell/reader-module.png)

Notez qu’un module **Web Service Output** a été ajouté au module **Train Model**.
Quand cette expérience est déployée comme service web, le point de terminaison associé à cette sortie retourne le modèle formé au format de fichier .ilearner.

Notez également que vous configurez un paramètre de service web qui définit l’URL utilisée par le module **Import Data**. Cela vous permet d’utiliser le paramètre pour spécifier des jeux de données de formation individuels visant à former le modèle pour chaque emplacement.
Il existe d’autres façons de procéder. Vous pouvez utiliser une requête SQL avec un paramètre de service web pour obtenir des données à partir d’une base de données d’Azure SQL Database. Vous pouvez également utiliser un module **Web Service Input** pour transmettre un jeu de données au service web.

![Un module de modèle entraîné est transmis à un module de sortie du service web](./media/create-models-and-endpoints-with-powershell/web-service-output.png)

Exécutons maintenant cette expérience de formation à l’aide de la valeur par défaut *rental001.csv* comme jeu de données de formation. Si vous affichez la sortie du module **Evaluate** (cliquez sur la sortie et sélectionnez **Visualize** ), vous constatez que vous obtenez une performance correcte de *AUC* = 0,91. À ce stade, vous êtes prêts à déployer un service web à partir de cette expérience de formation.

## <a name="deploy-the-training-and-scoring-web-services"></a>Déployer les services web de formation et de notation
Pour déployer le service web de formation, cliquez sur le bouton **Set Up Web Service** (Configurer le service web) sous le canevas d’expérience et sélectionnez **Deploy Web Service** (Déployer le service web). Nommez ce service web « Bike Rental Training ».

Vous devez à présent déployer le service web de notation.
Pour cela, cliquez sur **Set Up Web Service** (Configurer le service web) sous le canevas et sélectionnez **Predictive Web Service** (Service web prédictif). Une expérience de notation est créée.
Vous devez effectuer quelques petits ajustements pour qu’elle fonctionne en tant que service web. Supprimez la colonne d’étiquette « cnt » des données d’entrée et limitez la sortie à l’ID d’instance et à la valeur prédite correspondante.

Pour vous éviter ce travail, vous pouvez ouvrir [l’expérience prédictive](https://gallery.azure.ai/Experiment/Bike-Rental-Predicative-Experiment-1) déjà préparée dans la galerie.

Pour déployer le service web, exécutez l’expérience prédictive, puis cliquez sur le bouton **Deploy Web Service** sous le canevas. Nommez le service web de scoring « Bike Rental Scoring ».

## <a name="create-10-identical-web-service-endpoints-with-powershell"></a>Créer 10 points de terminaison de service web identiques avec PowerShell
Ce service web est fourni avec un point de terminaison par défaut. Mais celui-ci ne nous intéresse pas, car il ne peut pas être mis à jour. Vous devez créer 10 points de terminaison supplémentaires, un pour chaque emplacement. Vous pouvez pour cela utiliser PowerShell.

Tout d’abord, vous configurez l’environnement PowerShell :

```powershell
Import-Module .\AzureMLPS.dll
# Assume the default configuration file exists and is properly set to point to the valid Workspace.
$scoringSvc = Get-AmlWebService | where Name -eq 'Bike Rental Scoring'
$trainingSvc = Get-AmlWebService | where Name -eq 'Bike Rental Training'
```

Ensuite, nous allons exécuter la commande PowerShell suivante :

```powershell
# Create 10 endpoints on the scoring web service.
For ($i = 1; $i -le 10; $i++){
    $seq = $i.ToString().PadLeft(3, '0');
    $endpointName = 'rentalloc' + $seq;
    Write-Host ('adding endpoint ' + $endpointName + '...')
    Add-AmlWebServiceEndpoint -WebServiceId $scoringSvc.Id -EndpointName $endpointName -Description $endpointName     
}
```

Vous avez maintenant créé 10 points de terminaison, qui contiennent tous le même modèle formé sur *customer001.csv*. Vous pouvez les afficher dans le portail Azure.

![Afficher la liste des modèles entraînés dans le portail](./media/create-models-and-endpoints-with-powershell/created-endpoints.png)

## <a name="update-the-endpoints-to-use-separate-training-datasets-using-powershell"></a>Mettre à jour les points de terminaison pour utiliser des jeux de données de formation distincts à l’aide de PowerShell
L’étape suivante consiste à mettre à jour les points de terminaison avec des modèles formés de manière unique d’après les données individuelles de chaque client. Mais tout d’abord, vous devez générer ces modèles à partir du service web **Bike Rental Training**. Revenons au service web **Bike Rental Training** . Vous devez appeler son point de terminaison BES 10 fois avec 10 jeux de données de formation différents pour générer 10 modèles différents. Pour cela, utilisez l’applet de commande PowerShell **InovkeAmlWebServiceBESEndpoint**.

Vous devez également fournir des informations d’identification pour votre compte de stockage d’objets blob dans `$configContent`, à savoir les champs `AccountName`, `AccountKey` et `RelativeLocation`. Le `AccountName` peut être l’un de vos noms de compte, comme illustré dans le **portail Azure** (onglet *Stockage* ). Lorsque vous cliquez sur un compte de stockage, sa `AccountKey` est accessible en appuyant sur le bouton **Gérer les clés d’accès** situé au bas de la page et en copiant la *clé d’accès primaire*. Le `RelativeLocation` est le chemin d’accès relatif à votre espace de stockage où sera stocké le nouveau modèle. Par exemple, le chemin `hai/retrain/bike_rental/` dans le script suivant pointe vers un conteneur nommé `hai`, et `/retrain/bike_rental/` sont des sous-dossiers. Actuellement, vous ne pouvez pas créer de sous-dossiers via le portail de l’interface utilisateur, mais il existe [plusieurs explorateurs de stockage Azure](../../storage/common/storage-explorers.md) qui permettent de le faire. Nous vous recommandons de créer un conteneur dans votre stockage pour stocker les nouveaux modèles formés (fichiers .ilearner) comme suit : à partir de votre page de stockage, cliquez sur le bouton **Ajouter** en bas et nommez-le `retrain`. En résumé, les modifications nécessaires au script suivant se rapportent à `AccountName`, `AccountKey` et `RelativeLocation` (:`"retrain/model' + $seq + '.ilearner"`).

```powershell
# Invoke the retraining API 10 times
# This is the default (and the only) endpoint on the training web service
$trainingSvcEp = (Get-AmlWebServiceEndpoint -WebServiceId $trainingSvc.Id)[0];
$submitJobRequestUrl = $trainingSvcEp.ApiLocation + '/jobs?api-version=2.0';
$apiKey = $trainingSvcEp.PrimaryKey;
For ($i = 1; $i -le 10; $i++){
    $seq = $i.ToString().PadLeft(3, '0');
    $inputFileName = 'https://bostonmtc.blob.core.windows.net/hai/retrain/bike_rental/BikeRental' + $seq + '.csv';
    $configContent = '{ "GlobalParameters": { "URI": "' + $inputFileName + '" }, "Outputs": { "output1": { "ConnectionString": "DefaultEndpointsProtocol=https;AccountName=<myaccount>;AccountKey=<mykey>", "RelativeLocation": "hai/retrain/bike_rental/model' + $seq + '.ilearner" } } }';
    Write-Host ('training regression model on ' + $inputFileName + ' for rental location ' + $seq + '...');
    Invoke-AmlWebServiceBESEndpoint -JobConfigString $configContent -SubmitJobRequestUrl $submitJobRequestUrl -ApiKey $apiKey
}
```

> [!NOTE]
> Le point de terminaison BES est le seul mode pris en charge pour cette opération. Vous ne pouvez pas utiliser RRE pour générer des modèles formés.
> 
> 

Comme vous pouvez le voir ci-dessus, au lieu de construire 10 fichiers json de configuration de travaux BES différents, vous créez la chaîne de configuration de manière dynamique. Ensuite, vous la fournissez au paramètre *jobConfigString* de l’applet de commande **InvokeAmlWebServceBESEndpoint**. Il est inutile de conserver une copie sur le disque.

Si tout se passe bien, après un certain temps vous devriez voir 10 fichiers .ilearner, de *model001.ilearner* à *model010.ilearner* , dans votre compte de stockage Azure. Vous pouvez maintenant mettre à jour les 10 points de terminaison de service web de notation avec ces modèles à l’aide de l’applet de commande PowerShell **AmlWebServiceEndpoint-correctif**. N’oubliez pas que vous pouvez modifier uniquement les points de terminaison autres que les points de terminaison par défaut que vous avez créés précédemment par programmation.

```powershell
# Patch the 10 endpoints with respective .ilearner models
$baseLoc = 'http://bostonmtc.blob.core.windows.net/'
$sasToken = '<my_blob_sas_token>'
For ($i = 1; $i -le 10; $i++){
    $seq = $i.ToString().PadLeft(3, '0');
    $endpointName = 'rentalloc' + $seq;
    $relativeLoc = 'hai/retrain/bike_rental/model' + $seq + '.ilearner';
    Write-Host ('Patching endpoint ' + $endpointName + '...');
    Patch-AmlWebServiceEndpoint -WebServiceId $scoringSvc.Id -EndpointName $endpointName -ResourceName 'Bike Rental [trained model]' -BaseLocation $baseLoc -RelativeLocation $relativeLoc -SasBlobToken $sasToken
}
```

L’exécution devrait être assez rapide. Une fois l’exécution terminée, vous aurez créé 10 points de terminaison de service web prédictifs. Chacun d’eux contiendra un modèle formé de façon unique sur le jeu de données propre à un emplacement de location, tout ceci à partir d’une expérience de formation unique. Pour vérifier cela, vous pouvez essayer d’appeler ces points de terminaison à l’aide de l’applet de commande **InvokeAmlWebServiceRRSEndpoint** , en leur fournissant les mêmes données d’entrée. Vous devriez obtenir des résultats de prédiction différents, étant donné que les modèles sont formés avec des jeux d’apprentissage différents.

## <a name="full-powershell-script"></a>Script PowerShell complet
Voici l’intégralité du code source :

```powershell
Import-Module .\AzureMLPS.dll
# Assume the default configuration file exists and properly set to point to the valid workspace.
$scoringSvc = Get-AmlWebService | where Name -eq 'Bike Rental Scoring'
$trainingSvc = Get-AmlWebService | where Name -eq 'Bike Rental Training'

# Create 10 endpoints on the scoring web service
For ($i = 1; $i -le 10; $i++){
    $seq = $i.ToString().PadLeft(3, '0');
    $endpointName = 'rentalloc' + $seq;
    Write-Host ('adding endpoint ' + $endpontName + '...')
    Add-AmlWebServiceEndpoint -WebServiceId $scoringSvc.Id -EndpointName $endpointName -Description $endpointName     
}

# Invoke the retraining API 10 times to produce 10 regression models in .ilearner format
$trainingSvcEp = (Get-AmlWebServiceEndpoint -WebServiceId $trainingSvc.Id)[0];
$submitJobRequestUrl = $trainingSvcEp.ApiLocation + '/jobs?api-version=2.0';
$apiKey = $trainingSvcEp.PrimaryKey;
For ($i = 1; $i -le 10; $i++){
    $seq = $i.ToString().PadLeft(3, '0');
    $inputFileName = 'https://bostonmtc.blob.core.windows.net/hai/retrain/bike_rental/BikeRental' + $seq + '.csv';
    $configContent = '{ "GlobalParameters": { "URI": "' + $inputFileName + '" }, "Outputs": { "output1": { "ConnectionString": "DefaultEndpointsProtocol=https;AccountName=<myaccount>;AccountKey=<mykey>", "RelativeLocation": "hai/retrain/bike_rental/model' + $seq + '.ilearner" } } }';
    Write-Host ('training regression model on ' + $inputFileName + ' for rental location ' + $seq + '...');
    Invoke-AmlWebServiceBESEndpoint -JobConfigString $configContent -SubmitJobRequestUrl $submitJobRequestUrl -ApiKey $apiKey
}

# Patch the 10 endpoints with respective .ilearner models
$baseLoc = 'http://bostonmtc.blob.core.windows.net/'
$sasToken = '?test'
For ($i = 1; $i -le 10; $i++){
    $seq = $i.ToString().PadLeft(3, '0');
    $endpointName = 'rentalloc' + $seq;
    $relativeLoc = 'hai/retrain/bike_rental/model' + $seq + '.ilearner';
    Write-Host ('Patching endpoint ' + $endpointName + '...');
    Patch-AmlWebServiceEndpoint -WebServiceId $scoringSvc.Id -EndpointName $endpointName -ResourceName 'Bike Rental [trained model]' -BaseLocation $baseLoc -RelativeLocation $relativeLoc -SasBlobToken $sasToken
}
```
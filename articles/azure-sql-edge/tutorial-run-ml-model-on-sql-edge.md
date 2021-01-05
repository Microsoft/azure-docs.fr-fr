---
title: Déployer un modèle Machine Learning sur Azure SQL Edge à l'aide d'ONNX
description: Dans la troisième partie de ce tutoriel Azure SQL Edge en trois parties consacré à la prédiction des impuretés contenues dans le minerai de fer, vous allez exécuter les modèles Machine Learning ONNX sur SQL Edge.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: tutorial
author: VasiyaKrishnan
ms.author: vakrishn
ms.reviewer: sstein
ms.date: 05/19/2020
ms.openlocfilehash: 9e5bb037b88b7c370e31d05c2d20fc6f558a8b39
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93422193"
---
# <a name="deploy-ml-model-on-azure-sql-edge-using-onnx"></a>Déployer un modèle Machine Learning sur Azure SQL Edge à l'aide d'ONNX 

Dans la troisième partie de ce tutoriel en trois parties consacré à la prédiction des impuretés contenues dans le minerai de fer dans Azure SQL Edge, vous allez :

1. utiliser Azure Data Studio pour vous connecter à SQL Database dans l'instance d'Azure SQL Edge ;
2. prédire les impuretés contenues dans le minerai de fer à l'aide d'ONNX dans Azure SQL Edge.

## <a name="key-components"></a>Composants clés

1. La solution utilise une valeur par défaut de 500 millisecondes entre chaque message envoyé au hub Edge. Vous pouvez la changer dans le fichier **Program.cs**. 
   ```json
   TimeSpan messageDelay = configuration.GetValue("MessageDelay", TimeSpan.FromMilliseconds(500));
   ```
2. La solution a généré un message, avec les attributs suivants. Ajoutez ou supprimez les attributs en fonction des spécifications. 
```json
{
    timestamp 
    cur_Iron_Feed
    cur_Silica_Feed 
    cur_Starch_Flow 
    cur_Amina_Flow 
    cur_Ore_Pulp_pH
    cur_Flotation_Column_01_Air_Flow
    cur_Flotation_Column_02_Air_Flow
    cur_Flotation_Column_03_Air_Flow
    cur_Flotation_Column_04_Air_Flow
    cur_Flotation_Column_01_Level
    cur_Flotation_Column_02_Level
    cur_Flotation_Column_03_Level
    cur_Flotation_Column_04_Level
    cur_Iron_Concentrate
}
```

## <a name="connect-to-the-sql-database-in-the-azure-sql-edge-instance-to-train-deploy-and-test-the-ml-model"></a>Se connecter à SQL Database dans l’instance Azure SQL Edge pour entraîner, déployer et tester le modèle ML

1. Ouvrez Azure Data Studio.

2. Sous l'onglet **Bienvenue**, lancez une nouvelle connexion avec les informations suivantes :

   |_Champ_|_Valeur_|
   |-------|-------|
   |Type de connexion| Microsoft SQL Server|
   |Serveur|Adresse IP publique mentionnée dans la machine virtuelle créée pour cette démo|
   |Nom d’utilisateur|SA|
   |Mot de passe|Mot de passe fort utilisé lors de la création de l'instance d'Azure SQL Edge|
   |Base de données|Default|
   |Groupe de serveurs|Default|
   |Name (facultatif)|Entrez un nom facultatif|

3. Cliquez sur **Connexion**

4. Dans la section **Fichier**, ouvrez **/DeploymentScripts/MiningProcess_ONNX.jpynb** à partir du dossier où vous avez cloné les fichiers projet sur votre ordinateur.

5. Définissez le noyau sur Python 3.


## <a name="next-steps"></a>Étapes suivantes

Pour plus d'informations sur l'utilisation des modèles ONNX dans Azure SQL Edge, consultez [Machine Learning et IA avec ONNX dans SQL Edge](onnx-overview.md).
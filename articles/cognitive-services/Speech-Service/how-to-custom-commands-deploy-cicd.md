---
title: Déploiement continu avec Azure DevOps (préversion)
titleSuffix: Azure Cognitive Services
description: Dans cet article, vous allez apprendre à configurer un déploiement continu pour vos applications Commandes personnalisées. Vous créez les scripts pour prendre en charge les workflows de déploiement continu.
services: cognitive-services
author: xiaojul
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: xiaojul
ms.openlocfilehash: 6dda50a443babca88a0a650fde60df0744d0a1d2
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95021828"
---
# <a name="continuous-deployment-with-azure-devops"></a>Déploiement continu avec Azure DevOps

Dans cet article, vous allez apprendre à configurer un déploiement continu pour vos applications Commandes personnalisées. Les scripts pour prendre en charge le workflow CI/CD vous sont fournis.

## <a name="prerequisite"></a>Configuration requise
> [!div class = "checklist"]
> * Une application Commandes personnalisées pour le développement (DEV)
> * Une application Commandes personnalisées pour la production (PROD)
> * S’inscrire à [Azure Pipelines](/azure/devops/pipelines/get-started/pipelines-sign-up?view=azure-devops)

## <a name="exportimportpublish"></a>Exportation/importation/publication

Les scripts sont hébergés dans [Assistant vocal Cognitive Services - Commandes personnalisées](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant/tree/master/custom-commands). Clonez les scripts du répertoire bash dans votre référentiel. Veillez à conserver le même chemin d’accès.

### <a name="set-up-a-pipeline"></a>Configurer un pipeline 

1. Accédez à **Azure DevOps - Pipelines** et cliquez sur « New Pipeline » (Nouveau pipeline)
1. Dans la section **Connect** (Connexion), sélectionnez l’emplacement de votre référentiel dans lequel se trouvent ces scripts
1. Dans la section **Select** (Sélection), sélectionnez votre référentiel
1. Dans la section **Configure** (Configuration), sélectionnez « Starter pipeline » (Pipeline de démarrage)
1. Vous obtenez alors un éditeur avec un fichier YAML, et remplacez la section « steps » (étapes) par ce script.

    ```YAML
    steps:
    - task: Bash@3
      displayName: 'Export source app'
      inputs:
        targetType: filePath
        filePath: ./bash/export.sh
        arguments: '-r westus2 -s $(SubscriptionKey) -c $(Culture) -a $(SourceAppId) -f ExportedDialogModel.json'
        workingDirectory: bash
        failOnStderr: true
    
    - task: Bash@3
      displayName: 'Import to target app'
      inputs:
        targetType: filePath
        filePath: ./bash/import.sh
        arguments: '-r westus2 -s $(SubscriptionKey) -c $(Culture) -a $(TargetAppId) -f ExportedDialogModel.json'
        workingDirectory: bash
        failOnStderr: true
    
    - task: Bash@3
      displayName: 'Train and Publish target app'
      inputs:
        targetType: filePath
        filePath: './bash/train-and-publish.sh'
        arguments: '-r westus2 -s $(SubscriptionKey) -c $(Culture) -a $(TargetAppId)'
        workingDirectory: bash
        failOnStderr: true
    ```
    
1. Notez que ces scripts supposent que vous utilisez la région `westus2` ; si ce n’est pas le cas, mettez à jour les arguments des tâches en conséquence

    > [!div class="mx-imgBorder"]
    > ![Screenshot that highlights the region value in the arguments.](media/custom-commands/cicd-new-pipeline-yaml.png)

1. Dans le bouton « Enregistrer et exécuter », ouvrez la liste déroulante et cliquez sur « Enregistrer »

### <a name="hook-up-the-pipeline-with-your-application"></a>Raccorder le pipeline à votre application

1. Accédez à la page principale du pipeline.
1. Dans la liste déroulante dans l’angle supérieur droit, sélectionnez **Modifier le pipeline**. Elle vous permet d’accéder à un éditeur YAML. 
1. Dans l’angle supérieur droit en regard du bouton « Exécuter », sélectionnez **Variables**. Cliquez sur **Nouvelle variable**.
1. Ajoutez ces variables :
    
    | Variable | Description |
    | ------- | --------------- | ----------- |
    | SourceAppId | ID de l’application DEV |
    | TargetAppId | ID de l’application PROD |
    | SubscriptionKey | Clé d’abonnement utilisée pour les deux applications |
    | Culture | Culture des applications (par exemple, en-US) |

    > [!div class="mx-imgBorder"]
    > ![Charge utile d’activité d’envoi](media/custom-commands/cicd-edit-pipeline-variables.png)

1. Cliquez sur « Exécuter », puis sur le « travail » en cours d’exécution. 

    Vous devriez voir une liste des tâches en cours d’exécution contenant les éléments suivants : « Export source app », « Import to target app » et « Train and Publish target app »

## <a name="deploy-from-source-code"></a>Déployer à partir de code source

Si vous souhaitez conserver la définition de votre application dans un référentiel, nous fournissons les scripts pour les déploiements à partir du code source. Comme que les scripts sont de type bash, si vous utilisez Windows, vous devez installer le sous-système [Linux](/windows/wsl/install-win10).

Les scripts sont hébergés dans [Assistant vocal Cognitive Services - Commandes personnalisées](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant/tree/master/custom-commands). Clonez les scripts du répertoire bash dans votre référentiel. Veillez à conserver le même chemin d’accès.

### <a name="prepare-your-repository"></a>Préparer votre dépôt

1. Créez un répertoire pour votre application ; dans notre exemple, créez-en un nommé « apps ».
1. Mettez à jour les arguments du script bash ci-dessous, puis exécutez le script. Le modèle de boîte de dialogue de votre application est importé dans le fichier myapp.json
    ```BASH
    bash/export.sh -r <region> -s <subscriptionkey> -c en-us -a <appid> -f apps/myapp.json
    ```
    | Arguments | Description |
    | ------- | --------------- | ----------- |
    | region | région de l’application, soit westus2. |
    | subscriptionkey | clé d’abonnement de votre ressource Speech. |
    | appid | ID de l’application Commandes personnalisées que vous souhaitez exporter. |

1. Envoyez ces modifications à votre référentiel.

### <a name="set-up-a-pipeline"></a>Configurer un pipeline 

1. Accédez à **Azure DevOps - Pipelines** et cliquez sur « New Pipeline » (Nouveau pipeline)
1. Dans la section **Connect** (Connexion), sélectionnez l’emplacement de votre référentiel dans lequel se trouvent ces scripts
1. Dans la section **Select** (Sélection), sélectionnez votre référentiel
1. Dans la section **Configure** (Configuration), sélectionnez « Starter pipeline » (Pipeline de démarrage)
1. Vous obtenez alors un éditeur avec un fichier YAML, et remplacez la section « steps » (étapes) par ce script.

    ```YAML
    steps:
    - task: Bash@3
      displayName: 'Import app'
      inputs:
        targetType: filePath
        filePath: ./bash/import.sh
        arguments: '-r westus2 -s $(SubscriptionKey) -c $(Culture) -a $(TargetAppId) -f ../apps/myapp.json'
        workingDirectory: bash
        failOnStderr: true
    
    - task: Bash@3
      displayName: 'Train and Publish app'
      inputs:
        targetType: filePath
        filePath: './bash/train-and-publish.sh'
        arguments: '-r westus2 -s $(SubscriptionKey) -c $(Culture) -a $(TargetAppId)'
        workingDirectory: bash
        failOnStderr: true
    ```

    > [!NOTE]
    > ces scripts supposent que vous utilisez la région westus2 ; si ce n’est pas le cas, mettez à jour les arguments des tâches en conséquence

1. Dans le bouton « Enregistrer et exécuter », ouvrez la liste déroulante et cliquez sur « Enregistrer »

### <a name="hook-up-the-pipeline-with-your-target-applications"></a>Raccorder le pipeline à vos applications cibles

1. Accédez à la page principale du pipeline.
1. Dans la liste déroulante dans l’angle supérieur droit, sélectionnez **Modifier le pipeline**. Elle vous permet d’accéder à un éditeur YAML. 
1. Dans l’angle supérieur droit en regard du bouton « Exécuter », sélectionnez **Variables**. Cliquez sur **Nouvelle variable**.
1. Ajoutez ces variables :

    | Variable | Description |
    | ------- | --------------- | ----------- |
    | TargetAppId | ID de l’application PROD |
    | SubscriptionKey | Clé d’abonnement utilisée pour les deux applications |
    | Culture | Culture des applications (par exemple, en-US) |

1. Cliquez sur « Exécuter », puis sur le « travail » en cours d’exécution.
    Vous devriez voir une liste des tâches en cours d’exécution contenant les éléments suivants : « Import app » et « Train and Publish app »

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Voir des exemples sur GitHub](https://aka.ms/speech/cc-samples)
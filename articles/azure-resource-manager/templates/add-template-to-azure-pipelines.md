---
title: CI/CD avec Azure Pipelines et des modèles
description: Décrit comment configurer l’intégration continue dans Azure Pipelines à l’aide de modèles Azure Resource Manager. Montre comment utiliser un script PowerShell ou copier des fichiers vers un emplacement intermédiaire et le déployer à partir de là.
ms.topic: conceptual
ms.date: 10/01/2020
ms.openlocfilehash: 86ad2839375b73bf9595cf3369960e614ec03e67
ms.sourcegitcommit: bbd66b477d0c8cb9adf967606a2df97176f6460b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93233812"
---
# <a name="integrate-arm-templates-with-azure-pipelines"></a>Intégrer des modèles ARM avec Azure Pipelines

Vous pouvez intégrer des modèles Azure Resource Manager (modèles ARM) avec Azure Pipelines pour l’intégration continue et le déploiement continu (CI/CD). Le tutoriel [Intégration continue des modèles ARM avec Azure Pipelines](deployment-tutorial-pipeline.md) montre comment utiliser la [tâche de déploiement ARM](https://github.com/microsoft/azure-pipelines-tasks/blob/master/Tasks/AzureResourceManagerTemplateDeploymentV3/README.md) pour déployer un modèle à partir de votre dépôt GitHub. Cette approche fonctionne lorsque vous souhaitez déployer un modèle directement à partir d’un dépôt.

Dans cet article, vous allez apprendre deux autres façons de déployer des modèles avec Azure Pipelines. Cet article montre comment :

* **Ajouter une tâche exécutant un script Azure PowerShell**. Cette option présente l’avantage de fournir une cohérence tout au long du cycle de vie du développement, car vous pouvez utiliser le script que vous avez utilisé lors de l’exécution des tests locaux. Votre script déploie le modèle, mais peut également exécuter d’autres opérations telles que l’obtention de valeurs à utiliser en tant que paramètres.

   Visual Studio fournit le [projet de groupe de ressources Azure](create-visual-studio-deployment-project.md) qui inclut un script PowerShell. Le script organise les artefacts de votre projet comme un compte de stockage auquel Resource Manager peut accéder. Les artefacts sont des éléments dans votre projet comme les modèles liés, les scripts et les fichiers binaires d’application. Si vous souhaitez continuer à utiliser le script à partir du projet, utilisez la tâche de script PowerShell présentée dans cet article.

* **Ajouter des tâches pour copier et déployer des tâches**. Cette deuxième option est une alternative pratique au script de projet. Vous configurez deux tâches dans le pipeline. Une tâche importe les artefacts vers un emplacement accessible. L’autre tâche déploie le modèle à partir de cet emplacement.

## <a name="prepare-your-project"></a>Préparation du projet

Cet article suppose que votre modèle ARM et votre organisation Azure DevOps sont prêts pour la création du pipeline. Les étapes suivantes montrent comment vérifier que vous êtes prêt :

* Vous disposez d’une organisation Azure DevOps. Si vous n’en avez pas, [créez-en une gratuitement](/azure/devops/pipelines/get-started/pipelines-sign-up). Si votre équipe a déjà une organisation Azure DevOps, assurez-vous d’être un administrateur du projet Azure DevOps que vous souhaitez utiliser.

* Vous avez configuré une [connexion de service](/azure/devops/pipelines/library/connect-to-azure) pour votre abonnement Azure. Les tâches dans le pipeline s’exécutent sous l’identité du principal du service. Pour obtenir des instructions sur la création d’une connexion, consultez [Créer un projet DevOps](deployment-tutorial-pipeline.md#create-a-devops-project).

* Vous disposez d’un [modèle ARM](quickstart-create-templates-use-visual-studio-code.md) qui définit l’infrastructure de votre projet.

## <a name="create-pipeline"></a>Création d’un pipeline

1. Si vous n’avez pas ajouté un pipeline précédemment, vous devez en créer un nouveau. À partir de votre organisation Azure DevOps, sélectionnez **Pipelines** et **Nouveau pipeline**.

   ![Ajouter un nouveau pipeline](./media/add-template-to-azure-pipelines/new-pipeline.png)

1. Spécifiez où votre code est stocké. L’illustration suivante montre la sélection du **Git Azure Repos**.

   ![Sélectionner une source de code](./media/add-template-to-azure-pipelines/select-source.png)

1. À partir de cette source, sélectionnez le référentiel contenant le code de votre projet.

   ![Sélectionner un référentiel](./media/add-template-to-azure-pipelines/select-repo.png)

1. Sélectionnez le type de pipeline à créer. Vous pouvez sélectionner **Pipeline de démarrage**.

   ![Sélectionner un pipeline](./media/add-template-to-azure-pipelines/select-pipeline.png)

Vous êtes prêt à ajouter une tâche Azure PowerShell ou le fichier de copie et à déployer des tâches.

## <a name="azure-powershell-task"></a>Tâche Azure PowerShell

Cette section montre comment configurer le déploiement continu à l’aide d’une seule tâche qui exécute le script PowerShell dans votre projet. Si vous avez besoin d’un script PowerShell qui déploie un modèle, consultez [Deploy-AzTemplate.ps1](https://github.com/Azure/azure-quickstart-templates/blob/master/Deploy-AzTemplate.ps1) ou [Deploy-AzureResourceGroup.ps1](https://github.com/Azure/azure-quickstart-templates/blob/master/Deploy-AzureResourceGroup.ps1).

Le fichier YAML suivant crée une [tâche Azure PowerShell](/azure/devops/pipelines/tasks/deploy/azure-powershell):

```yml
trigger:
- master

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: AzurePowerShell@5
  inputs:
    azureSubscription: 'script-connection'
    ScriptType: 'FilePath'
    ScriptPath: './Deploy-AzTemplate.ps1'
    ScriptArguments: -Location 'centralus' -ResourceGroupName 'demogroup' -TemplateFile templates\mainTemplate.json
    azurePowerShellVersion: 'LatestVersion'
```

Lorsque vous définissez la tâche sur `AzurePowerShell@5`, le pipeline utilise le [module AZ](/powershell/azure/new-azureps-module-az). Si vous utilisez le module AzureRM dans votre script, définissez la tâche sur `AzurePowerShell@3`.

```yaml
steps:
- task: AzurePowerShell@3
```

Pour `azureSubscription`, indiquez le nom de la connexion de service créée.

```yaml
inputs:
    azureSubscription: '<your-connection-name>'
```

Pour `scriptPath`, fournissez le chemin d’accès relatif à partir du fichier de pipeline vers votre script. Vous pouvez rechercher dans votre référentiel pour afficher le chemin d’accès.

```yaml
ScriptPath: '<your-relative-path>/<script-file-name>.ps1'
```

Dans `ScriptArguments`, fournissez les paramètres nécessaires à votre script. L’exemple suivant montre quelques paramètres pour un script, mais vous devez personnaliser les paramètres de votre script.

```yaml
ScriptArguments: -Location 'centralus' -ResourceGroupName 'demogroup' -TemplateFile templates\mainTemplate.json
```

Lorsque vous sélectionnez **Enregistrer** , le pipeline de build s’exécute automatiquement. Revenez au résumé de votre pipeline de build et regardez l’état.

![Afficher les résultats](./media/add-template-to-azure-pipelines/view-results.png)

Vous pouvez sélectionner le pipeline en cours d’exécution pour afficher plus d’informations sur les tâches. Lorsqu’il se termine, vous consultez les résultats pour chaque étape.

## <a name="copy-and-deploy-tasks"></a>Tâches de copie et de déploiement

Cette section montre comment configurer un déploiement continu à l’aide de deux tâches. La première tâche importe les artefacts dans un compte de stockage et la deuxième tâche déploie le modèle.

Pour copier des fichiers dans un compte de stockage, le principal du service pour la connexion au service doit se voir attribuer le rôle Contributeur aux données Blob du stockage ou Propriétaire des données Blob du stockage. Pour plus d’informations, consultez [Bien démarrer avec AzCopy](../../storage/common/storage-use-azcopy-v10.md).

Le fichier YAML suivant montre la [tâche de copie de fichiers Azure](/azure/devops/pipelines/tasks/deploy/azure-file-copy).

```yml
trigger:
- master

pool:
  vmImage: 'windows-latest'

steps:
- task: AzureFileCopy@4
  inputs:
    SourcePath: 'templates'
    azureSubscription: 'copy-connection'
    Destination: 'AzureBlob'
    storage: 'demostorage'
    ContainerName: 'projecttemplates'
  name: AzureFileCopy
```

Il existe plusieurs parties de cette tâche à réviser pour votre environnement. `SourcePath` indique l’emplacement des artefacts relatifs au fichier de pipeline.

```yaml
SourcePath: '<path-to-artifacts>'
```

Pour `azureSubscription`, indiquez le nom de la connexion de service créée.

```yaml
azureSubscription: '<your-connection-name>'
```

Pour le nom du stockage et du conteneur, fournissez les noms du compte de stockage et du conteneur que vous souhaitez utiliser pour stocker les artefacts. Le compte de stockage doit exister.

```yaml
storage: '<your-storage-account-name>'
ContainerName: '<container-name>'
```

Après avoir créé la tâche Copier le fichier, vous êtes prêt à ajouter la tâche pour déployer le modèle intermédiaire.

Le YAML suivant montre la [tâche de déploiement du modèle Azure Resource Manager](https://github.com/microsoft/azure-pipelines-tasks/blob/master/Tasks/AzureResourceManagerTemplateDeploymentV3/README.md) :

```yaml
- task: AzureResourceManagerTemplateDeployment@3
  inputs:
    deploymentScope: 'Resource Group'
    azureResourceManagerConnection: 'copy-connection'
    subscriptionId: '00000000-0000-0000-0000-000000000000'
    action: 'Create Or Update Resource Group'
    resourceGroupName: 'demogroup'
    location: 'West US'
    templateLocation: 'URL of the file'
    csmFileLink: '$(AzureFileCopy.StorageContainerUri)templates/mainTemplate.json$(AzureFileCopy.StorageContainerSasToken)'
    csmParametersFileLink: '$(AzureFileCopy.StorageContainerUri)templates/mainTemplate.parameters.json$(AzureFileCopy.StorageContainerSasToken)'
    deploymentMode: 'Incremental'
    deploymentName: 'deploy1'
```

Il existe plusieurs parties de cette tâche à examiner plus en détail.

- `deploymentScope` : sélectionnez l’étendue du déploiement à l’aide des options `Management Group`, `Subscription` et `Resource Group`. Pour en savoir plus sur les étendues, consultez [Étendues de déploiement](deploy-rest.md#deployment-scope).

- `azureResourceManagerConnection`: Indiquez le nom de la connexion de service créée.

- `subscriptionId`: Spécifiez l’ID de l’abonnement cible. Cette propriété s’applique uniquement à l’étendue de déploiement du groupe de ressources et du déploiement de l’abonnement.

- `resourceGroupName` et `location` : indiquez le nom et l’emplacement du groupe de ressources au sein duquel vous souhaitez faire le déploiement. La tâche crée le groupe de ressources s’il n’existe pas.

   ```yml
   resourceGroupName: '<resource-group-name>'
   location: '<location>'
   ```

- `csmFileLink` : indiquez le lien pour le modèle intermédiaire. Lors de la définition de la valeur, utilisez les variables retournées à partir de la tâche de copie de fichiers. L’exemple suivant établit un lien vers un modèle nommé mainTemplate.json. Le dossier nommé **templates** est inclus, car c’est ici que se trouve la tâche de copie de fichiers dans laquelle le fichier a été copié. Dans votre pipeline, indiquez le chemin d’accès et le nom de votre modèle.

   ```yml
   csmFileLink: '$(AzureFileCopy.StorageContainerUri)templates/mainTemplate.json$(AzureFileCopy.StorageContainerSasToken)'
   ```

Votre pipeline ressemble à ce qui suit :

```yml
trigger:
- master

pool:
  vmImage: 'windows-latest'

steps:
- task: AzureFileCopy@4
  inputs:
    SourcePath: 'templates'
    azureSubscription: 'copy-connection'
    Destination: 'AzureBlob'
    storage: 'demostorage'
    ContainerName: 'projecttemplates'
  name: AzureFileCopy
- task: AzureResourceManagerTemplateDeployment@3
  inputs:
    deploymentScope: 'Resource Group'
    azureResourceManagerConnection: 'copy-connection'
    subscriptionId: '00000000-0000-0000-0000-000000000000'
    action: 'Create Or Update Resource Group'
    resourceGroupName: 'demogroup'
    location: 'West US'
    templateLocation: 'URL of the file'
    csmFileLink: '$(AzureFileCopy.StorageContainerUri)templates/mainTemplate.json$(AzureFileCopy.StorageContainerSasToken)'
    csmParametersFileLink: '$(AzureFileCopy.StorageContainerUri)templates/mainTemplate.parameters.json$(AzureFileCopy.StorageContainerSasToken)'
    deploymentMode: 'Incremental'
    deploymentName: 'deploy1'
```

Lorsque vous sélectionnez **Enregistrer** , le pipeline de build s’exécute automatiquement. Revenez au résumé de votre pipeline de build et regardez l’état.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’utilisation des modèles ARM avec GitHub Actions, consultez [Déployer des modèles Azure Resource Manager à l’aide de GitHub Actions](deploy-github-actions.md).

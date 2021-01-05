---
title: DevOps pour un pipeline d’ingestion des données
titleSuffix: Azure Machine Learning
description: Découvrez comment appliquer des pratiques DevOps pour créer un pipeline d’ingestion des données et préparer des données à l’aide d’Azure Data Factory et Azure Databricks.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, devx-track-python, data4ml
ms.author: iefedore
author: eedorenko
manager: davete
ms.reviewer: larryfr
ms.date: 06/23/2020
ms.openlocfilehash: fe2f35708f6a148f8db9ef6fd0a598e19e746fbd
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93358624"
---
# <a name="devops-for-a-data-ingestion-pipeline"></a>DevOps pour un pipeline d’ingestion des données

Dans la plupart des scénarios, une solution d’ingestion des données est une composition de scripts, d’appels de service et d’un pipeline qui orchestre toutes les activités. Dans cet article, vous découvrez comment appliquer les pratiques DevOps au cycle de vie de développement d’un pipeline commun d’ingestion des données qui prépare les données pour l’entraînement du modèle Machine Learning. Le pipeline est construit en utilisant les services Azure suivants :

* __Azure Data Factory__ : Lit les données brutes et orchestre la préparation des données.
* __Azure Databricks__  : Exécute un notebook Python qui transforme les données.
* __Azure Pipelines__  : Automatise un processus de développement et d’intégration continus.

## <a name="data-ingestion-pipeline-workflow"></a>Workflow du pipeline d’ingestion des données

Le pipeline d’ingestion des données implémente le workflow suivant :

1. Les données brutes sont lues dans un pipeline Azure Data Factory (ADF).
1. Le pipeline ADF envoie les données à un cluster Azure Databricks, qui exécute un notebook Python pour transformer les données.
1. Les données sont stockées dans un conteneur d’objets Blob, où elles peuvent être utilisées par Azure Machine Learning pour entraîner un modèle.

![Workflow du pipeline d’ingestion des données](media/how-to-cicd-data-ingestion/data-ingestion-pipeline.png)

## <a name="continuous-integration-and-delivery-overview"></a>Vue d’ensemble de l’intégration et de la livraison continues

Comme pour beaucoup de solutions logicielles, une équipe (par exemple des ingénieurs des données) travaille sur celle-ci. Les membres de cette équipe collaborent et partagent les mêmes ressources Azure, comme Azure Data Factory, Azure Databricks et des comptes Stockage Azure. La collection de ces ressources est un environnement de développement. Les ingénieurs de données contribuent à la même base de code source.

Un système de livraison et d’intégration continues automatise le processus de création, de test et de livraison (déploiement) de la solution. Le processus d’intégration continue (CI) effectue les tâches suivantes :

* Il assemble le code
* Il le vérifie avec des tests de qualité du code
* Il effectue des tests unitaires
* Il produit des artefacts comme du code testé et des modèles Azure Resource Manager

Le processus de déploiement continu (CD) déploie les artefacts dans les environnements en aval.

![Diagramme d’ingestion des données CI/CD](media/how-to-cicd-data-ingestion/cicd-data-ingestion.png)

Cet article explique comment automatiser les processus d’intégration continue et de déploiement continu grâce à [Azure Pipelines](https://azure.microsoft.com/services/devops/pipelines/).

## <a name="source-control-management"></a>Gestion du contrôle de code source

La gestion du contrôle de code source est nécessaire pour effectuer le suivi des modifications et permettre la collaboration entre les membres de l’équipe.
Par exemple, le code est stocké dans un dépôt Azure DevOps, GitHub ou GitLab. Le workflow de collaboration est basé sur un modèle de gestion de branches. Par exemple, [GitFlow](https://datasift.github.io/gitflow/IntroducingGitFlow.html).

### <a name="python-notebook-source-code"></a>Code source du notebook Python

Les ingénieurs de données utilisent le code source du notebook Python localement dans un IDE (par exemple, [Visual Studio Code](https://code.visualstudio.com)) ou directement dans l’espace de travail Databricks. Une fois les modifications de code terminées, elles sont fusionnées dans le dépôt selon une stratégie de gestion des branches.

> [!TIP] 
> Nous vous recommandons de stocker le code dans des fichiers `.py` plutôt qu’au format Jupyter Notebook `.ipynb`. Cela améliore la lisibilité du code et active les vérifications automatiques de la qualité du code dans le processus d’intégration continue.

### <a name="azure-data-factory-source-code"></a>Code source Azure Data Factory

Le code source des pipelines Azure Data Factory est une collection de fichiers JSON générés par un espace de travail Azure Data Factory. Normalement, les ingénieurs de données travaillent avec un concepteur visuel dans l’espace de travail Azure Data Factory plutôt qu’avec les fichiers de code source directement. 

Pour configurer l’espace de travail pour utiliser un dépôt de contrôle de code source, consultez [Créer avec l’intégration Azure Repos Git](../data-factory/source-control.md#author-with-azure-repos-git-integration).   

## <a name="continuous-integration-ci"></a>Intégration continue (CI)

L’objectif ultime du processus d’intégration continue est de rassembler le travail de l’équipe à partir du code source et de le préparer pour le déploiement dans les environnements en aval. Comme pour la gestion du code source, ce processus est différent pour les notebooks Python et les pipelines Azure Data Factory. 

### <a name="python-notebook-ci"></a>CI de notebooks Python

Le processus CI des notebooks Python récupère le code de la branche de collaboration (par exemple, * **maître** _ ou _*_développement_*_ ) et effectue les activités suivantes : _ Linting de code
* Test des unités
* Enregistrement du code en tant qu’artefact

L’extrait de code suivant illustre l’implémentation de ces étapes dans un pipeline * **yaml** _ Azure DevOps :

```yaml
steps:
- script: |
   flake8 --output-file=$(Build.BinariesDirectory)/lint-testresults.xml --format junit-xml  
  workingDirectory: '$(Build.SourcesDirectory)'
  displayName: 'Run flake8 (code style analysis)'  
  
- script: |
   python -m pytest --junitxml=$(Build.BinariesDirectory)/unit-testresults.xml $(Build.SourcesDirectory)
  displayName: 'Run unit tests'

- task: PublishTestResults@2
  condition: succeededOrFailed()
  inputs:
    testResultsFiles: '$(Build.BinariesDirectory)/_-testresults.xml'
    testRunTitle: 'Linting & Unit tests'
    failTaskOnFailedTests: true
  displayName: 'Publish linting and unit test results'

- publish: $(Build.SourcesDirectory)
    artifact: di-notebooks
```

Le pipeline utilise [flake8](https://pypi.org/project/flake8/) pour effectuer le linting du code Python. Il exécute les tests unitaires définis dans le code source et publie les résultats du linting et des tests afin qu’ils soient disponibles dans l’écran d’exécution du pipeline Azure :

![Linting de tests unitaires](media/how-to-cicd-data-ingestion/linting-unit-tests.png)

Si le linting et les tests unitaires sont réussis, le pipeline copie le code source dans le référentiel d’artefacts afin d’être utilisé lors des étapes de déploiement ultérieures.

### <a name="azure-data-factory-ci"></a>CI Azure Data Factory

Le processus d’intégration continue pour un pipeline Azure Data Factory est un goulot d’étranglement pour un pipeline d’ingestion des données. Il n’y a pas d’intégration continue. Un artefact déployable pour Azure Data Factory est une collection de modèles Azure Resource Manager. La seule façon de produire ces modèles consiste à cliquer sur le bouton * **publier** _ dans l’espace de travail Azure Data Factory.

1. Les ingénieurs des données fusionnent le code source de leurs branches de fonctionnalités dans la branche de collaboration, par exemple _*_maître_*_ ou _*_développement_*_. 
1. Une personne disposant des autorisations accordées clique sur le bouton _*_publier_*_ pour générer des modèles Azure Resource Manager à partir du code source de la branche de collaboration. 
1. L’espace de travail valide les pipelines (considérez cela comme du linting et des tests unitaires), génère des modèles Azure Resource Manager (considérez cela comme de la création) et enregistre les modèles générés dans une branche technique _*_adf_publish_*_ dans le même dépôt de code (considérez cela comme de la publication d’artefacts). Cette branche est créée automatiquement par l’espace de travail Azure Data Factory. 

Pour plus d’informations sur ce processus, consultez [Intégration et livraison continues dans Azure Data Factory](../data-factory/continuous-integration-deployment.md).

Il est important de s’assurer que les modèles Azure Resource Manager générés sont indépendants de l’environnement. Cela signifie que toutes les valeurs qui peuvent différer d’un environnement à l’autre sont paramétrées. Azure Data Factory est suffisamment intelligente pour exposer la plupart de ces valeurs en tant que paramètres. Par exemple, dans le modèle suivant, les propriétés de connexion à un espace de travail Azure Machine Learning sont exposées en tant que paramètres :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "factoryName": {
            "value": "devops-ds-adf"
        },
        "AzureMLService_servicePrincipalKey": {
            "value": ""
        },
        "AzureMLService_properties_typeProperties_subscriptionId": {
            "value": "0fe1c235-5cfa-4152-17d7-5dff45a8d4ba"
        },
        "AzureMLService_properties_typeProperties_resourceGroupName": {
            "value": "devops-ds-rg"
        },
        "AzureMLService_properties_typeProperties_servicePrincipalId": {
            "value": "6e35e589-3b22-4edb-89d0-2ab7fc08d488"
        },
        "AzureMLService_properties_typeProperties_tenant": {
            "value": "72f988bf-86f1-41af-912b-2d7cd611db47"
        }
    }
}
```

Toutefois, vous souhaiterez peut-être exposer vos propriétés personnalisées qui ne sont pas gérées par défaut par l’espace de travail Azure Data Factory. Dans le scénario de cet article, un pipeline Azure Data Factory appelle un notebook Python qui traite les données. Le notebook accepte un paramètre avec le nom d’un fichier de données d’entrée.

```Python
import pandas as pd
import numpy as np

data_file_name = getArgument("data_file_name")
data = pd.read_csv(data_file_name)

labels = np.array(data['target'])
...
```

Ce nom est différent pour les environnements _*_Dev_*_ , _*_QA_*_ , _*_UAT_*_ et _*_PROD_*_ . Dans un pipeline complexe avec plusieurs activités, il peut y avoir plusieurs propriétés personnalisées. Il est conseillé de collecter toutes ces valeurs dans un même emplacement et de les définir comme des _*_variables_*_ de pipeline :

![Capture d’écran montrant un bloc-notes appelé PrepareData et un pipeline d’exécution d’apprentissage automatique appelé ML Excute Pipeline en haut de l’onglet Variables sélectionné en-dessous, avec l’option permettant d’ajouter de nouvelles variables, chacune avec un nom, un type et une valeur par défaut.](media/how-to-cicd-data-ingestion/adf-variables.png)

Les activités du pipeline peuvent faire référence aux variables de pipeline tout en les utilisant réellement :

![Capture d’écran montrant un bloc-notes appelé PrepareData et un pipeline d’exécution d’apprentissage automatique appelé ML Excute Pipeline en haut avec l’onglet Paramètres sélectionné en-dessous.](media/how-to-cicd-data-ingestion/adf-notebook-parameters.png)

L’espace de travail Azure Data Factory _*_n’exposent pas_*_ les variables de pipeline en tant que paramètres par défaut des modèles Azure Resource Manager. L’espace de travail utilise le [modèle de paramétrage par défaut](../data-factory/continuous-integration-deployment.md#default-parameterization-template) dictant quelles propriétés de pipeline doivent être exposées comme paramètres de modèle Azure Resource Manager. Pour ajouter des variables de pipeline à la liste, mettez à jour la section `"Microsoft.DataFactory/factories/pipelines"` du [modèle de paramétrage par défaut](../data-factory/continuous-integration-deployment.md#default-parameterization-template) avec l’extrait de code suivant et placez le fichier JSON résultant à la racine du dossier source :

```json
"Microsoft.DataFactory/factories/pipelines": {
        "properties": {
            "variables": {
                "_": {
                    "defaultValue": "="
                }
            }
        }
    }
```

Cela obligera l’espace de travail Azure Data Factory à ajouter les variables à la liste de paramètres quand l’utilisateur cliquera sur le bouton * **publier** _ :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "factoryName": {
            "value": "devops-ds-adf"
        },
        ...
        "data-ingestion-pipeline_properties_variables_data_file_name_defaultValue": {
            "value": "driver_prediction_train.csv"
        }        
    }
}
```

Les valeurs du fichier JSON sont les valeurs par défaut configurées dans la définition du pipeline. Elles doivent être remplacées par les valeurs de l’environnement cible lors du déploiement du modèle Azure Resource Manager.

## <a name="continuous-delivery-cd"></a>Déploiement continu (CD)

Le processus de déploiement continu prend les artefacts et les déploie dans le premier environnement cible. Cela permet de s’assurer que la solution fonctionne en exécutant des tests. En cas de réussite, il continue dans l’environnement suivant. 

Le pipeline Azure de CD est constitué de plusieurs index représentant les environnements. Chaque index contient des [déploiements](/azure/devops/pipelines/process/deployment-jobs?view=azure-devops&preserve-view=true) et des [tâches](/azure/devops/pipelines/process/phases?tabs=yaml&view=azure-devops&preserve-view=true) qui effectuent les étapes suivantes :

_Déployer un notebook Python dans un espace de travail Azure Databricks
* Déployer un pipeline de Azure Data Factory 
* Exécuter le pipeline
* Vérifier le résultat de l’ingestion des données

Les index du pipeline peuvent être configurés avec des [approbations](/azure/devops/pipelines/process/approvals?tabs=check-pass&view=azure-devops&preserve-view=true) et des [portes](/azure/devops/pipelines/release/approvals/gates?view=azure-devops&preserve-view=true) qui offrent un contrôle supplémentaire sur la façon dont le processus de déploiement évolue dans la chaîne d’environnements.

### <a name="deploy-a-python-notebook"></a>Déployer un notebook Python

L’extrait de code suivant définit un [déploiement](/azure/devops/pipelines/process/deployment-jobs?view=azure-devops&preserve-view=true) de pipeline Azure qui copie un notebook Python sur un cluster Databricks :

```yaml
- stage: 'Deploy_to_QA'
  displayName: 'Deploy to QA'
  variables:
  - group: devops-ds-qa-vg
  jobs:
  - deployment: "Deploy_to_Databricks"
    displayName: 'Deploy to Databricks'
    timeoutInMinutes: 0
    environment: qa
    strategy:
      runOnce:
        deploy:
          steps:
            - task: UsePythonVersion@0
              inputs:
                versionSpec: '3.x'
                addToPath: true
                architecture: 'x64'
              displayName: 'Use Python3'

            - task: configuredatabricks@0
              inputs:
                url: '$(DATABRICKS_URL)'
                token: '$(DATABRICKS_TOKEN)'
              displayName: 'Configure Databricks CLI'    

            - task: deploynotebooks@0
              inputs:
                notebooksFolderPath: '$(Pipeline.Workspace)/di-notebooks'
                workspaceFolder: '/Shared/devops-ds'
              displayName: 'Deploy (copy) data processing notebook to the Databricks cluster'       
```            

Les artefacts produits par l’intégration continue sont automatiquement copiés vers l’agent de déploiement et sont disponibles dans le dossier `$(Pipeline.Workspace)`. Dans ce cas, la tâche de déploiement fait référence à l’artefact `di-notebooks` contenant le notebook Python. Ce [déploiement](/azure/devops/pipelines/process/deployment-jobs?view=azure-devops&preserve-view=true) utilise l’[extension Databricks Azure DevOps](https://marketplace.visualstudio.com/items?itemName=riserrad.azdo-databricks) pour copier les fichiers du notebook dans l’espace de travail Databricks.

Le stade `Deploy_to_QA` contient une référence au groupe de variables `devops-ds-qa-vg` défini dans le projet Azure DevOps. Les étapes de ce stade font référence aux variables de ce groupe de variables (par exemple `$(DATABRICKS_URL)` et `$(DATABRICKS_TOKEN)`). L’idée est que le stade suivant (par exemple `Deploy_to_UAT`) fonctionne avec les mêmes noms de variables que ceux définis dans son propre groupe de variables limité à UAT.

### <a name="deploy-an-azure-data-factory-pipeline"></a>Déployer un pipeline de Azure Data Factory

Un artefact déployable pour Azure Data Factory est un modèle Azure Resource Manager. Il sera déployé avec la tâche * **Déploiement du groupe de ressources Azure** _, comme illustré dans l’extrait de code suivant :

```yaml
  - deployment: "Deploy_to_ADF"
    displayName: 'Deploy to ADF'
    timeoutInMinutes: 0
    environment: qa
    strategy:
      runOnce:
        deploy:
          steps:
            - task: AzureResourceGroupDeployment@2
              displayName: 'Deploy ADF resources'
              inputs:
                azureSubscription: $(AZURE_RM_CONNECTION)
                resourceGroupName: $(RESOURCE_GROUP)
                location: $(LOCATION)
                csmFile: '$(Pipeline.Workspace)/adf-pipelines/ARMTemplateForFactory.json'
                csmParametersFile: '$(Pipeline.Workspace)/adf-pipelines/ARMTemplateParametersForFactory.json'
                overrideParameters: -data-ingestion-pipeline_properties_variables_data_file_name_defaultValue "$(DATA_FILE_NAME)"
```
La valeur du paramètre de nom de fichier de données provient de la variable `$(DATA_FILE_NAME)` définie dans un groupe de variables du stade AQ. De même, tous les paramètres définis dans _*_ARMTemplateForFactory.json_*_ peuvent être substitués. S’ils ne le sont pas, les valeurs par défaut sont utilisées.

### <a name="run-the-pipeline-and-check-the-data-ingestion-result"></a>Exécuter le pipeline et vérifier le résultat de l’ingestion des données

L’étape suivante consiste à s’assurer que la solution déployée fonctionne. La définition de travail suivante exploite un pipeline Azure Data Factory avec un [script PowerShell](https://github.com/microsoft/DataOps/tree/master/adf/utils) et exécute un notebook Python sur un cluster Azure Databricks. Le notebook vérifie si les données ont été correctement ingérées et valide le fichier de données de résultats dont le nom est `$(bin_FILE_NAME)`.

```yaml
  - job: "Integration_test_job"
    displayName: "Integration test job"
    dependsOn: [Deploy_to_Databricks, Deploy_to_ADF]
    pool:
      vmImage: 'ubuntu-latest'
    timeoutInMinutes: 0
    steps:
    - task: AzurePowerShell@4
      displayName: 'Execute ADF Pipeline'
      inputs:
        azureSubscription: $(AZURE_RM_CONNECTION)
        ScriptPath: '$(Build.SourcesDirectory)/adf/utils/Invoke-ADFPipeline.ps1'
        ScriptArguments: '-ResourceGroupName $(RESOURCE_GROUP) -DataFactoryName $(DATA_FACTORY_NAME) -PipelineName $(PIPELINE_NAME)'
        azurePowerShellVersion: LatestVersion
    - task: UsePythonVersion@0
      inputs:
        versionSpec: '3.x'
        addToPath: true
        architecture: 'x64'
      displayName: 'Use Python3'

    - task: configuredatabricks@0
      inputs:
        url: '$(DATABRICKS_URL)'
        token: '$(DATABRICKS_TOKEN)'
      displayName: 'Configure Databricks CLI'    

    - task: executenotebook@0
      inputs:
        notebookPath: '/Shared/devops-ds/test-data-ingestion'
        existingClusterId: '$(DATABRICKS_CLUSTER_ID)'
        executionParams: '{"bin_file_name":"$(bin_FILE_NAME)"}'
      displayName: 'Test data ingestion'

    - task: waitexecution@0
      displayName: 'Wait until the testing is done'
```

La dernière tâche du travail vérifie le résultat de l’exécution du notebook. Si elle retourne une erreur, elle définit l’état de l’exécution du pipeline sur Échec.

## <a name="putting-pieces-together"></a>Assemblage des éléments

Le pipeline CI/CD Azure complet se compose des phases suivantes : _CI
* Déployer dans QA
    * Déployer dans Databricks + déployer dans ADF
    * Test d’intégration

Il contient un nombre d’index * **Déployer** _ égal au nombre d’environnements cibles. Chaque index _*_Déployer_*_ contient deux [déploiements](/azure/devops/pipelines/process/deployment-jobs?view=azure-devops&preserve-view=true) qui s’exécutent en parallèle, et un [travail](/azure/devops/pipelines/process/phases?tabs=yaml&view=azure-devops&preserve-view=true) qui s’exécute après les déploiements pour tester la solution sur l’environnement.

Un exemple d’implémentation du pipeline est assemblé dans l’extrait de code _*_yaml_*_ suivant :

```yaml
variables:
- group: devops-ds-vg

stages:
- stage: 'CI'
  displayName: 'CI'
  jobs:
  - job: "CI_Job"
    displayName: "CI Job"
    pool:
      vmImage: 'ubuntu-latest'
    timeoutInMinutes: 0
    steps:
    - task: UsePythonVersion@0
      inputs:
        versionSpec: '3.x'
        addToPath: true
        architecture: 'x64'
      displayName: 'Use Python3'
    - script: pip install --upgrade flake8 flake8_formatter_junit_xml
      displayName: 'Install flake8'
    - checkout: self
    - script: |
       flake8 --output-file=$(Build.BinariesDirectory)/lint-testresults.xml --format junit-xml  
    workingDirectory: '$(Build.SourcesDirectory)'
    displayName: 'Run flake8 (code style analysis)'  
    - script: |
       python -m pytest --junitxml=$(Build.BinariesDirectory)/unit-testresults.xml $(Build.SourcesDirectory)
    displayName: 'Run unit tests'
    - task: PublishTestResults@2
    condition: succeededOrFailed()
    inputs:
        testResultsFiles: '$(Build.BinariesDirectory)/_-testresults.xml'
        testRunTitle: 'Linting & Unit tests'
        failTaskOnFailedTests: true
    displayName: 'Publish linting and unit test results'    

    # The CI stage produces two artifacts (notebooks and ADF pipelines).
    # The pipelines Azure Resource Manager templates are stored in a technical branch "adf_publish"
    - publish: $(Build.SourcesDirectory)/$(Build.Repository.Name)/code/dataingestion
      artifact: di-notebooks
    - checkout: git://${{variables['System.TeamProject']}}@adf_publish    
    - publish: $(Build.SourcesDirectory)/$(Build.Repository.Name)/devops-ds-adf
      artifact: adf-pipelines

- stage: 'Deploy_to_QA'
  displayName: 'Deploy to QA'
  variables:
  - group: devops-ds-qa-vg
  jobs:
  - deployment: "Deploy_to_Databricks"
    displayName: 'Deploy to Databricks'
    timeoutInMinutes: 0
    environment: qa
    strategy:
      runOnce:
        deploy:
          steps:
            - task: UsePythonVersion@0
              inputs:
                versionSpec: '3.x'
                addToPath: true
                architecture: 'x64'
              displayName: 'Use Python3'

            - task: configuredatabricks@0
              inputs:
                url: '$(DATABRICKS_URL)'
                token: '$(DATABRICKS_TOKEN)'
              displayName: 'Configure Databricks CLI'    

            - task: deploynotebooks@0
              inputs:
                notebooksFolderPath: '$(Pipeline.Workspace)/di-notebooks'
                workspaceFolder: '/Shared/devops-ds'
              displayName: 'Deploy (copy) data processing notebook to the Databricks cluster'             
  - deployment: "Deploy_to_ADF"
    displayName: 'Deploy to ADF'
    timeoutInMinutes: 0
    environment: qa
    strategy:
      runOnce:
        deploy:
          steps:
            - task: AzureResourceGroupDeployment@2
              displayName: 'Deploy ADF resources'
              inputs:
                azureSubscription: $(AZURE_RM_CONNECTION)
                resourceGroupName: $(RESOURCE_GROUP)
                location: $(LOCATION)
                csmFile: '$(Pipeline.Workspace)/adf-pipelines/ARMTemplateForFactory.json'
                csmParametersFile: '$(Pipeline.Workspace)/adf-pipelines/ARMTemplateParametersForFactory.json'
                overrideParameters: -data-ingestion-pipeline_properties_variables_data_file_name_defaultValue "$(DATA_FILE_NAME)"
  - job: "Integration_test_job"
    displayName: "Integration test job"
    dependsOn: [Deploy_to_Databricks, Deploy_to_ADF]
    pool:
      vmImage: 'ubuntu-latest'
    timeoutInMinutes: 0
    steps:
    - task: AzurePowerShell@4
      displayName: 'Execute ADF Pipeline'
      inputs:
        azureSubscription: $(AZURE_RM_CONNECTION)
        ScriptPath: '$(Build.SourcesDirectory)/adf/utils/Invoke-ADFPipeline.ps1'
        ScriptArguments: '-ResourceGroupName $(RESOURCE_GROUP) -DataFactoryName $(DATA_FACTORY_NAME) -PipelineName $(PIPELINE_NAME)'
        azurePowerShellVersion: LatestVersion
    - task: UsePythonVersion@0
      inputs:
        versionSpec: '3.x'
        addToPath: true
        architecture: 'x64'
      displayName: 'Use Python3'

    - task: configuredatabricks@0
      inputs:
        url: '$(DATABRICKS_URL)'
        token: '$(DATABRICKS_TOKEN)'
      displayName: 'Configure Databricks CLI'    

    - task: executenotebook@0
      inputs:
        notebookPath: '/Shared/devops-ds/test-data-ingestion'
        existingClusterId: '$(DATABRICKS_CLUSTER_ID)'
        executionParams: '{"bin_file_name":"$(bin_FILE_NAME)"}'
      displayName: 'Test data ingestion'

    - task: waitexecution@0
      displayName: 'Wait until the testing is done'                

```

## <a name="next-steps"></a>Étapes suivantes

* [Contrôle de code source dans Azure Data Factory](../data-factory/source-control.md)
* [Intégration et déploiement continus dans Azure Data Factory](../data-factory/continuous-integration-deployment.md)
* [DevOps pour Azure Databricks](https://marketplace.visualstudio.com/items?itemName=riserrad.azdo-databricks)
---
title: Utiliser Azure Pipelines pour créer et déployer des solutions HPC
description: Découvrez comment déployer un pipeline de build/mise en production pour une application HPC en cours d’exécution sur Azure Batch.
author: chrisreddington
ms.author: chredd
ms.date: 03/28/2019
ms.topic: how-to
ms.openlocfilehash: e87be0db65cf12a265566e0c05815722ce3cc609
ms.sourcegitcommit: 1d6ec4b6f60b7d9759269ce55b00c5ac5fb57d32
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94578873"
---
# <a name="use-azure-pipelines-to-build-and-deploy-hpc-solutions"></a>Utiliser Azure Pipelines pour créer et déployer des solutions HPC

Azure DevOps Services propose une gamme d’outils utilisés par les équipes de développement lors de la création d’une application personnalisée. Les outils fournis par Azure DevOps peuvent entraîner la création automatique et le test de solutions de calcul hautes performances. Cet article montre comment configurer une intégration continue (CI) et un déploiement continu (CD) à l’aide d’Azure Pipelines pour une solution de calcul haute performance déployée sur Azure Batch.

Azure Pipelines fournit une gamme de processus CI/CD pour créer, déployer, tester et surveiller les logiciels. Ces processus accélèrent la livraison des logiciels, ce qui vous permet de vous concentrer sur votre code plutôt que de prendre en charge l’infrastructure et les opérations.

## <a name="create-an-azure-pipeline"></a>Créer un pipeline Azure

Dans cet exemple, nous allons créer un pipeline de build/mise en production pour déployer une infrastructure Azure Batch et mettre un package d’application en production. En supposant que le code est développé localement, voici le flux de déploiement général :

![Diagramme montrant le flux de déploiement dans notre pipeline](media/batch-ci-cd/DeploymentFlow.png)

### <a name="setup"></a>Programme d’installation

Pour suivre les étapes décrites dans cet article, vous avez besoin d’une organisation Azure DevOps et d’un projet d’équipe.

* [Créer une organisation Azure DevOps](/azure/devops/organizations/accounts/create-organization)
* [Créer un projet dans Azure DevOps](/azure/devops/organizations/projects/create-project)

### <a name="source-control-for-your-environment"></a>Contrôle de code source pour votre environnement

Le contrôle de code source permet aux équipes de suivre les modifications apportées au codebase et d’inspecter les versions précédentes du code.

En général, le contrôle de code source va de pair avec le code de logiciel. Qu’en est-il de l’infrastructure sous-jacente ? Cela nous amène à l’infrastructure en tant que code, dans laquelle nous allons utiliser des modèles Azure Resource Manager ou d’autres solutions open source pour définir de façon déclarative notre infrastructure sous-jacente.

Cet exemple s’appuie fortement sur plusieurs modèles Resource Manager (documents JSON) et fichiers binaires existants. Vous pouvez copier ces exemples dans votre référentiel et les envoyer vers Azure DevOps.

La structure du codebase utilisée dans cet exemple ressemble à celle-ci :

* Un dossier **arm-templates** contenant un nombre de modèles Azure Resource Manager. Les modèles sont expliqués dans cet article.
* Un dossier **client-application**, qui est une copie de l’exemple [Traitement des fichiers Azure Batch .NET avec ffmpeg](https://github.com/Azure-Samples/batch-dotnet-ffmpeg-tutorial). Cela n’est pas nécessaire pour cet article.
* Un dossier **hpc-application**, soit la version Windows 64 bits de [ffmpeg 4.3.1](https://github.com/GyanD/codexffmpeg/releases/tag/4.3.1-2020-11-08).
* Un dossier **pipelines**. Il contient un fichier YAML décrivant notre processus de génération. Ce sujet est abordé dans cet article.

Cette section part du principe que vous êtes familiarisé avec la gestion de version et la conception de modèles Resource Manager. Si vous n’êtes pas familiarisé avec ces concepts, consultez les pages suivantes pour plus d’informations.

* [Qu’est-ce que le contrôle de code source ?](/azure/devops/user-guide/source-control)
* [Comprendre la structure et la syntaxe des modèles Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md)

#### <a name="azure-resource-manager-templates"></a>Modèles Microsoft Azure Resource Manager

Cet exemple s’appuie sur plusieurs modèles Resource Manager pour déployer notre solution. Pour ce faire, nous utilisons plusieurs modèles de fonctionnalités (semblables à des unités ou modules) qui implémentent une partie spécifique des fonctionnalités. Nous utilisons également un modèle de solution de bout en bout chargé de rassembler ces fonctionnalités sous-jacentes. Il existe plusieurs avantages à cette approche :

* Les modèles de fonctionnalités sous-jacents peuvent être testés individuellement.
* Les modèles de fonctionnalités sous-jacents peuvent être définis comme une norme à l’intérieur d’une organisation et peuvent être réutilisés dans plusieurs solutions.

Pour cet exemple, il existe un modèle de solution de bout en bout (deployment.json) qui déploie les trois modèles. Les modèles sous-jacents sont des modèles de fonctionnalités, responsables du déploiement d’un aspect spécifique de la solution.

![Exemple de structure de modèle liée utilisant des modèles Azure Resource Manager](media/batch-ci-cd/ARMTemplateHierarchy.png)

Le premier modèle que nous allons examiner est destiné à un compte Stockage Azure. Notre solution requiert un compte de stockage pour déployer l’application sur notre compte Batch. Il est important de connaître le [guide de référence de modèle Resource Manager pour les types de ressources Microsoft.Storage](/azure/templates/microsoft.storage/allversions) lors de la création de modèles Resource Manager pour les comptes Stockage.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "accountName": {
            "type": "string",
            "metadata": {
                 "description": "Name of the Azure Storage Account"
             }
         }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[parameters('accountName')]",
            "sku": {
                "name": "Standard_LRS"
            },
            "apiVersion": "2018-02-01",
            "location": "[resourceGroup().location]",
            "properties": {}
        }
    ],
    "outputs": {
        "blobEndpoint": {
          "type": "string",
          "value": "[reference(resourceId('Microsoft.Storage/storageAccounts', parameters('accountName'))).primaryEndpoints.blob]"
        },
        "resourceId": {
          "type": "string",
          "value": "[resourceId('Microsoft.Storage/storageAccounts', parameters('accountName'))]"
        }
    }
}
```

Ensuite, nous allons examiner le modèle de compte Azure Batch. Le compte Azure Batch agit comme une plateforme pour exécuter de nombreuses applications entre les pools (regroupement de machines). Il est important de connaître le [guide de référence de modèle Resource Manager pour les types de ressources Microsoft.Batch](/azure/templates/microsoft.batch/allversions) lors de la création de modèles Resource Manager pour les comptes Batch.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "batchAccountName": {
           "type": "string",
           "metadata": {
                "description": "Name of the Azure Batch Account"
            }
        },
        "storageAccountId": {
           "type": "string",
           "metadata": {
                "description": "ID of the Azure Storage Account"
            }
        }
    },
    "variables": {},
    "resources": [
        {
            "name": "[parameters('batchAccountName')]",
            "type": "Microsoft.Batch/batchAccounts",
            "apiVersion": "2017-09-01",
            "location": "[resourceGroup().location]",
            "properties": {
              "poolAllocationMode": "BatchService",
              "autoStorage": {
                  "storageAccountId": "[parameters('storageAccountId')]"
              }
            }
          }
    ],
    "outputs": {}
}
```

Le modèle suivant présente un exemple de création d’un pool Azure Batch (les ordinateurs principaux pour traiter nos applications). Il est important de connaître le [guide de référence de modèle Resource Manager pour les types de ressources Microsoft.Batch](/azure/templates/microsoft.batch/allversions) lors de la création de modèles Resource Manager pour les pools du compte Batch.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "batchAccountName": {
           "type": "string",
           "metadata": {
                "description": "Name of the Azure Batch Account"
           }
        },
        "batchAccountPoolName": {
            "type": "string",
            "metadata": {
                 "description": "Name of the Azure Batch Account Pool"
             }
         }
    },
    "variables": {},
    "resources": [
        {
            "name": "[concat(parameters('batchAccountName'),'/', parameters('batchAccountPoolName'))]",
            "type": "Microsoft.Batch/batchAccounts/pools",
            "apiVersion": "2017-09-01",
            "properties": {
                "deploymentConfiguration": {
                    "virtualMachineConfiguration": {
                        "imageReference": {
                            "publisher": "Canonical",
                            "offer": "UbuntuServer",
                            "sku": "18.04-LTS",
                            "version": "latest"
                        },
                        "nodeAgentSkuId": "batch.node.ubuntu 18.04"
                    }
                },
                "vmSize": "Standard_D1_v2"
            }
          }
    ],
    "outputs": {}
}
```

Enfin, nous avons un modèle qui se comporte comme un orchestrateur. Ce modèle est responsable du déploiement des modèles de fonctionnalités.

Vous trouverez également plus d’informations sur la [création de modèles Azure Resource Manager liés](../azure-resource-manager/templates/deployment-tutorial-linked-template.md) dans un article distinct.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "templateContainerUri": {
           "type": "string",
           "metadata": {
                "description": "URI of the Blob Storage Container containing the Azure Resouce Manager templates"
            }
        },
        "templateContainerSasToken": {
           "type": "string",
           "metadata": {
                "description": "The SAS token of the container containing the Azure Resouce Manager templates"
            }
        },
        "applicationStorageAccountName": {
            "type": "string",
            "metadata": {
                 "description": "Name of the Azure Storage Account"
            }
         },
        "batchAccountName": {
            "type": "string",
            "metadata": {
                 "description": "Name of the Azure Batch Account"
            }
         },
         "batchAccountPoolName": {
             "type": "string",
             "metadata": {
                  "description": "Name of the Azure Batch Account Pool"
              }
          }
    },
    "variables": {},
    "resources": [
        {
            "apiVersion": "2017-05-10",
            "name": "storageAccountDeployment",
            "type": "Microsoft.Resources/deployments",
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                  "uri": "[concat(parameters('templateContainerUri'), '/storageAccount.json', parameters('templateContainerSasToken'))]",
                  "contentVersion": "1.0.0.0"
                },
                "parameters": {
                    "accountName": {"value": "[parameters('applicationStorageAccountName')]"}
                }
            }
        },  
        {
            "apiVersion": "2017-05-10",
            "name": "batchAccountDeployment",
            "type": "Microsoft.Resources/deployments",
            "dependsOn": [
                "storageAccountDeployment"
            ],
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                  "uri": "[concat(parameters('templateContainerUri'), '/batchAccount.json', parameters('templateContainerSasToken'))]",
                  "contentVersion": "1.0.0.0"
                },
                "parameters": {
                    "batchAccountName": {"value": "[parameters('batchAccountName')]"},
                    "storageAccountId": {"value": "[reference('storageAccountDeployment').outputs.resourceId.value]"}
                }
            }
        },
        {
            "apiVersion": "2017-05-10",
            "name": "poolDeployment",
            "type": "Microsoft.Resources/deployments",
            "dependsOn": [
                "batchAccountDeployment"
            ],
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                  "uri": "[concat(parameters('templateContainerUri'), '/batchAccountPool.json', parameters('templateContainerSasToken'))]",
                  "contentVersion": "1.0.0.0"
                },
                "parameters": {
                    "batchAccountName": {"value": "[parameters('batchAccountName')]"},
                    "batchAccountPoolName": {"value": "[parameters('batchAccountPoolName')]"}
                }
            }
        }
    ],
    "outputs": {}
}
```

#### <a name="the-hpc-solution"></a>La solution HPC

L’infrastructure et le logiciel peuvent être définis en tant que code et figurer dans le même référentiel.

Pour cette solution, le package ffmpeg est utilisé en tant que package d’application. Le package ffmpeg peut être téléchargé [ici](https://www.videohelp.com/software?d=ffmpeg-3.3.4-win64-static.zip).

![Exemple de structure de référentiel Git](media/batch-ci-cd/git-repository.jpg)

Il existe quatre sections principales dans ce référentiel :

* Le dossier **arm-templates** qui stocke notre infrastructure en tant que code
* Le dossier **hpc-application** qui contient les fichiers binaires pour ffmpeg
* Le dossier **pipelines** qui contient la définition de notre pipeline de build.
* **Facultatif** : Le dossier **client-application** qui stocke le code pour l’application .NET. Nous n’utilisons pas cette méthode dans l’exemple, mais dans votre propre projet, vous pouvez lancer les exécutions de l’application par lots HPC via une application cliente.

> [!NOTE]
> Il s’agit d’un exemple d’une structure de codebase. Cette approche est utilisée pour démontrer que l’application, l’infrastructure et le code de pipeline sont stockés dans le même référentiel.

Maintenant que le code source est configuré, nous pouvons commencer la première génération.

## <a name="continuous-integration"></a>Intégration continue

[Azure Pipelines](/azure/devops/pipelines/get-started/) vous permet, au sein d’Azure DevOps Services, d’implémenter un pipeline de build, de test et de déploiement pour vos applications.

Dans cette étape du pipeline, les tests sont généralement exécutés pour valider le code et générer les éléments appropriés du logiciel. Le nombre et les types de tests, ainsi que toutes les tâches supplémentaires que vous exécutez, dépendent du build le plus large et de la stratégie de publication.

## <a name="preparing-the-hpc-application"></a>Préparation de l’application HPC

Dans cet exemple, nous allons nous concentrer sur le dossier **hpc-application**. Le dossier **hpc-application** est le logiciel ffmpeg qui sera exécuté à partir du compte Azure Batch.

1. Accédez à la section Builds d’Azure Pipelines dans votre organisation Azure DevOps. Créez un **Nouveau pipeline**.

    ![Créer un nouveau pipeline de build](media/batch-ci-cd/new-build-pipeline.jpg)

1. Pour créer un pipeline de build, vous disposez de deux méthodes :

    a. [À l’aide du concepteur visuel](/azure/devops/pipelines/get-started-designer). Pour cela, cliquez sur « Utiliser le concepteur visuel » sur la page **Nouveau pipeline**.

    b. [À l’aide de builds YAML](/azure/devops/pipelines/get-started-yaml). Vous pouvez créer un pipeline YAML en cliquant sur l’option Azure Repos ou GitHub sur la page Nouveau pipeline. Vous pouvez également stocker l’exemple ci-dessous dans votre contrôle de code source et référencer un fichier YAML existant en cliquant sur le concepteur visuel, puis à l’aide du modèle YAML.

    ```yml
    # To publish an application into Azure Batch, we need to
    # first zip the file, and then publish an artifact, so that
    # we can take the necessary steps in our release pipeline.
    steps:
    # First, we Zip up the files required in the Batch Account
    # For this instance, those are the ffmpeg files
    - task: ArchiveFiles@2
      displayName: 'Archive applications'
      inputs:
        rootFolderOrFile: hpc-application
        includeRootFolder: false
        archiveFile: '$(Build.ArtifactStagingDirectory)/package/$(Build.BuildId).zip'
    # Publish that zip file, so that we can use it as part
    # of our Release Pipeline later
    - task: PublishPipelineArtifact@0
      inputs:
        artifactName: 'hpc-application'
        targetPath: '$(Build.ArtifactStagingDirectory)/package'
    ```

1. Une fois la build configurée selon les besoins, sélectionnez **Enregistrer et mettre en file d’attente**. Si l’intégration continue avec activée (dans la section **Déclencheurs**), la build se déclenche automatiquement lorsqu’un nouveau commit est créé dans le référentiel, répondant aux conditions définies dans la build.

    ![Exemple d’un pipeline de build existant](media/batch-ci-cd/existing-build-pipeline.jpg)

1. Affichez les mises à jour automatiques de la progression de votre build dans Azure DevOps en accédant à la section **Build** d’Azure Pipelines. Sélectionnez la build appropriée à partir de votre définition de build.

    ![Afficher les sorties en direct à partir de votre build](media/batch-ci-cd/Build-1.jpg)

> [!NOTE]
> Si vous utilisez une application cliente pour exécuter votre application par lots HPC, vous devez créer une définition de build distincte pour l’application. Vous pouvez trouver différents guides pratiques dans la documentation [Azure Pipelines](/azure/devops/pipelines/get-started/index).

## <a name="continuous-deployment"></a>Déploiement continu

Azure Pipelines peut également être utilisé pour déployer votre application et l’infrastructure sous-jacente. Les [pipeline de mise en production](/azure/devops/pipelines/release) sont les composants qui permettent un déploiement continu et automatisent votre processus de publication.

### <a name="deploying-your-application-and-underlying-infrastructure"></a>Déploiement de l’application et de l’infrastructure sous-jacente

Il existe plusieurs étapes dans le processus de déploiement de l’infrastructure. Étant donné que nous avons utilisé des [modèles liés](../azure-resource-manager/templates/linked-templates.md), ces modèles doivent être accessibles à partir d’un point de terminaison public (HTTP ou HTTPS). Il peut s’agir d’un référentiel sur GitHub, d’un compte de stockage d’objets Blob Azure ou d’un autre emplacement de stockage. Les artefacts du modèle téléchargé peuvent rester sécurisés, car ils peuvent être conservés en mode privé, tout en étant accessibles grâce à une sorte de jeton de signature d’accès partagé (SAS). L’exemple suivant montre comment déployer une infrastructure avec des modèles à partir d’un objet blob Stockage Azure.

1. Créez une **Nouvelle définition de mise en production** et sélectionnez une définition vide. Nous devons ensuite renommer l’environnement qui vient d’être créé avec un nom approprié pour notre pipeline.

    ![Pipeline de mise en production initial](media/batch-ci-cd/Release-0.jpg)

1. Créez une dépendance sur le pipeline de build pour obtenir la sortie de notre application HPC.

    > [!NOTE]
    > Une fois encore, notez la valeur **Alias Source**, qui sera nécessaire lors de la création de tâches à l’intérieur de la définition de mise en production.

    ![Créer un lien d’artefact vers HPCApplicationPackage dans le pipeline de build approprié](media/batch-ci-cd/Release-1.jpg)

1. Créez un lien vers un autre artefact, cette fois un référentiel Azure. Cette action est nécessaire pour accéder aux modèles Resource Manager stockés dans le référentiel. Comme les modèles Resource Manager ne nécessitent pas de compilation, vous n’avez pas besoin de les envoyer via un pipeline de build.

    > [!NOTE]
    > Une fois encore, notez la valeur **Alias Source**, qui sera nécessaire lors de la création de tâches à l’intérieur de la définition de mise en production.

    ![Créer un lien d’artefact vers Azure Repos](media/batch-ci-cd/Release-2.jpg)

1. Accédez à la section **Variables**. Il est recommandé de créer un nombre de variables dans votre pipeline, afin de ne pas entrer les mêmes informations pour plusieurs tâches. Voici les variables utilisées dans cet exemple, et leur impact sur le déploiement.

    * **applicationStorageAccountName** : Nom du compte de stockage qui contient les fichiers binaires d’application HPC
    * **batchAccountApplicationName** : Nom de l’application dans le compte Azure Batch
    * **batchAccountName** : Nom du compte Azure Batch
    * **batchAccountPoolName** : Nom du pool de machines virtuelles qui exécutent le processus
    * **batchApplicationId** : ID unique de l’application Azure Batch
    * **batchApplicationVersion** : Version sémantique de votre application Batch (autrement dit, les fichiers binaires ffmpeg)
    * **location** : Emplacement des ressources Azure à déployer
    * **resourceGroupName** : Nom du groupe de ressources à créer, et l’emplacement vers lequel les ressources seront déployées
    * **storageAccountName** : Nom du compte de stockage qui contient les modèles Resource Manager liés

    ![Exemple de variables définies pour la version d’Azure Pipelines](media/batch-ci-cd/Release-4.jpg)

1. Accédez aux tâches de l’environnement de développement. Dans la capture instantanée ci-dessous, vous pouvez voir six tâches. Ces tâches sont : télécharger les fichiers ffmpeg compressés, déployer un compte de stockage pour héberger les modèles Resource Manager imbriqués, copier ces modèles Resource Manager sur le compte de stockage, déployer le compte Batch et les dépendances requises, créer une application dans le compte Azure Batch et charger le package d’application dans le compte Azure Batch.

    ![Exemple de tâches utilisées pour libérer l’application HPC dans Azure Batch](media/batch-ci-cd/Release-3.jpg)

1. Ajouter la tâche **Télécharger l’artefact de pipeline (préversion)** et définissez les propriétés suivantes :
    * **Nom d’affichage :** Télécharger ApplicationPackage sur l’agent
    * **Le nom de l’artefact à télécharger :** hpc-application
    * **Chemin d’accès pour télécharger** : $(System.DefaultWorkingDirectory)

1. Créez un compte de stockage pour stocker vos artefacts. Vous pouvez utiliser un compte de stockage existant de la solution, mais pour l’exemple autonome et l’isolation du contenu, nous allons créer un compte de stockage dédié pour nos artefacts (en particulier les modèles Resource Manager).

    Ajoutez la tâche **Déploiement de groupes de ressources Azure** et définissez les propriétés suivantes :
    * **Nom d’affichage :** Déployer un compte de stockage pour les modèles Resource Manager
    * **Abonnement Azure :** Sélectionnez l’abonnement Azure approprié
    * **Action** : Créer ou mettre à jour un groupe de ressources
    * **Groupe de ressources** : $(resourceGroupName)
    * **Emplacement** : $(location)
    * **Modèle** : $(System.ArtifactsDirectory)/ **{YourAzureRepoArtifactSourceAlias}** /arm-templates/storageAccount.json
    * **Remplacer les paramètres de modèle** : -accountName $(storageAccountName)

1. Téléchargez les artefacts à partir du contrôle de code source vers le compte de stockage. Il existe une tâche Azure Pipeline pour effectuer cette action. Dans le cadre de cette tâche, l’URL de conteneur du compte de stockage et le jeton SAP peuvent être générés dans une variable dans Azure Pipelines. Cela signifie qu’elle peut être réutilisée tout au long de la phase d’agent.

    Ajoutez la tâche **Copie de fichiers Azure** et définissez les propriétés suivantes :
    * **Source :** $(System.ArtifactsDirectory)/ **{YourAzureRepoArtifactSourceAlias}** /arm-templates/
    * **Type de connexion Azure** : Azure Resource Manager
    * **Abonnement Azure :** Sélectionnez l’abonnement Azure approprié
    * **Type de destination** : Objets blob Azure
    * **Compte de stockage RM** : $(storageAccountName)
    * **Nom du conteneur** : modèles
    * **URI de conteneur de stockage** : templateContainerUri
    * **Jeton SAP du conteneur de stockage** : templateContainerSasToken

1. Déployez le modèle d’orchestrateur. Souvenez-vous du modèle d’orchestrateur précédemment. Vous remarquerez les paramètres de l’URL de conteneur du compte de stockage, en plus du jeton SAS. Vous devriez remarquer que les variables requises dans le modèle Resource Manager sont stockées dans la section Variables de la définition de publication, ou ont été définies à partir d’une autre tâche Azure Pipelines (par exemple, une partie de la tâche de copie d’objets blob Azure).

    Ajoutez la tâche **Déploiement de groupes de ressources Azure** et définissez les propriétés suivantes :
    * **Nom d’affichage :** Déployer Azure Batch
    * **Abonnement Azure :** Sélectionnez l’abonnement Azure approprié
    * **Action** : Créer ou mettre à jour un groupe de ressources
    * **Groupe de ressources** : $(resourceGroupName)
    * **Emplacement** : $(location)
    * **Modèle** : $(System.ArtifactsDirectory)/ **{YourAzureRepoArtifactSourceAlias}** /arm-templates/deployment.json
    * **Remplacer les paramètres du modèle** : ```-templateContainerUri $(templateContainerUri) -templateContainerSasToken $(templateContainerSasToken) -batchAccountName $(batchAccountName) -batchAccountPoolName $(batchAccountPoolName) -applicationStorageAccountName $(applicationStorageAccountName)```

Une pratique courante consiste à utiliser les tâches Azure Key Vault. Si des stratégies d’accès appropriées sont définies pour le principal de service (connexion à votre abonnement Azure), ce dernier peut télécharger des secrets à partir d’un coffre de clés Azure et être utilisé en tant que variables dans votre pipeline. Le nom de la clé secrète est défini avec la valeur associée. Par exemple, un secret sshPassword peut être référencé avec $(sshPassword) dans la définition de mise en production.

1. Les étapes suivantes appellent l’interface Azure CLI. La première est utilisée pour créer une application dans Azure Batch et télécharger les packages associés.

    Ajoutez la tâche **Azure CLI** et définissez les propriétés suivantes :
    * **Nom d’affichage :** Créer une application dans un compte Azure Batch
    * **Abonnement Azure :** Sélectionnez l’abonnement Azure approprié
    * **Emplacement du script** : Script en ligne
    * **Script en ligne** : ```az batch application create --application-id $(batchApplicationId) --name $(batchAccountName) --resource-group $(resourceGroupName)```

1. La deuxième étape est utilisée pour charger les packages associés dans l’application. Dans notre cas, les fichiers ffmpeg.

    Ajoutez la tâche **Azure CLI** et définissez les propriétés suivantes :
    * **Nom d’affichage :** Télécharger un package vers un compte Azure Batch
    * **Abonnement Azure :** Sélectionnez l’abonnement Azure approprié
    * **Emplacement du script** : Script en ligne
    * **Script en ligne** : ```az batch application package create --application-id $(batchApplicationId)  --name $(batchAccountName)  --resource-group $(resourceGroupName) --version $(batchApplicationVersion) --package-file=$(System.DefaultWorkingDirectory)/$(Release.Artifacts.{YourBuildArtifactSourceAlias}.BuildId).zip```

    > [!NOTE]
    > Le numéro de version du package d’application est défini sur une variable. C’est pratique si vous souhaitez remplacer les versions précédentes du package, et si vous souhaitez contrôler manuellement le numéro de version du package envoyé vers Azure Batch.

1. Créez une nouvelle version en sélectionnant **Version > Créer une nouvelle version**. Une fois déclenchée, sélectionnez le lien vers la nouvelle version pour afficher l’état.

1. Vous pouvez afficher la sortie en direct à partir de l’agent en sélectionnant le bouton **Journaux** en dessous de votre environnement.

    ![Afficher l’état de la version](media/batch-ci-cd/Release-5.jpg)

### <a name="testing-the-environment"></a>Tester l’environnement

Une fois l’environnement configuré, vérifiez que les tests suivants peuvent être effectués avec succès.

Connectez-vous au nouveau compte Azure Batch à l’aide de l’interface Azure CLI à partir d’une invite de commandes PowerShell.

* Connectez-vous à votre compte Azure avec `az login` et suivez les instructions pour vous effectuer l’authentification.
* Authentifiez maintenant le compte Batch : `az batch account login -g <resourceGroup> -n <batchAccount>`

#### <a name="list-the-available-applications"></a>Répertorier les applications disponibles

```azurecli
az batch application list -g <resourcegroup> -n <batchaccountname>
```

#### <a name="check-the-pool-is-valid"></a>Vérifier que le pool est valide

```azurecli
az batch pool list
```

Notez la valeur de `currentDedicatedNodes` à partir de la sortie de cette commande. Cette valeur est ajustée dans le test suivant.

#### <a name="resize-the-pool"></a>Redimensionner le pool

Redimensionnez le pool afin que des nœuds de calcul soient disponibles pour les tests de tâches. Examinez la commande de liste de pool pour afficher l’état actuel jusqu’à ce que le redimensionnement soit terminé et les nœuds disponibles.

```azurecli
az batch pool resize --pool-id <poolname> --target-dedicated-nodes 4
```

## <a name="next-steps"></a>Étapes suivantes

En plus de cet article, il existe deux tutoriels qui utilisent ffmpeg, à l’aide de .NET et de Python. Consultez ces tutoriels pour obtenir plus d’informations sur la manière d’interagir avec un compte Batch via une application simple.

* [Exécuter une charge de travail parallèle avec Azure Batch à l’aide de l’API Python](tutorial-parallel-python.md)
* [Exécuter une charge de travail parallèle avec Azure Batch à l’aide de l’API .NET](tutorial-parallel-dotnet.md)

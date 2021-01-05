---
title: Configurer l’environnement de développement pour les scripts de déploiement dans les modèles | Microsoft Docs
description: Configurez l’environnement de développement pour les scripts de déploiement dans les modèles Azure Resource Manager.
services: azure-resource-manager
author: mumian
ms.service: azure-resource-manager
ms.topic: conceptual
ms.date: 12/14/2020
ms.author: jgao
ms.openlocfilehash: 4a7f21410bb97db0a7974870efb812c9954ac241
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97503554"
---
# <a name="configure-development-environment-for-deployment-scripts-in-templates"></a>Configurer l’environnement de développement pour les scripts de déploiement dans les modèles

Découvrez comment créer un environnement de développement pour développer et tester des scripts de déploiement avec une image de script de déploiement. Vous pouvez soit créer une [instance de conteneur Azure](../../container-instances/container-instances-overview.md), soit utiliser [Docker](https://docs.docker.com/get-docker/). Les deux solutions sont traitées dans cet article.

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas de script de déploiement, vous pouvez créer un fichier **hello.ps1** avec le contenu suivant :

```powershell
param([string] $name)
$output = 'Hello {0}' -f $name
Write-Output $output
$DeploymentScriptOutputs = @{}
$DeploymentScriptOutputs['text'] = $output
```

## <a name="use-azure-container-instance"></a>Utiliser une instance de conteneur Azure

Pour créer vos scripts sur votre ordinateur, vous devez créer un compte de stockage et monter celui-ci dans l’instance de conteneur. Cela vous permettra de charger votre script dans le compte de stockage et d’exécuter le script sur l’instance de conteneur.

> [!NOTE]
> Le compte de stockage que vous créez pour tester votre script n’est pas le même que celui qu’utilise le service de script de déploiement pour exécuter le script. Le service de script de déploiement crée un nom unique sous forme de partage de fichiers à chaque exécution.

### <a name="create-an-azure-container-instance"></a>Créer une instance de conteneur Azure

Le modèle ARM suivant crée une instance de conteneur et un partage de fichiers, puis monte ce dernier dans l’image de conteneur.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "projectName": {
      "type": "string",
      "metadata": {
        "description": "Specify a project name that is used for generating resource names."
      }
    },
    "containerImage": {
      "type": "string",
      "defaultValue": "mcr.microsoft.com/azuredeploymentscripts-powershell:az4.3",
      "metadata": {
        "description": "Specify the container image."
      }
    },
    "mountPath": {
      "type": "string",
      "defaultValue": "deploymentScript",
      "metadata": {
        "description": "Specify the mount path."
      }
    }
  },
  "variables": {
    "storageAccountName": "[concat(parameters('projectName'), 'store')]",
    "fileShareName": "[concat(parameters('projectName'), 'share')]",
    "containerGroupName": "[concat(parameters('projectName'), 'cg')]",
    "containerName": "[concat(parameters('projectName'), 'container')]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-06-01",
      "name": "[variables('storageAccountName')]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS",
        "tier": "Standard"
      },
      "kind": "StorageV2",
      "properties": {
        "accessTier": "Hot"
      }
    },
    {
      "type": "Microsoft.Storage/storageAccounts/fileServices/shares",
      "apiVersion": "2019-06-01",
      "name": "[concat(variables('storageAccountName'), '/default/', variables('fileShareName'))]",
      "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
      ]
    },
    {
      "type": "Microsoft.ContainerInstance/containerGroups",
      "apiVersion": "2019-12-01",
      "name": "[variables('containerGroupName')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
      ],
      "properties": {
        "containers": [
          {
            "name": "[variables('containerName')]",
            "properties": {
              "image": "[parameters('containerImage')]",
              "resources": {
                "requests": {
                  "cpu": 1,
                  "memoryInGb": 1.5
                }
              },
              "ports": [
                {
                  "protocol": "TCP",
                  "port": 80
                }
              ],
              "volumeMounts": [
                {
                  "name": "filesharevolume",
                  "mountPath": "[parameters('mountPath')]"
                }
              ],
              "command": [
                "/bin/sh",
                "-c",
                "pwsh -c 'Start-Sleep -Seconds 1800'"
              ]
            }
          }
        ],
        "osType": "Linux",
        "volumes": [
          {
            "name": "filesharevolume",
            "azureFile": {
              "readOnly": false,
              "shareName": "[variables('fileShareName')]",
              "storageAccountName": "[variables('storageAccountName')]",
              "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), '2019-06-01').keys[0].value]"
            }
          }
        ]
      }
    }
  ]
}
```
La valeur par défaut du chemin de montage est **deploymentScript**.  Il s’agit du chemin dans l’instance de conteneur où il est monté dans le partage de fichiers.

L’image de conteneur par défaut spécifiée dans le modèle est **mcr.microsoft.com/azuredeploymentscripts-powershell:az4.3"** .  Pour obtenir la liste des versions prises en charge d’Azure PowerShell et d’Azure CLI, consultez [Azure PowerShell ou Azure CLI](./deployment-script-template.md#prerequisites).

Le modèle interrompt l’instance de conteneur 1 800 secondes. Il faut compter 30 minutes avant que l’instance de conteneur passe à l’état terminal et que la session prenne fin.

Pour déployer le modèle, procédez comme suit :

```azurepowershell
$projectName = Read-Host -Prompt "Enter a project name that is used to generate resource names"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$templateFile = Read-Host -Prompt "Enter the template file path and file name"
$resourceGroupName = "${projectName}rg"

New-azResourceGroup -Location $location -name $resourceGroupName
New-AzResourceGroupDeployment -resourceGroupName $resourceGroupName -TemplateFile $templatefile -projectName $projectName
```

### <a name="upload-deployment-script"></a>Charger le script de déploiement

Chargez votre script de déploiement dans le compte de stockage. Voici un exemple PowerShell :

```azurepowershell
$projectName = Read-Host -Prompt "Enter the same project name that you used earlier"
$fileName = Read-Host -Prompt "Enter the deployment script file name with the path"

$resourceGroupName = "${projectName}rg"
$storageAccountName = "${projectName}store"
$fileShareName = "${projectName}share"

$context = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName).Context
Set-AzStorageFileContent -Context $context -ShareName $fileShareName -Source $fileName -Force
```

Vous pouvez aussi charger le fichier à partir du portail Azure et d’Azure CLI.

### <a name="test-the-deployment-script"></a>Tester le script de déploiement

1. À partir du portail Azure, ouvrez le groupe de ressources dans lequel vous avez déployé l’instance de conteneur et le compte de stockage.
1. Ouvrez le groupe de conteneurs. Le nom du groupe de conteneurs par défaut est le nom du projet avec **cg** ajouté. L’instance de conteneur doit se trouver à l’état **En cours d’exécution**.
1. Sélectionnez **Conteneurs** dans le menu de gauche. Vous devez voir une instance de conteneur.  Le nom de l’instance de conteneur est le nom du projet avec **container** ajouté.

    ![script de déploiement connecter instance de conteneur](./media/deployment-script-template-configure-dev/deployment-script-container-instance-connect.png)

1. Sélectionnez **Connecter**, puis **Connecter**. Si vous ne pouvez pas vous connecter à l’instance de conteneur, redémarrez le groupe de conteneurs, puis réessayez.
1. Dans le volet de la console, exécutez la commande suivante :

    ```
    cd deploymentScript
    ls
    pwsh ./hello.ps1 "John Dole"
    ```

    La sortie est **Hello John Dole**.

    ![script de déploiement instance de conteneur test](./media/deployment-script-template-configure-dev/deployment-script-container-instance-test.png)

## <a name="use-docker"></a>Utiliser Docker

Vous pouvez utiliser une image conteneur d’ancrage préconfigurée comme environnement de développement de script de déploiement. Pour installer le Docker, consultez [Obtenir le Docker](https://docs.docker.com/get-docker/).
Vous devez aussi configurer le partage de fichiers pour monter le répertoire, qui contient les scripts de déploiement dans le conteneur Docker.

1. Extrayez l’image conteneur du script de déploiement pour la mettre sur l’ordinateur local :

    ```command
    docker pull mcr.microsoft.com/azuredeploymentscripts-powershell:az4.3
    ```

    L’exemple utilise la version 4.3.0 de PowerShell.

    Pour tirer (pull) une image CLI d’un registre Microsoft Container Registry (MCR) :

    ```command
    docker pull mcr.microsoft.com/azure-cli:2.0.80
    ```

    Cet exemple utilise la version 2.0.80 de CLI. Le script de déploiement utilise les images conteneurs CLI par défaut trouvées [ici](https://hub.docker.com/_/microsoft-azure-cli).

1. Exécutez l’image Docker localement.

    ```command
    docker run -v <host drive letter>:/<host directory name>:/data -it mcr.microsoft.com/azuredeploymentscripts-powershell:az4.3
    ```

    Remplacez **&lt;host driver letter>** (lettre de lecteur hôte) et **&lt;host directory name>** (nom de répertoire hôte) par un dossier existant sur le lecteur partagé.  Le dossier est alors mappé au dossier **/data** dans le conteneur. Par exemple, pour mapper D:\docker :

    ```command
    docker run -v d:/docker:/data -it mcr.microsoft.com/azuredeploymentscripts-powershell:az4.3
    ```

    **-it** indique de conserver l’image conteneur active.

    Exemple CLI :

    ```command
    docker run -v d:/docker:/data -it mcr.microsoft.com/azure-cli:2.0.80
    ```

1. La capture d’écran suivante illustre comment exécuter un script PowerShell, étant donné que vous avez un fichier helloworld.ps1 dans le dossier partagé.

    ![Commande Docker pour un script de déploiement de modèle Resource Manager](./media/deployment-script-template/resource-manager-deployment-script-docker-cmd.png)

Dès lors que le script a été testé avec succès, vous pouvez l’utiliser en tant que script de déploiement dans vos modèles.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à utiliser des scripts de déploiement. Pour suivre un tutoriel sur les scripts de déploiement :

> [!div class="nextstepaction"]
> [Tutoriel : Utiliser des scripts de déploiement dans des modèles Azure Resource Manager](./template-tutorial-deployment-script.md)

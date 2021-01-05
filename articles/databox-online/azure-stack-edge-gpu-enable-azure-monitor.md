---
title: Activer Azure Monitor sur un appareil Azure Stack Edge Pro avec GPU
description: Décrit comment activer Azure Monitor sur un appareil Azure Stack Edge Pro avec GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 11/05/2020
ms.author: alkohli
ms.openlocfilehash: 3485c6ca5c2672fa48b6118a78600b9745994ce1
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96465275"
---
# <a name="enable-azure-monitor-on-your-azure-stack-edge-pro-gpu-device"></a>Activer Azure Monitor sur votre appareil Azure Stack Edge Pro avec GPU

La surveillance des conteneurs sur votre appareil Azure Stack Edge Pro avec GPU est essentielle, notamment lorsque vous exécutez plusieurs applications de calcul. Azure Monitor vous permet de collecter les journaux de conteneur ainsi que les métriques de mémoire et de processeur à partir du cluster Kubernetes s’exécutant sur votre appareil.

Cet article décrit la procédure à suivre pour activer Azure Monitor sur votre appareil et collecter les journaux de conteneur dans l’espace de travail Log Analytics. Actuellement, le magasin de métriques Azure Monitor n’est pas pris en charge avec votre appareil Azure Stack Edge Pro avec GPU. 


## <a name="prerequisites"></a>Prérequis

Avant de commencer, vous avez besoin des éléments suivants :

- Un appareil Azure Stack Edge Pro. Assurez-vous que l’appareil est activé conformément aux étapes décrites dans [Tutoriel : Activer votre appareil](azure-stack-edge-gpu-deploy-activate.md).
- Vous avez terminé l’étape **Configurer le calcul** du [Tutoriel : Configurer le calcul sur votre appareil Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-configure-compute.md) sur votre propre appareil. Votre appareil doit être associé à une ressource IoT Hub, un appareil IoT et un appareil IoT Edge.


## <a name="create-log-analytics-workspace"></a>Créez un espace de travail Log Analytics.

Pour créer un espace de travail Log Analytics, suivez la procédure ci-dessous. Un espace de travail Log Analytics est une unité de stockage logique dans laquelle les données de journal sont collectées et stockées.

1. Dans le portail Azure, sélectionnez **+ Créer une ressource**, recherchez **Espace de travail Log Analytics**, puis sélectionnez **Créer**. 
1. Dans l’**espace de travail Log Analytics**, configurez les paramètres suivants. Acceptez le reste comme valeur par défaut.

    1. Sous l’onglet **Informations de base**, indiquez l’abonnement, le groupe de ressources, le nom et la région de l’espace de travail. 

        ![Onglet Informations de base de l’espace de travail Log Analytics](media/azure-stack-edge-gpu-enable-azure-monitor/create-log-analytics-workspace-basics-1.png)  

    1. Sous l’onglet **Niveau tarifaire**, acceptez le **plan de paiement à l'utilisation** par défaut.

        ![Onglet Niveau tarifaire de l’espace de travail Log Analytics](media/azure-stack-edge-gpu-enable-azure-monitor/create-log-analytics-workspace-pricing-1.png) 

    1. Sous l’onglet **Vérifier + créer**, vérifiez les informations et sélectionnez **Créer**.

        ![Onglet Vérifier + créer de l’espace de travail Log Analytics](media/azure-stack-edge-gpu-enable-azure-monitor/create-log-analytics-workspace-review-create-1.png)

Pour plus d’informations, consultez la procédure détaillée décrite dans [Créer un espace de travail Log Analytics via le portail Azure](../azure-monitor/learn/quick-create-workspace.md).



## <a name="enable-container-insights"></a>Activer Container Insights

Procédez comme suit pour activer Container Insights sur votre espace de travail. 

1. Suivez la procédure détaillée décrite dans [Comment ajouter la solution Azure Monitor pour conteneurs](../azure-monitor/insights/container-insights-hybrid-setup.md#how-to-add-the-azure-monitor-containers-solution). Utilisez le fichier modèle suivant `containerSolution.json` :

    ```yml
    {
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceResourceId": {
            "type": "string",
            "metadata": {
                "description": "Azure Monitor Log Analytics Workspace Resource ID"
            }
        },
        "workspaceRegion": {
            "type": "string",
            "metadata": {
                "description": "Azure Monitor Log Analytics Workspace region"
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.Resources/deployments",
            "name": "[Concat('ContainerInsights', '-',  uniqueString(parameters('workspaceResourceId')))]",
            "apiVersion": "2017-05-10",
            "subscriptionId": "[split(parameters('workspaceResourceId'),'/')[2]]",
            "resourceGroup": "[split(parameters('workspaceResourceId'),'/')[4]]",
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {},
                    "variables": {},
                    "resources": [
                        {
                            "apiVersion": "2015-11-01-preview",
                            "type": "Microsoft.OperationsManagement/solutions",
                            "location": "[parameters('workspaceRegion')]",
                            "name": "[Concat('ContainerInsights', '(', split(parameters('workspaceResourceId'),'/')[8], ')')]",
                            "properties": {
                                "workspaceResourceId": "[parameters('workspaceResourceId')]"
                            },
                            "plan": {
                                "name": "[Concat('ContainerInsights', '(', split(parameters('workspaceResourceId'),'/')[8], ')')]",
                                "product": "[Concat('OMSGallery/', 'ContainerInsights')]",
                                "promotionCode": "",
                                "publisher": "Microsoft"
                            }
                        }
                    ]
                },
                "parameters": {}
            }
            }
        ]
    }
    ```

1. Obtenez l’ID de ressource et l’emplacement. Atteindre `Your Log Analytics workspace > General > Properties`. Copiez les informations suivantes :

    - **ID de ressource** qui correspond à l’ID de ressource Azure complet de l’espace de travail Azure Log Analytics. 
    - **Emplacement** qui correspond à la région Azure.

    ![Propriétés de l’espace de travail Log Analytics](media/azure-stack-edge-gpu-enable-azure-monitor/log-analytics-workspace-properties-1.png) 

1. Utilisez le fichier de paramètres suivant `containerSolutionParams.json`. Remplacez `workspaceResourceId` par l’ID de ressource et `workspaceRegion` par l’emplacement copié à l’étape précédente.

    ```yaml
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
        "workspaceResourceId": {
            "value": "/subscriptions/fa68082f-8ff7-4a25-95c7-ce9da541242f/resourcegroups/myaserg/providers/microsoft.operationalinsights/workspaces/myaseloganalyticsws"
        },
        "workspaceRegion": {
        "value": "westus"
        }
        }
    }
    ```
    
    Voici un exemple de sortie d’espace de travail Log Analytics avec Container Insights activé :
    
    ```powershell
    Requesting a Cloud Shell.Succeeded.
    Connecting terminal...
    MOTD: Switch to Bash from PowerShell: bash
    VERBOSE: Authenticating to Azure ...
    VERBOSE: Building your Azure drive ...
    
    PS /home/alpa> az account set -s fa68082f-8ff7-4a25-95c7-ce9da541242f
    PS /home/alpa> ls
    clouddrive  containerSolution.json
    PS /home/alpa> ls
    clouddrive  containerSolution.json  containerSolutionParams.json
    PS /home/alpa> az deployment group create --resource-group myaserg --name Testdeployment1 --template-file containerSolution.json --parameters containerSolutionParams.json
    {- Finished ..
        "id": "/subscriptions/fa68082f-8ff7-4a25-95c7-ce9da541242f/resourceGroups/myaserg/providers/Microsoft.Resources/deployments/Testdeployment1",
        "location": null,
        "name": "Testdeployment1",
        "properties": {
        "correlationId": "3a9045fe-2de0-428c-b17b-057508a8c575",
        "debugSetting": null,
        "dependencies": [],
        "duration": "PT11.1588316S",
        "error": null,
        "mode": "Incremental",
        "onErrorDeployment": null,
        "outputResources": [
            {
            "id": "/subscriptions/fa68082f-8ff7-4a25-95c7-ce9da541242f/resourceGroups/myaserg/providers/Microsoft.OperationsManagement/solutions/ContainerInsights(myaseloganalyticsws)",
            "resourceGroup": "myaserg"
            }
        ],
        "outputs": null,
        "parameters": {
            "workspaceRegion": {
            "type": "String",
            "value": "westus"
            },
            "workspaceResourceId": {
            "type": "String",
            "value": "/subscriptions/fa68082f-8ff7-4a25-95c7-ce9da541242f/resourcegroups/myaserg/providers/microsoft.operationalinsights/workspaces/myaseloganalyticsws"
            }
        },
        "parametersLink": null,
        "providers": [
            {
            "id": null,
            "namespace": "Microsoft.Resources",
            "registrationPolicy": null,
            "registrationState": null,
            "resourceTypes": [
                {
                "aliases": null,
                "apiProfiles": null,
                "apiVersions": null,
                "capabilities": null,
                "defaultApiVersion": null,
                "locations": [
                    null
                ],
                "properties": null,
                "resourceType": "deployments"
                }
            ]
            }
        ],
        "provisioningState": "Succeeded",
        "templateHash": "10500027184662969395",
        "templateLink": null,
        "timestamp": "2020-11-06T22:09:56.908983+00:00",
        "validatedResources": null
        },
        "resourceGroup": "myaserg",
        "tags": null,
        "type": "Microsoft.Resources/deployments"
    }
    PS /home/alpa>
    ```

## <a name="configure-azure-monitor-on-your-device"></a>Configurer Azure Monitor sur votre appareil

1. Accédez à la ressource Log Analytics nouvellement créée et copiez l’**ID d’espace de travail** et la **Clé primaire** (clé d’espace de travail).

    ![Gestion des agents dans l’espace de travail Log Analytics](media/azure-stack-edge-gpu-enable-azure-monitor/log-analytics-workspace-agents-management-1.png)

    Enregistrez ces informations pour les utiliser à l’étape suivante.

1. [Connectez-vous à l'interface PowerShell de l'appareil](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface). 
    
1. Utilisez l’ID et la clé de l’espace de travail Log Analytics avec la cmdlet suivante :

    `Set-HcsKubernetesAzureMonitorConfiguration -WorkspaceId <> -WorkspaceKey <>`

1. Une fois Azure Monitor activé, vous devez voir les journaux dans l’espace de travail Log Analytics. Pour afficher l’état du cluster Kubernetes déployé sur votre appareil, accédez à **Azure Monitor > Insights > Containers**. Pour l’option d’environnement, sélectionnez **Tous**. 

    ![Métriques dans l’espace de travail Log Analytics](media/azure-stack-edge-gpu-enable-azure-monitor/log-analytics-workspace-metrics-1.png)

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [surveiller les charges de travail Kubernetes via le tableau de bord Kubernetes](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md).

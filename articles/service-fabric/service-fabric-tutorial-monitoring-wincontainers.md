---
title: Superviser et diagnostiquer les conteneurs Windows
description: Dans ce tutoriel, vous configurez les journaux Azure Monitor pour la surveillance et les diagnostics de conteneurs Windows sur Azure Service Fabric.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc
ms.openlocfilehash: cf14cce631a505a951ec4d9c0955431b9a98527e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91840674"
---
# <a name="tutorial-monitor-windows-containers-on-service-fabric-using-azure-monitor-logs"></a>Tutoriel : surveiller des conteneurs Windows sur Service Fabric avec les journaux Azure Monitor

Il s’agit de la troisième partie d’un tutoriel, qui vous guide dans la configuration des journaux Azure Monitor pour superviser vos conteneurs Windows orchestrés sur Service Fabric.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * configurer les journaux Azure Monitor pour votre cluster Service Fabric ;
> * utiliser un espace de travail Log Analytics pour afficher et interroger des journaux d’activité à partir de vos conteneurs et de vos nœuds ;
> * configurer l’agent Log Analytics pour collecter des mesures sur les conteneurs et les nœuds.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce didacticiel, vous devez :

* disposer d’un cluster sur Azure, ou en [créer un avec ce didacticiel](service-fabric-tutorial-create-vnet-and-windows-cluster.md) ;
* [y déployer une application en conteneur](service-fabric-host-app-in-a-container.md).

## <a name="setting-up-azure-monitor-logs-with-your-cluster-in-the-resource-manager-template"></a>Configurer les journaux Azure Monitor avec un cluster dans le modèle Resource Manager

Si vous avez utilisé le [modèle fourni](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-OMS-UnSecure) dans la première partie de ce didacticiel, il devrait normalement inclure les ajouts suivants à un modèle Azure Resource Manager Service Fabric générique. Si, au contraire, vous cherchez à configurer votre propre cluster pour surveiller des conteneurs avec les journaux Azure Monitor :

* Apportez les modifications suivantes à votre modèle Resource Manager.
* Déployez-le avec PowerShell pour mettre à niveau votre cluster en [déployant le modèle](./service-fabric-cluster-creation-via-arm.md). Azure Resource Manager se rend compte que la ressource existe, et par conséquent la déploie comme une mise à niveau.

### <a name="adding-azure-monitor-logs-to-your-cluster-template"></a>Ajout de journaux Azure Monitor à votre modèle de cluster

Apportez les modifications suivantes à votre *template.json* :

1. Ajoutez le nom et l’emplacement de l’espace de travail Log Analytics à votre section *parameters* :

    ```json
    "omsWorkspacename": {
      "type": "string",
      "defaultValue": "[toLower(concat('sf',uniqueString(resourceGroup().id)))]",
      "metadata": {
        "description": "Name of your Log Analytics Workspace"
      }
    },
    "omsRegion": {
      "type": "string",
      "defaultValue": "East US",
      "allowedValues": [
        "West Europe",
        "East US",
        "Southeast Asia"
      ],
      "metadata": {
        "description": "Specify the Azure Region for your Log Analytics workspace"
      }
    }
    ```

    Pour modifier l’une des deux valeurs utilisées, ajoutez les mêmes paramètres à votre *template.parameters.json* et modifiez-y les valeurs utilisées.

2. Ajouter le nom de la solution et la solution à vos *variables* :

    ```json
    "omsSolutionName": "[Concat('ServiceFabric', '(', parameters('omsWorkspacename'), ')')]",
    "omsSolution": "ServiceFabric"
    ```

3. Ajoutez Microsoft Monitoring Agent comme extension de machine virtuelle. Recherchez la ressource des groupes de machines virtuelles identiques : *resources* >  *"apiVersion": "[variables(’vmssApiVersion’)]"* . Sous *properties* > *virtualMachineProfile* > *extensionProfile* > *extensions*, ajoutez la description de l’extension suivante sous l’extension *ServiceFabricNode* : 
    
    ```json
    {
        "name": "[concat(variables('vmNodeType0Name'),'OMS')]",
        "properties": {
            "publisher": "Microsoft.EnterpriseCloud.Monitoring",
            "type": "MicrosoftMonitoringAgent",
            "typeHandlerVersion": "1.0",
            "autoUpgradeMinorVersion": true,
            "settings": {
                "workspaceId": "[reference(resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename')), '2015-11-01-preview').customerId]"
            },
            "protectedSettings": {
                "workspaceKey": "[listKeys(resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename')),'2015-11-01-preview').primarySharedKey]"
            }
        }
    },
    ```

4. Ajoutez l’espace de travail Log Analytics en tant que ressource individuelle. Dans *resources*, après la ressource des groupes de machines virtuelles identiques, ajoutez le code suivant :

    ```json
    {
        "apiVersion": "2015-11-01-preview",
        "location": "[parameters('omsRegion')]",
        "name": "[parameters('omsWorkspacename')]",
        "type": "Microsoft.OperationalInsights/workspaces",
        "properties": {
            "sku": {
                "name": "Free"
            }
        },
        "resources": [
            {
                "apiVersion": "2015-11-01-preview",
                "name": "[concat(variables('applicationDiagnosticsStorageAccountName'),parameters('omsWorkspacename'))]",
                "type": "storageinsightconfigs",
                "dependsOn": [
                    "[concat('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]",
                    "[concat('Microsoft.Storage/storageAccounts/', variables('applicationDiagnosticsStorageAccountName'))]"
                ],
                "properties": {
                    "containers": [ ],
                    "tables": [
                        "WADServiceFabric*EventTable",
                        "WADWindowsEventLogsTable",
                        "WADETWEventTable"
                    ],
                    "storageAccount": {
                        "id": "[resourceId('Microsoft.Storage/storageaccounts/', variables('applicationDiagnosticsStorageAccountName'))]",
                        "key": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('applicationDiagnosticsStorageAccountName')),'2015-06-15').key1]"
                    }
                }
            },
            {
                "apiVersion": "2015-11-01-preview",
                "name": "System",
                "type": "datasources",
                "dependsOn": [
                    "[concat('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]"
                ],
                "kind": "WindowsEvent",
                "properties": {
                    "eventLogName": "System",
                    "eventTypes": [
                        {
                            "eventType": "Error"
                        },
                        {
                            "eventType": "Warning"
                        },
                        {
                            "eventType": "Information"
                        }
                    ]
                }
            }
        ]
    },
    {
        "apiVersion": "2015-11-01-preview",
        "location": "[parameters('omsRegion')]",
        "name": "[variables('omsSolutionName')]",
        "type": "Microsoft.OperationsManagement/solutions",
        "dependsOn": [
            "[concat('Microsoft.OperationalInsights/workspaces/', parameters('OMSWorkspacename'))]"
        ],
        "properties": {
            "workspaceResourceId": "[resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]"
        },
        "plan": {
            "name": "[variables('omsSolutionName')]",
            "publisher": "Microsoft",
            "product": "[Concat('OMSGallery/', variables('omsSolution'))]",
            "promotionCode": ""
        }
    },
    ```

Vous trouverez [ici](https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/d2ffa318581fc23ac7f1b0ab2b52db1a0d7b4ba7/5-VM-Windows-OMS-UnSecure/sfclusteroms.json) un exemple de modèle (utilisé dans la première partie de ce didacticiel) comportant toutes ces modifications, que vous pouvez référencer selon vos besoins. Ces modifications ajouteront un espace de travail Log Analytics à votre groupe de ressources. L’espace de travail sera configuré pour collecter les événements de la plateforme Service Fabric à partir des tables de stockage configurées avec l’agent [Diagnostics Azure pour Windows](service-fabric-diagnostics-event-aggregation-wad.md). L’agent Log Analytics (Microsoft Monitoring Agent) a également été ajouté à chaque nœud de votre cluster comme extension de machine virtuelle. Ainsi, quand vous mettez à l’échelle votre cluster, l’agent est automatiquement configuré sur chaque machine et raccordé au même espace de travail.

Déployez le modèle avec vos nouvelles modifications pour mettre à niveau votre cluster actuel. Vous devriez voir les ressources Log Analytics dans votre groupe de ressources une fois le processus terminé. Lorsque le cluster est prêt, déployez-y votre application en conteneur. À l’étape suivante, nous allons configurer le monitorage des conteneurs.

## <a name="add-the-container-monitoring-solution-to-your-log-analytics-workspace"></a>Ajouter la solution de supervision de conteneurs à un espace de travail Log Analytics

Pour configurer la solution Conteneur dans votre espace de travail, recherchez *Solution de supervision de conteneurs* et créez une ressource Conteneurs (sous la catégorie Monitorage + gestion).

![Ajout de la solution Conteneurs](./media/service-fabric-tutorial-monitoring-wincontainers/containers-solution.png)

Quand vous êtes invité à choisir l’*espace de travail Log Analytics*, sélectionnez l’espace de travail qui a été créé dans votre groupe de ressources, puis sélectionnez **Créer**. Une *solution de supervision de conteneurs* est alors ajoutée à votre espace de travail, et l’agent Log Analytics déployé par le modèle commence à collecter les journaux d’activité et les statistiques Docker.

Revenez à votre *groupe de ressources*, où vous devriez à présent voir la solution de supervision qui vient d’être ajoutée. Si vous la sélectionnez, la page de destination doit afficher le nombre d’images conteneurs en cours d’exécution.

*Notez que nous avons exécuté cinq instances du conteneur fabrikam de la [deuxième partie](service-fabric-host-app-in-a-container.md) du tutoriel*

![Page de destination de la solution Conteneur](./media/service-fabric-tutorial-monitoring-wincontainers/solution-landing.png)

Si vous sélectionnez la **solution de supervision de conteneurs**, vous accédez à un tableau de bord plus détaillé, ce qui vous permet de parcourir plusieurs panneaux et d’exécuter des requêtes dans les journaux Azure Monitor.

Étant donné que l’agent collecte des journaux d’activité Docker, il affiche par défaut *stdout* et *stderr*. Si vous faites défiler horizontalement, vous pouvez voir l’inventaire des images conteneur, l’état, les métriques et des exemples de requêtes que vous pouvez exécuter pour obtenir d’autres données utiles.

![Tableau de bord de la solution Conteneur](./media/service-fabric-tutorial-monitoring-wincontainers/container-metrics.png)

Si vous cliquez sur l’un de ces panneaux, vous accéderez à la requête Kusto qui génère la valeur affichée. Modifiez la requête avec *\** pour afficher les différents types de journaux d’activité collectés. À ce stade, vous pouvez interroger ou filtrer sur les performances du conteneur ou les journaux d’activité, ou bien examiner les événements de la plateforme Service Fabric. Vos agents émettent en permanence une pulsation sur chaque nœud, que vous pouvez examiner pour vérifier que les données sont toujours collectées sur toutes vos machines si votre configuration de clusters change.

![Requête de conteneur](./media/service-fabric-tutorial-monitoring-wincontainers/query-sample.png)

## <a name="configure-log-analytics-agent-to-pick-up-performance-counters"></a>Configurer l’agent Log Analytics de façon à collecter des compteurs de performances

Le fait de pouvoir modifier les compteurs de performances que vous souhaitez collecter par le biais de l’expérience utilisateur Log Analytics est un autre avantage de l’agent Log Analytics. Vous n’êtes donc pas obligé de configurer l’agent de diagnostics Azure ni d’effectuer une mise à niveau basée sur le modèle Resource Manager à chaque fois. Pour cela, sélectionnez l’**espace de travail OMS** dans la page de destination de votre solution de supervision de conteneurs (ou Service Fabric).

Vous accédez ainsi à votre espace de travail Log Analytics où vous pouvez consulter vos solutions, créer des tableaux de bord personnalisés et configurer l’agent Log Analytics. 
* Sélectionnez **Paramètres avancés** pour ouvrir le menu Paramètres avancés.
* Sélectionnez **Sources connectées** > **Serveurs Windows** pour vérifier que vous avez *cinq ordinateurs Windows connectés*.
* Sélectionnez **Données** > **Compteurs de performances Windows** pour rechercher et ajouter de nouveaux compteurs de performances. Vous verrez une liste de recommandations, fournie par les journaux Azure Monitor, de compteurs de performances que vous pouvez recueillir. Vous avez également la possibilité de rechercher d’autres compteurs. Vérifiez que les compteurs **Processor(_Total)\% Processor Time** et **Memory(*)\Available MBytes** sont collectés.

**Actualisez** votre solution de supervision de conteneurs après quelques minutes : vous devriez commencer à voir arriver des données *Performances de l’ordinateur*. Cela vous aidera à comprendre l’utilisation des ressources. Vous pouvez également utiliser ces mesures pour prendre les décisions adaptées quant à la mise à l’échelle de votre cluster, ou pour vérifier qu’un cluster équilibre votre charge comme prévu.

*Remarque : Vérifiez que vos filtres temporels sont définis de façon à vous permettre d’utiliser ces mesures.*

![Compteurs de performances 2](./media/service-fabric-tutorial-monitoring-wincontainers/perf-counters2.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * configurer les journaux Azure Monitor pour votre cluster Service Fabric ;
> * utiliser un espace de travail Log Analytics pour afficher et interroger des journaux d’activité à partir de vos conteneurs et de vos nœuds ;
> * configurer l’agent Log Analytics pour collecter des mesures sur les conteneurs et les nœuds.

Maintenant que vous avez configuré la supervision de votre application en conteneur, essayez les opérations suivantes :

* Configurez les journaux Azure Monitor pour un cluster Linux en suivant les mêmes étapes que ce tutoriel. Référencez [ce modèle](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Ubuntu-1-NodeType-Secure-OMS) pour apporter des modifications à votre modèle Resource Manager.
* Configurez les journaux Azure Monitor pour paramétrer l’[alerte automatisée](../azure-monitor/platform/alerts-overview.md) afin de faciliter la détection et les diagnostics.
* Explorez la liste Service Fabric de [compteurs de performances recommandés](service-fabric-diagnostics-event-generation-perf.md) à configurer pour vos clusters.
* Familiarisez-vous avec les fonctionnalités de [requêtes et recherches dans les journaux](../azure-monitor/log-query/log-query-overview.md) proposées par les journaux Azure Monitor.

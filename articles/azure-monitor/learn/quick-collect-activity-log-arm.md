---
title: Envoyer le journal d’activité Azure à un espace de travail Log Analytics à l’aide d’un modèle Azure Resource Manager
description: Utilisez des modèles Resource Manager pour créer un espace de travail Log Analytics et un paramètre de diagnostic afin d’envoyer le journal d’activité à Journaux Azure Monitor.
ms.subservice: logs
ms.topic: quickstart
ms.custom: subject-armqs, devx-track-azurecli
author: bwren
ms.author: bwren
ms.date: 06/25/2020
ms.openlocfilehash: abcaee8cffd36e22c918b9d82f37370df1d61c26
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92747030"
---
# <a name="quickstart-send-azure-activity-log-to-log-analytics-workspace-using-an-arm-template"></a>Démarrage rapide : Envoyer le journal d’activité Azure à un espace de travail Log Analytics à l’aide d’un modèle Resource Manager

Le journal d’activité est un journal de plateforme dans Azure qui fournit un aperçu de tous les événements de niveau d’abonnement. Les informations qu’il contient indiquent par exemple à quel moment une ressource a été modifiée ou une machine virtuelle a été démarrée. Vous pouvez afficher le journal d’activité dans le portail Azure ou récupérer des entrées avec PowerShell et l’interface CLI. Ce guide de démarrage rapide montre comment utiliser des modèles Azure Resource Manager (modèles ARM) pour créer un espace de travail Log Analytics et un paramètre de diagnostic afin d’envoyer le journal d’activité à Journaux Azure Monitor, où vous pouvez l’analyser à l’aide de [requêtes de journal](../log-query/log-query-overview.md) et activer d’autres fonctionnalités telles que les [alertes de journal](../platform/alerts-log-query.md) et les [classeurs](../platform/workbooks-overview.md).

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

## <a name="prerequisites"></a>Prérequis

- Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.
- Pour exécuter les commandes à partir de votre ordinateur local, installez Azure CLI ou les modules Azure PowerShell. Pour plus d’informations, consultez [Installer l’interface de ligne de commande Azure](/cli/azure/install-azure-cli) et [Installer Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="create-a-log-analytics-workspace"></a>Créer un espace de travail Log Analytics

### <a name="review-the-template"></a>Vérifier le modèle

Le modèle suivant crée un espace de travail Log Analytics vide. Enregistrez ce modèle sous le nom *CreateWorkspace.json* .

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "workspaceName": {
      "type": "string",
      "metadata": {
        "description": "Name of the workspace."
      }
    },
    "sku": {
      "type": "string",
      "defaultValue": "pergb2018",
      "allowedValues": [
        "pergb2018",
        "Free",
        "Standalone",
        "PerNode",
        "Standard",
        "Premium"
      ],
      "metadata": {
        "description": "Pricing tier: PerGB2018 or legacy tiers (Free, Standalone, PerNode, Standard or Premium) which are not available to all customers."
      }
    },
    "location": {
      "type": "string",
      "allowedValues": [
        "australiacentral",
        "australiaeast",
        "australiasoutheast",
        "brazilsouth",
        "canadacentral",
        "centralindia",
        "centralus",
        "eastasia",
        "eastus",
        "eastus2",
        "francecentral",
        "japaneast",
        "koreacentral",
        "northcentralus",
        "northeurope",
        "southafricanorth",
        "southcentralus",
        "southeastasia",
        "switzerlandnorth",
        "switzerlandwest",
        "uksouth",
        "ukwest",
        "westcentralus",
        "westeurope",
        "westus",
        "westus2"
      ],
      "metadata": {
        "description": "Specifies the location for the workspace."
      }
    },
    "retentionInDays": {
      "type": "int",
      "defaultValue": 120,
      "metadata": {
        "description": "Number of days to retain data."
      }
    },
    "resourcePermissions": {
      "type": "bool",
      "defaultValue": true,
      "metadata": {
        "description": "true to use resource or workspace permissions. false to require workspace permissions."
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.OperationalInsights/workspaces",
      "apiVersion": "2020-08-01",
      "name": "[parameters('workspaceName')]",
      "location": "[parameters('location')]",
      "properties": {
        "sku": {
          "name": "[parameters('sku')]"
        },
        "retentionInDays": "[parameters('retentionInDays')]",
        "features": {
          "searchVersion": 1,
          "legacy": 0,
          "enableLogAccessUsingOnlyResourcePermissions": "[parameters('resourcePermissions')]"
        }
      }
    }
  ]
}
```

Ce modèle définit une ressource :

- [Microsoft.OperationalInsights/workspaces](/azure/templates/microsoft.operationalinsights/workspaces)

### <a name="deploy-the-template"></a>Déployer le modèle

Déployez le modèle à l’aide de n’importe quelle méthode standard de [déploiement d’un modèle Resource Manager](../../azure-resource-manager/templates/deploy-portal.md), à l’image des exemples suivants qui utilisent l’interface CLI et PowerShell. Remplacez les exemples de valeurs **Resource Group** , **workspaceName** et **location** par les valeurs appropriées pour votre environnement. Le nom de l’espace de travail doit être unique parmi tous les abonnements Azure.

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

```azurecli
az login
az deployment group create \
    --name CreateWorkspace \
    --resource-group my-resource-group \
    --template-file CreateWorkspace.json \
    --parameters workspaceName='my-workspace-01' location='eastus'

```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName my-subscription
New-AzResourceGroupDeployment -Name AzureMonitorDeployment -ResourceGroupName my-resource-group -TemplateFile CreateWorkspace.json -workspaceName my-workspace-01 -location eastus
```

---

### <a name="validate-the-deployment"></a>Valider le déploiement

Vérifiez que l’espace de travail a été créé à l’aide de l’une des commandes suivantes. Remplacez les exemples de valeurs **Resource Group** et **workspaceName** par les valeurs que vous avez utilisées ci-dessus.

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

```azurecli
az monitor log-analytics workspace show --resource-group my-workspace-01 --workspace-name my-resource-group
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
Get-AzOperationalInsightsWorkspace -Name my-workspace-01 -ResourceGroupName my-resource-group
```

---

## <a name="create-diagnostic-setting"></a>Créer un paramètre de diagnostic

### <a name="review-the-template"></a>Vérifier le modèle

Le modèle suivant crée un paramètre de diagnostic qui envoie le journal d’activité à un espace de travail Log Analytics. Enregistrez ce modèle sous le nom *CreateDiagnosticSetting.json* .

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "settingName": {
        "type": "String"
    },
    "workspaceId": {
        "type": "String"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Insights/diagnosticSettings",
      "apiVersion": "2017-05-01-preview",
      "name": "[parameters('settingName')]",
      "dependsOn": [],
      "properties": {
        "workspaceId": "[parameters('workspaceId')]",
        "logs": [
          {
          "category": "Administrative",
          "enabled": true
          },
          {
          "category": "Alert",
          "enabled": true
          },
          {
          "category": "Autoscale",
          "enabled": true
          },
          {
          "category": "Policy",
          "enabled": true
          },
          {
          "category": "Recommendation",
          "enabled": true
          },
          {
          "category": "ResourceHealth",
          "enabled": true
          },
          {
          "category": "Security",
          "enabled": true
          },
          {
          "category": "ServiceHealth",
          "enabled": true
          }
        ]
      }
    }
  ]
}
```

Ce modèle définit une ressource :

- [Microsoft.Insights/diagnosticSettings](/azure/templates/microsoft.insights/diagnosticsettings)

### <a name="deploy-the-template"></a>Déployer le modèle

Déployez le modèle à l’aide de n’importe quelle méthode standard de [déploiement d’un modèle Resource Manager](../../azure-resource-manager/templates/deploy-portal.md), à l’image des exemples suivants qui utilisent l’interface CLI et PowerShell. Remplacez les exemples de valeurs **Resource Group** , **workspaceName** et **location** par les valeurs appropriées pour votre environnement. Le nom de l’espace de travail doit être unique parmi tous les abonnements Azure.

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

```azurecli
az deployment sub create --name CreateDiagnosticSetting --location eastus --template-file CreateDiagnosticSetting.json --parameters settingName='Send Activity log to workspace' workspaceId='/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/my-resource-group/providers/microsoft.operationalinsights/workspaces/my-workspace-01'

```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
New-AzSubscriptionDeployment -Name CreateDiagnosticSetting -location eastus -TemplateFile CreateDiagnosticSetting.json -settingName="Send Activity log to workspace" -workspaceId "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/my-resource-group/providers/microsoft.operationalinsights/workspaces/my-workspace-01"
```
---

### <a name="validate-the-deployment"></a>Valider le déploiement

Vérifiez que le paramètre de diagnostic a été créé à l’aide de l’une des commandes suivantes. Remplacez les exemples de valeurs pour l’abonnement et le nom du paramètre par les valeurs que vous avez utilisées ci-dessus.

> [!NOTE]
> Vous ne pouvez pas récupérer les paramètres de diagnostic de niveau d’abonnement à l’aide de PowerShell.

```azurecli
az monitor diagnostic-settings show --resource '/subscriptions/00000000-0000-0000-0000-000000000000' --name 'Send Activity log to workspace'
```

## <a name="generate-log-data"></a>Générer des données de journal

Seules les nouvelles entrées du journal d’activité étant envoyées à l’espace de travail Log Analytics, effectuez certaines actions dans votre abonnement qui seront journalisées, telles que le démarrage ou l’arrêt d’une machine virtuelle ou la création ou la modification d’une autre ressource. Vous devrez peut-être attendre quelques minutes pour que le paramètre de diagnostic soit créé et pour que les données soient écrites initialement dans l’espace de travail. Après ce délai, tous les événements écrits dans le journal d’activité sont envoyés à l’espace de travail en quelques secondes.

## <a name="retrieve-data-with-a-log-query"></a>Récupérer des données à l’aide d’une requête de journal

Utilisez le portail Azure pour récupérer des données de l’espace de travail à l’aide de Log Analytics. Dans le portail Azure, recherchez, puis sélectionnez **Monitor** .

![Portail Azure](media/quick-collect-activity-log/azure-portal-monitor.png)

Sélectionnez **Journaux** dans le menu **Azure Monitor** . Fermez la page **Exemples de requêtes** . Si l’étendue n’est pas définie sur l’espace de travail que vous avez créé, cliquez sur **Sélectionner une étendue** et recherchez-le.

![Étendue Log Analytics](media/quick-collect-activity-log/log-analytics-scope.png)

Dans la fenêtre de requête, tapez `AzureActivity`, puis cliquez sur **Exécuter** . Il s’agit d’une requête simple qui retourne tous les enregistrements de la table *AzureActivity* , qui contient tous les enregistrements envoyés à partir du journal d’activité.

![Requête simple](media/quick-collect-activity-log/query-01.png)

Développez l’un des enregistrements pour voir ses propriétés détaillées.

![Développer les propriétés](media/quick-collect-activity-log/expand-properties.png)

Essayez une requête plus complexe, telle que `AzureActivity | summarize count() by CategoryValue` qui donne un nombre d’événements résumés par catégorie.

![Requête complexe](media/quick-collect-activity-log/query-02.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous prévoyez d’utiliser d’autres guides de démarrage rapide et tutoriels, vous pouvez conserver ces ressources. Si vous n’en avez plus besoin, supprimez le groupe de ressources, ce qui va supprimer la règle d’alerte et les ressources associées. Pour supprimer le groupe de ressources avec Azure CLI ou Azure PowerShell

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

```azurecli
az group delete --name my-resource-group
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
Remove-AzResourceGroup -Name my-resource-group
```

---

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez configuré le journal d’activité à envoyer à un espace de travail Log Analytics. Vous pouvez à présent configurer d’autres données à collecter dans l’espace de travail, où vous pouvez les analyser à l’aide de [requêtes de journal](../log-query/log-query-overview.md) dans Azure Monitor et tirer parti de fonctionnalités telles que les [alertes de journal](../platform/alerts-log-query.md) et les [classeurs ](../platform/workbooks-overview.md). Ensuite, vous devriez collecter les [journaux des ressources](../platform/resource-logs.md) à partir de vos ressources Azure, qui complètent les données du journal d’activité en fournissant un insight sur les opérations qui ont été effectuées au sein de chaque ressource.

> [!div class="nextstepaction"]
> [Collecter et analyser des journaux de ressources avec Azure Monitor](tutorial-resource-logs.md)

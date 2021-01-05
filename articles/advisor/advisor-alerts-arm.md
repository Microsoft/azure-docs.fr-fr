---
title: Créer des alertes Azure Advisor pour les nouvelles recommandations à l’aide d’un modèle Resource Manager
description: Découvrez comment configurer une alerte pour de nouvelles recommandations d’Azure Advisor en utilisant un modèle Resource Manager (modèle ARM).
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 06/29/2020
ms.openlocfilehash: f03bf6eaf4f3045e00fc67efe6faa9f53d962089
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91629894"
---
# <a name="quickstart-create-azure-advisor-alerts-on-new-recommendations-using-an-arm-template"></a>Démarrage rapide : Créer des alertes Azure Advisor sur les nouvelles recommandations à l’aide d’un modèle Resource Manager

Cet article explique comment configurer une alerte pour les nouvelles recommandations à partir d’Azure Advisor à l’aide d’un modèle Resource Manager (modèle ARM).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Chaque fois qu’Azure Advisor détecte une nouvelle recommandation pour l’une de vos ressources, un événement est stocké dans le [journal d’activité Azure](../azure-monitor/platform/platform-logs-overview.md). Vous pouvez configurer des alertes pour ces événements à partir d’Azure Advisor en utilisant une expérience de création d’alertes spécifique aux recommandations. Vous pouvez sélectionner un abonnement et éventuellement un groupe de ressources afin de spécifier les ressources pour lesquelles vous souhaitez recevoir des alertes.

Vous pouvez également déterminer les types de recommandations à l’aide des propriétés suivantes :

- Category
- Niveau d’impact
- Type de recommandation

Vous pouvez également configurer l’action à entreprendre lorsqu’une alerte est déclenchée par :

- La sélection d’un groupe d’actions existant
- La création d’un nouveau groupe d’actions

Pour en savoir plus sur les groupes d’actions, consultez [Créer et gérer des groupes d’actions](../azure-monitor/platform/action-groups.md).

> [!NOTE]
> Les alertes Advisor sont actuellement disponibles uniquement pour les recommandations relatives à la haute disponibilité, aux performances et aux coûts. Les recommandations de sécurité ne sont pas prises en charge.

## <a name="prerequisites"></a>Prérequis

- Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.
- Pour exécuter les commandes à partir de votre ordinateur local, installez Azure CLI ou les modules Azure PowerShell. Pour plus d’informations, consultez [Installer l’interface de ligne de commande Azure](/cli/azure/install-azure-cli) et [Installer Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle suivant crée un groupe d’actions avec une cible de messagerie et active toutes les notifications d’intégrité de service pour l’abonnement cible. Enregistrez ce modèle sous le nom *CreateAdvisorAlert.json*.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "actionGroups_name": {
      "defaultValue": "advisorAlert",
      "type": "string"
    },
    "activityLogAlerts_name": {
      "defaultValue": "AdvisorAlertsTest",
      "type": "string"
    },
    "emailAddress": {
      "defaultValue": "<email address>",
      "type": "string"
    }
  },
  "variables": {
    "alertScope": "[concat('/','subscriptions','/',subscription().subscriptionId)]"
  },
  "resources": [
    {
      "comments": "Action Group",
      "type": "microsoft.insights/actionGroups",
      "apiVersion": "2019-06-01",
      "name": "[parameters('actionGroups_name')]",
      "location": "Global",
      "scale": null,
      "dependsOn": [],
      "tags": {},
      "properties": {
        "groupShortName": "[parameters('actionGroups_name')]",
        "enabled": true,
        "emailReceivers": [
          {
            "name": "[parameters('actionGroups_name')]",
            "emailAddress": "[parameters('emailAddress')]"
          }
        ],
        "smsReceivers": [],
        "webhookReceivers": []
      }
    },
    {
      "comments": "Azure Advisor Activity Log Alert",
      "type": "microsoft.insights/activityLogAlerts",
      "apiVersion": "2017-04-01",
      "name": "[parameters('activityLogAlerts_name')]",
      "location": "Global",
      "scale": null,
      "tags": {},
      "properties": {
        "scopes": [
          "[variables('alertScope')]"
        ],
        "condition": {
          "allOf": [
            {
              "field": "category",
              "equals": "Recommendation"
            },
            {
              "field": "properties.recommendationCategory",
              "equals": "Cost"
            },
            {
              "field": "properties.recommendationImpact",
              "equals": "Medium"
            },
            {
              "field": "operationName",
              "equals": "Microsoft.Advisor/recommendations/available/action"
            }
          ]
        },
        "actions": {
          "actionGroups": [
            {
              "actionGroupId": "[resourceId('microsoft.insights/actionGroups', parameters('actionGroups_name'))]",
              "webhookProperties": {}
            }
          ]
        },
        "enabled": true,
        "description": ""
      },
      "dependsOn": [
        "[resourceId('microsoft.insights/actionGroups', parameters('actionGroups_name'))]"
      ]
    }
  ]
}
```

Le modèle définit deux ressources :

- [Microsoft.Insights/actionGroups](/azure/templates/microsoft.insights/actiongroups)
- [Microsoft.Insights/activityLogAlerts](/azure/templates/microsoft.insights/activityLogAlerts)

## <a name="deploy-the-template"></a>Déployer le modèle

Déployez le modèle à l’aide de n’importe quelle méthode standard de [déploiement d’un modèle Resource Manager](../azure-resource-manager/templates/deploy-portal.md), à l’image des exemples suivants qui utilisent l’interface CLI et PowerShell. Remplacez les exemples de valeurs **Resource Group** et **emailAddress** par les valeurs appropriées pour votre environnement. Le nom de l’espace de travail doit être unique parmi tous les abonnements Azure.

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

```azurecli
az login
az deployment group create --name CreateAdvisorAlert --resource-group my-resource-group --template-file CreateAdvisorAlert.json --parameters emailAddress='user@contoso.com'
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName my-subscription
New-AzResourceGroupDeployment -Name CreateAdvisorAlert -ResourceGroupName my-resource-group -TemplateFile CreateAdvisorAlert.json -emailAddress user@contoso.com
```

---

## <a name="validate-the-deployment"></a>Valider le déploiement

Vérifiez que l’espace de travail a été créé à l’aide de l’une des commandes suivantes. Remplacez les exemples de valeurs **Resource Group** par la valeur que vous avez utilisée ci-dessus.

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

```azurecli
az monitor activity-log alert show --resource-group my-resource-group --name AdvisorAlertsTest
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
Get-AzActivityLogAlert -ResourceGroupName my-resource-group -Name AdvisorAlertsTest
```

---

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

- Obtenir une [vue d’ensemble des alertes du journal d’activité](../azure-monitor/platform/alerts-overview.md) et découvrir comment recevoir des alertes.
- En savoir plus sur les [groupes d’actions](../azure-monitor/platform/action-groups.md).

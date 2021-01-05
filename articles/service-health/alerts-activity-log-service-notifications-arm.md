---
title: Recevoir des alertes de journal d’activité sur les notifications de service Azure à l’aide d’un modèle Resource Manager
description: Soyez informé par SMS, e-mail ou webhook en cas de service Azure.
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 06/29/2020
ms.openlocfilehash: 688314a2057964c66baeacbbc49736ea436f5ec5
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91630217"
---
# <a name="quickstart-create-activity-log-alerts-on-service-notifications-using-an-arm-template"></a>Démarrage rapide : Créer des alertes de journal d’activité sur les notifications de service à l’aide d’un modèle Resource Manager

Cet article vous indique comment configurer des alertes de journal d’activité pour les notifications sur l’intégrité du service à l’aide d’un modèle Azure Resource Manager (modèle ARM).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Les notifications sur l’intégrité du service sont stockées dans le [journal d’activité Azure](../azure-monitor/platform/platform-logs-overview.md). Dans la mesure où le volume d’informations stockées dans le journal d’activité peut être important, il existe une interface utilisateur distincte permettant de faciliter l’affichage et la configuration d’alertes sur ces notifications.

Vous pouvez recevoir une alerte lorsqu’Azure envoie des notifications sur l’état du service sur votre abonnement Azure. Vous pouvez configurer l’alerte en fonction des éléments suivants :

- la classe de la notification sur l’intégrité du service (problèmes de service, maintenance planifiée, avis d’intégrité) ;
- l’abonnement affecté ;
- le ou les services affectés ;
- la ou les régions affectées ;

> [!NOTE]
> Les notifications d’intégrité de service n’envoient pas d’alerte sur les événements d’intégrité des ressources.

Vous pouvez également configurer à qui l’alerte doit être envoyée :

- sélectionner un groupe d’actions existant ;
- créer un nouveau groupe d’actions (qui peut être utilisé pour les alertes futures).

Pour en savoir plus sur les groupes d’actions, consultez [Créer et gérer des groupes d’actions](../azure-monitor/platform/action-groups.md).

## <a name="prerequisites"></a>Prérequis

- Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.
- Pour exécuter les commandes à partir de votre ordinateur local, installez Azure CLI ou les modules Azure PowerShell. Pour plus d’informations, consultez [Installer l’interface de ligne de commande Azure](/cli/azure/install-azure-cli) et [Installer Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle suivant crée un groupe d’actions avec une cible de messagerie et active toutes les notifications d’intégrité de service pour l’abonnement cible. Enregistrez ce modèle sous le nom *CreateServiceHealthAlert.json*.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "actionGroups_name": {
      "type": "String",
      "defaultValue": "SubHealth"
    },
    "activityLogAlerts_name": {
      "type": "String",
      "defaultValue": "ServiceHealthActivityLogAlert"
    },
    "emailAddress": {
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
      "comments": "Service Health Activity Log Alert",
      "type": "microsoft.insights/activityLogAlerts",
      "apiVersion": "2017-04-01",
      "name": "[parameters('activityLogAlerts_name')]",
      "location": "Global",
      "scale": null,
      "dependsOn": [
        "[resourceId('microsoft.insights/actionGroups', parameters('actionGroups_name'))]"
      ],
      "tags": {},
      "properties": {
        "scopes": [
          "[variables('alertScope')]"
        ],
        "condition": {
          "allOf": [
            {
              "field": "category",
              "equals": "ServiceHealth"
            },
            {
              "field": "properties.incidentType",
              "equals": "Incident"
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
      }
    }
  ]
}
```

Le modèle définit deux ressources :

- [Microsoft.Insights/actionGroups](/azure/templates/microsoft.insights/actiongroups)
- [Microsoft.Insights/activityLogAlerts](/azure/templates/microsoft.insights/activityLogAlerts)

## <a name="deploy-the-template"></a>Déployer le modèle

Déployez le modèle à l’aide de n’importe quelle méthode standard de [déploiement d’un modèle Resource Manager](../azure-resource-manager/templates/deploy-portal.md), à l’image des exemples suivants qui utilisent l’interface CLI et PowerShell. Remplacez les exemples de valeurs **Resource Group** et **emailAddress** par les valeurs appropriées pour votre environnement.

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

```azurecli
az login
az deployment group create --name CreateServiceHealthAlert --resource-group my-resource-group --template-file CreateServiceHealthAlert.json --parameters emailAddress='user@contoso.com'
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName my-subscription
New-AzResourceGroupDeployment -Name CreateServiceHealthAlert -ResourceGroupName my-resource-group -TemplateFile CreateServiceHealthAlert.json -emailAddress user@contoso.com
```

---

## <a name="validate-the-deployment"></a>Valider le déploiement

Vérifiez que l’espace de travail a été créé à l’aide de l’une des commandes suivantes. Remplacez les exemples de valeurs **Resource Group** par la valeur que vous avez utilisée ci-dessus.

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

```azurecli
az monitor activity-log alert show --resource-group my-resource-group --name ServiceHealthActivityLogAlert
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
Get-AzActivityLogAlert -ResourceGroupName my-resource-group -Name ServiceHealthActivityLogAlert
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

- Découvrez les [meilleures pratiques de configuration d’alertes Azure Service Health](https://www.microsoft.com/en-us/videoplayer/embed/RE2OtUa).
- Découvrez comment [configurer des notifications Push mobiles pour Azure Service Health](https://www.microsoft.com/en-us/videoplayer/embed/RE2OtUw).
- Découvrez comment [configurer des notifications de Webhook pour les systèmes de gestion de problème existants](service-health-alert-webhook-guide.md).
- En savoir plus sur les [notifications sur l’intégrité du service](service-notifications.md).
- En savoir plus sur la [limitation du débit des notifications](../azure-monitor/platform/alerts-rate-limiting.md).
- Consultez le [schéma webhook des alertes de journal d’activité](../azure-monitor/platform/activity-log-alerts-webhook.md).
- Obtenir une [vue d’ensemble des alertes du journal d’activité](../azure-monitor/platform/alerts-overview.md) et découvrir comment recevoir des alertes.
- En savoir plus sur les [groupes d’actions](../azure-monitor/platform/action-groups.md).

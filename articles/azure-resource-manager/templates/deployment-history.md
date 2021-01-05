---
title: Historique de déploiement
description: Décrit comment afficher les opérations de déploiement d’Azure Resource Manager avec le portail, PowerShell, la CLI Azure et l’API REST.
tags: top-support-issue
ms.topic: conceptual
ms.date: 09/23/2020
ms.openlocfilehash: 00e0c51244d5c191d4c9f05f689b90ece81ec5a4
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91284616"
---
# <a name="view-deployment-history-with-azure-resource-manager"></a>Afficher l’historique des déploiements avec Azure Resource Manager

Azure Resource Manager vous permet de voir l’historique de votre déploiement. Vous pouvez examiner des opérations spécifiques dans des déploiements antérieurs et voir quelles ressources ont été déployées. Cet historique contient des informations sur les erreurs éventuelles.

L’historique de déploiement de chaque groupe de ressources est limité à 800 déploiements. Quand vous vous approchez de la limite, les déploiements sont automatiquement supprimés de l’historique. Pour plus d’informations, consultez [Suppressions automatiques de l’historique de déploiement](deployment-history-deletions.md).

Pour obtenir de l’aide afin de résoudre des erreurs de déploiement spécifiques, consultez [Résoudre les erreurs courantes lors du déploiement de ressources sur Azure avec Azure Resource Manager](common-deployment-errors.md).

## <a name="get-deployments-and-correlation-id"></a>Récupérer des déploiements et l’ID de corrélation

Vous pouvez afficher les détails d’un déploiement via le Portail Azure, PowerShell, Azure CLI ou l’API REST. Chaque déploiement a un ID de corrélation, qui est utilisé pour suivre les événements connexes. Si vous [créez une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md), le support peut vous demander l’ID de corrélation. Le support utilise l’ID de corrélation pour identifier les opérations du déploiement qui a échoué.

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Sélectionnez le groupe de ressources que vous souhaitez examiner.

1. Sélectionnez le lien sous **Déploiements**.

   ![Sélectionner l’historique des déploiements](./media/deployment-history/select-deployment-history.png)

1. Sélectionnez un des déploiements dans l’historique des déploiements.

   ![Sélectionner un déploiement](./media/deployment-history/select-details.png)

1. Un résumé du déploiement s’affiche, y compris l’ID de corrélation.

    ![Résumé du déploiement](./media/deployment-history/show-correlation-id.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour répertorier tous les déploiements d’un groupe de ressources, utilisez la commande [Get-AzResourceGroupDeployment](/powershell/module/az.resources/Get-AzResourceGroupDeployment).

```azurepowershell-interactive
Get-AzResourceGroupDeployment -ResourceGroupName ExampleGroup
```

Pour obtenir un déploiement spécifique à partir d’un groupe de ressources, ajoutez le paramètre **DeploymentName**.

```azurepowershell-interactive
Get-AzResourceGroupDeployment -ResourceGroupName ExampleGroup -DeploymentName ExampleDeployment
```

Pour obtenir l’ID de corrélation, utilisez :

```azurepowershell-interactive
(Get-AzResourceGroupDeployment -ResourceGroupName ExampleGroup -DeploymentName ExampleDeployment).CorrelationId
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour lister le déploiement pour un groupe de ressources, utilisez [az deployment group list](/cli/azure/group/deployment#az-deployment-group-list).

```azurecli-interactive
az deployment group list --resource-group ExampleGroup
```

Pour obtenir un déploiement spécifique, utilisez [az deployment group show](/cli/azure/group/deployment#az-deployment-group-show).

```azurecli-interactive
az deployment group show --resource-group ExampleGroup --name ExampleDeployment
```

Pour obtenir l’ID de corrélation, utilisez :

```azurecli-interactive
az deployment group show --resource-group ExampleGroup --name ExampleDeployment --query properties.correlationId
```

# <a name="http"></a>[HTTP](#tab/http)

Pour répertorier les déploiements d’un groupe de ressources, utilisez l’opération suivante. Pour obtenir le numéro de version le plus récent de l’API à utiliser dans la requête, consultez [Déploiements – Liste par groupe de ressources](/rest/api/resources/deployments/listbyresourcegroup).

```
GET https://management.azure.com/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/Microsoft.Resources/deployments/?api-version={api-version}
```

Pour obtenir un déploiement spécifique, utilisez l’opération suivante. Pour obtenir le numéro de version le plus récent de l’API à utiliser dans la requête, consultez [Déploiements – Récupérer](/rest/api/resources/deployments/get).

```
GET https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/microsoft.resources/deployments/{deployment-name}?api-version={api-version}
```

La réponse inclut l’ID de corrélation.

```json
{
 ...
 "properties": {
   "mode": "Incremental",
   "provisioningState": "Failed",
   "timestamp": "2019-11-26T14:18:36.4518358Z",
   "duration": "PT26.2091817S",
   "correlationId": "47ff4228-bf2e-4ee5-a008-0b07da681230",
   ...
 }
}
```

---

## <a name="get-deployment-operations-and-error-message"></a>Récupérer des opérations de déploiement et un message d’erreur

Chaque déploiement peut comprendre plusieurs opérations. Pour plus d’informations sur un déploiement, consultez les opérations de déploiement. En cas d’échec d’un déploiement, les opérations de déploiement incluent un message d’erreur.

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Dans le résumé d’un déploiement, sélectionnez **Détails des opérations**.

    ![Sélectionner les détails des opérations](./media/deployment-history/get-operation-details.png)

1. Vous voyez les détails de cette étape du déploiement. Lorsqu’une erreur se produit, les détails incluent le message d’erreur.

    ![Afficher les détails de l’opération](./media/deployment-history/see-operation-details.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour afficher les opérations de déploiement relatives au déploiement vers un groupe de ressources, utilisez la commande [Get-AzResourceGroupDeploymentOperation](/powershell/module/az.resources/get-azdeploymentoperation).

```azurepowershell-interactive
Get-AzResourceGroupDeploymentOperation -ResourceGroupName ExampleGroup -DeploymentName ExampleDeploy
```

Pour afficher les opérations ayant échoué, filtrez les opérations à l’aide de l’état **Échec**.

```azurepowershell-interactive
(Get-AzResourceGroupDeploymentOperation -ResourceGroupName ExampleGroup -DeploymentName ExampleDeploy).Properties | Where-Object ProvisioningState -eq Failed
```

Pour obtenir le message d’état d’opérations ayant échoué, utilisez la commande suivante :

```azurepowershell-interactive
((Get-AzResourceGroupDeploymentOperation -ResourceGroupName ExampleGroup -DeploymentName ExampleDeploy ).Properties | Where-Object ProvisioningState -eq Failed).StatusMessage.error
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour afficher les opérations de déploiement relatives au déploiement vers un groupe de ressources, utilisez la commande [az deployment operation group list](/cli/azure/deployment/operation/group#az-deployment-operation-group-list). Vous devez disposer d’Azure CLI 2.6.0 ou version ultérieure.

```azurecli-interactive
az deployment operation group list --resource-group ExampleGroup --name ExampleDeployment
```

Pour afficher les opérations ayant échoué, filtrez les opérations à l’aide de l’état **Échec**.

```azurecli-interactive
az deployment operation group list --resource-group ExampleGroup --name ExampleDeploy --query "[?properties.provisioningState=='Failed']"
```

Pour obtenir le message d’état d’opérations ayant échoué, utilisez la commande suivante :

```azurecli-interactive
az deployment operation group list --resource-group ExampleGroup --name ExampleDeploy --query "[?properties.provisioningState=='Failed'].properties.statusMessage.error"
```

# <a name="http"></a>[HTTP](#tab/http)

Pour récupérer des opérations de déploiement, utilisez l’opération suivante. Pour obtenir le numéro de version le plus récent de l’API à utiliser dans la requête, consultez [Opérations de déploiement – Liste](/rest/api/resources/deploymentoperations/list).

```
GET https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/microsoft.resources/deployments/{deployment-name}/operations?$skiptoken={skiptoken}&api-version={api-version}
```

La réponse comprend un message d’erreur.

```json
{
  "value": [
    {
      "id": "/subscriptions/xxxx/resourceGroups/examplegroup/providers/Microsoft.Resources/deployments/exampledeploy/operations/13EFD9907103D640",
      "operationId": "13EFD9907103D640",
      "properties": {
        "provisioningOperation": "Create",
        "provisioningState": "Failed",
        "timestamp": "2019-11-26T14:18:36.3177613Z",
        "duration": "PT21.0580179S",
        "trackingId": "9d3cdac4-54f8-486c-94bd-10c20867b8bc",
        "serviceRequestId": "01a9d0fe-896b-4c94-a30f-60b70a8f1ad9",
        "statusCode": "BadRequest",
        "statusMessage": {
          "error": {
            "code": "InvalidAccountType",
            "message": "The AccountType Standard_LRS1 is invalid. For more information, see - https://aka.ms/storageaccountskus"
          }
        },
        "targetResource": {
          "id": "/subscriptions/xxxx/resourceGroups/examplegroup/providers/Microsoft.Storage/storageAccounts/storageq2czadzfgizc2",
          "resourceType": "Microsoft.Storage/storageAccounts",
          "resourceName": "storageq2czadzfgizc2"
        }
      }
    },
    ...
  ]
}
```

---

## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir de l’aide afin de résoudre des erreurs de déploiement spécifiques, consultez [Résoudre les erreurs courantes lors du déploiement de ressources sur Azure avec Azure Resource Manager](common-deployment-errors.md).
* Pour en savoir plus sur la gestion des déploiements dans l’historique, consultez [Suppressions automatiques de l’historique de déploiement](deployment-history-deletions.md).
* Pour valider votre déploiement avant son exécution, consultez [Déployer un groupe de ressources avec le modèle Azure Resource Manager](deploy-powershell.md).


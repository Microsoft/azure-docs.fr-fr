---
title: Restaurer en cas d’erreur vers un déploiement réussi
description: Spécifiez qu’un déploiement ayant échoué doit être restauré vers un déploiement réussi.
ms.topic: conceptual
ms.date: 10/04/2019
ms.openlocfilehash: 206c794996f58a4c5b6982c551ae50128ed4f5eb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "79460141"
---
# <a name="rollback-on-error-to-successful-deployment"></a>Restaurer en cas d’erreur vers un déploiement réussi

En cas d'échec du déploiement, vous pouvez automatiquement relancer un déploiement antérieur réussi à partir de votre historique de déploiement. Cette fonctionnalité est utile si le déploiement de votre infrastructure était dans un état correct connu et que vous souhaitez le restaurer à cet état. Il existe toutefois un certain nombre de restrictions et mises en garde :

- Le redéploiement est exécuté exactement comme le déploiement précédent, avec les mêmes paramètres. Vous ne pouvez pas changer les paramètres.
- Le déploiement précédent avait été exécuté en [mode complet](./deployment-modes.md#complete-mode). Les ressources qui n’étaient pas incluses dans le déploiement précédent sont supprimées, et les configurations de ressources sont définies à leur état précédent. Assurez-vous de bien comprendre les différents [modes de déploiement](./deployment-modes.md).
- Le redéploiement concerne uniquement les ressources ; il n’inclut pas les modifications de données.
- Vous pouvez utiliser cette fonctionnalité uniquement avec les déploiements de groupes de ressources, pas avec les déploiements de niveau abonnement ou groupe d’administration. Pour plus d’informations sur le déploiement au niveau de l’abonnement, consultez [Créer des groupes de ressources et des ressources au niveau de l’abonnement](./deploy-to-subscription.md).
- Vous pouvez uniquement utiliser cette option avec les déploiements de niveau racine. Les déploiements à partir d’un modèle imbriqué ne sont pas disponibles pour le redéploiement.

Pour utiliser cette option, vos déploiements doivent avoir des noms uniques afin de pouvoir être identifiés dans l’historique. Si les noms ne sont pas uniques, le déploiement actuellement en échec peut remplacer le déploiement réussi précédemment dans l’historique.

## <a name="powershell"></a>PowerShell

Pour redéployer le dernier déploiement réussi, ajoutez le paramètre `-RollbackToLastDeployment` comme indicateur.

```azurepowershell-interactive
New-AzResourceGroupDeployment -Name ExampleDeployment02 `
  -ResourceGroupName $resourceGroupName `
  -TemplateFile c:\MyTemplates\azuredeploy.json `
  -RollbackToLastDeployment
```

Pour redéployer un déploiement spécifique, utilisez le paramètre `-RollBackDeploymentName` et indiquez le nom du déploiement. Le déploiement spécifié doit avoir réussi.

```azurepowershell-interactive
New-AzResourceGroupDeployment -Name ExampleDeployment02 `
  -ResourceGroupName $resourceGroupName `
  -TemplateFile c:\MyTemplates\azuredeploy.json `
  -RollBackDeploymentName ExampleDeployment01
```

## <a name="azure-cli"></a>Azure CLI

Pour redéployer le dernier déploiement réussi, ajoutez le paramètre `--rollback-on-error` comme indicateur.

```azurecli-interactive
az deployment group create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters storageAccountType=Standard_GRS \
  --rollback-on-error
```

Pour redéployer un déploiement spécifique, utilisez le paramètre `--rollback-on-error` et indiquez le nom du déploiement. Le déploiement spécifié doit avoir réussi.

```azurecli-interactive
az deployment group create \
  --name ExampleDeployment02 \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters storageAccountType=Standard_GRS \
  --rollback-on-error ExampleDeployment01
```

## <a name="rest-api"></a>API REST

Pour redéployer le dernier déploiement réussi en cas d’échec du déploiement actuel, utilisez :

```json
{
  "properties": {
    "templateLink": {
      "uri": "http://mystorageaccount.blob.core.windows.net/templates/template.json",
      "contentVersion": "1.0.0.0"
    },
    "mode": "Incremental",
    "parametersLink": {
      "uri": "http://mystorageaccount.blob.core.windows.net/templates/parameters.json",
      "contentVersion": "1.0.0.0"
    },
    "onErrorDeployment": {
      "type": "LastSuccessful",
    }
  }
}
```

Pour redéployer un déploiement spécifique en cas d’échec du déploiement actuel, utilisez :

```json
{
  "properties": {
    "templateLink": {
      "uri": "http://mystorageaccount.blob.core.windows.net/templates/template.json",
      "contentVersion": "1.0.0.0"
    },
    "mode": "Incremental",
    "parametersLink": {
      "uri": "http://mystorageaccount.blob.core.windows.net/templates/parameters.json",
      "contentVersion": "1.0.0.0"
    },
    "onErrorDeployment": {
      "type": "SpecificDeployment",
      "deploymentName": "<deploymentname>"
    }
  }
}
```

Le déploiement spécifié doit avoir réussi.

## <a name="next-steps"></a>Étapes suivantes

- Pour déployer en toute sécurité votre service sur plusieurs régions, consultez [Azure Deployment Manager](deployment-manager-overview.md).
- Pour spécifier comment gérer les ressources présentes dans le groupe de ressources, mais non définies dans le modèle, consultez [Modes de déploiement Azure Resource Manager](deployment-modes.md).
- Pour comprendre comment définir des paramètres dans votre modèle, consultez [Comprendre la structure et la syntaxe des modèles Azure Resource Manager](template-syntax.md).
- Pour plus d’informations sur le déploiement d’un modèle qui nécessite un jeton SAP, consultez [Déploiement d’un modèle privé avec un jeton SAP](secure-template-with-sas-token.md).

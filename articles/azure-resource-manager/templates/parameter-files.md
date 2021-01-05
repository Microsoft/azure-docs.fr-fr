---
title: Créer un fichier de paramètres
description: Créer un fichier de paramètres pour transmettre des valeurs pendant le déploiement d’un modèle d’Azure Resource Manager
ms.topic: conceptual
ms.date: 09/01/2020
ms.openlocfilehash: 2b6d942b21594fa608127bb8f403e72295671005
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89276641"
---
# <a name="create-resource-manager-parameter-file"></a>Créer un fichier de paramètres Resource Manager

Au lieu de passer des paramètres en tant que valeurs inline dans votre script, il peut s’avérer plus facile d’utiliser un fichier JSON qui contient les valeurs des paramètres. Cet article vous indique comment créer ce fichier de paramètres.

## <a name="parameter-file"></a>Fichier de paramètres

Le format du fichier de paramètres est le suivant :

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "<first-parameter-name>": {
      "value": "<first-value>"
    },
    "<second-parameter-name>": {
      "value": "<second-value>"
    }
  }
}
```

Notez que les valeurs du paramètre sont stockées sous forme de texte brut dans le fichier de paramètres. Cette approche fonctionne pour les valeurs qui ne sont pas sensibles, telles que la spécification de la référence SKU pour une ressource. Elle ne fonctionne pas pour les valeurs sensibles, telles que les mots de passe. Si vous devez transmettre une valeur sensible en tant que paramètre, stockez la valeur dans un coffre de clés et référencez-le dans votre fichier de paramètres. La valeur sensible est récupérée en toute sécurité pendant le déploiement.

Le fichier de paramètres suivant comprend une valeur de texte brut et une valeur stockée dans un coffre de clés.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "<first-parameter-name>": {
      "value": "<first-value>"
    },
    "<second-parameter-name>": {
      "reference": {
        "keyVault": {
          "id": "<resource-id-key-vault>"
        },
        "secretName": "<secret-name>"
      }
    }
  }
}
```

Pour plus d’informations sur l’utilisation de valeurs d’un coffre de clés, consultez la page [Utiliser Azure Key Vault pour transmettre une valeur de paramètre sécurisée pendant le déploiement](key-vault-parameter.md).

## <a name="define-parameter-values"></a>Définissez des valeurs de paramètre

Pour savoir comment définir les valeurs des paramètres, ouvrez le modèle que vous déployez. Consultez la section Paramètres du modèle. L’exemple suivant montre les paramètres d’un modèle.

```json
"parameters": {
  "storagePrefix": {
    "type": "string",
    "maxLength": 11
  },
  "storageAccountType": {
    "type": "string",
    "defaultValue": "Standard_LRS",
    "allowedValues": [
    "Standard_LRS",
    "Standard_GRS",
    "Standard_ZRS",
    "Premium_LRS"
    ]
  }
}
```

Le premier détail à noter est le nom de chaque paramètre. Les valeurs de votre fichier de paramètres doivent correspondre aux noms.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storagePrefix": {
    },
    "storageAccountType": {
    }
  }
}
```

Notez le type de paramètre. Les valeurs de votre fichier de paramètres doivent être du même type. Pour ce modèle, vous pouvez fournir les deux paramètres sous forme de chaînes.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storagePrefix": {
      "value": ""
    },
    "storageAccountType": {
      "value": ""
    }
  }
}
```

Ensuite, recherchez une valeur par défaut. Si un paramètre a une valeur par défaut, vous pouvez fournir une valeur. Cela n’est cependant pas nécessaire.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storagePrefix": {
      "value": "" // This value must be provided.
    },
    "storageAccountType": {
      "value": "" // This value is optional. Template will use default value if not provided.
    }
  }
}
```

Enfin, examinez les valeurs autorisées et les restrictions telles que la longueur maximale. Elles vous indiquent la plage de valeurs que vous pouvez fournir pour le paramètre.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storagePrefix": {
      "value": "storage"
    },
    "storageAccountType": {
      "value": "Standard_ZRS"
    }
  }
}
```

Votre fichier de paramètres peut contenir uniquement des valeurs pour les paramètres définis dans le modèle. Si votre fichier de paramètres contient des paramètres supplémentaires qui ne correspondent pas à des paramètres définis dans le modèle, une erreur s’affiche.

## <a name="parameter-type-formats"></a>Formats de types de paramètres

L'exemple suivant indique les formats des différents types de paramètres.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "exampleString": {
      "value": "test string"
    },
    "exampleInt": {
      "value": 4
    },
    "exampleBool": {
      "value": true
    },
    "exampleArray": {
      "value": [
        "value 1",
        "value 2"
      ]
    },
    "exampleObject": {
      "value": {
        "property1": "value1",
        "property2": "value2"
      }
    }
   }
}
```

## <a name="deploy-template-with-parameter-file"></a>Déployer un modèle avec un fichier de paramètres

Pour passer un fichier de paramètres local avec Azure CLI, utilisez @ et le nom du fichier de paramètres.

```azurecli
az deployment group create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters @storage.parameters.json
```

Pour plus d’informations, consultez [Déployer des ressources à l’aide de modèles ARM et d’Azure CLI](./deploy-cli.md#parameters).

Pour passer un fichier de paramètres local avec Azure PowerShell, utilisez le paramètre `TemplateParameterFile`.

```azurepowershell
New-AzResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup `
  -TemplateFile c:\MyTemplates\azuredeploy.json `
  -TemplateParameterFile c:\MyTemplates\storage.parameters.json
```

Pour plus d’informations, consultez [Déployer des ressources à l’aide de modèles ARM et d’Azure PowerShell](./deploy-powershell.md#pass-parameter-values).

> [!NOTE]
> Il n’est pas possible d’utiliser un fichier de paramètres avec le panneau de modèle personnalisé dans le portail.

## <a name="file-name"></a>Nom de fichier

Généralement, vous devez ajouter **.parameters** au nom du modèle pour nommer le fichier de paramètres. Par exemple, si votre modèle est nommé **azuredeploy.json**, votre fichier de paramètres est **azuredeploy.parameters.json**. Cette convention d’affectation de noms vous permet de comprendre le lien entre le modèle et les paramètres.

Pour effectuer un déploiement dans différents environnements, créez plusieurs fichiers de paramètres. Lorsque vous nommez le fichier de paramètres, ajoutez un moyen permettant de définir son utilisation. Par exemple, utilisez **azuredeploy.parameters-dev.json** et **azuredeploy.parameters-prod.json**

## <a name="parameter-precedence"></a>Priorité des paramètres

Vous pouvez utiliser des paramètres inline et un fichier de paramètres local pendant la même opération de déploiement. Par exemple, vous pouvez spécifier certaines valeurs dans le fichier de paramètres local et ajouter d’autres valeurs inline pendant le déploiement. Si vous fournissez des valeurs pour un paramètre à la fois dans le fichier de paramètres local et inline, la valeur inline est prioritaire.

Il est possible d’utiliser un fichier de paramètres externe, en indiquant l’URI du fichier. Quand vous utilisez un fichier de paramètres externe, vous ne pouvez pas transmettre d’autres valeurs, qu’elles soient incluses ou extraites d’un fichier local. Tous les paramètres inline sont ignorés. Fournissez toutes les valeurs de paramètre dans le fichier externe.

## <a name="parameter-name-conflicts"></a>Conflits de noms de paramètre

Si votre modèle inclut un paramètre utilisant le même nom que l’un des paramètres dans la commande PowerShell, PowerShell présente le paramètre de votre modèle avec le suffixe **FromTemplate**. Par exemple, un paramètre nommé **ResourceGroupName** dans votre modèle est en conflit avec le paramètre **ResourceGroupName** dans la cmdlet [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment). Vous êtes invité à fournir une valeur pour **ResourceGroupNameFromTemplate**. Vous pouvez éviter cette confusion en utilisant des noms de paramètres qui ne sont pas utilisés pour les commandes de déploiement.


## <a name="next-steps"></a>Étapes suivantes

- Pour comprendre comment définir des paramètres dans votre modèle, consultez [Paramètres dans les modèles Azure Resource Manager](template-parameters.md).
- Pour plus d’informations sur l’utilisation de valeurs d’un coffre de clés, consultez la page [Utiliser Azure Key Vault pour transmettre une valeur de paramètre sécurisée pendant le déploiement](key-vault-parameter.md).
- Pour plus d’informations sur les paramètres, consultez [Paramètres dans les modèles Azure Resource Manager](template-parameters.md).

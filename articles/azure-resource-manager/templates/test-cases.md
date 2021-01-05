---
title: Cas de test pour la boîte à outils de test
description: Décrit les tests qui sont exécutés par la boîte à outils de test de modèle Resource Manager.
ms.topic: conceptual
ms.date: 12/03/2020
ms.author: tomfitz
author: tfitzmac
ms.openlocfilehash: ff9ad659e15a88725e4c3905ab6c623fda7610fd
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96600902"
---
# <a name="default-test-cases-for-arm-template-test-toolkit"></a>Cas de test par défaut de la boîte à outils de test de modèle Resource Manager

Cet article décrit les tests par défaut qui sont exécutés avec la [boîte à outils de test de modèle](test-toolkit.md). Il fournit des exemples qui réussissent le test ou y échouent. Il comprend le nom de chaque test.

## <a name="use-correct-schema"></a>Utiliser le schéma correct

Nom du test : **Le schéma DeploymentTemplate est correct**

Dans votre modèle, vous devez spécifier une valeur de schéma valide.

L’exemple suivant **réussit** ce test.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "resources": []
}
```

La propriété de schéma du modèle doit être définie sur l’un des schémas suivants :

* `https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#`
* `https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#`
* `https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#`
* `https://schema.management.azure.com/schemas/2019-08-01/tenantDeploymentTemplate.json#`
* `https://schema.management.azure.com/schemas/2019-08-01/managementGroupDeploymentTemplate.json`

## <a name="parameters-must-exist"></a>Parameters doit exister

Nom du test : **La propriété Parameters doit exister**

Votre modèle doit posséder un élément Parameters. Les paramètres sont essentiels pour que vos modèles soient réutilisables dans des environnements différents. Ajoutez des paramètres à votre modèle pour les valeurs qui changent lors du déploiement dans des environnements différents.

L’exemple suivant **réussit** ce test :

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "vmName": {
          "type": "string",
          "defaultValue": "linux-vm",
          "metadata": {
            "description": "Name for the Virtual Machine."
          }
      }
  },
  ...
```

## <a name="declared-parameters-must-be-used"></a>Les paramètres déclarés doivent être utilisés

Nom du test : **Les paramètres doivent être référencés**

Pour améliorer la lisibilité de votre modèle, supprimez tous les paramètres qui sont définis mais qui ne sont pas utilisés. Ce test recherche les paramètres qui ne sont utilisés nulle part dans le modèle. L’élimination des paramètres inutilisés facilite également le déploiement de votre modèle, car vous n’avez pas besoin de fournir des valeurs inutiles.

## <a name="secure-parameters-cant-have-hardcoded-default"></a>Les paramètres sécurisés ne peuvent pas avoir de valeur par défaut codée en dur

Nom du test : **Les paramètres de chaîne sécurisée ne peuvent pas avoir de valeur par défaut**

Ne fournissez pas de valeur par défaut codée en dur pour un paramètre sécurisé dans votre modèle. Une chaîne vide est correcte comme valeur par défaut.

Vous utilisez les types **SecureString** ou **SecureObject** sur les paramètres qui contiennent des valeurs sensibles, comme des mots de passe. Lorsqu’un paramètre utilise un type sécurisé, sa valeur n’est pas enregistrée ou stockée dans l’historique de déploiement. Cette action empêche un utilisateur malveillant de découvrir la valeur sensible.

Toutefois, lorsque vous fournissez une valeur par défaut, cette valeur est détectable par quiconque peut accéder au modèle ou à l’historique de déploiement.

L’exemple suivant **échoue** à ce test :

```json
"parameters": {
    "adminPassword": {
        "defaultValue": "HardcodedPassword",
        "type": "SecureString"
    }
}
```

Le prochain exemple **réussit** ce test :

```json
"parameters": {
    "adminPassword": {
        "type": "SecureString"
    }
}
```

## <a name="environment-urls-cant-be-hardcoded"></a>Les URL d’environnement ne peuvent pas être codées en dur

Nom du test : **DeploymentTemplate ne doit pas contenir d’URI codé en dur**

Ne codez pas en dur les URL d’environnement dans votre modèle. Au lieu de cela, utilisez la [fonction d’environnement](template-functions-deployment.md#environment) pour obtenir dynamiquement ces URL pendant le déploiement. Pour obtenir la liste des hôtes d’URL bloqués, consultez le [cas de test](https://github.com/Azure/arm-ttk/blob/master/arm-ttk/testcases/deploymentTemplate/DeploymentTemplate-Must-Not-Contain-Hardcoded-Uri.test.ps1).

L’exemple suivant **échoue** à ce test, car l’URL est codée en dur.

```json
"variables":{
    "AzureURL":"https://management.azure.com"
}
```

De même, le test **échoue** quand il est utilisé avec [concat](template-functions-string.md#concat) ou [uri](template-functions-string.md#uri).

```json
"variables":{
    "AzureSchemaURL1": "[concat('https://','gallery.azure.com')]",
    "AzureSchemaURL2": "[uri('gallery.azure.com','test')]"
}
```

L’exemple suivant **réussit** ce test.

```json
"variables": {
    "AzureSchemaURL": "[environment().gallery]"
},
```

## <a name="location-uses-parameter"></a>L’emplacement utilise le paramètre

Nom du test : **L’emplacement ne doit pas être codé en dur**

Vos modèles doivent comporter un paramètre d'emplacement nommé. Utilisez ce paramètre pour définir l'emplacement des ressources dans votre modèle. Dans le modèle principal (nommé azuredeploy.json ou mainTemplate.json), ce paramètre peut correspondre par défaut à l'emplacement du groupe de ressources. Dans les modèles liés ou imbriqués, le paramètre d'emplacement ne doit pas comporter d'emplacement par défaut.

Les utilisateurs de votre modèle peuvent disposer de régions limitées. Lorsque vous codez en dur l'emplacement des ressources, les utilisateurs peuvent ne pas être en mesure de créer une ressource dans cette région. Ils peuvent être bloqués même si vous définissez l'emplacement des ressources sur `"[resourceGroup().location]"`. Le groupe de ressources peut avoir été créé dans une région à laquelle les autres utilisateurs n'ont pas accès. L’utilisation du modèle est bloquée pour ces utilisateurs.

En fournissant un paramètre d'emplacement qui correspond par défaut à l'emplacement du groupe de ressources, les utilisateurs peuvent utiliser la valeur par défaut quand cela leur convient, mais aussi spécifier un autre emplacement.

L’exemple suivant **échoue** à ce test, car l’emplacement de la ressource est défini sur `resourceGroup().location`.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-06-01",
            "name": "storageaccount1",
            "location": "[resourceGroup().location]",
            "kind": "StorageV2",
            "sku": {
                "name": "Premium_LRS",
                "tier": "Premium"
            }
        }
    ]
}
```

Le prochain exemple utilise un paramètre d’emplacement mais **échoue** à ce test, car le paramètre d’emplacement est défini par défaut sur un emplacement codé en dur.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "defaultValue": "westus"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-06-01",
            "name": "storageaccount1",
            "location": "[parameters('location')]",
            "kind": "StorageV2",
            "sku": {
                "name": "Premium_LRS",
                "tier": "Premium"
            }
        }
    ],
    "outputs": {}
}
```

Au lieu de cela, créez un paramètre qui a pour valeur par défaut l’emplacement du groupe de ressources, mais qui permet aux utilisateurs de fournir une valeur différente. L'exemple suivant **réussit** ce test lorsque le modèle est utilisé comme modèle principal.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
                "description": "Location for the resources."
            }
        }
     },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-06-01",
            "name": "storageaccount1",
            "location": "[parameters('location')]",
            "kind": "StorageV2",
            "sku": {
                "name": "Premium_LRS",
                "tier": "Premium"
            }
        }
    ],
    "outputs": {}
}
```

En revanche, si l'exemple précédent est utilisé comme modèle lié, le test **échoue**. Lorsqu'il est utilisé comme modèle lié, supprimez la valeur par défaut.

## <a name="resources-should-have-location"></a>Les ressources doivent avoir un emplacement

Nom du test : **Les ressources doivent avoir un emplacement**

L’emplacement d’une ressource doit être défini sur une [expression de modèle](template-expressions.md) ou `global`. L’expression de modèle utilise généralement le paramètre d’emplacement décrit dans le test précédent.

L’exemple suivant **échoue** à ce test, car l’emplacement n’est pas une expression ou `global`.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "functions": [],
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-06-01",
            "name": "storageaccount1",
            "location": "westus",
            "kind": "StorageV2",
            "sku": {
                "name": "Premium_LRS",
                "tier": "Premium"
            }
        }
    ],
    "outputs": {}
}
```

L’exemple suivant **réussit** ce test.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "functions": [],
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Maps/accounts",
            "apiVersion": "2020-02-01-preview",
            "name": "demoMap",
            "location": "global",
            "sku": {
                "name": "S0"
            }
        }
    ],
    "outputs": {
    }
}
```

Le prochain exemple **réussit** ce test.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
                "description": "Location for the resources."
            }
        }
     },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-06-01",
            "name": "storageaccount1",
            "location": "[parameters('location')]",
            "kind": "StorageV2",
            "sku": {
                "name": "Premium_LRS",
                "tier": "Premium"
            }
        }
    ],
    "outputs": {}
}
```

## <a name="vm-size-uses-parameter"></a>La taille de machine virtuelle utilise un paramètre

Nom du test : **La taille de la machine virtuelle doit être un paramètre**

Évitez de coder en dur la taille de la machine virtuelle. Fournissez un paramètre afin que les utilisateurs de votre modèle puissent modifier la taille de la machine virtuelle déployée.

L’exemple suivant **échoue** à ce test.

```json
"hardwareProfile": {
    "vmSize": "Standard_D2_v3"
}
```

Au lieu de cela, fournissez un paramètre.

```json
"vmSize": {
    "type": "string",
    "defaultValue": "Standard_A2_v2",
    "metadata": {
        "description": "Size for the Virtual Machine."
    }
},
```

Ensuite, définissez la taille de la machine virtuelle sur ce paramètre.

```json
"hardwareProfile": {
    "vmSize": "[parameters('vmSize')]"
},
```

## <a name="min-and-max-values-are-numbers"></a>Les valeurs Min et Max sont des nombres

Nom du test : **Les valeurs Min et Max sont des nombres**

Si vous définissez des valeurs minimale et maximale pour un paramètre, spécifiez-les sous forme numérique.

L’exemple suivant **échoue** à ce test :

```json
"exampleParameter": {
    "type": "int",
    "minValue": "0",
    "maxValue": "10"
},
```

Au lieu de cela, fournissez les valeurs sous forme numérique. L’exemple suivant **réussit** ce test :

```json
"exampleParameter": {
    "type": "int",
    "minValue": 0,
    "maxValue": 10
},
```

Vous recevez également cet avertissement si vous fournissez une valeur minimale ou une valeur maximale, mais pas l’une et l’autre.

## <a name="artifacts-parameter-defined-correctly"></a>Paramètre des artefacts défini correctement

Nom du test : **paramètre artifacts**

Lorsque vous incluez des paramètres pour `_artifactsLocation` et `_artifactsLocationSasToken`, utilisez les valeurs par défaut et les types appropriés. Les conditions suivantes doivent être satisfaites pour réussir ce test :

* Si vous fournissez un paramètre, vous devez fournir l’autre
* `_artifactsLocation` doit être une **chaîne**
* `_artifactsLocation` doit posséder une valeur par défaut dans le modèle principal
* `_artifactsLocation` ne peut pas posséder une valeur par défaut dans un modèle imbriqué 
* Par défaut, `_artifactsLocation` doit être définie sur `"[deployment().properties.templateLink.uri]"` ou sur l’URL de référentiel brute
* `_artifactsLocationSasToken` doit être une **secureString**
* `_artifactsLocationSasToken` ne peut avoir qu’une chaîne vide comme valeur par défaut
* `_artifactsLocationSasToken` ne peut pas posséder une valeur par défaut dans un modèle imbriqué 

## <a name="declared-variables-must-be-used"></a>Les variables déclarées doivent être utilisées

Nom du test : **Les variables doivent être référencées**

Pour améliorer la lisibilité de votre modèle, supprimez toutes les variables qui sont définies mais qui ne sont pas utilisées. Ce test recherche les variables qui ne sont utilisées nulle part dans le modèle.

## <a name="dynamic-variable-should-not-use-concat"></a>La variable dynamique ne doit pas utiliser concat

Nom du test : **Les références de variables dynamiques ne doivent pas utiliser concat**

Parfois, vous devez construire dynamiquement une variable basée sur la valeur d’une autre variable ou d’un autre paramètre. N’utilisez pas la fonction [concat](template-functions-string.md#concat) lors de la définition de la valeur. Au lieu de cela, utilisez un objet qui comprend les options disponibles et récupérez dynamiquement l’une des propriétés de l’objet au cours du déploiement.

L’exemple suivant **réussit** ce test. La variable **currentImage** est définie de manière dynamique au cours du déploiement.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "osType": {
          "type": "string",
          "allowedValues": [
              "Windows",
              "Linux"
          ]
      }
  },
  "variables": {
    "imageOS": {
        "Windows": {
            "image": "Windows Image"
        },
        "Linux": {
            "image": "Linux Image"
        }
    },
    "currentImage": "[variables('imageOS')[parameters('osType')].image]"
  },
  "resources": [],
  "outputs": {
      "result": {
          "type": "string",
          "value": "[variables('currentImage')]"
      }
  }
}
```

## <a name="use-recent-api-version"></a>Utiliser la version récente de l’API

Nom du test : **apiVersions doit être récent**

La version de l’API de chaque ressource doit utiliser une version récente. Le test évalue la version que vous utilisez par rapport aux versions disponibles pour ce type de ressource.

## <a name="use-hardcoded-api-version"></a>Utiliser la version de l’API codée en dur

Nom du test : **Providers apiVersions n’est pas autorisé**

La version d’API d’un type de ressource détermine les propriétés disponibles. Fournissez une version d’API codée en dur dans votre modèle. Ne récupérez pas une version d’API qui est déterminée lors du déploiement. Vous ne saurez pas quelles propriétés sont disponibles.

L’exemple suivant **échoue** à ce test.

```json
"resources": [
    {
        "type": "Microsoft.Compute/virtualMachines",
        "apiVersion": "[providers('Microsoft.Compute', 'virtualMachines').apiVersions[0]]",
        ...
    }
]
```

L’exemple suivant **réussit** ce test.

```json
"resources": [
    {
       "type": "Microsoft.Compute/virtualMachines",
       "apiVersion": "2019-12-01",
       ...
    }
]
```

## <a name="properties-cant-be-empty"></a>Les propriétés ne peuvent pas être vides

Nom du test : **Le modèle ne doit pas contenir d’espaces vides**

Évitez de coder en dur les propriétés avec une valeur vide. Les valeurs vides incluent des chaînes, des objets ou des tableaux null et vides. Si vous avez défini une propriété sur une valeur vide, supprimez cette propriété de votre modèle. Toutefois, il est possible de définir une propriété sur une valeur vide pendant le déploiement, par exemple par le biais d’un paramètre.

## <a name="use-resource-id-functions"></a>Utiliser les fonctions ID de ressource

Nom du test : **Les ID doivent être dérivés de ResourceID**

Lorsque vous spécifiez un ID de ressource, utilisez l’une des fonctions d’ID de ressource. Les fonctions autorisées sont les suivantes :

* [resourceId](template-functions-resource.md#resourceid)
* [subscriptionResourceId](template-functions-resource.md#subscriptionresourceid)
* [tenantResourceId](template-functions-resource.md#tenantresourceid)
* [extensionResourceId](template-functions-resource.md#extensionresourceid)

N’utilisez pas la fonction concat pour créer un ID de ressource. L’exemple suivant **échoue** à ce test.

```json
"networkSecurityGroup": {
    "id": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/networkSecurityGroups/', variables('networkSecurityGroupName'))]"
}
```

Le prochain exemple **réussit** ce test.

```json
"networkSecurityGroup": {
    "id": "[resourceId('Microsoft.Network/networkSecurityGroups', variables('networkSecurityGroupName'))]"
}
```

## <a name="resourceid-function-has-correct-parameters"></a>La fonction ResourceId possède des paramètres corrects

Nom du test : **ResourceIds ne doit pas contenir**

Lors de la génération d’ID de ressource, n’utilisez pas de fonctions inutiles pour les paramètres facultatifs. Par défaut, la fonction [resourceId](template-functions-resource.md#resourceid) utilise le groupe actuel d’abonnements et de ressources. Vous n’avez pas besoin de fournir ces valeurs.  

L’exemple suivant **échoue** à ce test, car vous n’avez pas besoin de fournir l’ID d’abonnement et le nom de groupe de ressources actuels.

```json
"networkSecurityGroup": {
    "id": "[resourceId(subscription().subscriptionId, resourceGroup().name, 'Microsoft.Network/networkSecurityGroups', variables('networkSecurityGroupName'))]"
}
```

Le prochain exemple **réussit** ce test.

```json
"networkSecurityGroup": {
    "id": "[resourceId('Microsoft.Network/networkSecurityGroups', variables('networkSecurityGroupName'))]"
}
```

Ce test s’applique à :

* [resourceId](template-functions-resource.md#resourceid)
* [subscriptionResourceId](template-functions-resource.md#subscriptionresourceid)
* [tenantResourceId](template-functions-resource.md#tenantresourceid)
* [extensionResourceId](template-functions-resource.md#extensionresourceid)
* [reference](template-functions-resource.md#reference)
* [list*](template-functions-resource.md#list)

Concernant `reference` et `list*`, le test **échoue** quand vous utilisez `concat` pour construire l’ID de ressource.

## <a name="dependson-best-practices"></a>Bonnes pratiques pour dependsOn

Nom du test : **Bonnes pratiques pour dependsOn**

Lors de la définition des dépendances de déploiement, n’utilisez pas la fonction [if](template-functions-logical.md#if) pour tester une condition. Si une ressource dépend d’une ressource qui est [déployée de manière conditionnelle](conditional-resource-deployment.md), définissez la dépendance comme vous le feriez avec n’importe quelle ressource. Quand une ressource conditionnelle n’est pas déployée, Azure Resource Manager la supprime automatiquement des dépendances nécessaires.

L’exemple suivant **échoue** à ce test.

```json
"dependsOn": [
    "[if(equals(parameters('newOrExisting'),'new'), variables('storageAccountName'), '')]"
]
```

Le prochain exemple **réussit** ce test.

```json
"dependsOn": [
    "[variables('storageAccountName')]"
]
```

## <a name="nested-or-linked-deployments-cant-use-debug"></a>Les déploiements imbriqués ou liés ne peuvent pas utiliser debug

Nom du test : **Les ressources de déploiement ne doivent pas être en mode débogage**

Lorsque vous définissez un [modèle imbriqué ou lié](linked-templates.md) avec le type de ressource **Microsoft.Resources/deployments**, vous pouvez activer le débogage pour ce modèle. Le débogage est parfait lorsque vous avez besoin de tester ce modèle, mais vous devez le faire lorsque vous êtes prêt à utiliser le modèle en production.

## <a name="admin-user-names-cant-be-literal-value"></a>Les noms d’utilisateur admin ne peuvent pas être des valeurs littérales

Nom du test : **adminUsername ne doit pas être un littéral**

Lors de la définition d’un nom d’utilisateur administrateur, n’utilisez pas de valeur littérale.

L’exemple suivant **échoue** à ce test :

```json
"osProfile":  {
    "adminUserName":  "myAdmin"
},
```

Au lieu de cela, utilisez un paramètre. L’exemple suivant **réussit** ce test :

```json
"osProfile": {
    "adminUsername": "[parameters('adminUsername')]"
}
```

## <a name="use-latest-vm-image"></a>Utiliser la dernière image de machine virtuelle

Nom du test : **Les images de machine virtuelle doivent utiliser la dernière version**

Si votre modèle contient une machine virtuelle avec une image, assurez-vous qu’elle utilise la version la plus récente de l’image.

## <a name="use-stable-vm-images"></a>Utiliser des images de machine virtuelle stables

Nom du test : **Les machines virtuelles ne doivent pas être une préversion**

Les machines virtuelles ne doivent pas utiliser d’images de préversion.

L’exemple suivant **échoue** à ce test.

```json
"imageReference": {
    "publisher": "Canonical",
    "offer": "UbuntuServer",
    "sku": "16.04-LTS",
    "version": "latest-preview"
}
```

L’exemple suivant **réussit** ce test.

```json
"imageReference": {
    "publisher": "Canonical",
    "offer": "UbuntuServer",
    "sku": "16.04-LTS",
    "version": "latest"
},
```

## <a name="dont-use-managedidentity-extension"></a>Ne pas utiliser l’extension ManagedIdentity

Nom du test : **ManagedIdentityExtension ne doit pas être utilisée**

N’appliquez pas l’extension ManagedIdentity à une machine virtuelle. Pour plus d’informations, consultez [Comment cesser d’utiliser l’extension de machine virtuelle pour les identités managées et commencer à utiliser Azure Instance Metadata Service](../../active-directory/managed-identities-azure-resources/howto-migrate-vm-extension.md).

## <a name="outputs-cant-include-secrets"></a>Les sorties ne peuvent pas comprendre de secrets

Nom du test : **Les sorties ne doivent pas contenir de secrets**

N’incluez pas de valeurs dans la section outputs qui exposent potentiellement des secrets. La sortie d’un modèle est stockée dans l’historique de déploiement ; aussi un utilisateur malveillant peut-il y trouver ces informations.

L’exemple suivant **échoue** au test, car il comprend un paramètre sécurisé dans une valeur de sortie.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "secureParam": {
            "type": "securestring"
        }
    },
    "functions": [],
    "variables": {},
    "resources": [],
    "outputs": {
        "badResult": {
            "type": "string",
            "value": "[concat('this is the value ', parameters('secureParam'))]"
        }
    }
}
```

L’exemple suivant **échoue**, car il utilise une fonction [list*](template-functions-resource.md#list) dans les sorties.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storageName": {
            "type": "string"
        }
    },
    "functions": [],
    "variables": {},
    "resources": [],
    "outputs": {
        "badResult": {
            "type": "object",
            "value": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', parameters('storageName')), '2019-06-01')]"
        }
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’exécution de la boîte à outils de test, consultez [Utiliser le kit de ressources de test de modèle ARM](test-toolkit.md).
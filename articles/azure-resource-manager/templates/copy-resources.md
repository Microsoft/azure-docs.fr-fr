---
title: Déployer plusieurs instances de ressources
description: Utilisez l’opération copy et les tableaux dans un modèle Azure Resource Manager (modèle ARM) pour déployer un même type de ressource plusieurs fois.
ms.topic: conceptual
ms.date: 09/21/2020
ms.openlocfilehash: 47f3d693b84347973889a6003360d7113c427f4d
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905908"
---
# <a name="resource-iteration-in-arm-templates"></a>Itération de ressource dans les modèles ARM

Cet article explique comment créer plusieurs instances d’une ressource dans votre modèle Azure Resource Manager (modèle ARM). En ajoutant l’élément `copy` à la section resources de votre modèle, vous pouvez définir dynamiquement le nombre de ressources à déployer. Cela vous évite également de répéter la syntaxe du modèle.

Vous pouvez également utiliser `copy` avec [properties](copy-properties.md), [variables](copy-variables.md) et [outputs](copy-outputs.md).

Si vous devez spécifier si une ressource est déployée, consultez la page relative à l’[élément Condition](conditional-resource-deployment.md).

## <a name="syntax"></a>Syntaxe

L’élément `copy` utilise le format général suivant :

```json
"copy": {
  "name": "<name-of-loop>",
  "count": <number-of-iterations>,
  "mode": "serial" <or> "parallel",
  "batchSize": <number-to-deploy-serially>
}
```

La propriété `name` est toute valeur qui identifie la boucle. La propriété `count` spécifie le nombre d’itérations que vous souhaitez pour le type de ressource.

Utilisez les propriétés `mode` et `batchSize` pour spécifier si les ressources sont déployées en parallèle ou en séquence. Ces propriétés sont décrites dans [Série ou parallèle](#serial-or-parallel).

## <a name="copy-limits"></a>Limites de copie

Le nombre ne peut pas dépasser 800.

Le nombre ne peut pas être négatif. Il peut être zéro si vous déployez le modèle avec une version récente d’Azure CLI, de PowerShell ou de l’API REST. Plus précisément, vous devez utiliser :

* Azure PowerShell **2.6** ou version ultérieure
* Azure CLI **2.0.74** ou version ultérieure
* API REST version **2019-05-10** ou ultérieure
* Les [déploiements liés](linked-templates.md) doivent utiliser la version **2019-05-10** de l’API ou une version ultérieure pour le type de ressource de déploiement

Les versions antérieures de PowerShell, de l’interface CLI et de l’API REST ne prennent pas en charge le nombre zéro.

Soyez prudent lorsque vous utilisez le [déploiement en mode Complet](deployment-modes.md) avec des copies. Si vous redéployez dans un groupe de ressources en mode Complet, toutes les ressources qui ne sont pas spécifiées dans le modèle après la résolution de la boucle de copie sont supprimées.

## <a name="resource-iteration"></a>Itération de ressource

L’exemple suivant crée le nombre de comptes de stockage spécifiés dans le paramètre `storageCount`.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storageCount": {
            "type": "int",
            "defaultValue": 2
        }
    },
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-04-01",
            "name": "[concat(copyIndex(),'storage', uniqueString(resourceGroup().id))]",
            "location": "[resourceGroup().location]",
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {},
            "copy": {
                "name": "storagecopy",
                "count": "[parameters('storageCount')]"
            }
        }
    ],
    "outputs": {}
}
```

Notez que le nom de chaque ressource inclut la fonction `copyIndex()`, qui renvoie l’itération actuelle de la boucle. `copyIndex()` est basé sur zéro. Si bien que l’exemple suivant :

```json
"name": "[concat('storage', copyIndex())]",
```

Crée les noms suivants :

* storage0
* storage1
* storage2.

Pour décaler la valeur d’index, vous pouvez passer une valeur dans la fonction `copyIndex()`. Le nombre d’itérations est toujours spécifié dans l’élément copy, mais la valeur de `copyIndex` est décalée en fonction de la valeur spécifiée. Si bien que l’exemple suivant :

```json
"name": "[concat('storage', copyIndex(1))]",
```

Crée les noms suivants :

* storage1
* storage2
* storage3

L’opération copy se révèle utile lorsque vous travaillez avec des tableaux, car vous pouvez itérer sur chaque élément du tableau. Utilisez la fonction `length` sur le tableau pour spécifier le nombre d’itérations, et `copyIndex` pour récupérer l’index actuel dans le tableau.

L’exemple suivant crée un compte de stockage par nom fourni dans le paramètre.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "storageNames": {
          "type": "array",
          "defaultValue": [
            "contoso",
            "fabrikam",
            "coho"
          ]
      }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[concat(parameters('storageNames')[copyIndex()], uniqueString(resourceGroup().id))]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "properties": {},
      "copy": {
        "name": "storagecopy",
        "count": "[length(parameters('storageNames'))]"
      }
    }
  ],
  "outputs": {}
}
```

Si vous souhaitez retourner des valeurs à partir des ressources déployées, vous pouvez utiliser l’[élément copy dans la section outputs](copy-outputs.md).

## <a name="serial-or-parallel"></a>Série ou parallèle

Par défaut, Resource Manager crée les ressources en parallèle. Il n’applique aucune limite au nombre de ressources déployées en parallèle, à l’exception de la limite totale de 800 ressources dans le modèle. L’ordre de création n’est pas garanti.

Toutefois, vous souhaiterez peut-être spécifier que les ressources soient déployées en séquence. Par exemple, lors de la mise à jour d’un environnement de production, vous souhaiterez échelonner les mises à jour afin que seulement un certain nombre soient mises à jour à un moment donné. Pour déployer en série plusieurs instances d’une ressource, affectez à `mode` la valeur **serial** et à `batchSize` le nombre d’instances à déployer à la fois. Avec le mode série, Resource Manager crée une dépendance sur les instances précédentes de la boucle, afin de ne pas démarrer un lot tant que le précédent n’est pas terminé.

La valeur de `batchSize` ne peut pas dépasser la valeur de `count` dans l’élément copy.

Par exemple, pour déployer en série des comptes de stockage deux à la fois, utilisez :

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[concat(copyIndex(),'storage', uniqueString(resourceGroup().id))]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "copy": {
        "name": "storagecopy",
        "count": 4,
        "mode": "serial",
        "batchSize": 2
      },
      "properties": {}
    }
  ],
  "outputs": {}
}
```

La propriété `mode` accepte également **parallel**, qui est la valeur par défaut.

## <a name="depend-on-resources-in-a-loop"></a>En fonction des ressources dans une boucle

Vous spécifiez qu’une ressource est déployée après une autre ressource à l’aide de l’élément `dependsOn`. Pour déployer une ressource qui dépend de la collection de ressources dans une boucle, vous pouvez utiliser le nom de la boucle de copie dans l’élément dependsOn. L’exemple suivant montre comment déployer trois comptes de stockage avant de déployer la machine virtuelle. La définition complète de la machine virtuelle n’est pas affichée. Notez que le nom de l’élément copy a la valeur `storagecopy` et que l’élément dependsOn pour la machine virtuelle est également défini sur `storagecopy`.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[concat(copyIndex(),'storage', uniqueString(resourceGroup().id))]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "copy": {
        "name": "storagecopy",
        "count": 3
      },
      "properties": {}
    },
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2015-06-15",
      "name": "[concat('VM', uniqueString(resourceGroup().id))]",
      "dependsOn": ["storagecopy"],
      ...
    }
  ],
  "outputs": {}
}
```

## <a name="iteration-for-a-child-resource"></a>Itération d’une ressource enfant

Vous ne pouvez pas utiliser une boucle de copie pour une ressource enfant. Pour créer plusieurs instances d’une ressource que l’on définit en général comme imbriquée dans une autre ressource, vous devez au contraire la créer sous la forme d’une ressource de premier niveau. Vous définissez la relation avec la ressource parente par le biais des propriétés type et name.

Par exemple, supposons que vous définissez généralement un jeu de données comme une ressource enfant dans une fabrique de données.

```json
"resources": [
{
  "type": "Microsoft.DataFactory/datafactories",
  "name": "exampleDataFactory",
  ...
  "resources": [
    {
      "type": "datasets",
      "name": "exampleDataSet",
      "dependsOn": [
        "exampleDataFactory"
      ],
      ...
    }
  ]
```

Pour créer plusieurs jeux de données, déplacez-le en dehors de la fabrique de données. Le jeu de données doit être au même niveau que la fabrique de données, mais il est toujours une ressource enfant de la fabrique de données. Vous conservez la relation entre le jeu de données et la fabrique de données par le biais des propriétés type et name. Étant donné que le type ne peut plus peut être déduit à partir de sa position dans le modèle, vous devez fournir le type qualifié complet au format : `{resource-provider-namespace}/{parent-resource-type}/{child-resource-type}`.

Pour établir une relation parent/enfant avec une instance de la fabrique de données, fournissez un nom pour le jeu de données incluant le nom de la ressource parente. Utilisez le format : `{parent-resource-name}/{child-resource-name}`.

L’exemple ci-après illustre l’implémentation :

```json
"resources": [
{
  "type": "Microsoft.DataFactory/datafactories",
  "name": "exampleDataFactory",
  ...
},
{
  "type": "Microsoft.DataFactory/datafactories/datasets",
  "name": "[concat('exampleDataFactory', '/', 'exampleDataSet', copyIndex())]",
  "dependsOn": [
    "exampleDataFactory"
  ],
  "copy": {
    "name": "datasetcopy",
    "count": "3"
  },
  ...
}]
```

## <a name="example-templates"></a>Exemples de modèles

Les exemples suivants montrent des scénarios courants de création de plusieurs instances d’une ressource ou d’une propriété.

|Modèle  |Description  |
|---------|---------|
|[Copie de stockage](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/copystorage.json) |Déploie plusieurs comptes de stockage dont le nom comporte un numéro d’index . |
|[Copie de stockage en série](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/serialcopystorage.json) |Déploie plusieurs comptes de stockage un par un. Le nom inclut le numéro d’index. |
|[Copie de stockage avec tableau](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/copystoragewitharray.json) |Déploie plusieurs comptes de stockage. Le nom contient une valeur tirée d’un tableau. |
|[Déploiement de machine virtuelle avec un nombre variable de disques de données](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-windows-copy-datadisks) |Déploie plusieurs disques de données avec une machine virtuelle. |
|[Règles de sécurité multiples](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/multiplesecurityrules.json) |Déploie plusieurs règles de sécurité sur un groupe de sécurité réseau. Crée les règles de sécurité à partir d’un paramètre. Pour le paramètre, consultez [plusieurs fichiers de paramètre NSG](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/multiplesecurityrules.parameters.json). |

## <a name="next-steps"></a>Étapes suivantes

* Pour suivre un tutoriel, consultez [Tutoriel : Créer plusieurs instances de ressources grâce à des modèles ARM](template-tutorial-create-multiple-instances.md).
* Pour connaître les autres utilisations de l’élément copy, consultez :
  * [Itération de propriété dans les modèles ARM](copy-properties.md)
  * [Itération de variable dans les modèles ARM](copy-variables.md)
  * [Itération de sortie dans les modèles ARM](copy-outputs.md)
* Pour plus d’informations sur l’utilisation de l’élément copy avec les modèles imbriqués, consultez [Utilisation de l’élément copy](linked-templates.md#using-copy).
* Pour plus d’informations sur les différentes sections d’un modèle, consultez [Présentation de la structure et de la syntaxe des modèles ARM](template-syntax.md).
* Pour savoir comment déployer votre modèle, consultez [Déployer des ressources avec des modèles ARM et Azure PowerShell](deploy-powershell.md).

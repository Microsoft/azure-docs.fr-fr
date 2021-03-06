---
title: Ressources enfants dans les modèles
description: Décrit comment définir le nom et le type des ressources enfants dans un modèle Azure Resource Manager (modèle ARM).
ms.topic: conceptual
ms.date: 05/07/2021
ms.openlocfilehash: 48b44cad1fef7f09fe07fbb1c0e416ce947e586a
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/10/2021
ms.locfileid: "111969552"
---
# <a name="set-name-and-type-for-child-resources"></a>Définir le nom et le type des ressources enfants

Les ressources enfants sont des ressources qui existent uniquement dans le contexte d’une autre ressource. Par exemple, une [extension de machine virtuelle](/azure/templates/microsoft.compute/virtualmachines/extensions) ne peut pas exister sans [machine virtuelle](/azure/templates/microsoft.compute/virtualmachines). La ressource d’extension est un enfant de la machine virtuelle.

Chaque ressource parente accepte uniquement certains types de ressources comme ressources enfants. Le type de ressource de la ressource enfant comprend le type de ressource de la ressource parent. Par exemple, `Microsoft.Web/sites/config` et `Microsoft.Web/sites/extensions` sont des ressources enfant de la ressource `Microsoft.Web/sites`. Les types de ressource acceptés sont spécifiés dans le [schéma de modèle](https://github.com/Azure/azure-resource-manager-schemas) de la ressource parente.

Dans un modèle Azure Resource Manager (modèle ARM), vous pouvez spécifier la ressource enfant dans la ressource parent ou en dehors de la ressource parent. Les valeurs que vous fournissez pour le nom et le type de la ressource varient selon que la ressource enfant est définie dans la ressource parent ou en dehors de celle-ci.

## <a name="within-parent-resource"></a>Dans la ressource parent

L’exemple suivant illustre la ressource enfant incluse dans la propriété Ressources de la ressource parent.

```json
"resources": [
  {
    <parent-resource>
    "resources": [
      <child-resource>
    ]
  }
]
```

Les ressources enfants peuvent uniquement être définies sur cinq niveaux.

Lorsqu’elles sont définies dans le type de ressource parent, les valeurs de type et de nom sont formatées en un seul segment sans barres obliques.

```json
"type": "{child-resource-type}",
"name": "{child-resource-name}",
```

L’exemple suivant montre un réseau virtuel et un sous-réseau. Notez que le sous-réseau est inclus dans le tableau des ressources pour le réseau virtuel. Le nom est défini sur **Subnet1** et le type est défini sur **sous-réseaux**. La ressource enfant est marquée comme étant dépendante de la ressource parent, car la ressource parent doit exister pour que la ressource enfant puisse être déployée.

```json
"resources": [
  {
    "type": "Microsoft.Network/virtualNetworks",
    "apiVersion": "2018-10-01",
    "name": "VNet1",
    "location": "[parameters('location')]",
    "properties": {
      "addressSpace": {
        "addressPrefixes": [
          "10.0.0.0/16"
        ]
      }
    },
    "resources": [
      {
        "type": "subnets",
        "apiVersion": "2018-10-01",
        "name": "Subnet1",
        "dependsOn": [
          "VNet1"
        ],
        "properties": {
          "addressPrefix": "10.0.0.0/24"
        }
      }
    ]
  }
]
```

Le type de ressource complet est toujours `Microsoft.Network/virtualNetworks/subnets`. Vous ne fournissez pas `Microsoft.Network/virtualNetworks/`, car il est déduit du type de ressource parent.

Le nom de la ressource enfant est défini sur **Subnet 1**, mais le nom complet inclut le nom parent. Vous ne fournissez pas **VNet1**, car il est déduit de la ressource parent.

## <a name="outside-parent-resource"></a>En dehors de la ressource parent

L’exemple suivant montre la ressource enfant en dehors de la ressource parent. Vous pouvez utiliser cette approche si la ressource parent n’est pas déployée dans le même modèle ou si voulez utiliser une[copie](copy-resources.md) pour créer plusieurs ressources enfants.

```json
"resources": [
  {
    <parent-resource>
  },
  {
    <child-resource>
  }
]
```

En cas de définition en dehors de la ressource parent, vous mettez en forme le type et ajouter des barres obliques pour inclure le type et le nom du parent.

```json
"type": "{resource-provider-namespace}/{parent-resource-type}/{child-resource-type}",
"name": "{parent-resource-name}/{child-resource-name}",
```

L’exemple suivant montre un réseau virtuel et un sous-réseau qui sont tous deux définis au niveau de la racine. Notez que le sous-réseau est inclus dans le tableau des ressources pour le réseau virtuel. Le nom est défini sur **VNet1/Subnet1** et le type est défini sur `Microsoft.Network/virtualNetworks/subnets`. La ressource enfant est marquée comme étant dépendante de la ressource parent, car la ressource parent doit exister pour que la ressource enfant puisse être déployée.

```json
"resources": [
  {
    "type": "Microsoft.Network/virtualNetworks",
    "apiVersion": "2018-10-01",
    "name": "VNet1",
    "location": "[parameters('location')]",
    "properties": {
      "addressSpace": {
        "addressPrefixes": [
          "10.0.0.0/16"
        ]
      }
    }
  },
  {
    "type": "Microsoft.Network/virtualNetworks/subnets",
    "apiVersion": "2018-10-01",
    "name": "VNet1/Subnet1",
    "dependsOn": [
      "VNet1"
    ],
    "properties": {
      "addressPrefix": "10.0.0.0/24"
    }
  }
]
```

## <a name="next-steps"></a>Étapes suivantes

* Pour apprendre à créer des modèles ARM, consultez [Comprendre la structure et la syntaxe des modèles ARM](./syntax.md).
* Pour en savoir plus sur le format du nom de la ressource lors du référencement de la ressource, consultez la [fonction de référence](template-functions-resource.md#reference).
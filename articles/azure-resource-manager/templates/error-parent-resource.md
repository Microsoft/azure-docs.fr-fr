---
title: Erreurs de ressources parentes
description: Décrit comment résoudre les erreurs liées à des ressources parentes dans un modèle Azure Resource Manager.
ms.topic: troubleshooting
ms.date: 08/01/2018
ms.openlocfilehash: 474cb85d16382136e24e5502b87ba8a1a65488ef
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84673658"
---
# <a name="resolve-errors-for-parent-resources"></a>Résoudre les erreurs avec des ressources parentes

Cet article décrit les erreurs que vous pouvez rencontrer au cours du déploiement d’une ressource dépendant d’une ressource parente.

## <a name="symptom"></a>Symptôme

Au cours du déploiement d’une ressource qui est l’enfant d’une autre ressource, vous pouvez rencontrer l’erreur suivante :

```
Code=ParentResourceNotFound;
Message=Can not perform requested operation on nested resource. Parent resource 'exampleserver' not found."
```

## <a name="cause"></a>Cause :

Lorsqu’une ressource est l’enfant d’une autre ressource, la ressource parente doit exister avant de pouvoir créer la ressource enfant. Le nom de la ressource enfant définit la connexion à la ressource parente. Le nom de la ressource enfant est au format suivant : `<parent-resource-name>/<child-resource-name>`. Par exemple, une base de données SQL peut être définie de la manière suivante :

```json
{
  "type": "Microsoft.Sql/servers/databases",
  "name": "[concat(variables('databaseServerName'), '/', parameters('databaseName'))]",
  ...
```

Si vous déployez le serveur et la base de données dans le même modèle, mais que vous ne spécifiez pas de dépendance sur le serveur, le déploiement de la base de données peut commencer avant que le serveur n’ait été déployé.

Si la ressource parente existe déjà et n’est pas déployée dans le même modèle, vous obtenez cette erreur quand le Gestionnaire des ressources ne peut pas associer la ressource enfant au parent. Cette erreur peut se produire quand la ressource enfant n’est pas dans le format correct, ou que la ressource enfant est déployée dans un groupe de ressources différent du groupe de ressources de la ressource parente.

## <a name="solution"></a>Solution

Pour résoudre cette erreur quand les ressources parente et enfant sont déployées dans le même modèle, incluez une dépendance.

```json
"dependsOn": [
  "[variables('databaseServerName')]"
]
```

Pour résoudre cette erreur quand la ressource parente a déjà été déployée dans un autre modèle, vous ne définissez pas de dépendance. Au lieu de cela, déployez l’enfant dans le même groupe de ressources et fournissez le nom de la ressource parente.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "sqlServerName": {
      "type": "string"
    },
    "databaseName": {
      "type": "string"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Sql/servers/databases",
      "apiVersion": "2014-04-01",
      "name": "[concat(parameters('sqlServerName'), '/', parameters('databaseName'))]",
      "location": "[resourceGroup().location]",
      "properties": {
        "collation": "SQL_Latin1_General_CP1_CI_AS",
        "edition": "Basic"
      }
    }
  ],
  "outputs": {}
}
```

Pour plus d’informations, consultez [Définir l’ordre de déploiement des ressources dans les modèles Azure Resource Manager](define-resource-dependency.md).
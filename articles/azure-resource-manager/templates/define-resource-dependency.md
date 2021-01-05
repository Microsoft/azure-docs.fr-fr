---
title: Définir l’ordre de déploiement des ressources
description: Décrit la procédure permettant de définir une ressource comme dépendante d’une autre ressource au cours du déploiement afin de garantir le déploiement des ressources dans l'ordre adéquat.
ms.topic: conceptual
ms.date: 12/03/2019
ms.openlocfilehash: 84cea915565ec6ac9872681e1d4173abacb46ac4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85255209"
---
# <a name="define-the-order-for-deploying-resources-in-arm-templates"></a>Définir l’ordre de déploiement des ressources dans les modèles ARM

Quand vous déployez une ressource, vous devez éventuellement vous assurer au préalable que d’autres ressources existent. Par exemple, vous avez besoin d’un serveur SQL logique avant de déployer une base de données. Vous définissez cette relation en marquant une seule ressource comme dépendante de l'autre ressource. Pour définir une dépendance, vous devez utiliser l’élément **dependsOn** ou la fonction **reference**.

Resource Manager évalue les dépendances entre les ressources et les déploie dans leur ordre dépendant. Quand les ressources ne dépendent pas les unes des autres, Resource Manager les déploie en parallèle. Vous devez uniquement définir des dépendances pour les ressources qui sont déployées dans le même modèle.

## <a name="dependson"></a>dependsOn

Dans votre modèle, l’élément dependsOn vous permet de définir une ressource comme une dépendance sur une ou plusieurs ressources. Sa valeur est un tableau JSON de chaînes, chacune d’elles représentant un nom de ressource. Le tableau peut inclure des ressources [déployées de manière conditionnelle](conditional-resource-deployment.md). Quand une ressource conditionnelle n’est pas déployée, Azure Resource Manager la supprime automatiquement des dépendances nécessaires.

L’exemple suivant montre un groupe identique de machines virtuelles dépendant d’un équilibreur de charge, un réseau virtuel et une boucle qui crée plusieurs comptes de stockage. Ces autres ressources ne figurent pas dans l’exemple suivant, mais ont besoin d’exister ailleurs dans le modèle.

```json
{
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  "apiVersion": "2016-03-30",
  "name": "[variables('namingInfix')]",
  "location": "[variables('location')]",
  "tags": {
    "displayName": "VMScaleSet"
  },
  "dependsOn": [
    "[variables('loadBalancerName')]",
    "[variables('virtualNetworkName')]",
    "storageLoop",
  ],
  ...
}
```

Dans l’exemple précédent, une dépendance est incluse sur les ressources créées par le biais d’une boucle de copie nommée **storageLoop**. Pour obtenir un exemple, consultez [Création de plusieurs instances de ressources dans Azure Resource Manager](copy-resources.md).

Lors de la définition des dépendances, vous pouvez inclure l’espace de noms du fournisseur de ressources et le type de ressource pour éviter toute ambiguïté. Par exemple, pour distinguer un équilibrage de charge et un réseau virtuel qui peuvent avoir les mêmes noms que d’autres ressources, utilisez le format suivant :

```json
"dependsOn": [
  "[resourceId('Microsoft.Network/loadBalancers', variables('loadBalancerName'))]",
  "[resourceId('Microsoft.Network/virtualNetworks', variables('virtualNetworkName'))]"
]
```

Vous pouvez être tenté d’utiliser dependsOn pour mapper les relations entre vos ressources. Il est toutefois important de comprendre pourquoi vous le faites. Par exemple, pour documenter la manière dont les ressources sont liées entre elles, dependsOn n’est pas la bonne approche. Vous ne pouvez pas lancer de requête pour savoir quelles ressources ont été définies dans l’élément dependsOn après le déploiement. En utilisant dependsOn, vous risquez d’avoir un impact sur le temps de déploiement, car Resource Manager ne déploie pas en parallèle deux ressources qui ont une dépendance.

## <a name="child-resources"></a>Ressources enfants

La propriété de ressources vous permet de vous permet de spécifier les ressources enfants associées à la ressource en cours de définition. Les ressources enfants peuvent uniquement être définies sur cinq niveaux. Il est important de noter qu’aucune dépendance de déploiement implicite n’est créée entre une ressource enfant et la ressource parent. Si vous avez besoin de déployer la ressource enfant après la ressource parent, vous devez déclarer explicitement cette dépendance avec la propriété dependsOn.

Chaque ressource parente accepte uniquement certains types de ressources comme ressources enfants. Les types de ressource acceptés sont spécifiés dans le [schéma de modèle](https://github.com/Azure/azure-resource-manager-schemas) de la ressource parente. Le nom du type de ressource enfant inclut le nom du type de ressource parente. Par exemple, **Microsoft.Web/sites/config** et **Microsoft.Web/sites/extensions** sont deux ressources enfants de **Microsoft.Web/sites**.

L’exemple suivant montre un serveur SQL et une base de données. Notez qu’une dépendance explicite est définie entre la base de données et le serveur, même si la base de données est un enfant du serveur.

```json
"resources": [
  {
    "name": "[variables('sqlserverName')]",
    "apiVersion": "2014-04-01-preview",
    "type": "Microsoft.Sql/servers",
    "location": "[resourceGroup().location]",
    "tags": {
      "displayName": "SqlServer"
    },
    "properties": {
      "administratorLogin": "[parameters('administratorLogin')]",
      "administratorLoginPassword": "[parameters('administratorLoginPassword')]"
    },
    "resources": [
      {
        "name": "[parameters('databaseName')]",
        "apiVersion": "2014-04-01-preview",
        "type": "databases",
        "location": "[resourceGroup().location]",
        "dependsOn": [
          "[variables('sqlserverName')]"
        ],
        "tags": {
          "displayName": "Database"
        },
        "properties": {
          "edition": "[parameters('edition')]",
          "collation": "[parameters('collation')]",
          "maxSizeBytes": "[parameters('maxSizeBytes')]",
          "requestedServiceObjectiveName": "[parameters('requestedServiceObjectiveName')]"
        }
      }
    ]
  }
]
```

## <a name="reference-and-list-functions"></a>fonctions reference et list

La [fonction de référence](template-functions-resource.md#reference) permet à une expression de tirer sa valeur d’un autre nom JSON et de paires de valeurs ou de ressources runtime. Les [fonctions list*](template-functions-resource.md#list) renvoient les valeurs d’une ressource à partir d’une opération de liste.  Les expressions reference et list déclarent implicitement qu’une ressource dépend d’une autre, lorsque la ressource référencée est déployée dans le même modèle et désignée par son nom (pas par son ID). Si vous transmettez l’ID de ressource dans les fonctions reference ou list, une référence implicite n’est pas créée.

Le format général de la fonction reference est :

```json
reference('resourceName').propertyPath
```

Le format général de la fonction listKeys est :

```json
listKeys('resourceName', 'yyyy-mm-dd')
```

Dans l’exemple suivant, un point de terminaison CDN dépend explicitement du profil CDN et implicitement d’une application web.

```json
{
    "name": "[variables('endpointName')]",
    "apiVersion": "2016-04-02",
    "type": "endpoints",
    "location": "[resourceGroup().location]",
    "dependsOn": [
      "[variables('profileName')]"
    ],
    "properties": {
      "originHostHeader": "[reference(variables('webAppName')).hostNames[0]]",
      ...
    }
```

Vous pouvez utiliser cet élément ou l’élément dependsOn pour spécifier les dépendances, mais il est inutile d’utiliser les deux pour la même ressource dépendante. Si possible, utilisez une référence implicite pour éviter d’ajouter une dépendance inutile.

Pour plus d’informations, consultez la [fonction de référence](template-functions-resource.md#reference).

## <a name="circular-dependencies"></a>Dépendances circulaires

Resource Manager identifie les dépendances circulaires lors de la validation du modèle. Si vous recevez une erreur indiquant qu’il existe une dépendance circulaire, évaluez votre modèle pour déterminer si certaines dépendances sont inutiles et peuvent être supprimées. Si la suppression de ces dépendances n’a aucun effet, vous pouvez éliminer les dépendances circulaires en déplaçant certaines opérations de déploiement dans les ressources enfants qui sont déployées après les ressources présentant la dépendance circulaire. Par exemple, supposons que vous déployiez deux machines virtuelles, mais que vous deviez définir sur chacune d’elles des propriétés faisant référence les unes aux autres. Vous pouvez les déployer dans l’ordre suivant :

1. Machine virtuelle 1
2. Machine virtuelle 2
3. L’extension sur la machine virtuelle 1 dépend des machines virtuelles 1 et 2. L’extension définit sur la machine virtuelle 1 des valeurs qu’elle obtient de la machine virtuelle 2.
4. L’extension sur la machine virtuelle 2 dépend des machines virtuelles 1 et 2. L’extension définit sur la machine virtuelle 2 des valeurs qu’elle obtient de la machine virtuelle 1.

Pour plus d’informations sur l’évaluation de l’ordre de déploiement et la résolution des erreurs de dépendance, consultez l’article [Résolution des erreurs courantes dans des déploiements Azure avec Azure Resource Manager](common-deployment-errors.md).

## <a name="next-steps"></a>Étapes suivantes

* Pour un didacticiel, consultez [Didacticiel : créer des modèles Azure Resource Manager avec des ressources dépendantes](template-tutorial-create-templates-with-dependent-resources.md).
* Pour obtenir des recommandations lors de la définition des dépendances, consultez [Bonnes pratiques relatives aux modèles Azure Resource Manager](template-best-practices.md).
* Pour en savoir plus sur la résolution des problèmes liés aux dépendances lors du déploiement, consultez [Résolution des erreurs courantes dans des déploiements Azure avec Azure Resource Manager](common-deployment-errors.md).
* Pour en savoir plus sur la création de modèles Azure Resource Manager, consultez [Création de modèles](template-syntax.md).
* Pour obtenir la liste des fonctions disponibles dans un modèle, consultez [Fonctions de modèle](template-functions.md).


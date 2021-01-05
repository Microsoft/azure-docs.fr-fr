---
title: Créer et utiliser un fournisseur personnalisé
description: Ce tutoriel montre comment créer et utiliser un fournisseur personnalisé Azure. Les fournisseurs personnalisés vous permettent de modifier des workflows dans Azure.
author: jjbfour
ms.topic: tutorial
ms.date: 06/19/2019
ms.author: jobreen
ms.custom: devx-track-azurecli
ms.openlocfilehash: 4f425af7681b666b42fbcc70ac0e4c31d9df6d49
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87503750"
---
# <a name="create-and-use-a-custom-provider"></a>Créer et utiliser un fournisseur personnalisé

Un fournisseur personnalisé implique qu’un contrat soit passé entre Azure et un point de terminaison. Les fournisseurs personnalisés vous permettent de modifier des workflows dans Azure. Ce tutoriel montre comment créer un fournisseur personnalisé. Si vous ne savez pas encore ce qu’est un fournisseur personnalisé Azure, consultez la [présentation des fournisseurs de ressources personnalisés Azure](overview.md).

## <a name="create-a-custom-provider"></a>Créer un fournisseur personnalisé

> [!NOTE]
> Ce tutoriel n’explique pas comment créer un point de terminaison. Si vous ne disposez pas d’un point de terminaison RESTful, suivez le [tutoriel sur la création de points de terminaison RESTful](./tutorial-custom-providers-function-authoring.md), qui constitue la base du tutoriel actuel.

Lorsque vous avez créé un point de terminaison, vous pouvez créer un fournisseur personnalisé afin de générer un contrat entre le fournisseur et le point de terminaison. Un fournisseur personnalisé vous permet de spécifier une liste de définitions de point de terminaison :

```JSON
{
  "name": "myEndpointDefinition",
  "routingType": "Proxy",
  "endpoint": "https://<yourapp>.azurewebsites.net/api/<funcname>?code=<functionkey>"
}
```

Propriété | Obligatoire | Description
---|---|---
**name** | Oui | Nom de la définition de point de terminaison. Azure expose ce nom via son API sous /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders<br>/resourceProviders/{resourceProviderName}/{endpointDefinitionName}
**routingType** | Non | Type de contrat du point de terminaison. Si aucune valeur n’est spécifiée, la valeur par défaut est « proxy ».
**endpoint** | Oui | Point de terminaison vers lequel router les requêtes. Ce point de terminaison gère la réponse, ainsi que tous les effets secondaires de la requête.

La valeur de **endpoint** est l’URL du déclencheur de l’application de fonction Azure. Les espaces réservés `<yourapp>`, `<funcname>` et `<functionkey>` doivent être remplacés par les valeurs de l’application de fonction que vous avez créée.

## <a name="define-custom-actions-and-resources"></a>Définir des actions et des ressources personnalisées

Dans le fournisseur personnalisé, une liste de définitions de points de terminaison modélisée se trouve sous les propriétés **actions** et **resourceTypes**. La propriété **actions** est mappée aux actions personnalisées qui sont exposées par le fournisseur personnalisé, et la propriété **resourceTypes** correspond aux ressources personnalisées. Dans ce tutoriel, le fournisseur personnalisé comprend une propriété **actions** nommée `myCustomAction` et une propriété **resourceTypes** nommée `myCustomResources`.

```JSON
{
  "properties": {
    "actions": [
      {
        "name": "myCustomAction",
        "routingType": "Proxy",
        "endpoint": "https://<yourapp>.azurewebsites.net/api/<funcname>?code=<functionkey>"
      }
    ],
    "resourceTypes": [
      {
        "name": "myCustomResources",
        "routingType": "Proxy",
        "endpoint": "https://<yourapp>.azurewebsites.net/api/<funcname>?code=<functionkey>"
      }
    ]
  },
  "location": "eastus"
}
```

## <a name="deploy-the-custom-provider"></a>Déployer un fournisseur personnalisé

> [!NOTE]
> Vous devez remplacer les valeurs **endpoint** par l’URL de déclencheur de l’application de fonction créée précédemment dans ce tutoriel.

Vous pouvez déployer le fournisseur personnalisé ci-dessus à l’aide d’un modèle Azure Resource Manager :

```JSON
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "Microsoft.CustomProviders/resourceProviders",
            "name": "myCustomProvider",
            "apiVersion": "2018-09-01-preview",
            "location": "eastus",
            "properties": {
                "actions": [
                    {
                        "name": "myCustomAction",
                        "routingType": "Proxy",
                        "endpoint": "https://<yourapp>.azurewebsites.net/api/<funcname>?code=<functionkey>"
                    }
                ],
                "resourceTypes": [
                    {
                        "name": "myCustomResources",
                        "routingType": "Proxy",
                        "endpoint": "https://<yourapp>.azurewebsites.net/api/<funcname>?code=<functionkey>"
                    }
                ]
            }
        }
    ]
}
```

## <a name="use-custom-actions-and-resources"></a>Utiliser des actions et des ressources personnalisées

Après avoir créé un fournisseur personnalisé, vous pouvez utiliser les nouvelles API Azure. Les onglets suivants expliquent comment appeler et utiliser un fournisseur personnalisé.

### <a name="custom-actions"></a>Actions personnalisées

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

> [!NOTE]
> Vous devez remplacer les espaces réservés `{subscriptionId}` et `{resourceGroupName}` par l’abonnement et le groupe de ressources où vous avez déployé le fournisseur personnalisé.

```azurecli-interactive
az resource invoke-action --action myCustomAction \
                          --ids /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/myCustomProvider \
                          --request-body
                            '{
                                "hello": "world"
                            }'
```

Paramètre | Obligatoire | Description
---|---|---
*action* | Oui | Nom de l’action définie dans le fournisseur personnalisé
*ids* | Oui | ID de ressource du fournisseur personnalisé
*request-body* | Non | Corps de la requête qui sera envoyé au point de terminaison

# <a name="template"></a>[Modèle](#tab/template)

Aucun.

---

### <a name="custom-resources"></a>Ressources personnalisées

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

> [!NOTE]
> Vous devez remplacer les espaces réservés `{subscriptionId}` et `{resourceGroupName}` par l’abonnement et le groupe de ressources où vous avez déployé le fournisseur personnalisé.

#### <a name="create-a-custom-resource"></a>Créer une ressource personnalisée

```azurecli-interactive
az resource create --is-full-object \
                   --id /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/myCustomProvider/myCustomResources/myTestResourceName1 \
                   --properties
                    '{
                        "location": "eastus",
                        "properties": {
                            "hello" : "world"
                        }
                    }'
```

Paramètre | Obligatoire | Description
---|---|---
*is-full-object* | Oui | Indique si l’objet properties inclut d’autres options telles que l’emplacement, les étiquettes, une référence SKU ou un plan.
*id* | Oui | ID de ressource de la ressource personnalisée. Cet ID est une extension de l’ID de ressource du fournisseur personnalisé.
*properties* | Oui | Corps de la requête qui sera envoyé au point de terminaison.

#### <a name="delete-a-custom-resource"></a>Supprimer une ressource personnalisée

```azurecli-interactive
az resource delete --id /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/myCustomProvider/myCustomResources/myTestResourceName1
```

Paramètre | Obligatoire | Description
---|---|---
*id* | Oui | ID de ressource de la ressource personnalisée. Cet ID est une extension de l’ID de ressource du fournisseur personnalisé.

#### <a name="retrieve-a-custom-resource"></a>Récupérer une ressource personnalisée

```azurecli-interactive
az resource show --id /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/myCustomProvider/myCustomResources/myTestResourceName1
```

Paramètre | Obligatoire | Description
---|---|---
*id* | Oui | ID de ressource de la ressource personnalisée. Cet ID est une extension de l’ID de ressource du fournisseur personnalisé.

# <a name="template"></a>[Modèle](#tab/template)

Exemple de modèle Resource Manager :

```JSON
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "Microsoft.CustomProviders/resourceProviders/myCustomResources",
            "name": "myCustomProvider/myTestResourceName1",
            "apiVersion": "2018-09-01-preview",
            "location": "eastus",
            "properties": {
                "hello": "world"
            }
        }
    ]
}
```

Paramètre | Obligatoire | Description
---|---|---
*resourceTypeName* | Oui | Valeur `name` de la propriété **resourceTypes** définie dans le fournisseur personnalisé.
*resourceProviderName* | Oui | Nom de l’instance de fournisseur personnalisé.
*customResourceName* | Oui | Nom de la ressource personnalisée.

---

> [!NOTE]
> Une fois que vous en avez terminé avec le déploiement et l’utilisation du fournisseur personnalisé, pensez à nettoyer les ressources créées, y compris l’application de fonction Azure.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert les fournisseurs personnalisés. Pour plus d'informations, consultez les pages suivantes :

- [Procédure : ajouter des actions personnalisées à l’API REST Azure](./custom-providers-action-endpoint-how-to.md)
- [Procédure : ajouter des ressources personnalisées à l’API REST Azure](./custom-providers-resources-endpoint-how-to.md)

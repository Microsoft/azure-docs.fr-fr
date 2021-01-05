---
title: Ajout de ressources personnalisées à l’API REST Azure
description: Découvrez comment ajouter des ressources personnalisées à l’API REST Azure. Cet article vous expliquera les conditions requises et les meilleures pratiques pour les points de terminaison qui souhaitent implémenter des ressources personnalisées.
ms.topic: conceptual
ms.author: jobreen
author: jjbfour
ms.date: 06/20/2019
ms.openlocfilehash: b6c5f5b8e437ad2dc2e8a3be3f3f2ed03a613b44
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75649217"
---
# <a name="adding-custom-resources-to-azure-rest-api"></a>Ajout de ressources personnalisées à l’API REST Azure

Cet article passera par les conditions requises et les meilleures pratiques pour créer des points de terminaison de fournisseur de ressources personnalisées Azure qui implémentent les ressources personnalisées. Si vous n’êtes pas familiarisé avec les fournisseurs de ressources personnalisées Azure, consultez [la vue d’ensemble des fournisseurs de ressources personnalisées](overview.md).

## <a name="how-to-define-a-resource-endpoint"></a>Comment définir un point de terminaison de ressource

Un **point de terminaison** est une URL qui pointe vers un service, qui implémente le contrat sous-jacent entre ce dernier et Azure. Le point de terminaison est défini dans le fournisseur de ressources personnalisées. Il peut s’agir de n’importe quelle URL accessible publiquement. L’exemple ci-dessous a un **resourceType** appelé `myCustomResource` implémenté par `endpointURL`.

Exemple **ResourceProvider** :

```JSON
{
  "properties": {
    "resourceTypes": [
      {
        "name": "myCustomResource",
        "routingType": "Proxy, Cache",
        "endpoint": "https://{endpointURL}/"
      }
    ]
  },
  "location": "eastus",
  "type": "Microsoft.CustomProviders/resourceProviders",
  "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}",
  "name": "{resourceProviderName}"
}
```

## <a name="building-a-resource-endpoint"></a>Création d’un point de terminaison de ressource

Un **point de terminaison** qui implémente un **resourceType** doit gérer la requête et la réponse pour la nouvelle API dans Azure. Lorsqu’un fournisseur de ressources personnalisées avec un **resourceType** est créé, il génère un nouvel ensemble d’API dans Azure. Dans ce cas, le **resourceType** génère une nouvelle API de ressource Azure pour `PUT`, `GET` et `DELETE` afin d’exécuter CRUD sur une ressource unique, ainsi que `GET` pour récupérer toutes les ressources existantes :

Manipuler une ressource unique (`PUT`, `GET` et `DELETE`) :

``` JSON
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResource/{myCustomResourceName}
```

Récupérer toutes les ressources (`GET`) :

``` JSON
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResource
```

En ce qui concerne les ressources personnalisées, les fournisseurs de ressources personnalisées proposent deux types de **routingTypes** : « `Proxy`» et « `Proxy, Cache` ».

### <a name="proxy-routing-type"></a>type de routage proxy

Le **routingType** « `Proxy` » envoie par proxy toutes les méthodes de requête vers le **point de terminaison** spécifié dans le fournisseur de ressources personnalisées. Quand utiliser «`Proxy` » :

- Un contrôle total sur la réponse est nécessaire.
- Intégration de systèmes à des ressources existantes.

Pour en savoir plus sur les ressources « `Proxy` », consultez [Informations de référence sur les proxys de ressources personnalisées](proxy-resource-endpoint-reference.md)

### <a name="proxy-cache-routing-type"></a>type de routage de cache de proxy

Le **routingType** « `Proxy, Cache` » envoie par proxy les méthodes de requête `PUT` et `DELETE` uniquement vers le **point de terminaison** spécifié dans le fournisseur de ressources personnalisées. Le fournisseur de ressources personnalisées renvoie automatiquement les requêtes `GET` en fonction de ce qui est stocké dans son cache. Si une ressource personnalisée est marquée avec un cache, le fournisseur de ressources personnalisées ajoute/remplace également des champs dans la réponse pour que les API soient compatibles avec Azure. Quand utiliser «`Proxy, Cache` » :

- Création d’un nouveau système ne contenant aucune ressource existante.
- Utiliser un écosystème Azure existant.

Pour en savoir plus sur les ressources « `Proxy, Cache` », consultez [Informations de référence sur les caches de ressources personnalisées](proxy-cache-resource-endpoint-reference.md)

## <a name="creating-a-custom-resource"></a>Création d’une ressource personnalisée

Deux méthodes permettent de créer une ressource personnalisée dans un fournisseur de ressources personnalisées :

- Azure CLI
- Modèles Azure Resource Manager

### <a name="azure-cli"></a>Azure CLI

Créer une ressource personnalisée :

```azurecli-interactive
az resource create --is-full-object \
                   --id /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/{resourceTypeName}/{customResourceName} \
                   --properties \
                    '{
                        "location": "eastus",
                        "properties": {
                            "myProperty1": "myPropertyValue1",
                            "myProperty2": {
                                "myProperty3": "myPropertyValue3"
                            }
                        }
                    }'
```

Paramètre | Obligatoire | Description
---|---|---
is-full-object | *Oui* | Indique que l’objet properties inclut d’autres options telles que l’emplacement, les étiquettes, une référence SKU et/ou un plan.
id | *Oui* | ID de ressource de la ressource personnalisée. Il doit provenir de **ResourceProvider**
properties | *Oui* | Corps de la demande qui sera envoyé au **point de terminaison**.

Supprimer une ressource personnalisée Azure :

```azurecli-interactive
az resource delete --id /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/{resourceTypeName}/{customResourceName}
```

Paramètre | Obligatoire | Description
---|---|---
id | *Oui* | ID de ressource de la ressource personnalisée. Il doit provenir de **ResourceProvider**.

Récupérer une ressource personnalisée Azure :

```azurecli-interactive
az resource show --id /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/{resourceTypeName}/{customResourceName}
```

Paramètre | Obligatoire | Description
---|---|---
id | *Oui* | ID de ressource de la ressource personnalisée. Il doit provenir de **ResourceProvider**

### <a name="azure-resource-manager-template"></a>Modèle Azure Resource Manager

> [!NOTE]
> Les ressources nécessitent que la réponse contienne un `id`, `name` et `type` appropriés du **point de terminaison**.

Les modèles Azure Resource Manager nécessitent que `id`, `name` et `type` soient retournés correctement du point de terminaison en aval. Une réponse de ressource retournée doit se présenter sous la forme suivante :

Exemple de réponse de **point de terminaison** :

``` JSON
{
  "properties": {
    "myProperty1": "myPropertyValue1",
    "myProperty2": {
        "myProperty3": "myPropertyValue3"
    }
  },
  "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{customResourceName}",
  "name": "{customResourceName}",
  "type": "Microsoft.CustomProviders/resourceProviders/{resourceTypeName}"
}
```

Exemple de modèle Azure Resource Manager :

```JSON
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "Microsoft.CustomProviders/resourceProviders/{resourceTypeName}",
            "name": "{resourceProviderName}/{customResourceName}",
            "apiVersion": "2018-09-01-preview",
            "location": "eastus",
            "properties": {
                "myProperty1": "myPropertyValue1",
                "myProperty2": {
                    "myProperty3": "myPropertyValue3"
                }
            }
        }
    ]
}
```

Paramètre | Obligatoire | Description
---|---|---
resourceTypeName | *Oui* | **Nom** du **resourceType** défini dans le fournisseur de ressources personnalisées.
resourceProviderName | *Oui* | Nom de l’instance de fournisseur de ressources personnalisées.
customResourceName | *Oui* | Nom de la ressource personnalisée.

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble des fournisseurs de ressources personnalisées Azure](overview.md)
- [Démarrage rapide : Créer un fournisseur de ressources personnalisées Azure et déployer des ressources personnalisées](./create-custom-provider.md)
- [Tutoriel : Créer des actions et des ressources personnalisées dans Azure](./tutorial-get-started-with-custom-providers.md)
- [Guide pratique pour Ajout d’actions personnalisées à l’API REST Azure](./custom-providers-action-endpoint-how-to.md)
- [Reference : Informations de référence sur les proxys de ressources personnalisées](proxy-resource-endpoint-reference.md)
- [Reference : Informations de référence sur les caches de ressources personnalisées](proxy-cache-resource-endpoint-reference.md)

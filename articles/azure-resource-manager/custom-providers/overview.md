---
title: Vue d’ensemble des fournisseurs personnalisés
description: Découvrez les fournisseurs de ressources personnalisés Azure et comment étendre le plan de l’API Azure pour l’ajuster à vos workflows.
author: jjbfour
ms.topic: conceptual
ms.date: 06/19/2019
ms.author: jobreen
ms.openlocfilehash: 68b8bd187d58cd71778b8a922684cc3817a0715d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80398478"
---
# <a name="azure-custom-resource-providers-overview"></a>Présentation des fournisseurs de ressources personnalisés Azure

Les fournisseurs de ressources personnalisés Azure constituent une plateforme d’extensibilité à Azure. Cette dernière vous permet de définir des API personnalisées qui peuvent être utilisées pour enrichir l’expérience Azure par défaut. Cette documentation décrit :

- Comment créer et déployer un fournisseur de ressources personnalisé Azure.
- Comment utiliser des fournisseurs de ressources personnalisés Azure pour étendre les workflows existants.
- Où trouver des guides et exemples de code pour commencer.

![Vue d’ensemble du fournisseur personnalisé](./media/overview/overview.png)

> [!IMPORTANT]
> Les fournisseurs personnalisés sont actuellement en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge.
> Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="what-can-custom-resource-providers-do"></a>Utilité des fournisseurs de ressources personnalisés

Voici quelques exemples de ce que vous pouvez faire avec les fournisseurs de ressources personnalisés Azure :

- Étendre les API REST Azure Resource Manager pour inclure des services internes et externes.
- Activer des scénarios personnalisés sur des workflows Azure existants.
- Personnaliser le contrôle et l’effet des modèles Azure Resource Manager.

## <a name="what-is-a-custom-resource-provider"></a>Présentation d’un fournisseur de ressources personnalisé

Les fournisseurs de ressources personnalisés Azure sont mis en place en créant un contrat entre Azure et un point de terminaison. Ce contrat définit une liste des nouvelles ressources et actions via une nouvelle ressource, **Microsoft.CustomProviders/resourceProviders**. Le fournisseur de ressources personnalisé expose ensuite ces nouvelles API dans Azure. Les fournisseurs de ressources personnalisés Azure sont composés de 3 parties : le fournisseur de ressources personnalisé, les **points de terminaison** et les ressources personnalisées.

## <a name="how-to-build-custom-resource-providers"></a>Guide pratique pour créer des fournisseurs de ressources personnalisés Azure

Les fournisseurs de ressources personnalisés sont une liste de contrats entre Azure et des points de terminaison. Ce contrat décrit comment Azure doit interagir avec un point de terminaison. Le fournisseur de ressources agit comme un proxy et transfère les demandes et réponses vers et à partir du **point de terminaison** spécifié. Un fournisseur de ressources peut spécifier 2 types de contrat : [**resourceTypes**](./custom-providers-resources-endpoint-how-to.md) et [**actions**](./custom-providers-action-endpoint-how-to.md). Ceux-ci sont activés par le biais des définitions de point de terminaison. Une définition de point de terminaison se compose de 3 champs : **name**, **routingType** et **endpoint**.

Exemple de point de terminaison :

```JSON
{
  "name": "{endpointDefinitionName}",
  "routingType": "Proxy",
  "endpoint": "https://{endpointURL}/"
}
```

Propriété | Obligatoire | Description
---|---|---
name | *Oui* | Nom de la définition de point de terminaison. Azure expose ce nom via son API sous « /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/<br>resourceProviders/{resourceProviderName}/{endpointDefinitionName} »
routingType | *non* | Détermine le type de contrat avec le **point de terminaison**. En l’absence de spécification, « Proxy » est spécifié par défaut.
endpoint | *Oui* | Point de terminaison vers lequel router les requêtes. Il gère la réponse, ainsi que tous les effets secondaires de la requête.

### <a name="building-custom-resources"></a>Création de ressources personnalisées

**ResourceTypes** décrit les nouvelles ressources personnalisées qui sont ajoutées à Azure. Celles-ci exposent des méthodes CRUD RESTful de base. Consultez [d’autres informations sur la création de ressources personnalisées](./custom-providers-resources-endpoint-how-to.md).

Exemple de fournisseur de ressources personnalisé avec **resourceTypes** :

```JSON
{
  "properties": {
    "resourceTypes": [
      {
        "name": "myCustomResources",
        "routingType": "Proxy",
        "endpoint": "https://{endpointURL}/"
      }
    ]
  },
  "location": "eastus"
}
```

API ajoutées à Azure pour l’exemple ci-dessus :

HttpMethod | URI d’exemple | Description
---|---|---
PUT | /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/<br>providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/<br>myCustomResources/{customResourceName}?api-version=2018-09-01-preview | Appel d’API REST Azure pour créer une ressource.
Suppression | /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/<br>providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/<br>myCustomResources/{customResourceName}?api-version=2018-09-01-preview | Appel d’API REST Azure pour supprimer une ressource existante.
GET | /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/<br>providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/<br>myCustomResources/{customResourceName}?api-version=2018-09-01-preview | Appel d’API REST Azure pour récupérer une ressource existante.
GET | /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/<br>providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/<br>myCustomResources?api-version=2018-09-01-preview | Appel d’API REST Azure pour récupérer la liste des ressources existantes.

### <a name="building-custom-actions"></a>Création d’actions personnalisées

Les **actions** décrivent les nouvelles actions ajoutées à Azure. Celles-ci peuvent être exposées sur le fournisseur de ressources ou imbriquées sous un **resourceType**. Consultez [d’autres informations sur la création d’actions personnalisées](./custom-providers-action-endpoint-how-to.md).

Exemple de fournisseur de ressources personnalisé avec **actions** :

```JSON
{
  "properties": {
    "actions": [
      {
        "name": "myCustomAction",
        "routingType": "Proxy",
        "endpoint": "https://{endpointURL}/"
      }
    ]
  },
  "location": "eastus"
}
```

API ajoutées à Azure pour l’exemple ci-dessus :

HttpMethod | URI d’exemple | Description
---|---|---
POST | /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/<br>providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/<br>myCustomAction?api-version=2018-09-01-preview | Appel d’API REST Azure pour activer l’action.

## <a name="looking-for-help"></a>Besoin d’aide

Si vous avez des questions sur le développement de fournisseurs de ressources personnalisés Azure, postez-les sur [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-custom-providers). Avant de la publier, vérifiez si votre réponse a déjà été posée et a déjà reçu une réponse. Ajoutez le mot clé ```azure-custom-providers``` pour obtenir une réponse rapide !

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert les fournisseurs personnalisés. Passez au suivant pour créer un fournisseur personnalisé.

- [Démarrage rapide : Créer un fournisseur de ressources personnalisées Azure et déployer des ressources personnalisées](./create-custom-provider.md)
- [Tutoriel : Créer des actions et des ressources personnalisées dans Azure](./tutorial-get-started-with-custom-providers.md)
- [Guide pratique pour Ajout d’actions personnalisées à l’API REST Azure](./custom-providers-action-endpoint-how-to.md)
- [Guide pratique pour ajouter des ressources personnalisées à l’API REST Azure](./custom-providers-resources-endpoint-how-to.md)

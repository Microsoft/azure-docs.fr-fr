---
title: Définir les suggestions personnalisées de la Suggestion automatique - Recherche personnalisée Bing
titleSuffix: Azure Cognitive Services
description: La suggestion automatique personnalisée retourne une liste de suggestions de chaînes de requête qui correspondent à votre recherche.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-custom-search
ms.topic: conceptual
ms.date: 02/12/2019
ms.author: maheshb
ms.openlocfilehash: 986a7c8f1282b76c2ae94eebcc2c31dc3b3e7cad
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352012"
---
# <a name="configure-your-custom-autosuggest-experience"></a>Configurer votre expérience de Suggestion automatique personnalisée

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

La suggestion automatique personnalisée retourne une liste de suggestions de chaînes de requête qui correspondent à votre recherche. Les chaînes de requête suggérées sont basées sur la chaîne de requête partielle qui est fournie par l’utilisateur dans la zone de recherche. La liste contient un maximum de 10 suggestions. 

Vous spécifiez s’il faut retourner uniquement des suggestions personnalisées ou inclure également des suggestions Bing. Si vous incluez les suggestions Bing, les suggestions personnalisées apparaissent avant celles-ci. Si vous fournissez suffisamment de suggestions pertinentes, il est possible que la liste de suggestions retournée n’inclue pas les suggestions de Bing. Les suggestions Bing correspondent toujours au contexte de votre instance de Recherche personnalisée. 

Pour configurer des suggestions de requête de recherche pour votre instance, cliquez sur l’onglet **Suggestion automatique**.  

> [!NOTE]
> Pour utiliser cette fonctionnalité, vous devez vous abonner à la Recherche personnalisée au niveau approprié (voir [Tarifs](https://azure.microsoft.com/pricing/details/cognitive-services/bing-custom-search/)).

Les suggestions peuvent mettre jusqu’à 24 heures pour apparaître dans le point de terminaison de service (c’est-à-dire l’API ou l’interface utilisateur hébergée).

## <a name="enable-bing-suggestions"></a>Activer les suggestions Bing

Pour activer les suggestions Bing, placez le curseur **Automatic Bing suggestions** (Suggestions automatiques Bing) sur la position d’activation. Le curseur devient bleu.

## <a name="add-your-own-suggestions"></a>Ajouter vos propres suggestions

Pour ajouter vos propres suggestions de chaînes de requête, ajoutez-les à la liste qui se trouve sous **Suggestions définies par l’utilisateur**. Après avoir ajouté une suggestion à la liste, appuyez sur la touche Entrée ou cliquez sur l’icône **+** . Vous pouvez spécifier la suggestion dans n’importe quelle langue. Vous pouvez ajouter un maximum de 5 000 suggestions de chaînes de requête.

## <a name="upload-suggestions"></a>Charger des suggestions

Si vous le souhaitez, vous pouvez charger un fichier contenant la liste de suggestions. Le fichier doit contenir une seule chaîne de requête de recherche par ligne. Pour charger le fichier, cliquez sur l’icône de chargement, puis sélectionnez le fichier à charger. Le service extrait les suggestions du fichier et les ajoute à la liste.

## <a name="remove-suggestions"></a>Supprimer des suggestions

Pour supprimer une suggestion, cliquez sur l’icône de suppression en regard de la suggestion à éliminer.

## <a name="block-suggestions"></a>Bloquer des suggestions

Si vous incluez des suggestions Bing, vous pouvez ajouter une liste comprenant les chaînes de requête que Bing ne doit pas retourner. Pour ajouter une chaîne de requête à bloquer, cliquez sur **Show blocked suggestions** (Afficher les suggestions bloquées). Ajoutez la chaîne de requête à la liste, puis appuyez sur la touche Entrée ou cliquez sur l’icône **+** . Vous pouvez bloquer un maximum de 50 chaînes de requête.



[!INCLUDE [publish or revert](./includes/publish-revert.md)]

>[!NOTE]  
>Les modifications de configuration de la Suggestion automatique personnalisée peuvent demander jusqu’à 24 heures pour prendre effet.


## <a name="enabling-autosuggest-in-hosted-ui"></a>Activation de la suggestion automatique dans l’interface utilisateur hébergée

Pour activer les suggestions de chaînes de requête pour votre interface utilisateur hébergée, cliquez sur **Hosted UI** (Interface utilisateur hébergée). Faites défiler jusqu’à la section **Additional Configuration** (Configuration supplémentaire). Sous **Recherche Web**, sélectionnez **Activé** pour l’option **Enable autosuggest** (Activer la suggestion automatique). Pour activer la suggestion automatique, vous devez sélectionner une disposition qui comprenne une zone de recherche.


## <a name="calling-the-autosuggest-api"></a>Appel de l’API Suggestion automatique

Pour obtenir des suggestions de chaînes de requête à l’aide de l’API Recherche personnalisée Bing, envoyez une requête `GET` au point de terminaison suivant.

```
GET https://api.cognitive.microsoft.com/bingcustomsearch/v7.0/Suggestions 
```

La réponse contient une liste d’objets `SearchAction` contenant les chaînes de requête suggérées.

```
        {  
            "displayText" : "sailing lessons seattle",  
            "query" : "sailing lessons seattle",  
            "searchKind" : "CustomSearch"  
        },  
```

Chaque suggestion comporte les champs `displayText` et `query`. Le champ `displayText` correspond à la chaîne de requête suggérée que vous utilisez pour remplir la liste déroulante de la zone de recherche.

Si l’utilisateur sélectionne une chaîne de requête suggérée dans la liste déroulante, utilisez la chaîne de requête dans le champ `query` lorsque vous appelez [l’API Recherche personnalisée Bing](overview.md).


## <a name="next-steps"></a>Étapes suivantes

- [Obtenir des suggestions personnalisées]()
- [Rechercher votre instance personnalisée](./search-your-custom-view.md)
- [Configurer et consommer l’interface utilisateur hébergée personnalisée](./hosted-ui.md)
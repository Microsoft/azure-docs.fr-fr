---
title: 'ML Studio (classique) : Gérer les données Azure AI Gallery - Azure'
description: Vous pouvez exporter et supprimer d’Azure AI Gallery vos données utilisateurs intégrées au produit en utilisant l’interface ou l’API de catalogue AI Gallery. Cet article vous montre comment procéder.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18
ms.date: 05/25/2018
ms.reviewer: jmartens, mldocs
ms.openlocfilehash: 6276d5c8d333f29c55720f2255f2cdf27e1e36cc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91338803"
---
# <a name="view-and-delete-in-product-user-data-from-azure-ai-gallery"></a>Visualiser et supprimer d’Azure AI Gallery des données utilisateur intégrées au produit



Vous pouvez visualiser et supprimer d’Azure AI Gallery vos données utilisateurs intégrées au produit en utilisant l’interface ou l’API de catalogue AI Gallery. Cet article vous explique comment procéder.

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-dsr-and-stp-note.md)]

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

## <a name="view-your-data-in-ai-gallery-with-the-ui"></a>Visualiser vos données dans AI Gallery avec l’interface utilisateur

Vous pouvez visualiser les éléments que vous avez publiés via l’interface utilisateur du site web Azure AI Gallery. Les utilisateurs peuvent visualiser les solutions, projets et expériences publics et non répertoriés, ainsi que d’autres éléments publiés :

1.    Connectez-vous à [Azure AI Gallery](https://gallery.azure.ai/).
2.    Cliquez sur l’image du profil dans le coin supérieur droit, puis sur le nom du compte pour charger la page de votre profil.
3.    La page de profil affiche tous les éléments publiés sur la galerie, y compris les entrées non répertoriées.

## <a name="use-the-ai-gallery-catalog-api-to-view-your-data"></a>Utiliser l’API de catalogue AI Gallery pour visualiser vos données

Vous pouvez visualiser par programmation les données collectées via l’API de catalogue AI Gallery, qui est accessible sur https://catalog.cortanaanalytics.com/entities. Pour visualiser des données, vous avez besoin de votre ID d’auteur. Pour visualiser des entités non répertoriées via l’API de catalogue, vous avez besoin d’un jeton d’accès.

Les réponses du catalogue sont retournées au format JSON.

### <a name="get-an-author-id"></a>Obtenir un ID d’auteur
L’ID d’auteur est basé sur l’adresse e-mail utilisée lors de la publication sur Azure AI Gallery. Il ne change pas :

1. Connectez-vous à [Azure AI Gallery](https://gallery.azure.ai/).
2. Cliquez sur l’image du profil dans le coin supérieur droit, puis sur le nom du compte pour charger la page de votre profil.
3. L’URL dans la barre d’adresse affiche l’ID alphanumérique après `authorId=`. Par exemple, pour l’URL :\
    `https://gallery.azure.ai/Home/Author?authorId=99F1F5C6260295F1078187FA179FBE08B618CB62129976F09C6AF0923B02A5BA`

    ID de l’auteur :\
    `99F1F5C6260295F1078187FA179FBE08B618CB62129976F09C6AF0923B02A5BA`

### <a name="get-your-access-token"></a>Obtenir votre jeton d’accès

Vous avez besoin d’un jeton d’accès pour visualiser des entités non répertoriées via l’API de catalogue. Sans jeton d’accès, les utilisateurs peuvent néanmoins visualiser les entités publiques et d’autres informations utilisateur.

Pour obtenir un jeton d’accès, vous devez examiner l’en-tête `DataLabAccessToken` d’une requête HTTP que le navigateur fait auprès de l’API de catalogue quand vous êtes connecté :

1.    Connectez-vous à [Azure AI Gallery](https://gallery.azure.ai/).
2.    Cliquez sur l’image du profil dans le coin supérieur droit, puis sur le nom du compte pour charger la page de votre profil.
3.    Ouvrez le volet Outils de développement du navigateur en appuyant sur F12, sélectionnez l’onglet Réseau, puis actualisez la page. 
4. Filtrez les requêtes sur la chaîne *catalog* en la tapant dans la zone de texte Filtre.
5.    Dans les requêtes auprès de l’URL `https://catalog.cortanaanalytics.com/entities`, recherchez une requête GET, puis sélectionnez l’onglet *En-têtes*. Faites défiler jusqu’à la section *En-têtes des requêtes*.
6.    Le jeton alphanumérique se trouve sous l’en-tête `DataLabAccessToken`. Pour conserver vos données en sécurité, ne partagez pas ce jeton.

### <a name="view-user-information"></a>Visualiser les informations utilisateur
Avec l’ID d’auteur que vous avez obtenu lors des étapes précédentes, visualisez les informations d’un profil d’utilisateur en remplaçant `[AuthorId]` dans l’URL suivante :

`https://catalog.cortanaanalytics.com/users/[AuthorID]`

Par exemple, cette requête d’URL :

`https://catalog.cortanaanalytics.com/users/99F1F5C6260295F1078187FA179FBE08B618CB62129976F09C6AF0923B02A5BA`

Retourne une réponse comme :

```json
{"entities_count":9,"contribution_score":86.351575190956922,"scored_at":"2018-05-07T14:30:25.9305671+00:00","contributed_at":"2018-05-07T14:26:55.0381756+00:00","created_at":"2017-12-15T00:49:15.6733094+00:00","updated_at":"2017-12-15T00:49:15.6733094+00:00","name":"First Last","slugs":["First-Last"],"tenant_id":"14b2744cf8d6418c87ffddc3f3127242","community_id":"9502630827244d60a1214f250e3bbca7","id":"99F1F5C6260295F1078187FA179FBE08B618CB62129976F09C6AF0923B02A5BA","_links":{"self":"https://catalog.azureml.net/tenants/14b2744cf8d6418c87ffddc3f3127242/communities/9502630827244d60a1214f250e3bbca7/users/99F1F5C6260295F1078187FA179FBE08B618CB62129976F09C6AF0923B02A5BA"},"etag":"\"2100d185-0000-0000-0000-5af063010000\""}
```

### <a name="view-public-entities"></a>Visualiser des entités publiques

L’API de catalogue stocke des informations concernant les entités publiées sur Azure AI Gallery, que vous pouvez également consulter directement sur le [site web AI Gallery](https://gallery.azure.ai/). 

Pour visualiser des entités publiées, accédez à l’URL suivante, en remplaçant `[AuthorId]` par l’ID d’auteur obtenu dans [Obtenir un ID d’auteur](#get-an-author-id) ci-dessus.

`https://catalog.cortanaanalytics.com/entities?$filter=author/id eq '[AuthorId]'`

Par exemple :

`https://catalog.cortanaanalytics.com/entities?$filter=author/id eq '99F1F5C6260295F1078187FA179FBE08B618CB62129976F09C6AF0923B02A5BA'`

### <a name="view-unlisted-and-public-entities"></a>Visualiser des entités non répertoriées et publiques

Cette requête affiche seulement des entités publiques. Pour visualiser toutes vos entités, y compris celles qui sont non répertoriées, spécifiez le jeton d’accès obtenu à la section précédente.

1.    Avec un outil comme [Postman](https://www.getpostman.com), créez une requête HTTP GET sur l’URL du catalogue, comme décrit dans [Obtenir votre jeton d’accès](#get-your-access-token).
2.    Créez un en-tête de requête HTTP nommé `DataLabAccessToken`, avec la valeur définie sur le jeton d’accès.
3.    Envoyez la requête HTTP.

> [!TIP]
> Si les entités non répertoriées n’apparaissent pas dans les réponses de l’API de catalogue, la raison peut en être que l’utilisateur a un jeton d’accès non valide ou expiré. Déconnectez-vous d’Azure AI Gallery, puis répétez les étapes de [Obtenir votre jeton d’accès](#get-your-access-token) pour renouveler le jeton. 

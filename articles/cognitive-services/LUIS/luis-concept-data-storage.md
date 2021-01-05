---
title: Stockage des données – LUIS
titleSuffix: Azure Cognitive Services
description: LUIS stocke les données chiffrées dans un magasin de données Azure correspondant à la région spécifiée par la clé.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 12/07/2020
ms.openlocfilehash: a58bcff4e39c4a4a907cd8567b47b074ff299bd5
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97008450"
---
# <a name="data-storage-and-removal-in-language-understanding-luis-cognitive-services"></a>Stockage et suppression des données dans Language Understanding (LUIS) Cognitive Services

LUIS stocke les données chiffrées dans un magasin de données Azure correspondant à [la région](luis-reference-regions.md) spécifiée par la clé. 

* Les données utilisées pour l’apprentissage du modèle, telles que les entités, les intentions et les énoncés sont enregistrées dans LUIS pendant la durée de vie de l’application. Si un propriétaire ou un contributeur supprime l’application, ces données sont supprimées. Si une application n’a plus été utilisée depuis 90 jours, elle est supprimée. 

* Les auteurs d’applications peuvent choisir d’[activer la journalisation](luis-how-to-review-endpoint-utterances.md#log-user-queries-to-enable-active-learning) sur les énoncés envoyés à une application publiée. Si cette fonctionnalité est activée, les énoncés sont conservés pendant 30 jours et peuvent être consulté par l’auteur de l’application. Si la journalisation n’est pas activée lors de la publication de l’application, ces données ne sont pas stockées.

## <a name="export-and-delete-app"></a>Exporter et supprimer l’application
Les utilisateurs ont le contrôle total sur l’[exportation](luis-how-to-start-new-app.md#export-app) et la [suppression](luis-how-to-start-new-app.md#delete-app) de l’application. 

## <a name="utterances"></a>Énoncés

Les énoncés peuvent être stockées à deux emplacements différents. 

* Au cours du **processus de création**, les énoncés sont créés et stockés dans l’intention. Les énoncés dans les intentions sont requis pour un bon fonctionnement de l’application LUIS. Une fois que l’application est publiée et reçoit des requêtes au point de terminaison, chaîne de requête de la demande de point de terminaison, `log=false`, détermine si l’énoncé du point de terminaison est stocké. Si le point de terminaison est stocké, il fait alors partie des énoncés d’apprentissage actif trouvés dans la section **Build** du portail, dans la section **Passer en revue les énoncés des points de terminaison**. 
* Lorsque vous **examinez les énoncés des points de terminaison** et ajoutez un énoncé à une intention, l’énoncé n’est plus stocké dans le cadre des énoncés de point de terminaison à passer en revue. Il est ajouté aux intentions de l’application. 

<a name="utterances-in-an-intent"></a>

### <a name="delete-example-utterances-from-an-intent"></a>Supprimer des exemples d’énoncés d’une intention

Supprimer des exemples d’énoncés utilisés pour l’apprentissage de [LUIS](luis-reference-regions.md). Si vous supprimez un exemple d’énoncé de votre application LUIS, il est supprimé du service web LUIS et n’est plus disponible pour l’exportation.

<a name="utterances-in-review"></a>

### <a name="delete-utterances-in-review-from-active-learning"></a>Supprimer des énoncés de la révision de l’apprentissage actif

Vous pouvez supprimer des énoncés de la liste des énoncés d’utilisateurs que LUIS suggère dans la page **[Réviser les énoncés de point de terminaison](luis-how-to-review-endpoint-utterances.md)** . La suppression d’énoncés de cette liste a pour effet d’empêcher leur suggestion, mais pas de les supprimer des journaux d’activité.

Si vous ne souhaitez pas avoir d’énoncés d’apprentissage actif, vous pouvez [désactiver l’apprentissage actif](luis-how-to-review-endpoint-utterances.md#disable-active-learning). La désactivation de l’apprentissage actif désactive également la journalisation.

### <a name="disable-logging-utterances"></a>Désactiver la journalisation des énoncés
[La désactivation de l’apprentissage actif](luis-how-to-review-endpoint-utterances.md#disable-active-learning) désactive la journalisation.


<a name="accounts"></a>

## <a name="delete-an-account"></a>Supprimer un compte
Si vous n’avez pas migré, vous pouvez supprimer votre compte et toutes vos applications seront supprimées ainsi que leurs exemples d’énoncés et journaux. Les données sont conservées pendant 90 jours avant que le compte et les données ne soient définitivement supprimés.

La suppression d’un compte est possible à partir de la page **Paramètres**. Sélectionnez le nom de votre compte en haut à droite dans la barre de navigation supérieure pour accéder à la page **Paramètres**.

## <a name="delete-an-authoring-resource"></a>Supprimer une ressource de création
Si vous avez [migré vers une ressource de création](./luis-migration-authoring.md), la suppression de la ressource elle-même dans le portail Azure supprimera toutes vos applications associées à cette ressource, ainsi que leurs exemples d’énoncés et journaux. Les données sont conservées pendant 90 jours avant leur suppression définitive.    

Pour supprimer votre ressource, accédez au [portail Azure](https://ms.portal.azure.com/#home) et sélectionnez votre ressource de création LUIS. Accédez à l’onglet **Vue d’ensemble** et cliquez sur le bouton **Supprimer** en haut de la page. Confirmez ensuite que votre ressource a été supprimée. 

## <a name="data-inactivity-as-an-expired-subscription"></a>Inactivité de données considérée comme un abonnement arrivé à expiration
Dans le cadre de la conservation et de la suppression des données, une application LUIS inactive peut, à la _discrétion de Microsoft_ être traitée comme un abonnement arrivé à expiration. Une application est considérée comme inactive si elle répond aux critères suivants pendant les 90 derniers jours : 

* Elle n’a reçu **aucun** appel.
* Elle n’a pas été modifiée.
* Aucune clé actuelle ne lui est assignée.
* Aucun utilisateur ne s’y est connecté.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur l’exportation et la suppression d’une application](luis-how-to-start-new-app.md)
---
title: 'Démarrage rapide : Créer un index de recherche à l’aide des API REST'
titleSuffix: Azure Cognitive Search
description: Dans ce guide de démarrage rapide des API REST, découvrez comment appeler les API REST de Recherche cognitive Azure en utilisant Postman ou Visual Studio Code.
zone_pivot_groups: URL-test-interface-rest-apis
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: quickstart
ms.devlang: rest-api
ms.date: 11/17/2020
ms.openlocfilehash: 971ee806d8b5f9a336b40d96d500ce47d2f5166e
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94711335"
---
# <a name="quickstart-create-an-azure-cognitive-search-index-using-rest-apis"></a>Démarrage rapide : Créer un index Recherche cognitive Azure à l’aide des API REST

Cet article explique comment formuler des requêtes API REST de façon interactive à l’aide des [API REST de la Recherche cognitive Azure](/rest/api/searchservice) et d’un API client pour envoyer et recevoir les requêtes. Avec un API client et ces instructions, vous pouvez envoyer des requêtes et afficher les réponses avant d’écrire un code.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

::: zone pivot="url-test-tool-rest-postman"

[!INCLUDE [Send requests using Postman](includes/search-get-started-rest-postman.md)]

::: zone-end

::: zone pivot="url-test-tool-rest-vscode-ext"

[!INCLUDE [Send requests using Visual Studio Code](includes/search-get-started-rest-vscode-ext.md)]

::: zone-end

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous travaillez dans votre propre abonnement, il est recommandé, à la fin de chaque projet, de déterminer si vous avez toujours besoin des ressources que vous avez créées. Les ressources laissées en cours d’exécution peuvent vous coûter de l’argent. Vous pouvez supprimer les ressources une par une, ou choisir de supprimer le groupe de ressources afin de supprimer l’ensemble des ressources.

Vous pouvez rechercher et gérer les ressources dans le portail à l’aide des liens **Toutes les ressources** ou **Groupes de ressources** situés dans le volet de navigation de gauche.

Si vous utilisez un service gratuit, n’oubliez pas que vous êtes limité à trois index, indexeurs et sources de données. Vous pouvez supprimer des éléments un par un dans le portail pour ne pas dépasser la limite. 

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez effectuer les tâches de base, vous pouvez aller plus loin avec les appels d’API REST supplémentaires pour des fonctionnalités plus avancées, telles que les indexeurs ou la [configuration d’un pipeline d’enrichissement](cognitive-search-tutorial-blob.md) qui ajoute des transformations de contenu à l’indexation. Pour les étapes suivantes, nous vous recommandons le lien suivant :

> [!div class="nextstepaction"]
> [Tutoriel : Utiliser REST et l’IA pour générer du contenu pouvant faire l’objet de recherches à partir d’objets blob Azure](cognitive-search-tutorial-blob.md)
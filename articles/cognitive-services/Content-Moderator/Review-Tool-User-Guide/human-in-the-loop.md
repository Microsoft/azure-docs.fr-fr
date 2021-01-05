---
title: Découvrir les concepts de l’outil de révision - Content Moderator
titleSuffix: Azure Cognitive Services
description: Découvrez l’outil de révision de Content Moderator, un site Web qui coordonne les efforts de modération d’une IA et la vérification humaine.
services: cognitive-services
author: PatrickFarley
manager: mikemcca
ms.date: 03/15/2019
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.author: pafarley
ms.openlocfilehash: ddf4d17cd2631f5bc299e392c46bd9065bb13744
ms.sourcegitcommit: 4b76c284eb3d2b81b103430371a10abb912a83f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2020
ms.locfileid: "93146613"
---
# <a name="content-moderator-review-tool"></a>Outil de révision de Content Moderator

Azure Content Moderator fournit des services pour combiner la modération du contenu Machine Learning aux révisions humaines. Le site web de l’[outil de révision](https://contentmoderator.cognitive.microsoft.com) est un front-end convivial qui procure un accès détaillé à ces services.

## <a name="what-it-does"></a>Résultat

Quand il est utilisé conjointement avec les API de modération assistée par ordinateur, [l’outil de révision](https://contentmoderator.cognitive.microsoft.com) vous permet d’accomplir les tâches suivantes en relation avec le cycle de vie de modération du contenu :

- Utilisez un ensemble d’outils pour modérer le contenu dans plusieurs formats (texte, image et vidéo).
- Automatisez la création de [révisions](../review-api.md#reviews) humaines lorsque les résultats de l’API de modération deviennent disponibles.
- Affectez ou remontez les révisions de contenu à plusieurs équipes de révision organisées par catégorie de contenu ou niveau d’expérience.
- Utilisez des filtres de logique ([flux de travail](../review-api.md#workflows)) personnalisés ou ceux par défaut pour trier et suivre le contenu sans écrire de code.
- Utilisez des [connecteurs](./configure.md#connectors) pour traiter le contenu avec les services Microsoft PhotoDNA, Analyse de texte et Visage en plus des API Content Moderator.
- Récupérez les principales métriques de performances de vos processus de modération du contenu.

## <a name="review-tool-dashboard"></a>Tableau de bord de l’outil de révision

Dans l’onglet **Tableau de bord** , vous pouvez voir les métriques essentielles pour les révisions de contenu effectuées au sein de l’outil. Découvrez le nombre de révisions terminées, en attente et leur total pour le contenu vidéo, image et texte. 

Le tableau **Pending reviews** montre la répartition des utilisateurs et des sous-équipes qui ont des révisions en attente ou terminées, ainsi que la durée de contrat SLA restante. Vous pouvez sélectionner les éléments du tableau pour accéder à leurs révisions. La zone de recherche située au-dessus du tableau vous permet de filtrer les résultats par nom d’équipe, tandis que l’icône de **filtre** vous permet de filtrer d’après d’autres métriques.

L’onglet **Completed reviews** permet de voir le nombre total d’éléments traités ou terminés par des utilisateurs et des sous-équipes. Vous pouvez filtrer ces données de la même façon que les révisions en attente.

Un clic sur le texte dans le coin supérieur droit du tableau de bord permet d’afficher les métriques personnelles quotidiennes, qui indiquent le nombre de révisions effectuées pour chaque type de contenu.

> [!div class="mx-imgBorder"]
> ![Le tableau de bord de l’outil de révision dans un navigateur](images/0-dashboard.png)

## <a name="review-tool-credentials"></a>Informations d'identification de l’outil de révision

Lorsque vous vous inscrivez pour [l’outil de révision](https://contentmoderator.cognitive.microsoft.com), vous êtes invité à sélectionner une région Azure pour votre compte. Cela est dû au fait que l’[outil de révision](https://contentmoderator.cognitive.microsoft.com) génère une clé d’essai gratuit pour les services Azure Content Moderator. Vous aurez besoin de cette clé pour accéder aux services à partir d’un appel REST ou du SDK client. Vous pouvez voir votre URL de point de terminaison d’API et votre clé en sélectionnant **Admin** > **Credentials**.

> [!div class="mx-imgBorder"]
> ![Informations d’identification Content Moderator](images/settings-6-credentials.png)

## <a name="next-steps"></a>Étapes suivantes

Consultez [Configurer l’outil de révision](./configure.md) pour savoir comment accéder aux ressources de l’outil de révision et modifier ses paramètres.
---
title: Analyser la conservation utilisateur de l’application web avec Azure Application Insights
description: Combien d’utilisateurs reviennent vers votre application ?
ms.topic: conceptual
author: NumberByColors
ms.author: daviste
ms.date: 05/03/2017
ms.reviewer: mbullwin
ms.openlocfilehash: f7f6155d3230d234e1163ee12cd7685817331b93
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87323534"
---
# <a name="user-retention-analysis-for-web-applications-with-application-insights"></a>Analyse de la rétention utilisateur des applications web avec Azure Application Insights

La fonction de rétention [d’Azure Application Insights](./app-insights-overview.md) vous aide à analyser le nombre d’utilisateurs qui reviennent vers votre application et la fréquence à laquelle ils exécutent des tâches particulières ou atteignent des objectifs. Par exemple, si vous exécutez un site de jeu, vous pouvez comparer le nombre d’utilisateurs qui reviennent sur le site après avoir perdu à un jeu avec le nombre d’utilisateurs qui reviennent après avoir gagné. Cette information peut vous aider à améliorer l’expérience de vos utilisateurs et votre stratégie commerciale.

## <a name="get-started"></a>Bien démarrer

Si aucune donnée n’apparaît dans l’outil de rétention du portail Application Insights, [découvrez comment prendre en main les outils d’utilisation](usage-overview.md).

## <a name="the-retention-tool"></a>Outil de rétention

![Outil de rétention](./media/usage-retention/retention.png)

1. La barre d’outils permet aux utilisateurs de créer des rapports de rétention, d’ouvrir les rapports de rétention existants, d’enregistrer le rapport de rétention en cours directement ou à un autre emplacement, d’annuler les modifications apportées aux rapports enregistrés, d’actualiser les données du rapport, de partager des rapports par e-mail ou lien direct, et d’accéder à la page de documentation. 
2. Par défaut, la rétention affiche tous les utilisateurs qui ont effectué une action, puis sont revenus et ont effectué une autre action sur une période donnée. Vous pouvez choisir une combinaison différente d’événements afin de filtrer des activités utilisateur particulières.
3. Ajoutez un ou plusieurs filtres sur les propriétés. Par exemple, vous pouvez cibler les utilisateurs d’un pays ou d’une région spécifique. Cliquez sur **Mettre à jour** après avoir défini les filtres. 
4. Le graphique de rétention globale récapitule la rétention utilisateur sur la période sélectionnée. 
5. La grille affiche le nombre d’utilisateurs conservés en fonction du générateur de requêtes au point 2. Chaque ligne représente une cohorte d’utilisateurs ayant effectué l’un des événements pendant la période de temps indiquée. Chaque cellule de la ligne indique combien de cette cohorte sont revenus au moins une fois pendant une période ultérieure. Certains utilisateurs peuvent revenir pendant plusieurs périodes. 
6. Les cartes d’aperçu affichent les cinq principaux événements de lancement et les cinq principaux événements renvoyés pour aider les utilisateurs à mieux comprendre leur rapport de rétention. 

![Passage de la souris sur la fonction de rétention](./media/usage-retention/hover.png)

Les utilisateurs peuvent survoler les cellules de l’outil de rétention pour accéder au bouton Analytics ainsi qu’à des info-bulles qui expliquent la signification de la cellule. Le bouton Analytics redirige les utilisateurs vers l’outil Analytics avec une requête prédéfinie pour générer des utilisateurs à partir de la cellule. 

## <a name="use-business-events-to-track-retention"></a>Utiliser des événements commerciaux pour suivre la rétention

Pour obtenir l’analyse de rétention la plus utile, mesurez les événements qui représentent des activités commerciales significatives. 

Par exemple, un grand nombre d’utilisateurs peuvent ouvrir une page dans votre application sans jouer au jeu qu’elle affiche. Le suivi des vues de pages uniquement fournit ainsi une estimation inexacte du nombre de personnes qui reviennent pour jouer au jeu auquel ont déjà joué. Pour obtenir une vision claire des lecteurs qui reviennent, votre application doit envoyer un événement personnalisé lorsqu’un utilisateur joue réellement.  

Il est recommandé de coder les événements personnalisés qui représentent des actions commerciales clés et de les utiliser pour votre analyse de rétention. Pour capturer le résultat du jeu, vous devez écrire une ligne de code pour envoyer un événement personnalisé à Application Insights. Si vous l’écrivez dans le code de page web ou en Node.JS, voici à quoi il ressemble :

```JavaScript
    appinsights.trackEvent("won game");
```

Ou dans le code de serveur ASP.NET :

```csharp
   telemetry.TrackEvent("won game");
```

[Familiarisez-vous avec l’écriture d’événements personnalisés](./api-custom-events-metrics.md#trackevent).


## <a name="next-steps"></a>Étapes suivantes
- Pour activer les expériences d’utilisation, commencez à envoyer des [événements personnalisés](./api-custom-events-metrics.md#trackevent) ou des [affichages de page](./api-custom-events-metrics.md#page-views).
- Si vous envoyez déjà des événements personnalisés ou des affichages de page, explorez les outils d’utilisation pour savoir comment les utilisateurs emploient votre service.
    - [Utilisateurs, sessions, événements](usage-segmentation.md)
    - [Entonnoirs](usage-funnels.md)
    - [Flux d’utilisateurs](usage-flows.md)
    - [Classeurs](../platform/workbooks-overview.md)
    - [Ajouter du contexte utilisateur](usage-send-user-context.md)


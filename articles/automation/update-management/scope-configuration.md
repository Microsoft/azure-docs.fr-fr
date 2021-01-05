---
title: Limiter l’étendue du déploiement Update Management d’Azure Automation
description: Cet article explique comment utiliser les configurations d’étendue pour limiter l’étendue d’un déploiement Update Management.
services: automation
ms.date: 03/04/2020
ms.topic: conceptual
ms.custom: mvc
ms.openlocfilehash: 76063c479950d12985d5f3f52393f9bb0d5ecd8d
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92221559"
---
# <a name="limit-update-management-deployment-scope"></a>Limiter l’étendue du déploiement Update Management

Cet article explique comment vous pouvez utiliser les configurations d’étendue lors de l’utilisation de la fonctionnalité [Update Management](overview.md) pour déployer des mises à jour et des correctifs sur vos machines virtuelles. Pour plus d’informations, consultez [Ciblage des solutions de supervision dans Azure Monitor (préversion)](../../azure-monitor/insights/solution-targeting.md).

## <a name="about-scope-configurations"></a>À propos des configurations d’étendues

La configuration d’étendue est un groupe d’une ou plusieurs recherches enregistrées (requêtes) utilisé pour limiter l’étendue d’Update Management à des ordinateurs spécifiques. La configuration d’étendue est utilisée au sein de l’espace de travail Log Analytics pour cibler les ordinateurs qui activent la fonctionnalité. Lorsque vous ajoutez un ordinateur pour recevoir des mises à jour de Update Management, l’ordinateur est également ajouté à une recherche enregistrée dans l’espace de travail.

## <a name="set-the-scope-limit"></a>Définir la limite de l’étendue

Pour limiter l’étendue de votre déploiement Update Management :

1. Dans votre compte Automation, sélectionnez **Espace de travail lié** sous **Ressources connexes**.

2. Sélectionnez **Accéder à l’espace de travail**.

3. Sélectionnez **Configurations d’étendue (préversion)** , sous **Sources de données de l’espace de travail**.

4. Sélectionnez les points de suspension sur la droite de la configuration d’étendue `MicrosoftDefaultScopeConfig-Updates`, puis sélectionnez **Modifier**.

5. Dans le volet de modification, développez **Sélectionner des groupes d’ordinateurs**. Le volet Groupes d’ordinateurs affiche les recherches enregistrées et utilisées pour créer la configuration d’étendue. La recherche enregistrée utilisée par Update Management est :

    |Nom     |Category  |Alias  |
    |---------|---------|---------|
    |MicrosoftDefaultComputerGroup     | Mises à jour        | Updates__MicrosoftDefaultComputerGroup         |

6. Sélectionnez la recherche enregistrée pour afficher et modifier la requête utilisée pour remplir le groupe. L’image suivante montre la requête et ses résultats :

    [ ![Recherches enregistrées](./media/scope-configuration/logsearch.png)](./media/scope-configuration/logsearch-expanded.png#lightbox)

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez [interroger des journaux d’Azure Monitor](query-logs.md) pour analyser les évaluations de mise à jour, les déploiements et autres tâches de gestion associées. Vous trouverez des requêtes prédéfinies pour vous aider à commencer.

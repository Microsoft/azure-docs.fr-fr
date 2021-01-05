---
title: Comment configurer la rétention des données dans votre environnement - Azure Time Series Insights | Microsoft Docs
description: Découvrez comment configurer la rétention des données dans votre environnement Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.workload: big-data
ms.topic: conceptual
ms.date: 09/29/2020
ms.custom: seodec18
ms.openlocfilehash: 468b4f7ca7b0af4abc32df5d9ef64a74154d3de1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91569402"
---
# <a name="configuring-retention-in-azure-time-series-insights-gen1"></a>Configuration de la conservation des données dans Azure Time Series Insights Gen1

> [!CAUTION]
> Il s’agit d’un article Gen1.

Cet article explique comment configurer la **durée de conservation des données** et le **comportement de limite de stockage dépassée** dans Azure Time Series Insights.

## <a name="summary"></a>Résumé

Chaque environnement Azure Time Series Insights dispose d’un paramètre permettant de configurer la **durée de conservation des données**. La valeur est comprise entre 1 et 400 jours. Les données sont supprimées en fonction de la capacité de stockage ou de la durée de conservation de l’environnement (1-400), selon ce qui se présente en premier.

Chaque environnement Time Series Insights dispose d’un paramètre supplémentaire **Comportement de limite de stockage dépassée**. Ce paramètre contrôle le comportement d’entrée et de vidage lorsque la capacité maximale d’un environnement est atteinte. Vous pouvez choisir entre deux comportements :

- **Vidage des données anciennes** (par défaut)
- **Suspendre l’entrée**

Pour plus d’informations sur ces paramètres, afin de mieux les comprendre voir [Présentation de la conservation des données dans Time Series Insights](time-series-insights-concepts-retention.md).  

## <a name="configure-data-retention"></a>Configurer la rétention de données

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Recherchez votre environnement Azure Time Series Insights existant. Sélectionnez **Toutes les ressources** dans le menu de gauche du portail Azure. Sélectionnez votre environnement Azure Time Series Insights.

1. Sous le titre **paramètres**, sélectionnez **Configuration du stockage**.

    [![Sous Paramètres, sélectionnez Configuration du stockage](media/data-retention/configure-data-retention.png)](media/data-retention/configure-data-retention.png#lightbox)

1. Sélectionnez la **durée de conservation des données (en jours)** pour configurer la conservation à l’aide du curseur, ou saisissez un nombre dans la zone de texte.

1. Notez le paramètre **Capacité**, car il affecte la quantité maximale d’événements de données et la capacité de stockage totale pour le stockage des données.

1. Activez/désactivez le paramètre **Comportement de limite de stockage dépassée**. Sélectionnez le comportement **Vidage des données anciennes** ou **Suspendre l’entrée**.

    [![Suspendre l’entrée : accepter et enregistrer.](media/data-retention/pause-ingress-accept-and-save.png)](media/data-retention/pause-ingress-accept-and-save.png#lightbox)

1. Consultez la documentation pour comprendre les risques potentiels en matière de perte de données. Sélectionnez **Enregistrer** pour configurer les modifications.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations, voir [Présentation de la conservation des données dans Azure Time Series Insights](time-series-insights-concepts-retention.md).

- Apprenez [comment mettre à l’échelle votre environnement Azure Time Series Insights](time-series-insights-how-to-scale-your-environment.md).

- Découvrez la [planification de votre environnement](time-series-insights-environment-planning.md).

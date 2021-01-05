---
title: Étendue de requête de journal dans la fonctionnalité Log Analytics d’Azure Monitor
description: Décrit l’étendue et l’intervalle de temps pour une requête de journal dans la fonctionnalité Log Analytics d’Azure Monitor.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 09/09/2020
ms.openlocfilehash: ddb73de87789934aa66893bdbe6519011cf324b3
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96186097"
---
# <a name="log-query-scope-and-time-range-in-azure-monitor-log-analytics"></a>Étendue de requête de journal et intervalle de temps dans la fonctionnalité Log Analytics d’Azure Monitor
Lorsque vous exécutez un [requête de journal](log-query-overview.md) à l’aide de la fonctionnalité [Log Analytics sur le portail Azure](./log-analytics-tutorial.md), le jeu de données évalué par la requête dépend de l’étendue et de l’intervalle de temps que vous sélectionnez. Cet article décrit l’étendue et l’intervalle de temps, ainsi que la manière de les définir selon vos besoins. Il décrit également le comportement des différents types d’étendues.


## <a name="query-scope"></a>Étendue de requête
L’étendue de requête définit les enregistrements évalués par la requête. Elle inclut généralement tous les enregistrements dans un espace de travail Log Analytics ou une application Application Insights. Log Analytics vous permet également de définir une étendue pour une ressource Azure surveillée particulière. Un propriétaire de ressources peut ainsi se concentrer uniquement sur ses données, même si cette ressource écrit dans plusieurs espaces de travail.

L’étendue est toujours affichée en haut à gauche de la fenêtre de Log Analytics. Une icône indique si l’étendue est un espace de travail Log Analytics ou une application Application Insights. Aucune icône n’indique une autre ressource Azure.

![Étendue affichée dans le portail](media/scope/scope.png)

L’étendue est déterminée par la méthode que vous utilisez pour démarrer Log Analytics et, dans certains cas, vous pouvez la modifier en cliquant dessus. Le tableau suivant répertorie les différents types d’étendues utilisés et différents détails pour chacun d’eux.

> [!IMPORTANT]
> Si vous utilisez une application basée sur un espace de travail dans Application Insights, ses données sont stockées dans un espace de travail Log Analytics avec toutes les autres données de journal. Pour la compatibilité descendante, vous bénéficierez de l’expérience classique d’Application Insights en sélectionnant l’application comme étendue. Pour afficher ces données dans l’espace de travail Log Analytics, définissez l’étendue sur l’espace de travail.

| Étendue de requête | Enregistrements dans l’étendue | Comment sélectionner | Modification de l’étendue |
|:---|:---|:---|:---|
| Espace de travail Log Analytics | Tous les enregistrements dans l’espace de travail Log Analytics. | Sélectionnez **Journaux** dans le menu **Azure Monitor** ou le menu **Espaces de travail Log Analytics**.  | Peut modifier l’étendue en la définissant sur tout autre type de ressource. |
| Application Application Insights | Tous les enregistrements dans l’application Application Insights. | Sélectionnez **Journaux** dans le menu **Application Insights** de l’application. | Peut modifier uniquement l’étendue en la définissant sur toute autre application Application Insights. |
| Resource group | Enregistrements créés par toutes les ressources du groupe de ressources. Peut inclure des données de plusieurs espaces de travail Log Analytics. | Sélectionnez **Journaux** dans le menu du groupe de ressources. | Ne peut pas modifier l’étendue.|
| Abonnement | Enregistrements créés par toutes les ressources dans l’abonnement. Peut inclure des données de plusieurs espaces de travail Log Analytics. | Sélectionnez **Journaux** dans le menu d’abonnement.   | Ne peut pas modifier l’étendue. |
| Autres ressources Azure | Enregistrements créés par la ressource. Peut inclure des données de plusieurs espaces de travail Log Analytics.  | Sélectionnez **journaux** dans le menu de ressources.<br>OR<br>Sélectionnez **journaux** dans le menu **Azure Monitor** menu, puis sélectionnez une nouvelle étendue. | Ne peut modifier l’étendue qu’avec le même type de ressource. |

### <a name="limitations-when-scoped-to-a-resource"></a>Limitations en cas d’étendue à une ressource

Lorsque l’étendue de requête est un espace de travail Log Analytics ou une application Application Insights, toutes les options du portail et toutes les commandes de requête sont disponibles. Cependant, en cas d’étendue à une ressource, les options suivantes dans le portail ne sont pas disponibles, car elles sont associées à un espace de travail ou à une application uniques :

- Enregistrer
- Explorateur de requêtes
- Nouvelle règle d’alerte

Vous ne pouvez pas utiliser les commandes suivantes dans une requête étendue à une ressource, car l’étendue de la requête inclut déjà les espaces de travail contenant des données pour cette ressource ou cet ensemble de ressources :

- [app](app-expression.md)
- [workspace](workspace-expression.md)
 

## <a name="query-scope-limits"></a>Limites d’étendue de requête
La définition de l’étendue sur une ressource ou un ensemble de ressources est une fonctionnalité particulièrement puissante de Log Analytics dans la mesure où elle vous permet de consolider automatiquement les données distribuées dans une seule requête. Toutefois, cela peut affecter considérablement les performances si les données doivent être récupérées à partir d’espaces de travail dans plusieurs régions Azure.

Log Analytics vous aide à vous protéger contre une surcharge excessive des requêtes qui s’étendent sur des espaces de travail dans plusieurs régions en émettant un avertissement ou une erreur quand un certain nombre de régions sont utilisées. Votre requête reçoit un avertissement si l’étendue comprend des espaces de travail dans 5 régions ou plus. L’exécution se déroule néanmoins, mais peut prendre un temps excessif.

![Avertissement de requête](media/scope/query-warning.png)

L’exécution de votre requête sera bloquée si l’étendue comprend des espaces de travail dans 20 régions ou plus. Dans ce cas, vous êtes invité à réduire le nombre de régions de l’espace de travail et à essayer de réexécuter la requête. La liste déroulante affiche toutes les régions dans l’étendue de la requête, et vous devez réduire le nombre de régions avant de tenter de réexécuter la requête.

![Échec de la requête](media/scope/query-failed.png)


## <a name="time-range"></a>Plage temporelle
L’intervalle de temps spécifie le jeu d’enregistrements évalués pour la requête en fonction du moment de création de l’enregistrement. Cela est défini par la colonne **TimeGenerated** sur chaque enregistrement dans l’espace de travail ou l’application, comme spécifié dans le tableau suivant. Dans le cas d’une application Application Insights classique, la colonne **timestamp** est utilisée pour l’intervalle de temps.


Définissez l’intervalle de temps en le sélectionnant dans le sélecteur de temps en haut de la fenêtre de Log Analytics.  Vous pouvez sélectionner une période prédéfinie ou choisir **Personnaliser** pour spécifier un intervalle de temps spécifique.

![Sélecteur d’heure](media/scope/time-picker.png)

Si vous définissez un filtre dans la requête qui utilise la colonne d’heure standard comme indiqué dans le tableau ci-dessus, le sélecteur de temps est désactivé et remplacé par l’option **Définir dans la requête**. Dans ce cas, l’approche la plus efficace consiste à placer le filtre en haut de la requête afin que tout traitement ultérieur doive uniquement porter sur les enregistrements filtrés.

![Requête filtrée](media/scope/query-filtered.png)

Si vous utilisez la commande [workspace](workspace-expression.md) ou [app](app-expression.md) pour récupérer des données d’un autre espace de travail ou d’une autre application classique, le sélecteur d’heure peut se comporter différemment. Si l’étendue est un espace de travail Log Analytics et que vous utilisez la commande **app**, ou si l’étendue est une application Application Insights classique et que vous utilisez la commande **workspace**, il se peut que Log Analytics ne comprenne pas que la colonne utilisée dans le filtre doit déterminer le filtre de temps.

Dans l’exemple suivant, l’étendue est définie sur un espace de travail Log Analytics.  La requête utilise la commande **workspace** pour récupérer des données d’un autre espace de travail Log Analytics. Le sélecteur d’heure est remplacé par l’option **Définir dans la requête** parce qu’il voit un filtre qui utilise la colonne attendue **TimeGenerated**.

![Interroger avec la commande workspace](media/scope/query-workspace.png)

Si la requête utilise la commande **app** pour récupérer des données d’une application Application Insights classique, Log Analytics ne reconnaît pas la colonne **timestamp** dans le filtre et le sélecteur d’heure reste inchangé. Dans ce cas, les deux filtres sont appliqués. Dans l’exemple, seuls les enregistrements créés au cours des dernières 24 heures sont inclus dans la requête, même si celle-ci spécifie 7 jours dans la clause **where**.

![Query avec la commande app](media/scope/query-app.png)

## <a name="next-steps"></a>Étapes suivantes

- Suivez un [didacticiel sur l’utilisation de Log Analytics dans le portail Azure](./log-analytics-tutorial.md).
- Suive un [didacticiel sur l’écriture de requêtes](get-started-queries.md).
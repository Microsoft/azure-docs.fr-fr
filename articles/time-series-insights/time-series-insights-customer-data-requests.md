---
title: Fonctionnalités de requête de données client dans Azure Time Series Insights | Microsoft Docs
description: Apprenez en plus sur les fonctionnalités de requêtes de données client dans Azure Time Series Insights.
author: deepakpalled
ms.author: dpalled
manager: cshankar
ms.topic: conceptual
ms.service: time-series-insights
services: time-series-insights
ms.date: 10/02/2020
ms.custom: seodec18
ms.openlocfilehash: b5ecbabe4a5c3458c77d145fbce1281550470e00
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "96019070"
---
# <a name="summary-of-customer-data-request-features"></a>Résumé des fonctionnalités de demande de données client

Azure Time Series Insights est un service cloud géré avec des composants de stockage, d’analyse et de visualisation qui le rendent simple pour recevoir, stocker, explorer et analyser simultanément des milliards d’événements.

[!INCLUDE [gdpr-intro-sentence](../../includes/gdpr-intro-sentence.md)]

Pour afficher, exporter et supprimer des données personnelles pouvant être soumises à une requête d’objet de données, un administrateur de locataire Azure Time Series Insights peut utiliser le portail Azure ou les API REST. Le recours au portail Azure pour traiter les requêtes d’objet de données offre une méthode plus simple pour effectuer les opérations préférées par la plupart des utilisateurs.

## <a name="identifying-customer-data"></a>Identification des données client

Azure Time Series Insights considère les données personnelles comme des données associées aux administrateurs et utilisateurs de Time Series Insights. Time Series Insights stocke l’ID d’objet Azure Active Directory d’utilisateurs ayant accès à l’environnement. Le portail Azure affiche les adresses e-mail utilisateurs, mais ces adresses e-mail ne sont pas stockées dans Time Series Insights. Elles sont recherchées dynamiquement à l’aide de l’ID d’objet Azure Active Directory dans Azure Active Directory.

## <a name="deleting-customer-data"></a>Suppression des données client

Un administrateur de locataire peut supprimer des données client à l’aide du portail Azure.

[!INCLUDE [gdpr-dsr-and-stp-note](../../includes/gdpr-dsr-and-stp-note.md)]

Toutefois, avant de supprimer des données client via le portail, vous devez supprimer les stratégies d’accès de l’utilisateur de l’environnement Time Series Insights dans le portail Azure. Pour plus d’informations, consultez [Accorder l’accès aux données à un environnement Time Series Insights en utilisant le portail Azure](./concepts-access-policies.md)

Vous pouvez également effectuer des opérations de suppression sur des stratégies d’accès à l’aide de l’API REST. Pour plus d’informations, consultez [Stratégies d’accès - Supprimer](/rest/api/time-series-insights/management(gen1/gen2)/accesspolicies/delete).

Time Series Insights est intégré au panneau Stratégie dans le portail Azure. Time Series Insights et le panneau Stratégie vous permettent d’afficher, d’exporter et de supprimer des données utilisateur stockées dans le service. Toute action de suppression effectuée dans le panneau Stratégie du portail Azure entraîne la suppression des données utilisateur dans Time Series Insights. Par exemple, si un utilisateur dispose d’une requête personnelle enregistrée, cette requête est définitivement supprimée de l’Explorateur Time Series Insights. Si l’utilisateur dispose d’une requête partagée enregistrée, la requête est conservée, mais les informations utilisateur sont définitivement supprimées. La remarque suivante contient des instructions sur la façon d’accomplir ces tâches.

## <a name="exporting-customer-data"></a>Exportation des données client

Comme la suppression de données, un administrateur de locataire peut afficher et exporter des données stockées dans Time Series Insights à partir du panneau Stratégie dans le portail Azure.

[!INCLUDE [gdpr-dsr-and-stp-note](../../includes/gdpr-dsr-and-stp-note.md)]

Si vous êtes un administrateur de locataire, vous pouvez afficher les stratégies d’accès aux données dans l’environnement Time Series Insights dans le portail Azure. Pour plus d’informations, consultez [Accorder l’accès aux données à un environnement Time Series Insights en utilisant le portail Azure](./concepts-access-policies.md)

Il est également possible d’effectuer des opérations d’exportation sur des stratégies d’accès à l’aide de l’opération « répertorier par environnement » dans l’API REST fournie. Pour plus d’informations, consultez [Stratégies d’accès - Lister par environnement](/rest/api/time-series-insights/management(gen1/gen2)/accesspolicies/listbyenvironment).

## <a name="to-delete-data-stored-within-time-series-insights"></a>Pour supprimer des données stockées dans Time Series Insights

Les données personnelles peuvent suivre cette méthode dans le stockage Time Series Insights, un scénario différent des données utilisateur et administrateur. Si vous considérez les données stockées dans Time Series Insights comme des données personnelles, vous pouvez exporter et supprimer ces données en procédant comme suit :

### <a name="view-and-export-data"></a>Afficher et exporter des données

Pour afficher et exporter des données stockées dans Time Series Insights, vous devez rechercher ces données. Vous pouvez utiliser l’Explorateur Time Series Insights ou des API de requête Time Series Insights pour afficher et exporter des données. Pour afficher et exporter des données à l’aide de l’Explorateur Time Series Insights, recherchez tout d’abord les données utilisateur en question. Après la recherche, cliquez avec le bouton droit sur le graphique, puis sélectionnez **Explorer les événements**. La grille d’événements s’affiche et présente des options pour exporter les données au forme CSV et JSON.

Pour plus d’informations, consultez [Explorateur Azure Time Series Insights](time-series-insights-explorer.md).

### <a name="delete-data"></a>Suppression de données

Time Series Insights ne prend actuellement pas en charge la suppression granulaire de données. Toutefois, Time Series Insights offre la possibilité de supprimer des données client stockées dans Time Series Insights en configurant des stratégies de rétention. Vous pouvez définir la période de rétention de l’ensemble de l’environnement Time Series Insights sur un nombre quelconque de jours pour prendre en charge vos besoins de suppression.

Pour plus d’informations, consultez [Configuration de la conservation des données dans Time Series Insights](time-series-insights-how-to-configure-retention.md).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur [l’octroi d'un accès aux données de votre environnement Azure Time Series Insights](./concepts-access-policies.md).

* Affichez l’[explorateur Azure Time Series Insights](time-series-insights-explorer.md).

* En savoir plus sur la [Configuration de la conservation des données dans Time Series Insights](time-series-insights-how-to-configure-retention.md).
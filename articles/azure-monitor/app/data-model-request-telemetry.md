---
title: Modèle de données pour la télémétrie des requêtes – Azure Application Insights
description: Modèle de données Application Insights pour la télémétrie des requêtes
ms.topic: conceptual
ms.date: 01/07/2019
ms.reviewer: sergkanz
ms.openlocfilehash: 7a352f4ce3528d395599a91b53031c74b0873152
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87320559"
---
# <a name="request-telemetry-application-insights-data-model"></a>Télémétrie des requêtes : modèle de données Application Insights

Un élément de télémétrie de demande (dans [Application Insights](./app-insights-overview.md)) représente la séquence logique d’exécution déclenchée par une demande externe à votre application. Chaque exécution de requête est identifiée par un `ID` et une `url` uniques contenant tous les paramètres d’exécution. Vous pouvez regrouper des requêtes par `name` logique et définir la `source` de cette requête. L’exécution du code peut donner `success` ou `duration` et a une certain durée (`fail`). Les échecs et les réussites d’exécution peuvent être regroupés par `resultCode`. L’heure de début de la télémétrie des requêtes est définie sur le niveau enveloppe.

La télémétrie des requêtes prend en charge le modèle d’extensibilité standard en utilisant des propriétés (`properties`) et des mesures (`measurements`) personnalisées.

## <a name="name"></a>Name

Le nom de la requête représente le chemin de code utilisé pour traiter la requête. La valeur de faible cardinalité permet de mieux regrouper les requêtes. Pour les requêtes HTTP, elle représente la méthode HTTP et le modèle de chemin d’URL comme `GET /values/{id}` sans la valeur `id` réelle.

Le Kit de développement logiciel (SDK) Web d’Application Insights envoie le nom de la requête « en l’état », c’est-à-dire en respectant la casse. Le regroupement sur l’interface utilisateur est sensible à la casse, si bien que `GET /Home/Index` est comptabilisé séparément de `GET /home/INDEX`, même s’ils entraînent souvent la même exécution de contrôleur et d’action. Cela est dû au fait que les URL sont généralement [sensibles à la casse](https://www.w3.org/TR/WD-html40-970708/htmlweb.html). Vous souhaiterez peut-être savoir si toutes les erreurs `404` ont été générées pour des URL tapées en majuscules. Vous pouvez en apprendre plus sur la collecte de noms de requêtes par le SDK Web ASP.NET dans ce [billet de blog](https://apmtips.com/posts/2015-02-23-request-name-and-url/).

Longueur maximale : 1024 caractères

## <a name="id"></a>id

Identificateur d’une instance d’appel de requête. Utilisé pour la corrélation entre la requête et d’autres éléments de télémétrie. L’ID doit être globalement unique. Pour plus d’informations, consultez la page relative à la [corrélation](./correlation.md).

Longueur maximale : 128 caractères

## <a name="url"></a>Url

URL de requête avec tous les paramètres de chaîne de requête.

Longueur maximale : 2048 caractères

## <a name="source"></a>Source

Source de la requête. Il s’agit par exemple de la clé d’instrumentation de l’appelant ou de l’adresse IP de l’appelant. Pour plus d’informations, consultez la page relative à la [corrélation](./correlation.md).

Longueur maximale : 1024 caractères

## <a name="duration"></a>Duration

Durée de la requête au format : `DD.HH:MM:SS.MMMMMM`. Doit être positive et inférieure à `1000` jours. Ce champ est obligatoire car la télémétrie des requêtes représente l’opération avec un début et une fin.

## <a name="response-code"></a>Response code

Résultat de l’exécution d’une requête. Code d’état HTTP des requêtes HTTP. Cela peut-être une valeur `HRESULT` ou un type d’exception pour les autres types de requêtes.

Longueur maximale : 1024 caractères

## <a name="success"></a>Succès

Indication de la réussite ou non d’un appel. Ce champ est obligatoire. Lorsqu’il n’est pas défini explicitement sur `false`, une requête est considérée comme ayant réussi. Définissez cette valeur sur `false` si l’opération a été interrompue par une exception ou a retourné un code de résultat d’erreur.

Pour les applications Web, Application Insights définit les requêtes comme ayant réussi lorsque le code de réponse est inférieur à `400` ou égal à `401`. Toutefois, il arrive que ce mappage par défaut ne corresponde pas à la sémantique de l’application. Le code de réponse `404` peut indiquer « aucun enregistrement », qui peut faire partie d’un flux régulier. Il peut également indiquer un lien rompu. Pour les liens rompus, vous pouvez même implémenter une logique plus avancée. Vous ne pouvez marquer les liens rompus en tant qu’échecs que lorsque ces liens sont situés sur le même site en analysant le point d’accès de l’URL. Ou bien vous pouvez les marquer en tant qu’échecs lors de l’accès à partir de l’application mobile de l’entreprise. De même `301` et `302` indiquent un échec lors de l’accès à partir du client qui ne prend pas en charge la redirection.

Un contenu partiellement accepté `206` peut indiquer l’échec d’une requête globale. Par exemple, le point de terminaison d’Application Insights reçoit un lot d’éléments de télémétrie sous la forme d’une seule requête. Il retourne `206` lorsque certains éléments du lot n'ont pas été traités avec succès. La fréquence croissante de `206` indique un problème qui doit être examiné. Une logique similaire s’applique au code multi-état `207` où la réussite peut être le pire des codes de réponse distincts.

Vous pouvez en apprendre plus sur le code de résultat des requête et le code d’état dans ce [billet de blog](https://apmtips.com/posts/2016-12-03-request-success-and-response-code/).

## <a name="custom-properties"></a>Propriétés personnalisées

[!INCLUDE [application-insights-data-model-properties](../../../includes/application-insights-data-model-properties.md)]

## <a name="custom-measurements"></a>Mesures personnalisées

[!INCLUDE [application-insights-data-model-measurements](../../../includes/application-insights-data-model-measurements.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Écrire une télémétrie de demande personnalisée](./api-custom-events-metrics.md#trackrequest)
- Pour connaître les types et les modèles de données Application Insights, consultez [Modèle de données](data-model.md).
- Découvrez comment [configurer l’application ASP.NET Core](./asp-net.md) avec Application Insights.
- Découvrez quelles [plateformes](./platforms.md) sont prises en charge par Application Insights.


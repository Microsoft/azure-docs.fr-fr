---
title: Structures des données du service Mobility (préversion) dans Microsoft Azure Maps
description: Découvrez comment les données sont organisées en zones urbaines dans le service Mobility (préversion) Azure Maps. Consultez les champs qui stockent des informations sur les arrêts et les lignes de transport en commun.
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 8ffedc18ba331733723a6293756b60b733cc32cf
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96904718"
---
# <a name="data-structures-in-azure-maps-mobility-services-preview"></a>Structure de données dans le service Mobility Azure Maps 

> [!IMPORTANT]
> Le service Mobility d’Azure Maps est disponible en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).



Cet article présente le concept de zone urbaine dans le [service Mobility Azure Maps](/rest/api/maps/mobility). Nous aborderons certains des champs communs qui sont renvoyés lorsque ce service est interrogé pour les arrêts et les lignes de transport en commun. Nous vous recommandons de lire cet article avant de commencer à développer à l’aide des API du service Mobility.

## <a name="metro-area"></a>Zone urbaine

Les données du service Mobility (préversion) sont regroupées selon les zones urbaines prises en charge. Les zones urbaines ne suivent pas les limites des villes. Une zone urbaine peut contenir plusieurs villes : une ville densément peuplée et les villes environnantes. En fait, un pays ou une région peut être une zone urbaine. 

Le paramètre `metroID` est l’ID d’une zone urbaine qui peut être utilisé pour appeler l’[API Get Metro Area Info](/rest/api/maps/mobility/getmetroareainfopreview). Utilisez l’API « Get Metro » d’Azure Maps pour demander des types de transport en commun, des agences de transport, des alertes actives et des détails supplémentaires pour la zone urbaine choisie. Vous pouvez également demander les zones urbaines prises en charge et les metroID. Les ID de zones urbaines sont susceptibles d’être modifiés.

**metroID :** 522   **Nom :** Seattle-Tacoma-Bellevue

![Seattle-metro-area](./media/mobility-service-data-structure/seattle-metro.png)

## <a name="stop-ids"></a>ID d’arrêts

Les arrêts de transport peuvent avoir deux types d’ID, l’ID [General Transit Feed Specification (GFTS)](http://gtfs.org/) et l’ID d’arrêt Azure Maps. L’ID GFTS est appelé stopKey, et l’ID d’arrêt Azure Maps stopID. Lorsque vous faites souvent référence à des arrêts de transport, vous êtes encouragé à utiliser l’ID d’arrêt Azure Maps. stopID est plus stable et susceptible de rester le même tant que l’arrêt physique existe. L’ID d’arrêt GTFS est mis à jour plus souvent. Par exemple, l’ID d’arrêt GTFS peut être mis à jour en fonction de la requête du fournisseur GTFS ou lors de la publication d’une nouvelle version de GTFS. Bien que l’arrêt physique n’ait pas changé, l’ID d’arrêt GTFS peut changer.

Pour commencer, vous pouvez demander les arrêts de transport proches, en utilisant l’[API Get Nearby Transit](/rest/api/maps/mobility/getnearbytransitpreview).

## <a name="line-groups-and-lines"></a>Lignes et groupes de lignes

Le service Mobility (préversion) utilise un modèle de données parallèle pour les lignes et les groupes de lignes. Ce modèle est utilisé pour mieux traiter les modifications héritées des itinéraires [GTFS](http://gtfs.org/) et des données sur les voyages.


### <a name="line-groups"></a>Groupes de lignes

Un groupe de lignes est une entité regroupant toutes les lignes qui font logiquement partie du même groupe. En général, un groupe comprendra deux lignes, le trajet du point A au point B, et le trajet retour du point B au point A. Les deux lignes appartiennent à la même société de transports publics et ont le même numéro de ligne. Toutefois, il se peut qu’un groupe de lignes comprenne plus de deux lignes ou seulement une ligne.


### <a name="lines"></a>Lignes

Comme indiqué ci-dessus, chaque groupe de lignes comprend un ensemble de lignes. Chaque groupe de lignes est composé de deux lignes, et chaque ligne décrit une direction.  Toutefois, il existe des cas où un groupe de lignes est composé d’un plus grand nombre de lignes. Par exemple, une ligne fait parfois des détours dans un quartier donné et parfois non. Dans les deux cas, elle fonctionne sous le même numéro de ligne. En outre, un groupe de lignes peut être composé d’une seule ligne. Une ligne circulaire avec une seule direction est un groupe de lignes à une seule ligne.

Pour commencer, vous pouvez demander les groupes de lignes en utilisant l’[API Get Transit Line](/rest/api/maps/mobility/gettransitlineinfopreview).


## <a name="next-steps"></a>Étapes suivantes

Découvrez comment demander des données de transport public à l’aide du service Mobility (préversion) :

> [!div class="nextstepaction"]
> [Comment demander des données de transit](how-to-request-transit-data.md)

Découvrez comment demander des données en temps réel à l’aide du service Mobility (préversion) :

> [!div class="nextstepaction"]
> [Comment demander des données en temps réel](how-to-request-real-time-data.md)

Explorez la documentation relative à l’API du service Mobility Azure Maps (préversion) :

> [!div class="nextstepaction"]
> [Documentation sur l’API du service Mobility](/rest/api/maps/mobility)
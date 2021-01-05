---
title: Bonnes pratiques pour le service Route Azure Maps de Microsoft Azure Maps
description: Découvrez comment définir un itinéraire efficace pour les véhicules à l’aide du service Route de Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 09/02/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 743710ea0d40eb31375236d4e59b0b138a217518
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92895543"
---
# <a name="best-practices-for-azure-maps-route-service"></a>Meilleures pratiques d’utilisation du service Route Azure Maps

Les API Route Directions et Route Matrix du [service Route](/rest/api/maps/route) d’Azure Maps peuvent être utilisées pour calculer les heures d’arrivée estimées pour chaque itinéraire demandé. Les API Route tiennent compte de facteurs tels que les informations de trafic en temps réel et les données de trafic historiques, comme les vitesses routières typiques au jour ou à l’heure demandée. Les API retournent les itinéraires les plus courts ou les plus rapides disponibles vers plusieurs destinations à la fois à la suite ou dans un ordre optimisé, en fonction de l’heure ou de la distance. Les utilisateurs peuvent également demander des itinéraires spécialisés et des informations pour les piétons, les cyclistes et les véhicules commerciaux tels que les camions. Cet article vous présente les meilleures pratiques concernant l’appel du [service Route](/rest/api/maps/route) d’Azure Maps et vous montre comment effectuer les opérations suivantes :

 * Choisir entre les API Route Directions et Matrix Routing
 * Demander les temps de trajet prédits et historiques, en fonction des données de trafic historiques et en temps réel
 * Demander des détails sur l’itinéraire, comme la durée et la distance, pour l’itinéraire entier et chaque tronçon de l’itinéraire
 * Demander un itinéraire pour un véhicule commercial, comme un camion
 * Demander des informations sur le trafic sur un itinéraire, comme les embouteillages et les informations de péage
 * Demander un itinéraire constitué d’un ou de plusieurs arrêts (étapes)
 * Optimiser un itinéraire d’un ou de plusieurs arrêts pour obtenir le meilleur ordre pour visiter chaque arrêt (étape)
 * Optimiser les itinéraires alternatifs à l’aide des points de prise en charge (par exemple, proposer des itinéraires alternatifs qui passent par une station de recharge de véhicule électrique)
 * Utiliser le [service Route](/rest/api/maps/route) avec le SDK web Azure Maps

## <a name="prerequisites"></a>Prérequis

1. [Créer un compte Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
2. [Obtenir une clé d’abonnement principale](quick-demo-map-app.md#get-the-primary-key-for-your-account), également appelée clé primaire ou clé d’abonnement.

Pour plus d’informations sur la couverture du service Route, consultez [Couverture de routage](routing-coverage.md).

Cet article utilise l’[application Postman](https://www.postman.com/downloads/) pour générer des appels REST, mais vous pouvez choisir l’environnement de développement d’API de votre choix.

## <a name="choose-between-route-directions-and-matrix-routing"></a>Choisir entre Route Directions et Matrix Routing

Les API Route Directions retournent des instructions, notamment la durée du trajet et les coordonnées d’un itinéraire. L’API Route Matrix vous permet de calculer le temps de trajet et les distances pour un ensemble d’itinéraires définis par les emplacements d’origine et de destination. Pour chaque origine donnée, l’API Matrix calcule le coût (durée de trajet et distance) de l’itinéraire de cette origine vers chaque destination donnée. Toutes ces API vous permettent de spécifier des paramètres tels que l’heure de départ souhaitée, les heures d’arrivée et le type de véhicule (voiture, camion, etc.). Elles utilisent toutes des données de trafic prédictives ou en temps réel pour retourner les itinéraires les plus optimaux.

Envisagez d’appeler les API Route Directions si votre scénario consiste à :

* Demander l’itinéraire le plus court ou le plus rapide entre deux ou plusieurs lieux connus, afin d’obtenir des temps d’arrivée précis pour vos véhicules de livraison.
* Demander des conseils détaillés sur l’itinéraire, notamment la géométrie de l’itinéraire, pour visualiser les itinéraires sur la carte.
* Selon la liste des emplacements clients, calculer l’itinéraire le plus court possible pour accéder à chaque emplacement client et revenir à l’origine. Ce scénario est communément appelé le « problème du voyageur de commerce ». Vous pouvez transmettre jusqu’à 150 étapes (arrêts) dans une demande.
* Envoyer des lots de requêtes à l’API Route Directions Batches à l’aide d’un simple appel d’API.

Envisagez d’appeler l’API Matrix Routing si votre scénario consiste à :

* Calculer la durée du trajet ou la distance entre un ensemble d’origines et de destinations. Par exemple, vous avez 12 chauffeurs et vous devez trouver le chauffeur disponible le plus proche pour récupérer la livraison de nourriture auprès du restaurant.
* Trier les itinéraires potentiels en fonction de leur distance ou de leur durée de voyage réelle. L’API Matrix retourne uniquement les durées de trajet et les distances pour chaque combinaison d’origine et de destination.
* Regrouper les données en fonction du temps de trajet ou des distances. Par exemple, si votre entreprise compte 50 employés, rechercher tous les employés qui habitent à moins de 20 minutes en voiture de votre bureau.

Voici une comparaison pour illustrer certaines fonctionnalités des API Route Directions et Matrix :

| API Azure Maps | Nombre maximal de requêtes dans la demande | Zones à éviter | Acheminement des camions et véhicules électriques | Étapes et optimisation « Voyageur de commerce » | Points de prise en charge |
| :--------------: |  :--------------: |  :--------------: | :--------------: | :--------------: | :--------------: |
| Get Route Directions (Obtenir des directions) | 1 | | X | X | |
| Post Route Directions (Publier des directions) | 1 | X | X | X | X |
| Post Route Directions Batch (Publier des itinéraires par lots) | 700 | | X | X | |
| Post Route Matrix (Publier une matrice d’itinéraire) | 700 | | X | | |

Pour en savoir plus sur les capacités d’acheminement des véhicules électriques, consultez notre tutoriel sur la façon de [router des véhicules électriques à l’aide d’Azure Notebooks avec Python](tutorial-ev-routing.md).

## <a name="request-historic-and-real-time-data"></a>Demander des données historiques et en temps réel

Par défaut, le service Route part du principe que le mode de déplacement est un véhicule et que le départ est immédiat. Il retourne un itinéraire basé sur les conditions de circulation en temps réel, à moins qu’une demande de calcul d’itinéraire ne spécifie autre chose. Des restrictions de trafic en fonction d’un horaire fixe, par exemple « Les virages à gauche ne sont pas autorisés entre 16h et 18h », sont capturées et sont prises en compte par le moteur de recherche d’itinéraire. Les fermetures de route, par exemple en raison de travaux, sont prises en compte, sauf si vous demandez spécifiquement un itinéraire qui ignore le trafic en direct actuel. Pour ignorer le trafic actuel, définissez `traffic` sur `false` dans votre demande d’API.

La valeur **travelTimeInSeconds** du calcul d’itinéraire inclut le retard dû au trafic. Elle est générée en tirant parti des données actuelles et historiques concernant le temps de trajet, lorsque le départ est immédiat. Si vous partez à une heure ultérieure, les API retournent des temps de trajet prédits en fonction des données historiques.

Si vous incluez le paramètre **computeTravelTimeFor=All** dans votre demande, l’élément Summary de la réponse a les champs supplémentaires suivants, notamment les conditions de trafic historiques :

| Élément | Description|
| :--- | :--- |
| noTrafficTravelTimeInSeconds | Temps de trajet estimé calculé comme s’il n’y a aucun retard sur l’itinéraire dû aux conditions de circulation, par exemple en raison d’embouteillages |
| historicTrafficTravelTimeInSeconds | Temps de trajet estimé calculé à l’aide des données de trafic historiques en fonction d’un horaire donné |
| liveTrafficIncidentsTravelTimeInSeconds | Temps de trajet estimé calculé à l’aide de données de vitesse en temps réel |

Les sections suivantes montrent comment effectuer des appels aux API d’itinéraire à l’aide des paramètres décrits.

### <a name="sample-query"></a>Exemple de requête

Dans le premier exemple ci-dessous, l’heure de départ est définie sur le futur au moment de l’écriture.

```http
https://atlas.microsoft.com/route/directions/json?subscription-key=<Your-Azure-Maps-Primary-Subscription-Key>&api-version=1.0&query=51.368752,-0.118332:51.385426,-0.128929&travelMode=car&traffic=true&departAt=2025-03-29T08:00:20&computeTravelTimeFor=all
```

La réponse contient un élément Summary, comme celui ci-dessous. Étant donné que l’heure de départ est définie sur le futur, la valeur **trafficDelayInSeconds** est égale à zéro. La valeur **travelTimeInSeconds** est calculée à l’aide des données de trafic historiques en fonction d’un horaire donné. Ainsi, dans ce cas, la valeur **travelTimeInSeconds** est égale à la valeur **historicTrafficTravelTimeInSeconds** .

```json
"summary": {
    "lengthInMeters": 2131,
    "travelTimeInSeconds": 248,
    "trafficDelayInSeconds": 0,
    "departureTime": "2025-03-29T08:00:20Z",
    "arrivalTime": "2025-03-29T08:04:28Z",
    "noTrafficTravelTimeInSeconds": 225,
    "historicTrafficTravelTimeInSeconds": 248,
    "liveTrafficIncidentsTravelTimeInSeconds": 248
},
```

### <a name="sample-query"></a>Exemple de requête

Dans le deuxième exemple ci-dessous, nous disposons d’une demande d’itinéraire en temps réel, où le départ est immédiat. Ce n’est pas explicitement spécifié dans l’URL, car il s’agit de la valeur par défaut.

```http
https://atlas.microsoft.com/route/directions/json?subscription-key=<Your-Azure-Maps-Primary-Subscription-Key>&api-version=1.0&query=47.6422356,-122.1389797:47.6641142,-122.3011268&travelMode=car&traffic=true&computeTravelTimeFor=all
```

La réponse contient un résumé comme indiqué ci-dessous. En raison des embouteillages, la valeur **trafficDelaysInSeconds** est supérieure à zéro. Elle est également supérieure à **historicTrafficTravelTimeInSeconds** .

```json
"summary": {
    "lengthInMeters": 16637, 
    "travelTimeInSeconds": 2905, 
    "trafficDelayInSeconds": 1604, 
    "departureTime": "2020-02-28T01:00:20+00:00",
    "arrivalTime": "2020-02-28T01:48:45+00:00", 
    "noTrafficTravelTimeInSeconds": 872, 
    "historicTrafficTravelTimeInSeconds": 1976, 
    "liveTrafficIncidentsTravelTimeInSeconds": 2905 
},
```

## <a name="request-route-and-leg-details"></a>Demander les détails de l’itinéraire et des différentes étapes

Par défaut, le service Route retourne un tableau de coordonnées. La réponse contient les coordonnées qui composent le trajet dans une liste nommée `points`. La réponse du service Route inclut également la distance depuis le début de l’itinéraire et le temps écoulé estimé. Ces valeurs peuvent être utilisées pour calculer la vitesse moyenne de la totalité de l’itinéraire.

L’illustration suivante montre l’élément `points`.

![Élément Points](media/how-to-use-best-practices-for-routing/points-list-is-hidden-img.png)

Développez l’élément `point` pour afficher la liste des coordonnées du trajet :

![Élément Points développé](media/how-to-use-best-practices-for-routing/points-list-img.png)

Les API Route Directions prennent en charge différents formats d’instructions qui peuvent être utilisés en spécifiant le paramètre **instructionsType** . Pour mettre en forme les instructions et en faciliter le traitement informatique, utilisez **instructionsType=coded** . Utilisez **instructionsType =tagged** pour afficher des instructions sous forme de texte pour l’utilisateur. En outre, les instructions peuvent être mises en forme en tant que texte où certains éléments des instructions sont marqués, et l’instruction est présentée avec une mise en forme spéciale. Pour plus d’informations, consultez la [liste des types d’instructions pris en charge](/rest/api/maps/route/postroutedirections#routeinstructionstype).

Lorsque des instructions sont demandées, la réponse retourne un nouvel élément nommé `guidance`. L’élément `guidance` contient deux éléments d’information : les directions virage par virage et les instructions résumées.

![Type d’instructions](media/how-to-use-best-practices-for-routing/instructions-type-img.png)

L’élément `instructions` contient des instructions virage par virage pour le trajet et l’élément `instructionGroups` contient des instructions résumées. Chaque résumé des instructions couvre un segment du trajet qui peut couvrir plusieurs routes. Les API peuvent retourner des détails pour les sections d’un itinéraire. Par exemple, la plage de coordonnées d’un embouteillage ou la vitesse actuelle du trafic.

![Instructions virage par virage](media/how-to-use-best-practices-for-routing/instructions-turn-by-turn-img.png)

![Instructions résumées](media/how-to-use-best-practices-for-routing/instructions-summary-img.png)

## <a name="request-a-route-for-a-commercial-vehicle"></a>Demander un itinéraire pour un véhicule commercial

Les API Routing d’Azure Maps prennent en charge les itinéraires des véhicules commerciaux, notamment les camions. Les API tiennent compte des limites spécifiées. Il peut s’agir de la hauteur et du poids du véhicule, ou du transport de chargements dangereux. Par exemple, si un véhicule transporte des produits inflammables, le moteur de gestion d’itinéraire évite certains tunnels proches des zones résidentielles.

### <a name="sample-query"></a>Exemple de requête

L’exemple de demande ci-dessous interroge un itinéraire pour un camion. Le camion transporte des déchets dangereux de classe 1.

```http
https://atlas.microsoft.com/route/directions/json?subscription-key=<Your-Azure-Maps-Primary-Subscription-Key>&api-version=1.0&vehicleWidth=2&vehicleHeight=2&vehicleCommercial=true&vehicleLoadType=USHazmatClass1&travelMode=truck&instructionsType=text&query=51.368752,-0.118332:41.385426,-0.128929
```

L’API Route retourne des directions qui prennent en compte les dimensions du camion et les déchets dangereux. Vous pouvez lire les instructions d’itinéraire en développant l’élément `guidance`.

![Camion avec déchets de classe 1](media/how-to-use-best-practices-for-routing/truck-with-hazwaste-img.png)

### <a name="sample-query"></a>Exemple de requête

Si vous modifiez la classe US Hazmat dans la requête ci-dessus, vous obtenez un itinéraire différent reflétant cette modification.

```http
https://atlas.microsoft.com/route/directions/json?subscription-key=<Your-Azure-Maps-Primary-Subscription-Key>&api-version=1.0&vehicleWidth=2&vehicleHeight=2&vehicleCommercial=true&vehicleLoadType=USHazmatClass9&travelMode=truck&instructionsType=text&query=51.368752,-0.118332:41.385426,-0.128929
```

La réponse ci-dessous concerne un camion transportant une substance dangereuse de classe 9, qui est moins dangereuse qu’un matériau dangereux de classe 1. Lorsque vous développez l’élément `guidance` pour lire les directions, vous remarquez que les directions ne sont pas les mêmes. Il existe davantage d’instructions d’itinéraire pour le camion qui transporte un matériau dangereux de classe 1.



![Camion avec déchets de classe 9](media/how-to-use-best-practices-for-routing/truck-with-hazwaste9-img.png)



## <a name="request-traffic-information-along-a-route"></a>Demander des informations de circulation sur un itinéraire

Avec les API Route Directions d’Azure Maps, les développeurs peuvent demander des détails pour chaque type de section en incluant le paramètre `sectionType` dans la demande. Par exemple, vous pouvez demander les informations de vitesse pour chaque segment de l’embouteillage. Reportez-vous à la [liste de valeurs pour la clé sectionType](/rest/api/maps/route/getroutedirections#sectiontype) afin d’en savoir plus sur les différents détails que vous pouvez demander.

### <a name="sample-query"></a>Exemple de requête

La requête suivante définit `sectionType` sur `traffic`. Elle demande les sections qui contiennent des informations sur le trafic entre Seattle et San Diego.

```http
https://atlas.microsoft.com/route/directions/json?subscription-key=<Your-Azure-Maps-Primary-Subscription-Key>&api-version=1.0&sectionType=traffic&query=47.6062,-122.3321:32.7157,-117.1611
```

La réponse contient les sections qui conviennent au trafic le long des coordonnées données.

![Sections de trafic](media/how-to-use-best-practices-for-routing/traffic-section-type-img.png)

Cette option peut être utilisée pour colorer les sections lors du rendu de la carte, comme dans l’image ci-dessous : 

![Sections colorées affichées sur la carte](media/how-to-use-best-practices-for-routing/show-traffic-sections-img.png)

## <a name="calculate-and-optimize-a-multi-stop-route"></a>Calculer et optimiser un itinéraire à plusieurs arrêts

Azure Maps fournit actuellement deux formes d’optimisations d’itinéraire :

* Optimisations basées sur le type d’itinéraire demandé, sans modifier l’ordre des étapes. Vous trouverez ici les [types d’itinéraires pris en charge](/rest/api/maps/route/postroutedirections#routetype)

* L’optimisation des voyageurs de commerce, qui modifie l’ordre des étapes pour obtenir le meilleur ordre pour visiter chaque arrêt

Pour un itinéraire incluant plusieurs arrêts, jusqu’à 150 étapes peuvent être spécifiées dans une même demande d’itinéraire. Les coordonnées de début et de fin peuvent être les mêmes, dans le cas par exemple d’un aller-retour. Toutefois, vous devez fournir au moins une étape supplémentaire pour effectuer le calcul de l’itinéraire. Les étapes peuvent être ajoutées à la requête entre les coordonnées d’origine et de destination.

Si vous souhaitez optimiser le meilleur ordre pour visiter les étapes données, vous devez spécifier **computeBestOrder=true** . Ce scénario est également appelé « problème d’optimisation du voyageur de commerce ».

### <a name="sample-query"></a>Exemple de requête

La requête suivante demande le trajet pour six étapes, avec le paramètre `computeBestOrder` défini sur `false`. Il s’agit également de la valeur par défaut pour le paramètre `computeBestOrder`.

```http
https://atlas.microsoft.com/route/directions/json?api-version=1.0&subscription-key=<Your-Azure-Maps-Primary-Subscription-Key>&computeBestOrder=false&query=47.606544,-122.336502:47.759892,-122.204821:47.670682,-122.120415:47.480133,-122.213369:47.615556,-122.193689:47.676508,-122.206054:47.495472,-122.360861
```

La réponse décrit une longueur de trajet de 140 851 mètres et une durée de trajet de 9 991 secondes.

![Réponse non optimisée](media/how-to-use-best-practices-for-routing/non-optimized-response-img.png)

L’image ci-dessous illustre le trajet résultant de cette requête. Ce trajet est un itinéraire possible. Ce n’est pas le trajet optimal en fonction de l’heure ou de la distance.

![Image non optimisée](media/how-to-use-best-practices-for-routing/non-optimized-image-img.png)



L’ordre des étapes de cet itinéraire est le suivant : 0, 1, 2, 3, 4, 5 et 6.

### <a name="sample-query"></a>Exemple de requête

La requête suivante demande le trajet pour les six mêmes étapes, comme dans l’exemple ci-dessus. Cette fois-ci, le paramètre `computeBestOrder` est défini sur `true` (optimisation du voyageur de commerce).

```http
https://atlas.microsoft.com/route/directions/json?api-version=1.0&subscription-key=<Your-Azure-Maps-Primary-Subscription-Key>&computeBestOrder=true&query=47.606544,-122.336502:47.759892,-122.204821:47.670682,-122.120415:47.480133,-122.213369:47.615556,-122.193689:47.676508,-122.206054:47.495472,-122.360861
```

La réponse décrit une longueur de trajet de 91 814 mètres et une durée de trajet de 7 797 secondes. La distance de déplacement et le temps de trajet sont inférieurs ici, car l’API a retourné l’itinéraire optimisé.

![Réponse optimisée](media/how-to-use-best-practices-for-routing/optimized-response-img.png)

L’image ci-dessous illustre le trajet résultant de cette requête.

![Image optimisée](media/how-to-use-best-practices-for-routing/optimized-image-img.png)

L’itinéraire optimal présente l’ordre suivant pour les étapes : 0, 5, 1, 2, 4, 3 et 6.

>[!TIP]
> Les informations optimisées sur l’ordre des étapes du service Route fournissent un ensemble d’index. Ceux-ci excluent les index d’origine et de destination. Vous devez incrémenter ces valeurs de 1 pour tenir compte de l’origine. Ensuite, ajoutez votre destination à la fin pour obtenir la liste complète des étapes chronologiques.

## <a name="calculate-and-bias-alternative-routes-using-supporting-points"></a>Calculer et choisir des itinéraires alternatifs à l’aide de points de prise en charge

Vous pouvez avoir des situations où vous souhaitez recréer un itinéraire pour calculer zéro ou plusieurs itinéraires alternatifs pour un itinéraire de référence. Par exemple, vous voulez montrer aux clients des itinéraires alternatifs qui passent devant votre magasin. Dans ce cas, vous devez définir un emplacement à l’aide de points de prise en charge. Voici les étapes à suivre pour définir un emplacement :

1. Calculer un itinéraire en l’état et obtenir le trajet à partir de la réponse du service Route
2. Utilisez le trajet de l’itinéraire pour rechercher les emplacements souhaités le long du trajet de l’itinéraire ou à proximité. Par exemple, vous pouvez utiliser l’[API de point d’intérêt](/rest/api/maps/search/getsearchpoi) Azure Maps ou interroger vos propres données dans votre base de données.  
3. Ordonner les emplacements en fonction de la distance depuis le début de l’itinéraire
4. Ajoutez ces emplacements en tant que points de prise en charge dans une nouvelle demande d’itinéraire à l’[API Post Route Directions](/rest/api/maps/route/postroutedirections). Pour en savoir plus sur les points de prise en charge, consultez la documentation sur l’[API Post Route Directions](/rest/api/maps/route/postroutedirections#supportingpoints). 

Lors de l’appel de l’[API Post Route Directions](/rest/api/maps/route/postroutedirections), vous pouvez définir le temps de déviation minimal ou les contraintes de distance, ainsi que les points de prise en charge. Utilisez ces paramètres si vous souhaitez proposer d’autres itinéraires et que vous souhaitez également limiter le temps de trajet. Lorsque ces contraintes sont utilisées, les itinéraires secondaires suivent l’itinéraire de référence à partir du point d’origine pour le temps ou la distance donnés. En d’autres termes, les autres itinéraires divergent de l’itinéraire de référence en fonction des contraintes données.

L’image ci-dessous est un exemple de rendu d’itinéraires alternatifs avec des limites de déviation spécifiées pour le temps et la distance.

![Autres itinéraires](media/how-to-use-best-practices-for-routing/alternative-routes-img.png)

## <a name="use-the-routing-service-in-a-web-app"></a>Utiliser le service Route dans une application web

Le SDK web Azure Maps fournit un [module Service](/javascript/api/azure-maps-rest/). Ce module est une bibliothèque d’assistance facilitant l’utilisation des API REST Azure Maps dans des applications web ou Node.js, en utilisant JavaScript ou TypeScript. Le module Service peut être utilisé pour restituer les itinéraires retournés sur la carte. Le module détermine automatiquement l’API à utiliser avec les demandes GET et POST.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez :

> [!div class="nextstepaction"]
> [Service Route Azure Maps](/rest/api/maps/route)

> [!div class="nextstepaction"]
> [Utiliser le module Service](./how-to-use-services-module.md)

> [!div class="nextstepaction"]
> [Afficher un itinéraire sur la carte](./map-route.md)

> [!div class="nextstepaction"]
> [Package NPM Azure Maps](https://www.npmjs.com/package/azure-maps-rest  )
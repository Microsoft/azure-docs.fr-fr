---
title: Demander des données de transport public en temps réel avec le service Mobility (préversion) de Microsoft Azure Maps
description: Apprenez à demander des données de transport public en temps réel, telles que des arrivées à un arrêt spécifique. Pour ce faire, découvrez comment utiliser le service Mobility (préversion) d’Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: d3e3dc4b0e3bc64a38856da8344583b744ea62b6
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96906044"
---
# <a name="request-real-time-public-transit-data-using-the-azure-maps-mobility-services-preview"></a>Demander des données de transport public en temps réel avec le service Mobility (préversion) d’Azure Maps 

> [!IMPORTANT]
> Le service Mobility d’Azure Maps est disponible en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


Cet article vous montre comment utiliser le [service Mobility](/rest/api/maps/mobility) d’Azure Maps pour demander des données de transport public en temps réel.

Dans cet article, découvrez comment demander les horaires d’arrivée en temps réel de toutes les lignes de bus à un arrêt donné

## <a name="prerequisites"></a>Prérequis

Vous devez d’abord disposer d’un compte Azure Maps et d’une clé d’abonnement pour effectuer des appels vers les API de transports publics Azure Maps. Pour plus d'informations sur la création d'un compte Azure Maps, suivez les instructions de la section [Créer un compte](quick-demo-map-app.md#create-an-azure-maps-account). Suivez les étapes de la section [Obtenir la clé primaire](quick-demo-map-app.md#get-the-primary-key-for-your-account) afin de vous procurer la clé primaire de votre compte. Pour plus d’informations sur l’authentification dans Azure Maps, voir [Gérer l’authentification dans Azure Maps](./how-to-manage-authentication.md).

Cet article utilise [l’application Postman](https://www.getpostman.com/apps) pour générer des appels REST. Vous pouvez utiliser l’environnement de développement d’API que vous préférez.

## <a name="request-real-time-arrivals-for-a-stop"></a>Demander les horaires d’arrivée en temps réel à un arrêt donné

Pour demander les horaires d’arrivée en temps réel à un arrêt de transport public particulier, vous devez envoyer une demande à l’[API Real-time Arrivals](/rest/api/maps/mobility/getrealtimearrivalspreview) du [service Mobility](/rest/api/maps/mobility) (préversion) d’Azure Maps. Vous devez indiquer les paramètres **metroID** et **stopID** dans la demande. Pour en savoir plus sur l’obtention de ces paramètres, consultez notre guide sur la façon de [demander des itinéraires des transports publics](./how-to-request-transit-data.md).

Utilisons « 522 » comme numéro de zone urbaine, qui est le numéro de la zone « Seattle–Tacoma–Bellevue, WA ». Utilisons « 522---2060603 » comme numéro d’arrêt ; cet arrêt de bus se trouve à « Ne 24th St & 162nd Ave Ne, Bellevue WA ». Pour demander les horaires d’arrivée en temps réel des cinq prochains passages de bus à cet arrêt, effectuez les étapes suivantes :

1. Ouvrez l’application Postman, puis créez une collection destinée au stockage des demandes. En haut de l'application Postman, sélectionnez **New** (Nouveau). Dans la fenêtre **Create New** (Créer nouveau), sélectionnez **Collection**.  Nommez la collection puis sélectionnez le bouton **Create** (Créer).

2. Pour créer la demande, sélectionnez **New** à nouveau. Dans la fenêtre **Create New** (Créer nouveau), sélectionnez **Request** (Demande). Entrez un **Request name** (Nom de demande) pour la demande. Sélectionnez la collection que vous avez créée à l’étape précédente comme emplacement dans lequel enregistrer la demande. Sélectionnez ensuite **Enregistrer**.

    ![Créer une demande dans Postman](./media/how-to-request-transit-data/postman-new.png)

3. Sélectionnez la méthode HTTP **GET** sous l'onglet Builder (Générateur), puis entrez l'URL suivante pour créer une requête GET. Remplacez `{subscription-key}` par votre clé primaire Azure Maps.

    ```HTTP
    https://atlas.microsoft.com/mobility/realtime/arrivals/json?subscription-key={subscription-key}&api-version=1.0&metroId=522&query=522---2060603&transitType=bus
    ```

4. Après une demande réussie, vous recevrez la réponse suivante.  Notez que le paramètre scheduleType définit si l’horaire d’arrivée est estimé d’après des données en temps réel ou statiques.

    ```JSON
    {
        "results": [
            {
                "arrivalMinutes": 8,
                "scheduleType": "realTime",
                "patternId": "522---4143196",
                "line": {
                    "lineId": "522---3760143",
                    "lineGroupId": "522---666077",
                    "direction": "backward",
                    "agencyId": "522---5872",
                    "agencyName": "Metro Transit",
                    "lineNumber": "249",
                    "lineDestination": "South Bellevue S Kirkland P&R",
                    "transitType": "Bus"
                },
                "stop": {
                    "stopId": "522---2060603",
                    "stopKey": "71300",
                    "stopName": "NE 24th St & 162nd Ave NE",
                    "stopCode": "71300",
                    "position": {
                        "latitude": 47.631504,
                        "longitude": -122.125275
                    },
                    "mainTransitType": "Bus",
                    "mainAgencyId": "522---5872",
                    "mainAgencyName": "Metro Transit"
                }
            },
            {
                "arrivalMinutes": 25,
                "scheduleType": "realTime",
                "patternId": "522---3510227",
                "line": {
                    "lineId": "522---2756599",
                    "lineGroupId": "522---666063",
                    "direction": "forward",
                    "agencyId": "522---5872",
                    "agencyName": "Metro Transit",
                    "lineNumber": "226",
                    "lineDestination": "Bellevue Transit Center Crossroads",
                    "transitType": "Bus"
                },
                "stop": {
                    "stopId": "522---2060603",
                    "stopKey": "71300",
                    "stopName": "NE 24th St & 162nd Ave NE",
                    "stopCode": "71300",
                    "position": {
                        "latitude": 47.631504,
                        "longitude": -122.125275
                    },
                    "mainTransitType": "Bus",
                    "mainAgencyId": "522---5872",
                    "mainAgencyName": "Metro Transit"
                }
            }
        ]
    }
    ```

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment demander des données de transport public à l’aide du service Mobility (préversion) :

> [!div class="nextstepaction"]
> [Comment demander des données de transit](how-to-request-transit-data.md)

Explorez la documentation relative à l’API du service Mobility Azure Maps (préversion) :

> [!div class="nextstepaction"]
> [Documentation sur l’API du service Mobility](/rest/api/maps/mobility)
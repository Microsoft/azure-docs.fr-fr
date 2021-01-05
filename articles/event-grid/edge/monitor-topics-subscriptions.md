---
title: Superviser les rubriques et les abonnements aux événements - Azure Event Grid IoT Edge | Microsoft Docs
description: Superviser les rubriques et les abonnements aux événements
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: d230be4f74abd61ad7b7f0fdb3facb32ee63da73
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86171531"
---
# <a name="monitor-topics-and-event-subscriptions"></a>Superviser les rubriques et les abonnements aux événements

Event Grid sur Edge expose un certain nombre de métriques pour les rubriques et abonnements aux événements au format [d'exposition Prometheus](https://prometheus.io/docs/instrumenting/exposition_formats/). Cet article décrit les métriques disponibles et comment les activer.

## <a name="enable-metrics"></a>Activer les mesures

Configurez le module pour émettre des métriques en définissant la variable d’environnement `metrics__reporterType` sur `prometheus` dans les options de création du conteneur :

 ```json
        {
          "Env": [
            "metrics__reporterType=prometheus"
          ],
          "HostConfig": {
            "PortBindings": {
              "4438/tcp": [
                {
                    "HostPort": "4438"
                }
              ]
            }
          }
        }
 ```    

Les métriques seront disponibles au niveau `5888/metrics` du module pour http et `4438/metrics` pour HTTPS. Par exemple, `http://<modulename>:5888/metrics?api-version=2019-01-01-preview` pour http. À ce stade, un module de métriques peut interroger le point de terminaison pour collecter les métriques comme dans cet [exemple d'architecture](https://github.com/veyalla/ehm).

## <a name="available-metrics"></a>Métriques disponibles

Les rubriques et les abonnements aux événements émettent des métriques pour vous donner des insights en termes de remise des événements et de performances des modules.

### <a name="topic-metrics"></a>Métriques de rubrique

| Métrique | Description |
| ------ | ----------- |
| EventsReceived | Nombre d’événements publiés dans la rubrique
| UnmatchedEvents | Nombre d’événements publiés dans la rubrique ne correspondant pas à un abonnement à un événement et supprimés
| SuccessRequests | Nombre de requêtes de publication entrantes reçues par la rubrique
| SystemErrorRequests | Nombre de requêtes de publication entrantes ayant échoué en raison d’une erreur système interne
| UserErrorRequests | Nombre de requêtes de publication entrantes ayant échoué en raison d'une erreur de l’utilisateur, par exemple un JSON mal formé
| SuccessRequestLatencyMs | Latence de la réponse de requête publication en millisecondes


### <a name="event-subscription-metrics"></a>Métriques des abonnements aux événements

| Métrique | Description |
| ------ | ----------- |
| DeliverySuccessCounts | Nombre d’événements correctement remis au point de terminaison configuré
| DeliveryFailureCounts | Nombre d’événements dont la remise au point de terminaison configuré a échoué
| DeliverySuccessLatencyMs | Latence des événements correctement remis en millisecondes
| DeliveryFailureLatencyMs | Latence des échecs de remise d'événement en millisecondes
| SystemDelayForFirstAttemptMs | Délai système des événements avant la première tentative de remise en millisecondes
| DeliveryAttemptsCount | Nombre de tentatives de remise d’événement - Ayant abouti ou échoué
| ExpiredCounts | Nombre d’événements qui ont expiré et qui n’ont pas été remis au point de terminaison configuré
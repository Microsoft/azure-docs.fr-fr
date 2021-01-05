---
title: Enregistrement des événements sur Azure Event Hubs dans Gestion des API Azure | Microsoft Docs
description: Découvrez comment enregistrer des événements sur Azure Event Hubs dans Gestion des API Azure. Event Hubs est un service hautement évolutif d’entrée de données.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 88f6507d-7460-4eb2-bffd-76025b73f8c4
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 01/29/2018
ms.author: apimpm
ms.openlocfilehash: 5066169951409fa86aa75a64e8fc6d4189947f27
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92072405"
---
# <a name="how-to-log-events-to-azure-event-hubs-in-azure-api-management"></a>Comment enregistrer des événements sur Azure Event Hubs dans Gestion des API Azure
Les concentrateurs d'événements Azure sont un service d'entrée de données hautement évolutif qui peut traiter des millions d'événements par seconde afin que vous puissiez traiter et analyser les grandes quantités de données générées par vos périphériques connectés et vos applications. Les concentrateurs d'événements fonctionnent comme la « porte d'entrée » d’un pipeline d’événements, et une fois que les données sont collectées dans un concentrateur d'événements, elles peuvent être transformées et stockées à l'aide de n'importe quel fournisseur d'analyse en temps réel ou d’adaptateurs de traitement par lot ou de stockage. Les concentrateurs d'événements dissocient la production d'un flux d'événements de la consommation de ces événements, de manière à ce que les consommateurs d'événements puissent accéder aux événements selon leur propre planification.

Cet article, qui complète la vidéo [Intégrer la gestion des API Azure avec Event Hubs](https://azure.microsoft.com/documentation/videos/integrate-azure-api-management-with-event-hubs/) , explique comment consigner les événements Gestion des API à l’aide d’Azure Event Hubs.

## <a name="create-an-azure-event-hub"></a>Création d'un hub d'événements Azure

Vous trouverez des instructions détaillées pour créer un hub d’événements et obtenir les chaînes de connexion nécessaires pour envoyer et recevoir des événements sur le hub dans la section [Créer un espace de noms et un hub d’événements Event Hubs avec le Portail Azure](../event-hubs/event-hubs-create.md).

## <a name="create-an-api-management-logger"></a>Créer un enregistreur d’événements de gestion des API
Maintenant que vous disposez d’un hub d’événements, l’étape suivante consiste à configurer un [enregistreur d’événements](/rest/api/apimanagement/2019-12-01/logger) dans votre service Gestion des API afin qu’il puisse enregistrer des événements dans le hub d’événements.

Les enregistreurs d’événements de gestion des API peuvent être configurés à l’aide de l’ [API REST Gestion des API](/rest/api/apimanagement/ApiManagementREST/API-Management-REST). Pour obtenir des exemples détaillés de requêtes, consultez [Créer des enregistreurs d'événements](/rest/api/apimanagement/2019-12-01/logger/createorupdate).

## <a name="configure-log-to-eventhub-policies"></a>Configurer des stratégies Enregistrer sur event hub

Une fois que votre journal est configuré dans Gestion des API, vous pouvez configurer votre stratégie Enregistrer sur event hub pour enregistrer les événements de votre choix. La stratégie Enregistrer sur event hub peut être utilisée dans la section de la stratégie entrante ou dans la section de la stratégie sortante.

1. Accédez à votre instance APIM.
2. Sélectionnez l’onglet API.
3. Sélectionnez l’API à laquelle vous souhaitez ajouter la stratégie. Dans cet exemple, nous ajoutons une stratégie à **l’API Echo** dans le produit **Illimité**.
4. Sélectionnez **Toutes les opérations**.
5. En haut de l’écran, sélectionnez l’onglet Conception.
6. Dans la fenêtre Traitement entrant ou sortant, cliquez sur le triangle (à côté du crayon).
7. Sélectionnez l’Éditeur de code. Pour plus d’informations, consultez la section [Guide pratique pour configurer ou modifier des stratégies](set-edit-policies.md).
8. Placez votre curseur dans la section des stratégies `inbound` ou `outbound`.
9. Dans la fenêtre de droite, sélectionnez **Stratégies avancées** > **Consigner sur Event Hub**. Cela permet d’insérer le modèle d’instruction de stratégie `log-to-eventhub`.

```xml
<log-to-eventhub logger-id="logger-id">
    @{
        return new JObject(
            new JProperty("EventTime", DateTime.UtcNow.ToString()),
            new JProperty("ServiceName", context.Deployment.ServiceName),
            new JProperty("RequestId", context.RequestId),
            new JProperty("RequestIp", context.Request.IpAddress),
            new JProperty("OperationName", context.Operation.Name)
        ).ToString();
    }
</log-to-eventhub>
```
Remplacez `logger-id` par la valeur que vous avez utilisée pour `{loggerId}` dans l’URL de la requête pour créer l’enregistreur d’événements à l’étape précédente.

Vous pouvez utiliser toute expression qui renvoie une chaîne en tant que valeur pour l’élément `log-to-eventhub` . Dans cet exemple, une chaîne dans le format JSON contenant la date et l’heure, le nom du service, l’ID de la requête, son adresse IP et le nom de l’opération est consignée.

Cliquez sur **Enregistrer** pour enregistrer la configuration de la stratégie mise à jour. Dès qu’elle est enregistrée, la stratégie est active et les événements sont enregistrés dans l’Event Hub désigné.

> [!NOTE]
> La taille maximale des messages pris en charge pouvant être envoyée à un Event Hub par cette stratégie Gestion des API est de 200 kilooctets (ko). Si un message envoyé à un Event Hub a une taille supérieure à 200 ko, il est automatiquement tronqué. Le message tronqué est ensuite transféré vers Event Hubs.

## <a name="preview-the-log-in-event-hubs-by-using-azure-stream-analytics"></a>Afficher un aperçu du journal dans Event Hubs à l’aide d’Azure Stream Analytics

Vous pouvez afficher un aperçu du journal dans Event Hubs à l’aide de [requêtes Azure Stream Analytics](../event-hubs/process-data-azure-stream-analytics.md). 

1. Dans le Portail Azure, accédez au Event Hub auquel le journal envoie des événements. 
2. Sous **Fonctionnalités**, sélectionnez l’onglet **Traiter les données**.
3. Sur la vignette **Activer des insights en temps réel à partir d’événements**, sélectionnez **Explorer**.
4. Vous devez être en mesure d’afficher un aperçu du journal dans l’onglet **Aperçu de l’entrée**. Si les données qui apparaissent ne sont pas à jour, sélectionnez **Actualiser** pour afficher les événements les plus récents.

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur Azure Event Hubs
  * [Prise en main avec Azure Event Hubs](../event-hubs/event-hubs-c-getstarted-send.md)
  * [Réception de messages avec EventProcessorHost](../event-hubs/event-hubs-dotnet-standard-getstarted-send.md)
  * [Guide de programmation Event Hubs](../event-hubs/event-hubs-programming-guide.md)
* En savoir plus sur l’intégration de Gestion des API et Event Hubs
  * [Référence d’entité d’enregistreur](/rest/api/apimanagement/2019-12-01/logger)
  * [Référence de stratégie log-to-eventhub](./api-management-advanced-policies.md#log-to-eventhub)
  * [Surveiller des API avec la Gestion des API Azure, Event Hubs et Moesif](api-management-log-to-eventhub-sample.md)  
* En savoir plus sur [l’intégration avec Azure Application Insights](api-management-howto-app-insights.md)

[publisher-portal]: ./media/api-management-howto-log-event-hubs/publisher-portal.png
[create-event-hub]: ./media/api-management-howto-log-event-hubs/create-event-hub.png
[event-hub-connection-string]: ./media/api-management-howto-log-event-hubs/event-hub-connection-string.png
[event-hub-dashboard]: ./media/api-management-howto-log-event-hubs/event-hub-dashboard.png
[receiving-policy]: ./media/api-management-howto-log-event-hubs/receiving-policy.png
[sending-policy]: ./media/api-management-howto-log-event-hubs/sending-policy.png
[event-hub-policy]: ./media/api-management-howto-log-event-hubs/event-hub-policy.png
[add-policy]: ./media/api-management-howto-log-event-hubs/add-policy.png
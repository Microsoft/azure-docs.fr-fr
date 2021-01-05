---
title: Vue d’ensemble des enrichissements de messages Azure IoT Hub
description: Cet article présente les enrichissements de messages, qui donnent à IoT Hub la capacité de marquer les messages avec des informations supplémentaires avant que ces messages soient envoyés au point de terminaison désigné.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 05/10/2019
ms.author: robinsh
ms.openlocfilehash: 3975a57c095a8593e392e932bd125308853d3756
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92541517"
---
# <a name="message-enrichments-for-device-to-cloud-iot-hub-messages"></a>Enrichissements de messages pour les messages IoT Hub appareil-à-cloud

Les *enrichissements de messages* sont la capacité du IoT Hub à *horodater* des messages avec des informations supplémentaires avant que les messages ne soient envoyés au point de terminaison désigné. Vous pouvez utiliser les enrichissements de messages pour inclure des données permettant de simplifier le traitement en aval. Par exemple, enrichir des messages de télémétrie d’appareil avec une balise de jumeau d’appareil peut réduire la charge de clients en leur évitant d’effectuer des appels d’API doubles pour cette information.

![Flux des enrichissements de messages](./media/iot-hub-message-enrichments-overview/message-enrichments-flow.png)

Un enrichissement de message comporte trois éléments clés :

* Clé ou nom de l’enrichissement

* Une valeur

* Un ou plusieurs [points de terminaison](iot-hub-devguide-endpoints.md) pour lequel l’enrichissement doit être appliqué.

La **clé** est une chaîne. Une clé ne peut contenir que des caractères alphanumériques ou les caractères spéciaux suivants : trait d’union (`-`), trait de soulignement (`_`) et point (`.`).

La **valeur** peut être l’un des exemples suivants :

* N’importe quelle chaîne statique. Des valeurs dynamiques telles que des conditions, une logique et des fonctions ne sont pas autorisées. Par exemple, si vous développez une application SaaS utilisée par plusieurs clients, vous pouvez attribuer un identificateur à chaque client et rendre cet identificateur disponible dans l’application. Lorsque l’application s’exécute, IoT Hub horodate les messages de télémétrie de l’appareil avec l’identificateur du client, ce qui permet de traiter les messages différemment pour chaque client.

* Le nom de l’IoT Hub qui envoie le message. Cette valeur est *$iothubname*.

* Informations du jumeau d’appareil, comme son chemin d’accès. Des exemples seraient *$twin.tags.field* et *$twin.tags.latitude*.

   > [!NOTE]
   > Pour l’instant, seuls $iothubname, $twin.tags, $twin.properties.desired et $twin.properties.reported sont des variables prises en charge pour l’enrichissement de message.

Les enrichissements de message sont ajoutés en tant que propriétés d’application aux messages envoyés aux points de terminaison choisis.  

## <a name="applying-enrichments"></a>Application des enrichissements

Les messages peuvent provenir de n’importe quelle source de données prise en charge par [le routage des messages IoT Hub](iot-hub-devguide-messages-d2c.md), y compris les exemples suivants :

* données de télémétrie de l’appareil comme la température ou la pression
* notifications de modification du jumeau d’appareil, modifications du jumeau d’appareil
* événements du cycle de vie de l’appareil, tels que la date de la création ou de la suppression de l’appareil

Vous pouvez ajouter des enrichissements aux messages destinés au point de terminaison intégré d’un IoT Hub, ou aux messages acheminés vers des points de terminaison personnalisés comme le stockage d’objets blob Azure, une file d’attente Service Bus ou une rubrique Service Bus.

Vous pouvez ajouter des enrichissements aux messages qui sont publiés dans Event Grid en sélectionnant Event Grid comme point de terminaison. Nous créons un itinéraire par défaut dans IoT Hub pour la télémétrie d’appareil, selon votre abonnement Event Grid. Cet itinéraire unique peut gérer tous vos abonnements Event Grid. Vous pouvez configurer des enrichissements pour le point de terminaison de la grille d’événements après avoir créé l’abonnement Event Grid à la télémétrie d’appareil. Pour plus d’informations, consultez [React to IoT Hub events by using Event Grid to trigger actions](iot-hub-event-grid.md) (Réagir aux événements IoT Hub en utilisant Event Grid pour déclencher des actions).

Les enrichissements sont appliqués par point de terminaison. Si vous spécifiez cinq enrichissements à horodater pour un point de terminaison spécifique, tous les messages acheminés vers ce point de terminaison sont horodatés avec les cinq mêmes enrichissements.

Des enrichissements peuvent être configurés à l’aide des méthodes suivantes :

| **Méthode** | **Commande** |
| ----- | -----| 
| Portail | [Azure portal](https://portal.azure.com) | Voir le [tutoriel sur les enrichissements de messages](tutorial-message-enrichments.md) | 
| Azure CLI   | [az iot hub message-enrichment](/cli/azure/iot/hub/message-enrichment) |
| Azure PowerShell | [Add-AzIotHubMessageEnrichment](/powershell/module/az.iothub/add-aziothubmessageenrichment) |

L’ajout d’enrichissements de message n’ajoute pas de latence au routage des messages.

Pour essayer les enrichissements de messages, consultez le [tutoriel sur les enrichissements de messages](tutorial-message-enrichments.md)

## <a name="limitations"></a>Limites

* Vous pouvez ajouter jusqu’à 10 enrichissements par IoT Hub pour ces hubs dans le niveau de base ou standard. Pour les IoT Hubs dans le niveau gratuit, vous pouvez ajouter jusqu’à 2 enrichissements.

* Dans certains cas, si vous appliquez un enrichissement avec une valeur définie sur une balise ou une propriété dans le jumeau d’appareil, la valeur sera horodatée en tant que valeur de chaîne. Par exemple, si une valeur de l’enrichissement est définie sur $twin.tags.field, les messages sont horodatés avec la chaîne « $twin.tags.field » plutôt qu’avec la valeur de ce champ de l’appareil jumeau. Cela se produit dans les cas suivants :

   * Votre IoT Hub est dans le niveau de base. Les IoT Hubs de niveau de base ne prennent pas en charge les jumeaux d’appareil.

   * Votre IoT Hub est dans le niveau standard, mais l’appareil qui envoie le message n’a aucun jumeau d’appareil.

   * Votre IoT Hub est dans le niveau standard, mais le chemin d’accès de l’appareil jumeau utilisé pour la valeur de l’enrichissement n’existe pas. Par exemple, si la valeur de l’enrichissement est définie sur $twin.tags.location et que le jumeau d’appareil n’a pas de propriété location dans les balises, le message est horodaté avec la chaîne « $twin.tags.location ». 

* Les mises à jour d’un jumeau d’appareil peuvent prendre jusqu’à cinq minutes pour apparaître dans la valeur d’enrichissement correspondante.

* La taille totale du message, y compris des enrichissements, ne peut pas dépasser 256 Ko. Si la taille d’un message dépasse 256 Ko, l’IoT Hub supprimera le message. Vous pouvez utiliser les [métriques IoT Hub](monitor-iot-hub-reference.md#metrics) pour identifier et déboguer des erreurs lorsque les messages sont supprimés. Par exemple, vous pouvez analyser la métrique *Messages de télémétrie incompatibles* ( *d2c.telemetry.egress.invalid* ) dans les [métriques de routage](monitor-iot-hub-reference.md#routing-metrics). Pour plus d’informations, consultez [Surveiller IoT Hub](monitor-iot-hub.md).

* Les enrichissements de message ne s’appliquent pas aux événements de changement de jumeau numérique.

## <a name="pricing"></a>Tarifs

Les enrichissements de messages sont disponibles sans frais supplémentaires. Actuellement, vous êtes facturé lorsque vous envoyez un message à un IoT Hub. Vous n’êtes facturé qu’une fois pour ce message, même s’il est envoyé à plusieurs points de terminaison.

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants pour plus d’informations sur le routage des messages vers un IoT Hub :

* [Tutoriel sur les enrichissements de messages](tutorial-message-enrichments.md)

* [Utiliser le routage des messages IoT Hub pour envoyer des messages appareil-à-cloud à différents points de terminaison](iot-hub-devguide-messages-d2c.md)

* [Tutoriel : Routage d’IoT Hub](tutorial-routing.md)
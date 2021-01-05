---
title: 'Informations de référence : journal de ressource de Gestion des API Azure'
description: Informations de référence de schéma pour le journal de ressource GatewayLogs de Gestion des API Azure
services: api-management
author: dlepow
ms.service: api-management
ms.custom: mvc
ms.topic: tutorial
ms.date: 10/14/2020
ms.author: apimpm
ms.openlocfilehash: 064ac21e01239f7be3ddca9e48089ce880c6af58
ms.sourcegitcommit: 0d171fe7fc0893dcc5f6202e73038a91be58da03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93379900"
---
# <a name="reference-api-management-resource-log-schema"></a>Référence : Schéma du journal de ressource de Gestion des API

Cet article fournit des informations de référence de schéma pour le journal de ressource GatewayLogs de Gestion des API Azure. Les entrées de journal incluent également des champs dans le [schéma commun de niveau supérieur](../azure-monitor/platform/resource-logs-schema.md#top-level-common-schema).

Pour activer la collecte du journal de ressource dans Gestion des API, consultez [Surveiller les API publiées](api-management-howto-use-azure-monitor.md#resource-logs).

## <a name="gatewaylogs-schema"></a>Schéma GatewayLogs

Les propriétés suivantes sont journalisées pour chaque demande d’API.

| Propriété  | Type | Description |
| ------------- | ------------- | ------------- |
| method | string | Méthode HTTP de la requête entrante |
| url | string | URL de la requête entrante |
| responseCode | entier | Code d’état de la réponse HTTP envoyée à un client |
| responseSize | entier | Nombre d’octets envoyés à un client au cours du traitement de la requête | 
| cache | string | État d’implication du cache du service Gestion des API dans le traitement des requêtes (atteint, manqué, aucun) | 
| apiId | string | Identificateur d’entité d’API pour la requête actuelle | 
| operationId | string | Identificateur d’entité d’opération pour la requête actuelle | 
| clientProtocol | string | Version du protocole HTTP de la requête entrante |
| clientTime | entier | Nombre de millisecondes consacrées à l’ensemble des E/S du client (connexion, envoi et réception d’octets) | 
| apiRevision | string | Révision d’API pour la demande actuelle | 
| clientTlsVersion| string | Version TLS utilisée par le client qui envoie la demande |
| lastError | object | Pour une demande ayant échoué, détails sur la dernière erreur de traitement de la demande | 
| backendMethod | string | Méthode HTTP de la requête envoyée à un serveur principal |
| backendUrl | string | URL de la requête envoyée à un serveur principal |
| backendResponseCode | entier | Code de la réponse HTTP reçue d’un serveur principal |
| backedProtocol | string | Version du protocole HTTP de la requête envoyée à un service principal |
| backendTime | entier | Nombre de millisecondes consacrées à l’ensemble des E/S du back-end (connexion, envoi et réception d’octets) | 


## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur la supervision des API dans Gestion des API, consultez [Surveiller les API publiées](api-management-howto-use-azure-monitor.md).
* Découvrez en plus sur les [schémas communs et spécifiques de services pour les journaux de ressources Azure](../azure-monitor/platform/resource-logs-schema.md).


---
title: Configuration – Azure Event Grid IoT Edge | Microsoft Docs
description: Configuration dans Event Grid sur IoT Edge.
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: 632227579fd021a0d2ce1d0b1bb0b8a8288c5f47
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86171667"
---
# <a name="event-grid-configuration"></a>Configuration d’Event Grid

Event Grid fournit de nombreuses configurations modifiables environnement par environnement. La section suivante référence toutes les options disponibles et leurs valeurs par défaut.

## <a name="tls-configuration"></a>Configuration TLS

Pour plus d’informations sur l’authentification client en général, voir [Sécurité et authentification](security-authentication.md). Vous trouverez des exemples de son utilisation dans [cet article](configure-api-protocol.md).

| Nom de la propriété | Description |
| ---------------- | ------------ |
|`inbound__serverAuth__tlsPolicy`| Stratégie TLS du module Event Grid. Valeur par défaut : HTTPS uniquement.
|`inbound__serverAuth__serverCert__source`| Source du certificat de serveur utilisée par le module Event Grid pour sa configuration TLS. Valeur par défaut : IoT Edge.

## <a name="incoming-client-authentication"></a>Authentification de client entrant

Pour plus d’informations sur l’authentification client en général, voir [Sécurité et authentification](security-authentication.md). Vous trouverez des exemples dans [cet article](configure-client-auth.md).

| Nom de la propriété | Description |
| ---------------- | ------------ |
|`inbound__clientAuth__clientCert__enabled`| Propriété permettant d’activer et de désactiver l’authentification client par certificat. La valeur par défaut est true.
|`inbound__clientAuth__clientCert__source`| Source permettant de valider les certificats clients. Valeur par défaut : IoT Edge.
|`inbound__clientAuth__clientCert__allowUnknownCA`| Stratégie permettant d’autoriser un certificat client auto-signé. La valeur par défaut est true.
|`inbound__clientAuth__sasKeys__enabled`| Propriété permettant d’activer et de désactiver l’authentification client par clé SAP. Valeur par défaut : désactivée.
|`inbound__clientAuth__sasKeys__key1`| L’une des valeurs permettant de valider les demandes entrantes.
|`inbound__clientAuth__sasKeys__key2`| Deuxième valeur facultative permettant de valider les demandes entrantes.

## <a name="outgoing-client-authentication"></a>Authentification de client sortant
Pour plus d’informations sur l’authentification client en général, voir [Sécurité et authentification](security-authentication.md). Vous trouverez des exemples dans [cet article](configure-identity-auth.md).

| Nom de la propriété | Description |
| ---------------- | ------------ |
|`outbound__clientAuth__clientCert__enabled`| Propriété permettant d’activer/de désactiver l’attachement d’un certificat d’identité pour les demandes sortantes. La valeur par défaut est true.
|`outbound__clientAuth__clientCert__source`| Source permettant de récupérer le certificat sortant du module Event Grid. Valeur par défaut : IoT Edge.

## <a name="webhook-event-handlers"></a>Gestionnaires d’événements de webhook

Pour plus d’informations sur l’authentification client en général, voir [Sécurité et authentification](security-authentication.md). Vous trouverez des exemples dans [cet article](configure-webhook-subscriber-auth.md).

| Nom de la propriété | Description |
| ---------------- | ------------ |
|`outbound__webhook__httpsOnly`| Stratégie permettant de contrôler si seuls les abonnés HTTPS sont autorisés. Valeur par défaut : true (HTTPS uniquement).
|`outbound__webhook__skipServerCertValidation`| Indicateur permettant de contrôler s’il faut valider le certificat de l’abonné. La valeur par défaut est true.
|`outbound__webhook__allowUnknownCA`| Stratégie permettant de contrôler si l’abonné peut présenter un certificat auto-signé. La valeur par défaut est true. 

## <a name="delivery-and-retry"></a>Livraison et nouvelle tentative

Pour plus d’informations sur cette fonctionnalité en général, voir [Remise et nouvelle tentative](delivery-retry.md).

| Nom de la propriété | Description |
| ---------------- | ------------ |
| `broker__defaultMaxDeliveryAttempts` | Nombre maximal de tentatives de livraison d’un événement. La valeur par défaut est 30.
| `broker__defaultEventTimeToLiveInSeconds` | Durée de vie (TTL) en secondes après laquelle un événement sera supprimé s’il n’est pas remis. Valeur par défaut : **7 200** secondes.

## <a name="output-batching"></a>Traitement par lot des sorties

Pour plus d’informations sur cette fonctionnalité en général, voir [Remise et traitement par lot de sortie](delivery-output-batching.md).

| Nom de la propriété | Description |
| ---------------- | ------------ |
| `api__deliveryPolicyLimits__maxBatchSizeInBytes` | Valeur maximale autorisée pour le bouton `ApproxBatchSizeInBytes`. La valeur par défaut est `1_058_576`.
| `api__deliveryPolicyLimits__maxEventsPerBatch` | Valeur maximale autorisée pour le bouton `MaxEventsPerBatch`. La valeur par défaut est `50`.
| `broker__defaultMaxBatchSizeInBytes` | Taille maximale de demande de livraison quand seule la valeur `MaxEventsPerBatch` est spécifiée. La valeur par défaut est `1_058_576`.
| `broker__defaultMaxEventsPerBatch` | Nombre maximal d’événements à ajouter à un lot quand seule la valeur `MaxBatchSizeInBytes` est spécifiée. La valeur par défaut est `10`.

## <a name="metrics"></a>Mesures

Pour en savoir plus sur l’utilisation des métriques avec Event Grid sur IoT Edge, consultez les [rubriques et abonnements relatifs à l’analyse](monitor-topics-subscriptions.md)

| Nom de la propriété | Description |
| ---------------- | ------------ |
| `metrics__reporterType` | Type de rapporteur pour le point de terminaison de métriques. La valeur par défaut est `none` et désactive les métriques. La paramétrer sur `prometheus` active les métriques au format d’exposition Prometheus.
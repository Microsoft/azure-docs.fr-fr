---
title: Informations de référence sur l’API MQTT 5 d’Azure IoT Hub (préversion)
description: En savoir plus sur les informations de référence relatives à l’API MQTT 5 d’Azure IoT Hub
services: iot-hub
author: jlian
ms.service: iot-fundamentals
ms.topic: reference
ms.date: 11/19/2020
ms.author: jlian
ms.openlocfilehash: 5f0af7d6bf16a05fad1ca9df5db1729abd088010
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96602994"
---
# <a name="iot-hub-data-plane-mqtt-5-api-reference"></a>Informations de référence sur l’API MQTT 5 du plan de données IoT Hub

Ce document définit les opérations disponibles dans la version 2.0 (api-version : `2020-10-01-preview`) de l’API du plan de données IoT Hub.

## <a name="operations"></a>Operations

### <a name="get-twin"></a>Obtenir un jumeau

Obtenir l’état du jumeau

#### <a name="request"></a>Requête

**Nom de la rubrique :** `$iothub/twin/get`

**Propriétés :** aucune

**Charge utile :** vide

#### <a name="success-response"></a>Réponse en cas de réussite

**Propriétés :** aucune

**Charge utile** : Jumeau

#### <a name="alternative-responses"></a>Autres réponses

| État | Name | Description |
| :----- | :--- | :---------- |
| 0100 |  Demande incorrecte | Le message d’opération est incorrect et ne peut pas être traité. |
| 0101 |  Non autorisé | Le client n’est pas autorisé à effectuer l’opération. |
| 0102 |  Non autorisée | L’opération n’est pas autorisée. |
| 0501 |  Throttled | Le taux de requêtes est trop élevé par SKU. |
| 0502 |  Quota dépassé | Le quota quotidien par SKU actuel est dépassé. |
| 0601 |  Erreur de serveur | Erreur interne du serveur. |
| 0602 |  Délai d'expiration | L’opération a expiré avant d’être terminée. |
| 0603 |  Serveur occupé | Le serveur est occupé. |

#### <a name="pseudo-code-sample"></a>Exemple de pseudo-code

```

-> PUBLISH
    QoS: 0
    Topic: $iothub/twin/get
<- PUBLISH
    QoS: 0
    Topic: $iothub/responses

```

### <a name="patch-twin-reported"></a>Jumeau patch signalé

État signalé du jumeau patch

#### <a name="request"></a>Requête

**Nom de la rubrique :** `$iothub/twin/patch/reported`

**Propriétés** :

| Name | Type | Obligatoire | Description |
| :--- | :--- | :------- | :---------- |
| if-version | u64 | Non |  |

**Charge utile** : TwinState

#### <a name="success-response"></a>Réponse en cas de réussite

**Propriétés** :

| Name | Type | Obligatoire | Description |
| :--- | :--- | :------- | :---------- |
| version | u64 | Oui | Version de l’état signalé après l’application du patch |

**Charge utile :** vide

#### <a name="alternative-responses"></a>Autres réponses

| État | Name | Description |
| :----- | :--- | :---------- |
| 0104 |  Échec de la précondition | La condition préalable n’a pas été remplie, ce qui entraîne l’annulation de la requête. |
| 0100 |  Demande incorrecte | Le message d’opération est incorrect et ne peut pas être traité. |
| 0101 |  Non autorisé | Le client n’est pas autorisé à effectuer l’opération. |
| 0102 |  Non autorisée | L’opération n’est pas autorisée. |
| 0501 |  Throttled | Le taux de requêtes est trop élevé par SKU. |
| 0502 |  Quota dépassé | Le quota quotidien par SKU actuel est dépassé. |
| 0601 |  Erreur de serveur | Erreur interne du serveur. |
| 0602 |  Délai d'expiration | L’opération a expiré avant d’être terminée. |
| 0603 |  Serveur occupé | Le serveur est occupé. |

#### <a name="pseudo-code-sample"></a>Exemple de pseudo-code

```

-> PUBLISH
    QoS: 0
    Topic: $iothub/twin/patch/reported
    [if-version: <u64>]
<- PUBLISH
    QoS: 0
    Topic: $iothub/responses

```

### <a name="receive-commands"></a>Recevoir des commandes

Recevoir et gérer des commandes

#### <a name="message"></a>Message

**Nom de la rubrique :** `$iothub/commands`

**Propriétés** :

| Name | Type | Obligatoire | Description |
| :--- | :--- | :------- | :---------- |
| sequence-no | u64 | Oui | Numéro de séquence du message. |
| enqueued-time | time | Oui | Horodatage du moment où le message est entré dans le système. |
| delivery-count | u32 | Oui | Nombre de tentatives de remise du message. |
| creation-time | time | Non | Horodatage du moment où le message a été créé (fourni par l’expéditeur). |
| message-id | string | non | Identité du message (fournie par l’expéditeur). |
| user-id | string | non | Identité de l’utilisateur (fournie par l’expéditeur). |
| correlation-id | string | non | Identité de la corrélation (fournie par l’expéditeur). |
| Type de contenu | string | non | Détermine le type de contenu de la charge utile. |
| content-encoding | string | non | Détermine l’encodage du contenu de la charge utile. |

**Charge utile :** n’importe quelle séquence d’octets

#### <a name="success-acknowledgment"></a>Accusé de réception en cas de réussite

Indique que la commande a été acceptée pour être traitée par le client

**Propriétés :** aucune

**Charge utile :** vide

#### <a name="alternative-acknowledgments"></a>Autres accusés de réception

| Code de raison | État | Name | Description |
| :---------- | :----- | :--- | :---------- |
| 131 | 0603 | Arrêter | Indique que la commande ne sera pas traitée pour l’instant et qu’elle devra être à nouveau remise à l’avenir. |
| 131 | 0100 | Rejeter | Indique que la commande est rejetée par le client et qu’elle ne doit pas être retentée. |

#### <a name="pseudo-code-sample"></a>Exemple de pseudo-code

```

-> SUBSCRIBE
    - Topic: $iothub/commands
      QoS: 1
<- PUBLISH
    QoS: 1
    Topic: $iothub/commands
    sequence-no: <u64>enqueued-time: <time>delivery-count: <u32>[creation-time: <time>][message-id: <string>][user-id: <string>][correlation-id: <string>][Content Type: <string>][content-encoding: <string>]
    Payload: ...

-> PUBACK

```

### <a name="receive-direct-methods"></a>Recevoir des méthodes directes

Recevoir et gérer les appels de méthode directe

#### <a name="request"></a>Requête

**Nom de la rubrique :** `$iothub/methods/{name}`

**Propriétés :** aucune

**Charge utile :** n’importe quelle séquence d’octets

#### <a name="success-response"></a>Réponse en cas de réussite

**Propriétés** :

| Name | Type | Obligatoire | Description |
| :--- | :--- | :------- | :---------- |
| response-code | u32 | Oui |  |

**Charge utile :** n’importe quelle séquence d’octets

#### <a name="alternative-responses"></a>Autres réponses

| État | Name | Description |
| :----- | :--- | :---------- |
| 06A0 |  Indisponible | Indique que le client n’est pas accessible via cette connexion. |

#### <a name="pseudo-code-sample"></a>Exemple de pseudo-code

```

-> SUBSCRIBE
    - Topic: methods/{name}
      QoS: 0
<- SUBACK
<- PUBLISH
    QoS: 0
    Topic: $iothub/methods/{name}
-> PUBLISH
    QoS: 0
    Topic: $iothub/responses

```

### <a name="receive-twin-desired-state-changes"></a>Recevoir les modifications d’état souhaité du jumeau

Recevoir les mises à jour de l’état souhaité du jumeau

#### <a name="message"></a>Message

**Nom de la rubrique :** `$iothub/twin/patch/desired`

**Propriétés** :

| Name | Type | Obligatoire | Description |
| :--- | :--- | :------- | :---------- |
| version | u64 | Oui | Version de l’état souhaité correspondant à cette mise à jour |

**Charge utile** : TwinState

#### <a name="pseudo-code-sample"></a>Exemple de pseudo-code

```

-> SUBSCRIBE
    - Topic: $iothub/twin/patch/desired
      QoS: 0
<- PUBLISH
    QoS: 0
    Topic: $iothub/twin/patch/desired
    version: <u64>
    Payload: ...

```

### <a name="send-telemetry"></a>Envoyer des données de télémétrie

Publier le message sur le canal de télémétrie (EventHubs par défaut ou un autre point de terminaison) via la configuration du routage.

#### <a name="message"></a>Message

**Nom de la rubrique :** `$iothub/telemetry`

**Propriétés** :

| Name | Type | Obligatoire | Description |
| :--- | :--- | :------- | :---------- |
| Type de contenu | string | non | se traduit en propriété système `content-type` sur le message publié |
| content-encoding | string | non | se traduit en propriété système `content-encoding` sur le message publié |
| message-id | string | non | se traduit en propriété système `message-id` sur le message publié |
| user-id | string | non | se traduit en propriété système `user-id` sur le message publié |
| correlation-id | string | non | se traduit en propriété système `correlation-id` sur le message publié |
| creation-time | time | Non | se traduit en propriété `iothub-creation-time-utc` sur le message publié |

**Charge utile :** n’importe quelle séquence d’octets

#### <a name="success-acknowledgment"></a>Accusé de réception en cas de réussite

Le message a été correctement publié sur le canal de télémétrie.

**Propriétés :** aucune

**Charge utile :** vide

#### <a name="alternative-acknowledgments"></a>Autres accusés de réception

| Code de raison | État | Name | Description |
| :---------- | :----- | :--- | :---------- |
| 131 | 0100 | Demande incorrecte | Le message d’opération est incorrect et ne peut pas être traité. |
| 135 | 0101 | Non autorisé | Le client n’est pas autorisé à effectuer l’opération. |
| 131 | 0102 | Non autorisée | L’opération n’est pas autorisée. |
| 131 | 0601 | Erreur de serveur | Erreur interne du serveur. |
| 151 | 0501 | Throttled | Le taux de requêtes est trop élevé par SKU. |
| 151 | 0502 | Quota dépassé | Le quota quotidien par SKU actuel est dépassé. |
| 131 | 0602 | Délai d'expiration | L’opération a expiré avant d’être terminée. |
| 131 | 0603 | Serveur occupé | Le serveur est occupé. |

#### <a name="pseudo-code-sample"></a>Exemple de pseudo-code

```
-> PUBLISH
    QoS: 1
    Topic: $iothub/telemetry
    [Content Type: <string>]
    [content-encoding: <string>]
    [message-id: <string>]
    [user-id: <string>]
    [correlation-id: <string>]
    [creation-time: <time>]

<- PUBACK

```

## <a name="responses"></a>Réponses

### <a name="bad-request"></a>Demande incorrecte

Le message d’opération est incorrect et ne peut pas être traité.

**Code motif :** `131`

**État :** `0100`

**Propriétés** :

| Name | Type | Obligatoire | Description |
| :--- | :--- | :------- | :---------- |
| reason | string | non | contient des informations spécifiques sur ce qui n’est pas valide dans le message |

**Charge utile :** vide

### <a name="conflict"></a>Conflit

L’opération est en conflit avec une autre opération en cours.

**Code motif :** `131`

**État :** `0103`

**Propriétés** :

| Name | Type | Obligatoire | Description |
| :--- | :--- | :------- | :---------- |
| trace-id | string | non | ID de trace de la corrélation avec des diagnostics supplémentaires pour l’erreur |
| reason | string | non | contient des informations spécifiques sur ce qui n’est pas valide dans le message |

**Charge utile :** vide

### <a name="not-allowed"></a>Non autorisée

L’opération n’est pas autorisée.

**Code motif :** `131`

**État :** `0102`

**Propriétés** :

| Name | Type | Obligatoire | Description |
| :--- | :--- | :------- | :---------- |
| reason | string | non | contient des informations spécifiques sur ce qui n’est pas valide dans le message |

**Charge utile :** vide

### <a name="not-authorized"></a>Non autorisé

Le client n’est pas autorisé à effectuer l’opération.

**Code motif :** `135`

**État :** `0101`

**Propriétés** :

| Name | Type | Obligatoire | Description |
| :--- | :--- | :------- | :---------- |
| trace-id | string | non | ID de trace de la corrélation avec des diagnostics supplémentaires pour l’erreur |

**Charge utile :** vide

### <a name="not-found"></a>Introuvable

La ressource demandée n’existe pas.

**Code motif :** `131`

**État :** `0504`

**Propriétés** :

| Name | Type | Obligatoire | Description |
| :--- | :--- | :------- | :---------- |
| reason | string | non | contient des informations spécifiques sur ce qui n’est pas valide dans le message |

**Charge utile :** vide

### <a name="not-modified"></a>Non modifié

La ressource n’a pas été modifiée selon la condition préalable fournie.

**Code motif :** `0`

**État :** `0001`

**Propriétés :** aucune

**Charge utile :** vide

### <a name="precondition-failed"></a>Échec de la précondition

La condition préalable n’a pas été remplie, ce qui entraîne l’annulation de la requête.

**Code motif :** `131`

**État :** `0104`

**Propriétés :** aucune

**Charge utile :** vide

### <a name="quota-exceeded"></a>Quota dépassé

Le quota quotidien par SKU actuel est dépassé.

**Code motif :** `151`

**État :** `0502`

**Propriétés :** aucune

**Charge utile :** vide

### <a name="resource-exhausted"></a>Ressource épuisée

La ressource n’a pas la capacité nécessaire pour terminer l’opération.

**Code motif :** `131`

**État :** `0503`

**Propriétés** :

| Name | Type | Obligatoire | Description |
| :--- | :--- | :------- | :---------- |
| reason | string | non | contient des informations spécifiques sur ce qui n’est pas valide dans le message |

**Charge utile :** vide

### <a name="server-busy"></a>Serveur occupé

Le serveur est occupé.

**Code motif :** `131`

**État :** `0603`

**Propriétés** :

| Name | Type | Obligatoire | Description |
| :--- | :--- | :------- | :---------- |
| trace-id | string | non | ID de trace de la corrélation avec des diagnostics supplémentaires pour l’erreur |

**Charge utile :** vide

### <a name="server-error"></a>Erreur de serveur

Erreur interne du serveur.

**Code motif :** `131`

**État :** `0601`

**Propriétés** :

| Name | Type | Obligatoire | Description |
| :--- | :--- | :------- | :---------- |
| trace-id | string | non | ID de trace de la corrélation avec des diagnostics supplémentaires pour l’erreur |

**Charge utile :** vide

### <a name="target-failed"></a>Échec de la cible

La cible a répondu, mais la réponse est non valide ou malformée.

**Code motif :** `131`

**État :** `06A2`

**Propriétés** :

| Name | Type | Obligatoire | Description |
| :--- | :--- | :------- | :---------- |
| reason | string | non | contient des informations spécifiques sur ce qui n’est pas valide dans le message |

**Charge utile :** vide

### <a name="target-timeout"></a>Expiration du délai d’attente du ticket

Expiration du délai d’attente dont dispose la cible pour terminer la requête

**Code motif :** `131`

**État :** `06A1`

**Propriétés** :

| Name | Type | Obligatoire | Description |
| :--- | :--- | :------- | :---------- |
| trace-id | string | non | ID de trace de la corrélation avec des diagnostics supplémentaires pour l’erreur |
| reason | string | non | contient des informations spécifiques sur ce qui n’est pas valide dans le message |

**Charge utile :** vide

### <a name="target-unavailable"></a>Cible non disponible

La cible est inaccessible pour terminer la requête.

**Code motif :** `131`

**État :** `06A0`

**Propriétés :** aucune

**Charge utile :** vide

### <a name="throttled"></a>Throttled

Le taux de requêtes est trop élevé par SKU.

**Code motif :** `151`

**État :** `0501`

**Propriétés :** aucune

**Charge utile :** vide

### <a name="timeout"></a>Délai d'expiration

L’opération a expiré avant d’être terminée.

**Code motif :** `131`

**État :** `0602`

**Propriétés** :

| Name | Type | Obligatoire | Description |
| :--- | :--- | :------- | :---------- |
| trace-id | string | non | ID de trace de la corrélation avec des diagnostics supplémentaires pour l’erreur |

**Charge utile :** vide


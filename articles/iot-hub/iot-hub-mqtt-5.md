---
title: Prise en charge de MQTT 5 d’Azure IoT Hub (préversion)
description: En savoir plus sur la prise en charge de MQTT 5 d’Azure IoT Hub
services: iot-hub
author: jlian
ms.service: iot-fundamentals
ms.topic: conceptual
ms.date: 11/19/2020
ms.author: jlian
ms.openlocfilehash: fb2cc0b81083936a67bcd465e0408b9f4b53996b
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96602993"
---
# <a name="iot-hub-mqtt-5-support-overview-preview"></a>Vue d’ensemble de la prise en charge de MQTT 5 d’Azure IoT Hub (préversion)

**Version** : 2.0 **api-version :** 2020-10-01-preview

Ce document définit l’API du plan de données IoT Hub via le protocole MQTT version 5.0. Pour accéder aux définitions complètes de cette API, consultez [Informations de référence de l’API](iot-hub-mqtt-5-reference.md).

## <a name="prerequisites"></a>Prérequis

- [Activez le mode aperçu](iot-hub-preview-mode.md) sur un tout nouveau hub IoT pour essayer MQTT 5.
- Une connaissance préalable de la [spécification MQTT 5](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html) est requise.

## <a name="level-of-support-and-limitations"></a>Niveau de prise en charge et limitations

La prise en charge de MQTT 5 par IoT Hub est disponible en préversion et limitée de la façon suivante (communiquée au client via les propriétés `CONNACK`, sauf indication contraire) :

- Le [Kit de développement logiciel (SDK) d’appareil Azure IoT Hub](iot-hub-devguide-sdks.md) n’est encore prise en charge officiellement.
- Les identificateurs d’abonnement ne sont pas pris en charge.
- Les abonnements partagés ne sont pas pris en charge.
- `RETAIN` n’est pas pris en charge.
- `Maximum QoS` a la valeur `1`.
- `Maximum Packet Size` est `256 KiB` (sous réserve de restrictions supplémentaires par opération).
- Les ID client attribués ne sont pas pris en charge.
- `Keep Alive` est limité à `19 min` (délai maximal pour la vérification de l’activité : `28.5 min`).
- `Topic Alias Maximum` a la valeur `10`.
- `Response Information` n’est pas pris en charge ; `CONNACK` ne retourne pas la propriété `Response Information` même si `CONNECT` contient la propriété `Request Response Information`.
- `Receive Maximum` (nombre maximal de paquets `PUBLISH` sans accusé de réception en attente autorisés [dans la direction client-serveur] avec `QoS: 1`) est de `16`.
- Un client unique ne peut pas avoir plus de `50` abonnements.
  Lorsque la limite est atteinte, `SUBACK` retourne le code motif `0x97` (Quota dépassé) pour les abonnements.

## <a name="connection-lifecycle"></a>Cycle de vie de la connexion

### <a name="connection"></a>Connexion

Pour connecter un client à IoT Hub à l’aide de cette API, établissez la connexion par spécification MQTT 5.
Le client doit envoyer le paquet `CONNECT` dans les 30 secondes suivant l’établissement réussie d’une liaison TLS, ou le serveur ferme la connexion.
Voici un exemple de paquet `CONNECT` :

```yaml
-> CONNECT
    Protocol_Version: 5
    Clean_Start: 0
    Client_Id: D1
    Authentication_Method: SAS
    Authentication_Data: {SAS bytes}
    api-version: 2020-10-10
    host: abc.azure-devices.net
    sas-at: 1600987795320
    sas-expiry: 1600987195320
    client-agent: artisan;Linux
```

- La propriété `Authentication Method` est obligatoire et identifie la méthode d’authentification utilisée. Pour plus d’informations sur la méthode d’authentification, consultez [Authentification](#authentication).
- La gestion de la propriété `Authentication Data` dépend de `Authentication Method`. Si `Authentication Method` est défini sur `SAS`, `Authentication Data` est obligatoire et doit contenir une signature valide. Pour plus d’informations sur les données d’authentification, consultez [Authentification](#authentication).
- La propriété `api-version` est obligatoire et doit être définie sur la valeur de version de l’API fournie dans l’en-tête de cette spécification pour que celle-ci s’applique.
- La propriété `host` définit le nom d’hôte du locataire. Elle est requise, sauf si l’extension SNI a été présentée dans l’enregistrement Client Hello au cours de l’établissement d’une liaison TLS.
- `sas-at` définit l’heure de la connexion.
- `sas-expiry` définit l’heure d’expiration de la SAP fournie.
- `client-agent` communique éventuellement des informations sur le client qui crée la connexion.

> [!NOTE]
> `Authentication Method` et d’autres propriétés de la spécification avec des noms en majuscules sont des propriétés de première classe dans MQTT 5 : elles sont décrites en détail dans la spécification MQTT 5. `api-version` et d’autres propriétés avec des tirets sont des propriétés utilisateur spécifiques à l’API IoT Hub.

IoT Hub répond avec le paquet `CONNACK` une fois qu’il a terminé l’authentification et la récupération de données pour prendre en charge la connexion. Si la connexion est établie, `CONNACK` ressemble à ceci :

```yaml
<- CONNACK
    Session_Present: 1
    Reason_Code: 0x00
    Session_Expiry_Interval: 0xFFFFFFFF # included only if CONNECT specified value less than 0xFFFFFFFF and more than 0x00
    Receive_Maximum: 16
    Maximum_QoS: 1
    Retain_Available: 0
    Maximum_Packet_Size: 262144
    Topic_Alias_Maximum: 10
    Subscription_Identifiers_Available: 0
    Shared_Subscriptions_Available: 0
    Server_Keep_Alive: 1140 # included only if client did not specify Keep Alive or if it specified a bigger value
```

Ces propriétés de paquet `CONNACK` respectent la [spécification MQTT 5](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Toc3901080). Elles reflètent les capacités d’IoT Hub.

### <a name="authentication"></a>Authentification

La propriété `Authentication Method` sur le client `CONNECT` définit le type d’authentification qu’il utilise pour cette connexion :

- `SAS` : La signature d’accès partagé est fournie dans la propriété `Authentication Data` de `CONNECT`.
- `X509` : Le client s’appuie sur l’authentification par certificat client.

 L’authentification échoue si la méthode d’authentification ne correspond pas à la méthode configurée du client dans IoT Hub.

> [!NOTE]
> Cette API nécessite que la propriété `Authentication Method` soit définie dans le paquet `CONNECT`. Si la propriété `Authentication Method` n’est pas fournie, la connexion échoue avec la réponse `Bad Request`.

L’authentification par nom d’utilisateur/mot de passe utilisée dans les versions précédentes de l’API n’est pas prise en charge.

#### <a name="sas"></a>SAS

Avec l’authentification par signature d’accès partagé, le client doit fournir la signature du contexte de connexion. Cela prouve l’authenticité de la connexion MQTT. La signature doit être basée sur l’une des deux clés d’authentification de la configuration du client dans IoT Hub ou sur l’une des deux clés d’accès partagé d’une [stratégie d’accès partagé](iot-hub-devguide-security.md).

La chaîne à signer doit être formée comme suit :

```text
{host name}\n
{Client Id}\n
{sas-policy}\n
{sas-at}\n
{sas-expiry}\n
```

- `host name` est dérivé de l’extension SNI (présentée par le client dans l’enregistrement Client Hello au cours de l’établissement d’une liaison TLS) ou de la propriété utilisateur `host` dans le paquet `CONNECT`.
- `Client Id` est l’identificateur client dans le paquet `CONNECT`.
- `sas-policy` : Le cas échéant, définit la stratégie d’accès IoT Hub utilisée pour l’authentification. Elle est encodée en tant que propriété utilisateur sur le paquet `CONNECT`. Facultatif : L’omettre signifie que les paramètres d’authentification du registre de l’appareil seront utilisés à la place.
- `sas-at` : Le cas échéant, spécifie l’heure de la connexion (heure actuelle). Elle est encodée en tant que propriété utilisateur de type `time` sur le paquet `CONNECT`.
- `sas-expiry` définit l’heure d’expiration de l’authentification. Il s’agit d’une propriété utilisateur de type `time` sur le paquet `CONNECT`. Cette propriété est obligatoire.

Pour les paramètres facultatifs, en cas d’omission, une chaîne vide DOIT être utilisée à la place dans la chaîne à signer.

HMAC-SHA256 est utilisé pour hacher la chaîne en fonction de l’une des clés symétriques de l’appareil. La valeur de hachage est ensuite définie en tant que valeur de la propriété `Authentication Data`.

#### <a name="x509"></a>X509

Si la propriété `Authentication Method` est définie sur `X509`, IoT Hub authentifie la connexion sur la base du certificat client fourni.

#### <a name="reauthentication"></a>Réauthentification

Si vous utilisez l’authentification par signature d’accès partagé, nous vous recommandons d’utiliser des jetons d’authentification de courte durée. Pour maintenir l’authentification de la connexion et éviter une déconnexion pour cause d’expiration, le client doit se réauthentifier en envoyant le paquet `AUTH` avec `Reason Code: 0x19` (Réauthentification) :

```yaml
-> AUTH
    Reason_Code: 0x19
    Authentication_Method: SAS
    Authentication_Data: {SAS bytes}
    sas-at: {current time}
    sas-expiry: {SAS expiry time}
```

Règles :

- `Authentication Method` doit être identique à celle utilisée pour l’authentification initiale.
- Si la connexion a été authentifiée à l’origine à l’aide d’une signature d’accès partagé basée sur la stratégie d’accès partagé, la signature utilisée dans la réauthentification doit être basée sur la même stratégie.

Si la réauthentification réussit, IoT Hub envoie le paquet `AUTH` avec `Reason Code: 0x00` (Réussite). Dans le cas contraire, IoT Hub envoie le paquet `DISCONNECT` avec `Reason Code: 0x87` (Non autorisé) et ferme la connexion.

### <a name="disconnection"></a>Déconnexion

Le serveur peut déconnecter le client pour plusieurs raisons :

- Le client se comporte d’une manière à laquelle il est impossible de répondre directement par un accusé de réception (ou une réponse) négatif.
- Le serveur ne parvient pas à maintenir à jour l’état de la connexion.
- Un client avec la même identité s’est connecté.

Le serveur peut se déconnecter avec n’importe quel code motif défini dans la spécification MQTT 5.0. Mentions notables :

- `135` (Non autorisé) en cas d’échec de la réauthentification, d’expiration du jeton SAP actuel ou de modification des informations d’identification de l’appareil.
- `142` (Session reprise) quand une nouvelle connexion avec la même identité de client a été ouverte.
- `159` (Taux de connexion dépassé) lorsque le taux de connexion pour le hub IoT est dépassé.  
- `131` (Erreur spécifique à l’implémentation) est utilisé pour toutes les erreurs personnalisées définies dans cette API. Les propriétés `status` et `reason` sont utilisées pour communiquer des détails supplémentaires sur la cause de la déconnexion (pour plus d’informations, consultez [Réponse](#response)).

## <a name="operations"></a>Operations

Toutes les fonctionnalités de cette API sont exprimées en tant qu’opérations. Voici un exemple d’opération Send Telemetry :

```yaml
-> PUBLISH
    QoS: 1
    Packet_Id: 3
    Topic: $iothub/telemetry
    Payload: Hello

<- PUBACK
    Packet_Id: 3
    Reason_Code: 0
```

Pour une spécification complète des opérations de cette API, consultez [Informations de référence sur l’API](iot-hub-mqtt-5-reference.md).

> [!NOTE]
> Tous les exemples de cette spécification sont présentés du point de vue du client. Le signe `->` désigne l’envoi du paquet par le client et `<-` sa réception par le client.

### <a name="message-topics-and-subscriptions"></a>Rubriques de messages et abonnements

Les rubriques utilisées dans les messages des opérations de cette API commencent par `$iothub/`.
La sémantique du répartiteur MQTT ne s’appliquent pas à ces opérations (pour plus d’informations, consultez « [Rubriques commençant par \$](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Toc3901246) »).
Les rubriques qui commencent par `$iothub/` et qui ne sont pas définies dans cette API ne sont pas prises en charge :

- L’envoi de messages à une rubrique non définie donne lieu à une réponse `Not Found` (pour plus d’informations, consultez [Réponse](#response)).
- L’abonnement à une rubrique non définie donne lieu à `SUBACK` avec `Reason Code: 0x8F` (Filtre de rubrique non valide).

Les noms de rubriques et les noms de propriété respectent la casse et doivent correspondre exactement. Par exemple, `$iothub/telemetry/` n’est pas pris en charge, tandis que `$iothub/telemetry` l’est.

> [!NOTE]
> Les caractères génériques dans les abonnements sous `$iothub/..` ne sont pas pris en charge. Autrement dit, un client ne peut pas s’abonner à `$iothub/+` ou `$iothub/#`. Si vous tentez de le faire, cela donne lieu à `SUBACK` avec `Reason Code: 0xA2` (Abonnements avec caractères génériques non pris en charge). Seuls les caractères génériques à un seul segment (`+`) sont pris en charge à la place des paramètres de chemin d’accès dans le nom de rubrique pour les opérations qui en ont.

### <a name="interaction-types"></a>Types d’interaction

Toutes les opérations de cette API sont basées sur l’un des deux types d’interaction :

- Message avec accusé de réception facultatif (MessageAck)
- Demande-réponse (ReqRep)

Les opérations varient également selon la direction (déterminée par la direction du message initial échangé) :

- Client à serveur (C2S)
- Serveur à client (S2C)

Par exemple, Send Telemetry est une opération client à serveur de type « Message avec accusé de réception », tandis que Handle Direct Method est une opération serveur à client de type Demande-réponse.

#### <a name="message-acknowledgement-interactions"></a>Interactions entre messages et accusés de réception

L’interaction d’un message avec un accusé de réception facultatif (MessageAck) est exprimée sous la forme d’un échange de paquets `PUBLISH` et `PUBACK` dans MQTT. L’accusé de réception est facultatif et l’expéditeur peut choisir de ne pas le demander en envoyant le paquet `PUBLISH` avec `QoS: 0`.

> [!NOTE]
> Si les propriétés du paquet `PUBACK` doivent être tronquées en raison du paramètre `Maximum Packet Size` déclaré par le client, IoT Hub conservera autant de propriétés utilisateur que possible dans la limite donnée. Les propriétés utilisateur énumérées en premier ont plus de chance d’être envoyées que celles indiquées plus tard. La propriété `Reason String` a la priorité la plus faible.

##### <a name="example-of-simple-messageack-interaction"></a>Exemple d’interaction MessageAck simple

Message :

```yaml
PUBLISH
    QoS: 1
    Packet_Id: 34
    Topic: $iothub/{request.path}
    Payload: <any>
```

Accusé de réception (réussite) :

```yaml
PUBACK
    Packet_Id: 34
    Reason_Code: 0
```

#### <a name="request-response-interactions"></a>Interactions Demande-réponse

Dans les interactions Demande-réponse (ReqRep), la demande et la réponse se traduisent toutes deux par des paquets `PUBLISH` avec `QoS: 0`.

La propriété `Correlation Data` doit être définie dans les deux et est utilisée pour faire correspondre le paquet Réponse au paquet Demande.

Cette API utilise une rubrique à réponse unique `$iothub/responses` pour toutes les opérations ReqRep. Il n’est pas nécessaire de s’abonner/se désabonner de cette rubrique pour les opérations client à serveur : le serveur suppose que tous les clients sont abonnés.

##### <a name="example-of-simple-reqrep-interaction"></a>Exemple d’interaction ReqRep simple

Demande :

```yaml
PUBLISH
    QoS: 0
    Topic: $iothub/{request.path}
    Correlation_Data: 0x01 0xFA
    Payload: ...
```

Réponse (réussite) :

```yaml
PUBLISH
    QoS: 0
    Topic: $iothub/responses
    Correlation_Data: 0x01 0xFA
    Payload: ...
```

Les interactions ReqRep ne prennent pas en charge les paquets `PUBLISH` avec `QoS: 1` en tant que messages de demande ou de réponse. L’envoi de la demande `PUBLISH` donne lieu à la réponse `Bad Request`.

La longueur maximale prise en charge dans la propriété `Correlation Data` est de 16 octets. Si la propriété `Correlation Data` sur le paquet `PUBLISH` est définie sur une valeur supérieure à 16 octets, IoT Hub envoie le résultat `DISCONNECT` avec `Bad Request` et ferme la connexion. Ce comportement s’applique uniquement aux paquets échangés au sein de cette API.

> [!NOTE]
> Les données de corrélation sont une séquence arbitraire d’octets, c’est-à-dire qu’il n’est pas garanti que ce soit une chaîne UTF-8.
>
> ReqRep utilise des rubriques prédéfinies pour la réponse ; la propriété Response Topic dans le paquet `PUBLISH` de la demande (si elle est définie par l’expéditeur) est ignorée.

IoT Hub abonne automatiquement le client aux rubriques de réponse pour toutes les opérations ReqRep client à serveur. Même si le client se désinscrit de manière explicite de la rubrique de réponse, IoT Hub rétablit l’abonnement automatiquement. Pour les interactions ReqRep serveur à client, il est toujours nécessaire que l’appareil s’abonne.

### <a name="message-properties"></a>Propriétés de message

Les propriétés d’opération, définies par le système ou par l’utilisateur, sont exprimées en tant que propriétés de paquet dans MQTT 5.

Les noms des propriétés utilisateur respectent la casse et doivent être orthographiés exactement comme ils ont été définis. Par exemple, `Trace-ID` n’est pas pris en charge, tandis que `trace-id` l’est.

Les demandes dont les propriétés utilisateur sont hors spécification et sans préfixe `@` aboutissent à une erreur.

Les propriétés système sont encodées en tant que propriétés de première classe (par exemple, `Content Type`) ou en tant que propriétés utilisateur. La spécification fournit une liste exhaustive des propriétés système prises en charge.
Toutes les propriétés de première classe sont ignorées, sauf si leur prise en charge est explicitement indiquée dans la spécification.

Lorsque les propriétés définies par l’utilisateur sont autorisées, leurs noms doivent respecter le format `@{property name}`. Les propriétés définies par l’utilisateur ne prennent en charge que les valeurs de chaîne UTF-8 valides. Par exemple, la propriété `MyProperty1` avec la valeur `15` doit être encodée en tant que propriété utilisateur avec le nom `@MyProperty` et la valeur `15`.

Si IoT Hub ne reconnaît pas la propriété utilisateur, cela est considéré comme une erreur et IoT Hub répond par `PUBACK` avec `Reason Code: 0x83` (Erreur spécifique à l’implémentation) et `status: 0100` (Demande incorrecte). Si l’accusé de réception n’a pas été demandé (QoS: 0), le paquet `DISCONNECT` avec la même erreur est renvoyé et la connexion est arrêtée.

Cette API définit les types de données suivants en plus de `string` :

- `time` : nombre de millisecondes écoulées depuis `1970-01-01T00:00:00.000Z`. P. ex. : `1600987195320` pour `2020-09-24T22:39:55.320Z`.
- `u32` : nombre entier 32 bits non signé.
- `u64` : nombre entier 64 bits non signé.
- `i32` : nombre entier 32 bits signé.

### <a name="response"></a>response

Les interactions peuvent donner lieu à des résultats différents : `Success`, `Bad Request`, `Not Found` et autres.
Les résultats se distinguent les uns des autres par la propriété utilisateur `status`. La signification de `Reason Code` dans les paquets `PUBACK` (pour les interactions MessageAck) correspond à `status` dans la mesure du possible.

> [!NOTE]
> Si le client spécifie `Request Problem Information: 0` dans le paquet CONNECT, aucune propriété utilisateur n’est envoyée sur les paquets `PUBACK` pour se conformer à la spécification MQTT 5, notamment la propriété `status`. Dans ce cas, le client peut continuer à s’appuyer sur `Reason Code` pour déterminer si l’accusé de réception est positif ou négatif. 

Chaque interaction a une valeur par défaut (ou réussite). Elle a un `Reason Code` de `0` et une propriété `status` « non définie ». Sinon :

- Pour les interactions MessageAck, `PUBACK` obtient un `Reason Code` autre que 0x0 (Réussite). La propriété `status` peut être présente pour clarifier le résultat.
- Pour les interactions ReqRep, `PUBLISH` de la réponse obtient la propriété `status` définie.
- Étant donné qu’il n’existe aucun moyen de répondre directement aux interactions MessageAck avec `QoS: 0`, le paquet `DISCONNECT` est envoyé à la place avec les informations de réponse, puis se déconnecte.

Exemples :

Bad Request (MessageAck) :

```yaml
PUBACK
    Reason_Code: 131
    status: 0100
    reason: Unknown property `test`
```

Not Authorized (MessageAck) :

```yaml
PUBACK
    Reason_Code: 135
    status: 0101
```

Not Authorized (ReqRep) :

```yaml
PUBLISH
    QoS: 0
    Topic: $iothub/responses
    Correlation_Data: ...
    status: 0101
```

Le cas échéant, IoT Hub définit les propriétés utilisateur suivantes :

- `status` : code étendu d’IoT Hub pour l’état de l’opération. Ce code peut être utilisé pour différencier les résultats.
- `trace-id` : ID de trace pour l’opération ; IoT Hub peut conserver des diagnostics supplémentaires concernant l’opération qui pourraient être utilisés pour une investigation interne.
- `reason` : message lisible par l’homme fournissant des informations supplémentaires sur la raison pour laquelle l’opération s’est terminée dans un état indiqué par la propriété `status`.

> [!NOTE]
> Si le client définit la propriété `Maximum Packet Size` dans le paquet CONNECT sur une valeur très faible, toutes les propriétés utilisateur peuvent ne pas correspondre et n’apparaîtront pas dans le paquet.
> 
> `reason` est uniquement destiné à des personnes et ne doit pas être utilisé dans la logique du client. Cette API permet de modifier les messages à tout moment sans avertissement ou changement de version.
>
> Si le client envoie `RequestProblemInformation: 0` dans le paquet CONNECT, les propriétés utilisateur ne seront pas incluses dans les accusés de réception conformément à la [spécification MQTT 5](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Toc3901053).

#### <a name="status-code"></a>Code d’état

La propriété `status` contient le code d’état de l’opération. Elle est optimisée pour l’efficacité de la lecture par ordinateur.
Elle se compose d’un entier non signé de deux octets encodé au format hexadécimal dans une chaîne comme `0501`.
Structure du code (mappage de bits) :

```text
7 6 5 4 3 2 1 0 | 7 6 5 4 3 2 1 0
0 0 0 0 0 R T T | C C C C C C C C
```

Le premier octet est utilisé pour les indicateurs :

- Les bits 0 et 1 indiquent le type de résultat :
  - `00` : réussite
  - `01` : erreur client
  - `10` : erreur serveur
- Le bit 2 : `1` indique que l’erreur est renouvelable
- Les bits 3 à 7 sont réservés et doivent avoir la valeur `0`

Le deuxième octet contient un code de réponse réel et distinct. Les codes d’erreur avec différents indicateurs peuvent avoir la même valeur de deuxième octet. Par exemple, il peut y avoir des codes d’erreur `0001`, `0101`, `0201` et `0301` ayant une signification distincte.

Par exemple, `Too Many Requests` est une erreur client renouvelable avec son propre code de `1`. Sa valeur est `0000 0101 0000 0001` ou `0x0501`.

Les clients peuvent utiliser des bits de type pour déterminer si l’opération s’est correctement terminée. Les clients peuvent également utiliser le bit renouvelable pour décider s’il est judicieux de retenter l’opération.

## <a name="recommendations"></a>Recommandations

### <a name="session-management"></a>Gestion des sessions

Le paquet `CONNACK` porte la propriété `Session Present` pour indiquer si le serveur a restauré une session créée précédemment. Utilisez cette propriété pour déterminer s’il faut s’abonner à des rubriques ou ignorer cette étape, car l’abonnement a été effectué précédemment.

Pour s’appuyer sur `Session Present`, le client doit effectuer le suivi des abonnements qu’il a effectués (c’est-à-dire, qu’il a envoyé le paquet `SUBSCRIBE` et reçu `SUBACK` avec un code motif réussi), ou s’assurer de s’abonner à toutes les rubriques d’un échange `SUBSCRIBE`/`SUBACK`. Dans le cas contraire, si le client envoie deux paquets `SUBSCRIBE` et qu’un seul d’entre eux est traité correctement par le serveur, le serveur communiquera `Session Present: 1` dans `CONNACK` tout en n’acceptant qu’une partie des abonnements du client.

Pour éviter le cas où une version antérieure du client ne se serait pas abonnée à toutes les rubriques, il est préférable de s’abonner de manière inconditionnelle lorsque le comportement du client change (par exemple, dans le cadre de la mise à jour du microprogramme). En outre, pour s’assurer qu’aucun abonnement obsolète n’est conservé (sachant qu’il existe un nombre maximal d’abonnements autorisés), le client dois se désinscrire de manière explicite des abonnements qui ne sont plus utilisés.

### <a name="batching"></a>Traitement par lot

Il n’existe pas de format spécial pour envoyer un lot de messages. Pour réduire les frais d’opérations gourmandes en ressources du protocole TLS et de la mise en réseau, regroupez les paquets (`PUBLISH`, `PUBACK`, `SUBSCRIBE`, etc.) avant de les transmettre à la pile TLS/TCP sous-jacente. En outre, le client peut faciliter l’utilisation de l’alias de rubrique dans le « lot » :

- Placez le nom complet de la rubrique dans le premier paquet `PUBLISH` pour la connexion et associez-lui un alias de rubrique.
- Placez les paquets suivants dans la même rubrique avec un nom de rubrique et une propriété d’alias de rubrique vides.

## <a name="migration"></a>Migration

Cette section répertorie les modifications apportées à l’API par rapport à [l’API MQTT précédente](iot-hub-mqtt-support.md).

- Le protocole de transport est MQTT 5. (Précédemment, MQTT 3.1.1.)
- Les informations de contexte pour l’authentification par signature d’accès partagé sont contenues dans le paquet `CONNECT` directement au lieu d’être encodées avec la signature.
- La propriété Authentication Method est utilisée pour indiquer la méthode d’authentification utilisée.
- La signature d’accès partagé est placée dans la propriété Authentication Data. Auparavant, le champ de mot de passe était utilisé.
- Les rubriques relatives aux opérations sont différentes :
  - Télémétrie : `$iothub/telemetry` au lieu de `devices/{Client Id}/messages/events`.
  - Commandes : `$iothub/commands` au lieu de `devices/{Client Id}/messages/devicebound`.
  - Jumeau patch signalé : `$iothub/twin/patch/reported` au lieu de `$iothub/twin/PATCH/properties/reported`.
  - Informer du changement d’état souhaité du jumeau : `$iothub/twin/patch/desired` au lieu de `$iothub/twin/PATCH/properties/desired`.
- Il n’est pas nécessaire de s’abonner à la rubrique des opérations de demande-réponse client à serveur.
- Des propriétés utilisateur sont utilisées au lieu d’encoder les propriétés dans le segment du nom de rubrique.
- Les noms des propriétés sont orthographiés selon la convention d’affectation de noms « Dash-case » au lieu d’abréviations avec préfixe spécial. Les propriétés définies par l’utilisateur nécessitent désormais un préfixe à la place. Par exemple, `$.mid` est désormais `message-id`, tandis que `myProperty1` devient `@myProperty1`.
- La propriété Correlation Data est utilisée pour mettre en corrélation les messages de demande et de réponse pour les opérations de demande-réponse au lieu de la propriété `$rid` encodée dans la rubrique.
- La propriété `iothub-connection-auth-method` n’est plus estampillée sur les événements de télémétrie.
- Les commandes C2D ne seront pas vidées en l’absence d’abonnement de l’appareil. Elles resteront dans la file d’attente jusqu’à ce que l’appareil s’abonne ou qu’elles expirent.

## <a name="examples"></a>Exemples

### <a name="send-telemetry"></a>Envoyer des données de télémétrie

Message :

```yaml
-> PUBLISH
    QoS: 1
    Packet_Id: 31
    Topic: $iothub/telemetry
    @myProperty1: My String Value # optional
    creation-time: 1600987195320 # optional
    @ No_Rules-ForUser-PROPERTIES: Any UTF-8 string value # optional
    Payload: <data>
```

Accusé de réception :

```yaml
<- PUBACK
    Packet_Id: 31
    Reason_Code: 0
```

Autre accusé de réception (limité) :

```yaml
<- PUBACK
    Packet_Id: 31
    Reason_Code: 151
    status: 0501
```

---

### <a name="send-get-twins-state"></a>Envoyer l’état de Get Twin

Demande :

```yaml
-> PUBLISH
    QoS: 0
    Topic: $iothub/twin/get
    Correlation_Data: 0x01 0xFA
    Payload: <empty>
```

Réponse (réussite) :

```yaml
<- PUBLISH
    QoS: 0
    Topic: $iothub/responses
    Correlation_Data: 0x01 0xFA
    Payload: <twin/desired state>
```

Réponse (non autorisé) :

```yaml
<- PUBLISH
    QoS: 0
    Topic: $iothub/responses
    Correlation_Data: 0x01 0xFA
    status: 0102
    reason: Operation not allowed for `B2` SKU
    Payload: <empty>
```

---

### <a name="handle-direct-method-call"></a>Gérer l’appel de méthode directe

Demande :

```yaml
<- PUBLISH
    QoS: 0
    Topic: $iothub/methods/abc
    Correlation_Data: 0x0A 0x10
    Payload: <data>
```

Réponse (réussite) :

```yaml
-> PUBLISH
    QoS: 0
    Topic: $iothub/responses
    Correlation_Data: 0x0A 0x10
    response-code: 200 # user defined response code
    Payload: <data>
```

> [!NOTE]
> `status` n’est pas défini. Il s’agit d’une réponse de réussite.

Réponse non disponible de l’appareil :

```yaml
-> PUBLISH
    QoS: 0
    Topic: $iothub/responses
    Correlation_Data: 0x0A 0x10
    status: 0603
```

---

### <a name="error-while-using-qos-0-part-1"></a>Erreur lors de l’utilisation de QoS 0, partie 1

Demande :

```yaml
-> PUBLISH
    QoS: 0
    Topic: $iothub/twin/gett # misspelled topic name - server won't recognize it as Request-Response interaction
    Correlation_Data: 0x0A 0x10
    Payload: <data>
```

Réponse :

```yaml
<- DISCONNECT
    Reason_Code: 144
    reason: "Unsupported topic: `$iothub/twin/gett`"
```

---

### <a name="error-while-using-qos-0-part-2"></a>Erreur lors de l’utilisation de QoS 0, partie 2

Demande :

```yaml
-> PUBLISH # missing Correlation Data
    QoS: 0
    Topic: $iothub/twin/get
    Payload: <data>
```

Réponse :

```yaml
<- DISCONNECT
    Reason_Code: 131
    status: 0100
    reason: "`Correlation Data` property is missing"
```
## <a name="next-steps"></a>Étapes suivantes

- Pour accéder aux informations de référence sur l’API MQTT 5 en préversion, consultez [Informations de référence sur l’API MQTT 5 du plan de données IoT Hub](iot-hub-mqtt-5-reference.md).
- Pour suivre un exemple C#, consultez [Exemple de référentiel GitHub](https://github.com/Azure-Samples/iot-hub-mqtt-5-preview-samples-csharp).
---
title: API de service de mesure – Place de marché commerciale de Microsoft
description: L’API d’événement d’utilisation vous permet d’émettre des événements d’utilisation pour les offres SaaS dans Microsoft AppSource et la Place de marché Azure.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 05/26/2020
author: mingshen-ms
ms.author: mingshen
ms.openlocfilehash: b75964f8cfc41efc35858284dbffded3aa406eb6
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96436060"
---
# <a name="marketplace-metered-billing-apis"></a>API de facturation à la consommation de la Place de marché

Les API de facturation à la consommation doivent être utilisées lorsque le serveur de publication crée des dimensions de contrôle personnalisées pour une offre à publier dans l’Espace partenaires. L’intégration avec les API de facturation à la consommation est requise pour toute offre achetée ayant un ou plusieurs plans avec des dimensions personnalisées pour émettre des événements d’utilisation.

Pour plus d’informations sur la création de dimensions de contrôle personnalisées pour les SaaS, consultez [Facturation à la consommation pour SaaS](saas-metered-billing.md).

Pour plus d’informations sur la création de dimensions de contrôle personnalisées pour une offre Azure Application avec un plan d’application géré, consultez [Configuration des informations de configuration d’une offre Azure Application](../create-new-azure-apps-offer.md#configure-your-azure-application-offer-setup-details).

## <a name="enforcing-tls-12-note"></a>Note concernant la mise en œuvre de TLS 1.2

La version 1.2 de TLS a été mise en œuvre en tant que version minimale pour les communications HTTPs. Veillez à utiliser cette version de TLS dans votre code. Les versions 1.0 et 1.1 de TLS sont déconseillées et les tentatives de connexion sont refusées.

## <a name="metered-billing-single-usage-event"></a>Événement d’utilisation unique avec facturation à la consommation

L’API d’événement d’utilisation doit être appelée par le serveur de publication pour émettre des événements par rapport à une ressource active (souscrite) pour le plan acheté par le client. L’événement d’utilisation est émis séparément pour chaque dimension personnalisée du plan défini par le serveur de publication liée à l’offre.

Un seul événement d’utilisation peut être émis pour chaque heure d’un jour calendaire. Par exemple, à 8h15 aujourd’hui, vous pouvez émettre un événement d’utilisation. Si cet événement est accepté, l’événement d’utilisation suivant sera accepté à partir de 9h00 ce même jour. Si vous envoyez un événement supplémentaire entre 8h15 et 8h59m59s aujourd’hui, il sera rejeté en tant que doublon. Vous devez accumuler toutes les unités consommées au cours d’une heure, puis les émettre en un seul événement.

Un seul événement d’utilisation peut être émis pour chaque heure d’un jour calendaire par ressource. Si plusieurs unités sont consommées au cours d’une heure, accumulez toutes les unités consommées dans l’heure, puis émettez-les en un seul événement. Les événements d’utilisation peuvent être émis uniquement au cours des dernières 24 heures. Si vous émettez un événement d’utilisation à tout moment entre 8h00 et 8h59m59s (et qu’il est accepté) et que vous envoyez un événement supplémentaire pour le même jour entre 8h00 et 8h59m59s, il sera rejeté en tant que doublon.

**POST**: `https://marketplaceapi.microsoft.com/api/usageEvent?api-version=<ApiVersion>`

*Paramètres de requête :*

| Paramètre | Recommandation          |
| ---------- | ---------------------- |
| `ApiVersion` | Utilisez 2018-08-31. |
| | |

*En-têtes de requête :*

| Content-Type       | Utilisez `application/json`.  |
| ------------------ | ---------------------------- |
| `x-ms-requestid`     | Valeur de chaîne unique pour le suivi de la requête du client, de préférence un GUID. Si cette valeur n’est pas fournie, une valeur sera générée et fournie dans les en-têtes de réponse. |
| `x-ms-correlationid` | Valeur de chaîne unique pour l’opération sur le client. Ce paramètre sert à corréler tous les événements de l’opération client avec les événements côté serveur. Si cette valeur n’est pas fournie, une valeur sera générée et fournie dans les en-têtes de réponse. |
| `authorization`   | Jeton d’accès unique qui identifie le serveur ISV qui effectue cet appel d’API. Le format est `"Bearer <access_token>"` lorsque la valeur de jeton est récupérée par le serveur de publication, comme expliqué pour <br> <ul> <li> SaaS dans [Récupération du jeton avec une requête HTTP POST](./pc-saas-registration.md#get-the-token-with-an-http-post). </li> <li> Application managée dans [Stratégies d’authentification](./marketplace-metering-service-authentication.md). </li> </ul> |
| | |

*Exemple de corps de la demande :*

```json
{
  "resourceId": <guid>, // unique identifier of the resource against which usage is emitted. 
  "quantity": 5.0, // how many units were consumed for the date and hour specified in effectiveStartTime, must be greater than 0, can be integer or float value
  "dimension": "dim1", // custom dimension identifier
  "effectiveStartTime": "2018-12-01T08:30:14", // time in UTC when the usage event occurred, from now and until 24 hours back
  "planId": "plan1", // id of the plan purchased for the offer
}
```

>[!NOTE]
>`resourceId` a une signification différente pour l’application SaaS et pour l’application managée qui émet un compteur personnalisé. 

Pour les plans des applications managées Azure Application, l’ID `resourceId` correspond à l’application managée `resource group Id`. Un exemple de script permettant de l’extraire est disponible dans [Utilisation du jeton d’identités managées par Azure](./marketplace-metering-service-authentication.md#using-the-azure-managed-identities-token). 

Pour les offres SaaS, le paramètre `resourceId` est l’ID d’abonnement SaaS. Pour plus d’informations sur les abonnements SaaS, voir [Lister les abonnements](./pc-saas-fulfillment-api-v2.md#get-list-of-all-subscriptions).

### <a name="responses"></a>Réponses

Code : 200<br>
OK. L’émission d’utilisation a été acceptée et enregistrée côté Microsoft en vue d’un traitement et d’une facturation supplémentaires.

Exemple de charge utile de réponse : 

```json
{
  "usageEventId": <guid>, // unique identifier associated with the usage event in Microsoft records
  "status": "Accepted" // this is the only value in case of single usage event
  "messageTime": "2020-01-12T13:19:35.3458658Z", // time in UTC this event was accepted
  "resourceId": <guid>, // unique identifier of the resource against which usage is emitted. For SaaS it's the subscriptionId.
  "quantity": 5.0, // amount of emitted units as recorded by Microsoft
  "dimension": "dim1", // custom dimension identifier
  "effectiveStartTime": "2018-12-01T08:30:14", // time in UTC when the usage event occurred, as sent by the ISV
  "planId": "plan1", // id of the plan purchased for the offer
}
```

Code : 400 <br>
Demande incorrecte.

* Données de requête manquantes ou non valides fournies.
* `effectiveStartTime` est supérieur à 24 heures dans le passé. L’événement a expiré.
* L’abonnement SaaS n’affiche pas l’état Abonné.

Exemple de charge utile de réponse : 

```json
{
  "message": "One or more errors have occurred.",
  "target": "usageEventRequest",
  "details": [
    {
      "message": "The resourceId is required.",
      "target": "ResourceId",
      "code": "BadArgument"
    }
  ],
  "code": "BadArgument"
}
```

Code : 403<br>

Interdit. Le jeton d’autorisation n’est pas fourni, n’est pas valide ou a expiré.  Ou la demande tente d’accéder à un abonnement pour une offre publiée avec un ID d’application Azure AD différent de celui utilisé pour créer le jeton d’autorisation.

Code : 409<br>
Conflit. Un événement d’utilisation a déjà été signalé pour l’ID de ressource, la date d’utilisation effective et l’heure spécifiés.

Exemple de charge utile de réponse : 

```json
{
  "additionalInfo": {
    "acceptedMessage": {
      "usageEventId": "<guid>", //unique identifier associated with the usage event in Microsoft records
      "status": "Duplicate",
      "messageTime": "2020-01-12T13:19:35.3458658Z",
      "resourceId": "<guid>", //unique identifier of the resource against which usage is emitted.
      "quantity": 1.0,
      "dimension": "dim1",
      "effectiveStartTime": "2020-01-12T11:03:28.14Z",
      "planId": "plan1"
    }
  },
  "message": "This usage event already exist.",
  "code": "Conflict"
}
```

## <a name="metered-billing-batch-usage-event"></a>Événement d’utilisation en lot avec facturation à la consommation

L’API d’événement d’utilisation par lot vous permet d’émettre des événements d’utilisation pour plusieurs ressources achetées en même temps. Elle vous permet également d’émettre plusieurs événements d’utilisation pour la même ressource, à condition qu’ils soient pour différentes heures calendaires. Le nombre maximal d’événements dans un lot unique est de 25.

**POST :** `https://marketplaceapi.microsoft.com/api/batchUsageEvent?api-version=<ApiVersion>`

*Paramètres de requête :*

| Paramètre  | Recommandation     |
| ---------- | -------------------- |
| `ApiVersion` | Utilisez 2018-08-31. |

*En-têtes de requête :*

| Content-Type       | Utilisez `application/json`.       |
| ------------------ | ------ |
| `x-ms-requestid`     | Valeur de chaîne unique pour le suivi de la requête du client, de préférence un GUID. Si cette valeur n’est pas fournie, une valeur sera générée et fournie dans les en-têtes de réponse. |
| `x-ms-correlationid` | Valeur de chaîne unique pour l’opération sur le client. Ce paramètre sert à corréler tous les événements de l’opération client avec les événements côté serveur. Si cette valeur n’est pas fournie, une valeur sera générée et fournie dans les en-têtes de réponse. |
| `authorization`      | Jeton d’accès unique qui identifie le serveur ISV qui effectue cet appel d’API. Le format est `Bearer <access_token>` lorsque la valeur de jeton est récupérée par le serveur de publication, comme expliqué pour <br> <ul> <li> SaaS dans [Récupération du jeton avec une requête HTTP POST](./pc-saas-registration.md#get-the-token-with-an-http-post). </li> <li> Application managée dans [Stratégies d’authentification](./marketplace-metering-service-authentication.md). </li> </ul> |
| | |


*Exemple de corps de la demande :*

```json
{
  "request": [ // list of usage events for the same or different resources of the publisher
    { // first event
      "resourceId": "<guid1>", // Unique identifier of the resource against which usage is emitted. 
      "quantity": 5.0, // how many units were consumed for the date and hour specified in effectiveStartTime, must be greater than 0, can be integer or float value
      "dimension": "dim1", //Custom dimension identifier
      "effectiveStartTime": "2018-12-01T08:30:14",//Time in UTC when the usage event occurred, from now and until 24 hours back
      "planId": "plan1", // id of the plan purchased for the offer
    },
    { // next event
      "resourceId": "<guid2>", 
      "quantity": 39.0, 
      "dimension": "email", 
      "effectiveStartTime": "2018-11-01T23:33:10
      "planId": "gold", // id of the plan purchased for the offer
    }
  ]
}
```

>[!NOTE]
>`resourceId` a une signification différente pour l’application SaaS et pour l’application managée qui émet un compteur personnalisé. 

Pour les plans des applications managées Azure Application, l’ID `resourceId` correspond à l’application managée `resource group Id`. Un exemple de script permettant de l’extraire est disponible dans [Utilisation du jeton d’identités managées par Azure](./marketplace-metering-service-authentication.md#using-the-azure-managed-identities-token). 

Pour les offres SaaS, le paramètre `resourceId` est l’ID d’abonnement SaaS. Pour plus d’informations sur les abonnements SaaS, voir [Lister les abonnements](./pc-saas-fulfillment-api-v2.md#get-list-of-all-subscriptions).

### <a name="responses"></a>Réponses

Code : 200<br>
OK. L’émission d’utilisation en lot a été acceptée et enregistrée côté Microsoft en vue d’un traitement et d’une facturation supplémentaires. La liste de réponses est retournée avec l’état de chaque événement individuel dans le lot. Vous devez itérer au sein de la charge utile de réponse pour comprendre les réponses pour chaque événement d’utilisation individuel envoyé dans le cadre de l’événement de traitement par lots.

Exemple de charge utile de réponse : 

```json
{
  "count": 2, // number of records in the response
  "result": [
    { // first response
      "usageEventId": "<guid>", // unique identifier associated with the usage event in Microsoft records
      "status": "Accepted" // see list of possible statuses below,
      "messageTime": "2020-01-12T13:19:35.3458658Z", // Time in UTC this event was accepted by Microsoft,
      "resourceId": "<guid1>", // unique identifier of the resource against which usage is emitted.
      "quantity": 5.0, // amount of emitted units as recorded by Microsoft 
      "dimension": "dim1", // custom dimension identifier
      "effectiveStartTime": "2018-12-01T08:30:14",// time in UTC when the usage event occurred, as sent by the ISV
      "planId": "plan1", // id of the plan purchased for the offer
    },
    { // second response
      "status": "Duplicate",
      "messageTime": "0001-01-01T00:00:00",
      "error": {
        "additionalInfo": {
          "acceptedMessage": {
            "usageEventId": "<guid>",
            "status": "Duplicate",
            "messageTime": "2020-01-12T13:19:35.3458658Z",
            "resourceId": "<guid2>",
            "quantity": 1.0,
            "dimension": "email",
            "effectiveStartTime": "2020-01-12T11:03:28.14Z",
            "planId": "gold"
          }
        },
        "message": "This usage event already exist.",
        "code": "Conflict"
      },
      "resourceId": "<guid2>",
      "quantity": 1.0,
      "dimension": "email",
      "effectiveStartTime": "2020-01-12T11:03:28.14Z",
      "planId": "gold"
    }
  ]
}
```

Description du code d’état référencé dans la réponse de l’API `BatchUsageEvent` :

| Code d’état  | Description |
| ---------- | -------------------- |
| `Accepted` | Accepté. |
| `Expired` | Utilisation expirée. |
| `Duplicate` | Utilisation dupliquée fournie. |
| `Error` | Code d’erreur. |
| `ResourceNotFound` | La ressource d’utilisation fournie n’est pas valide. |
| `ResourceNotAuthorized` | Vous n’êtes pas autorisé à fournir l’utilisation pour cette ressource. |
| `InvalidDimension` | La dimension pour laquelle l’utilisation est transmise n’est pas valide pour cette offre/ce plan. |
| `InvalidQuantity` | La quantité passée est inférieure ou égale à 0. |
| `BadArgument` | L’entrée est manquante ou incorrecte. |

Code : 400<br>
Demande incorrecte. Le lot contenait plus de 25 événements d’utilisation.

Code : 403<br>
Interdit. Le jeton d’autorisation n’est pas fourni, n’est pas valide ou a expiré.  Ou la demande tente d’accéder à un abonnement pour une offre publiée avec un ID d’application Azure AD différent de celui utilisé pour créer le jeton d’autorisation.

## <a name="development-and-testing-best-practices"></a>Bonnes pratiques en matière de développement et de test

Pour tester l’émission des compteurs personnalisés, implémentez l’intégration avec l’API de contrôle, créez un plan pour votre offre SaaS publiée avec des dimensions personnalisées définies à l’aide d’un prix unitaire nul. Puis publiez cette offre en version préliminaire afin que seuls les utilisateurs limités puissent accéder à l’intégration et la tester.

Vous pouvez également utiliser un plan privé pour une offre live existante afin de limiter l’accès à ce plan pendant le test à un public limité.

## <a name="get-support"></a>Obtenir de l’aide

Suivez les instructions de la section [Support technique pour le programme Place de marché commerciale dans l’Espace partenaires](../support.md) pour comprendre les options du serveur de publication et pour ouvrir un ticket de support auprès de Microsoft.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les API de service de mesure, consultez [FAQ sur l’API de service de mesure de la Place de marché](./marketplace-metering-service-apis-faq.md).

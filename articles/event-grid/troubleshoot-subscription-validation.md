---
title: Azure Event Grid - Résoudre les problèmes de validation des abonnements
description: Cet article explique comment résoudre les problèmes de validation des abonnements.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 857760182675d5673a3b09495c2faaf7372a4164
ms.sourcegitcommit: 1cf157f9a57850739adef72219e79d76ed89e264
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94592938"
---
# <a name="troubleshoot-azure-event-grid-subscription-validations"></a>Résoudre des problèmes de validation d’abonnements Azure Event Grid
Si, pendant la création d’un abonnement aux événements, vous voyez un message d’erreur tel que `The attempt to validate the provided endpoint https://your-endpoint-here failed. For more details, visit https://aka.ms/esvalidation`, cela est le signe d’un échec dans le processus de négociation de la validation. Pour résoudre cette erreur, vérifiez les points suivants :

- Effectuez une requête HTTP POST sur l’URL de votre webhook avec un [exemple de SubscriptionValidationEvent](webhook-event-delivery.md#validation-details) dans le corps de la requête en utilisant Postman, Curl ou un autre outil similaire.
- Si votre webhook implémente un mécanisme d'établissement de liaison de validation synchrone, vérifiez que le code de validation est renvoyé dans le cadre de la réponse.
- Si votre webhook implémente un mécanisme d’établissement de liaison de validation asynchrone, assurez-vous que la requête HTTP POST renvoie 200 OK.
- Si votre webhook renvoie `403 (Forbidden)` dans la réponse, vérifiez qu’il se trouve derrière une instance Azure Application Gateway ou Web Application Firewall. Si tel est le cas, vous devez désactiver ces règles de pare-feu et effectuer une nouvelle requête HTTP POST :
    - 920300 (En-tête Accept manquant dans la requête)
    - 942430 (Détection restreinte des anomalies de caractères SQL [args] : nombre de caractères spéciaux dépassé [12])
    - 920230 (Détection d’encodage de plusieurs URL)
    - 942130 (Attaque par injection de code SQL : Tautologie SQL détectée.)
    - 931130 (Attaque possible par inclusion de fichier distant = Référence/Lien hors domaine)

> [!IMPORTANT]
> Pour plus d’informations sur la validation des points de terminaison des webhooks, consultez [Remise d’événements webhook](webhook-event-delivery.md).

Les sections suivantes vous montrent comment valider un abonnement à des événements à l’aide de Postman et de Curl.  

## <a name="validate-event-grid-event-subscription-using-postman"></a>Valider un abonnement à un événement Event Grid à l’aide de Postman
Voici un exemple d’utilisation de Postman pour valider un abonnement webhook à un événement Event Grid : 

![Validation d’un abonnement à un événement Event Grid à l’aide de Postman](./media/troubleshoot-subscription-validation/event-subscription-validation-postman.png)

Voici un exemple d’événement JSON **SubscriptionValidationEvent** :

```json
[
  {
    "id": "2d1781af-3a4c-4d7c-bd0c-e34b19da4e66",
    "topic": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "subject": "",
    "data": {
      "validationCode": "512d38b6-c7b8-40c8-89fe-f46f9e9622b6",
    },
    "eventType": "Microsoft.EventGrid.SubscriptionValidationEvent",
    "eventTime": "2018-01-25T22:12:19.4556811Z",
    "metadataVersion": "1",
    "dataVersion": "1"
  }
]
```

Voici la réponse de réussite de l’exemple :

```json
{
  "validationResponse": "512d38b6-c7b8-40c8-89fe-f46f9e9622b6"
}
```

Pour en savoir plus sur la validation d’événements Event Grid pour les webhooks, consultez [Validation de point de terminaison avec des événements Event Grid](webhook-event-delivery.md#endpoint-validation-with-event-grid-events).


## <a name="validate-event-grid-event-subscription-using-curl"></a>Valider un abonnement à un événement Event Grid à l’aide de Curl 
Voici un exemple de commande Curl pour valider un abonnement webhook à un événement Event Grid : 

```bash
curl -X POST -d '[{"id": "2d1781af-3a4c-4d7c-bd0c-e34b19da4e66","topic": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx","subject": "","data": {"validationCode": "512d38b6-c7b8-40c8-89fe-f46f9e9622b6"},"eventType": "Microsoft.EventGrid.SubscriptionValidationEvent","eventTime": "2018-01-25T22:12:19.4556811Z", "metadataVersion": "1","dataVersion": "1"}]' -H 'Content-Type: application/json' https://{your-webhook-url.com}
```

## <a name="validate-cloud-event-subscription-using-postman"></a>Valider un abonnement à un événement cloud à l’aide de Postman
Voici un exemple d’utilisation de Postman pour valider un abonnement webhook à un événement cloud : 

![Validation d’un abonnement à un événement cloud à l’aide de Postman](./media/troubleshoot-subscription-validation/cloud-event-subscription-validation-postman.png)

Utilisez la méthode **HTTP OPTIONS** pour la validation avec des événements cloud. Pour en savoir plus sur la validation d’événements cloud pour les webhooks, consultez [Validation de point de terminaison avec des événements cloud](webhook-event-delivery.md#endpoint-validation-with-event-grid-events).

## <a name="troubleshoot-event-subscription-validation"></a>Résoudre les problèmes de validation des abonnements aux événements

## <a name="next-steps"></a>Étapes suivantes
Si vous avez besoin d’une aide supplémentaire, publiez votre problème sur le forum [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-eventgrid) ou ouvrez un [ticket de support](https://azure.microsoft.com/support/options/). 

---
title: Déboguer les erreurs lors de l’exécution d’une application Commandes personnalisées
titleSuffix: Azure Cognitive Services
description: Dans cet article, vous apprendrez à déboguer des erreurs de runtime dans une application Commandes personnalisées.
services: cognitive-services
author: xiaojul
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: xiaojul
ms.openlocfilehash: 49d9b91df896646da7bf36e077d9f3c9187137dd
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95021794"
---
# <a name="debug-errors-when-running-a-custom-commands-application"></a>Déboguer les erreurs lors de l’exécution d’une application Commandes personnalisées

Cet article explique comment déboguer les erreurs qui apparaissent lors de l’exécution d’une application Commandes personnalisées. 

## <a name="connection-failed"></a>Échec de la connexion

Si vous exécutez une application Commandes personnalisées à partir d’une [application cliente (avec le kit de développement logiciel (SDK) Speech)](./how-to-custom-commands-setup-speech-sdk.md) ou du [client Assistant vocal Windows](./how-to-custom-commands-developer-flow-test.md), les erreurs de connexion répertoriées ci-dessous peuvent s’afficher :

| Code d'erreur | Détails |
| ------- | -------- |
| [401](#error-401) | AuthenticationFailure : WebSocket Upgrade failed with an authentication error |
| [1002](#error-1002)] | Le serveur a retourné le code d'état '404' alors que le code d'état attendu était '101'. |

### <a name="error-401"></a>Erreur 401
- La région spécifiée dans l’application cliente ne correspond pas à la région de l’application Commandes personnalisées

- La clé de la ressource Speech n’est pas valide
    
    Vérifiez que votre clé de ressource Speech est correcte.

### <a name="error-1002"></a>Erreur 1002 
- Votre application Commandes personnalisées n’est pas publiée
    
    Publiez votre application dans le portail.

- L’ID de votre application Commandes personnalisées n’est pas valide

    Assurez-vous que l’ID de votre application Commandes personnalisées est correct.
 application Commandes personnalisées en dehors de votre ressource Speech

    Assurez-vous que l’application Commandes personnalisées est créée sous votre ressource Speech.

Pour plus d’informations sur la résolution des problèmes de connexion, reportez-vous à la [résolution des problèmes client de l’assistant vocal Windows](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant/tree/master/clients/csharp-wpf#troubleshooting)


## <a name="dialog-is-canceled"></a>Le dialogue est annulé

Lors de l’exécution de votre application Commandes personnalisées, le dialogue est annulé lorsque des erreurs se produisent.

- Si vous testez l’application dans le portail, elle affiche directement la description de l’annulation ainsi qu’une erreur earcon. 

- Si vous exécutez l’application avec le [client Assistant vocal Windows](./how-to-custom-commands-developer-flow-test.md), un message d’erreur earcon s’affiche. Vous trouverez l’événement **Event: CancelledDialog** sous la section **Journaux d'activité**.

- Si vous suivez notre exemple d’application cliente [application cliente (avec le kit de développement logiciel (SDK) Speech)](./how-to-custom-commands-setup-speech-sdk.md), un message d’erreur earcon s’affiche. Vous trouverez l’événement **Event: CancelledDialog** sous la section **État**.

- Si vous créez votre propre application cliente, vous pouvez toujours concevoir des logiques personnalisées pour gérer les événements CancelledDialog.

L’événement CancelledDialog se compose du code d’annulation et de la description, comme indiqué ci-dessous :

| Code d’annulation | Description de l’annulation |
| ------- | --------------- | ----------- |
| [MaxTurnThresholdReached](#no-progress-was-made-after-the-max-number-of-turns-allowed) | Aucune progression après le nombre maximal de tours autorisés |
| [RecognizerQuotaExceeded](#recognizer-usage-quota-exceeded) | Dépassement du quota d’utilisation du module de reconnaissance |
| [RecognizerConnectionFailed](#connection-to-the-recognizer-failed) | Échec de la connexion au module de reconnaissance |
| [RecognizerUnauthorized](#this-application-cannot-be-accessed-with-the-current-subscription) | Impossible d’accéder à cette application avec l’abonnement actuel |
| [RecognizerInputExceededAllowedLength](#input-exceeds-the-maximum-supported-length) | L’entrée dépasse la longueur maximale prise en charge pour le module de reconnaissance |
| [RecognizerNotFound](#recognizer-not-found) | Module de reconnaissance introuvable |
| [RecognizerInvalidQuery](#invalid-query-for-the-recognizer) | Requête non valide pour le module de reconnaissance |
| [RecognizerError](#recognizer-return-an-error) | Le module de reconnaissance retourne une erreur |

### <a name="no-progress-was-made-after-the-max-number-of-turns-allowed"></a>Aucune progression après le nombre maximal de tours autorisés
Le dialogue est annulé lorsqu’un emplacement requis n’est pas correctement mis à jour après un certain nombre de tours. Le nombre maximal par défaut est 3.

### <a name="recognizer-usage-quota-exceeded"></a>Dépassement du quota d’utilisation du module de reconnaissance
Language Understanding (LUIS) limite l’utilisation des ressources. Généralement, l’erreur de dépassement du quota d’utilisation du module de reconnaissance peut être due à : 
- Votre création LUIS dépasse la limite

    Ajoutez une ressource de prédiction à votre application Commandes personnalisées : 
    1. accédez à **Paramètres**, ressource LUIS
    1. Choisissez une ressource de prédiction dans **Ressource de prédiction**, ou cliquez sur **Créer une ressource** 

- Votre ressource de prédiction LUIS dépasse la limite

    Si vous utilisez une ressource de prédiction F0, elle est limitée à 10 000/mois et 5 requêtes/seconde.

Pour plus d’informations sur les limites des ressources LUIS, consultez [Utilisation et limite d’une ressource Language Understanding](../luis/luis-limits.md#resource-usage-and-limits)

### <a name="connection-to-the-recognizer-failed"></a>Échec de la connexion au module de reconnaissance
En général, cela signifie que la connexion temporaire au module de reconnaissance Language Understanding (LUIS) a échoué. Essayez à nouveau, le problème devrait être résolu.

### <a name="this-application-cannot-be-accessed-with-the-current-subscription"></a>Impossible d’accéder à cette application avec l’abonnement actuel
Votre abonnement n’est pas autorisé à accéder à l’application LUIS. 

### <a name="input-exceeds-the-maximum-supported-length"></a>L’entrée dépasse la longueur maximale prise en charge
Votre entrée a dépassé la limite de 500 caractères. Nous n’autorisons au maximum 500 caractères pour l’énoncé d’entrée.

### <a name="invalid-query-for-the-recognizer"></a>Requête non valide pour le module de reconnaissance
Votre entrée a dépassé la limite de 500 caractères. Nous n’autorisons au maximum 500 caractères pour l’énoncé d’entrée.

### <a name="recognizer-return-an-error"></a>Le module de reconnaissance retourne une erreur
Le module de reconnaissance LUIS a renvoyé une erreur lors de la tentative de reconnaissance de votre entrée.

### <a name="recognizer-not-found"></a>Module de reconnaissance introuvable
Impossible de trouver le type de module de reconnaissance spécifié dans votre modèle de dialogue Commandes personnalisées. Actuellement, nous prenons uniquement en charge [Language Understanding (LUIS) Recognizer](https://www.luis.ai/).

## <a name="other-common-errors"></a>Autres erreurs courantes
### <a name="unexpected-response"></a>Réponse inattendue
Des réponses inattendues peuvent être à l’origine de plusieurs événements. Effectuons quelques vérifications pour commencer :
- Intentions Oui/Non dans des exemples de phrases

    Nous ne prenons actuellement pas en charge les intentions Oui/Non, sauf en cas d’utilisation d’une fonctionnalité de confirmation. Toutes les intentions Oui/Non définies dans des exemples de phrases ne sont pas détectées.

- Intentions et exemples de phrases similaires dans les commandes

    La précision de la reconnaissance LUIS peut être affectée lorsque deux commandes partagent des intentions et des exemples de phrases similaires. Vous pouvez essayer de rendre la fonctionnalité des commandes et les exemples de phrases aussi distincts que possible.

    Pour améliorer la précision de la reconnaissance, reportez-vous aux [meilleures pratiques LUIS](../luis/luis-concept-best-practices.md).

- Le dialogue est annulé
    
    Consultez les raisons de l’annulation dans la section ci-dessus.

### <a name="error-while-rendering-the-template"></a>Erreur lors du rendu du modèle
Un paramètre non défini est utilisé dans la réponse vocale. 

### <a name="object-reference-not-set-to-an-instance-of-an-object"></a>La référence d’objet n’a pas la valeur d’une instance d’un objet
La charge utile JSON définie dans l’action **Envoyer l’activité au client** contient un paramètre vide.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Voir des exemples sur GitHub](https://aka.ms/speech/cc-samples)
---
title: Interactions humaines et délais d’expiration dans l’extension Fonctions durables - Azure
description: Découvrez comment gérer des interactions humaines et les délais d’expiration dans l’extension Fonctions durables d’Azure Functions.
ms.topic: conceptual
ms.date: 12/07/2018
ms.author: azfuncdf
ms.openlocfilehash: 4e0f71369bc02fdce5625d9c74e1d52264ed86be
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80335758"
---
# <a name="human-interaction-in-durable-functions---phone-verification-sample"></a>Interaction humaine dans l’extension Fonctions durables : exemple de vérification par téléphone

Cette exemple indique comment créer une orchestration [Fonctions durables](durable-functions-overview.md) impliquant des interactions humaines. Lorsqu’une personne participe à un processus automatisé, ce dernier doit pouvoir envoyer des notifications à cette personne, et recevoir des réponses de manière asynchrone. Il doit également tenir compte du fait que la personne peut être indisponible. (C’est pour cela que les délais d’attente jouent un rôle d’autant plus important.)

Cet exemple implémente un système de vérification de téléphone SMS. Ces types de flux sont souvent utilisés lors de la vérification du numéro de téléphone d’un client, ou pour l’authentification multifacteur (MFA). Il s’agit d’un exemple efficace, car l’implémentation s’effectue à l’aide de quelques fonctions de petite taille. Aucune banque de données externe (base de données, par exemple) n’est requise.

[!INCLUDE [durable-functions-prerequisites](../../../includes/durable-functions-prerequisites.md)]

## <a name="scenario-overview"></a>Présentation du scénario

Grâce à la procédure de vérification par téléphone, vous pouvez vous assurer que les utilisateurs de votre application ont été authentifiés et ne sont pas des expéditeurs de courrier indésirable. L’authentification multifacteur est un cas d’usage courant pour la protection des comptes d’utilisateur contre les pirates. Le défi lié à l’implémentation de votre propre vérification par téléphone est qu’elle nécessite une **interaction avec état** avec un être humain. Un utilisateur final reçoit généralement du code (par exemple, un nombre à 4 chiffres) et doit répondre **dans un délai raisonnable**.

Les fonctions Azure Functions ordinaires sont sans état (comme de nombreux autres points de terminaison cloud sur d’autres plateformes). Ces types d’interactions impliquent donc la gestion explicite de l’état en externe, dans une base de données ou un magasin persistant. En outre, l’interaction doit être répartie en plusieurs fonctions, qui peuvent être coordonnées entre elles. Par exemple, il vous faut au moins une fonction permettant de choisir un code, de le rendre persistant à un stade quelconque et de l’envoyer au téléphone de l’utilisateur. En outre, vous avez besoin d’une autre fonction, au minimum, pour recevoir une réponse de l’utilisateur et la mapper à l’appel de fonction d’origine, afin de valider le code. Le délai d’expiration est également un facteur important lors de la gestion de la sécurité. Cela peut rapidement devenir complexe.

La complexité de ce scénario est considérablement réduite lorsque vous utilisez l’extension Fonctions durables. Comme vous pouvez le constater dans cet exemple, une fonction d’orchestrateur peut gérer facilement l’interaction avec état, sans impliquer de magasins de données externes. Étant donné que les fonctions d’orchestrateur sont *durables*, ces flux interactifs sont également hautement fiables.

## <a name="configuring-twilio-integration"></a>Configuration de l’intégration de Twilio

[!INCLUDE [functions-twilio-integration](../../../includes/functions-twilio-integration.md)]

## <a name="the-functions"></a>Les fonctions

Cet article détaille les fonctions suivantes de l’exemple d’application :

* `E4_SmsPhoneVerification`: Une [fonction d’orchestrateur](durable-functions-bindings.md#orchestration-trigger) qui effectue le processus de vérification par téléphone, y compris la gestion des délais d’attente et des nouvelles tentatives.
* `E4_SendSmsChallenge`: Une [fonction d’activité](durable-functions-bindings.md#activity-trigger) qui envoie un code par SMS.

### <a name="e4_smsphoneverification-orchestrator-function"></a>Fonction d'orchestrateur E4_SmsPhoneVerification

# <a name="c"></a>[C#](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/PhoneVerification.cs?range=17-70)]

> [!NOTE]
> Cela peut ne pas sembler évident, mais cette fonction d’orchestrateur est entièrement déterministe. En effet, la propriété `CurrentUtcDateTime` est utilisée pour calculer le délai d’expiration du minuteur et renvoie la même valeur à chaque réexécution à ce niveau du code d’orchestrateur. Ce comportement est important pour vérifier que le même paramètre `winner` provient de chaque appel répété à `Task.WhenAny`.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

La fonction **E4_SmsPhoneVerification** utilise le fichier *function.json* standard pour les fonctions d’orchestrateur.

[!code-json[Main](~/samples-durable-functions/samples/javascript/E4_SmsPhoneVerification/function.json)]

Voici le code qui implémente la fonction :

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E4_SmsPhoneVerification/index.js)]

> [!NOTE]
> Cela peut ne pas sembler évident, mais cette fonction d’orchestrateur est entièrement déterministe. En effet, la propriété `currentUtcDateTime` est utilisée pour calculer le délai d’expiration du minuteur et renvoie la même valeur à chaque réexécution à ce niveau du code d’orchestrateur. Ce comportement est important pour vérifier que le même paramètre `winner` provient de chaque appel répété à `context.df.Task.any`.

---

Une fois démarrée, cette fonction d’orchestrateur effectue les opérations suivantes :

1. Elle obtient le numéro de téléphone auquel elle va *envoyer* la notification par SMS.
2. Elle appelle la fonction **E4_SendSmsChallenge** pour envoyer un SMS à l’utilisateur, puis renvoie le code de demande d’accès à 4 chiffres attendu.
3. Elle crée un minuteur durable se déclenchant 90 secondes à partir de l’heure actuelle.
4. En parallèle avec le minuteur, elle attend un événement **SmsChallengeResponse** de la part de l’utilisateur.

L’utilisateur reçoit un SMS incluant le code à quatre chiffres. Il doit renvoyer ce code à quatre chiffres à l’instance de la fonction d’orchestrateur dans les 90 secondes, afin de terminer le processus de vérification. Si le code est incorrect, il peut effectuer trois nouvelles tentatives de saisie (dans les 90 secondes imparties).

> [!WARNING]
> Il est important [d’annuler les minuteurs](durable-functions-timers.md) si vous n’avez plus besoin qu’ils arrivent à expiration, comme dans l’exemple ci-dessus, quand une réponse à une stimulation est acceptée.

## <a name="e4_sendsmschallenge-activity-function"></a>Fonction d’activité E4_SendSmsChallenge

La fonction **E4_SendSmsChallenge** utilise la liaison Twilio pour envoyer le SMS incluant le code à quatre chiffres à l’utilisateur.

# <a name="c"></a>[C#](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/PhoneVerification.cs?range=72-89)]

> [!NOTE]
> Vous devez installer le package Nuget `Microsoft.Azure.WebJobs.Extensions.Twilio` pour exécuter l’exemple de code.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Le fichier *function.json* est défini comme suit :

[!code-json[Main](~/samples-durable-functions/samples/javascript/E4_SendSmsChallenge/function.json)]

Voici le code qui génère le code de demande d’accès à quatre chiffres, et envoie le SMS :

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E4_SendSmsChallenge/index.js)]

---

## <a name="run-the-sample"></a>Exécution de l'exemple

En utilisant les fonctions déclenchées via HTTP incluses dans l’exemple, vous pouvez démarrer l’orchestration en envoyant la requête HTTP POST suivante :

```
POST http://{host}/orchestrators/E4_SmsPhoneVerification
Content-Length: 14
Content-Type: application/json

"+1425XXXXXXX"
```

```
HTTP/1.1 202 Accepted
Content-Length: 695
Content-Type: application/json; charset=utf-8
Location: http://{host}/runtime/webhooks/durabletask/instances/741c65651d4c40cea29acdd5bb47baf1?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}

{"id":"741c65651d4c40cea29acdd5bb47baf1","statusQueryGetUri":"http://{host}/runtime/webhooks/durabletask/instances/741c65651d4c40cea29acdd5bb47baf1?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}","sendEventPostUri":"http://{host}/runtime/webhooks/durabletask/instances/741c65651d4c40cea29acdd5bb47baf1/raiseEvent/{eventName}?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}","terminatePostUri":"http://{host}/runtime/webhooks/durabletask/instances/741c65651d4c40cea29acdd5bb47baf1/terminate?reason={text}&taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}"}
```

La fonction d’orchestrateur reçoit le numéro de téléphone fourni, et lui envoie immédiatement un SMS incluant un code de vérification à 4 chiffres généré de manière aléatoire &mdash; par exemple, *2168*. Ensuite, la fonction attend une réponse pendant 90 secondes.

Pour répondre avec le code, vous pouvez utiliser [`RaiseEventAsync` (.NET) ou `raiseEvent` (JavaScript)](durable-functions-instance-management.md) à l’intérieur d’une autre fonction, ou appeler le Webhook HTTP POST **sendEventUrl** référencé dans la réponse 202 ci-dessus, en remplaçant`{eventName}` par le nom de l’événement, `SmsChallengeResponse` :

```
POST http://{host}/runtime/webhooks/durabletask/instances/741c65651d4c40cea29acdd5bb47baf1/raiseEvent/SmsChallengeResponse?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}
Content-Length: 4
Content-Type: application/json

2168
```

Si vous envoyez ce code avant l’expiration du minuteur, l’orchestration se termine ; le champ `output` est défini sur `true`, ce qui indique que la vérification a abouti.

```
GET http://{host}/runtime/webhooks/durabletask/instances/741c65651d4c40cea29acdd5bb47baf1?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}
```

```
HTTP/1.1 200 OK
Content-Length: 144
Content-Type: application/json; charset=utf-8

{"runtimeStatus":"Completed","input":"+1425XXXXXXX","output":true,"createdTime":"2017-06-29T19:10:49Z","lastUpdatedTime":"2017-06-29T19:12:23Z"}
```

Si vous laissez le minuteur arriver à expiration, ou si vous entrez un code erroné quatre fois, vous pouvez demander l’état et consulter une sortie de la fonction d’orchestration `false`, qui indique l’échec de la vérification par téléphone.

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 145

{"runtimeStatus":"Completed","input":"+1425XXXXXXX","output":false,"createdTime":"2017-06-29T19:20:49Z","lastUpdatedTime":"2017-06-29T19:22:23Z"}
```

## <a name="next-steps"></a>Étapes suivantes

Cet exemple a illustré quelques-unes des fonctionnalités avancées de Durable Functions, notamment les API `WaitForExternalEvent` et `CreateTimer`. Vous avez vu de quelle manière les combiner avec `Task.WaitAny` pour implémenter un système fiable de gestion du délai d’expiration, qui est souvent utile pour interagir avec des personnes. Vous pouvez approfondir vos connaissances se rapportant à l’utilisation de Fonctions durables par la lecture d’une série d’articles développant certaines rubriques spécifiques.

> [!div class="nextstepaction"]
> [Accéder au premier article de la série](durable-functions-bindings.md)

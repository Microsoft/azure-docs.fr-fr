---
title: Traitement des événements fiables Azure Functions
description: Éviter de rater des messages Event Hub dans Azure Functions
author: craigshoemaker
ms.topic: conceptual
ms.date: 10/01/2020
ms.author: cshoe
ms.openlocfilehash: fd784bb184ff9432efc569ac9fd40de93eec0b53
ms.sourcegitcommit: 0d171fe7fc0893dcc5f6202e73038a91be58da03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93379585"
---
# <a name="azure-functions-reliable-event-processing"></a>Traitement des événements fiables Azure Functions

Le traitement des événements est l’un des scénarios les plus couramment associés à une architecture serverless. Cet article explique comment créer un processeur de messages fiables avec Azure Functions pour éviter de perdre des messages.

## <a name="challenges-of-event-streams-in-distributed-systems"></a>Défis liés aux flux d’événements dans des systèmes distribués

Prenons l’exemple d’un système qui envoie des événements à un débit constant de 100 événements par seconde. À ce débit, dans un délai de quelques minutes, plusieurs instances Functions parallèles peuvent consommer les 100 événements entrants chaque seconde.

En revanche, les conditions moins optimales suivantes sont possibles :

- Que se passe-t-il si l’éditeur d’événements envoie un événement endommagé ?
- Que se passe-t-il si votre instance Functions rencontre des exceptions non gérées ?
- Que se passe-t-il si un système en aval est mis hors connexion ?

Comment pouvez-vous gérer ces situations tout en préservant le débit de votre application ?

Avec les files d’attente, les messages fiables arrivent naturellement. Associée à un déclencheur Functions, la fonction crée un verrou sur le message en file d’attente. Si le traitement échoue, le verrou est libéré pour permettre à une autre instance d’effectuer une nouvelle tentative de traitement. Le traitement se poursuit alors jusqu’à ce que le message soit évalué correctement ou qu’il soit ajouté à une file d’attente de messages incohérents.

Même quand un message en file d’attente reste dans un cycle de nouvelle tentative, d’autres exécutions parallèles continuent de dépiler les messages restants. Le résultat est que le débit global reste largement non affecté par un seul message incorrect. En revanche, les files d’attente de stockage ne garantissent pas l’ordre et ne sont pas optimisées pour les demandes de débit élevé exigées par Event Hubs.

À l’opposé, Azure Event Hubs n’inclut pas de concept de verrouillage. Pour autoriser des fonctionnalités comme le débit élevé, les groupes de consommateurs multiples et la capacité de relecture, les événements Event Hubs se comportent davantage comme un lecteur vidéo. Les événements sont lus à partir d’un point unique dans le flux par partition. À partir de l’emplacement du pointeur, vous pouvez lire en avant ou en arrière, mais vous devez choisir de déplacer le pointeur pour les événements à traiter.

Quand des erreurs se produisent dans un flux, si vous décidez de conserver le pointeur au même emplacement, le traitement des événements est bloqué jusqu’à ce que le pointeur soit avancé. En d’autres termes, si le pointeur est arrêté pour gérer des problèmes de traitement d’un événement unique, les événements non traités commencent à s’accumuler.

Azure Functions évite les blocages en avançant le pointeur du flux, indépendamment d’une réussite ou d’un échec. Puisque le pointeur continue à avancer, vos fonctions ont besoin de gérer les échecs de manière appropriée.

## <a name="how-azure-functions-consumes-event-hubs-events"></a>Comment Azure Functions consomme les événements Event Hubs

Azure Functions consomme les événements Event Hubs en parcourant les étapes suivantes :

1. Un pointeur est créé et conservé dans le stockage Azure pour chaque partition du hub d’événements.
2. Quand de nouveaux messages sont reçus (dans un lot par défaut), l’hôte tente de déclencher la fonction avec le lot de messages.
3. Si la fonction termine son exécution (avec ou sans exception), le pointeur avance et un point de contrôle est enregistré dans le compte de stockage.
4. Si les conditions empêchent l’exécution complète de la fonction, l’hôte ne parvient pas à faire avancer le pointeur. Si le pointeur n’avance pas, alors les vérifications ultérieures finissent par traiter les mêmes messages.
5. Les étapes 2 à 4 se répètent.

Ce comportement révèle quelques points importants :

- *Les exceptions non gérées peuvent entraîner une perte de messages.* Les exécutions qui se terminent par une exception continuent à faire avancer le pointeur.  Définir une [stratégie de nouvelles tentatives](./functions-bindings-error-pages.md#retry-policies-preview) va retarder la progression du pointeur jusqu’à ce que la totalité de la stratégie de nouvelles tentatives ait été évaluée.
- *Functions garantit au minimum une remise.* Votre code et vos systèmes dépendants ont peut-être besoin de [tenir compte du fait que le même message peut être reçu deux fois](./functions-idempotent.md).

## <a name="handling-exceptions"></a>Gestion des exceptions

En règle générale, chaque fonction doit inclure un [bloc try/catch](./functions-bindings-error-pages.md) au niveau de code le plus élevé. Plus précisément, toutes les fonctions qui consomment des événements Event Hubs doivent avoir un bloc `catch`. Ainsi, quand une exception est levée, le bloc catch gère l’erreur avant que le pointeur ne progresse.

### <a name="retry-mechanisms-and-policies"></a>Mécanismes et stratégies de nouvelle tentative

Certaines exceptions sont transitoires par nature et ne réapparaissent pas quand une opération est retentée un peu plus tard. C’est la raison pour laquelle la première étape consiste toujours de retenter l’opération.  Vous pouvez exploiter les [stratégies de nouvelles tentatives](./functions-bindings-error-pages.md#retry-policies-preview) de l’application de fonction ou la logique de nouvelle tentative d’auteur pendant l’exécution de la fonction.

L’introduction de comportements de gestion des erreurs dans vos fonctions vous permet de définir des stratégies de nouvelle tentative à la fois basiques et avancées. Par exemple, vous pouvez implémenter une stratégie qui suit un workflow illustré par les règles suivantes :

- Essayez d’insérer un message trois fois (avec un délai éventuel entre les nouvelles tentatives).
- Si le résultat final de toutes les tentatives est un échec, ajoutez alors un message à une file d’attente afin que le traitement puisse continuer sur le flux.
- Les messages endommagés ou non traités sont alors gérés ultérieurement.

> [!NOTE]
> [Polly](https://github.com/App-vNext/Polly) est un exemple de bibliothèque de résilience et de gestion des erreurs temporaires pour les applications C#.

## <a name="non-exception-errors"></a>Erreurs hors exception

Certains problèmes se produisent même si aucune erreur n’est présente. Prenons l’exemple d’un échec qui se produit au milieu d’une exécution. Dans ce cas, si une fonction ne termine pas son exécution, le décalage du pointeur ne se produit jamais. Si le pointeur n’avance pas, alors toute instance qui s’exécute après l’échec d’exécution continue à lire les mêmes messages. Cette situation offre une garantie de remise « au moins une fois ».

Veiller à ce que chaque message soit traité au moins une fois implique que certains messages peuvent l’être plusieurs fois. Vos applications de fonction doivent avoir conscience de cette éventualité et doivent être générées autour des [principes de l’idempotence](./functions-idempotent.md).

## <a name="stop-and-restart-execution"></a>Arrêter et redémarrer l’exécution

S’il est possible d’accepter certaines erreurs, que se passe-t-il si votre application rencontre des défaillances importantes ? Vous pouvez avoir envie d’arrêter de déclencher des événements tant que l’état du système n’est pas sain. La possibilité d’interrompre le traitement est souvent obtenue avec un modèle de disjoncteur. Ce dernier permet à votre application de « disjoncter » le processus d’événement pour le reprendre plus tard.

Deux éléments sont nécessaires pour implémenter un disjoncteur dans un processus d’événement :

- État partagé sur toutes les instances pour suivre et superviser l’intégrité du circuit
- Processus maître capable de gérer l’état du circuit (ouvert ou fermé)

Les détails de l’implémentation peuvent varier, mais pour partager l’état entre les instances, vous avez besoin d’un mécanisme de stockage. Vous pouvez choisir de stocker l’état dans le stockage Azure, un cache Redis ou tout autre compte accessible par une collection de fonctions.

[Azure Logic Apps](../logic-apps/logic-apps-overview.md) ou les [fonctions durables](./durable/durable-functions-overview.md) constituent une solution naturelle pour gérer le workflow et l’état du circuit. D’autres services peuvent fonctionner tout aussi bien, mais les applications logiques sont utilisées pour cet exemple. L’utilisation d’applications logiques vous permet de suspendre et de redémarrer l’exécution d’une fonction tout en bénéficiant du contrôle nécessaire pour implémenter le modèle de disjoncteur.

### <a name="define-a-failure-threshold-across-instances"></a>Définir un seuil d’échec entre les instances

Pour tenir compte des multiples instances qui traitent simultanément des événements, vous avez besoin de conserver l’état externe partagé pour superviser l’intégrité du circuit.

Une règle que vous pouvez choisir d’implémenter peut mettre en place ce mécanisme :

- S’il y a plus de 100 échecs finals dans un délai de 30 secondes sur toutes les instances, alors le circuit disjoncte et le déclenchement de nouveaux messages est arrêté.

Les détails de l’implémentation varient en fonction de vos besoins, mais en règle générale, vous pouvez créer un système qui effectue ce qui suit :

1. Journalisation des échecs dans un compte de stockage (Stockage Azure, Redis, etc.)
1. Quand un nouvel échec est journalisé, inspection du nombre d’échecs pour voir si le seuil est atteint (par exemple, plus de 100 échecs au cours des 30 dernières secondes).
1. Si le seuil est atteint, émission d’un événement vers Azure Event Grid indiquant au système de disjoncter le circuit.

### <a name="managing-circuit-state-with-azure-logic-apps"></a>Gestion de l’état du circuit avec Azure Logic Apps

La description suivante souligne une façon de créer une application logique Azure pour interrompre le traitement d’une application Functions.

Azure Logic Apps est fourni avec des connecteurs intégrés à différents services, inclut des orchestrations avec état et convient naturellement à la gestion de l’état du circuit. Une fois que le besoin de disjoncter le circuit est détecté, vous pouvez générer une application logique pour implémenter le workflow suivant :

1. Déclencher un workflow Event Grid et arrêter la fonction Azure (avec le connecteur de ressource Azure)
1. Envoyer un e-mail de notification incluant une option de redémarrage du workflow

Le destinataire de l’e-mail peut examiner l’intégrité du circuit et, le cas échéant, redémarrer le circuit par le biais d’un lien figurant dans l’e-mail de notification. Quand le workflow redémarre la fonction, les messages sont traités à partir du dernier point de contrôle Event Hub.

Grâce à cette approche, aucun message n’est perdu, tous les messages sont traités dans l’ordre et vous pouvez disjoncter le circuit aussi longtemps que nécessaire.

## <a name="resources"></a>Ressources

- [Exemples de traitement des événements fiables](https://github.com/jeffhollan/functions-csharp-eventhub-ordered-processing)
- [Disjoncteur d’entité durable Azure](https://github.com/jeffhollan/functions-durable-actor-circuitbreaker)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les ressources suivantes :

- [Gestion des erreurs Azure Functions](./functions-bindings-error-pages.md)
- [Automatiser le redimensionnement des images chargées à l’aide d’Event Grid](../event-grid/resize-images-on-storage-blob-upload-event.md?toc=%2Fazure%2Fazure-functions%2Ftoc.json&tabs=dotnet)
- [Créer une fonction qui s’intègre avec Azure Logic Apps](./functions-twitter-email.md)

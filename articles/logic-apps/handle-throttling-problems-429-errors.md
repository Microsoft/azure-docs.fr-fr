---
title: Gérer les problèmes de limitation de requêtes, ou les erreurs « 429 – Trop de requêtes »
description: Comment contourner les problèmes de limitation ou les erreurs « HTTP 429 Trop de requêtes » dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: deli, logicappspm
ms.topic: conceptual
ms.date: 04/13/2020
ms.openlocfilehash: ea153b1927a337be29c2eb69e2417cc250abf5e8
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94366048"
---
# <a name="handle-throttling-problems-429---too-many-requests-errors-in-azure-logic-apps"></a>Gérer les problèmes de limitation (429 – erreurs « Trop de requêtes ») dans Azure Logic Apps

Dans [Azure Logic Apps](../logic-apps/logic-apps-overview.md), votre application logique retourne une [erreur « HTTP 429 Trop de requêtes »](https://developer.mozilla.org/docs/Web/HTTP/Status/429) lorsqu’il y a une limitation. C’est le cas lorsque le nombre de requêtes dépasse la vitesse de gestion de la destination pendant une durée spécifique. La limitation peut créer des problèmes tels que le traitement différé des données, une réduction des performances de vitesse et des erreurs telles que le dépassement de la stratégie de nouvelles tentatives spécifiée.

![Limitation dans le connecteur SQL Server](./media/handle-throttling-problems-429-errors/example-429-too-many-requests-error.png)

Voici quelques types courants de limitations que votre application logique peut rencontrer :

* [Application logique](#logic-app-throttling)
* [Connecteur](#connector-throttling)
* [Service ou système de destination](#destination-throttling)

<a name="logic-app-throttling"></a>

## <a name="logic-app-throttling"></a>Limitation de l’application logique

Le service Azure Logic Apps a ses propres [limites de débit](../logic-apps/logic-apps-limits-and-config.md#throughput-limits). Par conséquent, si votre application logique dépasse ces limites, c’est votre ressource d’application logique qui est limitée, et pas simplement une instance ou une exécution spécifique.

Pour rechercher les événements de limitation à ce niveau, consultez le volet **Métriques** de votre application logique dans le portail Azure.

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur d’applications logiques.

1. Dans le menu de l’application logique, sous **Surveillance** , sélectionnez **Métriques**.

1. Sous **Titre du graphique** , sélectionnez **Ajouter une métrique** de manière à ajouter une métrique à la métrique existante.

1. Dans la première barre de métrique, dans la liste **MÉTRIQUE** , sélectionnez **Événements limités d’action**. Dans la deuxième barre de métrique, dans la liste **MÉTRIQUE** , sélectionnez **Événements limités de déclenchement**.

Pour gérer la limitation à ce niveau, les options suivantes s’offrent à vous :

* Limitez le nombre d’instances d’application logique qui peuvent s’exécuter simultanément.

  Par défaut, si la condition du déclencheur de votre application logique est remplie plusieurs fois en même temps, plusieurs instances de déclencheur pour votre application logique s’exécutent simultanément ou *en parallèle*. Ce comportement signifie que chaque instance de déclencheur s’active avant la fin de l’exécution de l’instance de workflow précédente.

  Bien que le nombre d’instances de déclencheur par défaut pouvant s’exécuter simultanément soit [illimité](../logic-apps/logic-apps-limits-and-config.md#concurrency-looping-and-debatching-limits), vous pouvez limiter ce nombre en [activant le paramètre de concurrence du déclencheur](../logic-apps/logic-apps-workflow-actions-triggers.md#change-trigger-concurrency) et, si nécessaire, sélectionner une limite autre que la valeur par défaut.

* Activer le mode de débit élevé.

  Une application logique a une [limite par défaut pour le nombre d’actions pouvant s’exécuter à un intervalle de roulement de 5 minutes](../logic-apps/logic-apps-limits-and-config.md#throughput-limits). Pour augmenter cette limite au nombre d’actions maximal, activez [mode de débit élevé](../logic-apps/logic-apps-limits-and-config.md#run-high-throughput-mode) sur votre application logique.

* Désactivez le comportement de décomposition du tableau (« fractionner sur ») dans les déclencheurs.

  Si un déclencheur retourne un tableau pour les actions de workflow restant à traiter, le [paramètre **Fractionner sur**](../logic-apps/logic-apps-workflow-actions-triggers.md#split-on-debatch) du déclencheur fractionne les éléments du tableau et démarre une instance de workflow pour chaque élément de ce dernier, déclenchant ainsi efficacement plusieurs exécutions simultanées jusqu’à la [limite de **Fractionner sur**](../logic-apps/logic-apps-limits-and-config.md#concurrency-looping-and-debatching-limits). Pour contrôler la limitation, désactivez le comportement **Fractionner sur** et faites en sorte que votre application logique traite l’ensemble du tableau avec un seul appel, plutôt que de gérer un seul élément par appel.

* Refactoriser les actions en applications logiques plus petites.

  Comme mentionné précédemment, une application logique est limitée à un [nombre par défaut d’actions pouvant être exécutées sur une période de 5 minutes](../logic-apps/logic-apps-limits-and-config.md#throughput-limits). Même si vous pouvez augmenter cette limite en activant le [mode de débit élevé](../logic-apps/logic-apps-limits-and-config.md#run-high-throughput-mode), vous pouvez également envisager de répartir les actions de votre application logique en applications logiques plus petites afin que le nombre d’actions exécutées dans chaque application logique reste inférieur à la limite. Ceci vous permet de réduire la charge pesant sur une seule ressource d’application logique et de répartir la charge sur plusieurs applications logiques. Cette solution fonctionne mieux pour les actions qui gèrent des jeux de données volumineux ou lancent un grand nombre d’actions exécutées simultanément, pour les itérations de boucle ou pour les actions au sein de chaque itération de boucle qui dépassent la limite d’exécution des actions.

  Par exemple, cette application logique effectue tout le travail pour obtenir des tables à partir d’une base de données SQL Server et obtient les lignes de chaque table. La boucle **Pour chaque** itère simultanément dans chaque table afin que l’action **Obtenir des lignes** retourne les lignes de chaque table. Selon les quantités de données contenues dans ces tables, ces actions peuvent dépasser la limite des actions exécutées.

  ![Application logique « avant « la refactorisation](./media/handle-throttling-problems-429-errors/refactor-logic-app-before-version.png)

  Après la refactorisation, l’application logique est désormais une application logique parente et enfant. Le parent obtient les tables de SQL Server, puis appelle une application logique enfant pour chaque table pour obtenir les lignes :

  ![Créer une application logique pour une action](./media/handle-throttling-problems-429-errors/refactor-logic-app-single-connection-1.png)

  Voici l’application logique enfant appelée par l’application logique parente afin d’obtenir les lignes pour chaque table :

  ![Créer une autre application logique pour une deuxième action](./media/handle-throttling-problems-429-errors/refactor-logic-app-single-connection-2.png)

<a name="connector-throttling"></a>

## <a name="connector-throttling"></a>Limitation des connecteurs

Chaque connecteur a ses propres limitations, que vous pouvez trouver à la page de référence technique du connecteur. Par exemple, le [connecteur Azure Service Bus](/connectors/servicebus/) a une limitation qui autorise jusqu’à 6 000 appels par minute, tandis que le connecteur SQL Server a des [limitations qui varient en fonction du type d’opération](/connectors/sql/).

Certains déclencheurs et actions, tels que HTTP, ont une [« stratégie de nouvelle tentative »](../logic-apps/logic-apps-exception-handling.md#retry-policies) que vous pouvez personnaliser en fonction des [limites de la stratégie de nouvelle tentative](../logic-apps/logic-apps-limits-and-config.md#retry-policy-limits) pour implémenter la gestion des exceptions. Cette stratégie spécifie si et à quelle fréquence un déclencheur ou une action réessaie d’effectuer une requête quand la requête d’origine expire ou échoue et génère une réponse 408, 429 ou 5xx. Ainsi, lorsque la limitation démarre et renvoie une erreur 429, Logic Apps suit la stratégie de nouvelle tentative lorsqu’elle est prise en charge.

Pour savoir si un déclencheur ou une action prend en charge les stratégies de nouvelle tentative, vérifiez les paramètres du déclencheur ou de l’action. Pour afficher les nouvelles tentatives d’un déclencheur ou d’une action, accédez à l’historique des exécutions de votre application logique, sélectionnez l’exécution que vous souhaitez examiner, puis développez le déclencheur ou l’action pour afficher des détails sur les entrées, les sorties et les nouvelles tentatives, par exemple :

![Afficher l’historique des exécutions, les nouvelles tentatives, les entrées et les sorties de l’action](./media/handle-throttling-problems-429-errors/example-429-too-many-requests-retries.png)

Bien que l’historique des nouvelles tentatives fournisse des informations sur les erreurs, vous risquez de rencontrer des problèmes de différenciation entre les limitations de connecteur et les [limitations de destination](#destination-throttling). Dans ce cas, vous devrez peut-être examiner les détails de la réponse ou effectuer des calculs d’intervalles de limitations pour identifier la source.

Pour les applications logiques dans le service multilocataire global Azure Logic Apps, la limitation a lieu au niveau de la *connexion*. Ainsi, par exemple, pour les applications logiques qui s’exécutent dans un [environnement de service d’intégration (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la limitation concerne toujours les connexions hors environnement ISE, car elles s’exécutent dans le service multilocataire global Azure Logic Apps. Les connexions ISE, qui sont créées par les connecteurs ISE, ne sont cependant pas limitées, car elles s’exécutent dans votre environnement ISE.

Pour gérer la limitation à ce niveau, les options suivantes s’offrent à vous :

* Configurez plusieurs connexions pour une seule action afin que l’application logique partitionne les données en vue de leur traitement.

  Pour cette option, déterminez si vous pouvez distribuer la charge de travail en divisant les requêtes d’une action sur plusieurs connexions vers la même destination à l’aide des mêmes informations d’identification.

  Par exemple, supposons que votre application logique obtienne des tables d’une base de données SQL Server, puis les lignes de chaque table. En fonction du nombre de lignes que vous devez traiter, vous pouvez utiliser plusieurs connexions et plusieurs boucles **Pour chaque** afin de diviser le nombre total de lignes en jeux plus petits pour le traitement. Ce scénario utilise deux boucles **Pour chaque** afin de diviser le nombre total de lignes par deux. La première boucle **Pour chaque** utilise une expression qui obtient la première moitié. L’autre boucle **Pour chaque** utilise une expression différente qui obtient la deuxième moitié, par exemple :<p>

    * Expression 1 : la fonction `take()` obtient le début d’une collection. Pour plus d’informations, consultez la [ **`take()`** fonction](workflow-definition-language-functions-reference.md#take).

      `@take(collection-or-array-name, div(length(collection-or-array-name), 2))`

    * Expression 2 : la fonction `skip()` supprime le début d’une collection et retourne tous les autres éléments. Pour plus d’informations, consultez la [ **`skip()`** fonction](workflow-definition-language-functions-reference.md#skip).

      `@skip(collection-or-array-name, div(length(collection-or-array-name), 2))`

    Voici un exemple visuel qui montre comment vous pouvez utiliser ces expressions :

    ![Créer et utiliser plusieurs connexions pour une seule action](./media/handle-throttling-problems-429-errors/create-multiple-connections-per-action.png)

* Configurer une connexion différente pour chaque action.

  Pour cette option, déterminez si vous pouvez distribuer la charge de travail en répartissant les demandes de chaque action sur leur propre connexion, même lorsque des actions se connectent au même service ou système et utilisent les mêmes informations d’identification.

  Par exemple, supposons que votre application logique obtienne les tables d’une base de données SQL Server et chaque ligne de chaque table. Vous pouvez utiliser des connexions distinctes pour que l’obtention des tables utilise une seule connexion, tandis que l’obtention de chaque ligne utilise une autre connexion.

  ![Créer et utiliser des connexions différentes pour chaque action](./media/handle-throttling-problems-429-errors/create-connection-per-action.png)

* Modifiez la concurrence ou le parallélisme sur une [boucle « For each »](../logic-apps/logic-apps-control-flow-loops.md#foreach-loop).

  Par défaut, les itérations de boucle « For each » s’exécutent en même temps jusqu’à la [limite de concurrence](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Si vous disposez d’un connecteur limité à l’intérieur d’une boucle « For each », vous pouvez réduire le nombre d’itérations de boucle qui s’exécutent en parallèle. Pour plus d’informations, consultez les rubriques suivantes :
  
  * [Boucles « For each » : modifier l’accès simultané ou exécuter séquentiellement](../logic-apps/logic-apps-control-flow-loops.md#sequential-foreach-loop)

  * [Schéma de langage de définition de workflow : boucles For each](../logic-apps/logic-apps-workflow-actions-triggers.md#foreach-action)

  * [Schéma de langage de définition de workflow : concurrence de boucles « For each »](../logic-apps/logic-apps-workflow-actions-triggers.md#change-for-each-concurrency)

  * [Schéma du langage de définition de workflow : exécuter des boucles « For each » séquentiellement](../logic-apps/logic-apps-workflow-actions-triggers.md#sequential-for-each)

<a name="destination-throttling"></a>

## <a name="destination-service-or-system-throttling"></a>Limitation du service ou du système de destination

Bien qu’un connecteur ait ses propres limitations, le service ou le système de destination appelé par le connecteur peut également en avoir. Par exemple, certaines API dans Microsoft Exchange Server ont des limitations plus strictes que le connecteur Office 365 Outlook.

Par défaut, les instances d’une application logique et toutes les boucles ou branches situées à l’intérieur de ces instances s’exécutent *en parallèle*. Ce comportement signifie que plusieurs instances peuvent appeler le même point de terminaison en même temps. Chaque instance ne connaît pas l’existence de l’autre. Par conséquent, les nouvelles tentatives d’actions ayant échoué peuvent créer des [conditions de concurrence](https://en.wikipedia.org/wiki/Race_condition) où plusieurs appels essaient de s’exécuter en même temps, mais pour aboutir, ces appels doivent parvenir au service ou au système de destination avant que la limitation ne démarre.

Par exemple, supposons que vous ayez un tableau de 100 éléments. Vous utilisez une boucle « For each » pour itérer dans tout le tableau et activer le contrôle d’accès concurrentiel de la boucle afin de pouvoir limiter le nombre d’itérations parallèles à 20 ou à la [limite par défaut actuelle](../logic-apps/logic-apps-limits-and-config.md#concurrency-looping-and-debatching-limits). Au sein de cette boucle, une action insère un élément du tableau dans une base de données SQL Server, ce qui n’autorise que 15 appels par seconde. Ce scénario entraîne un problème de limitation, car un backlog de nouvelles tentatives est accumulé, mais jamais exécuté.

Ce tableau décrit la chronologie de ce qui se produit dans la boucle lorsque l’intervalle avant une nouvelle tentative de l’action est de 1 seconde :

| Limite dans le temps | Nombre d’actions qui s’exécutent | Nombre d’actions qui échouent | Nombre de nouvelles tentatives en attente |
|---------------|----------------------------|-----------------------------|---------------------------|
| T + 0 seconde | 20 insertions | 5 échecs en raison de la limite SQL | 5 nouvelles tentatives |
| T + 0,5 seconde | 15 insertions en raison de 5 dernières tentatives en attente | Les 15 échouent en raison de la limite SQL précédente toujours en vigueur pendant 0,5 seconde supplémentaires | 20 nouvelles tentatives <br>(les 5 précédentes + 15 nouvelles) |
| T + 1 seconde | 20 insertions | 5 ayant échoué, plus 20 nouvelles tentatives précédentes, en raison de la limite SQL | 25 nouvelles tentatives (20 + 5 nouvelles)
|||||

Pour gérer la limitation à ce niveau, les options suivantes s’offrent à vous :

* Créez des applications logiques, chacune gérant une seule opération.

  * Si vous poursuivez avec l’exemple de scénario SQL Server dans cette section, vous pouvez créer une application logique qui place des éléments de tableau dans une file d’attente, telle qu’une [file d’attente Azure Service Bus](../connectors/connectors-create-api-servicebus.md). Vous créez ensuite une autre application logique qui effectue uniquement l’opération d’insertion pour chaque élément de cette file d’attente. De cette façon, une seule instance d’application logique s’exécute à un moment spécifique, termine l’opération d’insertion et passe à l’élément suivant de la file d’attente, ou l’instance obtient 429 erreurs, mais n’effectue pas de nouvelles tentatives infructueuses.

  * Créez une application logique parente qui appelle une application logique enfant ou imbriquée pour chaque action. Si le parent doit appeler différentes applications enfants en fonction du résultat du parent, vous pouvez utiliser une action de condition ou une action de commutation qui détermine l’application enfant à appeler. Ce modèle peut vous aider à réduire le nombre d’appels ou d’opérations.

    Supposons, par exemple, que vous disposiez de deux applications logiques, chacune avec un déclencheur d’interrogation qui vérifie votre compte de messagerie chaque minute pour trouver un sujet spécifique, tel que « Réussite » ou « Échec ». Cette configuration entraîne 120 appels par heure. Au lieu de cela, si vous créez une application logique parente unique qui interroge toutes les minutes, mais appelle une application logique enfant qui s’exécute selon que le sujet est « Succès » ou « Échec », vous divisez le nombre de vérifications d’interrogation de moitié, soit 60 ici.

* Configurer le traitement par lots.

  Si le service de destination prend en charge les opérations par lots, vous pouvez traiter la limitation en traitant des éléments par groupes ou par lots, plutôt qu’individuellement. Pour implémenter la solution de traitement par lots, vous devez créer une application logique « destinataire de lots » et une application logique « expéditeur de lots ». L’expéditeur de lots collecte des messages ou des éléments jusqu’à ce que vos critères spécifiés soient remplis, puis envoie ces messages ou éléments dans un groupe unique. Le récepteur de lots accepte ce groupe et traite ces messages ou éléments. Pour plus d’informations, consultez la section [Messages de traitement par lots dans des groupes](../logic-apps/logic-apps-batch-process-send-receive-messages.md).

* Utilisez les versions du webhook pour les déclencheurs et les actions, plutôt que les versions d’interrogation.

  Pourquoi ? Un déclencheur d’interrogation continue à vérifier le service ou le système de destination à intervalles spécifiques. Un intervalle très fréquent, par exemple toutes les secondes, peut créer des problèmes de limitation. Toutefois, une action ou un déclencheur de webhook, comme [Webhook HTTP](../connectors/connectors-native-webhook.md), crée un seul appel au service ou au système de destination, qui a lieu au moment de l’abonnement et demande que la destination informe le déclencheur ou l’action uniquement lorsqu’un événement se produit. Ainsi, le déclencheur ou l’action n’a pas à vérifier continuellement la destination.
  
  Par conséquent, si le service ou le système de destination prend en charge les webhooks ou fournit un connecteur avec une version de webhook, cette option est préférable à l’utilisation de la version d’interrogation. Pour identifier les déclencheurs et actions de webhook, confirmez qu’ils sont de type `ApiConnectionWebhook` ou qu’ils ne requièrent pas que vous spécifiiez une récurrence. Pour plus d’informations, consultez [déclencheur APIConnectionWebhook](../logic-apps/logic-apps-workflow-actions-triggers.md#apiconnectionwebhook-trigger) et [action APIConnectionWebhook](../logic-apps/logic-apps-workflow-actions-triggers.md#apiconnectionwebhook-action).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur [les limites et la configuration de Logic Apps](../logic-apps/logic-apps-limits-and-config.md)

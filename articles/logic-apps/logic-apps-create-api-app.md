---
title: Créer des API Web et des API REST pour Azure Logic Apps
description: Créer des API web et des API REST pour appeler vos API, services ou systèmes pour les intégrations système dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: jonfan, logicappspm
ms.topic: conceptual
ms.date: 05/26/2017
ms.openlocfilehash: 3fe98160cc10eb3607b8309a9a263d63380dcfb5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89073214"
---
# <a name="create-custom-apis-you-can-call-from-azure-logic-apps"></a>Créer des API personnalisées que vous pouvez appeler à partir d’Azure Logic Apps

Bien qu’Azure Logic Apps offre [des centaines de connecteurs](../connectors/apis-list.md), que vous pouvez utiliser dans les flux de travail d’application logique, vous voudrez peut-être appeler des API, des systèmes et des services qui ne sont pas disponibles en tant que connecteurs. Vous pouvez créer vos propres API personnalisées, qui fournissent des actions et des déclencheurs utilisables dans des applications logiques. Voici d’autres raisons qui peuvent vous amener à vouloir créer vos propres API de façon à les appeler à partir de workflows d’applications logiques :

* Étendre l’intégration de votre système actuel et les flux de travail d’intégration de données.
* Aider les clients à utiliser votre service pour gérer des tâches personnelles ou professionnelles.
* Étendre la portée, la détectabilité et l’utilisation de votre service.

En principe, les connecteurs sont des API web qui utilisent REST pour les interfaces enfichables, le [format de métadonnées Swagger](https://swagger.io/specification/) pour la documentation et JSON en tant que format d’échange de données. Étant donné que les connecteurs sont des API REST qui communiquent via des points de terminaison HTTP, vous pouvez utiliser n’importe quel langage, comme .NET, Java, Python ou Node.js, pour la création de connecteurs. Vous pouvez également héberger votre API sur [Azure App Service](../app-service/overview.md), une offre de plateforme en tant que service (PaaS) qui fournit une des manières les plus optimales, les plus simples et les plus évolutives d’héberger des API. 

Pour que les API personnalisées fonctionnent avec les applications logiques, votre API peut fournir des [*actions*](./logic-apps-overview.md#logic-app-concepts) qui effectuent des tâches spécifiques dans les flux de travail des applications logiques. Votre API peut également faire office de [*déclencheur*](./logic-apps-overview.md#logic-app-concepts) qui démarre le flux de travail d’une application logique lorsque de nouvelles données ou un événement répondent à une condition donnée. Cette rubrique décrit les modèles courants que vous pouvez suivre pour la création d’actions et de déclencheurs dans votre API, en fonction du comportement que vous souhaitez que votre API adopte.

Vous pouvez héberger vos API sur [Azure App Service](../app-service/overview.md), une offre PaaS (platform as a service) qui propose un hébergement d’API simple et hautement évolutif.

> [!TIP] 
> Bien que vous puissiez déployer vos API comme des applications web, envisagez la possibilité de les déployer comme des applications API, car cela pourrait vous faciliter la tâche lorsque vous créerez, hébergerez et consommerez des API dans le cloud et en local. Vous n’êtes pas obligé de modifier le code dans vos API. Il vous suffit de déployer votre code dans une application API. Par exemple, découvrez comment générer des applications API créées dans les langages suivants : 
> 
> * [ASP.NET](../app-service/quickstart-dotnetcore.md) 
> * [Java](../app-service/quickstart-java.md)
> * [Node.JS](../app-service/quickstart-nodejs.md)
> * [PHP](../app-service/quickstart-php.md)
> * [Python](../app-service/quickstart-python.md)
> * [Ruby](../app-service/quickstart-ruby.md)
>
> Pour obtenir des exemples d’application API générées pour les applications logiques, consultez le [dépôt GitHub Azure Logic Apps](https://github.com/logicappsio).

## <a name="how-do-custom-apis-differ-from-custom-connectors"></a>Quelles sont les différences entre les API personnalisées et les connecteurs personnalisés ?

Les API personnalisées et les [connecteurs personnalisés](../logic-apps/custom-connector-overview.md) sont des API Web qui utilisent REST pour les interfaces enfichables, le [format de métadonnées Swagger](https://swagger.io/specification/) pour la documentation et JSON comme format d’échange de données. Et, dans la mesure où ces API et connecteurs sont des API REST qui communiquent via des points de terminaison HTTP, vous pouvez utiliser n’importe quel langage, par exemple, .NET, Java, Python ou Node.js, pour créer des API et des connecteurs personnalisés.

Les API personnalisées permettent d’appeler des API qui ne sont pas des connecteurs, et fournissent des points de terminaison qui peuvent être appelés avec HTTP + Swagger, la Gestion des API Azure ou App Services. Les connecteurs personnalisés fonctionnent comme des API personnalisées, avec ces attributs en plus :

* Ils sont inscrits comme des ressources de connecteur Logic Apps dans Azure.
* Ils apparaissent sous forme d’icônes avec les connecteurs gérés par Microsoft dans le Concepteur Logic Apps.
* Ils sont uniquement accessibles aux auteurs du connecteur et aux utilisateurs de l’application logique qui ont le même client Azure Active Directory et un abonnement Azure dans la région où sont déployées les applications logiques.

Vous pouvez également proposer des connecteurs inscrits à la certification Microsoft. Ce processus vérifie que les connecteurs inscrits répondent aux critères d’utilisation publique et les met à la disposition des utilisateurs de Power Automate et de Microsoft Power Apps.

Pour plus d’informations sur les connecteurs personnalisés, consultez les pages : 

* [Vue d’ensemble des connecteurs personnalisés](../logic-apps/custom-connector-overview.md)
* [Créer des connecteurs personnalisés à partir d’API Web](/connectors/custom-connectors/create-web-api-connector)
* [Inscrire des connecteurs personnalisés dans Azure Logic Apps](/connectors/custom-connectors/)

## <a name="helpful-tools"></a>Outils utiles

Une API personnalisée fonctionne mieux avec les applications logiques lorsqu’elle dispose également d’un [document Swagger](https://swagger.io/specification/) qui décrit ses opérations et ses paramètres.
De nombreuses bibliothèques (telles que [Swashbuckle](https://github.com/domaindrivendev/Swashbuckle)) permettent de générer automatiquement le fichier Swagger pour vous. Pour annoter le fichier Swagger avec les noms d’affichage, les types de propriété, etc., vous pouvez également utiliser [TRex](https://github.com/nihaue/TRex) afin que votre fichier Swagger fonctionne bien avec les applications logiques.

<a name="actions"></a>

## <a name="action-patterns"></a>Modèles d’action

Pour que les applications logiques effectuent des tâches, votre API personnalisée doit fournir des [*actions*](./logic-apps-overview.md#logic-app-concepts). Chaque opération dans votre API renvoie à une action. Une action de base peut être un contrôleur qui accepte les requêtes HTTP et renvoie des réponses HTTP. Par exemple, une application logique envoie une requête HTTP à votre application web ou à votre application API. Votre application renvoie alors une réponse HTTP, ainsi que du contenu que l’application logique peut traiter.

Pour une action standard, vous pouvez écrire une méthode de requête HTTP dans votre API et décrire cette méthode dans un fichier Swagger. Vous pouvez ensuite appeler votre API directement avec une [action HTTP](../connectors/connectors-native-http.md) ou une action [HTTP + Swagger](../connectors/connectors-native-http-swagger.md). Par défaut, les réponses doivent être renvoyées dans le [délai d’expiration de la requête](./logic-apps-limits-and-config.md). 

![Modèle d’action standard](./media/logic-apps-create-api-app/standard-action.png)

<a name="pattern-overview"></a> Pour qu’une application logique attende pendant que votre API termine des tâches plus longues, votre API peut suivre le [modèle d’interrogation asynchrone](#async-pattern) ou le [modèle Webhook asynchrone](#webhook-actions), tous deux décrits dans cette rubrique. Pour comprendre les différences entre ces modèles, prenons l’exemple de la commande d’un gâteau personnalisé dans une boulangerie. Le modèle d’interrogation reviendrait à appeler la boulangerie toutes les 20 minutes pour vérifier si le gâteau est prêt. Le modèle Webhook correspond au cas où la boulangerie vous demande votre numéro de téléphone afin de vous appeler quand le gâteau est prêt.

Pour obtenir des exemples, visitez le [référentiel GitHub d’applications logiques](https://github.com/logicappsio). Découvrez également les [mesures d’utilisation des actions](logic-apps-pricing.md).

<a name="async-pattern"></a>

### <a name="perform-long-running-tasks-with-the-polling-action-pattern"></a>Effectuer des tâches longues avec le modèle d’action d’interrogation

Pour que votre API effectue des tâches qui peuvent durer plus longtemps que le [délai d’expiration des requêtes](./logic-apps-limits-and-config.md), vous pouvez utiliser le modèle d’interrogation asynchrone. Ce modèle force votre API à travailler dans un thread distinct, mais conserve une connexion active au moteur Logic Apps. De cette façon, l’application logique n’expire pas ou passe à l’étape suivante dans le flux de travail avant que votre API termine sa tâche.

Voici le modèle général :

1. Assurez-vous que le moteur sait que votre API a accepté la requête et a commencé à travailler.
2. Lorsque le moteur demande ensuite plusieurs fois l’état du travail, indiquez au moteur lorsque votre API a terminé sa tâche.
3. Renvoyez les données pertinentes au moteur afin que le flux de travail de l’application logique puisse continuer.

<a name="bakery-polling-action"></a> Revenons maintenant à l’analogie de la boulangerie pour le modèle d’interrogation et imaginez que vous appelez une boulangerie pour commander un gâteau personnalisé en livraison. La confection du gâteau prend du temps, et vous n’avez pas envie d’attendre au téléphone pendant que le boulanger travaille. La boulangerie confirme votre commande et vous dit d’appeler toutes les 20 minutes pour connaître l’avancement de la commande. Après 20 minutes, vous appelez la boulangerie, qui vous indique que votre gâteau n’est pas prêt et que vous devez rappeler dans 20 minutes. Ces aller-retours se poursuivent jusqu'à ce que vous appeliez, que la boulangerie vous dise que votre commande est prête et qu’elle vous livre le gâteau. 

Revenons à ce modèle d’interrogation pour votre API. La boulangerie représente votre API personnalisée, tandis que vous, le client achetant le gâteau, représentez le moteur Logic Apps. Lorsque le moteur appelle votre API avec une requête, votre API confirme la requête et répond avec l’intervalle de temps pendant lequel le moteur peut vérifier l’état du travail. Le moteur continue la vérification du statut jusqu'à ce que votre API réponde que le travail est effectué et renvoie des données à votre application logique, qui poursuit ensuite le flux de travail. 

![Modèle d’action d’interrogation](./media/logic-apps-create-api-app/custom-api-async-action-pattern.png)

Voici les étapes spécifiques que votre API doit suivre, décrites du point de vue de l’API :

1. Lorsque votre API reçoit une requête HTTP lui demandant de commencer à travailler, renvoyez immédiatement une réponse HTTP `202 ACCEPTED` avec l’en-tête `location` décrit plus loin dans cette étape. Cette réponse informe le moteur Logic Apps que votre API a reçu la requête, a accepté la charge utile de la requête (entrée de données) et est en train de la traiter. 
   
   La réponse `202 ACCEPTED` doit inclure ces en-têtes :
   
   * *Obligatoire* : un en-tête `location` qui spécifie le chemin absolu à une URL permettant au moteur Logic Apps de vérifier l’état du travail de votre API.

   * *Facultatif* : un en-tête `retry-after` qui spécifie le nombre de secondes pendant lesquelles le moteur doit attendre avant de vérifier l’URL `location` pour connaître l’état du travail. 

     Par défaut, le moteur vérifie toutes les 20 secondes. Pour spécifier un intervalle différent, incluez l’en-tête `retry-after` et le nombre de secondes jusqu'à l’interrogation suivante.

2. Une fois le temps indiqué écoulé, le moteur Logic Apps interroge l’URL `location` pour vérifier l’état du travail. Votre API doit effectuer ces vérifications et renvoyer ces réponses :
   
   * Si le travail est effectué, renvoyez la réponse HTTP `200 OK`, ainsi que la charge utile de réponse (entrée pour l’étape suivante).

   * Si le travail est encore en cours de traitement, renvoyez une autre réponse HTTP `202 ACCEPTED`, avec les mêmes en-têtes que la réponse d’origine.

Lorsque votre API suit ce modèle, vous n’avez rien à modifier dans la définition du flux de travail d’application logique pour poursuivre la vérification du statut du travail. Lorsque le moteur reçoit une réponse HTTP `202 ACCEPTED` et un en-tête `location` valide, le moteur respecte le modèle asynchrone et vérifie l’en-tête `location` jusqu'à ce que votre API renvoie une réponse autre que 202.

> [!TIP]
> Pour un exemple de modèle asynchrone, consultez cet [exemple de réponse de contrôleur asynchrone dans GitHub](https://github.com/logicappsio/LogicAppsAsyncResponseSample).

<a name="webhook-actions"></a>

### <a name="perform-long-running-tasks-with-the-webhook-action-pattern"></a>Effectuer des tâches longues avec le modèle d’action Webhook

Pour les tâches longues et le traitement asynchrone, vous avez également la possibilité d’utiliser le modèle Webhook. Ce modèle met en pause l’application logique afin qu’elle attende un « rappel » de votre API pour terminer le traitement avant de poursuivre le flux de travail. Ce rappel est un élément POST HTTP qui envoie un message vers une URL quand un événement se produit. 

<a name="bakery-webhook-action"></a> Revenons maintenant à l’analogie de la boulangerie pour le modèle Webhook et imaginez que vous appelez une boulangerie pour commander un gâteau personnalisé en livraison. La confection du gâteau prend du temps, et vous n’avez pas envie d’attendre au téléphone pendant que le boulanger travaille. La boulangerie confirme votre commande, mais cette fois, vous donnez votre numéro de téléphone afin d’être rappelé quand le gâteau est terminé. Cette fois, la boulangerie vous prévient lorsque votre commande est prête et vous livre le gâteau.

Appliquons cet exemple à votre API : la boulangerie représente votre API personnalisée, tandis que vous, le client achetant le gâteau, représentez le moteur Logic Apps. Le moteur appelle votre API avec une requête et inclut une URL de « rappel ».
Lorsque le travail est terminé, votre API utilise l’URL pour en informer le moteur et renvoyer les données à votre application logique, avant de continuer le flux de travail. 

Pour ce modèle, définissez deux points de terminaison sur votre contrôleur : `subscribe` et `unsubscribe`

*  Point de terminaison `subscribe` : quand l’exécution atteint l’action de votre API dans le workflow, le moteur Logic Apps appelle le point de terminaison `subscribe`. Cette étape entraîne la création par l’application logique d’une URL de rappel que votre API enregistre, avant d’attendre le rappel de votre API lorsque le travail est terminé. Votre API envoie ensuite un rappel avec un élément POST HTTP à l’URL et transmet à l’application logique les en-têtes et le contenu renvoyés en tant qu’entrées.

* Point de terminaison `unsubscribe` : si l’exécution de l’application logique est annulée, le moteur Logic Apps appelle le point de terminaison `unsubscribe`. Votre API peut alors annuler l’inscription de l’URL de rappel, et arrêter tous les processus selon les besoins.

![Modèle d’action Webhook](./media/logic-apps-create-api-app/custom-api-webhook-action-pattern.png)

Actuellement, le Concepteur d’application logique ne prend pas en charge la découverte des points de terminaison Webhook via Swagger. Pour ce modèle, vous devez donc ajouter une action [**Webhook**](../connectors/connectors-native-webhook.md) et spécifier l’URL, les en-têtes et le corps de votre requête. Voir également [Actions et déclencheurs de flux de travail](logic-apps-workflow-actions-triggers.md#apiconnection-webhook-action). Pour un exemple de modèle Webhook, consultez cet [exemple de déclencheur Webhook dans GitHub](https://github.com/logicappsio/LogicAppTriggersExample/blob/master/LogicAppTriggers/Controllers/WebhookTriggerController.cs).

Voici d’autres conseils et remarques :

* Pour transmettre l’URL de rappel, vous pouvez utiliser la fonction de flux de travail `@listCallbackUrl()` dans un de ces champs en fonction des besoins.

* Si vous possédez l’application logique et le service abonné, vous n’êtes pas obligé d’appeler le point de terminaison `unsubscribe` après l’appel de l’URL de rappel. Dans le cas contraire, le runtime Logic Apps doit appeler le point de terminaison `unsubscribe` pour signaler qu’aucun autre appel n’est attendu et autoriser le nettoyage des ressources côté serveur.

<a name="triggers"></a>

## <a name="trigger-patterns"></a>Modèles de déclencheur

Votre API personnalisée peut faire office de [*déclencheur*](./logic-apps-overview.md#logic-app-concepts) qui démarre une application logique lorsque de nouvelles données ou un événement répondent à une condition donnée. Ce déclencheur peut soit vérifier régulièrement, soit attendre et écouter, la présence de nouvelles données ou d’événements au niveau du point de terminaison de votre service. Si de nouvelles données ou un événement remplissent la condition indiquée, le déclencheur est activé et démarre l’application logique, qui est à l’écoute de ce déclencheur. Pour démarrer les applications logiques de cette manière, votre API peut suivre le modèle de [*déclencheur d’interrogation*](#polling-triggers) ou de [*déclencheur Webhook*](#webhook-triggers). Ces modèles sont semblables à leurs équivalents : [actions d’interrogation](#async-pattern) et [actions Webhook](#webhook-actions). Découvrez également les [mesures d’utilisation des déclencheurs](logic-apps-pricing.md).

<a name="polling-triggers"></a>

### <a name="check-for-new-data-or-events-regularly-with-the-polling-trigger-pattern"></a>Rechercher régulièrement les nouvelles données ou les événements avec le modèle de déclencheur d’interrogation

Un *déclencheur d’interrogation* fonctionne presque comme [l’action d’interrogation](#async-pattern) décrite précédemment dans cette rubrique. Le moteur Logic Apps appelle régulièrement et recherche de nouvelles données ou des événements au niveau du point de terminaison du déclencheur. Si le moteur trouve de nouvelles données ou un événement qui répond à la condition spécifiée, le déclencheur est activé. Le moteur crée alors une instance d’application logique qui traite les données en tant qu’entrée. 

![Modèle de déclencheur d’interrogation](./media/logic-apps-create-api-app/custom-api-polling-trigger-pattern.png)

> [!NOTE]
> Chaque requête d’interrogation compte comme une exécution de l’action, même lorsqu’aucune instance d’application logique n’est créée. Pour éviter que les mêmes données soient traitées plusieurs fois, votre déclencheur doit nettoyer les données qu’il a déjà lues et transmises à l’application logique.

Voici les étapes spécifiques d’un déclencheur d’interrogation, décrites du point de vue de l’API :

| Il existe de nouvelles données ou un nouvel événement ?  | Réponse de l’API | 
| ------------------------- | ------------ |
| Trouvé | Renvoyez un état HTTP `200 OK` avec la charge utile de réponse (entrée pour l’étape suivante). <br/>Cette réponse crée une instance d’application logique et démarre le flux de travail. | 
| Introuvable | Renvoyez un état HTTP `202 ACCEPTED` avec un en-tête `location` et un en-tête `retry-after`. <br/>Pour les déclencheurs, l’en-tête `location` doit également contenir un paramètre de requête `triggerState`, qui est généralement un « timestamp ». Votre API peut utiliser cet identificateur pour vérifier quand l’application logique a été déclenchée pour la dernière fois. | 
||| 

Par exemple, pour vérifier périodiquement la présence de nouveaux fichiers dans votre service, vous pouvez générer un déclencheur d’interrogation qui possède ces comportements :

| La requête inclut `triggerState` ? | Réponse de l’API | 
| -------------------------------- | -------------| 
| Non | Renvoyez un état HTTP `202 ACCEPTED` plus un en-tête `location` avec `triggerState` défini sur l’heure actuelle et l’intervalle `retry-after` sur 15 secondes. | 
| Oui | Vérifiez la présence de fichiers ajoutés après le `DateTime` pour `triggerState` dans votre service. | 
||| 

| Nombre de fichiers trouvés | Réponse de l’API | 
| --------------------- | -------------| 
| Fichier unique | Renvoyez un état HTTP `200 OK` et la charge utile du contenu, mettez à jour `triggerState` vers `DateTime` pour le fichier renvoyé et définissez l’intervalle `retry-after` sur 15 secondes. | 
| Fichiers multiples | Renvoyez un seul fichier à la fois et un état HTTP `200 OK`, mettez à jour `triggerState` et définissez l’intervalle `retry-after` sur 0 seconde. </br>Ces étapes informent le moteur que des données supplémentaires sont disponibles, et qu’il doit demander immédiatement des données à partir de l’URL dans l’en-tête `location`. | 
| Aucun fichier | Renvoyez un état HTTP `202 ACCEPTED`, ne modifiez pas `triggerState` et définissez l’intervalle `retry-after` sur 15 secondes. | 
||| 

> [!TIP]
> Pour obtenir un exemple de modèle de déclencheur d’interrogation, consultez cet [exemple de contrôleur de déclencheur d’interrogation dans GitHub](https://github.com/logicappsio/LogicAppTriggersExample/blob/master/LogicAppTriggers/Controllers/PollTriggerController.cs).

<a name="webhook-triggers"></a>

### <a name="wait-and-listen-for-new-data-or-events-with-the-webhook-trigger-pattern"></a>Attendre et écouter les nouvelles données ou les événements avec le modèle de déclencheur Webhook

Un déclencheur Webhook est un *déclencheur d’émission* qui attend et écoute les nouvelles données ou les événements au point de terminaison de votre service. Si de nouvelles données ou un événement remplissent la condition spécifiée, le déclencheur est activé et crée une instance d’application logique, qui traite ensuite les données en tant qu’entrée.
Les déclencheurs Webhook agissent de façon très similaire aux [actions Webhook](#webhook-actions) précédemment décrites dans cette rubrique. Ils sont configurés avec les points de terminaison `subscribe` et `unsubscribe`. 

* Point de terminaison `subscribe` : quand vous ajoutez et enregistrez un déclencheur de webhook dans votre application logique, le moteur Logic Apps appelle le point de terminaison `subscribe`. Cette étape entraîne la création par l’application logique d’une URL de rappel que votre API enregistre. Lorsque de nouvelles données ou un événement remplissent la condition spécifiée, votre API envoie un rappel avec un élément POST HTTP à l’URL. La charge utile de contenu et les en-têtes sont transmis à l’application logique en tant qu’entrée.

* Point de terminaison `unsubscribe` : si le déclencheur de webhook ou l’intégralité de l’application logique est supprimé, le moteur Logic Apps appelle le point de terminaison `unsubscribe`. Votre API peut alors annuler l’inscription de l’URL de rappel, et arrêter tous les processus selon les besoins.

![Modèle de déclencheur Webhook](./media/logic-apps-create-api-app/custom-api-webhook-trigger-pattern.png)

Actuellement, le Concepteur d’application logique ne prend pas en charge la découverte des points de terminaison Webhook via Swagger. Pour ce modèle, vous devez donc ajouter un déclencheur [**Webhook**](../connectors/connectors-native-webhook.md) et spécifier l’URL, les en-têtes et le corps de votre requête. Voir également [Déclencheur HTTPWebhook](logic-apps-workflow-actions-triggers.md#httpwebhook-trigger). Pour un exemple de modèle Webhook, consultez cet [exemple de contrôleur de déclencheur Webhook dans GitHub](https://github.com/logicappsio/LogicAppTriggersExample/blob/master/LogicAppTriggers/Controllers/WebhookTriggerController.cs).

Voici d’autres conseils et remarques :

* Pour transmettre l’URL de rappel, vous pouvez utiliser la fonction de flux de travail `@listCallbackUrl()` dans un de ces champs en fonction des besoins.

* Pour éviter que les mêmes données soient traitées plusieurs fois, votre déclencheur doit nettoyer les données qu’il a déjà lues et transmises à l’application logique.

* Si vous possédez l’application logique et le service abonné, vous n’êtes pas obligé d’appeler le point de terminaison `unsubscribe` après l’appel de l’URL de rappel. Dans le cas contraire, le runtime Logic Apps doit appeler le point de terminaison `unsubscribe` pour signaler qu’aucun autre appel n’est attendu et autoriser le nettoyage des ressources côté serveur.

## <a name="improve-security-for-calls-to-your-apis-from-logic-apps"></a>Améliorer la sécurité des appels à vos API à partir des applications logiques

Après avoir créé vos API personnalisées, configurez l’authentification de vos API afin de pouvoir les appeler en toute sécurité à partir d’applications logiques. Découvrez [comment améliorer la sécurité des appels aux API personnalisées à partir d’applications logiques](../logic-apps/logic-apps-custom-api-authentication.md).

## <a name="deploy-and-call-your-apis"></a>Déployer et appeler les API

Après avoir configuré l’authentification, passez au déploiement de vos API. Découvrez [comment déployer et appeler des API personnalisées à partir d’applications logiques](../logic-apps/logic-apps-custom-api-host-deploy-call.md).

## <a name="publish-custom-apis-to-azure"></a>Publier des API personnalisées dans Azure

Pour mettre vos API personnalisées à la disposition d’autres utilisateurs Logic Apps dans Azure, vous devez ajouter un dispositif de sécurité et les inscrire en tant que connecteurs Logic Apps. Pour plus d’informations, consultez la page [Vue d’ensemble des connecteurs personnalisés](../logic-apps/custom-connector-overview.md). 

Pour mettre vos API personnalisées à la disposition de tous les utilisateurs de Logic Apps, Power Automate et Microsoft Power Apps, vous devez ajouter un dispositif de sécurité, inscrire vos API en tant que connecteurs Logic Apps et les proposer au [Programme Microsoft Azure Certified](https://azure.microsoft.com/marketplace/programs/certified/logic-apps/). 

## <a name="get-support"></a>Obtenir de l’aide

* Pour obtenir de l’aide concernant les API personnalisées, contactez [customapishelp@microsoft.com](mailto:customapishelp@microsoft.com).

* Pour toute question, consultez la [page de questions Microsoft Q&A pour Azure Logic Apps](/answers/topics/azure-logic-apps.html).

* Afin de contribuer à améliorer Logic Apps, votez pour des idées ou soumettez-en sur le [site de commentaires des utilisateurs Logic Apps](https://aka.ms/logicapps-wish). 

## <a name="next-steps"></a>Étapes suivantes

* [Gérer les erreurs et exceptions](../logic-apps/logic-apps-exception-handling.md)
* [Appeler, déclencher ou imbriquer des applications logiques via des points de terminaison HTTP](../logic-apps/logic-apps-http-endpoint.md)
* [Mesure de l’utilisation des actions et des déclencheurs](../logic-apps/logic-apps-pricing.md)

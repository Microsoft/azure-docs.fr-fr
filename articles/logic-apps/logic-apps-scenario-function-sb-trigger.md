---
title: Appeler des applications logiques avec Azure Functions
description: Appeler ou déclencher des applications logiques à l’aide d’Azure Functions et d’Azure Service Bus
services: logic-apps
ms.suite: integration
ms.reviewer: jehollan, klam, logicappspm
ms.topic: article
ms.date: 11/08/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 25f761d85ebfd0ac16f182941c5b5c29636066bf
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000332"
---
# <a name="call-or-trigger-logic-apps-by-using-azure-functions-and-azure-service-bus"></a>Appeler ou déclencher des applications logiques à l’aide d’Azure Functions et d’Azure Service Bus

Vous pouvez utiliser [Azure Functions](../azure-functions/functions-overview.md) afin de déclencher une application logique quand vous devez déployer un écouteur ou une tâche de longue durée. Par exemple, vous pouvez créer une fonction qui écoute sur une file d’attente [Azure Service Bus](../service-bus-messaging/service-bus-messaging-overview.md) et qui déclenche immédiatement une application logique en tant que déclencheur d’émission.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Un espace de noms Azure Service Bus. Si vous n’avez pas d’espace de noms, [créez d’abord votre espace de noms](../service-bus-messaging/service-bus-create-namespace-portal.md).

* Une application de fonction Azure, qui est un conteneur pour vos fonctions. Si vous n’avez pas d’application de fonction, [créez d’abord votre application de fonction](../azure-functions/functions-create-first-azure-function.md), puis veillez à sélectionner .NET comme pile d’exécution.

* Des connaissances de base en [création d’applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md)

## <a name="create-logic-app"></a>Créer une application logique

Pour ce scénario, vous disposez d’une fonction exécutant chaque application logique que vous voulez déclencher. Créez d’abord une application logique qui démarre avec un déclencheur de requête HTTP. La fonction appelle ce point de terminaison chaque fois qu’un message de la file d’attente est reçu.

1. Connectez-vous au [Portail Azure](https://portal.azure.com) et créez une application logique vide.

   Si vous débutez avec les applications logiques, consultez le guide de [Démarrage rapide : Créer votre première application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md).

1. Dans la zone de recherche, entrez `http request`. Dans la liste des déclencheurs, sélectionnez **Lors de la réception d’une requête HTTP**.

   ![Sélectionner le déclencheur](./media/logic-apps-scenario-function-sb-trigger/when-http-request-received-trigger.png)

   Avec le déclencheur Requête, vous pouvez éventuellement entrer un schéma JSON à utiliser avec le message de file d’attente. Les schémas JSON aident le Concepteur d’application logique à comprendre la structure des données d’entrée et facilitent l’utilisation des sorties dans votre workflow.

1. Pour spécifier un schéma, entrez-le dans la zone **Schéma JSON de corps de la demande**, par exemple :

   ![Spécifier un schéma JSON](./media/logic-apps-scenario-function-sb-trigger/when-http-request-received-trigger-schema.png)

   Si vous n’avez pas de schéma, mais un exemple de charge utile au format JSON, vous pouvez générer un schéma à partir de cette charge utile.

   1. Dans le déclencheur de requête, sélectionnez **Utiliser l’exemple de charge utile pour générer le schéma**.

   1. Sous **Entrer ou coller un exemple de charge utile JSON**, entrez votre exemple de charge utile, puis sélectionnez **Terminé**.

      ![Entrer un exemple de charge utile](./media/logic-apps-scenario-function-sb-trigger/enter-sample-payload.png)

   Cet exemple de charge utile génère ce schéma, qui apparaît dans le déclencheur :

   ```json
   {
      "type": "object",
      "properties": {
         "address": {
            "type": "object",
            "properties": {
               "number": {
                  "type": "integer"
               },
               "street": {
                  "type": "string"
               },
               "city": {
                  "type": "string"
               },
               "postalCode": {
                  "type": "integer"
               },
               "country": {
                  "type": "string"
               }
            }
         }
      }
   }
   ```

1. Ajoutez les autres actions que vous voulez exécuter après réception du message de file d’attente.

   Par exemple, vous pouvez envoyer un e-mail avec le connecteur Office 365 Outlook.

1. Enregistrez votre application logique, ce qui génère l’URL de rappel du déclencheur dans cette application logique. Vous utilisez ensuite cette URL de rappel dans le code pour le déclencheur de file d’attente Azure Service Bus.

   L’URL apparaît s’affiche dans la propriété **HTTP POST URL**.

   ![URL de rappel générée pour le déclencheur](./media/logic-apps-scenario-function-sb-trigger/callback-URL-for-trigger.png)

## <a name="create-a-function"></a>Créer une fonction

Maintenant, créez la fonction qui agit comme déclencheur et écoute la file d’attente.

1. Sur le Portail Azure, ouvrez et développez votre application de fonction, si ce n’est pas déjà le cas. 

1. Sous le nom de votre application de fonction, développez **Fonctions**. Dans le volet **Fonctions**, sélectionnez **Nouvelle fonction**.

   ![Développer « Fonctions » et sélectionner « Nouvelle fonction »](./media/logic-apps-scenario-function-sb-trigger/add-new-function-to-function-app.png)

1. Sélectionnez ce modèle selon que vous avez créé une application de fonction où vous avez sélectionné .NET comme pile d’exécution ou que vous utilisez une application de fonction existante.

   * Pour les nouvelles applications de fonction, sélectionnez ce modèle : **Déclencheur File d’attente Service Bus**

     ![Sélectionner un modèle pour la nouvelle application de fonction](./media/logic-apps-scenario-function-sb-trigger/current-add-queue-trigger-template.png)

   * Pour une application de fonction existante, sélectionnez ce modèle : **Déclencheur File d’attente Service Bus - C#**

     ![Sélectionner un modèle pour une application de fonction existante](./media/logic-apps-scenario-function-sb-trigger/legacy-add-queue-trigger-template.png)

1. Dans le volet **Déclencheur File d’attente Service Bus**, fournissez un nom pour votre déclencheur, puis configurez la **Connexion Service Bus** pour la file d’attente, qui utilise l’écouteur `OnMessageReceive()` du Kit de développement logiciel (SDK) Azure Service Bus, puis sélectionnez **Créer**.

1. Écrivez une fonction de base pour appeler le point de terminaison d’application logique créé précédemment en utilisant le message de la file d’attente comme déclencheur. Avant d’écrire votre fonction, passez en revue les considérations suivantes :

   * Cet exemple utilise un type de contenu de message `application/json`, mais vous pouvez le modifier si nécessaire.
   
   * En raison de la possibilité d’exécution simultanée de fonctions, de volumes élevés ou de charges lourdes, évitez d’instancier la [classe HTTPClient](/dotnet/api/system.net.http.httpclient) avec l’instruction `using` et de créer directement des instances HTTPClient par requête. Pour plus d’informations, consultez [Utiliser HttpClientFactory pour implémenter des requêtes HTTP résilientes](/dotnet/architecture/microservices/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests#issues-with-the-original-httpclient-class-available-in-net-core).
   
   * Si possible, réutilisez l’instance de clients HTTP. Pour plus d’informations, consultez la rubrique [Gérer les connexions dans Azure Functions](../azure-functions/manage-connections.md).

   Cet exemple utilise la [méthode `Task.Run`](/dotnet/api/system.threading.tasks.task.run) en mode [asynchrone](/dotnet/csharp/language-reference/keywords/async). Pour plus d’informations, consultez l’article [Programmation asynchrone avec async et await](/dotnet/csharp/programming-guide/concepts/async/).

   ```csharp
   using System;
   using System.Threading.Tasks;
   using System.Net.Http;
   using System.Text;

   // Can also fetch from App Settings or environment variable
   private static string logicAppUri = @"https://prod-05.westus.logic.azure.com:443/workflows/<remaining-callback-URL>";

   // Reuse the instance of HTTP clients if possible: https://docs.microsoft.com/azure/azure-functions/manage-connections
   private static HttpClient httpClient = new HttpClient();

   public static async Task Run(string myQueueItem, TraceWriter log) 
   {
      log.Info($"C# ServiceBus queue trigger function processed message: {myQueueItem}");
      var response = await httpClient.PostAsync(logicAppUri, new StringContent(myQueueItem, Encoding.UTF8, "application/json")); 
   }
   ```

1. Pour tester la fonction, ajoutez un message de file d’attente à l’aide d’un outil comme [l’Explorateur Service Bus](https://github.com/paolosalvatori/ServiceBusExplorer).

   L’application logique se déclenche dès que la fonction reçoit le message.

## <a name="next-steps"></a>Étapes suivantes

* [Appeler, déclencher ou imbriquer des workflows avec des points de terminaison HTTP](../logic-apps/logic-apps-http-endpoint.md)

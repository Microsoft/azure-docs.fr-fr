---
title: Créer une fonction qui s’intègre avec Azure Logic Apps
description: Créez une fonction qui s’intègre à Azure Logic Apps et à Azure Cognitive Services pour catégoriser les sentiments de tweets et envoyer des notifications quand le sentiment est peu favorable.
author: craigshoemaker
ms.assetid: 60495cc5-1638-4bf0-8174-52786d227734
ms.topic: tutorial
ms.date: 04/27/2020
ms.author: cshoe
ms.custom: devx-track-csharp, mvc, cc996988-fb4f-47
ms.openlocfilehash: feb6b36f8e5e7bbec83d8882552484f68abfd56d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91537750"
---
# <a name="create-a-function-that-integrates-with-azure-logic-apps"></a>Créer une fonction qui s’intègre avec Azure Logic Apps

Azure Functions s’intègre avec Azure Logic Apps dans le Concepteur d’applications logiques. Cette intégration vous permet d’utiliser la puissance des fonctions dans les orchestrations avec d’autres services Azure et services tiers. 

Ce didacticiel vous montre comment utiliser Azure Functions avec Logic Apps et Cognitive Services sur Azure pour exécuter une analyse des sentiments sur les billets Twitter. Une fonction déclenchée via HTTP classe les tweets en vert, jaune ou rouge selon le score de sentiments. Un courrier électronique est envoyé lors de la détection de sentiments négatifs. 

![Image : deux premières étapes de l’application dans le Concepteur d’applications logiques](media/functions-twitter-email/00-logic-app-overview.png)

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créez une ressource API Cognitive Services.
> * Créer une fonction qui classe le sentiment des tweets.
> * Créer une application logique qui se connecte à Twitter.
> * Ajouter la détection des sentiments à l’application logique. 
> * Connecter l’application logique à la fonction.
> * Envoyer un courrier électronique en fonction de la réponse de la fonction.

## <a name="prerequisites"></a>Prérequis

+ Un compte [Twitter](https://twitter.com/) actif. 
+ Un compte [Outlook.com](https://outlook.com/) (pour l’envoi de notifications).

> [!NOTE]
> Si vous souhaitez utiliser le connecteur Gmail, seuls les comptes professionnels G-Suite peuvent utiliser ce connecteur sans restriction dans Logic Apps. Si vous disposez d’un compte de consommateur Gmail, vous pouvez utiliser le connecteur uniquement avec certains services et applications approuvés par Google, ou vous pouvez [créer une application cliente Google pour servir lors de l’authentification dans votre connecteur Gmail](/connectors/gmail/#authentication-and-bring-your-own-application). Pour plus d’informations, consultez [Stratégies de confidentialité et de sécurité des données pour les connecteurs Google dans Azure Logic Apps](../connectors/connectors-google-data-security-privacy-policy.md).

+ Cet article utilise comme point de départ les ressources créées dans [Créer votre première fonction à partir du portail Azure](functions-create-first-azure-function.md).
Si vous ne l’avez pas déjà fait, suivez ces étapes pour créer votre Function App.

## <a name="create-a-cognitive-services-resource"></a>Créer une ressource Cognitive Services

Les API Cognitive Services sont disponibles dans Azure en tant que ressources individuelles. Utilisez l’API Analyse de texte pour détecter le sentiment des tweets en cours d’analyse.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

2. Cliquez sur **Créer une ressource** en haut à gauche du portail Azure.

3. Cliquez sur **IA + Machine Learning** > **Analyse de texte**. Ensuite, utilisez les paramètres indiqués dans le tableau pour créer la ressource.

    ![Créer une page de ressource Cognitive](media/functions-twitter-email/01-create-text-analytics.png)

    | Paramètre      |  Valeur suggérée   | Description                                        |
    | --- | --- | --- |
    | **Nom** | MyCognitiveServicesAccnt | Choisissez un nom de compte unique. |
    | **Lieu** | USA Ouest | Utilisez l’emplacement le plus proche de vous. |
    | **Niveau tarifaire** | F0 | Démarrez avec le niveau le plus bas. Si vous manquez d’appels, choisissez un niveau plus élevé.|
    | **Groupe de ressources** | myResourceGroup | Utilisez le même groupe de ressources pour tous les services de ce didacticiel.|

4. Cliquez sur **Créer** pour créer votre ressource. 

5. Cliquez sur **Vue d’ensemble** et copiez la valeur du **Point de terminaison** dans un éditeur de texte. Cette valeur est utilisée lors de la création d’une connexion à l’API Cognitive Services.

    ![Paramètres de Cognitive Services](media/functions-twitter-email/02-cognitive-services.png)

6. Dans la colonne de navigation gauche, cliquez sur **Clés**, puis copiez la valeur de **Clé 1** et collez-la dans un éditeur de texte. Cette clé vous permet de connecter l’application logique à votre API Cognitive Services. 
 
    ![Clés Cognitive Services](media/functions-twitter-email/03-cognitive-serviecs-keys.png)

## <a name="create-the-function-app"></a>Créer l’application de fonction

Azure Functions offre un excellent moyen de se décharger des tâches de traitement dans un flux de travail d’applications logiques. Ce didacticiel utilise une fonction déclenchée via HTTP pour traiter des scores de sentiments de tweet à partir de Cognitive Services et renvoie une valeur de catégorie.  

[!INCLUDE [Create function app Azure portal](../../includes/functions-create-function-app-portal.md)]

## <a name="create-an-http-trigger-function"></a>Créer une fonction de déclencheur HTTP  

1. Dans le menu de gauche de la fenêtre **Fonctions**, sélectionnez **Fonctions**, puis **Ajouter** dans le menu supérieur.

2. Dans la fenêtre **Nouvelle fonction**, sélectionnez **Déclencheur HTTP**.

    ![Choisir une fonction de déclencheur HTTP](./media/functions-twitter-email/06-function-http-trigger.png)

3. Dans la page **Nouvelle fonction**, sélectionnez **Créer une fonction**.

4. Dans votre nouvelle fonction de déclencheur HTTP, sélectionnez **Code + test** dans le menu de gauche, remplacez le contenu du fichier `run.csx` par le code suivant, puis sélectionnez **Enregistrer** :

    ```csharp
    #r "Newtonsoft.Json"
    
    using System;
    using System.Net;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Logging;
    using Microsoft.Extensions.Primitives;
    using Newtonsoft.Json;
    
    public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
    {
        string category = "GREEN";
    
        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        log.LogInformation(string.Format("The sentiment score received is '{0}'.", requestBody));
    
        double score = Convert.ToDouble(requestBody);
    
        if(score < .3)
        {
            category = "RED";
        }
        else if (score < .6) 
        {
            category = "YELLOW";
        }
    
        return requestBody != null
            ? (ActionResult)new OkObjectResult(category)
            : new BadRequestObjectResult("Please pass a value on the query string or in the request body");
    }
    ```

    Le code de cette fonction renvoie une catégorie de couleur en fonction du score des sentiments reçu dans la requête. 

5. Pour tester la fonction, sélectionnez **Test** dans le menu supérieur. Sous l’onglet **Entrée**, entrez la valeur `0.2` dans le **Corps**, puis sélectionnez **Exécuter**. La valeur **RED** est renvoyée dans le **contenu de la réponse HTTP** sous l’onglet **Sortie**. 

    :::image type="content" source="./media/functions-twitter-email/07-function-test.png" alt-text="Définir les paramètres du proxy":::

Vous disposez maintenant d’une fonction permettant de classer les scores des sentiments. Ensuite, créez une application logique qui intègre votre fonction dans vos API Twitter et Cognitive Services. 

## <a name="create-a-logic-app"></a>Créer une application logique   

1. Dans le portail Azure, en haut à gauche, cliquez sur le bouton **Créer une ressource**.

2. Cliquez sur **Web** > **Application logique**.
 
3. Ensuite, saisissez une valeur pour le **Nom**, comme `TweetSentiment`, et utilisez les paramètres spécifiés dans le tableau.

    ![Créer une application logique dans le portail Azure](./media/functions-twitter-email/08-logic-app-create.png)

    | Paramètre      |  Valeur suggérée   | Description                                        |
    | ----------------- | ------------ | ------------- |
    | **Nom** | TweetSentiment | Choisissez un nom approprié pour votre application. |
    | **Groupe de ressources** | myResourceGroup | Choisissez le même groupe de ressources existant que précédemment. |
    | **Lieu** | USA Est | Choisissez un emplacement proche de vous. |    

4. Une fois que vous avez entré les valeurs de paramètre appropriées, cliquez sur **Créer** pour créer votre application logique. 

5. Une fois que l’application est créée, cliquez sur votre nouvelle application logique épinglée au tableau de bord. Dans le Concepteur d’applications logiques, faites défiler vers le bas, puis cliquez sur le modèle **Application logique vide**. 

    ![Modèle d’applications logiques vides](media/functions-twitter-email/09-logic-app-create-blank.png)

Vous pouvez maintenant utiliser le Concepteur d’applications logiques pour ajouter des services et des déclencheurs à votre application.

## <a name="connect-to-twitter"></a>Se connecter à Twitter

Tout d’abord, créez une connexion à votre compte Twitter. L’application logique interroge les tweets qui déclenchent l’exécution de l’application.

1. Dans le concepteur, cliquez sur le service **Twitter**, puis cliquez sur le déclencheur **Lorsqu’un nouveau tweet est publié**. Connectez-vous à votre compte Twitter et autorisez Logic Apps à utiliser votre compte.

2. Utilisez les paramètres de déclencheur Twitter spécifiés dans le tableau. 

    ![Paramètres du connecteur Twitter](media/functions-twitter-email/10-tweet-settings.png)

    | Paramètre      |  Valeur suggérée   | Description                                        |
    | ----------------- | ------------ | ------------- |
    | **Texte de recherche** | #Azure | Utilisez un mot-dièse suffisamment populaire pour générer de nouveaux tweets dans l’intervalle sélectionné. Lors de l’utilisation du niveau Gratuit, si votre mot-dièse est trop populaire, vous pouvez rapidement épuiser le quota de transactions de votre API Cognitive Services. |
    | **Intervalle** | 15 | Le temps écoulé entre les requêtes de Twitter, en unités de fréquence. |
    | **Fréquence** | Minute | L’unité de fréquence utilisée pour l’interrogation de Twitter.  |

3.  Cliquez sur **Enregistrer** pour vous connecter à votre compte Twitter. 

Votre application est maintenant connectée à Twitter. Ensuite, connectez-vous à l’analyse de texte pour détecter les sentiments des tweets collectés.

## <a name="add-sentiment-detection"></a>Ajouter la détection de sentiments

1. Cliquez sur **Nouvelle étape**, puis sélectionnez **Ajouter une action**.

2. Dans **Choisir une action**, saisissez **Analyse de texte**, puis cliquez sur l’action **Détecter le sentiment**.
    
    ![Capture d’écran montrant la section « Choisir une action » avec « Analyse de texte » dans la zone de recherche et l’action « Détecter le sentiment » sélectionnée. ](media/functions-twitter-email/11-detect-sentiment.png)

3. Tapez un nom de connexion, comme `MyCognitiveServicesConnection`, collez la clé de votre API Cognitive Services et le point de terminaison Cognitive Services mis de côté dans un éditeur de texte, puis cliquez sur **Créer**.

    ![Nouvelle étape, puis Ajouter une action](media/functions-twitter-email/12-connection-settings.png)

4. Ensuite, saisissez **Texte du Tweet** dans la zone de texte, puis cliquez sur **Nouvelle étape**.

    ![Définir le texte à analyser](media/functions-twitter-email/13-analyze-tweet-text.png)

Maintenant que la détection de sentiment est configurée, vous pouvez ajouter une connexion à votre fonction qui utilise la sortie du score de sentiments.

## <a name="connect-sentiment-output-to-your-function"></a>Connecter la sortie des sentiments à votre fonction

1. Dans le Concepteur d’applications logiques, cliquez sur **Nouvelle étape** > **Ajouter une action**, filtrer sur **Azure Functions** et cliquez sur **Choisir une fonction Azure**.

    ![Detect Sentiment (Détecter le sentiment)](media/functions-twitter-email/14-azure-functions.png)
  
4. Sélectionnez l’application de fonction que vous avez créée précédemment.

    ![Capture d’écran montrant la section « Choisir une action » avec une application de fonction sélectionnée.](media/functions-twitter-email/15-select-function.png)

5. Sélectionnez la fonction que vous avez créée pour ce didacticiel.

    ![Sélectionner une fonction](media/functions-twitter-email/16-select-function.png)

4. Dans **Corps de la requête**, cliquez sur **Score**, puis sur **Enregistrer**.

    ![Score](media/functions-twitter-email/17-function-input-score.png)

À présent, votre fonction est déclenchée lorsqu’un score de sentiment est envoyé à partir de l’application logique. La fonction renvoie une catégorie de couleur à l’application logique. Ensuite, ajoutez une notification par courrier électronique qui est envoyée lorsqu’une valeur de sentiment **RED** est renvoyée par la fonction. 

## <a name="add-email-notifications"></a>Ajouter des notifications par courrier électronique

La dernière partie du flux de travail consiste à déclencher un courrier électronique lorsque le sentiment est évalué comme _RED_. Cet article utilise un connecteur Outlook.com. Vous pouvez effectuer des étapes similaires pour utiliser un connecteur Gmail ou Office 365 Outlook.   

1. Dans le Concepteur d’applications logiques, cliquez sur **Nouvelle étape** > **Ajouter une condition**. 

    ![Ajouter une condition à l’application logique.](media/functions-twitter-email/18-add-condition.png)

2. Cliquez sur **Choisir une valeur**, puis cliquez sur **Corps**. Sélectionnez **Est égal à**, cliquez sur **Choisir une valeur** et saisissez `RED`, puis cliquez sur **Enregistrer**. 

    ![Choisir une action pour la condition.](media/functions-twitter-email/19-condition-settings.png)    

3. Dans **SI OUI**, cliquez sur **Ajouter une action**, recherchez `outlook.com`, cliquez sur **Envoyer un e-mail**, puis connectez-vous à votre compte Outlook.com.

    ![Capture d’écran montrant la section « IF TRUE » avec « outlook.com » dans la zone de recherche et l’action « Envoyer un e-mail » sélectionnée.](media/functions-twitter-email/20-add-outlook.png)

    > [!NOTE]
    > Si vous n’avez pas de compte Outlook.com, vous pouvez choisir un autre connecteur, comme Gmail ou Office 365 Outlook

4. Dans l’action **Envoyer un courrier électronique**, utilisez les paramètres de messagerie indiqués dans le tableau. 

    ![Configurez le courrier électronique pour l’action Envoyer un courrier électronique.](media/functions-twitter-email/21-configure-email.png)
    
| Paramètre      |  Valeur suggérée   | Description  |
| ----------------- | ------------ | ------------- |
| **To** | Saisissez votre adresse de messagerie | L’adresse de messagerie qui reçoit la notification. |
| **Subject** | Sentiment de tweet négatif détecté  | La ligne d’objet de la notification par courrier électronique.  |
| **Corps** | Texte du tweet, Emplacement | Cliquez sur les paramètres **Texte du tweet** et **Emplacement**. |

1. Cliquez sur **Enregistrer**.

Maintenant que le flux de travail est terminé, vous pouvez activer l’application logique et observer la fonction en action.

## <a name="test-the-workflow"></a>Tester le flux de travail

1. Dans le Concepteur d’application logique, cliquez sur **Exécuter** pour démarrer l’application.

2. Dans la colonne de gauche, cliquez sur **Vue d’ensemble** pour connaître l’état de l’application logique. 
 
    ![État d’exécution de l’application logique](media/functions-twitter-email/22-execution-history.png)

3. (Facultatif) Cliquez sur une des exécutions pour afficher les détails de l’exécution.

4. Accédez à votre fonction, affichez les journaux d’activité et vérifiez que les valeurs des sentiments ont été reçues et traitées.
 
    ![Affichez les journaux d’activité de fonction](media/functions-twitter-email/sent.png)

5. Lorsqu’un sentiment potentiellement négatif est détecté, vous recevez un courrier électronique. Si vous n’avez pas reçu de courrier électronique, vous pouvez modifier le code de fonction pour renvoyer RED à chaque fois :

    ```csharp
    return (ActionResult)new OkObjectResult("RED");
    ```

    Après avoir vérifié les notifications par courrier électronique, rétablissez le code d’origine :

    ```csharp
    return requestBody != null
        ? (ActionResult)new OkObjectResult(category)
        : new BadRequestObjectResult("Please pass a value on the query string or in the request body");
    ```

    > [!IMPORTANT]
    > Une fois que vous avez terminé ce didacticiel, désactivez l’application logique. En désactivant l’application, vous évitez d’être facturé pour les exécutions et d’épuiser les transactions dans votre API Cognitive Services.

Maintenant, vous savez à quel point il est facile d’intégrer des fonctions dans un flux de travail Logic Apps.

## <a name="disable-the-logic-app"></a>Désactiver l’application logique

Pour désactiver l’application logique, cliquez sur **Vue d’ensemble**, puis cliquez sur **Désactiver** en haut de l’écran. La désactivation de l’application empêche son exécution et évite d’entraîner des frais, sans supprimer l’application.

![Journaux d’activité de fonction](media/functions-twitter-email/disable-logic-app.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Créez une ressource API Cognitive Services.
> * Créer une fonction qui classe le sentiment des tweets.
> * Créer une application logique qui se connecte à Twitter.
> * Ajouter la détection des sentiments à l’application logique. 
> * Connecter l’application logique à la fonction.
> * Envoyer un courrier électronique en fonction de la réponse de la fonction.

Passez au didacticiel suivant pour apprendre à créer une API sans serveur pour votre fonction.

> [!div class="nextstepaction"] 
> [Créer une API sans serveur à l’aide d’Azure Functions](functions-create-serverless-api.md)

Pour plus d’informations sur Logic Apps, voir [Azure Logic Apps](../logic-apps/logic-apps-overview.md).

---
title: Tutoriel - Exécuter Azure Functions dans des travaux Azure Stream Analytics
description: Dans ce didacticiel, vous apprenez à configurer Azure Functions comme récepteur de sortie pour les travaux Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: tutorial
ms.custom: mvc, devx-track-csharp
ms.date: 01/27/2020
ms.openlocfilehash: bb2eb36e4116c17efb20946b0da4586678838f3b
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96862001"
---
# <a name="tutorial-run-azure-functions-from-azure-stream-analytics-jobs"></a>Tutoriel : Exécuter Azure Functions à partir des travaux Azure Stream Analytics 

Vous pouvez exécuter Azure Functions à partir d’Azure Stream Analytics en configurant Functions comme l’un des récepteurs de sortie de la tâche Stream Analytics. Functions offre une expérience de calcul à la demande basée sur des événements, qui vous permet d’implémenter le code qui est déclenché par les événements qui se produisent dans Azure ou dans des services tiers. Comme ce logiciel peut répondre à des déclencheurs, il constitue l’outil de sortie logique pour des travaux Stream Analytics.

Stream Analytics appelle Functions via des déclencheurs HTTP. L’adaptateur de sortie de Functions permet aux utilisateurs de connecter Functions à Stream Analytics, de telle sorte que les événements puissent être déclenchés en fonction des requêtes Stream Analytics. 

> [!NOTE]
> Il n’est pas possible d’établir une connexion à Azure Functions au sein d’un réseau virtuel à partir d’une tâche Stream Analytics qui s’exécute dans un cluster multilocataire.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer et exécuter un travail Stream Analytics
> * Créer une instance Cache Redis Azure
> * Création d’une fonction Azure
> * Consulter les résultats dans Cache Redis Azure

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="configure-a-stream-analytics-job-to-run-a-function"></a>Configurer un travail Stream Analytics pour exécuter une fonction 

Cette section explique comment configurer un travail Stream Analytics pour exécuter une fonction qui écrit des données dans Cache Redis Azure. Le travail Stream Analytics lit les événements d’Azure Event Hubs et exécute une requête qui appelle la fonction. Cette fonction lit les données à partir du travail Stream Analytics et les écrit dans Cache Redis Azure.

![Schéma montrant les relations entre les services Azure](./media/stream-analytics-with-azure-functions/image1.png)

## <a name="create-a-stream-analytics-job-with-event-hubs-as-input"></a>Créer un travail Stream Analytics en utilisant Event Hubs comme entrée

Suivez le didacticiel [Détection des fraudes en temps réel](stream-analytics-real-time-fraud-detection.md) pour créer un concentrateur d’événements, démarrer l’application de générateur d’événements et créer un travail Stream Analytics. Ignorez les étapes de création de la requête et de la sortie. Consultez plutôt les sections suivantes pour configurer la sortie Azure Functions.

## <a name="create-an-azure-cache-for-redis-instance"></a>Créer une instance Cache Redis Azure

1. Créez un cache dans Cache Redis Azure en suivant les étapes décrites dans la section [Create a cache](../azure-cache-for-redis/cache-dotnet-how-to-use-azure-redis-cache.md#create-a-cache) (Créer un cache).  

2. Une fois le cache créé, sous **Paramètres**, sélectionnez **Clés d’accès**. Notez la **chaîne de connexion primaire**.

   ![Capture d’écran de la chaîne de connexion de Cache Redis Azure](./media/stream-analytics-with-azure-functions/image2.png)

## <a name="create-a-function-in-azure-functions-that-can-write-data-to-azure-cache-for-redis"></a>Créer une fonction dans Azure Functions qui permet d’écrire des données dans Cache Redis Azure

1. Consultez la section [Créer une application de fonction](../azure-functions/functions-create-first-azure-function.md#create-a-function-app) de la documentation Functions. Cette section vous explique comment créer une application de fonction et une [fonction déclenchée par HTTP dans Azure Functions](../azure-functions/functions-create-first-azure-function.md#create-function) en utilisant le langage C#.  

2. Accédez à la fonction **run.csx**. Mettez-la à jour avec le code suivant. Remplacez **« \<your Azure Cache for Redis connection string goes here\> »** par la chaîne de connexion principale d’Azure Cache pour Redis que vous avez récupérée dans la section précédente. 

    ```csharp
    using System;
    using System.Net;
    using System.Threading.Tasks;
    using StackExchange.Redis;
    using Newtonsoft.Json;
    using System.Configuration;

    public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
    {
        log.Info($"C# HTTP trigger function processed a request. RequestUri={req.RequestUri}");
    
        // Get the request body
        dynamic dataArray = await req.Content.ReadAsAsync<object>();

        // Throw an HTTP Request Entity Too Large exception when the incoming batch(dataArray) is greater than 256 KB. Make sure that the size value is consistent with the value entered in the Stream Analytics portal.

        if (dataArray.ToString().Length > 262144)
        {
            return new HttpResponseMessage(HttpStatusCode.RequestEntityTooLarge);
        }
        var connection = ConnectionMultiplexer.Connect("<your Azure Cache for Redis connection string goes here>");
        log.Info($"Connection string.. {connection}");
    
        // Connection refers to a property that returns a ConnectionMultiplexer
        IDatabase db = connection.GetDatabase();
        log.Info($"Created database {db}");
    
        log.Info($"Message Count {dataArray.Count}");

        // Perform cache operations using the cache object. For example, the following code block adds few integral data types to the cache
        for (var i = 0; i < dataArray.Count; i++)
        {
            string time = dataArray[i].time;
            string callingnum1 = dataArray[i].callingnum1;
            string key = time + " - " + callingnum1;
            db.StringSet(key, dataArray[i].ToString());
            log.Info($"Object put in database. Key is {key} and value is {dataArray[i].ToString()}");
       
            // Simple get of data types from the cache
            string value = db.StringGet(key);
            log.Info($"Database got: {value}");
        }

        return req.CreateResponse(HttpStatusCode.OK, "Got");
    }

   ```

   Lorsque Stream Analytics reçoit une erreur « HTTP Request Entity Too Large » de la part de la fonction, il réduit la taille des lots envoyés à Functions. Le code suivant garantit que Stream Analytics n’envoie pas de lots trop volumineux. Vérifiez également que les valeurs de taille et de nombre de lots maximum utilisées dans la fonction correspondent à celles qui ont été saisies dans le portail Stream Analytics.

    ```csharp
    if (dataArray.ToString().Length > 262144)
        {
            return new HttpResponseMessage(HttpStatusCode.RequestEntityTooLarge);
        }
   ```

3. Dans un éditeur de texte de votre choix, créez un fichier JSON nommé **project.json**. Collez le code suivant et enregistrez-le sur votre ordinateur local. Ce fichier contient les dépendances du package NuGet dont a besoin la fonction C#.  
   
    ```json
    {
        "frameworks": {
            "net46": {
                "dependencies": {
                    "StackExchange.Redis":"1.1.603",
                    "Newtonsoft.Json": "9.0.1"
                }
            }
        }
    }

   ```
 
4. Revenez au portail Azure. Sous l’onglet **Fonctionnalités de la plateforme**, accédez à votre fonction. Sous **Outils de développement**, sélectionnez **Éditeur App Service**. 
 
   ![Capture d’écran montrant l’onglet Fonctionnalités de la plateforme, avec Éditeur App Service sélectionné.](./media/stream-analytics-with-azure-functions/image3.png)

5. Dans l’éditeur App Service, cliquez avec le bouton droit sur votre répertoire racine et téléchargez le fichier **project.json**. Une fois le téléchargement terminé, actualisez la page. Vous devriez maintenant voir un fichier généré automatiquement appelé **project.lock.json**. Ce fichier généré automatiquement contient des références aux fichiers .dll spécifiés dans le fichier project.json.  

   ![Capture d’écran montrant l’option Charger des fichiers sélectionnée dans le menu.](./media/stream-analytics-with-azure-functions/image4.png)

## <a name="update-the-stream-analytics-job-with-the-function-as-output"></a>Mettre à jour le travail Stream Analytics en utilisant la fonction comme sortie

1. Dans le portail Azure, ouvrez votre travail Stream Analytics.  

2. Accédez à votre fonction et sélectionnez **Vue d’ensemble** > **Sorties** > **Ajouter**. Pour ajouter une nouvelle sortie, sélectionnez **Azure Function** pour l’option de récepteur. L’adaptateur de sortie Functions a les propriétés suivantes :  

   |**Nom de la propriété**|**Description**|
   |---|---|
   |Alias de sortie| Nom convivial que vous utilisez dans la requête du travail pour faire référence à la sortie. |
   |Option d’importation| Vous pouvez utiliser la fonction de l’abonnement actuel ou renseigner manuellement les paramètres si la fonction se trouve dans un autre abonnement. |
   |Function App| Nom de votre application Functions. |
   |Fonction| Nom de la fonction dans votre application Functions (nom de votre fonction run.csx).|
   |Taille de lot maximale|Définit la taille maximale de chaque lot de sortie envoyé à votre fonction, en octets. Par défaut, cette valeur est définie à 262 144 octets (256 Ko).|
   |Nombre maximal de lots|Spécifie le nombre maximal d’événements dans chaque lot envoyé à la fonction. La valeur par défaut est 100. Cette propriété est facultative.|
   |Clé|Vous permet d’utiliser une fonction d’un autre abonnement. Indiquez la valeur de la clé pour accéder à votre fonction. Cette propriété est facultative.|

3. Indiquez un nom pour l’alias de sortie. Dans ce didacticiel, il est nommé **saop1**, mais vous pouvez utiliser n’importe quel nom de votre choix. Renseignez les autres détails.

4. Ouvrez votre travail Stream Analytics et mettez à jour la requête comme suit. Si vous n’avez pas nommé votre récepteur de sortie **saop1**, n’oubliez pas de modifier le nom dans la requête.  

   ```sql
    SELECT
            System.Timestamp as Time, CS1.CallingIMSI, CS1.CallingNum as CallingNum1,
            CS2.CallingNum as CallingNum2, CS1.SwitchNum as Switch1, CS2.SwitchNum as Switch2
        INTO saop1
        FROM CallStream CS1 TIMESTAMP BY CallRecTime
           JOIN CallStream CS2 TIMESTAMP BY CallRecTime
            ON CS1.CallingIMSI = CS2.CallingIMSI AND DATEDIFF(ss, CS1, CS2) BETWEEN 1 AND 5
        WHERE CS1.SwitchNum != CS2.SwitchNum
   ```

5. Démarrez l’application telcodatagen.exe en exécutant la commande suivante dans la ligne de commande. La commande utilise le format `telcodatagen.exe [#NumCDRsPerHour] [SIM Card Fraud Probability] [#DurationHours]`.  
   
   ```cmd
   telcodatagen.exe 1000 0.2 2
   ```
    
6.  Démarrez le travail Stream Analytics.

## <a name="check-azure-cache-for-redis-for-results"></a>Consulter les résultats dans Cache Redis Azure

1. Accédez au Portail Azure et recherchez votre instance Cache Redis Azure. Sélectionnez **Console**.  

2. Utilisez les [commandes Cache Redis Azure](https://redis.io/commands) pour vérifier que vos données se trouvent dans Cache Redis Azure. (La commande prend le format Get {clé}.) Par exemple :

   **Get "12/19/2017 21:32:24 - 123414732"**

   Cette commande doit produire la valeur de la clé spécifiée :

   ![Capture d’écran de la sortie de Cache Redis Azure](./media/stream-analytics-with-azure-functions/image5.png)

## <a name="error-handling-and-retries"></a>Gestion des erreurs et nouvelles tentatives

Si une défaillance se produit lors de l’envoi d’événements à Azure Functions, Stream Analytics retente la plupart des opérations. Toutes les exceptions HTTP sont retentées jusqu’à réussite, à l’exception de l’erreur HTTP 413 (Entité trop volumineuse). L’erreur causée par une entité trop volumineuse est traitée comme une erreur de données soumise à la [stratégie Retenter ou supprimer](stream-analytics-output-error-policy.md).

> [!NOTE]
> Le délai d’expiration des requêtes HTTP adressées à Azure Functions par Stream Analytics est défini sur 100 secondes. Si le traitement d’un lot par votre application Azure Functions prend plus de 100 secondes, Stream Analytics génère une erreur et retente l’opération.

Toute nouvelle tentative après expiration du délai peut entraîner l’écriture d’événements en double dans le récepteur de sortie. Lorsque Stream Analytics retente l’opération pour un lot ayant échoué, la nouvelle tentative concerne tous les événements dans le lot. Prenons l’exemple d’un lot de 20 événements qui sont envoyés à Azure Functions à partir de Stream Analytics. Supposons qu’Azure Functions traite les 10 premiers événements de ce lot en 100 secondes. Après ces 100 secondes, Stream Analytics interrompt la requête, car il n’a pas reçu de réponse positive de la part d’Azure Functions, et une autre requête est envoyée pour le même lot. Les dix premiers événements du lot sont retraités par Azure Functions, ce qui crée un doublon. 

## <a name="known-issues"></a>Problèmes connus

Dans le portail Azure, lorsque vous essayez de rétablir les valeurs maximales de taille et de nombre de lots à des valeurs vides (par défaut), la valeur bascule sur la valeur précédemment saisie au moment de la sauvegarde. Dans ce cas, entrez manuellement les valeurs par défaut pour ces champs.

Actuellement, l’utilisation du [routage HTTP](/sandbox/functions-recipes/routes?tabs=csharp) sur votre instance Azure Functions n’est pas prise en charge par Stream Analytics.

La prise en charge de la connexion aux applications Azure Functions hébergées dans un réseau virtuel n’est pas activée.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources, le travail de streaming et toutes les ressources associées. La suppression du travail évite la facturation des unités de streaming consommées par le travail. Si vous envisagez d’utiliser le travail à l’avenir, vous pouvez l’arrêter et le redémarrer plus tard lorsque vous en avez besoin. Si vous ne pensez pas continuer à utiliser ce travail, supprimez toutes les ressources créées pendant ce guide de démarrage rapide en procédant comme suit :

1. Dans le menu de gauche du portail Azure, cliquez sur **Groupes de ressources**, puis sur le nom de la ressource que vous avez créée.  
2. Sur la page de votre groupe de ressources, cliquez sur **Supprimer**, tapez le nom de la ressource à supprimer dans la zone de texte, puis cliquez sur **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé une tâche Stream Analytics simple qui exécute une fonction Azure. Pour en savoir plus sur les travaux Stream Analytics, passez au didacticiel suivant :

> [!div class="nextstepaction"]
> [Exécuter des fonctions JavaScript définies par l’utilisateur dans des travaux Stream Analytics](stream-analytics-javascript-user-defined-functions.md)

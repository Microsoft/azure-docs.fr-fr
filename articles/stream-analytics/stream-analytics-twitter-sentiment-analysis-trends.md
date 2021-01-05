---
title: Analyse des sentiments Twitter en temps réel avec Azure Stream Analytics
description: Cet article explique comment utiliser Stream Analytics pour l’analyse des sentiments Twitter en temps réel. Aide pas à pas allant de la génération d’événements à la gestion des données sur un tableau de bord en direct.
services: stream-analytics
author: mamccrea
ms.author: mamccrea
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: how-to
ms.date: 02/10/2020
ms.openlocfilehash: 0941e3d5141b5b8841f5d37e3db0d0b1b1474547
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019818"
---
# <a name="real-time-twitter-sentiment-analysis-in-azure-stream-analytics"></a>Analyse de sentiments Twitter en temps réel dans Azure Stream Analytics

Cet article vous explique comment créer une solution d’analyse de sentiments sur les réseaux sociaux en intégrant des événements Twitter en temps réel dans Azure Event Hubs. Vous écrirez une requête Azure Stream Analytics pour analyser les données, puis stocker les résultats pour une utilisation ultérieure ou créer un tableau de bord [Power BI](https://powerbi.com/) pour fournir des informations en temps réel.

Les outils d’analyse des réseaux sociaux aident les organisations à comprendre les tendances. Les sujets tendances sont les sujets et les attitudes apparaissant dans un grand nombre de publications sur les réseaux sociaux. L’analyse de sentiments, aussi appelée *exploration d’opinions*, utilise des outils d’analyse de réseaux sociaux pour déterminer les attitudes envers un produit ou une idée. 

L’analyse de tendances Twitter en temps réel constitue un excellent exemple d’outil d’analyse, car le modèle d’abonnement mot-dièse vous permet de suivre des mots-clés spécifiques (mots-dièse) et de développer l’analyse des sentiments sur le flux.

## <a name="scenario-social-media-sentiment-analysis-in-real-time"></a>Scénario : Analyse de sentiments en temps réel sur les réseaux sociaux

Une entreprise qui dispose d’un site web de médias souhaite obtenir un avantage sur ses concurrents en présentant des contenus immédiatement pertinents pour ses lecteurs. Elle utilise l’analyse des réseaux sociaux sur des sujets pertinents pour ses lecteurs en effectuant une analyse de sentiments en temps réel des données de Twitter.

Pour identifier les tendances en temps réel sur Twitter, l’entreprise doit analyser en temps réel le volume et les sentiments des tweets relatifs aux principaux sujets.

## <a name="prerequisites"></a>Prérequis

Dans ce guide pratique, vous utilisez une application cliente qui se connecte à Twitter, puis recherchez des tweets contenant certains mots-dièse (que vous pouvez définir). Pour exécuter l’application et analyser les tweets à l’aide d’Azure Stream Analytics, vous devez disposer des éléments suivants :

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/).

* Un [compte Twitter](https://twitter.com).

* L’application TwitterClientCore, qui lit le flux Twitter. Pour obtenir à cette application, téléchargez [TwitterClientCore](https://github.com/Azure/azure-stream-analytics/tree/master/DataGenerators/TwitterClientCore).

* Installez le [CLI .NET Core](/dotnet/core/tools/?tabs=netcore2x) version 2.1.0.

## <a name="create-an-event-hub-for-streaming-input"></a>Créer un Event Hub pour l’entrée de diffusion en continu

L’exemple d’application génère des événements et les transmet à un concentrateur Azure Event Hub. Azure Event Hubs est la méthode privilégiée d’ingestion des événements dans Stream Analytics. Pour plus d’informations, voir la [Documentation relative aux concentrateurs Azure Event Hubs](../event-hubs/event-hubs-about.md).

### <a name="create-an-event-hub-namespace-and-event-hub"></a>Créer un espace de noms Event Hub et un concentrateur Event Hub
Dans cette section, vous allez créer un espace de noms Event Hub et y ajouter un Event Hub. Les espaces de noms Event Hub sont utilisés pour regrouper logiquement des instances Event Hub associées. 

1. Connectez-vous au Portail Azure et sélectionnez **Créer une ressource**. Ensuite, recherchez **Event Hubs** et sélectionnez **Créer**.

2. Dans la page **Créer un espace de noms**, entrez un nom d’espace de noms. Vous pouvez utiliser n’importe quel nom pour l’espace de noms, mais il doit être valide pour une URL et unique dans Azure. 
    
3. Sélectionnez un niveau tarifaire et un abonnement, puis créez ou choisissez un groupe de ressources. Ensuite, choisissez un emplacement et sélectionnez **Créer**. 
 
4. Lorsque le déploiement de l’espace de noms est terminé, accédez à votre groupe de ressources et recherchez l’espace de noms Event Hub dans la liste de ressources Azure. 

5. Dans le nouvel espace de noms, sélectionnez **+&nbsp;Event Hub**. 

6. Nommez le nouveau Event Hub *socialtwitter-eh*. Vous pouvez utiliser un autre nom. Le cas échéant, prenez-en note, car vous devrez l’utiliser ultérieurement. Vous n’avez pas besoin de définir d’autres options pour le concentrateur Event Hub.
 
7. Sélectionnez **Create** (Créer).

### <a name="grant-access-to-the-event-hub"></a>Accorder un accès au concentrateur Event Hub

Pour qu’un processus puisse envoyer des données à un Event Hub, ce dernier a besoin d’une stratégie autorisant l’accès. La stratégie d’accès génère une chaîne de connexion qui inclut des informations d’autorisation.

1.  Dans la barre de navigation sur le côté gauche de votre espace de noms Event Hub, sélectionnez **Event Hubs**, qui se trouve dans la section **Entités**. Ensuite, sélectionnez l’Event Hub que vous venez de créer.

2.  Dans la barre de navigation sur le côté gauche, sélectionnez **Stratégies d’accès partagé** situé sous **Paramètres**.

    >[!NOTE]
    >Il existe une option Stratégies d’accès partagé en-dessous pour l’espace de noms Event Hub et pour le Event Hub. Assurez-vous que vous travaillez dans le contexte de votre Event Hub, et non dans l’ensemble de l’espace de noms Event Hub.

3.  Dans la page des stratégies d’accès, sélectionnez **+ Ajouter**. Entrez ensuite *socialtwitter-access* comme **nom de stratégie** et cochez la case **Gérer**.
 
4.  Sélectionnez **Create** (Créer).

5.  Une fois la stratégie déployée, sélectionnez-la dans la liste des stratégies d’accès partagé.

6.  Recherchez la zone intitulée **Chaîne de connexion-Clé primaire**, puis sélectionnez le bouton de copie situé à côté de la chaine de connexion.
 
7.  Collez la chaîne de connexion dans un éditeur de texte. Vous aurez besoin de cette chaîne de connexion pour la section suivante, une fois que vous lui aurez apporté quelques légères modifications.

   La chaîne de connexion ressemble à ce qui suit :
   
   ```
   Endpoint=sb://EVENTHUBS-NAMESPACE.servicebus.windows.net/;SharedAccessKeyName=socialtwitter-access;SharedAccessKey=Gw2NFZw6r...FxKbXaC2op6a0ZsPkI=;EntityPath=socialtwitter-eh
   ```

   Notez que la chaîne de connexion contient plusieurs paires clé-valeur, séparées par des points-virgules : `Endpoint`, `SharedAccessKeyName`, `SharedAccessKey` et `EntityPath`.  

   > [!NOTE]
   > Pour des raisons de sécurité, des parties de la chaîne de connexion indiquée dans l’exemple ont été supprimées.

## <a name="configure-and-start-the-twitter-client-application"></a>Configurer et démarrer l’application client Twitter

L’application cliente obtient les événements de tweet directement à partir de Twitter. Pour ce faire, elle a besoin de l’autorisation d’appeler les API de diffusion Twitter. Pour configurer cette autorisation, créez une application dans Twitter, qui génère des informations d’identification uniques (par exemple, un jeton OAuth). Vous pouvez ensuite configurer l’application cliente pour qu’elle utilise ces informations d’identification lorsqu’elle émet des appels d’API. 

### <a name="create-a-twitter-application"></a>Création d'une application Twitter
Si vous ne possédez pas encore une application Twitter que vous pouvez utiliser pour ce guide pratique, vous pouvez en créer une. Vous devez déjà posséder un compte Twitter.

> [!NOTE]
> Dans Twitter, il est possible que le processus exact pour créer une application et obtenir les clés, secrets ainsi que le jeton soit différent. Si les instructions qui suivent ne correspondent pas ce que vous voyez sur le site Twitter, consultez la documentation pour développeurs Twitter.

1. Dans un navigateur web, accédez à [Twitter For Developers](https://developer.twitter.com/en/apps), créez un compte de développeur, puis sélectionnez **Create an app** (Créer une application). Vous voyez normalement un message indiquant que vous devez demander un compte de développeur Twitter. N’hésitez pas à le faire et, une fois votre application approuvée, vous devriez voir un e-mail de confirmation. L’approbation d’un compte de développeur peut prendre plusieurs jours.

   ![Capture d’écran montrant le bouton Créer une application.](./media/stream-analytics-twitter-sentiment-analysis-trends/provide-twitter-app-details.png "Détails de l’application Twitter")

2. Sur la page **Créer une application**, renseignez les informations de la nouvelle application, puis sélectionnez **Créer votre application Twitter**.

   ![Capture d’écran montrant le volet Détails de l’application dans lequel vous pouvez entrer des valeurs pour votre application.](./media/stream-analytics-twitter-sentiment-analysis-trends/provide-twitter-app-details-create.png "Détails de l’application Twitter")

3. Dans la page de l’application, sélectionnez l’onglet **Keys and Tokens** (Clés et jetons), et copiez les valeurs de **Consumer API Key** (Clé d’API du consommateur) et **Consumer API Secret Key** (Clé secrète d’API du consommateur). Sélectionnez aussi **Create** (Créer) sous **Access Token and Access Token Secret** (Jeton d’accès et secret du jeton d’accès) pour générer les jetons d’accès. Copiez les valeurs de **Jeton d’accès** et **Secret du jeton d’accès**.

   Enregistrez les valeurs que vous avez récupérées pour l’application Twitter. Vous aurez besoin des valeurs ultérieurement.

> [!NOTE]
> Les clés et secrets de l’application Twitter permettent d’accéder à votre compte Twitter. Considérez que ces informations sont sensibles, tout comme votre mot de passe Twitter. Par exemple, ne les intégrez pas à une application que vous proposerez à d’autres utilisateurs. 

### <a name="configure-the-client-application"></a>Configurer l’application cliente

Nous avons créé une application cliente qui se connecte aux données de Twitter par le biais des [API de diffusion Twitter](https://dev.twitter.com/streaming/overview) pour collecter des événements de tweet sur un ensemble de sujets spécifique.

Pour que l’application s’exécute, vous devez indiquer certaines informations telles que les clés Twitter et la chaîne de connexion du concentrateur Event Hub.

1. Veillez à avoir téléchargé l’application [TwitterClientCore](https://github.com/Azure/azure-stream-analytics/tree/master/DataGenerators/TwitterClientCore), comme indiqué dans les conditions préalables.

2. Utilisez un éditeur de texte pour ouvrir le fichier *App.config*. Apportez les modifications suivantes à l’élément `<appSettings>` :

   * Définissez `oauth_consumer_key` sur la clé du client Twitter (clé API). 
   * Définissez `oauth_consumer_secret` sur le secret du client Twitter (clé secrète de l’API).
   * Définissez `oauth_token` sur le jeton d’accès Twitter.
   * Définissez `oauth_token_secret` sur le secret du jeton d’accès Twitter.
   * Définissez `EventHubNameConnectionString` sur la chaîne de connexion.
   * Définissez `EventHubName` sur le nom de l’Event Hub (autrement dit, la valeur du chemin d’accès de l’entité).

3. Ouvrez la ligne de commande et accédez au répertoire où se trouve votre application TwitterClientCore. Utilisez la commande `dotnet build` pour générer le projet. Utilisez ensuite la commande `dotnet run` pour exécuter l’application. L’application envoie des tweets à votre Event Hub.

## <a name="create-a-stream-analytics-job"></a>Création d’un travail Stream Analytics

Maintenant que nous avons un flux d’événements de tweet diffusé en temps réel depuis Twitter, vous pouvez configurer un travail Stream Analytics pour analyser ces événements en temps réel.

1. Dans le Portail Azure, accédez à votre groupe de ressources et sélectionnez **+ Ajouter**. Recherchez ensuite **Tâche Stream Analytics** et sélectionnez **Créer**.

2. Nommez le travail `socialtwitter-sa-job`, puis spécifiez un abonnement, un groupe de ressources et un emplacement.

    Il est judicieux de placer le travail et le concentrateur Event Hub dans la même région afin d’optimiser les performances. Ce faisant, vous ne payez pas pour transférer des données entre les régions.

3. Sélectionnez **Create** (Créer). Ensuite, accédez à votre tâche une fois le déploiement terminé.

## <a name="specify-the-job-input"></a>Spécification de l’entrée de travail

1. Dans votre tâche Stream Analytics, sélectionnez **Entrées** dans le menu de gauche sous **Topologie de la tâche**.

2. Sélectionnez **+&nbsp;Ajouter une entrée de flux** > **Event Hub**. Remplissez le formulaire **Nouvelle entrée** avec les informations suivantes :

   |**Paramètre**  |**Valeur suggérée**  |**Description**  |
   |---------|---------|---------|
   |Alias d’entrée| *TwitterStream* | Spécifiez un alias pour l’entrée. |
   |Abonnement  | \<Your subscription\> |  Sélectionnez l’abonnement Azure que vous souhaitez utiliser. |
   |Espace de noms Event Hub | *asa-twitter-eventhub* |
   |Nom de l’Event Hub | *socialtwitter-eh* | Choisissez *Utiliser l’existant*. Ensuite, sélectionnez l’Event Hub que vous avez créé.|
   |Type de compression d’événement| GZip | Type de compression des données.|

   Conservez les valeurs par défaut restantes et sélectionnez **Enregistrer**.

## <a name="specify-the-job-query"></a>Spécification de la requête de travail

Stream Analytics prend en charge un modèle de requête simple et déclaratif pour la description des transformations. Pour plus d’informations sur ce langage, consultez la page [Références sur le langage des requêtes d’Azure Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference). Ce guide pratique aborde la création et le test de plusieurs requêtes sur des données Twitter.

Pour comparer le nombre de mentions entre les sujets, vous pouvez utiliser une [fenêtre bascule](/stream-analytics-query/tumbling-window-azure-stream-analytics) pour obtenir le nombre de mentions par sujet toutes les cinq secondes.

1. Dans votre tâche **Vue d’ensemble**, sélectionnez **Modifier la requête** dans la partie supérieure droite de la zone de requête. Azure répertorie les entrées et sorties qui sont configurées pour le travail. Vous pouvez également utiliser Azure pour créer une requête visant à transformer le flux d’entrée lorsqu’il est envoyé vers la sortie.

2. Dans l’éditeur de requête, modifiez la requête comme suit :

   ```sql
   SELECT *
   FROM TwitterStream
   ```

3. Les données d’événement issues des messages doivent apparaître dans la fenêtre **Aperçu de l’entrée** sous votre requête. Vérifiez que **Affichage** est défini sur **JSON**. Si vous ne voyez pas de données, assurez-vous que votre générateur de données envoie des événements à votre Event Hub et que vous avez sélectionné **GZip** comme type de compression pour l’entrée.

4. Sélectionnez **Tester la requête** et notez les résultats dans la fenêtre **Résultats du test** sous votre requête.

5. Modifiez la requête dans l’éditeur de code par ce qui suit, puis sélectionnez **Tester la requête** :

   ```sql
   SELECT System.Timestamp as Time, text
   FROM TwitterStream
   WHERE text LIKE '%Azure%'
   ```

6. Cette requête retourne tous les tweets qui incluent le mot clé *Azure*.

## <a name="create-an-output-sink"></a>Créer un récepteur de sortie

Vous avez défini un flux d’événements, une entrée de concentrateur Event Hub pour ingérer des événements, et une requête pour effectuer une transformation sur le flux. La dernière étape consiste à définir un récepteur de sortie pour le travail.  

Dans ce guide pratique, vous écrivez les événements de tweet agrégés de la requête de travail dans un stockage Blob Azure.  Selon les besoins de votre application, vous pouvez également transmettre vos résultats à Azure SQL Database, le stockage Table Azure, Event Hubs ou Power BI.

## <a name="specify-the-job-output"></a>Spécification de la sortie du travail

1. Sous la section **Topologie de la tâche** dans le menu de navigation gauche, sélectionnez **Sorties**. 

2. Dans la page **Sorties**, cliquez sur **+&nbsp;Ajouter** et **Stockage d’objets blob/Data Lake Storage Gen2** :

   * **Alias de sortie** : utilisez le nom `TwitterStream-Output`. 
   * **Options d'importation** : Sélectionnez **Sélectionner un stockage parmi vos abonnements**.
   * **Compte de stockage**. Sélectionnez votre compte de stockage.
   * **Conteneur** : Sélectionnez **Créer**, puis entrez `socialtwitter`.
   
4. Sélectionnez **Enregistrer**.   

## <a name="start-the-job"></a>Démarrage du travail

Une entrée de travail, une requête et une sortie sont spécifiées. Vous êtes prêt à démarrer le travail Stream Analytics.

1. Assurez-vous que l’application TwitterClientCore est en cours d’exécution. 

2. Dans la vue d’ensemble du travail, sélectionnez **Démarrer**.

3. Dans la page **Démarrer le travail**, sélectionnez **Maintenant** pour l’option **Heure de début de la sortie de la tâche**, puis sélectionnez **Démarrer**.

## <a name="get-support"></a>Obtenir de l’aide
Pour obtenir de l’aide supplémentaire, consultez notre [page de questions Microsoft Q&A pour Azure Stream Analytics](/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Étapes suivantes
* [Présentation d’Azure Stream Analytics](stream-analytics-introduction.md)
* [Prise en main d'Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Mise à l’échelle des travaux Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Références sur le langage des requêtes d'Azure Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)
* [Références sur l’API REST de gestion d’Azure Stream Analytics](/rest/api/streamanalytics/)
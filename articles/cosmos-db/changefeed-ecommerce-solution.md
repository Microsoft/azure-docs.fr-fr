---
title: Utiliser le flux de modification Azure Cosmos DB pour visualiser l’Analytique données en temps réel
description: Cet article explique comment une société de distribution peut utiliser le flux de modification pour comprendre les modèles utilisateur, effectuer une analyse des données en temps réel et les visualiser.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: how-to
ms.date: 05/28/2019
ms.author: sngun
ms.custom: devx-track-java
ms.openlocfilehash: e7b75c71d64054e38630677ecd38f8e3e2483c12
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97606332"
---
# <a name="use-azure-cosmos-db-change-feed-to-visualize-real-time-data-analytics"></a>Utiliser le flux de modification Azure Cosmos DB pour visualiser l’Analytique données en temps réel
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Le flux de modification Azure Cosmos DB est un mécanisme permettant d’obtenir un flux d’enregistrements continu et incrémentiel à partir d’un conteneur Azure Cosmos à mesure que ces enregistrements sont créés ou modifiés. La prise en charge du flux de modification consiste à identifier les modifications apportées au conteneur. Il renvoie ensuite la liste chronologique de documents qui ont été modifiés, dans l’ordre dans lequel ils ont été modifiés. Pour plus d’informations sur le flux de modification, consultez l’article [Utilisation du support de flux de modification](change-feed.md). 

Cet article explique comment une société d’e-commerce peut utiliser le flux de modification pour comprendre les modèles utilisateur, pour effectuer l’analyse des données en temps réel et pour visualiser ces données. Vous allez analyser différents événements tels que la consultation d’un article par un utilisateur, l’ajout d’un article à un panier ou l’achat d’un article. Lorsque l’un de ces événements survient, un enregistrement est créé, puis consigné par le flux de modification. Le flux de modification déclenche ensuite une série d’étapes aboutissant à la visualisation de métriques qui analysent les performances et l’activité de la société. Les métriques que vous pouvez visualiser comprennent notamment les recettes, le nombre de visiteurs uniques du site, les articles les plus populaires et le prix moyen des articles qui sont consultés par rapport aux articles ajoutés à un panier et aux articles achetés. Ces exemples de métriques peuvent aider une société d’e-commerce à évaluer la popularité de son site, à développer ses stratégies publicitaires et de tarification et à prendre des décisions concernant le stock dans lequel elle doit investir.

Si vous souhaitez découvrir une vidéo concernant la solution avant de commencer, regardez la vidéo suivante :

> [!VIDEO https://www.youtube.com/embed/AYOiMkvxlzo]
>

## <a name="solution-components"></a>Composants de la solution
Le diagramme ci-après illustre le flux de données et les composants impliqués dans la solution :

:::image type="content" source="./media/changefeed-ecommerce-solution/project-visual.png" alt-text="Éléments visuels du projet" border="false":::
 
1. **Génération de données :** le simulateur de données est utilisé pour générer des données de vente au détail qui représentent des événements, tels que la consultation d’un article par un utilisateur, l’ajout d’un article à un panier et l’achat d’un article. Vous pouvez générer un vaste ensemble d’échantillons de données à l’aide du générateur de données. Les échantillons de données générés contiennent des documents au format suivant :
   
    ```json
    {
      "CartID": 2486,
      "Action": "Viewed",
      "Item": "Women's Denim Jacket",
      "Price": 31.99
    }
    ```

2. **Cosmos DB :** les données générées sont stockées dans un conteneur Azure Cosmos.  

3. **Flux de modification :** le flux de modification identifie les modifications apportées au conteneur Azure Cosmos. Chaque fois qu’un nouveau document est ajouté à la collection (autrement dit, lors de chaque événement tel que la consultation d’un article par un utilisateur, l’ajout d’un article à un panier ou l’achat d’un article), le flux de modification déclenche une [fonction Azure](../azure-functions/functions-overview.md).  

4. **Fonction Azure :** la fonction Azure traite les nouvelles données et les envoie à un [hub d’événements Azure](../event-hubs/event-hubs-about.md).  

5. **Event Hub :** le hub d’événements Azure stocke ces événements et les envoie à [Azure Stream Analytics](../stream-analytics/stream-analytics-introduction.md) pour les soumettre à une analyse approfondie.  

6. **Azure Stream Analytics :** Azure Stream Analytics définit les requêtes permettant de traiter les événements et de procéder à l’analytique de données en temps réel. Ces données sont ensuite envoyées à [Microsoft Power BI](/power-bi/desktop-what-is-desktop).  

7. **Power BI :** Power BI permet de visualiser les données envoyées par Azure Stream Analytics. Vous pouvez générer un tableau de bord afin de visualiser l’évolution des métriques en temps réel.  

## <a name="prerequisites"></a>Prérequis

* Microsoft .NET Framework 4.7.1 ou une version ultérieure

* Microsoft .NET Core 2.1 (ou une version ultérieure)

* Visual Studio avec les charges de travail Développement pour la plateforme Windows universelle, Développement d’applications de bureau .NET, ainsi que Développement web et ASP.NET

* Abonnement Microsoft Azure

* Compte Microsoft Power BI

* [Labo du flux de modification Azure Cosmos DB](https://github.com/Azure-Samples/azure-cosmos-db-change-feed-dotnet-retail-sample) téléchargeable à partir de GitHub 

## <a name="create-azure-resources"></a>Créer des ressources Azure 

Vous devez créer les ressources Azure requises par la solution : Azure Cosmos DB, compte de stockage, Event Hub et Stream Analytics. Vous déploierez ces ressources par le biais d’un modèle Azure Resource Manager. Pour déployer ces ressources, procédez comme suit : 

1. Définissez la stratégie d’exécution Windows PowerShell sur **Sans restriction**. Pour effectuer cette opération, ouvrez **Windows PowerShell en tant qu’administrateur** et exécutez les commandes suivantes :

   ```powershell
   Get-ExecutionPolicy
   Set-ExecutionPolicy Unrestricted 
   ```

2. À partir du référentiel GitHub que vous avez téléchargé à l’étape précédente, accédez au dossier **Azure Resource Manager**, puis ouvrez le fichier appelé **parameters.json**.  

3. Fournissez les valeurs des paramètres cosmosdbaccount_name, eventhubnamespace_name et storageaccount_name comme indiqué dans le fichier **parameters.json**. Utilisez les noms que vous attribuerez à chacune de vos ressources par la suite.  

4. À partir de **Windows PowerShell**, accédez au dossier **Azure Resource Manager**, puis exécutez la commande suivante :

   ```powershell
   .\deploy.ps1
   ```
5. Lorsque vous y êtes invité, entrez votre **ID d’abonnement** Azure, la valeur **changefeedlab** comme nom du groupe de ressources, ainsi que la valeur **run1** pour le nom du déploiement. Le déploiement complet des ressources peut prendre jusqu’à 10 minutes.

## <a name="create-a-database-and-the-collection"></a>Créer une base de données et la collection

Vous allez à présent créer une collection destinée à stocker les événements du site d’e-commerce. Chaque fois qu’un utilisateur consultera un article, ajoutera un article à son panier ou achètera un article, la collection recevra un enregistrement incluant l’action (« Affiché », « Ajouté » ou « Acheté »), le nom et le prix de l’article concerné, ainsi que le numéro d’identification du panier utilisateur impliqué.

1. Accédez au [portail Azure](https://portal.azure.com/) et recherchez le **Compte Azure Cosmos DB** créé par le déploiement du modèle.  

2. Dans le volet **Explorateur de données**, sélectionnez **Nouvelle collection**, puis renseignez le formulaire avec les informations suivantes :  

   * Pour le champ **ID de base de données**, sélectionnez **Créer**, puis entrez **changefeedlabdatabase**. Laissez la case **Provision database throughput** (Provisionner le débit de base de données) décochée.  
   * Pour le champ **ID de la collection**, entrez **changefeedlabcollection**.  
   * Pour le champ **Clé de partition**, entrez **/Item**. Ce champ respecte la casse ; par conséquent, veillez à entrer cette valeur correctement.  
   * Pour le champ **Débit**, entrez la valeur **10000**.  
   * Cliquez sur le bouton **OK**.  

3. Ensuite, créez une autre collection nommée **leases** (baux) pour le traitement du flux de modification. La collection de baux coordonne le traitement du flux de modification entre les différents threads de travail. Une collection distincte est utilisée pour stocker les baux avec un bail par partition.  

4. Réaccédez au volet **Explorateur de données**, sélectionnez **Nouvelle collection**, puis renseignez le formulaire avec les informations suivantes :

   * Pour le champ **ID de base de données**, sélectionnez **Utiliser l’existante**, puis entrez **changefeedlabdatabase**.  
   * Pour le champ **ID de la collection**, entrez **leases**.  
   * Pour le champ **Capacité de stockage**, sélectionnez **Fixe**.  
   * Laissez le champ **Débit** défini sur sa valeur par défaut.  
   * Cliquez sur le bouton **OK**.

## <a name="get-the-connection-string-and-keys"></a>Obtenir la chaîne de connexion et les clés

### <a name="get-the-azure-cosmos-db-connection-string"></a>Obtenir la chaîne de connexion Azure Cosmos DB

1. Accédez au [portail Azure](https://portal.azure.com/) et recherchez le **Compte Azure Cosmos DB** créé par le déploiement du modèle.  

2. Accédez au volet **Clés**, puis copiez la CHAÎNE DE CONNEXION PRINCIPALE dans un Bloc-notes ou dans un autre document auquel vous aurez accès tout au long du labo. Vous devez lui attribuer l’étiquette **Chaîne de connexion Cosmos DB**. Vous devrez copier cette chaîne dans votre code par la suite ; par conséquent, notez-la dans un endroit facile à mémoriser.

### <a name="get-the-storage-account-key-and-connection-string"></a>Obtenir la clé du compte de stockage et la chaîne de connexion

Les comptes Stockage Azure permettent aux utilisateurs de stocker des données. Dans le cadre de ce labo, vous allez utiliser un compte de stockage pour stocker les données utilisées par la fonction Azure. La fonction Azure est déclenchée lorsqu’une modification est apportée à la collection.

1. Réaccédez à votre groupe de ressources et ouvrez le compte de stockage que vous avez créé précédemment.  

2. Dans le menu de gauche, sélectionnez **Clés d’accès**.  

3. Copiez les valeurs figurant sous **clé 1** dans un Bloc-notes ou dans un autre document auquel vous aurez accès tout au long du labo. Attribuez l’étiquette **Clé de stockage** à la **Clé** et l’étiquette **Chaîne de connexion de stockage** à la **Chaîne de connexion**. Vous devrez copier ces chaînes dans votre code par la suite ; par conséquent, notez-les dans un endroit facile à mémoriser.  

### <a name="get-the-event-hub-namespace-connection-string"></a>Obtenir la chaîne de connexion de l’espace de noms Event Hub

Un Event Hub Azure reçoit les données d’événement, puis stocke, traite et transfère les données. Dans le cadre de ce labo, l’Event Hub Azure reçoit un document chaque fois qu’un nouvel événement se produit (autrement dit, lorsqu’un article est consulté par un utilisateur, ajouté au panier d’un utilisateur ou acheté par un utilisateur), puis transfère ce document à Azure Stream Analytics.

1. Réaccédez à votre groupe de ressources et ouvrez **l’Espace de noms Event Hub** que vous avez créé et nommé dans le labo préalable.  

2. Dans le menu de gauche, sélectionnez **Stratégies d’accès partagé**.  

3. Sélectionnez **RootManageSharedAccessKey**. Copiez la **Clé primaire de la chaîne de connexion** dans un Bloc-notes ou dans un autre document auquel vous aurez accès tout au long du labo. Attribuez-lui l’étiquette de chaîne de connexion **Espace de noms Event Hub**. Vous devrez copier cette chaîne dans votre code par la suite ; par conséquent, notez-la dans un endroit facile à mémoriser.

## <a name="set-up-azure-function-to-read-the-change-feed"></a>Configurer une fonction Azure pour la lecture du flux de modification

Quand un document est créé, ou qu’un document existant est modifié dans un conteneur Cosmos, le flux de modification ajoute automatiquement ce document modifié à son historique des modifications de la collection. Vous allez à présent créer et exécuter une fonction Azure qui traite le flux de modification. Lorsqu’un document sera créé ou modifié dans la collection que vous avez créée, la fonction Azure sera déclenchée par le flux de modification. La fonction Azure enverra alors le document modifié à l’Event Hub.

1. Réaccédez au référentiel que vous avez cloné sur votre appareil.  

2. Cliquez avec le bouton droit sur le fichier nommé **ChangeFeedLabSolution.sln**, puis sélectionnez **Ouvrir avec Visual Studio**.  

3. Accédez au fichier **local.settings.json** dans Visual Studio. Ensuite, utilisez les valeurs que vous avez enregistrées précédemment pour renseigner les champs vides.  

4. Accédez au fichier **ChangeFeedProcessor.cs**. Dans les paramètres de la fonction **Run**, procédez comme suit :  

   * Remplacez le texte **YOUR COLLECTION NAME HERE** par le nom de votre collection. Si vous avez suivi les instructions précédentes, le nom de votre collection est changefeedlabcollection.  
   * Remplacez le texte **YOUR LEASES COLLECTION NAME HERE** par le nom de votre collection de baux. Si vous avez suivi les instructions précédentes, le nom de votre collection de baux est **leases**.  
   * Dans la partie supérieure de Visual Studio, assurez-vous que la zone Projet de démarrage figurant à gauche de la flèche verte indique **ChangeFeedFunction**.  
   * Sélectionnez **Démarrer** en haut de la page pour exécuter le programme.  
   * Vous pouvez vous assurer que la fonction s’exécute lorsque l’application de console indique « Job host started » (Hôte de travail démarré).

## <a name="insert-data-into-azure-cosmos-db"></a>Insérer des données dans Azure Cosmos DB 

Pour voir la manière dont le flux de modification traite les nouvelles actions dans un site d’e-commerce, vous devez simuler des données qui représentent la consultation d’articles du catalogue de produits par les utilisateurs, l’ajout de ces articles au panier des utilisateurs et l’achat des articles figurant dans le panier. Ces données sont arbitraires et sont destinées à répliquer l’aspect des données d’un site d’e-commerce.

1. Réaccédez au référentiel dans l’Explorateur de fichiers, puis cliquez avec le bouton droit sur le fichier **ChangeFeedFunction.sln** pour le rouvrir dans une nouvelle fenêtre Visual Studio.  

2. Accédez au fichier **App.config**. À l’intérieur du bloc `<appSettings>`, ajoutez le point de terminaison et la **CLÉ PRIMAIRE** unique de votre compte Azure Cosmos DB que vous avez récupérés précédemment.  

3. Ajoutez les noms de **collection** et de **base de données**. (Ces noms doivent être définis sur **changefeedlabcollection** et sur **changefeedlabdatabase**, sauf si vous avez choisi d’attribuer des noms différents.)

   :::image type="content" source="./media/changefeed-ecommerce-solution/update-connection-string.png" alt-text="Mettre à jour les chaînes de connexion":::
 
4. Enregistrez les modifications dans tous les fichiers modifiés.  

5. Dans la partie supérieure de Visual Studio, assurez-vous que la zone **Projet de démarrage** figurant à gauche de la flèche verte indique **DataGenerator**. Ensuite, sélectionnez **Démarrer** en haut de la page pour exécuter le programme.  
 
6. Attendez que le programme s’exécute. L’apparition d’étoiles signifie que les données affluent. Laissez le programme s’exécuter, car il est important que vous collectiez un maximum de données.  

7. Si vous accédez au [portail Azure](https://portal.azure.com/), puis au compte Cosmos DB dans votre groupe de ressources, et enfin à **Data Explorer**, vous verrez les données aléatoires importées dans votre collection **changefeedlabcollection** .
 
   :::image type="content" source="./media/changefeed-ecommerce-solution/data-generated-in-portal.png" alt-text="Données générées dans le portail":::

## <a name="set-up-a-stream-analytics-job"></a>Configurer un travail Stream Analytics

Azure Stream Analytics est un service cloud entièrement géré pour le traitement en temps réel des données de streaming. Dans le cadre de ce labo, vous allez utiliser Stream Analytics pour traiter les nouveaux événements à partir de l’Event Hub (par exemple, article consulté, ajouté à un panier ou acheté), pour incorporer ces événements dans l’Analytique données en temps réel, puis pour les envoyer dans Power BI à des fins de visualisation.

1. À partir du [portail Azure](https://portal.azure.com/), accédez à votre groupe de ressources, puis à **streamjob1** (travail Stream Analytics que vous avez créé dans le cadre du labo préalable).  

2. Sélectionnez **Entrées** comme illustré ci-dessous.  

   :::image type="content" source="./media/changefeed-ecommerce-solution/create-input.png" alt-text="Création d’une entrée":::

3. Sélectionnez **+ Ajouter une entrée de flux**. Dans le menu déroulant, sélectionnez **Event Hub**.  

4. Renseignez le nouveau formulaire d’entrée avec les informations suivantes :

   * Dans le champ d’alias **Entrée**, tapez **input**.  
   * Sélectionnez l’option **Sélectionner un hub d’événements dans vos abonnements**.  
   * Définissez le champ **Abonnement** sur votre abonnement.  
   * Dans le champ **Espace de noms Event Hub**, entrez le nom de l’espace de noms Event Hub que vous avez créé au cours du labo préalable.  
   * Dans le champ **Nom de l’Event Hub**, sélectionnez l’option **Utiliser l’existant**, puis choisissez **event-hub1** dans le menu déroulant.  
   * Laissez le champ **Event Hub policy** (Stratégie Event Hub) défini sur sa valeur par défaut.  
   * Laissez le champ **Format de sérialisation de l’événement**, défini sur la valeur **JSON**.  
   * Laissez le champ **Codage** défini sur la valeur **UTF-8**.  
   * Laissez le champ **Type de compression d’événement** défini sur **Aucun**.  
   * Sélectionnez le bouton **Enregistrer**.

5. Réaccédez à la page du travail Stream Analytics, puis sélectionnez **Sorties**.  

6. Sélectionnez **Ajouter**. Ensuite, sélectionnez **Power BI** dans le menu déroulant.  

7. Pour créer une sortie Power BI afin de visualiser le prix moyen, procédez comme suit :

   * Dans le champ **Alias de sortie**, entrez **averagePriceOutput**.  
   * Laissez le champ **Espace de travail de groupe** défini sur **Autoriser la connexion pour charger des espaces de travail**.  
   * Dans le champ **Nom du jeu de données**, entrez **averagePrice**.  
   * Dans le champ **Nom de la table**, entrez **averagePrice**.  
   * Cliquez sur le bouton **Autoriser**, puis suivez les instructions pour autoriser la connexion à Power BI.  
   * Sélectionnez le bouton **Enregistrer**.  

8. Revenez à **streamjob1**, puis cliquez sur **Modifier la requête**.

   :::image type="content" source="./media/changefeed-ecommerce-solution/edit-query.png" alt-text="Modifier la requête":::
 
9. Collez la requête ci-après dans la fenêtre de requête. La requête **AVERAGE PRICE** calcule le prix moyen de tous les articles consultés par les utilisateurs, de tous les articles ajoutés au panier des utilisateurs et de tous les articles achetés par les utilisateurs. Ce métrique peut aider les sociétés d’e-commerce à déterminer le prix de vente des articles, ainsi que le stock dans lequel investir. Par exemple, si le prix moyen des articles consultés est nettement plus élevé que celui des articles achetés, une société peut choisir d’ajouter des articles moins chers à son stock.

   ```sql
   /*AVERAGE PRICE*/      
   SELECT System.TimeStamp AS Time, Action, AVG(Price)  
    INTO averagePriceOutput  
    FROM input  
    GROUP BY Action, TumblingWindow(second,5) 
   ```
10. Ensuite, cliquez sur **Enregistrer** dans le coin supérieur gauche.  

11. À présent, revenez à **streamjob1**, puis cliquez sur le bouton **Démarrer** en haut de la page. Le démarrage d’Azure Stream Analytics peut nécessiter quelques minutes, mais le programme finit par passer de l’état « Démarrage en cours » à l’état « Exécution en cours ».

## <a name="connect-to-power-bi"></a>Se connecter à Power BI

Power BI est une suite d’outils d’analyse métier permettant d’analyser des données et de partager des insights. Il constitue un excellent exemple de visualisation stratégique des données analysées.

1. Connectez-vous à Power BI et accédez à **Mon espace de travail** en ouvrant le menu situé à gauche de la page.  

2. Sélectionnez **+ Créer** dans le coin supérieur droit, puis sélectionnez **Tableau de bord** pour créer un tableau de bord.  

3. Sélectionnez **+ Ajouter une vignette** dans le coin supérieur droit.  

4. Sélectionnez **Données de streaming personnalisées**, puis cliquez sur le bouton **Suivant**.  
 
5. Sélectionnez **averagePrice** dans **YOUR DATASETS** (VOS JEUX DE DONNÉES), puis cliquez sur **Suivant**.  

6. Pour le champ **Type de visualisation**, choisissez **Graphique à barres groupées** dans le menu déroulant. Sous **Axe**, ajoutez Action. Ignorez la zone **Légende** sans y ajouter quoi que ce soit. Ensuite, dans la section appelée **Valeur**, ajoutez **avg**. Sélectionnez **Suivant**, intitulez votre graphique, puis sélectionnez **Appliquer**. Un nouveau graphique devrait désormais apparaître sur votre tableau de bord.  

7. À présent, si vous souhaitez visualiser d’autres métriques, vous pouvez réaccéder à **streamjob1**, puis créer trois sorties supplémentaires avec les champs suivants.

   a. **Alias de sortie :** incomingRevenueOutput, Nom du jeu de données : incomingRevenue, Nom de la table : incomingRevenue  
   b. **Alias de sortie :** top5Output, Nom du jeu de données : top5, Nom de la table : top5  
   c. **Alias de sortie :** uniqueVisitorCountOutput, Nom du jeu de données : uniqueVisitorCount, Nom de la table : uniqueVisitorCount

   Ensuite, cliquez sur **Modifier la requête** et collez les requêtes ci-après **au-dessus** de celle que vous avez déjà écrite.

   ```sql
    /*TOP 5*/
    WITH Counter AS
    (
    SELECT Item, Price, Action, COUNT(*) AS countEvents
    FROM input
    WHERE Action = 'Purchased'
    GROUP BY Item, Price, Action, TumblingWindow(second,30)
    ), 
    top5 AS
    (
    SELECT DISTINCT
    CollectTop(5)  OVER (ORDER BY countEvents) AS topEvent
    FROM Counter
    GROUP BY TumblingWindow(second,30)
    ), 
    arrayselect AS 
    (
    SELECT arrayElement.ArrayValue
    FROM top5
    CROSS APPLY GetArrayElements(top5.topevent) AS arrayElement
    ) 
    SELECT arrayvalue.value.item, arrayvalue.value.price,   arrayvalue.value.countEvents
    INTO top5Output
    FROM arrayselect

    /*REVENUE*/
    SELECT System.TimeStamp AS Time, SUM(Price)
    INTO incomingRevenueOutput
    FROM input
    WHERE Action = 'Purchased'
    GROUP BY TumblingWindow(hour, 1)

    /*UNIQUE VISITORS*/
    SELECT System.TimeStamp AS Time, COUNT(DISTINCT CartID) as uniqueVisitors
    INTO uniqueVisitorCountOutput
    FROM input
    GROUP BY TumblingWindow(second, 5)
   ```
   
   La requête TOP 5 calcule les 5 articles les plus fréquemment achetés, classés par nombre de ventes. Ce métrique aide les sociétés d’e-commerce à évaluer les articles les plus populaires et peut influencer les décisions de la société en matière de publicité, de tarification et de stock.

   La requête REVENUE calcule les recettes en additionnant à chaque minute les prix de tous les articles achetés. Ce métrique peut aider les sociétés d’e-commerce à évaluer leurs performances financières et à déterminer les heures de la journée qui contribuent à générer les recettes les plus élevées. Cette information peut avoir une incidence sur la stratégie globale de la société, notamment sur le plan marketing.

   La requête UNIQUE VISITORS calcule toutes les 5 secondes le nombre de visiteurs uniques qui sont présents sur le site en détectant les numéros d’identification de panier uniques. Ce métrique peut aider les sociétés d’e-commerce à évaluer l’activité de leurs site et à élaborer un moyen d’attirer davantage de clients.

8. À présent, vous pouvez également ajouter des vignettes pour ces jeux de données.

   * Pour la requête Top 5, il serait judicieux de créer un histogramme groupé dont l’axe correspond aux articles et dont la valeur correspond au nombre d’articles.  
   * Pour la requête Revenue, il serait logique de créer un graphique en courbes dont l’axe correspond au temps et dont la valeur correspond au cumul des prix. La fenêtre de temps à afficher doit être la plus étendue possible afin de fournir le maximum d’informations.  
   * Pour la requête Unique Visitors, il serait pertinent d’opter pour une visualisation du nombre de visiteurs uniques sous forme de carte.

   Voici un exemple de tableau de bord contenant ces graphiques :

   :::image type="content" source="./media/changefeed-ecommerce-solution/visualizations.png" alt-text="Capture d'écran d'un exemple de tableau de bord contenant des graphiques intitulés Average Price of Items by Action, Unique Visitors, Revenue et Top 5 Items Purchased.":::

## <a name="optional-visualize-with-an-e-commerce-site"></a>Facultatif : Visualiser les données à l’aide d’un site d’e-commerce

Vous allez à présent découvrir comment utiliser votre nouvel outil d’Analytique données pour vous connecter à un véritable site d’e-commerce. Pour générer le site d’e-commerce, utilisez une base de données Azure Cosmos afin de stocker la liste des catégories de produits (Pour femmes, Pour hommes, Unisexe), le catalogue de produits et la liste des articles les plus populaires.

1. Revenez au [portail Azure](https://portal.azure.com/), à votre **Compte Cosmos DB**, puis à **Explorateur de données**.  

   Sous **changefeedlabdatabase**, ajoutez deux collections - **products** et **categories** avec une capacité de stockage Fixe.

   Sous **changefeedlabdatabase**, ajoutez une autre collection nommée **topItems**, et **/Item** comme clé de partition.

2. Cliquez sur la collection **topItems**, puis sous **Mise à l’échelle et paramètres**, définissez le paramètre **Durée de vie** sur **30 secondes** pour que la collection topItems soit mise à jour toutes les 30 secondes.

   :::image type="content" source="./media/changefeed-ecommerce-solution/time-to-live.png" alt-text="Durée de vie":::

3. Afin de remplir la collection **topItems** avec les articles les plus fréquemment achetés, réaccédez à **streamjob1**, puis ajoutez une nouvelle **Sortie**. Sélectionnez **Cosmos DB**.

4. Renseignez les champs obligatoires comme illustré ci-après.

   :::image type="content" source="./media/changefeed-ecommerce-solution/cosmos-output.png" alt-text="Sortie Cosmos":::
 
5. Si vous avez ajouté la requête facultative TOP 5 dans la partie précédente du labo, passez à la partie 5a. Dans le cas contraire, passez à la partie 5b.

   5a. Dans **streamjob1**, sélectionnez **Modifier la requête**, puis collez la requête ci-après dans votre éditeur de requête Azure Stream Analytics sous la requête TOP 5, mais au-dessus du reste des requêtes.

   ```sql
   SELECT arrayvalue.value.item AS Item, arrayvalue.value.price, arrayvalue.value.countEvents
   INTO topItems
   FROM arrayselect
   ```
   5b. Dans **streamjob1**, sélectionnez **Modifier la requête**, puis collez la requête ci-après dans votre éditeur de requête Azure Stream Analytics au-dessus de toutes les autres requêtes.

   ```sql
   /*TOP 5*/
   WITH Counter AS
   (
   SELECT Item, Price, Action, COUNT(*) AS countEvents
   FROM input
   WHERE Action = 'Purchased'
   GROUP BY Item, Price, Action, TumblingWindow(second,30)
   ), 
   top5 AS
   (
   SELECT DISTINCT
   CollectTop(5)  OVER (ORDER BY countEvents) AS topEvent
   FROM Counter
   GROUP BY TumblingWindow(second,30)
   ), 
   arrayselect AS 
   (
   SELECT arrayElement.ArrayValue
   FROM top5
   CROSS APPLY GetArrayElements(top5.topevent) AS arrayElement
   ) 
   SELECT arrayvalue.value.item AS Item, arrayvalue.value.price, arrayvalue.value.countEvents
   INTO topItems
   FROM arrayselect
   ```

6. Ouvrez **EcommerceWebApp.sln** et accédez au fichier **Web.config** dans **l’Explorateur de solutions**.  

7. À l’intérieur du bloc `<appSettings>`, ajoutez **l’URI** et la **CLÉ PRIMAIRE** que vous avez enregistrés précédemment aux emplacements signalés par **your URI here** et **your primary key here**. Ensuite, ajoutez vos **nom de base de données** et **nom de collection** comme indiqué. (Ces noms doivent être définis sur **changefeedlabdatabase** et sur **changefeedlabcollection**, sauf si vous avez choisi d’attribuer des noms différents.)

   Renseignez le **nom de la collection des produits**, le **nom de la collection des catégories** et le **nom de la collection des articles les plus populaires** comme indiqué. (Ces noms doivent être définis sur **products, categories et topItems**, sauf si vous avez choisi d’attribuer des noms différents.)  

8. Dans **EcommerceWebApp.sln**, recherchez et ouvrez le **Dossier d’extraction**. Puis ouvrez le fichier **Web.config** dans ce dossier.  

9. À l’intérieur du bloc `<appSettings>`, ajoutez **l’URI** et la **CLÉ PRIMAIRE** que vous avez enregistrés précédemment aux emplacements signalés. Ensuite, ajoutez vos **nom de base de données** et **nom de collection** comme indiqué. (Ces noms doivent être définis sur **changefeedlabdatabase** et sur **changefeedlabcollection**, sauf si vous avez choisi d’attribuer des noms différents.)  

10. Sélectionnez **Démarrer** en haut de la page pour exécuter le programme.  

11. Vous pouvez à présent expérimenter le site d’e-commerce. Chaque fois que vous consultez un article, ajoutez un article à votre panier, modifiez la quantité d’un article dans votre panier ou achetez un article, ces événements sont transmis par le biais du flux de modification Cosmos DB à l’Event Hub, à Azure Stream Analytics, puis à Power BI. Nous vous recommandons de continuer à exécuter DataGenerator pour générer un volume significatif de données de trafic web et proposer un ensemble réaliste de « Produits recherchés » sur le site d’e-commerce.

## <a name="delete-the-resources"></a>Supprimer les ressources

Pour supprimer les ressources que vous avez créées dans le cadre de ce labo, accédez au groupe de ressources dans le [portail Azure](https://portal.azure.com/), puis sélectionnez **Supprimer le groupe de ressources** dans le menu situé en haut de la page et suivez les instructions fournies.

## <a name="next-steps"></a>Étapes suivantes 
  
* Pour plus d’informations sur le flux de modification, consultez l’article [Utilisation du support de flux de modification dans Azure Cosmos DB](change-feed.md).
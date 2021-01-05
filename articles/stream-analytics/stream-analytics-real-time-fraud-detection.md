---
title: Détection des fraudes en temps réel à l’aide d’Azure Stream Analytics
description: Apprenez à créer une solution de détection des fraudes en temps réel avec Stream Analytics. Utilisez un concentrateur d’événements pour le traitement des événements en temps réel.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 03/24/2020
ms.custom: seodec18
ms.openlocfilehash: ba216e41672e1d19e552b3f82a2ea65da7d3a435
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96007084"
---
# <a name="get-started-using-azure-stream-analytics-real-time-fraud-detection"></a>Bien démarrer avec Azure Stream Analytics : Détection des fraudes en temps réel

Ce didacticiel illustre de bout en bout l’utilisation d’Azure Stream Analytics. Vous allez apprendre à effectuer les actions suivantes : 

* Insérer des événements de flux dans une instance d’Azure Event Hubs. Dans ce tutoriel, vous allez utiliser une application qui simule un flux d’enregistrements de métadonnées de téléphone mobile.

* Écrire des requêtes Stream Analytics de type SQL pour transformer des données, en agrégeant des informations ou en recherchant des modèles. Vous allez apprendre à utiliser une requête pour examiner le flux entrant et rechercher les appels pouvant être frauduleux.

* Envoyer les résultats vers un récepteur de sortie (stockage) que vous pouvez analyser pour obtenir des informations supplémentaires. Dans ce cas, vous enverrez les données des appels suspects au stockage Blob Azure.

Dans ce didacticiel, nous utilisons l’exemple de la détection de fraudes en temps réel sur la base de données d’appels téléphoniques. La technique illustrée convient également à d’autres types de détection de fraude, comme l’usurpation d’identité ou la fraude à la carte de crédit. 

## <a name="scenario-telecommunications-and-sim-fraud-detection-in-real-time"></a>Scénario : Détection des fraudes de télécommunication et SIM en temps réel

Une société de télécommunication dispose d’un volume important de données pour les appels entrants. La société souhaite détecter les appels frauduleux en temps réel afin de pouvoir informer ses clients ou arrêter un service à partir d’un nombre donné. Un type de fraude à la carte SIM implique plusieurs appels simultanés provenant d’une même identité, mais à des emplacements géographiquement distincts. Pour détecter ce type de fraude, la société doit examiner les enregistrements téléphoniques entrants et rechercher des modèles spécifiques : dans ce cas précis, des appels passés en même temps dans différents pays/régions. Tous les enregistrements téléphoniques qui s’inscrivent dans cette catégorie sont écrits dans l’espace de stockage en vue d’une analyse ultérieure.

## <a name="prerequisites"></a>Prérequis

Dans ce didacticiel, vous allez simuler des données d’appels téléphoniques à l’aide d’une application cliente générant un exemple de métadonnées d’appel téléphonique. Certains des enregistrements produits par l’application ressemblent à des appels frauduleux. 

Avant de commencer, veillez à disposer des éléments qui suivent :

* Un compte Azure.
* L’application de génération d’événements d’appel téléphonique, [TelcoGenerator.zip](https://download.microsoft.com/download/8/B/D/8BD50991-8D54-4F59-AB83-3354B69C8A7E/TelcoGenerator.zip), qui peut être téléchargée dans le Centre de téléchargement Microsoft. Décompressez ce package dans un dossier de votre ordinateur. Pour afficher le code source et exécuter l’application dans un débogueur, vous pouvez obtenir le code source de l’application dans [GitHub](https://aka.ms/azure-stream-analytics-telcogenerator). 

    >[!NOTE]
    >Windows peut bloquer le fichier .zip téléchargé. Si vous ne pouvez pas le décompresser, cliquez avec le bouton droit sur le fichier et sélectionnez **Propriétés**. Si le message « Ce fichier provient d’un autre ordinateur et peut éventuellement être bloqué pour protéger cet ordinateur. » est affiché, sélectionnez l’option **Débloquer**, puis cliquez sur **Appliquer**.

Si vous souhaitez examiner les résultats du travail Stream Analytics, vous avez également besoin d’un outil pour afficher le contenu d’un conteneur de stockage Blob Azure. Si vous utilisez Visual Studio, vous pouvez utiliser [Azure Tools pour Visual Studio](/visualstudio/azure/vs-azure-tools-storage-resources-server-explorer-browse-manage) ou [Visual Studio Cloud Explorer](/visualstudio/azure/vs-azure-tools-resources-managing-with-cloud-explorer). Vous pouvez également installer des outils autonomes comme l’[Explorateur Stockage Azure](https://storageexplorer.com/) ou [Cerulean](https://www.cerebrata.com/products/cerulean/features/azure-storage). 

## <a name="create-an-azure-event-hubs-to-ingest-events"></a>Créer un concentrateur Azure Event Hubs pour ingérer les événements

Pour analyser un flux de données, *ingérez-le* dans Azure. Pour ingérer des données, il est courant d’utiliser [Azure Event Hubs](../event-hubs/event-hubs-about.md), qui vous permet d’ingérer des millions d’événements par seconde, puis de traiter et de stocker les informations d’événement. Pour ce tutoriel, vous allez créer un concentrateur Event Hub, puis indiquer à l’application de génération d’événements d’appel d’envoyer des données d’appel à ce concentrateur.

>[!NOTE]
>Pour consulter une version plus détaillée de cette procédure, voir [Créer un espace de noms Event Hubs et un concentrateur d’événements avec le portail Azure](../event-hubs/event-hubs-create.md). 

### <a name="create-a-namespace-and-event-hub"></a>Créer un concentrateur Event Hub et un espace de noms
Dans cette procédure, vous allez commencer par créer un espace de noms Event Hub, puis ajouter un concentrateur Event Hub à cet espace de noms. Les espaces de noms Event Hub sont utilisés pour regrouper logiquement des instances Event Hub associées. 

1. Connectez-vous au Portail Azure, puis cliquez sur **Créer une ressource** en haut à gauche de l’écran.

2. Sélectionnez **Tous les services** dans le menu de gauche, puis l’**étoile (`*`)** à côté d’**Event Hubs** dans la catégorie **Analytique**. Confirmez l’ajout d’**Event Hubs** dans **FAVORIS** dans le menu de navigation de gauche. 

   ![Rechercher Event Hubs](./media/stream-analytics-real-time-fraud-detection/select-event-hubs-menu.png)

3. Sélectionnez **Event Hubs** sous **FAVORIS** dans le menu de navigation de gauche, puis **Ajouter** dans la barre d’outils.

   ![Bouton Ajouter](./media/stream-analytics-real-time-fraud-detection/event-hubs-add-toolbar.png)

4. Dans le volet **Créer un espace de noms**, entrez un nom d’espace de noms, par exemple `<yourname>-eh-ns-demo`. Vous pouvez utiliser n’importe quel nom pour l’espace de noms, mais il doit être valide pour une URL et unique dans Azure. 
    
5. Sélectionnez un abonnement, créez ou choisissez un groupe de ressources, puis cliquez sur **Créer**.

    <br/><img src="./media/stream-analytics-real-time-fraud-detection/stream-analytics-create-eventhub-namespace-new-portal.png" alt="Create event hub namespace in Azure portal" width="300px"/>

6. Lorsque le déploiement de l’espace de noms est terminé, recherchez l’espace de noms Event Hub dans la liste de ressources Azure. 

7. Cliquez sur le nouvel espace de noms, puis, dans le volet d’espace de noms, cliquez sur **Hub d’événements**.

   ![Bouton Ajouter un hub d’événements permettant de créer un concentrateur Event Hub](./media/stream-analytics-real-time-fraud-detection/stream-analytics-create-eventhub-button-new-portal.png)    
 
8. Nommez le nouveau concentrateur Event Hub `asa-eh-frauddetection-demo`. Vous pouvez utiliser un autre nom. Le cas échéant, prenez-en note, car vous devrez l’utiliser ultérieurement. Pour le moment, vous n’avez pas besoin de définir d’autres options pour le concentrateur Event Hub.

    <br/><img src="./media/stream-analytics-real-time-fraud-detection/stream-analytics-create-eventhub-new-portal.png" alt="Name event hub in Azure portal" width="400px"/>
    
9. Cliquez sur **Créer**.

### <a name="grant-access-to-the-event-hub-and-get-a-connection-string"></a>Accorder l’accès au concentrateur Event Hub et obtenir une chaîne de connexion

Pour qu’un processus puisse envoyer des données à un concentrateur Event Hub, ce concentrateur doit disposer d’une stratégie autorisant un accès approprié. La stratégie d’accès génère une chaîne de connexion qui inclut des informations d’autorisation.

1. Dans le volet d’espace de noms, cliquez sur **Hubs d’événements**, puis sur le nom de votre nouvel hub d’événements.

2. Dans le volet de hub d’événements, cliquez sur **Stratégies d’accès partagé**, puis sur **+&nbsp;Ajouter**.

    > [!NOTE]
    > Veillez à utiliser le concentrateur Event Hub et pas l’espace de noms Event Hub.

3. Ajoutez la stratégie nommée `asa-policy-manage-demo` et, pour **Revendication**, sélectionnez **Gérer**.

    <br/><img src="./media/stream-analytics-real-time-fraud-detection/stream-analytics-create-shared-access-policy-manage-new-portal.png" alt="Create shared access policy for Stream Analytics" width="300px"/>
 
4. Cliquez sur **Créer**.

5. Une fois la stratégie déployée, cliquez dessus dans la liste des stratégies d’accès partagé.

6. Recherchez la zone intitulée **CHAÎNE DE CONNEXION-CLÉ PRIMAIRE**, puis cliquez sur le bouton de copie situé à côté de la chaine de connexion. 

    <br/><img src="./media/stream-analytics-real-time-fraud-detection/stream-analytics-shared-access-policy-copy-connection-string-new-portal.png" alt="Stream Analytics shared access policy" width="300px"/>
 
7. Collez la chaîne de connexion dans un éditeur de texte. Vous aurez besoin de cette chaîne de connexion pour la section suivante, une fois que vous lui aurez apporté quelques légères modifications.

    La chaîne de connexion ressemble à ce qui suit :

    `Endpoint=sb://YOURNAME-eh-ns-demo.servicebus.windows.net/;SharedAccessKeyName=asa-policy-manage-demo;SharedAccessKey=Gw2NFZwU1Di+rxA2T+6hJYAtFExKRXaC2oSQa0ZsPkI=;EntityPath=asa-eh-frauddetection-demo`

    Notez que la chaîne de connexion contient plusieurs paires clé-valeur, séparées par des points-virgules : `Endpoint`, `SharedAccessKeyName`, `SharedAccessKey` et `EntityPath`.  


## <a name="configure-and-start-the-event-generator-application"></a>Configuration et démarrage de l’application de génération d’événements

Avant de démarrer l’application TelcoGenerator, vous devez la configurer afin qu’elle envoie des enregistrements d’appels au concentrateur Event Hub que vous venez de créer.

### <a name="configure-the-telcogenerator-app"></a>Configurer l’application TelcoGenerator

1. Dans l’éditeur dans lequel vous avez copié la chaîne de connexion, prenez note de la valeur `EntityPath`, puis supprimez la paire `EntityPath` (n’oubliez pas de supprimer le point-virgule qui la précède). 

2. Dans le dossier où vous avez décompressé le fichier TelcoGenerator.zip, ouvrez le fichier telcodatagen.exe.config dans un éditeur. (Comme il existe plusieurs fichiers .config, veillez à ouvrir celui qui convient.)

3. Dans l’élément `<appSettings>` :

   * Définissez la valeur de la clé `EventHubName` sur le nom du concentrateur Event Hub (autrement dit, la valeur du chemin d’accès de l’entité).
   * Définissez la valeur de la clé `Microsoft.ServiceBus.ConnectionString` sur la chaîne de connexion. 

   La section `<appSettings>` doit ressembler à l’exemple qui suit :

    ```xml
    <appSettings>
     <!-- Service Bus specific app setings for messaging connections -->
     <add key="EventHubName" value="asa-eh-ns-demo"/>
     <add key="Microsoft.ServiceBus.ConnectionString" value="Endpoint=sb://asa-eh-ns-demo.servicebus.windows.net/;SharedAccessKeyName=asa-policy-manage-demo;SharedAccessKey=GEcnTKf2//1MRn6SN1A2u0O76MP9pj3v0Ccyf1su4Zo="/>
   </appSettings>
    ```
 
4. Enregistrez le fichier . 

### <a name="start-the-app"></a>Démarrer l’application

1. Ouvrez une fenêtre Commande et accédez au dossier dans lequel l’application TelcoGenerator est décompressée.

2. Entrez la commande suivante :

    ```console
    telcodatagen.exe 1000 0.2 2
    ```

   Les paramètres sont les suivants : 

   * Nombre d’enregistrements des détails des appels par heure. 
   * Probabilité de fraude de carte SIM : Fréquence, exprimée en pourcentage de l’ensemble des appels, à laquelle l’application doit simuler un appel frauduleux. La valeur 0,2 signifie qu’environ 20 % des enregistrements d’appels semblent frauduleux.
   * Durée en heures. Nombre d’heures pendant lesquelles l’application doit s’exécuter. Vous pouvez également arrêter l’application à tout moment en appuyant sur Ctrl+C au niveau de la ligne de commande.

   Après quelques secondes, l’application commence à afficher des enregistrements des appels téléphoniques à l’écran à mesure qu’elle les envoie au concentrateur Event Hub.

Voici certains champs clés que vous utiliserez dans cette application de détection des fraudes en temps réel :

|**Enregistrement**|**Définition**|
|----------|--------------|
|`CallrecTime`|Horodatage de l’heure de début d’appel. |
|`SwitchNum`|Commutateur téléphonique utilisé pour connecter l’appel. Pour cet exemple, les commutateurs sont des chaînes qui représentent le pays/la région d’origine (États-Unis, Chine, Royaume-Uni, Allemagne ou Australie). |
|`CallingNum`|Numéro de téléphone de l’appelant. |
|`CallingIMSI`|Identité de l’abonné mobile international (IMSI). Il s’agit de l’identificateur unique de l’appelant. |
|`CalledNum`|Numéro de téléphone du destinataire de l’appel. |
|`CalledIMSI`|Identité de l'abonné mobile international (IMSI). Il s’agit de l’identificateur unique du destinataire de l’appel. |


## <a name="create-a-stream-analytics-job-to-manage-streaming-data"></a>Créer un travail Stream Analytics pour gérer les données de flux

Maintenant que vous disposez d’un flux des événements d’appel, vous pouvez configurer un travail Stream Analytics. Le travail lit les données à partir du concentrateur Event Hub que vous avez configuré. 

### <a name="create-the-job"></a>Créer le travail 

1. Dans le portail Azure, cliquez sur **Créer une ressource** > **Internet des objets** > **tâche Stream Analytics**.

2. Nommez le travail `asa_frauddetection_job_demo`, puis spécifiez un abonnement, un groupe de ressources et un emplacement.

    Il est judicieux de placer le travail et le concentrateur Event Hub dans la même région afin d’optimiser les performances. Ce faisant, vous ne payez pas pour transférer des données entre les régions.

    <br/><img src="./media/stream-analytics-real-time-fraud-detection/stream-analytics-create-sa-job-new-portal.png" alt="Create Stream Analytics job in portal" width="300px"/>

3. Cliquez sur **Créer**.

    Le travail est créé, et le portail en affiche les détails. Rien n’est en cours d’exécution cependant. Vous devez configurer le travail pour qu’il puisse être démarré.

### <a name="configure-job-input"></a>Configurer les entrées du travail

1. Dans le tableau de bord ou le volet **Toutes les ressources**, recherchez et sélectionnez le travail Stream Analytics `asa_frauddetection_job_demo`. 
2. Dans la section **Aperçu** du volet du travail Stream Analytics, cliquez sur la zone **Entrée**.

   ![Zone d’entrée sous Topologie dans le volet du travail Stream Analytics](./media/stream-analytics-real-time-fraud-detection/stream-analytics-sa-job-input-box-new-portal.png)
 
3. Cliquez sur **Ajouter une entrée de flux** et sélectionnez **Event Hub**. Remplissez ensuite la page Nouvelle entrée avec les informations suivantes :

   |**Paramètre**  |**Valeur suggérée**  |**Description**  |
   |---------|---------|---------|
   |Alias d’entrée  |  CallStream   |  Saisissez un nom pour identifier l’entrée de la tâche.   |
   |Abonnement   |  \<Your subscription\> |  Sélectionnez l’abonnement Azure dans lequel vous avez créé un Event Hub.   |
   |Espace de noms Event Hub  |  asa-eh-ns-demo |  Entrez le nom de l’espace de noms Event Hub.   |
   |Nom de l’Event Hub  | asa-eh-frauddetection-demo | Sélectionnez le nom de votre Event Hub.   |
   |Nom de la stratégie du hub d’événements  | asa-policy-manage-demo | Sélectionnez la stratégie d’accès que vous avez créée précédemment.   |

    </br>
    <img src="./media/stream-analytics-real-time-fraud-detection/stream-analytics-create-sa-input-new-portal.png" alt="Create Stream Analytics input in portal" width="300px"/>


4. Cliquez sur **Créer**.

## <a name="create-queries-to-transform-real-time-data"></a>Créer des requêtes pour transformer des données en temps réel

À ce stade, vous disposez d’un travail Stream Analytics configuré pour lire un flux de données entrantes. L’étape suivante consiste à créer une requête qui analysera les données en temps réel. Stream Analytics prend en charge un modèle de requête simple et déclaratif pour la description des transformations dans le cadre du traitement en temps réel. Les requêtes utilisent un langage de type SQL dont certaines extensions sont propres à Stream Analytics. 

Une requête simple peut lire simplement toutes les données entrantes. Toutefois, vous créez souvent des requêtes qui recherchent des données spécifiques ou des relations dans les données. Dans cette section du didacticiel, vous allez créer et tester plusieurs requêtes pour découvrir quelques méthodes via lesquelles vous pouvez transformer un flux d’entrée pour l’analyse. 

Les requêtes que vous créez ici affichent uniquement les données transformées à l’écran. Dans une section ultérieure, vous allez configurer un récepteur de sortie et une requête qui écriront les données transformées dans ce récepteur.

Pour plus d’informations sur ce langage, consultez la page [Références sur le langage des requêtes d’Azure Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference).

### <a name="get-sample-data-for-testing-queries"></a>Obtenir des exemples de données pour tester des requêtes

L’application TelcoGenerator envoie des enregistrements d’appels au concentrateur Event Hub, et votre travail Stream Analytics est configuré pour la lecture d’un concentrateur Event Hub. Vous pouvez utiliser une requête pour tester le travail afin de vous assurer qu’il lit correctement. Pour tester une requête dans la console Azure, vous avez besoin d’exemples de données. Pour cette procédure pas à pas, vous allez extraire des exemples de données à partir du flux entrant dans le concentrateur Event Hub.

1. Assurez-vous que l’application TelcoGenerator s’exécute et qu’elle produit des enregistrements d’appels.
2. Dans le portail, retournez dans le volet du travail Stream Analytics. (Si vous avez fermé le volet, recherchez `asa_frauddetection_job_demo` dans le volet **Toutes les ressources**.)
3. Cliquez dans la zone **Requête**. Azure répertorie les entrées et sorties qui sont configurées pour le travail. Vous pouvez également utiliser Azure pour créer une requête qui vous permettra de transformer le flux d’entrée lorsqu’il est envoyé vers la sortie.
4. Dans le volet **Requête**, cliquez sur les points situés à côté de l’entrée `CallStream`, puis sélectionnez **Exemple de données de l’entrée**.

   ![Options du menu permettant d’utiliser des exemples de données pour l’entrée du travail Stream Analytics avec l’option Exemple de données de l’entrée sélectionnée](./media/stream-analytics-real-time-fraud-detection/stream-analytics-create-sample-data-from-input.png)


5. Définissez l’option **Minutes** sur 3, puis cliquez sur **OK**. 
    
   ![Options d’échantillonnage du flux d’entrée, avec 3 minutes sélectionnées](./media/stream-analytics-real-time-fraud-detection/stream-analytics-input-create-sample-data.png)

    À partir du flux d’entrée, Azure échantillonne les données pour une valeur correspondant à une durée de 3 minutes, et vous informe lorsque l’échantillon est prêt. (Cette opération prend quelques instants.) 

L’exemple de données est stocké temporairement et disponible tant que la fenêtre de requête est ouverte. Si vous la fermez, l’exemple de données est abandonné, et vous devez en créer un autre. 

Vous pouvez également obtenir un fichier .json qui contient des exemples de données [de GitHub](https://github.com/Azure/azure-stream-analytics/blob/master/Sample%20Data/telco.json), puis charger ce fichier .json dont vous pouvez utiliser les exemples de données pour l’entrée `CallStream`. 

### <a name="test-using-a-pass-through-query"></a>Procéder à un test à l’aide d’une requête directe

Si vous voulez archiver tous les événements, vous pouvez utiliser une requête directe pour lire tous les champs dans la charge utile de l’événement.

1. Dans la fenêtre de requête, entrez la requête suivante :
        
   ```SQL
   SELECT 
       *
   FROM 
       CallStream
   ```

    >[!NOTE]
    >Comme dans SQL, les mots clés ne respectent pas la casse, et les espaces blancs ne sont pas significatifs.

    Dans cette requête, `CallStream` est l’alias que vous avez spécifié lorsque vous avez créé l’entrée. Si vous en avez utilisé un autre, utilisez plutôt ce nom.

2. Cliquez sur **Test**.

    Le travail Stream Analytics exécute la requête dans l’exemple de données et affiche la sortie au bas de la fenêtre. Les résultats indiquent que le concentrateur Event Hub et le travail Stream Analytics sont correctement configurés. (Comme indiqué, vous allez créer ultérieurement un récepteur de sortie dans lequel la requête pourra écrire des données.)

   ![Sortie du travail Stream Analytics, affichant 73 enregistrements générés](./media/stream-analytics-real-time-fraud-detection/stream-analytics-sa-job-sample-output.png)

    Le nombre exact d’enregistrements que vous pouvez observer dépend du nombre d’enregistrements qui ont été capturés dans votre échantillon de 3 minutes.
 
### <a name="reduce-the-number-of-fields-using-a-column-projection"></a>Réduire le nombre de champs à l’aide d’une projection de colonne

Dans de nombreux cas, il n’est pas nécessaire d’insérer toutes les colonnes du flux d’entrée pour l’analyse. Vous pouvez utiliser une requête pour projeter moins de champs retournés que dans la requête directe.

1. Dans l’éditeur de code, modifiez la requête comme suit :

    ```SQL
    SELECT CallRecTime, SwitchNum, CallingIMSI, CallingNum, CalledNum 
    FROM 
        CallStream
    ```

2. Cliquez de nouveau sur **Test**. 

   ![Sortie du travail Stream Analytics pour la projection avec 25 enregistrements](./media/stream-analytics-real-time-fraud-detection/stream-analytics-sa-job-sample-output-projection.png)
 
### <a name="count-incoming-calls-by-region-tumbling-window-with-aggregation"></a>Nombre d’appels entrants par région : Fenêtre bascule avec agrégation

Supposons que vous souhaitiez compter le nombre d’appels entrants par région. Dans les données de flux, lorsque vous souhaitez effectuer des fonctions d’agrégation telles que le comptage, vous devez segmenter le flux en unités temporelles (puisque le flux de données proprement dit est en réalité un nombre infini). Pour ce faire, utilisez une [fonction de fenêtre](stream-analytics-window-functions.md) Stream Analytics. Vous pouvez alors utiliser les données contenues dans cette fenêtre comme unité.

Pour cette transformation, vous souhaitez une séquence de fenêtres temporelles ne se chevauchant pas ; chaque fenêtre contient un ensemble distinct de données que vous pouvez regrouper et agréger. Ce type de fenêtre est appelé *fenêtre bascule*. Dans la fenêtre bascule, vous pouvez obtenir le nombre des appels entrants, regroupés par `SwitchNum`, qui représente le pays/région d’origine de l’appel. 

1. Dans l’éditeur de code, modifiez la requête comme suit :

    ```SQL
    SELECT 
        System.Timestamp as WindowEnd, SwitchNum, COUNT(*) as CallCount 
    FROM
        CallStream TIMESTAMP BY CallRecTime 
    GROUP BY TUMBLINGWINDOW(s, 5), SwitchNum
    ```

    Cette requête utilise le mot-clé `Timestamp By` dans la clause `FROM` pour spécifier le champ d’horodatage à utiliser dans le flux d’entrée pour définir la fenêtre bascule. Dans ce cas, la fenêtre divise les données en segments en fonction du champ `CallRecTime` dans chaque enregistrement. (Si aucun champ n’est spécifié, l’opération de fenêtrage utilise l’heure d’arrivée de chaque événement dans le concentrateur Event Hub.) Voir « Heure d’arrivée par rapport à l’heure de l’application » dans [Informations de référence sur le langage de requête Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference). 

    La projection inclut `System.Timestamp`, qui retourne un horodatage pour la fin de chaque fenêtre. 

    Pour spécifier que vous voulez utiliser une fenêtre bascule, vous utilisez la fonction [TUMBLINGWINDOW](/stream-analytics-query/tumbling-window-azure-stream-analytics) dans la clause `GROUP BY`. Dans la fonction, indiquez une unité de temps (d’une microseconde à un jour) et une taille de fenêtre (nombre d’unités). Dans cet exemple, comme la fenêtre bascule est constituée d’intervalles de 5 secondes, vous obtenez un nombre d’appels par pays/région pour chaque période de 5 secondes.

2. Cliquez de nouveau sur **Test**. Dans les résultats, notez que les horodatages indiqués sous **WindowEnd** sont exprimés par incréments de 5 secondes.

   ![Sortie du travail Stream Analytics pour l’agrégation, affichant 13 enregistrements](./media/stream-analytics-real-time-fraud-detection/stream-analytics-sa-job-sample-output-aggregation.png)
 
### <a name="detect-sim-fraud-using-a-self-join"></a>Détecter une fraude à la carte SIM à l’aide d’une jointure réflexive

Pour cet exemple, envisagez l’utilisation frauduleuse suivante : appels provenant d’un même utilisateur, mais à différents lieux, espacés de 5 secondes à chaque fois. Par exemple, un même utilisateur ne peut pas légitimement passer simultanément un appel depuis les États-Unis et l’Australie. 

Pour ces cas, vous pouvez utiliser une jointure réflexive des données de flux pour joindre le flux à lui-même en fonction de la valeur `CallRecTime`. Vous pouvez ensuite rechercher les enregistrements d’appels où la valeur de `CallingIMSI` (le numéro d’origine) est identique, mais où la valeur de `SwitchNum` (pays/région d’origine) n’est pas la même.

Si vous utilisez une jointure avec des données de flux, la jointure doit indiquer certaines limites relatives à l’intervalle pouvant séparer des lignes correspondantes dans le temps. (Comme indiqué précédemment, les données de flux sont effectivement sans fin.) Les limites temporelles de la relation sont spécifiées dans la clause `ON` de la jointure, à l’aide de la fonction `DATEDIFF`. Dans ce cas, la jointure est basée sur un intervalle de 5 secondes des données d’appels.

1. Dans l’éditeur de code, modifiez la requête comme suit : 

    ```SQL
    SELECT  System.Timestamp as Time, 
        CS1.CallingIMSI, 
        CS1.CallingNum as CallingNum1, 
        CS2.CallingNum as CallingNum2, 
        CS1.SwitchNum as Switch1, 
        CS2.SwitchNum as Switch2 
    FROM CallStream CS1 TIMESTAMP BY CallRecTime 
        JOIN CallStream CS2 TIMESTAMP BY CallRecTime 
        ON CS1.CallingIMSI = CS2.CallingIMSI 
        AND DATEDIFF(ss, CS1, CS2) BETWEEN 1 AND 5 
    WHERE CS1.SwitchNum != CS2.SwitchNum
    ```

    Cette requête est semblable aux jointures SQL excepté pour la fonction `DATEDIFF` de la jointure. Cette version de `DATEDIFF` est propre à Stream Analytics et doit apparaître dans la clause `ON...BETWEEN`. Les paramètres sont une unité de temps (des secondes dans cet exemple) et les alias des deux sources pour la jointure. Cela diffère de la fonction `DATEDIFF` SQL standard.

    La clause `WHERE` inclut la condition qui marque l’appel frauduleux : les commutateurs d’origine ne sont pas identiques. 

2. Cliquez de nouveau sur **Test**. 

   ![Sortie du travail Stream Analytics pour la jointure réflexive, affichant 6 enregistrements générés](./media/stream-analytics-real-time-fraud-detection/stream-analytics-sa-job-sample-output-self-join.png)

3. Cliquez sur **Enregistrer** pour enregistrer la requête de jointure réflexive dans le cadre du travail Stream Analytics. (Elle n’enregistre pas l’exemple de données.)

    <br/><img src="./media/stream-analytics-real-time-fraud-detection/stream-analytics-query-editor-save-button-new-portal.png" alt="Save Stream Analytics query in portal" width="300px"/>

## <a name="create-an-output-sink-to-store-transformed-data"></a>Créer un récepteur de sortie pour stocker les données transformées

Vous avez défini un flux d’événements, une entrée de concentrateur Event Hub pour ingérer des événements, et une requête pour effectuer une transformation sur le flux. La dernière étape consiste à définir un récepteur de sortie pour le travail, c’est-à-dire un emplacement pour y écrire le flux transformé. 

Vous pouvez utiliser de nombreuses ressources comme récepteurs de sortie : une base de données SQL Server, le stockage Table, le stockage Data Lake, Power BI et même un autre concentrateur Event Hub. Pour ce tutoriel, vous allez écrire le flux dans Stockage Blob Azure, qui est le choix par défaut pour la collecte des informations sur les événements pour une analyse ultérieure, car il prend en charge les données non structurées.

Si vous possédez déjà un compte de stockage d’objets blob, vous pouvez l’utiliser. Pour ce tutoriel, vous allez apprendre à créer un compte de stockage.

### <a name="create-an-azure-blob-storage-account"></a>Créer un compte de stockage Blob Azure

1. Dans le coin supérieur gauche du portail Azure, sélectionnez **Créer une ressource** > **Stockage** > **Compte de stockage**. Dans la page de travail du compte de stockage, définissez le paramètre **Nom** sur « asaehstorage », le paramètre **Emplacement** sur « USA Est », le paramètre **Groupe de ressources** sur « asa-eh-ns-rg » (pour bénéficier de meilleures performances, hébergez le compte de stockage dans le même groupe de ressources que le travail Stream Analytics). Vous pouvez conserver les valeurs par défaut des autres paramètres.  

   ![Créer un compte de stockage dans le portail Azure](./media/stream-analytics-real-time-fraud-detection/stream-analytics-storage-account-create.png)

2. Dans le portail Azure, retournez dans le volet du travail Stream Analytics. (Si vous avez fermé le volet, recherchez `asa_frauddetection_job_demo` dans le volet **Toutes les ressources**.)

3. Dans la section **Topologie de la tâche**, cliquez sur la zone **Sortie**.

4. Dans le volet **Sorties**, cliquez sur **Ajouter** et sélectionnez **stockage Blob**. Remplissez ensuite la page Nouvelle sortie avec les informations suivantes :

   |**Paramètre**  |**Valeur suggérée**  |**Description**  |
   |---------|---------|---------|
   |Alias de sortie  |  CallStream-FraudulentCalls   |  Saisissez un nom pour identifier la sortie de la tâche.   |
   |Abonnement   |  \<Your subscription\> |  Sélectionnez l’abonnement Azure contenant le compte de stockage que vous avez créé. Le compte de stockage peut être dans le même abonnement ou dans un abonnement distinct. Cet exemple suppose que vous avez créé le compte de stockage dans le même abonnement. |
   |Compte de stockage  |  asaehstorage |  Entrez le nom du compte de stockage que vous avez créé. |
   |Conteneur  | asa-fraudulentcalls-demo | Choisissez Créer et entrez un nom de conteneur. |

    <br/>
    <img src="./media/stream-analytics-real-time-fraud-detection/stream-analytics-create-output-blob-storage-new-console.png" alt="Create blob output for Stream Analytics job" width="300px"/>
    
5. Cliquez sur **Enregistrer**. 


## <a name="start-the-streaming-analytics-job"></a>Démarrer le travail Stream Analytics

Le travail est maintenant configuré. Vous avez spécifié une entrée (le concentrateur Event Hub), une transformation (la requête pour rechercher des appels frauduleux) et une sortie (un stockage d’objets blob). À présent, vous pouvez démarrer le travail. 

1. Vérifiez que l’application TelcoGenerator est en cours d’exécution.

2. Dans le volet du travail, cliquez sur **Démarrer**. Dans le volet **Démarrer le travail**, sélectionnez **Maintenant** pour l’option Heure de début de la sortie de la tâche. 

   ![Démarrage de la tâche Stream Analytics](./media/stream-analytics-real-time-fraud-detection/stream-analytics-sa-job-start.png)



## <a name="examine-the-transformed-data"></a>Examiner les données transformées

À présent, vous disposez d’un travail Stream Analytics complet. Le travail examine un flux de métadonnées d’appels téléphoniques, recherche les appels frauduleux en temps réel et écrit des informations liées à ces appels frauduleux dans un espace de stockage. 

Pour terminer ce didacticiel, vous souhaiterez peut-être examiner les données capturées par le travail Stream Analytics. Les données sont écrites dans Stockage Blob Azure dans des segments (fichiers). Vous pouvez utiliser n’importe quel outil pour lire Stockage Blob Azure. Comme indiqué dans la section Prérequis, vous pouvez utiliser des extensions Azure dans Visual Studio ou un outil comme l’[Explorateur Stockage Azure](https://storageexplorer.com/) ou [Cerulean](https://www.cerebrata.com/products/cerulean/features/azure-storage). 

Lorsque vous examinez les contenus d’un fichier dans le stockage d’objets blob, ce que vous voyez ressemble à ce qui suit :

   ![Stockage d’objets blob Azure avec sortie Stream Analytics](./media/stream-analytics-real-time-fraud-detection/stream-analytics-sa-job-blob-storage-view.png)
 

## <a name="clean-up-resources"></a>Nettoyer les ressources

Il existe des articles supplémentaires sur les scénarios de détection des fraudes qui s’appuient sur les ressources que vous avez créées dans ce didacticiel. Si vous souhaitez poursuivre, consultez les suggestions faites sous **Étapes suivantes**.

Toutefois, si vous en avez terminé et n’avez pas besoin des ressources que vous avez créées, vous pouvez les supprimer afin de n’encourir aucuns frais Azure inutiles. Dans ce cas, nous vous suggérons de procéder comme suit :

1. Arrêtez le travail Stream Analytics. Dans le volet **Travaux**, cliquez sur **Arrêter** en haut.
2. Arrêtez l’application TelcoGenerator. Dans la fenêtre de commande où vous avez démarré l’application, appuyez sur Ctrl+C.
3. Si vous avez créé un compte de stockage d’objets blob pour ce didacticiel, supprimez-le. 
4. Supprimez le travail Stream Analytics.
5. Supprimez le concentrateur Event Hub.
6. Supprimez l’espace de noms Event Hub.

## <a name="get-support"></a>Obtenir de l’aide

Pour obtenir de l’aide supplémentaire, consultez notre [page de questions Microsoft Q&A sur Azure Stream Analytics](/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez poursuivre ce didacticiel en lisant l’article suivant :

* [Stream Analytics et Power BI : Tableau de bord d’analytique en temps réel des données de streaming](stream-analytics-power-bi-dashboard.md). Cet article explique comment envoyer la sortie de l’application TelcoStreaming du travail Stream Analytics à Power BI pour l’analyse et la visualisation en temps réel.

Pour plus d’informations sur Stream Analytics en général, consultez les articles suivants :

* [Présentation d’Azure Stream Analytics](stream-analytics-introduction.md)
* [Mise à l’échelle des travaux Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Références sur le langage des requêtes d'Azure Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)
* [Références sur l’API REST de gestion d’Azure Stream Analytics](/rest/api/streamanalytics/)
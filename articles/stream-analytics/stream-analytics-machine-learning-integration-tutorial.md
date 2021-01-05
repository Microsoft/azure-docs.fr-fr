---
title: Intégration d’Azure Stream Analytics avec Azure Machine Learning Studio (classique)
description: Cet article explique comment configurer rapidement un travail Azure Stream Analytics simple avec Azure Machine Learning Studio (classique) à l’aide d’une fonction définie par l’utilisateur.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 08/12/2020
ms.custom: seodec18
ms.openlocfilehash: 9b08b59090d9dd23405f8a0ba86ce608e3a64902
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011385"
---
# <a name="do-sentiment-analysis-with-azure-stream-analytics-and-azure-machine-learning-studio-classic"></a>Effectuer une analyse des sentiments avec Azure Stream Analytics et Azure Machine Learning Studio (classique)

Cet article explique comment configurer un travail Azure Stream Analytics simple qui utilise Azure Machine Learning Studio (classique) pour l’analyse des sentiments. Vous allez utiliser un modèle d’analyse des sentiments Studio (classique) de la galerie Cortana Intelligence pour analyser la diffusion des données de texte et déterminer l’évaluation des sentiments.

> [!TIP]
> Il est vivement recommandé d’utiliser des [UDF Azure Machine Learning](machine-learning-udf.md) au lieu d’une UDF Azure Machine Learning Studio (classique) pour améliorer les performances et la fiabilité.

Vous pourrez appliquer les instructions de cet article dans les scénarios suivants :

* Analyse des sentiments en temps réel sur la diffusion de données Twitter en continu.
* Analyse des enregistrements de conversations de clients avec le personnel de support.
* Évaluation des commentaires sur les forums, blogs et vidéos.
* Nombreux autres scénarios de notation prédictive en temps réel.

La tâche Streaming Analytics créée applique le modèle d’analyse des sentiments en tant que fonction définie par l’utilisateur (UDF) sur l’exemple de données de texte à partir du magasin d’objets blob. La sortie (le résultat de l’analyse des sentiments) est écrite dans le même magasin d’objets blob, mais dans un fichier CSV différent. 

## <a name="prerequisites"></a>Prérequis

Avant de commencer, veillez à disposer des éléments qui suivent :

* Un abonnement Azure actif.

* Un fichier CSV contenant des données Twitter. Vous pouvez télécharger un exemple de fichier à partir de [GitHub](https://github.com/Azure/azure-stream-analytics/blob/master/Sample%20Data/sampleinput.csv), ou créer votre propre fichier. Dans un scénario réel, vous recevriez les données directement à partir d’un flux de données Twitter.

## <a name="create-a-storage-container-and-upload-the-csv-input-file"></a>Créer un conteneur de stockage et charger le fichier d’entrée CSV

Dans cette étape, vous allez charger un fichier CSV dans votre conteneur de stockage.

1. Dans le portail Azure, sélectionnez **Créer une ressource** > **Stockage** > **Compte de stockage**.

2. Remplissez l’onglet *De base* avec les détails suivants, et conservez les valeurs par défaut pour les champs restants :

   |Champ  |Valeur  |
   |---------|---------|
   |Abonnement|Choisissez votre abonnement.|
   |Resource group|Choisir votre groupe de ressources.|
   |Nom du compte de stockage|Entrez un nom pour votre compte de stockage. Le nom doit être unique dans tout Azure.|
   |Emplacement|Choisissez un emplacement. Toutes les ressources doivent utiliser le même emplacement.|
   |Type de compte|BlobStorage|

   ![indiquer les détails du compte de stockage](./media/stream-analytics-machine-learning-integration-tutorial/create-storage-account1.png)

3. Sélectionnez **Vérifier + créer**. Ensuite, sélectionnez **créer** pour déployer votre compte de stockage.

4. Une fois le déploiement terminé, accédez à votre compte de stockage. Sous **Service BLOB**, sélectionnez **Conteneurs**. Sélectionnez **+ Conteneur** pour créer un conteneur.

   ![Créer un conteneur de stockage d’objets blob pour les entrées](./media/stream-analytics-machine-learning-integration-tutorial/create-storage-account2.png)

5. Fournissez un nom pour le conteneur et vérifiez que le **Niveau d’accès public** est défini sur **Privé**. Sélectionnez **Créer** lorsque vous avez terminé.

   ![spécifier les détails du conteneur d’objets blob](./media/stream-analytics-machine-learning-integration-tutorial/create-storage-account3.png)

6. Accédez au conteneur que vous venez de créer, puis sélectionnez **Charger**. Chargez le fichier **sampleinput.csv** que vous avez téléchargé précédemment.

   ![Bouton ’Télécharger’ d’un conteneur](./media/stream-analytics-machine-learning-integration-tutorial/create-sa-upload-button.png)

## <a name="add-the-sentiment-analytics-model-from-the-cortana-intelligence-gallery"></a>Ajout du modèle d’analyse de sentiments de la galerie Cortana Intelligence

Maintenant que les exemples de données sont dans un objet blob, vous pouvez activer le modèle d’analyse des sentiments dans la galerie Cortana Intelligence.

1. Accédez au [modèle d’analyse prédictive des sentiments](https://gallery.cortanaintelligence.com/Experiment/Predictive-Mini-Twitter-sentiment-analysis-Experiment-1) dans la galerie Cortana Intelligence.  

2. Sélectionnez **Ouvrir dans Studio (classique)** .  
   
   ![Stream Analytics Azure Machine Learning Studio (classique), ouvrir Studio (classique)](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-open-ml-studio.png)  

3. Connectez-vous pour accéder à l’espace de travail. Sélectionnez un emplacement.

4. Sélectionnez **Exécuter** en bas de la page. Le processus s’exécute. Cela dure environ une minute.

   ![Exécuter l’expérience dans Studio (classique)](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-run-experiment.png)  

5. Après l’exécution du processus, sélectionnez **Déployer un service web** au bas de la page.

   ![Déployer l’expérience dans Studio (Classic) en tant que service web](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-deploy-web-service.png)  

6. Pour vérifier que le modèle d’analyse des sentiments est prêt à être utilisé, sélectionnez le bouton **Tester**. Entrez du texte, par exemple « I love Microsoft ».

   ![Tester l’expérience dans Studio (classique)](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-test.png)  

   Si le test fonctionne, un résultat similaire au texte suivant s’affiche :

   ![Résultats de test dans Studio (classique)](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-test-results.png)  

7. Dans la colonne **Applications**, sélectionnez le lien **Classeur Excel 2010 ou version antérieure** pour télécharger un classeur Excel. Le classeur contient la clé de l’API et l’URL dont vous aurez besoin pour configurer le travail Stream Analytics.

    ![Stream Analytics Azure Machine Learning Studio (classique), aperçu rapide](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-quick-glance.png)  

## <a name="create-a-stream-analytics-job-that-uses-the-studio-classic-model"></a>Créer un travail Stream Analytics qui utilise le modèle Studio (classique)

Vous pouvez maintenant créer un travail Stream Analytics lisant les exemples de tweets à partir du fichier CSV dans le stockage d’objets blob.

### <a name="create-the-job"></a>Créer le travail

Accédez au [portail Azure](https://portal.azure.com) et créez un travail Stream Analytics. Si vous n’êtes pas familiarisé avec ce processus, consultez [Créer un travail Stream Analytics à l’aide du portail Azure](stream-analytics-quick-create-portal.md).

### <a name="configure-the-job-input"></a>Configurer les entrées du travail

Le travail reçoit les entrées à partir du fichier CSV chargé précédemment dans le stockage d’objets blob.

1. Accédez à votre travail Stream Analytics. Sous **Topologie du travail**, sélectionnez l’option **Entrées**. Sélectionnez **Ajouter une entrée de flux** >**Stockage Blob**.

2. Entrez les détails du **Stockage Blob** avec les valeurs suivantes :

   |Champ  |Valeur  |
   |---------|---------|
   |Alias d’entrée|Donnez un nom à votre entrée. Souvenez-vous de cet alias lorsque vous écrivez votre requête.|
   |Abonnement|Sélectionnez votre abonnement.|
   |Compte de stockage|Sélectionnez le compte de stockage que vous avez créé à l’étape précédente.|
   |Conteneur|Accédez au conteneur que vous avez créé à l’étape précédente.|
   |Format de sérialisation de l’événement|CSV|

3. Sélectionnez **Enregistrer**.

### <a name="configure-the-job-output"></a>Configurer la sortie du travail

Le travail envoie les résultats vers le stockage d’objets blob d’où il a reçu les entrées.

1. Accédez à votre travail Stream Analytics. Sous **Topologie du travail**, sélectionnez l’option **Sorties**. Sélectionnez **Ajouter** > **Stockage Blob**.

2. Remplissez les formulaire **Stockage Blob** avec ces valeurs :

   |Champ  |Valeur  |
   |---------|---------|
   |Alias d’entrée|Donnez un nom à votre entrée. Souvenez-vous de cet alias lorsque vous écrivez votre requête.|
   |Abonnement|Sélectionnez votre abonnement.|
   |Compte de stockage|Sélectionnez le compte de stockage que vous avez créé à l’étape précédente.|
   |Conteneur|Accédez au conteneur que vous avez créé à l’étape précédente.|
   |Format de sérialisation de l’événement|CSV|

3. Sélectionnez **Enregistrer**.

### <a name="add-the-studio-classic-function"></a>Ajouter la fonction Studio (classique)

Précédemment, vous avez publié un modèle Studio (classique) en tant que service web. Dans notre scénario, lorsque le travail Stream Analysis s’exécute, il envoie chaque exemple de tweet d’entrée au service web pour l’analyse des sentiments. Le service web Studio (classique) renvoie un sentiment (`positive`, `neutral` ou `negative`) et associe au tweet une probabilité qu’il soit positif.

Dans cette section du didacticiel, vous allez définir une fonction dans le travail Stream Analysis. Cette fonction peut être appelée pour envoyer un tweet au service web et renvoyer la réponse.

1. Assurez-vous que vous disposez de l’URL du service web et de la clé d’API téléchargée précédemment dans le classeur Excel.

2. Accédez à votre travail Stream Analytics. Sélectionnez ensuite **fonctions** >  **+ Ajouter** > **Azure ML Studio**.

3. Remplissez le formulaire **Azure ML Studio** avec les valeurs suivantes :

   |Champ  |Valeur  |
   |---------|---------|
   | Alias de fonction | Utilisez le nom `sentiment` et sélectionnez **Fournir les paramètres de fonction Azure Machine Learning manuellement**, ce qui vous donne la possibilité d’entrer l’URL et la clé.      |
   | URL| Collez l’URL du service web.|
   |Clé | Collez la clé d’API. |

4. Sélectionnez **Enregistrer**.

### <a name="create-a-query-to-transform-the-data"></a>Créer une requête de transformation des données

Stream Analytics utilise une requête SQL déclarative pour examiner l’entrée et la traiter. Dans cette section, vous allez créer une requête lisant chaque tweet d’entrée, puis appelant la fonction Studio (classique) pour effectuer l’analyse des sentiments. La requête envoie ensuite le résultat vers la sortie définie (stockage d’objets blob).

1. Revenez à la vue d’ensemble du travail de Stream Analytics.

2. Sous **Topologie de la tâche**, sélectionnez **Requête**.

3. Entrez la requête suivante :

    ```SQL
    WITH sentiment AS (  
    SELECT text, sentiment1(text) as result 
    FROM <input>  
    )  

    SELECT text, result.[Score]  
    INTO <output>
    FROM sentiment  
    ```    

   La requête appelle la fonction que vous avez créée précédemment (`sentiment`), pour effectuer l’analyse des sentiments sur chaque tweet dans l’entrée.

4. Sélectionnez **Enregistrer** pour enregistrer la requête.

## <a name="start-the-stream-analytics-job-and-check-the-output"></a>Démarrer le travail Stream Analytics et observer le résultat

Vous pouvez maintenant démarrer le travail Stream Analytics.

### <a name="start-the-job"></a>Démarrage du travail

1. Revenez à la vue d’ensemble du travail de Stream Analytics.

2. En haut de la page, sélectionnez **Démarrer**.

3. Dans le panneau **Démarrer la tâche**, sélectionnez **Personnalisé**, puis un jour antérieur au moment où vous avez chargé le fichier CSV sur le Stockage Blob. Lorsque vous avez terminé, sélectionnez **Démarrer**.  

### <a name="check-the-output"></a>Consulter la sortie

1. Laissez quelques minutes au travail pour s’exécuter avant de constater une activité dans la zone **Surveillance**.

2. Si vous utilisez habituellement un outil spécifique pour consulter le contenu du stockage blob, utilisez-le pour examiner le conteneur. Vous pouvez également effectuer les étapes suivantes dans le portail Azure :

      1. Dans le portail, recherchez votre compte de stockage, puis, dans celui-ci, recherchez le conteneur. Le conteneur présente deux fichiers : le fichier qui contient les exemples de tweets et un fichier CSV généré par le travail Stream Analytics.
      2. Cliquez sur le fichier généré avec le bouton droit de la souris, puis sélectionnez **Télécharger**.

3. Ouvrez le fichier CSV généré. Vous verrez quelque chose de similaire à l’exemple suivant :  

   ![Stream Analytics Azure Machine Learning Studio (classique), affichage CSV](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-csv-view.png)  

### <a name="view-metrics"></a>Afficher les mesures

Vous pouvez également afficher les métriques relatives à la fonction Studio (classique). Les métriques suivantes relatives à la fonction s’affichent dans la zone **Surveillance** de la vue d’ensemble du travail :

* **Requêtes de fonction** indique le nombre de requêtes adressées au service web Studio (classique).  
* **Événements de fonction** indique le nombre d’événements dans la requête. Par défaut, chaque requête envoyée à un service web Studio (classique) contient jusqu’à 1 000 événements.

## <a name="next-steps"></a>Étapes suivantes

* [Présentation d’Azure Stream Analytics](stream-analytics-introduction.md)
* [Références sur le langage des requêtes d'Azure Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)
* [Intégrer l’API REST et Machine Learning Studio (classique)](stream-analytics-how-to-configure-azure-machine-learning-endpoints-in-stream-analytics.md)
* [Références sur l’API REST de gestion d’Azure Stream Analytics](/rest/api/streamanalytics/)
---
title: Intégration de tableau de bord Power BI à Azure Stream Analytics
description: Cet article décrit comment utiliser un tableau de bord Power BI en temps réel pour visualiser les données d’une tâche Azure Stream Analytics.
author: jseb225
ms.author: jeanb
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 11/16/2020
ms.openlocfilehash: 4e3f31442c5fa645e27a640d8facf86aed20aa75
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006693"
---
# <a name="stream-analytics-and-power-bi-a-real-time-analytics-dashboard-for-streaming-data"></a>Stream Analytics et Power BI : Tableau de bord d’analytique en temps réel des données de streaming

Azure Stream Analytics vous permet de tirer parti de [Microsoft Power BI](https://powerbi.com/), l’un des principaux outils d’analyse décisionnelle. Dans cet article, vous allez découvrir comment créer des outils d’analyse décisionnelle en utilisant Power BI comme sortie pour vos travaux Azure Stream Analytics. Vous découvrez aussi comment créer et utiliser un tableau de bord temps réel mis à jour en continu par le travail Stream Analytics.

Cet article est la suite du didacticiel [Détection des fraudes en temps réel](stream-analytics-real-time-fraud-detection.md) de Stream Analytics. Il s’appuie sur le flux de travail créé dans ce didacticiel et ajoute une sortie Power BI afin que vous puissiez visualiser les appels téléphoniques frauduleux qui sont détectés par un travail Stream Analytics. 

Vous pouvez visionner [une vidéo](https://www.youtube.com/watch?v=SGUpT-a99MA) illustrant ce scénario.


## <a name="prerequisites"></a>Prérequis

Avant de commencer, veillez à disposer des éléments qui suivent :

* Un compte Azure.
* Un compte pour Power BI Pro. Vous pouvez utiliser un compte professionnel ou un compte scolaire.
* Une version terminée du didacticiel [Détection des fraudes en temps réel](stream-analytics-real-time-fraud-detection.md). Le didacticiel comprend une application qui génère des métadonnées d’appels téléphoniques fictifs. Dans le didacticiel, vous créez un concentrateur Event Hub et lui envoyez des données d’appels téléphoniques de diffusion en continu. Vous écrivez une requête qui détecte les appels frauduleux (appels simultanés provenant d’un même numéro à différents emplacements). 


## <a name="add-power-bi-output"></a>Ajouter une sortie Power BI
Dans le didacticiel de détection des fraudes en temps réel, la sortie est envoyée vers le stockage Blob Azure. Dans cette section, vous ajoutez une sortie qui envoie des informations à Power BI.

1. Dans le portail Azure, ouvrez le travail Stream Analytics que vous avez créé précédemment. Si vous avez utilisé le nom suggéré, le travail est nommé `sa_frauddetection_job_demo`.

2. Dans le menu de gauche, sélectionnez **Sorties** sous **Topologie de la tâche**. Ensuite, sélectionnez **+ Ajouter** et choisissez **Power BI** dans le menu déroulant.

3. Sélectionnez **+ Ajouter** > **Power BI**. Remplissez ensuite le formulaire avec les informations suivantes et sélectionnez **Autoriser** pour utiliser votre propre identité d’utilisateur pour vous connecter à Power BI (le jeton est valide pendant 90 jours). 

>[!NOTE]
>En ce qui concerne les travaux de production, nous vous recommandons de vous connecter pour [utiliser l’identité managée pour authentifier votre travail Azure Stream Analytics vers Power BI](./powerbi-output-managed-identity.md).

   |**Paramètre**  |**Valeur suggérée**  |
   |---------|---------|
   |Alias de sortie  |  CallStream-PowerBI  |
   |Nom du jeu de données  |   sa-dataset  |
   |Nom de la table |  fraudulent-calls  |

   ![Configurer une sortie Stream Analytics](media/stream-analytics-power-bi-dashboard/configure-stream-analytics-output.png)

   > [!WARNING]
   > Si Power BI dispose d’un jeu de données et d’une table portant les mêmes noms que ceux que vous spécifiez dans le travail Stream Analytics, les données existantes sont écrasées.
   > Nous vous recommandons de ne pas créer explicitement ce jeu de données et cette table dans votre compte Power BI. Ceux-ci sont automatiquement créés au démarrage du travail Stream Analytics et lorsque celui-ci se met à injecter des sorties dans Power BI. Si la requête de travail ne retourne aucun résultat, le jeu de données et la table ne sont pas créés.
   >

4. Lorsque vous sélectionnez **Autoriser**, une fenêtre contextuelle s’ouvre et vous invite à fournir des informations d’identification pour vous authentifier auprès de votre compte Power BI. Une fois que l’autorisation a réussi, **enregistrez** les paramètres.

8. Cliquez sur **Créer**.

Le jeu de données est créé avec les paramètres suivants :

* **defaultRetentionPolicy : BasicFIFO** : Les données sont de type « premier entré, premier sorti » (FIFO), avec un maximum de 200 000 lignes.
* **defaultMode : hybrid** : le jeu de données prend en charge les vignettes de streaming, également appelées « push », et les visuels basés sur des rapports traditionnels. Pour le contenu de type push, les données sont mises à jour en continu, dans le cas présent à partir du travail Stream Analytics, sans qu’il soit nécessaire de planifier l’actualisation du côté Power BI.

Pour le moment, vous ne pouvez pas créer de jeux de données avec d’autres indicateurs.

Pour plus d’informations sur les jeux de données Power BI, consultez la référence [API REST Power BI](/rest/api/power-bi/).


## <a name="write-the-query"></a>Écrire la requête

1. Fermez le panneau **Sorties** et retournez dans le panneau du travail.

2. Cliquez dans la zone **Requête**. 

3. Entrez la requête suivante. Cette requête est similaire à la requête de jointure réflexive que vous avez créée dans le didacticiel de détection des fraudes, à la différence près que cette requête envoie les résultats vers la nouvelle sortie que vous avez créée (`CallStream-PowerBI`). 

    >[!NOTE]
    >Si vous n’avez pas nommé l’entrée `CallStream` dans le didacticiel de détection des fraudes, remplacez votre nom par `CallStream` dans les clauses **FROM** et **JOIN** de la requête.

   ```SQL
   /* Our criteria for fraud:
   Calls made from the same caller to two phone switches in different locations (for example, Australia and Europe) within five seconds */

   SELECT System.Timestamp AS WindowEnd, COUNT(*) AS FraudulentCalls
   INTO "CallStream-PowerBI"
   FROM "CallStream" CS1 TIMESTAMP BY CallRecTime
   JOIN "CallStream" CS2 TIMESTAMP BY CallRecTime

   /* Where the caller is the same, as indicated by IMSI (International Mobile Subscriber Identity) */
   ON CS1.CallingIMSI = CS2.CallingIMSI

   /* ...and date between CS1 and CS2 is between one and five seconds */
   AND DATEDIFF(ss, CS1, CS2) BETWEEN 1 AND 5

   /* Where the switch location is different */
   WHERE CS1.SwitchNum != CS2.SwitchNum
   GROUP BY TumblingWindow(Duration(second, 1))
   ```

4. Cliquez sur **Enregistrer**.


## <a name="test-the-query"></a>Tester la requête

Cette étape est facultative, mais recommandée. 

1. Si l’application TelcoStreaming n’est pas en cours d’exécution, démarrez-la en procédant comme suit :

    * Ouvrez l’invite de commandes.
    * Accédez au dossier dans lequel se trouvent les fichiers telcogenerator.exe et telcodatagen.exe.config modifiés.
    * Exécutez la commande suivante :

       `telcodatagen.exe 1000 .2 2`

2. Dans la page **Requête** pour votre tâche Stream Analytics, cliquez sur les points situés à côté de l’entrée `CallStream`, puis sélectionnez **Exemple de données de l’entrée**.

3. Spécifiez que vous souhaitez une valeur de trois minutes de données, puis cliquez sur **OK**. Patientez jusqu’à ce que vous soyez informé de l’échantillonnage des données.

4. Cliquez sur **Test** et passez en revue les résultats.

## <a name="run-the-job"></a>Exécuter le travail

1. Vérifiez que l’application TelcoStreaming est en cours d’exécution.

2. Accédez à la page **Vue d’ensemble** pour votre tâche Stream Analytics et sélectionnez **Démarrer**.

    ![Démarrage de la tâche Stream Analytics](./media/stream-analytics-power-bi-dashboard/stream-analytics-sa-job-start-output.png)

Le travail Stream Analytics commence par rechercher les appels frauduleux dans le flux entrant. Le travail crée également le jeu de données et la table dans Power BI et commence à leur envoyer des données sur les appels frauduleux.


## <a name="create-the-dashboard-in-power-bi"></a>Créer le tableau de bord dans Power BI

1. Accédez à [Powerbi.com](https://powerbi.com), puis connectez-vous à l’aide de votre compte professionnel ou scolaire. Si la requête du travail Stream Analytics génère des résultats, vous voyez que votre jeu de données est déjà créé :

    ![Emplacement du jeu de données de streaming dans Power BI](./media/stream-analytics-power-bi-dashboard/stream-analytics-streaming-dataset.png)

2. Dans votre espace de travail, cliquez sur **+&nbsp;Créer**.

    ![Bouton Créer dans l’espace de travail Power BI](./media/stream-analytics-power-bi-dashboard/pbi-create-dashboard.png)

3. Créez un tableau de bord et nommez-le `Fraudulent Calls`.

    ![Créer un tableau de bord et lui donner un nom dans l’espace de travail Power BI](./media/stream-analytics-power-bi-dashboard/pbi-create-dashboard-name.png)

4. En haut de la fenêtre, cliquez sur **Ajouter une vignette**, sélectionnez **DONNÉES DE STREAMING PERSONNALISÉES**, puis cliquez sur **Suivant**.

    ![Vignette de jeu de données de streaming personnalisées dans Power BI](./media/stream-analytics-power-bi-dashboard/custom-streaming-data.png)

5. Sous **VOS JEUX DE DONNÉES**, sélectionnez votre jeu de données, puis cliquez sur **Suivant**.

    ![Votre jeu de données de streaming dans Power BI](./media/stream-analytics-power-bi-dashboard/your-streaming-dataset.png)

6. Sous **Type de visualisation**, sélectionnez **Carte**, puis, dans la liste **Champs**, sélectionnez **fraudulentcalls**.

    ![Détails de la visualisation de la nouvelle vignette](./media/stream-analytics-power-bi-dashboard/add-fraudulent-calls-tile.png)

7. Cliquez sur **Suivant**.

8. Renseignez les détails de la vignette, par exemple un titre et un sous-titre.

    ![Titre et sous-titre de la nouvelle vignette](./media/stream-analytics-power-bi-dashboard/pbi-new-tile-details.png)

9. Cliquez sur **Appliquer**.

    Nous avons à présent un compteur de fraudes.

    ![Compteur de fraudes dans le tableau de bord Power BI](./media/stream-analytics-power-bi-dashboard/power-bi-fraud-counter-tile.png)

8. Suivez à nouveau les étapes pour ajouter une vignette (à partir de l’étape 4). Cette fois, procédez comme suit :

    * Lorsque vous accédez à **Type de visualisation**, sélectionnez **Graphique en courbes**. 
    * Ajoutez un axe, puis sélectionnez **windowend**. 
    * Ajoutez une valeur, puis sélectionnez **fraudulentcalls**.
    * Pour **Fenêtre de temps à afficher**, sélectionnez les 10 dernières minutes.

      ![Créer une vignette pour le graphique en courbes dans Power BI](./media/stream-analytics-power-bi-dashboard/pbi-create-tile-line-chart.png)

9. Cliquez sur **Suivant**, ajoutez un titre et un sous-titre, puis cliquez sur **Appliquer**.

     Le tableau de bord Power BI présente désormais deux vues de données sur les appels frauduleux détectés dans les données de diffusion en continu.

     ![Tableau de bord Power BI complété affichant deux vignettes pour les appels frauduleux](./media/stream-analytics-power-bi-dashboard/pbi-dashboard-fraudulent-calls-finished.png)

## <a name="learn-about-limitations-and-best-practices"></a>En savoir plus sur les limites et les meilleures pratiques
Actuellement, Power BI peut être appelé environ une fois par seconde. Les éléments visuels de streaming prennent en charge des paquets de 15 Ko. Au-delà de cette taille, ils échouent (mais les transmissions de type push continuent de fonctionner). En raison de cette limite, Power BI s’applique naturellement dans les cas où Azure Stream Analytics débouche sur une réduction significative de la charge des données. Nous vous recommandons d’utiliser une fenêtre bascule ou une fenêtre récurrente pour vous assurer que la transmission de type push des données est effectuée au plus une fois par seconde et que votre requête respecte les exigences de débit.

Vous pouvez utiliser l’équation suivante pour calculer la valeur à donner à votre fenêtre en secondes :

![Équation pour calculer la valeur à donner à la fenêtre en secondes](./media/stream-analytics-power-bi-dashboard/compute-window-seconds-equation.png)  

Par exemple :

* Vous disposez de 1 000 appareils qui envoient des données à des intervalles d’une seconde.
* Vous utilisez la référence Power BI Pro qui prend en charge 1 000 000 lignes par heure.
* Vous souhaitez publier la quantité moyenne de données par appareil sur Power BI.

L’équation est donc la suivante :

![Équation en fonction d’exemples de critère](./media/stream-analytics-power-bi-dashboard/power-bi-example-equation.png)  

Étant donné cette configuration, vous pouvez modifier la requête d’origine et la remplacer par ce qui suit :

```SQL
    SELECT
        MAX(hmdt) AS hmdt,
        MAX(temp) AS temp,
        System.TimeStamp AS time,
        dspl
    INTO "CallStream-PowerBI"
    FROM
        Input TIMESTAMP BY time
    GROUP BY
        TUMBLINGWINDOW(ss,4),
        dspl
```

### <a name="renew-authorization"></a>Renouveler une autorisation
Si le mot de passe a été modifié depuis la création ou la dernière authentification de votre travail, vous devez réauthentifier votre compte Power BI. Si Azure AD Multi-Factor Authentication est configuré sur votre locataire Azure Active Directory (Azure AD), vous devez également renouveler l’autorisation Power BI toutes les deux semaines. Si vous ne la renouvelez pas, vous pouvez observer des symptômes comme un manque de sortie du travail ou une `Authenticate user error` dans les journaux d’activité des opérations.

De même, si un travail démarre après l’expiration du jeton, une erreur se produit et le travail échoue. Pour résoudre ce problème, arrêtez le travail en cours d’exécution et accédez à votre sortie Power BI. Pour éviter toute perte de données, sélectionnez le lien **Renouveler l’autorisation**, puis redémarrez votre travail depuis l’**heure du dernier arrêt**.

Une fois que l’autorisation a été actualisée avec Power BI, une alerte verte apparaît dans la zone d’autorisation pour indiquer que le problème a été résolu.

## <a name="next-steps"></a>Étapes suivantes
* [Présentation d’Azure Stream Analytics](stream-analytics-introduction.md)
* [Prise en main d'Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Sorties Stream Analytics](stream-analytics-define-outputs.md)
* [Références sur le langage des requêtes Azure Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)
* [Références sur l’API REST de gestion d’Azure Stream Analytics](/rest/api/streamanalytics/)
* [Utiliser des identités managées pour authentifier votre travail Azure Stream Analytics dans Power BI](./powerbi-output-managed-identity.md)

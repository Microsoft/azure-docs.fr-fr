---
title: Activités de création de branches et de chaînage dans un pipeline à l’aide du portail Azure
description: Découvrez comment contrôler le flux de contrôle des données dans le pipeline Azure Data Factory à l’aide du Portail Azure.
services: data-factory
author: dcstwh
ms.author: weetok
manager: anandsub
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.custom: seo-lt-2019; seo-dt-2019
ms.date: 01/11/2018
ms.openlocfilehash: dc82cfdc4e5a063e7c5cb833b617da58023d1ba6
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96497433"
---
# <a name="branching-and-chaining-activities-in-an-azure-data-factory-pipeline-using-the-azure-portal"></a>Création de branches et de chaînage pour les activités dans un pipeline Azure Data Factory à l’aide du portail Azure

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Dans ce didacticiel, vous créez un pipeline Data Factory qui présente certaines des fonctionnalités de flux de contrôle. Ce pipeline est une simple copie depuis un conteneur Stockage Blob Azure vers un autre conteneur dans le même compte de stockage. Si l’activité de copie réussit, le pipeline envoie les détails de l’opération de copie réussie (par exemple, la quantité de données écrites) dans un e-mail d’avis de réussite. Si l’activité de copie échoue, le pipeline envoie les détails de l’échec de la copie (par exemple, le message d’erreur) dans un e-mail d’avis d’échec. Tout au long de ce didacticiel, vous allez apprendre à passer des paramètres.

Vue d’ensemble du scénario : ![Le diagramme montre le Stockage Blob Azure qui est la cible d’une copie, laquelle, en cas de réussite, envoie un e-mail avec des détails ou, en cas d’échec, envoie un e-mail avec les détails de l’erreur.](media/tutorial-control-flow-portal/overview.png)

Dans ce tutoriel, vous allez effectuer les étapes suivantes :

> [!div class="checklist"]
> * Créer une fabrique de données.
> * Créer un service lié Stockage Azure
> * Créer un jeu de données d’objet blob Azure
> * Créer un pipeline contenant une activité de copie et une activité web
> * Envoyer les sorties des activités aux activités ultérieures
> * Utiliser des variables système et de passage de paramètres
> * Démarrer une exécution de pipeline
> * Surveiller les exécutions de pipeline et d’activité

Ce tutoriel utilise le portail Azure. Vous pouvez utiliser d’autres mécanismes pour interagir avec Azure Data Factory. Consultez « Guides de démarrage rapide » dans la table des matières.

## <a name="prerequisites"></a>Prérequis

* **Abonnement Azure**. Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.
* **Compte Stockage Azure**. Vous utilisez le stockage blob comme magasins de données **source**. Si vous n’avez pas de compte de stockage Azure, consultez l’article [Créer un compte de stockage](../storage/common/storage-account-create.md) pour découvrir comment en créer un.
* **Azure SQL Database**. Vous utilisez la base de données en tant que magasin de données **récepteur**. Si vous n’avez pas de base de données dans Azure SQL Database, consultez l’article [Créer une base de données dans Azure SQL Database](../azure-sql/database/single-database-create-quickstart.md) pour savoir comme en créer une.

### <a name="create-blob-table"></a>Créer la table d’objets blob

1. Lancez le Bloc-notes. Copiez le texte suivant et enregistrez-le comme fichier **input.txt** sur votre disque.

    ```
    John,Doe
    Jane,Doe
    ```
2. Utilisez des outils tels que l’ [Explorateur de stockage Azure](https://storageexplorer.com/) pour effectuer les étapes suivantes :
    1. Créer le conteneur **adfv2branch**.
    2. Créer le dossier **entrée** dans le conteneur **adfv2branch**.
    3. Télécharger le fichier **input.txt** au conteneur.

## <a name="create-email-workflow-endpoints"></a>Créer des points de terminaison de flux de travail d’e-mail
Pour déclencher l’envoi d’un e-mail depuis le pipeline, vous utilisez [Logic Apps](../logic-apps/logic-apps-overview.md) pour définir le flux de travail. Pour plus d’informations sur la création d’un flux de travail Logic Apps, consultez [Guide pratique pour créer une application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md).

### <a name="success-email-workflow"></a>Flux de travail d’un e-mail d’avis de réussite
Créez un flux de travail Logic Apps nommé `CopySuccessEmail`. Définissez le déclencheur du flux de travail comme `When an HTTP request is received` et ajoutez une action `Office 365 Outlook – Send an email`.

![Flux de travail d’un e-mail d’avis de réussite](media/tutorial-control-flow-portal/success-email-workflow.png)

Pour le déclencheur de votre requête, renseignez `Request Body JSON Schema` avec le texte JSON suivant :

```json
{
    "properties": {
        "dataFactoryName": {
            "type": "string"
        },
        "message": {
            "type": "string"
        },
        "pipelineName": {
            "type": "string"
        },
        "receiver": {
            "type": "string"
        }
    },
    "type": "object"
}
```

La requête dans le concepteur d’application logique doit ressembler à l’image suivante :

![Concepteur d’application logique - requête](media/tutorial-control-flow-portal/logic-app-designer-request.png)

Pour l’action **Envoyer un e-mail**, personnalisez le mode de mise en forme de l’e-mail, en utilisant les propriétés passées dans le schéma JSON du corps de la requête. Voici un exemple :

![Concepteur d’application logique - action Envoyer un e-mail](media/tutorial-control-flow-portal/send-email-action-2.png)

Enregistrez le workflow. Prenez note de l’URL de votre requête HTTP Post pour votre flux de travail d’e-mail d’avis de réussite :

```
//Success Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000
```

### <a name="fail-email-workflow"></a>Flux de travail d’un e-mail d’avis d’échec
Suivez les mêmes étapes pour créer un autre flux de travail Logic Apps **CopyFailEmail**. Dans le déclencheur de la requête, `Request Body JSON schema` est identique. Modifiez la mise en forme de votre e-mail, notamment `Subject`, pour l’adapter à un avis d’échec. Voici un exemple :

![Concepteur d’application logique - flux de travail d’un e-mail d’avis d’échec](media/tutorial-control-flow-portal/fail-email-workflow-2.png)

Enregistrez le workflow. Prenez note de l’URL de votre requête HTTP Post pour votre flux de travail d’e-mail d’avis d’échec :

```
//Fail Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000
```

Vous devez maintenant avoir deux URL de flux de travail :

```
//Success Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000

//Fail Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000
```

## <a name="create-a-data-factory"></a>Créer une fabrique de données

1. Lancez le navigateur web **Microsoft Edge** ou **Google Chrome**. L’interface utilisateur de Data Factory n’est actuellement prise en charge que par les navigateurs web Microsoft Edge et Google Chrome.
1. Dans le menu de gauche, sélectionnez **Créer une ressource** > **Données + Analytique** > **Data Factory** :

   ![Sélection Data Factory dans le volet « Nouveau »](./media/quickstart-create-data-factory-portal/new-azure-data-factory-menu.png)

2. Dans la page **Nouvelle fabrique de données**, entrez **ADFTutorialDataFactory** comme **nom**.

     ![Page Nouvelle fabrique de données](./media/tutorial-control-flow-portal/new-azure-data-factory.png)

   Le nom de la fabrique de données Azure doit être un nom **global unique**. Si l’erreur suivante s’affiche, changez le nom de la fabrique de données (par exemple, votrenomADFTutorialDataFactory), puis tentez de la recréer. Consultez l’article [Data Factory - Règles d’affectation des noms](naming-rules.md) pour savoir comment nommer les artefacts Data Factory.

   *Le nom de fabrique de données « ADFTutorialDataFactory » n’est pas disponible*.

3. Sélectionnez l’**abonnement** Azure dans lequel vous voulez créer la fabrique de données.
4. Pour le **groupe de ressources**, effectuez l’une des opérations suivantes :

      - Sélectionnez **Utiliser l’existant**, puis sélectionnez un groupe de ressources existant dans la liste déroulante.
      - Sélectionnez **Créer**, puis entrez le nom d’un groupe de ressources.   
         
        Pour plus d’informations sur les groupes de ressources, consultez [Utilisation des groupes de ressources pour gérer vos ressources Azure](../azure-resource-manager/management/overview.md).  
4. Sélectionnez **V2** pour la **version**.
5. Sélectionnez **l’emplacement** de la fabrique de données. Seuls les emplacements pris en charge sont affichés dans la liste déroulante. Les magasins de données (Stockage Azure, Azure SQL Database, etc.) et les services de calcul (HDInsight, etc.) utilisés par la fabrique de données peuvent être proposés dans d’autres régions.
6. Sélectionnez **Épingler au tableau de bord**.     
7. Cliquez sur **Créer**.      
8. Sur le tableau de bord, vous voyez la vignette suivante avec l’état : **Déploiement de Data Factory**.

    ![mosaïque déploiement de fabrique de données](media/tutorial-control-flow-portal/deploying-data-factory.png)
9. Une fois la création terminée, la page **Data Factory** s’affiche comme sur l’image.

   ![Page d’accueil Data Factory](./media/tutorial-control-flow-portal/data-factory-home-page.png)
10. Cliquez sur la vignette **Créer et surveiller** pour lancer l’interface utilisateur d’Azure Data Factory dans un onglet séparé.


## <a name="create-a-pipeline"></a>Créer un pipeline
Dans cette étape, vous allez créer un pipeline avec une activité de copie et deux activités web. Vous utilisez les fonctionnalités suivantes pour créer le pipeline :

- Paramètres du pipeline auxquels les jeux de données ont accès.
- Activité Web pour appeler des flux de travail Logic Apps pour envoyer des messages de réussite/échec.
- Connexion d’une activité à une autre activité (cas de réussite et échec)
- Utilisation de la sortie d’une activité en tant qu’entrée de l’activité suivante

1. Sur la page **Prise en main** de l’interface utilisateur de Data Factory, cliquez sur la vignette **Créer un pipeline**.  

   ![Page de prise en main](./media/tutorial-control-flow-portal/get-started-page.png)
3. Dans la fenêtre Propriétés pour le pipeline, basculez vers l’onglet **Paramètres** et utilisez le bouton **Nouveau** pour ajouter les trois paramètres suivants de type chaîne : sourceBlobContainer, sinkBlobContainer et récepteur.

    - **sourceBlobContainer** - paramètre dans le pipeline consommé par le jeu de données d’objet blob source.
    - **sinkBlobContainer** - paramètre dans le pipeline consommé par le jeu de données d’objet blob récepteur
    - **receiver** - ce paramètre est utilisé par les deux activités web dans le pipeline qui envoient des e-mails de réussite ou d’échec au récepteur dont l’adresse e-mail est spécifiée par ce paramètre.

   ![Menu Nouveau pipeline](./media/tutorial-control-flow-portal/pipeline-parameters.png)
4. Dans la boîte à outils **Activités**, développez **Flux de données** et glissez-déposez l’activité de **copie** vers la surface du concepteur de pipeline.

   ![Glisser-déposer de l’activité de copie](./media/tutorial-control-flow-portal/drag-drop-copy-activity.png)
5. Dans la fenêtre **Propriétés** pour l’activité de **copie** en bas, basculez vers l’onglet **Source**, puis cliquez sur **+ Nouveau**. Vous créez un jeu de données source pour l’activité de copie dans cette étape.

   ![Capture d’écran montrant comment créer un jeu de données source pour l’activité de copie.](./media/tutorial-control-flow-portal/new-source-dataset-button.png)
6. Dans la fenêtre **Nouveau jeu de données**, sélectionnez **Stockage Blob Azure**, puis cliquez sur **Terminer**.

   ![Sélectionner le stockage Blob Azure](./media/tutorial-control-flow-portal/select-azure-blob-storage.png)
7. Vous voyez un nouvel **onglet** intitulé **AzureBlob1**. Modifiez le nom du jeu de données à **SourceBlobDataset**.

   ![Paramètres généraux du jeu de données](./media/tutorial-control-flow-portal/dataset-general-page.png)
8. Basculez vers l’onglet **Connexion** dans la fenêtre **Propriétés**, puis cliquez sur Nouveau pour **Service lié**. Vous créez un service lié qui relie votre compte de stockage Azure à la fabrique de données à cette étape.

   ![Connexion à un jeu de données - nouveau service lié](./media/tutorial-control-flow-portal/dataset-connection-new-button.png)
9. Dans la fenêtre **Nouveau service lié**, procédez comme suit :

    1. Entrez **AzureStorageLinkedService** pour **Nom**.
    2. Sélectionnez votre compte de stockage Azure pour le **Nom du compte de stockage**.
    3. Cliquez sur **Enregistrer**.

   ![Nouveau service lié Stockage Azure](./media/tutorial-control-flow-portal/new-azure-storage-linked-service.png)
12. Saisissez `@pipeline().parameters.sourceBlobContainer` pour le dossier et `emp.txt` pour le nom du fichier. Le paramètre de pipeline sourceBlobContainer vous permet de définir le chemin d’accès de dossier pour le jeu de données.

    ![Paramètres du jeu de données source](./media/tutorial-control-flow-portal/source-dataset-settings.png)

13. Basculez vers l’onglet **pipeline** (ou) cliquez sur le pipeline dans l’arborescence. Vérifiez que **SourceBlobDataset** est sélectionné comme **jeu de données source**.
      
   ![Jeu de données source](./media/tutorial-control-flow-portal/pipeline-source-dataset-selected.png)

13. Dans la fenêtre Propriétés, basculez vers l’onglet **Récepteur**, puis cliquez sur **+ Nouveau** pour **Jeu de données récepteur**. Vous créez un jeu de données récepteur pour l’activité de copie dans cette étape d’une façon similaire à la création du jeu de données source.

    ![Bouton Nouveau jeu de données récepteur](./media/tutorial-control-flow-portal/new-sink-dataset-button.png)
14. Dans la fenêtre **Nouveau jeu de données**, sélectionnez **Stockage Blob Azure**, puis cliquez sur **Terminer**.
15. Sur la page de paramètres **Général** pour le jeu de données, entrez **SinkBlobDataset** comme **nom**.
16. Basculez vers l’onglet **Connexions**, et procédez comme suit :

    1. Sélectionnez **AzureStorageLinkedService** comme **Service lié**.
    2. Saisissez `@pipeline().parameters.sinkBlobContainer` pour le dossier.
    1. Entrez `@CONCAT(pipeline().RunId, '.txt')` pour le nom de fichier. L’expression utilise l’ID de l’exécution de pipeline actuelle comme nom de fichier. Pour obtenir la liste des variables système et expressions prises en charge, consultez [Variables système](control-flow-system-variables.md) et [Langage d’expression](control-flow-expression-language-functions.md).

        ![Paramètres du jeu de données récepteur](./media/tutorial-control-flow-portal/sink-dataset-settings.png)
17. Basculez vers l’onglet **Pipeline** en haut. Développez **Général** dans la boîte à outils **Activités**, et glissez-déposez une activité **Web** vers la surface du concepteur de pipeline. Définissez le nom de l’activité sur **SendSuccessEmailActivity**. L’activité web permet d’appeler n’importe quel point de terminaison REST. Pour plus d’informations sur l’activité, consultez [Activité web](control-flow-web-activity.md). Ce pipeline utilise une activité web pour appeler le flux de travail d’un e-mail Logic Apps.

    ![Glisser-déposer la première activité Web](./media/tutorial-control-flow-portal/success-web-activity-general.png)
18. Basculez vers l’onglet **Paramètres** à partir de l’onglet **Général**, et procédez comme suit :
    1. Pour l’**URL**, spécifiez l’URL du flux de travail Logic Apps qui envoie l’e-mail de réussite.  
    2. Sélectionnez **POST** comme **méthode**.
    3. Cliquez sur le lien **+ Ajouter un en-tête** dans la section **En-têtes**.
    4. Ajoutez un en-tête **Content-Type** et affectez-lui la valeur **application/json**.
    5. Spécifiez le texte JSON suivant pour le **corps**.

        ```json
        {
            "message": "@{activity('Copy1').output.dataWritten}",
            "dataFactoryName": "@{pipeline().DataFactory}",
            "pipelineName": "@{pipeline().Pipeline}",
            "receiver": "@pipeline().parameters.receiver"
        }
        ```
        Le corps du message contient les propriétés suivantes :

       - Message - Passage de la valeur `@{activity('Copy1').output.dataWritten`. Accède à une propriété de l’activité de copie précédente et passe la valeur de dataWritten. Pour un échec, passez la sortie de l’erreur au lieu de `@{activity('CopyBlobtoBlob').error.message`.
       - Nom de la fabrique de données - Passage de la valeur `@{pipeline().DataFactory}`. Il s’agit d’une variable système, qui vous permet d’accéder au nom de la fabrique de données correspondante. Consultez l’article [Variables système](control-flow-system-variables.md) pour obtenir la liste des variables système.
       - Nom du pipeline : Passage de la valeur `@{pipeline().Pipeline}`. Il s’agit également d’une variable système, qui vous permet d’accéder au nom du pipeline correspondant.
       - Récepteur - Passage de la valeur "\@pipeline().parameters.receiver"). Accès aux paramètres de pipeline.

         ![Paramètres pour la première activité Web](./media/tutorial-control-flow-portal/web-activity1-settings.png)         
19. Connectez l’activité de **copie** à l’activité **Web** en faisant glisser le bouton vert proche l’activité de copie vers l’activité Web.

    ![Connecter l’activité de copie avec la première activité Web](./media/tutorial-control-flow-portal/connect-copy-web-activity1.png)
20. Faites glisser une autre activité **Web** à partir de la boîte à outils Activités vers la zone du concepteur de pipeline et configurez le **nom** comme **SendFailureEmailActivity**.

    ![Nom de la deuxième activité Web](./media/tutorial-control-flow-portal/web-activity2-name.png)
21. Basculez vers l’onglet **Paramètres**, et procédez comme suit :

    1. Pour l’**URL**, spécifiez l’URL du flux de travail Logic Apps qui envoie l’e-mail d’échec.  
    2. Sélectionnez **POST** comme **méthode**.
    3. Cliquez sur le lien **+ Ajouter un en-tête** dans la section **En-têtes**.
    4. Ajoutez un en-tête **Content-Type** et affectez-lui la valeur **application/json**.
    5. Spécifiez le texte JSON suivant pour le **corps**.

        ```json
        {
            "message": "@{activity('Copy1').error.message}",
            "dataFactoryName": "@{pipeline().DataFactory}",
            "pipelineName": "@{pipeline().Pipeline}",
            "receiver": "@pipeline().parameters.receiver"
        }
        ```

        ![Paramètres pour la deuxième activité Web](./media/tutorial-control-flow-portal/web-activity2-settings.png)         
22. Sélectionnez l’activité de **copie** dans le concepteur de pipeline, puis cliquez sur le bouton **+->** , puis sélectionnez **Erreur**.  

    ![Capture d’écran montrant comment sélectionner une erreur sur l’activité de copie dans le concepteur de pipeline.](./media/tutorial-control-flow-portal/select-copy-failure-link.png)
23. Faites glisser le bouton **rouge** proche de l’activité de copie vers la deuxième activité Web **SendFailureEmailActivity**. Vous pouvez déplacer les activités afin que le pipeline ressemble à l’image suivante :

    ![Pipeline complète avec toutes les activités](./media/tutorial-control-flow-portal/full-pipeline.png)
24. Pour valider le pipeline, cliquez sur le bouton **Valider** dans la barre d’outils. Fermez la fenêtre **Sortie de validation du pipeline** en cliquant sur le bouton **>>** .

    ![Valider le pipeline](./media/tutorial-control-flow-portal/validate-pipeline.png)
24. Pour publier les entités (jeux de données, pipelines, etc.) sur Data Factory, cliquez sur **Publish All** (Publier tout). Patientez jusqu’à voir le message **Publication réussie**.

    ![Publish](./media/tutorial-control-flow-portal/publish-button.png)

## <a name="trigger-a-pipeline-run-that-succeeds"></a>Déclencher une exécution de pipeline qui réussit
1. Pour **déclencher** une exécution de pipeline, cliquez sur **Déclencher** dans la barre d’outils, puis sur **Déclencher maintenant**.

    ![Déclencher une exécution du pipeline](./media/tutorial-control-flow-portal/trigger-now-menu.png)
2. Dans la fenêtre **Exécution de pipeline**, procédez comme suit :

    1. Entrez **adftutorial/adfv2branch/input** pour le paramètre **sourceBlobContainer**.
    2. Entrez **adftutorial/adfv2branch/output** pour le paramètre **sinkBlobContainer**.
    3. Entrez une **adresse e-mail** du **récepteur**.
    4. Cliquez sur **Terminer**

        ![Paramètres de l’exécution de pipeline](./media/tutorial-control-flow-portal/pipeline-run-parameters.png)

## <a name="monitor-the-successful-pipeline-run"></a>Surveiller l’exécution de pipeline réussie

1. Pour surveiller l’exécution de pipeline, basculez vers l’onglet **Surveiller** sur la gauche. Vous voyez l’exécution de pipeline que vous avez déclenchée manuellement. Utilisez le bouton **Actualiser** pour actualiser la liste.

    ![Exécution de pipeline réussie](./media/tutorial-control-flow-portal/monitor-success-pipeline-run.png)
2. Pour **voir les exécutions d’activité** associées à l’exécution de ce pipeline, cliquez sur le premier lien dans la colonne **Actions**. Vous pouvez revenir à la vue précédente en cliquant sur **Pipelines** en haut. Utilisez le bouton **Actualiser** pour actualiser la liste.

    ![Capture d’écran montrant comment afficher la liste des exécutions d’activités.](./media/tutorial-control-flow-portal/activity-runs-success.png)

## <a name="trigger-a-pipeline-run-that-fails"></a>Déclencher une exécution de pipeline qui échoue
1. Basculez vers l’onglet **Modifier** sur la gauche.
2. Pour **déclencher** une exécution de pipeline, cliquez sur **Déclencher** dans la barre d’outils, puis sur **Déclencher maintenant**.
3. Dans la fenêtre **Exécution de pipeline**, procédez comme suit :

    1. Entrez **adftutorial/dummy/input** pour le paramètre **sourceBlobContainer**. Assurez-vous que le dossier factice n’existe pas dans le conteneur adftutorial.
    2. Entrez **adftutorial/dummy/output** pour le paramètre **sinkBlobContainer**.
    3. Entrez une **adresse e-mail** du **récepteur**.
    4. Cliquez sur **Terminer**.

## <a name="monitor-the-failed-pipeline-run"></a>Surveiller l’exécution du pipeline qui a échoué

1. Pour surveiller l’exécution de pipeline, basculez vers l’onglet **Surveiller** sur la gauche. Vous voyez l’exécution de pipeline que vous avez déclenchée manuellement. Utilisez le bouton **Actualiser** pour actualiser la liste.

    ![Exécution de pipeline qui a échoué](./media/tutorial-control-flow-portal/monitor-failure-pipeline-run.png)
2. Cliquez sur le lien **Erreur** pour l’exécution de pipeline afin d’afficher des détails sur l’erreur de pipeline.

    ![Erreur de pipeline](./media/tutorial-control-flow-portal/pipeline-error-message.png)
2. Pour **voir les exécutions d’activité** associées à l’exécution de ce pipeline, cliquez sur le premier lien dans la colonne **Actions**. Utilisez le bouton **Actualiser** pour actualiser la liste. Notez que l’activité de copie dans le pipeline a échoué. L’activité Web a réussi à envoyer l’e-mail d’échec au destinataire spécifié.

    ![Exécutions d’activités](./media/tutorial-control-flow-portal/activity-runs-failure.png)
4. Cliquez sur le lien **Erreur** dans la colonne **Actions** afin d’afficher des détails sur l’erreur.

    ![Erreur d’exécution d’activité](./media/tutorial-control-flow-portal/activity-run-error.png)

## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez effectué les étapes suivantes :

> [!div class="checklist"]
> * Créer une fabrique de données.
> * Créer un service lié Stockage Azure
> * Créer un jeu de données d’objet blob Azure
> * Créer un pipeline qui contient une activité de copie et une activité web
> * Envoyer les sorties des activités aux activités ultérieures
> * Utiliser des variables système et de passage de paramètres
> * Démarrer une exécution de pipeline
> * Surveiller les exécutions de pipeline et d’activité

Vous pouvez maintenant passer à la section Concepts pour plus d’informations sur Azure Data Factory.
> [!div class="nextstepaction"]
>[Pipelines et activités](concepts-pipelines-activities.md)

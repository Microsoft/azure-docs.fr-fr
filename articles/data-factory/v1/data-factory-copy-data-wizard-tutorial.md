---
title: 'Tutoriel : Créer un pipeline à l’aide de l’Assistant de copie '
description: Dans ce didacticiel, vous allez créer un pipeline Azure Data Factory avec une activité de copie, à l’aide de l’Assistant de copie et de Data Factory.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.assetid: b87afb8e-53b7-4e1b-905b-0343dd096198
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 93360e48dad13b9ec57175d31ecb61d32974f066
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93128400"
---
# <a name="tutorial-create-a-pipeline-with-copy-activity-using-data-factory-copy-wizard"></a>Tutoriel : Créer un pipeline avec l’activité de copie à l’aide de l’Assistant de copie de Data Factory
> [!div class="op_single_selector"]
> * [Vue d’ensemble et composants requis](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Assistant de copie](data-factory-copy-data-wizard-tutorial.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Modèle Azure Resource Manager](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [API .NET](data-factory-copy-activity-tutorial-using-dotnet-api.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez le [tutoriel sur l’activité de copie](../quickstart-create-data-factory-dot-net.md). 


Ce tutoriel vous montre comment utiliser l’**Assistant de copie** pour copier des données à partir d’un stockage Blob Azure dans Azure SQL Database. 

**L’Assistant de copie** Azure Data Factory vous permet de créer rapidement un pipeline de données qui copie les données d’un magasin de données source pris en charge dans un magasin de données de destination pris en charge. Par conséquent, nous vous recommandons d’utiliser l’Assistant en vue de créer un exemple de pipeline pour votre scénario de déplacement de données. Pour obtenir la liste des magasins de données pris en charge en tant que sources et destinations, consultez [Magasins de données pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats) .  

Ce didacticiel vous montre comment créer une fabrique de données Azure, lancer l’Assistant Copie et suivre une série d’étapes pour fournir des informations sur votre scénario d’ingestion/déplacement de données. Une fois les étapes de l’Assistant terminées, celui-ci crée automatiquement un pipeline avec une activité de copie pour copier des données d’un stockage Blob Azure vers Azure SQL Database. Pour plus d’informations sur l’activité de copie, consultez [Activités de déplacement des données](data-factory-data-movement-activities.md).

## <a name="prerequisites"></a>Prérequis
Assurez-vous que vous respectez la configuration requise décrite dans l’article [Vue d’ensemble du didacticiel](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) avant de suivre ce didacticiel.

## <a name="create-data-factory"></a>Créer une fabrique de données
Dans cette étape, vous allez utiliser le portail Azure pour créer une fabrique de données Azure nommée **ADFTutorialDataFactory**.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Cliquez sur **Créer une ressource** en haut à gauche, cliquez sur **Données + analyse**, puis sur **Data Factory**. 
   
   ![Nouveau -> DataFactory](./media/data-factory-copy-data-wizard-tutorial/new-data-factory-menu.png)
2. Dans le panneau **Nouvelle fabrique de données** :
   
   1. Saisissez **ADFTutorialDataFactory** dans le champ **Nom**.
       Le nom de la fabrique de données Azure doit être un nom global unique. Si l’erreur `Data factory name “ADFTutorialDataFactory” is not available` s’affiche, changez le nom de la fabrique de données (par exemple, votrenomADFTutorialDataFactoryAAAAMMJJ), puis tentez de la recréer. Consultez la rubrique [Data Factory - Règles d’affectation des noms](data-factory-naming-rules.md) pour savoir comment nommer les artefacts Data Factory.  
      
       ![Nom de la fabrique de données indisponible](./media/data-factory-copy-data-wizard-tutorial/getstarted-data-factory-not-available.png)    
   2. Sélectionnez votre **abonnement** Azure.
   3. Pour Groupe de ressources, effectuez l’une des opérations suivantes : 
      
      - Sélectionnez **Utiliser l’existant** pour sélectionner un groupe de ressources existant.
      - Sélectionnez **Créer un nouveau** pour entrer un nom pour un groupe de ressources.
          
        Certaines étapes de ce tutoriel utilisent le nom : **ADFTutorialResourceGroup** pour le groupe de ressources. Pour plus d’informations sur les groupes de ressources, consultez [Utilisation des groupes de ressources pour gérer vos ressources Azure](../../azure-resource-manager/management/overview.md).
   4. Sélectionnez un **emplacement** pour la fabrique de données.
   5. Sélectionnez la case à cocher **Épingler au tableau de bord** en bas du panneau.  
   6. Cliquez sur **Créer**.
      
       ![Panneau Nouvelle fabrique de données](media/data-factory-copy-data-wizard-tutorial/new-data-factory-blade.png)            
3. Une fois la création terminée, le panneau **Data Factory** s’affiche comme sur l’image suivante :
   
   ![Page d’accueil Data Factory](./media/data-factory-copy-data-wizard-tutorial/getstarted-data-factory-home-page.png)

## <a name="launch-copy-wizard"></a>Lancer l’Assistant Copie
1. Dans le panneau Fabrique de données, cliquez sur **Copier les données** pour lancer **l’Assistant de copie**. 
   
   > [!NOTE]
   > Si vous voyez que le navigateur web est bloqué au niveau « Autorisation... », désactivez/décochez l’option **Block third party cookies and site data** (Bloquer les cookies et les données de site tiers) dans les paramètres du navigateur (ou) laissez cette option activée et créez une exception pour **login.microsoftonline.com**, puis essayez de relancer l’Assistant.
2. Dans la page **Propriétés** :
   
   1. Saisissez **CopyFromBlobToAzureSql** dans **Nom de la tâche**.
   2. Saisissez une **Description** (facultative).
   3. Modifiez les champs **Date et heure de début** et **Date et heure de fin** de manière à définir la date de fin sur la date du jour et la date de début cinq jours plus tôt.  
   4. Cliquez sur **Suivant**.  
      
      ![Outil de copie - page Propriétés](./media/data-factory-copy-data-wizard-tutorial/copy-tool-properties-page.png) 
3. Dans la page **Source data store** (Magasin de données source), cliquez sur la vignette **Stockage d’objets blob Azure**. Cette page sert à spécifier le magasin de données source pour la tâche de copie. 
   
    ![Outil de copie - page Source data store (Magasin de données source)](./media/data-factory-copy-data-wizard-tutorial/copy-tool-source-data-store-page.png)
4. Dans la page **Specify the Azure Blob storage account** (Spécifier le compte de stockage d’objets blob Azure) :
   
   1. Saisissez **AzureStorageLinkedService** dans **Nom du service lié**.
   2. Vérifiez que l’option **À partir des abonnements** est sélectionnée pour **Account selection method** (Méthode de sélection du compte).
   3. Sélectionnez votre **abonnement** Azure.  
   4. Sélectionnez un **compte de stockage Azure** dans la liste des comptes de stockage Azure disponibles dans l’abonnement sélectionné. Vous pouvez également choisir de saisir manuellement les paramètres du compte de stockage en sélectionnant l’option **Saisir manuellement** dans **Account selection method** (Méthode de sélection de compte). Cliquez ensuite sur **Suivant**. 
      
      ![Outil de copie - spécifiez le compte de stockage d’objets blob Azure](./media/data-factory-copy-data-wizard-tutorial/copy-tool-specify-azure-blob-storage-account.png)
5. Dans la page **Choose the input file or folder** (Choisir le fichier ou le dossier d’entrée) :
   
   1. Double-cliquez sur **adftutorial** (dossier).
   2. Sélectionnez **emp.txt**, puis cliquez sur **Choisir**.
      
      ![Capture d’écran montrant l’option Choisir pour votre fichier d’entrée.](./media/data-factory-copy-data-wizard-tutorial/copy-tool-choose-input-file-or-folder.png)
6. Sur la page **Choose the input file or folder (Choisir le fichier ou le dossier d’entrée)** , cliquez sur **Suivant**. Ne sélectionnez pas **copie binaire**. 
   
    ![Capture d’écran montrant l’option Copie binaire pour votre entrée.](./media/data-factory-copy-data-wizard-tutorial/chose-input-file-folder.png) 
7. Dans la page **File format settings** (Paramètres de format de fichier), vous pouvez voir les délimiteurs et le schéma qui sont détectés automatiquement par l’Assistant en analysant le fichier. Vous pouvez également entrer les délimiteurs manuellement pour que l’Assistant copie arrête leur détection automatique ou pour remplacer les délimiteurs détectés. Une fois que vous avez vérifié les délimiteurs et afficher un aperçu des données, cliquez sur **Suivant**. 
   
    ![Outil de copie - Paramètres de format de fichier](./media/data-factory-copy-data-wizard-tutorial/copy-tool-file-format-settings.png)  
8. Dans la page de la banque de données de destination, cliquez sur la vignette **Azure SQL Database**, puis sur **Suivant**.
   
    ![Outil de copie - Choisir une banque de destination](./media/data-factory-copy-data-wizard-tutorial/choose-destination-store.png)
9. Dans la page **Specify the Azure SQL database** (Spécifier la base de données Azure SQL Database) :
   
   1. Saisissez **AzureSqlLinkedService** dans le champ **Nom de la connexion**.
   2. Vérifiez que l’option **À partir des abonnements** est sélectionnée pour **Server / database selection method** (Méthode de sélection du serveur/de la base de données).
   3. Sélectionnez votre **abonnement** Azure.  
   4. Sélectionnez le **Nom du serveur** et la **Base de données**.
   5. Saisissez le **Nom d’utilisateur** et le **Mot de passe**.
   6. Cliquez sur **Suivant**.  
      
      ![Outil de copie - Spécifier Azure SQL Database](./media/data-factory-copy-data-wizard-tutorial/specify-azure-sql-database.png)
10. Dans la page **Mappage de table**, sélectionnez **emp** dans la liste déroulante du champ **Destination**, puis cliquez sur **Flèche vers le bas** (facultatif) pour afficher le schéma et un aperçu des données.
    
     ![Outil de copie - Mappage de Table](./media/data-factory-copy-data-wizard-tutorial/copy-tool-table-mapping-page.png) 
11. Dans la page **Mappage de schéma** cliquez sur **Suivant**.
    
    ![Outil de copie - Mappage de schéma](./media/data-factory-copy-data-wizard-tutorial/schema-mapping-page.png)
12. Dans la page **Paramètres de performances** cliquez sur **Suivant**. 
    
    ![Capture d’écran montrant la page Paramètres de performances, dans laquelle vous pouvez sélectionner Suivant.](./media/data-factory-copy-data-wizard-tutorial/performance-settings.png)
13. Passez en revue les informations contenues dans la page **Résumé**, puis cliquez sur **Terminer**. L’Assistant crée deux services liés, deux jeux de données (entrée et sortie) et un pipeline dans la fabrique de données (d’où vous avez lancé l’Assistant Copie). 
    
    ![Capture d’écran montrant la page Résumé, dans laquelle vous pouvez sélectionner Suivant.](./media/data-factory-copy-data-wizard-tutorial/summary-page.png)

## <a name="launch-monitor-and-manage-application"></a>Lancer l’application Surveiller et gérer
1. Dans la page **Déploiement**, cliquez sur le lien : `Click here to monitor copy pipeline`.
   
   ![Outil de copie - Déploiement réussi](./media/data-factory-copy-data-wizard-tutorial/copy-tool-deployment-succeeded.png)  
2. L’application d’analyse est lancée dans un onglet distinct de votre navigateur web.   
   
   ![Application de surveillance](./media/data-factory-copy-data-wizard-tutorial/monitoring-app.png)   
3. Pour afficher le dernier état des tranches horaires, cliquez sur le bouton **Actualiser** dans la liste **FENÊTRES D’ACTIVITÉ** en bas. Cette opération affiche cinq fenêtres d’activité pour cinq jours entre les heures de début et de fin du pipeline. Comme la liste n’est pas actualisée automatiquement, vous devrez peut-être cliquer sur Actualiser plusieurs fois avant que toutes les fenêtres d’activité soient à l’état Prêt. 
4. Sélectionnez une fenêtre d’activité dans la liste. Affichez les informations la concernant dans **l’Explorateur de fenêtres d’activité** à droite.

    ![Détails de la fenêtre d’activité](media/data-factory-copy-data-wizard-tutorial/activity-window-details.png)    

    Notez que les dates, 11, 12, 13, 14 et 15 sont en vert, ce qui signifie que les tranches de sortie quotidiennes de ces dates ont déjà été produites. Ce codage couleur apparaît également sur le pipeline et dans le jeu de données de sortie de la vue du diagramme. À l’étape précédente, notez que deux tranches ont déjà été produites et une tranche est en cours de traitement. Les deux autres sont en attente de traitement (comme l’indique le codage couleur). 

    Pour plus d’informations sur l’utilisation de cette application, consultez l’article [Surveiller et gérer les pipelines Azure Data Factory à l’aide de la nouvelle application de surveillance et de gestion](data-factory-monitor-manage-app.md).

## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez utilisé le stockage Blob Azure comme magasin de données source et Azure SQL Database comme banque de données de destination dans une opération de copie. Le tableau ci-dessous contient la liste des magasins de données pris en charge en tant que sources et destinations par l’activité de copie : 

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

Pour plus d’informations sur les champs/propriétés affichés dans l’Assistant de copie d’un magasin de données, cliquez sur le lien du magasin de données dans la table. 
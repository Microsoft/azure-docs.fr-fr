---
title: 'Transformer des données avec Spark dans Azure Data Factory '
description: Ce didacticiel fournit des instructions détaillées sur la transformation des données à l’aide d’une activité Spark dans Azure Data Factory.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.date: 01/10/2018
author: nabhishek
ms.author: abnarain
manager: anandsub
ms.openlocfilehash: 5b0bcdd66e17fb93a560b6073c13e3170e3ab37b
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010144"
---
# <a name="transform-data-in-the-cloud-by-using-a-spark-activity-in-azure-data-factory"></a>Transformer des données dans le cloud à l’aide d’une activité Spark dans Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Dans ce didacticiel, vous utilisez le portail Azure pour créer un pipeline Azure Data Factory. Ce pipeline transforme les données à l’aide de l’activité Spark et un service lié HDInsight de la demande. 

Dans ce tutoriel, vous allez effectuer les étapes suivantes :

> [!div class="checklist"]
> * Créer une fabrique de données. 
> * Créer un pipeline qui utilise une activité Spark.
> * Déclencher une exécution du pipeline.
> * Surveiller l’exécution du pipeline.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

* **Compte Azure Storage**. Vous créez un script Python et un fichier d’entrée, puis vous les chargez vers le stockage Azure. La sortie du programme Spark est stockée dans ce compte de stockage. Le cluster Spark sur demande utilise le même compte de stockage comme stockage principal.  

> [!NOTE]
> HdInsight prend en charge uniquement les comptes de stockage universels avec un niveau standard. Assurez-vous que le compte n’est pas un compte de stockage premium ou un objet blob uniquement.

* **Azure PowerShell**. Suivez les instructions de la page [Installation et configuration d’Azure PowerShell](/powershell/azure/install-Az-ps).


### <a name="upload-the-python-script-to-your-blob-storage-account"></a>Charger le script Python dans votre compte de stockage d’objets Blob
1. Créez un fichier Python nommé **WordCount_Spark.py** avec le contenu suivant : 

    ```python
    import sys
    from operator import add
    
    from pyspark.sql import SparkSession
    
    def main():
        spark = SparkSession\
            .builder\
            .appName("PythonWordCount")\
            .getOrCreate()
            
        lines = spark.read.text("wasbs://adftutorial@<storageaccountname>.blob.core.windows.net/spark/inputfiles/minecraftstory.txt").rdd.map(lambda r: r[0])
        counts = lines.flatMap(lambda x: x.split(' ')) \
            .map(lambda x: (x, 1)) \
            .reduceByKey(add)
        counts.saveAsTextFile("wasbs://adftutorial@<storageaccountname>.blob.core.windows.net/spark/outputfiles/wordcount")
        
        spark.stop()
    
    if __name__ == "__main__":
        main()
    ```
1. Remplacez *&lt;storageAccountName&gt;* par le nom de votre compte de stockage Azure. Puis enregistrez le fichier. 
1. Dans le stockage Blob Azure, créez un conteneur nommé **adftutorial** s’il n’existe pas. 
1. Créez un dossier nommé **spark**.
1. Créez un sous-dossier nommé **script** sous le dossier **spark**. 
1. Téléchargez le fichier **WordCount_Spark.py** dans le sous-dossier **script**. 


### <a name="upload-the-input-file"></a>Télécharger le fichier d’entrée
1. Créez un fichier nommé **minecraftstory.txt** avec du texte. Le programme Spark compte le nombre de mots dans ce texte. 
1. Créez un sous-dossier nommé **inputfiles** sous le dossier **spark**. 
1. Chargez le fichier **minecraftstory.txt** dans le sous-dossier **inputfiles**. 

## <a name="create-a-data-factory"></a>Créer une fabrique de données

1. Lancez le navigateur web **Microsoft Edge** ou **Google Chrome**. L’interface utilisateur de Data Factory n’est actuellement prise en charge que par les navigateurs web Microsoft Edge et Google Chrome.
1. Sélectionnez **Nouveau** dans le menu de gauche, sélectionnez **Données + Analytique**, puis **Data Factory**. 
   
   ![Sélection Data Factory dans le volet « Nouveau »](./media/tutorial-transform-data-spark-portal/new-azure-data-factory-menu.png)
1. Dans le volet **Nouvelle fabrique de données**, entrez **ADFTutorialDataFactory** sous **Nom**. 
      
   ![Page « Nouvelle fabrique de données »](./media/tutorial-transform-data-spark-portal/new-azure-data-factory.png)
 
   Le nom de la fabrique de données Azure doit être un nom *global unique*. Si vous voyez l’erreur suivante, modifiez le nom de la fabrique de données. Par exemple, utilisez **&lt;votrenom&gt;ADFTutorialDataFactory**. Consultez l’article [Data Factory - Règles d’affectation des noms](naming-rules.md) pour en savoir plus sur les règles d’affectation des noms d’artefacts Data Factory.
  
   ![Erreur quand le nom n’est pas disponible](./media/tutorial-transform-data-spark-portal/name-not-available-error.png)
1. Pour **Abonnement**, sélectionnez l’abonnement Azure dans lequel vous voulez créer la fabrique de données. 
1. Pour **Groupe de ressources**, réalisez l’une des opérations suivantes :
     
   - Sélectionnez **Utiliser l’existant**, puis sélectionnez un groupe de ressources existant dans la liste déroulante. 
   - Sélectionnez **Créer**, puis entrez le nom d’un groupe de ressources.   
         
   Certaines étapes de ce guide de démarrage rapide supposent que vous utilisez le nom **ADFTutorialResourceGroup** pour le groupe de ressources. Pour plus d’informations sur les groupes de ressources, consultez [Utilisation des groupes de ressources pour gérer vos ressources Azure](../azure-resource-manager/management/overview.md).  
1. Pour **Version**, sélectionnez **V2**.
1. Pour **Emplacement**, sélectionnez l’emplacement de la fabrique de données. 

   Pour obtenir la liste des régions Azure dans lesquelles Data Factory est actuellement disponible, sélectionnez les régions qui vous intéressent dans la page suivante, puis développez **Analytique** pour localiser **Data Factory** : [Disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/). Les magasins de données (tels que Stockage Azure et Azure SQL Database) et les services de calcul (comme HDInsight) utilisés par Data Factory peuvent se trouver dans d’autres régions.

1. Sélectionnez **Create** (Créer).

1. Une fois la création terminée, la page **Data Factory** s’affiche. Sélectionnez la vignette **Créer et surveiller** pour démarrer l’application d’interface utilisateur (IU) de Data Factory dans un onglet séparé.

    ![Page d’accueil de la fabrique de données, avec la vignette « Créer et surveiller »](./media/tutorial-transform-data-spark-portal/data-factory-home-page.png)

## <a name="create-linked-services"></a>Créez des services liés
Vous créez deux services liés dans cette section : 
    
- Un **service lié au stockage Azure** relie un compte de stockage Azure à la fabrique de données. Ce stockage est utilisé par le cluster HDInsight à la demande. Il contient également le script Spark à exécuter. 
- Un **service lié HDInsight à la demande**. Azure Data Factory crée un cluster HDInsight et exécute le programme Spark automatiquement. Il supprime ensuite le cluster HDInsight une fois que le cluster inactif pendant une période préconfigurée. 

### <a name="create-an-azure-storage-linked-service"></a>Créer un service lié Stockage Azure

1. Dans la page **Prise en main**, basculez vers l’onglet **Modifier** dans le volet gauche. 

   ![Page « Prise en main »](./media/tutorial-transform-data-spark-portal/get-started-page.png)

1. Cliquez sur **Connexions** au bas de la fenêtre, puis cliquez sur **+ Nouveau**. 

   ![Boutons pour la création d’une nouvelle connexion](./media/tutorial-transform-data-spark-portal/new-connection.png)
1. Dans la fenêtre **Nouveau service lié**, sélectionnez **Magasin de données** > **Stockage Blob Azure**, puis cliquez sur **Continuer**. 

   ![Sélectionner la vignette « Stockage Blob Azure »](./media/tutorial-transform-data-spark-portal/select-azure-storage.png)
1. Pour **Nom du compte de stockage**, sélectionnez le nom dans la liste, puis cliquez sur **Enregistrer**. 

   ![Zone pour spécifier le nom du compte de stockage](./media/tutorial-transform-data-spark-portal/new-azure-storage-linked-service.png)


### <a name="create-an-on-demand-hdinsight-linked-service"></a>Créer un service lié HDInsight à la demande

1. Cliquez de nouveau sur le bouton **+ Nouveau** pour créer un nouveau service lié. 
1. Dans la fenêtre **Nouveau service lié**, sélectionnez **Calcul** > **Azure HDInsight**, puis cliquez sur **Continuer**. 

   ![Sélectionner la vignette « Azure HDInsight »](./media/tutorial-transform-data-spark-portal/select-azure-hdinsight.png)
1. Dans la fenêtre **Nouveau service lié**, procédez comme suit : 

   a. Entrez **AzureHDInsightLinkedService** pour le **Nom**.
   
   b. Vérifiez que **HDInsight à la demande** est le **Type** sélectionné.
   
   c. Sélectionnez **AzureBlobStorage1** pour **Service lié Stockage Azure**. Vous avez déjà créé ce service lié. Si vous avez utilisé un nom différent, spécifiez le nom correct. 
   
   d. Sélectionnez **spark** pour **Type de Cluster**.
   
   e. Entrez l’ID du principal de service ayant l’autorisation de créer un cluster HDInsight pour **ID du principal de service**. 
   
      Ce principal de service doit être membre du rôle Contributeur de l’abonnement ou du groupe de ressources dans lequel le cluster est créé. Pour plus d’informations, consultez [Créer une application Active Directory et un principal de service](../active-directory/develop/howto-create-service-principal-portal.md). L’**ID de principal de service** est équivalent à l’*ID d’application* et une **clé de principal de service** est équivalente à la valeur d’un *secret client*.
   
   f. Pour **Clé du principal de service**, saisissez la clé. 
   
   g. Pour **Groupe de ressources**, sélectionnez le même groupe de ressources que celui utilisé lors de la création de la fabrique de données. Le cluster Spark est créé dans ce groupe de ressources. 
   
   h. Développer **le type de système d’exploitation**.
   
   i. Entrez un nom pour le **nom d’utilisateur du cluster**. 
   
   j. Entrez le **mot de passe du cluster** correspondant à l’utilisateur. 
   
   k. Sélectionnez **Terminer**. 

   ![Paramètres du service lié HDInsight](./media/tutorial-transform-data-spark-portal/azure-hdinsight-linked-service-settings.png)

> [!NOTE]
> Azure HDInsight présente une limite relative au nombre total de cœurs que vous pouvez utiliser dans chaque région Azure prise en charge. Pour le service lié HDInsight à la demande, le cluster HDInsight est créé au même emplacement de stockage Azure que celui utilisé comme stockage principal. Vérifiez que vous disposez des quotas de cœurs suffisants pour pouvoir créer le cluster avec succès. Pour plus d’informations, reportez-vous à l’article [Configurer des clusters dans HDInsight avec Hadoop, Spark, Kafka et bien plus encore](../hdinsight/hdinsight-hadoop-provision-linux-clusters.md). 

## <a name="create-a-pipeline"></a>Créer un pipeline

1. Cliquez sur le bouton **+** (plus), puis sélectionnez **Pipeline** dans le menu.

   ![Boutons pour créer un nouveau pipeline](./media/tutorial-transform-data-spark-portal/new-pipeline-menu.png)
1. Dans la boîte à outils **Activités**, étendez le **HDInsight**. Faites glisser l’activité **Spark** depuis la boîte à outils **Activités** vers la surface du concepteur de pipeline. 

   ![Faire glisser l’activité Spark](./media/tutorial-transform-data-spark-portal/drag-drop-spark-activity.png)
1. Dans les propriétés de la fenêtre d’activité **Spark** en bas, procédez comme suit : 

   a. Basculez vers l’onglet **Cluster HDI**.
   
   b. Sélectionnez **AzureHDInsightLinkedService** créé lors de la procédure précédente. 
        
   ![Spécifier le service lié HDInsight](./media/tutorial-transform-data-spark-portal/select-hdinsight-linked-service.png)
1. Basculez vers l’onglet **Script/Jar**, et procédez comme suit : 

   a. Sélectionnez **AzureBlobStorage1** pour **Service lié au travail**.
   
   b. Cliquez sur **Parcourir le stockage**.

   ![Spécifier le script Spark dans l’onglet « Script/Jar »](./media/tutorial-transform-data-spark-portal/specify-spark-script.png)
   
   c. Accédez au dossier **adftutorial/spark/script**, sélectionnez **WordCount_Spark.py**, puis cliquez sur **Terminer**.      

1. Pour valider le pipeline, cliquez sur le bouton **Valider** dans la barre d’outils. Cliquez sur le bouton **>>** flèche droite (>>) pour fermer la fenêtre de validation. 
    
   ![Bouton de validation](./media/tutorial-transform-data-spark-portal/validate-button.png)
1. Sélectionnez **Publier tout**. L’interface utilisateur de Data Factory publie des entités (services liés et pipelines) sur le service Azure Data Factory. 
    
   ![Bouton Publier tout](./media/tutorial-transform-data-spark-portal/publish-button.png)


## <a name="trigger-a-pipeline-run"></a>Déclencher une exécution du pipeline
Sélectionnez **Ajouter déclencheur** dans la barre d’outils, puis **Déclencher maintenant**. 

![Boutons Déclencher et Déclencher maintenant](./media/tutorial-transform-data-spark-portal/trigger-now-menu.png)

## <a name="monitor-the-pipeline-run"></a>Surveiller l’exécution du pipeline.

1. Basculez vers l’onglet **Surveiller**. Vérifiez qu’un pipeline est exécuté. La création d’un cluster Spark prend 20 minutes environ. 
   
1. Cliquez régulièrement sur **Actualiser** pour vérifier l’état de l’exécution des pipelines. 

   ![Onglet de Surveillance des exécutions de pipelines, avec le bouton « Actualiser »](./media/tutorial-transform-data-spark-portal/monitor-tab.png)

1. Pour voir les exécutions d’activités associées à l’exécution du pipeline, cliquez sur le lien **Afficher les exécutions d’activités** dans la colonne **Actions**.

   ![État de l’exécution de pipelines](./media/tutorial-transform-data-spark-portal/pipeline-run-succeeded.png) 

   Vous pouvez basculer vers la vue des exécutions de pipelines en cliquant sur le lien **Toutes les exécutions de pipeline** en haut.

   ![Vue des exécutions d’activités](./media/tutorial-transform-data-spark-portal/activity-runs.png)

## <a name="verify-the-output"></a>Vérifier la sortie
Vérifiez que le fichier de sortie est créé dans le dossier spark/otuputfiles/wordcount du conteneur adftutorial. 

![Emplacement du fichier de sortie](./media/tutorial-transform-data-spark-portal/verity-output.png)

Le fichier doit contenir chaque mot du fichier texte entrée et le nombre d’apparitions du mot dans le fichier. Par exemple : 

```
(u'This', 1)
(u'a', 1)
(u'is', 1)
(u'test', 1)
(u'file', 1)
```

## <a name="next-steps"></a>Étapes suivantes
Le pipeline dans cet exemple transforme les données à l’aide de l’activité Spark et un service lié HDInsight de la demande. Vous avez appris à : 

> [!div class="checklist"]
> * Créer une fabrique de données. 
> * Créer un pipeline qui utilise une activité Spark.
> * Déclencher une exécution du pipeline.
> * Surveiller l’exécution du pipeline.

Passez au tutoriel suivant pour découvrir comment transformer des données en exécutant un script Hive sur un cluster Azure HDInsight qui se trouve dans un réseau virtuel : 

> [!div class="nextstepaction"]
> [Tutoriel : Transformer des données à l’aide de Hive dans un réseau virtuel Azure](tutorial-transform-data-hive-virtual-network-portal.md).






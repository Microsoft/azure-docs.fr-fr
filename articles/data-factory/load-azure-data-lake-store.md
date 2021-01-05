---
title: Charger des données dans Azure Data Lake Storage Gen1
description: Utiliser Azure Data Factory pour copier des données dans Azure Data Lake Storage Gen1
services: data-factory
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 01/17/2018
ms.openlocfilehash: 0a111a1a44b508d721f16a038d0919334383d2c5
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93125000"
---
# <a name="load-data-into-azure-data-lake-storage-gen1-by-using-azure-data-factory"></a>Charger des données dans Azure Data Lake Storage Gen1 à l’aide d’Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

[Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-overview.md) (connu préalablement sous le nom Azure Data Lake Store) est un référentiel d’entreprise à très grande échelle pour les charges de travail d’analyse du Big Data. Data Lake Storage Gen1 vous permet de capturer des données de toute taille, de tout type et dont les vitesses d’ingestion sont variées. Les données sont capturées à un emplacement unique, à des fins d’analytique opérationnelle et exploratoire.

Azure Data Factory est un service informatique d’intégration de données informatique intégralement managé. Vous pouvez utiliser le service pour remplir le lac de données avec les données de votre système existant et gagner du temps lors de la création de vos solutions d’analyse.

Azure Data Factory offre les avantages suivants pour le chargement des données dans Data Lake Storage Gen1 :

* **Facilité de configuration**  : assistant intuitif en 5 étapes. Aucun script nécessaire.
* **Prise en charge étendue du magasin de données**  : prise en charge intégrée d’un ensemble complet de magasins de données locaux et dans le cloud. Pour une liste détaillée, consultez le tableau [Banques de données prises en charge](copy-activity-overview.md#supported-data-stores-and-formats).
* **Sécurité et conformité**  : les données sont transférées via HTTPS ou ExpressRoute. La présence globale du service garantit que vos données ne quittent jamais les limites géographiques.
* **Hautes performances**  : la vitesse de chargement des données dans Data Lake Storage Gen1 peut atteindre 1 Gbit/s. Pour en savoir plus, voir [Performances de l’activité de copie](copy-activity-performance.md).

Cet article explique comment utiliser l’outil de copie de données Data Factory pour _charger les données d’Amazon S3 dans Data Lake Storage Gen1_. Vous pouvez procéder de même pour copier des données à partir d’autres types de banques de données.

> [!NOTE]
> Pour en savoir plus, consultez [Copier des données vers ou depuis Azure Data Lake Storage Gen1 à l’aide d’Azure Data Factory](connector-azure-data-lake-store.md).

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure : Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* Compte Data Lake Storage Gen1 : si vous n’avez pas de compte Data Lake Storage Gen1, consultez les instructions décrites dans la section [Créer un compte Data Lake Storage Gen1](../data-lake-store/data-lake-store-get-started-portal.md#create-a-data-lake-storage-gen1-account).
* Amazon S3 : Cet article explique comment copier des données à partir d’Amazon S3. Vous pouvez utiliser d’autres magasins de données en procédant de la même façon.

## <a name="create-a-data-factory"></a>Créer une fabrique de données

1. Dans le menu de gauche, sélectionnez **Créer une ressource** > **Analytics** > **Data Factory**  :
   
   ![Sélection Data Factory dans le volet « Nouveau »](./media/quickstart-create-data-factory-portal/new-azure-data-factory-menu.png)

2. Sur la page **Nouvelle fabrique de données** , fournissez les valeurs des champs qui apparaissent dans l’image suivante : 
      
   ![Page Nouvelle fabrique de données](./media/load-data-into-azure-data-lake-store//new-azure-data-factory.png)
 
    * **Name**  : Entrez un nom global unique pour votre fabrique de données Azure. Si l’erreur « Le nom de fabrique de données \"LoadADLSG1Demo\" n’est pas disponible » apparaît, saisissez un autre nom pour la fabrique de données. Par exemple, utilisez le nom _**votrenom**_**ADFTutorialDataFactory**. Essayez à nouveau de créer la fabrique de données. Pour savoir comment nommer les artefacts Data Factory, voir [Data Factory - Règles d’affectation des noms](naming-rules.md).
    * **Abonnement** : Sélectionnez l’abonnement Azure dans lequel créer la fabrique de données. 
    * **Groupe de ressources** : Sélectionnez un groupe de ressources existant dans la liste déroulante ou sélectionnez l’option **Créer** et entrez le nom d’un groupe de ressources. Pour plus d’informations sur les groupes de ressources, consultez [Utilisation des groupes de ressources pour gérer vos ressources Azure](../azure-resource-manager/management/overview.md).  
    * **Version**  : Sélectionnez **V2**.
    * **Emplacement** : Sélectionnez l’emplacement de la fabrique de données. Seuls les emplacements pris en charge sont affichés dans la liste déroulante. Les magasins de données utilisés par la fabrique de données peuvent se trouver dans d’autres emplacements et régions. Ces magasins de données incluent Azure Data Lake Storage Gen1, Stockage Azure, Azure SQL Database, etc.

3. Sélectionnez **Create** (Créer).
4. Une fois la création terminée, accédez à votre fabrique de données. La page d’accueil **Data Factory** devrait s’afficher comme dans l’image suivante : 
   
   ![Page d’accueil Data Factory](./media/load-data-into-azure-data-lake-store/data-factory-home-page.png)

   Sélectionnez la vignette **Créer et surveiller** pour lancer l’application d’intégration de données dans un onglet séparé.

## <a name="load-data-into-data-lake-storage-gen1"></a>Charger des données dans Data Lake Storage Gen1

1. Dans la page **Prise en main** , sélectionnez la vignette **Copier les données** pour démarrer l’outil Copier les données : 

   ![Vignette de l’outil Copier les données](./media/load-data-into-azure-data-lake-store/copy-data-tool-tile.png)
2. Dans la page **Propriétés** , spécifiez **CopyFromAmazonS3ToADLS** dans le champ **Nom de tâche** , puis cliquez sur **Suivant** :

    ![Page Propriétés](./media/load-data-into-azure-data-lake-store/copy-data-tool-properties-page.png)
3. Dans la page **Banques de données sources** , cliquez sur **+ Créer une connexion** :

    ![Page Magasin de données sources](./media/load-data-into-azure-data-lake-store/source-data-store-page.png)
    
    Sélectionnez **Amazon S3** , puis sélectionnez **Continuer**.
    
    ![Page Banque de données sources s3](./media/load-data-into-azure-data-lake-store/source-data-store-page-s3.png)
    
4. Sur la page **Spécifier la connexion Amazon S3** , procédez comme suit : 
   1. Spécifiez la valeur du champ **ID de clé d’accès**.
   2. Spécifiez la valeur **Clé d’accès secrète**.
   3. Sélectionnez **Terminer**.
   
      ![Capture d’écran montrant le volet Nouveau service lié dans lequel vous pouvez entrer des valeurs.](./media/load-data-into-azure-data-lake-store/specify-amazon-s3-account.png)
   
   4. Vous voyez une nouvelle connexion. Sélectionnez **Suivant**.
   
   ![Capture d’écran montrant votre nouvelle connexion.](./media/load-data-into-azure-data-lake-store/specify-amazon-s3-account-created.png)
   
5. Sur la page de **sélection du fichier ou dossier d’entrée** , accédez au dossier et au fichier sur lesquels effectuer la copie. Sélectionnez le dossier ou le fichier ; cliquez sur **Choisir** , puis sur **Suivant** :

    ![Choisir le fichier ou le dossier d’entrée](./media/load-data-into-azure-data-lake-store/choose-input-folder.png)

6. Choisissez le comportement de copie en sélectionnant les options de **copie récursive des fichiers** et de **copie binaire** (copie des fichiers en l’état). Sélectionnez **Suivant** :

    ![Capture d’écran représentant la page Choisir le fichier ou le dossier d’entrée sur laquelle vous pouvez sélectionner Copier le fichier de façon récursive et Copie binaire.](./media/load-data-into-azure-data-lake-store/specify-binary-copy.png)
    
7. Sur la page **Banque de données de destination** , cliquez sur **+ Créer une connexion** , puis sélectionnez **Azure Data Lake Storage Gen1** et sélectionnez **Continuer** :

    ![Page Magasin de données de destination](./media/load-data-into-azure-data-lake-store/destination-data-storage-page.png)

8. Sur la page **New Linked Service (Azure Data Lake Storage Gen1)** (Nouveau service lié (Azure Data Lake Storage Gen1)), effectuez les étapes suivantes : 

   1. Sélectionnez votre compte Data Lake Storage Gen1 pour **Nom du compte Data Lake Store**.
   2. Spécifiez le **Locataire** , puis cliquez sur Terminer.
   3. Sélectionnez **Suivant**.
   
   > [!IMPORTANT]
   > Dans cette procédure pas à pas, vous utilisez une identité managée pour les ressources Azure, afin d’authentifier votre compte Data Lake Storage Gen1. Veillez à accorder à la fonctionnalité MSI les autorisations appropriées dans Data Lake Storage Gen1, en suivant [ces instructions](connector-azure-data-lake-store.md#managed-identity).
   
   ![Spécifier le compte Data Lake Storage Gen1](./media/load-data-into-azure-data-lake-store/specify-adls.png)
9. Dans la page de **sélection du fichier ou dossier de sortie** , saisissez **copyfroms3** dans le champ du nom du dossier de sortie, puis sélectionnez **Suivant** : 

    ![Capture d’écran représentant le chemin d’accès au dossier que vous entrez.](./media/load-data-into-azure-data-lake-store/specify-adls-path.png)

10. Sur la page **Paramètres** , cliquez sur **Suivant** :

    ![Page Paramètres](./media/load-data-into-azure-data-lake-store/copy-settings.png)
11. Dans la page **Résumé** , vérifiez les paramètres, puis cliquez sur **Suivant** :

    ![Page de résumé](./media/load-data-into-azure-data-lake-store/copy-summary.png)
12. Dans la page **Déploiement** , sélectionnez **Surveiller** pour surveiller le pipeline (tâche) :

    ![Page Déploiement](./media/load-data-into-azure-data-lake-store/deployment-page.png)
13. Notez que l’onglet **Surveiller** sur la gauche est sélectionné automatiquement. La colonne **Actions** comprend les liens permettant d’afficher les détails de l’exécution de l’activité et de réexécuter le pipeline :

    ![Surveiller des exécutions de pipelines](./media/load-data-into-azure-data-lake-store/monitor-pipeline-runs.png)
14. Pour afficher les exécutions d’activités associées à l’exécution du pipeline, sélectionnez le lien **Afficher les exécutions d’activités** dans la colonne **Actions**. Il n’y a qu’une seule activité (activité de copie) dans le pipeline ; vous ne voyez donc qu’une seule entrée. Pour revenir à l’affichage des exécutions du pipeline, sélectionnez le lien **Pipelines** affiché en haut de la fenêtre. Sélectionnez **Actualiser** pour actualiser la liste. 

    ![Surveiller des exécutions d’activités](./media/load-data-into-azure-data-lake-store/monitor-activity-runs.png)

15. Pour surveiller l’exécution de chaque activité de copie, cliquez sur le lien **Détails** sous **Actions** dans la page de surveillance des activités. Vous pouvez suivre les informations détaillées comme le volume de données copiées à partir de la source dans le récepteur, le débit des données, les étapes d’exécution avec une durée correspondante et les configurations utilisées :

    ![Détails du suivi de l'exécution des activités](./media/load-data-into-azure-data-lake-store/monitor-activity-run-details.png)

16. Vérifiez que les données sont copiées dans votre compte Data Lake Store Gen1 : 

    ![Vérifier la sortie Data Lake Storage Gen1](./media/load-data-into-azure-data-lake-store/adls-copy-result.png)

## <a name="next-steps"></a>Étapes suivantes

Lisez l’article suivant pour en savoir plus sur la prise en charge de Data Lake Storage Gen1 : 

> [!div class="nextstepaction"]
>[Connecteur Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md)

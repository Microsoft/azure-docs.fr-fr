---
title: Charger des données dans Azure Data Lake Storage Gen2
description: Utiliser Azure Data Factory pour copier des données dans Azure Data Lake Storage Gen2
services: data-factory
documentationcenter: ''
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 11/09/2020
ms.openlocfilehash: ca9ca495f2b3449b5aeb933bbd8d312fc9341fd9
ms.sourcegitcommit: dc342bef86e822358efe2d363958f6075bcfc22a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94554115"
---
# <a name="load-data-into-azure-data-lake-storage-gen2-with-azure-data-factory"></a>Charger des données dans Azure Data Lake Storage Gen2 avec Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Azure Data Lake Storage Gen2 est un ensemble de fonctionnalités dédiées à l'analytique du Big Data et intégrées au service [Stockage Blob Azure](../storage/blobs/storage-blobs-introduction.md). Il vous permet d’interagir avec vos données selon les deux paradigmes que sont le système de fichiers et le stockage d’objets.

Azure Data Factory (ADF) est un service d’intégration de données informatiques complètement managé. Vous pouvez utiliser le service pour remplir le lac avec des données provenant d’un ensemble étendu de banques de données locales et cloud lors de la création de vos solutions d’analytique. Pour une liste détaillée des connecteurs pris en charge, consultez le tableau de [Banques de données prises en charge](copy-activity-overview.md#supported-data-stores-and-formats).

Azure Data Factory offre une solution de déplacement des données managées qui est évolutive. En raison de l’architecture évolutive d’Azure Data Factory elle peut ingérer des données à un débit élevé. Pour en savoir plus, voir [Performances de l’activité de copie](copy-activity-performance.md).

Cet article vous explique comment utiliser l’outil de copie de données de Data Factory pour charger des données depuis le _service ’Amazon Web Services S3_ dans _Azure Data Lake Store Gen2_. Vous pouvez procéder de même pour copier des données à partir d’autres types de banques de données.

>[!TIP]
>Pour copier des données à partir d’Azure Data Lake Storage Gen1 dans Gen2, reportez-vous à [cette procédure pas à pas spécifique](load-azure-data-lake-storage-gen2-from-gen1.md).

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure : Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* Compte de stockage Azure avec Data Lake Storage Gen2 activé : Si vous n’avez pas de compte de stockage, [créez-en un](https://ms.portal.azure.com/#create/Microsoft.StorageAccount-ARM).
* Compte AWS avec un compartiment S3 qui contient des données : Cet article explique comment copier des données à partir d’Amazon S3. Vous pouvez utiliser d’autres magasins de données en procédant de la même façon.

## <a name="create-a-data-factory"></a>Créer une fabrique de données

1. Dans le menu de gauche, sélectionnez **Créer une ressource** > **Intégration** > **Data Factory** :
   
   ![Sélection Data Factory dans le volet « Nouveau »](./media/doc-common-process/new-azure-data-factory-menu.png)

2. Sur la page **Nouvelle fabrique de données**, fournissez les valeurs des champs suivants :
 
    * **Name** : Entrez un nom global unique pour votre fabrique de données Azure. Si l’erreur « Le nom de fabrique de données *NomDeVotreFabriqueDeDonnées* n’est pas disponible » apparaît, saisissez un autre nom pour la fabrique de données. Par exemple, utilisez le nom _**votrenom**_**ADFTutorialDataFactory**. Essayez à nouveau de créer la fabrique de données. Pour savoir comment nommer les artefacts Data Factory, voir [Data Factory - Règles d’affectation des noms](naming-rules.md).
    * **Abonnement**: Sélectionnez l’abonnement Azure dans lequel créer la fabrique de données. 
    * **Groupe de ressources** : Sélectionnez un groupe de ressources existant dans la liste déroulante ou sélectionnez l’option **Créer** et entrez le nom d’un groupe de ressources. Pour plus d’informations sur les groupes de ressources, consultez [Utilisation des groupes de ressources pour gérer vos ressources Azure](../azure-resource-manager/management/overview.md).  
    * **Version** : Sélectionnez **V2**.
    * **Emplacement** : Sélectionnez l’emplacement de la fabrique de données. Seuls les emplacements pris en charge sont affichés dans la liste déroulante. Les magasins de données utilisés par la fabrique de données peuvent se trouver dans d’autres emplacements et régions. 

3. Sélectionnez **Create** (Créer).

4. Une fois la création terminée, accédez à votre fabrique de données. La page d’accueil **Data Factory** devrait s’afficher comme dans l’image suivante : 
   
   ![Page d’accueil Data Factory](./media/doc-common-process/data-factory-home-page.png)

   Sélectionnez la vignette **Créer et surveiller** pour lancer l’application d’intégration de données dans un onglet séparé.

## <a name="load-data-into-azure-data-lake-storage-gen2"></a>Charger des données dans Azure Data Lake Storage Gen2

1. Dans la page **Prise en main**, sélectionnez la vignette **Copier les données** pour démarrer l’outil Copier les données.

2. Dans la page **Propriétés**, spécifiez **CopyFromAmazonS3ToADLS** dans le champ **Nom de tâche**, puis cliquez sur **Suivant**.

    ![Page Propriétés](./media/load-azure-data-lake-storage-gen2/copy-data-tool-properties-page.png)
3. Dans la page **Banque de données sources**, cliquez sur **+ Créer une connexion**. Sélectionnez **Amazon S3** dans la galerie des connecteurs, puis sélectionnez **Continuer**.
    
    ![Page Banque de données sources s3](./media/load-azure-data-lake-storage-gen2/source-data-store-page-s3.png)
    
4. Dans la page **Nouveau service lié (Amazon S3)** , procédez comme suit :

   1. Spécifiez la valeur du champ **ID de clé d’accès**.
   2. Spécifiez la valeur **Clé d’accès secrète**.
   3. Cliquez sur **Tester la connexion** pour vérifier les paramètres, puis sélectionnez **Créer**.

      ![Spécification du compte Amazon S3](./media/load-azure-data-lake-storage-gen2/specify-amazon-s3-account.png)
   4. Vous voyez qu’une nouvelle connexion AmazonS3 est créée. Sélectionnez **Suivant**. 

5. Sur la page de **sélection du fichier ou dossier d’entrée**, accédez au dossier et au fichier sur lesquels effectuer la copie. Sélectionnez le dossier ou le fichier, puis sélectionnez **Choisir**.

    ![Choisir le fichier ou le dossier d’entrée](./media/load-azure-data-lake-storage-gen2/choose-input-folder.png)

6. Spécifiez le comportement de copie en cochant les options **Copier les fichiers de façon récursive** et **Copie binaire**. Sélectionnez **Suivant**.

    ![Capture d’écran affichant la zone Choisir le fichier ou le dossier d’entrée, dans laquelle vous pouvez sélectionner Copie binaire et Copier les fichiers de façon récursive.](./media/load-azure-data-lake-storage-gen2/specify-binary-copy.png)
    
7. Sur la page **Banque de données de destination**, cliquez sur **+ Créer une connexion**, puis sélectionnez **Azure Data Lake Storage Gen2** et sélectionnez **Continuer**.

    ![Page Magasin de données de destination](./media/load-azure-data-lake-storage-gen2/destination-data-storage-page.png)

8. Sur la page **Nouveau service lié (Azure Data Lake Storage Gen2)** , procédez comme suit :

   1. Sélectionnez votre compte activé pour Data Lake Storage Gen2 dans la liste déroulante « Nom du compte de stockage ».
   2. Sélectionnez **Créer** pour créer la connexion. Sélectionnez ensuite **Suivant**.   

        ![Indiquer un compte Azure Data Lake Storage Gen2](./media/load-azure-data-lake-storage-gen2/specify-azure-data-lake-storage.png)

9. Dans la page de **sélection du fichier ou dossier de sortie**, saisissez **copyfroms3** dans le champ du nom du dossier de sortie, puis sélectionnez **Suivant**. ADF crée le système de fichiers ADLS Gen2 et les sous-dossiers correspondants pendant la copie s’ils n’existent pas.

    ![Capture d’écran représentant le chemin d’accès au dossier que vous entrez.](./media/load-azure-data-lake-storage-gen2/specify-adls-path.png)

10. Dans la page **Paramètres**, sélectionnez **Suivant** pour utiliser les paramètres par défaut.

    ![Page Paramètres](./media/load-azure-data-lake-storage-gen2/copy-settings.png)

11. Dans la page **Résumé**, vérifiez les paramètres, puis cliquez sur **Suivant**.

    ![Page de résumé](./media/load-azure-data-lake-storage-gen2/copy-summary.png)

12. Sur la page **Déploiement**, sélectionnez **Surveiller** pour surveiller le pipeline (tâche). 
 
13. Lorsque l’exécution du pipeline se termine avec succès, vous voyez une exécution de pipeline qui est déclenchée par un déclencheur manuel. Vous pouvez utiliser les liens sous la colonne **NOM DU PIPELINE** pour voir les détails de l’activité et réexécuter le pipeline.

    ![Surveiller des exécutions de pipelines](./media/load-azure-data-lake-storage-gen2/monitor-pipeline-runs.png)

14. Pour voir les exécutions d’activités associées à l’exécution du pipeline, sélectionnez le lien **CopyFromAmazonS3ToADLS** sous la colonne NOM DU PIPELINE. Pour plus de détails sur l’opération de copie, sélectionnez le lien **Détails** (icône en forme de lunettes) dans la colonne NOM DE L’ACTIVITÉ. Vous pouvez suivre les informations détaillées comme le volume de données copiées à partir de la source dans le récepteur, le débit des données, les étapes d’exécution avec une durée correspondante et la configuration utilisée.
 
    ![Surveiller des exécutions d’activités](./media/load-azure-data-lake-storage-gen2/monitor-activity-runs.png)
    
    ![Détails du suivi de l'exécution des activités](./media/load-azure-data-lake-storage-gen2/monitor-activity-run-details.png)

15. Sélectionnez Actualiser pour actualiser l’affichage. Sélectionnez **Toutes les exécutions de pipelines** en haut pour revenir à la vue Exécutions de pipelines.

16. Vérifiez que les données sont copiées dans votre compte Azure Data Lake Store Gen2 :

## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble des activités de copie](copy-activity-overview.md)
* [Connecteur Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md)

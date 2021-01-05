---
title: Copier des données depuis Azure Data Lake Storage Gen1 vers Gen2
description: Utiliser Azure Data Factory pour copier des données depuis Azure Data Lake Storage Gen1 vers Gen2
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
ms.date: 05/13/2019
ms.openlocfilehash: 203b62bdeb2ef83d884188c5d1753b6a70050361
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93042725"
---
# <a name="copy-data-from-azure-data-lake-storage-gen1-to-gen2-with-azure-data-factory"></a>Copier les données depuis Azure Data Lake Storage Gen1 vers Gen2 avec Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Azure Data Lake Storage Gen2 est un ensemble de fonctionnalités dédiées à l’analytique de Big Data qui est intégrée au service [Stockage Blob Azure](../storage/blobs/storage-blobs-introduction.md). Il vous permet d’interagir avec vos données selon les deux paradigmes que sont le système de fichiers et le stockage d’objets.

Si vous utilisez actuellement Azure Data Lake Storage Gen1, vous pouvez essayer Azure Data Lake Storage Gen2 en copiant les données de Data Lake Storage Gen1 vers Gen2 à l’aide du service Azure Data Factory.

Azure Data Factory est un service informatique d’intégration de données informatique intégralement managé. Vous pouvez utiliser le service pour remplir le lac avec des données provenant d’un ensemble étendu de banques de données locales et cloud quand vous créez vos solutions d’analytique. Pour obtenir une liste des connecteurs pris en charge, consultez le tableau des [banques de données prises en charge](copy-activity-overview.md#supported-data-stores-and-formats).

Azure Data Factory offre une solution de déplacement des données managées qui est évolutive. En raison de l’architecture scale-out de Data Factory ,elle peut ingérer des données à un débit élevé. Pour plus d’informations, consultez [Performances de l’activité de copie](copy-activity-performance.md).

Cet article vous explique comment utiliser l’outil de copie de données de Data Factory pour copier des données depuis Azure Data Lake Storage Gen1 vers Azure Data Lake Storage Gen2. Vous pouvez procéder de même pour copier des données à partir d’autres types de banques de données.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* Compte Azure Data Lake Storage Gen1 contenant des données.
* Compte de stockage Azure avec Data Lake Storage Gen2 activé. Si vous n’avez pas de compte de stockage, [créez-en un](https://ms.portal.azure.com/#create/Microsoft.StorageAccount-ARM).

## <a name="create-a-data-factory"></a>Créer une fabrique de données

1. Dans le menu de gauche, sélectionnez **Créer une ressource** > **Données + Analytique** > **Data Factory**.
   
   ![Sélection Data Factory dans le volet Nouveau](./media/quickstart-create-data-factory-portal/new-azure-data-factory-menu.png)

2. Dans la page **Nouvelle fabrique de données** , entrez les valeurs appropriées dans les champs de l’image suivante : 
      
   ![Page Nouvelle fabrique de données](./media/load-azure-data-lake-storage-gen2-from-gen1/new-azure-data-factory.png)
 
    * **Name**  : Entrez un nom global unique pour votre fabrique de données Azure. Si l’erreur « Le nom de fabrique de données \"LoadADLSDemo\" n’est pas disponible » apparaît, saisissez un autre nom pour la fabrique de données. Par exemple, utilisez le nom _**votrenom**_ **ADFTutorialDataFactory**. Recréez la fabrique de données. Pour savoir comment nommer les artefacts Data Factory, voir [Data Factory - Règles d’affectation des noms](naming-rules.md).
    * **Abonnement** : Sélectionnez l’abonnement Azure dans lequel créer la fabrique de données. 
    * **Groupe de ressources** : Sélectionnez un groupe de ressources existant dans la liste déroulante. Vous pouvez également sélectionner l’option **Nouveau** et entrez le nom d’un groupe de ressources. Pour plus d’informations sur les groupes de ressources, consultez [Utilisation des groupes de ressources pour gérer vos ressources Azure](../azure-resource-manager/management/overview.md). 
    * **Version**  : Sélectionnez **V2**.
    * **Emplacement** : Sélectionnez l’emplacement de la fabrique de données. Seuls les emplacements pris en charge sont affichés dans la liste déroulante. Les banques de données utilisées par la fabrique de données peuvent se trouver dans d’autres emplacements et régions. 

3. Sélectionnez **Create** (Créer).
4. Une fois la création terminée, accédez à votre fabrique de données. La page d’accueil **Data Factory** devrait s’afficher comme dans l’image suivante : 
   
   ![Page d’accueil Data Factory](./media/load-azure-data-lake-storage-gen2-from-gen1/data-factory-home-page.png)

5. Sélectionnez la vignette **Créer et surveiller** pour lancer l’application d’intégration de données dans un onglet séparé.

## <a name="load-data-into-azure-data-lake-storage-gen2"></a>Charger des données dans Azure Data Lake Storage Gen2

1. Dans la page **Prise en main** , sélectionnez la vignette **Copier les données** pour démarrer l’outil de copie des données. 

   ![Vignette de l’outil de copie des données](./media/load-azure-data-lake-storage-gen2-from-gen1/copy-data-tool-tile.png)
2. Dans la page **Propriétés** , spécifiez **CopyFromADLSGen1ToGen2** dans le champ **Nom de tâche**. Sélectionnez **Suivant**.

    ![Page Propriétés](./media/load-azure-data-lake-storage-gen2-from-gen1/copy-data-tool-properties-page.png)
3. Dans la page **Banque de données sources** , sélectionnez **+ Créer une connexion**.

    ![Page Magasin de données sources](./media/load-azure-data-lake-storage-gen2-from-gen1/source-data-store-page.png)
    
4. Sélectionnez **Azure Data Lake Storage Gen1** dans la galerie des connecteurs, puis sélectionnez **Continuer**.
    
    ![Page de la banque de données sources d’Azure Data Lake Storage Gen1](./media/load-azure-data-lake-storage-gen2-from-gen1/source-data-store-page-adls-gen1.png)
    
5. Dans la page **Spécifier la connexion Azure Data Lake Storage Gen1** , effectuez ces étapes :

   a. Sélectionnez votre Data Lake Storage Gen1 pour le nom du compte, puis spécifiez ou confirmez le **locataire**.
  
   b. Sélectionnez **Tester la connexion** pour vérifier les paramètres. Sélectionnez **Terminer**.
  
   c. Vous voyez qu’une nouvelle connexion a été créée. Sélectionnez **Suivant**.
   
   > [!IMPORTANT]
   > Dans cette procédure pas à pas, vous utilisez une identité managée pour des ressources Azure afin d’authentifier votre solution Azure Data Lake Storage Gen1. Pour accorder à l’identité managée les autorisations appropriées dans Azure Data Lake Storage Gen1, suivez [ces instructions](connector-azure-data-lake-store.md#managed-identity).
   
   ![Indiquer un compte Azure Data Lake Storage Gen1](./media/load-azure-data-lake-storage-gen2-from-gen1/specify-adls-gen1-account.png)
      
6. Dans la page de **sélection du fichier ou dossier d’entrée** , accédez au fichier ou au dossier où vous voulez copier les données. Sélectionnez le dossier ou le fichier, puis sélectionnez **Choisir**.

    ![Choisir le fichier ou le dossier d’entrée](./media/load-azure-data-lake-storage-gen2-from-gen1/choose-input-folder.png)

7. Spécifiez le comportement de copie en sélectionnant les options **Copier les fichiers de façon récursive** et **Copie binaire**. Sélectionnez **Suivant**.

    ![Capture d’écran représentant la page Choose the input file or folder (Choisir le fichier ou le dossier d’entrée) dans laquelle vous pouvez sélectionner Copy file recursively (Copie récursive de fichier) et Binary Copy (Copie binaire).](./media/load-azure-data-lake-storage-gen2-from-gen1/specify-binary-copy.png)
    
8. Dans la page **Banque de données de destination** , sélectionnez **+ Créer une connexion** > **Azure Data Lake Storage Gen2** > **Continuer**.

    ![Page Magasin de données de destination](./media/load-azure-data-lake-storage-gen2-from-gen1/destination-data-storage-page.png)

9. Dans la page **Spécifier la connexion Azure Data Lake Storage Gen2** , effectuez ces étapes :

   a. Sélectionnez votre compte activé pour Data Lake Storage Gen2 dans la liste déroulante **Nom du compte de stockage**.
   
   b. Sélectionnez **Terminer** pour créer la connexion. Sélectionnez ensuite **Suivant**.
   
   ![Indiquer un compte Azure Data Lake Storage Gen2](./media/load-azure-data-lake-storage-gen2-from-gen1/specify-adls-gen2-account.png)

10. Dans la page de **sélection du fichier ou dossier de sortie** , entrez **copyfromadlsgen1** comme nom du dossier de sortie, puis sélectionnez **Suivant**. Data Factory crée le système de fichiers et les sous-dossiers Azure Data Lake Storage Gen2 correspondants pendant la copie s’il ne les trouve pas.

    ![Capture d’écran représentant le chemin d’accès au dossier que vous entrez.](./media/load-azure-data-lake-storage-gen2-from-gen1/specify-adls-gen2-path.png)

11. Dans la page **Paramètres** , sélectionnez **Suivant** pour utiliser les paramètres par défaut.

12. Dans la page **Résumé** , vérifiez les paramètres, puis cliquez sur **Suivant**.

    ![Page de résumé](./media/load-azure-data-lake-storage-gen2-from-gen1/copy-summary.png)
13. Dans la page **Déploiement** , sélectionnez **Surveiller** pour superviser le pipeline.

    ![Page Déploiement](./media/load-azure-data-lake-storage-gen2-from-gen1/deployment-page.png)
14. Notez que l’onglet **Surveiller** sur la gauche est sélectionné automatiquement. La colonne **Actions** comprend les liens permettant d’afficher les détails de l’exécution d’activité et de réexécuter le pipeline.

    ![Surveiller des exécutions de pipelines](./media/load-azure-data-lake-storage-gen2-from-gen1/monitor-pipeline-runs.png)

15. Pour afficher les exécutions d’activités associées à l’exécution du pipeline, sélectionnez le lien **Afficher les exécutions d’activités** dans la colonne **Actions**. Il n’y a qu’une seule activité (activité de copie) dans le pipeline ; vous ne voyez donc qu’une seule entrée. Pour revenir à l’affichage des exécutions du pipeline, sélectionnez le lien **Pipelines** affiché en haut de la fenêtre. Sélectionnez **Actualiser** pour actualiser la liste. 

    ![Surveiller des exécutions d’activités](./media/load-azure-data-lake-storage-gen2-from-gen1/monitor-activity-runs.png)

16. Pour surveiller les détails d’exécution de chaque activité de copie, sélectionnez le lien **Détails** (image de lunettes) sous **Actions** dans la vue de surveillance des activités. Vous pouvez suivre les informations détaillées comme le volume de données copiées à partir de la source dans le récepteur, le débit des données, les étapes d’exécution avec une durée correspondante et les configurations utilisées.

    ![Détails du suivi de l'exécution des activités](./media/load-azure-data-lake-storage-gen2-from-gen1/monitor-activity-run-details.png)

17. Vérifiez que les données ont bien été copiées dans votre compte Azure Data Lake Storage Gen2.

## <a name="best-practices"></a>Meilleures pratiques

Pour évaluer la mise à niveau d’Azure Data Lake Storage Gen1 vers Azure Data Lake Storage Gen2 en général, consultez [Mettre à niveau vos solutions d’analytique de Big Data d’Azure Data Lake Storage Gen1 vers Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-migrate-gen1-to-gen2.md). Les sections suivantes présentent les bonnes pratiques d’utilisation du service Data Factory pour effectuer une mise à niveau des données de Data Lake Storage Gen1 vers Data Lake Storage Gen2.

### <a name="data-partition-for-historical-data-copy"></a>Partitionner les données pour la copie de données historiques

- Si vos données dans Data Lake Storage Gen1 représentent une taille totale inférieure à 30 To et sont réparties dans moins d’un million de fichiers, vous pouvez copier toutes les données en exécutant une seule activité de copie.
- Si vous avez une plus grande quantité de données à copier, ou si vous souhaitez pouvoir gérer la migration des données par lots et traiter chaque lot dans un laps de temps défini, il est préférable de partitionner les données. Autre avantage, le partitionnement réduit le risque de survenue de problèmes inattendus.

Utilisez une preuve de concept pour vérifier la solution de bout en bout et tester le débit de copie dans votre environnement. Principales étapes de la preuve de concept : 

1. Créez un pipeline Data Factory avec une seule activité de copie pour copier plusieurs téraoctets de données de Data Lake Storage Gen1 vers Data Lake Storage Gen2. Ceci vous donne une base de référence de copie. Démarrez avec 128 [unités d’intégration de données](copy-activity-performance-features.md#data-integration-units). 
2. Sur la base du débit de copie obtenu à l’étape 1, calculez la durée estimée à prévoir pour migrer toutes les données. 
3. (Facultatif) Créez une table de contrôle et définissez le filtre de fichiers pour partitionner les fichiers à migrer. Il y a deux méthodes possibles pour partitionner les fichiers : 

    - Partitionner les fichiers par nom de dossier ou par nom de dossier avec un filtre de caractères génériques. Nous recommandons cette méthode.
    - Partitionner les fichiers en fonction de l’heure de dernière modification du fichier.

### <a name="network-bandwidth-and-storage-io"></a>Bande passante réseau et E/S de stockage 

Vous pouvez contrôler la concurrence entre les travaux de copie Data Factory qui lisent les données de Data Lake Storage Gen1 et ceux qui écrivent les données dans Data Lake Storage Gen2. Cela vous permet de gérer efficacement l’utilisation de ces E/S de stockage afin que le processus de migration n’impacte pas les travaux habituels sur Data Lake Storage Gen1.

### <a name="permissions"></a>Autorisations 

Dans Data Factory, le [connecteur Data Lake Storage Gen1](connector-azure-data-lake-store.md) prend en charge le principal de service et l’identité managée pour les authentifications de ressources Azure. Le [connecteur Data Lake Storage Gen2](connector-azure-data-lake-storage.md) prend en charge la clé de compte, le principal de service et l’identité managée pour les authentifications de ressources Azure. Pour permettre à Data Factory de parcourir et copier l’ensemble des fichiers ou des listes de contrôle d’accès dont vous avez besoin, accordez des autorisations suffisamment élevées au compte utilisé pour parcourir, lire ou écrire tous les fichiers et définir des listes de contrôle d’accès, le cas échéant. Attribuez au compte le rôle de super utilisateur ou de propriétaire durant la période de migration. 

### <a name="preserve-acls-from-data-lake-storage-gen1"></a>Conserver les listes de contrôle d’accès de Data Lake Storage Gen1

Si vous souhaitez répliquer les listes de contrôle d’accès ainsi que les fichiers de données quand vous migrez de Data Lake Storage Gen1 vers Data Lake Storage Gen2, consultez [Conserver les listes de contrôle d’accès à partir de Data Lake Storage Gen1](connector-azure-data-lake-storage.md#preserve-acls). 

### <a name="incremental-copy"></a>Copie incrémentielle 

Vous pouvez utiliser plusieurs approches pour charger uniquement les fichiers nouveaux ou mis à jour à partir de Data Lake Storage Gen1 :

- Chargez les fichiers nouveaux ou mis à jour en fonction du nom de fichier ou de dossier partitionné chronologiquement. Par exemple : /2019/05/13/*.
- Chargez les fichiers nouveaux ou mis à jour en fonction de la date de dernière modification.
- Identifiez les fichiers nouveaux ou mis à jour par une solution ou un outil tiers. Ensuite, passez le nom de fichier ou de dossier au pipeline Data Factory via le paramètre ou une table ou un fichier. 

La fréquence appropriée pour effectuer le chargement incrémentiel dépend du nombre total de fichiers dans Azure Data Lake Storage Gen1 et du volume de fichiers nouveaux ou mis à jour devant être chargés à chaque fois. 

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Vue d’ensemble de l’activité de copie](copy-activity-overview.md)
> [Connecteur Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md)
> [Connecteur Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md)
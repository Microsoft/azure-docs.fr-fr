---
title: Transformer des données avec Hive dans un réseau virtuel Azure en utilisant le portail Azure
description: Ce didacticiel fournit des instructions détaillées de transformation des données à l’aide de l’activité Hive dans Azure Data Factory.
services: data-factory
ms.service: data-factory
ms.workload: data-services
author: nabhishek
ms.author: abnarain
manager: anandsub
ms.topic: tutorial
ms.custom: seo-dt-2019
ms.date: 01/04/2018
ms.openlocfilehash: f6e1f80ebba0aa67cf2bab4a772ebc1765cd8298
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92634366"
---
# <a name="transform-data-in-azure-virtual-network-using-hive-activity-in-azure-data-factory-using-the-azure-portal"></a>Transformer des données dans un réseau virtuel Azure en utilisant l’activité Hive d’Azure Data Factory à partir du portail Azure

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Dans ce didacticiel, vous utilisez le portail Azure pour créer un pipeline Azure Data Factory qui transforme des données à l’aide d’une activité Hive sur un cluster HDInsight qui se trouve dans un réseau virtuel Azure (VNet). Dans ce tutoriel, vous allez effectuer les étapes suivantes :

> [!div class="checklist"]
> * Créer une fabrique de données. 
> * Créer un runtime d’intégration auto-hébergé
> * Créer les services liés Stockage Azure et Azure HDInsight
> * Créer un pipeline avec une activité Hive.
> * Déclencher une exécution du pipeline.
> * Surveiller l’exécution du pipeline. 
> * Vérifier la sortie

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

- **Compte Stockage Azure** . Vous créez un script Hive et le téléchargez vers le stockage Azure. La sortie du script Hive est stockée dans ce compte de stockage. Dans cet exemple, le cluster HDInsight utilise ce compte de stockage Azure en tant que stockage principal. 
- **Réseau virtuel Azure.** Si vous ne disposez pas d’un réseau virtuel Azure, créez-le en suivant [ces instructions](../virtual-network/quick-create-portal.md). Dans cet exemple, HDInsight est dans un réseau virtuel Azure. Voici un exemple de configuration du réseau virtuel Azure. 

    ![Création d’un réseau virtuel](media/tutorial-transform-data-using-hive-in-vnet-portal/create-virtual-network.png)
- **Cluster HDInsight.** Créez un cluster HDInsight et joignez-le au réseau virtuel que vous avez créé à l’étape précédente en suivant les instructions de cet article : [Étendre Azure HDInsight à l’aide d’un réseau virtuel Azure](../hdinsight/hdinsight-plan-virtual-network-deployment.md). Voici un exemple de configuration de HDInsight dans un réseau virtuel. 

    ![HDInsight dans un réseau virtuel](media/tutorial-transform-data-using-hive-in-vnet-portal/hdinsight-virtual-network-settings.png)
- **Azure PowerShell** . Suivez les instructions de la page [Installation et configuration d’Azure PowerShell](/powershell/azure/install-Az-ps).
- **Une machine virtuelle** . Créez une machine virtuelle Azure et joignez-la au réseau virtuel qui contient votre cluster HDInsight. Pour plus d’informations, reportez-vous à [Créer des machines virtuelles](../virtual-network/quick-create-portal.md#create-virtual-machines). 

### <a name="upload-hive-script-to-your-blob-storage-account"></a>Téléchargez le script Hive sur votre compte de stockage Blob

1. Créez un fichier SQL Hive nommé **hivescript.hql** avec le contenu suivant :

   ```sql
   DROP TABLE IF EXISTS HiveSampleOut; 
   CREATE EXTERNAL TABLE HiveSampleOut (clientid string, market string, devicemodel string, state string)
   ROW FORMAT DELIMITED FIELDS TERMINATED BY ' ' 
   STORED AS TEXTFILE LOCATION '${hiveconf:Output}';

   INSERT OVERWRITE TABLE HiveSampleOut
   Select 
       clientid,
       market,
       devicemodel,
       state
   FROM hivesampletable
   ```
2. Dans votre stockage Blob Azure, créez un conteneur nommé **adftutorial** s’il n’existe pas.
3. Créez un dossier nommé **hivescripts** .
4. Téléchargez le fichier **hivescript.hql** dans le sous-dossier **hivescripts** .

## <a name="create-a-data-factory"></a>Créer une fabrique de données

1. Lancez le navigateur web **Microsoft Edge** ou **Google Chrome** . L’interface utilisateur de Data Factory n’est actuellement prise en charge que par les navigateurs web Microsoft Edge et Google Chrome.
1. Connectez-vous au [portail Azure](https://portal.azure.com/).    
2. Cliquez sur **Nouveau** dans le menu de gauche, puis sur **Données + analyse** et sur **Data Factory** . 
   
   ![Nouveau -> DataFactory](./media/tutorial-transform-data-using-hive-in-vnet-portal/new-data-factory-menu.png)
3. Dans la page **Nouvelle fabrique de données** , entrez **ADFTutorialHiveFactory** comme **nom** . 
      
     ![Page Nouvelle fabrique de données](./media/tutorial-transform-data-using-hive-in-vnet-portal/new-azure-data-factory.png)
 
   Le nom de la fabrique de données Azure doit être un nom **global unique** . Si l’erreur suivante s’affiche, changez le nom de la fabrique de données (par exemple, votrenomMyAzureSsisDataFactory), puis tentez de la recréer. Consultez l’article [Data Factory - Règles d’affectation des noms](naming-rules.md) pour savoir comment nommer les artefacts Data Factory.
  
    *Le nom de fabrique de données « MyAzureSsisDataFactory » n’est pas disponible*
3. Sélectionnez l’ **abonnement** Azure dans lequel vous voulez créer la fabrique de données. 
4. Pour le **groupe de ressources** , effectuez l’une des opérations suivantes :
     
   - Sélectionnez **Utiliser l’existant** , puis sélectionnez un groupe de ressources existant dans la liste déroulante. 
   - Sélectionnez **Créer** , puis entrez le nom d’un groupe de ressources.   
         
     Pour plus d’informations sur les groupes de ressources, consultez [Utilisation des groupes de ressources pour gérer vos ressources Azure](../azure-resource-manager/management/overview.md).  
4. Sélectionnez **V2** pour la **version** .
5. Sélectionnez **l’emplacement** de la fabrique de données. Seuls les emplacements pris en charge pour la création de fabriques de données sont affichés dans la liste.
6. Sélectionnez **Épingler au tableau de bord** .     
7. Cliquez sur **Créer** .
8. Sur le tableau de bord, vous voyez la vignette suivante avec l’état : **Déploiement de Data Factory** . 

     ![mosaïque déploiement de fabrique de données](media/tutorial-transform-data-using-hive-in-vnet-portal/deploying-data-factory.png)
9. Une fois la création terminée, la page **Data Factory** s’affiche comme sur l’image.
   
    ![Page d’accueil Data Factory](./media/tutorial-transform-data-using-hive-in-vnet-portal/data-factory-home-page.png)
10. Cliquez sur **Créer et surveiller** pour lancer l’interface utilisateur (IU) de Data Factory dans un onglet séparé.
11. Dans la page **Prise en main** , basculez vers l’onglet **Modifier** dans le volet gauche comme illustré dans l’image suivante : 

    ![Onglet Modifier](./media/tutorial-transform-data-using-hive-in-vnet-portal/get-started-page.png)

## <a name="create-a-self-hosted-integration-runtime"></a>Créer un runtime d’intégration auto-hébergé
Le cluster Hadoop se trouvant dans un réseau virtuel, vous devez installer un runtime d’intégration auto-hébergé (IR) dans le même réseau virtuel. Dans cette section, vous créez une nouvelle machine virtuelle, vous la joignez au même réseau virtuel et vous installez IR auto-hébergé sur celle-ci. L’IR auto-hébergé permet au service Data Factory de distribuer le traitement des requêtes à un service de calcul tel que HDInsight dans un réseau virtuel. Il vous permet également de déplacer des données vers/depuis des magasins de données dans un réseau virtuel dans Azure. Vous utilisez un IR auto-hébergé lorsque le magasin de données ou le calcul se trouve également dans un environnement local. 

1. Dans l’interface utilisateur d’Azure Data Factory, cliquez sur **Connexions** au bas de la fenêtre, basculez vers l’onglet **Runtimes d’intégration** , puis cliquez sur le bouton **+ Nouveau** dans la barre d’outils. 

   ![Menu Nouveau runtime d’intégration](./media/tutorial-transform-data-using-hive-in-vnet-portal/new-integration-runtime-menu.png)
2. Dans la fenêtre **Installation du runtime d’intégration** , sélectionnez l’option **Perform data movement and dispatch activities to external computes** (Effectuer des activités de déplacement des données et distribuer des activités à des services de calcul), puis cliquez sur **Suivant** . 

   ![Sélectionner l’option d’exécution d’activités de déplacement des données et de distribution des activités](./media/tutorial-transform-data-using-hive-in-vnet-portal/select-perform-data-movement-compute-option.png)
3. Sélectionnez **Réseau privé** , puis cliquez sur **Suivant** .
    
   ![Sélectionner un réseau privé](./media/tutorial-transform-data-using-hive-in-vnet-portal/select-private-network.png)
4. Entrez **MySelfHostedIR** pour le **Nom** , puis cliquez sur **Suivant** . 

   ![Spécifier le nom du runtime d’intégration](./media/tutorial-transform-data-using-hive-in-vnet-portal/integration-runtime-name.png) 
5. Copiez la **clé d’authentification** du runtime d’intégration en cliquant sur le bouton Copier, puis enregistrez-la. Gardez la fenêtre ouverte. Vous utilisez cette clé pour inscrire l’IR installé sur une machine virtuelle. 

   ![Copier la clé d’authentification](./media/tutorial-transform-data-using-hive-in-vnet-portal/copy-key.png)

### <a name="install-ir-on-a-virtual-machine"></a>Installer IR sur une machine virtuelle

1. Sur la machine virtuelle Azure, téléchargez le [runtime d’intégration autohébergé](https://www.microsoft.com/download/details.aspx?id=39717). Utilisez la **clé d’authentification** obtenue à l’étape précédente pour inscrire manuellement le runtime d’intégration auto-hébergé. 

    ![Inscrire le runtime d’intégration](media/tutorial-transform-data-using-hive-in-vnet-portal/register-integration-runtime.png)

2. Le message suivant s’affiche une fois que le runtime d’intégration auto-hébergé est bien inscrit. 
   
    ![Inscription réussie](media/tutorial-transform-data-using-hive-in-vnet-portal/registered-successfully.png)
3. Cliquez sur **Lancer Configuration Manager** . La page suivante apparaît une fois que le nœud est connecté au service cloud : 
   
    ![Le nœud est connecté](media/tutorial-transform-data-using-hive-in-vnet-portal/node-is-connected.png)

### <a name="self-hosted-ir-in-the-azure-data-factory-ui"></a>IR auto-hébergé dans l’interface utilisateur d’Azure Data Factory

1. Dans l’ **interface utilisateur d’Azure Data Factory** , vous devez voir le nom de la machine virtuelle auto-hébergée et son état.

   ![Nœuds auto-hébergés existants](./media/tutorial-transform-data-using-hive-in-vnet-portal/existing-self-hosted-nodes.png)
2. Cliquez sur **Terminer** pour fermer la fenêtre **Installation du runtime d’intégration** . L’IR auto-hébergé apparaît dans la liste des runtime d’intégration.

   ![IR auto-hébergé dans la liste](./media/tutorial-transform-data-using-hive-in-vnet-portal/self-hosted-ir-in-list.png)


## <a name="create-linked-services"></a>Créez des services liés

Cette section explique comment créer et déployer deux services liés :
- Un **service lié au stockage Azure** relie un compte de stockage Azure à la fabrique de données. Il s’agit du stockage principal utilisé par votre cluster HDInsight. Dans ce cas, vous utilisez ce compte de stockage Azure pour stocker le script Hive et la sortie du script.
- Un **service lié HDInsight** . Azure Data Factory soumet le script Hive à ce cluster HDInsight en vue de son exécution.

### <a name="create-azure-storage-linked-service"></a>Créer le service lié Stockage Azure

1. Basculez vers l’onglet **Services liés** , puis cliquez sur **Nouveau** .

   ![Bouton de nouveau service lié](./media/tutorial-transform-data-using-hive-in-vnet-portal/new-linked-service.png)    
2. Dans la fenêtre **Nouveau service lié** , sélectionnez **Stockage Blob Azure** , puis cliquez sur **Continuer** . 

   ![Sélectionner le stockage Blob Azure](./media/tutorial-transform-data-using-hive-in-vnet-portal/select-azure-storage.png)
3. Dans la fenêtre **Nouveau service lié** , procédez comme suit :

    1. Entrez **AzureStorageLinkedService** pour **Nom** .
    2. Sélectionnez **MySelfHostedIR** pour **se connecter via le runtime d’intégration** .
    3. Sélectionnez votre compte de stockage Azure pour le **Nom du compte de stockage** . 
    4. Pour tester la connexion au compte de stockage, cliquez sur **Tester la connexion** .
    5. Cliquez sur **Enregistrer** .
   
        ![Spécifier le compte de stockage Blob Azure](./media/tutorial-transform-data-using-hive-in-vnet-portal/specify-azure-storage-account.png)

### <a name="create-hdinsight-linked-service"></a>Créer un service lié Azure HDInsight

1. Cliquez de nouveau sur **Nouveau** pour créer un autre service lié. 
    
   ![Bouton de nouveau service lié](./media/tutorial-transform-data-using-hive-in-vnet-portal/new-linked-service.png)    
2. Basculez vers l’onglet **Calcul** , sélectionnez **Azure HDInsight** , puis cliquez sur **Continuer** .

    ![Sélectionner Azure HDInsight](./media/tutorial-transform-data-using-hive-in-vnet-portal/select-hdinsight.png)
3. Dans la fenêtre **Nouveau service lié** , procédez comme suit :

    1. Entrez **AzureHDInsightLinkedService** pour le **Nom** .
    2. Sélectionnez **Apportez votre propre HDInsight** . 
    3. Sélectionnez votre cluster HDInsight pour **Cluster Hdi** . 
    4. Entrez le **nom d’utilisateur** pour le cluster HDInsight.
    5. Entrez le **mot de passe** correspondant à l’utilisateur. 
    
        ![Paramètres Azure HDInsight](./media/tutorial-transform-data-using-hive-in-vnet-portal/specify-azure-hdinsight.png)

Cet article part du principe que vous avez accès au cluster via Internet. Par exemple, vous devez pouvoir vous connecter au cluster à l’adresse `https://clustername.azurehdinsight.net`. Cette adresse utilise la passerelle publique qui n’est pas disponible si vous avez utilisé des groupes de sécurité réseau ou des itinéraires définis par l’utilisateur pour restreindre l’accès à partir d’Internet. Pour que la fabrique de données puisse envoyer des travaux au cluster HDInsight sur le réseau virtuel Azure, vous devez configurer votre réseau virtuel Azure de telle sorte que l’URL puisse être résolue sur l’adresse IP privée de la passerelle utilisée par HDInsight.

1. À partir du portail Azure, ouvrez le réseau virtuel dans lequel HDInsight se trouve. Ouvrez l’interface réseau dont le nom commence par `nic-gateway-0`. Notez son adresse IP privée. Par exemple, 10.6.0.15. 
2. Si votre réseau virtuel Azure contient un serveur DNS, mettez à jour l’enregistrement DNS pour que l’URL du cluster HDInsight `https://<clustername>.azurehdinsight.net` puisse être résolu en `10.6.0.15`. Si vous n’avez pas de serveur DNS dans votre réseau virtuel Azure, vous pouvez utiliser une solution de contournement temporaire en modifiant le fichier hosts (C:\Windows\System32\drivers\etc) de toutes les machines virtuelles inscrites en tant que nœuds runtime d’intégration auto-hébergés en ajoutant une entrée semblable à la suivante : 

    `10.6.0.15 myHDIClusterName.azurehdinsight.net`

## <a name="create-a-pipeline"></a>Créer un pipeline 
Au cours de cette étape, vous allez créer un pipeline avec une activité Hive. L’activité exécute le script Hive en vue de renvoyer des données à partir d’un exemple de table et de les enregistrer sur un chemin d’accès que vous avez défini.

Notez les points suivants :

- **scriptPath** pointe vers le chemin d’accès au script Hive sur le compte de stockage Azure que vous avez utilisé pour MyStorageLinkedService. Le chemin d'accès respecte la casse.
- **Output** est un argument utilisé dans le script Hive. Utilisez le format `wasbs://<Container>@<StorageAccount>.blob.core.windows.net/outputfolder/` pour le faire pointer vers un dossier existant de votre stockage Azure. Le chemin d'accès respecte la casse. 

1. Dans l’interface utilisateur de Data Factory, cliquez sur **+ (plus)** dans le volet gauche, puis cliquez sur **Pipeline** . 

    ![Menu Nouveau pipeline](./media/tutorial-transform-data-using-hive-in-vnet-portal/new-pipeline-menu.png)
2. Dans la boîte à outils **Activités** , développez l’activité **HDInsight** et glissez-déposez l’activité **Hive** vers la surface du concepteur de pipeline. 

    ![Glisser-déposer l’activité Hive](./media/tutorial-transform-data-using-hive-in-vnet-portal/drag-drop-hive-activity.png)
3. Dans la fenêtre des propriétés, basculez vers l’onglet **Cluster HDI** , puis sélectionnez **AzureHDInsightLinkedService** pour **Service lié HDInsight** .

    ![Sélectionner un service lié HDInsight](./media/tutorial-transform-data-using-hive-in-vnet-portal/select-hdinsight-linked-service.png)
4. Basculez vers l’onglet **Scripts** , et procédez comme suit : 

    1. Sélectionnez **AzureStorageLinkedService** pour **Service lié de script** . 
    2. Pour **Chemin d’accès au fichier** , cliquez sur **Parcourir le stockage** . 
 
        ![Parcourir le stockage](./media/tutorial-transform-data-using-hive-in-vnet-portal/browse-storage-hive-script.png)
    3. Dans la fenêtre **Choisir un fichier ou dossier** , accédez au dossier **hivescripts** du conteneur **adftutorial** , sélectionnez **hivescript.hql** , puis cliquez sur **Terminer** .  
        
        ![Choisir un fichier ou un dossier](./media/tutorial-transform-data-using-hive-in-vnet-portal/choose-file-folder.png) 
    4. Vérifiez que **adftutorial/hivescripts/hivescript.hql** pour **Chemin d’accès au fichier** apparaît.

        ![Paramètres de script](./media/tutorial-transform-data-using-hive-in-vnet-portal/confirm-hive-script-settings.png)
    5. Dans l’ **onglet Script** , développez la section **Avancé** . 
    6. Cliquez sur **Auto-fill from script** (Remplissage automatique à partir du script) pour **Paramètres** . 
    7. Entrez la valeur du paramètre **Sortie** au format suivant : `wasbs://<Blob Container>@<StorageAccount>.blob.core.windows.net/outputfolder/`. Par exemple : `wasbs://adftutorial@mystorageaccount.blob.core.windows.net/outputfolder/`.
 
        ![Arguments de script](./media/tutorial-transform-data-using-hive-in-vnet-portal/script-arguments.png)
1. Pour publier les artefacts Data Factory, cliquez sur **Publier** .

    ![Capture d’écran montrant l’option de publication sur Data Factory.](./media/tutorial-transform-data-using-hive-in-vnet-portal/publish.png)

## <a name="trigger-a-pipeline-run"></a>Déclencher une exécution du pipeline

1. Validez tout d’abord le pipeline en cliquant sur le bouton **Valider** dans la barre d’outils. Fermez la fenêtre **Pipeline Validation Output** (Sortie de validation du pipeline) en cliquant sur la **flèche droite (>>)** . 

    ![Valider le pipeline](./media/tutorial-transform-data-using-hive-in-vnet-portal/validate-pipeline.png) 
2. Pour déclencher une exécution du pipeline, cliquez sur Déclencher dans la barre d’outils, puis sur Déclencher maintenant. 

    ![Déclencher maintenant](./media/tutorial-transform-data-using-hive-in-vnet-portal/trigger-now-menu.png)

## <a name="monitor-the-pipeline-run"></a>Surveiller l’exécution du pipeline.

1. Basculez vers l’onglet **Surveiller** sur la gauche. Vous voyez une exécution du pipeline dans la liste **Exécutions du pipeline** . 

    ![Surveiller des exécutions de pipelines](./media/tutorial-transform-data-using-hive-in-vnet-portal/monitor-pipeline-runs.png)
2. Pour actualiser la liste, cliquez sur **Actualiser** .
4. Pour afficher les exécutions d’activités associées aux exécutions du pipeline, cliquez sur **Afficher les exécutions d’activités** dans la colonne **Action** . Les autres liens d’action permettent d’arrêter/de réexécuter le pipeline. 

    ![Afficher les exécutions d’activités](./media/tutorial-transform-data-using-hive-in-vnet-portal/view-activity-runs-link.png)
5. Vous ne voyez qu’une seule exécution d’activité car il n’y a qu’une seule activité dans le pipeline de type **HDInsightHive** . Pour revenir à l’affichage précédent, cliquez sur le lien **Pipelines** en haut.

    ![Exécutions d’activités](./media/tutorial-transform-data-using-hive-in-vnet-portal/view-activity-runs.png)
6. Vérifiez qu’un fichier de sortie apparaît bien dans **outputfolder** dans le conteneur **adftutorial** . 

    ![Fichier de sortie](./media/tutorial-transform-data-using-hive-in-vnet-portal/output-file.png)

## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez effectué les étapes suivantes : 

> [!div class="checklist"]
> * Créer une fabrique de données. 
> * Créer un runtime d’intégration auto-hébergé
> * Créer les services liés Stockage Azure et Azure HDInsight
> * Créer un pipeline avec une activité Hive.
> * Déclencher une exécution du pipeline.
> * Surveiller l’exécution du pipeline. 
> * Vérifier la sortie

Passez au tutoriel suivant pour en savoir plus sur la transformation des données en utilisant un cluster Spark sur Azure :

> [!div class="nextstepaction"]
>[Création de branche et chaînage du flux de contrôle Data Factory](tutorial-control-flow-portal.md)
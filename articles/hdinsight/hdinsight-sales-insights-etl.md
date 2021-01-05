---
title: 'Tutoriel : Créer un pipeline ETL de bout en bout pour dériver des insights de ventes dans Azure HDInsight'
description: Découvrez comment créer et utiliser des pipelines ETL avec Azure HDInsight pour dériver des insights à partir de données de ventes à l’aide de clusters à la demande Spark et de Power BI.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: tutorial
ms.custom: hdinsightactive, devx-track-azurecli
ms.date: 04/15/2020
ms.openlocfilehash: 1031c34a44a253c7458ef78c6371b88014e882ed
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92746471"
---
# <a name="tutorial-create-an-end-to-end-data-pipeline-to-derive-sales-insights-in-azure-hdinsight"></a>Tutoriel : Créer un pipeline de données de bout en bout pour dériver des insights de ventes dans Azure HDInsight

Dans ce tutoriel, vous allez créer un pipeline de données de bout en bout qui effectue des opérations d’extraction, de transformation et de chargement. Le pipeline utilise des clusters [Apache Spark](./spark/apache-spark-overview.md) et Apache Hive exécutés sur Azure HDInsight pour interroger et manipuler les données. Vous utiliserez également des technologies comme Azure Data Lake Storage Gen2 pour le stockage des données et Power BI pour la visualisation.

Ce pipeline de données combine les données de différents magasins, supprime les données indésirables, ajoute de nouvelles données et recharge le tout dans votre stockage afin que vous puissiez visualiser les insights métier. Apprenez-en davantage sur les pipelines ETL dans [Extraire, transformer et charger (ETL) à l’échelle](./hadoop/apache-hadoop-etl-at-scale.md).

![Architecture ETL](./media/hdinsight-sales-insights-etl/architecture.png)

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

* Azure CLI : version 2.2.0 ou ultérieure. Voir [Installer l’interface de ligne de commande Azure](/cli/azure/install-azure-cli).

* jq, processeur JSON en ligne de commande.  Voir [https://stedolan.github.io/jq/](https://stedolan.github.io/jq/).

* Membre du [rôle intégré Azure Propriétaire](../role-based-access-control/built-in-roles.md).

* Si vous utilisez PowerShell pour déclencher le pipeline Data Factory, vous avez besoin du [module Az](/powershell/azure/).

* [Power BI Desktop](https://aka.ms/pbiSingleInstaller), pour visualiser les insights métier à la fin de ce tutoriel.

## <a name="create-resources"></a>Créer des ressources

### <a name="clone-the-repository-with-scripts-and-data"></a>Cloner le dépôt avec des scripts et des données

1. Connectez-vous à votre abonnement Azure. Si vous prévoyez d’utiliser Azure Cloud Shell, sélectionnez **Essayer** dans le coin supérieur droit du bloc de code. Sinon, entrez la commande suivante :

    ```azurecli-interactive
    az login

    # If you have multiple subscriptions, set the one to use
    # az account set --subscription "SUBSCRIPTIONID"
    ```

1. Assurez-vous que vous êtes membre du rôle Azure [Propriétaire](../role-based-access-control/built-in-roles.md). Remplacez `user@contoso.com` par votre compte, puis entrez la commande suivante :

    ```azurecli
    az role assignment list \
    --assignee "user@contoso.com" \
    --role "Owner"
    ```

    Si aucun enregistrement n’est retourné, vous n’êtes pas membre et ne pouvez pas suivre ce tutoriel.

1. Téléchargez les données et les scripts de ce tutoriel à partir du [dépôt ETL HDInsight d’insights de ventes](https://github.com/Azure-Samples/hdinsight-sales-insights-etl). Entrez la commande suivante :

    ```bash
    git clone https://github.com/Azure-Samples/hdinsight-sales-insights-etl.git
    cd hdinsight-sales-insights-etl
    ```

1. Vérifiez que les `salesdata scripts templates` ont été créés. Pour ce faire, utilisez la commande suivante :

   ```bash
   ls
   ```

### <a name="deploy-azure-resources-needed-for-the-pipeline"></a>Déployer les ressources Azure nécessaires pour le pipeline

1. Ajoutez des autorisations d’exécution pour tous les scripts en entrant la commande suivante :

    ```bash
    chmod +x scripts/*.sh
    ````

1. Définissez la variable pour le groupe de ressources. Remplacez `RESOURCE_GROUP_NAME` par le nom d’un groupe de ressources existant ou nouveau, puis entrez la commande ci-après :

    ```bash
    resourceGroup="RESOURCE_GROUP_NAME"
    ```

1. Exécutez le script. Remplacez `LOCATION` par la valeur voulue, puis entrez la commande ci-après :

    ```bash
    ./scripts/resources.sh $resourceGroup LOCATION
    ```

    Si vous ne savez pas quelle région spécifier, vous pouvez récupérer la liste des régions prises en charge pour votre abonnement avec la commande [az account list-locations](/cli/azure/account#az-account-list-locations).

    La commande déploie les ressources suivantes :

    * Compte Stockage Blob Azure. Ce compte contiendra les données de ventes de l’entreprise.
    * Compte Azure Data Lake Storage Gen2. Ce compte servira de compte de stockage pour les deux clusters HDInsight. Pour en savoir plus sur HDInsight et Data Lake Storage Gen2, consultez [Azure HDInsight integration with Data Lake Storage Gen2](https://azure.microsoft.com/blog/azure-hdinsight-integration-with-data-lake-storage-gen-2-preview-acl-and-security-update/).
    * Identité managée affectée par l’utilisateur. Ce compte donne aux clusters HDInsight l’accès au compte Data Lake Storage Gen2.
    * Un cluster Apache Spark. Ce cluster sera utilisé pour nettoyer et transformer les données brutes.
    * Cluster [Interactive Query](./interactive-query/apache-interactive-query-get-started.md) Apache Hive. Ce cluster permet d’interroger les données de ventes et de les visualiser avec Power BI.
    * Réseau virtuel Azure pris en charge par des règles de groupe de sécurité réseau. Ce réseau virtuel permet aux clusters de communiquer et sécurise leurs communications.

La création du cluster peut prendre environ 20 minutes.

Le mot de passe par défaut utilisé pour l’accès SSH aux clusters est `Thisisapassword1`. Si vous souhaitez changer le mot de passe, accédez au fichier `./templates/resourcesparameters_remainder.json`, puis modifiez le mot de passe pour les paramètres `sparksshPassword`, `sparkClusterLoginPassword`, `llapClusterLoginPassword` et `llapsshPassword`.

### <a name="verify-deployment-and-collect-resource-information"></a>Vérifier le déploiement et recueillir des informations sur les ressources

1. Si vous souhaitez vérifier l’état de votre déploiement, accédez au groupe de ressources dans le portail Azure. Sous **Paramètres** , sélectionnez **Déploiements** , puis votre déploiement. Ici, vous pouvez voir les ressources qui ont été déployées avec succès et celles qui sont toujours en cours de déploiement.

1. Pour voir les noms des clusters, entrez la commande suivante :

    ```bash
    sparkClusterName=$(cat resourcesoutputs_remainder.json | jq -r '.properties.outputs.sparkClusterName.value')
    llapClusterName=$(cat resourcesoutputs_remainder.json | jq -r '.properties.outputs.llapClusterName.value')

    echo "Spark Cluster" $sparkClusterName
    echo "LLAP cluster" $llapClusterName
    ```

1. Pour voir le compte de stockage Azure et la clé d’accès, entrez la commande suivante :

    ```azurecli
    blobStorageName=$(cat resourcesoutputs_storage.json | jq -r '.properties.outputs.blobStorageName.value')

    blobKey=$(az storage account keys list \
        --account-name $blobStorageName \
        --resource-group $resourceGroup \
        --query [0].value -o tsv)

    echo $blobStorageName
    echo $blobKey
    ```

1. Pour voir le compte Data Lake Storage Gen2 et la clé d’accès, entrez la commande suivante :

    ```azurecli
    ADLSGen2StorageName=$(cat resourcesoutputs_storage.json | jq -r '.properties.outputs.adlsGen2StorageName.value')

    adlsKey=$(az storage account keys list \
        --account-name $ADLSGen2StorageName \
        --resource-group $resourceGroup \
        --query [0].value -o tsv)

    echo $ADLSGen2StorageName
    echo $adlsKey
    ```

### <a name="create-a-data-factory"></a>Créer une fabrique de données

Azure Data Factory est un outil qui permet d’automatiser des pipelines Azure. Ce n’est pas l’unique moyen d’effectuer ces tâches, mais c’est un excellent moyen d’automatiser les processus. Pour plus d’informations sur Azure Data Factory, consultez la [documentation Azure Data Factory](https://azure.microsoft.com/services/data-factory/).

Cette fabrique de données aura un pipeline avec deux activités :

* La première activité copiera les données de Stockage Blob Azure dans le compte de stockage Data Lake Storage Gen2 afin d’imiter l’ingestion des données.
* La deuxième activité transformera les données dans le cluster Spark. Le script transforme les données en supprimant les colonnes indésirables. Il ajoute également une nouvelle colonne qui calcule le chiffre d’affaires généré par une transaction unique.

Pour configurer votre pipeline Azure Data Factory, exécutez la commande ci-dessous.  Vous devez toujours vous trouver au niveau du répertoire `hdinsight-sales-insights-etl`.

```bash
blobStorageName=$(cat resourcesoutputs_storage.json | jq -r '.properties.outputs.blobStorageName.value')
ADLSGen2StorageName=$(cat resourcesoutputs_storage.json | jq -r '.properties.outputs.adlsGen2StorageName.value')

./scripts/adf.sh $resourceGroup $ADLSGen2StorageName $blobStorageName
```

Ce script effectue les opérations suivantes :

1. Il crée un principal de service avec des autorisations `Storage Blob Data Contributor` sur le compte de stockage Data Lake Storage Gen2.
1. Il obtient un jeton d’authentification pour autoriser les requêtes POST à l’[API REST de système de fichiers Data Lake Storage Gen2](https://docs.microsoft.com/rest/api/storageservices/datalakestoragegen2/filesystem/create).
1. Il renseigne le nom réel de votre compte de stockage Data Lake Storage Gen2 dans les fichiers `sparktransform.py` et `query.hql`.
1. Il obtient les clés de stockage pour le compte Data Lake Storage Gen2 et le compte de stockage Blob.
1. Il crée un autre déploiement de ressources pour créer un pipeline Azure Data Factory, avec ses services liés et ses activités associés. Il transmet les clés de stockage en tant que paramètres au fichier de modèle pour que les services liés puissent accéder correctement aux comptes de stockage.

## <a name="run-the-data-pipeline"></a>Exécuter le pipeline de données

### <a name="trigger-the-data-factory-activities"></a>Déclencher les activités Data Factory

La première activité dans le pipeline Data Factory que vous avez créée déplace les données de Stockage Blob vers Data Lake Storage Gen2. La deuxième activité applique les transformations Spark aux données et enregistre les fichiers .csv transformés à un nouvel emplacement. L’exécution du pipeline entier peut prendre quelques minutes.

Pour récupérer le nom Data Factory, entrez la commande suivante :

```azurecli
cat resourcesoutputs_adf.json | jq -r '.properties.outputs.factoryName.value'
```

Pour déclencher le pipeline, vous pouvez effectuer l’une des deux actions suivantes :

* Déclenchez le pipeline Data Factory dans PowerShell. Remplacez `RESOURCEGROUP` et `DataFactoryName` par les valeurs appropriées, puis exécutez les commandes suivantes :

    ```powershell
    # If you have multiple subscriptions, set the one to use
    # Select-AzSubscription -SubscriptionId "<SUBSCRIPTIONID>"

    $resourceGroup="RESOURCEGROUP"
    $dataFactory="DataFactoryName"

    $pipeline =Invoke-AzDataFactoryV2Pipeline `
        -ResourceGroupName $resourceGroup `
        -DataFactory $dataFactory `
        -PipelineName "IngestAndTransform"

    Get-AzDataFactoryV2PipelineRun `
        -ResourceGroupName $resourceGroup  `
        -DataFactoryName $dataFactory `
        -PipelineRunId $pipeline
    ```

    Réexécutez `Get-AzDataFactoryV2PipelineRun` si nécessaire pour superviser la progression.

    ou

* Ouvrez la fabrique de données et sélectionnez **Créer et surveiller** . Déclenchez le pipeline `IngestAndTransform` à partir du portail. Pour plus d’informations sur le déclenchement de pipelines par le biais du portail, consultez [Créer des clusters Apache Hadoop à la demande dans HDInsight avec Azure Data Factory](hdinsight-hadoop-create-linux-clusters-adf.md#trigger-a-pipeline).

Pour vérifier que le pipeline a été exécuté, vous pouvez effectuer l’une des opérations suivantes :

* Accédez à la section **Superviser** de votre fabrique de données par le biais du portail.
* Dans l’Explorateur Stockage Azure, accédez à votre compte de stockage Data Lake Storage Gen 2. Accédez au système de fichiers `files`, puis au dossier `transformed` et vérifiez son contenu pour voir si le pipeline a réussi.

Pour découvrir d’autres façons de transformer des données avec HDInsight, consultez [cet article sur l’utilisation de Jupyter Notebook](/azure/hdinsight/spark/apache-spark-load-data-run-query).

### <a name="create-a-table-on-the-interactive-query-cluster-to-view-data-on-power-bi"></a>Créer une table sur le cluster Interactive Query pour afficher les données sur Power BI

1. Copiez le fichier `query.hql` sur le cluster LLAP en utilisant SCP. Entrez la commande :

    ```bash
    llapClusterName=$(cat resourcesoutputs_remainder.json | jq -r '.properties.outputs.llapClusterName.value')
    scp scripts/query.hql sshuser@$llapClusterName-ssh.azurehdinsight.net:/home/sshuser/
    ```

    Rappel : Le mot de passe par défaut est `Thisisapassword1`.

1. Utilisez SSH pour accéder au cluster LLAP. Entrez la commande :

    ```bash
    ssh sshuser@$llapClusterName-ssh.azurehdinsight.net
    ```

1. Utilisez la commande suivante pour exécuter le script :

    ```bash
    beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -f query.hql
    ```

    Ce script crée une table managée sur le cluster Interactive Query, à laquelle vous pouvez accéder à partir de Power BI.

### <a name="create-a-power-bi-dashboard-from-sales-data"></a>Créer un tableau de bord Power BI à partir de données de ventes

1. Ouvrez Power BI Desktop.

1. Dans le menu, accédez à **Obtenir des données** > **Plus...**  > **Azure** > **HDInsight Interactive Query** .

1. Sélectionnez **Connecter** .

1. Dans la boîte de dialogue **HDInsight Interactive Query**  :
    1. Dans la zone de texte **Serveur** , entrez le nom de votre cluster LLAP au format `https://LLAPCLUSTERNAME.azurehdinsight.net`.
    1. Dans la zone de texte **Base de données** , entrez `default`.
    1. Sélectionnez **OK** .

1. Dans la boîte de dialogue **AzureHive**  :
    1. Dans la zone de texte **Nom d’utilisateur** , entrez `admin`.
    1. Dans la zone de texte **Mot de passe** , entrez `Thisisapassword1`.
    1. Sélectionnez **Connecter** .

1. Dans **Navigateur** , sélectionnez `sales` et/ou `sales_raw` pour afficher un aperçu des données. Une fois les données chargées, vous pouvez faire des essais avec le tableau de bord que vous souhaitez créer. Pour bien démarrer avec les tableaux de bord Power BI, consultez les liens suivants :

* [Introduction aux tableaux de bord pour les concepteurs Power BI](https://docs.microsoft.com/power-bi/service-dashboards)
* [Tutoriel : Bien démarrer avec le service Power BI](https://docs.microsoft.com/power-bi/service-get-started)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne pensez pas continuer à utiliser cette application, supprimez toutes les ressources à l’aide de la commande suivante afin de ne pas être facturé.

1. Pour supprimer le groupe de ressources, entrez la commande ci-après :

    ```azurecli
    az group delete -n $resourceGroup
    ```

1. Pour supprimer le principal de service, entrez les commandes ci-après :

    ```azurecli
    servicePrincipal=$(cat serviceprincipal.json | jq -r '.name')
    az ad sp delete --id $servicePrincipal
    ```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Extraire, transformer et charger (ETL) à l’échelle](./hadoop/apache-hadoop-etl-at-scale.md)

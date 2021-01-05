---
title: 'Tutoriel : Créer un cluster Apache Kafka activé pour le proxy REST dans HDInsight avec Azure CLI'
description: Découvrez comment effectuer des opérations Apache Kafka à l’aide d’un proxy REST Kafka sur Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: hrasheed
ms.service: hdinsight
ms.topic: tutorial
ms.date: 02/27/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 68dddcbc5771ef1a8b5d6ea423674a1c6845a5e6
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92539477"
---
# <a name="tutorial-create-an-apache-kafka-rest-proxy-enabled-cluster-in-hdinsight-using-azure-cli"></a>Tutoriel : Créer un cluster Apache Kafka activé pour le proxy REST dans HDInsight avec Azure CLI

Dans ce tutoriel, vous découvrez comment créer un cluster [activé pour le proxy REST](./rest-proxy.md) Apache Kafka dans Azure HDInsight en utilisant l’interface CLI Azure. Azure HDInsight est un service d’analytique open source managé et complet pour les entreprises. Apache Kafka est une plateforme de streaming open source distribuée. Elle est souvent utilisée comme broker de messages, car elle propose des fonctionnalités similaires à une file d’attente de messages de publication/abonnement. Le proxy REST Kafka vous permet d’interagir avec votre cluster Kafka via une [API REST](/rest/api/hdinsight-kafka-rest-proxy/) sur HTTP. Azure CLI est l’interface de ligne de commande multiplateforme de Microsoft pour la gestion de ressources Azure.

L’API Apache Kafka n’est accessible qu’aux ressources se trouvant dans le même réseau virtuel. Vous pouvez accéder au cluster directement avec SSH. Pour connecter d’autres services, réseaux ou machines virtuelles à Apache Kafka, vous devez tout d’abord créer un réseau virtuel, puis créer les ressources au sein du réseau. Pour plus d’informations, consultez [Se connecter à Apache Kafka en utilisant un réseau virtuel](./apache-kafka-connect-vpn-gateway.md).

Dans ce tutoriel, vous allez voir comment :

> [!div class="checklist"]
> * Prérequis pour le proxy REST Kafka
> * Créer un cluster Apache Kafka en utilisant Azure CLI

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

* Une application inscrite auprès d’Azure AD. Les applications clientes que vous écrivez pour interagir avec le proxy REST Kafka utilisent l’identificateur et le secret de cette application pour s’authentifier auprès d’Azure. Pour plus d’informations, consultez [Inscrire une application auprès de la plateforme d’identités Microsoft](../../active-directory/develop/quickstart-register-app.md).

* Un groupe de sécurité Azure AD avec votre application inscrite en tant que membre. Ce groupe de sécurité sera utilisé pour contrôler les applications qui sont autorisées à interagir avec le proxy REST. Pour plus d’informations sur la création de groupes Azure AD, consultez [Créer un groupe de base et ajouter des membres avec Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

* Azure CLI. Vérifiez que vous disposez de la version 2.0.79 au minimum. Voir [Installer l’interface de ligne de commande Azure](/cli/azure/install-azure-cli).

## <a name="create-an-apache-kafka-cluster"></a>Créer un cluster Apache Kafka

1. Connectez-vous à votre abonnement Azure.

    ```azurecli
    az login
    
    # If you have multiple subscriptions, set the one to use
    # az account set --subscription "SUBSCRIPTIONID"
    ```

1. Définir des variables d’environnement L’utilisation des variables de cet article est basée sur Bash. De petites modifications seront nécessaires si vous utilisez d'autres environnements.

    |Variable | Description |
    |---|---|
    |resourceGroupName|Remplacez RESOURCEGROUPNAME par le nom de votre groupe de ressources.|
    |location|Remplacez LOCATION par une région où le cluster sera créé. Pour obtenir la liste des emplacements valides, utilisez la commande `az account list-locations`|
    |clusterName|Remplacez CLUSTERNAME par un nom global unique pour votre nouveau cluster.|
    |storageAccount|Remplacez STORAGEACCOUNTNAME par le nom de votre nouveau compte de stockage.|
    |httpPassword|Remplacez PASSWORD par le mot de passe de la connexion au cluster, **admin** .|
    |sshPassword|Remplacez PASSWORD par un mot de passe pour le nom d’utilisateur de Secure Shell, **sshuser** .|
    |securityGroupName|Remplacez SECURITYGROUPNAME par le nom du groupe de sécurité AAD client pour le proxy REST Kafka. La variable sera passée au paramètre `--kafka-client-group-name` pour `az-hdinsight-create`.|
    |securityGroupID|Remplacez SECURITYGROUPID par l’ID du groupe de sécurité AAD client pour le proxy REST Kafka. La variable sera passée au paramètre `--kafka-client-group-id` pour `az-hdinsight-create`.|
    |storageContainer|Conteneur de stockage que le cluster utilisera ; laissez-le tel quel pour ce tutoriel. Cette variable sera définie avec le nom du cluster.|
    |workernodeCount|Nombre de nœuds Worker dans le cluster ; laissez-le tel quel pour ce tutoriel. Pour garantir une haute disponibilité, Kafka nécessite un minimum de 3 nœuds Worker|
    |clusterType|Type de cluster HDInsight ; laissez-le tel quel pour ce tutoriel.|
    |clusterVersion|Version du cluster HDInsight ; laissez-la telle quelle pour ce tutoriel. Le proxy REST Kafka nécessite au minimum la version 4.0 du cluster.|
    |componentVersion|Version de Kafka ; laissez-la telle quelle pour ce tutoriel. Le proxy REST Kafka nécessite au minimum la version 2.1 du composant.|

    Mettez à jour les variables avec les valeurs souhaitées. Entrez ensuite les commandes CLI pour définir les variables d'environnement.

    ```azurecli
    export resourceGroupName=RESOURCEGROUPNAME
    export location=LOCATION
    export clusterName=CLUSTERNAME
    export storageAccount=STORAGEACCOUNTNAME
    export httpPassword='PASSWORD'
    export sshPassword='PASSWORD'
    export securityGroupName=SECURITYGROUPNAME
    export securityGroupID=SECURITYGROUPID

    export storageContainer=$(echo $clusterName | tr "[:upper:]" "[:lower:]")
    export workernodeCount=3
    export clusterType=kafka
    export clusterVersion=4.0
    export componentVersion=kafka=2.1
    ```

1. [Créez le groupe de ressources](/cli/azure/group#az-group-create) en entrant la commande ci-dessous :

    ```azurecli
     az group create \
        --location $location \
        --name $resourceGroupName
    ```

1. [Créez un compte de stockage Azure](/cli/azure/storage/account#az-storage-account-create) en saisissant la commande ci-dessous :

    ```azurecli
    # Note: kind BlobStorage is not available as the default storage account.
    az storage account create \
        --name $storageAccount \
        --resource-group $resourceGroupName \
        --https-only true \
        --kind StorageV2 \
        --location $location \
        --sku Standard_LRS
    ```

1. [Extrayez la clé primaire](/cli/azure/storage/account/keys#az-storage-account-keys-list) du compte de stockage Azure et stockez-la dans une variable en entrant la commande ci-dessous :

    ```azurecli
    export storageAccountKey=$(az storage account keys list \
        --account-name $storageAccount \
        --resource-group $resourceGroupName \
        --query [0].value -o tsv)
    ```

1. [Créez un conteneur de stockage Azure](/cli/azure/storage/container#az-storage-container-create) en saisissant la commande suivante :

    ```azurecli
    az storage container create \
        --name $storageContainer \
        --account-key $storageAccountKey \
        --account-name $storageAccount
    ```

1. [Créez le cluster HDInsight](/cli/azure/hdinsight#az-hdinsight-create). Avant d’entrer la commande, notez les paramètres suivants :

    1. Paramètres nécessaires pour les clusters Kafka :

        |Paramètre | Description|
        |---|---|
        |--type|La valeur doit être **Kafka** .|
        |--workernode-data-disks-per-node|Nombre de disques de données à utiliser par nœud Worker. HDInsight Kafka est pris en charge seulement avec des disques de données. Ce tutoriel utilise une valeur de **2** .|

    1. Paramètres nécessaires pour le proxy REST Kafka :

        |Paramètre | Description|
        |---|---|
        |--kafka-management-node-size|Taille du nœud. Ce tutoriel utilise la valeur **Standard_D4_v2** .|
        |--kafka-client-group-id|ID du groupe de sécurité AAD client pour le proxy REST Kafka. La valeur est passée depuis la variable **$securityGroupID** .|
        |--kafka-client-group-name|Nom du groupe de sécurité AAD client pour le proxy REST Kafka. La valeur est passée depuis la variable **$securityGroupName** .|
        |--version|La version du cluster HDInsight doit être au moins 4.0. La valeur est passée depuis la variable **$clusterVersion** .|
        |--component-version|La version de Kafka doit être au moins 2.1. La valeur est passée depuis la variable **$componentVersion** .|
    
        Si vous voulez créer le cluster sans proxy REST, éliminez `--kafka-management-node-size`, `--kafka-client-group-id` et `--kafka-client-group-name` de la commande `az hdinsight create`.

    1. Si vous disposez d’un réseau virtuel existant, ajoutez les paramètres `--vnet-name` et `--subnet` ainsi que leurs valeurs.

    Entrez la commande suivante pour créer le cluster :

    ```azurecli
    az hdinsight create \
        --name $clusterName \
        --resource-group $resourceGroupName \
        --type $clusterType \
        --component-version $componentVersion \
        --http-password $httpPassword \
        --http-user admin \
        --location $location \
        --ssh-password $sshPassword \
        --ssh-user sshuser \
        --storage-account $storageAccount \
        --storage-account-key $storageAccountKey \
        --storage-container $storageContainer \
        --version $clusterVersion \
        --workernode-count $workernodeCount \
        --workernode-data-disks-per-node 2 \
        --kafka-management-node-size "Standard_D4_v2" \
        --kafka-client-group-id $securityGroupID \
        --kafka-client-group-name "$securityGroupName"
    ```

    Le processus de création de cluster peut prendre plusieurs minutes. En règle générale, il dure environ 15 minutes.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Après avoir terminé ce tutoriel, vous souhaiterez peut-être supprimer le cluster. Avec HDInsight, vos données sont stockées dans le stockage Azure. Vous pouvez ainsi supprimer un cluster en toute sécurité s’il n’est pas en cours d’utilisation. Vous devez également payer pour un cluster HDInsight, même quand vous ne l’utilisez pas. Étant donné que les frais pour le cluster sont bien plus élevés que les frais de stockage, mieux vaut supprimer les clusters quand ils ne sont pas utilisés.

Entrez tout ou partie des commandes suivantes pour supprimer des ressources :

```azurecli
# Remove cluster
az hdinsight delete \
    --name $clusterName \
    --resource-group $resourceGroupName

# Remove storage container
az storage container delete \
    --account-name $storageAccount  \
    --name $storageContainer

# Remove storage account
az storage account delete \
    --name $storageAccount  \
    --resource-group $resourceGroupName

# Remove resource group
az group delete \
    --name $resourceGroupName
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez créé avec succès un cluster activé pour le proxy Apache Kafka REST dans Azure HDInsight avec Azure CLI, utilisez du code Python pour interagir avec le proxy REST :

> [!div class="nextstepaction"]
> [Créer un exemple d’application](./rest-proxy.md#client-application-sample)
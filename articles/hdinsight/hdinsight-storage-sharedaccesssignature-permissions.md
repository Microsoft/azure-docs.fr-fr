---
title: Restreindre l’accès à l’aide de signatures d’accès partagé - Azure HDInsight
description: Découvrez comment utiliser les signatures d’accès partagé pour limiter l’accès HDInsight aux données stockées dans Stockage Blob Azure.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020, devx-track-azurecli
ms.date: 04/28/2020
ms.openlocfilehash: 141db7feee987b7fffc578e19c60bd94ad56d239
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97511635"
---
# <a name="use-azure-blob-storage-shared-access-signatures-to-restrict-access-to-data-in-hdinsight"></a>Utiliser des signatures d’accès partagé Stockage Blob Azure pour restreindre l’accès aux données dans HDInsight

HDInsight dispose d’un accès total aux données dans les comptes Stockage Blob Azure associés au cluster. Vous pouvez utiliser des signatures d’accès partagé sur le conteneur d’objets blob pour restreindre l’accès aux données. Les signatures d’accès partagé (SAS) sont une fonctionnalité des comptes Stockage Blob Azure qui vous permet de limiter l’accès aux données. Par exemple, en fournissant un accès en lecture seule aux données.

> [!IMPORTANT]  
> Pour une solution utilisant Apache Ranger, envisagez d’utiliser les clusters HDInsight joints à un domaine. Pour plus d’informations, consultez le document [Configurer des clusters HDInsight joints à un domaine (préversion)](./domain-joined/apache-domain-joined-configure-using-azure-adds.md).

> [!WARNING]  
> HDInsight doit disposer d’un accès total au stockage par défaut pour le cluster.

## <a name="prerequisites"></a>Prérequis

* Un client SSH. Pour plus d’informations, consultez [Se connecter à HDInsight (Apache Hadoop) à l’aide de SSH](./hdinsight-hadoop-linux-use-ssh-unix.md).

* Un [conteneur de stockage](../storage/blobs/storage-quickstart-blobs-portal.md) existant.  

* Si vous utilisez PowerShell, vous aurez besoin du [module Az](/powershell/azure/).

* Si vous voulez utiliser Azure CLI et que vous ne l’avez pas encore installé, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

* Si vous utilisez [Python](https://www.python.org/downloads/), version 2.7 ou ultérieure.

* Si vous utilisez le C#, la version de Visual Studio doit être 2013 ou ultérieure.

* Le schéma d’URI pour votre compte de stockage. Ce schéma sera `wasb://` pour Stockage Blob Azure, `abfs://` pour Azure Data Lake Storage Gen2 ou `adl://` pour Azure Data Lake Storage Gen1. Si le transfert sécurisé est activé pour Stockage Blob Azure, l’URI sera `wasbs://`.

* Un cluster HDInsight existant auquel ajouter une signature d’accès partagé. Si ce n’est pas le cas, vous pouvez utiliser Azure PowerShell pour créer un cluster et ajouter une signature d’accès partagé lors de la création du cluster.

* Les fichiers d’exemple à partir de [https://github.com/Azure-Samples/hdinsight-dotnet-python-azure-storage-shared-access-signature](https://github.com/Azure-Samples/hdinsight-dotnet-python-azure-storage-shared-access-signature). Ce dépôt contient les éléments suivants :

  * Un projet Visual Studio permettant de créer un conteneur de stockage, une stratégie stockée et une SAP pour une utilisation avec HDInsight.
  * Un script Python permettant de créer un conteneur de stockage, une stratégie stockée et une SAP pour une utilisation avec HDInsight
  * Un script PowerShell permettant de créer un cluster HDInsight et de le configurer afin d’utiliser la SAP. Une version mise à jour est utilisée plus loin.
  * Un exemple de fichier : `hdinsight-dotnet-python-azure-storage-shared-access-signature-master\sampledata\sample.log`

## <a name="shared-access-signatures"></a>Les signatures d’accès partagé

Il existe deux types de signatures d’accès partagé :

* `Ad hoc`: l’heure de début, l’heure d’expiration et les autorisations associées à cette SAP sont spécifiées sur l’URI de SAP.

* `Stored access policy`: une stratégie d’accès stockée est définie sur un conteneur de ressources, tel qu’un conteneur d’objets blob. et permet de gérer les contraintes d’une ou de plusieurs signatures d’accès partagé. Lorsque vous associez une signature d'accès partagé à une stratégie d'accès stockée, la signature hérite des contraintes (heure de début, heure d'expiration et autorisations) définies pour la stratégie.

La différence entre les deux formes est importante pour un scénario clé : la révocation. Une signature d’accès partagé étant une URL, toute personne qui l’obtient peut l’utiliser. Peu importe qui l’a demandée initialement. Si une SAP est publiée publiquement, elle peut être utilisée par n’importe qui. Une clé d'accès partagé qui est distribuée est valide jusqu'à ce que l'un des quatre événements suivants ait lieu :

1. L'heure d'expiration spécifiée sur la signature d'accès partagé est atteinte.

2. Le délai d’expiration spécifié sur la stratégie d’accès stockée référencée par la SAP est atteint. Dans les scénarios suivants, l’heure d’expiration est atteinte :

    * L’intervalle de temps est écoulé.
    * La stratégie d’accès stockée est modifiée pour que son heure d’expiration soit passée. Modifier l’heure d’expiration est un moyen de révoquer la SAP.

3. La stratégie d'accès stockée référencée par la signature d'accès partagé est supprimée, ce qui est une autre manière de révoquer la signature d'accès partagé. Si vous recréez la stratégie d’accès stockée avec le même nom, tous les jetons SAS de la stratégie précédente sont valides (si l’heure d’expiration sur la signature d’accès partagé n’est pas passée). Si vous avez l’intention de révoquer la SAP, veillez à utiliser un nom différent si vous recréez la stratégie d’accès avec une heure d’expiration située dans le futur.

4. La clé de compte qui a été utilisée pour créer la signature d'accès partagé est régénérée. Cette régénération provoque l’échec de l’authentification de toutes les applications qui utilisent la clé précédente. Mettez à jour tous les composants vers la nouvelle clé.

> [!IMPORTANT]  
> L’URI d’une signature d’accès partagé est associé à la clé du compte utilisée pour créer la signature et à la stratégie d’accès stockée correspondante (le cas échéant). Si aucune stratégie d’accès stockée n’est spécifiée, la seule façon de révoquer une signature d’accès partagé consiste à modifier la clé du compte.

Nous vous recommandons de toujours utiliser des stratégies d’accès stockées. Lorsque vous utilisez des stratégies stockées, vous pouvez révoquer des signatures ou étendre la date d’expiration, selon vos besoins. Les étapes décrites dans ce document utilisent les stratégies d’accès stockées pour générer des SAP.

Pour plus d’informations sur les SAP, voir [Présentation du modèle SAP](../storage/common/storage-sas-overview.md)

## <a name="create-a-stored-policy-and-sas"></a>Créer une stratégie stockée et une SAP

Enregistrez le jeton SAP généré à la fin de chaque méthode. Le jeton ressemblera à la sortie suivante :

```output
?sv=2018-03-28&sr=c&si=myPolicyPS&sig=NAxefF%2BrR2ubjZtyUtuAvLQgt%2FJIN5aHJMj6OsDwyy4%3D
```

### <a name="using-powershell"></a>Utilisation de PowerShell

Remplacez `RESOURCEGROUP`, `STORAGEACCOUNT`, et `STORAGECONTAINER` par les valeurs appropriées pour votre conteneur de stockage existant. Accédez au répertoire `hdinsight-dotnet-python-azure-storage-shared-access-signature-master` ou révisez le paramètre `-File` pour qu’il contienne le chemin d’accès absolu pour `Set-AzStorageblobcontent`. Saisissez la commande PowerShell suivante :

```powershell
$resourceGroupName = "RESOURCEGROUP"
$storageAccountName = "STORAGEACCOUNT"
$containerName = "STORAGECONTAINER"
$policy = "myPolicyPS"

# Login to your Azure subscription
$sub = Get-AzSubscription -ErrorAction SilentlyContinue
if(-not($sub))
{
    Connect-AzAccount
}

# If you have multiple subscriptions, set the one to use
# Select-AzSubscription -SubscriptionId "<SUBSCRIPTIONID>"

# Get the access key for the Azure Storage account
$storageAccountKey = (Get-AzStorageAccountKey `
                                -ResourceGroupName $resourceGroupName `
                                -Name $storageAccountName)[0].Value

# Create an Azure Storage context
$storageContext = New-AzStorageContext `
                                -StorageAccountName $storageAccountName `
                                -StorageAccountKey $storageAccountKey

# Create a stored access policy for the Azure storage container
New-AzStorageContainerStoredAccessPolicy `
   -Container $containerName `
   -Policy $policy `
   -Permission "rl" `
   -ExpiryTime "12/31/2025 08:00:00" `
   -Context $storageContext

# Get the stored access policy or policies for the Azure storage container
Get-AzStorageContainerStoredAccessPolicy `
    -Container $containerName `
    -Context $storageContext

# Generates an SAS token for the Azure storage container
New-AzStorageContainerSASToken `
    -Name $containerName `
    -Policy $policy `
    -Context $storageContext

<# Removes a stored access policy from the Azure storage container
Remove-AzStorageContainerStoredAccessPolicy `
    -Container $containerName `
    -Policy $policy `
    -Context $storageContext
#>

# upload a file for a later example
Set-AzStorageblobcontent `
    -File "./sampledata/sample.log" `
    -Container $containerName `
    -Blob "samplePS.log" `
    -Context $storageContext
```

### <a name="using-azure-cli"></a>Utilisation de l’interface de ligne de commande Azure

L’utilisation de variables dans cette section est basée sur un environnement Windows. De petites modifications seront nécessaires si vous utilisez d’autres environnements.

1. Remplacez `STORAGEACCOUNT` et `STORAGECONTAINER` par les valeurs appropriées pour votre conteneur de stockage existant.

    ```azurecli
    # set variables
    set AZURE_STORAGE_ACCOUNT=STORAGEACCOUNT
    set AZURE_STORAGE_CONTAINER=STORAGECONTAINER

    #Login
    az login

    # If you have multiple subscriptions, set the one to use
    # az account set --subscription SUBSCRIPTION

    # Retrieve the primary key for the storage account
    az storage account keys list --account-name %AZURE_STORAGE_ACCOUNT% --query "[0].{PrimaryKey:value}" --output table
    ```

2. Définissez la clé primaire récupérée sur une variable pour une utilisation ultérieure. Remplacez `PRIMARYKEY` par la valeur récupérée à l’étape précédente, puis entrez la commande suivante :

    ```console
    #set variable for primary key
    set AZURE_STORAGE_KEY=PRIMARYKEY
    ```

3. Accédez au répertoire `hdinsight-dotnet-python-azure-storage-shared-access-signature-master` ou révisez le paramètre `--file` pour qu’il contienne le chemin d’accès absolu pour `az storage blob upload`. Exécutez les commandes restantes :

    ```azurecli
    # Create stored access policy on the containing object
    az storage container policy create --container-name %AZURE_STORAGE_CONTAINER% --name myPolicyCLI --account-key %AZURE_STORAGE_KEY% --account-name %AZURE_STORAGE_ACCOUNT% --expiry 2025-12-31 --permissions rl

    # List stored access policies on a containing object
    az storage container policy list --container-name %AZURE_STORAGE_CONTAINER% --account-key %AZURE_STORAGE_KEY% --account-name %AZURE_STORAGE_ACCOUNT%

    # Generate a shared access signature for the container
    az storage container generate-sas --name %AZURE_STORAGE_CONTAINER% --policy-name myPolicyCLI --account-key %AZURE_STORAGE_KEY% --account-name %AZURE_STORAGE_ACCOUNT%

    # Reversal
    # az storage container policy delete --container-name %AZURE_STORAGE_CONTAINER% --name myPolicyCLI --account-key %AZURE_STORAGE_KEY% --account-name %AZURE_STORAGE_ACCOUNT%

    # upload a file for a later example
    az storage blob upload --container-name %AZURE_STORAGE_CONTAINER% --account-key %AZURE_STORAGE_KEY% --account-name %AZURE_STORAGE_ACCOUNT% --name sampleCLI.log --file "./sampledata/sample.log"
    ```

### <a name="using-python"></a>Utilisation de Python

Ouvrez le fichier `SASToken.py` et remplacez `storage_account_name`, `storage_account_key` et `storage_container_name` par les valeurs appropriées pour votre conteneur de stockage existant, puis exécutez le script.

Vous devrez peut-être exécuter `pip install --upgrade azure-storage` si vous recevez le message d’erreur `ImportError: No module named azure.storage`.

### <a name="using-c"></a>Utilisation de C\#

1. Ouvrez la solution dans Visual Studio.

2. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **SASExample**, puis sélectionnez **Propriétés**.

3. Sélectionnez **Paramètres** et ajoutez des valeurs pour les entrées suivantes :

    |Élément |Description |
    |---|---|
    |StorageConnectionString|chaîne de connexion pour le compte de stockage pour lequel vous souhaitez créer une stratégie stockée et une SAP. Le format doit être `DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=mykey`, où `myaccount` est le nom de votre compte de stockage et `mykey` est la clé pour le compte de stockage.|
    |ContainerName|conteneur du compte de stockage auquel vous souhaitez restreindre l’accès.|
    |SASPolicyName|nom à utiliser pour la stratégie stockée à créer.|
    |FileToUpload|chemin d’un fichier qui est chargé dans le conteneur.|

4. Exécutez le projet. Enregistrez le jeton de stratégie SAP, le nom du compte de stockage et le nom du conteneur. Ces valeurs sont utilisées quand vous associez le compte de stockage à votre cluster HDInsight.

## <a name="use-the-sas-with-hdinsight"></a>Utilisation de la SAP avec HDInsight

Quand vous créez un cluster HDInsight, vous devez spécifier un compte de stockage principal. Vous pouvez également spécifier des comptes de stockage supplémentaires. Ces deux méthodes d’ajout de stockage nécessitent un accès complet aux comptes de stockage et aux conteneurs qui sont utilisés.

Utilisez une signature d’accès partagé pour limiter l’accès au conteneur. Ajoutez une entrée personnalisée à la configuration **core-site** pour le cluster. Vous pouvez ajouter l’entrée pendant la création du cluster à l’aide de PowerShell ou après la création du cluster à l’aide d’Ambari.

### <a name="create-a-cluster-that-uses-the-sas"></a>Créer un cluster qui utilise la signature d’accès partagé

Remplacez `CLUSTERNAME`, `RESOURCEGROUP`, `DEFAULTSTORAGEACCOUNT`, `STORAGECONTAINER`, `STORAGEACCOUNT` et `TOKEN` par les valeurs appropriées. Entrez les commandes PowerShell :

```powershell
$clusterName = 'CLUSTERNAME'
$resourceGroupName = 'RESOURCEGROUP'

# Replace with the Azure data center you want to the cluster to live in
$location = 'eastus'

# Replace with the name of the default storage account TO BE CREATED
$defaultStorageAccountName = 'DEFAULTSTORAGEACCOUNT'

# Replace with the name of the SAS container CREATED EARLIER
$SASContainerName = 'STORAGECONTAINER'

# Replace with the name of the SAS storage account CREATED EARLIER
$SASStorageAccountName = 'STORAGEACCOUNT'

# Replace with the SAS token generated earlier
$SASToken = 'TOKEN'

# Default cluster size (# of worker nodes), version, and type
$clusterSizeInNodes = "4"
$clusterVersion = "3.6"
$clusterType = "Hadoop"

# Login to your Azure subscription
$sub = Get-AzSubscription -ErrorAction SilentlyContinue
if(-not($sub))
{
    Connect-AzAccount
}

# If you have multiple subscriptions, set the one to use
# Select-AzSubscription -SubscriptionId "<SUBSCRIPTIONID>"

# Create an Azure Storage account and container
New-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $defaultStorageAccountName `
    -Location $location `
    -SkuName Standard_LRS `
    -Kind StorageV2 `
    -EnableHttpsTrafficOnly 1

$defaultStorageAccountKey = (Get-AzStorageAccountKey `
                                -ResourceGroupName $resourceGroupName `
                                -Name $defaultStorageAccountName)[0].Value

$defaultStorageContext = New-AzStorageContext `
                                -StorageAccountName $defaultStorageAccountName `
                                -StorageAccountKey $defaultStorageAccountKey

# Create a blob container. This holds the default data store for the cluster.
New-AzStorageContainer `
    -Name $clusterName `
    -Context $defaultStorageContext

# Cluster login is used to secure HTTPS services hosted on the cluster
$httpCredential = Get-Credential `
    -Message "Enter Cluster login credentials" `
    -UserName "admin"

# SSH user is used to remotely connect to the cluster using SSH clients
$sshCredential = Get-Credential `
    -Message "Enter SSH user credentials" `
    -UserName "sshuser"

# Create the configuration for the cluster
$config = New-AzHDInsightClusterConfig

$config = $config | Add-AzHDInsightConfigValue `
    -Spark2Defaults @{} `
    -Core @{"fs.azure.sas.$SASContainerName.$SASStorageAccountName.blob.core.windows.net"=$SASToken}

# Create the HDInsight cluster
New-AzHDInsightCluster `
    -Config $config `
    -ResourceGroupName $resourceGroupName `
    -ClusterName $clusterName `
    -Location $location `
    -ClusterSizeInNodes $clusterSizeInNodes `
    -ClusterType $clusterType `
    -OSType Linux `
    -Version $clusterVersion `
    -HttpCredential $httpCredential `
    -SshCredential $sshCredential `
    -DefaultStorageAccountName "$defaultStorageAccountName.blob.core.windows.net" `
    -DefaultStorageAccountKey $defaultStorageAccountKey `
    -DefaultStorageContainer $clusterName

<# REVERSAL
Remove-AzHDInsightCluster `
    -ResourceGroupName $resourceGroupName `
    -ClusterName $clusterName

Remove-AzStorageContainer `
    -Name $clusterName `
    -Context $defaultStorageContext

Remove-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $defaultStorageAccountName

Remove-AzResourceGroup `
    -Name $resourceGroupName
#>
```

> [!IMPORTANT]  
> Lorsque vous êtes invité à saisir le nom d’utilisateur et le mot de passe HTTP/s ou SSH, vous devez fournir un mot de passe qui répond aux critères suivants :
>
> * Il doit contenir au moins 10 caractères.
> * Il doit contenir au moins un chiffre.
> * Il doit contenir au moins un caractère non alphanumérique.
> * Il doit contenir au moins une lettre minuscule ou une lettre majuscule.

L’exécution de ce script prend un certain temps, environ 15 minutes en général. Lorsque le script se termine sans erreur, le cluster a été créé.

### <a name="use-the-sas-with-an-existing-cluster"></a>Utiliser la signature d’accès partagé avec un cluster existant

Si vous disposez d’un cluster existant, vous pouvez ajouter la SAP pour la configuration **core-site** en procédant comme suit :

1. Ouvrez l’interface utilisateur web Ambari de votre cluster. L’adresse de cette page est `https://YOURCLUSTERNAME.azurehdinsight.net`. À l’invite, authentifiez-vous auprès du cluster au moyen du nom et du mot de passe d’administrateur que vous avez utilisés lors de la création du cluster.

1. Accédez à **HDFS** > **Configurations** > **Avancé** > **Configuration core-site personnalisée**.

1. Développez la section **Configuration core-site personnalisée**, faites défiler jusqu’à la fin, puis sélectionnez **Ajouter une propriété…** . Utilisez les valeurs suivantes pour les champs **Clé** et **Valeur** :

    * **Clé** : `fs.azure.sas.CONTAINERNAME.STORAGEACCOUNTNAME.blob.core.windows.net`
    * **Valeur** : la SAS retournée par l’une des méthodes exécutées précédemment.

    Remplacez `CONTAINERNAME` par le nom du conteneur que vous avez utilisé avec l’application C# ou SAP. Remplacez `STORAGEACCOUNTNAME` par le nom du compte de stockage utilisé.

    Sélectionnez **Ajouter** pour enregistrer cette clé et cette valeur.

1. Cliquez sur le bouton **Enregistrer** pour enregistrer les modifications apportées à la configuration. Lorsque vous y êtes invité, ajoutez une description de la modification (« Ajout d’un accès de stockage SAP », par exemple), puis sélectionnez **Enregistrer**.

    Sélectionnez **OK** lorsque les modifications ont été effectuées.

   > [!IMPORTANT]  
   > Vous devez redémarrer plusieurs services pour que la modification prenne effet.

1. Une liste déroulante **Redémarrer** s’affiche. Sélectionnez **Redémarrer tous les éléments affectés** dans la liste déroulante, puis __Confirmer le redémarrage__.

    Répétez ce processus pour les entrées **MapReduce2** et **YARN**.

1. Une fois les services redémarrés, sélectionnez chacun d’entre eux et désactivez le mode maintenance à partir de la liste déroulante **Actions de service**.

## <a name="test-restricted-access"></a>Tester l’accès restreint

Procédez comme suit pour vérifier que vous pouvez uniquement lire et répertorier les éléments sur le compte de stockage SAP.

1. Connectez-vous au cluster. Remplacez `CLUSTERNAME` par le nom de votre cluster, puis entrez la commande suivante :

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

2. Pour répertorier le contenu du conteneur, utilisez la commande suivante à partir de l’invite :

    ```bash
    hdfs dfs -ls wasbs://SASCONTAINER@SASACCOUNTNAME.blob.core.windows.net/
    ```

    Remplacez `SASCONTAINER` avec le nom du conteneur créé pour le compte de stockage SAP. Remplacez `SASACCOUNTNAME` par le nom du compte de stockage utilisé pour la SAP.

    La liste inclut le fichier chargé lors de la création du conteneur et de la SAP.

3. Utilisez la commande suivante pour vérifier que vous pouvez lire le contenu du fichier. Remplacez `SASCONTAINER` et `SASACCOUNTNAME` comme dans l’étape précédente. Remplacez `sample.log` avec le nom du fichier affiché dans la commande précédente :

    ```bash
    hdfs dfs -text wasbs://SASCONTAINER@SASACCOUNTNAME.blob.core.windows.net/sample.log
    ```

    Cette commande répertorie le contenu du fichier.

4. Pour télécharger le fichier sur le système de fichiers local, utilisez la commande suivante :

    ```bash
    hdfs dfs -get wasbs://SASCONTAINER@SASACCOUNTNAME.blob.core.windows.net/sample.log testfile.txt
    ```

    Cette commande télécharge le fichier sur un fichier local nommé **testfile.txt**.

5. Utilisez la commande suivante pour charger le fichier local sur un nouveau fichier nommé **testupload.txt** sur le stockage SAP :

    ```bash
    hdfs dfs -put testfile.txt wasbs://SASCONTAINER@SASACCOUNTNAME.blob.core.windows.net/testupload.txt
    ```

    Vous recevez un message similaire au texte suivant :

    ```output
    put: java.io.IOException
    ```

    Cette erreur se produit, car l’emplacement de stockage est en lecture + liste uniquement. Pour placer les données sur le stockage par défaut pour le cluster, accessible en écriture, utilisez la commande suivante :

    ```bash
    hdfs dfs -put testfile.txt wasbs:///testupload.txt
    ```

    Cette fois, l’opération doit se terminer normalement.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris comment ajouter un stockage à accès limité à votre cluster HDInsight, découvrez d’autres façons de travailler avec des données sur votre cluster :

* [Utiliser SSH avec HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md)
* [Autoriser les utilisateurs à accéder à Apache Ambari Views](hdinsight-authorize-users-to-ambari.md)

---
title: PowerShell – HDInsight avec Azure Data Lake Storage Gen1 – Stockage complémentaire – Azure
description: Découvrez comment utiliser Azure PowerShell pour configurer un cluster HDInsight avec Azure Data Lake Storage Gen1 comme stockage supplémentaire.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: b7cb9d5c5c2ca850678d3f3194a9af8de526ada4
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92103388"
---
# <a name="use-azure-powershell-to-create-an-hdinsight-cluster-with-azure-data-lake-storage-gen1-as-additional-storage"></a>Utiliser Azure PowerShell pour créer un cluster HDInsight avec Azure Data Lake Storage Gen1 (comme stockage complémentaire)

> [!div class="op_single_selector"]
> * [Utilisation du portail](data-lake-store-hdinsight-hadoop-use-portal.md)
> * [Utilisation de PowerShell (pour le stockage par défaut)](data-lake-store-hdinsight-hadoop-use-powershell-for-default-storage.md)
> * [Utilisation de PowerShell (pour le stockage supplémentaire)](data-lake-store-hdinsight-hadoop-use-powershell.md)
> * [Utilisation du gestionnaire des ressources](data-lake-store-hdinsight-hadoop-use-resource-manager-template.md)
>
>

Découvrez comment utiliser Azure PowerShell pour configurer un cluster HDInsight avec Azure Data Lake Storage Gen1 **comme stockage supplémentaire**. Pour obtenir des instructions sur la création d’un cluster HDInsight avec Data Lake Storage Gen1 comme stockage par défaut, voir [Créer un cluster HDInsight avec Data Lake Storage Gen1 comme stockage par défaut](data-lake-store-hdinsight-hadoop-use-powershell-for-default-storage.md).

> [!NOTE]
> Si vous vous apprêtez à utiliser Data Lake Storage Gen1 en tant que stockage supplémentaire pour le cluster HDInsight, nous vous recommandons vivement de procéder ainsi lorsque vous créez le cluster comme décrit dans cet article. L’ajout de Data Lake Storage Gen1 en tant que stockage supplémentaire à un cluster HDInsight existant est un processus complexe sujet aux erreurs.
>

Pour les types de clusters pris en charge, Data Lake Storage Gen1 est utilisé comme compte de stockage par défaut ou supplémentaire. Lorsque Data Lake Storage Gen1 est utilisé comme espace de stockage supplémentaire, le compte de stockage par défaut pour les clusters est toujours le Stockage Blob Azure (WASB), et les fichiers associés au cluster (par exemple, les journaux d’activité, etc.) sont écrits dans le stockage par défaut, tandis que les données que vous souhaitez traiter peuvent être stockées dans Data Lake Storage Gen1. L’utilisation de Data Lake Storage Gen1 en tant que compte de stockage supplémentaire n’affecte pas les performances ni la capacité de lecture/écriture sur le stockage à partir du cluster.

## <a name="using-data-lake-storage-gen1-for-hdinsight-cluster-storage"></a>Utilisation de Data Lake Storage Gen1 pour le stockage de cluster HDInsight

Voici quelques considérations importantes pour l’utilisation de HDInsight avec Data Lake Storage Gen1 :

* L’option permettant de créer des clusters HDInsight avec accès au Data Lake Storage Gen1 comme stockage supplémentaire est disponible si vous utilisez HDInsight versions 3.2, 3.4, 3.5 et 3.6.

Pour configurer HDInsight afin qu’il fonctionne avec Data Lake Storage Gen1 à l’aide de PowerShell, la procédure est la suivante :

* Créer un compte Data Lake Storage Gen1 
* Configurer l’authentification pour définir un accès à Data Lake Storage Gen1 en fonction du rôle
* Créer un cluster HDInsight avec authentification à Data Lake Storage Gen1
* Lancer une tâche de test sur le cluster

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Avant de commencer ce didacticiel, vous devez disposer des éléments suivants :

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Azure PowerShell 1.0 ou version ultérieure**. Consultez [Installation et configuration d’Azure PowerShell](/powershell/azure/).
* **Kit de développement logiciel (SDK) Windows**. Vous pouvez l’installer à partir [d’ici](https://dev.windows.com/en-us/downloads). Il vous permet de créer un certificat de sécurité.
* **Principal du service Azure Active Directory**. Les étapes de ce didacticiel indiquent comment créer un principal du service dans Azure AD. Toutefois, vous devez être administrateur Azure AD pour pouvoir créer un principal du service. Si vous êtes administrateur Azure AD, vous pouvez ignorer ce prérequis et poursuivre le didacticiel.

    **Si vous n’êtes pas administrateur Azure AD**, vous ne pouvez pas effectuer les étapes nécessaires à la création d’un principal du service. Dans ce cas, votre administrateur Azure AD doit d’abord créer un principal de service. Vous pourrez ensuite créer un cluster HDInsight avec Data Lake Storage Gen1. En outre, le principal du service doit être créé à l’aide d’un certificat, comme décrit dans [Create a service principal with certificate](../active-directory/develop/howto-authenticate-service-principal-powershell.md#create-service-principal-with-certificate-from-certificate-authority) (Créer un principal du service avec certificat).

## <a name="create-a-data-lake-storage-gen1-account"></a>Créer un compte Data Lake Storage Gen1 
Pour créer un compte Data Lake Storage Gen1, procédez comme suit.

1. Sur votre bureau, ouvrez une nouvelle fenêtre Azure PowerShell et entrez l'extrait de code suivant. Lorsque vous êtes invité à vous connecter, vérifiez que vous vous connectez en tant qu’administrateur/propriétaire de l’abonnement :

    ```azurepowershell
    # Log in to your Azure account
    Connect-AzAccount

    # List all the subscriptions associated to your account
    Get-AzSubscription

    # Select a subscription
    Set-AzContext -SubscriptionId <subscription ID>

    # Register for Data Lake Storage Gen1
    Register-AzResourceProvider -ProviderNamespace "Microsoft.DataLakeStore"
    ```

   > [!NOTE]
   > Si vous recevez une erreur similaire à `Register-AzResourceProvider : InvalidResourceNamespace: The resource namespace 'Microsoft.DataLakeStore' is invalid` pendant l’inscription du fournisseur de ressources Data Lake Storage Gen1, il est possible que votre abonnement n’ait pas été approuvé pour Data Lake Storage Gen1. Veillez à activer votre abonnement Azure pour la version préliminaire publique de Data Lake Storage Gen1 en suivant ces [instructions](data-lake-store-get-started-portal.md).
   >
   >
2. Un compte de stockage Data Lake Storage Gen1 est associé à un groupe de ressources Azure. Commencez par créer un groupe de ressources Azure.

    ```azurepowershell
    $resourceGroupName = "<your new resource group name>"
    New-AzResourceGroup -Name $resourceGroupName -Location "East US 2"
    ```

    Vous devriez obtenir un résultat similaire à ceci :

    ```output
    ResourceGroupName : hdiadlgrp
    Location          : eastus2
    ProvisioningState : Succeeded
    Tags              :
    ResourceId        : /subscriptions/<subscription-id>/resourceGroups/hdiadlgrp
    ```

3. Créez un compte de stockage avec Data Lake Storage Gen1. Le nom de compte que vous spécifiez doit contenir uniquement des lettres minuscules et des chiffres.

    ```azurepowershell
    $dataLakeStorageGen1Name = "<your new storage account with Data Lake Storage Gen1 name>"
    New-AzDataLakeStoreAccount -ResourceGroupName $resourceGroupName -Name $dataLakeStorageGen1Name -Location "East US 2"
    ```

    Un résultat similaire à ce qui suit s’affiche normalement :

    ```output
    ...
    ProvisioningState           : Succeeded
    State                       : Active
    CreationTime                : 5/5/2017 10:53:56 PM
    EncryptionState             : Enabled
    ...
    LastModifiedTime            : 5/5/2017 10:53:56 PM
    Endpoint                    : hdiadlstore.azuredatalakestore.net
    DefaultGroup                :
    Id                          : /subscriptions/<subscription-id>/resourceGroups/hdiadlgrp/providers/Microsoft.DataLakeStore/accounts/hdiadlstore
    Name                        : hdiadlstore
    Type                        : Microsoft.DataLakeStore/accounts
    Location                    : East US 2
    Tags                        : {}
    ```

5. Téléchargez des exemples de données sur Data Lake Storage Gen1. Nous les utiliserons plus loin dans cet article pour vérifier que les données sont accessibles à partir d'un cluster HDInsight. Si vous recherchez des exemples de données à charger, vous pouvez récupérer le dossier **Données Ambulance** dans le [Référentiel Git Azure Data Lake](https://github.com/MicrosoftBigData/usql/tree/master/Examples/Samples/Data/AmbulanceData).

    ```azurepowershell
    $myrootdir = "/"
    Import-AzDataLakeStoreItem -AccountName $dataLakeStorageGen1Name -Path "C:\<path to data>\vehicle1_09142014.csv" -Destination $myrootdir\vehicle1_09142014.csv
    ```

## <a name="set-up-authentication-for-role-based-access-to-data-lake-storage-gen1"></a>Configurer l’authentification pour définir un accès à Data Lake Storage Gen1 en fonction du rôle

Chaque abonnement Azure est associé à un Azure Active Directory. Les utilisateurs et services qui accèdent aux ressources de l’abonnement avec le portail Azure ou l’API Azure Resource Manager doivent au préalable s’authentifier avec cette application Azure Active Directory. L’accès est accordé aux abonnements et services Azure en leur affectant le rôle approprié sur une ressource Azure.  Pour les services, un principal de service identifie le service dans Azure Active Directory (Azure AD). Cette section montre comment accorder l’accès à une ressource Azure (le compte de stockage Data Lake Storage Gen1 créé précédemment) à un service d’application, comme HDInsight, en créant un principal de service pour l’application et en lui attribuant des rôles avec Azure PowerShell.

Pour configurer l’authentification Active Directory pour Data Lake Storage Gen1, vous devez effectuer les tâches suivantes.

* Créer un certificat auto-signé
* Créer une application dans Azure Active Directory et un principal du service

### <a name="create-a-self-signed-certificate"></a>Créer un certificat auto-signé

Assurez-vous que le [SDK Windows](https://dev.windows.com/en-us/downloads) est installé avant de suivre la procédure décrite dans cette section. Vous devez également avoir créé un répertoire, comme **C:\mycertdir**, où sera créé le certificat.

1. Dans la fenêtre PowerShell, accédez à l’emplacement où vous avez installé le Windows SDK (en général, `C:\Program Files (x86)\Windows Kits\10\bin\x86`) et servez-vous de l’utilitaire [MakeCert][makecert] pour créer un certificat auto-signé et une clé privée. Utilisez les commandes suivantes.

    ```azurepowershell
    $certificateFileDir = "<my certificate directory>"
    cd $certificateFileDir

    makecert -sv mykey.pvk -n "cn=HDI-ADL-SP" CertFile.cer -r -len 2048
    ```

    Vous devrez entrer le mot de passe de la clé privée. Une fois la commande exécutée avec succès, **CertFile.cer** et **mykey.pvk** apparaissent dans le répertoire du certificat que vous avez spécifié.
2. Servez-vous de l’utilitaire [Pvk2Pfx][pvk2pfx] pour convertir en un fichier .pfx les fichiers .pvk et .cer créés par MakeCert. Exécutez la commande suivante :

    ```azurepowershell
    pvk2pfx -pvk mykey.pvk -spc CertFile.cer -pfx CertFile.pfx -po <password>
    ```

    Lorsque vous y êtes invité, entrez le mot de passe de la clé privée spécifié au préalable. La valeur que vous donnez au paramètre **-po** est le mot de passe associé au fichier .pfx. Une fois la commande exécutée avec succès, CertFile.pfx est visible dans le répertoire du certificat que vous avez spécifié.

### <a name="create-an-azure-active-directory-and-a-service-principal"></a>Créer une application Azure Active Directory et un principal du service

Cette section décrit les étapes pour créer un principal du service pour une application Azure Active Directory, attribuer un rôle au principal du service et vous authentifier en tant que principal du service en fournissant un certificat. Exécutez les commandes suivantes pour créer une application dans Azure Active Directory.

1. Collez les applets de commande suivantes dans la fenêtre de la console PowerShell. Vérifiez que la valeur que vous donnez à la propriété **-DisplayName** est unique. En outre, les valeurs pour **-HomePage** et **-IdentiferUris** sont des valeurs d’espace réservé et ne sont pas vérifiées.

    ```azurepowershell
    $certificateFilePath = "$certificateFileDir\CertFile.pfx"

    $password = Read-Host -Prompt "Enter the password" # This is the password you specified for the .pfx file

    $certificatePFX = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($certificateFilePath, $password)

    $rawCertificateData = $certificatePFX.GetRawCertData()

    $credential = [System.Convert]::ToBase64String($rawCertificateData)

    $application = New-AzADApplication `
        -DisplayName "HDIADL" `
        -HomePage "https://contoso.com" `
        -IdentifierUris "https://mycontoso.com" `
        -CertValue $credential  `
        -StartDate $certificatePFX.NotBefore  `
        -EndDate $certificatePFX.NotAfter

    $applicationId = $application.ApplicationId
    ```

2. Créez un principal du service avec l'ID d'application.

    ```azurepowershell
    $servicePrincipal = New-AzADServicePrincipal -ApplicationId $applicationId

     $objectId = $servicePrincipal.Id
    ```

3. Accordez au principal du service l’accès au dossier et au fichier Data Lake Storage Gen1 auxquels vous accéderez à partir du cluster HDInsight. L’extrait de code ci-dessous fournit l’accès à la racine du compte de stockage Data Lake Storage Gen1 (où vous avez copié l’exemple de fichier de données) et le fichier lui-même.

    ```azurepowershell
    Set-AzDataLakeStoreItemAclEntry -AccountName $dataLakeStorageGen1Name -Path / -AceType User -Id $objectId -Permissions All
    Set-AzDataLakeStoreItemAclEntry -AccountName $dataLakeStorageGen1Name -Path /vehicle1_09142014.csv -AceType User -Id $objectId -Permissions All
    ```

## <a name="create-an-hdinsight-linux-cluster-with-data-lake-storage-gen1-as-additional-storage"></a>Créer un cluster Linux HDInsight avec Data Lake Storage Gen1 comme stockage par supplémentaire

Dans cette section, nous créons un cluster HDInsight Hadoop Linux avec Data Lake Storage Gen1 comme stockage supplémentaire. Pour cette version, le cluster HDInsight et le compte de stockage avec Data Lake Storage Gen1 doivent se trouver au même emplacement.

1. Commencez par récupérer l’ID du client de l’abonnement. Vous en aurez besoin ultérieurement.

    ```azurepowershell
    $tenantID = (Get-AzContext).Tenant.TenantId
    ```

2. Pour cette version, pour un cluster Hadoop, Data Lake Storage Gen1 ne peut être utilisé que comme stockage supplémentaire pour le cluster. Le stockage par défaut sera toujours le Stockage Blob Azure (WASB). Par conséquent, nous allons tout d'abord créer le compte de stockage et les conteneurs de stockage requis par le cluster.

    ```azurepowershell
    # Create an Azure storage account
    $location = "East US 2"
    $storageAccountName = "<StorageAccountName>"   # Provide a Storage account name

    New-AzStorageAccount -ResourceGroupName $resourceGroupName -StorageAccountName $storageAccountName -Location $location -Type Standard_GRS

    # Create an Azure Blob Storage container
    $containerName = "<ContainerName>"              # Provide a container name
    $storageAccountKey = (Get-AzStorageAccountKey -Name $storageAccountName -ResourceGroupName $resourceGroupName)[0].Value
    $destContext = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
    New-AzStorageContainer -Name $containerName -Context $destContext
    ```

3. Créez le cluster HDInsight. Utilisez les applets de commande suivantes.

    ```azurepowershell
    # Set these variables
    $clusterName = $containerName                   # As a best practice, have the same name for the cluster and container
    $clusterNodes = <ClusterSizeInNodes>            # The number of nodes in the HDInsight cluster
    $httpCredentials = Get-Credential
    $sshCredentials = Get-Credential

    New-AzHDInsightCluster -ClusterName $clusterName -ResourceGroupName $resourceGroupName -HttpCredential $httpCredentials -Location $location -DefaultStorageAccountName "$storageAccountName.blob.core.windows.net" -DefaultStorageAccountKey $storageAccountKey -DefaultStorageContainer $containerName  -ClusterSizeInNodes $clusterNodes -ClusterType Hadoop -Version "3.4" -OSType Linux -SshCredential $sshCredentials -ObjectID $objectId -AadTenantId $tenantID -CertificateFilePath $certificateFilePath -CertificatePassword $password
    ```

    Après exécution de l'applet de commande, le résultat énumère les détails du cluster.


## <a name="run-test-jobs-on-the-hdinsight-cluster-to-use-the-data-lake-storage-gen1"></a>Exécuter des tâches de test sur le cluster HDInsight pour utiliser Data Lake Storage Gen1
Après avoir configuré un cluster HDInsight, vous pouvez exécuter des travaux test sur le cluster pour vérifier que le cluster HDInsight peut accéder à Data Lake Storage Gen1. Pour ce faire, nous allons exécuter un exemple de tâche Hive qui crée une table avec les exemples de données que vous avez chargés précédemment dans votre compte de stockage avec Data Lake Storage Gen1.

Dans cette section, vous allez utiliser SSH dans le cluster HDInsight Linux que vous avez créé et exécuter l’exemple de requête Hive.

* Si vous utilisez un client Windows pour utiliser SSH dans le cluster, consultez la rubrique [Utilisation de SSH avec Hadoop Linux sur HDInsight à partir de Windows](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).
* Si vous utilisez un client Linux pour utiliser SSH dans le cluster, consultez la rubrique [Utilisation de SSH avec Hadoop Linux sur HDInsight à partir de Linux](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

1. Une fois connecté, démarrez l'interface de ligne de commande Hive à l'aide de la commande suivante :

    ```azurepowershell
    hive
    ```

2. À l’aide de l’interface de ligne de commande, entrez les instructions suivantes pour créer une table nommée **vehicles** en utilisant les exemples de données dans Data Lake Storage Gen1 :

    ```azurepowershell
    DROP TABLE vehicles;
    CREATE EXTERNAL TABLE vehicles (str string) LOCATION 'adl://<mydatalakestoragegen1>.azuredatalakestore.net:443/';
    SELECT * FROM vehicles LIMIT 10;
    ```

    Le résultat doit ressembler à ce qui suit :

    ```output
    1,1,2014-09-14 00:00:03,46.81006,-92.08174,51,S,1
    1,2,2014-09-14 00:00:06,46.81006,-92.08174,13,NE,1
    1,3,2014-09-14 00:00:09,46.81006,-92.08174,48,NE,1
    1,4,2014-09-14 00:00:12,46.81006,-92.08174,30,W,1
    1,5,2014-09-14 00:00:15,46.81006,-92.08174,47,S,1
    1,6,2014-09-14 00:00:18,46.81006,-92.08174,9,S,1
    1,7,2014-09-14 00:00:21,46.81006,-92.08174,53,N,1
    1,8,2014-09-14 00:00:24,46.81006,-92.08174,63,SW,1
    1,9,2014-09-14 00:00:27,46.81006,-92.08174,4,NE,1
    1,10,2014-09-14 00:00:30,46.81006,-92.08174,31,N,1
    ```

## <a name="access-data-lake-storage-gen1-using-hdfs-commands"></a>Accéder à Data Lake Storage Gen1 avec les commandes HDFS
Une fois que vous avez configuré le cluster HDInsight pour qu’il utilise Data Lake Storage Gen1, vous pouvez utiliser les commandes de l’interpréteur de commandes HDFS pour accéder au magasin.

Dans cette section, vous allez utiliser SSH dans le cluster HDInsight Linux que vous avez créé et exécuter les commandes HDFS.

* Si vous utilisez un client Windows pour utiliser SSH dans le cluster, consultez la rubrique [Utilisation de SSH avec Hadoop Linux sur HDInsight à partir de Windows](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).
* Si vous utilisez un client Linux pour utiliser SSH dans le cluster, consultez la rubrique [Utilisation de SSH avec Hadoop Linux sur HDInsight à partir de Linux](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

Une fois connecté, utilisez la commande suivante du système de fichiers HDFS pour lister les fichiers du compte de stockage avec Data Lake Storage Gen1.

```azurepowershell
hdfs dfs -ls adl://<storage account with Data Lake Storage Gen1 name>.azuredatalakestore.net:443/
```

Le fichier que vous avez téléchargé dans Data Lake Storage Gen1 doit y figurer.

```output
15/09/17 21:41:15 INFO web.CaboWebHdfsFileSystem: Replacing original urlConnectionFactory with org.apache.hadoop.hdfs.web.URLConnectionFactory@21a728d6
Found 1 items
-rwxrwxrwx   0 NotSupportYet NotSupportYet     671388 2015-09-16 22:16 adl://mydatalakestoragegen1.azuredatalakestore.net:443/mynewfolder
```

Vous pouvez également utiliser la commande `hdfs dfs -put` pour charger des fichiers dans Data Lake Storage Gen1, puis utiliser `hdfs dfs -ls` pour vérifier si les fichiers ont été chargés avec succès.

## <a name="see-also"></a>Voir aussi
* [Utiliser Data Lake Storage Gen1 avec des clusters Azure HDInsight](../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen1.md)
* [Portail : créer un cluster HDInsight pour utiliser Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md)

[makecert]: /windows-hardware/drivers/devtest/makecert
[pvk2pfx]: /windows-hardware/drivers/devtest/pvk2pfx
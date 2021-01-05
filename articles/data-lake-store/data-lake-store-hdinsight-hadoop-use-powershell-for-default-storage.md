---
title: PowerShell – Cluster HDInsight avec Data Lake Storage Gen1 – Azure
description: Utiliser Azure PowerShell pour créer et utiliser les clusters Azure HDInsight avec Azure Data Lake Storage Gen1
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 5e899f28cf5b3c11ae5f935d7bc273c566214225
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97606774"
---
# <a name="create-hdinsight-clusters-with-azure-data-lake-storage-gen1-as-default-storage-by-using-powershell"></a>Créer des clusters HDInsight avec Data Lake Storage Gen1 comme stockage par défaut à l’aide de PowerShell

> [!div class="op_single_selector"]
> * [Utiliser le portail Azure](data-lake-store-hdinsight-hadoop-use-portal.md)
> * [Utiliser PowerShell (pour le stockage par défaut)](data-lake-store-hdinsight-hadoop-use-powershell-for-default-storage.md)
> * [Utiliser PowerShell (pour le stockage supplémentaire)](data-lake-store-hdinsight-hadoop-use-powershell.md)
> * [Utiliser Resource Manager](data-lake-store-hdinsight-hadoop-use-resource-manager-template.md)

Découvrez comment utiliser Azure PowerShell pour configurer des clusters Azure HDInsight avec Azure Data Lake Storage Gen1 comme stockage par défaut. Pour obtenir des instructions sur la création d’un cluster HDInsight avec Data Lake Storage Gen1 comme stockage supplémentaire, consultez [Créer un cluster HDInsight avec Data Lake Storage Gen1 en tant que stockage supplémentaire](data-lake-store-hdinsight-hadoop-use-powershell.md).

Voici quelques considérations importantes pour l’utilisation de HDInsight avec Data Lake Storage Gen1 :

* L’option permettant de créer des clusters HDInsight avec accès à Data Lake Storage Gen1 comme stockage par défaut est disponible si vous utilisez HDInsight versions 3.5 et 3.6.

* L’option permettant de créer des clusters HDInsight avec accès à Data Lake Storage Gen1 comme stockage par défaut n’est *pas disponible* pour les clusters HDInsight Premium.

Pour configurer HDInsight de façon à fonctionner avec Data Lake Storage Gen1 à l’aide de PowerShell, suivez les instructions des cinq prochaines sections.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Avant de commencer le didacticiel, veillez à ce que vos mots de passe répondent aux exigences suivantes :

* **Un abonnement Azure** : Consultez [Créez votre compte Azure gratuit](https://azure.microsoft.com/pricing/free-trial/).
* **Azure PowerShell 1.0 ou version supérieure** : Voir [Guide pratique pour installer et configurer PowerShell](/powershell/azure/).
* **Kit de développement logiciel (SDK) Windows** : Pour installer le Kit de développement logiciel (SDK) Windows, consultez [Téléchargements et outils pour Windows 10](https://dev.windows.com/downloads). Le Kit de développement logiciel (SDK) vous permet de créer un certificat de sécurité.
* **Principal de service Azure Active Directory** : Ce didacticiel décrit la procédure de création d’un principal de service dans Azure Active Directory (Azure AD). Toutefois, vous devez être administrateur Azure AD pour pouvoir créer un principal du service. Si vous êtes administrateur, vous pouvez ignorer ce prérequis et poursuivre le didacticiel.

    >[!NOTE]
    >Vous pouvez créer un principal de service uniquement si vous être administrateur Azure AD. Votre administrateur Azure AD doit créer un principal de service. Vous pouvez ensuite créer un cluster HDInsight avec Data Lake Storage Gen1. Le principal de service doit être créé à l’aide d’un certificat, comme décrit dans [Créer un principal de service avec certificat](../active-directory/develop/howto-authenticate-service-principal-powershell.md#create-service-principal-with-certificate-from-certificate-authority).
    >

## <a name="create-an-azure-data-lake-storage-gen1-account"></a>Créer un compte Azure Data Lake Storage Gen1

Pour créer un compte Data Lake Storage Gen1, procédez comme suit :

1. Sur votre bureau, ouvrez une fenêtre PowerShell et entrez l’extrait de code ci-dessous. Lorsque vous êtes invité à vous connecter, connectez-vous en tant qu’administrateur ou propriétaire de l’abonnement. 

    ```azurepowershell
    # Sign in to your Azure account
    Connect-AzAccount

    # List all the subscriptions associated to your account
    Get-AzSubscription

    # Select a subscription
    Set-AzContext -SubscriptionId <subscription ID>

    # Register for Data Lake Storage Gen1
    Register-AzResourceProvider -ProviderNamespace "Microsoft.DataLakeStore"
    ```

    > [!NOTE]
    > Si vous recevez une erreur similaire à `Register-AzResourceProvider : InvalidResourceNamespace: The resource namespace 'Microsoft.DataLakeStore' is invalid` pendant l’inscription du fournisseur de ressources Data Lake Storage Gen1, il est possible que votre abonnement ne soit pas approuvé pour Data Lake Storage Gen1. Pour activer votre abonnement Azure pour Data Lake Storage Gen1, suivez les instructions de la [Prise en main d’Azure Data Lake Storage Gen1 avec le portail Azure](data-lake-store-get-started-portal.md).
    >

2. Un compte Data Lake Storage Gen1 est associé à un groupe de ressources Azure. Commencez par créer un groupe de ressources.

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

3. Créer un compte Data Lake Storage Gen1. Le nom de compte que vous spécifiez doit contenir uniquement des lettres minuscules et des chiffres.

    ```azurepowershell
    $dataLakeStorageGen1Name = "<your new Data Lake Storage Gen1 name>"
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

4. Utiliser Data Lake Storage Gen1 en tant que stockage par défaut vous oblige à spécifier un chemin d’accès racine où les fichiers spécifiques au cluster sont copiés lors de la création du cluster. Pour créer un chemin d’accès racine, qui, dans l’extrait de code ci-dessous est **hdiadlcluster/clusters/** , utilisez les applets de commande ci-dessous :

    ```azurepowershell
    $myrootdir = "/"
    New-AzDataLakeStoreItem -Folder -AccountName $dataLakeStorageGen1Name -Path $myrootdir/clusters/hdiadlcluster
    ````

## <a name="set-up-authentication-for-role-based-access-to-data-lake-storage-gen1"></a>Configurer l’authentification pour définir un accès à Data Lake Storage Gen1 en fonction du rôle
Chaque abonnement Azure est associé à une entité Azure AD. Les utilisateurs et services qui accèdent aux ressources par le biais du portail Azure ou de l’API Azure Resource Manager doivent au préalable s’authentifier auprès d’Azure AD. L’accès est accordé aux abonnements et services Azure en leur affectant le rôle approprié sur une ressource Azure. Pour les services, un principal de service identifie le service dans Azure AD.

Cette section montre comment accorder l’accès à une ressource Azure (le compte Azure Data Lake Storage Gen1 créé précédemment) à un service d’application, comme HDInsight. Pour ce faire, créez un principal de service pour l’application, puis attribuez-lui des rôles avec Azure PowerShell.

Pour configurer l’authentification Active Directory pour Data Lake Storage Gen1, effectuez les tâches présentées dans les deux sections suivantes.

### <a name="create-a-self-signed-certificate"></a>Créer un certificat auto-signé
Assurez-vous que le [SDK Windows](https://dev.windows.com/en-us/downloads) est installé avant de suivre la procédure décrite dans cette section. Vous devez également avoir créé un répertoire, comme *C:\mycertdir*, où sera créé le certificat.

1. Dans la fenêtre PowerShell, accédez à l'emplacement où vous avez installé le Kit de développement logiciel (SDK) Windows (en général, *C:\Program Files (x86)\Windows Kits\10\bin\x86*) et utilisez l'utilitaire [MakeCert][makecert] pour créer un certificat auto-signé et une clé privée. Utilisez les commandes suivantes :

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

    Lorsque vous y êtes invité, entrez le mot de passe de la clé privée spécifié au préalable. La valeur que vous donnez au paramètre **-po** est le mot de passe associé au fichier .pfx. Une fois la commande exécutée avec succès, un **CertFile.pfx** est visible dans le répertoire du certificat que vous avez spécifié.

### <a name="create-an-azure-ad-and-a-service-principal"></a>Créer une application Azure AD et un principal de service
Cette section décrit les étapes pour créer un principal de service pour une application Azure AD, attribuer un rôle au principal de service et vous authentifier en tant que principal de service en fournissant un certificat. Pour créer une application dans Azure AD, exécutez les commandes suivantes :

1. Collez les applets de commande suivantes dans la fenêtre de la console PowerShell. Vérifiez que la valeur que vous donnez à la propriété **-DisplayName** est unique. Les valeurs pour **-HomePage** et **-IdentiferUris** sont des valeurs d’espace réservé et ne sont pas vérifiées.

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

2. Créez un principal de service en utilisant l’ID d’application.

    ```azurepowershell
    $servicePrincipal = New-AzADServicePrincipal -ApplicationId $applicationId

    $objectId = $servicePrincipal.Id
    ```

3. Accordez au principal du service l’accès à la racine Data Lake Storage Gen1 et à tous les dossiers du chemin d’accès racine spécifié précédemment. Utilisez les applets de commande suivantes :

    ```azurepowershell
    Set-AzDataLakeStoreItemAclEntry -AccountName $dataLakeStorageGen1Name -Path / -AceType User -Id $objectId -Permissions All
    Set-AzDataLakeStoreItemAclEntry -AccountName $dataLakeStorageGen1Name -Path /clusters -AceType User -Id $objectId -Permissions All
    Set-AzDataLakeStoreItemAclEntry -AccountName $dataLakeStorageGen1Name -Path /clusters/hdiadlcluster -AceType User -Id $objectId -Permissions All
    ```

## <a name="create-an-hdinsight-linux-cluster-with-data-lake-storage-gen1-as-the-default-storage"></a>Créer un cluster Linux HDInsight avec Data Lake Storage Gen1 comme stockage par défaut

Dans cette section, vous allez créer un cluster HDInsight Hadoop Linux avec Data Lake Storage Gen1 comme stockage par défaut. Pour cette version, le cluster HDInsight et Data Lake Storage Gen1 doivent se situer au même emplacement.

1. Récupérez l’ID de locataire d’abonnement et stockez-le pour une utilisation ultérieure.

    ```azurepowershell
    $tenantID = (Get-AzContext).Tenant.TenantId
    ```

2. Créez le cluster HDInsight en utilisant les applets de commande suivantes :

    ```azurepowershell
    # Set these variables

    $location = "East US 2"
    $storageAccountName = $dataLakeStorageGen1Name    # Data Lake Storage Gen1 account name
        $storageRootPath = "<Storage root path you specified earlier>"     # e.g. /clusters/hdiadlcluster
        $clusterName = "<unique cluster name>"
    $clusterNodes = <ClusterSizeInNodes>            # The number of nodes in the HDInsight cluster
    $httpCredentials = Get-Credential
    $sshCredentials = Get-Credential

    New-AzHDInsightCluster `
           -ClusterType Hadoop `
           -OSType Linux `
           -ClusterSizeInNodes $clusterNodes `
           -ResourceGroupName $resourceGroupName `
           -ClusterName $clusterName `
           -HttpCredential $httpCredentials `
           -Location $location `
           -DefaultStorageAccountType AzureDataLakeStore `
           -DefaultStorageAccountName "$storageAccountName.azuredatalakestore.net" `
           -DefaultStorageRootPath $storageRootPath `
           -Version "3.6" `
           -SshCredential $sshCredentials `
           -AadTenantId $tenantId `
           -ObjectId $objectId `
           -CertificateFilePath $certificateFilePath `
           -CertificatePassword $password
    ```

    Après exécution de l’applet de commande, le résultat énumère les détails du cluster.

## <a name="run-test-jobs-on-the-hdinsight-cluster-to-use-data-lake-storage-gen1"></a>Exécuter des tâches de test sur le cluster HDInsight pour utiliser Data Lake Storage Gen1
Une fois que vous avez configuré un cluster HDInsight, vous pouvez exécuter des tâches de test sur le cluster pour vérifier qu’il peut accéder à Data Lake Storage Gen1. Pour ce faire, exécutez un exemple de tâche Hive qui crée une table avec les exemples de données déjà disponibles dans Data Lake Storage Gen1 dans *\<cluster root>/example/data/sample.log*.

Dans cette section, vous établissez une connexion SSH (Secure Shell) au cluster HDInsight Linux que vous avez créé, puis vous exécutez l’exemple de requête Hive.

* Si vous utilisez un client Windows pour établir la connexion SSH au cluster, consultez [Utilisation de SSH avec Hadoop Linux sur HDInsight à partir de Windows](../hdinsight/hdinsight-hadoop-linux-use-ssh-windows.md).
* Si vous utilisez un client Linux pour établir la connexion SSH au cluster, consultez [Utilisation de SSH avec Hadoop Linux sur HDInsight à partir de Linux](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

1. Une fois que vous avez établi la connexion, démarrez l'interface de ligne de commande Hive à l’aide de la commande suivante :

    ```powershell
    hive
    ```

2. À l’aide de l’interface de ligne de commande, entrez les instructions suivantes pour créer une table nommée **vehicles** en utilisant les exemples de données dans Data Lake Storage Gen1 :

    ```azurepowershell
    DROP TABLE log4jLogs;
        CREATE EXTERNAL TABLE log4jLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)
        ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
        STORED AS TEXTFILE LOCATION 'adl:///example/data/';
        SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log' GROUP BY t4;
    ```

Vous devriez voir la sortie de la requête dans la console SSH.

>[!NOTE]
>Le chemin d’accès aux exemples de données de la commande CREATE TABLE ci-dessus est `adl:///example/data/`, où `adl:///` est la racine du cluster. Pour l’exemple de la racine de cluster spécifiée dans ce didacticiel, la commande est `adl://hdiadlstore.azuredatalakestore.net/clusters/hdiadlcluster`. Vous pouvez utiliser l’alternative la plus courte ou fournir le chemin d’accès complet vers la racine du cluster.
>

## <a name="access-data-lake-storage-gen1-by-using-hdfs-commands"></a>Accéder à Data Lake Storage Gen1 avec les commandes HDFS
Une fois que vous avez configuré le cluster HDInsight pour qu’il utilise Data Lake Storage Gen1, vous pouvez utiliser les commande de l’interpréteur de commandes HDFS (Hadoop Distributed File System) pour accéder au magasin.

Dans cette section, vous établissez une connexion SSH au cluster HDInsight Linux que vous avez créé, puis vous exécutez les commandes HDFS.

* Si vous utilisez un client Windows pour établir la connexion SSH au cluster, consultez [Utilisation de SSH avec Hadoop Linux sur HDInsight à partir de Windows](../hdinsight/hdinsight-hadoop-linux-use-ssh-windows.md).
* Si vous utilisez un client Linux pour établir la connexion SSH au cluster, consultez [Utilisation de SSH avec Hadoop Linux sur HDInsight à partir de Linux](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

Une fois que vous avez établi la connexion, répertoriez les fichiers dans Data Lake Storage Gen1 en utilisant la commande de système de fichiers HDFS suivante.

```azurepowershell
hdfs dfs -ls adl:///
```

Vous pouvez également utiliser la commande `hdfs dfs -put` pour charger des fichiers dans Data Lake Storage Gen1, puis utiliser `hdfs dfs -ls` pour vérifier si les fichiers ont été chargés avec succès.

## <a name="see-also"></a>Voir aussi
* [Utiliser Data Lake Storage Gen1 avec des clusters Azure HDInsight](../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen1.md)
* [Portail Azure : créer un cluster HDInsight pour utiliser Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md)

[makecert]: /windows-hardware/drivers/devtest/makecert
[pvk2pfx]: https://msdn.microsoft.com/library/windows/desktop/ff550672(v=vs.85).aspx
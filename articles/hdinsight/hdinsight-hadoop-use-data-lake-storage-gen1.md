---
title: Utiliser Data Lake Storage Gen1 avec Hadoop dans Azure HDInsight
description: Découvrez comment interroger des données à partir d’Azure Data Lake Storage Gen1 et stocker les résultats de votre analyse.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017,seoapr2020
ms.date: 04/24/2020
ms.openlocfilehash: 5949bab7bdf11b11e0ff71f9054098ed83d95ab4
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92539834"
---
# <a name="use-data-lake-storage-gen1-with-azure-hdinsight-clusters"></a>Utiliser Data Lake Storage Gen1 avec des clusters Azure HDInsight

> [!Note]
> Déployer de nouveaux clusters HDInsight à l’aide d’[Azure Data Lake Storage Gen2](hdinsight-hadoop-use-data-lake-storage-gen2.md) pour améliorer les performances et bénéficier des nouvelles fonctionnalités.

Pour analyser des données sur un cluster HDInsight, vous pouvez stocker les données dans [`Azure Blob storage`](../storage/common/storage-introduction.md), [Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-overview.md) ou [Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md). Toutes les options de stockage vous permettent de supprimer de manière sécurisée des clusters HDInsight servant au calcul, sans perte de données utilisateur.

Dans cet article, vous découvrez le fonctionnement de Data Lake Storage Gen1 avec des clusters HDInsight. Pour savoir comment Stockage Blob Azure fonctionne avec les clusters HDInsight, consultez [Utiliser Stockage Blob Azure avec des clusters Azure HDInsight](hdinsight-hadoop-use-blob-storage.md). Pour plus d’informations sur la création d’un cluster HDInsight, consultez [Créer des clusters Apache Hadoop dans HDInsight](hdinsight-hadoop-provision-linux-clusters.md).

> [!NOTE]  
> Data Lake Storage Gen1 est toujours accessible par le biais d’un canal sécurisé ; il n’y a donc aucun nom de schéma de système de fichiers `adls`. Vous utilisez toujours `adl`.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="availability-for-hdinsight-clusters"></a>Disponibilité pour les clusters HDInsight

Apache Hadoop prend en charge une notion de système de fichiers par défaut. Le système de fichiers par défaut implique un schéma et une autorité par défaut. Il peut également être utilisé pour résoudre les chemins d'accès relatifs. Pendant le processus de création du cluster HDInsight, spécifiez un conteneur de blobs dans Stockage Azure comme système de fichiers par défaut. Sinon, avec HDInsight 3.5 et versions ultérieures, vous pouvez sélectionner Stockage Blob Azure ou Azure Data Lake Storage Gen1 comme système de fichiers par défaut avec quelques exceptions. Le cluster et le compte de stockage doivent être hébergés dans la même région.

Les clusters HDInsight peuvent utiliser Data Lake Storage Gen1 de deux manières :

* Comme stockage par défaut
* Comme stockage supplémentaire, avec Stockage Blob Azure comme stockage par défaut.

Actuellement, seules certain(e)s versions/types de cluster HDInsight prennent en charge l’utilisation de Data Lake Storage Gen1 comme stockage par défaut et de comptes de stockage supplémentaires :

| Type de cluster HDInsight | Utiliser Data Lake Storage Gen1 comme stockage par défaut | Utiliser Data Lake Storage Gen1 comme stockage supplémentaire| Notes |
|------------------------|------------------------------------|---------------------------------------|------|
| HDInsight version 4.0 | Non | Non |ADLS Gen1 n’est pas pris en charge avec HDInsight 4.0 |
| HDInsight version 3.6 | Oui | Oui | Tout sauf HBase|
| HDInsight version 3.5 | Oui | Oui | Tout sauf HBase|
| HDInsight version 3.4 | Non | Oui | |
| HDInsight version 3.3 | Non | Non | |
| HDInsight version 3.2 | Non | Oui | |
| Storm | | |Vous pouvez utiliser Data Lake Storage Gen1 pour écrire des données à partir d’une topologie Storm. Vous pouvez aussi utiliser Data Lake Storage Gen1 pour stocker des données de référence qui peuvent ensuite être lues par une topologie Storm.|

> [!WARNING]  
> HDInsight HBase n’est pas pris en charge avec Azure Data Lake Storage Gen1

L’utilisation de Data Lake Storage Gen1 comme compte de stockage supplémentaire n’affecte pas les performances, ni même la capacité à lire ou écrire dans Stockage Blob Azure à partir du cluster.

## <a name="use-data-lake-storage-gen1-as-default-storage"></a>Utiliser Data Lake Storage Gen1 comme stockage par défaut

Quand HDInsight est déployé avec Data Lake Storage Gen1 en tant que stockage par défaut, les fichiers associés au cluster sont stockés dans `adl://mydatalakestore/<cluster_root_path>/`, où `<cluster_root_path>` correspond au nom d’un dossier que vous créez dans Data Lake Storage. En spécifiant un chemin racine pour chaque cluster, vous pouvez utiliser le même compte Data Lake Storage pour plusieurs clusters. Par conséquent, vous pouvez avoir une configuration dans laquelle :

* Cluster1 peut utiliser le chemin d’accès `adl://mydatalakestore/cluster1storage`
* Cluster2 peut utiliser le chemin d’accès `adl://mydatalakestore/cluster2storage`

Notez que les deux clusters utilisent le même compte Data Lake Storage Gen1 **mydatalakestore** . Chaque cluster a accès à son propre système de fichiers racine dans Data Lake Storage. L’expérience de déploiement avec le Portail Azure vous invite à utiliser un nom de dossier du type **/clusters/\<clustername>** comme chemin racine.

Pour utiliser Data Lake Storage Gen1 comme stockage par défaut, vous devez accorder au principal de service l’accès aux chemins suivants :

* La racine du compte Data Lake Storage Gen1.  Par exemple : adl://mydatalakestore/.
* Dossier pour tous les dossiers de cluster.  Par exemple : adl://mydatalakestore/clusters.
* Dossier pour le cluster.  Par exemple : adl://mydatalakestore/clusters/cluster1storage.

Pour plus d’informations sur la création du principal de service et l’octroi de l’accès, consultez Configurer l’accès à Data Lake Storage.

### <a name="extracting-a-certificate-from-azure-keyvault-for-use-in-cluster-creation"></a>Extraction d’un certificat à partir d’Azure Key Vault pour une utilisation lors de la création du cluster

Si le certificat de votre principal de service est stocké dans Azure Key Vault, vous devez convertir le certificat au format approprié. Les extraits de code suivants montrent comment effectuer la conversion.

Tout d’abord, téléchargez le certificat à partir de Key Vault et extrayez le `SecretValueText`.

```powershell
$certPassword = Read-Host "Enter Certificate Password"
$cert = (Get-AzureKeyVaultSecret -VaultName 'MY-KEY-VAULT' -Name 'MY-SECRET-NAME')
$certValue = [System.Convert]::FromBase64String($cert.SecretValueText)
```

Ensuite, convertissez le `SecretValueText` en certificat.

```powershell
$certObject = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList $certValue,$null,"Exportable, PersistKeySet"
$certBytes = $certObject.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pkcs12, $certPassword.SecretValueText);
$identityCertificate = [System.Convert]::ToBase64String($certBytes)
```

Vous pouvez ensuite utiliser le `$identityCertificate` pour déployer un nouveau cluster, comme dans l’extrait de code suivant :

```powershell
New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $pathToArmTemplate `
    -identityCertificate $identityCertificate `
    -identityCertificatePassword $certPassword.SecretValueText `
    -clusterName  $clusterName `
    -clusterLoginPassword $SSHpassword `
    -sshPassword $SSHpassword `
    -servicePrincipalApplicationId $application.ApplicationId
```

## <a name="use-data-lake-storage-gen1-as-additional-storage"></a>Utiliser Data Lake Storage Gen1 comme stockage supplémentaire

Vous pouvez également utiliser Data Lake Storage Gen1 en tant que stockage supplémentaire pour le cluster. En pareil cas, le stockage par défaut du cluster peut être un compte Stockage Blob Azure ou Azure Data Lake Storage Gen1. Quand vous exécutez des tâches HDInsight sur des données stockées dans Azure Data Lake Storage Gen1 en tant que stockage supplémentaire, utilisez le chemin complet. Par exemple :

`adl://mydatalakestore.azuredatalakestore.net/<file_path>`

Il n’y a encore aucun **cluster_root_path** dans l’URL. Cela est dû au fait que Data Lake Storage n’est pas un stockage par défaut dans ce cas. Il vous suffit donc de fournir le chemin d’accès aux fichiers.

Pour pouvoir utiliser Data Lake Storage Gen1 comme stockage supplémentaire, accordez au principal de service l’accès aux chemins où sont stockés vos fichiers.  Par exemple :

`adl://mydatalakestore.azuredatalakestore.net/<file_path>`

Pour plus d’informations sur la création du principal de service et l’octroi de l’accès, consultez Configurer l’accès à Data Lake Storage.

## <a name="use-more-than-one-data-lake-storage-gen1-account"></a>Utiliser plusieurs comptes Data Lake Storage Gen1

Vous pouvez effectuer l’ajout d’un compte Data Lake Storage supplémentaire et l’ajout de plusieurs comptes Data Lake Storage. Accordez au cluster HDInsight l’autorisation sur les données dans un ou plusieurs comptes Data Lake Storage. Consultez Configurer l’accès à Data Lake Storage Gen1 ci-dessous.

## <a name="configure-data-lake-storage-gen1-access"></a>Configurer l’accès à Data Lake Storage Gen1

Pour configurer l’accès à Data Lake Storage Gen1 à partir de votre cluster HDInsight, vous devez disposer d’un principal de service Azure Active Directory (Azure AD). Vous pouvez créer un principal de service uniquement si vous être administrateur Azure AD. Le principal de service doit être créé avec un certificat. Pour plus d’informations, consultez [Démarrage rapide : Configurer des clusters dans HDInsight](./hdinsight-hadoop-provision-linux-clusters.md), et [Create service principal with self-signed-certificate](../active-directory/develop/howto-authenticate-service-principal-powershell.md#create-service-principal-with-self-signed-certificate) (Créer un principal du service avec un certificat auto-signé).

> [!NOTE]  
> Si vous comptez utiliser Azure Data Lake Storage Gen1 comme stockage supplémentaire pour le cluster HDInsight, nous vous recommandons vivement de procéder ainsi quand vous créez le cluster comme décrit dans cet article. L’ajout d’Azure Data Lake Storage Gen1 en tant que stockage supplémentaire à un cluster HDInsight existant n’est pas pris en charge.

Pour plus d’informations sur le modèle de contrôle d’accès, consultez [Contrôle d’accès dans Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-access-control.md).

## <a name="access-files-from-the-cluster"></a>Accès aux fichiers à partir du cluster

Il existe plusieurs méthodes pour accéder aux fichiers dans Data Lake Storage à partir d’un cluster HDInsight.

* **Utilisation du nom complet** . Avec cette approche, vous fournissez le chemin d’accès complet au fichier auquel vous souhaitez accéder.

    ```
    adl://<data_lake_account>.azuredatalakestore.net/<cluster_root_path>/<file_path>
    ```

* **Utilisation du format de chemin d’accès raccourci** . Avec cette approche, vous remplacez le chemin d’accès à la racine du cluster par :

    ```
    adl:///<file path>
    ```

* **Utilisation du chemin d’accès relatif** . Avec cette approche, vous fournissez uniquement le chemin d’accès relatif au fichier auquel vous souhaitez accéder.

    ```
    /<file.path>/
    ```

### <a name="data-access-examples"></a>Exemples d’accès aux données

Les exemples sont basés sur une [connexion ssh](./hdinsight-hadoop-linux-use-ssh-unix.md) au nœud principal du cluster. Les exemples utilisent les trois schémas d’URI. Remplacez `DATALAKEACCOUNT` et `CLUSTERNAME` par les valeurs correspondantes.

#### <a name="a-few-hdfs-commands"></a>Quelques commandes hdfs

1. Créez un fichier sur le stockage local.

    ```bash
    touch testFile.txt
    ```

1. Créez des répertoires sur le stockage en cluster.

    ```bash
    hdfs dfs -mkdir adl://DATALAKEACCOUNT.azuredatalakestore.net/clusters/CLUSTERNAME/sampledata1/
    hdfs dfs -mkdir adl:///sampledata2/
    hdfs dfs -mkdir /sampledata3/
    ```

1. Copiez les données du stockage local vers le stockage en cluster.

    ```bash
    hdfs dfs -copyFromLocal testFile.txt adl://DATALAKEACCOUNT.azuredatalakestore.net/clusters/CLUSTERNAME/sampledata1/
    hdfs dfs -copyFromLocal testFile.txt adl:///sampledata2/
    hdfs dfs -copyFromLocal testFile.txt /sampledata3/
    ```

1. Affichez le contenu du répertoire sur le stockage en cluster.

    ```bash
    hdfs dfs -ls adl://DATALAKEACCOUNT.azuredatalakestore.net/clusters/CLUSTERNAME/sampledata1/
    hdfs dfs -ls adl:///sampledata2/
    hdfs dfs -ls /sampledata3/
    ```

#### <a name="creating-a-hive-table"></a>Création d’une table Hive

Trois emplacements de fichiers sont indiqués à titre d’illustration. Pour l’exécution réelle, n’utilisez qu’une seule des entrées `LOCATION`.

```hql
DROP TABLE myTable;
CREATE EXTERNAL TABLE myTable (
    t1 string,
    t2 string,
    t3 string,
    t4 string,
    t5 string,
    t6 string,
    t7 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
STORED AS TEXTFILE
LOCATION 'adl://DATALAKEACCOUNT.azuredatalakestore.net/clusters/CLUSTERNAME/example/data/';
LOCATION 'adl:///example/data/';
LOCATION '/example/data/';
```

## <a name="identify-storage-path-from-ambari"></a>Identifier le chemin de stockage à partir d’Ambari

Pour identifier le chemin d’accès complet au magasin par défaut configuré, accédez à **HDFS** > **Configs** et entrez `fs.defaultFS` dans la zone de texte filtre.

## <a name="create-hdinsight-clusters-with-access-to-data-lake-storage-gen1"></a>Créer des clusters HDInsight ayant accès à Data Lake Storage Gen1

Suivez les liens ci-dessous pour obtenir des instructions détaillées sur la manière de créer des clusters HDInsight avec accès à Data Lake Storage Gen1.

* [Utilisation du portail](./hdinsight-hadoop-provision-linux-clusters.md)
* [Utilisation de PowerShell (avec Data Lake Storage Gen1 en tant que stockage par défaut)](../data-lake-store/data-lake-store-hdinsight-hadoop-use-powershell-for-default-storage.md)
* [Utilisation de PowerShell (avec Data Lake Storage Gen1 en tant que stockage supplémentaire)](../data-lake-store/data-lake-store-hdinsight-hadoop-use-powershell.md)
* [Utilisation de modèles Azure](../data-lake-store/data-lake-store-hdinsight-hadoop-use-resource-manager-template.md)

## <a name="refresh-the-hdinsight-certificate-for-data-lake-storage-gen1-access"></a>Actualiser le certificat HDInsight pour l’accès à Data Lake Storage Gen1

L’exemple de code PowerShell suivant lit un certificat à partir d’un fichier local ou d’Azure Key Vault, et met à jour votre cluster HDInsight avec le nouveau certificat pour accéder à Azure Data Lake Storage Gen1. Fournissez votre propre nom de cluster HDInsight, nom de groupe de ressources, ID d’abonnement, `app ID` et chemin local du certificat. Tapez le mot de passe quand vous y êtes invité.

```powershell-interactive
$clusterName = '<clustername>'
$resourceGroupName = '<resourcegroupname>'
$subscriptionId = '01234567-8a6c-43bc-83d3-6b318c6c7305'
$appId = '01234567-e100-4118-8ba6-c25834f4e938'
$addNewCertKeyCredential = $true
$certFilePath = 'C:\localfolder\adls.pfx'
$KeyVaultName = "my-key-vault-name"
$KeyVaultSecretName = "my-key-vault-secret-name"
$certPassword = Read-Host "Enter Certificate Password"
# certSource
# 0 - create self signed cert
# 1 - read cert from file path
# 2 - read cert from key vault
$certSource = 0

Login-AzAccount
Select-AzSubscription -SubscriptionId $subscriptionId

if($certSource -eq 0)
{
    Write-Host "Generating new SelfSigned certificate"

    $cert = New-SelfSignedCertificate -CertStoreLocation "cert:\CurrentUser\My" -Subject "CN=hdinsightAdlsCert" -KeySpec KeyExchange
    $certBytes = $cert.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pkcs12, $certPassword);
    $certString = [System.Convert]::ToBase64String($certBytes)
}
elseif($certSource -eq 1)
{

    Write-Host "Reading the cert file from path $certFilePath"

    $cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2($certFilePath, $certPassword)
    $certString = [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes($certFilePath))
}
elseif($certSource -eq 2)
{

    Write-Host "Reading the cert file from Azure Key Vault $KeyVaultName"

    $cert = (Get-AzureKeyVaultSecret -VaultName $KeyVaultName -Name $KeyVaultSecretName)
    $certValue = [System.Convert]::FromBase64String($cert.SecretValueText)
    $certObject = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList $certValue, $null,"Exportable, PersistKeySet"

    $certBytes = $certObject.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pkcs12, $certPassword.SecretValueText);

    $certString =[System.Convert]::ToBase64String($certBytes)
}

if($addNewCertKeyCredential)
{
    Write-Host "Creating new KeyCredential for the app"
    $keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData())
    New-AzADAppCredential -ApplicationId $appId -CertValue $keyValue -EndDate $cert.NotAfter -StartDate $cert.NotBefore
    Write-Host "Waiting for 7 minutes for the permissions to get propagated"
    Start-Sleep -s 420 #7 minutes
}

Write-Host "Updating the certificate on HDInsight cluster..."

Invoke-AzResourceAction `
    -ResourceGroupName $resourceGroupName `
    -ResourceType 'Microsoft.HDInsight/clusters' `
    -ResourceName $clusterName `
    -ApiVersion '2015-03-01-preview' `
    -Action 'updateclusteridentitycertificate' `
    -Parameters @{ ApplicationId = $appId; Certificate = $certString; CertificatePassword = $certPassword.ToString() } `
    -Force
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à utiliser Azure Data Lake Storage Gen1 compatible avec HDFS avec HDInsight. Ce stockage vous permet de créer des solutions d’acquisition de données d’archivage à long terme et adaptables. Vous pouvez aussi utiliser HDInsight pour déverrouiller les informations à l’intérieur des données structurées et non structurées stockées.

Pour plus d'informations, consultez les pages suivantes :

* [Démarrage rapide : Configurer des clusters dans HDInsight](./hdinsight-hadoop-provision-linux-clusters.md)
* [Créer un cluster HDInsight pour utiliser Data Lake Storage Gen1 à l’aide d’Azure PowerShell](../data-lake-store/data-lake-store-hdinsight-hadoop-use-powershell.md)
* [Téléchargement de données vers HDInsight](hdinsight-upload-data.md)
* [Utiliser des signatures d’accès partagé Stockage Blob Azure pour restreindre l’accès aux données avec HDInsight](hdinsight-storage-sharedaccesssignature-permissions.md)
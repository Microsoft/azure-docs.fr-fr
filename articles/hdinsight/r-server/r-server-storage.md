---
title: Solutions de stockage Azure pour ML Services sur HDInsight – Azure
description: Découvrez les différentes options de stockage disponibles avec ML Services sur HDInsight.
ms.service: hdinsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 01/02/2020
ms.openlocfilehash: 1b684fde9123d3c12d5d69c1daec1c53c6519c44
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91855291"
---
# <a name="azure-storage-solutions-for-ml-services-on-azure-hdinsight"></a>Solutions de stockage Azure pour ML Services sur Azure HDInsight

ML Services sur HDInsight peut utiliser différentes solutions de stockage pour conserver les données, le code ou les objets qui contiennent des résultats d’analyse. Ces solutions incluent les options suivantes :

- [stockage d’objets blob Azure](https://azure.microsoft.com/services/storage/blobs/)
- [Azure Data Lake Storage Gen1](https://azure.microsoft.com/services/storage/data-lake-storage/)
- [Stockage Fichier Azure](https://azure.microsoft.com/services/storage/files/)

Vous avez également la possibilité d’accéder à plusieurs conteneurs ou comptes de stockage Azure avec votre cluster HDInsight. Le Stockage Fichier Azure est une option de stockage de données pratique, que l’on peut utiliser sur le nœud périphérique pour monter d’un partage de fichiers de stockage Azure sur, par exemple, le système de fichiers Linux. Mais les partages de fichiers Azure peuvent être montés et utilisés par tous les systèmes qui fonctionnent sur un système d’exploitation pris en charge, par exemple, Windows ou Linux.

Lors de la création d’un cluster Apache Hadoop dans HDInsight, vous spécifiez un compte de **stockage Blob Azure** ou bien **Data Lake Storage Gen1**. Un conteneur de stockage spécifique de ce compte contient le système de fichiers du cluster créé (par exemple, le système de fichiers DFS Hadoop). Pour plus d’informations et pour obtenir de l’aide, consultez les pages :

- [Utilisation du stockage d’objets blob Azure avec HDInsight](../hdinsight-hadoop-use-blob-storage.md)
- [Utiliser Data Lake Storage Gen1 avec des clusters Azure HDInsight](../hdinsight-hadoop-use-data-lake-storage-gen1.md)

## <a name="use-azure-blob-storage-accounts-with-ml-services-cluster"></a>Utiliser des comptes de stockage Blob Azure avec un cluster ML Services

Si vous avez spécifié plusieurs comptes de stockage au moment de créer votre cluster ML Services, les instructions suivantes expliquent comment utiliser un compte secondaire pour l’accès aux données et les opérations sur un cluster ML Services. Imaginons les éléments suivants : un compte de stockage **storage1** avec un conteneur par défaut appelé **container1** et un compte de stockage **storage2** avec un conteneur **container2**.

> [!WARNING]  
> Pour des raisons de performances, le cluster HDInsight est créé dans le même centre de données que le compte de stockage principal que vous spécifiez. L’utilisation d’un compte de stockage dans un autre emplacement que le cluster HDInsight n’est pas prise en charge.

### <a name="use-the-default-storage-with-ml-services-on-hdinsight"></a>Utiliser le stockage par défaut avec ML Services sur HDInsight

1. À l’aide d’un client SSH, connectez-vous au nœud de périphérie de votre cluster. Pour en savoir plus sur l’utilisation de SSH avec des clusters HDInsight, consultez la page [Utiliser SSH avec HDInsight](../hdinsight-hadoop-linux-use-ssh-unix.md).
  
2. Copiez un exemple de fichier, mysamplefile.csv, dans le répertoire /share.

    ```bash
    hadoop fs –mkdir /share
    hadoop fs –copyFromLocal mycsv.scv /share
    ```

3. Dans R Studio ou une autre console R, écrivez le code R pour nommer le nœud **default** et définir l’emplacement du fichier auquel vous souhaitez accéder.  

    ```R
    myNameNode <- "default"
    myPort <- 0

    #Location of the data:  
    bigDataDirRoot <- "/share"  

    #Define Spark compute context:
    mySparkCluster <- RxSpark(nameNode=myNameNode, consoleOutput=TRUE)

    #Set compute context:
    rxSetComputeContext(mySparkCluster)

    #Define the Hadoop Distributed File System (HDFS) file system:
    hdfsFS <- RxHdfsFileSystem(hostName=myNameNode, port=myPort)

    #Specify the input file to analyze in HDFS:
    inputFile <-file.path(bigDataDirRoot,"mysamplefile.csv")
    ```

Toutes les références de fichiers et de répertoires pointent vers le compte de stockage `wasbs://container1@storage1.blob.core.windows.net`. Il s’agit du **compte de stockage par défaut** associé au cluster HDInsight.

### <a name="use-the-additional-storage-with-ml-services-on-hdinsight"></a>Utiliser le stockage supplémentaire avec ML Services sur HDInsight

Supposons maintenant que vous souhaitiez traiter un fichier appelé mysamplefile1.csv, qui se trouve dans le répertoire /private du conteneur **container2**, dans le compte de stockage **storage2**.

Dans votre code R, pointez la référence du nœud de nom vers le compte de stockage **storage2** .

```R
myNameNode <- "wasbs://container2@storage2.blob.core.windows.net"
myPort <- 0

#Location of the data:
bigDataDirRoot <- "/private"

#Define Spark compute context:
mySparkCluster <- RxSpark(consoleOutput=TRUE, nameNode=myNameNode, port=myPort)

#Set compute context:
rxSetComputeContext(mySparkCluster)

#Define HDFS file system:
hdfsFS <- RxHdfsFileSystem(hostName=myNameNode, port=myPort)

#Specify the input file to analyze in HDFS:
inputFile <-file.path(bigDataDirRoot,"mysamplefile1.csv")
```

Toutes les références de fichier et de répertoire pointent désormais vers le compte de stockage `wasbs://container2@storage2.blob.core.windows.net`. Il s’agit du **nom de nœud** que vous avez spécifié.

Configurez le répertoire `/user/RevoShare/<SSH username>` sur **storage2** comme suit :

```bash
hadoop fs -mkdir wasbs://container2@storage2.blob.core.windows.net/user
hadoop fs -mkdir wasbs://container2@storage2.blob.core.windows.net/user/RevoShare
hadoop fs -mkdir wasbs://container2@storage2.blob.core.windows.net/user/RevoShare/<RDP username>
```

## <a name="use-azure-data-lake-storage-gen1-with-ml-services-cluster"></a>Utiliser Azure Data Lake Storage Gen1 avec un cluster ML Services

Pour utiliser Data Lake Storage Gen1 avec votre cluster HDInsight, vous devez permettre à ce dernier d’accéder à chaque instance Azure Data Lake Storage Gen1 que vous souhaitez utiliser. Pour obtenir des instructions sur l’utilisation du portail Azure en vue de créer un cluster HDInsight avec une instance Azure Data Lake Storage Gen1 utilisée comme stockage par défaut ou comme stockage supplémentaire, consultez [Créer un cluster HDInsight avec Data Lake Storage Gen1 à l’aide du portail Azure](../../data-lake-store/data-lake-store-hdinsight-hadoop-use-portal.md).

Vous utilisez alors le stockage de votre script R à peu près de la même manière qu’un compte de stockage Azure secondaire, comme le décrit la procédure précédente.

### <a name="add-cluster-access-to-your-azure-data-lake-storage-gen1"></a>Ajouter un accès de cluster à votre instance Azure Data Lake Storage Gen1

Vous accédez à Data Lake Storage Gen1 en utilisant un principal du service Azure Active Directory (Azure AD) associé à votre cluster HDInsight.

1. Lorsque vous créez votre cluster HDInsight, sélectionnez **Identité Azure AD de cluster** à partir de l’onglet **Source de données**.

2. Dans la boîte de dialogue **Identité Azure AD de cluster**, sous **Sélectionner un principal de service AD**, sélectionnez **Créer**.

Après avoir attribué un nom et un mot de passe au principal du service, cliquez sur **Gérer l’accès à ADLS** pour l’associer à votre instance Data Lake Storage.

Il est également possible d’ajouter un accès à un ou plusieurs comptes Data Lake Storage Gen1 pour le cluster après sa création. Ouvrez l’entrée d’une instance Data Lake Storage Gen1 sur le portail Azure et accédez à **Explorateur de données > Accès > Ajouter**.

### <a name="how-to-access-data-lake-storage-gen1-from-ml-services-on-hdinsight"></a>Comment accéder à Data Lake Storage Gen1 à partir de ML Services sur HDInsight

Dès lors que vous avez accès à Data Lake Storage Gen1, vous pouvez utiliser ce stockage dans un cluster ML Services sur HDInsight de la même façon qu'un compte de stockage Azure secondaire. La seule différence est que le préfixe **wasbs://** devient **adl://** , comme suit :

```R
# Point to the ADL Storage (e.g. ADLtest)
myNameNode <- "adl://rkadl1.azuredatalakestore.net"
myPort <- 0

# Location of the data (assumes a /share directory on the ADL account)
bigDataDirRoot <- "/share"  

# Define Spark compute context
mySparkCluster <- RxSpark(consoleOutput=TRUE, nameNode=myNameNode, port=myPort)

# Set compute context
rxSetComputeContext(mySparkCluster)

# Define HDFS file system
hdfsFS <- RxHdfsFileSystem(hostName=myNameNode, port=myPort)

# Specify the input file in HDFS to analyze
inputFile <-file.path(bigDataDirRoot,"mysamplefile.csv")
```

Les commandes suivantes sont utilisées pour configurer l’instance Data Lake Storage Gen1 avec le répertoire RevoShare, et pour ajouter également le fichier .csv de l’exemple précédent :

```bash
hadoop fs -mkdir adl://rkadl1.azuredatalakestore.net/user
hadoop fs -mkdir adl://rkadl1.azuredatalakestore.net/user/RevoShare
hadoop fs -mkdir adl://rkadl1.azuredatalakestore.net/user/RevoShare/<user>

hadoop fs -mkdir adl://rkadl1.azuredatalakestore.net/share

hadoop fs -copyFromLocal /usr/lib64/R Server-7.4.1/library/RevoScaleR/SampleData/mysamplefile.csv adl://rkadl1.azuredatalakestore.net/share

hadoop fs –ls adl://rkadl1.azuredatalakestore.net/share
```

## <a name="use-azure-file-storage-with-ml-services-on-hdinsight"></a>Utiliser le Stockage Fichier Azure avec ML Services sur HDInsight

Il existe également une option de stockage de données pratique pour le nœud de périmètre, nommée [Azure Files](https://azure.microsoft.com/services/storage/files/). Elle vous permet de monter un partage de fichiers Azure Storage sur le système de fichiers Linux. Cette option peut être utile pour stocker des fichiers de données, des scripts R et des objets de résultats qui pourront se révéler nécessaires par la suite, lorsqu’il sera judicieux d’utiliser le système de fichiers natif sur le nœud périphérique plutôt que HDFS.

Le principal avantage des fichiers Azure est que les partages de fichiers peuvent être montés et utilisés par tout système disposant d’un système d’exploitation pris en charge, tel que Windows ou Linux. Par exemple, ils peuvent être utilisés par un autre cluster HDInsight que vous ou un membre de votre équipe avez, par une machine virtuelle Azure ou même par un système local. Pour plus d'informations, consultez les pages suivantes :

- [Guide pratique pour utiliser le Stockage Fichier Azure avec Linux](../../storage/files/storage-how-to-use-files-linux.md)
- [Guide pratique pour utiliser le Stockage Fichier Azure sous Windows](../../storage/files/storage-dotnet-how-to-use-files.md)

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble du cluster ML Services sur HDInsight](r-server-overview.md)
- [Options de contexte de calcul pour un cluster ML Services sur HDInsight](r-server-compute-contexts.md)
- [Utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight](../hdinsight-hadoop-use-data-lake-storage-gen2.md)

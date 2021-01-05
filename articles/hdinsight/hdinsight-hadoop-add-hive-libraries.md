---
title: Bibliothèques Apache Hive durant la création d’un cluster - Azure HDInsight
description: Découvrez comment ajouter des bibliothèques Apache Hive (fichiers jar) à un cluster HDInsight pendant la création du cluster.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: H1Hack27Feb2017,hdinsightactive
ms.date: 02/14/2020
ms.openlocfilehash: c678372fbd54e528a8a16eacc601e815cfd32e58
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86082231"
---
# <a name="add-custom-apache-hive-libraries-when-creating-your-hdinsight-cluster"></a>Ajouter des bibliothèques Apache Hive personnalisées lors de la création de votre cluster HDInsight

Découvrez comment précharger des bibliothèques [Apache Hive](https://hive.apache.org/) dans HDInsight. Ce document contient des informations sur l’utilisation d’une action de script pour précharger des bibliothèques durant la création d’un cluster. Les bibliothèques ajoutées à l’aide de la procédure de ce document sont disponibles dans Hive : il n’est pas nécessaire d’utiliser [ADD JAR](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli) pour les charger.

## <a name="how-it-works"></a>Fonctionnement

Quand vous créez un cluster, vous pouvez utiliser une action de script pour modifier les nœuds de cluster au fur et à mesure qu’ils sont créés. Le script dans ce document accepte un seul paramètre, qui correspond à l’emplacement des bibliothèques. Cet emplacement doit se trouver dans un compte de stockage Azure, et les bibliothèques doivent être stockées sous forme de fichiers jar.

Lors de la création du cluster, le script énumère les fichiers, les copie sur le répertoire `/usr/lib/customhivelibs/` sur le nœud principal et le nœud de travail, puis les ajoute à la propriété `hive.aux.jars.path` dans le fichier `core-site.xml`. Pour les clusters basés sur Linux, il met également à jour le fichier `hive-env.sh` avec l'emplacement des fichiers.

L’utilisation de l’action de script dans cet article rend les bibliothèques disponibles lors de l’utilisation d’un client Hive pour **WebHCat** et **HiveServer2**.

## <a name="the-script"></a>Le script

**Emplacement du script**

[https://hdiconfigactions.blob.core.windows.net/setupcustomhivelibsv01/setup-customhivelibs-v01.ps1](https://hdiconfigactions.blob.core.windows.net/setupcustomhivelibsv01/setup-customhivelibs-v01.ps1)

### <a name="requirements"></a>Spécifications

* Les scripts doivent être appliqués à la fois aux **nœuds principaux** et aux **nœuds de travail**.

* Les fichiers jar à installer doivent être stockés dans le stockage d'objets Blob Azure dans un **seul conteneur**.

* Le compte de stockage contenant la bibliothèque de fichiers jar **doit** être lié au cluster HDInsight lors de la création. Il doit s’agir du compte de stockage par défaut ou d’un compte ajouté via les __paramètres du compte de stockage__.

* Le chemin WASB vers le conteneur doit être spécifié en tant que paramètre à l'action de script. Par exemple, si les fichiers jar sont stockés dans un conteneur nommé **libs** sur un compte de stockage nommé **mystorage**, le paramètre serait `wasbs://libs@mystorage.blob.core.windows.net/`.

  > [!NOTE]  
  > Ce document suppose que vous avez déjà créé un compte de stockage et un conteneur d'objets blob et que vous avez déjà chargé les fichiers dans ce dernier.
  >
  > Si vous n'avez pas créé de compte de stockage, vous pouvez le faire via le [portail Azure](https://portal.azure.com). Vous pouvez ensuite utiliser un utilitaire tel que [Azure Storage Explorer](https://storageexplorer.com/) pour créer un conteneur dans le compte et y télécharger des fichiers.

## <a name="create-a-cluster-using-the-script"></a>Créer un cluster à l'aide du script

1. Démarrez le provisionnement d’un cluster à l’aide de la procédure décrite dans [Provisionner des clusters HDInsight sous Linux](hdinsight-hadoop-provision-linux-clusters.md), mais n’achevez pas le provisionnement. Vous pouvez également utiliser Azure PowerShell ou le Kit de développement logiciel (SDK) .NET HDInsight pour créer un cluster à l’aide de ce script. Pour plus d’informations sur ces méthodes, consultez [Personnaliser des clusters HDInsight à l’aide d’actions de script](hdinsight-hadoop-customize-cluster-linux.md). Dans le portail Azure, sous l’onglet **Configuration + prix**, sélectionnez **+ Ajouter une action de script**.

1. Pour **Stockage**, si le compte de stockage contenant la bibliothèque de fichiers jar sera différent du compte utilisé pour le cluster, effectuez la procédure sous **Comptes de stockage supplémentaires**.

1. Pour **Actions de script**, entrez les informations suivantes :

    |Propriété |Valeur |
    |---|---|
    |Type de script|- Personnalisé|
    |Name|Bibliothèques |
    |URI de script bash|`https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v01.sh`|
    |Type(s) de nœud|Head, Worker|
    |Paramètres|entrez l’adresse WASB du compte de stockage et du conteneur qui contiennent les fichiers jar. Par exemple : `wasbs://libs@mystorage.blob.core.windows.net/`.|

    > [!NOTE]
    > Pour Apache Spark 2.1, utilisez cet URI de script bash : `https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v00.sh`.

1. Continuez l’approvisionnement du cluster, comme indiqué dans [Approvisionner des clusters HDInsight sous Linux](hdinsight-hadoop-provision-linux-clusters.md).

Quand la création du cluster est terminée, vous pouvez utiliser les fichiers jar ajoutés avec ce script à partir de Hive sans avoir à utiliser l’instruction `ADD JAR`.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d'informations sur l’utilisation de Hive, consultez la rubrique [Utilisation d’Apachee Hive avec HDInsight](hadoop/hdinsight-use-hive.md)

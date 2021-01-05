---
title: Utilisation de l'interface CLI HDFS avec Azure Data Lake Storage Gen2
description: Utilisez l’interface CLI du système de fichiers distribué Hadoop (HDFS) pour Azure Data Lake Storage Gen2. Créez un conteneur, obtenez une liste de fichiers ou de répertoires, et bien plus encore.
services: storage
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 12/06/2018
ms.author: normesta
ms.subservice: data-lake-storage-gen2
ms.reviewer: artek
ms.openlocfilehash: d2b36dd600efa864913e0087c49bffd556e8330d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95912397"
---
# <a name="using-the-hdfs-cli-with-data-lake-storage-gen2"></a>Utilisation de l’interface CLI HDFS avec Data Lake Storage Gen2

Vous pouvez accéder aux données et les gérer dans votre compte de stockage à l’aide d’une interface de ligne de commande comme vous le feriez avec un [Système de fichiers DFS Hadoop (HDFS)](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html). Cet article fournit quelques exemples qui vous aideront à démarrer.

HDInsight permet d’accéder au conteneur distribué connecté localement aux nœuds de calcul. Ce conteneur est accessible à l’aide de l’interpréteur de commandes qui interagit directement avec HDFS et d’autres systèmes de fichiers pris en charge par Hadoop.

Pour plus d’informations sur l’interface de ligne de commande HDFS, voir la [documentation officielle](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html) et le [Guide des autorisations HDFS](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsPermissionsGuide.html).

>[!NOTE]
>Si vous utilisez Azure Databricks au lieu de HDInsight, et souhaitez interagir avec vos données à l’aide d’une interface de ligne de commande, vous pouvez utiliser celle de Databricks pour interagir avec le système de fichiers Databricks. Voir [Interface de ligne de commande Databricks](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html).

## <a name="use-the-hdfs-cli-with-an-hdinsight-hadoop-cluster-on-linux"></a>Utiliser l’interface CLI HDFS avec un cluster HDInsight Hadoop sur Linux

Tout d’abord, établissez l’[accès à distance aux services](../../hdinsight/hdinsight-hadoop-linux-information.md#remote-access-to-services). Si vous choisissez [SSH](../../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md), l’exemple de code PowerShell se présente comme suit :

```powershell
#Connect to the cluster via SSH.
ssh sshuser@clustername-ssh.azurehdinsight.net
#Execute basic HDFS commands. Display the hierarchy.
hdfs dfs -ls /
#Create a sample directory.
hdfs dfs -mkdir /samplefolder
```
La chaîne de connexion se trouve dans la section « Connexion SSH + Cluster » du panneau du cluster HDInsight dans le portail Azure. Les informations d’identification SSH ont été spécifiées au moment de la création du cluster.

>[!IMPORTANT]
>La facturation du cluster HDInsight démarre après la création du cluster et s’arrête à sa suppression. La facturation est effectuée au prorata des minutes écoulées. Par conséquent, vous devez toujours supprimer votre cluster lorsqu’il n’est plus utilisé. Pour découvrir comment supprimer un cluster, consultez notre [article à ce sujet](../../hdinsight/hdinsight-delete-cluster.md). Toutefois, les données stockées dans un compte de stockage avec Azure Data Lake Storage Gen2 persistent même après la suppression du cluster HDInsight.

## <a name="create-a-container"></a>Créez un conteneur.

`hdfs dfs -D "fs.azure.createRemoteFileSystemDuringInitialization=true" -ls abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/`

* Remplacez l’espace réservé `<container-name>` par le nom que vous souhaitez donner à votre conteneur.

* Remplacez la valeur d’espace réservé `<storage-account-name>` par le nom de votre compte de stockage.

## <a name="get-a-list-of-files-or-directories"></a>Obtenir la liste des fichiers ou répertoires

`hdfs dfs -ls <path>`

Remplacez l’espace réservé `<path>` par l’URI du conteneur ou du dossier du conteneur.

Par exemple : `hdfs dfs -ls abfs://my-file-system@mystorageaccount.dfs.core.windows.net/my-directory-name`

## <a name="create-a-directory"></a>Créer un répertoire

`hdfs dfs -mkdir [-p] <path>`

Remplacez l’espace réservé `<path>` par le nom du conteneur racine ou par un dossier de votre conteneur.

Par exemple : `hdfs dfs -mkdir abfs://my-file-system@mystorageaccount.dfs.core.windows.net/`

## <a name="delete-a-file-or-directory"></a>Supprimer un fichier ou répertoire

`hdfs dfs -rm <path>`

Remplacez l'espace réservé `<path>` par l'URI du fichier ou du dossier à supprimer.

Par exemple : `hdfs dfs -rmdir abfs://my-file-system@mystorageaccount.dfs.core.windows.net/my-directory-name/my-file-name`

## <a name="display-the-access-control-lists-acls-of-files-and-directories"></a>Afficher les listes de contrôle d’accès (ACL) des fichiers et répertoires

`hdfs dfs -getfacl [-R] <path>`

Exemple :

`hdfs dfs -getfacl -R /dir`

Voir [getfacl](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html#getfacl)

## <a name="set-acls-of-files-and-directories"></a>Définir les listes ACL des fichiers et répertoires

`hdfs dfs -setfacl [-R] [-b|-k -m|-x <acl_spec> <path>]|[--set <acl_spec> <path>]`

Exemple :

`hdfs dfs -setfacl -m user:hadoop:rw- /file`

Voir [setfacl](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html#setfacl)

## <a name="change-the-owner-of-files"></a>Changer le propriétaire des fichiers

`hdfs dfs -chown [-R] <new_owner>:<users_group> <URI>`

Voir [chown](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html#chown)

## <a name="change-group-association-of-files"></a>Changer l’association de groupe des fichiers

`hdfs dfs -chgrp [-R] <group> <URI>`

Voir [chgrp](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html#chgrp)

## <a name="change-the-permissions-of-files"></a>Changer les autorisations des fichiers

`hdfs dfs -chmod [-R] <mode> <URI>`

Voir [chmod](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html#chmod)

Vous pouvez voir la liste complète des commandes en accédant au site web [Apache Hadoop 2.4.1 File System Shell Guide](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html).

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser un compte Azure Data Lake Storage Gen2 dans Azure Databricks](./data-lake-storage-quickstart-create-databricks-account.md)

* [En savoir plus sur les listes de contrôle d’accès sur les fichiers et répertoires](./data-lake-storage-access-control.md)
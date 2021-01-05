---
title: Utiliser SCP avec Apache Hadoop dans Azure HDInsight
description: Ce document fournit des informations sur la connexion à HDInsight à l’aide des commandes SSH et SCP.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: seoapr2020
ms.date: 04/22/2020
ms.openlocfilehash: 44c87a3415a236f614510aa3163ad2e9cbd1f64f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91856175"
---
# <a name="use-scp-with-apache-hadoop-in-azure-hdinsight"></a>Utiliser SCP avec Apache Hadoop dans Azure HDInsight

Cet article fournit des informations sur le transfert sécurisé de fichiers à l’aide de votre cluster HDInsight.

## <a name="copy-files"></a>Copie des fichiers

L’utilitaire `scp` vous permet de copier des fichiers à partir ou à destination de nœuds spécifiques dans le cluster. Par exemple, la commande ci-après copie le fichier `test.txt` du système local vers le nœud principal primaire :

```bash
scp test.txt sshuser@clustername-ssh.azurehdinsight.net:
```

Étant donné qu’aucun chemin d’accès n’est spécifié après le caractère `:`, le fichier est placé dans le répertoire de base `sshuser`.

L’exemple ci-après copie le fichier `test.txt` du répertoire de base `sshuser` du nœud principal primaire vers le système local :

```bash
scp sshuser@clustername-ssh.azurehdinsight.net:test.txt .
```

La commande `scp` peut uniquement accéder au système de fichiers de nœuds spécifiques au sein du cluster. Elle ne permet pas d’accéder aux données du stockage compatible avec HDFS pour le cluster.

Utilisez la commande `scp` lorsque vous avez besoin de charger une ressource en vue de son utilisation à partir d’une session SSH. Par exemple, chargez un script Python, puis exécutez le script à partir d’une session SSH.

Pour plus d’informations sur le chargement direct de données dans le stockage compatible avec HDFS, consultez les documents suivants :

* [HDInsight avec Stockage Azure](hdinsight-hadoop-use-blob-storage.md)
* [HDInsight avec Azure Data Lake Storage Gen1](../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen1.md).

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser SSH avec HDInsight](./hdinsight-hadoop-linux-use-ssh-unix.md)
* [Utiliser des nœuds de périmètre vides dans HDInsight](hdinsight-apps-use-edge-node.md#access-an-edge-node)

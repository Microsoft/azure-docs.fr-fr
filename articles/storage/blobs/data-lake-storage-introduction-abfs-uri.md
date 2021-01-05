---
title: Utiliser l’URI Azure Data Lake Storage Gen2
description: En savoir plus sur la syntaxe d’URI pour l’identificateur de schéma ABFS, qui représente le pilote Azure Blob File System (pilote Hadoop FileSystem pour Azure Data Lake Storage Gen2).
author: normesta
ms.topic: conceptual
ms.author: normesta
ms.date: 12/06/2018
ms.service: storage
ms.subservice: data-lake-storage-gen2
ms.reviewer: jamesbak
ms.openlocfilehash: 948b5aa0ad015f9f3c693e13219ec034724687c0
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95913162"
---
# <a name="use-the-azure-data-lake-storage-gen2-uri"></a>Utiliser l’URI Azure Data Lake Storage Gen2

Le pilote du [système de fichiers Hadoop](https://www.aosabook.org/en/hdfs.html) compatible avec Azure Data Lake Storage Gen2 est connu par son identificateur de schéma `abfs` (système de fichiers blob Azure). Conformément à d’autres pilotes de système de fichiers Hadoop, le pilote ABFS emploie un format d’URI pour traiter les fichiers et les répertoires dans un compte Data Lake Storage Gen2.

## <a name="uri-syntax"></a>Syntaxe d’URI

La syntaxe de l’URI Data Lake Storage Gen2 varie selon que votre compte de stockage est configuré ou non avec Data Lake Storage Gen2 comme système de fichiers par défaut.

Si le compte Data Lake Storage Gen2 que vous ciblez **n’est pas** défini comme système de fichiers par défaut à la création du compte, la syntaxe d’URI raccourcie est la suivante :

<pre>abfs[s]<sup>1</sup>://&lt;file_system&gt;<sup>2</sup>@&lt;account_name&gt;<sup>3</sup>.dfs.core.windows.net/&lt;path&gt;<sup>4</sup>/&lt;file_name&gt;<sup>5</sup></pre>

1. **Identificateur de schéma** : le protocole `abfs` est utilisé comme identificateur de schéma. Vous avez la possibilité de vous connecter avec ou sans une connexion TLS (Transport Layer Security), précédemment connue sous le nom de connexion SSL (Secure Sockets Layer). Utilisez `abfss` pour vous connecter avec une connexion TLS.

2. **Système de fichiers** : emplacement parent qui contient les fichiers et dossiers. C’est l’équivalent des conteneurs du service Stockage Blob Azure.

3. **Nom du compte** : nom donné à votre compte de stockage à sa création.

4. **Chemins** : représentation délimitée par des barres obliques (`/`) de la structure de répertoires.

5. **Nom de fichier** : nom du fichier en question. Ce paramètre est facultatif si vous utilisez un répertoire.

Toutefois, si le compte que vous ciblez est défini comme système de fichiers par défaut à la création du compte, la syntaxe d’URI raccourcie est la suivante :

<pre>/&lt;path&gt;<sup>1</sup>/&lt;file_name&gt;<sup>2</sup></pre>

1. **Chemin d’accès** : représentation délimitée par des barres obliques (`/`) de la structure de répertoires.

2. **Nom de fichier** : nom du fichier en question.


## <a name="next-steps"></a>Étapes suivantes

- [Utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
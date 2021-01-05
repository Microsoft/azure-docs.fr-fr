---
title: Vue d’ensemble d’Azure Data Lake Storage Gen2 dans HDInsight
description: Vue d’ensemble de Data Lake Storage Gen2 dans HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: seoapr2020
ms.date: 04/21/2020
ms.openlocfilehash: a7ae1a0f77d0d19fc48695ddc5cc3d3a14d65ab9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82195126"
---
# <a name="azure-data-lake-storage-gen2-overview-in-hdinsight"></a>Vue d’ensemble d’Azure Data Lake Storage Gen2 dans HDInsight

Azure Data Lake Storage Gen2 offre les principales fonctionnalités d’Azure Data Lake Storage Gen1 et les intègre au Stockage Blob Azure. Ces fonctionnalités incluent un système de fichiers compatible avec Hadoop, Azure Active Directory (Azure AD) et les listes de contrôle d’accès (ACL) POSIX. Cette combinaison vous permet de tirer parti des performances d’Azure Data Lake Storage Gen1, tout en utilisant la hiérarchisation et la gestion du cycle de vie des données du stockage Blob.

Pour plus d’informations sur Azure Data Lake Storage Gen2, consultez [Présentation d’Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md).

## <a name="core-functionality-of-azure-data-lake-storage-gen2"></a>Fonctionnalités principales d’Azure Data Lake Storage Gen2

* **Accès compatible avec Hadoop :** dans Azure Data Lake Storage Gen2, vous pouvez gérer les données et y accéder comme vous le feriez avec un système de fichiers HDFS (Hadoop Distributed File System). Le pilote ABFS (Azure Blob Filesystem) est disponible dans tous les environnements Apache Hadoop, notamment Azure HDInsight et Azure Databricks. Utilisez ABFS pour accéder aux données stockées dans Data Lake Storage Gen2.

* **Surensemble d’autorisations POSIX :** Le modèle de sécurité pour Data Lake Gen2 prend en charge les autorisations ACL et POSIX, ainsi que certaines granularités supplémentaires spécifiques à Data Lake Storage Gen2. Les paramètres peuvent être configurés par le biais d’outils d’administration ou de frameworks comme Apache Hive et Apache Spark.

* **Rentabilité :** Data Lake Storage Gen2 offre une capacité de stockage et des transactions à bas coût. Les cycles de vie de Stockage Blob Azure vous aident à réduire les coûts en adaptant les frais facturés au fil du cheminement des données dans leur cycle de vie.

* **Compatibilité avec les outils, les frameworks et les applications de Stockage Blob :** Data Lake Storage Gen2 continue de fonctionner avec une large gamme d’outils, de frameworks et d’applications pour Stockage Blob.

* **Pilote optimisé :** Le pilote ABFS est optimisé spécifiquement pour l’analytique Big Data. Les API REST correspondantes sont exposées par le biais du point de terminaison DFS (Distributed File System), dfs.core.windows.net.

## <a name="whats-new-for-azure-data-lake-storage-gen-2"></a>Nouveautés d’Azure Data Lake Storage Gen2

### <a name="managed-identities-for-secure-file-access"></a>Identités managées pour l’accès sécurisé aux fichiers

Azure HDInsight utilise des identités managées pour sécuriser l’accès du cluster aux fichiers dans Azure Data Lake Storage Gen2. Les identités managées sont une fonctionnalité d’Azure Active Directory qui fournit des services Azure avec un ensemble d’informations d’identification gérées automatiquement. Ces informations d’identification peuvent être utilisées pour s’authentifier auprès de n’importe quel service prenant en charge l’authentification Active Directory. L’utilisation des identités managées vous évite de stocker les informations d’identification dans les fichiers de code ou de configuration.

Pour plus d’informations, voir [Identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/overview.md).

### <a name="azure-blob-file-system-driver"></a>Pilote Azure Blob File System

Les applications Apache Hadoop peuvent lire et écrire en mode natif des données dans un stockage sur disque local. Un pilote de système de fichiers Hadoop comme ABFS permet aux applications Hadoop d’utiliser un stockage cloud. Il fonctionne en émulant les opérations normales du système de fichiers Hadoop. Le pilote convertit les commandes reçues de l'application en opérations qui sont comprises par la plateforme de stockage cloud réelle.

Avant, le pilote de système de fichiers Hadoop convertissait toutes les opérations du système de fichiers en appels d’API REST Stockage Azure du côté client. Il appelait ensuite l’API REST. Cette conversion côté client entraînait cependant plusieurs appels d’API REST pour une même opération du système de fichiers, comme le renommage d’un fichier. ABFS a déplacé la logique du système de fichiers Hadoop du côté client vers le côté serveur. L’API Azure Data Lake Storage Gen2 s’exécute désormais en parallèle avec l’API d’objet blob. Cette migration améliore les performances, car les opérations courantes du système de fichiers Hadoop peuvent désormais être exécutées avec un seul appel d’API REST.

Pour plus d’informations, consultez [Pilote ABFS (Azure Blob Filesystem) : pilote Stockage Azure dédié pour Hadoop](../storage/blobs/data-lake-storage-abfs-driver.md).

### <a name="uri-scheme-for-azure-data-lake-storage-gen-2"></a>Schéma d’URI Azure Data Lake Storage Gen2

Azure Data Lake Storage Gen2 utilise un nouveau schéma d’URI pour accéder aux fichiers de Stockage Azure à partir d’HDInsight :

`abfs://<FILE_SYSTEM_NAME>@<ACCOUNT_NAME>.dfs.core.windows.net/<PATH>`

Le schéma d’URI fournit un accès chiffré SSL.

`<FILE_SYSTEM_NAME>` identifie le chemin du système de fichiers Data Lake Storage Gen2.

`<ACCOUNT_NAME>` identifie le nom du compte Stockage Azure. Un nom de domaine complet (FQDN) est requis.

`<PATH>` correspond au nom du chemin HDFS du fichier ou du répertoire.

Si vous ne spécifiez pas de valeurs pour `<FILE_SYSTEM_NAME>` et `<ACCOUNT_NAME>`, le système de fichiers par défaut est utilisé. Pour les fichiers du système de fichiers par défaut, utilisez un chemin relatif ou absolu. Par exemple, vous pouvez référencer le fichier `hadoop-mapreduce-examples.jar` fourni avec les clusters HDInsight en utilisant un des chemins suivants :

```
abfs://myfilesystempath@myaccount.dfs.core.windows.net/example/jars/hadoop-mapreduce-examples.jar
abfs:///example/jars/hadoop-mapreduce-examples.jar /example/jars/hadoop-mapreduce-examples.jar
```

> [!NOTE]
> Le nom du fichier est `hadoop-examples.jar` sur les clusters HDInsight version 2.1 et 1.6. Quand vous utilisez des fichiers en dehors de HDInsight, vous constaterez que la plupart des utilitaires ne reconnaissent pas le format ABFS et attendent plutôt un format de chemin de base, comme `example/jars/hadoop-mapreduce-examples.jar`.

Pour plus d’informations, consultez [Utiliser l’URI d’Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction-abfs-uri.md).

## <a name="next-steps"></a>Étapes suivantes

* [Présentation d’Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md)
* [Introduction à Azure Storage](../storage/common/storage-introduction.md)
* [Vue d’ensemble d’Azure Data Lake Storage Gen1](./overview-data-lake-storage-gen1.md)
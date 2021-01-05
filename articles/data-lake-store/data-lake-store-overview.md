---
title: Qu’est-ce qu’Azure Data Lake Storage Gen1 ? | Microsoft Docs
description: Présentation de Data Lake Storage Gen1 (anciennement Azure Data Lake Store) et de son apport par rapport à d’autres magasins de données
services: data-lake-store
author: twooley
ms.service: data-lake-store
ms.topic: overview
ms.date: 04/17/2019
ms.author: twooley
ms.openlocfilehash: 52c00359fb4e759d9517dee958e31c00feb2b33e
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92149248"
---
# <a name="what-is-azure-data-lake-storage-gen1"></a>Qu’est-ce qu’Azure Data Lake Storage Gen1 ?

[!INCLUDE [data-lake-storage-gen1-rename-note.md](../../includes/data-lake-storage-gen1-rename-note.md)]

Azure Data Lake Storage Gen1 est un référentiel d’entreprise à très grande échelle pour les charges de travail d’analyse du Big Data. Azure Data Lake vous permet de capturer les données de toute taille, de tout type et à toute vitesse d'ingestion dans un emplacement unique en vue d'une analyse opérationnelle et exploratoire.

Data Lake Storage Gen1 est accessible à partir de Hadoop (disponible avec le cluster HDInsight) avec les API REST compatibles WebHDFS. Il est conçu pour permettre l’analyse des données stockées et met l’accent sur les performances des scénarios d’analyse des données. Data Lake Storage Gen1 intègre toutes les capacités de qualité professionnelle requises en matière de sécurité, de facilité de gestion, d’extensibilité, de fiabilité et de disponibilité.

![Azure Data Lake](./media/data-lake-store-overview/data-lake-store-concept.png)

## <a name="key-capabilities"></a>Fonctionnalités clés

Voici quelques-unes des fonctionnalités clés de Data Lake Storage Gen1.

### <a name="built-for-hadoop"></a>Conçu pour Hadoop

Data Lake Storage Gen1 est un système de fichiers Apache Hadoop compatible avec HDFS (Hadoop Distributed File System) et fonctionnant avec l’écosystème Hadoop. Vos applications ou services HDInsight existants, qui utilisent l’API WebHDFS, peuvent s’intégrer facilement à Data Lake Storage Gen1. Data Lake Storage Gen1 offre également une interface REST compatible WebHDFS pour les applications.

Vous pouvez facilement analyser les données stockées dans Data Lake Storage Gen1 avec les infrastructures d’analyse Hadoop, comme MapReduce ou Hive. Vous pouvez approvisionner et configurer les clusters Azure HDInsight pour accéder directement aux données stockées dans Data Lake Storage Gen1.

### <a name="unlimited-storage-petabyte-files"></a>Stockage illimité, fichiers de l’ordre du pétaoctet

Data Lake Storage Gen1 offre un stockage illimité et peut stocker une grande variété de données à des fins d’analyse. Il n’impose aucune limite de taille de compte, de taille de fichier ou de quantité de données stockées dans un Data Lake. La taille des fichiers individuels peut varier d’un kilooctet à plusieurs pétaoctets. Les données sont stockées durablement en créant des copies multiples. De plus, il n’existe aucune limite de durée de stockage des données dans le lac de données.

### <a name="performance-tuned-for-big-data-analytics"></a>Performances optimisées pour l'analyse du Big Data

Data Lake Storage Gen1 est conçu pour faire fonctionner des systèmes d’analyse à grande échelle, nécessitant un débit élevé pour interroger et analyser de grands volumes de données. Le lac de données répartit les différentes parties des fichiers sur plusieurs serveurs de stockage distincts. Cela améliore le débit de lecture lors de la lecture du fichier en parallèle de l'analyse de données.

### <a name="enterprise-ready-highly-available-and-secure"></a>Service adapté aux entreprises : sécurisé et hautement disponible

Data Lake Storage Gen1 offre une fiabilité et une disponibilité aux normes industrielles. Vos données sont stockées durablement en créant des copies redondantes pour vous prémunir contre toute défaillance imprévue.

Data Lake Storage Gen1 fournit également une sécurité de niveau entreprise des données stockées. Pour plus d’informations, consultez [Sécurisation des données dans Azure Data Lake Storage Gen1](#DataLakeStoreSecurity).

### <a name="all-data"></a>Toutes les données

Data Lake Storage Gen1 peut stocker des données dans leur format natif, sans nécessiter de transformations préalables. Data Lake Storage Gen1 ne requiert pas la définition d’un schéma avant le chargement des données, laissant ainsi à l’infrastructure d’analyse individuelle le soin d’interpréter les données et de définir un schéma au moment de l’analyse. La possibilité de stocker les fichiers de formats et de tailles arbitraires rend possible la gestion par Data Lake Storage Gen1 de données structurées, semi-structurées et non structurées.

Les conteneurs Data Lake Storage Gen1 pour données sont essentiellement des dossiers et des fichiers. Vous travaillez sur les données stockées à l’aide de SDK, du portail Azure et d’Azure Powershell. Si vous placez vos données dans le magasin en utilisant ces interfaces et les conteneurs appropriés, vous pouvez stocker n’importe quel type de données. Data Lake Storage Gen1 n’effectue aucune opération particulière sur les données en fonction du type de données qu’il contient.

## <a name="securing-data"></a><a name="DataLakeStoreSecurity"></a>Sécurisation des données

Data Lake Storage Gen1 utilise Azure Active Directory pour l’authentification et les listes de contrôle d’accès pour gérer l’accès à vos données.

| Fonctionnalité | Description |
| --- | --- |
| Authentification |Data Lake Storage Gen1 s’intègre à Azure AD pour la gestion des identités et des accès de toutes les données stockées dans Data Lake Storage Gen1. Grâce à cette intégration, Data Lake Storage Gen1 tire parti de toutes les fonctionnalités d’Azure AD, notamment l’authentification multifacteur, l’accès conditionnel, le contrôle d’accès en fonction du rôle Azure, la supervision de l’utilisation des applications, la supervision de la sécurité et les alertes, etc. Data Lake Storage Gen1 prend en charge le protocole OAuth 2.0 pour l’authentification auprès de l’interface REST. Consultez [Data Lake Storage Gen1 authentication](data-lakes-store-authentication-using-azure-active-directory.md) (Authentification Data Lake Storage Gen1).|
| Contrôle d’accès |Data Lake Storage Gen1 offre un contrôle d’accès grâce à la prise en charge des autorisations de type POSIX exposées par le protocole WebHDFS. Vous pouvez activer les ACL au niveau du dossier racine, des sous-dossiers et des fichiers individuels. Pour plus d’informations sur le fonctionnement des listes ACL dans le contexte de Data Lake Storage Gen1, consultez [Contrôle d’accès dans Data Lake Storage Gen1](data-lake-store-access-control.md). |
| Chiffrement |Data Lake Storage Gen1 assure également le chiffrement des données stockées dans le compte. Vous spécifiez les paramètres de chiffrement lors de la création d’un compte Data Lake Storage Gen1. Vous pouvez choisir de chiffrer vos données ou non. Pour plus d’informations, consultez l’article [Chiffrement dans Data Lake Storage Gen1](data-lake-store-encryption.md). Pour savoir comment définir une configuration avec chiffrement, consultez [Prise en main de Data Lake Storage Gen1 à l’aide du Portail Azure](data-lake-store-get-started-portal.md). |

Pour savoir comment sécuriser les données dans Data Lake Storage Gen1, consultez [Sécurisation des données dans Azure Data Lake Storage Gen1](data-lake-store-secure-data.md).

## <a name="application-compatibility"></a>Compatibilité des applications

Data Lake Storage Gen1 est compatible avec la plupart des composants open source de l’écosystème Hadoop. Par ailleurs, il s’intègre facilement avec d’autres services Azure. Pour en savoir plus sur l’utilisation de Data Lake Storage Gen1 avec des composants open source et d’autres services Azure, suivez les liens suivants :

- Consultez [Applications et services compatibles avec Azure Data Lake Storage Gen1](data-lake-store-compatible-oss-other-applications.md) pour obtenir une liste des applications open source interopérables avec Data Lake Storage Gen1.
- Consultez [Intégration avec d’autres services Azure](data-lake-store-integrate-with-other-services.md) pour comprendre l’utilisation de Data Lake Storage Gen1 avec d’autres services Azure afin d’augmenter le nombre de scénarios.
- Consultez la page [Scénarios de données impliquant Data Lake Storage Gen1](data-lake-store-data-scenarios.md) pour découvrir comment utiliser Data Lake Storage Gen1 pour des scénarios tels que la réception, le traitement, le téléchargement et la visualisation de données.

## <a name="data-lake-storage-gen1-file-system"></a>Système de fichiers Data Lake Storage Gen1

Data Lake Storage Gen1 est accessible via le système de fichiers AzureDataLakeFilesystem (adl://) dans les environnements Hadoop (disponible avec le cluster HDInsight). Les applications et les services qui utilisent adl:// peuvent tirer parti d’autres optimisations des performances, qui ne sont actuellement pas disponibles dans WebHDFS. Par conséquent, Data Lake Storage Gen1 vous laisse la liberté, soit de bénéficier de performances optimales en utilisant adl:// (option recommandée), soit de maintenir le code existant en continuant d’utiliser l’API WebHDFS directement. Azure HDInsight tire pleinement parti du système de fichiers AzureDataLakeFilesystem pour optimiser les performances sur Data Lake Storage Gen1.

Vous pouvez accéder à vos données dans Data Lake Storage Gen1 à l’aide de `adl://<data_lake_storage_gen1_name>.azuredatalakestore.net`. Pour plus d’informations sur la façon d’accéder aux données dans Data Lake Storage Gen1, consultez [Afficher les propriétés des données stockées](data-lake-store-get-started-portal.md#properties).

## <a name="next-steps"></a>Étapes suivantes

- [Prise en main de Data Lake Storage Gen1 à l’aide du portail Azure](data-lake-store-get-started-portal.md)
- [Prise en main de Data Lake Storage Gen1 à l’aide du kit de développement logiciel (SDK) .NET](data-lake-store-get-started-net-sdk.md)
- [Utiliser Azure HDInsight avec Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md)
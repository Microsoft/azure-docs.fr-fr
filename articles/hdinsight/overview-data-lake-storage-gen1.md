---
title: Vue d’ensemble d’Azure Data Lake Storage Gen1 dans HDInsight
description: Vue d’ensemble de Data Lake Storage Gen1 dans HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: seoapr2020
ms.date: 04/21/2020
ms.openlocfilehash: b598494500124ca4a70e8e2d7bfef023fbf7d498
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92328774"
---
# <a name="azure-data-lake-storage-gen1-overview-in-hdinsight"></a>Vue d’ensemble d’Azure Data Lake Storage Gen1 dans HDInsight

Azure Data Lake Storage Gen1 est un référentiel d’entreprise à très grande échelle pour les charges de travail d’analytique du Big Data. À l’aide d’Azure Data Lake, vous pouvez capturer des données de toute taille, de tout type et dont les vitesses d’ingestion sont variées. Les données sont capturées à un emplacement unique à des fins d’analytique opérationnelle et exploratoire.

Accédez à Data Lake Storage Gen1 à partir d’Hadoop (disponible avec un cluster HDInsight) avec les API REST compatibles WebHDFS. Data Lake Storage Gen1 est conçu pour permettre l’analytique des données stockées, et optimisé pour les performances des scénarios d’analytique des données. Gen1 offre les capacités qui sont essentielles pour les cas d’usage professionnels du monde réel. Ces capacités incluent la sécurité, la facilité de gestion, l’adaptabilité, la fiabilité et la disponibilité.

Pour obtenir des informations détaillées sur Azure Data Lake Store Gen1, consultez [Vue d’ensemble d’Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-overview.md).

Voici quelques-unes des fonctionnalités clés de Data Lake Storage Gen1.

## <a name="compatibility-with-hadoop"></a>Compatibilité avec Hadoop

Data Lake Storage Gen1 est un système de fichiers Apache Hadoop compatible avec HDFS et l’environnement Hadoop.  Les applications ou services HDInsight qui utilisent l’API WebHDFS peuvent s’intégrer facilement à Data Lake Storage Gen1. Data Lake Storage Gen1 offre également une interface REST compatible WebHDFS pour les applications.

Les données stockées dans Data Lake Storage Gen1 peuvent être facilement analysées à l’aide des infrastructures analytiques Hadoop. Des infrastructures telles que MapReduce ou Hive. Les clusters Azure HDInsight peuvent être provisionnés et configurés pour accéder directement aux données stockées dans Data Lake Storage Gen1.

## <a name="unlimited-storage-petabyte-files"></a>Stockage illimité, fichiers de l’ordre du pétaoctet

Data Lake Storage Gen1 offre un stockage illimité et est approprié pour le stockage de différents types de données à des fins d’analytique. Il n’impose aucune limite à la taille des comptes ou des fichiers, ni à la quantité de données pouvant être stockées dans un lac de données. La taille des fichiers individuels varie de l’ordre du kilo-octet à celui du pétaoctet, ce qui fait de Data Lake Storage Gen1 un choix idéal pour stocker tout type de données. Les données sont stockées durablement en créant des copies multiples. Par ailleurs, il n’existe aucune limite quant à la durée pendant laquelle les données peuvent être stockées dans le lac de données.

## <a name="performance-tuning-for-big-data-analytics"></a>Optimisation des performances pour l’analytique du Big Data

Data Lake Storage Gen1 est conçu pour les systèmes analytiques. Il s’agit de systèmes nécessitant un débit élevé pour interroger et analyser de grandes quantités de données. Le Data Lake répartit les différentes parties des fichiers sur plusieurs serveurs de stockage distincts. Quand vous analysez des données, cette configuration améliore le débit de lecture en cas de lecture du fichier en parallèle.

## <a name="readiness-for-enterprise-highly-available-and-secure"></a>Préparation pour l’entreprise : sécurisé et hautement disponible

Data Lake Storage Gen1 offre une fiabilité et une disponibilité aux normes industrielles. Les ressources de données sont stockés durablement : des copies redondantes offrent une protection contre toute défaillance inattendue. Les entreprises peuvent utiliser Data Lake Storage Gen1 dans leurs solutions et l’intégrer à leur plateforme de données existante.

Data Lake Storage Gen1 fournit également une sécurité de niveau entreprise des données stockées. Pour plus d’informations, consultez [Sécurisation des données dans Azure Data Lake Storage Gen1](#data-security-in-data-lake-storage-gen1).

## <a name="flexible-data-structures"></a>Structures de données flexibles

Data Lake Storage Gen1 peut stocker des données dans leur format natif, en l’état, sans nécessiter de transformations préalables. Data Lake Storage Gen1 ne nécessite pas la définition d’un schéma avant le chargement des données. Le framework d’analytique interprète les données et définit un schéma au moment de l’analyse. Data Lake Storage Gen1 est capable de gérer des données structurées, ainsi que des données semi-structurées et non structurées.

Les conteneurs Data Lake Storage Gen1 pour données sont essentiellement des dossiers et des fichiers. Vous travaillez sur les données stockées à l’aide de SDK, du portail Azure et d’Azure Powershell. Les données sont placées dans le magasin à l’aide de ces interfaces, et ces conteneurs peuvent stocker n’importe quel type de données. Data Lake Storage Gen1 n’effectue aucune opération particulière sur les données en fonction du type de données.

## <a name="data-security-in-data-lake-storage-gen1"></a>Sécurité des données dans Data Lake Storage Gen1

Data Lake Storage Gen1 utilise Azure Active Directory pour l’authentification, et des listes de contrôle d’accès pour gérer l’accès à vos données.

| **Fonctionnalité** | **Description** |
| --- | --- |
| Authentification |Data Lake Storage Gen1 s’intègre à Azure Active Directory (Azure AD) pour la gestion des identités et des accès de toutes les données stockées dans Data Lake Storage Gen1. Grâce à cette intégration, Data Lake Storage Gen1 tire parti de toutes les fonctionnalités d’Azure AD, Ces fonctionnalités incluent : l’authentification multifacteur, l’accès conditionnel et le contrôle d’accès en fonction du rôle Azure. Cela inclut également la surveillance de l’utilisation des applications, la surveillance de la sécurité et les alertes, etc. Data Lake Storage Gen1 prend en charge le protocole OAuth 2.0 pour l’authentification auprès de l’interface REST. Voir [Authentification auprès d’Azure Data Lake Storage Gen1 à l’aide d’Azure Active Directory](../data-lake-store/data-lakes-store-authentication-using-azure-active-directory.md).|
| Contrôle d’accès |Data Lake Storage Gen1 offre un contrôle d’accès grâce à la prise en charge des autorisations de type POSIX exposées par le protocole WebHDFS. Les ACL peuvent être activées au niveau du dossier racine, des sous-dossiers et des fichiers individuels. Pour plus d’informations sur le fonctionnement des listes ACL dans le contexte de Data Lake Storage Gen1, consultez [Contrôle d’accès dans Data Lake Storage Gen1](../data-lake-store/data-lake-store-access-control.md). |
| Chiffrement |Data Lake Storage Gen1 assure également le chiffrement des données stockées dans le compte. Vous spécifiez les paramètres de chiffrement lors de la création d’un compte Data Lake Storage Gen1. Vous pouvez choisir de chiffrer vos données ou non. Pour plus d’informations, consultez l’article [Chiffrement dans Data Lake Storage Gen1](../data-lake-store/data-lake-store-encryption.md). Pour savoir comment définir une configuration avec chiffrement, consultez [Bien démarrer avec Azure Data Lake Storage Gen1 à l’aide du Portail Azure](../data-lake-store/data-lake-store-get-started-portal.md). |

Pour en savoir plus sur la sécurisation des données dans Data Lake Storage Gen1, consultez [Sécurisation des données stockées dans Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-secure-data.md).

## <a name="applications-that-are-compatible-with-data-lake-storage-gen1"></a>Applications compatibles avec Data Lake Storage Gen1

Data Lake Storage Gen1 est compatible avec la plupart des composants open source de l’environnement Hadoop. Par ailleurs, il s’intègre facilement avec d’autres services Azure.  Suivez les liens ci-dessous pour en savoir plus sur l’utilisation de Data Lake Storage Gen1 avec des composants open source et d’autres services Azure.

* Voir [Applications Big Data open source qui fonctionnent avec Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-compatible-oss-other-applications.md).
* Consultez [Intégration d’Azure Data Lake Storage Gen1 à d’autres services Azure](../data-lake-store/data-lake-store-integrate-with-other-services.md) pour comprendre comment utiliser Data Lake Storage Gen1 avec d’autres services Azure afin d’augmenter le nombre de scénarios.
* Voir [Utilisation d’Azure Data Lake Storage Gen1 pour le Big Data](../data-lake-store/data-lake-store-data-scenarios.md).

## <a name="data-lake-storage-gen1-file-system-adl"></a>Système de fichiers Data Lake Storage Gen1 (adl://)

Dans les environnements Hadoop, vous pouvez accéder à Data Lake Storage Gen1 par le biais du nouveau système de fichiers AzureDataLakeFilesystem (adl://). Les performances des applications et services qui utilisent `adl://` peuvent être optimisées de plusieurs manières qui ne sont actuellement pas disponibles dans WebHDFS. Par conséquent, vous avez la possibilité de bénéficier des meilleures performances à l’aide du système de fichiers adl:// recommandé ou de conserver le code existant en continuant à utiliser directement l’API WebHDFS. Azure HDInsight tire pleinement parti du système de fichiers AzureDataLakeFilesystem pour optimiser les performances sur Data Lake Storage Gen1.

Accédez à vos données dans Data Lake Storage Gen1 en utilisant l’URI suivant :

`adl://<data_lake_storage_gen1_name>.azuredatalakestore.net`

Pour plus d’informations sur la façon d’accéder aux données dans Data Lake Storage Gen1, consultez [Actions disponibles sur les données stockées](../data-lake-store/data-lake-store-get-started-portal.md#properties).

## <a name="next-steps"></a>Étapes suivantes

* [Présentation d’Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md)
* [Introduction à Azure Storage](../storage/common/storage-introduction.md)
* [Vue d’ensemble d’Azure Data Lake Storage Gen2](./overview-data-lake-storage-gen2.md)
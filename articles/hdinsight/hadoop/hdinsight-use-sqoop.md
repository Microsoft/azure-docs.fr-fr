---
title: Exécuter des tâches Apache Sqoop avec Azure HDInsight (Apache Hadoop)
description: Découvrez comment utiliser Azure PowerShell sur une station de travail pour exécuter des importations/exportations de Sqoop entre un cluster Hadoop et une base de données Azure SQL.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 12/06/2019
ms.openlocfilehash: c09a910ea5f6ac5c66a94c73bda49d1f65630708
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92545223"
---
# <a name="use-apache-sqoop-with-hadoop-in-hdinsight"></a>Utiliser Apache Sqoop avec Hadoop dans HDInsight

[!INCLUDE [sqoop-selector](../../../includes/hdinsight-selector-use-sqoop.md)]

Découvrez comment utiliser Apache Sqoop dans HDInsight pour importer et exporter des données entre un cluster HDInsight et Azure SQL Database.

Bien qu’Apache Hadoop soit un choix naturel pour traiter des données non structurées et semi-structurées, comme des journaux et des fichiers, il peut également être nécessaire de traiter des données structurées stockées dans des bases de données relationnelles.

[Apache Sqoop](https://sqoop.apache.org/docs/1.99.7/user.html) est un outil conçu pour transférer des données entre des clusters Hadoop et des bases de données relationnelles. Vous pouvez l’utiliser pour importer dans un système de fichiers distribués Hadoop (HDFS) des données depuis un système de gestion de base de données relationnelle (SGBDR), tel que SQ Server, MySQL ou Oracle, transformer ces données dans Hadoop avec MapReduce ou Apache Hive, et exporter à nouveau les données dans un SGBDR. Dans cet article, vous allez utiliser Azure SQL Database comme base de données relationnelle.

> [!IMPORTANT]  
> Cet article configure un environnement de test pour effectuer le transfert de données. Vous choisissez ensuite une méthode de transfert de données pour cet environnement à partir d’une des méthodes de la section [Exécuter des tâches Sqoop](#run-sqoop-jobs).

Pour obtenir la liste des versions de Sqoop prises en charge par les clusters HDInsight, consultez [Quels sont les composants et versions Hadoop disponibles avec HDInsight ?](../hdinsight-component-versioning.md)

## <a name="understand-the-scenario"></a>Présentation du scénario

Le cluster HDInsight inclut des exemples de données. Vous utilisez les deux éléments suivants :

* Un fichier journal Apache Log4j, qui se trouve dans `/example/data/sample.log`. Ce fichier inclut les journaux d’activité suivants :

```text
2012-02-03 18:35:34 SampleClass6 [INFO] everything normal for id 577725851
2012-02-03 18:35:34 SampleClass4 [FATAL] system problem at id 1991281254
2012-02-03 18:35:34 SampleClass3 [DEBUG] detail for id 1304807656
...
```

* Une table Hive nommée `hivesampletable`, qui référence le fichier de données qui se trouve dans `/hive/warehouse/hivesampletable`. Cette table contient des données relatives aux appareils mobiles.
  
  | Champ | Type de données |
  | --- | --- |
  | clientid |string |
  | querytime |string |
  | market |string |
  | deviceplatform |string |
  | devicemake |string |
  | devicemodel |string |
  | state |string |
  | country |string |
  | querydwelltime |double |
  | sessionid |bigint |
  | sessionpagevieworder |bigint |

Dans cet article, vous utilisez ces deux jeux de données pour tester l’importation et l’exportation Sqoop.

## <a name="set-up-test-environment"></a><a name="create-cluster-and-sql-database"></a>Configurer un environnement de test

Le cluster, la base de données SQL et d’autres objets sont créés via le portail Azure en utilisant un modèle Azure Resource Manager. Le modèle se trouve dans les [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-hdinsight-linux-with-sql-database/). Le modèle Resource Manager appelle un package bacpac pour déployer les schémas de table sur une base de données SQL.  Le package bacpac est situé dans le conteneur d’objets blob public https://hditutorialdata.blob.core.windows.net/usesqoop/SqoopTutorial-2016-2-23-11-2.bacpac. Si vous souhaitez utiliser un conteneur privé pour stocker les fichiers bacpac, appliquez les valeurs suivantes au modèle :

```json
"storageKeyType": "Primary",
"storageKey": "<TheAzureStorageAccountKey>",
```

> [!NOTE]  
> L’importation à l’aide d’un modèle ou du portail Azure ne prend en charge que l’importation d’un fichier BACPAC à partir du stockage Blob Azure.

1. Sélectionnez l’image suivante pour ouvrir le modèle Resource Manager dans le portail Azure.

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-linux-with-sql-database%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-use-sqoop/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

2. Entrez les propriétés suivantes :

    |Champ |Valeur |
    |---|---|
    |Abonnement |Sélectionnez votre abonnement Azure dans la liste déroulante.|
    |Resource group |Sélectionnez votre groupe de ressources dans la liste déroulante, ou créez-en un|
    |Emplacement |Sélectionnez une région dans la liste déroulante.|
    |Nom du cluster |Entrez un nom pour le cluster Hadoop. Utilisez seulement des lettres minuscules.|
    |Nom d’utilisateur de connexion au cluster |Conservez la valeur préremplie `admin`.|
    |Mot de passe de connexion au cluster |Entrez un mot de passe.|
    |Nom d’utilisateur SSH |Conservez la valeur préremplie `sshuser`.|
    |Mot de passe SSH |Entrez un mot de passe.|
    |Nom de connexion de l’administrateur SQL |Conservez la valeur préremplie `sqluser`.|
    |Mot de passe de l’administrateur SQL |Entrez un mot de passe.|
    |Emplacement des artefacts | Utilisez la valeur par défaut, sauf si vous voulez utiliser votre propre fichier bacpac dans un emplacement différent.|
    |Jeton SAS de l’emplacement des artefacts |Laisser vide.|
    |Nom du fichier Bacpac |Utilisez la valeur par défaut, sauf si vous voulez utiliser votre propre fichier bacpac.|
    |Emplacement |Utilisez la valeur par défaut.|

    Le nom du [serveur SQL logique](../../azure-sql/database/logical-servers.md) sera `<ClusterName>dbserver`. Le nom de la base de données sera `<ClusterName>db`. Le nom du compte de stockage par défaut sera `e6qhezrh2pdqu`.

3. **J’accepte les termes et conditions mentionnés ci-dessus** .

4. Sélectionnez **Achat** . Une nouvelle vignette intitulée Envoi du déploiement pour Déploiement de modèle s’affiche. La création du cluster et de la base de données SQL prend environ 20 minutes.

## <a name="run-sqoop-jobs"></a>Exécuter des tâches Sqoop

HDInsight peut exécuter des tâches Sqoop à l’aide de différentes méthodes. Utilisez la table suivante pour choisir la méthode qui vous convient, puis cliquez sur le lien pour obtenir une présentation détaillée.

| **Utilisez-le** si vous souhaitez... | ... un interpréteur de commandes **interactif** | ... un traitement par **lots** | ...depuis ce **système d’exploitation cluster** |
|:--- |:---:|:---:|:--- |:--- |
| [SSH](apache-hadoop-use-sqoop-mac-linux.md) |? |? |Linux, Unix, Mac OS X ou Windows |
| [Kit de développement logiciel (SDK) .NET pour Hadoop](apache-hadoop-use-sqoop-dotnet-sdk.md) |&nbsp; |?  |Windows (pour l’instant) |
| [Azure PowerShell](apache-hadoop-use-sqoop-powershell.md) |&nbsp; |? |Windows |

## <a name="limitations"></a>Limites

* Exportation en bloc : avec HDInsight sous Linux, le connecteur Sqoop utilisé pour exporter des données vers Microsoft SQL Server ou SQL Database ne prend pas en charge les insertions en bloc.
* Traitement par lots : avec HDInsight sous Linux, lorsque vous utilisez le commutateur `-batch` pour effectuer des insertions, Sqoop effectue plusieurs insertions plutôt qu’un traitement par lots des opérations d’insertion.

## <a name="next-steps"></a>Étapes suivantes

Vous maîtrisez à présent l’utilisation de Sqoop. Pour plus d'informations, consultez les rubriques suivantes :

* [Utilisation d’Apache Hive avec HDInsight](./hdinsight-use-hive.md)
* [Chargez des données dans HDInsight](../hdinsight-upload-data.md) : Découvrez d’autres méthodes pour charger des données dans HDInsight ou le stockage Blob Azure.
* [Utiliser Apache Sqoop pour échanger des données entre Apache Hadoop sur HDInsight et SQL Database](./apache-hadoop-use-sqoop-mac-linux.md)
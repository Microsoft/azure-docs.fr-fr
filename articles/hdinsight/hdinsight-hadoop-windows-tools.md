---
title: Utiliser un PC Windows avec Hadoop sur HDInsight - Azure
description: Travailler à partir d’un PC Windows dans Hadoop sur HDInsight. Gérer et interroger des clusters avec les outils PowerShell, Visual Studio et Linux. Développer des solutions Big Data avec .NET.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive,hdiseo17may2017
ms.date: 12/20/2019
ms.openlocfilehash: 6482b2ec07a0473ddbea2098d37c5d67c8b68704
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92539698"
---
# <a name="work-in-the-apache-hadoop-ecosystem-on-hdinsight-from-a-windows-pc"></a>Travailler dans l’écosystème Apache Hadoop sur HDInsight à partir d’un PC Windows

Découvrez plus d’informations sur les options de développement et de gestion sur le PC Windows pour travailler dans l’écosystème Apache Hadoop sur HDInsight.

HDInsight est basé sur Apache Hadoop, et sur des composants et des technologies Hadoop open source développées sur Linux. HDInsight 3.4 et ultérieur utilise la distribution Linux Ubuntu comme système d’exploitation sous-jacent pour le cluster. Vous pouvez cependant travailler avec HDInsight à partir d’un client Windows ou de l’environnement de développement Windows.

## <a name="use-powershell-for-deployment-and-management-tasks"></a>Utiliser PowerShell pour les tâches de gestion et de déploiement

Azure PowerShell est un environnement de création de scripts qui vous permet de contrôler et d’automatiser le déploiement et la gestion de tâches dans HDInsight à partir de Windows.

Voici des exemples de tâches que vous pouvez effectuer avec PowerShell :

* [Créer des clusters à l’aide de PowerShell](hdinsight-hadoop-create-linux-clusters-azure-powershell.md)
* [Exécuter des requêtes Apache Hive avec PowerShell](hadoop/apache-hadoop-use-hive-powershell.md)
* [Gérer des clusters avec PowerShell](hdinsight-administer-use-powershell.md)

Procédez comme suit pour [installer et configurer Azure PowerShell](/powershell/azure/install-az-ps), et pour obtenir la version la plus récente.

## <a name="utilities-you-can-run-in-a-browser"></a>Utilitaires que vous pouvez exécuter dans un navigateur

Les utilitaires suivants ont une interface utilisateur web qui s’exécute dans un navigateur :
* **[Azure Cloud Shell](../cloud-shell/overview.md)** est un interpréteur de commandes interactif de lignes de commande qui s’exécute dans votre navigateur et depuis le Portail Azure.

* L’ **[interface utilisateur web d’Apache Ambari](hdinsight-hadoop-manage-ambari.md)** est un utilitaire de gestion et de surveillance disponible dans le portail Azure, qui peut être utilisé pour gérer différents types de tâches, comme :
    * [Utiliser Apache Ambari avec l’API REST](hdinsight-hadoop-manage-ambari-rest-api.md)
    * [Vue d’Apache Hive dans Apache Ambari](hadoop/apache-hadoop-use-hive-ambari-view.md)
    * [Vue d’Apache Tez dans Apache Ambari](./index.yml)

## <a name="data-lake-hadoop-tools-for-visual-studio"></a>Data Lake Tools (Hadoop) pour Visual Studio

Utilisez Data Lake Tools pour Visual Studio pour déployer et gérer des topologies Storm. Data Lake Tools installe également le SDK SCP.NET, qui vous permet de développer des topologies Storm C# avec Visual Studio.

Avant de passer aux exemples qui suivent, [installez et essayez Data Lake Tools pour Visual Studio](hadoop/apache-hadoop-visual-studio-tools-get-started.md).

Voici des exemples de tâches que vous pouvez effectuer avec Visual Studio et Data Lake Tools pour Visual Studio :
* [Déployer et gérer des topologies Storm depuis Visual Studio](storm/apache-storm-deploy-monitor-topology-linux.md)
* [Développer des topologies C# pour Storm à l’aide de Visual Studio](storm/apache-storm-develop-csharp-visual-studio-topology.md). Les fichiers binaires incluent des exemples de modèles pour les topologies Storm que vous pouvez connecter à des bases de données, comme Azure Cosmos DB et SQL Database.

## <a name="visual-studio-and-the-net-sdk"></a>Visual Studio et le SDK .NET

Vous pouvez utiliser Visual Studio avec le SDK .NET pour gérer des clusters et développer des applications Big Data. Vous pouvez utiliser d’autres environnements de développement intégrés pour les tâches suivantes, mais les exemples sont présentés dans Visual Studio.

Voici des exemples de tâches que vous pouvez effectuer avec le SDK .NET dans Visual Studio :
* [SDK Azure HDInsight pour .NET](/dotnet/api/overview/azure/hdinsight?view=azure-dotnet&preserve-view=true).
* [Exécuter des requêtes Apache Hive avec le SDK .NET](hadoop/apache-hadoop-use-hive-dotnet-sdk.md)
* [Utiliser des fonctions C# définies par l’utilisateur avec le streaming Apache Hive et Apache Pig sur Apache Hadoop](hadoop/apache-hadoop-hive-pig-udf-dotnet-csharp.md)

## <a name="intellij-idea-and-eclipse-ide-for-spark-clusters"></a>Intellij IDEA et IDE Eclipse pour les clusters Spark

[Intellij IDEA](https://www.jetbrains.com/idea/download) et l’[IDE Eclipse](https://www.eclipse.org/downloads/) peuvent tous deux être utilisés pour :
* Développer et soumettre une application Scala Spark sur un cluster HDInsight Spark.
* Accéder à des ressources de cluster Spark.
* Développer et exécuter une application Scala Spark localement.

Ces articles expliquent comment :
* IntelliJ IDEA : [Créer des applications Apache Spark en utilisant le kit de ressources Azure pour le plug-in Intellij et le SDK Scala.](spark/apache-spark-intellij-tool-plugin.md)
* IDE Eclipse ou IDE Scala pour Eclipse : [Créer des applications Apache Spark et Azure Toolkit pour Eclipse](spark/apache-spark-eclipse-tool-plugin.md)

## <a name="notebooks-on-spark-for-data-scientists"></a>Notebooks sur Spark pour les scientifiques des données

Les clusters Apache Spark dans HDInsight incluent les notebooks et les noyaux Apache Zeppelin qui peuvent être utilisés avec les notebooks Jupyter.

* [Découvrez comment utiliser les noyaux sur des clusters Apache Spark avec des notebooks Jupyter pour tester des applications Spark](spark/apache-spark-zeppelin-notebook.md)
* [Découvrez comment utiliser des notebooks Apache Zeppelin sur des clusters Apache Spark pour exécuter des tâches Spark](spark/apache-spark-jupyter-notebook-kernels.md)

## <a name="run-linux-based-tools-and-technologies-on-windows"></a>Exécuter sur Windows des outils et des technologies basés sur Linux

Si vous rencontrez une situation où vous devez utiliser un outil ou une technologie disponible uniquement dans Linux, envisagez les options suivantes :

* **Bash sur Ubuntu sur Windows 10** fournit un sous-système Linux sur Windows. Bash vous permet d’exécuter directement des utilitaires Linux sans avoir à gérer une installation Linux dédiée. Pour connaître les étapes d’installation, consultez [Guide d’installation de sous-systèmes Windows pour Linux sur Windows 10](/windows/wsl/install-win10).  Il est également possible d’utiliser d’autres [interpréteurs de commandes Unix](https://www.gnu.org/software/bash/).
* **Docker pour Windows** fournit un accès à de nombreux outils basés sur Linux et peut être exécuté directement à partir de Windows. Par exemple, vous pouvez utiliser Docker pour exécuter le client Beeline pour Hive directement depuis Windows. Vous pouvez également utiliser Docker pour exécuter un notebook Jupyter local et vous connecter à distance à Spark sur HDInsight. [Bien démarrer avec Docker pour Windows](https://docs.docker.com/docker-for-windows/)
* **[MobaXTerm](https://mobaxterm.mobatek.net/)** vous permet de parcourir graphiquement le système de fichiers du cluster via une connexion SSH.

## <a name="cross-platform-tools"></a>Outils multiplateformes

L’interface de ligne de commande Azure (CLI) est l’interface de ligne de commande multiplateforme de Microsoft pour la gestion de ressources Azure.  Pour plus d’informations, consultez [Interface de ligne de commande Azure (CLI)](/cli/azure/).

## <a name="next-steps"></a>Étapes suivantes

Si vous êtes novice dans l’utilisation de clusters Linux, consultez les articles suivants :
* [Configurer Apache Hadoop, Apache Kafka, Apache Spark ou d’autres clusters](hdinsight-hadoop-provision-linux-clusters.md)
* [Conseils pour les clusters HDInsight sur Linux](hdinsight-hadoop-linux-information.md)
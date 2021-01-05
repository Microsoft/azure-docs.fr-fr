---
title: Connecter Excel à Apache Hadoop à l’aide de Power Query - Azure HDInsight
description: Découvrez comment tirer profit des outils décisionnels et utiliser Power Query pour Excel afin d’accéder aux données stockées dans Hadoop sur HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/17/2019
ms.openlocfilehash: 8664efd40bb5392f56803515f09cccc800fdf21c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91397111"
---
# <a name="connect-excel-to-apache-hadoop-by-using-power-query"></a>Connecter Excel à Apache Hadoop à l’aide de Power Query

Une fonctionnalité clé de la solution de données volumineuses (« big data ») de Microsoft est l’intégration d’outils décisionnels (BI) Microsoft à des clusters Apche Hadoop dans Azure HDInsight. Un exemple important de cette intégration est la possibilité de connecter Excel au compte Azure Storage contenant les données associées à votre cluster Hadoop en utilisant le complément Microsoft Power Query pour Excel. Cet article vous explique comment configurer et utiliser Power Query pour interroger des données associées à un cluster Hadoop géré avec HDInsight.

## <a name="prerequisites"></a>Prérequis

* Un cluster Apache Hadoop sur HDInsight. Consultez [Bien démarrer avec HDInsight sur Linux](./apache-hadoop-linux-tutorial-get-started.md).
* Une station de travail fonctionnant sous Windows 10, 7, Windows Server 2008 R2 ou une version ultérieure.
* Microsoft 365 Apps for enterprise, Office 2016, Office Professionnel Plus 2013, l’édition autonome d’Excel 2013 ou Office Professionnel Plus 2010.

## <a name="install-microsoft-power-query"></a>Installez Microsoft Power Query

Power Query permet d'importer des données produites ou générées par un travail Hadoop s'exécutant sur un cluster HDInsight.

Dans Excel 2016, Power Query a été intégré dans le ruban de données sous la section Obtenir et transformer. Pour les versions Excel plus anciennes, téléchargez Microsoft Power Query pour Excel depuis le [Centre de téléchargement Microsoft](https://go.microsoft.com/fwlink/?LinkID=286689) et installez-le.

## <a name="import-hdinsight-data-into-excel"></a>Importation de données HDInsight dans Excel

Le complément Power Query pour Excel facilite l’importation de données depuis votre cluster HDInsight dans Excel, où des outils décisionnels tels que PowerPivot et Power Map peuvent être utilisés pour inspecter, analyser et présenter les données.

1. Lancez Excel.

1. Créez un classeur vide.

1. Effectuez les étapes suivantes, selon la version d’Excel :

   * Excel 2016

     * Sélectionnez > **Données** > **Récupérer des données** > **À partir d’Azure** > **À partir d’Azure HDInsight(HDFS)** .

       ![HDI.PowerQuery.SelectHdiSource.2016](./media/apache-hadoop-connect-excel-power-query/powerquery-selecthdisource-excel2016.png)

   * Excel 2013/2010

     * Sélectionnez **Power Query** > **À partir d’Azure** > **À partir de Microsoft Azure HDInsight**.

       ![HDI.PowerQuery.SelectHdiSource](./media/apache-hadoop-connect-excel-power-query/powerquery-selecthdisource.png)

       **Remarque :** Si vous ne voyez pas le menu **Power Query**, accédez à **Fichier** > **Options** > **Compléments** et sélectionnez **Compléments COM** dans la zone déroulante **Gérer** en bas de la page. Sélectionnez le bouton **OK** et vérifiez que la case correspondant au complément Power Query pour Excel a été cochée.

       **Remarque :** Power Query vous permet également d’importer des données à partir de HDFS en sélectionnant **À partir d’autres sources**.

1. Dans la boîte de dialogue **Azure HDInsight (HDFS)** , dans la zone de texte **Nom du compte ou URL**, entrez le nom du compte de stockage d’objets BLOB Azure associé à votre cluster. Sélectionnez ensuite **OK**. Il peut s’agir du compte de stockage par défaut ou d’un compte de stockage lié.  Le format est `https://StorageAccountName.blob.core.windows.net/`.

1. Pour le champ **Clé du compte**, saisissez la clé du compte de stockage d’objets blob, puis sélectionnez **Connecter**. (Entrez les informations sur le compte uniquement la première fois que vous accédez à ce Store.)

1. Dans le volet **Navigateur** situé à gauche de l’Éditeur de requête, double-cliquez sur le nom du conteneur de stockage d’objets Blob associé à votre cluster. Par défaut, le nom du conteneur est identique à celui du cluster.

1. Localisez **HiveSampleData.txt** dans la colonne **Nom** (le chemin du dossier est **../hive/warehouse/hivesampletable/** ), puis sélectionnez **Binaire** à gauche de HiveSampleData.txt. HiveSampleData.txt est fourni avec tout le cluster. Si vous le souhaitez, vous pouvez utiliser votre propre fichier.

    ![HDI - Excel Power Query, importer des données](./media/apache-hadoop-connect-excel-power-query/powerquery-importdata.png)

1. Le cas échéant, vous pouvez renommer le nom des colonnes. Quand vous êtes prêt, sélectionnez **Fermer et charger**.  Les données ont été chargées dans votre classeur :

    ![HDI - Excel Power Query, table importée](./media/apache-hadoop-connect-excel-power-query/powerquery-importedtable.png)

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à utiliser Power Query pour extraire des données de HDInsight et les importer dans Excel. De la même façon, vous pouvez extraire des données de HDInsight et les importer dans Azure SQL Database. Il est également possible de charger des données dans HDInsight. Pour en savoir plus, consultez les articles suivants :

* [Visualiser des données Apache Hive à l’aide de Microsoft Power BI dans Azure HDInsight](apache-hadoop-connect-hive-power-bi.md).
* [Visualiser des données Interactive Query Hive à l’aide de Power BI dans Azure HDInsight](../interactive-query/apache-hadoop-connect-hive-power-bi-directquery.md).
* [Utiliser Apache Zeppelin pour exécuter des requêtes Apache Hive dans Azure HDInsight](../interactive-query/hdinsight-connect-hive-zeppelin.md).
* [Connecter Excel à HDInsight avec le pilote ODBC Microsoft Hive](apache-hadoop-connect-excel-hive-odbc-driver.md).
* [Se connecter à Azure HDInsight et exécuter des requêtes Apache Hive avec Data Lake Tools pour Visual Studio](apache-hadoop-visual-studio-tools-get-started.md).
* [Utiliser Azure HDInsight Tools pour Visual Studio Code](../hdinsight-for-vscode.md).
* [Charger des données dans HDInsight](./../hdinsight-upload-data.md).

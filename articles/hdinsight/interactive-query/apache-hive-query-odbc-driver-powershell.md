---
title: Requêtes Apache Hive avec le pilote ODBC et PowerShell – Azure HDInsight
description: Utilisez le pilote ODBC Microsoft Hive et PowerShell pour interroger les clusters Apache Hive sur Azure HDInsight.
keywords: hive,hive odbc,powershell
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: tutorial
ms.date: 06/27/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 211b23db56b6deebb519b9eaac4164ea83edabac
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89078569"
---
# <a name="tutorial-query-apache-hive-with-odbc-and-powershell"></a>Tutoriel : Requête Apache Hive avec ODBC et PowerShell

Les pilotes Microsoft ODBC constituent une méthode flexible pour pour interagir avec différents types de sources de données, notamment Apache Hive. Vous pouvez écrire du code dans des langages de script, tels que PowerShell, qui utilisent des pilotes ODBC pour ouvrir une connexion vers votre cluster Hive, transmettre une requête de votre choix et afficher les résultats.

Dans ce didacticiel, vous allez apprendre à effectuer les tâches suivantes :

> [!div class="checklist"]
> * Télécharger et installer le pilote ODBC Microsoft Hive
> * Créer une source de données ODBC Apache Hive liée à votre cluster
> * Transmettre une requête d’exemples d’information à partir de votre cluster en utilisant PowerShell

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce didacticiel, vous devez disposer des éléments suivants :

* Un cluster Interactive Query sur HDInsight. Pour en créer un, consultez [Prise en main d’Azure HDInsight](../hdinsight-hadoop-provision-linux-clusters.md). Sélectionnez **Interactive Query** comme type de cluster.

## <a name="install-microsoft-hive-odbc-driver"></a>Installation du pilote ODBC Microsoft Hive

Téléchargez et installez le [pilote ODBC Microsoft Hive](https://www.microsoft.com/download/details.aspx?id=40886).

## <a name="create-apache-hive-odbc-data-source"></a>Créer une source de données ODBC Apache Hive

La procédure suivante explique comment créer une source de données ODBC Apache Hive.

1. Dans Windows, accédez à **Démarrer** > **Outils d’administration Windows** > **Sources de données ODBC (32 bits)/(64 bits)** .  La fenêtre **Administrateur de sources de données ODBC** s’ouvre.

    ![Administrateur de sources de données ODBC](./media/apache-hive-query-odbc-driver-powershell/hive-odbc-driver-dsn-setup.png "Configurer un DSN à l’aide de l’Administrateur de sources de données ODBC")

1. Dans l’onglet **DSN utilisateur**, sélectionnez **Ajouter** pour ouvrir la fenêtre **Créer une nouvelle source de données**.

1. Sélectionnez **Pilote ODBC Microsoft Hive**, puis **Terminer** pour ouvrir la fenêtre **Microsoft Hive ODBC Driver DSN Setup** (Configuration du DSN du pilote ODBC Microsoft Hive).

1. Tapez ou sélectionnez les valeurs suivantes :

   | Propriété | Description |
   | --- | --- |
   |  Nom de la source de données |Donnez un nom à votre source de données |
   |  Host(s) (Hôte(s)) |Entrez `CLUSTERNAME.azurehdinsight.net`. Par exemple : `myHDICluster.azurehdinsight.net` |
   |  Port |Utilisez **443**.|
   |  Base de données |Utilisez **Default**. |
   |  Mechanism |Sélectionnez **Windows Azure HDInsight Service**. |
   |  User Name |Entrez le nom de l’utilisateur HTTP du cluster HDInsight. Le nom d’utilisateur par défaut est **admin**. |
   |  Mot de passe |Entrez le mot de passe du cluster HDInsight. Cochez la case **Enregistrer le mot de passe (chiffré)** .|

1. Facultatif : Sélectionnez **Options avancées...**  

   | Paramètre | Description |
   | --- | --- |
   |  Use Native Query |Une fois sélectionné, le pilote ODBC ne tente PAS de convertir TSQL en HiveQL. Utilisez cette option uniquement si vous êtes sûr que vous soumettez des instructions HiveQL brutes à 100 %. Si vous effectuez une connexion à SQL Server ou Azure SQL Database, ne sélectionnez pas cette option. |
   |  Rows fetched per block |Lors de l’extraction d’un grand nombre d’enregistrements, la définition de ce paramètre peut être nécessaire pour garantir des performances optimales. |
   |  Default string column length, Binary column length, Decimal column scale |Les précisions et longueurs des types de données peuvent affecter la façon dont les données sont renvoyées. Elles entraînent le renvoi d’informations incorrectes en raison d’une perte de précision et/ou de troncations. |

    ![Options avancées de configuration DSN](./media/apache-hive-query-odbc-driver-powershell/odbc-data-source-advanced-options.png "Options avancées de configuration DSN")

1. Sélectionnez **Tester** pour tester la source de données. Une fois que la source de données est configurée correctement, le message suivant apparaît **SUCCESS!** .  

1. Sélectionnez **OK** pour fermer la fenêtre de test.  

1. Sélectionnez **OK** pour fermer la fenêtre **Microsoft Hive ODBC Driver DSN Setup** (Configuration du DSN du pilote ODBC Microsoft Hive).  

1. Sélectionnez **OK** pour fermer la fenêtre **Administrateur de sources de données ODBC**.  

## <a name="query-data-with-powershell"></a>Interroger des données avec PowerShell

Le script PowerShell suivant est une fonction qu’ODBC utilise pour interroger un cluster Hive.

```powershell
function Get-ODBC-Data {

   param(
   [string]$query=$(throw 'query is required.'),
   [string]$dsn,  
   [PSCredential] $cred = (Get-Credential)  
   )

   $conn = New-Object System.Data.Odbc.OdbcConnection
   $uname = $cred.UserName

   $pswd = (New-Object System.Net.NetworkCredential -ArgumentList "", $cred.Password).Password
   $conn.ConnectionString = "DSN=$dsn;Uid=$uname;Pwd=$pswd;"
   $conn.open()
   $cmd = New-object System.Data.Odbc.OdbcCommand($query,$conn)

   $ds = New-Object system.Data.DataSet

   (New-Object system.Data.odbc.odbcDataAdapter($cmd)).fill($ds) #| out-null
   $conn.close()
   $ds.Tables
}
```

L’extrait de code suivant utilise la fonction précédente pour exécuter une requête sur le cluster Interactive Query que vous avez créé au début du didacticiel. Remplacez `DATASOURCENAME` par le **nom de source de données** que vous avez spécifié dans la fenêtre **Microsoft Hive ODBC Driver DSN Setup** (Configuration du DSN du pilote ODBC Microsoft Hive). Lorsque vous êtes invité à saisir vos informations d’identification, entrez le nom d’utilisateur et le mot de passe que vous avez entrés sous **Nom de connexion au cluster** et **Mot de passe de connexion au cluster** lorsque vous avez créé le cluster.

```powershell

$dsn = "DATASOURCENAME"

$query = "select count(distinct clientid) AS total_clients from hivesampletable"

Get-ODBC-Data -query $query -dsn $dsn
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources, le cluster HDInsight et le compte de stockage. Pour ce faire, sélectionnez le groupe de ressources dans lequel le cluster a été créé, puis cliquez sur **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à utiliser le pilote ODBC Microsoft Hive et PowerShell pour récupérer des données à partir de votre cluster Azure HDInsight Interactive Query.

> [!div class="nextstepaction"]
> [Connecter Excel à Apache Hive à l’aide d’ODBC](../hadoop/apache-hadoop-connect-excel-hive-odbc-driver.md)

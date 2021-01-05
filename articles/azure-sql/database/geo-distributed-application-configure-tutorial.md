---
title: Implémenter une solution répartie géographiquement
description: Apprenez à configurer votre base de données dans Azure SQL Database et l'application cliente pour le basculement vers une base de données répliquée, puis à tester le basculement.
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.custom: sqldbrb=1, devx-track-azurecli, devx-track-azurepowershell
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: mathoma, sstein
ms.date: 03/12/2019
ms.openlocfilehash: 98e3eb4927b8eb9e52fd974c1ef7c417aff2ad54
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93422788"
---
# <a name="tutorial-implement-a-geo-distributed-database-azure-sql-database"></a>Tutoriel : Implémenter une base de données géo-distribuée (Azure SQL Database)
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Configurez une base de données SQL Database et une application cliente pour le basculement vers une région distante, puis testez votre plan de basculement. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
>
> - Créer un [groupe de basculement](auto-failover-group-overview.md)
> - Exécuter une application Java pour interroger une base de données dans SQL Database
> - Test de basculement

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager est toujours pris en charge par Azure SQL Database, mais tous les développements futurs sont destinés au module Az.Sql. Pour ces cmdlets, voir [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm.

Pour suivre le tutoriel, vérifiez que les éléments suivants sont installés :

- [Azure PowerShell](/powershell/azure/)
- Une base de données unique dans Azure SQL Database Pour en créer une :
  - [Le portail Azure](single-database-create-quickstart.md)
  - [L’interface de ligne de commande Microsoft Azure](az-cli-script-samples-content-guide.md)
  - [PowerShell](powershell-script-content-guide.md)

  > [!NOTE]
  > Ce tutoriel utilise l’exemple de base de données *AdventureWorksLT*.

- Java et Maven. Consultez [Générer une application à l’aide de SQL Server](https://www.microsoft.com/sql-server/developer-get-started/), mettez en surbrillance **Java** et sélectionnez votre environnement, puis suivez les étapes.

> [!IMPORTANT]
> Veillez à configurer des règles de pare-feu pour pouvoir utiliser l’adresse IP publique de l’ordinateur sur lequel vous effectuez les étapes de ce tutoriel. Les règles de pare-feu au niveau de la base de données sont répliquées automatiquement sur le serveur secondaire.
>
> Pour plus d’informations, consultez [Créer une règle de pare-feu au niveau du serveur](/sql/relational-databases/system-stored-procedures/sp-set-database-firewall-rule-azure-sql-database) ou, pour déterminer l’adresse IP utilisée pour la règle de pare-feu au niveau du serveur pour votre ordinateur, consultez [Créer une règle de pare-feu au niveau du serveur](firewall-create-server-level-portal-quickstart.md).  

## <a name="create-a-failover-group"></a>Créer un groupe de basculement

À l'aide d'Azure PowerShell, créez des [groupes de basculement](auto-failover-group-overview.md) entre un serveur existant et un nouveau serveur situé dans une autre région. Ajoutez ensuite l’exemple de base de données au groupe de basculement.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

> [!IMPORTANT]
> [!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

Pour créer un groupe de basculement, exécutez le script suivant :

```powershell
$admin = "<adminName>"
$password = "<password>"
$resourceGroup = "<resourceGroupName>"
$location = "<resourceGroupLocation>"
$server = "<serverName>"
$database = "<databaseName>"
$drLocation = "<disasterRecoveryLocation>"
$drServer = "<disasterRecoveryServerName>"
$failoverGroup = "<globallyUniqueFailoverGroupName>"

# create a backup server in the failover region
New-AzSqlServer -ResourceGroupName $resourceGroup -ServerName $drServer `
    -Location $drLocation -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential `
    -ArgumentList $admin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))

# create a failover group between the servers
New-AzSqlDatabaseFailoverGroup –ResourceGroupName $resourceGroup -ServerName $server `
    -PartnerServerName $drServer –FailoverGroupName $failoverGroup –FailoverPolicy Automatic -GracePeriodWithDataLossHours 2

# add the database to the failover group
Get-AzSqlDatabase -ResourceGroupName $resourceGroup -ServerName $server -DatabaseName $database | `
    Add-AzSqlDatabaseToFailoverGroup -ResourceGroupName $resourceGroup -ServerName $server -FailoverGroupName $failoverGroup
```

# <a name="the-azure-cli"></a>[L’interface de ligne de commande Microsoft Azure](#tab/azure-cli)

> [!IMPORTANT]
> Exécutez `az login` pour vous connecter à Azure.

```azurecli
$admin = "<adminName>"
$password = "<password>"
$resourceGroup = "<resourceGroupName>"
$location = "<resourceGroupLocation>"
$server = "<serverName>"
$database = "<databaseName>"
$drLocation = "<disasterRecoveryLocation>" # must be different then $location
$drServer = "<disasterRecoveryServerName>"
$failoverGroup = "<globallyUniqueFailoverGroupName>"

# create a backup server in the failover region
az sql server create --admin-password $password --admin-user $admin `
    --name $drServer --resource-group $resourceGroup --location $drLocation

# create a failover group between the servers
az sql failover-group create --name $failoverGroup --partner-server $drServer `
    --resource-group $resourceGroup --server $server --add-db $database `
    --failover-policy Automatic --grace-period 2
```

* * *

Vous pouvez également changer les paramètres de géoréplication dans le portail Azure. Pour cela, sélectionnez votre base de données, puis **Paramètres** > **Géoréplication**.

![Paramètres de géoréplication](./media/geo-distributed-application-configure-tutorial/geo-replication.png)

## <a name="run-the-sample-project"></a>Exécuter l’exemple de projet

1. Dans la console, créez un projet Maven avec la commande suivante :

   ```bash
   mvn archetype:generate "-DgroupId=com.sqldbsamples" "-DartifactId=SqlDbSample" "-DarchetypeArtifactId=maven-archetype-quickstart" "-Dversion=1.0.0"
   ```

1. Tapez **Y** , puis appuyez sur **Entrée**.

1. Passez au répertoire du nouveau projet.

   ```bash
   cd SqlDbSample
   ```

1. À l’aide de votre éditeur habituel, ouvrez le fichier *pom.xml* dans le dossier de votre projet.

1. Ajoutez la dépendance Pilote Microsoft JDBC pour SQL Server en ajoutant la section `dependency` suivante. La dépendance doit être collée dans la section `dependencies` plus grande.

   ```xml
   <dependency>
     <groupId>com.microsoft.sqlserver</groupId>
     <artifactId>mssql-jdbc</artifactId>
    <version>6.1.0.jre8</version>
   </dependency>
   ```

1. Spécifiez la version de Java en ajoutant la section `properties` après la section `dependencies` :

   ```xml
   <properties>
     <maven.compiler.source>1.8</maven.compiler.source>
     <maven.compiler.target>1.8</maven.compiler.target>
   </properties>
   ```

1. Assurez la prise en charge des fichiers manifeste en ajoutant la section `build` après la section `properties` :

   ```xml
   <build>
     <plugins>
       <plugin>
         <groupId>org.apache.maven.plugins</groupId>
         <artifactId>maven-jar-plugin</artifactId>
         <version>3.0.0</version>
         <configuration>
           <archive>
             <manifest>
               <mainClass>com.sqldbsamples.App</mainClass>
             </manifest>
           </archive>
        </configuration>
       </plugin>
     </plugins>
   </build>
   ```

1. Enregistrez et fermez le fichier *pom.xml*.

1. Ouvrez le fichier *App.java* situé à l’emplacement ..\SqlDbSample\src\main\java\com\sqldbsamples et remplacez son contenu par le code suivant :

   ```java
   package com.sqldbsamples;

   import java.sql.Connection;
   import java.sql.Statement;
   import java.sql.PreparedStatement;
   import java.sql.ResultSet;
   import java.sql.Timestamp;
   import java.sql.DriverManager;
   import java.util.Date;
   import java.util.concurrent.TimeUnit;

   public class App {

      private static final String FAILOVER_GROUP_NAME = "<your failover group name>";  // add failover group name
  
      private static final String DB_NAME = "<your database>";  // add database name
      private static final String USER = "<your admin>";  // add database user
      private static final String PASSWORD = "<your password>";  // add database password

      private static final String READ_WRITE_URL = String.format("jdbc:" +
         "sqlserver://%s.database.windows.net:1433;database=%s;user=%s;password=%s;encrypt=true;" +
         "hostNameInCertificate=*.database.windows.net;loginTimeout=30;", +
         FAILOVER_GROUP_NAME, DB_NAME, USER, PASSWORD);
      private static final String READ_ONLY_URL = String.format("jdbc:" +
         "sqlserver://%s.secondary.database.windows.net:1433;database=%s;user=%s;password=%s;encrypt=true;" +
         "hostNameInCertificate=*.database.windows.net;loginTimeout=30;", +
         FAILOVER_GROUP_NAME, DB_NAME, USER, PASSWORD);

      public static void main(String[] args) {
         System.out.println("#######################################");
         System.out.println("## GEO DISTRIBUTED DATABASE TUTORIAL ##");
         System.out.println("#######################################");
         System.out.println("");

         int highWaterMark = getHighWaterMarkId();

         try {
            for(int i = 1; i < 1000; i++) {
                //  loop will run for about 1 hour
                System.out.print(i + ": insert on primary " +
                   (insertData((highWaterMark + i))?"successful":"failed"));
                TimeUnit.SECONDS.sleep(1);
                System.out.print(", read from secondary " +
                   (selectData((highWaterMark + i))?"successful":"failed") + "\n");
                TimeUnit.SECONDS.sleep(3);
            }
         } catch(Exception e) {
            e.printStackTrace();
      }
   }

   private static boolean insertData(int id) {
      // Insert data into the product table with a unique product name so we can find the product again
      String sql = "INSERT INTO SalesLT.Product " +
         "(Name, ProductNumber, Color, StandardCost, ListPrice, SellStartDate) VALUES (?,?,?,?,?,?);";

      try (Connection connection = DriverManager.getConnection(READ_WRITE_URL);
              PreparedStatement pstmt = connection.prepareStatement(sql)) {
         pstmt.setString(1, "BrandNewProduct" + id);
         pstmt.setInt(2, 200989 + id + 10000);
         pstmt.setString(3, "Blue");
         pstmt.setDouble(4, 75.00);
         pstmt.setDouble(5, 89.99);
         pstmt.setTimestamp(6, new Timestamp(new Date().getTime()));
         return (1 == pstmt.executeUpdate());
      } catch (Exception e) {
         return false;
      }
   }

   private static boolean selectData(int id) {
      // Query the data previously inserted into the primary database from the geo replicated database
      String sql = "SELECT Name, Color, ListPrice FROM SalesLT.Product WHERE Name = ?";

      try (Connection connection = DriverManager.getConnection(READ_ONLY_URL);
              PreparedStatement pstmt = connection.prepareStatement(sql)) {
         pstmt.setString(1, "BrandNewProduct" + id);
         try (ResultSet resultSet = pstmt.executeQuery()) {
            return resultSet.next();
         }
      } catch (Exception e) {
         return false;
      }
   }

   private static int getHighWaterMarkId() {
      // Query the high water mark id stored in the table to be able to make unique inserts
      String sql = "SELECT MAX(ProductId) FROM SalesLT.Product";
      int result = 1;
      try (Connection connection = DriverManager.getConnection(READ_WRITE_URL);
              Statement stmt = connection.createStatement();
              ResultSet resultSet = stmt.executeQuery(sql)) {
         if (resultSet.next()) {
             result =  resultSet.getInt(1);
            }
         } catch (Exception e) {
          e.printStackTrace();
         }
         return result;
      }
   }
   ```

1. Enregistrez et fermez le fichier *App.java*.

1. Dans la console de commandes, exécutez la commande suivante :

   ```bash
   mvn package
   ```

1. Démarrez l’application. Celle-ci s’exécute pendant environ 1 heure jusqu’à ce que vous l’arrêtiez manuellement, ce qui vous donne le temps d’exécuter le test de basculement.

   ```bash
   mvn -q -e exec:java "-Dexec.mainClass=com.sqldbsamples.App"
   ```

   ```output
   #######################################
   ## GEO DISTRIBUTED DATABASE TUTORIAL ##
   #######################################

   1. insert on primary successful, read from secondary successful
   2. insert on primary successful, read from secondary successful
   3. insert on primary successful, read from secondary successful
   ...
   ```

## <a name="test-failover"></a>Test de basculement

Exécutez les scripts suivants pour simuler un basculement et observer les résultats de l’application. Notez l’échec de certaines insertions et sélections durant la migration de base de données.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Vous pouvez vérifier le rôle du serveur de reprise d’activité pendant le test avec la commande suivante :

```powershell
(Get-AzSqlDatabaseFailoverGroup -FailoverGroupName $failoverGroup `
    -ResourceGroupName $resourceGroup -ServerName $drServer).ReplicationRole
```

Pour tester un basculement :

1. Démarrer un basculement manuel du groupe de basculement :

   ```powershell
   Switch-AzSqlDatabaseFailoverGroup -ResourceGroupName $resourceGroup `
    -ServerName $drServer -FailoverGroupName $failoverGroup
   ```

1. Rétablir le groupe de basculement sur le serveur principal :

   ```powershell
   Switch-AzSqlDatabaseFailoverGroup -ResourceGroupName $resourceGroup `
    -ServerName $server -FailoverGroupName $failoverGroup
   ```

# <a name="the-azure-cli"></a>[L’interface de ligne de commande Microsoft Azure](#tab/azure-cli)

Vous pouvez vérifier le rôle du serveur de reprise d’activité pendant le test avec la commande suivante :

```azurecli
az sql failover-group show --name $failoverGroup --resource-group $resourceGroup --server $drServer
```

Pour tester un basculement :

1. Démarrer un basculement manuel du groupe de basculement :

   ```azurecli
   az sql failover-group set-primary --name $failoverGroup --resource-group $resourceGroup --server $drServer
   ```

1. Rétablir le groupe de basculement sur le serveur principal :

   ```azurecli
   az sql failover-group set-primary --name $failoverGroup --resource-group $resourceGroup --server $server
   ```

* * *

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez configuré une base de données dans Azure SQL Database et une application pour le basculement vers une région distante, puis testé votre plan de basculement. Vous avez appris à :

> [!div class="checklist"]
>
> - Créer un groupe de basculement de géoréplication
> - Exécuter une application Java pour interroger une base de données dans SQL Database
> - Test de basculement

Passez au didacticiel suivant pour apprendre à ajouter une instance d’Azure SQL Managed Instance à un groupe de basculement :

> [!div class="nextstepaction"]
> [Ajouter une instance d’Azure SQL Managed Instance à un groupe de basculement](../managed-instance/failover-group-add-instance-tutorial.md)

---
title: Utiliser Apache Maven pour construire un client Java HBase pour Azure HDInsight
description: Découvrez comment utiliser Apache Maven pour créer une application Apache HBase basées sur Java, puis la déployer vers HBase sur Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seodec18, devx-track-java
ms.date: 12/24/2019
ms.openlocfilehash: 61b8aaf9ea61ebe85eac6708d7390c386dea2696
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92547909"
---
# <a name="build-java-applications-for-apache-hbase"></a>Créer des applications Java pour Apache HBase

Découvrez comment créer une application [Apache HBase](https://hbase.apache.org/) en Java. Utilisez ensuite l’application avec HBase sur Azure HDInsight.

La procédure décrite dans ce document utilise [Apache Maven](https://maven.apache.org/) pour créer et générer le projet. Maven est un outil de gestion de projets logiciels et d’inclusion qui vous permet de créer des logiciels, de la documentation et des rapports pour des projets Java.

## <a name="prerequisites"></a>Prérequis

* Un cluster Apache HBase sur HDInsight. Consultez [Bien démarrer avec Apache HBase](./apache-hbase-tutorial-get-started-linux.md).

* [Kit de développeur Java (JDK), version 8](/azure/developer/java/fundamentals/java-jdk-long-term-support).

* [Apache Maven](https://maven.apache.org/download.cgi) correctement [installé](https://maven.apache.org/install.html) en fonction d’Apache.  Maven est un système de génération de projet pour les projets Java.

* Un client SSH. Pour plus d’informations, consultez [Se connecter à HDInsight (Apache Hadoop) à l’aide de SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

* Si vous utilisez PowerShell, vous aurez besoin du [module AZ](/powershell/azure/).

* Un éditeur de texte Cet article utilise Microsoft Notepad.

## <a name="test-environment"></a>Environnement de test

L’environnement utilisé pour cet article était un ordinateur exécutant Windows 10.  Les commandes ont été exécutées dans une invite de commandes, et les différents fichiers ont été modifiés avec le bloc-notes. Modifiez en conséquence pour votre environnement.

Dans une invite de commandes, entrez les commandes suivantes afin de créer un environnement de travail :

```cmd
IF NOT EXIST C:\HDI MKDIR C:\HDI
cd C:\HDI
```

## <a name="create-a-maven-project"></a>Création d’un projet Maven

1. Entrez la commande suivante pour créer un projet Maven nommé **hbaseapp**  :

    ```cmd
    mvn archetype:generate -DgroupId=com.microsoft.examples -DartifactId=hbaseapp -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

    cd hbaseapp
    mkdir conf
    ```

    Cette commande crée un répertoire nommé `hbaseapp` à l’emplacement actuel, qui contient un projet Maven de base. La deuxième commande définit le répertoire de travail sur `hbaseapp`. La troisième commande crée le répertoire `conf`, qui sera utilisé ultérieurement. Le répertoire `hbaseapp` contient les éléments suivants :

    * `pom.xml`:  le modèle d’objet du projet ([POM](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html)) contient les informations et la configuration utilisées pour générer le projet.
    * `src\main\java\com\microsoft\examples`: contient votre code d’application.
    * `src\test\java\com\microsoft\examples`: contient des tests pour votre application.

2. Supprimez l’exemple de code généré. Supprimez les fichiers d’application et de test générés `AppTest.java` et `App.java` en entrant les commandes ci-dessous :

    ```cmd
    DEL src\main\java\com\microsoft\examples\App.java
    DEL src\test\java\com\microsoft\examples\AppTest.java
    ```

## <a name="update-the-project-object-model"></a>Mise à jour du modèle d'objet du projet

Pour bénéficier d’une référence complète du fichier pom.xml, accédez à l’adresse https://maven.apache.org/pom.html.  Ouvrez `pom.xml` en entrant la commande ci-dessous :

```cmd
notepad pom.xml
```

### <a name="add-dependencies"></a>Ajout de dépendances

Dans `pom.xml`, ajoutez le texte suivant dans la section `<dependencies>` :

```xml
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-shaded-client</artifactId>
    <version>1.1.2</version>
</dependency>
<dependency>
    <groupId>org.apache.phoenix</groupId>
    <artifactId>phoenix-core</artifactId>
    <version>4.14.1-HBase-1.1</version>
</dependency>
```  

Cette section indique que le projet a besoin des composants **hbase-client** et **phoenix-core** . Au moment de la compilation, ces dépendances sont téléchargées à partir du référentiel Maven par défaut. Vous pouvez utiliser la [Recherche du référentiel central Maven](https://search.maven.org/artifact/org.apache.hbase/hbase-client/1.1.2/jar) pour en savoir plus sur cette dépendance.

> [!IMPORTANT]  
> Le numéro de version du client hbase doit correspondre à la version d’Apache HBase fournie avec votre cluster HDInsight. Utilisez le tableau suivant pour trouver le numéro de version correct.

| Version de cluster HDInsight | Version Apache HBase à utiliser |
| --- | --- |
| 3.6 | 1.1.2 |
| 4.0 | 2.0.0 |

Pour plus d’informations sur les versions et composants HDInsight, voir [Quels sont les différents composants Apache Hadoop disponibles avec HDInsight ?](../hdinsight-component-versioning.md)

### <a name="build-configuration"></a>Configuration de build

Les plug-ins Maven permettent de personnaliser les étapes de génération du projet, Cette section est utilisée pour ajouter des plug-ins, des ressources et d’autres options de configuration de build.

Ajoutez le code suivant au fichier `pom.xml`, puis enregistrez celui-ci. Ce texte doit être contenu entre les balises `<project>...</project>` dans le fichier, par exemple entre `</dependencies>` et `</project>`.

```xml
<build>
    <sourceDirectory>src</sourceDirectory>
    <resources>
    <resource>
        <directory>${basedir}/conf</directory>
        <filtering>false</filtering>
        <includes>
        <include>hbase-site.xml</include>
        </includes>
    </resource>
    </resources>
    <plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
        <configuration>
            <source>1.8</source>
            <target>1.8</target>
        </configuration>
        </plugin>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>3.2.1</version>
        <configuration>
        <transformers>
            <transformer implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer">
            </transformer>
        </transformers>
        </configuration>
        <executions>
        <execution>
            <phase>package</phase>
            <goals>
            <goal>shade</goal>
            </goals>
        </execution>
        </executions>
    </plugin>
    </plugins>
</build>
```

Cette section configure une ressource (`conf/hbase-site.xml`) contenant des informations de configuration pour HBase.

> [!NOTE]  
> Vous pouvez également définir les valeurs de configuration par code. Consultez les commentaires dans l’exemple `CreateTable`.

Cette section configure également les plug-ins [Apache Maven Compiler Plugin](https://maven.apache.org/plugins/maven-compiler-plugin/) et [Apache Maven Shade Plugin](https://maven.apache.org/plugins/maven-shade-plugin/). Le plug-in compiler est utilisé pour compiler la topologie. Le plug-in shade est utilisé pour empêcher la duplication de licence dans le package JAR généré par Maven. Ce plug-in est utilisé pour prévenir l’erreur relative aux fichiers de licence dupliqués lors de l’exécution sur le cluster HDInsight. Le fait d’utiliser le plug-in maven-shade-plugin avec l’implémentation `ApacheLicenseResourceTransformer` permet d’éviter cette erreur.

Le plug-in maven-shade-plugin produit également un uber jar contenant toutes les dépendances requises par l'application.

### <a name="download-the-hbase-sitexml"></a>Télécharger le fichier hbase-site.xml

Utilisez la commande suivante pour copier la configuration de HBase à partir du cluster HBase vers le répertoire `conf`. Remplacez `CLUSTERNAME` par le nom de votre cluster HDInsight, puis entrez la commande :

```cmd
scp sshuser@CLUSTERNAME-ssh.azurehdinsight.net:/etc/hbase/conf/hbase-site.xml ./conf/hbase-site.xml
```

## <a name="create-the-application"></a>Création de l'application

### <a name="implement-a-createtable-class"></a>Implémenter une classe CreateTable

Entrez la commande ci-dessous pour créer et ouvrir un nouveau fichier `CreateTable.java`. Sélectionnez **Oui** à l'invite pour créer un fichier.

```cmd
notepad src\main\java\com\microsoft\examples\CreateTable.java
```

Copiez et collez ensuite le code Java ci-dessous dans le nouveau fichier. Fermez le fichier.

```java
package com.microsoft.examples;
import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.HBaseAdmin;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.client.HTable;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.util.Bytes;

public class CreateTable {
    public static void main(String[] args) throws IOException {
    Configuration config = HBaseConfiguration.create();

    // Example of setting zookeeper values for HDInsight
    // in code instead of an hbase-site.xml file
    //
    // config.set("hbase.zookeeper.quorum",
    //            "zookeepernode0,zookeepernode1,zookeepernode2");
    //config.set("hbase.zookeeper.property.clientPort", "2181");
    //config.set("hbase.cluster.distributed", "true");
    //
    //NOTE: Actual zookeeper host names can be found using Ambari:
    //curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/hosts"

    //Linux-based HDInsight clusters use /hbase-unsecure as the znode parent
    config.set("zookeeper.znode.parent","/hbase-unsecure");

    // create an admin object using the config
    HBaseAdmin admin = new HBaseAdmin(config);

    // create the table...
    HTableDescriptor tableDescriptor = new HTableDescriptor(TableName.valueOf("people"));
    // ... with two column families
    tableDescriptor.addFamily(new HColumnDescriptor("name"));
    tableDescriptor.addFamily(new HColumnDescriptor("contactinfo"));
    admin.createTable(tableDescriptor);

    // define some people
    String[][] people = {
        { "1", "Marcel", "Haddad", "marcel@fabrikam.com"},
        { "2", "Franklin", "Holtz", "franklin@contoso.com" },
        { "3", "Dwayne", "McKee", "dwayne@fabrikam.com" },
        { "4", "Rae", "Schroeder", "rae@contoso.com" },
        { "5", "Rosalie", "burton", "rosalie@fabrikam.com"},
        { "6", "Gabriela", "Ingram", "gabriela@contoso.com"} };

    HTable table = new HTable(config, "people");

    // Add each person to the table
    //   Use the `name` column family for the name
    //   Use the `contactinfo` column family for the email
    for (int i = 0; i< people.length; i++) {
        Put person = new Put(Bytes.toBytes(people[i][0]));
        person.add(Bytes.toBytes("name"), Bytes.toBytes("first"), Bytes.toBytes(people[i][1]));
        person.add(Bytes.toBytes("name"), Bytes.toBytes("last"), Bytes.toBytes(people[i][2]));
        person.add(Bytes.toBytes("contactinfo"), Bytes.toBytes("email"), Bytes.toBytes(people[i][3]));
        table.put(person);
    }
    // flush commits and close the table
    table.flushCommits();
    table.close();
    }
}
```

Ce code correspond à la classe `CreateTable`, qui crée une table appelée `people` et la remplit avec des utilisateurs prédéfinis.

### <a name="implement-a-searchbyemail-class"></a>Implémenter une classe SearchByEmail

Entrez la commande ci-dessous pour créer et ouvrir un nouveau fichier `SearchByEmail.java`. Sélectionnez **Oui** à l'invite pour créer un fichier.

```cmd
notepad src\main\java\com\microsoft\examples\SearchByEmail.java
```

Copiez et collez ensuite le code Java ci-dessous dans le nouveau fichier. Fermez le fichier.

```java
package com.microsoft.examples;
import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.HTable;
import org.apache.hadoop.hbase.client.Scan;
import org.apache.hadoop.hbase.client.ResultScanner;
import org.apache.hadoop.hbase.client.Result;
import org.apache.hadoop.hbase.filter.RegexStringComparator;
import org.apache.hadoop.hbase.filter.SingleColumnValueFilter;
import org.apache.hadoop.hbase.filter.CompareFilter.CompareOp;
import org.apache.hadoop.hbase.util.Bytes;
import org.apache.hadoop.util.GenericOptionsParser;

public class SearchByEmail {
    public static void main(String[] args) throws IOException {
    Configuration config = HBaseConfiguration.create();

    // Use GenericOptionsParser to get only the parameters to the class
    // and not all the parameters passed (when using WebHCat for example)
    String[] otherArgs = new GenericOptionsParser(config, args).getRemainingArgs();
    if (otherArgs.length != 1) {
        System.out.println("usage: [regular expression]");
        System.exit(-1);
    }

    // Open the table
    HTable table = new HTable(config, "people");

    // Define the family and qualifiers to be used
    byte[] contactFamily = Bytes.toBytes("contactinfo");
    byte[] emailQualifier = Bytes.toBytes("email");
    byte[] nameFamily = Bytes.toBytes("name");
    byte[] firstNameQualifier = Bytes.toBytes("first");
    byte[] lastNameQualifier = Bytes.toBytes("last");

    // Create a regex filter
    RegexStringComparator emailFilter = new RegexStringComparator(otherArgs[0]);
    // Attach the regex filter to a filter
    //   for the email column
    SingleColumnValueFilter filter = new SingleColumnValueFilter(
        contactFamily,
        emailQualifier,
        CompareOp.EQUAL,
        emailFilter
    );

    // Create a scan and set the filter
    Scan scan = new Scan();
    scan.setFilter(filter);

    // Get the results
    ResultScanner results = table.getScanner(scan);
    // Iterate over results and print  values
    for (Result result : results ) {
        String id = new String(result.getRow());
        byte[] firstNameObj = result.getValue(nameFamily, firstNameQualifier);
        String firstName = new String(firstNameObj);
        byte[] lastNameObj = result.getValue(nameFamily, lastNameQualifier);
        String lastName = new String(lastNameObj);
        System.out.println(firstName + " " + lastName + " - ID: " + id);
        byte[] emailObj = result.getValue(contactFamily, emailQualifier);
        String email = new String(emailObj);
        System.out.println(firstName + " " + lastName + " - " + email + " - ID: " + id);
    }
    results.close();
    table.close();
    }
}
```

La classe `SearchByEmail` peut être utilisée pour lancer des requêtes sur les lignes d’adresses électroniques. Puisqu'elle utilise un filtre d'expression régulière, vous pouvez fournir une chaîne ou une expression régulière lorsque vous utilisez la classe.

### <a name="implement-a-deletetable-class"></a>Implémenter une classe DeleteTable

Entrez la commande ci-dessous pour créer et ouvrir un nouveau fichier `DeleteTable.java`. Sélectionnez **Oui** à l'invite pour créer un fichier.

```cmd
notepad src\main\java\com\microsoft\examples\DeleteTable.java
```

Copiez et collez ensuite le code Java ci-dessous dans le nouveau fichier. Fermez le fichier.

```java
package com.microsoft.examples;
import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.HBaseAdmin;

public class DeleteTable {
    public static void main(String[] args) throws IOException {
    Configuration config = HBaseConfiguration.create();

    // Create an admin object using the config
    HBaseAdmin admin = new HBaseAdmin(config);

    // Disable, and then delete the table
    admin.disableTable("people");
    admin.deleteTable("people");
    }
}
```

La classe `DeleteTable` nettoie les tables HBase créées dans cet exemple en désactivant puis en supprimant la table créée par la classe `CreateTable`.

## <a name="build-and-package-the-application"></a>Génération et package de l'application

1. À partir du répertoire `hbaseapp`, utilisez la commande suivante pour générer un fichier jar contenant l’application :

    ```cmd
    mvn clean package
    ```

    Cette commande génère et empaquète l’application dans un fichier .jar.

2. Une fois la commande exécutée, le répertoire `hbaseapp/target` contient un fichier nommé `hbaseapp-1.0-SNAPSHOT.jar`.

   > [!NOTE]  
   > Le fichier `hbaseapp-1.0-SNAPSHOT.jar` est un fichier uber jar. Il contient toutes les dépendances requises pour exécuter l’application.

## <a name="upload-the-jar-and-run-jobs-ssh"></a>Téléchargement du fichier JAR et exécution des travaux (SSH)

Les étapes suivantes utilisent `scp` pour copier le fichier jar sur le nœud principal d’Apache HBase sur le cluster HDInsight. La commande `ssh` est ensuite utilisée pour se connecter au cluster et exécuter l’exemple directement sur le nœud principal.

1. Chargez le fichier jar dans le cluster. Remplacez `CLUSTERNAME` par le nom de votre cluster HDInsight, puis entrez la commande suivante :

    ```cmd
    scp ./target/hbaseapp-1.0-SNAPSHOT.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net:hbaseapp-1.0-SNAPSHOT.jar
    ```

2. Connectez-vous au cluster HBase. Remplacez `CLUSTERNAME` par le nom de votre cluster HDInsight, puis entrez la commande suivante :

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

3. Pour créer une nouvelle table HBase à l’aide de l’application Java, utilisez la commande suivante dans votre connexion SSH ouverte :

    ```bash
    yarn jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.CreateTable
    ```

    Cette commande crée une table HBase nommée **people** la remplit de données.

4. Pour rechercher des adresses e-mail stockées dans la table, utilisez la commande suivante :

    ```bash
    yarn jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.SearchByEmail contoso.com
    ```

    Vous obtenez les résultats suivants :

    ```console
    Franklin Holtz - ID: 2
    Franklin Holtz - franklin@contoso.com - ID: 2
    Rae Schroeder - ID: 4
    Rae Schroeder - rae@contoso.com - ID: 4
    Gabriela Ingram - ID: 6
    Gabriela Ingram - gabriela@contoso.com - ID: 6
    ```

5. Utilisez la commande suivante pour supprimer le tableau :

    ```bash
    yarn jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.DeleteTable
    ```

## <a name="upload-the-jar-and-run-jobs-powershell"></a>Téléchargement du fichier JAR et exécution des travaux (PowerShell)

Les étapes suivantes utilisent le [module AZ](/powershell/azure/new-azureps-module-az) Azure PowerShell pour charger le fichier jar dans le stockage par défaut de votre cluster Apache HBase. Des applets de commande HDInsight sont ensuite utilisés pour exécuter les exemples à distance.

1. Après avoir installé et configuré le module AZ, créez un fichier nommé `hbase-runner.psm1`. Utilisez le texte suivant comme contenu de ce fichier :

   ```powershell
    <#
    .SYNOPSIS
    Copies a file to the primary storage of an HDInsight cluster.
    .DESCRIPTION
    Copies a file from a local directory to the blob container for
    the HDInsight cluster.
    .EXAMPLE
    Start-HBaseExample -className "com.microsoft.examples.CreateTable"
    -clusterName "MyHDInsightCluster"

    .EXAMPLE
    Start-HBaseExample -className "com.microsoft.examples.SearchByEmail"
    -clusterName "MyHDInsightCluster"
    -emailRegex "contoso.com"

    .EXAMPLE
    Start-HBaseExample -className "com.microsoft.examples.SearchByEmail"
    -clusterName "MyHDInsightCluster"
    -emailRegex "^r" -showErr
    #>

    function Start-HBaseExample {
    [CmdletBinding(SupportsShouldProcess = $true)]
    param(
    #The class to run
    [Parameter(Mandatory = $true)]
    [String]$className,

    #The name of the HDInsight cluster
    [Parameter(Mandatory = $true)]
    [String]$clusterName,

    #Only used when using SearchByEmail
    [Parameter(Mandatory = $false)]
    [String]$emailRegex,

    #Use if you want to see stderr output
    [Parameter(Mandatory = $false)]
    [Switch]$showErr
    )

    Set-StrictMode -Version 3

    # Is the Azure module installed?
    FindAzure

    # Get the login for the HDInsight cluster
    $creds=Get-Credential -Message "Enter the login for the cluster" -UserName "admin"

    # The JAR
    $jarFile = "wasb:///example/jars/hbaseapp-1.0-SNAPSHOT.jar"

    # The job definition
    $jobDefinition = New-AzHDInsightMapReduceJobDefinition `
        -JarFile $jarFile `
        -ClassName $className `
        -Arguments $emailRegex

    # Get the job output
    $job = Start-AzHDInsightJob `
        -ClusterName $clusterName `
        -JobDefinition $jobDefinition `
        -HttpCredential $creds
    Write-Host "Wait for the job to complete ..." -ForegroundColor Green
    Wait-AzHDInsightJob `
        -ClusterName $clusterName `
        -JobId $job.JobId `
        -HttpCredential $creds
    if($showErr)
    {
    Write-Host "STDERR"
    Get-AzHDInsightJobOutput `
                -Clustername $clusterName `
                -JobId $job.JobId `
                -HttpCredential $creds `
                -DisplayOutputType StandardError
    }
    Write-Host "Display the standard output ..." -ForegroundColor Green
    Get-AzHDInsightJobOutput `
                -Clustername $clusterName `
                -JobId $job.JobId `
                -HttpCredential $creds
    }

    <#
    .SYNOPSIS
    Copies a file to the primary storage of an HDInsight cluster.
    .DESCRIPTION
    Copies a file from a local directory to the blob container for
    the HDInsight cluster.
    .EXAMPLE
    Add-HDInsightFile -localPath "C:\temp\data.txt"
    -destinationPath "example/data/data.txt"
    -ClusterName "MyHDInsightCluster"
    .EXAMPLE
    Add-HDInsightFile -localPath "C:\temp\data.txt"
    -destinationPath "example/data/data.txt"
    -ClusterName "MyHDInsightCluster"
    -Container "MyContainer"
    #>

    function Add-HDInsightFile {
        [CmdletBinding(SupportsShouldProcess = $true)]
        param(
            #The path to the local file.
            [Parameter(Mandatory = $true)]
            [String]$localPath,

            #The destination path and file name, relative to the root of the container.
            [Parameter(Mandatory = $true)]
            [String]$destinationPath,

            #The name of the HDInsight cluster
            [Parameter(Mandatory = $true)]
            [String]$clusterName,

            #If specified, overwrites existing files without prompting
            [Parameter(Mandatory = $false)]
            [Switch]$force
        )

        Set-StrictMode -Version 3

        # Is the Azure module installed?
        FindAzure

        # Get authentication for the cluster
        $creds=Get-Credential

        # Does the local path exist?
        if (-not (Test-Path $localPath))
        {
            throw "Source path '$localPath' does not exist."
        }

        # Get the primary storage container
        $storage = GetStorage -clusterName $clusterName

        # Upload file to storage, overwriting existing files if -force was used.
        Set-AzStorageBlobContent -File $localPath `
            -Blob $destinationPath `
            -force:$force `
            -Container $storage.container `
            -Context $storage.context
    }

    function FindAzure {
        # Is there an active Azure subscription?
        $sub = Get-AzSubscription -ErrorAction SilentlyContinue
        if(-not($sub))
        {
            Connect-AzAccount
        }
    }

    function GetStorage {
        param(
            [Parameter(Mandatory = $true)]
            [String]$clusterName
        )
        $hdi = Get-AzHDInsightCluster -ClusterName $clusterName
        # Does the cluster exist?
        if (!$hdi)
        {
            throw "HDInsight cluster '$clusterName' does not exist."
        }
        # Create a return object for context & container
        $return = @{}
        $storageAccounts = @{}

        # Get storage information
        $resourceGroup = $hdi.ResourceGroup
        $storageAccountName=$hdi.DefaultStorageAccount.split('.')[0]
        $container=$hdi.DefaultStorageContainer
        $storageAccountKey=(Get-AzStorageAccountKey `
            -Name $storageAccountName `
        -ResourceGroupName $resourceGroup)[0].Value
        # Get the resource group, in case we need that
        $return.resourceGroup = $resourceGroup
        # Get the storage context, as we can't depend
        # on using the default storage context
        $return.context = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
        # Get the container, so we know where to
        # find/store blobs
        $return.container = $container
        # Return storage accounts to support finding all accounts for
        # a cluster
        $return.storageAccount = $storageAccountName
        $return.storageAccountKey = $storageAccountKey

        return $return
    }
    # Only export the verb-phrase things
    export-modulemember *-*
   ```

    Ce fichier contient deux modules :

   * **Add-HDInsightFile** permet de charger des fichiers dans le cluster
   * **Start-HBaseExample** permet d’exécuter les classes créées antérieurement

2. Enregistrez le fichier `hbase-runner.psm1` dans le répertoire `hbaseapp`.

3. Inscrivez les modules auprès d’Azure PowerShell. Ouvrez une nouvelle fenêtre PowerShell Azure et modifiez la commande ci-dessous en remplaçant `CLUSTERNAME` par le nom de votre cluster. Puis, entrez les commandes suivantes :

    ```powershell
    cd C:\HDI\hbaseapp
    $myCluster = "CLUSTERNAME"
    Import-Module .\hbase-runner.psm1
    ```

4. Utilisez la commande suivante pour charger le `hbaseapp-1.0-SNAPSHOT.jar` dans votre cluster.

    ```powershell
    Add-HDInsightFile -localPath target\hbaseapp-1.0-SNAPSHOT.jar -destinationPath example/jars/hbaseapp-1.0-SNAPSHOT.jar -clusterName $myCluster
    ```

    Lorsque vous y êtes invité, entrez le nom de connexion (admin) et le mot de passe du cluster. La commande charge le `hbaseapp-1.0-SNAPSHOT.jar` dans l’emplacement `example/jars` dans le stockage principal de votre cluster.

5. Pour créer une table à l’aide de la `hbaseapp`, utilisez la commande suivante :

    ```powershell
    Start-HBaseExample -className com.microsoft.examples.CreateTable -clusterName $myCluster
    ```

    Lorsque vous y êtes invité, entrez le nom de connexion (admin) et le mot de passe du cluster.

    Cette commande crée une table nommée **people** dans HBase sur votre cluster HDInsight. Cette commande n’affiche aucune sortie dans la fenêtre de console.

6. Pour rechercher des entrées dans la table, utilisez la commande suivante :

    ```powershell
    Start-HBaseExample -className com.microsoft.examples.SearchByEmail -clusterName $myCluster -emailRegex contoso.com
    ```

    Lorsque vous y êtes invité, entrez le nom de connexion (admin) et le mot de passe du cluster.

    Cette commande utilise la classe `SearchByEmail` pour rechercher les lignes où la famille de colonnes `contactinformation` et la colonne `email` contiennent la chaîne `contoso.com`. Les résultats suivants doivent s'afficher :

    ```output
    Franklin Holtz - ID: 2
    Franklin Holtz - franklin@contoso.com - ID: 2
    Rae Schroeder - ID: 4
    Rae Schroeder - rae@contoso.com - ID: 4
    Gabriela Ingram - ID: 6
    Gabriela Ingram - gabriela@contoso.com - ID: 6
    ```

    L’utilisation de **fabrikam.com** pour la valeur `-emailRegex` renvoie les utilisateurs dont le champ email contient **fabrikam.com** . Vous pouvez également utiliser des expressions régulières comme termes de recherche. Par exemple, **^r** renvoie les adresses e-mail qui commencent par la lettre « r ».

7. Utilisez la commande suivante pour supprimer le tableau :

    ```PowerShell
    Start-HBaseExample -className com.microsoft.examples.DeleteTable -clusterName $myCluster
    ```

### <a name="no-results-or-unexpected-results-when-using-start-hbaseexample"></a>Aucun résultat ou résultat inattendu lors de l'utilisation de Start-HBaseExample

Utilisez le paramètre `-showErr` pour afficher l’erreur standard (STDERR) produite lors de l'exécution de la tâche.

## <a name="next-steps"></a>Étapes suivantes

[Découvrez comment utiliser SQLLine avec Apache HBase](apache-hbase-query-with-phoenix.md)
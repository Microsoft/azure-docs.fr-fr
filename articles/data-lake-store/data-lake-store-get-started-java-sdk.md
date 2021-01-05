---
title: SDK Java - Opérations de gestion du système de fichiers sur Data Lake Storage Gen1 - Azure
description: Utilisez le kit SDK Java pour Azure Data Lake Storage Gen1 pour effectuer des opérations de gestion du système de fichiers sur Data Lake Storage Gen1, comme la création de dossiers, et le chargement et le téléchargement de fichiers de données.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.custom: devx-track-java
ms.author: twooley
ms.openlocfilehash: a2c55a2d3277bbb6c3cf72f5ea703780d2a5e9bd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87318842"
---
# <a name="filesystem-operations-on-azure-data-lake-storage-gen1-using-java-sdk"></a>Opérations de gestion du système de fichiers sur Azure Data Lake Storage Gen1 à l’aide du Kit de développement logiciel (SDK) Java
> [!div class="op_single_selector"]
> * [Kit de développement logiciel (SDK) .NET](data-lake-store-data-operations-net-sdk.md)
> * [Kit SDK Java](data-lake-store-get-started-java-sdk.md)
> * [REST API](data-lake-store-data-operations-rest-api.md)
> * [Python](data-lake-store-data-operations-python.md)
>
> 

Découvrez comment utiliser le Kit de développement logiciel (SDK) Java Azure Data Lake Storage Gen1 pour effectuer des opérations de base comme créer des dossiers ou charger et télécharger des fichiers de données. Pour plus d’informations sur Data Lake Storage Gen1, consultez [Azure Data Lake Storage Gen1](data-lake-store-overview.md).

Vous pouvez accéder à la documentation de l’API du Kit de développement logiciel (SDK) Java pour Data Lake Storage Gen1 dans la [documentation de l’API Java Azure Data Lake Storage Gen1](https://azure.github.io/azure-data-lake-store-java/javadoc/).

## <a name="prerequisites"></a>Prérequis
* Kit de développement Java (JDK 7 ou version ultérieure, utilisant Java version 1.7 ou ultérieure)
* Compte Data Lake Storage Gen1. Suivez les instructions de [Prise en main d’Azure Data Lake Storage Gen1 avec le portail Azure](data-lake-store-get-started-portal.md).
* [Maven](https://maven.apache.org/install.html). Ce didacticiel utilise Maven pour les dépendances de build et de projet. Bien qu’il soit possible de créer sans utiliser un système de build comme Maven ou Gradle, ces systèmes facilitent considérablement la gestion des dépendances.
* (Facultatif) Et un environnement IDE comme [IntelliJ IDEA](https://www.jetbrains.com/idea/download/) ou [Eclipse](https://www.eclipse.org/downloads/) ou similaire.

## <a name="create-a-java-application"></a>Création d’une application Java
L’exemple de code disponible [sur GitHub](https://azure.microsoft.com/documentation/samples/data-lake-store-java-upload-download-get-started/) vous guide tout au long du processus de création de fichiers dans le magasin, de concaténation de fichiers, de téléchargement d’un fichier et de suppression de certains fichiers du compte. Cette section de l’article vous guide tout au long des principales parties du code.

1. Créez un projet Maven en utilisant [mvn archetype](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html) à partir de la ligne de commande ou à l’aide d’un environnement de développement intégré (IDE). Vous trouverez [ici](https://www.jetbrains.com/help/idea/2016.1/creating-and-running-your-first-java-application.html) des instructions sur la création d’un projet Java à l’aide d’IntelliJ. Vous trouverez [ici](https://help.eclipse.org/mars/index.jsp?topic=%2Forg.eclipse.jdt.doc.user%2FgettingStarted%2Fqs-3.htm) des instructions sur la création d’un projet à l’aide d’Eclipse. 

2. Ajoutez les dépendances suivantes à votre fichier Maven **pom.xml**. Ajoutez l’extrait de code suivant avant la balise **\</project>** :
   
    ```xml
    <dependencies>
        <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-data-lake-store-sdk</artifactId>
        <version>2.1.5</version>
        </dependency>
        <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-nop</artifactId>
        <version>1.7.21</version>
        </dependency>
    </dependencies>
    ```
   
    La première dépendance consiste à utiliser le kit de développement logiciel (SDK) Data Lake Storage Gen1 (`azure-data-lake-store-sdk`) à partir du référentiel maven. La seconde dépendance consiste à spécifier le framework de journalisation (`slf4j-nop`) à utiliser pour cette application. Le SDK Data Lake Storage Gen1 utilise la façade de journalisation [SLF4J](https://www.slf4j.org/). Cela vous permet de choisir parmi plusieurs frameworks de journalisation populaires, par exemple Log4j, la journalisation Java, Logback, etc. Vous pouvez aussi n’utiliser aucune journalisation. Pour cet exemple, nous désactivons la journalisation et, par conséquent, nous utilisons la liaison **slf4j-nop**. Vous trouverez [ici](https://www.slf4j.org/manual.html#projectDep) des informations pour utiliser d’autres options de journalisation.

3. Ajoutez les instructions import ci-après à votre application.

    ```java
    import com.microsoft.azure.datalake.store.ADLException;
    import com.microsoft.azure.datalake.store.ADLStoreClient;
    import com.microsoft.azure.datalake.store.DirectoryEntry;
    import com.microsoft.azure.datalake.store.IfExists;
    import com.microsoft.azure.datalake.store.oauth2.AccessTokenProvider;
    import com.microsoft.azure.datalake.store.oauth2.ClientCredsTokenProvider;

    import java.io.*;
    import java.util.Arrays;
    import java.util.List;
    ```

## <a name="authentication"></a>Authentification

* Pour en savoir plus sur l’authentification des utilisateurs accédant à votre application, consultez [Authentification des utilisateurs finaux auprès de Data Lake Storage Gen1 avec le Kit SDK Java](data-lake-store-end-user-authenticate-java-sdk.md).
* Pour en savoir plus sur l’authentification entre les services dans le cadre de votre application, consultez [Authentification de service à service auprès de Data Lake Storage Gen1 à l’aide de Java](data-lake-store-service-to-service-authenticate-java.md).

## <a name="create-a-data-lake-storage-gen1-client"></a>Créer un client Data Lake Storage Gen1
La création d’un objet [ADLStoreClient](https://azure.github.io/azure-data-lake-store-java/javadoc/) vous oblige à spécifier les noms du compte Data Lake Storage Gen1 et du fournisseur de jetons que vous avez générés quand vous vous êtes authentifié avec Data Lake Storage Gen1 (voir la section [Authentification](#authentication)). Le nom du compte Data Lake Storage Gen1 doit être un nom de domaine complet. Par exemple, remplacez **FILL-IN-HERE** (à remplir) par quelque chose comme **mydatalakestoragegen1.azuredatalakestore.net**.

```java
private static String accountFQDN = "FILL-IN-HERE";  // full account FQDN, not just the account name
ADLStoreClient client = ADLStoreClient.createClient(accountFQDN, provider);
```

Les extraits de code des sections suivantes contiennent des exemples d’opérations de gestion du système de fichiers communes. Vous pouvez consulter toute la [documentation sur l’API SDK Java Data Lake Storage Gen1](https://azure.github.io/azure-data-lake-store-java/javadoc/) de l’objet **ADLStoreClient** pour afficher d’autres opérations.

## <a name="create-a-directory"></a>Créer un répertoire

L’extrait de code suivant permet de créer la structure d’un répertoire à la racine du compte Data Lake Storage Gen1 spécifié.

```java
// create directory
client.createDirectory("/a/b/w");
System.out.println("Directory created.");
```

## <a name="create-a-file"></a>Créer un fichier

L’extrait de code suivant permet de créer un fichier (c.txt) dans la structure du répertoire et d’y écrire des données.

```java
// create file and write some content
String filename = "/a/b/c.txt";
OutputStream stream = client.createFile(filename, IfExists.OVERWRITE  );
PrintStream out = new PrintStream(stream);
for (int i = 1; i <= 10; i++) {
    out.println("This is line #" + i);
    out.format("This is the same line (%d), but using formatted output. %n", i);
}
out.close();
System.out.println("File created.");
```

Vous pouvez aussi créer un fichier (d.txt) à l’aide de tableaux d’octets.

```java
// create file using byte arrays
stream = client.createFile("/a/b/d.txt", IfExists.OVERWRITE);
byte[] buf = getSampleContent();
stream.write(buf);
stream.close();
System.out.println("File created using byte array.");
```

La définition de la fonction `getSampleContent` (utilisée dans l’extrait de code précédent) est disponible dans l’exemple sur [GitHub](https://azure.microsoft.com/documentation/samples/data-lake-store-java-upload-download-get-started/). 

## <a name="append-to-a-file"></a>Ajout à un fichier

L’extrait de code suivant permet d’ajouter du contenu à un fichier existant.

```java
// append to file
stream = client.getAppendStream(filename);
stream.write(getSampleContent());
stream.close();
System.out.println("File appended.");
```

La définition de la fonction `getSampleContent` (utilisée dans l’extrait de code précédent) est disponible dans l’exemple sur [GitHub](https://azure.microsoft.com/documentation/samples/data-lake-store-java-upload-download-get-started/).

## <a name="read-a-file"></a>Lire un fichier

L’extrait de code suivant permet de lire le contenu d’un fichier dans un compte Data Lake Storage Gen1.

```java
// Read File
InputStream in = client.getReadStream(filename);
BufferedReader reader = new BufferedReader(new InputStreamReader(in));
String line;
while ( (line = reader.readLine()) != null) {
    System.out.println(line);
}
reader.close();
System.out.println();
System.out.println("File contents read.");
```

## <a name="concatenate-files"></a>Concaténation de fichiers

L’extrait de code suivant permet de concaténer deux fichiers dans un compte Data Lake Storage Gen1. En cas de réussite, le fichier concaténé remplace les deux fichiers existants.

```java
// concatenate the two files into one
List<String> fileList = Arrays.asList("/a/b/c.txt", "/a/b/d.txt");
client.concatenateFiles("/a/b/f.txt", fileList);
System.out.println("Two files concatenated into a new file.");
```

## <a name="rename-a-file"></a>Renommer un fichier

L’extrait de code suivant permet de renommer un fichier dans un compte Data Lake Storage Gen1.

```java
//rename the file
client.rename("/a/b/f.txt", "/a/b/g.txt");
System.out.println("New file renamed.");
```

## <a name="get-metadata-for-a-file"></a>Obtenir les métadonnées d’un fichier

L’extrait de code suivant permet d’obtenir les métadonnées d’un fichier du compte Data Lake Storage Gen1.

```java
// get file metadata
DirectoryEntry ent = client.getDirectoryEntry(filename);
printDirectoryInfo(ent);
System.out.println("File metadata retrieved.");
```

## <a name="set-permissions-on-a-file"></a>Définir les autorisations sur un fichier

L’extrait de code suivant permet de définir les autorisations sur le fichier que vous avez créé dans la section précédente.

```java
// set file permission
client.setPermission(filename, "744");
System.out.println("File permission set.");
```

## <a name="list-directory-contents"></a>Afficher le contenu du répertoire

L’extrait de code suivant permet d’afficher le contenu d’un répertoire de manière récursive.

```java
// list directory contents
List<DirectoryEntry> list = client.enumerateDirectory("/a/b", 2000);
System.out.println("Directory listing for directory /a/b:");
for (DirectoryEntry entry : list) {
    printDirectoryInfo(entry);
}
System.out.println("Directory contents listed.");
```

La définition de la fonction `printDirectoryInfo` (utilisée dans l’extrait de code précédent) est disponible dans l’exemple sur [GitHub](https://azure.microsoft.com/documentation/samples/data-lake-store-java-upload-download-get-started/).

## <a name="delete-files-and-folders"></a>Supprimer des fichiers et dossiers

L’extrait de code suivant permet de supprimer les fichiers et dossiers spécifiques d’un compte Data Lake Storage Gen1 de manière récursive.

```java
// delete directory along with all the subdirectories and files in it
client.deleteRecursive("/a");
System.out.println("All files and folders deleted recursively");
promptEnterKey();
```

## <a name="build-and-run-the-application"></a>Génération et exécution de l’application
1. Pour une exécution au sein d’un environnement IDE, recherchez et appuyez sur le bouton **Run** (Exécuter). Pour une exécution depuis Maven, utilisez [exec:exec](https://www.mojohaus.org/exec-maven-plugin/exec-mojo.html).
2. Pour produire un fichier jar autonome que vous pouvez exécuter à partir de la ligne de commande, créez le fichier jar en incluant toutes les dépendances à l’aide du [plug-in assembly Maven](https://maven.apache.org/plugins/maven-assembly-plugin/usage.html). Le fichier pom.xml de l'[exemple de code source sur GitHub](https://github.com/Azure-Samples/data-lake-store-java-upload-download-get-started/blob/master/pom.xml) contient un exemple.

## <a name="next-steps"></a>Étapes suivantes
* [Explorer JavaDoc pour le SDK Java](https://azure.github.io/azure-data-lake-store-java/javadoc/)
* [Sécuriser les données dans Data Lake Storage Gen1](data-lake-store-secure-data.md)



---
title: Utilisation des activités personnalisées dans un pipeline Azure Data Factory
description: Découvrez comment créer des activités personnalisées et les utiliser dans un pipeline Azure Data Factory.
services: data-factory
documentationcenter: ''
ms.assetid: 8dd7ba14-15d2-4fd9-9ada-0b2c684327e9
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
author: nabhishek
ms.author: abnarain
ms.custom: devx-track-csharp
manager: anandsub
robots: noindex
ms.openlocfilehash: 0ef6c97f7924c890bb6665100259970372f1cd26
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97606944"
---
# <a name="use-custom-activities-in-an-azure-data-factory-version-1-pipeline"></a>Utilisation des activités personnalisées dans un pipeline Azure Data Factory version 1
> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](data-factory-use-custom-activities.md)
> * [Version 2 (version actuelle)](../transform-data-using-dotnet-custom-activity.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Activités personnalisées dans V2](../transform-data-using-dotnet-custom-activity.md).

Vous pouvez utiliser deux types d’activités dans un pipeline Azure Data Factory.

- Les [activités de déplacement des données](data-factory-data-movement-activities.md) permettent de déplacer des données entre des [magasins de données source et récepteur pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats).
- [Activités de transformation de données](data-factory-data-transformation-activities.md) pour transformer des données à l’aide de services de calcul, comme Azure HDInsight, Azure Batch et Azure Machine Learning (classique).

Pour déplacer des données vers ou à partir d’un magasin de données qui n’est pas pris en charge par Data Factory, créez une **activité personnalisée** avec votre propre logique de déplacement des données et utilisez cette activité dans un pipeline. De même, si vous devez transformer et traiter les données d’une manière qui n’est pas prise en charge par Data Factory, créez une activité personnalisée avec votre propre logique de transformation des données et utilisez cette activité dans un pipeline.

Vous pouvez configurer une activité personnalisée pour qu’elle s’exécute sur un pool **Azure Batch** de machines virtuelles. Lorsque vous utilisez Azure Batch, vous pouvez utiliser uniquement un pool Azure Batch.

La procédure suivante fournit des instructions pas à pas pour créer une activité .NET personnalisée et utiliser cette activité personnalisée dans un pipeline. La procédure pas à pas utilise un service lié **Azure Batch**.

> [!IMPORTANT]
> - Il n’est pas possible d’utiliser une passerelle de gestion des données à partir d’une activité personnalisée pour accéder à des sources de données locales. Actuellement, la [passerelle de gestion des données](data-factory-data-management-gateway.md) prend en charge uniquement l’activité de copie et l’activité de procédure stockée dans Data Factory.

## <a name="walkthrough-create-a-custom-activity"></a>Procédure pas à pas : création d’une activité personnalisée
### <a name="prerequisites"></a>Prérequis
* Visual Studio 2012/2013/2015/2017
* Téléchargez et installez le [Kit de développement logiciel (SDK) Azure .NET](https://azure.microsoft.com/downloads/)

### <a name="azure-batch-prerequisites"></a>Configuration requise pour Azure Batch
Dans la procédure pas à pas, vous allez exécuter vos activités .NET personnalisées à l’aide de la ressource de calcul Azure Batch. **Azure Batch** est une plateforme qui permet d’exécuter efficacement des applications de calcul haute performance (HPC) en parallèle et à grande échelle dans le cloud. Azure Batch planifie les travaux nécessitant une grande quantité de ressources système à exécuter sur une **collection gérée de machines virtuelles**. Il peut mettre automatiquement à l’échelle les ressources de calcul pour répondre aux besoins de vos travaux. Consultez l’article [Notions de base d’Azure Batch][batch-technical-overview] pour une présentation détaillée du service Azure Batch.

Pour ce didacticiel, créez un compte Azure Batch avec un pool de machines virtuelles. Voici la procédure à suivre :

1. Créez un compte **Azure Batch** via le [portail Azure](https://portal.azure.com). Consultez l’article [Créer et gérer un compte Azure Batch][batch-create-account] pour obtenir des instructions.
2. Notez le nom du pool, l’URI, la clé et le nom du compte Azure Batch. Vous en avez besoin pour créer un service lié Azure Batch.
    1. Sur la page d’accueil du compte Azure Batch, vous voyez une **URL** au format suivant : `https://myaccount.westus.batch.azure.com`. Dans cet exemple, **myaccount** est le nom du compte Azure Batch. L’URI que vous utilisez dans la définition de service lié est l’URL sans le nom du compte. Par exemple : `https://<region>.batch.azure.com`.
    2. Cliquez sur **Clés** dans le menu de gauche et copiez la **CLÉ D’ACCÈS PRIMAIRE**.
    3. Pour utiliser un pool existant, cliquez sur **Pools** dans le menu, puis notez **l’ID** du pool. Si vous n’avez pas de pool existant, passez à l’étape suivante.
2. Créez un **pool Azure Batch**.

   1. Dans le [portail Azure](https://portal.azure.com), cliquez sur **Parcourir** dans le menu de gauche, puis cliquez sur **Comptes Batch**.
   2. Sélectionnez votre compte Azure Batch pour ouvrir le panneau **Compte Batch** .
   3. Cliquez sur la vignette **Pools** .
   4. Dans le panneau **Pools** , cliquez sur le bouton Ajouter de la barre d’outils pour ajouter un pool.
      1. Entrez un ID pour le pool (ID du pool). Notez **l’ID du pool**, car vous en aurez besoin lors de la création de la solution Data Factory.
      2. Spécifiez **Windows Server 2012 R2** pour le paramètre de famille du système d’exploitation.
      3. Sélectionnez le **niveau tarifaire du nœud**.
      4. Entrez **2** comme valeur du paramètre **Quantité dédiée cible**.
      5. Entrez **2** comme valeur du paramètre **Nombre maximal de tâches par nœud**.
   5. Cliquez sur **OK** pour créer le pool.
   6. Notez **l’ID** du pool.

### <a name="high-level-steps"></a>Procédure générale
Voici les deux étapes principales que vous effectuez dans le cadre de cette procédure pas à pas :

1. Créer une activité personnalisée qui contient une logique simple de transformation/traitement des données.
2. Créer une fabrique de données Azure avec un pipeline qui utilise l’activité personnalisée.

### <a name="create-a-custom-activity"></a>création d'une activité personnalisée
Pour créer une activité .NET personnalisée, créez un projet de **bibliothèque de classes .NET** contenant une classe qui implémente l’interface **IDotNetActivity**. Cette interface ne possède qu’une méthode : [Execute](/dotnet/api/microsoft.azure.management.datafactories.runtime.idotnetactivity) et sa signature est :

```csharp
public IDictionary<string, string> Execute(
    IEnumerable<LinkedService> linkedServices,
    IEnumerable<Dataset> datasets,
    Activity activity,
    IActivityLogger logger)
```

La méthode accepte quatre paramètres :

- **linkedServices**. Cette propriété est une liste énumérable de services liés Data Store référencés par les jeux de données d’entrée/sortie de l’activité.
- **jeux de données**. Cette propriété est une liste énumérable de jeux de données d’entrée/sortie de l’activité. Vous pouvez utiliser ce paramètre pour obtenir les emplacements et les schémas définis par les jeux de données d’entrée et de sortie.
- **activity**. Cette propriété représente l’activité actuelle. Elle peut être utilisée pour accéder aux propriétés étendues associées à l’activité personnalisée. Consultez la section [Accéder aux propriétés étendues](#access-extended-properties) pour plus d’informations.
- **logger**. Cet objet vous permet d’écrire des commentaires de débogage qui apparaîtront dans le journal utilisateur pour le pipeline.

La méthode retourne un dictionnaire qui peut être utilisé pour enchaîner ultérieurement des activités personnalisées. Cette fonctionnalité n’étant pas encore implémentée, seul un dictionnaire vide est retourné par la méthode.

### <a name="procedure"></a>Procédure

1. Créez un projet de **bibliothèque de classes .NET** .
   
    <ol type="a">
     <li>Lancez Visual Studio.</li>
     <li>Cliquez sur <b>Fichier</b>, pointez le curseur de la souris sur <b>Nouveau</b>, puis cliquez sur <b>Projet</b>.</li>
     <li>Développez <b>Modèles</b>, puis sélectionnez <b>Visual C#</b>. Dans cette procédure pas à pas, vous utilisez C#, mais vous pouvez utiliser un autre langage .NET pour développer l’activité personnalisée.</li>
     <li>Sélectionnez <b>Bibliothèque de classes</b> dans la liste des types de projet, sur la droite. Dans Visual Studio, choisissez <b>Bibliothèque de classes (.NET Framework)</b> .</li>
     <li>Entrez <b>MyDotNetActivity</b> for the <b>Nom</b>.</li>
     <li>Sélectionnez <b>C:\ADFGetStarted</b> comme <b>Emplacement</b>.</li>
     <li>Cliquez sur <b>OK</b> pour créer le projet.</li>
   </ol>

2. Cliquez sur **Outils**, pointez le curseur de la souris sur **Gestionnaire de package NuGet**, puis cliquez sur **Console du gestionnaire de package**.

3. Dans la Console du gestionnaire de package, exécutez la commande suivante pour importer l’élément **Microsoft.Azure.Management.DataFactories**.

    ```powershell
    Install-Package Microsoft.Azure.Management.DataFactories
    ```

4. Importez le package NuGet **Azure Storage** dans le projet.

    ```powershell
    Install-Package WindowsAzure.Storage -Version 4.3.0
    ```

    > [!IMPORTANT]
    > Le lanceur du service Data Factory requiert la version 4.3 du package Windows Azure Storage. Si vous ajoutez une référence à une version ultérieure de l’assembly Azure Storage à votre projet d’activité personnalisée, vous obtenez une erreur lors de l’exécution de l’activité. Pour résoudre l’erreur, consultez la section [Isolation du domaine d’application](#appdomain-isolation).
5. Ajoutez les instructions **using** ci-après dans le fichier source du projet.

    ```csharp

    // Comment these lines if using VS 2017
    using System.IO;
    using System.Globalization;
    using System.Diagnostics;
    using System.Linq;
    // --------------------

    // Comment these lines if using <= VS 2015
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Text;
    using System.Threading.Tasks;
    // ---------------------

    using Microsoft.Azure.Management.DataFactories.Models;
    using Microsoft.Azure.Management.DataFactories.Runtime;

    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;
    ```

6. Remplacez le nom de **l’espace de noms** par **MyDotNetActivityNS**.

    ```csharp
    namespace MyDotNetActivityNS
    ```

7. Remplacez le nom de la classe par **MyDotNetActivity** et dérivez-le de l’interface **IDotNetActivity**, comme indiqué dans l’extrait de code suivant :

    ```csharp
    public class MyDotNetActivity : IDotNetActivity
    ```

8. Implémentez (ajoutez) la méthode **Execute** de l’interface **IDotNetActivity** dans la classe **MyDotNetActivity** et copiez l’exemple de code suivant dans la méthode.

    L’exemple suivant compte le nombre d’occurrences du terme recherché (Microsoft) dans chaque objet blob associé à une tranche de données.

    ```csharp
    /// <summary>
    /// Execute method is the only method of IDotNetActivity interface you must implement.
    /// In this sample, the method invokes the Calculate method to perform the core logic.
    /// </summary>

    public IDictionary<string, string> Execute(
        IEnumerable<LinkedService> linkedServices,
        IEnumerable<Dataset> datasets,
        Activity activity,
        IActivityLogger logger)
    {
        // get extended properties defined in activity JSON definition
        // (for example: SliceStart)
        DotNetActivity dotNetActivity = (DotNetActivity)activity.TypeProperties;
        string sliceStartString = dotNetActivity.ExtendedProperties["SliceStart"];

        // to log information, use the logger object
        // log all extended properties
        IDictionary<string, string> extendedProperties = dotNetActivity.ExtendedProperties;
        logger.Write("Logging extended properties if any...");
        foreach (KeyValuePair<string, string> entry in extendedProperties)
        {
            logger.Write("<key:{0}> <value:{1}>", entry.Key, entry.Value);
        }

        // linked service for input and output data stores
        // in this example, same storage is used for both input/output
        AzureStorageLinkedService inputLinkedService;

        // get the input dataset
        Dataset inputDataset = datasets.Single(dataset => dataset.Name == activity.Inputs.Single().Name);

        // declare variables to hold type properties of input/output datasets
        AzureBlobDataset inputTypeProperties, outputTypeProperties;

        // get type properties from the dataset object
        inputTypeProperties = inputDataset.Properties.TypeProperties as AzureBlobDataset;

        // log linked services passed in linkedServices parameter
        // you will see two linked services of type: AzureStorage
        // one for input dataset and the other for output dataset
        foreach (LinkedService ls in linkedServices)
            logger.Write("linkedService.Name {0}", ls.Name);

        // get the first Azure Storage linked service from linkedServices object
        // using First method instead of Single since we are using the same
        // Azure Storage linked service for input and output.
        inputLinkedService = linkedServices.First(
            linkedService =>
            linkedService.Name ==
            inputDataset.Properties.LinkedServiceName).Properties.TypeProperties
            as AzureStorageLinkedService;

        // get the connection string in the linked service
        string connectionString = inputLinkedService.ConnectionString;

        // get the folder path from the input dataset definition
        string folderPath = GetFolderPath(inputDataset);
        string output = string.Empty; // for use later.

        // create storage client for input. Pass the connection string.
        CloudStorageAccount inputStorageAccount = CloudStorageAccount.Parse(connectionString);
        CloudBlobClient inputClient = inputStorageAccount.CreateCloudBlobClient();

        // initialize the continuation token before using it in the do-while loop.
        BlobContinuationToken continuationToken = null;
        do
        {   // get the list of input blobs from the input storage client object.
            BlobResultSegment blobList = inputClient.ListBlobsSegmented(folderPath,
                                     true,
                                     BlobListingDetails.Metadata,
                                     null,
                                     continuationToken,
                                     null,
                                     null);

            // Calculate method returns the number of occurrences of
            // the search term (“Microsoft”) in each blob associated
            // with the data slice. definition of the method is shown in the next step.

            output = Calculate(blobList, logger, folderPath, ref continuationToken, "Microsoft");

        } while (continuationToken != null);

        // get the output dataset using the name of the dataset matched to a name in the Activity output collection.
        Dataset outputDataset = datasets.Single(dataset => dataset.Name == activity.Outputs.Single().Name);

        // get type properties for the output dataset
        outputTypeProperties = outputDataset.Properties.TypeProperties as AzureBlobDataset;

        // get the folder path from the output dataset definition
        folderPath = GetFolderPath(outputDataset);

        // log the output folder path
        logger.Write("Writing blob to the folder: {0}", folderPath);

        // create a storage object for the output blob.
        CloudStorageAccount outputStorageAccount = CloudStorageAccount.Parse(connectionString);
        // write the name of the file.
        Uri outputBlobUri = new Uri(outputStorageAccount.BlobEndpoint, folderPath + "/" + GetFileName(outputDataset));

        // log the output file name
        logger.Write("output blob URI: {0}", outputBlobUri.ToString());

        // create a blob and upload the output text.
        CloudBlockBlob outputBlob = new CloudBlockBlob(outputBlobUri, outputStorageAccount.Credentials);
        logger.Write("Writing {0} to the output blob", output);
        outputBlob.UploadText(output);

        // The dictionary can be used to chain custom activities together in the future.
        // This feature is not implemented yet, so just return an empty dictionary.

        return new Dictionary<string, string>();
    }
    ```

9. Ajoutez les méthodes d’assistance suivantes :

    ```csharp
    /// <summary>
    /// Gets the folderPath value from the input/output dataset.
    /// </summary>

    private static string GetFolderPath(Dataset dataArtifact)
    {
        if (dataArtifact == null || dataArtifact.Properties == null)
        {
            return null;
        }

        // get type properties of the dataset
        AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;
        if (blobDataset == null)
        {
            return null;
        }

        // return the folder path found in the type properties
        return blobDataset.FolderPath;
    }

    /// <summary>
    /// Gets the fileName value from the input/output dataset.
    /// </summary>

    private static string GetFileName(Dataset dataArtifact)
    {
        if (dataArtifact == null || dataArtifact.Properties == null)
        {
            return null;
        }

        // get type properties of the dataset
        AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;
        if (blobDataset == null)
        {
            return null;
        }

        // return the blob/file name in the type properties
        return blobDataset.FileName;
    }

    /// <summary>
    /// Iterates through each blob (file) in the folder, counts the number of instances of search term in the file,
    /// and prepares the output text that is written to the output blob.
    /// </summary>

    public static string Calculate(BlobResultSegment Bresult, IActivityLogger logger, string folderPath, ref BlobContinuationToken token, string searchTerm)
    {
        string output = string.Empty;
        logger.Write("number of blobs found: {0}", Bresult.Results.Count<IListBlobItem>());
        foreach (IListBlobItem listBlobItem in Bresult.Results)
        {
            CloudBlockBlob inputBlob = listBlobItem as CloudBlockBlob;
            if ((inputBlob != null) && (inputBlob.Name.IndexOf("$$$.$$$") == -1))
            {
                string blobText = inputBlob.DownloadText(Encoding.ASCII, null, null, null);
                logger.Write("input blob text: {0}", blobText);
                string[] source = blobText.Split(new char[] { '.', '?', '!', ' ', ';', ':', ',' }, StringSplitOptions.RemoveEmptyEntries);
                var matchQuery = from word in source
                                 where word.ToLowerInvariant() == searchTerm.ToLowerInvariant()
                                 select word;
                int wordCount = matchQuery.Count();
                output += string.Format("{0} occurrences(s) of the search term \"{1}\" were found in the file {2}.\r\n", wordCount, searchTerm, inputBlob.Name);
            }
        }
        return output;
    }
    ```

    La méthode GetFolderPath renvoie le chemin d’accès au dossier vers lequel pointe le jeu de données et la méthode GetFileName renvoie le nom de l’objet blob/fichier vers lequel pointe le jeu de données. Si folderPath est défini avec des variables telles que {Year}, {Month}, {Day}, etc., la méthode retourne la chaîne en l’état, sans remplacer les variables par les valeurs d’exécution. Consultez la section [Accéder aux propriétés étendues](#access-extended-properties) pour plus d’informations sur l’accès à SliceStart, SliceEnd, etc.

    ```JSON
    "name": "InputDataset",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "fileName": "file.txt",
            "folderPath": "adftutorial/inputfolder/",
    ```

    La méthode Calculate calcule le nombre d’instances du mot-clé Microsoft dans les fichiers d’entrée (objets blob du dossier). Le terme de recherche (« Microsoft ») est codé en dur dans le code.

10. Compilez le projet. Cliquez sur l’option **Générer** du menu, puis sur **Générer la solution**.

    > [!IMPORTANT]
    > Définissez la version 4.5.2 de .NET Framework comme infrastructure cible pour votre projet : cliquez avec le bouton droit sur le projet et cliquez sur **Propriétés** pour définir l’infrastructure cible. Data Factory ne prend pas en charge les activités personnalisées compilées avec les versions de .NET Framework ultérieures à la version 4.5.2.

11. Lancez **l’Explorateur Windows** et accédez au dossier **bin\debug** ou **bin\release** (selon le type de build).

12. Créez un fichier zip **MyDotNetActivity.zip** contenant tous les fichiers binaires dans le dossier \<project folder\>\bin\Debug. Incluez le fichier **MyDotNetActivity.pdb** afin d’obtenir des détails supplémentaires tels que le numéro de ligne du code source à l’origine du problème en cas de défaillance.

    > [!IMPORTANT]
    > Tous les fichiers contenus dans le fichier zip de l’activité personnalisée doivent se trouver au **premier niveau** et ne doivent pas contenir de sous-dossiers.

    ![Fichiers de sortie binaires](./media/data-factory-use-custom-activities/Binaries.png)

13. Si nécessaire, créez un conteneur de blobs nommé **customactivitycontainer**.

14. Chargez MyDotNetActivity.zip en tant qu’objet blob dans le customactivitycontainer dans un compte Stockage Blob Azure **à usage général** (pas un Stockage Blob chaud/froid) référencé par AzureStorageLinkedService.

> [!IMPORTANT]
> Si vous ajoutez ce projet d’activité .NET à une solution dans Visual Studio qui contient un projet Data Factory, et que vous ajoutez une référence au projet d’activité .NET à partir du projet d’application Data Factory, vous n’avez pas besoin d’effectuer les deux dernières étapes pour créer manuellement le fichier zip et le télécharger dans le stockage Blob Azure à usage général. Lorsque vous publiez des entités Data Factory à l'aide de Visual Studio, ces étapes sont effectuées automatiquement par le processus de publication. Pour plus d’informations, consultez la section [Projet Data Factory dans Visual Studio](#data-factory-project-in-visual-studio).

## <a name="create-a-pipeline-with-custom-activity"></a>Créer un pipeline avec une activité personnalisée

Vous avez créé une activité personnalisée et chargé le fichier zip avec des fichiers binaires dans un conteneur de blobs dans un compte de stockage Azure **à usage général**. Dans cette section, vous allez créer une fabrique de données Azure avec un pipeline qui utilise l’activité personnalisée.

Le jeu de données d’entrée de l’activité personnalisée représente les blobs (fichiers) contenus dans le dossier customactivityinput du conteneur adftutorial dans le stockage Blob. Le jeu de données de sortie de l’activité représente les blobs de sortie contenus dans le dossier customactivityoutput du conteneur adftutorial dans le stockage Blob.

Créez le fichier **file.txt** avec le contenu suivant, puis chargez-le dans le dossier **customactivityinput** du conteneur **adftutorial**. S’il n’existe pas déjà, créez le conteneur adftutorial.

```
test custom activity Microsoft test custom activity Microsoft
```

Le dossier d’entrée correspond à une tranche dans Azure Data Factory, même si le dossier contient au moins deux fichiers. Lorsque chaque tranche est traitée par le pipeline, l’activité personnalisée effectue une itération dans tous les objets blob du dossier d’entrée pour la tranche en question.

Un fichier de sortie à une ou plusieurs lignes (le même nombre de lignes que le nombre de blobs contenus dans le dossier d’entrée) apparaîtra dans le dossier adftutorial\customactivityoutput :

```
2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2016-11-16-00/file.txt.
```


Voici les étapes que vous effectuez dans cette section :

1. Création d'une **fabrique de données**.
2. Créez des **services liés** pour le pool Azure Batch de machines virtuelles sur lesquelles l’activité personnalisée doit être exécutée et le stockage Azure qui contient les blobs d’entrée et de sortie.
3. Créez des **jeux de données** d’entrée et de sortie qui représentent les entrées/sorties de l’activité personnalisée.
4. Créez un **pipeline** qui utilise l’activité personnalisée.

> [!NOTE]
> Créez le fichier **file.txt** et chargez-le dans un conteneur d’objets blob, si vous ne l’avez pas déjà fait. Consultez les instructions fournies dans la section précédente.

### <a name="step-1-create-the-data-factory"></a>Étape 1 : Création de la fabrique de données
1. Une fois connecté au portail Azure, procédez comme suit :
   1. Dans le menu de gauche, sélectionnez **Créer une ressource**.
   2. Dans le panneau **Nouveau**, cliquez sur **Données et analyses**.
   3. Cliquez sur **Data Factory** dans le panneau **Analyse des données**.

      ![Nouveau menu Azure Data Factory](media/data-factory-use-custom-activities/new-azure-data-factory-menu.png)
2. Dans le panneau **Nouvelle fabrique de données**, spécifiez le nom **CustomActivityFactory**. Le nom de la fabrique de données Azure doit être un nom global unique. Si vous recevez le message d’erreur : **Data factory name “CustomActivityFactory” is not available** (Le nom de la fabrique de données « CustomActivityFactory » n’est pas disponible), changez le nom de la fabrique de données (par exemple, **votrenomCustomActivityFactory**), puis tentez de la recréer.

    ![Nouveau panneau Azure Data Factory](media/data-factory-use-custom-activities/new-azure-data-factory-blade.png)
3. Cliquez sur **NOM DU GROUPE DE RESSOURCES** pour sélectionner un groupe de ressources existant, ou créez un groupe de ressources.
4. Veillez à utiliser **l’abonnement** et la **région** correspondant à ceux dans lesquels vous voulez créer la fabrique de données.
5. Cliquez sur **Créer** dans le panneau **Nouvelle fabrique de données**.
6. La fabrique de données apparaît comme étant en cours de création dans le **Tableau de bord** du portail Azure.
7. Une fois la fabrique de données créée, son contenu apparaîtra dans le panneau correspondant indiquant.

    ![Panneau Data Factory](media/data-factory-use-custom-activities/data-factory-blade.png)

### <a name="step-2-create-linked-services"></a>Étape 2 : Créez des services liés
Les services liés se chargent de lier des magasins de données ou des services de calcul à une fabrique de données Azure. Dans cette étape, vous allez lier vos comptes Stockage Azure et Azure Batch à votre fabrique de données.

#### <a name="create-azure-storage-linked-service"></a>Créer le service lié Stockage Azure
1. Cliquez sur la vignette **Créer et déployer** dans le panneau **DATA FACTORY** de **CustomActivityFactory**. Data Factory Editor s’affiche.
2. Cliquez sur **Nouvelle banque de données** dans la barre de commandes et choisissez **Stockage Azure**. Le script JSON de création d’un service lié Stockage Microsoft Azure doit apparaître dans l’éditeur.

    ![Nouveau magasin de données - Stockage Azure](media/data-factory-use-custom-activities/new-data-store-menu.png)
3. Remplacez `<accountname>` par le nom de votre compte de stockage Azure et `<accountkey>` par la clé d’accès du compte de stockage Azure. Pour découvrir comment obtenir votre clé d’accès au stockage, consultez [Gérer les clés d’accès au compte de stockage](../../storage/common/storage-account-keys-manage.md).

    ![Service lié Stockage Azure](media/data-factory-use-custom-activities/azure-storage-linked-service.png)
4. Cliquez sur l’option **Déployer** de la barre de commandes pour déployer le service lié.

#### <a name="create-azure-batch-linked-service"></a>Créer un service lié Azure Batch
1. Dans Data Factory Editor, dans la barre d’outils, cliquez sur **... Plus** dans la barre de commandes, cliquez sur **Nouveau calcul**, puis sélectionnez **Azure Batch** dans le menu.

    ![Nouveau calcul - Azure Batch](media/data-factory-use-custom-activities/new-azure-compute-batch.png)
2. Apportez les modifications suivantes au script JSON :

   1. Spécifiez le nom du compte Azure Batch pour la propriété **accountName** . L’**URL** dans le **panneau du compte Azure Batch** est au format suivant : `http://accountname.region.batch.azure.com`. Pour la propriété **batchUri** dans le JSON, vous devez supprimer `accountname.` de l’URL et utiliser `accountname` pour la propriété JSON `accountName`.
   2. Spécifiez la clé du compte Azure Batch pour la propriété **accessKey** .
   3. Spécifiez le nom du pool que vous avez créé dans le cadre de la configuration requise pour la propriété **poolName** . Vous pouvez aussi spécifier l’ID du pool au lieu du nom du pool.
   4. Spécifiez l’URI Azure Batch pour la propriété **batchUri** . Exemple : `https://westus.batch.azure.com`.
   5. Spécifiez **AzureStorageLinkedService** for the **linkedServiceName** .

        ```json
        {
          "name": "AzureBatchLinkedService",
          "properties": {
            "type": "AzureBatch",
            "typeProperties": {
              "accountName": "myazurebatchaccount",
              "batchUri": "https://westus.batch.azure.com",
              "accessKey": "<yourbatchaccountkey>",
              "poolName": "myazurebatchpool",
              "linkedServiceName": "AzureStorageLinkedService"
            }
          }
        }
        ```

       Pour la propriété **poolName** , vous pouvez également spécifier l’ID du pool au lieu du nom du pool.

### <a name="step-3-create-datasets"></a>Étape 3 : Créez les jeux de données
Dans cette étape, vous allez créer des jeux de données pour représenter les données d’entrée et de sortie.

#### <a name="create-input-dataset"></a>Créer le jeu de données d’entrée
1. Dans **l’éditeur** de la fabrique de données, cliquez sur **... Plus** dans la barre de commandes, cliquez sur **Nouveau jeu de données**, puis sélectionnez **Stockage Blob Azure** dans le menu déroulant.
2. Remplacez le script JSON affiché dans le volet droit par l’extrait de code JSON suivant :

    ```json
    {
        "name": "InputDataset",
        "properties": {
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService",
            "typeProperties": {
                "folderPath": "adftutorial/customactivityinput/",
                "format": {
                    "type": "TextFormat"
                }
            },
            "availability": {
                "frequency": "Hour",
                "interval": 1
            },
            "external": true,
            "policy": {}
        }
    }
    ```

   Vous créerez un pipeline plus loin dans cette procédure pas à pas avec comme heure de début : 2016-11-16T00:00:00Z et comme heure de fin : 2016-11-16T05:00:00Z. Ce pipeline est planifié pour produire des données toutes les heures, ce qui signifie que l’on obtient cinq tranches d’entrée/sortie comprises entre **00**:00:00 -> **05**:00:00.

   Les paramètres **frequency** et **interval** du jeu de données d’entrée sont respectivement définis sur **Hour** et sur **1**, ce qui signifie que la tranche d’entrée est disponible toutes les heures. Dans cet exemple, il s’agit du même fichier (file.txt) que celui contenu dans le dossier d’entrée.

   Voici les heures de début de chaque tranche, chacune étant représentée par la variable système SliceStart dans l’extrait de code JSON ci-dessus.
3. Cliquez sur **Déployer** sur la barre d’outils pour créer et déployer le **jeu de données d’entrée**. Vérifiez que le message **TABLE CORRECTEMENT CRÉÉE** s’affiche dans la barre de titre de l’éditeur.

#### <a name="create-an-output-dataset"></a>Créer un jeu de données de sortie
1. Dans **Data Factory Editor**, cliquez sur **... Plus** dans la barre de commandes, cliquez sur **Nouveau jeu de données**, puis sélectionnez **Stockage Blob Azure**.
2. Remplacez le script JSON affiché dans le volet droit par le script JSON suivant :

    ```JSON
    {
        "name": "OutputDataset",
        "properties": {
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService",
            "typeProperties": {
                "fileName": "{slice}.txt",
                "folderPath": "adftutorial/customactivityoutput/",
                "partitionedBy": [
                    {
                        "name": "slice",
                        "value": {
                            "type": "DateTime",
                            "date": "SliceStart",
                            "format": "yyyy-MM-dd-HH"
                        }
                    }
                ]
            },
            "availability": {
                "frequency": "Hour",
                "interval": 1
            }
        }
    }
    ```

     L’emplacement de sortie est le suivant : **adftutorial/customactivityoutput/** et le nom du fichier de sortie est aaaa-MM-dd-HH.txt, où « aaaa-MM-dd-HH.txt » correspond à l’année, au mois, au jour et à l’heure de la tranche produite. Pour en savoir plus, consultez la page [Référence du développeur][adf-developer-reference].

    Un objet blob/fichier de sortie est généré pour chaque tranche d’entrée. Voici la procédure de nommage des fichiers de sortie pour chaque tranche. Tous les fichiers de sortie sont générés dans un seul dossier de sortie : **adftutorial\customactivityoutput**.

   | Tranche | Heure de début | Fichier de sortie |
   |:--- |:--- |:--- |
   | 1 |2016-11-16T00:00:00 |2016-11-16-00.txt |
   | 2 |2016-11-16T01:00:00 |2016-11-16-01.txt |
   | 3 |2016-11-16T02:00:00 |2016-11-16-02.txt |
   | 4 |2016-11-16T03:00:00 |2016-11-16-03.txt |
   | 5 |2016-11-16T04:00:00 |2016-11-16-04.txt |

    N’oubliez pas que tous les fichiers d’un dossier d’entrée font partie d’une tranche associée aux heures de début indiquées ci-dessus. Lorsque cette tranche est traitée, l’activité personnalisée parcourt chaque fichier et génère une ligne dans le fichier de sortie avec le nombre d’occurrences du terme de recherche (« Microsoft »). Si le dossier d’entrée comporte trois fichiers, trois lignes apparaissent dans le fichier de sortie pour chaque tranche horaire : 2016-11-16-00.txt, 2016-11-16:01:00:00.txt, et ainsi de suite.
3. Cliquez sur **Déployer** dans la barre de commandes pour déployer le **jeu de données de sortie**.

### <a name="create-and-run-a-pipeline-that-uses-the-custom-activity"></a>Créer et exécuter un pipeline qui utilise l’activité personnalisée
1. Dans Data Factory Editor, dans la barre d’outils, cliquez sur **... Plus**, puis sélectionnez **Nouveau pipeline** dans la barre de commandes.
2. Remplacez le script JSON affiché dans le volet droit par le script JSON suivant :

    ```JSON
    {
      "name": "ADFTutorialPipelineCustom",
      "properties": {
        "description": "Use custom activity",
        "activities": [
          {
            "Name": "MyDotNetActivity",
            "Type": "DotNetActivity",
            "Inputs": [
              {
                "Name": "InputDataset"
              }
            ],
            "Outputs": [
              {
                "Name": "OutputDataset"
              }
            ],
            "LinkedServiceName": "AzureBatchLinkedService",
            "typeProperties": {
              "AssemblyName": "MyDotNetActivity.dll",
              "EntryPoint": "MyDotNetActivityNS.MyDotNetActivity",
              "PackageLinkedService": "AzureStorageLinkedService",
              "PackageFile": "customactivitycontainer/MyDotNetActivity.zip",
              "extendedProperties": {
                "SliceStart": "$$Text.Format('{0:yyyyMMddHH-mm}', Time.AddMinutes(SliceStart, 0))"
              }
            },
            "Policy": {
              "Concurrency": 2,
              "ExecutionPriorityOrder": "OldestFirst",
              "Retry": 3,
              "Timeout": "00:30:00",
              "Delay": "00:00:00"
            }
          }
        ],
        "start": "2016-11-16T00:00:00Z",
        "end": "2016-11-16T05:00:00Z",
        "isPaused": false
      }
    }
    ```

    Notez les points suivants :

   * **Concurrency** est défini sur **2** pour que les deux tranches soient traitées en parallèle sur 2 machines virtuelles dans le pool Azure Batch.
   * Il existe une activité dans la section des activités ; elle est du type suivant : **DotNetActivity**.
   * **AssemblyName** est défini sur le nom de la DLL : **MyDotnetActivity.dll**.
   * Le paramètre **EntryPoint** est défini sur **MyDotNetActivityNS.MyDotNetActivity**.
   * **PackageLinkedService** est défini sur **AzureStorageLinkedService**, qui pointe vers le stockage d’objets blob contenant le fichier .zip de l’activité personnalisée. Si vous utilisez des comptes de stockage différents pour les fichiers d’entrée/sortie et le fichier zip de l’activité personnalisée, vous créez un autre service lié Stockage Azure. Cet article suppose que vous utilisez le même compte Stockage Azure.
   * Le paramètre **PackageFile** est défini sur **customactivitycontainer/MyDotNetActivity.zip**. Il est au format : conteneur_du_zip/nom_du_zip.zip.
   * L’activité personnalisée utilise **InputDataset** comme entrée et **OutputDataset** comme sortie.
   * La propriété linkedServiceName de l’activité personnalisée pointe vers **AzureBatchLinkedService**, ce qui indique à Azure Data Factory que l’activité personnalisée doit s’exécuter sur des machines virtuelles Azure Batch.
   * La propriété **isPaused** est définie sur **false**. Le pipeline s’exécute immédiatement dans cet exemple car les tranches débutent à une date antérieure. Vous pouvez définir cette propriété sur true pour suspendre le pipeline et lui réaffecter la valeur false pour reprendre l’exécution.
   * **Cinq** heures séparent les heures de **début** et de **fin**, et les tranches sont produites toutes les heures, ce qui signifie que cinq tranches sont produites par le pipeline.
3. Cliquez sur **Déployer** dans la barre de commandes pour déployer le pipeline.

### <a name="monitor-the-pipeline"></a>Surveiller le pipeline
1. Dans le panneau Data Factory du portail Azure, cliquez sur **Diagramme**.

    ![Vignette de diagramme](./media/data-factory-use-custom-activities/DataFactoryBlade.png)
2. Dans la vue de diagramme, cliquez sur OutputDataset.

    ![Vue schématique](./media/data-factory-use-custom-activities/diagram.png)
3. Les cinq tranches de sortie doivent être à l’état Prêt. Dans le cas contraire, cela signifie qu’elles n’ont pas encore été générées.

   ![Tranches de sortie](./media/data-factory-use-custom-activities/OutputSlices.png)
4. Vérifiez que les fichiers de sortie sont générés dans le stockage d’objets blob, dans le conteneur **adftutorial** .

   ![Sortie de l’activité personnalisée][image-data-factory-output-from-custom-activity]
5. Si vous ouvrez le fichier de sortie, la sortie doit avoir l’aspect suivant :

    ```
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2016-11-16-00/file.txt.
    ```
6. Utilisez le [portail Azure][azure-preview-portal] ou les applets de commande Azure PowerShell pour surveiller votre fabrique de données, pipelines et jeux de données. Vous pouvez voir les messages de l’outil **ActivityLogger** dans le code concernant l’activité personnalisée dans les journaux d’activité (plus précisément, user-0.log) que vous pouvez télécharger à partir du portail ou à l’aide d’applets de commande.

   ![Téléchargement des journaux d’activités personnalisées][image-data-factory-download-logs-from-custom-activity]

Pour découvrir la procédure détaillée de surveillance des jeux de données et des pipelines, consultez l’article [Surveiller et gérer les pipelines](data-factory-monitor-manage-pipelines.md) .

## <a name="data-factory-project-in-visual-studio"></a>Projet Data Factory dans Visual Studio
Vous pouvez créer et publier des entités Data Factory à l’aide de Visual Studio au lieu d’utiliser le portail Azure. Pour en savoir plus sur la création et la publication d’entités Data Factory à l’aide de Visual Studio, consultez [Créer votre premier pipeline à l’aide de Visual Studio](data-factory-build-your-first-pipeline-using-vs.md) et [Copier des données à partir d’un objet blob Azure vers SQL Azure](data-factory-copy-activity-tutorial-using-visual-studio.md).

Si vous créez un projet Data Factory dans Visual Studio, suivez les étapes supplémentaires ci-dessous :

1. Ajoutez le projet Data Factory à la solution Visual Studio qui contient le projet d’activité personnalisée.
2. Ajoutez une référence au projet d’activité .NET à partir du projet Data Factory. Cliquez avec le bouton droit sur le projet Data Factory, pointez sur **Ajouter**, puis cliquez sur **Référence**.
3. Dans la boîte de dialogue **Ajouter une référence**, sélectionnez le projet **MyDotNetActivity**, puis cliquez sur **OK**.
4. Créez et publiez la solution.

    > [!IMPORTANT]
    > Lorsque vous publiez des entités Data Factory, un fichier zip est automatiquement créé pour vous et chargé vers le conteneur d’objets blob customactivitycontainer. Si le conteneur n’existe pas, il est également créé automatiquement.

## <a name="data-factory-and-batch-integration"></a>Intégration de Data Factory et Batch
Le service Data Factory crée un travail dans Azure Batch sous le nom **adf-poolname:job-xxx**. Cliquez sur **Travaux** dans le menu de gauche.

![Azure Data Factory - Travaux Batch](media/data-factory-use-custom-activities/data-factory-batch-jobs.png)

Une tâche est créée pour chaque exécution d’activité d’une tranche. Si cinq tranches sont prêtes à être traitées, cinq tâches seront créées dans ce travail. S’il existe plusieurs nœuds de calcul dans le pool Batch, deux tranches ou plus peuvent s’exécuter en parallèle. Vous pouvez également avoir plusieurs tranches exécutées sur le même nœud de calcul si le nombre maximum de tâches par nœud de calcul est défini sur une valeur supérieure à 1.

![Azure Data Factory - Tâches de travaux Batch](media/data-factory-use-custom-activities/data-factory-batch-job-tasks.png)

Le diagramme suivant illustre la relation entre les tâches Azure Data Factory et les tâches Batch.

![Data Factory et Batch](./media/data-factory-use-custom-activities/DataFactoryAndBatch.png)

## <a name="troubleshoot-failures"></a>Résoudre les défaillances
Quelques techniques de base peuvent être utilisées pour résoudre les défaillances :

1. Si vous voyez l’erreur suivante, vous utilisez peut-être un stockage Blob chaud/froid plutôt qu’un stockage Blob Azure à usage général. Chargez le fichier zip dans un **compte de stockage Blob Azure à usage général**.

    ```
    Error in Activity: Job encountered scheduling error. Code: BlobDownloadMiscError Category: ServerError Message: Miscellaneous error encountered while downloading one of the specified Azure Blob(s).
    ```
2. Si vous voyez l’erreur suivante, confirmez que le nom de la classe figurant dans le fichier CS correspond au nom que vous avez spécifié pour la propriété **EntryPoint** dans le JSON du pipeline. Dans la procédure pas à pas, le nom de la classe est : MyDotNetActivity et le point d’entrée dans le JSON est : MyDotNetActivityNS.**MyDotNetActivity**.

    ```
    MyDotNetActivity assembly does not exist or doesn't implement the type Microsoft.DataFactories.Runtime.IDotNetActivity properly
    ```

   Si les noms ne correspondent pas, vérifiez que tous les fichiers binaires se trouvent dans le **dossier racine** du fichier zip. Autrement dit, lorsque vous ouvrez le fichier .zip, vous devez voir tous les fichiers dans le dossier racine, et non dans les sous-dossiers.
3. Si la tranche d’entrée n’est pas définie sur **Prêt**, vérifiez que la structure du dossier d’entrée est correcte et que le fichier **file.txt** est présent dans les dossiers d’entrée.
3. Dans la méthode **Execute** de votre activité personnalisée, utilisez l’objet **IActivityLogger** pour journaliser les informations qui vous aident à résoudre d’éventuels problèmes. Les messages enregistrés s’affichent dans les fichiers journaux utilisateur (un ou plusieurs fichiers nommés user-0.log, user-1.log, user-2.log, etc.).

   Dans le panneau **OutputDataset**, cliquez sur la tranche pour afficher le panneau **TRANCHE DE DONNÉES** correspondant à cette tranche. Les **activités exécutées** pour cette tranche s’affichent. Vous devez normalement voir une exécution d’activité pour la tranche. Si vous cliquez sur Exécuter dans la barre de commandes, vous pouvez démarrer une autre exécution d’activité pour la même tranche.

   Lorsque vous cliquez sur l’exécution d’activité, le panneau **DÉTAILS DE L’EXÉCUTION D’ACTIVITÉ** s’affiche avec une liste de fichiers journaux. Les messages consignés s’afficheront dans le fichier user_0.log. Lorsqu’une erreur se produit, vous verrez trois exécutions d’activité car le nombre de tentatives est défini sur 3 dans le script JSON du pipeline et de l’activité. Lorsque vous cliquez sur l’exécution de l’activité, vous accédez aux fichiers journaux qui vous permettront de résoudre l’erreur.

   Dans la liste des fichiers journaux, cliquez sur le fichier **user-0.log**. Le volet droit affiche les résultats de l’utilisation de la méthode **IActivityLogger.Write** . Si vous ne voyez pas tous les messages, vérifiez si vous avez plusieurs fichiers journaux nommés user_1.log, user_2.log, etc. Si ce n’est pas le cas, le code peut avoir échoué après le dernier message enregistré.

   En outre, consultez **system-0.log** pour vérifier les exceptions et messages d’erreur système éventuels.
4. Incluez le fichier **PDB** dans le fichier zip afin que les détails de l’erreur incluent des informations telles que la **pile des appels** quand une erreur se produit.
5. Tous les fichiers contenus dans le fichier zip de l’activité personnalisée doivent se trouver au **premier niveau** et ne doivent pas contenir de sous-dossiers.
6. Assurez-vous que les paramètres **assemblyName** (MyDotNetActivity.dll), **entryPoint** (MyDotNetActivityNS.MyDotNetActivity), **packageFile** (customactivitycontainer/Mydotnetactivity.zip) et **packageLinkedService** (qui doit pointer vers le Stockage Blob Azure **à usage général** contenant le fichier .zip) sont définis sur des valeurs correctes.
7. Si vous avez corrigé une erreur et souhaitez relancer le traitement de la tranche, cliquez avec le bouton droit sur la tranche dans le panneau **OutputDataset** puis cliquez sur **Exécuter**.
8. Si vous voyez l’erreur suivante, vous utilisez une version du package Stockage Azure supérieure à 4.3.0. Le lanceur du service Data Factory requiert la version 4.3 du package Windows Azure Storage. Consultez la section [Isolation du domaine d’application](#appdomain-isolation) pour obtenir une autre solution si vous devez utiliser la dernière version de l’assembly du stockage Azure.

    ```
    Error in Activity: Unknown error in module: System.Reflection.TargetInvocationException: Exception has been thrown by the target of an invocation. ---> System.TypeLoadException: Could not load type 'Microsoft.WindowsAzure.Storage.Blob.CloudBlob' from assembly 'Microsoft.WindowsAzure.Storage, Version=4.3.0.0, Culture=neutral,
    ```

    Si vous pouvez utiliser la version 4.3.0 du package du stockage Azure, supprimez la référence existante à la version supérieure à 4.3.0 du package du stockage Azure. Exécutez ensuite la commande suivante à partir de la console du gestionnaire de package NuGet.

    ```powershell
    Install-Package WindowsAzure.Storage -Version 4.3.0
    ```

    Créez le projet. Supprimez la version supérieure à 4.3.0 de l'assembly Stockage Azure du dossier bin/Debug. Créez un fichier zip avec les fichiers binaires et le fichier PDB. Remplacez l’ancien fichier zip par celui-ci dans le conteneur d’objets blob (customactivitycontainer). Exécutez de nouveau les tranches ayant échoué (en cliquant avec le bouton droit sur la tranche, puis en choisissant Exécuter).
8. L’activité personnalisée n’utilise pas le ficher **app.config** de votre package. Par conséquent, si votre code lit des chaînes de connexion dans le fichier de configuration, il ne fonctionne pas lors de l’exécution. Quand vous utilisez Azure Batch, la meilleure pratique consiste à stocker les clés secrètes dans un coffre de clés **Azure KeyVault**, à utiliser un principal de service basé sur certificat pour protéger le **coffre de clés** et à distribuer le certificat à un pool Azure Batch. L’activité personnalisée .NET peut alors accéder aux secrets du coffre de clés au moment de l’exécution. Cette solution est générique et peut s’adapter à n’importe quel type de clé secrète, et pas uniquement aux chaînes de connexion.

   Il existe une solution plus simple (mais non recommandée) : vous pouvez créer un **service lié SQL Azure** avec des paramètres de chaîne de connexion, puis créer un jeu de données qui utilise le service lié et chaîner le jeu de données à l’activité .NET personnalisée en tant que jeu de données d’entrée factice. Vous pouvez ensuite accéder à la chaîne de connexion du service lié dans le code de l’activité personnalisée.

## <a name="update-custom-activity"></a>Mettre à jour l’activité personnalisée
Si vous mettez à jour le code de l’activité personnalisée, créez et téléchargez le fichier .zip qui contient les nouveaux fichiers binaires pour le stockage d’objets blob.

## <a name="appdomain-isolation"></a>Isolation du domaine d’application
Consultez [Cross AppDomain Sample](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/CrossAppDomainDotNetActivitySample) (Exemple d’isolation entre domaines d’application) pour savoir comment créer une activité personnalisée qui n’est pas limitée aux versions d’assembly utilisées par le lanceur d’Azure Data Factory (par exemple, WindowsAzure.Storage v4.3.0, Newtonsoft.Json v6.0.x, et ainsi de suite).

## <a name="access-extended-properties"></a>Accéder aux propriétés étendues
Vous pouvez déclarer des propriétés étendues dans l’activité JSON, comme dans l’exemple suivant :

```JSON
"typeProperties": {
  "AssemblyName": "MyDotNetActivity.dll",
  "EntryPoint": "MyDotNetActivityNS.MyDotNetActivity",
  "PackageLinkedService": "AzureStorageLinkedService",
  "PackageFile": "customactivitycontainer/MyDotNetActivity.zip",
  "extendedProperties": {
    "SliceStart": "$$Text.Format('{0:yyyyMMddHH-mm}', Time.AddMinutes(SliceStart, 0))",
    "DataFactoryName": "CustomActivityFactory"
  }
},
```

Dans l’exemple, il existe deux propriétés étendues : **SliceStart** et **DataFactoryName**. La valeur de SliceStart est basée sur la variable système SliceStart. Consultez [Variables système](data-factory-functions-variables.md) pour obtenir la liste des variables système prises en charge. La valeur de DataFactoryName est codée en dur sous la forme CustomActivityFactory.

Pour accéder à ces propriétés étendues dans la méthode **Execute**, utilisez un code semblable au suivant :

```csharp
// to get extended properties (for example: SliceStart)
DotNetActivity dotNetActivity = (DotNetActivity)activity.TypeProperties;
string sliceStartString = dotNetActivity.ExtendedProperties["SliceStart"];

// to log all extended properties
IDictionary<string, string> extendedProperties = dotNetActivity.ExtendedProperties;
logger.Write("Logging extended properties if any...");
foreach (KeyValuePair<string, string> entry in extendedProperties)
{
    logger.Write("<key:{0}> <value:{1}>", entry.Key, entry.Value);
}
```

## <a name="auto-scaling-of-azure-batch"></a>Mise à l’échelle automatique d’Azure Batch
Vous pouvez aussi créer un pool Azure Batch avec la fonctionnalité **autoscale** . Par exemple, vous pouvez créer un pool Azure Batch avec 0 machine virtuelle dédiée et une formule de mise à l’échelle automatique en fonction du nombre de tâches en attente.

L’exemple de formule fourni ici entraîne le comportement suivant : Lors de sa création, le pool ne contient qu’une seule machine virtuelle. La métrique $PendingTasks définit le nombre de tâches dans l’état En cours d’exécution + Actif (en file d’attente).  Cette formule recherche le nombre moyen de tâches en attente au cours des 180 dernières secondes et définit TargetDedicated en conséquence. Elle garantit que TargetDedicated ne va jamais au-delà de 25 machines virtuelles. Par conséquent, à mesure que de nouvelles tâches sont envoyées, le pool s’accroît automatiquement et, au fil de la réalisation des tâches, les machines virtuelles se libèrent une à une et la mise à l’échelle automatique réduit ces machines virtuelles. Vous pouvez ajuster startingNumberOfVMs et maxNumberofVMs selon vos besoins.

Formule de mise à l’échelle automatique :

```
startingNumberOfVMs = 1;
maxNumberofVMs = 25;
pendingTaskSamplePercent = $PendingTasks.GetSamplePercent(180 * TimeInterval_Second);
pendingTaskSamples = pendingTaskSamplePercent < 70 ? startingNumberOfVMs : avg($PendingTasks.GetSample(180 * TimeInterval_Second));
$TargetDedicated=min(maxNumberofVMs,pendingTaskSamples);
```

Pour plus d’informations, consultez [Mettre automatiquement à l’échelle les nœuds de calcul dans un pool Azure Batch](../../batch/batch-automatic-scaling.md) .

Si le pool utilise la valeur par défaut du paramètre [autoScaleEvaluationInterval](/rest/api/batchservice/pool/enableautoscale), le service Batch peut mettre 15 à 30 minutes à préparer la machine virtuelle avant d’exécuter l’activité personnalisée.  Si le pool utilise une autre valeur pour autoScaleEvaluationInterval, le service Batch peut prendre la durée d’autoScaleEvaluationInterval + 10 minutes.


## <a name="create-a-custom-activity-by-using-net-sdk"></a>Créer une activité personnalisée à l’aide du kit .NET SDK
Dans la procédure pas à pas de cet article, vous créez une fabrique de données avec un pipeline qui utilise l’activité personnalisée à l’aide du portail Azure. Le code suivant montre comment créer la fabrique de données à l’aide du kit .NET SDK à la place. Vous trouverez plus d’informations sur l’utilisation du Kit SDK pour créer des pipelines par programme dans l’article [Créer un pipeline avec une activité de copie à l’aide de l’API .NET](data-factory-copy-activity-tutorial-using-dotnet-api.md).

```csharp
using System;
using System.Configuration;
using System.Collections.ObjectModel;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure;
using Microsoft.Azure.Management.DataFactories;
using Microsoft.Azure.Management.DataFactories.Models;
using Microsoft.Azure.Management.DataFactories.Common.Models;

using Microsoft.IdentityModel.Clients.ActiveDirectory;
using System.Collections.Generic;

namespace DataFactoryAPITestApp
{
    class Program
    {
        static void Main(string[] args)
        {
            // create data factory management client

            // TODO: replace ADFTutorialResourceGroup with the name of your resource group.
            string resourceGroupName = "ADFTutorialResourceGroup";

            // TODO: replace APITutorialFactory with a name that is globally unique. For example: APITutorialFactory04212017
            string dataFactoryName = "APITutorialFactory";

            TokenCloudCredentials aadTokenCredentials = new TokenCloudCredentials(
                ConfigurationManager.AppSettings["SubscriptionId"],
                GetAuthorizationHeader().Result);

            Uri resourceManagerUri = new Uri(ConfigurationManager.AppSettings["ResourceManagerEndpoint"]);

            DataFactoryManagementClient client = new DataFactoryManagementClient(aadTokenCredentials, resourceManagerUri);

            Console.WriteLine("Creating a data factory");
            client.DataFactories.CreateOrUpdate(resourceGroupName,
                new DataFactoryCreateOrUpdateParameters()
                {
                    DataFactory = new DataFactory()
                    {
                        Name = dataFactoryName,
                        Location = "westus",
                        Properties = new DataFactoryProperties()
                    }
                }
            );

            // create a linked service for input data store: Azure Storage
            Console.WriteLine("Creating Azure Storage linked service");
            client.LinkedServices.CreateOrUpdate(resourceGroupName, dataFactoryName,
                new LinkedServiceCreateOrUpdateParameters()
                {
                    LinkedService = new LinkedService()
                    {
                        Name = "AzureStorageLinkedService",
                        Properties = new LinkedServiceProperties
                        (
                            // TODO: Replace <accountname> and <accountkey> with name and key of your Azure Storage account.
                            new AzureStorageLinkedService("DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>")
                        )
                    }
                }
            );

            // create a linked service for output data store: Azure SQL Database
            Console.WriteLine("Creating Azure Batch linked service");
            client.LinkedServices.CreateOrUpdate(resourceGroupName, dataFactoryName,
                new LinkedServiceCreateOrUpdateParameters()
                {
                    LinkedService = new LinkedService()
                    {
                        Name = "AzureBatchLinkedService",
                        Properties = new LinkedServiceProperties
                        (
                            // TODO: replace <batchaccountname> and <yourbatchaccountkey> with name and key of your Azure Batch account
                            new AzureBatchLinkedService("<batchaccountname>", "https://westus.batch.azure.com", "<yourbatchaccountkey>", "myazurebatchpool", "AzureStorageLinkedService")
                        )
                    }
                }
            );

            // create input and output datasets
            Console.WriteLine("Creating input and output datasets");
            string Dataset_Source = "InputDataset";
            string Dataset_Destination = "OutputDataset";

            Console.WriteLine("Creating input dataset of type: Azure Blob");
            client.Datasets.CreateOrUpdate(resourceGroupName, dataFactoryName,

                new DatasetCreateOrUpdateParameters()
                {
                    Dataset = new Dataset()
                    {
                        Name = Dataset_Source,
                        Properties = new DatasetProperties()
                        {
                            LinkedServiceName = "AzureStorageLinkedService",
                            TypeProperties = new AzureBlobDataset()
                            {
                                FolderPath = "adftutorial/customactivityinput/",
                                Format = new TextFormat()
                            },
                            External = true,
                            Availability = new Availability()
                            {
                                Frequency = SchedulePeriod.Hour,
                                Interval = 1,
                            },

                            Policy = new Policy() { }
                        }
                    }
                });

            Console.WriteLine("Creating output dataset of type: Azure Blob");
            client.Datasets.CreateOrUpdate(resourceGroupName, dataFactoryName,
                new DatasetCreateOrUpdateParameters()
                {
                    Dataset = new Dataset()
                    {
                        Name = Dataset_Destination,
                        Properties = new DatasetProperties()
                        {
                            LinkedServiceName = "AzureStorageLinkedService",
                            TypeProperties = new AzureBlobDataset()
                            {
                                FileName = "{slice}.txt",
                                FolderPath = "adftutorial/customactivityoutput/",
                                PartitionedBy = new List<Partition>()
                                {
                                    new Partition()
                                    {
                                        Name = "slice",
                                        Value = new DateTimePartitionValue()
                                        {
                                            Date = "SliceStart",
                                            Format = "yyyy-MM-dd-HH"
                                        }
                                    }
                                }
                            },
                            Availability = new Availability()
                            {
                                Frequency = SchedulePeriod.Hour,
                                Interval = 1,
                            },
                        }
                    }
                });

            Console.WriteLine("Creating a custom activity pipeline");
            DateTime PipelineActivePeriodStartTime = new DateTime(2017, 3, 9, 0, 0, 0, 0, DateTimeKind.Utc);
            DateTime PipelineActivePeriodEndTime = PipelineActivePeriodStartTime.AddMinutes(60);
            string PipelineName = "ADFTutorialPipelineCustom";

            client.Pipelines.CreateOrUpdate(resourceGroupName, dataFactoryName,
                new PipelineCreateOrUpdateParameters()
                {
                    Pipeline = new Pipeline()
                    {
                        Name = PipelineName,
                        Properties = new PipelineProperties()
                        {
                            Description = "Use custom activity",

                            // Initial value for pipeline's active period. With this, you won't need to set slice status
                            Start = PipelineActivePeriodStartTime,
                            End = PipelineActivePeriodEndTime,
                            IsPaused = false,

                            Activities = new List<Activity>()
                            {
                                new Activity()
                                {
                                    Name = "MyDotNetActivity",
                                    Inputs = new List<ActivityInput>()
                                    {
                                        new ActivityInput() {
                                            Name = Dataset_Source
                                        }
                                    },
                                    Outputs = new List<ActivityOutput>()
                                    {
                                        new ActivityOutput()
                                        {
                                            Name = Dataset_Destination
                                        }
                                    },
                                    LinkedServiceName = "AzureBatchLinkedService",
                                    TypeProperties = new DotNetActivity()
                                    {
                                        AssemblyName = "MyDotNetActivity.dll",
                                        EntryPoint = "MyDotNetActivityNS.MyDotNetActivity",
                                        PackageLinkedService = "AzureStorageLinkedService",
                                        PackageFile = "customactivitycontainer/MyDotNetActivity.zip",
                                        ExtendedProperties = new Dictionary<string, string>()
                                        {
                                            { "SliceStart", "$$Text.Format('{0:yyyyMMddHH-mm}', Time.AddMinutes(SliceStart, 0))"}
                                        }
                                    },
                                    Policy = new ActivityPolicy()
                                    {
                                        Concurrency = 2,
                                        ExecutionPriorityOrder = "OldestFirst",
                                        Retry = 3,
                                        Timeout = new TimeSpan(0,0,30,0),
                                        Delay = new TimeSpan()
                                    }
                                }
                            }
                        }
                    }
                });
        }

        public static async Task<string> GetAuthorizationHeader()
        {
            AuthenticationContext context = new AuthenticationContext(ConfigurationManager.AppSettings["ActiveDirectoryEndpoint"] + ConfigurationManager.AppSettings["ActiveDirectoryTenantId"]);
            ClientCredential credential = new ClientCredential(
                ConfigurationManager.AppSettings["ApplicationId"],
                ConfigurationManager.AppSettings["Password"]);
            AuthenticationResult result = await context.AcquireTokenAsync(
                resource: ConfigurationManager.AppSettings["WindowsManagementUri"],
                clientCredential: credential);

            if (result != null)
                return result.AccessToken;

            throw new InvalidOperationException("Failed to acquire token");
        }
    }
}
```

## <a name="debug-custom-activity-in-visual-studio"></a>Déboguer une activité personnalisée dans Visual Studio
L’exemple [Azure Data Factory - Environnement local](https://github.com/gbrueckl/Azure.DataFactory.LocalEnvironment) sur GitHub inclut un outil qui vous permet de déboguer des activités .NET personnalisées dans Visual Studio.

## <a name="sample-custom-activities-on-github"></a>Exemples d’activités personnalisées sur GitHub
| Exemple | Rôle des activités personnalisées |
| --- | --- |
| [Téléchargeur de données HTTP](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/HttpDataDownloaderSample). |Télécharge des données à partir d'un point de terminaison HTTP vers Stockage Blob Azure à l'aide d’une activité C# personnalisée dans Data Factory. |
| [Exemple d’analyse d’opinions Twitter](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/TwitterAnalysisSample-CustomC%23Activity) |Appelle un modèle Azure Machine Learning Studio (classique) et effectue l’analyse, la notation, la prédiction, etc. des opinions. |
| [Exécuter un script R](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/RunRScriptUsingADFSample). |Appelle un script R en exécutant RScript.exe sur votre cluster HDInsight, sur lequel R est installé. |
| [Activité .NET entre AppDomains](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/CrossAppDomainDotNetActivitySample) |Utilise des versions d’assembly différentes de celles utilisées par le lanceur de Data Factory |
| [Retraiter un modèle dans Azure Analysis Services](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/AzureAnalysisServicesProcessSample) |  Retraite un modèle dans Azure Analysis Services. |

[batch-net-library]: ../../batch/quick-run-dotnet.md
[batch-create-account]: ../../batch/batch-account-create-portal.md
[batch-technical-overview]:../../azure-sql/database/sql-database-paas-overview.md
[batch-get-started]: ../../batch/quick-run-dotnet.md
[use-custom-activities]: data-factory-use-custom-activities.md
[troubleshoot]: data-factory-troubleshoot.md
[data-factory-introduction]: data-factory-introduction.md
[azure-powershell-install]: https://github.com/Azure/azure-sdk-tools/releases


[developer-reference]: /previous-versions/azure/dn834987(v=azure.100)
[cmdlet-reference]: https://go.microsoft.com/fwlink/?LinkId=517456

[new-azure-batch-account]: /previous-versions/azure/mt125880(v=azure.100)
[new-azure-batch-pool]: /previous-versions/azure/mt125936(v=azure.100)
[azure-batch-blog]: /archive/blogs/windowshpc/using-azure-powershell-to-manage-azure-batch-account

[nuget-package]: https://go.microsoft.com/fwlink/?LinkId=517478
[adf-developer-reference]: /previous-versions/azure/dn834987(v=azure.100)
[azure-preview-portal]: https://portal.azure.com/

[adfgetstarted]: data-factory-copy-data-from-azure-blob-storage-to-sql-database.md
[hivewalkthrough]: data-factory-data-transformation-activities.md

[image-data-factory-output-from-custom-activity]: ./media/data-factory-use-custom-activities/OutputFilesFromCustomActivity.png

[image-data-factory-download-logs-from-custom-activity]: ./media/data-factory-use-custom-activities/DownloadLogsFromCustomActivity.png
---
title: Traiter des jeux de données volumineux à l’aide de Data Factory et de Batch
description: Décrit comment traiter de grandes quantités de données dans un pipeline Azure Data Factory en utilisant la capacité de traitement parallèle d’Azure Batch.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 365de79db450eda897621ccde3b92478ed93ecc3
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96496158"
---
# <a name="process-large-scale-datasets-by-using-data-factory-and-batch"></a>Traiter des jeux de données volumineux à l’aide de Data Factory et de Batch
> [!NOTE]
> Cet article s’applique à la version 1 d’Azure Data Factory, qui est mise à la disposition générale. Si vous utilisez la version actuelle du service Data Factory, consultez [Activités personnalisées dans Data Factory](../transform-data-using-dotnet-custom-activity.md).

Cet article décrit une architecture d’un exemple de solution qui déplace et traite des jeux de données volumineux de manière automatique et planifiée. Il fournit également une procédure de bout en bout pour implémenter la solution à l’aide d’Azure Data Factory et d’Azure Batch.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Cet article est plus long que nos articles habituels, car il contient une procédure pas à pas avec un exemple de solution complète. Si vous débutez avec Azure Batch et Azure Data Factory, vous découvrirez ces services et comment ils fonctionnent ensemble. Si vous connaissez un peu ces services et que vous concevez/élaborez une solution, vous pouvez vous concentrer sur la section Architecture de l'article. Si vous développez un prototype ou une solution, vous pourrez suivre les instructions détaillées de la procédure pas à pas. N’hésitez pas à nous faire part de vos commentaires sur ce contenu et son utilisation.

Tout d’abord, examinons comment les services Data Factory et Batch permettent de traiter des jeux de données volumineux dans le cloud.     


## <a name="why-azure-batch"></a>Pourquoi Azure Batch ?
 Vous pouvez utiliser Azure Batch pour exécuter efficacement et à grande échelle des applications de calcul haute performance (HPC) en parallèle dans le cloud. Ce service de plateforme planifie l’exécution des travaux nécessitant beaucoup de ressources système sur une collection gérée de machines virtuelles. Il dimensionne automatiquement les ressources de calcul en fonction des besoins de vos travaux.

Avec le service Batch, vous définissez des ressources de calcul Azure pour exécuter vos applications en parallèle et à grande échelle. Vous pouvez exécuter des travaux à la demande ou de manière planifiée. Il est inutile de créer, configurer et gérer manuellement un cluster HPC, certaines machines virtuelles, des réseaux virtuels ou une infrastructure complexe de planification de tâches et de travaux.

 Si vous n’êtes pas familier avec Azure Batch, consultez les articles suivants qui vous aideront à comprendre l’architecture et l’implémentation de la solution décrite dans cet article :   

* [Notions de base d’Azure Batch](../../azure-sql/database/sql-database-paas-overview.md)
* [Aperçu des fonctionnalités d’Azure Batch](../../batch/batch-service-workflow-features.md)

Si vous le souhaitez, consultez la [documentation Azure Batch](../../batch/index.yml) pour en savoir plus sur Azure Batch.

## <a name="why-azure-data-factory"></a>Pourquoi Azure Data Factory ?
Data Factory est un service d’intégration de données dans le cloud qui gère et automatise le déplacement et la transformation des données. Vous pouvez utiliser Azure Data Factory pour créer des pipelines de données managées qui déplacent les données de magasins de données locaux et cloud vers un magasin de données centralisé. Le stockage d’objets blob Azure en est un exemple. Vous pouvez utiliser Azure Data Factory pour traiter/transformer des données à l’aide de services tels qu’Azure HDInsight et Azure Machine Learning. Vous pouvez également planifier des pipelines de données à exécuter de façon planifiée (par exemple, toutes les heures, tous les jours et toutes les semaines). Vous surveillez et gérez les pipelines en un clin d’œil pour identifier les problèmes et prendre les mesures adéquates.

  Si vous n’êtes pas familier avec Azure Data Factory, les articles suivants vous aideront à comprendre l’architecture et l’implémentation de la solution décrite dans cet article :  

* [Présentation de Data Factory](data-factory-introduction.md)
* [Générer votre premier pipeline de données](data-factory-build-your-first-pipeline.md)   

Si vous le souhaitez, consultez la [documentation Data Factory](/rest/api/datafactory/v1/data-factory-data-factory) pour en savoir plus sur Data Factory.

## <a name="data-factory-and-batch-together"></a>Intégration de Data Factory et Batch
Azure Data Factory fournit des activités intégrées. Par exemple, l’activité de copie permet de copier/déplacer les données depuis un magasin de données source vers un magasin de données de destination. L’activité Hive est utilisée pour traiter les données à l’aide de clusters Hadoop (HDInsight) sur Azure. Pour obtenir la liste des activités de transformation prises en charge, consultez [Activités de transformation des données](data-factory-data-transformation-activities.md).

Vous pouvez également créer des activités .NET personnalisées pour déplacer ou traiter des données avec votre propre logique. Vous pouvez exécuter ces activités sur un cluster HDInsight ou sur un pool Azure Batch de machines virtuelles. Lorsque vous utilisez Azure Batch, vous pouvez configurer la mise à l’échelle automatique du pool (ajouter ou supprimer des machines virtuelles en fonction de la charge de travail) selon une formule que vous définissez.     

## <a name="architecture-of-a-sample-solution"></a>Architecture d’une solution exemple
  L’architecture décrite dans cet article est celle d’une solution simple. Elle convient également à des scénarios complexes, tels que la modélisation du risque par les services financiers, le traitement et le rendu d’images, ou l’analyse génomique.

Le diagramme illustre comment Azure Data Factory orchestre le traitement et le déplacement des données. Il montre également comment Azure Batch traite les données en parallèle. Téléchargez et imprimez le diagramme pour le consulter facilement (11 x 17 pouces ou format A3). Pour accéder au diagramme et l’imprimer facilement, consultez [HPC and data orchestration by using Batch and Data Factory (Orchestration de HPC et des données à l’aide d’Azure Batch et d’Azure Data Factory)](https://go.microsoft.com/fwlink/?LinkId=717686).

[![Diagramme de traitement des données à grande échelle](./media/data-factory-data-processing-using-batch/image1.png)](https://go.microsoft.com/fwlink/?LinkId=717686)

La liste suivante fournit les étapes de base du processus. La solution inclut du code et des explications relatives à la génération de la solution de bout en bout.

* **Configurez Azure Batch avec un pool de nœuds de calcul (machines virtuelles).** Vous pouvez spécifier le nombre de nœuds et la taille de chacun d’eux.

* **Créez une instance Azure Data Factory** configurée avec des entités qui représentent respectivement le stockage d’objets blob, le service de calcul d’Azure Batch, les données d’E/S, ainsi qu’un flux de travail/pipeline avec des activités qui déplacent et transforment des données.

* **Créez une activité .NET personnalisée dans le pipeline Azure Data Factory.** L’activité est votre code utilisateur qui s’exécute sur le pool Azure Batch.

* **Stockez de grandes quantités de données d’entrée en tant qu’objets blob dans le Stockage Microsoft Azure.** Les données sont divisées en tranches logiques (généralement basées sur l’heure).

* **Data Factory copie vers l’emplacement secondaire les données qui sont traitées en parallèle**.

* **Azure Data Factory exécute l’activité personnalisée à l’aide du pool alloué par Azure Batch.** Data Factory peut exécuter plusieurs activités simultanément. Chaque activité traite une tranche de données. Les résultats sont stockés dans le stockage Azure.

* **Azure Data Factory déplace les résultats finaux vers un troisième emplacement**, soit pour les distribuer via une application, soit pour les traiter avec d’autres outils.

## <a name="implementation-of-the-sample-solution"></a>Implémentation de la solution exemple
La solution exemple est volontairement simple. Elle vous montre comment utiliser Azure Data Factory et Azure Batch pour traiter des jeux de données. La solution compte le nombre d’occurrences du terme recherché (Microsoft) dans les fichiers d’entrée organisés en une série chronologique. Il renvoie ensuite le nombre dans des fichiers de sortie.

**Durée :** si vous maîtrisez les notions de base d’Azure, d’Azure Data Factory et d’Azure Batch et que vous disposez des éléments requis ci-dessous, cette solution est opérationnelle en 1 à 2 heures.

### <a name="prerequisites"></a>Prérequis
#### <a name="azure-subscription"></a>Abonnement Azure
Si vous n’avez pas d’abonnement Azure, vous pouvez créer rapidement un compte Azure gratuit. Pour plus d’informations, consultez la page [Créez votre compte gratuit Azure dès aujourd’hui](https://azure.microsoft.com/pricing/free-trial/).

#### <a name="azure-storage-account"></a>Compte Azure Storage
Vous utilisez un compte de stockage pour stocker les données dans ce didacticiel. Si vous ne possédez pas encore de compte de stockage, consultez [Create a storage account (Créer un compte de stockage)](../../storage/common/storage-account-create.md). L’exemple de solution utilise un stockage d’objets blob.

#### <a name="azure-batch-account"></a>Compte Azure Batch
Créer un compte de stockage à l’aide du [portail Azure](https://portal.azure.com/). Pour plus d’informations, consultez [Create and manage a Batch account (Créer et gérer un compte Azure Batch)](../../batch/batch-account-create-portal.md). Notez le nom et la clé du compte Azure Batch. Vous pouvez également créer un compte Azure Batch à l’aide de la cmdlet [New-AzBatchAccount](/powershell/module/az.batch/new-azbatchaccount). Pour savoir comment utiliser cette applet de commande, consultez [Get started with Batch PowerShell cmdlets (Prise en main des applets de commande PowerShell d’Azure Batch)](../../batch/batch-powershell-cmdlets-get-started.md).

La solution exemple utilise Azure Batch (indirectement via un pipeline Azure Data Factory) pour traiter des données en parallèle sur un pool de nœuds de calcul (une collection gérée de machines virtuelles).

#### <a name="azure-batch-pool-of-virtual-machines"></a>Pool Azure Batch de machines virtuelles
Créez un pool Azure Batch comprenant au moins deux nœuds de calcul.

1. Dans le [portail Azure](https://portal.azure.com), cliquez sur **Parcourir** dans le menu de gauche, puis cliquez sur **Comptes Batch**.

1. Sélectionnez votre compte Azure Batch pour ouvrir le panneau **Compte Batch**.

1. Sélectionnez la vignette **Pools**.

1. Dans le panneau **Pools**, cliquez sur le bouton **Ajouter** de la barre d’outils pour ajouter un pool.

   a. Entrez un ID pour le pool (**ID du pool**). Notez l’ID du pool. Vous en aurez besoin pour créer la solution Azure Data Factory.

   b. Spécifiez **Windows Server 2012 R2** pour le paramètre **Famille de systèmes d’exploitation**.

   c. Sélectionnez le **niveau tarifaire du nœud**.

   d. Entrez **2** comme valeur du paramètre **Quantité dédiée cible**.

   e. Entrez **2** comme valeur du paramètre **Nombre maximal de tâches par nœud**.

   f. Sélectionnez **OK** pour créer le pool.

#### <a name="azure-storage-explorer"></a>Explorateur de stockage Azure
Vous utilisez [Explorateur Stockage Azure 6](https://azurestorageexplorer.codeplex.com/) ou [CloudXplorer](https://clumsyleaf.com/products/cloudxplorer) (à partir de ClumsyLeaf Software) pour inspecter et modifier les données dans vos projets de stockage. Vous pouvez également examiner et modifier les données dans les journaux d’activité de vos applications hébergées dans le cloud.

1. Créez un conteneur nommé **mycontainer** avec un accès privé (sans accès anonyme).

1. Si vous utilisez CloudXplorer, créez des dossiers et des sous-dossiers avec la structure suivante :

   ![Structure de dossiers et sous-dossiers](./media/data-factory-data-processing-using-batch/image3.png)

   `Inputfolder` et `outputfolder` sont des dossiers de niveau supérieur de `mycontainer`. Le dossier `inputfolder` contient des sous-dossiers horodatés (AAAA-MM-JJ-HH).

   Si vous utilisez Explorateur Stockage Azure, à l’étape suivante, vous chargez les fichiers nommés `inputfolder/2015-11-16-00/file.txt`, `inputfolder/2015-11-16-01/file.txt` et ainsi de suite. Cette étape crée automatiquement les dossiers.

1. Créez sur votre ordinateur un fichier texte **file.txt** contenant le mot clé **Microsoft**. Par exemple, « activité de test personnalisé Microsoft ».

1. Chargez le fichier dans les dossiers d’entrée suivants du stockage d’objets blob Azure :

   ![Dossiers d’entrée](./media/data-factory-data-processing-using-batch/image4.png)

   Si vous utilisez Explorateur Stockage Azure, chargez le fichier **file.txt** dans **mycontainer**. Cliquez sur **Copier** dans la barre d’outils pour créer une copie de l’objet blob. Dans la boîte de dialogue **Copy Blob** (Copie de l’objet blob), remplacez le **nom d’objet blob de destination** par `inputfolder/2015-11-16-00/file.txt`. Répétez cette étape pour créer `inputfolder/2015-11-16-01/file.txt`, `inputfolder/2015-11-16-02/file.txt`, `inputfolder/2015-11-16-03/file.txt`, `inputfolder/2015-11-16-04/file.txt` et ainsi de suite. Cette action crée automatiquement les dossiers.

1. Créez un autre conteneur nommé `customactivitycontainer`. Chargez le fichier zip de l’activité personnalisée dans ce conteneur.

#### <a name="visual-studio"></a>Visual Studio
Installez Microsoft Visual Studio 2012 ou version ultérieure pour créer l’activité Azure Batch personnalisée à utiliser dans la solution Azure Data Factory.

### <a name="high-level-steps-to-create-the-solution"></a>Principales étapes pour créer la solution
1. Créez une activité personnalisée contenant la logique de traitement des données.

1. Créez une fabrique de données qui utilise l’activité personnalisée.

### <a name="create-the-custom-activity"></a>Création de l’activité personnalisée
L’activité personnalisée d’Azure Data Factory est au cœur de cet exemple de solution. Cette solution exemple utilise Azure Batch pour exécuter l’activité personnalisée. Pour plus d’informations sur le développement d’activités personnalisées et leur utilisation dans des pipelines Azure Data Factory, consultez [Use custom activities in a data factory pipeline (Utiliser des activités personnalisées dans un pipeline Azure Data Factory)](data-factory-use-custom-activities.md).

Pour créer une activité personnalisée .NET utilisable dans un pipeline Azure Data Factory, vous devez créer un projet de bibliothèque de classes .NET, avec une classe qui implémente l’interface IDotNetActivity. Cette interface ne possède qu’une méthode : Execute. Voici la signature de la méthode :

```csharp
public IDictionary<string, string> Execute(
            IEnumerable<LinkedService> linkedServices,
            IEnumerable<Dataset> datasets,
            Activity activity,
            IActivityLogger logger)
```

Cette méthode a quelques composants clés qu’il est important de comprendre :

* La méthode accepte quatre paramètres :

  * **linkedServices**. Ce paramètre est une liste énumérable de services liés qui relient les sources de données d’entrée/sortie (par exemple, Azure Blob Storage) à la fabrique de données. Dans cet exemple, il n’y a qu’un service lié de type Stockage Microsoft Azure utilisé à la fois pour l’entrée et la sortie.
  * **jeux de données**. Ce paramètre est une liste énumérable de jeux de données. Vous pouvez utiliser ce paramètre pour obtenir les emplacements et les schémas définis par les jeux de données d’entrée et de sortie.
  * **activity**. Ce paramètre représente l’entité de calcul. En l’occurrence, il s’agit d’un service Azure Batch.
  * **logger**. Il permet d’écrire des commentaires de débogage qui constituent le journal utilisateur du pipeline.
* La méthode retourne un dictionnaire qui peut être utilisé pour enchaîner ultérieurement des activités personnalisées. Cette fonctionnalité n’étant pas encore implémentée, la méthode ne renvoie qu’un dictionnaire vide.

#### <a name="procedure-create-the-custom-activity"></a>Procédure : Création de l’activité personnalisée
1. Créez un projet de bibliothèque de classes .NET dans Visual Studio.

   a. Démarrez Visual Studio 2012/2013/2015.

   b. Sélectionnez **Fichier** > **Nouveau** > **Projet**.

   c. Développez **Modèles**, puis sélectionnez **Visual C\#** . Dans cette procédure pas à pas, vous utilisez C\#, mais vous pouvez utiliser un autre langage .NET pour développer l’activité personnalisée.

   d. Sélectionnez **Bibliothèque de classes** dans la liste des types de projet, sur la droite.

   e. Entrez **MyDotNetActivity** for the **Nom**.

   f. Sélectionnez **C:\\ADF** comme **emplacement**. Créez le dossier **ADF**, s’il n’existe pas.

   g. Sélectionnez **OK** pour créer le projet.

1. Cliquez sur **Outils** > **Gestionnaire de package NuGet** > **Console du Gestionnaire de package**.

1. Dans la Console du Gestionnaire de package, exécutez la commande suivante pour importer Microsoft.Azure.Management.DataFactories :

    ```powershell
    Install-Package Microsoft.Azure.Management.DataFactories
    ```
1. Importez le package NuGet **Azure Storage** dans le projet. Vous en avez besoin, car vous utilisez l’API de stockage d’objets Blob dans cet exemple :

    ```powershell
    Install-Package Az.Storage
    ```
1. Ajoutez les instructions using suivantes au fichier source du projet :

    ```csharp
    using System.IO;
    using System.Globalization;
    using System.Diagnostics;
    using System.Linq;

    using Microsoft.Azure.Management.DataFactories.Models;
    using Microsoft.Azure.Management.DataFactories.Runtime;

    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;
    ```
1. Remplacez le nom de l’espace de noms par **MyDotNetActivityNS**.

    ```csharp
    namespace MyDotNetActivityNS
    ```
1. Remplacez le nom de la classe par **MyDotNetActivity** et dérivez-la de l’interface **IDotNetActivity**, comme indiqué ci-dessous :

    ```csharp
    public class MyDotNetActivity : IDotNetActivity
    ```
1. Implémentez (ajoutez) la méthode **Execute** de l’interface **IDotNetActivity** dans la classe **MyDotNetActivity**. Copiez l’exemple de code suivant dans la méthode. Pour une explication de la logique utilisée dans cette méthode, consultez la section [Méthode Execute](#execute-method).

    ```csharp
    /// <summary>
    /// The Execute method is the only method of IDotNetActivity interface you must implement.
    /// In this sample, the method invokes the Calculate method to perform the core logic.  
    /// </summary>
    public IDictionary<string, string> Execute(
       IEnumerable<LinkedService> linkedServices,
       IEnumerable<Dataset> datasets,
       Activity activity,
       IActivityLogger logger)
    {

       // Declare types for the input and output data stores.
       AzureStorageLinkedService inputLinkedService;

       Dataset inputDataset = datasets.Single(dataset => dataset.Name == activity.Inputs.Single().Name);

       foreach (LinkedService ls in linkedServices)
           logger.Write("linkedService.Name {0}", ls.Name);

       // Use the First method instead of Single because we are using the same
       // Azure Storage linked service for input and output.
       inputLinkedService = linkedServices.First(
           linkedService =>
           linkedService.Name ==
           inputDataset.Properties.LinkedServiceName).Properties.TypeProperties
           as AzureStorageLinkedService;

       string connectionString = inputLinkedService.ConnectionString; // To create an input storage client.
       string folderPath = GetFolderPath(inputDataset);
       string output = string.Empty; // for use later.

       // Create the storage client for input. Pass the connection string.
       CloudStorageAccount inputStorageAccount = CloudStorageAccount.Parse(connectionString);
       CloudBlobClient inputClient = inputStorageAccount.CreateCloudBlobClient();

       // Initialize the continuation token before using it in the do-while loop.
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

           // The Calculate method returns the number of occurrences of
           // the search term "Microsoft" in each blob associated
           // with the data slice.
           //
           // The definition of the method is shown in the next step.
           output = Calculate(blobList, logger, folderPath, ref continuationToken, "Microsoft");

       } while (continuationToken != null);

       // Get the output dataset by using the name of the dataset matched to a name in the Activity output collection.
       Dataset outputDataset = datasets.Single(dataset => dataset.Name == activity.Outputs.Single().Name);

       folderPath = GetFolderPath(outputDataset);

       logger.Write("Writing blob to the folder: {0}", folderPath);

       // Create a storage object for the output blob.
       CloudStorageAccount outputStorageAccount = CloudStorageAccount.Parse(connectionString);
       // Write the name of the file.
       Uri outputBlobUri = new Uri(outputStorageAccount.BlobEndpoint, folderPath + "/" + GetFileName(outputDataset));

       logger.Write("output blob URI: {0}", outputBlobUri.ToString());
       // Create a blob and upload the output text.
       CloudBlockBlob outputBlob = new CloudBlockBlob(outputBlobUri, outputStorageAccount.Credentials);
       logger.Write("Writing {0} to the output blob", output);
       outputBlob.UploadText(output);

       // The dictionary can be used to chain custom activities together in the future.
       // This feature is not implemented yet, so just return an empty dictionary.
       return new Dictionary<string, string>();
    }
    ```
1. Ajoutez les méthodes d’assistance suivantes à la classe. Ces méthodes sont appelées par la méthode **Execute** . Surtout, la méthode **Calculate** isole le code qui effectue une itération dans chaque objet blob.

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

       AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;
       if (blobDataset == null)
       {
           return null;
       }

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

       AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;
       if (blobDataset == null)
       {
           return null;
       }

       return blobDataset.FileName;
    }

    /// <summary>
    /// Iterates through each blob (file) in the folder, counts the number of instances of the search term in the file,
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
    La méthode GetFolderPath renvoie le chemin d’accès au dossier vers lequel pointe le jeu de données et la méthode GetFileName renvoie le nom de l’objet blob/fichier vers lequel pointe le jeu de données.

    ```csharp

    "name": "InputDataset",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "fileName": "file.txt",
            "folderPath": "mycontainer/inputfolder/{Year}-{Month}-{Day}-{Hour}",
    ```

    La méthode Calculate calcule le nombre d’instances du mot-clé Microsoft dans les fichiers d’entrée (objets blob du dossier). Le terme recherché (Microsoft) est codé en dur dans le code.

1. Compilez le projet. Cliquez sur l’option **Générer** du menu, puis sur **Générer la solution**.

1. Démarrez l’Explorateur Windows et accédez au dossier **bin\\debug** ou **bin\\release**. Le choix du dossier varie selon le type de build.

1. Créez un fichier zip **MyDotNetActivity.zip** contenant tous les fichiers binaires dans le dossier **\\bin\\Debug**. Vous pouvez également inclure le fichier MyDotNetActivity.**pdb** afin d’obtenir des détails supplémentaires, comme le numéro de ligne du code source à l’origine du problème en cas de défaillance.

   ![La liste du dossier bin\Debug](./media/data-factory-data-processing-using-batch/image5.png)

1. Chargez le fichier **MyDotNetActivity.zip** en tant qu’objet blob dans le conteneur d’objets blob `customactivitycontainer` du Stockage Blob Azure que le service lié StorageLinkedService dans ADFTutorialDataFactory utilise. S’il n’existe pas déjà, créez le conteneur d’objets blob `customactivitycontainer`.

#### <a name="execute-method"></a>Méthode Execute
Cette section fournit des informations supplémentaires concernant le code dans la méthode Execute.

1. Les membres nécessaires pour l’itération dans la collection d’entrée se trouvent dans l’espace de noms [Microsoft.WindowsAzure.Storage.Blob](/java/api/com.microsoft.azure.storage.blob) . L’itération dans la collection d’objets blob requiert l’utilisation de la classe **BlobContinuationToken**. Vous devez utiliser une boucle do-while en utilisant le jeton pour sortir de la boucle. Pour plus d’informations, consultez [Use Blob storage from .NET (Utiliser le stockage d’objets blob à partir de .NET)](../../storage/blobs/storage-quickstart-blobs-dotnet.md). Une boucle de base est illustrée ici :

    ```csharp
    // Initialize the continuation token.
    BlobContinuationToken continuationToken = null;
    do
    {
    // Get the list of input blobs from the input storage client object.
    BlobResultSegment blobList = inputClient.ListBlobsSegmented(folderPath,

                         true,
                                   BlobListingDetails.Metadata,
                                   null,
                                   continuationToken,
                                   null,
                                   null);
    // Return a string derived from parsing each blob.

     output = Calculate(blobList, logger, folderPath, ref continuationToken, "Microsoft");

    } while (continuationToken != null);

    ```
   Pour plus d’informations, consultez la documentation de la méthode [ListBlobsSegmented](/java/api/com.microsoft.azure.storage.blob.cloudblobcontainer.listblobssegmented).

1. Le code permettant de parcourir l’ensemble d’objets blob s’inscrit logiquement dans la boucle do-while. Dans la méthode **Execute**, la boucle do-while transmet la liste d’objets blob à une méthode nommée **Calculate**. La méthode renvoie une variable de chaîne nommée **output** , qui correspond au résultat de l’itération dans tous les objets blob du segment.

   Elle renvoie le nombre d’occurrences du terme recherché (Microsoft) dans l’objet blob transmis à la méthode **Calculate**.

    ```csharp
    output += string.Format("{0} occurrences of the search term \"{1}\" were found in the file {2}.\r\n", wordCount, searchTerm, inputBlob.Name);
    ```
1. Une fois la méthode **Calculate** exécutée, elle doit écrire le résultat dans un nouvel objet blob. Pour chaque ensemble d’objets blob traités, un nouvel objet blob peut être écrit avec les résultats correspondants. Pour écrire dans un nouvel objet blob, commencez par rechercher le jeu de données de sortie.

    ```csharp
    // Get the output dataset by using the name of the dataset matched to a name in the Activity output collection.
    Dataset outputDataset = datasets.Single(dataset => dataset.Name == activity.Outputs.Single().Name);
    ```
1. Le code appelle également la méthode d’assistance **GetFolderPath** pour récupérer le chemin d’accès au dossier (nom du conteneur de stockage).

    ```csharp
    folderPath = GetFolderPath(outputDataset);
    ```
   La méthode GetFolderPath effectue un cast de l’objet DataSet en un AzureBlobDataSet, qui comporte une propriété nommée FolderPath.

    ```csharp
    AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;

    return blobDataset.FolderPath;
    ```
1. Le code appelle la méthode **GetFileName** pour récupérer le nom du fichier (nom de l’objet blob). Le code est similaire au code précédent utilisé pour obtenir le chemin d’accès au dossier.

    ```csharp
    AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;

    return blobDataset.FileName;
    ```
1. Le nom du fichier est écrit grâce à la création d’un objet URI. Le constructeur d’URI utilise la propriété **BlobEndpoint** pour renvoyer le nom du conteneur. Le nom de fichier et le chemin du dossier sont ajoutés pour construire l’URI de l’objet blob de sortie.  

    ```csharp
    // Write the name of the file.
    Uri outputBlobUri = new Uri(outputStorageAccount.BlobEndpoint, folderPath + "/" + GetFileName(outputDataset));
    ```
1. Une fois le nom du fichier écrit, vous pouvez écrire la chaîne de sortie de la méthode **Calculate** dans un nouvel objet blob :

    ```csharp
    // Create a blob and upload the output text.
    CloudBlockBlob outputBlob = new CloudBlockBlob(outputBlobUri, outputStorageAccount.Credentials);
    logger.Write("Writing {0} to the output blob", output);
    outputBlob.UploadText(output);
    ```

### <a name="create-the-data-factory"></a>Création de la fabrique de données
Dans la section [Création de l’activité personnalisée](#create-the-custom-activity), vous avez créé une activité personnalisée et chargé le fichier zip contenant les fichiers binaires et le fichier PDB dans un conteneur d’objets blob. Dans cette section, vous créez une fabrique de données avec un pipeline qui utilise l’activité personnalisée.

Le jeu de données d’entrée de l’activité personnalisée représente les objets blob (fichiers) contenus dans le dossier d’entrée (`mycontainer\\inputfolder`) du stockage d’objets blob. Le jeu de données de sortie de l’activité représente les objets blob de sortie contenus dans le dossier de sortie (`mycontainer\\outputfolder`) du stockage d’objets blob.

Déposez un ou plusieurs fichiers dans les dossiers d’entrée :

```
mycontainer -\> inputfolder
    2015-11-16-00
    2015-11-16-01
    2015-11-16-02
    2015-11-16-03
    2015-11-16-04
```

Par exemple, déposez un fichier (file.txt) avec le contenu suivant dans chacun des dossiers :

```
test custom activity Microsoft test custom activity Microsoft
```

Chaque dossier d’entrée correspond à une tranche dans Azure Data Factory, même si le dossier contient plusieurs fichiers. Lorsque chaque tranche est traitée par le pipeline, l’activité personnalisée effectue une itération dans tous les objets blob du dossier d’entrée pour la tranche en question.

Vous voyez cinq fichiers de sortie avec le même contenu. Par exemple, le fichier de sortie résultant du traitement du fichier figurant dans le dossier 2015-11-16-00 a le contenu suivant :

```
2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file.txt.
```

Si vous déposez plusieurs fichiers (file.txt, file2.txt, file3.txt) ayant le même contenu dans le dossier d’entrée, le contenu suivant apparaît dans le fichier de sortie. Dans cet exemple, chaque dossier (2015-11-16-00, etc.) correspond à une tranche, même si le dossier contient plusieurs fichiers d’entrée.

```csharp
2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file.txt.
2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file2.txt.
2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file3.txt.
```

Le fichier de sortie comprend désormais trois lignes, une par fichier d’entrée (objet blob) dans le dossier associé à la tranche (2015-11-16-00).

Une tâche est créée pour chaque exécution d’activité. Dans cet exemple, le pipeline ne contient qu’une seule activité. Quand le pipeline traite une tranche, l’activité personnalisée s’exécute sur Azure Batch pour traiter la tranche. Comme il y a cinq tranches (chacune pouvant contenir plusieurs objets blob ou fichiers), cinq tâches sont créées dans Azure Batch. Quand une tâche s’exécute sur Azure Batch, c’est l’activité personnalisée qui s’exécute.

La procédure pas à pas suivante fournit des détails supplémentaires.

#### <a name="step-1-create-the-data-factory"></a>Étape 1 : Création de la fabrique de données
1. Une fois connecté au [portail Azure](https://portal.azure.com/), procédez comme suit :

   a. Cliquez sur **NOUVEAU** dans le menu de gauche.

   b. Dans le panneau **Nouveau**, cliquez sur **Données + Analytique**.

   c. Cliquez sur **Data Factory** dans le panneau **Analyse des données**.

1. Dans le panneau **Nouvelle fabrique de données**, spécifiez le nom **CustomActivityFactory**. Le nom de la fabrique de données doit être un nom global unique. Si l’erreur "Data factory name “CustomActivityFactory” is not available" (Le nom de la fabrique de données « CustomActivityFactory » n’est pas disponible) s’affiche, changez le nom de la fabrique de données. Par exemple, utilisez votre_nomCustomActivityFactory et recréez la fabrique de données.

1. Sélectionnez **NOM DU GROUPE DE RESSOURCES** puis sélectionnez ou créez un groupe de ressources.

1. Vérifiez que l’abonnement et la région correspondant à ceux dans lesquels vous voulez créer la fabrique de données sont corrects.

1. Sélectionnez **Créer** dans le panneau **Nouvelle fabrique de données**.

1. La fabrique de données est créée dans le tableau de bord du portail.

1. Une fois la fabrique de données créée, la page **Fabrique de données** correspondante s’affiche et indique son contenu.

   ![Page de la fabrique de données](./media/data-factory-data-processing-using-batch/image6.png)

#### <a name="step-2-create-linked-services"></a>Étape 2 : Créez des services liés
Les services liés relient des magasins de données ou des services de calcul à une fabrique de données. À cette étape, vous liez votre compte de stockage et votre compte Azure Batch à votre fabrique de données.

#### <a name="create-an-azure-storage-linked-service"></a>Créer un service lié Stockage Azure
1. Cliquez sur la vignette **Créer et déployer** dans le panneau **Fabrique de données** de **CustomActivityFactory**. Le Data Factory Editor apparaît.

1. Sélectionnez **Nouvelle banque de données** dans la barre de commandes et choisissez **Stockage Azure**. Le script JSON que vous utilisez pour créer un service lié au stockage dans l’éditeur s’affiche.

   ![Nouveau magasin de données](./media/data-factory-data-processing-using-batch/image7.png)

1. Remplacez **nom de compte** par le nom de votre compte de stockage. Remplacez **clé de compte** par la clé d’accès du compte de stockage. Pour découvrir comment obtenir votre clé d’accès au stockage, consultez [Gérer les clés d’accès au compte de stockage](../../storage/common/storage-account-keys-manage.md).

1. Sélectionnez **Déployer** dans la barre de commandes pour déployer le service lié.

   ![Déployer](./media/data-factory-data-processing-using-batch/image8.png)

#### <a name="create-an-azure-batch-linked-service"></a>Créer un service lié Azure Batch
À cette étape, vous créez un service lié pour votre compte Azure Batch, qui sert à exécuter l’activité personnalisée de la fabrique de données.

1. Sélectionnez **Nouveau calcul** dans la barre de commande et choisissez **Azure Batch.** Le script JSON que vous utilisez pour créer un service lié Azure Batch dans l’éditeur s’affiche.

1. Dans le script JSON :

   a. Remplacez **nom de compte** par le nom de votre compte Azure Batch.

   b. Remplacez **clé d’accès** par la clé d’accès du compte Azure Batch.

   c. Entrez l’ID du pool de la propriété **poolName**. Pour cette propriété, vous pouvez spécifier le nom ou l’ID du pool.

   d. Entrez l’URI du lot pour la propriété JSON **batchUri** .

      > [!IMPORTANT]
      > L’URL dans le panneau **Compte Batch** est au format suivant : \<accountname\>.\<region\>.batch.azure.com. Pour la propriété **batchUri** dans le fichier JSON, vous devez supprimer a88"nom_compte."** de l’URL. par exemple `"batchUri": "https://eastus.batch.azure.com"`.
      >
      >

      ![Panneau Compte Batch](./media/data-factory-data-processing-using-batch/image9.png)

      Pour la propriété **poolName**, vous pouvez également spécifier l’ID du pool au lieu du nom du pool.

      > [!NOTE]
      > Le service Data Factory ne prend pas en charge l’option à la demande pour Azure Batch, contrairement à HDInsight. Vous ne pouvez utiliser que votre propre pool Batch dans une fabrique de données.
      >
      >

   e. Spécifiez **StorageLinkedService** for the **StorageLinkedService** . Vous avez créé ce service lié à l’étape précédente. Ce stockage est utilisé en tant que zone de transit pour les fichiers et les journaux d’activité.

1. Sélectionnez **Déployer** dans la barre de commandes pour déployer le service lié.

#### <a name="step-3-create-datasets"></a>Étape 3 : Créez les jeux de données
Dans cette étape, vous allez créer des jeux de données pour représenter les données d’entrée et de sortie.

#### <a name="create-the-input-dataset"></a>Créer le jeu de données d’entrée
1. Dans Data Factory Editor, sélectionnez le bouton **Nouveau jeu de données** dans la barre d’outils. Sélectionnez **Stockage Blob Azure** dans la liste déroulante.

1. Remplacez le script JSON affiché dans le volet droit par le script JSON suivant :

    ```json
    {
       "name": "InputDataset",
       "properties": {
           "type": "AzureBlob",
           "linkedServiceName": "AzureStorageLinkedService",
           "typeProperties": {
               "folderPath": "mycontainer/inputfolder/{Year}-{Month}-{Day}-{Hour}",
               "format": {
                   "type": "TextFormat"
               },
               "partitionedBy": [
                   {
                       "name": "Year",
                       "value": {
                           "type": "DateTime",
                           "date": "SliceStart",
                           "format": "yyyy"
                       }
                   },
                   {
                       "name": "Month",
                       "value": {
                           "type": "DateTime",
                           "date": "SliceStart",
                           "format": "MM"
                       }
                   },
                   {
                       "name": "Day",
                       "value": {
                           "type": "DateTime",
                           "date": "SliceStart",
                           "format": "dd"
                       }
                   },
                   {
                       "name": "Hour",
                       "value": {
                           "type": "DateTime",
                           "date": "SliceStart",
                           "format": "HH"
                       }
                   }
               ]
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

    Plus loin dans cette procédure pas à pas, vous allez créer un pipeline avec l’heure de début 2015-11-16T00:00:00Z et l’heure de fin 2015-11-16T05:00:00Z. Il est planifié pour produire des données toutes les heures, si bien que cinq tranches d’entrée/sortie sont obtenues (entre **00**:00:00 -\> **05**:00:00).

    Les paramètres **frequency** et **interval** du jeu de données d’entrée sont respectivement définis sur **Hour** et sur **1**, ce qui signifie que la tranche d’entrée est disponible toutes les heures.

    L’heure de début de chaque tranche est représentée par la variable système **SliceStart** dans l’extrait de code JSON ci-dessus. Voici les heures de début de chaque tranche.

    | **Tranche** | **Heure de début**          |
    |-----------|-------------------------|
    | 1         | 2015-11-16T **00**:00:00 |
    | 2         | 2015-11-16T **01**:00:00 |
    | 3         | 2015-11-16T **02**:00:00 |
    | 4         | 2015-11-16T **03**:00:00 |
    | 5         | 2015-11-16T **04**:00:00 |

    La valeur **folderPath** est calculée à l’aide de la partie année, mois, jour et heure de l’heure de début de la tranche (**SliceStart**). Voici comment un dossier d’entrée est mappé à une tranche.

    | **Tranche** | **Heure de début**          | **Dossier d’entrée**  |
    |-----------|-------------------------|-------------------|
    | 1         | 2015-11-16T **00**:00:00 | 2015-11-16-**00** |
    | 2         | 2015-11-16T **01**:00:00 | 2015-11-16-**01** |
    | 3         | 2015-11-16T **02**:00:00 | 2015-11-16-**02** |
    | 4         | 2015-11-16T **03**:00:00 | 2015-11-16-**03** |
    | 5         | 2015-11-16T **04**:00:00 | 2015-11-16-**04** |

1. Sélectionnez **Déployer** dans la barre d’outils pour créer et déployer la table **InputDataset**.

#### <a name="create-the-output-dataset"></a>Créer le jeu de données de sortie
À cette étape, vous créez un autre jeu de données de type AzureBlob pour représenter les données de sortie.

1. Dans Data Factory Editor, sélectionnez le bouton **Nouveau jeu de données** dans la barre d’outils. Sélectionnez **Stockage Blob Azure** dans la liste déroulante.

1. Remplacez le script JSON affiché dans le volet droit par le script JSON suivant :

    ```json
    {
       "name": "OutputDataset",
       "properties": {
           "type": "AzureBlob",
           "linkedServiceName": "AzureStorageLinkedService",
           "typeProperties": {
               "fileName": "{slice}.txt",
               "folderPath": "mycontainer/outputfolder",
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

    Un objet blob/fichier de sortie est généré pour chaque tranche d’entrée. Voici la procédure de nommage des fichiers de sortie pour chaque tranche. Tous les fichiers de sortie sont générés dans un seul dossier de sortie : `mycontainer\\outputfolder`.

    | **Tranche** | **Heure de début**          | **Fichier de sortie**       |
    |-----------|-------------------------|-----------------------|
    | 1         | 2015-11-16T **00**:00:00 | 2015-11-16-**00.txt** |
    | 2         | 2015-11-16T **01**:00:00 | 2015-11-16-**01.txt** |
    | 3         | 2015-11-16T **02**:00:00 | 2015-11-16-**02.txt** |
    | 4         | 2015-11-16T **03**:00:00 | 2015-11-16-**03.txt** |
    | 5         | 2015-11-16T **04**:00:00 | 2015-11-16-**04.txt** |

    N’oubliez pas que tous les fichiers figurant dans un dossier d’entrée (par exemple, 2015-11-16-00) font partie d’une tranche associée à l’heure de début 2015-11-16-00. Lorsque cette tranche est traitée, l’activité personnalisée lit chaque fichier et génère une ligne dans le fichier de sortie avec le nombre d’occurrences du terme recherché (Microsoft). Si le dossier 2015-11-16-00 contient trois fichiers, le fichier de sortie 2015-11-16-00.txt contient trois lignes.

1. Sélectionnez **Déployer** dans la barre d’outils pour créer et déployer la table **OutputDataset**.

#### <a name="step-4-create-and-run-the-pipeline-with-a-custom-activity"></a>Étape 4 : Créer et exécuter le pipeline avec une activité personnalisée
À cette étape, vous créez un pipeline avec une seule activité, l’activité personnalisée que vous avez créée précédemment.

> [!IMPORTANT]
> Si vous n’avez pas chargé le fichier **file.txt** dans les dossiers d’entrée du conteneur d’objets blob, faites-le avant de créer le pipeline. La propriété **isPaused** étant réglée sur false dans le fichier JSON, le pipeline s’exécute immédiatement, car la date de **début** est passée.
>
>

1. Dans Data Factory Editor, sélectionnez **Nouveau pipeline** dans la barre de commandes. Si vous ne voyez pas la commande, sélectionnez le symbole de points de suspension pour l’afficher.

1. Remplacez le script JSON affiché dans le volet droit par le script JSON suivant :

    ```json
    {
       "name": "PipelineCustom",
       "properties": {
           "description": "Use custom activity",
           "activities": [
               {
                   "type": "DotNetActivity",
                   "typeProperties": {
                       "assemblyName": "MyDotNetActivity.dll",
                       "entryPoint": "MyDotNetActivityNS.MyDotNetActivity",
                       "packageLinkedService": "AzureStorageLinkedService",
                       "packageFile": "customactivitycontainer/MyDotNetActivity.zip"
                   },
                   "inputs": [
                       {
                           "name": "InputDataset"
                       }
                   ],
                   "outputs": [
                       {
                           "name": "OutputDataset"
                       }
                   ],
                   "policy": {
                       "timeout": "00:30:00",
                       "concurrency": 5,
                       "retry": 3
                   },
                   "scheduler": {
                       "frequency": "Hour",
                       "interval": 1
                   },
                   "name": "MyDotNetActivity",
                   "linkedServiceName": "AzureBatchLinkedService"
               }
           ],
           "start": "2015-11-16T00:00:00Z",
           "end": "2015-11-16T05:00:00Z",
           "isPaused": false
      }
    }
    ```
   Notez les points suivants :

   * Le pipeline ne contient qu’une seule activité de type **DotNetActivity**.
   * Le paramètre **AssemblyName** contient le nom de la DLL **MyDotnetActivity.dll**.
   * Le paramètre **EntryPoint** est défini sur **MyDotNetActivityNS.MyDotNetActivity**. Il s’agit essentiellement \<namespace\>.\<classname\> dans votre code.
   * **PackageLinkedService** est défini sur **StorageLinkedService**, qui pointe vers le stockage d’objets blob contenant le fichier zip de l’activité personnalisée. Si vous utilisez différents comptes de stockage pour les fichiers d’entrée/sortie et le fichier zip de l’activité personnalisée, vous devez créer un autre service lié Stockage Azure. Cet article suppose que vous utilisiez le même compte de stockage.
   * Le paramètre **PackageFile** est défini sur **customactivitycontainer/MyDotNetActivity.zip**. Il est au format suivant \<containerforthezip\>/\<nameofthezip.zip\>.
   * L’activité personnalisée utilise **InputDataset** comme entrée et **OutputDataset** comme sortie.
   * La propriété **linkedServiceName** de l’activité personnalisée pointe vers **AzureBatchLinkedService**, ce qui indique à Azure Data Factory que l’activité personnalisée doit s’exécuter sur Azure Batch.
   * Le paramètre **concurrency** est important. Si vous utilisez la valeur par défaut (1), même si vous avez plusieurs nœuds de calcul dans le pool Azure Batch, les tranches sont traitées successivement. Par conséquent, vous ne profitez pas de la capacité de traitement en parallèle d’Azure Batch. Si vous réglez **concurrency** sur une valeur plus élevée (par exemple, 2), deux tranches (correspondant à deux tâches dans Azure Batch) peuvent être traitées simultanément. Dans ce cas, les deux machines virtuelles dans le pool Azure Batch sont utilisées. Attribuez une valeur appropriée à la propriété concurrency.
   * Par défaut, une seule tâche (tranche) est exécutée sur une machine virtuelle à un moment donné. Par défaut, **Maximum tasks per VM (Nombre maximal de tâches par machine virtuelle)** est défini sur 1 pour un pool Azure Batch. Dans le cadre de la configuration requise, vous avez créé un pool avec cette propriété réglée sur 2. Par conséquent, deux tranches Data Factory peuvent s’exécuter sur une machine virtuelle en même temps.
     - La propriété **isPaused** est réglée sur false par défaut. Le pipeline s’exécute immédiatement dans cet exemple car les tranches débutent à une date antérieure. Vous pouvez régler cette propriété sur **true** pour suspendre le pipeline et lui réattribuer la valeur **false** pour reprendre l’exécution.
     -   Les heures de **début** et de **fin** sont séparées de cinq heures. Comme les tranches sont produites toutes les heures, le pipeline génère cinq tranches.

1. Sélectionnez **Déployer** dans la barre de commandes pour déployer le pipeline.

#### <a name="step-5-test-the-pipeline"></a>Étape 5 : Tester le pipeline
Au cours de cette étape, vous testez le pipeline en déposant des fichiers dans les dossiers d’entrée. Commençons par tester le pipeline avec un fichier pour chaque dossier d’entrée.

1. Dans le panneau **Fabrique de données** du portail Azure, sélectionnez **Diagramme**.

   ![Diagramme](./media/data-factory-data-processing-using-batch/image10.png)

1. Dans la vue **Diagramme**, double-cliquez sur le jeu de données d’entrée **InputDataset**.

   ![InputDataset](./media/data-factory-data-processing-using-batch/image11.png)

1. Le panneau **InputDataset** apparaît avec les cinq tranches prêtes. Notez **l’HEURE DE DÉBUT DE LA TRANCHE** et **l’HEURE DE FIN DE LA TRANCHE** pour chaque tranche.

   ![Heures de début et de fin de la tranche d’entrée](./media/data-factory-data-processing-using-batch/image12.png)

1. Dans la vue **Diagramme**, sélectionnez **OutputDataset**.

1. Les cinq tranches de sortie s’affichent dans l’état **Prêt** si elles ont été générées.

   ![Heures de début et de fin de la tranche de sortie](./media/data-factory-data-processing-using-batch/image13.png)

1. Utilisez le portail pour afficher les tâches associées aux tranches et voir la machine virtuelle sur laquelle chaque tranche a été exécutée. Pour plus d’informations, consultez la section [Data Factory and Batch integration (Intégration de Data Factory et de Batch)](#data-factory-and-batch-integration).

1. Les fichiers de sortie apparaissent sous `mycontainer` dans `outputfolder` dans votre stockage d’objets blob.

   ![Fichiers de sortie dans le stockage](./media/data-factory-data-processing-using-batch/image15.png)

   Cinq tranches de sortie sont indiquées, une pour chaque tranche d’entrée. Chaque fichier de sortie a un contenu similaire à ce qui suit :

    ```
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file.txt.
    ```
   Le diagramme suivant illustre comment les tranches Data Factory sont mappées à des tâches dans Azure Batch. Dans cet exemple, une tranche n’est exécutée qu’une seule fois.

   ![Schéma de mappage de tranche](./media/data-factory-data-processing-using-batch/image16.png)

1. À présent, essayons avec plusieurs fichiers dans un dossier. Créez les fichiers **file2.txt**, **file3.txt**, **file4.txt** et **file5.txt** avec le même contenu que le fichier file.txt dans le dossier **2015-11-06-01**.

1. Dans le dossier de sortie, supprimez le fichier de sortie **2015-11-16-01.txt**.

1. Dans le panneau **OutputDataset**, cliquez avec le bouton droit sur la tranche dont l’**HEURE DE DÉBUT DE LA TRANCHE** est **16/11/2015 01:00:00 AM**. Sélectionnez **Exécuter** pour réexécuter/retraiter la tranche. La tranche contient maintenant cinq fichiers au lieu d’un seul.

    ![Exécuter](./media/data-factory-data-processing-using-batch/image17.png)

1. Une fois la tranche exécutée, quand elle est dans l’état **Prêt**, vérifiez le contenu de son fichier de sortie (**2015-11-16-01.txt**). Le fichier de sortie apparaît sous `mycontainer` dans `outputfolder` dans votre stockage d’objets blob. Il doit y avoir une ligne pour chaque fichier de la tranche.

    ```
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file.txt.
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file2.txt.
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file3.txt.
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file4.txt.
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file5.txt.
    ```

> [!NOTE]
> Si vous n’avez pas supprimé le fichier de sortie 2015-11-16-01.txt avant d’essayer avec cinq fichiers d’entrée, vous voyez une ligne de l’exécution de la tranche précédente et cinq lignes de l’exécution de la tranche actuelle. Par défaut, le contenu est ajouté à la fin du fichier de sortie s’il existe.
>
>

#### <a name="data-factory-and-batch-integration"></a>Intégration de Data Factory et Batch
Le service Data Factory crée un travail dans Azure Batch avec le nom `adf-poolname:job-xxx`.

![Travaux Azure Batch](media/data-factory-data-processing-using-batch/data-factory-batch-jobs.png)

Une tâche dans le travail est créée pour chaque exécution d’activité d’une tranche. Si 10 tranches sont prêtes à être traitées, 10 tâches sont créées dans le travail. Plusieurs tranches peuvent être exécutées en parallèle si vous disposez de plusieurs nœuds de calcul dans le pool. Si le nombre maximum de tâches par nœud de calcul est défini sur une valeur supérieure à 1, plusieurs tranches peuvent s’exécuter sur le même nœud de calcul.

Dans cet exemple, il y a cinq tranches, donc cinq tâches dans Azure Batch. Avec la propriété **concurrency** définie sur **5** dans le fichier JSON du pipeline dans Azure Data Factory, et le paramètre **Maximum tasks per VM (Nombre maximal de tâches par machine virtuelle)** défini sur **2** dans le pool Azure Batch avec **2** machines virtuelles, les tâches s’exécutent rapidement. (Vérifiez les heures de début et de fin des tâches.)

Utilisez le portail pour afficher le travail Azure Batch et ses tâches associées aux tranches et voir sur quelle machine virtuelle chaque tranche s’est exécutée.

![Tâches de travail Azure Batch](media/data-factory-data-processing-using-batch/data-factory-batch-job-tasks.png)

### <a name="debug-the-pipeline"></a>Déboguer le pipeline
Le débogage consiste à utiliser quelques techniques de base.

1. Si la tranche d’entrée n’est pas définie sur **Prêt**, vérifiez que la structure du dossier d’entrée est correcte et que le fichier file.txt existe dans les dossiers d’entrée.

   ![Structure du dossier d’entrée](./media/data-factory-data-processing-using-batch/image3.png)

1. Dans la méthode **Execute** de votre activité personnalisée, utilisez l’objet **IActivityLogger** pour journaliser les informations qui vous aident à résoudre d’éventuels problèmes. Les messages consignés s’affichent dans le fichier user\_0.log.

   Dans le panneau **OutputDataset**, sélectionnez la tranche pour afficher le panneau **Tranche de données** correspondant à cette tranche. Une exécution d’activité correspondant à la tranche apparaît sous **Exécutions de l’activité**. Si vous cliquez sur **Exécuter** dans la barre de commandes, vous pouvez lancer une autre exécution d’activité pour la même tranche.

   Lorsque vous sélectionnez l’exécution de l’activité, le panneau **Détails de l’exécution de l’activité** s’affiche avec une liste de fichiers journaux. Les messages consignés s’affichent dans le fichier user\_0.log. Lorsqu’une erreur se produit, vous verrez trois exécutions d’activité car le nombre de tentatives est défini sur 3 dans le script JSON du pipeline et de l’activité. Lorsque vous sélectionnez l’exécution de l’activité, vous voyez les fichiers journaux qui vous permettront de résoudre l’erreur.

   ![Panneaux OutputDataset et de tranche de données](./media/data-factory-data-processing-using-batch/image18.png)

   Dans la liste des fichiers journaux, cliquez sur le fichier **user-0.log**. Le volet droit affiche les résultats de l’utilisation de la méthode **IActivityLogger.Write**.

   ![Panneau Détails de l’exécution de l’activité](./media/data-factory-data-processing-using-batch/image19.png)

   Consultez le fichier system-0.log pour vérifier les exceptions et messages d’erreur système.

    ```
    Trace\_T\_D\_12/6/2015 1:43:35 AM\_T\_D\_\_T\_D\_Verbose\_T\_D\_0\_T\_D\_Loading assembly file MyDotNetActivity...

    Trace\_T\_D\_12/6/2015 1:43:35 AM\_T\_D\_\_T\_D\_Verbose\_T\_D\_0\_T\_D\_Creating an instance of MyDotNetActivityNS.MyDotNetActivity from assembly file MyDotNetActivity...

    Trace\_T\_D\_12/6/2015 1:43:35 AM\_T\_D\_\_T\_D\_Verbose\_T\_D\_0\_T\_D\_Executing Module

    Trace\_T\_D\_12/6/2015 1:43:38 AM\_T\_D\_\_T\_D\_Information\_T\_D\_0\_T\_D\_Activity e3817da0-d843-4c5c-85c6-40ba7424dce2 finished successfully
    ```
1. Incluez le fichier **PDB** dans le fichier zip, afin que les détails de l’erreur fournissent des informations comme la pile des appels quand une erreur se produit.

1. Tous les fichiers contenus dans le fichier zip de l’activité personnalisée doivent se trouver au niveau supérieur et ne contenir aucun sous-dossier.

   ![Liste des fichiers zip de l’activité personnalisée](./media/data-factory-data-processing-using-batch/image20.png)

1. Assurez-vous que les paramètres **assemblyName** (MyDotNetActivity.dll), **entryPoint** (MyDotNetActivityNS.MyDotNetActivity), **packageFile** (customactivitycontainer/Mydotnetactivity.zip) et **packageLinkedService** (qui doit pointer vers le Stockage Blob Azure contenant le fichier zip) ont des valeurs correctes.

1. Si vous avez corrigé une erreur et que vous souhaitez relancer le traitement de la tranche, cliquez avec le bouton droit sur la tranche dans le panneau **OutputDataset** puis sélectionnez **Exécuter**.

   ![Option Exécuter du panneau de OutputDataset](./media/data-factory-data-processing-using-batch/image21.png)

   > [!NOTE]
   > Un conteneur se trouve dans votre stockage d’objets blob nommé `adfjobs`. Ce conteneur n’est pas automatiquement supprimé, mais vous pouvez le supprimer en toute sécurité après avoir testé la solution. De même, la solution Data Factory crée un travail Azure Batch nommé `adf-\<pool ID/name\>:job-0000000001`. Si vous le souhaitez, vous pouvez supprimer ce travail après avoir testé la solution.
   >
   >
1. L’activité personnalisée n’utilise pas le ficher **app.config** de votre package. Par conséquent, si votre code lit des chaînes de connexion dans le fichier de configuration, il ne fonctionne pas lors de l’exécution. La meilleure pratique lorsque vous utilisez Azure Batch consiste à stocker les secrets dans Azure Key Vault. Ensuite, utilisez un principal de service basé sur le certificat pour protéger le coffre de clés et distribuez le certificat au pool Azure Batch. L’activité personnalisée .NET peut alors accéder aux secrets du coffre de clés lors de l’exécution. Cette solution générique peut s’adapter à n’importe quel type de clé secrète, et pas seulement à une chaîne de connexion.

    Il existe une solution plus facile, mais elle n’est pas recommandée. Vous pouvez créer un service lié à SQL Database avec des paramètres de chaîne de connexion. Ensuite, vous pouvez créer un jeu de données qui utilise le service lié et chaîner le jeu de données comme un jeu de données d’entrée factice à l’activité .NET personnalisée. Vous pouvez ensuite accéder à la chaîne de connexion du service lié dans le code de l’activité personnalisée. Cela devrait fonctionner correctement lors de l’exécution.  

#### <a name="extend-the-sample"></a>Étendre l’exemple
Vous pouvez étendre cet exemple pour en savoir plus sur les fonctionnalités Azure Data Factory et Azure Batch. Par exemple, pour traiter des tranches dans une autre plage de temps, procédez comme suit :

1. Ajoutez les sous-dossiers suivants dans `inputfolder` : 2015-11-16-05, 2015-11-16-06, 201-11-16-07, 2011-11-16-08, and 2015-11-16-09. Placez les fichiers d’entrée dans ces dossiers. Modifiez l’heure de fin pour le pipeline de `2015-11-16T05:00:00Z` à `2015-11-16T10:00:00Z`. Dans la vue **Diagramme**, double-cliquez sur **InputDataset** et vérifiez que les tranches d’entrée sont prêtes. Double-cliquez sur **OuptutDataset** pour voir l’état des tranches de sortie. Si leur état est **Prêt**, vérifiez les fichiers de sortie dans le dossier de sortie.

1. Augmentez ou diminuez la valeur du paramètre **concurrency** pour comprendre comment il affecte les performances de votre solution, en particulier le traitement effectué sur Azure Batch. Pour plus d’informations sur le paramètre **concurrency**, consultez « Étape 4 : Créer et exécuter le pipeline avec une activité personnalisée ».

1. Créez un pool avec une valeur **Maximum tasks per VM**(Nombre maximal de tâches par machine virtuelle) supérieure/inférieure. Pour utiliser le pool créé, mettez à jour le service lié Azure Batch dans la solution Data Factory. Pour plus d’informations sur le paramètre **Maximum tasks per VM** (Nombre maximal de tâches par machine virtuelle), consultez « Étape 4 : Créer et exécuter le pipeline avec une activité personnalisée ».

1. Créez un pool Azure Batch avec la fonctionnalité **autoscale**. La mise à l’échelle automatique des nœuds de calcul dans un pool Azure Batch est en fait un ajustement dynamique de la puissance de traitement utilisée par votre application.

    L’exemple de formule fourni ici produit le comportement suivant. Lors de sa création, le pool commence par une machine virtuelle. La métrique $PendingTasks définit le nombre de tâches dans l’état En cours d’exécution et Actif (en file d’attente). Cette formule recherche le nombre moyen de tâches en attente au cours des 180 dernières secondes et définit TargetDedicated en conséquence. Elle garantit que TargetDedicated ne va jamais au-delà de 25 machines virtuelles. À mesure que de nouvelles tâches sont envoyées, le pool s’accroît automatiquement. Lorsque les tâches se terminent, les machines virtuelles se libèrent une par une et la mise à l’échelle automatique les réduit. Vous pouvez ajuster startingNumberOfVMs et maxNumberofVMs en fonction de vos besoins.

    Formule de mise à l’échelle automatique :

    ```
    startingNumberOfVMs = 1;
    maxNumberofVMs = 25;
    pendingTaskSamplePercent = $PendingTasks.GetSamplePercent(180 * TimeInterval_Second);
    pendingTaskSamples = pendingTaskSamplePercent < 70 ? startingNumberOfVMs : avg($PendingTasks.GetSample(180 * TimeInterval_Second));
    $TargetDedicated=min(maxNumberofVMs,pendingTaskSamples);
    ```

   Pour plus d’informations, consultez [Mettre automatiquement à l’échelle les nœuds de calcul dans un pool Azure Batch](../../batch/batch-automatic-scaling.md).

   Si le pool utilise la valeur par défaut du paramètre [autoScaleEvaluationInterval](/rest/api/batchservice/pool/enableautoscale), le service Batch peut mettre 15 à 30 minutes pour préparer la machine virtuelle avant d’exécuter l’activité personnalisée. Si le pool utilise une autre valeur pour autoScaleEvaluationInterval, le service Batch peut prendre la durée d’autoScaleEvaluationInterval + 10 minutes.

1. Dans l’exemple de solution, la méthode **Execute** appelle la méthode **Calculate** qui traite une tranche de données d’entrée pour produire une tranche de données de sortie. Vous pouvez écrire votre propre méthode pour traiter les données d’entrée et remplacer l’appel de la méthode **Calculate** dans la méthode **Execute** par un appel à votre méthode.

### <a name="next-steps-consume-the-data"></a>Étapes suivantes : Consommer les données
Après avoir traité des données, vous pouvez les consommer avec des outils en ligne tels que Power BI. Voici des liens pour vous aider à comprendre Power BI et comment l’utiliser dans Azure :

* [Explorer un jeu de données dans Power BI](https://powerbi.microsoft.com/documentation/powerbi-service-get-data/)
* [Prise en main de Power BI Desktop](/power-bi/fundamentals/desktop-getting-started)
* [Actualisation des données dans Power BI](https://powerbi.microsoft.com/documentation/powerbi-refresh-data/)
* [Azure et Power BI : vue d’ensemble](https://powerbi.microsoft.com/documentation/powerbi-azure-and-power-bi/)

## <a name="references"></a>References
* [Azure Data Factory](https://azure.microsoft.com/documentation/services/data-factory/).

  * [Introduction to the Data Factory service (Présentation du service Azure Data Factory)](data-factory-introduction.md)
  * [Get started with Data Factory (Prise en main de Data Factory)](data-factory-build-your-first-pipeline.md)
  * [Use custom activities in a Data Factory pipeline (Utilisation des activités personnalisées dans un pipeline Data Factory)](data-factory-use-custom-activities.md)
* [Azure Batch](https://azure.microsoft.com/documentation/services/batch/)

  * [Notions de base d’Azure Batch](../../azure-sql/database/sql-database-paas-overview.md)
  * [Vue d’ensemble des fonctionnalités Batch](../../batch/batch-service-workflow-features.md))
  * [Create and manage a Batch account in the Azure portal (Création et gestion d’un compte Azure Batch dans le portail Azure)](../../batch/batch-account-create-portal.md)
  * [Get started with the Batch client library for .NET (Prise en main de la bibliothèque Batch pour .NET)](../../batch/quick-run-dotnet.md)

[batch-explorer]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[batch-explorer-walkthrough]: /archive/blogs/windowshpc/azure-batch-explorer-sample-walkthrough
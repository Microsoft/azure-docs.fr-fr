---
title: Tutoriel C# sur l’utilisation de l’IA sur des objets blob Azure
titleSuffix: Azure Cognitive Search
description: Parcourez un exemple d’extraction de texte et de traitement en langage naturel sur du contenu dans Stockage Blob en utilisant C# et le SDK .NET de Recherche cognitive Azure.
manager: nitinme
author: MarkHeff
ms.author: maheff
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 10/05/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: da7a80842bec68fde8cc44401bb04c2dd061741f
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92787956"
---
# <a name="tutorial-ai-generated-searchable-content-from-azure-blobs-using-the-net-sdk"></a>Tutoriel : Contenu recherchable généré par l’IA issu d’objets blob Azure avec le SDK .NET

Si vous avez du texte non structuré ou des images dans le stockage Blob Azure, un [pipeline d’enrichissement par IA](cognitive-search-concept-intro.md) peut extraire des informations et créer du contenu pour les scénarios de recherche en texte intégral ou d’exploration de connaissances. 

Dans ce didacticiel, vous apprendrez à :

> [!div class="checklist"]
> * Configurer un environnement de développement
> * Définir un pipeline sur des objets blob qui utilise la reconnaissance optique de caractères, la détection de la langue et la reconnaissance d’entités et d’expressions clés
> * Exécuter le pipeline pour appeler des transformations et pour créer et charger un index de recherche
> * Explorez les résultats à l’aide de la recherche en texte intégral et d’une syntaxe de requête enrichie.

Si vous n’avez pas d’abonnement Azure, ouvrez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="overview"></a>Vue d’ensemble

Ce tutoriel utilise C# et la bibliothèque de client **Azure.Search.Documents** pour créer une source de données, un index, un indexeur et des compétences.

L’ensemble de compétences utilise des compétences intégrées basées sur les API Cognitive Services. Les étapes du pipeline incluent la reconnaissance optique de caractères (OCR) sur des images, la détection de la langue d’un texte, l’extraction des expressions clés et la reconnaissance d’entités (organisations). Les nouvelles informations sont stockées dans de nouveaux champs que vous pouvez exploiter dans les requêtes, les facettes et les filtres.

## <a name="prerequisites"></a>Prérequis

* [Visual Studio](https://visualstudio.microsoft.com/downloads/)
* [Package NuGet Azure.Search.Documents 11.x](https://www.nuget.org/packages/Azure.Search.Documents) 
* [Stockage Azure](https://azure.microsoft.com/services/storage/)
* [Recherche cognitive Azure](https://azure.microsoft.com/services/search/)

> [!Note]
> Vous pouvez utiliser le service de recherche gratuit pour ce tutoriel. Avec un service de recherche gratuit, vous êtes limité à trois index, trois indexeurs et trois sources de données. Ce didacticiel crée une occurrence de chaque élément. Avant de commencer, veillez à disposer de l’espace suffisant sur votre service pour accepter les nouvelles ressources.

## <a name="download-sample-data"></a>Télécharger l'exemple de données

Les exemples de données sont constitués de 14 fichiers de type de contenu mixte que vous chargerez sur Stockage Blob Azure à une étape ultérieure.

1. Ouvrez ce [dossier OneDrive](https://1drv.ms/f/s!As7Oy81M_gVPa-LCb5lC_3hbS-4) et en haut à gauche, cliquez sur **Télécharger** pour copier les fichiers sur votre ordinateur. 

1. Cliquez avec le bouton droit sur le fichier zip et sélectionnez **Tout extraire** . Il y a 14 fichiers de différents types. Vous en utiliserez 7 dans le cadre de cet exercice.

Vous pouvez également télécharger le code source de ce tutoriel. Le code source se trouve dans le dossier **tutorial-ai-enrichment/v11** du dépôt [azure-search-dotnet-samples](https://github.com/Azure-Samples/azure-search-dotnet-samples).

## <a name="1---create-services"></a>1 - Créer les services

Ce tutoriel utilise Recherche cognitive Azure pour l’indexation et les requêtes, Cognitive Services pour l’enrichissement par IA sur le back-end, et Stockage Blob Azure pour fournir les données. Ce tutoriel reste sous l’allocation gratuite de 20 transactions par indexeur par jour sur Cognitive Services : les seuls services que vous devez créer sont donc la recherche et le stockage.

Si possible, créez les deux services dans la même région et le même groupe de ressources pour des raisons de proximité et de facilité de gestion. Dans la pratique, votre compte de stockage Azure peut être dans une région quelconque.

### <a name="start-with-azure-storage"></a>Démarrer avec le stockage Azure

1. [Connectez-vous au portail Azure](https://portal.azure.com/) et cliquez sur **+ Créer une ressource** .

1. Recherchez un *compte de stockage* et sélectionnez l’offre de compte de stockage Microsoft.

   ![Créer un compte de stockage](media/cognitive-search-tutorial-blob/storage-account.png "Créer un compte de stockage")

1. Sous l’onglet Bases, les éléments suivants sont obligatoires. Acceptez les valeurs par défaut pour tout le reste.

   * **Groupe de ressources** . Sélectionnez un groupe existant ou créez-en un, mais utilisez le même groupe pour tous les services afin de pouvoir les gérer collectivement.

   * **Nom du compte de stockage** . Si vous pensez que vous pouvez avoir plusieurs ressources du même type, utilisez le nom pour lever l’ambiguïté par type et par région, par exemple *blobstoragewestus* . 

   * **Emplacement** . Si possible, choisissez le même emplacement que celui utilisé pour Recherche cognitive Azure et Cognitive Services. Un emplacement unique annule les frais liés à la bande passante.

   * **Type de compte** . Choisissez la valeur par défaut, *StorageV2 (v2 universel)* .

1. Cliquez sur **Vérifier + créer** pour créer le service.

1. Une fois qu’il est créé, cliquez sur **Accéder à la ressource** pour ouvrir la page Vue d’ensemble.

1. Cliquez sur le service **Objets blob** .

1. Cliquez sur **+ Conteneur** pour créer un conteneur et nommez-le *cog-search-demo* .

1. Sélectionnez *cog-search-demo* , puis cliquez sur **Charger** pour ouvrir le dossier dans lequel vous avez enregistré les fichiers téléchargés. Sélectionnez les quatorze fichiers, puis cliquez sur **OK** pour charger.

   ![Charger les exemples de fichiers](media/cognitive-search-quickstart-blob/sample-data.png "Charger les exemples de fichiers")

1. Avant de quitter Stockage Azure, obtenez une chaîne de connexion afin de pouvoir formuler une connexion dans Recherche cognitive Azure. 

   1. Revenez à la page Vue d’ensemble de votre compte de stockage (nous avons utilisé *blobstoragewestus* comme exemple). 

   1. Dans le volet de navigation gauche, sélectionnez **Clés d’accès** et copiez l’une des chaînes de connexion. 

   La chaîne de connexion est une URL similaire à l’exemple suivant :

      ```http
      DefaultEndpointsProtocol=https;AccountName=blobstoragewestus;AccountKey=<your account key>;EndpointSuffix=core.windows.net
      ```

1. Enregistrez la chaîne de connexion dans le Bloc-Notes. Vous en aurez besoin plus tard lors de la configuration de la connexion à la source de données.

### <a name="cognitive-services"></a>Cognitive Services

L’enrichissement par IA s’appuie sur Cognitive Services, notamment Analyse de texte et Vision par ordinateur pour le traitement des images et du langage naturel. Si votre objectif était de réaliser un prototype ou un projet réel, vous devriez à ce stade provisionner Cognitive Services (dans la même région que Recherche cognitive Azure) afin de pouvoir l’attacher aux opérations d’indexation.

Cependant, dans le cadre de cet exercice, vous pouvez ignorer le provisionnement des ressources, car Recherche cognitive Azure peut se connecter à Cognitive Services en arrière-plan et vous fournir 20 transactions gratuites par exécution de l’indexeur. Comme ce tutoriel utilise 14 transactions, l’allocation gratuite est suffisante. Pour les projets de plus grande envergure, prévoyez de provisionner Cognitive Services au niveau S0 du paiement à l’utilisation. Pour plus d’informations, consultez [Attacher Cognitive Services](cognitive-search-attach-cognitive-services.md).

### <a name="azure-cognitive-search"></a>Recherche cognitive Azure

Le troisième composant est Recherche cognitive Azure, que vous pouvez [créer dans le portail](search-create-service-portal.md), ou bien [recherchez un service de recherche existant](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) dans votre abonnement.

Vous pouvez utiliser le niveau gratuit pour effectuer cette procédure pas à pas. 

### <a name="copy-an-admin-api-key-and-url-for-azure-cognitive-search"></a>Copier une clé API d’administration et une URL pour Recherche cognitive Azure

Pour interagir avec votre service Recherche cognitive Azure, vous devrez disposer de l’URL du service et d’une clé d’accès. Un service de recherche est créé avec les deux. Ainsi, si vous avez ajouté la Recherche cognitive Azure à votre abonnement, effectuez ce qui suit pour obtenir les informations nécessaires :

1. [Connectez-vous au portail Azure](https://portal.azure.com/), puis dans la page **Vue d’ensemble** du service de recherche, récupérez l’URL. Voici un exemple de point de terminaison : `https://mydemo.search.windows.net`.

1. Dans **Paramètres** > **Clés** , copiez une clé d’administration pour avoir des droits d’accès complets sur le service. Il existe deux clés d’administration interchangeables, fournies pour assurer la continuité de l’activité au cas où vous deviez en remplacer une. Vous pouvez utiliser la clé primaire ou secondaire sur les demandes d’ajout, de modification et de suppression d’objets.

   Obtenez aussi la clé de requête. Il est recommandé d’émettre des demandes de requête avec un accès en lecture seule.

   ![Obtenir le nom du service, les clés d’administration et les clés de requête](media/search-get-started-javascript/service-name-and-keys.png)

L’utilisation d’une clé valide permet d’établir, en fonction de chaque demande, une relation de confiance entre l’application qui envoie la demande et le service qui en assure le traitement.

## <a name="2---set-up-your-environment"></a>2 - Configurer votre environnement

Commencez par ouvrir Visual Studio et créer un nouveau projet d’application console pouvant s’exécuter sur .NET Core.

### <a name="install-azuresearchdocuments"></a>Installer Azure.Search.Documents

Le [SDK .NET de la Recherche cognitive Azure](/dotnet/api/overview/azure/search) comprend une bibliothèque de client qui vous permet de gérer vos index, sources de données, indexeurs et compétences ainsi que de charger et gérer des documents et d’exécuter des requêtes, sans avoir à gérer les détails de HTTP et de JSON. Cette bibliothèque de client est distribuée en tant que package NuGet.

Pour ce projet, installez la version 11 ou ultérieure de la bibliothèque `Azure.Search.Documents` et la dernière version de `Microsoft.Extensions.Configuration`.

1. Dans Visual Studio, sélectionnez **Outils** > **Gestionnaire de package NuGet** > **Gérer les packages NuGet pour la solution...** .

1. Recherchez [Azure.Search.Document](https://www.nuget.org/packages/Azure.Search.Documents).

1. Sélectionnez la version la plus récente, puis cliquez sur **Installer** .

1. Répétez les étapes précédentes pour installer [Microsoft.Extensions.Configuration](https://www.nuget.org/packages/Microsoft.Extensions.Configuration) et [Microsoft.Extensions.Configuration.Json](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Json).

### <a name="add-service-connection-information"></a>Ajouter les informations de connexion au service

1. Cliquez avec le bouton droit sur votre projet dans l’Explorateur de solutions et sélectionnez **Ajouter** > **Nouvel élément...** . 

1. Nommez le fichier `appsettings.json` et sélectionnez **Ajouter** . 

1. Incluez ce fichier dans votre répertoire de sortie.
    1. Cliquez avec le bouton droit sur `appsettings.json`, puis sélectionnez **Propriétés** . 
    1. Remplacez la valeur **Copier dans le répertoire de sortie** par **Copie du plus récent** .

1. Copiez le code JSON ci-dessous dans votre nouveau fichier JSON.

    ```json
    {
      "SearchServiceUri": "Put your search service URI here",
      "SearchServiceAdminApiKey": "Put your primary or secondary API key here",
      "SearchServiceQueryApiKey": "Put your query API key here",
      "AzureBlobConnectionString": "Put your Azure Blob connection string here",
    }
    ```

Ajoutez les informations relatives à votre service de recherche, ainsi qu'à votre compte de stockage d'objets blob. Rappelez-vous que vous pouvez récupérer ces informations à partir des étapes de configuration du service indiquées dans la section précédente.

Pour **SearchServiceUri** , entrez l’URL complète.

### <a name="add-namespaces"></a>Ajouter des espaces de noms

Dans `Program.cs`, ajoutez les espaces de noms suivants.

```csharp
using Azure;
using Azure.Search.Documents.Indexes;
using Azure.Search.Documents.Indexes.Models;
using Microsoft.Extensions.Configuration;
using System;
using System.Collections.Generic;
using System.Linq;

namespace EnrichwithAI
```

### <a name="create-a-client"></a>Créer un client

Créez une instance d’un `SearchIndexClient` et d’un `SearchIndexerClient` sous `Main`.

```csharp
public static void Main(string[] args)
{
    // Create service client
    IConfigurationBuilder builder = new ConfigurationBuilder().AddJsonFile("appsettings.json");
    IConfigurationRoot configuration = builder.Build();

    string searchServiceUri = configuration["SearchServiceUri"];
    string adminApiKey = configuration["SearchServiceAdminApiKey"];
    string cognitiveServicesKey = configuration["CognitiveServicesKey"];

    SearchIndexClient indexClient = new SearchIndexClient(new Uri(searchServiceUri), new AzureKeyCredential(adminApiKey));
    SearchIndexerClient indexerClient = new SearchIndexerClient(new Uri(searchServiceUri), new AzureKeyCredential(adminApiKey));
}
```

> [!NOTE]
> Les clients se connectent à votre service de recherche. Pour éviter l’ouverture d’un trop grand nombre de connexions, essayez de partager une seule instance dans votre application, si possible. Les méthodes sont thread-safe pour activer le partage.
> 

### <a name="add-function-to-exit-the-program-during-failure"></a>Ajouter une fonction pour quitter le programme pendant des événements d’échec

Ce tutoriel est destiné à vous aider à comprendre chaque étape du pipeline d’indexation. En cas de problème critique empêchant le programme de créer la source de données, l’ensemble de compétences, l’index ou l’indexeur, le programme génère le message d’erreur et se ferme afin que le problème puisse être compris et résolu.

Ajoutez `ExitProgram` à `Main` pour gérer les scénarios qui nécessitent la fermeture du programme.

```csharp
private static void ExitProgram(string message)
{
    Console.WriteLine("{0}", message);
    Console.WriteLine("Press any key to exit the program...");
    Console.ReadKey();
    Environment.Exit(0);
}
```

## <a name="3---create-the-pipeline"></a>3 - Créer le pipeline

Dans Recherche cognitive Azure, le traitement de l’IA se produit pendant l’indexation (ou l’ingestion de données). Cette partie de la procédure pas à pas crée quatre objets : source de données, définition d’index, ensemble de compétences, indexeur. 

### <a name="step-1-create-a-data-source"></a>Étape 1 : Création d'une source de données

`SearchIndexerClient` a une propriété [`DataSourceName`](/dotnet/api/azure.search.documents.indexes.models.searchindexer.datasourcename) que vous pouvez définir sur un objet `SearchIndexerDataSourceConnection`. Cet objet fournit toutes les méthodes dont vous avez besoin pour créer, afficher, mettre à jour ou supprimer des sources de données de la Recherche cognitive Azure.

Créer une nouvelle instance `SearchIndexerDataSourceConnection` en appelant `indexerClient.CreateOrUpdateDataSourceConnection(dataSource)`. Le code suivant crée une source de données de type `AzureBlob`.

```csharp
private static SearchIndexerDataSourceConnection CreateOrUpdateDataSource(SearchIndexerClient indexerClient, IConfigurationRoot configuration)
{
    SearchIndexerDataSourceConnection dataSource = new SearchIndexerDataSourceConnection(
        name: "demodata",
        type: SearchIndexerDataSourceType.AzureBlob,
        connectionString: configuration["AzureBlobConnectionString"],
        container: new SearchIndexerDataContainer("cog-search-demo"))
    {
        Description = "Demo files to demonstrate cognitive search capabilities."
    };

    // The data source does not need to be deleted if it was already created
    // since we are using the CreateOrUpdate method
    try
    {
        indexerClient.CreateOrUpdateDataSourceConnection(dataSource);
    }
    catch (Exception ex)
    {
        Console.WriteLine("Failed to create or update the data source\n Exception message: {0}\n", ex.Message);
        ExitProgram("Cannot continue without a data source");
    }

    return dataSource;
}
```

Pour une requête réussie, la méthode renvoie la source de données créée. En cas de problème lié à la requête, comme un paramètre non valide, la méthode lève une exception.

Ajoutez maintenant une ligne dans `Main` pour appeler la fonction `CreateOrUpdateDataSource` que vous venez d’ajouter.

```csharp
// Create or Update the data source
Console.WriteLine("Creating or updating the data source...");
SearchIndexerDataSourceConnection dataSource = CreateOrUpdateDataSource(indexerClient, configuration);
```

Créez et exécutez la solution. Dans la mesure où il s’agit de votre première demande, vérifiez dans le portail Azure que la source de données a bien été créée dans la Recherche cognitive Azure. Dans la page de présentation du service de recherche, vérifiez que la liste Sources de données comporte un nouvel élément. Vous devrez peut-être attendre quelques minutes que la page du portail soit actualisée.

  ![Vignette Sources de données dans le portail](./media/cognitive-search-tutorial-blob/data-source-tile.png "Vignette Sources de données dans le portail")

### <a name="step-2-create-a-skillset"></a>Étape 2 : Créer un ensemble de compétences

Dans cette section, vous définissez un ensemble d’étapes d’enrichissement que vous souhaitez appliquer à vos données. Chaque étape d’enrichissement est appelée *compétence* et l’ensemble des étapes d’enrichissement, *ensemble de compétences* . Ce tutoriel utilise des [compétences cognitives prédéfinies](cognitive-search-predefined-skills.md) pour l’ensemble de compétences :

* [Reconnaissance optique de caractères](cognitive-search-skill-ocr.md) pour reconnaître le texte imprimé et manuscrit dans des fichiers image.

* [Fusion de texte](cognitive-search-skill-textmerger.md) pour consolider en un champ unique du texte issu d’une collection de champs.

* [Détection de la langue](cognitive-search-skill-language-detection.md) pour identifier la langue du contenu.

* [Fractionnement de texte](cognitive-search-skill-textsplit.md) pour découper un texte long en plus petits morceaux avant d’appeler les compétences d’extraction de phrases clés et de reconnaissance d’entités. Les compétences d’extraction de phrases clés et de reconnaissance d’entités acceptent les entrées de 50 000 caractères maximum. Certains fichiers d’exemple doivent être fractionnés pour satisfaire cette limite.

* [Reconnaissance d’entité](cognitive-search-skill-entity-recognition.md) pour extraire les noms d’organisations du contenu dans le conteneur d’objets blob.

* [Extraction de phrases clés](cognitive-search-skill-keyphrases.md) pour extraire les principales expressions clés.

Lors du traitement initial, la Recherche cognitive Azure interprète chaque document pour extraire le contenu de fichiers de différents formats. Le texte originaire du fichier source est placé dans un champ `content` généré, un pour chaque document. Par conséquent, définissez l’entrée en tant que `"/document/content"` de manière à utiliser ce texte. Le contenu de l’image est placé dans un champ `normalized_images` généré, spécifié dans un ensemble de compétences en tant que `/document/normalized_images/*`.

Les sorties peuvent être mappées à un index, utilisées comme entrée d’une compétence en aval, ou les deux, comme c’est le cas avec le code de langue. Dans l’index, un code de langue est utile pour le filtrage. En tant qu’entrée, le code de langue est utilisé par les compétences d’analyse de texte pour informer les règles linguistiques en matière de césure de mots.

Pour plus d’informations sur les principes de base des ensembles de compétences, consultez [Guide pratique pour définir un ensemble de compétences](cognitive-search-defining-skillset.md).

### <a name="ocr-skill"></a>Compétence de reconnaissance optique des caractères

La compétence **OCR** extrait le texte des images. Cette compétence suppose l’existence d’un champ normalized_images. Pour générer ce champ, plus loin dans ce didacticiel, nous définirons la configuration ```"imageAction"``` dans la définition de l’indexeur sur ```"generateNormalizedImages"```.

```csharp
private static OcrSkill CreateOcrSkill()
{
    List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>();
    inputMappings.Add(new InputFieldMappingEntry("image")
    {
        Source = "/document/normalized_images/*"
    });

    List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>();
    outputMappings.Add(new OutputFieldMappingEntry("text")
    {
        TargetName = "text"
    });

    OcrSkill ocrSkill = new OcrSkill(inputMappings, outputMappings)
    {
        Description = "Extract text (plain and structured) from image",
        Context = "/document/normalized_images/*",
        DefaultLanguageCode = OcrSkillLanguage.En,
        ShouldDetectOrientation = true
    };

    return ocrSkill;
}
```

### <a name="merge-skill"></a>Compétence de fusion de texte

Dans cette section, vous allez créer une compétence **Fusion** qui fusionne le champ de contenu de document avec le texte généré par la compétence de reconnaissance optique des caractères.

```csharp
private static MergeSkill CreateMergeSkill()
{
    List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>();
    inputMappings.Add(new InputFieldMappingEntry("text")
    {
        Source = "/document/content"
    });
    inputMappings.Add(new InputFieldMappingEntry("itemsToInsert")
    {
        Source = "/document/normalized_images/*/text"
    });
    inputMappings.Add(new InputFieldMappingEntry("offsets")
    {
        Source = "/document/normalized_images/*/contentOffset"
    });

    List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>();
    outputMappings.Add(new OutputFieldMappingEntry("mergedText")
    {
        TargetName = "merged_text"
    });

    MergeSkill mergeSkill = new MergeSkill(inputMappings, outputMappings)
    {
        Description = "Create merged_text which includes all the textual representation of each image inserted at the right location in the content field.",
        Context = "/document",
        InsertPreTag = " ",
        InsertPostTag = " "
    };

    return mergeSkill;
}
```

### <a name="language-detection-skill"></a>Compétence Détection de langue

La compétence **Détection de langue** détecte la langue du texte d’entrée et renvoie un code de langue unique pour chaque document soumis dans la requête. Nous allons utiliser la sortie de la compétence **Détection de langue** en tant qu'entrée pour la compétence **Fractionnement de texte** .

```csharp
private static LanguageDetectionSkill CreateLanguageDetectionSkill()
{
    List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>();
    inputMappings.Add(new InputFieldMappingEntry("text")
    {
        Source = "/document/merged_text"
    });

    List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>();
    outputMappings.Add(new OutputFieldMappingEntry("languageCode")
    {
        TargetName = "languageCode"
    });

    LanguageDetectionSkill languageDetectionSkill = new LanguageDetectionSkill(inputMappings, outputMappings)
    {
        Description = "Detect the language used in the document",
        Context = "/document"
    };

    return languageDetectionSkill;
}
```

### <a name="text-split-skill"></a>Compétence Fractionnement de texte

La compétence **Fractionnement** ci-dessous fractionnera le texte par pages et limitera la longueur de la page à 4 000 caractères comme indiqué par `String.Length`. L’algorithme essaiera de fractionner le texte en blocs dont la taille n'excède pas `maximumPageLength`. Dans ce cas, l’algorithme fera de son mieux pour arrêter la phrase sur une limite de phrase, de sorte que la taille du bloc peut être légèrement inférieure à `maximumPageLength`.

```csharp
private static SplitSkill CreateSplitSkill()
{
    List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>();
    inputMappings.Add(new InputFieldMappingEntry("text")
    {
        Source = "/document/merged_text"
    });
    inputMappings.Add(new InputFieldMappingEntry("languageCode")
    {
        Source = "/document/languageCode"
    });

    List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>();
    outputMappings.Add(new OutputFieldMappingEntry("textItems")
    {
        TargetName = "pages",
    });

    SplitSkill splitSkill = new SplitSkill(inputMappings, outputMappings)
    {
        Description = "Split content into pages",
        Context = "/document",
        TextSplitMode = TextSplitMode.Pages,
        MaximumPageLength = 4000,
        DefaultLanguageCode = SplitSkillLanguage.En
    };

    return splitSkill;
}
```

### <a name="entity-recognition-skill"></a>Compétence de reconnaissance d’entités

Cette instance `EntityRecognitionSkill` est configurée pour reconnaître le type de catégorie `organization`. La compétence **Reconnaissance d’entités** peut également reconnaître les types de catégories `person` et `location`.

Notez que le champ « contexte » contient la valeur ```"/document/pages/*"``` avec un astérisque, ce qui signifie que l’étape d’enrichissement est appelée pour chaque page sous ```"/document/pages"```.

```csharp
private static EntityRecognitionSkill CreateEntityRecognitionSkill()
{
    List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>();
    inputMappings.Add(new InputFieldMappingEntry("text")
    {
        Source = "/document/pages/*"
    });

    List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>();
    outputMappings.Add(new OutputFieldMappingEntry("organizations")
    {
        TargetName = "organizations"
    });

    EntityRecognitionSkill entityRecognitionSkill = new EntityRecognitionSkill(inputMappings, outputMappings)
    {
        Description = "Recognize organizations",
        Context = "/document/pages/*",
        DefaultLanguageCode = EntityRecognitionSkillLanguage.En
    };
    entityRecognitionSkill.Categories.Add(EntityCategory.Organization);

    return entityRecognitionSkill;
}
```

### <a name="key-phrase-extraction-skill"></a>Compétence Extraction de phrases clés

Comme l'instance `EntityRecognitionSkill` précédemment créée, la compétence **Extraction de phrases clés** est appelée pour chaque page du document.

```csharp
private static KeyPhraseExtractionSkill CreateKeyPhraseExtractionSkill()
{
    List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>();
    inputMappings.Add(new InputFieldMappingEntry("text")
    {
        Source = "/document/pages/*"
    });
    inputMappings.Add(new InputFieldMappingEntry("languageCode")
    {
        Source = "/document/languageCode"
    });

    List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>();
    outputMappings.Add(new OutputFieldMappingEntry("keyPhrases")
    {
        TargetName = "keyPhrases"
    });

    KeyPhraseExtractionSkill keyPhraseExtractionSkill = new KeyPhraseExtractionSkill(inputMappings, outputMappings)
    {
        Description = "Extract the key phrases",
        Context = "/document/pages/*",
        DefaultLanguageCode = KeyPhraseExtractionSkillLanguage.En
    };

    return keyPhraseExtractionSkill;
}
```

### <a name="build-and-create-the-skillset"></a>Générer et créer l'ensemble de compétences

Générez l'`Skillset` en utilisant les compétences que vous avez créées.

```csharp
private static SearchIndexerSkillset CreateOrUpdateDemoSkillSet(SearchIndexerClient indexerClient, IList<SearchIndexerSkill> skills,string cognitiveServicesKey)
{
    SearchIndexerSkillset skillset = new SearchIndexerSkillset("demoskillset", skills)
    {
        Description = "Demo skillset",
        CognitiveServicesAccount = new CognitiveServicesAccountKey(cognitiveServicesKey)
    };

    // Create the skillset in your search service.
    // The skillset does not need to be deleted if it was already created
    // since we are using the CreateOrUpdate method
    try
    {
        indexerClient.CreateOrUpdateSkillset(skillset);
    }
    catch (RequestFailedException ex)
    {
        Console.WriteLine("Failed to create the skillset\n Exception message: {0}\n", ex.Message);
        ExitProgram("Cannot continue without a skillset");
    }

    return skillset;
}
```

Ajoutez les lignes suivantes à `Main`.

```csharp
// Create the skills
Console.WriteLine("Creating the skills...");
OcrSkill ocrSkill = CreateOcrSkill();
MergeSkill mergeSkill = CreateMergeSkill();
EntityRecognitionSkill entityRecognitionSkill = CreateEntityRecognitionSkill();
LanguageDetectionSkill languageDetectionSkill = CreateLanguageDetectionSkill();
SplitSkill splitSkill = CreateSplitSkill();
KeyPhraseExtractionSkill keyPhraseExtractionSkill = CreateKeyPhraseExtractionSkill();

// Create the skillset
Console.WriteLine("Creating or updating the skillset...");
List<SearchIndexerSkill> skills = new List<SearchIndexerSkill>();
skills.Add(ocrSkill);
skills.Add(mergeSkill);
skills.Add(languageDetectionSkill);
skills.Add(splitSkill);
skills.Add(entityRecognitionSkill);
skills.Add(keyPhraseExtractionSkill);

SearchIndexerSkillset skillset = CreateOrUpdateDemoSkillSet(indexerClient, skills, cognitiveServicesKey);
```

### <a name="step-3-create-an-index"></a>Étape 3 : Création d'un index

Dans cette section, vous définissez le schéma d’index en spécifiant les champs à inclure dans l’index de recherche et les attributs de recherche pour chaque champ. Les champs ont un type et peuvent prendre des attributs qui déterminent la façon dont le champ est utilisé (pour la recherche, le tri, etc.). Les noms des champs dans un index ne sont pas tenus de correspondre exactement aux noms des champs dans la source. Dans une étape ultérieure, vous ajoutez des mappages de champs dans un indexeur pour connecter les champs sources et de destination. Pour cette étape, définissez l’index à l’aide de conventions d’affectation de noms de champs appropriées à votre application de recherche.

Cet exercice utilise les champs et les types de champ suivants :

| Noms de champs | Types de champ |
| --- | --- |
| `id` | Edm.String |
| `content` | Edm.String |
| `languageCode` | Edm.String |
| `keyPhrases` | List<Edm.String> |
| `organizations` | List<Edm.String> |

#### <a name="create-demoindex-class"></a>Créer la classe DemoIndex

Les champs de cet index sont définis à l’aide d’une classe de modèle. Chaque propriété de la classe de modèle comporte des attributs qui déterminent les comportements liés à la recherche du champ d’index correspondant. 

Nous allons ajouter la classe de modèle à un nouveau fichier C#. Cliquez avec le bouton droit sur votre projet, sélectionnez **Ajouter** > **Nouvel élément...** , sélectionnez « Classe » et nommez le fichier `DemoIndex.cs`, puis sélectionnez **Ajouter** .

Veillez à indiquer que vous souhaitez utiliser des types à partir des espaces de noms `Azure.Search.Documents.Indexes` et `System.Text.Json.Serialization`.

Ajoutez la définition de classe de modèle ci-dessous à `DemoIndex.cs` et incluez-la dans le même espace de noms que celui où vous créerez l’index.

```csharp
using Azure.Search.Documents.Indexes;
using System.Text.Json.Serialization;

namespace EnrichwithAI
{
    // The SerializePropertyNamesAsCamelCase is currently unsupported as of this writing. 
    // Replace it with JsonPropertyName
    public class DemoIndex
    {
        [SearchableField(IsSortable = true, IsKey = true)]
        [JsonPropertyName("id")]
        public string Id { get; set; }

        [SearchableField]
        [JsonPropertyName("content")]
        public string Content { get; set; }

        [SearchableField]
        [JsonPropertyName("languageCode")]
        public string LanguageCode { get; set; }

        [SearchableField]
        [JsonPropertyName("keyPhrases")]
        public string[] KeyPhrases { get; set; }

        [SearchableField]
        [JsonPropertyName("organizations")]
        public string[] Organizations { get; set; }
    }
}
```

Maintenant que vous avez défini une classe de modèle, de retour dans `Program.cs`, vous pouvez créer facilement une définition d’index. Le nom de cet index sera `demoindex`. Si un index portant ce nom existe déjà, il sera supprimé.

```csharp
private static SearchIndex CreateDemoIndex(SearchIndexClient indexClient)
{
    FieldBuilder builder = new FieldBuilder();
    var index = new SearchIndex("demoindex")
    {
        Fields = builder.Build(typeof(DemoIndex))
    };

    try
    {
        indexClient.GetIndex(index.Name);
        indexClient.DeleteIndex(index.Name);
    }
    catch (RequestFailedException ex) when (ex.Status == 404)
    {
        //if the specified index not exist, 404 will be thrown.
    }

    try
    {
        indexClient.CreateIndex(index);
    }
    catch (RequestFailedException ex)
    {
        Console.WriteLine("Failed to create the index\n Exception message: {0}\n", ex.Message);
        ExitProgram("Cannot continue without an index");
    }

    return index;
}
```

Pendant le test, vous serez peut-être amené à tenter de créer l’index plusieurs fois. Dès lors, vérifiez si l’index que vous vous apprêtez à créer existe avant de tenter de le créer.

Ajoutez les lignes suivantes à `Main`.

```csharp
// Create the index
Console.WriteLine("Creating the index...");
SearchIndex demoIndex = CreateDemoIndex(indexClient);
```

Ajoutez l’instruction using suivante pour résoudre la référence ambigüe.

```csharp
using Index = Azure.Search.Documents.Indexes.Models;
```

Pour en savoir plus sur les concepts d’index, consultez [Créer un index (API REST)](/rest/api/searchservice/create-index).

### <a name="step-4-create-and-run-an-indexer"></a>Étape 4 : Créer et exécuter un indexeur

À ce stade, vous avez créé une source de données, un ensemble de compétences et un index. Ces trois composants deviennent partie intégrante d’un [indexeur](search-indexer-overview.md) qui extrait chaque élément pour l’insérer dans une opération unique à plusieurs phases. Pour lier ces éléments en un indexeur, vous devez définir des mappages de champs.

* Les fieldMappings sont traités avant l’ensemble de compétences, en mappant les champs sources de la source de données sur des champs cibles dans un index. Si les noms et types de champ sont identiques aux deux extrémités, aucun mappage n’est nécessaire.

* Les outputFieldMappings sont traités après l’ensemble de compétences, en référençant les sourceFieldNames qui n’existent pas tant que le décodage de document ou l’enrichissement ne les ont pas créés. targetFieldName est un champ dans un index.

En plus de lier des entrées à des sorties, vous pouvez également utiliser les mappages de champs pour aplatir les structures de données. Pour plus d’informations, consultez [Guide pratique pour mapper des champs enrichis sur un index pouvant faire l’objet d’une recherche](cognitive-search-output-field-mapping.md).

```csharp
private static SearchIndexer CreateDemoIndexer(SearchIndexerClient indexerClient, SearchIndexerDataSourceConnection dataSource, SearchIndexerSkillset skillSet, SearchIndex index)
{
    IndexingParameters indexingParameters = new IndexingParameters()
    {
        MaxFailedItems = -1,
        MaxFailedItemsPerBatch = -1,
    };
    indexingParameters.Configuration.Add("dataToExtract", "contentAndMetadata");
    indexingParameters.Configuration.Add("imageAction", "generateNormalizedImages");

    SearchIndexer indexer = new SearchIndexer("demoindexer", dataSource.Name, index.Name)
    {
        Description = "Demo Indexer",
        SkillsetName = skillSet.Name,
        Parameters = indexingParameters
    };

    FieldMappingFunction mappingFunction = new FieldMappingFunction("base64Encode");
    mappingFunction.Parameters.Add("useHttpServerUtilityUrlTokenEncode", true);

    indexer.FieldMappings.Add(new FieldMapping("metadata_storage_path")
    {
        TargetFieldName = "id",
        MappingFunction = mappingFunction

    });
    indexer.FieldMappings.Add(new FieldMapping("content")
    {
        TargetFieldName = "content"
    });

    indexer.OutputFieldMappings.Add(new FieldMapping("/document/pages/*/organizations/*")
    {
        TargetFieldName = "organizations"
    });
    indexer.OutputFieldMappings.Add(new FieldMapping("/document/pages/*/keyPhrases/*")
    {
        TargetFieldName = "keyPhrases"
    });
    indexer.OutputFieldMappings.Add(new FieldMapping("/document/languageCode")
    {
        TargetFieldName = "languageCode"
    });

    try
    {
        indexerClient.GetIndexer(indexer.Name);
        indexerClient.DeleteIndexer(indexer.Name);
    }
    catch (RequestFailedException ex) when (ex.Status == 404)
    {
        //if the specified indexer not exist, 404 will be thrown.
    }

    try
    {
        indexerClient.CreateIndexer(indexer);
    }
    catch (RequestFailedException ex)
    {
        Console.WriteLine("Failed to create the indexer\n Exception message: {0}\n", ex.Message);
        ExitProgram("Cannot continue without creating an indexer");
    }

    return indexer;
}
```

Ajoutez les lignes suivantes à `Main`.

```csharp
// Create the indexer, map fields, and execute transformations
Console.WriteLine("Creating the indexer and executing the pipeline...");
SearchIndexer demoIndexer = CreateDemoIndexer(indexerClient, dataSource, skillset, demoIndex);
```

Le traitement de l’indexeur peut prendre un certain temps. Même si le jeu de données est petit, les compétences analytiques exigent des calculs intensifs. Certaines compétences, telles que l’analyse d’image, sont des opérations longues.

> [!TIP]
> La création d’un indexeur appelle le pipeline. En cas de problèmes pour atteindre les données, mapper les entrées et les sorties, ou ordonner les opérations, ceux-ci apparaissent à ce stade.

### <a name="explore-creating-the-indexer"></a>Explorer la création de l’indexeur

Le code affecte à `"maxFailedItems"` la valeur -1, ce qui indique au moteur d’indexation d’ignorer les erreurs au cours de l’importation des données. Cela est utile car très peu de documents figurent dans la source de données de démonstration. Pour une source de données plus volumineuse, vous définiriez une valeur supérieure à 0.

Notez également la définition de `"dataToExtract"` sur `"contentAndMetadata"`. Cette instruction indique à l’indexeur d’extraire automatiquement le contenu de fichiers de différents formats, ainsi que les métadonnées associées à chaque fichier.

Lorsque le contenu est extrait, vous pouvez définir `imageAction` pour extraire le texte des images trouvées dans la source de données. La configuration `"imageAction"` définie sur `"generateNormalizedImages"`, associée à la compétence de reconnaissance optique des caractères et à la compétence de fusion de texte, indique à l’indexeur d’extraire le texte des images (par exemple, le mot « stop » d’un panneau de signalisation Stop) et de l’incorporer dans le champ de contenu. Ce comportement s’applique aux images intégrées dans les documents (pensez à une image dans un fichier PDF), ainsi qu’aux images trouvées dans la source de données, par exemple un fichier JPG.

<a name="check-indexer-status"></a>

## <a name="4---monitor-indexing"></a>4 - Surveiller l’indexation

Une fois l’indexeur défini, il s’exécute automatiquement lorsque vous envoyez la demande. Selon les compétences cognitives que vous avez définies, l’indexation peut prendre plus de temps que prévu. Pour savoir si l’indexeur est toujours en cours d’exécution, utilisez la méthode `GetStatus`.

```csharp
private static void CheckIndexerOverallStatus(SearchIndexerClient indexerClient, SearchIndexer indexer)
{
    try
    {
        var demoIndexerExecutionInfo = indexerClient.GetIndexerStatus(indexer.Name);

        switch (demoIndexerExecutionInfo.Value.Status)
        {
            case IndexerStatus.Error:
                ExitProgram("Indexer has error status. Check the Azure Portal to further understand the error.");
                break;
            case IndexerStatus.Running:
                Console.WriteLine("Indexer is running");
                break;
            case IndexerStatus.Unknown:
                Console.WriteLine("Indexer status is unknown");
                break;
            default:
                Console.WriteLine("No indexer information");
                break;
        }
    }
    catch (RequestFailedException ex)
    {
        Console.WriteLine("Failed to get indexer overall status\n Exception message: {0}\n", ex.Message);
    }
}
```

`demoIndexerExecutionInfo` correspond à l'historique actuel d'état et d'exécution d'un indexeur.

Les avertissements sont courants avec certaines combinaisons de fichiers sources et de compétences et n’indiquent pas toujours un problème. Dans ce tutoriel, les avertissements sont sans gravité (par exemple, aucune entrée de texte à partir des fichiers JPEG).

Ajoutez les lignes suivantes à `Main`.

```csharp
// Check indexer overall status
Console.WriteLine("Check the indexer overall status...");
CheckIndexerOverallStatus(indexerClient, demoIndexer);
```

## <a name="5---search"></a>5 - Recherche

Dans les applications console des tutoriels sur Recherche cognitive Azure, nous ajoutons généralement un délai de 2 secondes avant l’exécution des requêtes qui retournent des résultats, mais comme l’enrichissement prend plusieurs minutes, nous allons fermer l’application console et utiliser une autre approche.

L’option la plus simple consiste à utiliser l’[Explorateur de recherche](search-explorer.md) dans le portail. Vous pouvez d’abord exécuter une requête vide qui retourne tous les documents ou une recherche plus ciblée qui retourne un contenu de champ créé par le pipeline. 

1. Dans le portail Azure, dans la page de présentation de la recherche, sélectionnez **index** .

1. Recherchez **`demoindex`** dans la liste. Il doit comporter 14 documents. Si le nombre de documents est égal à zéro, l’indexeur est toujours en cours d’exécution ou la page n’a pas encore été actualisée. 

1. Sélectionnez **`demoindex`** . L’Explorateur de recherche est le premier onglet.

1. Le contenu peut faire l’objet d’une recherche dès que le premier document est chargé. Pour vérifier que le contenu existe, exécutez une requête non spécifiée en cliquant sur **Rechercher** . Cette requête retourne tous les documents indexés, ce qui vous donne une idée du contenu de l’index.

1. Collez ensuite la chaîne suivante pour obtenir des résultats plus faciles à gérer : `search=*&$select=id, languageCode, organizations`

<a name="reset"></a>

## <a name="reset-and-rerun"></a>Réinitialiser et réexécuter

Dans les premières étapes expérimentales du développement, l’approche la plus pratique pour les itérations de conception consiste à supprimer les objets de Recherche cognitive Azure et à autoriser votre code à les reconstruire. Les noms des ressources sont uniques. La suppression d’un objet vous permet de le recréer en utilisant le même nom.

L’exemple de code pour ce tutoriel recherche les objets existants et les supprime pour vous permettre de réexécuter votre code. Vous pouvez aussi utiliser le portail pour supprimer les index, les indexeurs et les ensembles de compétences.

## <a name="takeaways"></a>Éléments importants à retenir

Ce tutoriel présente les étapes de base pour générer un pipeline d’indexation enrichie via la création de composants : une source de données, un ensemble de compétences, un index et un indexeur.

Les [compétences intégrées](cognitive-search-predefined-skills.md) ont été introduites, ainsi que la définition d’un ensemble de compétences et les mécanismes de chaînage de compétences via des entrées et des sorties. Vous avez également appris que `outputFieldMappings` est requis dans la définition de l’indexeur pour acheminer les valeurs enrichies du pipeline dans un index de recherche, sur un service de Recherche cognitive Azure.

Enfin, vous avez appris à tester les résultats et réinitialiser le système pour des itérations ultérieures. Vous avez appris qu’émettre des requêtes par rapport à l’index retourne la sortie créée par le pipeline d’indexation enrichie. Vous avez également appris à vérifier l’état de l’indexeur et quels objets supprimer avant de réexécuter un pipeline.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous travaillez dans votre propre abonnement, il est judicieux à la fin d’un projet de supprimer les ressources dont vous n’avez plus besoin. Les ressources laissées en cours d’exécution peuvent vous coûter de l’argent. Vous pouvez supprimer les ressources une par une, ou choisir de supprimer le groupe de ressources afin de supprimer l’ensemble des ressources.

Vous pouvez rechercher et gérer les ressources dans le portail à l’aide des liens Toutes les ressources ou Groupes de ressources situés dans le volet de navigation de gauche.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous êtes familiarisé avec tous les objets d’un pipeline d’enrichissement par IA, examinons de plus près les définitions des ensembles de compétences et les compétences individuelles.

> [!div class="nextstepaction"]
> [Guide pratique pour créer un ensemble de compétences](cognitive-search-defining-skillset.md)
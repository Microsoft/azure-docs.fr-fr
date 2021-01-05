---
title: 'Tutoriel : Coder une application cliente'
titleSuffix: Azure Digital Twins
description: Tutoriel expliquant comment écrire le code minimal d’une application cliente à l’aide du kit SDK (C#) .NET.
author: baanders
ms.author: baanders
ms.date: 11/02/2020
ms.topic: tutorial
ms.service: digital-twins
ms.openlocfilehash: 3225fff1c82822dee990804f934ada86068841e8
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011256"
---
# <a name="tutorial-coding-with-the-azure-digital-twins-apis"></a>Tutoriel : Codage avec les API Azure Digital Twins

Il arrive souvent aux développeurs qui travaillent avec Azure Digital Twins de devoir écrire une application cliente pour interagir avec leur instance du service Azure Digital Twins. Ce tutoriel destiné aux développeurs fournit une introduction à la programmation par rapport au service Azure Digital Twins à l’aide du [SDK Azure Digital Twins pour .NET (C#)](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true). Il décrit étape par étape comment écrire une application console cliente C# à partir de rien.

> [!div class="checklist"]
> * Configurer le projet
> * Démarrer avec le code de projet   
> * Exemple de code complet
> * Nettoyer les ressources
> * Étapes suivantes

## <a name="prerequisites"></a>Prérequis

Ce tutoriel utilise la ligne de commande pour la création et la configuration du projet. Vous pouvez donc utiliser n’importe quel éditeur de code pour effectuer les exercices.

Pour commencer, il vous faut :
* Un éditeur de code.
* **.NET Core 3.1** sur votre ordinateur de développement. Vous pouvez télécharger cette version du kit SDK .NET Core pour plusieurs plateformes à partir du site [Télécharger .NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/3.1).

### <a name="prepare-an-azure-digital-twins-instance"></a>Préparer une instance Azure Digital Twins

[!INCLUDE [Azure Digital Twins: instance prereq](../../includes/digital-twins-prereq-instance.md)]

[!INCLUDE [Azure Digital Twins: local credentials prereq (outer)](../../includes/digital-twins-local-credentials-outer.md)]

## <a name="set-up-project"></a>Configurer le projet

Une fois que vous êtes prêt à utiliser votre instance Azure Digital Twins, commencez à configurer le projet d’application cliente. 

Ouvrez une invite de commandes ou une autre fenêtre de console sur votre ordinateur, puis créez un répertoire de projet vide dans lequel vous souhaitez stocker votre travail au cours de ce tutoriel. Nommez le répertoire comme vous le souhaitez (par exemple *TutorielCodeDigitalTwins*).

Accédez au nouveau répertoire.

Une fois dans le répertoire du projet, **créez un projet d’application console .NET vide**. Dans la fenêtre de commande, vous pouvez exécuter la commande suivante afin de créer un projet C# minimal pour la console :

```cmd/sh
dotnet new console
```

Plusieurs fichiers seront alors créés dans votre répertoire, notamment un nommé *Program.cs* où vous écrirez la plupart de votre code.

Laissez la fenêtre de commande ouverte parce que vous allez l’utiliser tout au long du tutoriel.

Ensuite, **ajouter deux dépendances à votre projet** qui seront nécessaires pour utiliser Azure Digital Twins. Vous pouvez utiliser les liens ci-dessous pour accéder aux packages sur NuGet ; vous y trouverez les commandes de console (y compris pour l’interface CLI .NET) qui permettent d’ajouter à votre projet la dernière version de chaque package.
* [**Azure.DigitalTwins.Core**](https://www.nuget.org/packages/Azure.DigitalTwins.Core). Il s’agit du package pour le [SDK Azure Digital Twins pour .NET](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true). 
* [**Azure.Identity**](https://www.nuget.org/packages/Azure.Identity). Cette bibliothèque fournit des outils pour faciliter l’authentification auprès d’Azure.

## <a name="get-started-with-project-code"></a>Démarrer avec le code de projet

Dans cette section, vous allez commencer à écrire le code de votre nouveau projet d’application pour fonctionner avec Azure Digital Twins. Les actions couvertes sont les suivantes :
* Authentification auprès du service
* Chargement d’un modèle
* Interception des erreurs
* Création de jumeaux numériques
* Création de relations
* Interrogation de jumeaux numériques

Vous trouverez également une section contenant le code complet à la fin du tutoriel. Vous pouvez l’utiliser en guise de référence pour vérifier votre programme au fur et à mesure.

Pour commencer, ouvrez le fichier *Program.cs* dans un éditeur de code de votre choix. Vous verrez un modèle de code minimal qui ressemble à ceci :

```csharp
using System;

namespace DigitalTwinsCodeTutorial
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");
        }
    }
}
```

Tout d’abord, ajoutez des lignes `using` en haut du code pour tirer les dépendances nécessaires.

```csharp
using Azure.DigitalTwins.Core;
using Azure.Identity;
```

Ensuite, vous allez ajouter du code à ce fichier pour activer certaines fonctionnalités. 

### <a name="authenticate-against-the-service"></a>S’authentifier auprès du service

La première chose que votre application doit faire, c’est s’authentifier auprès du service Azure Digital Twins. Vous pouvez ensuite créer une classe cliente de service pour accéder aux fonctions du SDK.

Pour vous authentifier, vous avez besoin du *nom d’hôte* de votre instance Azure Digital Twins.

Dans *Program.cs*, collez le code suivant sous la ligne « Hello, World ! » dans la méthode `Main`. Définissez la valeur de `adtInstanceUrl` sur le *hostName* de votre instance Azure Digital Twins.

```csharp
string adtInstanceUrl = "https://<your-Azure-Digital-Twins-instance-hostName>"; 
var credential = new DefaultAzureCredential();
DigitalTwinsClient client = new DigitalTwinsClient(new Uri(adtInstanceUrl), credential);
Console.WriteLine($"Service client created – ready to go");
```

Enregistrez le fichier. 

Dans votre fenêtre de commande, exécutez le code avec cette commande : 

```cmd/sh
dotnet run
```

Cette opération restaure les dépendances lors de la première exécution, puis exécute le programme. 
* Si aucune erreur ne se produit, le programme imprime *Service client created - ready to go*.
* Étant donné qu’il n’y a pas encore de gestion des erreurs dans ce projet, si un problème quelconque se produit, une exception est levée par le code.

### <a name="upload-a-model"></a>Charger un modèle

Azure Digital Twins n’a aucun vocabulaire de domaine intrinsèque. C’est vous qui définissez, à l’aide de **modèles**, les types d’éléments de votre environnement que vous pouvez représenter dans Azure Digital Twins. Les [modèles](concepts-twins-graph.md) sont similaires aux classes dans les langages de programmation orientés objet. Ils fournissent des modèles de [jumeaux numériques](concepts-models.md) définis par l’utilisateur à suivre et à instancier ultérieurement. Ils sont écrits dans un langage de type JSON appelé **DTDL (Digital Twins Definition Language)** .

La première étape de la création d’une solution Azure Digital Twins consiste à définir au moins un modèle dans un fichier DTDL.

Dans le répertoire où vous avez créé votre projet, créez un fichier *.json* nommé *SampleModel.json*. Collez-y le corps de fichier suivant : 

```json
{
  "@id": "dtmi:example:SampleModel;1",
  "@type": "Interface",
  "displayName": "SampleModel",
  "contents": [
    {
      "@type": "Relationship",
      "name": "contains"
    },
    {
      "@type": "Property",
      "name": "data",
      "schema": "string"
    }
  ],
  "@context": "dtmi:dtdl:context;2"
}
```

> [!TIP]
> Si vous utilisez Visual Studio pour ce tutoriel, vous souhaiterez peut-être sélectionner le nouveau fichier JSON et affecter la valeur *Copier si plus récent* ou *Toujours copier* à la propriété *Copier dans le répertoire de sortie* dans l’inspecteur de propriété. Cela permettra à Visual Studio de trouver le fichier JSON avec le chemin par défaut quand vous exécuterez le programme avec **F5** lors des étapes suivantes du tutoriel.

> [!TIP] 
> Il existe un [exemple de validateur DTDL](/samples/azure-samples/dtdl-validator/dtdl-validator) indépendant du langage qui vous permet de vérifier les documents de modèle et la validité du DTDL. Il repose sur la bibliothèque de l’analyseur DTDL, qui est décrite en détail dans [*Guide pratique pour analyser et valider les modèles*](how-to-parse-models.md).

Ensuite, ajoutez du code à *Program.cs* pour charger le modèle que vous venez de créer dans votre instance Azure Digital Twins.

Tout d’abord, ajoutez quelques instructions `using` au début du fichier :

```csharp
using System.Threading.Tasks;
using System.IO;
using System.Collections.Generic;
using Azure;
```

Ensuite, préparez l’utilisation des méthodes asynchrones dans le SDK de service C# en modifiant la signature de la méthode `Main` pour autoriser l’exécution asynchrone. 

```csharp
static async Task Main(string[] args)
```

> [!NOTE]
> L’utilisation de `async` n’est pas strictement nécessaire, car le SDK fournit également des versions synchrones de tous les appels. Ce tutoriel utilise `async`.

Vient ensuite le premier bloc de code qui interagit avec le service Azure Digital Twins. Ce code charge le fichier DTDL que vous avez créé à partir de votre disque, puis le charge sur votre instance de service Azure Digital Twins. 

Collez le code suivant sous le code d’autorisation que vous avez ajouté plus tôt.

```csharp
Console.WriteLine();
Console.WriteLine($"Upload a model");
var typeList = new List<string>();
string dtdl = File.ReadAllText("SampleModel.json");
typeList.Add(dtdl);
// Upload the model to the service
await client.CreateModelsAsync(typeList);
```

Dans votre fenêtre de commande, exécutez le programme avec cette commande : 

```cmd/sh
dotnet run
```
La mention « Upload a model » sera affichée dans la sortie, indiquant que ce code a été atteint, mais il n’existe pas encore de sortie pour préciser si le chargement a été effectué correctement.

Pour ajouter une instruction print indiquant tous les modèles chargés avec succès sur l’instance, ajoutez le code suivant juste après la section précédente :

```csharp
// Read a list of models back from the service
Console.WriteLine("Models uploaded to the instance:");
AsyncPageable<DigitalTwinsModelData> modelDataList = client.GetModelsAsync();
await foreach (DigitalTwinsModelData md in modelDataList)
{
    Console.WriteLine($"{md.Id}");
}
```

**Avant de réexécuter le programme pour tester ce nouveau code**, rappelez-vous que la dernière fois que vous avez exécuté le programme, vous avez déjà chargé votre modèle. Azure Digital Twins ne vous permet pas de charger deux fois le même modèle. Par conséquent, si vous tentez de charger à nouveau le même modèle, le programme doit lever une exception.

Avec cela à l’esprit, réexécutez le programme avec cette commande dans votre fenêtre de commande :

```cmd/sh
dotnet run
```

Le programme doit lever une exception. Quand vous essayez de charger un modèle qui a déjà été chargé, le service retourne une erreur de « requête incorrecte » par le biais de l’API REST. Par conséquent, le SDK client Azure Digital Twins lèvera à son tour une exception pour chaque code de retour de service autre qu’un succès. 

La section suivante traite des exceptions de ce type et explique comment les gérer dans votre code.

### <a name="catch-errors"></a>Intercepter les erreurs

Pour empêcher que le programme se plante, vous pouvez ajouter du code d’exception autour du code de chargement du modèle. Encapsulez l’appel client existant `await client.CreateModelsAsync(typeList)` dans un gestionnaire try/catch comme suit :

```csharp
try {
    await client.CreateModelsAsync(typeList);
} catch (RequestFailedException rex) {
    Console.WriteLine($"Load model: {rex.Status}:{rex.Message}");
}
```

À présent, si vous exécutez le programme avec `dotnet run` dans votre fenêtre de commande, vous verrez que vous obtenez un code d’erreur. La sortie du code de création de modèle indique l’erreur suivante :

:::image type="content" source= "media/tutorial-code/model-error.png" alt-text="Sortie du programme, affichant le message « 409:Service request failed. État : 409 (Conflict). », suivi du contenu de l’erreur indiquant que dtmi:example:SampleModel;1 existe déjà":::

Désormais, ce tutoriel encapsulera tous les appels aux méthodes de service dans des gestionnaires try/catch.

### <a name="create-digital-twins"></a>Créer des jumeaux numériques

Maintenant que vous avez chargé un modèle dans Azure Digital Twins, vous pouvez utiliser cette définition de modèle pour créer des **jumeaux numériques**. Les [jumeaux numériques](concepts-twins-graph.md) sont des instances d’un modèle ; ils représentent les entités au sein de votre environnement d’entreprise (par exemple les capteurs dans une ferme, les salles d’un bâtiment ou les voyants d’une voiture). Cette section crée quelques jumeaux numériques basés sur le modèle que vous avez chargé.

Ajoutez le code suivant à la fin de la méthode `Main` pour créer et initialiser trois jumeaux numériques basés sur ce modèle.

```csharp
// Initialize twin data
BasicDigitalTwin twinData = new BasicDigitalTwin();
twinData.Metadata.ModelId = "dtmi:example:SampleModel;1";
twinData.Contents.Add("data", $"Hello World!");

string prefix="sampleTwin-";
for(int i=0; i<3; i++) {
    try {
        twinData.Id = $"{prefix}{i}";
        await client.CreateOrReplaceDigitalTwinAsync<BasicDigitalTwin>(twinData.Id, twinData);
        Console.WriteLine($"Created twin: {prefix}{i}");
    } catch(RequestFailedException rex) {
        Console.WriteLine($"Create twin error: {rex.Status}:{rex.Message}");  
    }
}
```

Dans votre fenêtre de commande, exécutez le programme avec `dotnet run`. Dans la sortie, recherchez les messages print indiquant que *sampleTwin-0*, *sampleTwin-1* et *sampleTwin-2* ont été créés. 

Ensuite, réexécutez le programme. 

Notez qu’aucune erreur n’est générée quand les jumeaux sont créés la deuxième fois, bien qu’ils existent déjà après la première exécution. Contrairement à la création de modèle, la création de jumeau est, au niveau REST, un appel *PUT* avec une sémantique *upsert*. Cela signifie que si un jumeau existe déjà, une tentative de création du même jumeau remplacera simplement le jumeau d’origine. Aucune erreur n’est levée.

### <a name="create-relationships"></a>Créer des relations

Ensuite, vous pouvez créer des **relations** entre les jumeaux que vous avez créés, afin de les raccorder sur un **graphe de jumeaux**. Les [graphes de jumeaux](concepts-twins-graph.md) servent à représenter votre environnement entier.

Ajoutez une **nouvelle méthode statique** à la classe `Program`, sous la méthode `Main` (le code a désormais deux méthodes) :

```csharp
public async static Task CreateRelationship(DigitalTwinsClient client, string srcId, string targetId)
{
    var relationship = new BasicRelationship
    {
        TargetId = targetId,
        Name = "contains"
    };

    try
    {
        string relId = $"{srcId}-contains->{targetId}";
        await client.CreateOrReplaceRelationshipAsync(srcId, relId, relationship);
        Console.WriteLine("Created relationship successfully");
    }
    catch (RequestFailedException rex) {
        Console.WriteLine($"Create relationship error: {rex.Status}:{rex.Message}");
    }
}
```

Ensuite, ajoutez le code suivant à la fin de la méthode `Main` pour appeler la méthode `CreateRelationship` et utiliser le code que vous venez d’écrire :

```csharp
// Connect the twins with relationships
await CreateRelationship(client, "sampleTwin-0", "sampleTwin-1");
await CreateRelationship(client, "sampleTwin-0", "sampleTwin-2");
```

Dans votre fenêtre de commande, exécutez le programme avec `dotnet run`. Dans la sortie, recherchez des instructions print indiquant que les deux relations ont été créées avec succès.

Notez qu’Azure Digital Twins ne vous permet pas de créer une relation s’il en existe déjà une portant le même ID. Par conséquent, si vous exécutez le programme plusieurs fois, vous verrez des exceptions lors de la création de la relation. Ce code intercepte les exceptions et les ignore. 

### <a name="list-relationships"></a>Lister les relations

Le code que vous allez ensuite ajouter vous permet d’afficher la liste des relations que vous avez créées.

Ajoutez la **nouvelle méthode** suivante à la classe `Program` :

```csharp
public async static Task ListRelationships(DigitalTwinsClient client, string srcId)
{
    try {
        AsyncPageable<BasicRelationship> results = client.GetRelationshipsAsync<BasicRelationship>(srcId);
        Console.WriteLine($"Twin {srcId} is connected to:");
        await foreach (BasicRelationship rel in results)
        {
            Console.WriteLine($" -{rel.Name}->{rel.TargetId}");
        }
    } catch (RequestFailedException rex) {
        Console.WriteLine($"Relationship retrieval error: {rex.Status}:{rex.Message}");   
    }
}
```

Ensuite, ajoutez le code suivant à la fin de la méthode `Main` pour appeler le code `ListRelationships` :

```csharp
//List the relationships
await ListRelationships(client, "sampleTwin-0");
```

Dans votre fenêtre de commande, exécutez le programme avec `dotnet run`. Vous devez voir une liste de toutes les relations que vous avez créées dans une instruction de sortie similaire à celle-ci :

:::image type="content" source= "media/tutorial-code/list-relationships.png" alt-text="Sortie du programme, contenant un message indiquant « Twin sampleTwin-0 is connected to: contains->sampleTwin-1, -contains->sampleTwin-2 »":::

### <a name="query-digital-twins"></a>Interroger des jumeaux numériques

L’une des principales fonctionnalités d’Azure Digital Twins est la capacité à [interroger facilement et efficacement](concepts-query-language.md) votre graphe de jumeaux pour répondre à des questions sur votre environnement.

La dernière section de code à ajouter dans ce tutoriel exécute une requête sur l’instance Azure Digital Twins. La requête utilisée dans cet exemple retourne tous les jumeaux numériques de l’instance.

Ajoutez cette instruction `using` pour permettre l’utilisation de la classe `JsonSerializer` pour présenter plus aisément les informations de jumeau numérique :

```csharp
using System.Text.Json;
```

Ensuite, ajoutez le code suivant à la fin de la méthode `Main` :

```csharp
// Run a query for all twins   
string query = "SELECT * FROM digitaltwins";
AsyncPageable<BasicDigitalTwin> result = client.QueryAsync<BasicDigitalTwin>(query);

await foreach (BasicDigitalTwin twin in result)
{
    Console.WriteLine(JsonSerializer.Serialize(twin));
    Console.WriteLine("---------------");
}
```

Dans votre fenêtre de commande, exécutez le programme avec `dotnet run`. La sortie doit afficher tous les jumeaux numériques de cette instance.

## <a name="complete-code-example"></a>Exemple de code complet

À ce stade du tutoriel, vous disposez d’une application cliente complète, capable d’effectuer des actions de base sur Azure Digital Twins. Pour référence, le code complet du programme dans *Program.cs* est indiqué ci-dessous :

```csharp
using System;
using Azure.DigitalTwins.Core;
using Azure.Identity;
using System.Threading.Tasks;
using System.IO;
using System.Collections.Generic;
using Azure;
using System.Text.Json;

namespace minimal
{
    class Program
    {
        static async Task Main(string[] args)
        {
            Console.WriteLine("Hello World!");
            
            string adtInstanceUrl = "https://<your-Azure-Digital-Twins-instance-hostName>"; 
            
            var credential = new DefaultAzureCredential();
            DigitalTwinsClient client = new DigitalTwinsClient(new Uri(adtInstanceUrl), credential);
            Console.WriteLine($"Service client created – ready to go");

            Console.WriteLine();
            Console.WriteLine($"Upload a model");
            var typeList = new List<string>();
            string dtdl = File.ReadAllText("SampleModel.json");
            typeList.Add(dtdl);

            // Upload the model to the service
            try {
                await client.CreateModelsAsync(typeList);
            } catch (RequestFailedException rex) {
                Console.WriteLine($"Load model: {rex.Status}:{rex.Message}");
            }
            // Read a list of models back from the service
            Console.WriteLine("Models uploaded to the instance:");
            AsyncPageable<DigitalTwinsModelData> modelDataList = client.GetModelsAsync();
            await foreach (DigitalTwinsModelData md in modelDataList)
            {
                Console.WriteLine($"{md.Id}");
            }

            // Initialize twin data
            BasicDigitalTwin twinData = new BasicDigitalTwin();
            twinData.Metadata.ModelId = "dtmi:example:SampleModel;1";
            twinData.Contents.Add("data", $"Hello World!");
            
            string prefix="sampleTwin-";
            for(int i=0; i<3; i++) {
                try {
                    twinData.Id = $"{prefix}{i}";
                    await client.CreateOrReplaceDigitalTwinAsync<BasicDigitalTwin>(twinData.Id, twinData);
                    Console.WriteLine($"Created twin: {prefix}{i}");
                } catch(RequestFailedException rex) {
                    Console.WriteLine($"Create twin error: {rex.Status}:{rex.Message}");  
                }
            }

            // Connect the twins with relationships
            await CreateRelationship(client, "sampleTwin-0", "sampleTwin-1");
            await CreateRelationship(client, "sampleTwin-0", "sampleTwin-2");

            //List the relationships
            await ListRelationships(client, "sampleTwin-0");

            // Run a query for all twins   
            string query = "SELECT * FROM digitaltwins";
            AsyncPageable<BasicDigitalTwin> result = client.QueryAsync<BasicDigitalTwin>(query);
            
            await foreach (BasicDigitalTwin twin in result)
            {
                Console.WriteLine(JsonSerializer.Serialize(twin));
                Console.WriteLine("---------------");
            }
        }

        public async static Task CreateRelationship(DigitalTwinsClient client, string srcId, string targetId)
        {
            var relationship = new BasicRelationship
            {
                TargetId = targetId,
                Name = "contains"
            };
        
            try
            {
                string relId = $"{srcId}-contains->{targetId}";
                await client.CreateOrReplaceRelationshipAsync(srcId, relId, relationship);
                Console.WriteLine("Created relationship successfully");
            }
            catch (RequestFailedException rex) {
                Console.WriteLine($"Create relationship error: {rex.Status}:{rex.Message}");
            }
        }
        
        public async static Task ListRelationships(DigitalTwinsClient client, string srcId)
        {
            try {
                AsyncPageable<BasicRelationship> results = client.GetRelationshipsAsync<BasicRelationship>(srcId);
                Console.WriteLine($"Twin {srcId} is connected to:");
                await foreach (BasicRelationship rel in results)
                {
                    Console.WriteLine($" -{rel.Name}->{rel.TargetId}");
                }
            } catch (RequestFailedException rex) {
                Console.WriteLine($"Relationship retrieval error: {rex.Status}:{rex.Message}");   
            }
        }

    }
}
```
## <a name="clean-up-resources"></a>Nettoyer les ressources
 
L’instance utilisée dans ce tutoriel peut être réutilisée dans le tutoriel suivant, intitulé [*Tutoriel : Explorer les bases avec un exemple d’application cliente*](tutorial-command-line-app.md). Si vous envisagez de passer au tutoriel suivant, vous pouvez conserver l’instance Azure Digital Twins que vous avez configurée ici.
 
[!INCLUDE [digital-twins-cleanup-basic.md](../../includes/digital-twins-cleanup-basic.md)]

Pour finir, supprimez le dossier de projet que vous avez créé sur votre ordinateur local.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé une application console cliente .NET à partir de rien. Vous avez écrit du code pour cette application cliente afin d’effectuer les actions de base sur une instance Azure Digital Twins.

Passez au tutoriel suivant pour découvrir les opérations que vous pouvez effectuer avec cet exemple d’application cliente : 

> [!div class="nextstepaction"]
> [*Tutoriel : Explorer les bases avec un exemple d’application cliente*](tutorial-command-line-app.md)

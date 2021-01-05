---
title: Vérifiez le texte par rapport à une liste de termes personnalisée en C# - Content Moderator
titleSuffix: Azure Cognitive Services
description: Comment modérer du texte avec des listes de termes personnalisées à l’aide du Kit de développement logiciel (SDK) Content Moderator pour C#.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 10/24/2019
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: 93d90232fb530a6c14c40558fc6a9974a1da42de
ms.sourcegitcommit: 693df7d78dfd5393a28bf1508e3e7487e2132293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92900928"
---
# <a name="check-text-against-a-custom-term-list-in-c"></a>Vérifier du texte par rapport à une liste de termes personnalisée en C#

La liste générale de termes par défaut d’Azure Content Moderator est suffisante pour la plupart des besoins de modération de contenu. Toutefois, vous devrez probablement rechercher des termes spécifiques à votre organisation. Par exemple, vous voudrez peut-être identifier les noms de concurrents pour un examen approfondi. 

Vous pouvez utiliser le [kit SDK Content Moderator pour .NET](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) afin de créer des listes de termes personnalisées à utiliser avec l’API Modération de texte.

Cet article fournit des informations et des exemples de code qui vont vous aider à prendre en main le Kit de développement logiciel (SDK) Content Moderator pour .NET afin d’effectuer les opérations suivantes :
- Créer une liste.
- Ajouter des termes à une liste.
- Passer au crible un texte pour détecter des termes par rapport aux termes d’une liste.
- Supprimer des termes d’une liste.
- Supprimer une liste.
- Modifier les informations d’une liste.
- Actualiser l’index afin que les changements apportés à la liste soient inclus dans une nouvelle analyse.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/cognitive-services/) avant de commencer. 

## <a name="sign-up-for-content-moderator-services"></a>S’inscrire aux services Content Moderator

Avant de pouvoir utiliser les services Content Moderator sur l’API REST ou le Kit SDK, vous avez besoin d’une clé d’abonnement. Dans le [portail Azure](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesContentModerator), abonnez-vous au service Content Moderator pour en obtenir une.

## <a name="create-your-visual-studio-project"></a>Créer votre projet Visual Studio

1. Ajoutez un nouveau projet **Console app (.NET Framework)** à votre solution.

1. Nommez le projet **TermLists** . Sélectionnez ce projet comme unique projet de démarrage de la solution.

### <a name="install-required-packages"></a>Installer les packages nécessaires

Installez les packages NuGet suivants pour le projet TermLists :

- Microsoft.Azure.CognitiveServices.ContentModerator
- Microsoft.Rest.ClientRuntime
- Microsoft.Rest.ClientRuntime.Azure
- Newtonsoft.Json

### <a name="update-the-programs-using-statements"></a>Mettre à jour les instructions using du programme

Ajoutez les instructions `using` suivantes.

```csharp
using Microsoft.Azure.CognitiveServices.ContentModerator;
using Microsoft.Azure.CognitiveServices.ContentModerator.Models;
using Newtonsoft.Json;
using System;
using System.Collections.Generic;
using System.IO;
using System.Threading;
```

### <a name="create-the-content-moderator-client"></a>Créer le client Content Moderator

Ajoutez le code suivant afin de créer un client Content Moderator pour votre abonnement. Mettez à jour les champs `AzureEndpoint` et `CMSubscriptionKey` avec les valeurs de votre URL de point de terminaison et de votre clé d’abonnement. Vous pouvez les trouver sous l’onglet **Démarrage rapide** de votre ressource dans le Portail Azure.

```csharp
/// <summary>
/// Wraps the creation and configuration of a Content Moderator client.
/// </summary>
/// <remarks>This class library contains insecure code. If you adapt this 
/// code for use in production, use a secure method of storing and using
/// your Content Moderator subscription key.</remarks>
public static class Clients
{
    /// <summary>
    /// The base URL fragment for Content Moderator calls.
    /// </summary>
    private static readonly string AzureEndpoint = "YOUR ENDPOINT URL";

    /// <summary>
    /// Your Content Moderator subscription key.
    /// </summary>
    private static readonly string CMSubscriptionKey = "YOUR API KEY";

    /// <summary>
    /// Returns a new Content Moderator client for your subscription.
    /// </summary>
    /// <returns>The new client.</returns>
    /// <remarks>The <see cref="ContentModeratorClient"/> is disposable.
    /// When you have finished using the client,
    /// you should dispose of it either directly or indirectly. </remarks>
    public static ContentModeratorClient NewClient()
    {
        // Create and initialize an instance of the Content Moderator API wrapper.
        ContentModeratorClient client = new ContentModeratorClient(new ApiKeyServiceClientCredentials(CMSubscriptionKey));

        client.Endpoint = AzureEndpoint;
        return client;
    }
}
```

### <a name="add-private-properties"></a>Ajouter des propriétés privées

Ajoutez les propriétés privées suivantes dans la classe Program de l’espace de noms TermLists.

```csharp
/// <summary>
/// The language of the terms in the term lists.
/// </summary>
private const string lang = "eng";

/// <summary>
/// The minimum amount of time, in milliseconds, to wait between calls
/// to the Content Moderator APIs.
/// </summary>
private const int throttleRate = 3000;

/// <summary>
/// The number of minutes to delay after updating the search index before
/// performing image match operations against the list.
/// </summary>
private const double latencyDelay = 0.5;
```

## <a name="create-a-term-list"></a>Créer une liste de termes

Vous créez une liste de termes avec **ContentModeratorClient.ListManagementTermLists.Create** . Le premier paramètre de **Create** est une chaîne qui contient un type MIME, qui doit être « application/json ». Pour plus d’informations, consultez les [informations de référence sur l’API](https://westus2.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f67f). Le deuxième paramètre est un objet **Body** qui contient un nom et une description de la nouvelle liste de termes.

> [!NOTE]
> Il existe une limite maximale de **5 listes de termes** , chaque liste ne devant **pas dépasser 10 000 termes** .

Ajoutez la définition de méthode suivante à la classe Program de l’espace de noms TermLists.

> [!NOTE]
> Votre clé de service Content Moderator a une limite de fréquence des requêtes par seconde. Si vous dépassez cette limite, le Kit de développement logiciel (SDK) lève une exception avec le code d’erreur 429. Une clé de niveau gratuit a une limite de fréquence d’une demande par seconde (RSP).

```csharp
/// <summary>
/// Creates a new term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <returns>The term list ID.</returns>
static string CreateTermList (ContentModeratorClient client)
{
    Console.WriteLine("Creating term list.");

    Body body = new Body("Term list name", "Term list description");
    TermList list = client.ListManagementTermLists.Create("application/json", body);
    if (false == list.Id.HasValue)
    {
        throw new Exception("TermList.Id value missing.");
    }
    else
    {
        string list_id = list.Id.Value.ToString();
        Console.WriteLine("Term list created. ID: {0}.", list_id);
        Thread.Sleep(throttleRate);
        return list_id;
    }
}
```

## <a name="update-term-list-name-and-description"></a>Mettre à jour le nom et la description d’une liste de termes

Mettez à jour les informations d’une liste de termes avec **ContentModeratorClient.ListManagementTermLists.Update** . Le premier paramètre d’ **Update** est l’ID de la liste de termes. Le deuxième paramètre est un type MIME, qui doit être « application/json ». Pour plus d’informations, consultez les [informations de référence sur l’API](https://westus2.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f685). Le troisième paramètre est un objet **Body** qui contient le nouveau nom et la nouvelle description.

Ajoutez la définition de méthode suivante à la classe Program de l’espace de noms TermLists.

```csharp
/// <summary>
/// Update the information for the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list to update.</param>
/// <param name="name">The new name for the term list.</param>
/// <param name="description">The new description for the term list.</param>
static void UpdateTermList (ContentModeratorClient client, string list_id, string name = null, string description = null)
{
    Console.WriteLine("Updating information for term list with ID {0}.", list_id);
    Body body = new Body(name, description);
    client.ListManagementTermLists.Update(list_id, "application/json", body);
    Thread.Sleep(throttleRate);
}
```

## <a name="add-a-term-to-a-term-list"></a>Ajoutez un terme à une liste de termes

Ajoutez la définition de méthode suivante à la classe Program de l’espace de noms TermLists.

```csharp
/// <summary>
/// Add a term to the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list to update.</param>
/// <param name="term">The term to add to the term list.</param>
static void AddTerm (ContentModeratorClient client, string list_id, string term)
{
    Console.WriteLine("Adding term \"{0}\" to term list with ID {1}.", term, list_id);
    client.ListManagementTerm.AddTerm(list_id, term, lang);
    Thread.Sleep(throttleRate);
}
```

## <a name="get-all-terms-in-a-term-list"></a>Obtenir tous les termes d’une liste de termes

Ajoutez la définition de méthode suivante à la classe Program de l’espace de noms TermLists.

```csharp
/// <summary>
/// Get all terms in the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list from which to get all terms.</param>
static void GetAllTerms(ContentModeratorClient client, string list_id)
{
    Console.WriteLine("Getting terms in term list with ID {0}.", list_id);
    Terms terms = client.ListManagementTerm.GetAllTerms(list_id, lang);
    TermsData data = terms.Data;
    foreach (TermsInList term in data.Terms)
    {
        Console.WriteLine(term.Term);
    }
    Thread.Sleep(throttleRate);
}
```

## <a name="add-code-to-refresh-the-search-index"></a>Ajouter du code pour actualiser l’index de recherche

Après avoir apporté des changements à une liste de termes, actualisez son index de recherche pour inclure ces changements la prochaine fois que vous utilisez la liste de termes pour passer le texte au crible. Ce fonctionnement est semblable à celui d’un moteur de recherche sur votre bureau (s’il est activé) ou d’un moteur de recherche web qui actualise en permanence son index pour inclure les nouveaux fichiers ou les nouvelles pages.

Actualisez un index de recherche de liste de termes avec **ContentModeratorClient.ListManagementTermLists.RefreshIndexMethod** .

Ajoutez la définition de méthode suivante à la classe Program de l’espace de noms TermLists.

```csharp
/// <summary>
/// Refresh the search index for the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list to refresh.</param>
static void RefreshSearchIndex (ContentModeratorClient client, string list_id)
{
    Console.WriteLine("Refreshing search index for term list with ID {0}.", list_id);
    client.ListManagementTermLists.RefreshIndexMethod(list_id, lang);
    Thread.Sleep((int)(latencyDelay * 60 * 1000));
}
```

## <a name="screen-text-using-a-term-list"></a>Passer au crible du texte à l’aide d’une liste de termes

Passez au crible du texte à l’aide d’une liste de termes avec **ContentModeratorClient.TextModeration.ScreenText** , qui prend les paramètres suivants.

- La langue des termes de la liste.
- Un type MIME, qui peut être « text/html », « text/xml », « text/markdown » ou « text/plain ».
- Le texte à passer au crible.
- Une valeur booléenne. Définissez ce champ sur **true** pour corriger automatiquement le texte avant de le passer au crible.
- Une valeur booléenne. Définissez ce champ sur **true** pour détecter des données personnelles dans le texte.
- L’ID de la liste de termes.

Pour plus d’informations, consultez les [informations de référence sur l’API](https://westus2.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66f).

**ScreenText** retourne un objet **Screen** , qui a une propriété **Terms** listant tous les termes détectés par Content Moderator lors du passage au crible du texte. Notez que si Content Moderator n’a détecté aucun terme lors du passage au crible du texte, la propriété **Terms** a la valeur **null** .

Ajoutez la définition de méthode suivante à la classe Program de l’espace de noms TermLists.

```csharp
/// <summary>
/// Screen the indicated text for terms in the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list to use to screen the text.</param>
/// <param name="text">The text to screen.</param>
static void ScreenText (ContentModeratorClient client, string list_id, string text)
{
    Console.WriteLine("Screening text: \"{0}\" using term list with ID {1}.", text, list_id);
    Screen screen = client.TextModeration.ScreenText(lang, "text/plain", text, false, false, list_id);
    if (null == screen.Terms)
    {
        Console.WriteLine("No terms from the term list were detected in the text.");
    }
    else
    {
        foreach (DetectedTerms term in screen.Terms)
        {
            Console.WriteLine(String.Format("Found term: \"{0}\" from list ID {1} at index {2}.", term.Term, term.ListId, term.Index));
        }
    }
    Thread.Sleep(throttleRate);
}
```

## <a name="delete-terms-and-lists"></a>Supprimer des termes et des listes

La suppression d’un terme ou d’une liste est simple. Utilisez le SDK pour effectuer les tâches suivantes :

- Supprimer un terme. ( **ContentModeratorClient.ListManagementTerm.DeleteTerm** )
- Supprimer tous les termes d’une liste sans supprimer la liste. ( **ContentModeratorClient.ListManagementTerm.DeleteAllTerms** )
- Supprimer une liste et tout son contenu. ( **ContentModeratorClient.ListManagementTermLists.Delete** )

### <a name="delete-a-term"></a>Supprimer un terme

Ajoutez la définition de méthode suivante à la classe Program de l’espace de noms TermLists.

```csharp
/// <summary>
/// Delete a term from the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list from which to delete the term.</param>
/// <param name="term">The term to delete.</param>
static void DeleteTerm (ContentModeratorClient client, string list_id, string term)
{
    Console.WriteLine("Removed term \"{0}\" from term list with ID {1}.", term, list_id);
    client.ListManagementTerm.DeleteTerm(list_id, term, lang);
    Thread.Sleep(throttleRate);
}
```

### <a name="delete-all-terms-in-a-term-list"></a>Supprimer tous les termes d’une liste de termes

Ajoutez la définition de méthode suivante à la classe Program de l’espace de noms TermLists.

```csharp
/// <summary>
/// Delete all terms from the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list from which to delete all terms.</param>
static void DeleteAllTerms (ContentModeratorClient client, string list_id)
{
    Console.WriteLine("Removing all terms from term list with ID {0}.", list_id);
    client.ListManagementTerm.DeleteAllTerms(list_id, lang);
    Thread.Sleep(throttleRate);
}
```

### <a name="delete-a-term-list"></a>Supprimer une liste de termes

Ajoutez la définition de méthode suivante à la classe Program de l’espace de noms TermLists.

```csharp
/// <summary>
/// Delete the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list to delete.</param>
static void DeleteTermList (ContentModeratorClient client, string list_id)
{
    Console.WriteLine("Deleting term list with ID {0}.", list_id);
    client.ListManagementTermLists.Delete(list_id);
    Thread.Sleep(throttleRate);
}
```

## <a name="compose-the-main-method"></a>Composer la méthode Main

Ajoutez la définition de méthode **Main** à la classe **Program** de l’espace de noms **TermLists** . Pour finir, fermez la classe **Program** et l’espace de noms **TermLists** .

```csharp
static void Main(string[] args)
{
    using (var client = Clients.NewClient())
    {
        string list_id = CreateTermList(client);

        UpdateTermList(client, list_id, "name", "description");
        AddTerm(client, list_id, "term1");
        AddTerm(client, list_id, "term2");

        GetAllTerms(client, list_id);

        // Always remember to refresh the search index of your list
        RefreshSearchIndex(client, list_id);

        string text = "This text contains the terms \"term1\" and \"term2\".";
        ScreenText(client, list_id, text);

        DeleteTerm(client, list_id, "term1");

        // Always remember to refresh the search index of your list
        RefreshSearchIndex(client, list_id);

        text = "This text contains the terms \"term1\" and \"term2\".";
        ScreenText(client, list_id, text);

        DeleteAllTerms(client, list_id);
        DeleteTermList(client, list_id);

        Console.WriteLine("Press ENTER to close the application.");
        Console.ReadLine();
    }
}
```

## <a name="run-the-application-to-see-the-output"></a>Exécuter l’application pour voir la sortie

La sortie de votre console se présente comme suit :

```console
Creating term list.
Term list created. ID: 252.
Updating information for term list with ID 252.

Adding term "term1" to term list with ID 252.
Adding term "term2" to term list with ID 252.

Getting terms in term list with ID 252.
term1
term2

Refreshing search index for term list with ID 252.

Screening text: "This text contains the terms "term1" and "term2"." using term list with ID 252.
Found term: "term1" from list ID 252 at index 32.
Found term: "term2" from list ID 252 at index 46.

Removed term "term1" from term list with ID 252.

Refreshing search index for term list with ID 252.

Screening text: "This text contains the terms "term1" and "term2"." using term list with ID 252.
Found term: "term2" from list ID 252 at index 46.

Removing all terms from term list with ID 252.
Deleting term list with ID 252.
Press ENTER to close the application.
```

## <a name="next-steps"></a>Étapes suivantes

Obtenez le [SDK Content Moderator pour .NET](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) et la [solution Visual Studio](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/ContentModerator) pour ce guide de démarrage rapide et d’autres guides de démarrage rapide Content Moderator pour .NET, puis commencez votre intégration.

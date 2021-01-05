---
title: Guide pratique pour gérer les écritures simultanées dans les ressources
titleSuffix: Azure Cognitive Search
description: Utilisez l’accès concurrentiel optimiste pour éviter les collisions entre des mises à jour ou des suppressions d’index, d’indexeurs et de sources de données de la Recherche cognitive Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 1cb8d578c05166f88ed7e91681dd6b5f15b1e3e5
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94358641"
---
# <a name="how-to-manage-concurrency-in-azure-cognitive-search"></a>Gestion de l’accès concurrentiel dans la Recherche cognitive Azure

Lors de la gestion de ressources de la Recherche cognitive Azure telles que des index et des sources de données, il est important de mettre à jour les ressources en toute sécurité, surtout si elles sont accessibles simultanément par différents composants de votre application. Lorsque deux clients mettent à jour une ressource en même temps sans coordination, cela peut créer des conditions de concurrence. Pour éviter ce problème, la Recherche cognitive Azure offre un *modèle d’accès concurrentiel optimiste*. Aucun verrou n’est appliqué aux ressources. Au lieu de cela, chaque ressource présente une étiquette d’entité (ETag) qui identifie la version de la ressource afin que vous puissiez élaborer des requêtes sans risquer de remplacements accidentels.

> [!Tip]
> Le code conceptuel de cet [exemple de solution C#](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetETagsExplainer) illustre le fonctionnement du contrôle d’accès concurrentiel dans la Recherche cognitive Azure. Le code crée des conditions qui appellent le contrôle d’accès concurrentiel. La lecture du [fragment de code ci-dessous](#samplecode) est probablement suffisante pour la plupart des développeurs, mais si vous souhaitez l’exécuter, modifiez le fichier appsettings.json pour y ajouter le nom du service et une clé API d’administration. Pour une URL de service `http://myservice.search.windows.net`, le nom du service est `myservice`.

## <a name="how-it-works"></a>Fonctionnement

L’accès concurrentiel optimiste est implémenté via des contrôles de conditions d’accès dans les appels d’API écrivant dans des index, des indexeurs, des sources de données et des ressources synonymMap.

Toutes les ressources présentent une [*étiquette d’entité (ETag)*](https://en.wikipedia.org/wiki/HTTP_ETag) qui fournit des informations sur la version de l’objet. En vérifiant d’abord l’ETag, vous pouvez éviter les mises à jour simultanées dans un flux de travail classique (obtention, modification locale, mise à jour) en vous assurant que l’ETag de la ressource correspond à celui de votre copie locale.

+ L’API REST utilise un [ETag](/rest/api/searchservice/common-http-request-and-response-headers-used-in-azure-search) sur l’en-tête de demande.
+ Le Kit de développement logiciel (SDK) .NET spécifie l’ETag via un objet accessCondition en définissant l’en-tête [If-Match | If-Match-None](/rest/api/searchservice/common-http-request-and-response-headers-used-in-azure-search) sur la ressource. Les objets qui utilisent des ETags, tels que [SynonymMap.ETag](/dotnet/api/azure.search.documents.indexes.models.synonymmap.etag) et [SearchIndex.ETag](/dotnet/api/azure.search.documents.indexes.models.searchindex.etag), ont un objet accessCondition.

À chaque fois que vous mettez à jour une ressource, son ETag change automatiquement. Lorsque vous implémentez la gestion de l’accès concurrentiel, vous placez simplement sur la requête de mise à jour une condition préalable qui exige que la ressource distante présente le même ETag que la copie de la ressource que vous avez modifiée sur le client. Si un processus simultané a déjà modifié la ressource distante, l’ETag ne correspondra pas à la condition préalable et la requête échouera avec l’erreur HTTP 412. Si vous utilisez le Kit de développement logiciel (SDK) .NET, cela se manifeste sous la forme d’une exception `CloudException`, où la méthode d’extension `IsAccessConditionFailed()` renvoie la valeur true.

> [!Note]
> Il n’existe qu’un seul mécanisme pour l’accès concurrentiel. Celui-ci est systématiquement utilisé, peu importe l’API employée pour les mises à jour de ressources.

<a name="samplecode"></a>
## <a name="use-cases-and-sample-code"></a>Cas d’usage et exemple de code

Le code suivant illustre les contrôles accessCondition pour les opérations de mise à jour de clé :

+ Échec de la mise à jour si la ressource n’existe plus
+ Échec de la mise à jour si la version de la ressource change

### <a name="sample-code-from-dotnetetagsexplainer-program"></a>Exemple de code tiré du [programme DotNetETagsExplainer](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetETagsExplainer)

```csharp
    class Program
    {
        // This sample shows how ETags work by performing conditional updates and deletes
        // on an Azure Cognitive Search index.
        static void Main(string[] args)
        {
            IConfigurationBuilder builder = new ConfigurationBuilder().AddJsonFile("appsettings.json");
            IConfigurationRoot configuration = builder.Build();

            SearchServiceClient serviceClient = CreateSearchServiceClient(configuration);

            Console.WriteLine("Deleting index...\n");
            DeleteTestIndexIfExists(serviceClient);

            // Every top-level resource in Azure Cognitive Search has an associated ETag that keeps track of which version
            // of the resource you're working on. When you first create a resource such as an index, its ETag is
            // empty.
            Index index = DefineTestIndex();
            Console.WriteLine(
                $"Test index hasn't been created yet, so its ETag should be blank. ETag: '{index.ETag}'");

            // Once the resource exists in Azure Cognitive Search, its ETag will be populated. Make sure to use the object
            // returned by the SearchServiceClient! Otherwise, you will still have the old object with the
            // blank ETag.
            Console.WriteLine("Creating index...\n");
            index = serviceClient.Indexes.Create(index);
            
            Console.WriteLine($"Test index created; Its ETag should be populated. ETag: '{index.ETag}'");

            // ETags let you do some useful things you couldn't do otherwise. For example, by using an If-Match
            // condition, we can update an index using CreateOrUpdate and be guaranteed that the update will only
            // succeed if the index already exists.
            index.Fields.Add(new Field("name", AnalyzerName.EnMicrosoft));
            index =
                serviceClient.Indexes.CreateOrUpdate(
                    index,
                    accessCondition: AccessCondition.GenerateIfExistsCondition());

            Console.WriteLine(
                $"Test index updated; Its ETag should have changed since it was created. ETag: '{index.ETag}'");

            // More importantly, ETags protect you from concurrent updates to the same resource. If another
            // client tries to update the resource, it will fail as long as all clients are using the right
            // access conditions.
            Index indexForClient1 = index;
            Index indexForClient2 = serviceClient.Indexes.Get("test");

            Console.WriteLine("Simulating concurrent update. To start, both clients see the same ETag.");
            Console.WriteLine($"Client 1 ETag: '{indexForClient1.ETag}' Client 2 ETag: '{indexForClient2.ETag}'");

            // Client 1 successfully updates the index.
            indexForClient1.Fields.Add(new Field("a", DataType.Int32));
            indexForClient1 =
                serviceClient.Indexes.CreateOrUpdate(
                    indexForClient1,
                    accessCondition: AccessCondition.IfNotChanged(indexForClient1));

            Console.WriteLine($"Test index updated by client 1; ETag: '{indexForClient1.ETag}'");

            // Client 2 tries to update the index, but fails, thanks to the ETag check.
            try
            {
                indexForClient2.Fields.Add(new Field("b", DataType.Boolean));
                serviceClient.Indexes.CreateOrUpdate(
                    indexForClient2,
                    accessCondition: AccessCondition.IfNotChanged(indexForClient2));

                Console.WriteLine("Whoops; This shouldn't happen");
                Environment.Exit(1);
            }
            catch (CloudException e) when (e.IsAccessConditionFailed())
            {
                Console.WriteLine("Client 2 failed to update the index, as expected.");
            }

            // You can also use access conditions with Delete operations. For example, you can implement an
            // atomic version of the DeleteTestIndexIfExists method from this sample like this:
            Console.WriteLine("Deleting index...\n");
            serviceClient.Indexes.Delete("test", accessCondition: AccessCondition.GenerateIfExistsCondition());

            // This is slightly better than using the Exists method since it makes only one round trip to
            // Azure Cognitive Search instead of potentially two. It also avoids an extra Delete request in cases where
            // the resource is deleted concurrently, but this doesn't matter much since resource deletion in
            // Azure Cognitive Search is idempotent.

            // And we're done! Bye!
            Console.WriteLine("Complete.  Press any key to end application...\n");
            Console.ReadKey();
        }

        private static SearchServiceClient CreateSearchServiceClient(IConfigurationRoot configuration)
        {
            string searchServiceName = configuration["SearchServiceName"];
            string adminApiKey = configuration["SearchServiceAdminApiKey"];

            SearchServiceClient serviceClient =
                new SearchServiceClient(searchServiceName, new SearchCredentials(adminApiKey));
            return serviceClient;
        }

        private static void DeleteTestIndexIfExists(SearchServiceClient serviceClient)
        {
            if (serviceClient.Indexes.Exists("test"))
            {
                serviceClient.Indexes.Delete("test");
            }
        }

        private static Index DefineTestIndex() =>
            new Index()
            {
                Name = "test",
                Fields = new[] { new Field("id", DataType.String) { IsKey = true } }
            };
    }
}
```

## <a name="design-pattern"></a>Modèle de conception

Un modèle de conception pour l’implémentation de l’accès concurrentiel optimiste doit inclure une boucle qui effectue une nouvelle tentative de contrôle de la condition d’accès, un test de la condition d’accès et récupère éventuellement une ressource mise à jour avant d’essayer de réappliquer les modifications.

Cet extrait de code illustre l’ajout d’une ressource synonymMap à un index existant. Ce code est tiré de l’[exemple C# des synonymes pour la Recherche cognitive Azure](search-synonyms-tutorial-sdk.md).

L’extrait de code obtient l’index « hotel », vérifie la version de l’objet pour une opération de mise à jour, lève une exception si la condition échoue, puis retente l’opération (jusqu’à trois fois), en commençant par extraire l’index du serveur pour obtenir sa dernière version.

```csharp
        private static void EnableSynonymsInHotelsIndexSafely(SearchServiceClient serviceClient)
        {
            int MaxNumTries = 3;

            for (int i = 0; i < MaxNumTries; ++i)
            {
                try
                {
                    Index index = serviceClient.Indexes.Get("hotels");
                    index = AddSynonymMapsToFields(index);

                    // The IfNotChanged condition ensures that the index is updated only if the ETags match.
                    serviceClient.Indexes.CreateOrUpdate(index, accessCondition: AccessCondition.IfNotChanged(index));

                    Console.WriteLine("Updated the index successfully.\n");
                    break;
                }
                catch (CloudException e) when (e.IsAccessConditionFailed())
                {
                    Console.WriteLine($"Index update failed : {e.Message}. Attempt({i}/{MaxNumTries}).\n");
                }
            }
        }
        
        private static Index AddSynonymMapsToFields(Index index)
        {
            index.Fields.First(f => f.Name == "category").SynonymMaps = new[] { "desc-synonymmap" };
            index.Fields.First(f => f.Name == "tags").SynonymMaps = new[] { "desc-synonymmap" };
            return index;
        }
```

## <a name="next-steps"></a>Étapes suivantes

Examinez l’[exemple C# de synonymes](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToSynonyms) pour avoir une illustration de la mise à jour en toute sécurité d’un index existant.

Essayez de modifier l’un des exemples suivants pour inclure des ETags ou des objets AccessCondition.

+ [Exemple d’API REST sur GitHub](https://github.com/Azure-Samples/search-rest-api-getting-started)
+ [Exemple de SDK .NET sur GitHub](https://github.com/Azure-Samples/search-dotnet-getting-started). Cette solution inclut le projet « DotNetEtagsExplainer », qui contient le code présenté dans cet article.

## <a name="see-also"></a>Voir aussi

[En-têtes de demande et de réponse HTTP communément utilisés](/rest/api/searchservice/common-http-request-and-response-headers-used-in-azure-search)
[Codes d’état HTTP](/rest/api/searchservice/http-status-codes)
[Opérations d’index (API REST)](/rest/api/searchservice/index-operations)
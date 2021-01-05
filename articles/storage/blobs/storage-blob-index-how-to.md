---
title: Utiliser des balises d’index de blob pour gérer et rechercher des données sur Stockage Blob Azure
description: Découvrez comment utiliser des balises d’index de blobs pour catégoriser, gérer et interroger afin de découvrir des objets blob.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 11/19/2020
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.reviewer: klaasl
ms.custom: devx-track-csharp
ms.openlocfilehash: 411815ca2f947c47b8dfb0d2e5d61f8ea18f3545
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95541247"
---
# <a name="use-blob-index-tags-preview-to-manage-and-find-data-on-azure-blob-storage"></a>Utiliser des balises d’index de blob (préversion) pour gérer et rechercher des données sur Stockage Blob Azure

Les balises d’index de blob catégorisent les données de votre compte de stockage à l’aide d’attributs de balise clé-valeur. Ces balises sont automatiquement indexées et exposées en tant qu’index multidimensionnel pouvant faire l’objet d’une recherche pour trouver facilement des données. Cet article explique comment définir, obtenir et trouver des données à l’aide de balises d’index d’objets blob.

> [!NOTE]
> L’index de blobs est actuellement disponible en préversion publique dans les régions **Canada Centre**, **Canada Est**, **France Centre** et **France Sud**. Pour en savoir plus sur cette fonctionnalité ainsi que sur les problèmes et limitations connus, consultez [Gérer et rechercher des données Blob Azure avec des balises d’index de blob (préversion)](storage-manage-find-blobs.md).

## <a name="prerequisites"></a>Prérequis

# <a name="portal"></a>[Portail](#tab/azure-portal)

- Un abonnement Azure inscrit et approuvé pour l’accès à la préversion de l’index de blob
- Un accès au [portail Azure](https://portal.azure.com/)

# <a name="net"></a>[.NET](#tab/net)

L’index de blob étant disponible en préversion, le package de stockage .NET est publié dans le flux NuGet de préversion. Cette bibliothèque est susceptible d’être modifiée au cours de la période de préversion.

1. Configurez votre projet Visual Studio pour prendre en main la bibliothèque de client Stockage Blob Azure v12 pour .NET. Pour en savoir plus, consultez [Démarrage rapide .NET](storage-quickstart-blobs-dotnet.md).

2. Dans le Gestionnaire de package NuGet, recherchez le package **Azure.Storage.Blobs**, puis installez la version **12.7.0-preview.1** ou une version ultérieure pour votre projet. Vous pouvez également exécuter la commande PowerShell : `Install-Package Azure.Storage.Blobs -Version 12.7.0-preview.1`.

   Pour savoir comment procéder, consultez [Rechercher et installer un package](/nuget/consume-packages/install-use-packages-visual-studio#find-and-install-a-package).

3. Ajoutez les instructions using suivantes au début de votre fichier de code.

    ```csharp
    using Azure;
    using Azure.Storage.Blobs;
    using Azure.Storage.Blobs.Models;
    using Azure.Storage.Blobs.Specialized;
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    ```

---

## <a name="upload-a-new-blob-with-index-tags"></a>Charger un nouvel objet blob avec des balises d’index

Cette tâche peut être effectuée par un [Propriétaire des données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) ou un principal de sécurité qui a reçu l’autorisation d’accès à l’[opération du fournisseur de ressources Azure](../../role-based-access-control/resource-provider-operations.md#microsoftstorage) `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write` via un rôle Azure personnalisé.

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Dans le [portail Azure](https://portal.azure.com/), sélectionnez votre compte de stockage. 

2. Sous **Service BLOB**, accédez à l’option **Conteneurs**, puis sélectionnez votre conteneur.

3. Sélectionnez le bouton **Charger** et naviguez dans votre système de fichiers local pour rechercher un fichier à charger en tant qu’objet blob de blocs.

4. Développez la liste déroulante **Avancé**, puis accédez à la section **Balises d’index d’objets blob**.

5. Entrez les balises d’index d’objets blob clé/valeur que vous souhaitez appliquer à vos données.

6. Sélectionnez le bouton **Charger** pour charger l’objet blob.

:::image type="content" source="media/storage-blob-index-concepts/blob-index-upload-data-with-tags.png" alt-text="Capture d’écran du portail Azure montrant comment charger un blob avec des balises d’index.":::

# <a name="net"></a>[.NET](#tab/net)

L’exemple suivant montre comment créer un objet blob d’ajout avec des balises définies lors de la création.

```csharp
static async Task BlobIndexTagsOnCreate()
   {
      BlobServiceClient serviceClient = new BlobServiceClient(ConnectionString);
      BlobContainerClient container = serviceClient.GetBlobContainerClient("mycontainer");

      try
      {
          // Create a container
          await container.CreateIfNotExistsAsync();

          // Create an append blob
          AppendBlobClient appendBlobWithTags = container.GetAppendBlobClient("myAppendBlob0.logs");

          // Blob index tags to upload
          AppendBlobCreateOptions appendOptions = new AppendBlobCreateOptions();
          appendOptions.Tags = new Dictionary<string, string>
          {
              { "Sealed", "false" },
              { "Content", "logs" },
              { "Date", "2020-04-20" }
          };

          // Upload data with tags set on creation
          await appendBlobWithTags.CreateAsync(appendOptions);
      }
      finally
      {
      }
   }
```

---

## <a name="get-set-and-update-blob-index-tags"></a>Obtenir, définir et mettre à jour des balises d’index d’objets blob

L’obtention de balises d’index blob peut être réalisée par un [Propriétaire des données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) ou un principal de sécurité qui a reçu l’autorisation d’accès à l’[opération du fournisseur de ressources Azure](../../role-based-access-control/resource-provider-operations.md#microsoftstorage) `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/read` via un rôle Azure personnalisé.

La définition et la mise à jour de balises d’index blob peuvent être effectuées par un [Propriétaire des données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) ou un principal de sécurité qui a reçu l’autorisation d’accès à l’[opération du fournisseur de ressources Azure](../../role-based-access-control/resource-provider-operations.md#microsoftstorage) `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write` via un rôle Azure personnalisé.

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Dans le [portail Azure](https://portal.azure.com/), sélectionnez votre compte de stockage. 

2. Sous **service BLOB**, accédez à l’option **Conteneurs**, puis sélectionnez votre conteneur.

3. Sélectionnez votre blob dans la liste des blobs du conteneur sélectionné.

4. L’onglet Vue d’ensemble de l’objet blob affiche les propriétés de votre objet blob, dont les **balises d’index d’objets blob**.

5. Vous pouvez obtenir, définir, modifier ou supprimer n’importe quelle balise d’index clé/valeur pour votre objet blob.

6. Sélectionnez le bouton **Enregistrer** pour confirmer les mises à jour de votre objet blob.

:::image type="content" source="media/storage-blob-index-concepts/blob-index-get-set-tags.png" alt-text="Capture d’écran du portail Azure montrant comment récupérer, définir, mettre à jour et supprimer des balises d’index sur des blobs.":::

# <a name="net"></a>[.NET](#tab/net)

```csharp
static async Task BlobIndexTagsExample()
   {
      BlobServiceClient serviceClient = new BlobServiceClient(ConnectionString);
      BlobContainerClient container = serviceClient.GetBlobContainerClient("mycontainer");

      try
      {
          // Create a container
          await container.CreateIfNotExistsAsync();

          // Create a new append blob
          AppendBlobClient appendBlob = container.GetAppendBlobClient("myAppendBlob1.logs");
          await appendBlob.CreateAsync();

          // Set or update blob index tags on existing blob
          Dictionary<string, string> tags = new Dictionary<string, string>
          {
              { "Project", "Contoso" },
              { "Status", "Unprocessed" },
              { "Sealed", "true" }
          };
          await appendBlob.SetTagsAsync(tags);

          // Get blob index tags
          Response<IDictionary<string, string>> tagsResponse = await appendBlob.GetTagsAsync();
          Console.WriteLine(appendBlob.Name);
          foreach (KeyValuePair<string, string> tag in tagsResponse.Value)
          {
              Console.WriteLine($"{tag.Key}={tag.Value}");
          }

          // List blobs with all options returned including blob index tags
          await foreach (BlobItem blobItem in container.GetBlobsAsync(BlobTraits.All))
          {
              Console.WriteLine(Environment.NewLine + blobItem.Name);
              foreach (KeyValuePair<string, string> tag in blobItem.Tags)
              {
                  Console.WriteLine($"{tag.Key}={tag.Value}");
              }
          }

          // Delete existing blob index tags by replacing all tags
          Dictionary<string, string> noTags = new Dictionary<string, string>();
          await appendBlob.SetTagsAsync(noTags);

      }
      finally
      {
      }
   }
```

---

## <a name="filter-and-find-data-with-blob-index-tags"></a>Filtrer et rechercher des données avec des balises d’index de blobs

Cette tâche peut être effectuée par un [Propriétaire des données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) ou un principal de sécurité qui a reçu l’autorisation d’accès à l’[opération du fournisseur de ressources Azure](../../role-based-access-control/resource-provider-operations.md#microsoftstorage) `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/filter/action` via un rôle Azure personnalisé.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Dans le portail Azure, le filtre des balises d’index de blobs applique automatiquement le paramètre `@container` pour définir l’étendue de votre conteneur sélectionné. Si vous souhaitez filtrer et rechercher des données balisées dans votre compte de stockage, utilisez nos API REST, nos Kits de développement logiciel (SDK) ou nos outils.

1. Dans le [Portail Azure](https://portal.azure.com/), sélectionnez votre compte de stockage. 

2. Sous **Service BLOB**, accédez à l’option **Conteneurs**, puis sélectionnez votre conteneur.

3. Sélectionnez le bouton **Filtre des balises d’index d’objets blob** pour filtrer à l’intérieur du conteneur sélectionné.

4. Entrez une clé de balise d’index de blobs et une valeur de balise.

5. Sélectionnez le bouton **Filtre des balises d’index d’objets blob** pour ajouter des filtres de balises (jusqu’à 10).

:::image type="content" source="media/storage-blob-index-concepts/blob-index-tag-filter-within-container.png" alt-text="Capture d’écran du portail Azure montrant comment filtrer et rechercher des blobs balisés à l’aide de balises d’index.":::

# <a name="net"></a>[.NET](#tab/net)

```csharp
static async Task FindBlobsByTagsExample()
   {
      BlobServiceClient serviceClient = new BlobServiceClient(ConnectionString);
      BlobContainerClient container1 = serviceClient.GetBlobContainerClient("mycontainer");
      BlobContainerClient container2 = serviceClient.GetBlobContainerClient("mycontainer2");

      // Blob index queries and selection
      String singleEqualityQuery = @"""Archive"" = 'false'";
      String andQuery = @"""Archive"" = 'false' AND ""Priority"" = '01'";
      String rangeQuery = @"""Date"" >= '2020-04-20' AND ""Date"" <= '2020-04-30'";
      String containerScopedQuery = @"@container = 'mycontainer' AND ""Archive"" = 'false'";

      String queryToUse = containerScopedQuery;

      try
      {
          // Create a container
          await container1.CreateIfNotExistsAsync();
          await container2.CreateIfNotExistsAsync();

          // Create append blobs
          AppendBlobClient appendBlobWithTags0 = container1.GetAppendBlobClient("myAppendBlob00.logs");
          AppendBlobClient appendBlobWithTags1 = container1.GetAppendBlobClient("myAppendBlob01.logs");
          AppendBlobClient appendBlobWithTags2 = container1.GetAppendBlobClient("myAppendBlob02.logs");
          AppendBlobClient appendBlobWithTags3 = container2.GetAppendBlobClient("myAppendBlob03.logs");
          AppendBlobClient appendBlobWithTags4 = container2.GetAppendBlobClient("myAppendBlob04.logs");
          AppendBlobClient appendBlobWithTags5 = container2.GetAppendBlobClient("myAppendBlob05.logs");
           
          // Blob index tags to upload
          CreateAppendBlobOptions appendOptions = new CreateAppendBlobOptions();
          appendOptions.Tags = new Dictionary<string, string>
          {
              { "Archive", "false" },
              { "Priority", "01" },
              { "Date", "2020-04-20" }
          };
          
          CreateAppendBlobOptions appendOptions2 = new CreateAppendBlobOptions();
          appendOptions2.Tags = new Dictionary<string, string>
          {
              { "Archive", "true" },
              { "Priority", "02" },
              { "Date", "2020-04-24" }
          };

          // Upload data with tags set on creation
          await appendBlobWithTags0.CreateAsync(appendOptions);
          await appendBlobWithTags1.CreateAsync(appendOptions);
          await appendBlobWithTags2.CreateAsync(appendOptions2);
          await appendBlobWithTags3.CreateAsync(appendOptions);
          await appendBlobWithTags4.CreateAsync(appendOptions2);
          await appendBlobWithTags5.CreateAsync(appendOptions2);

          // Find Blobs given a tags query
          Console.WriteLine("Find Blob by Tags query: " + queryToUse + Environment.NewLine);

          List<TaggedBlobItem> blobs = new List<TaggedBlobItem>();
          await foreach (TaggedBlobItem taggedBlobItem in serviceClient.FindBlobsByTagsAsync(queryToUse))
          {
              blobs.Add(taggedBlobItem);
          }

          foreach (var filteredBlob in blobs)
          {
              Console.WriteLine($"BlobIndex result: ContainerName= {filteredBlob.ContainerName}, " +
                  $"BlobName= {filteredBlob.Name}");
          }

      }
      finally
      {
      }
   }
```

---

## <a name="lifecycle-management-with-blob-index-tag-filters"></a>Gestion du cycle de vie avec des filtres de balises d’index d’objets blob

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Dans le [Portail Azure](https://portal.azure.com/), sélectionnez votre compte de stockage. 

2. Sous **Service blob**, accédez à l’option **Gestion du cycle de vie**.

3. Sélectionnez *Ajouter une règle*, puis complétez les champs du formulaire Ensemble d’actions.

4. Sélectionnez **Ensemble de filtres** pour ajouter un filtre facultatif pour la correspondance des préfixes et des index de blob.

  :::image type="content" source="media/storage-blob-index-concepts/blob-index-match-lifecycle-filter-set.png" alt-text="Capture d’écran du portail Azure montrant comment ajouter des balises d’index pour la gestion de cycle de vie.":::

5. Sélectionnez **Vérifier + ajouter** pour passer en revue les paramètres de règle.

  :::image type="content" source="media/storage-blob-index-concepts/blob-index-lifecycle-management-example.png" alt-text="Capture d’écran du portail Azure montrant une règle de gestion de cycle de vie avec des exemples de filtre de balises d’index de blob":::

6. Sélectionnez **Ajouter** pour appliquer la nouvelle règle à la stratégie de gestion du cycle de vie.

# <a name="net"></a>[.NET](#tab/net)

Des stratégies de [gestion du cycle de vie](storage-lifecycle-management-concepts.md) sont appliquées pour chaque compte de stockage au niveau du plan de contrôle. Pour .NET, installez la [bibliothèque de stockage de gestion Microsoft Azure](https://www.nuget.org/packages/Microsoft.Azure.Management.Storage/) version 16.0.0 ou ultérieure.

---

## <a name="next-steps"></a>Étapes suivantes

 - Pour en savoir plus sur les balises d’index de blob, consultez [Gérer et rechercher des données Blob Azure avec des balises d’index de blob (préversion)](storage-manage-find-blobs.md ).
 - Pour en savoir plus sur la gestion de cycle de vie, consultez [Gérer le cycle de vie de Stockage Blob Azure](storage-lifecycle-management-concepts.md).
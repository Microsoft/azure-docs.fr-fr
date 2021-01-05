---
title: Vue d’ensemble des objets blob de pages Azure | Microsoft Docs
description: Une vue d’ensemble des objets blobs de pages Azure et de leurs avantages, comportant des cas d’utilisation avec des exemples de scripts.
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 06/15/2020
ms.author: tamram
ms.reviewer: wielriac
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: aada418b4f74c38a2a35c793deb85b94b703fb89
ms.sourcegitcommit: 8c3a656f82aa6f9c2792a27b02bbaa634786f42d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97629355"
---
# <a name="overview-of-azure-page-blobs"></a>Vue d’ensemble des objets blob de pages Azure

Stockage Azure offre trois types de stockage d’objets blob : Objets blob de blocs, d’ajout et de pages. Les objets blob de blocs sont constitués de blocs et sont parfaitement adaptés pour stocker des fichiers texte ou binaires, et pour charger efficacement des fichiers volumineux. Les objets blob d’ajouts sont également constitués de blocs, mais ils sont optimisés pour les opérations d’ajout, ce qui les rend idéaux pour les scénarios de journalisation. Les objets blob de pages sont constitués de pages jusqu’à 512 octets, jusqu’à 8 To de taille totale et sont conçus pour les opérations de lecture/écriture aléatoires fréquentes. Les objets blob de pages constituent la base des disques IaaS Azure. Cet article se concentre sur la description des fonctionnalités et des avantages des objets blob de pages.

Les objets blob de pages représentent une collection de pages de 512 octets, qui offrent la possibilité de lire/écrire des plages arbitraires d’octets. Les objets blob de pages sont donc particulièrement adaptés au stockage de structures de données basées sur des index et diverses comme les disques de système d’exploitation et de données pour machines virtuelles et bases de données. Par exemple, Azure SQL DB utilise des objets blob de pages comme stockage permanent sous-jacent pour ses bases de données. Les objets blob de pages sont également souvent utilisés pour les fichiers avec des mises à jour basées sur une plage.  

Les principales fonctionnalités des objets blob de pages Azure résident dans son interface REST, la durabilité du stockage sous-jacent et les fonctionnalités de migration parfaite vers Azure. Ces fonctionnalités sont abordées plus en détail dans la section suivante. De plus, les objets blob de pages Azure sont actuellement pris en charge sur deux types de stockage : Stockage Premium et Standard. Stockage Premium est plus particulièrement conçu pour les charges de travail nécessitant de hautes performances et une faible latence, ce qui rend les objets blob de pages premium idéaux pour des scénarios de stockage hautes performances. Les comptes de stockage Standard sont plus économiques pour l’exécution de charges de travail insensibles à la latence.

## <a name="restrictions"></a>Restrictions

Les objets blob de pages ne peuvent utiliser que le niveau d’accès **chaud**, et ne peuvent donc pas utiliser les niveaux **froid** et **archive**. Pour plus d’informations sur les niveaux d’accès, consultez [Niveaux d’accès pour Stockage Blob Azure : chaud, froid et archive](storage-blob-storage-tiers.md).

## <a name="sample-use-cases"></a>Exemples de cas d’utilisation

Étudions quelques cas d’utilisation des objets blob de pages avec des disques IaaS Azure. Les objets blob de pages Azure constituent la base de la plateforme de disques virtuels pour IaaS Azure. Le système d’exploitation Azure et les disques de données sont implémentés en tant que disques virtuels dans lesquels les données sont conservées durablement dans la plateforme Stockage Azure, puis remises aux machines virtuelles pour des performances optimales. Les disques Azure sont conservés au [format VHD](/previous-versions/windows/it-pro/windows-7/dd979539(v=ws.10)) Hyper-V et stockés en tant qu’[objet blob de pages](/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs#about-page-blobs) dans Stockage Azure. En plus des disques virtuels pour machines virtuelles IaaS Azure, les objets blob de pages permettent également des scénarios PaaS et DBaaS tels que le service Azure SQL DB qui utilise actuellement des objets blob de pages pour stocker des données SQL, permettant ainsi des opérations de lecture/écriture aléatoires rapides pour la base de données. Dans un autre exemple, si vous disposez d’un service PaaS pour accéder aux fichiers multimédias partagés d’applications collaboratives d’édition vidéo, les objets blob de pages permettent un accès rapide à des emplacements aléatoires dans le média. Cela permet également à plusieurs utilisateurs de modifier et de fusionner rapidement et efficacement un même média. 

Des services Microsoft internes comme Azure Site Recovery et Azure Backup, ainsi que de nombreux développeurs tiers, ont implémenté des innovations de pointe à l’aide de l’interface REST des objets blob de pages. Voici quelques-uns des scénarios uniques implémentés sur Azure : 

* Gestion des captures instantanées incrémentielle orientée application : Les applications peuvent exploiter les instantanés d’objet blob de pages et les API REST pour enregistrer les points de contrôle d’application sans duplication coûteuse de données. Stockage Azure prend en charge les instantanés locaux pour les objets blob de pages, qui ne nécessitent pas la copie de l’intégralité de l’objet blob. Ces API d’instantané publiques permettent également d’accéder à et de copier des deltas entre deux instantanés.
* Migration dynamique d’application et de données locales vers le cloud : Copiez les données locales et utilisez les API REST pour écrire directement dans l’objet blob de pages Azure pendant l’exécution de la machine virtuelle locale. Lorsque la cible est interceptée, vous pouvez basculer rapidement vers la machine virtuelle Azure à l’aide de ces données. De cette façon, vous pouvez migrer vos machines virtuelles et disques virtuels locaux vers le cloud avec un temps d’arrêt minime, car la migration des données se produit en arrière-plan pendant que vous utilisez la machine virtuelle. Le temps d’arrêt nécessaire pour le basculement est donc relativement court (quelques minutes).
* Accès partagé [basé sur SAS](../common/storage-sas-overview.md), qui permet des scénarios tels que plusieurs lecteurs et un auteur unique avec prise en charge du contrôle d’accès concurrentiel.

## <a name="pricing"></a>Tarifs

Les deux types de stockage proposés avec des objets blob de pages ont leur propre modèle de tarification. Les blob de pages Premium suivent le modèle de tarification des disques managés, tandis que les objets blob de pages standard sont facturés en fonction de la taille utilisée et du nombre de transactions. Pour plus d’informations, consultez la [page de tarification d’objet blob de pages Azure](https://azure.microsoft.com/pricing/details/storage/page-blobs/).

## <a name="page-blob-features"></a>Fonctionnalités d’objet blob de pages

### <a name="rest-api"></a>API REST

Consultez le document suivant pour commencer à [développer à l’aide d’objets blob de pages](./storage-quickstart-blobs-dotnet.md). Par exemple, vous pouvez voir comment accéder aux objets blob de pages à l’aide de la bibliothèque de client de stockage pour .NET. 

Le diagramme suivant décrit les relations globales entre le compte, les conteneurs et les objets blob de pages.

![Capture d’écran montrant les relations entre le compte, les conteneurs et les objets blob de pages](./media/storage-blob-pageblob-overview/storage-blob-pageblob-overview-figure1.png)

#### <a name="creating-an-empty-page-blob-of-a-specified-size"></a>Création d’un objet blob de pages vide d’une taille spécifique

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Commencez par obtenir une référence à un conteneur. Pour créer un objet blob de pages, appelez la méthode GetPageBlobClient, puis la méthode [PageBlobClient.Create](/dotnet/api/azure.storage.blobs.specialized.pageblobclient.create). Transmettez la taille maximale de l’objet blob à créer. Cette taille doit être un multiple de 512 octets.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_CreatePageBlob":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Pour créer un objet blob de pages, nous créons tout d’abord un objet **CloudBlobClient**, avec l’URI de base pour accéder au stockage blob de votre compte de stockage (*pbaccount* dans la figure 1), ainsi que l’objet **StorageCredentialsAccountAndKey**, comme dans l’exemple ci-dessous. L’exemple montre ensuite la création d’une référence à un objet **CloudBlobContainer**, puis la création du conteneur (*testvhds*) s’il n’existe pas déjà. Ensuite, à l’aide de l’objet **CloudBlobContainer**, nous pouvons créer une référence à un objet **CloudPageBlob** en spécifiant le nom de l’objet blob de pages (os4.vhd) auquel nous voulons accéder. Pour créer l’objet blob de pages, appelez [CloudPageBlob.Create](/dotnet/api/microsoft.azure.storage.blob.cloudpageblob.create) en indiquant la taille maximale de l’objet blob à créer. *blobSize* doit être un multiple de 512 octets.

```csharp
using Microsoft.Azure;
using Microsoft.Azure.Storage;
using Microsoft.Azure.Storage.Blob;

long OneGigabyteAsBytes = 1024 * 1024 * 1024;
// Retrieve storage account from connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the blob client.
CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

// Retrieve a reference to a container.
CloudBlobContainer container = blobClient.GetContainerReference("testvhds");

// Create the container if it doesn't already exist.
container.CreateIfNotExists();

CloudPageBlob pageBlob = container.GetPageBlobReference("os4.vhd");
pageBlob.Create(16 * OneGigabyteAsBytes);
```

---

#### <a name="resizing-a-page-blob"></a>Redimensionnement d’un objet blob de pages

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Pour redimensionner un objet blob de pages après sa création, utilisez la méthode [Redimensionner](/dotnet/api/azure.storage.blobs.specialized.pageblobclient.resize). La taille demandée doit être un multiple de 512 octets.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_ResizePageBlob":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Pour redimensionner un objet blob de pages après sa création, utilisez la méthode [Redimensionner](/dotnet/api/microsoft.azure.storage.blob.cloudpageblob.resize). La taille demandée doit être un multiple de 512 octets.

```csharp
pageBlob.Resize(32 * OneGigabyteAsBytes);
```

---

#### <a name="writing-pages-to-a-page-blob"></a>Écriture de pages sur un objet blob de pages

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Pour écrire des pages, utilisez la méthode [PageBlobClient.UploadPages](/dotnet/api/azure.storage.blobs.specialized.pageblobclient.uploadpages).  

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_WriteToPageBlob":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Pour écrire des pages, utilisez la méthode [CloudPageBlob.WritePages](/dotnet/api/microsoft.azure.storage.blob.cloudpageblob.beginwritepages).  

```csharp
pageBlob.WritePages(dataStream, startingOffset); 
```

---

Cela vous permet d’écrire un ensemble séquentiel de pages jusqu’à 4 Mo. Le décalage écrit doit commencer sur une limite de 512 octets (startingOffset % 512 == 0) et se terminer sur une limite de 512 - 1. 

Dès qu’une requête d’écriture pour un ensemble séquentiel de pages réussit dans le service blob et est répliquée à des fins de durabilité et de résilience, l’écriture est validée et sa réussite est signalée au client.  

Le diagramme ci-dessous montre 2 opérations d’écriture distinctes :

![Diagramme montrant les deux options d’écriture distinctes.](./media/storage-blob-pageblob-overview/storage-blob-pageblob-overview-figure2.png)

1.  Une opération d’écriture commençant au décalage 0 de 1 024 octets de longueur 
2.  Une opération d’écriture commençant au décalage 4096 d’une longueur de 1 024 

#### <a name="reading-pages-from-a-page-blob"></a>Lecture de pages d’un objet blob de pages

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Pour lire des pages, utilisez la méthode [PageBlobClient.Download](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.download) pour lire une plage d’octets à partir de l’objet blob de pages. 

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_ReadFromPageBlob":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Pour lire des pages, utilisez la méthode [CloudPageBlob.DownloadRangeToByteArray](/dotnet/api/microsoft.azure.storage.blob.icloudblob.downloadrangetobytearray) pour lire une plage d’octets de l’objet blob de pages. 

```csharp
byte[] buffer = new byte[rangeSize];
pageBlob.DownloadRangeToByteArray(buffer, bufferOffset, pageBlobOffset, rangeSize); 
```

---

Cela vous permet de télécharger le blob complet ou une plage d’octets commençant à un décalage quelconque dans le blob. Lors de la lecture, le décalage ne doit pas nécessairement commencer sur un multiple de 512. Lors de la lecture d’octets d’une page NUL, le service retourne zéro octet.

La figure suivante montre une opération de lecture avec un décalage de 256 et une taille de plage de 4352. Les données retournées sont mises en surbrillance en orange. Des zéros sont retournés pour les pages NUL.

![Diagramme montrant une opération de lecture avec un décalage de 256 et une taille de plage de 4 352.](./media/storage-blob-pageblob-overview/storage-blob-pageblob-overview-figure3.png)

Si vous disposez d’un objet blob peu rempli, vous pouvez vous contenter de télécharger les régions de page valides pour éviter les coûts d’entrée de zéro octet et pour réduire la latence du téléchargement.  

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Pour déterminer les pages adossées à des données, utilisez [CloudPageBlob.GetPageRanges](/dotnet/api/azure.storage.blobs.specialized.pageblobclient.getpageranges). Vous pouvez alors énumérer les plages retournées et télécharger les données dans chaque plage. 

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_ReadValidPageRegionsFromPageBlob":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Pour déterminer les pages contenant des données, utilisez [CloudPageBlob.GetPageRanges](/dotnet/api/microsoft.azure.storage.blob.cloudpageblob.getpageranges). Vous pouvez alors énumérer les plages retournées et télécharger les données dans chaque plage. 

```csharp
IEnumerable<PageRange> pageRanges = pageBlob.GetPageRanges();

foreach (PageRange range in pageRanges)
{
    // Calculate the range size
    int rangeSize = (int)(range.EndOffset + 1 - range.StartOffset);

    byte[] buffer = new byte[rangeSize];

    // Read from the correct starting offset in the page blob and
    // place the data in the bufferOffset of the buffer byte array
    pageBlob.DownloadRangeToByteArray(buffer, bufferOffset, range.StartOffset, rangeSize); 

    // Then use the buffer for the page range just read
}
```

---

#### <a name="leasing-a-page-blob"></a>Location d’un objet blob de pages

L’opération de blob de bail établit et gère un verrou sur un blob pour les opérations d’écriture et de suppression. Cette opération est utile dans les scénarios où un objet blob de pages est accessible à partir de plusieurs clients, pour faire en sorte qu’un seul client à la fois puisse écrire dans l’objet blob. Les disques Azure, par exemple, utilisent ce mécanisme de leasing pour garantir que le disque n’est géré que par une seule machine virtuelle. La durée du verrou peut être de 15 à 60 secondes, ou peut être infinie. Consultez la documentation [ici](/rest/api/storageservices/lease-blob) pour plus de détails.

En plus des API REST riches en fonctionnalités, les objets blob de pages offrent également un accès partagé, une durabilité et une sécurité renforcée. Nous aborderons ces avantages plus en détails dans les paragraphes suivants. 

### <a name="concurrent-access"></a>Accès simultané

L’API REST des objets blob de pages et son mécanisme de bail permettent aux applications d’accéder à l’objet blob de pages à partir de plusieurs clients. Par exemple, supposons que vous devez créer un service cloud distribué qui partage des objets de stockage avec plusieurs utilisateurs. Il pourrait s’agir d’une application web proposant une collection d’images importante à plusieurs utilisateurs. Une option d’implémentation consiste à utiliser une machine virtuelle avec des disques attachés. Ceci implique des inconvénients, notamment (i) la contrainte qu’un disque ne peut être attaché qu’à une seule machine virtuelle, limitant ainsi l’évolutivité, la flexibilité et augmentant les risques. En cas de problème lié à la machine virtuelle ou au service exécuté sur la machine virtuelle, et ensuite dû au bail, l’image est inaccessible jusqu’à ce que le bail expire ou qu’il soit interrompu ; et (ii) le coût supplémentaire lié à une machine virtuelle IaaS. 

Une autre option consiste à utiliser les objets blob de pages directement via les API REST de Stockage Azure. Cette option supprime la nécessité de machines virtuelles IaaS coûteuses, offre la flexibilité maximale d’accès direct à partir de plusieurs clients, simplifie le modèle de déploiement classique en supprimant la nécessité de joindre et de détacher des disques, et élimine le risque de problèmes sur la machine virtuelle. Elle offre également le même niveau de performances pour les opérations de lecture/écriture aléatoires qu’un disque

### <a name="durability-and-high-availability"></a>Durabilité et haute disponibilité

Les stockages Standard et Premium constituent un stockage durable où les données d’objet blob de pages sont toujours répliquées pour garantir la durabilité et la haute disponibilité. Pour plus d’informations sur la redondance du Stockage Azure, consultez cette [documentation](../common/storage-redundancy.md). Azure a fourni de façon cohérente une durabilité de classe Entreprise pour les disques IaaS et les objets blob de pages, avec un [taux de défaillance annuel](https://en.wikipedia.org/wiki/Annualized_failure_rate) inégalé dans le secteur de zéro pour cent.

### <a name="seamless-migration-to-azure"></a>Migration parfaite vers Azure

Pour les clients et développeurs qui souhaitent implémenter leur propre solution de sauvegarde personnalisée, Azure propose également des instantanés incrémentiels ne contenant que les deltas. Cette fonctionnalité permet d’éviter le coût de la copie initiale complète, ce qui réduit considérablement le coût de sauvegarde. Avec la possibilité de lire et de copier efficacement des données différentielles, il s’agit d’une autre fonctionnalité puissante qui permet encore plus d’innovations plus les développeurs, offrant ainsi la meilleure expérience de sauvegarde et de reprise après sinistre sur Azure. Vous pouvez configurer votre propre solution de sauvegarde ou de reprise après sinistre pour vos machines virtuelles sur Azure à l’aide d’un [instantané d’objet blob](/rest/api/storageservices/snapshot-blob), de l’API [Get Page Ranges](/rest/api/storageservices/get-page-ranges) et de l’API [Incremental Copy Blob](/rest/api/storageservices/incremental-copy-blob), que vous pouvez utiliser pour copier facilement les données incrémentielles pour la reprise après sinistre. 

Nombre d’entreprises disposent en outre des charges de travail critiques déjà en cours d’exécution dans les centres de données locaux. Pour migrer la charge de travail vers le cloud, un des principales préoccupations serait liée au temps d’arrêt nécessaire pour copier les données et au risque de problèmes imprévus après le basculement. Dans de nombreux cas, le temps d’arrêt peut être un frein à la migration vers le cloud. Grâce à l’API REST des objets blob de pages, Azure résout ce problème en permettant une migration vers le cloud avec une interruption minimale des charges de travail critiques. 

Pour obtenir des exemples sur la prise d’un instantané et la restauration d’un objet blob de pages à partir d’un instantané, reportez-vous à l’article relatif à la [configuration d’un processus de sauvegarde à l’aide d’instantanés incrémentiels](../../virtual-machines/windows/incremental-snapshots.md).
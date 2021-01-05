---
title: Gérer l’expiration du stockage Blob Azure
titleSuffix: Azure Content Delivery Network
description: Découvrez les options de contrôle de la durée de vie des objets blob dans la mise en cache Azure CDN.
services: cdn
documentationcenter: ''
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: ad4801e9-d09a-49bf-b35c-efdc4e6034e8
ms.service: azure-cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: how-to
ms.date: 02/1/2018
ms.author: mazha
ms.openlocfilehash: 206ff6f888229356743bebb816cf03e4f7a7504b
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92778710"
---
# <a name="manage-expiration-of-azure-blob-storage-in-azure-cdn"></a>Gérer l’expiration du stockage Blob Azure dans Azure CDN
> [!div class="op_single_selector"]
> * [Contenu web Azure](cdn-manage-expiration-of-cloud-service-content.md)
> * [stockage d’objets blob Azure](cdn-manage-expiration-of-blob-content.md)
> 
> 

Le [service Stockage Blob](../storage/common/storage-introduction.md#blob-storage) dans Stockage Azure fait partie des différentes origines Azure intégrées à Azure Content Delivery Network (CDN). Tout contenu d’objet BLOB publiquement accessible peut être mis en cache dans Azure CDN jusqu’à l’expiration de sa durée de vie. La durée de vie est déterminée par l’en-tête `Cache-Control`dans la réponse HTTP du serveur d’origine. Cet article décrit plusieurs façons de définir l’en-tête `Cache-Control` d’un blob dans Stockage Azure.

Vous pouvez également contrôler les paramètres de cache à partir du portail Azure, en définissant les règles de mise en cache CDN. Si vous créez une règle de mise en cache, puis définissez son comportement sur **Remplacer** ou **Ignorer le cache** , les paramètres de mise en cache fournis à l’origine décrits dans cet article sont ignorés. Pour plus d’informations sur les concepts généraux de mise en cache, consultez la section [Fonctionnement de la mise en cache](cdn-how-caching-works.md).

> [!TIP]
> Vous pouvez choisir de ne pas définir de durée de vie sur un objet blob. Dans ce cas, Azure CDN applique automatiquement une durée de vie par défaut de sept jours, sauf si vous avez configuré des règles de mise en cache dans le portail Azure. Cette valeur TTL par défaut s’applique uniquement aux optimisations de remise web générales. Pour optimiser les fichiers volumineux, la durée de vie par défaut est de 1 jour, et pour les optimisations du streaming multimédia, la durée de vie par défaut est de 1 an.
> 
> Pour plus d’informations sur la façon dont Azure CDN accélère l’accès aux fichiers et à d’autres ressources, consultez [Vue d’ensemble d’Azure Content Delivery Network](cdn-overview.md).
> 
> Pour plus d’informations sur le stockage Blob Azure, consultez [Présentation du Stockage Blob](../storage/blobs/storage-blobs-introduction.md).
 

## <a name="setting-cache-control-headers-by-using-cdn-caching-rules"></a>Définition d’en-têtes Cache-Control à l’aide de règles de mise en cache CDN
La méthode recommandée pour définir l’en-tête `Cache-Control` d’un objet blob consiste à utiliser des règles de mise en cache dans le portail Azure. Pour plus d’informations sur les règles de mise en cache CDN, consultez [Contrôler le comportement de mise en cache d’Azure Content Delivery Network au moyen de règles de mise en cache](cdn-caching-rules.md).

> [!NOTE] 
> Les règles de mise en cache sont disponibles uniquement pour les profils **CDN Azure Standard fourni par Verizon** et **CDN Azure Standard fourni par Akamai** . Pour les profils **CDN Azure Premium fourni par Verizon** , vous devez utiliser le [moteur de règles du CDN Azure](./cdn-verizon-premium-rules-engine.md) dans le portail **Gérer** pour une fonctionnalité similaire.

**Pour accéder à la page des règles de mise en cache CDN** :

1. Dans le portail Azure, sélectionnez un profil CDN, puis sélectionnez un point de terminaison pour l’objet blob.

2. Dans le volet gauche, sous Paramètres, sélectionnez **Règles de mise en cache** .

   ![Bouton Règles de mise en cache CDN](./media/cdn-manage-expiration-of-blob-content/cdn-caching-rules-btn.png)

   La page **Règles de mise en cache** s’affiche.

   ![Page de mise en cache du CDN](./media/cdn-manage-expiration-of-blob-content/cdn-caching-page.png)


**Pour définir les en-têtes Cache-Control du service de stockage d’objets blob à l’aide de règles de mise en cache générales :**

1. Sous **Règles de mise en cache générales** , définissez **Comportement de mise en cache des chaînes de requête** sur **Ignorer les chaînes de requête** , puis définissez **Comportement de mise en cache** sur **Remplacer** .
      
2. Pour **Durée d’expiration du cache** , entrez 3 600 dans la zone **Secondes** ou 1 dans la zone **Heures** . 

   ![Exemple de règles de mise en cache générales CDN](./media/cdn-manage-expiration-of-blob-content/cdn-global-caching-rules-example.png)

   Cette règle de mise en cache générale définit une durée de mise en cache d’une heure et affecte toutes les requêtes au point de terminaison. Elle se substitue à tout en-tête HTTP `Cache-Control` ou `Expires` qui sont envoyés par le serveur d’origine spécifié par le point de terminaison.   

3. Sélectionnez **Enregistrer** .
 
**Pour définir les en-têtes Cache-Control d’un fichier d’objets blob à l’aide de règles de mise en cache personnalisées :**

1. Sous **Règles de mise en cache personnalisées** , créez deux conditions de correspondance :

     R. Pour la première condition de correspondance, définissez **Condition de correspondance** sur **Chemin** , puis entrez `/blobcontainer1/*` pour **Valeur(s) de correspondance** . Définissez **Comportement de mise en cache** sur **Remplacer** , puis entrez 4 dans le champ **Heures** .

    B. Pour la deuxième condition de correspondance, définissez **Condition de correspondance** sur **Chemin** , puis entrez `/blobcontainer1/blob1.txt` pour **Valeur(s) de correspondance** . Définissez **Comportement de mise en cache** sur **Remplacer** , puis entrez 2 dans le champ **Heures** .

    ![Exemple de règles de mise en cache personnalisées CDN](./media/cdn-manage-expiration-of-blob-content/cdn-custom-caching-rules-example.png)

    La première règle de mise en cache personnalisée définit une durée de mise en cache de quatre heures pour tous les fichiers d’objets blob du dossier `/blobcontainer1` présent sur le serveur d’origine spécifié par votre point de terminaison. La deuxième règle remplace la première règle pour le fichier d’objets blob `blob1.txt` uniquement, et définit une durée de mise en cache de deux heures pour celui-ci.

2. Sélectionnez **Enregistrer** .


## <a name="setting-cache-control-headers-by-using-azure-powershell"></a>Définition d’en-têtes Cache-Control à l’aide de PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[Azure PowerShell](/powershell/azure/) est l’un des moyens les plus rapides et puissants de gérer vos services Azure. Utilisez l’applet de commande `Get-AzStorageBlob` pour obtenir une référence à l’objet BLOB, puis définissez la propriété `.ICloudBlob.Properties.CacheControl`. 

Par exemple :

```powershell
# Create a storage context
$context = New-AzStorageContext -StorageAccountName "<storage account name>" -StorageAccountKey "<storage account key>"

# Get a reference to the blob
$blob = Get-AzStorageBlob -Context $context -Container "<container name>" -Blob "<blob name>"

# Set the CacheControl property to expire in 1 hour (3600 seconds)
$blob.ICloudBlob.Properties.CacheControl = "max-age=3600"

# Send the update to the cloud
$blob.ICloudBlob.SetProperties()
```

> [!TIP]
> Vous pouvez également utiliser PowerShell pour [gérer vos profils et points de terminaison CDN](cdn-manage-powershell.md).
> 
>

## <a name="setting-cache-control-headers-by-using-net"></a>Définition d’en-têtes Cache-Control à l’aide de .NET
Pour spécifier l’en-tête `Cache-Control` d’un objet blob à l’aide de code .NET, utilisez la [bibliothèque cliente Stockage Azure pour .NET](../storage/blobs/storage-quickstart-blobs-dotnet.md) pour définir la propriété [CloudBlob.Properties.CacheControl](/dotnet/api/microsoft.azure.storage.blob.blobproperties.cachecontrol).

Par exemple :

```csharp
class Program
{
    const string connectionString = "<storage connection string>";
    static void Main()
    {
        // Retrieve storage account information from connection string
        CloudStorageAccount storageAccount = CloudStorageAccount.Parse(connectionString);

        // Create a blob client for interacting with the blob service.
        CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

        // Create a reference to the container
        CloudBlobContainer <container name> = blobClient.GetContainerReference("<container name>");

        // Create a reference to the blob
        CloudBlob <blob name> = container.GetBlobReference("<blob name>");

        // Set the CacheControl property to expire in 1 hour (3600 seconds)
        blob.Properties.CacheControl = "max-age=3600";

        // Update the blob's properties in the cloud
        blob.SetProperties();
    }
}
```

> [!TIP]
> D’autres exemples de code .NET sont disponibles dans les [exemples Stockage Blob Azure pour .NET](https://azure.microsoft.com/documentation/samples/storage-blob-dotnet-getting-started/).
> 

## <a name="setting-cache-control-headers-by-using-other-methods"></a>Définition d’en-têtes Cache-Control à l’aide d’autres méthodes

### <a name="azure-storage-explorer"></a>Explorateur de stockage Azure
Avec l’ [Explorateur Stockage Microsoft Azure](https://azure.microsoft.com/features/storage-explorer/), vous pouvez afficher et modifier vos ressources de stockage blob, y compris les propriétés telles que *CacheControl* . 

Pour mettre à jour la propriété *CacheControl* d’un objet blob avec l’Explorateur Stockage Azure :
   1. Sélectionnez un objet blob, puis sélectionnez **Propriétés** dans le menu contextuel. 
   2. Faites défiler jusqu’à la propriété *CacheControl* .
   3. Entrez une valeur, puis sélectionnez **Enregistrer** .


![Propriétés de l’Explorateur Stockage Azure](./media/cdn-manage-expiration-of-blob-content/cdn-storage-explorer-properties.png)

### <a name="azure-command-line-interface"></a>Interface de ligne de commande Azure
Avec l’[interface de ligne de commande Azure](/cli/azure) (CLI), vous pouvez gérer des ressources blob Azure à partir de la ligne de commande. Pour définir l’en-tête cache-control lorsque vous téléchargez un objet blob avec l’interface de ligne de commande Azure, définissez la propriété *cacheControl* à l’aide du `-p` commutateur. L’exemple suivant montre comment définir la durée de vie sur 1 heure (3 600 secondes) :
  
```azurecli
azure storage blob upload -c <connectionstring> -p cacheControl="max-age=3600" .\<blob name> <container name> <blob name>
```

### <a name="azure-storage-services-rest-api"></a>API REST des services Stockage Azure
Vous pouvez utiliser l’ [API REST des services Stockage Azure](/rest/api/storageservices/) pour définir explicitement la propriété *x-ms-blob-cache-control* en utilisant les opérations suivantes sur une demande :
  
   - [Put Blob](/rest/api/storageservices/Put-Blob)
   - [Put Block List](/rest/api/storageservices/Put-Block-List)
   - [Set Blob Properties](/rest/api/storageservices/Set-Blob-Properties)

## <a name="testing-the-cache-control-header"></a>Test de l’en-tête Cache-Control
Vous pouvez facilement vérifier la durée de vie de vos objets blob. Avec les [outils de développement](https://developer.microsoft.com/microsoft-edge/platform/documentation/f12-devtools-guide/) de votre navigateur, vérifiez que votre objet blob comprend l’en-tête de réponse `Cache-Control`. Vous pouvez également utiliser un outil tel que [Wget](https://www.gnu.org/software/wget/), [Postman](https://www.getpostman.com/) ou [Fiddler](https://www.telerik.com/fiddler) pour examiner les en-têtes de réponse.

## <a name="next-steps"></a>Étapes suivantes
* [Comment gérer l’expiration des contenus de service cloud dans Azure CDN](cdn-manage-expiration-of-cloud-service-content.md)
* [En savoir plus sur les concepts de mise en cache](cdn-how-caching-works.md)
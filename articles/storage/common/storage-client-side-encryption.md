---
title: Chiffrement côté client avec .NET pour Microsoft Azure Storage | Microsoft Docs
description: La bibliothèque cliente de stockage Azure pour .NET prend en charge le chiffrement côté client et l’intégration au coffre de clés Azure pour assurer une sécurité maximale à vos applications Azure Storage.
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 11/10/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-csharp
ms.openlocfilehash: 5f2d3ba12fa65beb7156e056c23e44b028cbb520
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94445062"
---
# <a name="client-side-encryption-and-azure-key-vault-for-microsoft-azure-storage"></a>Chiffrement côté client et Azure Key Vault pour Microsoft Azure Storage
[!INCLUDE [storage-selector-client-side-encryption-include](../../../includes/storage-selector-client-side-encryption-include.md)]

## <a name="overview"></a>Vue d’ensemble
La [bibliothèque cliente Stockage Azure pour .NET](/dotnet/api/overview/azure/storage) prend en charge le chiffrement des données au sein des applications clientes avant le chargement vers Stockage Azure, ainsi que le déchiffrement des données pendant leur téléchargement vers le client. La bibliothèque prend également en charge l’intégration au [coffre de clés Azure](https://azure.microsoft.com/services/key-vault/) pour la gestion des clés de compte de stockage.

Pour obtenir une procédure pas à pas sur le processus de chiffrement des objets Blob à l’aide du chiffrement côté client et d’Azure Key Vault, consultez [Chiffrement et déchiffrement d’objets blob dans Microsoft Azure Storage à l'aide d'Azure Key Vault](../blobs/storage-encrypt-decrypt-blobs-key-vault.md).

Pour le chiffrement côté client avec Java, consultez la page [Chiffrement côté client avec Java pour Microsoft Azure Storage](storage-client-side-encryption-java.md).

## <a name="encryption-and-decryption-via-the-envelope-technique"></a>Chiffrement et déchiffrement via la technique d’enveloppe
Les processus de chiffrement et de déchiffrement s’effectuent selon la technique d’enveloppe.

### <a name="encryption-via-the-envelope-technique"></a>Chiffrement via la technique d’enveloppe
Le chiffrement via la technique d’enveloppe fonctionne de la façon suivante :

1. La bibliothèque cliente du stockage Azure génère une clé de chiffrement de contenu (CEK) qui est une clé symétrique à usage unique.
2. Les données utilisateur sont chiffrées à l'aide de cette clé de chiffrement de contenu.
3. La clé de chiffrement de contenu est ensuite encapsulée (chiffrée) à l’aide de la clé de chiffrement de clés (KEK). La clé de chiffrement de clés est identifiée par un identificateur de clé et peut être une paire de clés asymétriques ou une clé symétrique pouvant être gérée localement ou stockée dans des coffres de clés Azure.
   
    La bibliothèque cliente du stockage n’a jamais accès à la clé de chiffrement de clés. Elle appelle l’algorithme d’encapsulage de clés fourni par Key Vault. Si besoin est, les utilisateurs peuvent choisir d'utiliser des fournisseurs personnalisés pour l’encapsulage/le désencapsulage de clés.

4. Les données chiffrées sont ensuite téléchargées sur le service Azure Storage. La clé encapsulée avec des métadonnées de chiffrement supplémentaires est stockée en tant que métadonnées (sur un objet blob) ou interpolée avec les données chiffrées (messages de la file d’attente et entités de la table).

### <a name="decryption-via-the-envelope-technique"></a>Déchiffrement via la technique d’enveloppe
Le déchiffrement via la technique d’enveloppe fonctionne de la façon suivante :

1. La bibliothèque cliente suppose que l’utilisateur gère la clé de chiffrement à clé (KEK) localement ou dans des coffres de clés Azure. L’utilisateur n’est pas obligé de savoir quelle clé a été spécifiquement utilisée pour le chiffrement. Il est en effet possible d’utiliser et de configurer un programme de résolution de clés qui résout les différents identificateurs de clés.
2. La bibliothèque cliente télécharge les données chiffrées ainsi que  tout le  matériel de chiffrement stocké sur le service.
3. La clé de chiffrement de contenu (CEK) encapsulée est ensuite désencapsulée (déchiffrée) à l’aide de la clé de chiffrement de clés (KEK). Là encore, la bibliothèque cliente n’a pas accès à la clé de chiffrement de clés. Elle appelle simplement l’algorithme de désencapsulage personnalisée ou du fournisseur Key Vault.
4. La clé de chiffrement de contenu (CEK) est alors utilisée pour déchiffrer les données utilisateur chiffrées.

## <a name="encryption-mechanism"></a>Mécanisme de chiffrement
La bibliothèque cliente du stockage utilise [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) pour chiffrer les données utilisateur. Plus précisément, le mode [CBC (Cipher Block Chaining)](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Cipher-block_chaining_.28CBC.29) avec AES. Chaque service fonctionnant un peu différemment, nous allons les étudier un par un ici.

### <a name="blobs"></a>Objets blob
La bibliothèque cliente prend actuellement en charge le chiffrement des objets blob entiers uniquement. Les téléchargements complets et de plages sont tous deux pris en charge.

Au cours du chiffrement, la bibliothèque cliente génère un vecteur d’initialisation aléatoire (IV) de 16 octets avec une clé de chiffrement de contenu (CEK) aléatoire de 32 octets, puis effectue le chiffrement d’enveloppe des données d’objets blob à l’aide de ces informations. La clé de chiffrement de contenu encapsulée ainsi que des métadonnées de chiffrement supplémentaires sont ensuite stockées en tant que métadonnées d’objet blob en même temps que l’objet blob chiffré sur le service.

> [!WARNING]
> Si vous modifiez ou téléchargez vos propres métadonnées pour l’objet blob, vous devez vous assurer que ces métadonnées sont conservées. Si vous téléchargez de nouvelles métadonnées sans ces métadonnées, la clé de chiffrement de contenu encapsulée, le vecteur d’initialisation et d’autres métadonnées seront perdus et le contenu de l’objet blob ne sera plus jamais récupérable.
> 
> 

Lors du téléchargement d’un objet blob entier, la clé de chiffrement de contenu encapsulée est désencapsulée et utilisée en combinaison avec le vecteur d’initialisation (stocké en tant que métadonnées d’objet blob dans cet exemple) pour renvoyer les données déchiffrées aux utilisateurs.

Le téléchargement d’une plage arbitraire dans l’objet blob chiffré implique d’ajuster la plage fournie par les utilisateurs pour obtenir une petite quantité de données supplémentaires susceptibles d’être utilisées pour déchiffrer la plage demandée.

Tous les types d’objets blob (objets blob de blocs, objets blob de pages et objets blob d’ajouts) peuvent être chiffrés/déchiffrés à l’aide de ce schéma.

### <a name="queues"></a>Files d’attente
Dans la mesure où les messages de la file d’attente peuvent avoir n’importe quel format, la bibliothèque cliente définit un format personnalisé qui inclut le vecteur d’initialisation (IV) et la clé de chiffrement de contenu (CEK) chiffrée dans le texte du message.

Au cours du chiffrement, la bibliothèque cliente génère un vecteur d’initialisation aléatoire de 16 octets avec une clé de chiffrement de contenu aléatoire de 32 octets, puis effectue le chiffrement d’enveloppe du texte du message de la file d’attente à l’aide de ces informations. La clé de chiffrement de contenu encapsulée et les métadonnées de chiffrement supplémentaires sont ensuite ajoutées au message chiffré de la file d’attente. Ce message modifié (illustré ci-dessous) est stocké sur le service.

```xml
<MessageText>{"EncryptedMessageContents":"6kOu8Rq1C3+M1QO4alKLmWthWXSmHV3mEfxBAgP9QGTU++MKn2uPq3t2UjF1DO6w","EncryptionData":{…}}</MessageText>
```

Au cours du déchiffrement, la clé encapsulée est extraite du message de la file d’attente et désencapsulée. Le vecteur d’initialisation est également extrait du message de la file d’attente et utilisé en combinaison avec la clé désencapsulée pour déchiffrer les données du message de la file d’attente. Notez que les métadonnées de chiffrement sont peu volumineuses (moins de 500 octets). Donc, même si elles entrent en compte dans la limite de 64 Ko du message de la file d’attente, leur impact reste facile à gérer. Notez que le message chiffré est encodé en base64, comme illustré dans l’extrait de code ci-dessus, ce qui augmente également la taille du message envoyé.

### <a name="tables"></a>Tables
> [!NOTE]
> Le service de table est pris en charge dans la bibliothèque de client stockage Azure jusqu’à la version 9.x uniquement.
> 
> 

La bibliothèque cliente prend en charge le chiffrement des propriétés de l’entité pour les opérations d’insertion et de remplacement.

> [!NOTE]
> La fusion n’est pas prise en charge pour le moment. Si un sous-ensemble de propriétés a été chiffré précédemment à l’aide d’une clé différente, la fusion des nouvelles propriétés et la mise à jour des métadonnées entraîne une perte de données. L’opération de fusion nécessite d’effectuer des appels de service supplémentaires pour lire l’entité pré-existante à partir du service ou d’utiliser une nouvelle clé par propriété. Ces deux solutions ne conviennent pas pour des raisons de performances.
> 
> 

Le chiffrement des données d’une table fonctionne de la manière suivante :  

1. Les utilisateurs spécifient les propriétés à chiffrer.
2. La bibliothèque cliente génère un vecteur d’initialisation (IV) aléatoire de 16 octets et une clé de chiffrement de contenu (CEK) aléatoire de 32 octets pour chaque entité, puis effectue le chiffrement d’enveloppe sur les propriétés individuelles à chiffrer en dérivant un nouveau vecteur d’initialisation par propriété. La propriété chiffrée est stockée en tant que données binaires.
3. La clé de chiffrement de contenu encapsulée et certaines métadonnées de chiffrement supplémentaires sont ensuite stockées sous la forme de deux propriétés réservées supplémentaires. La première propriété réservée (_ClientEncryptionMetadata1) est une propriété de type chaîne qui conserve les informations sur le vecteur d’initialisation, la version et la clé encapsulée. La seconde propriété réservée (_ClientEncryptionMetadata2) est une propriété de type binaire qui conserve les informations sur les propriétés chiffrées. Les informations contenues dans cette seconde propriété (_ClientEncryptionMetadata2) sont elles-mêmes chiffrées.
4. En raison de ces propriétés réservées supplémentaires requises pour le chiffrement, les utilisateurs ne peuvent désormais avoir que 250 propriétés personnalisées au lieu de 252. La taille totale de l’entité doit être inférieure à 1 Mo.

Notez que seules les propriétés de type chaîne peuvent être chiffrées. Si d’autres types de propriétés doivent être chiffrés, ils doivent être convertis en chaînes. Les chaînes chiffrées sont stockées sur le service en tant que propriétés binaires, et elles sont converties en chaînes après le déchiffrement.

Pour les tables, outre la stratégie de chiffrement, les utilisateurs doivent spécifier les propriétés à chiffrer. Pour ce faire, il faut spécifier un attribut EncryptProperty \(pour les entités POCO qui dérivent de TableEntity) ou un programme de résolution de chiffrement dans les options de requête. Un programme de résolution de chiffrement est un délégué qui prend une clé de partition, une clé de ligne et un nom de propriété, puis renvoie une valeur booléenne indiquant si cette propriété doit être chiffrée. Au cours du chiffrement, la bibliothèque cliente utilise ces informations pour décider si une propriété doit être chiffrée lors de l’écriture en ligne. Le délégué fournit également la possibilité de définir la manière dont les propriétés sont chiffrées l’aide d’un programme logique. (Par exemple, si X, alors chiffrer la propriété A ; sinon chiffrer les propriétés A et B.) Notez qu’il n’est pas nécessaire de fournir ces informations lors de la lecture ou de l’interrogation des entités.

### <a name="batch-operations"></a>Opérations de traitement par lots
Dans les opérations de traitement par lots, la même clé de chiffrement de clés (KEK) est utilisée pour toutes les lignes d’une même opération, car la bibliothèque cliente n’accepte qu’un seul objet d’options (et par conséquent, une seule stratégie/clé de chiffrement de clés) par opération de traitement par lots. Toutefois, la bibliothèque cliente génère en interne un nouveau vecteur d’initialisation (IV) aléatoire et une clé de chiffrement de contenu (CEK) aléatoire par ligne dans le lot. Les utilisateurs peuvent également choisir de chiffrer différentes propriétés pour chaque opération dans le lot en définissant ce comportement dans le programme de résolution de chiffrement.

### <a name="queries"></a>Requêtes
> [!NOTE]
> Comme les entités sont chiffrées, vous ne pouvez pas exécuter des requêtes qui filtrent sur une propriété chiffrée.  Si vous essayez, les résultats seront incorrects, car le service essaiera de comparer les données chiffrées aux données non chiffrées.
> 
> 
> Pour effectuer des opérations de requête, vous devez spécifier un programme de résolution de clé capable de résoudre toutes les clés dans le jeu de résultats. Si une entité contenue dans le résultat de la requête ne peut pas être résolue par rapport à un fournisseur, la bibliothèque cliente génère une erreur. Pour toute requête effectuant des projections côté serveur, la bibliothèque cliente ajoute par défaut les propriétés de métadonnées de chiffrement spéciales (_ClientEncryptionMetadata1 et _ClientEncryptionMetadata2) aux colonnes sélectionnées.

## <a name="azure-key-vault"></a>Azure Key Vault
Azure Key Vault permet de protéger les clés de chiffrement et les secrets utilisés par les services et les applications cloud. En utilisant Azure Key Vault, les utilisateurs peuvent chiffrer les clés et secrets (tels que les clés d’authentification, les clés de compte de stockage, les clés de chiffrement de données, les fichiers .PFX et les mots de passe) à l’aide de clés protégées par des modules de sécurité matériels (HSM). Pour plus d’informations, consultez la page [Qu’est-ce qu’Azure Key Vault ?](../../key-vault/general/overview.md)

La bibliothèque cliente de stockage utilise les interfaces Key Vault dans la bibliothèque principale afin de fournir une infrastructure commune de gestion des clés sur Azure. Les utilisateurs peuvent tirer parti des bibliothèques Key Vault pour bénéficier de tous les avantages supplémentaires qu’ils fournissent, par exemple des fonctionnalités utiles sur des fournisseurs de clés de cloud et locaux symétriques/RSA simples et transparents, ainsi que sur l’agrégation et la mise en cache.

### <a name="interface-and-dependencies"></a>Interfaces et dépendances

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Il existe deux packages nécessaires pour l’intégration du coffre de clés :

* Azure.Core contient les interfaces `IKeyEncryptionKey` et `IKeyEncryptionKeyResolver`. La bibliothèque cliente de stockage pour .NET le définit déjà en tant que dépendance.
* Azure.Security.KeyVault.Keys (v4. x) contient le client REST de coffre de clés, ainsi que les clients de chiffrement utilisés avec le chiffrement côté client.

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Il existe trois packages de coffre de clés :

* Microsoft.Azure.KeyVault.Core contient IKey et IKeyResolver. Il s’agit d’un petit package sans dépendances. La bibliothèque cliente de stockage pour .NET le définit en tant que dépendance.
* Microsoft.Azure.KeyVault (v3.x) contient le client REST du coffre de clés.
* Microsoft.Azure.KeyVault.Extensions (v3.x) contient le code d’extension qui inclut des implémentations d’algorithmes de chiffrement, RSAKey et SymmetricKey. Il repose sur les espaces de noms Core et KeyVault, et fournit une fonctionnalité permettant de définir un programme de résolution d’agrégation (lorsque les utilisateurs veulent utiliser plusieurs fournisseurs de clés) et un programme de résolution de clé de mise en cache. Bien que la bibliothèque cliente de stockage ne dépende pas directement de ce package, si les utilisateurs veulent utiliser Azure Key Vault pour stocker leurs clés ou utiliser les extensions du coffre de clés pour recourir aux fournisseurs de chiffrement en local et dans le cloud, ils ont besoin de ce package.

Vous trouverez plus d’informations concernant l’utilisation de Key Vault dans la v11 dans les [exemples de code de chiffrement dans la v11](https://github.com/Azure/azure-storage-net/tree/master/Samples/GettingStarted/EncryptionSamples).

---

Le coffre de clés est conçu pour les clés principales de valeur élevée et les seuils de limitation par coffre de clés sont définies avec cela à l’esprit. Lors du chiffrement côté client avec le coffre de clés, il est préférable d’utiliser les clés principales Symmetric stockées en tant que secrets dans le coffre de clés et mises en cache localement. Les utilisateurs doivent procéder comme suit :

1. Créer un secret hors connexion et le télécharger dans le coffre de clés.
2. Utiliser l’identificateur de base du secret comme paramètre pour résoudre la version actuelle du secret pour le chiffrement et mettre en cache ces informations localement. Utiliser CachingKeyResolver pour la mise en cache ; les utilisateurs ne doivent pas implémenter leur propre programme logique de mise en cache.
3. Utiliser le programme de résolution de mise en cache en tant qu’entrée lors de la création de la stratégie de chiffrement.

## <a name="best-practices"></a>Meilleures pratiques
La prise en charge du chiffrement est disponible uniquement dans la bibliothèque cliente de stockage pour .NET. Windows Phone et Windows Runtime ne prennent pas en charge le chiffrement pour le moment.

> [!IMPORTANT]
> Tenez compte des points importants suivants quand vous utilisez le chiffrement côté client :
> 
> * Pendant la lecture d’un objet blob chiffré ou l’écriture dans un objet blob chiffré, utilisez les commandes de chargement d’objets entiers et de téléchargement d’objets blob entiers/par plage. N’écrivez pas dans un objet blob chiffré à l’aide d’opérations de protocole telles que Put Block, Put Block List, Write Pages, Clear Pages ou Append Block au risque d’endommager l’objet blob chiffré et de le rendre illisible.
> * Pour les tables, une contrainte similaire existe. Veillez à ne pas mettre à jour les propriétés chiffrées sans aussi mettre à jour les métadonnées de chiffrement.
> * Si vous définissez des métadonnées sur l’objet blob chiffré, vous risquez de remplacer les métadonnées relatives au chiffrement et nécessaires au déchiffrement, car la définition des métadonnées n’est pas additive. Cela est également vrai pour les instantanés : évitez de spécifier des métadonnées lors de la création d’un instantané d’objet blob chiffré. Si des métadonnées doivent être définies, veillez à appeler la méthode **FetchAttributes** pour obtenir les métadonnées de chiffrement actuelles et éviter des écritures simultanées pendant la définition des métadonnées.
> * Activez la propriété **RequireEncryption** dans les options de requête par défaut pour les utilisateurs qui doivent recourir uniquement à des données chiffrées. Pour plus d’informations, consultez la section ci-dessous.
>
>

## <a name="client-api--interface"></a>API/Interface cliente
Les utilisateurs peuvent fournir uniquement une clé, uniquement un programme de résolution ou les deux. Les clés sont identifiées à l’aide d’un identificateur de clé et fournissent le programme logique pour l’encapsulage/le désencapsulage. Les programmes de résolution sont utilisés pour résoudre une clé pendant le processus de déchiffrement. Il définit une méthode de résolution qui renvoie une clé avec un identificateur de clé. Les utilisateurs ont ainsi la possibilité de choisir entre plusieurs clés gérées dans plusieurs emplacements.

* Pour le chiffrement, la clé est toujours utilisée et l’absence de clé entraîne une erreur.
* Pour le déchiffrement :
  * Si la clé est spécifiée et que son identificateur correspond à l’identificateur de clé nécessaire, la clé est utilisée pour le déchiffrement. Dans le cas contraire, le programme de résolution est tenté. S’il n’existe aucun programme de résolution pour cette tentative, une erreur est générée.
  * S’il est spécifié, le programme de résolution de clé est appelé pour obtenir la clé. Si le programme de résolution est spécifié, mais ne comporte pas de mappage pour l’identificateur de clé, une erreur est générée.

### <a name="requireencryption-mode-v11-only"></a>Mode RequireEncryption (v11 uniquement)
Les utilisateurs peuvent éventuellement activer un mode de fonctionnement dans lequel tous les chargements et téléchargements doivent être chiffrés. Dans ce mode, les tentatives de chargement de données sans une stratégie de chiffrement ou de téléchargement de données non chiffrées sur le service échouent sur le client. La propriété **RequireEncryption** de l’objet d’options de requête contrôle ce comportement. Si votre application chiffre tous les objets stockés dans Azure Storage, vous pouvez définir la propriété **RequireEncryption** sur les options de requête par défaut pour l’objet client de service. Par exemple, définissez **CloudBlobClient.DefaultRequestOptions.RequireEncryption** sur **true** pour imposer le chiffrement pour toutes les opérations d’objet blob effectuées par le biais de cet objet client.


### <a name="blob-service-encryption"></a>Chiffrement du service BLOB


# <a name="net-v12"></a>[.NET v12](#tab/dotnet)
Créez un objet **ClientSideEncryptionOptions** et définissez-le sur la création du client à l’aide de **SpecializedBlobClientOptions**. Vous ne pouvez pas définir des options de chiffrement par API. Tout le reste est géré par la bibliothèque cliente en interne.

```csharp
// Your key and key resolver instances, either through KeyVault SDK or an external implementation
IKeyEncryptionKey key;
IKeyEncryptionKeyResolver keyResolver;

// Create the encryption options to be used for upload and download.
ClientSideEncryptionOptions encryptionOptions = new ClientSideEncryptionOptions(ClientSideEncryptionVersion.V1_0)
{
   KeyEncryptionKey = key,
   KeyResolver = keyResolver,
   // string the storage client will use when calling IKeyEncryptionKey.WrapKey()
   KeyWrapAlgorithm = "some algorithm name"
};

// Set the encryption options on the client options
BlobClientOptions options = new SpecializedBlobClientOptions() { ClientSideEncryption = encryptionOptions };

// Get your blob client with client-side encryption enabled.
// Client-side encryption options are passed from service to container clients, and container to blob clients.
// Attempting to construct a BlockBlobClient, PageBlobClient, or AppendBlobClient from a BlobContainerClient
// with client-side encryption options present will throw, as this functionality is only supported with BlobClient.
BlobClient blob = new BlobServiceClient(connectionString, options).GetBlobContainerClient("myContainer").GetBlobClient("myBlob");

// Upload the encrypted contents to the blob.
blob.Upload(stream);

// Download and decrypt the encrypted contents from the blob.
MemoryStream outputStream = new MemoryStream();
blob.DownloadTo(outputStream);
```

Un **BlobServiceClient** n’est pas nécessaire pour appliquer des options de chiffrement. Elles peuvent également être passées dans les constructeurs **BlobContainerClient**/**BlobClient** qui acceptent les objets **BlobClientOptions**.

Si un objet **BlobClient** souhaité existe déjà mais sans options de chiffrement côté client, il existe une méthode d’extension pour créer une copie de cet objet avec le **ClientSideEncryptionOptions** donné. Cette méthode d’extension évite la surcharge liée à la construction d’un nouvel objet **BlobClient** à partir de zéro.

```csharp
using Azure.Storage.Blobs.Specialized;

// Your existing BlobClient instance and encryption options
BlobClient plaintextBlob;
ClientSideEncryptionOptions encryptionOptions;

// Get a copy of plaintextBlob that uses client-side encryption
BlobClient clientSideEncryptionBlob = plaintextBlob.WithClientSideEncryptionOptions(encryptionOptions);
```

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)
Créez un objet **BlobEncryptionPolicy** et définissez-le dans les options de requête (par API ou au niveau client à l’aide de **DefaultRequestOptions** ). Tout le reste est géré par la bibliothèque cliente en interne.

```csharp
// Create the IKey used for encryption.
RsaKey key = new RsaKey("private:key1" /* key identifier */);

// Create the encryption policy to be used for upload and download.
BlobEncryptionPolicy policy = new BlobEncryptionPolicy(key, null);

// Set the encryption policy on the request options.
BlobRequestOptions options = new BlobRequestOptions() { EncryptionPolicy = policy };

// Upload the encrypted contents to the blob.
blob.UploadFromStream(stream, size, null, options, null);

// Download and decrypt the encrypted contents from the blob.
MemoryStream outputStream = new MemoryStream();
blob.DownloadToStream(outputStream, null, options, null);
```

---

### <a name="queue-service-encryption"></a>Chiffrement du service de File d’attente
# <a name="net-v12"></a>[.NET v12](#tab/dotnet)
Créez un objet **ClientSideEncryptionOptions** et définissez-le sur la création du client à l’aide de **SpecializedQueueClientOptions**. Vous ne pouvez pas définir des options de chiffrement par API. Tout le reste est géré par la bibliothèque cliente en interne.

```csharp
// Your key and key resolver instances, either through KeyVault SDK or an external implementation
IKeyEncryptionKey key;
IKeyEncryptionKeyResolver keyResolver;

// Create the encryption options to be used for upload and download.
ClientSideEncryptionOptions encryptionOptions = new ClientSideEncryptionOptions(ClientSideEncryptionVersion.V1_0)
{
   KeyEncryptionKey = key,
   KeyResolver = keyResolver,
   // string the storage client will use when calling IKeyEncryptionKey.WrapKey()
   KeyWrapAlgorithm = "some algorithm name"
};

// Set the encryption options on the client options
QueueClientOptions options = new SpecializedQueueClientOptions() { ClientSideEncryption = encryptionOptions };

// Get your queue client with client-side encryption enabled.
// Client-side encryption options are passed from service to queue clients.
QueueClient queue = new QueueServiceClient(connectionString, options).GetQueueClient("myQueue");

// Send an encrypted queue message.
queue.SendMessage("Hello, World!");

// Download queue messages, decrypting ones that are detected to be encrypted
QueueMessage[] queue.ReceiveMessages(); 
```

Un **QueueServiceClient** n’est pas nécessaire pour appliquer des options de chiffrement. Elles peuvent également être passées dans les constructeurs **QueueClient** qui acceptent les objets **QueueClientOptions**.

Si un objet **QueueClient** souhaité existe déjà mais sans options de chiffrement côté client, il existe une méthode d’extension pour créer une copie de cet objet avec le **ClientSideEncryptionOptions** donné. Cette méthode d’extension évite la surcharge liée à la construction d’un nouvel objet **QueueClient** à partir de zéro.

```csharp
using Azure.Storage.Queues.Specialized;

// Your existing QueueClient instance and encryption options
QueueClient plaintextQueue;
ClientSideEncryptionOptions encryptionOptions;

// Get a copy of plaintextQueue that uses client-side encryption
QueueClient clientSideEncryptionQueue = plaintextQueue.WithClientSideEncryptionOptions(encryptionOptions);
```

Certains utilisateurs peuvent avoir des files d’attente où tous les messages reçus ne peuvent pas être déchiffrés correctement et la clé ou le programme de résolution doit lever une exception. La dernière ligne de l’exemple ci-dessus lèvera dans ce cas, et aucun des messages reçus ne sera accessible. Dans ces scénarios, la sous-classe **QueueClientSideEncryptionOptions** peut être utilisée pour fournir des options de chiffrement aux clients. Il expose un événement **DecryptionFailed** qui se déclenche chaque fois qu’un message de file d’attente ne peut pas être déchiffré, tant qu’au moins un appel a été ajouté à l’événement. Les messages ayant échoué individuellement peuvent être traités de cette manière, et ils sont filtrés à partir du dernier **QueueMessage []** retourné par **ReceiveMessages**.

```csharp
// Create your encryption options using the sub-class.
QueueClientSideEncryptionOptions encryptionOptions = new QueueClientSideEncryptionOptions(ClientSideEncryptionVersion.V1_0)
{
   KeyEncryptionKey = key,
   KeyResolver = keyResolver,
   // string the storage client will use when calling IKeyEncryptionKey.WrapKey()
   KeyWrapAlgorithm = "some algorithm name"
};

// Add a handler to the DecryptionFailed event.
encryptionOptions.DecryptionFailed += (source, args) => {
   QueueMessage failedMessage = (QueueMessage)source;
   Exception exceptionThrown = args.Exception;
   // do something
};

// Use these options with your client objects.
QueueClient queue = new QueueClient(connectionString, queueName, new SpecializedQueueClientOptions()
{
   ClientSideEncryption = encryptionOptions
});

// Retrieve 5 messages from the queue.
// Assume 5 messages come back and one throws during decryption.
QueueMessage[] messages = queue.ReceiveMessages(maxMessages: 5).Value;
Debug.Assert(messages.Length == 4)
```

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)
Créez un objet **QueueEncryptionPolicy** et définissez-le dans les options de requête (par API ou au niveau client à l’aide de **DefaultRequestOptions** ). Tout le reste est géré par la bibliothèque cliente en interne.

```csharp
// Create the IKey used for encryption.
 RsaKey key = new RsaKey("private:key1" /* key identifier */);

 // Create the encryption policy to be used for upload and download.
 QueueEncryptionPolicy policy = new QueueEncryptionPolicy(key, null);

 // Add message
 QueueRequestOptions options = new QueueRequestOptions() { EncryptionPolicy = policy };
 queue.AddMessage(message, null, null, options, null);

 // Retrieve message
 CloudQueueMessage retrMessage = queue.GetMessage(null, options, null);
```

---

### <a name="table-service-encryption-v11-only"></a>Chiffrement du service de Table (v11 uniquement)
En plus de créer une stratégie de chiffrement et de la définir dans les options de requête, vous devez spécifier un **EncryptionResolver** dans **TableRequestOptions** ou définir l’attribut [EncryptProperty] sur l’entité.

#### <a name="using-the-resolver"></a>Utilisation du programme de résolution

```csharp
// Create the IKey used for encryption.
 RsaKey key = new RsaKey("private:key1" /* key identifier */);

 // Create the encryption policy to be used for upload and download.
 TableEncryptionPolicy policy = new TableEncryptionPolicy(key, null);

 TableRequestOptions options = new TableRequestOptions()
 {
    EncryptionResolver = (pk, rk, propName) =>
     {
        if (propName == "foo")
         {
            return true;
         }
         return false;
     },
     EncryptionPolicy = policy
 };

 // Insert Entity
 currentTable.Execute(TableOperation.Insert(ent), options, null);

 // Retrieve Entity
 // No need to specify an encryption resolver for retrieve
 TableRequestOptions retrieveOptions = new TableRequestOptions()
 {
    EncryptionPolicy = policy
 };

 TableOperation operation = TableOperation.Retrieve(ent.PartitionKey, ent.RowKey);
 TableResult result = currentTable.Execute(operation, retrieveOptions, null);
```

#### <a name="using-attributes"></a>Utilisation des attributs
Comme mentionné ci-dessus, si l’entité implémente TableEntity, les propriétés peuvent être décorées avec l’attribut [EncryptProperty] au lieu de spécifier un **EncryptionResolver**.

```csharp
[EncryptProperty]
 public string EncryptedProperty1 { get; set; }
```

## <a name="encryption-and-performance"></a>Chiffrement et performances
Notez que le chiffrement de vos données de stockage affecte les performances. La clé de contenu et le vecteur d’initialisation doivent être générés, le contenu proprement dit doit être chiffré et des métadonnées supplémentaires doivent être mises en forme et téléchargées. Cette surcharge varie selon la quantité de données chiffrées. Nous recommandons de tester systématiquement les performances des applications au cours du développement.

## <a name="next-steps"></a>Étapes suivantes
* [Didacticiel : Chiffrement et déchiffrement d’objets blob dans Microsoft Azure Storage à l'aide d'Azure Key Vault](../blobs/storage-encrypt-decrypt-blobs-key-vault.md)
* Télécharger la [Bibliothèque cliente Azure Storage pour le package NuGet .NET](https://www.nuget.org/packages/WindowsAzure.Storage)
* Télécharger les packages NuGet [Core](https://www.nuget.org/packages/Microsoft.Azure.KeyVault.Core/), [Client](https://www.nuget.org/packages/Microsoft.Azure.KeyVault/) et [Extensions](https://www.nuget.org/packages/Microsoft.Azure.KeyVault.Extensions/) d’Azure Key Vault  
* Consulter la [documentation d’Azure Key Vault](../../key-vault/general/overview.md)

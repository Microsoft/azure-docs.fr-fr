---
title: Tutoriel - Chiffrer et déchiffrer des objets blob avec Azure Key Vault
titleSuffix: Azure Storage
description: Découvrez comment chiffrer et déchiffrer un objet blob en utilisant le chiffrement côté client avec Azure Key Vault.
services: storage
author: tamram
ms.service: storage
ms.topic: tutorial
ms.date: 12/04/2019
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: ddc9dbf77c04ea95e5b873c45de4c0df109514c7
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95544443"
---
# <a name="tutorial---encrypt-and-decrypt-blobs-using-azure-key-vault"></a>Tutoriel - Chiffrer et déchiffrer des objets blob avec Azure Key Vault

Ce didacticiel décrit comment utiliser le chiffrement de stockage côté client avec Azure Key Vault. Il vous explique comment chiffrer et déchiffrer un objet blob dans une application console à l'aide de ces technologies.

**Durée estimée :** 20 minutes

Pour plus d’informations générales sur Azure Key Vault, consultez la page [Présentation d’Azure Key Vault](../../key-vault/general/overview.md).

Pour plus d’informations générales sur le chiffrement côté client du Stockage Azure, consultez [Chiffrement côté client et Azure Key Vault pour Microsoft Azure Storage](../common/storage-client-side-encryption.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, vous devez disposer des éléments suivants :

* Un compte Azure Storage
* Visual Studio 2013 ou une version ultérieure
* Azure PowerShell

## <a name="overview-of-client-side-encryption"></a>Vue d’ensemble du chiffrement côté client

Pour une vue d’ensemble du chiffrement côté client du Stockage Azure, consultez [Chiffrement côté client et Azure Key Vault pour Microsoft Azure Storage](../common/storage-client-side-encryption.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)

Voici une brève description du fonctionnement du chiffrement côté client :

1. Le Kit de développement logiciel (SDK) client d’Azure Storage génère une clé de chiffrement de contenu (CEK) qui est une clé symétrique à usage unique.
2. Les données du client sont chiffrées à l’aide de cette clé de chiffrement de contenu.
3. La clé de chiffrement de contenu est ensuite encapsulée (chiffrée) à l’aide de la clé de chiffrement de clés (KEK). La clé de chiffrement de clés est identifiée par un identificateur de clé et peut être une paire de clés asymétriques ou une clé symétrique pouvant être gérée localement ou stockée dans Azure Key Vault. Le client Storage n’a jamais accès à la clé de chiffrement de clés. Il appelle simplement l'algorithme d’encapsulage de clés fourni par Key Vault. Si besoin est, les clients peuvent choisir d’utiliser des fournisseurs personnalisés pour l’encapsulage/le désencapsulage de clés.
4. Les données chiffrées sont ensuite téléchargées sur le service Azure Storage.

## <a name="set-up-your-azure-key-vault"></a>Configurer votre coffre de clés Azure

Pour continuer avec ce didacticiel, vous devez effectuer les étapes suivantes qui sont décrites dans le didacticiel [Démarrage rapide : définir et récupérer un secret depuis Azure Key Vault à l’aide d’une application web .NET](../../key-vault/secrets/quick-create-net.md) :

* Création d’un coffre de clés
* Ajout d’une clé ou d’un secret au coffre de clés
* Inscrivez une application auprès d’Azure Active Directory.
* Autorisation de l’application à utiliser la clé ou le secret

Notez les valeurs ClientID et ClientSecret qui ont été générées lors de l'inscription d'une application avec Azure Active Directory.

Créez les deux clés dans le coffre de clés. Pour la suite du didacticiel, nous partons du principe que vous avez utilisé les noms suivants : ContosoKeyVault et TestRSAKey1.

## <a name="create-a-console-application-with-packages-and-appsettings"></a>Création d’une application console avec les packages et AppSettings

Dans Visual Studio, créez une application console.

Ajoutez les packages nuget nécessaires dans la Console du gestionnaire de package.

```powershell
Install-Package Microsoft.Azure.ConfigurationManager
Install-Package Microsoft.Azure.Storage.Common
Install-Package Microsoft.Azure.Storage.Blob
Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory

Install-Package Microsoft.Azure.KeyVault
Install-Package Microsoft.Azure.KeyVault.Extensions
```

Ajoutez la section AppSettings au fichier App.Config.

```xml
<appSettings>
    <add key="accountName" value="myaccount"/>
    <add key="accountKey" value="theaccountkey"/>
    <add key="clientId" value="theclientid"/>
    <add key="clientSecret" value="theclientsecret"/>
    <add key="container" value="stuff"/>
</appSettings>
```

Ajoutez les directives `using` suivantes et assurez-vous d’ajouter une référence à l’assembly System.Configuration du projet.

```csharp
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using System.Configuration;
using Microsoft.Azure;
using Microsoft.Azure.Storage;
using Microsoft.Azure.Storage.Auth;
using Microsoft.Azure.Storage.Blob;
using Microsoft.Azure.KeyVault;
using System.Threading;
using System.IO;
```

## <a name="add-a-method-to-get-a-token-to-your-console-application"></a>Ajouter une méthode pour obtenir un jeton pour votre application console

La méthode suivante est utilisée par les classes Key Vault qui doivent s’authentifier pour accéder à votre coffre de clés.

```csharp
private async static Task<string> GetToken(string authority, string resource, string scope)
{
    var authContext = new AuthenticationContext(authority);
    ClientCredential clientCred = new ClientCredential(
        CloudConfigurationManager.GetSetting("clientId"),
        CloudConfigurationManager.GetSetting("clientSecret"));
    AuthenticationResult result = await authContext.AcquireTokenAsync(resource, clientCred);

    if (result == null)
        throw new InvalidOperationException("Failed to obtain the JWT token");

    return result.AccessToken;
}
```

## <a name="access-azure-storage-and-key-vault-in-your-program"></a>Accéder au stockage Azure et à Key Vault dans votre programme

Ajoutez le code suivant à la méthode Main().

```csharp
// This is standard code to interact with Blob storage.
StorageCredentials creds = new StorageCredentials(
    CloudConfigurationManager.GetSetting("accountName"),
    CloudConfigurationManager.GetSetting("accountKey")
);
CloudStorageAccount account = new CloudStorageAccount(creds, useHttps: true);
CloudBlobClient client = account.CreateCloudBlobClient();
CloudBlobContainer contain = client.GetContainerReference(CloudConfigurationManager.GetSetting("container"));
contain.CreateIfNotExists();

// The Resolver object is used to interact with Key Vault for Azure Storage.
// This is where the GetToken method from above is used.
KeyVaultKeyResolver cloudResolver = new KeyVaultKeyResolver(GetToken);
```

> [!NOTE]
> Modèles d'objet Key Vault
> 
> Il est important de comprendre qu'il n'y a que deux modèles d'objet Key Vault à connaître : l’un est basé sur l'API REST (espace de noms KeyVault) et l'autre est une extension pour le chiffrement côté client.
> 
> Le client Key Vault interagit avec l’API REST et comprend les secrets et les clés web JSON pour les deux genres d’éléments qui sont contenus dans Key Vault.
> 
> Les extensions Key Vault sont des classes qui semblent avoir été créées spécifiquement pour le chiffrement côté client dans Azure Storage. Elles contiennent une interface pour les clés (IKey) et des classes basées sur le concept d’un programme de résolution de clés. Vous devez connaître deux implémentations d'IKey : RSAKey et SymmetricKey. Elles coïncident désormais avec les éléments contenus dans un coffre de clés, mais à ce stade, ce sont des classes indépendantes (la clé et le secret récupérés par le client Key Vault n'implémentent donc pas IKey).
> 
> 

## <a name="encrypt-blob-and-upload"></a>Chiffrement et téléchargement d’objets blob

Ajoutez le code suivant pour chiffrer un objet blob et le télécharger sur votre compte de stockage Azure. La méthode **ResolveKeyAsync** utilisée renvoie un IKey.

```csharp
// Retrieve the key that you created previously.
// The IKey that is returned here is an RsaKey.
var rsa = cloudResolver.ResolveKeyAsync(
            "https://contosokeyvault.vault.azure.net/keys/TestRSAKey1", 
            CancellationToken.None).GetAwaiter().GetResult();

// Now you simply use the RSA key to encrypt by setting it in the BlobEncryptionPolicy.
BlobEncryptionPolicy policy = new BlobEncryptionPolicy(rsa, null);
BlobRequestOptions options = new BlobRequestOptions() { EncryptionPolicy = policy };

// Reference a block blob.
CloudBlockBlob blob = contain.GetBlockBlobReference("MyFile.txt");

// Upload using the UploadFromStream method.
using (var stream = System.IO.File.OpenRead(@"C:\Temp\MyFile.txt"))
    blob.UploadFromStream(stream, stream.Length, null, options, null);
```

> [!NOTE]
> Si vous examinez le constructeur BlobEncryptionPolicy, vous remarquerez qu'il peut accepter une clé et/ou un programme de résolution. Rappelez-vous que vous ne pouvez pas utiliser un programme de résolution pour le chiffrement, car celui-ci ne prend pour l’instant pas en charge de clé par défaut.

## <a name="decrypt-blob-and-download"></a>Déchiffrement et téléchargement d’objets blob

Le déchiffrement s’effectue réellement quand les classes du programme de résolution sont pertinentes. L’ID de la clé utilisée pour le chiffrement est associé à l’objet blob dans ses métadonnées. Il est donc inutile de récupérer la clé et de mémoriser l’association entre la clé et les objets blob. Il vous suffit de vous assurer que la clé reste bien dans Key Vault.   

La clé privée d’une clé RSA reste dans Key Vault. Ainsi, la clé chiffrée des métadonnées d’objets blob qui contient la clé de chiffrement de contenu (CEK) est envoyée à Key Vault pour que le déchiffrement puisse avoir lieu.

Ajoutez le code suivant pour déchiffrer l’objet blob que vous venez de télécharger.

```csharp
// In this case, we will not pass a key and only pass the resolver because
// this policy will only be used for downloading / decrypting.
BlobEncryptionPolicy policy = new BlobEncryptionPolicy(null, cloudResolver);
BlobRequestOptions options = new BlobRequestOptions() { EncryptionPolicy = policy };

using (var np = File.Open(@"C:\data\MyFileDecrypted.txt", FileMode.Create))
    blob.DownloadToStream(np, null, options, null);
```

> [!NOTE]
> Il existe deux autres genres de programmes de résolution vous permettant de gérer plus facilement vos clés, à savoir : AggregateKeyResolver et CachingKeyResolver.

## <a name="use-key-vault-secrets"></a>Utiliser les secrets Key Vault

L’utilisation d’un secret avec le chiffrement côté client s’effectue via la classe SymmetricKey, car un secret est essentiellement une clé symétrique. Mais, comme indiqué ci-dessus, un secret dans Key Vault ne correspond pas exactement à une valeur SymmetricKey. Plusieurs choses sont à savoir ici :

* La clé d'une valeur SymmetricKey doit avoir une longueur fixe : 128, 192, 256, 384 ou 512 bits.
* La clé d’une valeur SymmetricKey doit être codée en Base64.
* Un secret Key Vault qui est utilisé comme une valeur SymmetricKey doit avoir un type de contenu « application/octet-stream » dans Key Vault.

Voici un exemple de création de secret dans Key Vault, effectuée dans PowerShell, qui peut être utilisé comme une valeur SymmetricKey.
Veuillez noter que la valeur codée en dur, $key, est fournie uniquement à des fins de démonstration. Dans votre propre code, vous devez générer cette clé.

```powershell
// Here we are making a 128-bit key so we have 16 characters.
//     The characters are in the ASCII range of UTF8 so they are
//    each 1 byte. 16 x 8 = 128.
$key = "qwertyuiopasdfgh"
$b = [System.Text.Encoding]::UTF8.GetBytes($key)
$enc = [System.Convert]::ToBase64String($b)
$secretvalue = ConvertTo-SecureString $enc -AsPlainText -Force

// Substitute the VaultName and Name in this command.
$secret = Set-AzureKeyVaultSecret -VaultName 'ContosoKeyVault' -Name 'TestSecret2' -SecretValue $secretvalue -ContentType "application/octet-stream"
```

Dans votre application console, vous pouvez utiliser le même appel qu’auparavant pour récupérer ce secret comme une valeur SymmetricKey.

```csharp
SymmetricKey sec = (SymmetricKey) cloudResolver.ResolveKeyAsync(
    "https://contosokeyvault.vault.azure.net/secrets/TestSecret2/",
    CancellationToken.None).GetAwaiter().GetResult();
```

Vous avez terminé. Vous n’avez plus qu’à l’utiliser !

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’utilisation du Stockage Microsoft Azure avec C#, consultez [Bibliothèque cliente Stockage Microsoft Azure pour .NET](/previous-versions/azure/dn261237(v=azure.100)).

Pour plus d’informations sur l’API REST Blob, consultez [API REST du service BLOB](/rest/api/storageservices/Blob-Service-REST-API).

Pour obtenir les dernières informations sur Stockage Microsoft Azure, consultez le [Blog de l’équipe Stockage Microsoft Azure](/archive/blogs/windowsazurestorage/).
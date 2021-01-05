---
title: Créer une SAP de compte avec .NET
titleSuffix: Azure Storage
description: Découvrez comment créer une signature d’accès partagé (SAP) de compte à l’aide de la bibliothèque cliente .NET.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 11/12/2020
ms.author: tamram
ms.reviewer: dineshm
ms.subservice: common
ms.custom: devx-track-csharp
ms.openlocfilehash: 2918b845430a6fc6dc59eca7041c114fc9d06515
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97092208"
---
# <a name="create-an-account-sas-with-net"></a>Créer une SAP de compte avec .NET

[!INCLUDE [storage-auth-sas-intro-include](../../../includes/storage-auth-sas-intro-include.md)]

Cet article explique comment utiliser la clé du compte de stockage pour créer une SAP de compte avec la [bibliothèque cliente Stockage Azure pour .NET](/dotnet/api/overview/azure/storage).

## <a name="create-an-account-sas"></a>Créer une SAP de compte

### <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

Une SAP de compte est signée avec la clé d’accès au compte. Utilisez la classe [StorageSharedKeyCredential](/dotnet/api/azure.storage.storagesharedkeycredential) afin de créer les informations d’identification utilisées pour signer le jeton SAS. Créez ensuite un objet [AccountSasBuilder](/dotnet/api/azure.storage.sas.accountsasbuilder) et appelez [ToSasQueryParameters](/dotnet/api/azure.storage.sas.accountsasbuilder.tosasqueryparameters) pour obtenir la chaîne de jeton SAP.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Sas.cs" id="Snippet_GetAccountSASToken":::

### <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

Pour créer une SAP de compte pour un conteneur, appelez la méthode [CloudStorageAccount.GetSharedAccessSignature](/dotnet/api/microsoft.azure.storage.cloudstorageaccount.getsharedaccesssignature).

L’exemple de code suivant crée une SAP de compte valide pour le service BLOB et le service de fichiers, et donne au client des autorisations d’accès en lecture, en écriture et en liste pour accéder aux API au niveau du service. Le SAP de compte limitant le protocole à HTTPS, la demande doit être effectuée avec ce protocole. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```csharp
static string GetAccountSASToken()
{
    // To create the account SAS, you need to use Shared Key credentials. Modify for your account.
    const string ConnectionString = "DefaultEndpointsProtocol=https;AccountName=<storage-account>;AccountKey=<account-key>";
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConnectionString);

    // Create a new access policy for the account.
    SharedAccessAccountPolicy policy = new SharedAccessAccountPolicy()
        {
            Permissions = SharedAccessAccountPermissions.Read | SharedAccessAccountPermissions.Write | SharedAccessAccountPermissions.List,
            Services = SharedAccessAccountServices.Blob | SharedAccessAccountServices.File,
            ResourceTypes = SharedAccessAccountResourceTypes.Service,
            SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
            Protocols = SharedAccessProtocol.HttpsOnly
        };

    // Return the SAS token.
    return storageAccount.GetSharedAccessSignature(policy);
}
```

---

## <a name="use-an-account-sas-from-a-client"></a>Utilisation d’une SAP de compte à partir d’un client

Afin utiliser la SAP de compte pour accéder aux API au niveau du service pour le service BLOB, construisez un client de service d’objet blob à l’aide de la SAP et du point de terminaison de stockage d’objets blob de votre compte de stockage.

### <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Sas.cs" id="Snippet_UseAccountSAS":::

### <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

Dans cet extrait de code, remplacez l’espace réservé `<storage-account>` par le nom de votre compte de stockage.

```csharp
static void UseAccountSAS(string sasToken)
{
    // Create new storage credentials using the SAS token.
    StorageCredentials accountSAS = new StorageCredentials(sasToken);
    // Use these credentials and the account name to create a Blob service client.
    CloudStorageAccount accountWithSAS = new CloudStorageAccount(accountSAS, "<storage-account>", endpointSuffix: null, useHttps: true);
    CloudBlobClient blobClientWithSAS = accountWithSAS.CreateCloudBlobClient();

    // Now set the service properties for the Blob client created with the SAS.
    blobClientWithSAS.SetServiceProperties(new ServiceProperties()
    {
        HourMetrics = new MetricsProperties()
        {
            MetricsLevel = MetricsLevel.ServiceAndApi,
            RetentionDays = 7,
            Version = "1.0"
        },
        MinuteMetrics = new MetricsProperties()
        {
            MetricsLevel = MetricsLevel.ServiceAndApi,
            RetentionDays = 7,
            Version = "1.0"
        },
        Logging = new LoggingProperties()
        {
            LoggingOperations = LoggingOperations.All,
            RetentionDays = 14,
            Version = "1.0"
        }
    });

    // The permissions granted by the account SAS also permit you to retrieve service properties.
    ServiceProperties serviceProperties = blobClientWithSAS.GetServiceProperties();
    Console.WriteLine(serviceProperties.HourMetrics.MetricsLevel);
    Console.WriteLine(serviceProperties.HourMetrics.RetentionDays);
    Console.WriteLine(serviceProperties.HourMetrics.Version);
}
```

---

## <a name="next-steps"></a>Étapes suivantes

- [Accorder un accès limité aux ressources du Stockage Azure à l’aide des signatures d’accès partagé (SAP)](storage-sas-overview.md)
- [Créer une SAP de compte](/rest/api/storageservices/create-account-sas)

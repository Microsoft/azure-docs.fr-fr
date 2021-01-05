---
title: Créer une stratégie d’accès stockée avec .NET
titleSuffix: Azure Storage
description: Utilisez le stockage Azure et .NET pour créer une stratégie d’accès stockée. Exercez des niveaux supplémentaires de contrôle sur les signatures d’accès partagé au niveau du service sur le serveur.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 06/16/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-csharp
ms.openlocfilehash: 6cd6147fbe38710bcefd580e71be1d6f5d446a21
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89010747"
---
# <a name="create-a-stored-access-policy-with-net"></a>Créer une stratégie d’accès stockée avec .NET

Une stratégie d’accès stockée fournit un niveau de contrôle supplémentaire sur les signatures d’accès partagé (SAP) au niveau du service côté serveur. La définition d’une stratégie d’accès stockée sert à regrouper des signatures d’accès partagé et à fournir des restrictions supplémentaires pour les signatures d’accès partagé liées par la stratégie. Vous pouvez utiliser une stratégie d'accès stockée pour modifier l'heure de début, l'heure d'expiration ou les autorisations d'une SAP, ou de la révoquer après sa publication.
  
Les ressources de Stockage Azure suivantes prennent en charge les stratégies d’accès stockées :  
  
- Conteneurs d’objets blob  
- Partages de fichiers  
- Files d’attente  
- Tables  
  
> [!NOTE]
> Une stratégie d’accès stockée sur un conteneur peut être associée à une signature d’accès partagé accordant des autorisations au conteneur lui-même ou aux objets blob qu’il contient. De même, une stratégie d’accès stockée sur un partage de fichiers peut être associée à une signature d’accès partagé accordant des autorisations au partage lui-même ou aux fichiers qu’il contient.  
>
> Les stratégies d’accès stockées sont prises en charge pour une SAP de service uniquement. Les stratégies d’accès stockées ne sont pas prises en charge pour les SAP de compte ou les SAP de délégation d’utilisateur.  

Pour plus d’informations sur les stratégies d’accès stockées, consultez [Définir une stratégie d’accès stockée](/rest/api/storageservices/define-stored-access-policy).

## <a name="create-a-stored-access-policy"></a>création d’une stratégie d’accès stockée

L’opération REST sous-jacente pour créer une stratégie d’accès stockée est [Set Container ACL](/rest/api/storageservices/set-container-acl). Vous devez autoriser l’opération à créer une stratégie d’accès stockée via une clé partagée en utilisant les clés d’accès au compte figurant dans une chaîne de connexion. L’autorisation de l’opération **Set Container ACL** avec des informations d’identification Azure AD n’est pas prise en charge. Pour plus d’informations, consultez [Autorisations pour l’appel d’opérations de données d’objet BLOB et de file d’attente](/rest/api/storageservices/authorize-with-azure-active-directory#permissions-for-calling-blob-and-queue-data-operations).

Les exemples de code suivants créent une stratégie d’accès stockée sur un conteneur. Vous pouvez utiliser la stratégie d’accès pour spécifier des contraintes pour une SAP de service sur le conteneur ou sur ses objets blob.

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Pour créer une stratégie d’accès stockée sur un conteneur avec la version 12 de la bibliothèque de client .NET pour Stockage Azure, appelez l’une des méthodes suivantes :

- [BlobContainerClient.SetAccessPolicy](/dotnet/api/azure.storage.blobs.blobcontainerclient.setaccesspolicy)
- [BlobContainerClient.SetAccessPolicyAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.setaccesspolicyasync)

L’exemple suivant crée une stratégie d’accès stockée effective pendant une journée, qui accorde des autorisations de lecture/écriture :

```csharp
async static Task CreateStoredAccessPolicyAsync(string containerName)
{
    string connectionString = "";

    // Use the connection string to authorize the operation to create the access policy.
    // Azure AD does not support the Set Container ACL operation that creates the policy.
    BlobContainerClient containerClient = new BlobContainerClient(connectionString, containerName);

    try
    {
        await containerClient.CreateIfNotExistsAsync();

        // Create one or more stored access policies.
        List<BlobSignedIdentifier> signedIdentifiers = new List<BlobSignedIdentifier>
        {
            new BlobSignedIdentifier
            {
                Id = "mysignedidentifier",
                AccessPolicy = new BlobAccessPolicy
                {
                    StartsOn = DateTimeOffset.UtcNow.AddHours(-1),
                    ExpiresOn = DateTimeOffset.UtcNow.AddDays(1),
                    Permissions = "rw"
                }
            }
        };
        // Set the container's access policy.
        await containerClient.SetAccessPolicyAsync(permissions: signedIdentifiers);
    }
    catch (RequestFailedException e)
    {
        Console.WriteLine(e.ErrorCode);
        Console.WriteLine(e.Message);
    }
    finally
    {
        await containerClient.DeleteAsync();
    }
}
```

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Pour créer une stratégie d’accès stockée sur un conteneur avec la version 12 de la bibliothèque de client .NET pour Stockage Azure, appelez l’une des méthodes suivantes :

- [CloudBlobContainer.SetPermissions](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.setpermissions)
- [CloudBlobContainer.SetPermissionsAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.setpermissionsasync)

L’exemple suivant crée une stratégie d’accès stockée effective pendant une journée, qui accorde des autorisations de lecture, d’écriture et d’affichage de liste :

```csharp
private static async Task CreateStoredAccessPolicyAsync(CloudBlobContainer container, string policyName)
{
    // Create a new stored access policy and define its constraints.
    // The access policy provides create, write, read, list, and delete permissions.
    SharedAccessBlobPolicy sharedPolicy = new SharedAccessBlobPolicy()
    {
        // When the start time for the SAS is omitted, the start time is assumed to be the time when Azure Storage receives the request.
        SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
        Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.List |
            SharedAccessBlobPermissions.Write
    };

    // Get the container's existing permissions.
    BlobContainerPermissions permissions = await container.GetPermissionsAsync();

    // Add the new policy to the container's permissions, and set the container's permissions.
    permissions.SharedAccessPolicies.Add(policyName, sharedPolicy);
    await container.SetPermissionsAsync(permissions);
}
```

---

## <a name="see-also"></a>Voir aussi

- [Accorder un accès limité aux ressources du Stockage Azure à l’aide des signatures d’accès partagé (SAP)](storage-sas-overview.md)
- [Définir une stratégie d’accès stockée](/rest/api/storageservices/define-stored-access-policy)
- [Configuration des chaînes de connexion du Stockage Azure](storage-configure-connection-string.md)

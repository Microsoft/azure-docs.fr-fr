---
title: Autoriser l’accès aux données avec une identité managée
titleSuffix: Azure Storage
description: Utilisez des identités managées pour les ressources Azure afin d’autoriser l’accès à des données de blob et de files d’attente à partir d’applications s’exécutant dans des machines virtuelles Azure, des applications de fonction, etc.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/07/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: ccc545b15f16879582c671b082cab40f6b11aa08
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96778969"
---
# <a name="authorize-access-to-blob-and-queue-data-with-managed-identities-for-azure-resources"></a>Autoriser l’accès à des données blob et de files d’attente avec des identités managées pour les ressources Azure

Le stockage d’objets blob et de files d’attente Azure prend en charge l’authentification Azure Active Directory (Azure AD) avec des [identités managées pour ressources Azure](../../active-directory/managed-identities-azure-resources/overview.md). Les identités managées pour ressources Azure autorisent l’accès à des objets blob et à des données de files d’attente en utilisant les informations d’identification Azure AD d’applications s’exécutant dans Machines virtuelles Azure, d’applications de fonction, de groupe de machines virtuelles identiques, et d’autres services. En utilisant des identités managées pour ressources Azure et Azure AD Authentication, vous pouvez éviter de stocker des informations d’identification avec les applications qui s’exécutent dans le cloud.  

Cet article montre comment autoriser l’accès au blob ou aux données de files d’attente avec une machine virtuelle Azure utilisant des identités managées pour des ressources Azure. Il explique également comment tester votre code dans l’environnement de développement.

## <a name="enable-managed-identities-on-a-vm"></a>Activer les identités managées sur une machine virtuelle

Avant de pouvoir utiliser les identités managées pour ressources Azure pour autoriser l’accès à des objets blob et à des files d’attente à partir de votre machine virtuelle, vous devez activer les identités managées pour ressources Azure sur la machine virtuelle. Pour savoir comment activer des identités managées pour ressources Azure, consultez un de ces articles :

- [Azure portal](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)
- [Azure PowerShell](../../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)
- [Azure CLI](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)
- [Modèle Azure Resource Manager](../../active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm.md)
- [Bibliothèques clientes Azure Resource Manager](../../active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm.md)

Pour plus d’informations sur les identités managées, consultez [Identités managées pour les ressources Azure](../../active-directory/managed-identities-azure-resources/overview.md).

## <a name="authenticate-with-the-azure-identity-library"></a>S’authentifier avec la bibliothèque d’identité Azure

La bibliothèque cliente Azure Identity fournit la prise en charge de l’authentification par jeton Azure AD pour le [SDK Azure](https://github.com/Azure/azure-sdk). Les versions les plus récentes des bibliothèques clientes Stockage Azure pour .NET, Java, Python et JavaScript s’intègrent à la bibliothèque d’identité Azure pour fournir un moyen simple et sécurisé d’acquérir un jeton OAuth 2.0 pour l’autorisation des demandes de Stockage Azure.

L’un des avantages de la bibliothèque de client Azure Identity est qu’elle vous permet d’utiliser le même code pour vous authentifier, que votre application soit exécutée dans l’environnement de développement ou dans Azure. La bibliothèque de client Azure Identity pour .NET authentifie un principal de sécurité. Lorsque votre code s’exécute dans Azure, le principal de sécurité est une identité managée pour les ressources Azure. Dans l’environnement de développement, étant donné que l’identité managée n’existe pas, la bibliothèque de client authentifie l’utilisateur ou le principal de service à des fins de test.

Après l’authentification, la bibliothèque de client d’identité Azure obtient des informations d’identification de jeton. Ces informations d’identification de jeton sont ensuite encapsulées dans l’objet client de service que vous créez pour effectuer des opérations sur le stockage Azure. La bibliothèque gère ceci pour vous, en toute transparence, en obtenant les informations d’identification de jeton appropriées.

Pour plus d’informations sur la bibliothèque cliente Azure Identity pour .NET, consultez [Bibliothèque cliente Azure Identity pour .NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/identity/Azure.Identity). Pour obtenir une documentation de référence sur la bibliothèque cliente Azure Identity, consultez [Espace de noms Azure.Identity](/dotnet/api/azure.identity).

### <a name="assign-azure-roles-for-access-to-data"></a>Attribuer des rôles Azure pour l’accès aux données

Lorsqu’un principal de sécurité Azure AD tente d’accéder aux données blob ou de file d’attente, ce principal de sécurité doit avoir des autorisations sur la ressource. Que le principal de sécurité soit une identité managée dans Azure ou un compte d’utilisateur Azure AD exécutant du code dans l’environnement de développement, le principal de sécurité doit se voir attribuer un rôle Azure qui accorde l’accès aux données de blob ou de file d’attente dans Stockage Azure. Pour plus d’informations sur l’attribution d’autorisations via Azure RBAC, consultez la section intitulée **Attribuer des rôles Azure pour les droits d’accès** dans [Autoriser l’accès aux blobs et files d’attente Azure à l’aide d’Azure Active Directory](../common/storage-auth-aad.md#assign-azure-roles-for-access-rights).

> [!NOTE]
> Lorsque vous créez un compte de stockage Azure, aucune autorisation d’accès aux données ne vous est automatiquement attribuée via Azure AD. Vous devez vous attribuer explicitement un rôle Azure pour le Stockage Azure. Vous pouvez l’attribuer au niveau de votre abonnement, groupe de ressources, compte de stockage, conteneur ou file d’attente.
>
> Avant de vous attribuer un rôle pour l’accès aux données, vous pouvez accéder aux données de votre compte de stockage via le portail Azure, car ce dernier peut également utiliser la clé de compte pour l’accès aux données. Pour plus d’informations, consultez [Choisir comment autoriser l’accès à des données blobs dans le portail Azure](../blobs/authorize-data-operations-portal.md).

### <a name="authenticate-the-user-in-the-development-environment"></a>Authentifier l’utilisateur dans l’environnement de développement

Lorsque votre code s’exécute dans l’environnement de développement, l’authentification peut être gérée automatiquement ou nécessiter une connexion du navigateur, selon les outils que vous utilisez. Par exemple, Microsoft Visual Studio prend en charge l’authentification unique (SSO), afin que le compte d’utilisateur actif Azure AD soit automatiquement utilisé pour l’authentification. Pour plus d’informations sur l’authentification unique, consultez [Authentification unique aux applications](../../active-directory/manage-apps/what-is-single-sign-on.md).

D’autres outils de développement peuvent vous inviter à vous connecter via un navigateur web.

### <a name="authenticate-a-service-principal-in-the-development-environment"></a>Authentifier un principal de service dans l’environnement de développement

Si votre environnement de développement ne prend pas en charge l’authentification unique ou la connexion via un navigateur web, vous pouvez utiliser un principal de service pour vous authentifier à partir de l’environnement de développement.

#### <a name="create-the-service-principal"></a>Créer le principal du service

Pour créer un principal du service avec Azure CLI et attribuer un rôle Azure, appelez la commande [az ad sp create-for-rbac](/cli/azure/ad/sp#az-ad-sp-create-for-rbac). Fournissez un rôle d’accès aux données de Stockage Azure à attribuer au nouveau principal du service. Indiquez également l’étendue de l’attribution de rôle. Pour plus d’informations sur les rôles intégrés fournis pour Stockage Azure, consultez [Rôles intégrés Azure](../../role-based-access-control/built-in-roles.md).

Si vous ne disposez pas des autorisations suffisantes pour attribuer un rôle au principal du service, vous devrez peut-être demander au propriétaire du compte ou à l’administrateur d’effectuer l’attribution de rôle.

L’exemple suivant utilise Azure CLI pour créer un principal du service et lui attribuer le rôle de **lecteur de données d’objets blob de stockage** avec une étendue de compte

```azurecli-interactive
az ad sp create-for-rbac \
    --name <service-principal> \
    --role "Storage Blob Data Contributor" \
    --scopes /subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>
```

La commande `az ad sp create-for-rbac` retourne une liste de propriétés de principal du service au format JSON. Copiez ces valeurs pour pouvoir les utiliser afin de créer les variables d’environnement nécessaires à l’étape suivante.

```json
{
    "appId": "generated-app-ID",
    "displayName": "service-principal-name",
    "name": "http://service-principal-uri",
    "password": "generated-password",
    "tenant": "tenant-ID"
}
```

> [!IMPORTANT]
> La propagation des attributions de rôles Azure peut prendre plusieurs minutes.

#### <a name="set-environment-variables"></a>Définir des variables d’environnement

La bibliothèque cliente Azure Identity lit les valeurs de trois variables d’environnement au moment de l’exécution pour authentifier le principal du service. Le tableau suivant indique la valeur à définir pour chaque variable d’environnement.

|Variable d’environnement|Valeur
|-|-
|`AZURE_CLIENT_ID`|ID de l’application pour le principal du service
|`AZURE_TENANT_ID`|ID de locataire Azure AD du principal du service
|`AZURE_CLIENT_SECRET`|Mot de passe généré pour le principal du service

> [!IMPORTANT]
> Après avoir défini les variables d’environnement, fermez et rouvrez votre fenêtre de console. Si vous utilisez Visual Studio ou un autre environnement de développement, vous devrez peut-être redémarrer l’environnement de développement afin qu’il enregistre les nouvelles variables d’environnement.

Pour plus d’informations, consultez [Créer une identité pour une application Azure dans le portail](../../active-directory/develop/howto-create-service-principal-portal.md).

[!INCLUDE [storage-install-packages-blob-and-identity-include](../../../includes/storage-install-packages-blob-and-identity-include.md)]

## <a name="net-code-example-create-a-block-blob"></a>Exemple de code .NET : créer un objet blob de blocs

Ajoutez les directives `using` suivantes à votre code pour utiliser les bibliothèques de client Azure Identity et Stockage Azure.

```csharp
using Azure;
using Azure.Identity;
using Azure.Storage.Blobs;
using System;
using System.IO;
using System.Text;
using System.Threading.Tasks;
```

Pour obtenir les informations d’identification d’un jeton que votre code peut utiliser pour autoriser les requêtes au stockage Azure, créez une instance de la classe [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential). L’exemple de code suivant montre comment obtenir des informations d’identification du jeton authentifié et comment l’utiliser pour créer un objet client de service, puis utiliser le client de service pour charger un nouveau blob :

```csharp
async static Task CreateBlockBlobAsync(string accountName, string containerName, string blobName)
{
    // Construct the blob container endpoint from the arguments.
    string containerEndpoint = string.Format("https://{0}.blob.core.windows.net/{1}",
                                                accountName,
                                                containerName);

    // Get a credential and create a client object for the blob container.
    BlobContainerClient containerClient = new BlobContainerClient(new Uri(containerEndpoint),
                                                                    new DefaultAzureCredential());

    try
    {
        // Create the container if it does not exist.
        await containerClient.CreateIfNotExistsAsync();

        // Upload text to a new block blob.
        string blobContents = "This is a block blob.";
        byte[] byteArray = Encoding.ASCII.GetBytes(blobContents);

        using (MemoryStream stream = new MemoryStream(byteArray))
        {
            await containerClient.UploadBlobAsync(blobName, stream);
        }
    }
    catch (RequestFailedException e)
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
}
```

> [!NOTE]
> Pour autoriser les requêtes sur des données d’objets BLOB ou de file d’attente avec Azure AD, vous devez utiliser HTTPS pour ces requêtes.

## <a name="next-steps"></a>Étapes suivantes

- [Gérer les droits d’accès aux données de stockage avec le contrôle RBAC Azure](./storage-auth-aad-rbac-portal.md).
- [Utiliser Azure AD avec des applications de stockage](storage-auth-aad-app.md).
- [Exécuter des commandes PowerShell avec des informations d’identification Azure AD pour accéder aux données d’objet blob](../blobs/authorize-data-operations-powershell.md)

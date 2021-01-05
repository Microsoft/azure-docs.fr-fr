---
author: mikben
ms.service: azure-communication-services
ms.topic: include
ms.date: 9/1/2020
ms.author: mikben
ms.openlocfilehash: 9a9f8fdda1bc853057f3eb858e85b938357397cd
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94886112"
---
## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/dotnet/).
- La dernière version de la [bibliothèque de client .NET Core](https://dotnet.microsoft.com/download/dotnet-core) pour votre système d’exploitation.
- Procurez-vous la dernière version de la [bibliothèque de client .NET Identity](/dotnet/api/azure.identity?view=azure-dotnet).
- Procurez-vous la dernière version de la [bibliothèque de client .NET Management](../../concepts/sdk-options.md).

## <a name="installing-the-client-library"></a>Installation de la bibliothèque de client

Tout d’abord, incluez la bibliothèque de client Communication Services Management dans votre projet C# :

```csharp
using Azure.ResourceManager.Communication;
```

## <a name="subscription-id"></a>Identifiant d’abonnement

Vous devez connaître l’ID de votre abonnement Azure. Vous pouvez l’obtenir à partir du portail :

1.  Connectez-vous à votre compte Azure
2.  Sélectionnez Abonnements dans la barre latérale gauche
3.  Sélectionnez l’abonnement à utiliser
4.  Cliquez sur Vue d’ensemble
5.  Sélectionnez votre ID d’abonnement

Dans ce guide de démarrage rapide, nous supposons que vous avez stocké l’ID d’abonnement dans une variable d’environnement appelée `AZURE_SUBSCRIPTION_ID`.

## <a name="authentication"></a>Authentification

Pour communiquer avec Azure Communication Services, vous devez d’abord vous authentifier auprès d’Azure. Cette authentification s’effectue généralement à l’aide d’une identité de principal de service.

### <a name="option-1-managed-identity"></a>Option 1 : Identité managée

Si votre code s’exécute en tant que service dans Azure, la méthode la plus simple pour vous authentifier est d’acquérir une identité managée à partir d’Azure. Découvrez-en plus sur les [identités managées](../../../active-directory/managed-identities-azure-resources/overview.md).

[Services Azure qui prennent en charge les identités managées](../../../active-directory/managed-identities-azure-resources/services-support-managed-identities.md)

[Guide pratique pour utiliser des identités managées avec App Service et Azure Functions](../../../app-service/overview-managed-identity.md?tabs=dotnet)

#### <a name="system-assigned-managed-identity"></a>[Identité managée affectée par le système](../../../app-service/overview-managed-identity.md?tabs=dotnet#add-a-system-assigned-identity)

```csharp
using Azure.Identity;
using Azure.ResourceManager.Communication;
using Azure.ResourceManager.Communication.Models;
using System;
...
var subscriptionId = "AZURE_SUBSCRIPTION_ID";
var acsClient = new CommunicationManagementClient(subscriptionId, new ManagedIdentityCredential());
```

#### <a name="user-assigned-managed-identity"></a>[Identité managée affectée par l’utilisateur](../../../app-service/overview-managed-identity.md?tabs=dotnet#add-a-user-assigned-identity)

Le ClientId de l’identité managée que vous avez créée doit être passé explicitement à `ManagedIdentityCredential`.

```csharp
using Azure.Identity;
using Azure.ResourceManager.Communication;
using Azure.ResourceManager.Communication.Models;
using System;
...
var subscriptionId = "AZURE_SUBSCRIPTION_ID";
var managedIdentityCredential = new ManagedIdentityCredential("AZURE_CLIENT_ID");
var acsClient = new CommunicationManagementClient(subscriptionId, managedIdentityCredential);
```

### <a name="option-2-service-principal"></a>Option n°2 : Principal de service

Au lieu d’utiliser une identité managée, vous pouvez aussi vous authentifier auprès d’Azure à l’aide d’un principal de service que vous gérez vous-même. Pour plus d’informations, consultez la documentation relative à la [création et gestion d’un principal de service dans Azure Active Directory](../../../active-directory/develop/howto-create-service-principal-portal.md).

Après avoir créé votre principal de service, vous devez collecter les informations suivantes à son sujet à partir du portail Azure :

- **ID client**
- **Clé secrète client**
- **Tenant ID**

Stockez ces valeurs dans les variables d’environnement `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET` et `AZURE_TENANT_ID`, respectivement. Vous pouvez ensuite créer un client Communication Services Management de la façon suivante :

```csharp
using Azure.Identity;
using Azure.ResourceManager.Communication;
using Azure.ResourceManager.Communication.Models;
using System;
...
var subscriptionId = Environment.GetEnvironmentVariable("AZURE_SUBSCRIPTION_ID");
var acsClient = new CommunicationManagementClient(subscriptionId, new EnvironmentCredential());
```

### <a name="option-3-user-identity"></a>Option 3 : Identité de l’utilisateur

Si vous souhaitez appeler Azure pour le compte d’un utilisateur interactif, au lieu d’utiliser une identité de service, utilisez le code suivant pour créer un client Azure Communication Services Management. Une fenêtre de navigateur s’ouvrira pour inviter l’utilisateur à entrer ses informations d’identification MSA ou Azure AD.

```csharp
using Azure.Identity;
using Azure.ResourceManager.Communication;
using Azure.ResourceManager.Communication.Models;
using System;
...
var subscriptionId = Environment.GetEnvironmentVariable("AZURE_SUBSCRIPTION_ID");
var communicationServiceClient = new CommunicationManagementClient(subscriptionId, new InteractiveBrowserCredential());
```

## <a name="managing-communication-services-resources"></a>Gestion des ressources Communication Services

### <a name="interacting-with-azure-resources"></a>Interaction avec des ressources Azure

Maintenant que vous êtes authentifié, vous pouvez effectuer des appels d’API à l’aide de votre client Management.

Dans chacun des exemples suivants, nous attribuons les ressources Communication Services à un groupe de ressources existant.

Si vous avez besoin de créer un groupe de ressources, vous pouvez le faire à l’aide du [portail Azure](../../../azure-resource-manager/management/manage-resource-groups-portal.md) ou de la [bibliothèque de client Azure Resource Manager](https://github.com/Azure/azure-sdk-for-net/blob/master/doc/mgmt_preview_quickstart.md).

### <a name="create-and-manage-a-communication-services-resource"></a>Créer et gérer une ressource Communication Services

L’instance du client de bibliothèque Communication Services Management (``Azure.ResourceManager.Communication.CommunicationManagementClient``) peut être utilisée pour effectuer des opérations sur des ressources Communication Services.

#### <a name="create-a-communication-services-resource"></a>Créer une ressource Communication Services

Quand vous créez une ressource Communication Services, vous devez spécifier le nom du groupe de ressources et le nom de la ressource. Notez que la propriété `Location` est toujours `global` et, pendant la préversion publique, la valeur `DataLocation` doit être définie à `UnitedStates`.

```csharp
var resourceGroupName = "myResourceGroupName";
var resourceName = "myResource";
var resource = new CommunicationServiceResource { Location = "Global", DataLocation = "UnitedStates"  };
var operation = await acsClient.CommunicationService.StartCreateOrUpdateAsync(resourceGroupName, resourceName, resource);
await operation.WaitForCompletionAsync();
```

#### <a name="update-a-communication-services-resource"></a>Mettre à jour une ressource Communication Services

```csharp
...
var resourceGroupName = "myResourceGroupName";
var resourceName = "myResource";
var resource = new CommunicationServiceResource { Location = "Global", DataLocation = "UnitedStates" };
resource.Tags.Add("environment","test");
resource.Tags.Add("department","tech");
// Use existing resource name and new resource object
var operation = await acsClient.CommunicationService.StartCreateOrUpdateAsync(resourceGroupName, resourceName, resource);
await operation.WaitForCompletionAsync();
```

#### <a name="list-all-communication-services-resources"></a>Lister toutes les ressources Communication Services

```csharp
var resources = acsClient.CommunicationService.ListBySubscription();
foreach (var resource in resources)
{
    Console.WriteLine(resource.Name);
}
```

#### <a name="delete-a-communication-services-resource"></a>Supprimer une ressource Communication Services

```csharp
var resourceGroupName = "myResourceGroupName";
var resourceName = "myResource";
await acsClient.CommunicationService.StartDeleteAsync(resourceGroupName, resourceName);
```

## <a name="managing-keys-and-connection-strings"></a>Gestion des clés et des chaînes de connexion

Chaque ressource Communication Services a une paire de clés d’accès et les chaînes de connexion correspondantes. Ces clés sont accessibles par le biais de la bibliothèque de client Management, puis elles sont utilisées par d’autres bibliothèques de client Communication Services pour s’authentifier auprès d’Azure Communication Services.

#### <a name="get-access-keys-for-a-communication-services-resource"></a>Obtenir des clés d’accès pour une ressource Communication Services

```csharp
var resourceGroupName = "myResourceGroupName";
var resourceName = "myResource";
var keys = await acsClient.CommunicationService.ListKeysAsync(resourceGroupName, resourceName);

Console.WriteLine(keys.Value.PrimaryConnectionString);
Console.WriteLine(keys.Value.SecondaryConnectionString);
```

#### <a name="regenerate-an-access-key-for-a-communication-services-resource"></a>Regénérer une clé d’accès pour une ressource Communication Services

```csharp
var resourceGroupName = "myResourceGroupName";
var resourceName = "myResource";
var keyParams = new RegenerateKeyParameters { KeyType = KeyType.Primary };
var keys = await acsClient.CommunicationService.RegenerateKeyAsync(resourceGroupName, resourceName, keyParams);

Console.WriteLine(keys.Value.PrimaryKey);
```
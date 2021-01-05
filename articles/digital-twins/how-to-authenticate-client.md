---
title: Écrire le code d’authentification de l’application
titleSuffix: Azure Digital Twins
description: Voir comment écrire un code d’authentification dans une application cliente
author: baanders
ms.author: baanders
ms.date: 10/7/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: bf7b829d70af27850affe619d47ed4a4f5ec1bea
ms.sourcegitcommit: 58f12c358a1358aa363ec1792f97dae4ac96cc4b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93279911"
---
# <a name="write-client-app-authentication-code"></a>Écrire le code d’authentification de l’application cliente

Après avoir [configuré une instance et une authentification Azure Digital Twins](how-to-set-up-instance-portal.md), vous pouvez créer une application cliente que vous allez utiliser pour interagir avec l’instance. Après avoir configuré un projet client de démarrage, vous devez **écrire du code dans cette application cliente pour l’authentifier** auprès de l’instance Azure Digital Twins.

Azure Digital Twins effectue l’authentification à l’aide de [jetons de sécurité Azure AD Security basés sur OAUTH 2.0](../active-directory/develop/security-tokens.md#json-web-tokens-jwts-and-claims). Pour authentifier votre kit de développement logiciel (SDK), vous devez obtenir un jeton du porteur avec les autorisations appropriées sur Azure Digital Twins et le transmettre avec vos appels d’API. 

Cet article explique comment obtenir des informations d’identification à l’aide de la bibliothèque cliente `Azure.Identity`. Bien que cet article présente des exemples de code en C#, tels que ce que vous écrivez pour le [kit de développement logiciel (SDK) .NET (C#)](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true), vous pouvez utiliser une version de `Azure.Identity` quel que soit le kit de développement logiciel (SDK) que vous utilisez (pour plus d’informations sur les kits de développement logiciel (SDK) disponibles pour Azure Digital Twins, consultez [*Guide pratique : Utiliser les API et les kits de développement logiciel (SDK) Azure Digital Twins*](how-to-use-apis-sdks.md)).

## <a name="prerequisites"></a>Prérequis

Tout d’abord, suivez les étapes de configuration décrites dans [*Guide pratique : Configurer une instance et l’authentification*](how-to-set-up-instance-portal.md). Cela permet de s’assurer que vous disposez d’une instance Azure Digital Twins et que votre utilisateur dispose d’autorisations d’accès. Une fois cette configuration terminée, vous êtes prêt à écrire le code de l’application cliente.

Pour continuer, vous aurez besoin d’un projet d’application cliente dans lequel écrire votre code. Si vous n’avez pas encore configuré de projet d’application cliente, créez un projet de base à utiliser avec ce didacticiel dans le langage de votre choix.

## <a name="common-authentication-methods-with-azureidentity"></a>Méthodes d’authentification courantes avec Azure.Identity

`Azure.Identity` est une bibliothèque cliente qui fournit plusieurs méthodes d’obtention d’informations d’identification que vous pouvez utiliser pour obtenir un jeton du porteur et vous authentifier avec votre kit de développement logiciel (SDK). Bien que cet article donne des exemples en C# , vous pouvez afficher `Azure.Identity` pour plusieurs langages, notamment...

* [.NET (C#)](/dotnet/api/azure.identity?preserve-view=true&view=azure-dotnet)
* [Java](/java/api/overview/azure/identity-readme?preserve-view=true&view=azure-java-stable)
* [JavaScript](/javascript/api/overview/azure/identity-readme?preserve-view=true&view=azure-node-latest)
* [Python](/python/api/overview/azure/identity-readme?preserve-view=true&view=azure-python)

Trois méthodes d’obtention d’informations d’identification courantes dans `Azure.Identity` sont les suivantes :

* [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential?preserve-view=true&view=azure-dotnet) fournit un flux d’authentification `TokenCredential` par défaut pour les applications qui seront déployées sur Azure, et est **le choix recommandé pour le développement local**. Il peut également être activé pour essayer les deux autres méthodes recommandées dans cet article. Il encapsule `ManagedIdentityCredential` et peut accéder à `InteractiveBrowserCredential` à l’aide d’une variable de configuration.
* [ManagedIdentityCredential](/dotnet/api/azure.identity.managedidentitycredential?preserve-view=true&view=azure-dotnet) fonctionne parfaitement lorsqu’il vous faut des [identités managées (MSI)](../active-directory/managed-identities-azure-resources/overview.md) et peut être utilisé avec Azure Functions et pour déployer sur des services Azure.
* [InteractiveBrowserCredential](/dotnet/api/azure.identity.interactivebrowsercredential?preserve-view=true&view=azure-dotnet) est destiné aux applications interactives et peut être utilisé pour créer un client de kit de développement logiciel (SDK) authentifié

L’exemple suivant montre comment utiliser chacune de ces méthodes avec le kit de développement logiciel (SDK) .NET (C#).

## <a name="authentication-examples-net-c-sdk"></a>Exemples d’authentification : Kit de développement logiciel (SDK) .NET (C#)

Cette section propose un exemple C# d’utilisation du kit de développement logiciel (SDK) .NET fourni pour écrire du code d’authentification.

Tout d’abord, incluez le package du kit de développement logiciel (SDK) `Azure.DigitalTwins.Core` et le package `Azure.Identity` dans votre projet. Selon les outils que vous choisissez, vous pouvez inclure les packages à l’aide du gestionnaire de package Visual Studio ou de l’outil en ligne de commande `dotnet`. 

Vous devez également ajouter les instructions using suivantes au code de votre projet :

```csharp
using Azure.Identity;
using Azure.DigitalTwins.Core;
```

Ajoutez ensuite du code pour obtenir des informations d’identification à l’aide de l’une des méthodes dans `Azure.Identity`.

### <a name="defaultazurecredential-method"></a>Méthode DefaultAzureCredential

[DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential?preserve-view=true&view=azure-dotnet) fournit un flux d’authentification `TokenCredential` par défaut pour les applications qui seront déployées sur Azure, et est **le choix recommandé pour le développement local**.

Pour utiliser les informations d’identification Azure par défaut, vous avez besoin de l’URL de l’instance Azure Digital Twins ([instructions pour les trouver](how-to-set-up-instance-portal.md#verify-success-and-collect-important-values)).

Voici un exemple de code pour ajouter un `DefaultAzureCredential` à votre projet :

```csharp
// The URL of your instance, starting with the protocol (https://)
private static string adtInstanceUrl = "https://<your-Azure-Digital-Twins-instance-URL>";

//...

DigitalTwinsClient client;
try
{
    var credential = new DefaultAzureCredential();
    client = new DigitalTwinsClient(new Uri(adtInstanceUrl), credential);
} catch(Exception e)
{
    Console.WriteLine($"Authentication or client creation error: {e.Message}");
    Environment.Exit(0);
}
```

#### <a name="set-up-local-azure-credentials"></a>Configurer les informations d’identification Azure locales

[!INCLUDE [Azure Digital Twins: local credentials prereq (inner)](../../includes/digital-twins-local-credentials-inner.md)]

### <a name="managedidentitycredential-method"></a>Méthode ManagedIdentityCredential

La méthode [ManagedIdentityCredential](/dotnet/api/azure.identity.managedidentitycredential?preserve-view=true&view=azure-dotnet) fonctionne parfaitement lorsqu’il vous faut des [identités managées (MSI)](../active-directory/managed-identities-azure-resources/overview.md), en cas d’utilisation d’Azure Functions, par exemple.

Cela signifie que vous pouvez utiliser `ManagedIdentityCredential` dans le même projet que `DefaultAzureCredential` ou `InteractiveBrowserCredential`, pour authentifier une autre partie du projet.

Pour utiliser les informations d’identification Azure par défaut, vous avez besoin de l’URL de l’instance Azure Digital Twins ([instructions pour les trouver](how-to-set-up-instance-portal.md#verify-success-and-collect-important-values)).

Dans une fonction Azure, vous pouvez utiliser les informations d’identification de l’identité managée comme suit :

```csharp
ManagedIdentityCredential cred = new ManagedIdentityCredential(adtAppId);
DigitalTwinsClientOptions opts = 
    new DigitalTwinsClientOptions { Transport = new HttpClientTransport(httpClient) });
client = new DigitalTwinsClient(new Uri(adtInstanceUrl), cred, opts);
```

### <a name="interactivebrowsercredential-method"></a>Méthode InteractiveBrowserCredential

La méthode [InteractiveBrowserCredential](/dotnet/api/azure.identity.interactivebrowsercredential?preserve-view=true&view=azure-dotnet) est destinée aux applications interactives et affiche un navigateur web pour l’authentification. Vous pouvez l’utiliser à la place de `DefaultAzureCredential` lorsque vous avez besoin d’une authentification interactive.

Pour utiliser les informations d’identification du navigateur interactif, vous avez besoin d’une **inscription d’application** disposant d’autorisations sur les API Azure Digital Twins. Pour plus d’informations sur la configuration de cette inscription d’application, consultez [*Guide pratique : Créer une inscription d’application*](how-to-create-app-registration.md). Une fois l’inscription d’application configurée, vous avez besoin de...
* l’ *ID (client) d’application* de l’inscription d’application ( [instructions pour le trouver](how-to-create-app-registration.md#collect-client-id-and-tenant-id))
* l’ *ID (locataire) de répertoire* de l’inscription d’application ( [instructions pour le trouver](how-to-create-app-registration.md#collect-client-id-and-tenant-id))
* l’URL de l’instance Azure Digital Twins ([instructions pour la trouver](how-to-set-up-instance-portal.md#verify-success-and-collect-important-values))

Voici un exemple de code permettant de créer un client du kit de développement logiciel (SDK) authentifié à l’aide de `InteractiveBrowserCredential`.

```csharp
// Your client / app registration ID
private static string clientId = "<your-client-ID>"; 
// Your tenant / directory ID
private static string tenantId = "<your-tenant-ID>";
// The URL of your instance, starting with the protocol (https://)
private static string adtInstanceUrl = "https://<your-Azure-Digital-Twins-instance-URL>";

//...

DigitalTwinsClient client;
try
{
    var credential = new InteractiveBrowserCredential(tenantId, clientId);
    client = new DigitalTwinsClient(new Uri(adtInstanceUrl), credential);
} catch(Exception e)
{
    Console.WriteLine($"Authentication or client creation error: {e.Message}");
    Environment.Exit(0);
}
```

>[!NOTE]
> Bien que vous puissiez placer l’ID client, l’ID de locataire et l’URL de l’instance directement dans le code comme indiqué ci-dessus, il peut être judicieux de faire en sorte que votre code récupère ces valeurs à partir d’un fichier de configuration ou d’une variable d’environnement.

#### <a name="other-notes-about-authenticating-azure-functions"></a>Autres remarques sur l’authentification Azure Functions

Consultez le [*Guide pratique : Configurer une fonction Azure pour le traitement des données*](how-to-create-azure-function.md) pour obtenir un exemple plus complet sur quelques-unes des options de configuration importantes dans le contexte des fonctions.

En outre, pour utiliser l’authentification dans une fonction, n’oubliez pas :
* [Activer une identité managée](../app-service/overview-managed-identity.md?tabs=dotnet)
* D’utiliser des [variables d'environnement](/sandbox/functions-recipes/environment-variables?tabs=csharp) le cas échéant
* Affectez des autorisations à l’application de fonction pour lui permettre d’accéder aux API Digital Twins. Pour plus d’informations sur les processus Azure Functions, consultez [*Guide pratique : Configurer une fonction Azure pour le traitement des données*](how-to-create-azure-function.md).

## <a name="other-credential-methods"></a>Autres méthodes d’informations d’identification

Si les scénarios d’authentification indiqués ci-dessus ne couvrent pas les besoins de votre application, vous pouvez explorer les autres types d’authentification proposés dans la [**Plateforme d’identité Microsoft**](../active-directory/develop/v2-overview.md#getting-started). La documentation de cette plateforme couvre des scénarios d’authentification supplémentaires, organisés par type d’application.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur le fonctionnement de la sécurité dans Azure Digital Twins :
* [*Concepts : Sécurité pour les solutions Azure Digital Twins*](concepts-security.md)

Sinon, maintenant que l’authentification est configurée, passez à la création de modèles dans votre instance :
* [*Guide pratique : Gérer des modèles personnalisés*](how-to-manage-model.md)
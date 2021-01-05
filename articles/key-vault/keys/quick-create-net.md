---
title: Démarrage rapide - Bibliothèque de client de clés Azure Key Vault pour .NET (version 4)
description: Découvrez comment créer, récupérer et supprimer des clés d’un coffre de clés Azure en utilisant la bibliothèque de client .NET (version 4)
author: msmbaldwin
ms.author: mbaldwin
ms.date: 09/23/2020
ms.service: key-vault
ms.subservice: keys
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.openlocfilehash: 9a1404969b67a0c4919902a407427227ce397d9a
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97825937"
---
# <a name="quickstart-azure-key-vault-key-client-library-for-net-sdk-v4"></a>Démarrage rapide : Bibliothèque de client de clés Azure Key Vault pour .NET (SDK v4)

Découvrez comment démarrer avec la bibliothèque de client de clés Azure Key Vault pour .NET. [Azure Key Vault](../general/overview.md) est un service cloud qui fournit un magasin de clés de chiffrement sécurisé. Vous pouvez stocker des clés de chiffrement, des mots de passe, des certificats et d’autres secrets de manière sécurisée. Vous pouvez créer et gérer des coffres de clés Azure grâce au portail Azure. Dans ce guide de démarrage rapide, vous allez découvrir comment créer, récupérer et supprimer des clés dans un coffre de clés Azure en utilisant la bibliothèque de client de clés .NET.

Ressources de la bibliothèque de client de clés Key Vault :

[Documentation de référence sur l’API](/dotnet/api/azure.security.keyvault.keys) | [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/keyvault) | [Package (NuGet)](https://www.nuget.org/packages/Azure.Security.KeyVault.keys/)

Pour plus d’informations sur Key Vault et les clés, consultez :
- [Vue d’ensemble du coffre de clés](../general/overview.md)
- [Vue d’ensemble des clés](about-keys.md)

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure : [créez-en un gratuitement](https://azure.microsoft.com/free/dotnet)
* [SDK .NET Core 3.1 ou ultérieur](https://dotnet.microsoft.com/download/dotnet-core)
* [Azure CLI](/cli/azure/install-azure-cli)
* Un coffre de clés. Vous pouvez en créer un en utilisant le [portail Azure](../general/quick-create-portal.md), [Azure CLI](../general/quick-create-cli.md) ou [Azure PowerShell](../general/quick-create-powershell.md)

Ce guide de démarrage rapide utilise `dotnet` et Azure CLI

## <a name="setup"></a>Programme d’installation

Ce guide de démarrage rapide utilise la bibliothèque Azure Identity avec Azure CLI pour authentifier l’utilisateur auprès des services Azure. Les développeurs peuvent également utiliser Visual Studio ou Visual Studio Code pour authentifier leurs appels. Pour plus d’informations, consultez [Authentifier le client avec la bibliothèque de client Azure Identity](/dotnet/api/overview/azure/identity-readme?#authenticate-the-client&preserve-view=true).

### <a name="sign-in-to-azure"></a>Connexion à Azure

1. Exécutez la commande `login`.

    ```azurecli-interactive
    az login
    ```

    Si l’interface CLI peut ouvrir votre navigateur par défaut, elle le fait et charge une page de connexion Azure par la même occasion.

    Sinon, ouvrez une page de navigateur à l’adresse [https://aka.ms/devicelogin](https://aka.ms/devicelogin) et entrez le code d’autorisation affiché dans votre terminal.

2. Dans le navigateur, connectez-vous avec les informations d’identification de votre compte.

#### <a name="grant-access-to-your-key-vault"></a>Accorder l’accès à votre coffre de clés

Créez une stratégie d’accès pour votre coffre de clés, qui accorde des autorisations de clé à votre compte d’utilisateur.

```console
az keyvault set-policy --name <your-key-vault-name> --upn user@domain.com --key-permissions delete get list create purge
```

### <a name="create-new-net-console-app"></a>Créer une application console .NET

1. Dans un shell de commandes, exécutez la commande suivante pour créer un projet nommé `key-vault-console-app` :

    ```dotnetcli
    dotnet new console --name key-vault-console-app
    ```

1. Accédez au répertoire *key-vault-console-app* nouvellement créé, puis exécutez la commande suivante pour générer le projet :

    ```dotnetcli
    dotnet build
    ```

    La sortie de génération ne doit contenir aucun avertissement ni erreur.
    
    ```console
    Build succeeded.
     0 Warning(s)
     0 Error(s)
    ```

### <a name="install-the-packages"></a>Installer les packages

À partir de l’interpréteur de commandes, installez la bibliothèque de client de clés Azure Key Vault pour .NET :

```dotnetcli
dotnet add package Azure.Security.KeyVault.Keys
```

Pour ce guide de démarrage rapide, vous devez également installer la bibliothèque de client du SDK Azure pour Azure Identity :

```dotnetcli
dotnet add package Azure.Identity
```

#### <a name="set-environment-variables"></a>Définir des variables d’environnement

Cette application utilise le nom de coffre de clés en tant que variable d’environnement appelée `KEY_VAULT_NAME`.

Windows
```cmd
set KEY_VAULT_NAME=<your-key-vault-name>
````
Windows PowerShell
```powershell
$Env:KEY_VAULT_NAME=<your-key-vault-name>
```

macOS ou Linux
```bash
export KEY_VAULT_NAME=<your-key-vault-name>
```

## <a name="object-model"></a>Modèle objet

La bibliothèque de client de clés Azure Key Vault pour .NET vous permet de gérer des clés. La section [Exemples de code](#code-examples) montre comment créer un client et définir, récupérer et supprimer une clé.

## <a name="code-examples"></a>Exemples de code

### <a name="add-directives"></a>Ajouter des directives

Ajoutez les directives suivantes en haut de *Program.cs* :

```csharp
using System;
using Azure.Identity;
using Azure.Security.KeyVault.Keys;
```

### <a name="authenticate-and-create-a-client"></a>Authentifier et créer un client

Dans ce guide de démarrage rapide, l’utilisateur connecté est utilisé pour l’authentification auprès du coffre de clés, qui est la méthode recommandée pour le développement local. Pour les applications déployées sur Azure, l’identité managée doit être affectée à App Service ou à une machine virtuelle. Pour plus d’informations, consultez [Vue d’ensemble des identités managées](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

Dans l’exemple ci-dessous, le nom de votre coffre de clés est étendu à l’URI du coffre de clés, au format « https://\<your-key-vault-name\>.vault.azure.net ». Cet exemple utilise la classe [« DefaultAzureCredential() »](/dotnet/api/azure.identity.defaultazurecredential) de la [bibliothèque Azure Identity](https://docs.microsoft.com/dotnet/api/overview/azure/identity-readme), ce qui permet d’utiliser le même code dans différents environnements avec des options différentes pour fournir une identité. Pour plus d’informations sur l’authentification auprès du coffre de clés, consultez le [Guide du développeur](https://docs.microsoft.com/azure/key-vault/general/developers-guide#authenticate-to-key-vault-in-code).

```csharp
var keyVaultName = Environment.GetEnvironmentVariable("KEY_VAULT_NAME");
var kvUri = "https://" + keyVaultName + ".vault.azure.net";

var client = new KeyClient(new Uri(kvUri), new DefaultAzureCredential());
```

### <a name="save-a-key"></a>Enregistrer une clé

Pour cette tâche, utilisez la méthode [CreateKeyAsync](/dotnet/api/azure.security.keyvault.keys.keyclient.createkeyasync). Les paramètres de la méthode acceptent un nom de clé et le [type de clé](https://docs.microsoft.com/dotnet/api/azure.security.keyvault.keys.keytype).

```csharp
var key = await client.CreateKeyAsync("myKey", KeyType.Rsa);
```

> [!NOTE]
> Si le nom de clé existe, le code ci-dessus crée une nouvelle version de cette clé.

### <a name="retrieve-a-key"></a>Récupérer une clé

Vous pouvez maintenant récupérer la clé créée précédemment à l’aide de la méthode [GetKeyAsync](/dotnet/api/azure.security.keyvault.keys.keyclient.getkeyasync).

```csharp
var key = await client.GetKeyAsync("myKey");
```

### <a name="delete-a-key"></a>Supprimer une clé

Pour finir, nous allons supprimer et purger la clé de votre coffre de clés avec les méthodes [StartDeleteKeyAsync](/dotnet/api/azure.security.keyvault.keys.keyclient.startdeletekeyasync) et [PurgeDeletedKeyAsync](/dotnet/api/azure.security.keyvault.keys.keyclient.purgedeletedkeyasync).

```csharp
var operation = await client.StartDeleteKeyAsync("myKey");

// You only need to wait for completion if you want to purge or recover the key.
await operation.WaitForCompletionAsync();

var key = operation.Value;
await client.PurgeDeletedKeyAsync("myKey");
```

## <a name="sample-code"></a>Exemple de code

Modifiez l’application console .NET Core pour interagir avec le coffre de clés en effectuant les étapes suivantes :

- Remplacez le code dans *Program.cs* par le code suivant :

    ```csharp
    using System;
    using System.Threading.Tasks;
    using Azure.Identity;
    using Azure.Security.KeyVault.Keys;
    
    namespace key_vault_console_app
    {
        class Program
        {
            static async Task Main(string[] args)
            {
                const string keyName = "myKey";
                var keyVaultName = Environment.GetEnvironmentVariable("KEY_VAULT_NAME");
                var kvUri = $"https://{keyVaultName}.vault.azure.net";
    
                var client = new KeyClient(new Uri(kvUri), new DefaultAzureCredential());
    
                Console.Write($"Creating a key in {keyVaultName} called '{keyName}' ...");
                var createdKey = await client.CreateKeyAsync(keyName, KeyType.Rsa);
                Console.WriteLine("done.");
    
                Console.WriteLine($"Retrieving your key from {keyVaultName}.");
                var key = await client.GetKeyAsync(keyName);
                Console.WriteLine($"Your key version is '{key.Value.Properties.Version}'.");
    
                Console.Write($"Deleting your key from {keyVaultName} ...");
                var deleteOperation = await client.StartDeleteKeyAsync(keyName);
                // You only need to wait for completion if you want to purge or recover the key.
                await deleteOperation.WaitForCompletionAsync();
                Console.WriteLine("done.");

                Console.Write($"Purging your key from {keyVaultName} ...");
                await client.PurgeDeletedKeyAsync(keyName);
                Console.WriteLine(" done.");
            }
        }
    }
    ```
### <a name="test-and-verify"></a>Tester et vérifier

1.  Exécutez la commande suivante pour générer le projet :

    ```dotnetcli
    dotnet build
    ```

1. Exécutez la commande suivante pour exécuter l’application.

    ```dotnetcli
    dotnet run
    ```

1. Quand vous y êtes invité, entrez une valeur de secret. Par exemple, mySecretPassword.

    Une variante de la sortie suivante apparaît :

    ```console
    Creating a key in mykeyvault called 'myKey' ... done.
    Retrieving your key from mykeyvault.
    Your key version is '8532359bced24e4bb2525f2d2050738a'.
    Deleting your key from jl-kv ... done
    Purging your key from <your-unique-keyvault-name> ... done.   
    ```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un coffre de clés, stocké une clé et récupéré cette clé. 

Pour plus d’informations sur Key Vault et comment l’intégrer à vos applications, consultez les articles suivants :

- Lire la [vue d’ensemble Azure Key Vault](../general/overview.md)
- Lire la [Vue d’ensemble des clés](about-keys.md)
- Consultez un [Tutoriel sur l’accès à Key Vault depuis une application App Service](../general/tutorial-net-create-vault-azure-web-app.md)
- Consultez un [Tutoriel sur l’accès à Key Vault depuis une machine virtuelle](../general/tutorial-net-virtual-machine.md)
- Consulter le [Guide du développeur Azure Key Vault](../general/developers-guide.md)
- Passer en revue les [bonnes pratiques relatives à Azure Key Vault](../general/best-practices.md)

---
title: Démarrage rapide – Bibliothèque de client de secrets Azure Key Vault pour JavaScript (version 4)
description: Apprenez à créer, récupérer et supprimer des secrets d’un coffre de clés Azure à l’aide de la bibliothèque de client JavaScript
author: msmbaldwin
ms.author: mbaldwin
ms.date: 12/6/2020
ms.service: key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.custom: devx-track-js
ms.openlocfilehash: e7f861d7fb10ed5dd13b5883ba70b553daa67892
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97825175"
---
# <a name="quickstart-azure-key-vault-secret-client-library-for-javascript-version-4"></a>Démarrage rapide : Bibliothèque de client de secrets Azure Key Vault pour JavaScript (version 4)

Bien démarrer avec la bibliothèque de client de secrets Azure Key Vault pour JavaScript. [Azure Key Vault](../general/overview.md) est un service cloud qui fournit un magasin de secrets sécurisé. Vous pouvez stocker des clés, des mots de passe, des certificats et d’autres secrets en toute sécurité. Vous pouvez créer et gérer des coffres de clés Azure grâce au portail Azure. Dans ce guide de démarrage rapide, vous allez découvrir comment créer, récupérer et supprimer des secrets dans un coffre de clés Azure à l’aide de la bibliothèque de client JavaScript.

Ressources de la bibliothèque de client Key Vault :

[Documentation de référence sur les API](/javascript/api/overview/azure/key-vault-index) | [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/keyvault) | [Package (npm)](https://www.npmjs.com/package/@azure/keyvault-secrets)

Pour plus d’informations sur Key Vault et les secrets, consultez :
- [Vue d’ensemble du coffre de clés](../general/overview.md)
- [Vue d’ensemble des secrets](about-secrets.md).

## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
- [Node.js](https://nodejs.org) actuel pour votre système d’exploitation.
- [Azure CLI](/cli/azure/install-azure-cli)
- Un coffre de clés. Vous pouvez en créer un en utilisant le [portail Azure](../general/quick-create-portal.md), [Azure CLI](../general/quick-create-cli.md) ou [Azure PowerShell](../general/quick-create-powershell.md).

Ce guide de démarrage rapide suppose que vous exécutez [Azure CLI](/cli/azure/install-azure-cli).

## <a name="sign-in-to-azure"></a>Connexion à Azure

1. Exécutez la commande `login`.

    ```azurecli-interactive
    az login
    ```

    Si l’interface CLI peut ouvrir votre navigateur par défaut, elle le fait et charge une page de connexion Azure par la même occasion.

    Sinon, ouvrez une page de navigateur à l’adresse [https://aka.ms/devicelogin](https://aka.ms/devicelogin) et entrez le code d’autorisation affiché dans votre terminal.

2. Dans le navigateur, connectez-vous avec les informations d’identification de votre compte.

## <a name="create-new-nodejs-application"></a>Créer une application Node.js

Créez ensuite une application Node.js pouvant être déployée dans le Cloud. 

1. Dans une interface de commande, créez un dossier nommé `key-vault-node-app` :

```azurecli
mkdir key-vault-node-app
```

1. Accédez au répertoire *key-vault-node-app* nouvellement créé, puis exécutez la commande « init » pour initialiser le projet node :

```azurecli
cd key-vault-node-app
npm init -y
```

## <a name="install-key-vault-packages"></a>Installer des packages Key Vault

Dans la fenêtre de console, installez la [bibliothèque de secrets](https://www.npmjs.com/package/@azure/keyvault-secrets) Azure Key Vault pour Node.js.

```azurecli
npm install @azure/keyvault-secrets
```

Installez le package [azure.identity](https://www.npmjs.com/package/@azure/identity) pour l’authentification auprès d’un coffre de clés.

```azurecli
npm install @azure/identity
```

## <a name="set-environment-variables"></a>Définir des variables d’environnement

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
```cmd
export KEY_VAULT_NAME=<your-key-vault-name>
```

## <a name="grant-access-to-your-key-vault"></a>Accorder l’accès à votre coffre de clés

Créez une stratégie d’accès pour votre coffre de clés, qui accorde des autorisations de secret à votre compte d’utilisateur.

```azurecli
az keyvault set-policy --name <YourKeyVaultName> --upn user@domain.com --secret-permissions delete get list set purge
```

## <a name="code-examples"></a>Exemples de code

Les exemples de code ci-dessous vous montrent comment créer un client et définir, récupérer et supprimer un secret. 

### <a name="set-up-the-app-framework"></a>Configurer le framework d’application

1. Créer un fichier texte et l’enregistrer en tant que « index.js »

1. Ajouter les appels require pour charger des modules Azure et Node.js

1. Créer la structure du programme, y compris la gestion des exceptions de base

```javascript
const readline = require('readline');

function askQuestion(query) {
    const rl = readline.createInterface({
        input: process.stdin,
        output: process.stdout,
    });

    return new Promise(resolve => rl.question(query, ans => {
        rl.close();
        resolve(ans);
    }))
}

async function main() {
    
}

main().then(() => console.log('Done')).catch((ex) => console.log(ex.message));
```

### <a name="add-directives"></a>Ajouter des directives

Ajoutez les directives suivantes en haut de votre code :

```javascript
const { DefaultAzureCredential } = require("@azure/identity");
const { SecretClient } = require("@azure/keyvault-secrets");
```

### <a name="authenticate-and-create-a-client"></a>Authentifier et créer un client

Dans ce guide de démarrage rapide, l’utilisateur connecté est utilisé pour l’authentification auprès du coffre de clés, qui est la méthode recommandée pour le développement local. Pour les applications déployées sur Azure, l’identité managée doit être affectée à App Service ou à une machine virtuelle. Pour plus d’informations, consultez [Vue d’ensemble des identités managées](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

Dans l’exemple ci-dessous, le nom de votre coffre de clés est étendu à l’URI du coffre de clés, au format « https://\<your-key-vault-name\>.vault.azure.net ». Cet exemple utilise la classe [« DefaultAzureCredential() »](https://docs.microsoft.com/javascript/api/@azure/identity/defaultazurecredential) de la [bibliothèque Azure Identity](https://docs.microsoft.com/javascript/api/overview/azure/identity-readme), ce qui permet d’utiliser le même code dans différents environnements avec des options différentes pour fournir une identité. Pour plus d’informations sur l’authentification auprès du coffre de clés, consultez le [Guide du développeur](https://docs.microsoft.com/azure/key-vault/general/developers-guide#authenticate-to-key-vault-in-code).

Ajoutez le code suivant à la fonction « main() ».

```javascript
const keyVaultName = process.env["KEY_VAULT_NAME"];
const KVUri = "https://" + keyVaultName + ".vault.azure.net";

const credential = new DefaultAzureCredential();
const client = new SecretClient(KVUri, credential);
```

### <a name="save-a-secret"></a>Enregistrer un secret

Maintenant que votre application est authentifiée, vous pouvez placer un secret dans votre coffre avec la [méthode setSecret](/javascript/api/@azure/keyvault-secrets/secretclient?#setsecret-string--string--setsecretoptions-). Un nom pour le secret est nécessaire : dans cet exemple, nous utilisons « mySecret ».  

```javascript
await client.setSecret(secretName, secretValue);
```

### <a name="retrieve-a-secret"></a>Récupérer un secret

Vous pouvez maintenant récupérer la valeur définie précédemment avec la [méthode getSecret](/javascript/api/@azure/keyvault-secrets/secretclient?#getsecret-string--getsecretoptions-).

```javascript
const retrievedSecret = await client.getSecret(secretName);
 ```

Votre secret est désormais enregistré en tant que `retrievedSecret.value`.

### <a name="delete-a-secret"></a>supprimer un secret

Pour finir, nous allons supprimer et purger le secret de votre coffre de clés avec les méthodes [beginDeleteSecret](https://docs.microsoft.com/javascript/api/@azure/keyvault-secrets/secretclient?#beginDeleteSecret_string__BeginDeleteSecretOptions_) et [purgeDeletedSecret](https://docs.microsoft.com/javascript/api/@azure/keyvault-secrets/secretclient?#purgeDeletedSecret_string__PurgeDeletedSecretOptions_).

```javascript
const deletePoller = await client.beginDeleteSecret(secretName);
await deletePoller.pollUntilDone();
await client.purgeDeletedSecret(secretName);
```

## <a name="sample-code"></a>Exemple de code

```javascript
const { DefaultAzureCredential } = require("@azure/identity");
const { SecretClient } = require("@azure/keyvault-secrets");

const readline = require('readline');

function askQuestion(query) {
    const rl = readline.createInterface({
        input: process.stdin,
        output: process.stdout,
    });

    return new Promise(resolve => rl.question(query, ans => {
        rl.close();
        resolve(ans);
    }))
}

async function main() {

  const keyVaultName = process.env["KEY_VAULT_NAME"];
  const KVUri = "https://" + keyVaultName + ".vault.azure.net";

  const credential = new DefaultAzureCredential();
  const client = new SecretClient(KVUri, credential);

  const secretName = "mySecret";
  var secretValue = await askQuestion("Input the value of your secret > ");

  console.log("Creating a secret in " + keyVaultName + " called '" + secretName + "' with the value '" + secretValue + "` ...");
  await client.setSecret(secretName, secretValue);

  console.log("Done.");

  console.log("Forgetting your secret.");
  secretValue = "";
  console.log("Your secret is '" + secretValue + "'.");

  console.log("Retrieving your secret from " + keyVaultName + ".");

  const retrievedSecret = await client.getSecret(secretName);

  console.log("Your secret is '" + retrievedSecret.value + "'.");

  console.log("Deleting your secret from " + keyVaultName + " ...");
  const deletePoller = await client.beginDeleteSecret(secretName);
  await deletePoller.pollUntilDone();
  console.log("Done.");
  
  console.log("Purging your secret from {keyVaultName} ...");
  await client.purgeDeletedSecret(secretName);
  
}

main().then(() => console.log('Done')).catch((ex) => console.log(ex.message));

```

## <a name="test-and-verify"></a>Tester et vérifier

1. Exécutez les commandes suivantes pour exécuter l’application.

    ```azurecli
    npm install
    npm index.js
    ```

1. Quand vous y êtes invité, entrez une valeur de secret. Par exemple, mySecretPassword.

    Une variante de la sortie suivante apparaît :

    ```azurecli
    Input the value of your secret > mySecretPassword
    Creating a secret in <your-unique-keyvault-name> called 'mySecret' with the value 'mySecretPassword' ... done.
    Forgetting your secret.
    Your secret is ''.
    Retrieving your secret from <your-unique-keyvault-name>.
    Your secret is 'mySecretPassword'.
    Deleting your secret from <your-unique-keyvault-name> ... done.  
    Purging your secret from <your-unique-keyvault-name> ... done.   
    ```


## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un coffre de clés, stocké un secret et récupéré ce secret. Pour en savoir plus sur Key Vault et sur la manière de l’intégrer à vos applications, consultez les articles ci-dessous.

- Lire la [vue d’ensemble Azure Key Vault](../general/overview.md)
- Lire une [présentation des secrets Azure Key Vault](about-secrets.md).
- Découvrir comment [Sécuriser l’accès à un coffre de clés](../general/secure-your-key-vault.md)
- Consulter le [Guide du développeur Azure Key Vault](../general/developers-guide.md)
- Passer en revue les [bonnes pratiques relatives à Azure Key Vault](../general/best-practices.md)

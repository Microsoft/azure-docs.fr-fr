---
title: Démarrage rapide – Bibliothèque de client de clés Azure Key Vault pour JavaScript (version 4)
description: Apprenez à créer, récupérer et supprimer des clés d’un coffre de clés Azure à l’aide de la bibliothèque de client JavaScript
author: msmbaldwin
ms.author: mbaldwin
ms.date: 12/6/2020
ms.service: key-vault
ms.subservice: keys
ms.topic: quickstart
ms.custom: devx-track-js
ms.openlocfilehash: 1b23fa9f9cbf7b385a04835149b5d53cc42351eb
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97825886"
---
# <a name="quickstart-azure-key-vault-key-client-library-for-javascript-version-4"></a>Démarrage rapide : Bibliothèque de client de clés Azure Key Vault pour JavaScript (version 4)

Commencez à utiliser la bibliothèque de client de clés Azure Key Vault pour JavaScript. [Azure Key Vault](../general/overview.md) est un service cloud qui fournit un magasin de clés de chiffrement sécurisé. Vous pouvez stocker des clés, des mots de passe, des certificats et d’autres secrets en toute sécurité. Vous pouvez créer et gérer des coffres de clés Azure grâce au portail Azure. Dans ce guide de démarrage rapide, vous allez découvrir comment créer, récupérer et supprimer des clés dans un coffre de clés Azure en utilisant la bibliothèque de client de clés JavaScript.

Ressources de la bibliothèque de client Key Vault :

[Documentation de référence sur les API](/javascript/api/overview/azure/key-vault-index) | [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/keyvault) | [Package (npm)](https://www.npmjs.com/package/@azure/keyvault-keys)

Pour plus d’informations sur Key Vault et les clés, consultez :
- [Vue d’ensemble du coffre de clés](../general/overview.md)
- [Vue d’ensemble des clés](about-keys.md)

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

À partir de la fenêtre de la console, installez la [bibliothèque de clés](https://www.npmjs.com/package/@azure/keyvault-keys) Azure Key Vault pour Node.js.

```azurecli
npm install @azure/keyvault-keys
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

Créez une stratégie d’accès pour votre coffre de clés, qui accorde des autorisations de clé à votre compte d’utilisateur.

```azurecli
az keyvault set-policy --name <YourKeyVaultName> --upn user@domain.com --key-permissions delete get list create purge
```

## <a name="code-examples"></a>Exemples de code

Les exemples de code ci-dessous vous montrent comment créer un client et comment définir, récupérer et supprimer une clé. 

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
const { KeyClient } = require("@azure/keyvault-keys");
```

### <a name="authenticate-and-create-a-client"></a>Authentifier et créer un client

Dans ce guide de démarrage rapide, l’utilisateur connecté est utilisé pour l’authentification auprès du coffre de clés, qui est la méthode recommandée pour le développement local. Pour les applications déployées sur Azure, l’identité managée doit être affectée à App Service ou à une machine virtuelle. Pour plus d’informations, consultez [Vue d’ensemble des identités managées](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

Dans l’exemple ci-dessous, le nom de votre coffre de clés est étendu à l’URI du coffre de clés, au format « https://\<your-key-vault-name\>.vault.azure.net ». Cet exemple utilise la classe [« DefaultAzureCredential() »](https://docs.microsoft.com/javascript/api/@azure/identity/defaultazurecredential) de la [bibliothèque Azure Identity](https://docs.microsoft.com/javascript/api/overview/azure/identity-readme), ce qui permet d’utiliser le même code dans différents environnements avec des options différentes pour fournir une identité. Pour plus d’informations sur l’authentification auprès du coffre de clés, consultez le [Guide du développeur](https://docs.microsoft.com/azure/key-vault/general/developers-guide#authenticate-to-key-vault-in-code).

Ajoutez le code suivant à la fonction « main() ».

```javascript
const keyVaultName = process.env["KEY_VAULT_NAME"];
const KVUri = "https://" + keyVaultName + ".vault.azure.net";

const credential = new DefaultAzureCredential();
const client = new KeyClient(KVUri, credential);
```

### <a name="save-a-key"></a>Enregistrer une clé

À présent que votre application est authentifiée, vous pouvez placer une clé dans votre coffre de clés à l’aide de la [méthode createKey](/javascript/api/@azure/keyvault-keys/keyclient?#createKey_string__KeyType__CreateKeyOptions_). Les paramètres de la méthode acceptent un nom de clé et le [type clé](https://docs.microsoft.com/javascript/api/@azure/keyvault-keys/keytype).

```javascript
await client.createKey(keyName, keyType);
```

### <a name="retrieve-a-key"></a>Récupérer une clé

Vous pouvez maintenant récupérer la valeur définie précédemment avec la [méthode getKey](/javascript/api/@azure/keyvault-keys/keyclient?#getKey_string__GetKeyOptions_).

```javascript
const retrievedKey = await client.getKey(keyName);
 ```

### <a name="delete-a-key"></a>Supprimer une clé

Pour finir, nous allons supprimer et purger la clé de votre coffre de clés avec les méthodes [beginDeleteKey](https://docs.microsoft.com/javascript/api/@azure/keyvault-keys/keyclient?#beginDeleteKey_string__BeginDeleteKeyOptions_) et [purgeDeletedKey](https://docs.microsoft.com/javascript/api/@azure/keyvault-keys/keyclient?#purgeDeletedKey_string__PurgeDeletedKeyOptions_).

```javascript
const deletePoller = await client.beginDeleteKey(keyName);
await deletePoller.pollUntilDone();
await client.purgeDeletedKey(keyName);
```

## <a name="sample-code"></a>Exemple de code

```javascript
const { DefaultAzureCredential } = require("@azure/identity");
const { KeyClient } = require("@azure/keyvault-keys");

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
  const client = new KeyClient(KVUri, credential);

  const keyName = "myKey";
  
  console.log("Creating a key in " + keyVaultName + " called '" + keyName + "` ...");
  await client.createKey(keyName, "RSA");

  console.log("Done.");

  console.log("Retrieving your key from " + keyVaultName + ".");

  const retrievedKey = await client.getKey(keyName);

  console.log("Your key version is '" + retrievedKey.properties.version + "'.");

  console.log("Deleting your key from " + keyVaultName + " ...");
  const deletePoller = await client.beginDeleteKey(keyName);
  await deletePoller.pollUntilDone();
  console.log("Done.");
  
  console.log("Purging your key from {keyVaultName} ...");
  await client.purgeDeletedKey(keyName);
  
}

main().then(() => console.log('Done')).catch((ex) => console.log(ex.message));

```

## <a name="test-and-verify"></a>Tester et vérifier

Exécutez les commandes suivantes pour exécuter l’application.

```azurecli
npm install
npm index.js
```

Une variante de la sortie suivante apparaît :

```azurecli
Creating a key in <your-unique-keyvault-name> called 'myKey' ... done.
Retrieving your key from mykeyvault.
Your key version is '8532359bced24e4bb2525f2d2050738a'.
Deleting your key from <your-unique-keyvault-name> ... done.  
Purging your key from <your-unique-keyvault-name> ... done.   
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un coffre de clés, stocké une clé et récupéré cette clé. Pour en savoir plus sur Key Vault et sur la manière de l’intégrer à vos applications, consultez les articles ci-dessous.

- Lire la [vue d’ensemble Azure Key Vault](../general/overview.md)
- Lire une [présentation des clés Azure Key Vault](about-keys.md)
- Découvrir comment [Sécuriser l’accès à un coffre de clés](../general/secure-your-key-vault.md)
- Consulter le [Guide du développeur Azure Key Vault](../general/developers-guide.md)
- Passer en revue les [bonnes pratiques relatives à Azure Key Vault](../general/best-practices.md)

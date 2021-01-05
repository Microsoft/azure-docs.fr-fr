---
title: Démarrage rapide – Bibliothèque de client de certificats Azure Key Vault pour JavaScript (version 4)
description: Découvrir comment créer, récupérer et supprimer des certificats dans un coffre de clés Azure en utilisant la bibliothèque de client JavaScript
author: msmbaldwin
ms.author: mbaldwin
ms.date: 12/6/2020
ms.service: key-vault
ms.subservice: certificates
ms.topic: quickstart
ms.custom: devx-track-js
ms.openlocfilehash: a96c3a2dc3b013f79799f20595bf5b5f575b363b
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97826089"
---
# <a name="quickstart-azure-key-vault-certificate-client-library-for-javascript-version-4"></a>Démarrage rapide : Bibliothèque de client de certificats Azure Key Vault pour JavaScript (version 4)

Démarrez avec la bibliothèque de client de certificats Azure Key Vault pour JavaScript. [Azure Key Vault](../general/overview.md) est un service cloud qui fournit un magasin de certificats sécurisé. Vous pouvez stocker des clés, des mots de passe, des certificats et d’autres secrets en toute sécurité. Vous pouvez créer et gérer des coffres de clés Azure grâce au portail Azure. Dans ce guide de démarrage rapide, vous allez découvrir comment créer, récupérer et supprimer des certificats dans un coffre de clés Azure en utilisant la bibliothèque de client JavaScript.

Ressources de la bibliothèque de client Key Vault :

[Documentation de référence sur les API](/javascript/api/overview/azure/key-vault-index) | [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/keyvault) | [Package (npm)](https://www.npmjs.com/package/@azure/keyvault-certificates)

Pour plus d’informations sur Key Vault et les certificats, consultez :
- [Vue d’ensemble du coffre de clés](../general/overview.md)
- [Vue d’ensemble des certificats](about-certificates.md)

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

À partir de la fenêtre de la console, installez la [bibliothèque de certificats](https://www.npmjs.com/package/@azure/keyvault-certificates) Azure Key Vault pour Node.js.

```azurecli
npm install @azure/keyvault-certificates
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

Créez une stratégie d’accès pour votre coffre de clés, qui accorde des autorisations de certificat à votre compte d’utilisateur.

```azurecli
az keyvault set-policy --name <YourKeyVaultName> --upn user@domain.com --certificate-permissions delete get list create purge
```

## <a name="code-examples"></a>Exemples de code

Les exemples de code ci-dessous vous montrent comment créer un client et comment définir, récupérer et supprimer un certificat. 

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
const { CertificateClient } = require("@azure/keyvault-certificates");
```

### <a name="authenticate-and-create-a-client"></a>Authentifier et créer un client

Dans ce guide de démarrage rapide, l’utilisateur connecté est utilisé pour l’authentification auprès du coffre de clés, qui est la méthode recommandée pour le développement local. Pour les applications déployées sur Azure, l’identité managée doit être affectée à App Service ou à une machine virtuelle. Pour plus d’informations, consultez [Vue d’ensemble des identités managées](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

Dans l’exemple ci-dessous, le nom de votre coffre de clés est étendu à l’URI du coffre de clés, au format « https://\<your-key-vault-name\>.vault.azure.net ». Cet exemple utilise la classe [« DefaultAzureCredential() »](https://docs.microsoft.com/javascript/api/@azure/identity/defaultazurecredential) de la [bibliothèque Azure Identity](https://docs.microsoft.com/javascript/api/overview/azure/identity-readme), ce qui permet d’utiliser le même code dans différents environnements avec des options différentes pour fournir une identité. Pour plus d’informations sur l’authentification auprès du coffre de clés, consultez le [Guide du développeur](https://docs.microsoft.com/azure/key-vault/general/developers-guide#authenticate-to-key-vault-in-code).

Ajoutez le code suivant à la fonction « main() ».

```javascript
const keyVaultName = process.env["KEY_VAULT_NAME"];
const KVUri = "https://" + keyVaultName + ".vault.azure.net";

const credential = new DefaultAzureCredential();
const client = new Certificate(KVUri, credential);
```

### <a name="save-a-certificate"></a>Enregistrer un certificat

Maintenant que votre application est authentifiée, vous pouvez placer un certificat dans votre coffre de clés à l’aide de la méthode [beginCreateCertificate](/javascript/api/@azure/keyvault-certificates/certificateclient?#beginCreateCertificate_string__CertificatePolicy__BeginCreateCertificateOptions_). Un nom de certificat et la stratégie de certificat [CertificatePolicy](https://docs.microsoft.com/javascript/api/@azure/keyvault-certificates/certificatepolicy) avec [CertificatePolicyProperties](https://docs.microsoft.com/javascript/api/@azure/keyvault-certificates/certificatepolicyproperties) sont nécessaires.

```javascript
const certificatePolicy = {
  issuerName: "Self",
  subject: "cn=MyCert"
};
const createPoller = await client.beginCreateCertificate(certificateName, certificatePolicy);
const certificate = await poller.pollUntilDone();
```

> [!NOTE]
> Si le nom du certificat existe, le code ci-dessus crée une nouvelle version de ce certificat.
### <a name="retrieve-a-certificate"></a>Récupérer un certificat

Vous pouvez désormais récupérer la valeur définie précédemment avec la méthode [getCertificate](/javascript/api/@azure/keyvault-certificates/certificateclient?#getCertificate_string__GetCertificateOption).

```javascript
const retrievedCertificate = await client.getCertificate(certificateName);
 ```

### <a name="delete-a-certificate"></a>Supprimer un certificat

Pour finir, nous allons supprimer et purger le certificat de votre coffre de clés avec les méthodes [beginDeleteCertificate]https://docs.microsoft.com/javascript/api/@azure/keyvault-certificates/certificateclient?#beginDeleteCertificate_string__BeginDeleteCertificateOptions_) et [purgeDeletedCertificate](https://docs.microsoft.com/javascript/api/@azure/keyvault-certificates/certificateclient?#purgeDeletedCertificate_string__PurgeDeletedCertificateOptions_).

```javascript
const deletePoller = await client.beginDeleteCertificate(certificateName);
await deletePoller.pollUntilDone();
await client.purgeDeletedCertificate(certificateName);
```

## <a name="sample-code"></a>Exemple de code

```javascript
const { DefaultAzureCredential } = require("@azure/identity");
const { CertificateClient } = require("@azure/keyvault-certificates");

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

  const string certificateName = "myCertificate";
  const keyVaultName = process.env["KEY_VAULT_NAME"];
  const KVUri = "https://" + keyVaultName + ".vault.azure.net";

  const credential = new DefaultAzureCredential();
  const client = new CertificateClient(KVUri, credential);

  console.log("Creating a certificate in " + keyVaultName + " called '" + certificateName +  "` ...");
  const certificatePolicy = {
  issuerName: "Self",
  subject: "cn=MyCert"
  };
  const createPoller = await client.beginCreateCertificate(certificateName, certificatePolicy);
  const certificate = await poller.pollUntilDone();

  console.log("Done.");

  console.log("Retrieving your certificate from " + keyVaultName + ".");

  const retrievedCertificate = await client.getCertificate(certificateName);

  console.log("Your certificate version is '" + retrievedCertificate.properties.version + "'.");

  console.log("Deleting your certificate from " + keyVaultName + " ...");
  const deletePoller = await client.beginDeleteCertificate(certificateName);
  await deletePoller.pollUntilDone();
  console.log("Done.");
  
  console.log("Purging your certificate from {keyVaultName} ...");
  await client.purgeDeletedCertificate(certificateName);
  
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
Creating a certificate in mykeyvault called 'myCertificate' ... done.
Retrieving your certificate from mykeyvault.
Your certificate version is '8532359bced24e4bb2525f2d2050738a'.
Deleting your certificate from mykeyvault ... done
Purging your certificate from mykeyvault ... done 
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un coffre de clés, stocké un certificat et récupéré ce certificat. Pour en savoir plus sur Key Vault et sur la manière de l’intégrer à vos applications, consultez les articles ci-dessous.

- Lire la [vue d’ensemble Azure Key Vault](../general/overview.md)
- Lire la [vue d’ensemble des certificats](about-certificates.md)
- Consultez un [Tutoriel sur l’accès à Key Vault depuis une application App Service](../general/tutorial-net-create-vault-azure-web-app.md)
- Consultez un [Tutoriel sur l’accès à Key Vault depuis une machine virtuelle](../general/tutorial-net-virtual-machine.md)
- Consulter le [Guide du développeur Azure Key Vault](../general/developers-guide.md)
- Passer en revue les [bonnes pratiques relatives à Azure Key Vault](../general/best-practices.md)

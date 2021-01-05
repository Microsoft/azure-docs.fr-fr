---
title: Démarrage rapide – Bibliothèque cliente Python Azure Key Vault – Gérer les secrets
description: Découvrez comment créer, récupérer et supprimer des secrets d’un coffre de clés Azure à l’aide de la bibliothèque cliente Python
author: msmbaldwin
ms.author: mbaldwin
ms.date: 09/03/2020
ms.service: key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.custom: devx-track-python, devx-track-azurecli
ms.openlocfilehash: 68c58769302bbefd29b483a8fda225201d242dd9
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96483642"
---
# <a name="quickstart-azure-key-vault-secret-client-library-for-python"></a>Démarrage rapide : Bibliothèque de client de secrets Azure Key Vault pour Python

Bien démarrer avec la bibliothèque de client de secrets Azure Key Vault pour Python. Suivez les étapes ci-dessous pour installer le package et tester un exemple de code relatif à des tâches de base. En stockant des secrets à l’aide de Key Vault, vous évitez de les stocker dans votre code, ce qui renforce la sécurité de votre application.

[Documentation de référence de l’API](/python/api/overview/azure/keyvault-secrets-readme) | [Code source bibliothèqueC](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/keyvault/azure-keyvault-secrets) | [Package (Index package Python)](https://pypi.org/project/azure-keyvault-secrets/)

## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
- [Python 2.7+ ou 3.5.3+](/azure/developer/python/configure-local-development-environment)
- [Azure CLI](/cli/azure/install-azure-cli)

Ce guide de démarrage rapide suppose que vous exécutez [Azure CLI](/cli/azure/install-azure-cli) dans une fenêtre de terminal Linux.


## <a name="set-up-your-local-environment"></a>Configurer votre environnement local
Ce guide de démarrage rapide utilise la bibliothèque Azure Identity avec Azure CLI pour authentifier l’utilisateur auprès des services Azure. Les développeurs peuvent également utiliser Visual Studio ou Visual Studio Code pour authentifier leurs appels. Pour plus d’informations, consultez [Authentifier le client avec la bibliothèque de client Azure Identity](/java/api/overview/azure/identity-readme).

### <a name="sign-in-to-azure"></a>Connexion à Azure

1. Exécutez la commande `login`.

    ```azurecli-interactive
    az login
    ```

    Si l’interface CLI peut ouvrir votre navigateur par défaut, elle le fait et charge une page de connexion Azure par la même occasion.

    Sinon, ouvrez une page de navigateur à l’adresse [https://aka.ms/devicelogin](https://aka.ms/devicelogin) et entrez le code d’autorisation affiché dans votre terminal.

2. Dans le navigateur, connectez-vous avec les informations d’identification de votre compte.

### <a name="install-the-packages"></a>Installer les packages

1. Dans un terminal ou une invite de commandes, créez un dossier de projet approprié, puis créez et activez un environnement virtuel Python comme décrit dans [Utiliser des environnements virtuels Python](/azure/developer/python/configure-local-development-environment?tabs=cmd#use-python-virtual-environments).

1. Installez la bibliothèque d’identités Azure Active Directory :

    ```terminal
    pip install azure-identity
    ```


1. Installez la bibliothèque de secrets Key Vault :

    ```terminal
    pip install azure-keyvault-secrets
    ```

### <a name="create-a-resource-group-and-key-vault"></a>Créer un groupe de ressources et un coffre de clés

[!INCLUDE [Create a resource group and key vault](../../../includes/key-vault-python-qs-rg-kv-creation.md)]

### <a name="grant-access-to-your-key-vault"></a>Accorder l’accès à votre coffre de clés

Créez une stratégie d’accès pour votre coffre de clés qui accorde une autorisation de secret à votre compte d’utilisateur.

```console
az keyvault set-policy --name <YourKeyVaultName> --upn user@domain.com --secret-permissions delete get list set
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
```cmd
export KEY_VAULT_NAME=<your-key-vault-name>
```

## <a name="create-the-sample-code"></a>Créer l’exemple de code

La bibliothèque de client de secrets Azure Key Vault pour Python vous permet de gérer des secrets. L’exemple de code suivant vous montre comment créer un client et définir, récupérer et supprimer un secret.

Créez un fichier nommé *kv_secrets.py* qui contient ce code.

```python
import os
import cmd
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

keyVaultName = os.environ["KEY_VAULT_NAME"]
KVUri = f"https://{keyVaultName}.vault.azure.net"

credential = DefaultAzureCredential()
client = SecretClient(vault_url=KVUri, credential=credential)

secretName = input("Input a name for your secret > ")
secretValue = input("Input a value for your secret > ")

print(f"Creating a secret in {keyVaultName} called '{secretName}' with the value '{secretValue}' ...")

client.set_secret(secretName, secretValue)

print(" done.")

print(f"Retrieving your secret from {keyVaultName}.")

retrieved_secret = client.get_secret(secretName)

print(f"Your secret is '{retrieved_secret.value}'.")
print(f"Deleting your secret from {keyVaultName} ...")

poller = client.begin_delete_secret(secretName)
deleted_secret = poller.result()

print(" done.")
```

## <a name="run-the-code"></a>Exécuter le code

Vérifiez que le code de la section précédente se trouve dans un fichier nommé *kv_secrets.py*. Exécutez ensuite le code avec la commande suivante :

```terminal
python kv_secrets.py
```

- Si vous rencontrez des erreurs d’autorisation, vérifiez que vous avez exécuté la commande [`az keyvault set-policy`](#grant-access-to-your-key-vault).
- Le fait de réexécuter le code avec le même nom de secret a pour effet de générer l’erreur « Le secret (en conflit) <name> est actuellement à l’état supprimé mais récupérable. » Utilisez un nom de secret différent.

## <a name="code-details"></a>Détails du code

### <a name="authenticate-and-create-a-client"></a>Authentifier et créer un client

Dans ce guide de démarrage rapide, l’utilisateur connecté est utilisé pour l’authentification auprès du coffre de clés, qui est la méthode recommandée pour le développement local. Pour les applications déployées sur Azure, l’identité managée doit être affectée à App Service ou à une machine virtuelle. Pour plus d’informations, consultez [Vue d’ensemble des identités managées](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

Dans l’exemple ci-dessous, le nom de votre coffre de clés est étendu à l’URI du coffre de clés, au format « https://\<your-key-vault-name\>.vault.azure.net ». Cet exemple utilise la classe [« DefaultAzureCredential() »](https://docs.microsoft.com/python/api/azure-identity/azure.identity.defaultazurecredential), qui permet d’utiliser le même code dans différents environnements avec des options différentes pour fournir une identité. Pour plus d’informations, consultez [Authentification des informations d’identification Azure par défaut](https://docs.microsoft.com/python/api/overview/azure/identity-readme). 

```python
credential = DefaultAzureCredential()
client = SecretClient(vault_url=KVUri, credential=credential)
```

### <a name="save-a-secret"></a>Enregistrer un secret

Une fois que vous avez obtenu l’objet client pour le coffre de clés, vous pouvez stocker un secret à l’aide de la méthode [set_secret](/python/api/azure-keyvault-secrets/azure.keyvault.secrets.secretclient?#set-secret-name--value----kwargs-) : 

```python
client.set_secret(secretName, secretValue)
```

L’appel de `set_secret` génère un appel à l’API REST Azure pour le coffre de clés.

Au moment de traiter la demande, Azure authentifie l’identité de l’appelant (le principal du service) à partir de l’objet d’informations d’identification que vous avez fourni au client.

### <a name="retrieve-a-secret"></a>Récupérer un secret

Pour lire un secret à partir de Key Vault, utilisez la méthode [get_secret](/python/api/azure-keyvault-secrets/azure.keyvault.secrets.secretclient?#get-secret-name--version-none----kwargs-) :

```python
retrieved_secret = client.get_secret(secretName)
 ```

La valeur du secret est contenue dans `retrieved_secret.value`.

Vous pouvez aussi récupérer un secret avec la commande Azure CLI [az keyvault secret show](/cli/azure/keyvault/secret?#az-keyvault-secret-show).

### <a name="delete-a-secret"></a>Supprimer un secret

Pour supprimer un secret, utilisez la méthode [begin_delete_secret](/python/api/azure-keyvault-secrets/azure.keyvault.secrets.secretclient?#begin-delete-secret-name----kwargs-) :

```python
poller = client.begin_delete_secret(secretName)
deleted_secret = poller.result()
```

La méthode `begin_delete_secret` est asynchrone et retourne un objet observateur. L’appel de la méthode `result` de l’observateur attend la fin de son exécution.

Vous pouvez vérifier que le secret a été supprimé avec la commande Azure CLI [az keyvault secret show](/cli/azure/keyvault/secret?#az-keyvault-secret-show).

Une fois supprimé, un secret reste à l’état supprimé mais récupérable pour un temps. Si vous réexécutez le code, utilisez un nom de secret différent.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous voulez aussi tenter une expérience avec des [certificats](../certificates/quick-create-python.md) et des [clés](../keys/quick-create-python.md), vous pouvez réutiliser le coffre de clés créé dans cet article.

Sinon, quand vous en avez terminé avec les ressources créées dans cet article, utilisez la commande suivante pour supprimer le groupe de ressources et toutes les ressources qu’il contient :

```azurecli
az group delete --resource-group KeyVault-PythonQS-rg
```

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble Azure Key Vault](../general/overview.md)
- [Sécuriser l’accès à un coffre de clés](../general/secure-your-key-vault.md)
- [Guide du développeur Azure Key Vault](../general/developers-guide.md)
- [Bonnes pratiques relatives à Azure Key Vault](../general/best-practices.md)
- [S’authentifier auprès de Key Vault](../general/authentication.md)

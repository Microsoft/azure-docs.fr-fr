---
title: Tutoriel - Utiliser Azure Key Vault avec une machine virtuelle dans Python | Microsoft Docs
description: Dans ce tutoriel, vous allez configurer sur une machine virtuelle une application Python pour lire un secret dans votre coffre de clés.
services: key-vault
author: msmbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 07/20/2020
ms.author: mbaldwin
ms.custom: mvc, devx-track-python, devx-track-azurecli
ms.openlocfilehash: ae62bf353f8a92c4408d4a38a91771ad60a13107
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93285300"
---
# <a name="tutorial-use-azure-key-vault-with-a-virtual-machine-in-python"></a>Tutoriel : Utiliser Azure Key Vault avec une machine virtuelle dans Python

Azure Key Vault vous permet de protéger les clés, secrets et certificats, notamment les clés API et les chaînes de connexion de base de données.

Dans ce tutoriel, vous allez configurer une application Python pour lire des informations dans Azure Key Vault à l’aide d’identités managées pour les ressources Azure. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Création d’un coffre de clés
> * Stocker un secret dans Key Vault
> * Créer une machine virtuelle Azure Linux
> * Activer une [identité managée](../../active-directory/managed-identities-azure-resources/overview.md) pour la machine virtuelle
> * Octroyer les autorisations nécessaires à l’application console pour lire les données provenant de Key Vault
> * Récupérer un secret à partir de Key Vault

Avant de continuer, lisez les [concepts de base de Key Vault](basic-concepts.md). 

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="prerequisites"></a>Prérequis

Pour Windows, Mac et Linux :
  * [Git](https://git-scm.com/downloads)
  * Ce didacticiel nécessite que vous exécutiez l’interface Azure CLI localement. Vous devez avoir installé Azure CLI 2.0.4 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau l’interface CLI, consultez l’article [Installation d’Azure CLI 2.0](/cli/azure/install-azure-cli).

## <a name="log-in-to-azure"></a>Connexion à Azure

Pour vous connecter à Azure à l’aide de l’interface CLI, entrez :

```azurecli
az login
```

## <a name="create-a-resource-group-and-key-vault"></a>Créer un groupe de ressources et un coffre de clés

[!INCLUDE [Create a resource group and key vault](../../../includes/key-vault-rg-kv-creation.md)]

## <a name="populate-your-key-vault-with-a-secret"></a>Remplir votre coffre de clés avec un secret

[!INCLUDE [Create a secret](../../../includes/key-vault-create-secret.md)]

## <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

Créez une machine virtuelle nommée **myVM** à l’aide de l’une des méthodes suivantes :

| Linux | Windows |
|--|--|
| [Azure CLI](../../virtual-machines/linux/quick-create-cli.md) | [Azure CLI](../../virtual-machines/windows/quick-create-cli.md) |
| [PowerShell](../../virtual-machines/linux/quick-create-powershell.md) | [PowerShell](../../virtual-machines/windows/quick-create-powershell.md) |
| [Azure portal](../../virtual-machines/linux/quick-create-portal.md) | [Le portail Azure](../../virtual-machines/windows/quick-create-portal.md) |

Pour créer une machine virtuelle Linux à l’aide de l’interface Azure CLI, utilisez la commande [az vm create](/cli/azure/vm).  L’exemple suivant ajoute un compte d’utilisateur nommé *azureuser*. Le paramètre `--generate-ssh-keys` permet de générer automatiquement une clé SSH et de la placer dans l’emplacement de clé par défaut ( *~/.ssh* ). 

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```

Notez la valeur de `publicIpAddress` dans la sortie.

## <a name="assign-an-identity-to-the-vm"></a>Affecter une identité à la machine virtuelle

Créez une identité attribuée par le système pour la machine virtuelle avec la commande Azure CLI [az vm identity assign](/cli/azure/vm/identity?view=azure-cli-latest#az-vm-identity-assign) :

```azurecli
az vm identity assign --name "myVM" --resource-group "myResourceGroup"
```

Notez l’identité affectée par le système qui est affichée dans le code suivant. La sortie de la commande ci-dessus doit être la suivante : 

```output
{
  "systemAssignedIdentity": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "userAssignedIdentities": {}
}
```

## <a name="assign-permissions-to-the-vm-identity"></a>Attribuer des autorisations à l’identité de machine virtuelle

À présent, vous pouvez attribuer à votre coffre de clés les autorisations d’identité créées précédemment en exécutant la commande suivante :

```azurecli
az keyvault set-policy --name "<your-unique-keyvault-name>" --object-id "<systemAssignedIdentity>" --secret-permissions get list
```

## <a name="log-in-to-the-vm"></a>Se connecter à la machine virtuelle

Pour vous connecter à la machine virtuelle, suivez les instructions de [Se connecter à une machine virtuelle Linux dans Azure](../../virtual-machines/linux/login-using-aad.md) ou [Connexion à une machine virtuelle Azure exécutant Windows](../../virtual-machines/windows/connect-logon.md).


Pour vous connecter à une machine virtuelle Linux, vous pouvez utiliser la commande ssh avec le paramètre « <publicIpAddress> » indiqué à l’étape [Créer une machine virtuelle](#create-a-virtual-machine) :

```terminal
ssh azureuser@<PublicIpAddress>
```

## <a name="install-python-libraries-on-the-vm"></a>Installer les bibliothèques Python sur la machine virtuelle

Sur la machine virtuelle, installez les deux bibliothèques Python que nous utiliserons dans notre script Python : `azure-keyvault-secrets` et `azure.identity`.  

Sur une machine virtuelle Linux, par exemple, vous pouvez les installer à l’aide de `pip3` :

```bash
pip3 install azure-keyvault-secrets

pip3 install azure.identity
```

## <a name="create-and-edit-the-sample-python-script"></a>Créer et modifier l’exemple de script Python

Sur la machine virtuelle, créez un fichier Python nommé **sample.py**. Modifiez le fichier pour y inclure le code suivant en remplaçant <your-unique-keyvault-name> par le nom de votre coffre de clés :

```python
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

keyVaultName = "<your-unique-keyvault-name>"
KVUri = f"https://{keyVaultName}.vault.azure.net"
secretName = "mySecret"

credential = DefaultAzureCredential()
client = SecretClient(vault_url=KVUri, credential=credential)
retrieved_secret = client.get_secret(secretName)

print(f"The value of secret '{secretName}' in '{keyVaultName}' is: '{retrieved_secret.value}'")
```

## <a name="run-the-sample-python-app"></a>Exécuter l’exemple d’application Python

Enfin, exécutez **sample.py**. Si tout s’est bien passé, la valeur de votre secret doit être retournée :

```bash
python3 sample.py

The value of secret 'mySecret' in '<your-unique-keyvault-name>' is: 'Success!'
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand ils ne sont plus nécessaires, supprimez la machine virtuelle et le coffre de clés.  Vous pouvez le faire rapidement en supprimant simplement le groupe de ressources auquel ils appartiennent :

```azurecli
az group delete -g myResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

[API REST Azure Key Vault](/rest/api/keyvault/)
---
title: Accéder au Stockage Blob Azure en utilisant Azure Databricks et Azure Key Vault
description: Dans ce tutoriel, vous allez découvrir comment accéder à Stockage Blob Azure à partir d’Azure Databricks en utilisant un secret stocké dans Azure Key Vault
author: msmbaldwin
ms.author: mbaldwin
ms.service: key-vault
ms.topic: tutorial
ms.date: 06/16/2020
ms.openlocfilehash: ba80d78cbc7d34b1496daffbd489a1d0dbfed8b4
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93285667"
---
# <a name="tutorial-access-azure-blob-storage-using-azure-databricks-and-azure-key-vault"></a>Tutoriel : Accéder au Stockage Blob Azure en utilisant Azure Databricks et Azure Key Vault

Dans ce tutoriel, vous allez découvrir comment accéder à Stockage Blob Azure à partir d’Azure Databricks en utilisant un secret stocké dans Azure Key Vault. 

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un compte de stockage et un conteneur d’objets blob avec Azure CLI
> * Créer un coffre de clés et définir un secret
> * Créer un espace de travail Azure Databricks et ajouter une étendue de secrets Key Vault
> * Accéder à votre conteneur d’objets blob à partir de l’espace de travail Azure Databricks

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

Avant de commencer ce tutoriel, installez l’[interface de ligne de commande Azure](/cli/azure/install-azure-cli-windows?view=azure-cli-latest).

## <a name="create-a-storage-account-and-blob-container-with-azure-cli"></a>Créer un compte de stockage et un conteneur d’objets blob avec Azure CLI

Pour pouvoir utiliser des objets blob, vous devez d’abord créer un compte de stockage universel. Si vous n’avez pas de [groupe de ressources](/cli/azure/group?view=azure-cli-latest#az-group-create), créez-en un avant d’exécuter la commande. La commande suivante crée et affiche les métadonnées du conteneur de stockage. Copiez la valeur d’ **ID**.

```azurecli
az storage account create --name contosoblobstorage5 --resource-group contosoResourceGroup --location eastus --sku Standard_ZRS --encryption-services blob
```

![Sortie de la commande ci-dessus dans la console. L’ID est mis en évidence en vert pour l’utilisateur final.](../media/databricks-command-output-1.png)

Avant de pouvoir créer un conteneur dans lequel charger l’objet blob, vous devez vous attribuer le rôle [Contributeur aux données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor). Pour cet exemple, le rôle est attribué au compte de stockage que vous avez créé précédemment.

```azurecli
az role assignment create --role "Storage Blob Data Contributor" --assignee t-trtr@microsoft.com --scope "/subscriptions/885e24c8-7a36-4217-b8c9-eed31e110504/resourceGroups/contosoResourceGroup5/providers/Microsoft.Storage/storageAccounts/contosoblobstorage5
```

Dès lors que vous avez affecté le rôle au compte de stockage, vous pouvez créer un conteneur pour votre objet blob.

```azurecli
az storage container create --account-name contosoblobstorage5 --name contosocontainer5 --auth-mode login
```

Une fois le conteneur créé, vous pouvez charger un objet blob (fichier de votre choix) dans le conteneur. Dans cet exemple, un fichier .txt nommé helloworld est chargé.

```azurecli
az storage blob upload --account-name contosoblobstorage5 --container-name contosocontainer5 --name helloworld --file helloworld.txt --auth-mode login
```

Vérifiez qu’il se trouve dans le conteneur en listant les objets blob contenus dans ce dernier.

```azurecli
az storage blob list --account-name contosoblobstorage5 --container-name contosocontainer5 --output table --auth-mode login
```

![Sortie de la commande ci-dessus dans la console. Elle présente le fichier qui vient d’être stocké dans le conteneur.](../media/databricks-command-output-2.png)

Obtenez la valeur de **key1** de votre conteneur de stockage à l’aide de la commande suivante. Copiez la valeur.

```azurecli
az storage account keys list -g contosoResourceGroup5 -n contosoblobstorage5
```

![Sortie de la commande ci-dessus dans la console. La valeur de key1 est mise en évidence dans un cadre vert.](../media/databricks-command-output-3.png)

## <a name="create-a-key-vault-and-set-a-secret"></a>Créer un coffre de clés et définir un secret

Vous allez créer un coffre de clés à l’aide de la commande suivante. Cette commande présente aussi les métadonnées du coffre de clés. Copiez la valeur d’ **ID** et de **vaultUri**.

```azurecli
az keyvault create --name contosoKeyVault10 --resource-group contosoResourceGroup5 --location eastus
```

![Image](../media/databricks-command-output-4.png)
![Sortie de la commande ci-dessus dans la console. Les éléments ID et vaultUri sont tous deux mis en évidence en vert pour l’utilisateur.](../media/databricks-command-output-5.png)

Pour créer le secret, utilisez la commande suivante. Attribuez au secret la valeur de **key1** de votre compte de stockage.

```azurecli
az keyvault secret set --vault-name contosoKeyVault10 --name storageKey --value "value of your key1"
```

## <a name="create-an-azure-databricks-workspace-and-add-key-vault-secret-scope"></a>Créer un espace de travail Azure Databricks et ajouter une étendue de secrets Key Vault

La procédure décrite dans cette section ne peut pas être effectuée via la ligne de commande. Suivez ce [guide](/azure/databricks/scenarios/store-secrets-azure-key-vault#create-an-azure-databricks-workspace-and-add-a-secret-scope). Vous aurez besoin d’accéder au [portail Azure](https://ms.portal.azure.com/#home) pour :

1. Créer votre ressource Azure Databricks
1. Lancer votre espace de travail
1. Créer une étendue de secrets reposant sur Key Vault

## <a name="access-your-blob-container-from-azure-databricks-workspace"></a>Accéder à votre conteneur d’objets blob à partir de l’espace de travail Azure Databricks

La procédure décrite dans cette section ne peut pas être effectuée via la ligne de commande. Suivez ce [guide](/azure/databricks/scenarios/store-secrets-azure-key-vault#access-your-blob-container-from-azure-databricks). Vous devrez utiliser l’espace de travail Azure Databricks pour :

1. Créer un **Nouveau cluster**
1. Créer un **Nouveau Notebook**
1. Compléter les champs correspondants dans le script Python
1. Exécuter le script Python

```python
dbutils.fs.mount(
source = "wasbs://<your-container-name>@<your-storage-account-name>.blob.core.windows.net",
mount_point = "/mnt/<mount-name>",
extra_configs = {"<conf-key>":dbutils.secrets.get(scope = "<scope-name>", key = "<key-name>")})

df = spark.read.text("/mnt/<mount-name>/<file-name>")

df.show()
```

## <a name="next-steps"></a>Étapes suivantes

Vérifiez que le coffre de clés est récupérable :
> [!div class="nextstepaction"]
> [Nettoyez vos ressources](../../azure-resource-manager/management/delete-resource-group.md?tabs=azure-powershell)
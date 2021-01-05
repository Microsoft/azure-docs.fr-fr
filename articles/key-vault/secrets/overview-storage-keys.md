---
title: Gérer les clés de compte de stockage avec Azure Key Vault et l’interface de ligne de commande Azure
description: Les clés de compte de stockage assurent une intégration transparente entre Azure Key Vault et l’accès au compte de stockage Azure basé sur les clés.
ms.topic: tutorial
services: key-vault
ms.service: key-vault
ms.subservice: secrets
author: msmbaldwin
ms.author: mbaldwin
manager: rkarlin
ms.date: 09/18/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 8473d3a19a86027b5b01af59d24833dc40cd1fe9
ms.sourcegitcommit: 30906a33111621bc7b9b245a9a2ab2e33310f33f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/22/2020
ms.locfileid: "95242353"
---
# <a name="manage-storage-account-keys-with-key-vault-and-the-azure-cli"></a>Gérer les clés de compte de stockage avec Key Vault et l’interface de ligne de commande Azure
> [!IMPORTANT]
> Nous vous recommandons d’utiliser l’intégration de Stockage Azure avec Azure Active Directory (Azure AD), le service Microsoft basé sur le cloud qui gère les identités et les accès. L’intégration d’Azure AD est disponible pour les [objets blob et les files d’attente Azure](../../storage/common/storage-auth-aad.md) et fournit un accès basé sur les jetons OAuth2 au Stockage Azure (comme Azure Key Vault). Azure AD vous permet d’authentifier votre application cliente en utilisant une identité d’application ou d’utilisateur plutôt que les informations d’identification du compte de stockage. Vous pouvez utiliser une [identité Azure AD managée](../../active-directory/managed-identities-azure-resources/index.yml) lors de l’exécution sur Azure. Les identités managées suppriment l’authentification du client ainsi que le stockage des informations d’identification dans ou avec votre application. Utilisez la solution ci-dessous uniquement lorsque l’authentification Azure AD n’est pas possible.

Un compte de stockage Azure utilise des informations d’identification comprenant un nom de compte et une clé. La clé qui est générée automatiquement sert de mot de passe et non de clé de chiffrement. Key Vault gère les clés de compte de stockage en les regénérant régulièrement dans le compte de stockage. De plus, il fournit des jetons de signature d’accès partagé pour permettre un accès délégué aux ressources de votre compte de stockage.

Vous pouvez utiliser la fonctionnalité de clé de compte de stockage gérée de Key Vault pour lister (synchroniser) les clés avec un compte de stockage Azure et regénérer (faire tourner) régulièrement les clés. Vous pouvez gérer les clés des comptes de stockage et des comptes de stockage classiques.

Lorsque vous utilisez la fonctionnalité de clé de compte de stockage managé, tenez compte des points suivants :

- Les valeurs de clés ne sont jamais retournées en réponse à l’appelant.
- Seul Key Vault doit gérer vos clés de compte de stockage. Ne gérez pas les clés vous-même et évitez d’interférer avec les processus de Key Vault.
- Un seul objet Key Vault doit gérer les clés de compte de stockage. Vous ne devez pas autoriser la gestion des clients par des objets multiples.
- Régénérez les clés à l’aide de Key Vault uniquement. Ne régénérez pas manuellement vos clés de compte de stockage.

## <a name="service-principal-application-id"></a>ID d’application du principal de service

Un locataire Azure AD fournit à chaque application inscrite un [principal de service](../../active-directory/develop/developer-glossary.md#service-principal-object), Le principal de service fait office d’ID d’application, qui est utilisé lors de la configuration de l’autorisation pour l’accès aux autres ressources Azure à l’aide du contrôle d’accès en fonction du rôle (RBAC) Azure.

Key Vault est une application Microsoft préinscrite dans tous les locataires Azure AD. Key Vault est inscrit sous le même ID d’application dans chaque cloud Azure.

| Locataires | Cloud | ID de l'application |
| --- | --- | --- |
| Azure AD | Azure Government | `7e7c393b-45d0-48b1-a35e-2905ddf8183c` |
| Azure AD | Azure public | `cfa8b339-82a2-471a-a3c9-0fc0be7a4093` |
| Autres  | Quelconque | `cfa8b339-82a2-471a-a3c9-0fc0be7a4093` |

## <a name="prerequisites"></a>Prérequis

Pour suivre ce guide, vous devez d’abord effectuer ce qui suit :

- [Installer l’interface de ligne de commande Microsoft Azure](/cli/azure/install-azure-cli).
- [Création d’un coffre de clés](quick-create-cli.md)
- [Création d’un compte de stockage Azure](../../storage/common/storage-account-create.md?tabs=azure-cli). Le nom du compte de stockage doit contenir uniquement des lettres minuscules et des chiffres. Le nom doit compter 3 à 24 caractères.
      
## <a name="manage-storage-account-keys"></a>Gérer les clés de compte de stockage

### <a name="connect-to-your-azure-account"></a>Se connecter au compte Azure

Authentifiez votre session Azure CLI à l’aide des commandes [az login](/powershell/module/az.accounts/connect-azaccount?view=azps-2.5.0).

```azurecli-interactive
az login
``` 

### <a name="give-key-vault-access-to-your-storage-account"></a>Octroyer un accès Key Vault à votre compte de stockage

Utilisez la commande Azure CLI [az role assignment create](/cli/azure/role/assignment) pour permettre à Key Vault d’accéder à votre compte de stockage. Fournissez à la commande les valeurs de paramètre suivantes :

- `--role`: Transmettez le rôle Azure « Rôle de service d’opérateur de clé de compte de stockage ». Ce rôle limite l’étendue d’accès à votre compte de stockage. Pour un type de compte classique, transmettez « Rôle de service d’opérateur de clé de compte de stockage classique » à la place.
- `--assignee`: Transmettez la valeur « https://vault.azure.net  », qui est l’URL de Key Vault dans le cloud public Azure. (Pour le cloud Azure Government, utilisez « --asingee-object-id » à la place, consultez [ID d’application du principal du service](#service-principal-application-id).)
- `--scope`: Transmettez l’ID de ressource de votre compte de stockage, qui se présente sous la forme `/subscriptions/<subscriptionID>/resourceGroups/<StorageAccountResourceGroupName>/providers/Microsoft.Storage/storageAccounts/<YourStorageAccountName>`. Pour trouver votre ID d’abonnement, utilisez la commande Azure CLI [az account list](/cli/azure/account?#az-account-list) ; pour trouver le nom de votre compte de stockage et le groupe de ressources du compte de stockage, utilisez la commande Azure CLI [az storage account list](/cli/azure/storage/account?#az-storage-account-list).

```azurecli-interactive
az role assignment create --role "Storage Account Key Operator Service Role" --assignee 'https://vault.azure.net' --scope "/subscriptions/<subscriptionID>/resourceGroups/<StorageAccountResourceGroupName>/providers/Microsoft.Storage/storageAccounts/<YourStorageAccountName>"
 ```
### <a name="give-your-user-account-permission-to-managed-storage-accounts"></a>Autoriser votre compte d’utilisateur à accéder à des comptes de stockage managés

Utilisez l’interface de ligne de commande Azure [az keyvault-set-policy](/cli/azure/keyvault?#az-keyvault-set-policy) pour mettre à jour la stratégie d’accès au coffre de clés et accorder des autorisations de compte de stockage à votre compte d’utilisateur.

```azurecli-interactive
# Give your user principal access to all storage account permissions, on your Key Vault instance

az keyvault set-policy --name <YourKeyVaultName> --upn user@domain.com --storage-permissions get list delete set update regeneratekey getsas listsas deletesas setsas recover backup restore purge
```

Notez que les autorisations pour les comptes de stockage ne sont pas disponibles sur la page « Stratégies d’accès » du compte de stockage dans le portail Azure.
### <a name="create-a-key-vault-managed-storage-account"></a>Créer un compte de stockage managé Key Vault

 Créez un compte de stockage managé Key Vault à l’aide de la commande Azure CLI [az keyvault storage](/cli/azure/keyvault/storage?#az-keyvault-storage-add). Définissez une période de régénération de 90 jours. Lorsque le moment est venu d’opérer une rotation, KeyVault régénère la clé qui n’est pas active, puis définit la clé nouvellement créée comme active. Une seule clé à la fois est utilisée pour émettre des jetons SAP, c’est la clé active. Fournissez à la commande les valeurs de paramètre suivantes :

- `--vault-name`: transmettez le nom de votre coffre de clés. Pour trouver le nom de votre coffre de clés, utilisez la commande Azure CLI [az keyvault list](/cli/azure/keyvault?#az-keyvault-list).
- `-n`: transmettez le nom de votre compte de stockage Pour trouver le nom de votre compte de stockage, utilisez la commande Azure CLI [az storage account list](/cli/azure/storage/account?#az-storage-account-list).
- `--resource-id`: Transmettez l’ID de ressource de votre compte de stockage, qui se présente sous la forme `/subscriptions/<subscriptionID>/resourceGroups/<StorageAccountResourceGroupName>/providers/Microsoft.Storage/storageAccounts/<YourStorageAccountName>`. Pour trouver votre ID d’abonnement, utilisez la commande Azure CLI [az account list](/cli/azure/account?#az-account-list) ; pour trouver le nom de votre compte de stockage et le groupe de ressources du compte de stockage, utilisez la commande Azure CLI [az storage account list](/cli/azure/storage/account?#az-storage-account-list).
   
 ```azurecli-interactive
az keyvault storage add --vault-name <YourKeyVaultName> -n <YourStorageAccountName> --active-key-name key1 --auto-regenerate-key --regeneration-period P90D --resource-id "/subscriptions/<subscriptionID>/resourceGroups/<StorageAccountResourceGroupName>/providers/Microsoft.Storage/storageAccounts/<YourStorageAccountName>"
 ```

## <a name="shared-access-signature-tokens"></a>Jetons de signature d’accès partagé

Vous pouvez également demander à Key Vault de générer des jetons de signature d’accès partagé. Une signature d'accès partagé fournit un accès délégué aux ressources de votre compte de stockage. Vous pouvez accorder aux clients l’accès aux ressources dans votre compte de stockage sans partager les clés de votre compte. Une signature d’accès partagé constitue un moyen sécurisé de partager vos ressources de stockage sans compromettre vos clés de compte.

Les commandes figurant dans cette section effectuent les actions suivantes :

- Définissez une définition de signature d’accès partagé `<YourSASDefinitionName>`. La définition pointe sur un compte de stockage managé Key Vault `<YourStorageAccountName>` dans votre coffre de clés `<YourKeyVaultName>`.
- Créez un jeton de signature d’accès partagé pour les services Blob, Fichier, Table et File d’attente. Le jeton est créé pour les types de ressource Service, Conteneur et Objet. Le jeton est créé avec toutes les autorisations, via le protocole https et avec les dates de début et de fin spécifiées.
- Définissez une définition de signature d’accès partagé au stockage géré par Key Vault dans le coffre. La définition comporte l’URI du modèle du jeton de signature d’accès partagé créé. La définition possède le type de signature d’accès partagé `account` et est valide pendant N jours.
- Vérifiez que la signature d’accès partagé a été enregistrée dans votre coffre de clés en tant que secret.

### <a name="create-a-shared-access-signature-token"></a>Créer un jeton de signature d’accès partagé

Créez une définition de signature d’accès partagé à l’aide de la commande Azure CLI [az storage account generate-sas](/cli/azure/storage/account?#az-storage-account-generate-sas). Cette opération nécessite les autorisations `storage` et `setsas`.


```azurecli-interactive
az storage account generate-sas --expiry 2020-01-01 --permissions rw --resource-types sco --services bfqt --https-only --account-name <YourStorageAccountName> --account-key 00000000
```
Une fois l’opération exécutée avec succès, copiez la sortie.

```console
"se=2020-01-01&sp=***"
```

Cette sortie est transmise au paramètre `--template-uri` à l’étape suivante.

### <a name="generate-a-shared-access-signature-definition"></a>Générer une définition de signature d’accès partagé

Utilisez la commande Azure CLI [az keyvault storage sas-definition create](/cli/azure/keyvault/storage/sas-definition?#az-keyvault-storage-sas-definition-create), en transmettant la sortie de l’étape précédente au paramètre `--template-uri`, afin de créer une définition de signature d’accès partagé.  Vous pouvez fournir le nom de votre choix au paramètre `-n`.

```azurecli-interactive
az keyvault storage sas-definition create --vault-name <YourKeyVaultName> --account-name <YourStorageAccountName> -n <YourSASDefinitionName> --validity-period P2D --sas-type account --template-uri <OutputOfSasTokenCreationStep>
```

### <a name="verify-the-shared-access-signature-definition"></a>Vérifier la définition de la signature d’accès partagé

Vous pouvez vérifier que la définition de signature d’accès partagé a été stockée dans votre coffre de clés à l’aide de la commande Azure CLI [az keyvault storage sas-definition show](/cli/azure/keyvault/storage/sas-definition?#az_keyvault_storage_sas_definition_show).

Vous pouvez maintenant utiliser la commande [az keyvault storage sas-definition show](/cli/azure/keyvault/storage/sas-definition?#az_keyvault_storage_sas_definition_show) et la propriété `id` pour afficher le contenu de ce secret.

```azurecli-interactive
az keyvault storage sas-definition show --id https://<YourKeyVaultName>.vault.azure.net/storage/<YourStorageAccountName>/sas/<YourSASDefinitionName>
```

## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur [les clés, les secrets et les certificats](/rest/api/keyvault/).
- Passez en revue les articles de [blog de l’équipe Azure Key Vault](/archive/blogs/kv/).
- Consultez la documentation de référence sur la commande [az keyvault storage](/cli/azure/keyvault/storage).

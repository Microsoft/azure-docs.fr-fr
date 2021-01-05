---
title: 'Démarrage rapide : Créer un blob avec Azure CLI'
titleSuffix: Azure Storage
description: Dans ce guide de démarrage rapide, vous allez apprendre à utiliser Azure CLI pour charger un objet blob dans Stockage Azure, télécharger un objet blob et répertorier les objets blob d’un conteneur.
services: storage
author: tamram
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.date: 08/17/2020
ms.author: tamram
ms.custom: devx-track-azurecli
ms.openlocfilehash: fa502f5ca95b1726da7f00f987b35be362ae865a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021756"
---
# <a name="quickstart-create-download-and-list-blobs-with-azure-cli"></a>Démarrage rapide : Créer, télécharger et lister des objets blob avec Azure CLI

L’interface de ligne de commande Azure (Azure CLI) est l’expérience de ligne de commande d’Azure pour gérer les ressources Azure. Vous pouvez l’utiliser dans votre navigateur avec Azure Cloud Shell. Vous pouvez également l’installer sur Windows, Linux ou MacOS, et l’exécuter à partir de la ligne de commande. Dans ce guide de démarrage rapide, vous apprenez à utiliser Azure CLI pour charger et télécharger des données vers et à partir du stockage Blob Azure.

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment-h3.md)]

- Cet article nécessite la version 2.0.46 ou ultérieure de l’interface Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="authorize-access-to-blob-storage"></a>Autoriser l’accès au stockage Blob

Vous pouvez autoriser l’accès au stockage Blob à partir d’Azure CLI avec des informations d’identification Azure AD ou en utilisant la clé d’accès au compte de stockage. Il est recommandé d’utiliser des informations d’identification Azure AD. Cet article explique comment autoriser les opérations de stockage Blob en utilisant Azure AD.

Les commandes Azure CLI pour les opérations de données sur le stockage Blob prennent en charge le paramètre `--auth-mode`, qui vous permet de spécifier la façon dont une opération donnée est autorisée. Définissez le paramètre `--auth-mode` sur `login` pour autoriser les informations d’identification Azure AD. Pour plus d’informations, consultez [Autoriser l’accès aux données d’objet blob et de file d’attente avec Azure CLI](./authorize-data-operations-cli.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

Seules les opérations sur les données de stockage Blob prennent en charge le paramètre `--auth-mode`. Les opérations de gestion, comme la création d’un groupe de ressources ou d’un compte de stockage, utilisent automatiquement des informations d’identification Azure AD pour l’autorisation.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources Azure avec la commande [az group create](/cli/azure/group). Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.

N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```azurecli
az group create \
    --name <resource-group> \
    --location <location>
```

## <a name="create-a-storage-account"></a>Créez un compte de stockage.

Créez un compte de stockage à usage général avec la commande [az storage account create](/cli/azure/storage/account). Le compte de stockage à usage général peut être utilisé pour les quatre services : objets blob, fichiers, tables et files d’attente.

N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```azurecli
az storage account create \
    --name <storage-account> \
    --resource-group <resource-group> \
    --location <location> \
    --sku Standard_ZRS \
    --encryption-services blob
```

## <a name="create-a-container"></a>Créez un conteneur.

Les objets blob sont toujours chargés dans un conteneur. Vous pouvez organiser des groupes d’objets blob dans des conteneurs de la même façon que vous organisez vos fichiers dans les dossiers de l’ordinateur. Créez un conteneur pour stocker des objets blob avec la commande [az storage container create](/cli/azure/storage/container).

L’exemple suivant utilise votre compte Azure AD pour autoriser l’opération à créer le conteneur. Avant de créer le conteneur, attribuez-vous le rôle [Contributeur aux données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor). Même si vous êtes le propriétaire du compte, vous avez besoin d’autorisations explicites pour effectuer des opérations de données sur le compte de stockage. Pour plus d’informations sur l’attribution de rôles Azure, consultez [Utiliser Azure CLI pour attribuer un rôle Azure pour l’accès](../common/storage-auth-aad-rbac-cli.md?toc=/azure/storage/blobs/toc.json).  

N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```azurecli
az ad signed-in-user show --query objectId -o tsv | az role assignment create \
    --role "Storage Blob Data Contributor" \
    --assignee @- \
    --scope "/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>"

az storage container create \
    --account-name <storage-account> \
    --name <container> \
    --auth-mode login
```

> [!IMPORTANT]
> La propagation des attributions de rôles Azure peut prendre plusieurs minutes.

Vous pouvez également utiliser la clé de compte de stockage pour autoriser l’opération de création du conteneur. Pour plus d’informations sur l’autorisation d’opérations de données avec Azure CLI, consultez [Autoriser l’accès aux données d’objet blob et de file d’attente avec Azure CLI](./authorize-data-operations-cli.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

## <a name="upload-a-blob"></a>Charger un objet blob

Stockage Blob prend en charge les objets blob de blocs, d’ajout et de pages. Les exemples de ce guide de démarrage rapide montrent comment utiliser des objets blob de blocs.

Commencez par créer un fichier à charger dans un objet blob de blocs. Si vous utilisez Azure Cloud Shell, utilisez la commande suivante pour créer un fichier :

```bash
vi helloworld
```

Lorsque le fichier s’ouvre, appuyez sur **Insérer**. Entrez *Hello world*, puis appuyez sur **Échap**. Entrez ensuite *:x*, puis appuyez sur **Entrée**.

Dans cet exemple, vous chargez un objet blob dans le conteneur que vous avez créé à la dernière étape avec la commande [az storage blob upload](/cli/azure/storage/blob). Il n’est pas nécessaire de spécifier le chemin du fichier, car celui-ci a été créé dans le répertoire racine. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```azurecli
az storage blob upload \
    --account-name <storage-account> \
    --container-name <container> \
    --name helloworld \
    --file helloworld \
    --auth-mode login
```

Cette opération crée l’objet blob s’il n’existe pas déjà, et le remplace s’il existe. Chargez autant de fichiers que vous le souhaitez avant de continuer.

Pour charger plusieurs fichiers à la fois, vous pouvez utiliser la commande [az storage blob upload-batch](/cli/azure/storage/blob).

## <a name="list-the-blobs-in-a-container"></a>Créer la liste des objets blob d’un conteneur

Listez les objets blob d’un conteneur avec la commande [az storage blob list](/cli/azure/storage/blob). N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```azurecli
az storage blob list \
    --account-name <storage-account> \
    --container-name <container> \
    --output table \
    --auth-mode login
```

## <a name="download-a-blob"></a>Télécharger un objet blob

Utilisez la commande [az storage blob download](/cli/azure/storage/blob) pour télécharger l’objet blob chargé au préalable. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```azurecli
az storage blob download \
    --account-name <storage-account> \
    --container-name <container> \
    --name helloworld \
    --file ~/destination/path/for/file \
    --auth-mode login
```

## <a name="data-transfer-with-azcopy"></a>Transfert de données avec AzCopy

L’utilitaire en ligne de commande AzCopy assure un transfert de données hautement performant et scriptable pour Stockage Azure. Vous pouvez utiliser AzCopy pour transférer des données depuis et vers le stockage Blob et Azure Files. Pour plus d’informations sur AzCopy v10, la dernière version d’AzCopy, consultez [Bien démarrer avec AzCopy](../common/storage-use-azcopy-v10.md). Pour plus d’informations sur l’utilisation d’AzCopy v10 avec le stockage Blob, consultez [Transférer des données avec AzCopy et le stockage Blob](../common/storage-use-azcopy-blobs.md).

L’exemple suivant utilise AzCopy pour charger un fichier local dans un objet blob. Pensez à remplacer les valeurs de l’exemple par vos propres valeurs :

```bash
azcopy login
azcopy copy 'C:\myDirectory\myTextFile.txt' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt'
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous voulez supprimer les ressources que vous avez créées dans le cadre de ce guide de démarrage rapide, notamment le compte de stockage, supprimez le groupe de ressources à l’aide de la commande [az group delete](/cli/azure/group). N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```azurecli
az group delete \
    --name <resource-group> \
    --no-wait
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à transférer des fichiers entre un système de fichiers local et un conteneur du stockage Blob Azure. Pour en savoir plus sur l’utilisation du Stockage Blob à l’aide d’Azure CLI, explorez les exemples Azure CLI afin de découvrir le Stockage Blob.

> [!div class="nextstepaction"]
> [Exemples Azure CLI pour le Stockage Blob](./storage-samples-blobs-cli.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
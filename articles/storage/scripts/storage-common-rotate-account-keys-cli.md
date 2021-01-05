---
title: Exemple de script Azure CLI - Rotation des clés d’accès au compte de stockage | Microsoft Docs
description: Créez un compte de stockage Azure, puis récupérez ses clés d’accès et organisez leur rotation.
services: storage
author: tamram
ms.service: storage
ms.subservice: blobs
ms.devlang: cli
ms.topic: sample
ms.date: 10/20/2020
ms.author: tamram
ms.custom: devx-track-azurecli
ms.openlocfilehash: 08e1b3837863b197f8463a0d969e78afab2b9858
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92370404"
---
# <a name="create-a-storage-account-and-rotate-its-account-access-keys"></a>Créer un compte de stockage et faire tourner ses clés d’accès

Ce script crée un compte de stockage Azure, affiche les clés d’accès au nouveau compte, puis renouvelle (fait tourner) les clés.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Exemple de script

[!code-azurecli-interactive[main](../../../cli_scripts/storage/rotate-storage-account-keys/rotate-storage-account-keys.sh "Rotate storage account keys")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement

Exécutez la commande suivante pour supprimer le groupe de ressources, le compte de stockage et toutes les ressources associées.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>Explication du script

Ce script utilise les commandes suivantes pour créer le compte de stockage, récupérer ses clés d’accès et les faire tourner. Chaque élément du tableau renvoie à une documentation spécifique de la commande.

| Commande | Notes |
|---|---|
| [az group create](/cli/azure/group) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [az storage account create](/cli/azure/storage/account) | Crée un compte de stockage Azure dans le groupe de ressources spécifié. |
| [az storage account keys list](/cli/azure/storage/account/keys) | Affiche les clés de l’accès au compte de stockage spécifié. |
| [az storage account keys renew](/cli/azure/storage/account/keys) | Régénère la clé d’accès au compte de stockage principal ou secondaire. |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’interface Azure CLI, consultez la [documentation relative à l’interface Azure CLI](/cli/azure).

Vous trouverez des exemples supplémentaires de scripts CLI de stockage dans les [exemples Azure CLI pour le stockage d’objets blob Azure](../blobs/storage-samples-blobs-cli.md).

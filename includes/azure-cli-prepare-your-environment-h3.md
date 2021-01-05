---
ms.topic: include
ms.date: 09/10/2020
author: dbradish-microsoft
ms.author: dbradish
manager: barbkess
ms.technology: azure-cli
ms.service: azure-cli
ms.devlang: azurecli
ms.custom: devx-track-azurecli
ms.openlocfilehash: 01e1f956270ebbc9b9d5d0ebdfdff52875cafedf
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000048"
---
### <a name="prepare-your-environment-for-the-azure-cli"></a>Préparation de votre environnement pour Azure CLI

- Utilisez [Azure Cloud Shell](../articles/cloud-shell/quickstart.md) avec l’environnement Bash.

   [![Lancement de l’incorporation](https://shell.azure.com/images/launchcloudshell.png "Lancement d’Azure Cloud Shell")](https://shell.azure.com)   
- Si vous préférez, [installez](/cli/azure/install-azure-cli) l’interface Azure CLI pour exécuter les commandes de référence de l’interface de ligne de commande.
   - Si vous utilisez une installation locale, connectez-vous à Azure CLI à l’aide de la commande [az login](/cli/azure/reference-index#az-login).  Pour finir le processus d’authentification, suivez les étapes affichées dans votre terminal.  Consultez [Se connecter avec Azure CLI](/cli/azure/authenticate-azure-cli) pour connaître les autres options de connexion.
  - Lorsque vous y êtes invité, installez les extensions Azure CLI lors de la première utilisation.  Pour plus d’informations sur les extensions, consultez [Utiliser des extensions avec Azure CLI](/cli/azure/azure-cli-extensions-overview).
  - Exécutez [az version](/cli/azure/reference-index?#az_version) pour rechercher la version et les bibliothèques dépendantes installées. Pour effectuer une mise à niveau vers la dernière version, exécutez [az upgrade](/cli/azure/reference-index?#az_upgrade).
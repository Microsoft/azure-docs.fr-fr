---
title: Fichier Include
description: Fichier Include
services: virtual-machines
author: msmbaldwin
ms.service: virtual-machines
ms.topic: include
ms.date: 10/06/2019
ms.author: mbaldwin
ms.custom: include file, devx-track-azurecli
ms.openlocfilehash: 1c1d438f0322942a1e68c0af74de8d5e2d77c77a
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95555012"
---
Vous pouvez activer et gérer Azure Disk Encryption par le biais d’[Azure CLI](/cli/azure) et d’[Azure PowerShell](/powershell/azure/new-azureps-module-az). Pour ce faire, vous devez installer les outils localement et vous connecter à votre abonnement Azure.

### <a name="azure-cli"></a>Azure CLI

[Azure CLI 2.0](/cli/azure) est un outil en ligne de commande pour la gestion des ressources Azure. L’interface CLI est conçue pour interroger les données de manière flexible, pour prendre en charge les opérations de longue durée en tant que processus non bloquants et pour simplifier l’écriture de scripts. Vous pouvez l’installer localement en effectuant les étapes décrites dans [Installer Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest).

Pour [vous connecter à votre compte Azure avec Azure CLI](/cli/azure/authenticate-azure-cli), utilisez la commande [az login](/cli/azure/reference-index?view=azure-cli-latest#az-login).

```azurecli
az login
```

Si vous voulez sélectionner un locataire sous lequel vous connecter, utilisez :
    
```azurecli
az login --tenant <tenant>
```

Si vous avez plusieurs abonnements et que vous voulez spécifier un de ceux-ci, récupérez la liste de vos abonnements avec [az account list](/cli/azure/account#az-account-list) et spécifiez le compte avec [az account set](/cli/azure/account#az-account-set).
     
```azurecli
az account list
az account set --subscription "<subscription name or ID>"
```

Pour plus d’informations, consultez [Bien démarrer avec Azure CLI 2.0](/cli/azure/get-started-with-azure-cli). 

### <a name="azure-powershell"></a>Azure PowerShell
Le [module az Azure PowerShell](/powershell/azure/new-azureps-module-az) fournit un ensemble de cmdlets qui utilise le modèle [Azure Resource Manager](../articles/azure-resource-manager/management/overview.md) pour gérer vos ressources Azure. Vous pouvez l’utiliser dans votre navigateur avec [Azure Cloud Shell](../articles/cloud-shell/overview.md), ou vous pouvez l’installer sur votre ordinateur local à l’aide des instructions fournies dans [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). 

Si vous l’avez déjà installé localement, veillez à utiliser la dernière version du Kit de développement logiciel (SDK) Azure PowerShell pour configurer Azure Disk Encryption. Téléchargez la dernière version [d’Azure PowerShell](https://github.com/Azure/azure-powershell/releases).

Pour [vous connecter à votre compte Azure avec Azure PowerShell](/powershell/azure/authenticate-azureps?view=azps-2.5.0), utilisez la cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount?view=azps-2.5.0).

```powershell
Connect-AzAccount
```

Si vous avez plusieurs abonnements et que vous souhaitez en spécifier un, utilisez la cmdlet [Get-AzSubscription](/powershell/module/Az.Accounts/Get-AzSubscription) pour les lister, puis la cmdlet [Set-AzContext](/powershell/module/az.accounts/set-azcontext?view=azps-2.5.0) :

```powershell
Set-AzContext -Subscription -Subscription <SubscriptionId>
```

L’exécution de la cmdlet [Get-AzContext](/powershell/module/Az.Accounts/Get-AzContext) permet de vérifier que l’abonnement approprié a été sélectionné.

Pour vérifier que les cmdlets Azure Disk Encryption sont installées, utilisez la cmdlet [Get-command](/powershell/module/microsoft.powershell.core/get-command?view=powershell-6) :
     
```powershell
Get-command *diskencryption*
```
Pour plus d’informations, consultez [Bien démarrer avec Azure PowerShell](/powershell/azure/get-started-azureps).
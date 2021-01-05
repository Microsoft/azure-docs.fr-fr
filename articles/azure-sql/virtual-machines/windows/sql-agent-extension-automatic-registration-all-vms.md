---
title: Inscription automatique auprès de l’extension SQL IaaS Agent
description: Découvrez comment activer la fonctionnalité d’inscription automatique pour inscrire automatiquement toutes les machines virtuelles SQL Server passées et futures auprès de l’extension SQL IaaS Agent à l’aide du portail Azure.
author: MashaMSFT
ms.author: mathoma
tags: azure-service-management
ms.service: virtual-machines-sql
ms.subservice: management
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 11/07/2020
ms.openlocfilehash: 1ef7943586123a1870ed9a2d0c21aa8b5fd38c1c
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359997"
---
# <a name="automatic-registration-with-sql-iaas-agent-extension"></a>Inscription automatique auprès de l’extension SQL IaaS Agent
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Activez la fonctionnalité d’inscription automatique dans le portail Azure pour inscrire automatiquement toutes les instances actuelles et futures de SQL Server sur les machines virtuelles Azure auprès de [l’extension SQL IaaS Agent](sql-server-iaas-agent-extension-automate-management.md) en mode léger. 

Cet article vous apprend à activer la fonctionnalité d’inscription automatique. Vous pouvez également [inscrire une seule machine virtuelle](sql-agent-extension-manually-register-single-vm.md) ou [inscrire vos machines virtuelles en bloc](sql-agent-extension-manually-register-vms-bulk.md) à l’aide de l’extension SQL IaaS Agent. 

## <a name="overview"></a>Vue d’ensemble

Le fait d’inscrire votre machine virtuelle SQL Server auprès de [l’extension SQL IaaS Agent](sql-server-iaas-agent-extension-automate-management.md) déverrouille un ensemble complet de fonctionnalités. 

Lorsque l’inscription automatique est activée, un travail est exécuté quotidiennement pour détecter si SQL Server est installé ou non sur toutes les machines virtuelles non inscrites de l’abonnement. Pour ce faire, copiez les fichiers binaires de l’extension SQL IaaS Agent sur la machine virtuelle, puis exécutez un utilitaire ponctuel qui recherche la ruche du registre SQL Server. Si la ruche SQL Server est détectée, la machine virtuelle est inscrite auprès de l’extension en mode léger. Si aucune ruche SQL Server n’existe dans le registre, les fichiers binaires sont supprimés.

Une fois l’inscription automatique activée pour un abonnement, toutes les machines virtuelles actuelles et futures sur lesquelles SQL Server est installé sont inscrites auprès de l’extension SQL IaaS Agent en mode léger. Vous devez toujours [mettre à niveau manuellement vers le mode de gestion complet](sql-agent-extension-manually-register-single-vm.md#upgrade-to-full) pour tirer parti de l’ensemble complet de fonctionnalités. 

> [!IMPORTANT]
> L’extension SQL IaaS Agent collecte des données dans le seul but de fournir d’autres avantages aux clients lors de l’utilisation de SQL Server dans Machines virtuelles Azure. Microsoft n’utilisera pas ces données pour les audits de gestion des licences sans le consentement préalable du client. Pour plus d’informations, consultez l’[Avenant à la déclaration de confidentialité de SQL Server](/sql/sql-server/sql-server-privacy#non-personal-data).

## <a name="prerequisites"></a>Prérequis

Pour inscrire votre machine virtuelle SQL Server auprès de l’extension, voici ce dont vous avez besoin : 

- Un [abonnement Azure](https://azure.microsoft.com/free/) et, au minimum, les autorisations du [rôle Contributeur](../../../role-based-access-control/built-in-roles.md#all).
- Une [machine virtuelle Windows Server 2008 R2 (ou ultérieure)](../../../virtual-machines/windows/quick-create-portal.md) de modèle de ressource Azure avec [SQL Server](https://www.microsoft.com/sql-server/sql-server-downloads) déployé sur le cloud public ou sur le cloud Azure Government. Windows Server 2008 n’est pas pris en charge. 


## <a name="enable"></a>Activer

Pour activer l’inscription automatique de vos machines virtuelles SQL Server dans le portail Azure, procédez comme suit :

1. Connectez-vous au [Portail Azure](https://portal.azure.com).
1. Accédez à la page de ressource [**machines virtuelles SQL**](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.SqlVirtualMachine%2FSqlVirtualMachines). 
1. Sélectionnez **l’inscription automatique de la machine virtuelle SQL Server** pour ouvrir la page **Inscription automatique**. 

   Sélectionner :::image type="content" source="media/sql-agent-extension-automatic-registration-all-vms/automatic-registration.png" alt-text="l’inscription automatique de la machine virtuelle SQL Server pour ouvrir la page Inscription automatique":::

1. Choisissez votre abonnement dans le menu déroulant. 
1. Lisez les conditions générales et, si vous acceptez, sélectionnez **J’accepte**. 
1. Sélectionnez **Inscrire** pour activer la fonctionnalité et inscrire automatiquement toutes les machines virtuelles SQL Server actuelles et futures auprès de l’extension SQL IaaS Agent. Cela ne redémarre pas le service SQL Server sur les machines virtuelles. 

## <a name="disable"></a>Désactiver

Pour désactiver la fonctionnalité d’inscription automatique, utilisez [Azure CLI](/cli/azure/install-azure-cli) ou [Azure PowerShell](/powershell/azure/install-az-ps) . Lorsque la fonctionnalité d’inscription automatique est désactivée, les machines virtuelles SQL Server ajoutées à l’abonnement doivent être inscrites manuellement auprès de l’extension SQL IaaS Agent. Cela n’annule pas l’inscription des machines virtuelles SQL Server existantes qui ont déjà été inscrites.



# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour désactiver l’inscription automatique à l’aide d’Azure CLI, exécutez la commande suivante : 

```azurecli-interactive
az feature unregister --namespace Microsoft.SqlVirtualMachine --name BulkRegistration
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Pour désactiver l’inscription automatique à l’aide d’Azure PowerShell, exécutez la commande suivante : 

```powershell-interactive
Unregister-AzProviderFeature -FeatureName BulkRegistration -ProviderNamespace Microsoft.SqlVirtualMachine
```

---

## <a name="enable-for-multiple-subscriptions"></a>Activer quatre abonnements multiples

Vous pouvez activer la fonctionnalité d’inscription automatique pour plusieurs abonnements Azure à l’aide de PowerShell. 

Pour ce faire, procédez comme suit :

1. Enregistrez [ce script](https://github.com/microsoft/tigertoolbox/blob/master/AzureSQLVM/RegisterSubscriptionsToSqlVmAutomaticRegistration.ps1) dans un fichier `.ps1`, par exemple `EnableBySubscription.ps1`. 
1. Accédez à l’emplacement où vous avez enregistré le script à l’aide d’une invite de commandes d’administration ou d’une fenêtre PowerShell. 
1. Connectez-vous à Azur (`az login`).
1. Exécutez le script en passant SubscriptionIds comme paramètres, comme   
   `.\EnableBySubscription.ps1 -SubscriptionList SubscriptionId1,SubscriptionId2`

   Par exemple : 

   ```console
   .\EnableBySubscription.ps1 -SubscriptionList a1a1a-aa11-11aa-a1a1-a11a111a1,b2b2b2-bb22-22bb-b2b2-b2b2b2bb
   ```

Les erreurs d’inscription sont stockées dans `RegistrationErrors.csv` situé dans le même répertoire que celui où vous avez enregistré et exécuté le script `.ps1`. 

## <a name="next-steps"></a>Étapes suivantes

Mettez à niveau votre mode d’administration vers le mode [complet](sql-agent-extension-manually-register-single-vm.md#upgrade-to-full) afin de tirer parti de toutes les fonctionnalités de l’extension SQL IaaS Agent. 

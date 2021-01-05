---
title: Inscrire auprès de l’extension SQL IaaS Agent
description: Inscrivez votre machine virtuelle Azure SQL Server auprès de l’extension SQL IaaS Agent pour activer les fonctionnalités des machines virtuelles SQL Server déployées en dehors de Place de marché Azure, ainsi que la conformité et la gestion améliorée.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.subservice: management
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 11/07/2020
ms.author: mathoma
ms.reviewer: jroth
ms.custom: devx-track-azurecli, devx-track-azurepowershell, contperf-fy21q2
ms.openlocfilehash: e7a8f54abbadb63c870c4d92843699c67f59752c
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97505628"
---
# <a name="register-sql-server-vm-with-sql-iaas-agent-extension"></a>Inscrire une machine virtuelle SQL Server auprès de l’extension SQL IaaS Agent
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Le fait d’inscrire votre machine virtuelle SQL Server auprès de [l’extension SQL IaaS Agent](sql-server-iaas-agent-extension-automate-management.md) déverrouille une multitude de fonctionnalités pour votre SQL Server sur une machine virtuelle Azure. 

Cet article vous apprend à inscrire une machine virtuelle SQL Server individuelle auprès de l’extension SQL IaaS Agent. Vous pouvez également inscrire toutes les machines virtuelles SQL Server [automatiquement](sql-agent-extension-automatic-registration-all-vms.md) ou [plusieurs machines virtuelles scriptées en bloc](sql-agent-extension-manually-register-vms-bulk.md).


## <a name="overview"></a>Vue d’ensemble

S’inscrire auprès de [l’extension SQL Server IaaS Agent](sql-server-iaas-agent-extension-automate-management.md) crée la _ressource_ de **machine virtuelle SQL** dans votre abonnement ; il s’agit d’une ressource _distincte_ de la ressource de machine virtuelle. Le fait de désinscrire votre machine virtuelle SQL Server de l’extension va supprimer la _ressource_ de **machine virtuelle SQL**, mais pas la machine virtuelle elle-même.

Pendant le déploiement d’une image de machine virtuelle SQL Server de la Place de marché Azure via le portail Azure, la machine virtuelle SQL Server est inscrite automatiquement à l’extension. Toutefois, si vous choisissez d’installer automatiquement SQL Server sur une machine virtuelle Azure ou de configurer une machine virtuelle Azure à partir d’un disque dur virtuel personnalisé, vous devez inscrire votre machine virtuelle SQL Server auprès de l’extension SQL IaaS Agent pour déverrouiller toutes les fonctionnalités et capacités de gestion. 

Pour utiliser l’extension SQL IaaS Agent, vous devez d’abord [inscrire votre abonnement auprès du fournisseur **Microsoft.SqlVirtualMachine**](#register-subscription-with-rp), ce qui donne à l’extension SQL IaaS Agent la capacité de créer des ressources dans cet abonnement spécifique.

> [!IMPORTANT]
> L’extension SQL IaaS Agent collecte des données dans le seul but de fournir d’autres avantages aux clients lors de l’utilisation de SQL Server dans Machines virtuelles Azure. Microsoft n’utilisera pas ces données pour les audits de gestion des licences sans le consentement préalable du client. Pour plus d’informations, consultez l’[Avenant à la déclaration de confidentialité de SQL Server](/sql/sql-server/sql-server-privacy#non-personal-data).

## <a name="prerequisites"></a>Prérequis

Pour inscrire votre machine virtuelle SQL Server auprès de l’extension, voici ce dont vous avez besoin : 

- Un [abonnement Azure](https://azure.microsoft.com/free/).
- Une [machine virtuelle Windows Server 2008 (ou version ultérieure)](../../../virtual-machines/windows/quick-create-portal.md) de modèle de ressource Azure avec [SQL Server 2008 (ou version ultérieure)](https://www.microsoft.com/sql-server/sql-server-downloads), déployée sur le cloud public ou le cloud Azure Government. 
- La version la plus récente d’[Azure CLI](/cli/azure/install-azure-cli) ou d’[Azure PowerShell (5.0 minimum)](/powershell/azure/install-az-ps). 


## <a name="register-subscription-with-rp"></a>Inscrire un abonnement auprès d’un fournisseur de ressources

Pour inscrire votre machine virtuelle SQL Server auprès de l’extension SQL IaaS Agent, vous devez d’abord inscrire votre abonnement auprès du fournisseur **Microsoft.SqlVirtualMachine**. Ainsi, l’extension SQL IaaS Agent peut créer des ressources dans votre abonnement.  Pour ce faire, vous pouvez utiliser le portail Azure, l’interface Azure CLI ou Azure PowerShell.

### <a name="azure-portal"></a>Portail Azure

1. Ouvrez le portail Azure et accédez à **Tous les services**. 
1. Accédez à **Abonnements** et sélectionnez l’abonnement qui vous intéresse.  
1. Dans la page **Abonnements**, accédez à **Extensions**. 
1. Entrez **sql** dans le filtre pour afficher les extensions liées à SQL. 
1. Sélectionnez **Inscrire**, **Réinscrire** ou **Désinscrire** pour le fournisseur **Microsoft.SqlVirtualMachine**, en fonction de l’action souhaitée. 

   ![Modifier le fournisseur](./media/sql-agent-extension-manually-register-single-vm/select-resource-provider-sql.png)


### <a name="command-line"></a>Ligne de commande

Inscrivez votre abonnement Azure auprès du fournisseur **Microsoft.SqlVirtualMachine** en utilisant Azure CLI ou Azure PowerShell. 

# <a name="azure-cli"></a>[Azure CLI](#tab/bash)

```azurecli-interactive
# Register the SQL IaaS Agent extension to your subscription 
az provider register --namespace Microsoft.SqlVirtualMachine 
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

```powershell-interactive
# Register the SQL IaaS Agent extension to your subscription
Register-AzResourceProvider -ProviderNamespace Microsoft.SqlVirtualMachine
```

---

## <a name="register-with-extension"></a>S’inscrire auprès de l’extension

Il existe trois modes d’administration pour l’extension [SQL Server IaaS Agent](sql-server-iaas-agent-extension-automate-management.md#management-modes). 

L’inscription de l’extension en mode d’administration complet redémarre le service SQL Server ; il est donc recommandé d’inscrire tout d’abord l’extension en mode léger, puis de procéder à la [mise à niveau vers le mode complet](#upgrade-to-full) pendant une fenêtre de maintenance. 

### <a name="lightweight-management-mode"></a>Mode de gestion léger

Utilisez l’interface de ligne de commande Azure ou Azure PowerShell pour inscrire votre machine virtuelle SQL Server auprès de l’extension en mode léger. Le service SQL Server ne sera pas redémarré. Vous pouvez ensuite effectuer une mise à niveau vers le mode complet à tout moment ; cependant, comme cela entraîne le redémarrage du service SQL Server, nous vous recommandons d’attendre une fenêtre de maintenance planifiée. 

Indiquez une licence SQL Server de type paiement à l’utilisation (`PAYG`) pour payer en fonction de l’utilisation, Azure Hybrid Benefit (`AHUB`) pour utiliser votre propre licence ou récupération d’urgence (`DR`) pour activer la [licence de réplica de récupération d’urgence gratuite](business-continuity-high-availability-disaster-recovery-hadr-overview.md#free-dr-replica-in-azure).

Les instances de cluster de basculement et les déploiements multi-instances ne peuvent être inscrits auprès de l’extension SQL IaaS Agent qu’en mode léger. 

# <a name="azure-cli"></a>[Azure CLI](#tab/bash)

Inscrire une machine virtuelle SQL Server en mode allégé avec l’interface de ligne de commande Azure : 

  ```azurecli-interactive
  # Register Enterprise or Standard self-installed VM in Lightweight mode
  az sql vm create --name <vm_name> --resource-group <resource_group_name> --location <vm_location> --license-type <license_type> 
  ```


# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Inscrire une machine virtuelle SQL Server en mode léger avec Azure PowerShell :  


  ```powershell-interactive
  # Get the existing compute VM
  $vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
          
  # Register SQL VM with 'Lightweight' SQL IaaS agent
  New-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -Location $vm.Location `
    -LicenseType <license_type>  -SqlManagementType LightWeight  
  ```

---

### <a name="full-management-mode"></a>Mode de gestion complet

L’inscription de votre machine virtuelle SQL Server en mode complet redémarre le service SQL Server. Agissez avec précaution. 

Pour inscrire directement votre machine virtuelle SQL Server en mode complet (et éventuellement redémarrer votre service SQL Server), utilisez la commande Azure PowerShell suivante : 

  ```powershell-interactive
  # Get the existing  Compute VM
  $vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
        
  # Register with SQL IaaS Agent extension in full mode
  New-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -SqlManagementType Full
  ```

### <a name="noagent-management-mode"></a>Mode de gestion sans agent 

Les instances de SQL Server 2008 et 2008 R2 installées sur Windows Server 2008 (_non R2_) peuvent être inscrites auprès de l’extension SQL IaaS Agent en [mode sans agent](sql-server-iaas-agent-extension-automate-management.md#management-modes). Cette option garantit la conformité et permet de surveiller la machine virtuelle SQL Server dans le portail Azure avec des fonctionnalités limitées.


Pour le **type de licence**, spécifiez `AHUB`, `PAYG` ou `DR`. Pour l’**offre d’image**, spécifiez `SQL2008-WS2008` ou `SQL2008R2-WS2008`.

Pour inscrire votre SQL Server 2008 (`SQL2008-WS2008`) ou 2008 R2 (`SQL2008R2-WS2008`) sur une instance Windows Server 2008, utilisez l’extrait de code Azure PowerShell ou Azure CLI suivant : 


# <a name="azure-cli"></a>[Azure CLI](#tab/bash)

Inscrire votre machine virtuelle SQL Server en mode sans agent avec l’interface de ligne de commande Azure : 

  ```azurecli-interactive
   az sql vm create -n sqlvm -g myresourcegroup -l eastus |
   --license-type <license type>  --sql-mgmt-type NoAgent 
   --image-sku Enterprise --image-offer <image offer> 
 ```
 

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)


Inscrire votre machine virtuelle SQL Server en mode sans agent avec Azure PowerShell : 

  ```powershell-interactive
  # Get the existing compute VM
  $vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
          
  New-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -Location $vm.Location `
    -LicenseType <license type> -SqlManagementType NoAgent -Sku Standard -Offer <image offer>
  ```

---

## <a name="verify-mode"></a>Vérifier le mode

Vous pouvez afficher le mode actuel de votre agent SQL Server IaaS en utilisant Azure PowerShell : 

```powershell-interactive
# Get the SqlVirtualMachine
$sqlvm = Get-AzSqlVM -Name $vm.Name  -ResourceGroupName $vm.ResourceGroupName
$sqlvm.SqlManagementType
```

## <a name="upgrade-to-full"></a>Mettre à niveau vers le mode complet  

Les machines virtuelles SQL Server qui ont inscrit l’extension en mode *léger* peuvent effectuer une mise à niveau vers le mode _complet_ à l’aide du portail Azure, de l’interface de ligne de commande Azure ou d’Azure PowerShell. Les machines virtuelles SQL Server en mode _sans agent_ peuvent passer en mode _complet_ une fois que le système d’exploitation est mis à niveau vers Windows 2008 R2 et versions ultérieures. Il est impossible de passer à une version inférieure. Pour ce faire, vous devez [désinscrire](#unregister-from-extension) la machine virtuelle SQL Server de l’extension SQL IaaS Agent. Cette opération supprime la _ressource_ de **machine virtuelle SQL**, mais ne supprime pas la machine virtuelle elle-même. 


### <a name="azure-portal"></a>Portail Azure

Pour mettre à niveau l’extension en mode complet à l’aide du portail Azure, procédez comme suit : 

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Accédez à votre ressource [machines virtuelles SQL](manage-sql-vm-portal.md#access-the-sql-virtual-machines-resource). 
1. Sélectionnez votre machine virtuelle SQL Server, puis choisissez **Vue d’ensemble**. 
1. Pour les machines virtuelles SQL Server avec les modes IaaS NoAgent ou léger, sélectionnez le message **Seules les mises à jour de type de licence et d’édition sont disponibles avec l’extension IaaS SQL**.

   ![Sélections de changement de mode à partir du portail](./media/sql-agent-extension-manually-register-single-vm/change-sql-iaas-mode-portal.png)

1. Activez la case à cocher **J’accepte de redémarrer le service SQL Server sur la machine virtuelle**, puis sélectionnez **Confirmer** pour mettre à niveau votre mode IaaS vers le mode complet. 

    ![Case à cocher J’accepte de redémarrer le service SQL Server sur la machine virtuelle](./media/sql-agent-extension-manually-register-single-vm/enable-full-mode-iaas.png)

### <a name="command-line"></a>Ligne de commande

# <a name="azure-cli"></a>[Azure CLI](#tab/bash)

Pour mettre à niveau l’extension en mode complet, exécutez l’extrait de code Azure CLI suivant :

  ```azurecli-interactive
  # Update to full mode
  az sql vm update --name <vm_name> --resource-group <resource_group_name> --sql-mgmt-type full  
  ```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Pour mettre à niveau l’extension en mode complet, exécutez l’extrait de code Azure PowerShell suivant :

  ```powershell-interactive
  # Get the existing  Compute VM
  $vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
        
  # Register with SQL IaaS Agent extension in full mode
  Update-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -SqlManagementType Full
  ```

---

## <a name="verify-registration-status"></a>Vérifier l’état de l’inscription
Vous pouvez vérifier si votre machine virtuelle SQL Server a déjà été inscrite auprès de l’extension SQL IaaS Agent via le portail Azure, l’interface de ligne de commande Azure ou Azure PowerShell. 

### <a name="azure-portal"></a>Portail Azure 

Pour vérifier l’état d’inscription à l’aide du portail Azure, procédez comme suit : 

1. Connectez-vous au [portail Azure](https://portal.azure.com). 
1. Accédez à vos [machines virtuelles SQL Server](manage-sql-vm-portal.md).
1. Sélectionnez votre machine virtuelle SQL Server dans la liste. Si votre machine virtuelle SQL Server n’est pas listée ici, il est probable qu’elle n’a pas été inscrite auprès de l’extension SQL IaaS Agent. 
1. Examinez la valeur sous **État**. Si **État** indique **Réussi**, la machine virtuelle SQL Server a bien été inscrite auprès de l’extension SQL IaaS Agent. 

   ![Vérifier l’état avec l’inscription avec le fournisseur de ressources SQL](./media/sql-agent-extension-manually-register-single-vm/verify-registration-status.png)

### <a name="command-line"></a>Ligne de commande

Vérifiez l’état d’inscription actuel d’une machine virtuelle SQL Server à l’aide d’Azure CLI ou d’Azure PowerShell. `ProvisioningState` affichera `Succeeded` si l’inscription a réussi. 

# <a name="azure-cli"></a>[Azure CLI](#tab/bash)

Pour vérifier l’état d’inscription à l’aide d’Azure CLI, exécutez l’extrait de code suivant :  

  ```azurecli-interactive
  az sql vm show -n <vm_name> -g <resource_group>
 ```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Pour vérifier l’état d’inscription à l’aide d’Azure PowerShell, exécutez l’extrait de code suivant :

  ```powershell-interactive
  Get-AzSqlVM -Name <vm_name> -ResourceGroupName <resource_group>
  ```

---

Une erreur indique que la machine virtuelle SQL Server n’a pas été inscrite auprès de l’extension. 


## <a name="unregister-from-extension"></a>Désinscrire de l’extension

Pour désinscrire votre machine virtuelle SQL Server de l’extension SQL IaaS Agent, supprimez la *ressource* de machine virtuelle SQL à l’aide du portail Azure ou de l’interface de ligne de commande Azure. La suppression de la *ressource* de machine virtuelle SQL n’entraîne pas la suppression de la machine virtuelle SQL Server proprement dite. Toutefois, suivez attentivement les étapes, car il est possible de supprimer par inadvertance la machine virtuelle lors de la tentative de suppression de la *ressource*. 

La désinscription de la machine virtuelle SQL de l’extension SQL IaaS Agent est nécessaire pour passer du mode d’administration complet à une version inférieure. 

### <a name="azure-portal"></a>Portail Azure

Pour désinscrire votre machine virtuelle SQL Server de l’extension à l’aide du portail Azure, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Accédez à la ressource de machine virtuelle SQL. 
  
   ![Ressource Machines virtuelles SQL](./media/sql-agent-extension-manually-register-single-vm/sql-vm-manage.png)

1. Sélectionnez **Supprimer**. 

   ![Sélectionnez Supprimer dans la barre de navigation supérieure](./media/sql-agent-extension-manually-register-single-vm/delete-sql-vm-resource.png)

1. Saisissez le nom de la machine virtuelle SQL et **désactivez la case à cocher à côte de celle-ci**.

   ![Décochez la machine virtuelle pour empêcher la suppression de la machine virtuelle réelle, puis sélectionnez Supprimer pour procéder à la suppression de la ressource de machine virtuelle SQL.](./media/sql-agent-extension-manually-register-single-vm/confirm-delete-of-resource-uncheck-box.png)

   >[!WARNING]
   > Si vous ne désactivez pas la case à cocher en regard du nom de la machine virtuelle, vous *supprimez entièrement* celle-ci. Désactivez la case à cocher pour désinscrire la machine virtuelle SQL Server de l’extension *sans supprimer la machine virtuelle proprement dite*. 

1. Sélectionnez **Supprimer** pour confirmer la suppression de la *ressource* de machine virtuelle SQL et non pas la machine virtuelle SQL Server. 

### <a name="command-line"></a>Ligne de commande

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour désinscrire votre machine virtuelle SQL Server de l’extension à l’aide d’Azure CLI, utilisez la commande [az sql vm delete](/cli/azure/sql/vm?view=azure-cli-latest&preserve-view=true#az-sql-vm-delete). Cela supprimera la *ressource* de machine virtuelle SQL Server, mais ne supprimera pas la machine virtuelle. 


```azurecli-interactive
az sql vm delete 
  --name <SQL VM resource name> |
  --resource-group <Resource group name> |
  --yes 
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour désinscrire votre machine virtuelle SQL Server de l’extension à l’aide d’Azure PowerShell, utilisez la commande [Remove-AzSqlVM](/powershell/module/az.sqlvirtualmachine/remove-azsqlvm). Cela supprimera la *ressource* de machine virtuelle SQL Server, mais ne supprimera pas la machine virtuelle. 

```powershell-interactive
Remove-AzSqlVM -ResourceGroupName <resource_group_name> -Name <VM_name>
```

---


## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants : 

* [Vue d’ensemble de SQL Server sur une machine virtuelle Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Questions fréquentes (FAQ) pour SQL Server sur une machine virtuelle Windows](frequently-asked-questions-faq.md)  
* [Guide des tarifs pour SQL Server sur une machine virtuelle Windows](pricing-guidance.md)
* [Notes de publication pour SQL Server sur une machine virtuelle Windows](../../database/doc-changes-updates-release-notes.md)

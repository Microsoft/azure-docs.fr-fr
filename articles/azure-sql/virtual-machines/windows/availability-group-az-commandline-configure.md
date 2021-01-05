---
title: Configurer un groupe de disponibilité (PowerShell et Azure CLI)
description: Utilisez PowerShell et Azure CLI pour créer le cluster de basculement Windows, l’écouteur de groupe de disponibilité et l’équilibreur de charge interne sur une machine virtuelle SQL Server dans Azure.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 08/20/2020
ms.author: mathoma
ms.reviewer: jroth
ms.custom: seo-lt-2019, devx-track-azurecli
ms.openlocfilehash: 865ee3a5aeb8a2dd06d8759ba04d02259d2b4bee
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359963"
---
# <a name="use-powershell-or-az-cli-to-configure-an-availability-group-for-sql-server-on-azure-vm"></a>Utiliser PowerShell ou Azure CLI pour configurer un groupe de disponibilité pour SQL Server sur une machine virtuelle Azure 
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Cet article explique comment utiliser [PowerShell](/powershell/scripting/install/installing-powershell) ou [Azure CLI](/cli/azure/sql/vm) pour déployer un cluster de basculement Windows, ajouter des machines virtuelles SQL Server au cluster et créer l’équilibreur de charge interne et l’écouteur pour un groupe de disponibilité Always On. 

Le déploiement du groupe de disponibilité Always On s’effectue encore manuellement via SQL Server Management Studio (SSMS) ou Transact-SQL (T-SQL). 

Bien que cet article utilise PowerShell ou l’interface Azure CLI pour configurer l’environnement du groupe de disponibilité, vous pouvez également utiliser [le portail Azure](availability-group-azure-portal-configure.md) à l’aide des [modèles de démarrage rapide Azure](availability-group-quickstart-template-configure.md) ou [le faire manuellement](availability-group-manually-configure-tutorial.md). 

## <a name="prerequisites"></a>Prérequis

Pour configurer un groupe de disponibilité Always On, vous devez disposer des éléments suivants : 

- Un [abonnement Azure](https://azure.microsoft.com/free/).
- Un groupe de ressources avec un contrôleur de domaine. 
- Une ou plusieurs [machines virtuelles jointes à un domaine dans Azure qui exécutent SQL Server 2016 (ou version ultérieure) Édition Entreprise](./create-sql-vm-portal.md) dans le *même* groupe à haute disponibilité ou dans des zones de disponibilité *différentes* et qui sont [inscrites auprès de l’extension SQL IaaS Agent](sql-agent-extension-manually-register-single-vm.md).  
- La version la plus récente de [PowerShell](/powershell/scripting/install/installing-powershell) ou de [Azure CLI](/cli/azure/install-azure-cli). 
- Deux adresses IP disponibles (non utilisées par une entité) : une pour l’équilibreur de charge interne et l’autre pour l’écouteur de groupe de disponibilité dans le même sous-réseau que le groupe de disponibilité. Si vous utilisez un équilibreur de charge existant, une seule adresse IP disponible suffit pour l’écouteur de groupe de disponibilité. 

## <a name="permissions"></a>Autorisations

Vous avez besoin des autorisations de compte suivantes pour configurer le groupe de disponibilité Always On à l’aide d’Azure CLI : 

- Un compte d’utilisateur de domaine existant qui dispose d’une autorisation de **création d’objet ordinateur** dans le domaine. Par exemple, un compte d’administrateur de domaine dispose généralement d’une autorisation suffisante (par exemple : account@domain.com). _Ce compte doit également faire partie du groupe administrateur local sur chaque machine virtuelle pour créer le cluster._
- Le compte d’utilisateur du domaine qui contrôle SQL Server. 
 
## <a name="create-a-storage-account"></a>Créez un compte de stockage. 

Le cluster a besoin d’un compte de stockage configuré en tant que témoin de cloud. Vous pouvez utiliser un compte de stockage existant ou en créer un. Si vous voulez utiliser un compte de stockage existant, passez à la section suivante. 

L’extrait de code suivant crée le compte de stockage : 

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
# Create the storage account
# example: az storage account create -n 'cloudwitness' -g SQLVM-RG -l 'West US' `
#  --sku Standard_LRS --kind StorageV2 --access-tier Hot --https-only true

az storage account create -n <name> -g <resource group name> -l <region> `
  --sku Standard_LRS --kind StorageV2 --access-tier Hot --https-only true
```

>[!TIP]
> Vous pouvez rencontrer l’erreur `az sql: 'vm' is not in the 'az sql' command group` si vous utilisez une version obsolète d’Azure CLI. Téléchargez la [dernière version d’Azure CLI](/cli/azure/install-azure-cli-windows) pour ne plus avoir cette erreur.


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell-interactive
# Create the storage account
# example: New-AzStorageAccount -ResourceGroupName SQLVM-RG -Name cloudwitness `
#    -SkuName Standard_LRS -Location West US -Kind StorageV2 `
#    -AccessTier Hot -EnableHttpsTrafficOnly

New-AzStorageAccount -ResourceGroupName <resource group name> -Name <name> `
    -SkuName Standard_LRS -Location <region> -Kind StorageV2 `
    -AccessTier Hot -EnableHttpsTrafficOnly
```

---

## <a name="define-cluster-metadata"></a>Définir les métadonnées du cluster

Le groupe de commandes [az sql vm group](/cli/azure/sql/vm/group) d’Azure CLI gère les métadonnées du service Cluster de basculement Windows Server (WSFC) qui héberge le groupe de disponibilité. Les métadonnées de cluster englobent le domaine Active Directory, les comptes de cluster, les comptes de stockage à utiliser en tant que témoin de cloud et la version de SQL Server. Utilisez [az sql vm group create](/cli/azure/sql/vm/group#az-sql-vm-group-create) pour définir les métadonnées de WSFC de sorte qu’à la première machine virtuelle SQL Server ajoutée, le cluster soit créé comme défini. 

L’extrait de code suivant définit les métadonnées du cluster :

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
# Define the cluster metadata
# example: az sql vm group create -n Cluster -l 'West US' -g SQLVM-RG `
#  --image-offer SQL2017-WS2016 --image-sku Enterprise --domain-fqdn domain.com `
#  --operator-acc vmadmin@domain.com --bootstrap-acc vmadmin@domain.com --service-acc sqlservice@domain.com `
#  --sa-key '4Z4/i1Dn8/bpbseyWX' `
#  --storage-account 'https://cloudwitness.blob.core.windows.net/'

az sql vm group create -n <cluster name> -l <region ex:eastus> -g <resource group name> `
  --image-offer <SQL2016-WS2016 or SQL2017-WS2016> --image-sku Enterprise --domain-fqdn <FQDN ex: domain.com> `
  --operator-acc <domain account ex: testop@domain.com> --bootstrap-acc <domain account ex:bootacc@domain.com> `
  --service-acc <service account ex: testservice@domain.com> `
  --sa-key '<PublicKey>' `
  --storage-account '<ex:https://cloudwitness.blob.core.windows.net/>'
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell-interactive
# Define the cluster metadata
# example: $group = New-AzSqlVMGroup -Name Cluster -Location West US ' 
#  -ResourceGroupName SQLVM-RG -Offer SQL2017-WS2016
#  -Sku Enterprise -DomainFqdn domain.com -ClusterOperatorAccount vmadmin@domain.com
#  -ClusterBootstrapAccount vmadmin@domain.com  -SqlServiceAccount sqlservice@domain.com 
#  -StorageAccountUrl '<ex:https://cloudwitness.blob.core.windows.net/>' `
#  -StorageAccountPrimaryKey '4Z4/i1Dn8/bpbseyWX'

$group = New-AzSqlVMGroup -Name <name> -Location <regio> 
  -ResourceGroupName <resource group name> -Offer <SQL201?-WS201?> 
  -Sku Enterprise -DomainFqdn <FQDN> -ClusterOperatorAccount <domain account> 
  -ClusterBootstrapAccount <domain account>  -SqlServiceAccount <service account> 
  -StorageAccountUrl '<ex:StorageAccountUrl>' `
  -StorageAccountPrimaryKey '<PublicKey>'
```

---

## <a name="add-vms-to-the-cluster"></a>Ajouter des machines virtuelles au cluster

Ajouter la première machine virtuelle SQL Server au cluster permet de créer le cluster. La commande [az sql vm add-to-group](/cli/azure/sql/vm#az-sql-vm-add-to-group) crée le cluster avec le nom précédemment attribué, installe le rôle de cluster sur les machines virtuelles SQL Server et les ajoute au cluster. Les utilisations suivantes de la commande `az sql vm add-to-group` ajoutent des machines virtuelles SQL Server supplémentaires au cluster nouvellement créé. 

L’extrait de code suivant crée le cluster et lui ajoute la première machine virtuelle SQL Server : 

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
# Add SQL Server VMs to cluster
# example: az sql vm add-to-group -n SQLVM1 -g SQLVM-RG --sqlvm-group Cluster `
#  -b Str0ngAzur3P@ssword! -p Str0ngAzur3P@ssword! -s Str0ngAzur3P@ssword!
# example: az sql vm add-to-group -n SQLVM2 -g SQLVM-RG --sqlvm-group Cluster `
#  -b Str0ngAzur3P@ssword! -p Str0ngAzur3P@ssword! -s Str0ngAzur3P@ssword!

az sql vm add-to-group -n <VM1 Name> -g <Resource Group Name> --sqlvm-group <cluster name> `
  -b <bootstrap account password> -p <operator account password> -s <service account password>
az sql vm add-to-group -n <VM2 Name> -g <Resource Group Name> --sqlvm-group <cluster name> `
  -b <bootstrap account password> -p <operator account password> -s <service account password>
```
Utilisez cette commande pour ajouter d’autres machines virtuelles SQL Server au cluster. Modifiez uniquement le paramètre `-n` pour le nom de machine virtuelle SQL Server. 

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell-interactive
# Add SQL Server VMs to cluster
# example: $sqlvm1 = Get-AzSqlVM -Name SQLVM1 -ResourceGroupName SQLVM-RG
# $sqlvm2 = Get-AzSqlVM -Name SQLVM2  -ResourceGroupName SQLVM-RG

# $sqlvmconfig1 = Set-AzSqlVMConfigGroup -SqlVM $sqlvm1 `
#  -SqlVMGroup $group -ClusterOperatorAccountPassword Str0ngAzur3P@ssword! `
#  -SqlServiceAccountPassword Str0ngAzur3P@ssword! `
#  -ClusterBootstrapAccountPassword Str0ngAzur3P@ssword!

# $sqlvmconfig2 = Set-AzSqlVMConfigGroup -SqlVM $sqlvm2 `
#  -SqlVMGroup $group -ClusterOperatorAccountPassword Str0ngAzur3P@ssword! `
#  -SqlServiceAccountPassword Str0ngAzur3P@ssword! `
#  - ClusterBootstrapAccountPassword Str0ngAzur3P@ssword!

$sqlvm1 = Get-AzSqlVM -Name <VM1 Name> -ResourceGroupName <Resource Group Name>
$sqlvm2 = Get-AzSqlVM -Name <VM2 Name> -ResourceGroupName <Resource Group Name>

$sqlvmconfig1 = Set-AzSqlVMConfigGroup -SqlVM $sqlvm1 `
   -SqlVMGroup $group -ClusterOperatorAccountPassword <operator account password> `
   -SqlServiceAccountPassword <service account password> `
   -ClusterBootstrapAccountPassword <bootstrap account password>

Update-AzSqlVM -ResourceId $sqlvm1.ResourceId -SqlVM $sqlvmconfig1

$sqlvmconfig2 = Set-AzSqlVMConfigGroup -SqlVM $sqlvm2 `
   -SqlVMGroup $group -ClusterOperatorAccountPassword <operator account password> `
   -SqlServiceAccountPassword <service account password> `
   -ClusterBootstrapAccountPassword <bootstrap account password>

Update-AzSqlVM -ResourceId $sqlvm2.ResourceId -SqlVM $sqlvmconfig2
```

---


## <a name="validate-cluster"></a>Valider le cluster 

Pour qu’un cluster de basculement soit pris en charge par Microsoft, il doit réussir la validation de cluster. Connectez-vous à la machine virtuelle via la méthode de votre choix, telle que le protocole RDP (Remote Desktop Protocol) et confirmez que votre cluster a réussi la validation avant de continuer. Dans le cas contraire, votre cluster se trouve dans un état non pris en charge. 

Vous pouvez valider le cluster à l’aide du Gestionnaire du cluster de basculement ou de la commande PowerShell suivante :

   ```powershell
   Test-Cluster –Node ("<node1>","<node2>") –Include "Inventory", "Network", "System Configuration"
   ```

## <a name="create-availability-group"></a>Créer un groupe de disponibilité

Créez manuellement le groupe de disponibilité comme vous le feriez normalement, en utilisant [SQL Server Management Studio](/sql/database-engine/availability-groups/windows/use-the-availability-group-wizard-sql-server-management-studio), [PowerShell](/sql/database-engine/availability-groups/windows/create-an-availability-group-sql-server-powershell) ou [Transact-SQL](/sql/database-engine/availability-groups/windows/create-an-availability-group-transact-sql). 

>[!IMPORTANT]
> Ne créez *pas* d’écouteur pour l’instant, car cette opération sera effectuée via Azure CLI dans les sections suivantes.  

## <a name="create-internal-load-balancer"></a>Créer un équilibrage de charge interne

[!INCLUDE [sql-ag-use-dnn-listener](../../includes/sql-ag-use-dnn-listener.md)]

L’écouteur de groupe de disponibilité Always On a besoin d’une instance interne d’Azure Load Balancer. L’équilibreur de charge interne fournit une adresse IP « flottante » pour l’écouteur de groupe de disponibilité afin d’accélérer les opérations de basculement et de reconnexion. Si les machines virtuelles SQL Server d’un groupe de disponibilité font partie du même groupe à haute disponibilité, vous pouvez utiliser un équilibreur de charge de base. Dans le cas contraire, vous devez utiliser un équilibreur de charge standard.  

> [!NOTE]
> L’équilibreur de charge interne doit se trouver dans le même réseau virtuel que les instances de machine virtuelle SQL Server. 

L’extrait de code suivant crée l’équilibreur de charge interne :

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
# Create the internal load balancer
# example: az network lb create --name sqlILB -g SQLVM-RG --sku Standard `
# --vnet-name SQLVMvNet --subnet default

az network lb create --name sqlILB -g <resource group name> --sku Standard `
  --vnet-name <VNet Name> --subnet <subnet name>
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell-interactive
# Create the internal load balancer
# example: New-AzLoadBalancer -name sqlILB -ResourceGroupName SQLVM-RG  `
#  -Sku Standard -Location West US

New-AzLoadBalancer -name sqlILB -ResourceGroupName <resource group name> `
   -Sku Standard -Location <region>
```

---

>[!IMPORTANT]
> La ressource d’adresse IP publique de chaque machine virtuelle SQL Server doit avoir une référence SKU standard compatible avec l’équilibreur de charge standard. Pour déterminer la référence SKU de la ressource d’adresse IP publique de votre machine virtuelle, accédez à **Groupe de ressources**, sélectionnez votre ressources **Adresse IP publique** pour la machine virtuelle SQL Server souhaitée, puis recherchez la valeur sous **Référence SKU** dans le volet **Vue d’ensemble**.  

## <a name="create-listener"></a>Créer un écouteur

Après avoir créé manuellement le groupe de disponibilité, vous pouvez créer l’écouteur à l’aide de la commande [az sql vm ag-listener](/cli/azure/sql/vm/group/ag-listener#az-sql-vm-group-ag-listener-create). 

L’*ID de ressource de sous-réseau* est la valeur de `/subnets/<subnetname>` ajoutée à l’ID de ressource de la ressource de réseau virtuel. Pour identifier l’ID de ressource de sous-réseau :
   1. Accédez au groupe de ressources sur le [portail Azure](https://portal.azure.com). 
   1. Sélectionnez la ressource de réseau virtuel. 
   1. Sélectionnez **Propriétés** dans le volet **Paramètres**. 
   1. Identifiez l’ID de ressource du réseau virtuel et ajoutez `/subnets/<subnetname>` à la fin de celui-ci pour créer l’ID de ressource de sous-réseau. Par exemple :
      - Votre ID de ressource de réseau virtuel est : `/subscriptions/a1a1-1a11a/resourceGroups/SQLVM-RG/providers/Microsoft.Network/virtualNetworks/SQLVMvNet`
      - Le nom de votre sous-réseau est : `default`
      - Votre ID de ressource de sous-réseau est donc : `/subscriptions/a1a1-1a11a/resourceGroups/SQLVM-RG/providers/Microsoft.Network/virtualNetworks/SQLVMvNet/subnets/default`


L’extrait de code suivant crée l’écouteur de groupe de disponibilité :

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
# Create the availability group listener
# example: az sql vm group ag-listener create -n AGListener -g SQLVM-RG `
#  --ag-name SQLAG --group-name Cluster --ip-address 10.0.0.27 `
#  --load-balancer sqlilb --probe-port 59999  `
#  --subnet /subscriptions/a1a1-1a11a/resourceGroups/SQLVM-RG/providers/Microsoft.Network/virtualNetworks/SQLVMvNet/subnets/default `
#  --sqlvms sqlvm1 sqlvm2

az sql vm group ag-listener create -n <listener name> -g <resource group name> `
  --ag-name <availability group name> --group-name <cluster name> --ip-address <ag listener IP address> `
  --load-balancer <lbname> --probe-port <Load Balancer probe port, default 59999>  `
  --subnet <subnet resource id> `
  --sqlvms <names of SQL VM's hosting AG replicas, ex: sqlvm1 sqlvm2>
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell-interactive

# example: New-AzAvailabilityGroupListener -Name AGListener -ResourceGroupName SQLVM-RG `
#    -AvailabilityGroupName SQLAG  -GroupName Cluster `
#    -IpAddress 10.0.0.27 -LoadBalancerResourceId sqlilb `
#    -ProbePort 59999 `
#    -SubnetId /subscriptions/a1a1-1a11a/resourceGroups/SQLVM-RG/providers/Microsoft.Network/virtualNetworks/SQLVMvNet/subnets/default `
#    -SqlVirtualMachineId sqlvm1 sqlvm2


New-AzAvailabilityGroupListener -Name <listener name> -ResourceGroupName <resource group name> `
   -AvailabilityGroupName <availability group name> -GroupName <cluster name> `
   -IpAddress <ag listener IP address> -LoadBalancerResourceId <lbid> `
   -ProbePort <Load Balancer probe port, default 59999> `
   -SubnetId <subnet resource id> `
   -SqlVirtualMachineId <names of SQL VM's hosting AG replicas>
```

---

## <a name="modify-number-of-replicas"></a>Modifier le nombre de réplicas 
Le déploiement d’un groupe de disponibilité sur des machines virtuelles SQL Server hébergées dans Azure ajoute une couche de complexité supplémentaire. Le fournisseur de ressources et le groupe de machines virtuelles gèrent désormais les ressources. De ce fait, quand vous ajoutez ou supprimez des réplicas dans le groupe de disponibilité, il est nécessaire d’effectuer une étape supplémentaire de mise à jour des métadonnées de l’écouteur avec des informations sur les machines virtuelles SQL Server. Quand vous modifiez le nombre de réplicas dans le groupe de disponibilité, vous devez aussi utiliser la commande [az sql vm group ag-listener update](/cli/azure/sql/vm/group/ag-listener#az-sql-vm-group-ag-listener-update) pour mettre à jour l’écouteur avec les métadonnées des machines virtuelles SQL Server. 


### <a name="add-a-replica"></a>Ajouter un réplica

Pour ajouter un nouveau réplica au groupe de disponibilité :

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Ajoutez la machine virtuelle SQL Server au groupe de clusters :
   ```azurecli-interactive

   # Add the SQL Server VM to the cluster group
   # example: az sql vm add-to-group -n SQLVM3 -g SQLVM-RG --sqlvm-group Cluster `
   # -b Str0ngAzur3P@ssword! -p Str0ngAzur3P@ssword! -s Str0ngAzur3P@ssword!

   az sql vm add-to-group -n <VM3 Name> -g <Resource Group Name> --sqlvm-group <cluster name> `
   -b <bootstrap account password> -p <operator account password> -s <service account password>
   ```

1. Utilisez SQL Server Management Studio pour ajouter l’instance SQL Server en tant que réplica dans le groupe de disponibilité.
1. Ajoutez les métadonnées de la machine virtuelle SQL Server à l’écouteur :

   ```azurecli-interactive
   # Update the listener metadata with the new VM
   # example: az sql vm group ag-listener update -n AGListener `
   # -g sqlvm-rg --group-name Cluster --sqlvms sqlvm1 sqlvm2 sqlvm3

   az sql vm group ag-listener update -n <Listener> `
   -g <RG name> --group-name <cluster name> --sqlvms <SQL VMs, along with new SQL VM>
   ```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Ajoutez la machine virtuelle SQL Server au groupe de clusters :

   ```powershell-interactive
   # Add the SQL Server VM to the cluster group
   # example: $sqlvm3 = Get-AzSqlVM -Name SQLVM3 -ResourceGroupName SQLVM-RG 
   # $group = Get-AzSqlVMGroup -ResourceGroupName SQLVM-RG -Name Cluster
   # $sqlvmconfig3 = Set-AzSqlVMConfigGroup -SqlVM $sqlvm3 -SqlVMGroup $group `
   #  -ClusterOperatorAccountPassword Str0ngAzur3P@ssword! `
   #  -SqlServiceAccountPassword Str0ngAzur3P@ssword! `
   #  -ClusterBootstrapAccountPassword Str0ngAzur3P@ssword!

   $sqlvm3 = Get-AzSqlVM -Name <VM1 Name> -ResourceGroupName <Resource Group Name>
   $group = Get-AzSqlVMGroup  -ResourceGroupName <resource group name> -Name <name>
   $sqlvmconfig3 = Set-AzSqlVMConfigGroup -SqlVM $sqlvm3 -SqlVMGroup $group `
   -ClusterOperatorAccountPassword <operator account password> `
   -SqlServiceAccountPassword <service account password> `
   -ClusterBootstrapAccountPassword <bootstrap account password>

   Update-AzSqlVM -ResourceId $sqlvm3.ResourceId -SqlVM $sqlvmconfig3
   ```

1. Utilisez SQL Server Management Studio pour ajouter l’instance SQL Server en tant que réplica dans le groupe de disponibilité.
1. Ajoutez les métadonnées de la machine virtuelle SQL Server à l’écouteur :

   ```powershell-interactive
   # Update the listener metadata with the new VM
   # example: Update-AzAvailabilityGroupListener -Name AGListener -ResourceGroupName SQLVM-RG `
   #   -SqlVMGroupName Cluster -SqlVirtualMachineId SQLVM3

   Update-AzAvailabilityGroupListener -Name <Listener> -ResourceGroupName <Resource Group Name> `
      -SqlVMGroupName <cluster name> -SqlVirtualMachineId <SQL VMs, along with new SQL VM> 

   ```

---

### <a name="remove-a-replica"></a>Supprimer un réplica

Pour supprimer un réplica du groupe de disponibilité :

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Supprimez le réplica du groupe de disponibilité à l’aide de SQL Server Management Studio. 
1. Supprimez les métadonnées de la machine virtuelle SQL Server de l’écouteur :
   ```azurecli-interactive
   # Update the listener metadata by removing the VM from the SQLVMs list
   # example: az sql vm group ag-listener update -n AGListener `
   # -g sqlvm-rg --group-name Cluster --sqlvms sqlvm1 sqlvm2

   az sql vm group ag-listener update -n <Listener> `
   -g <RG name> --group-name <cluster name> --sqlvms <SQL VMs that remain>
   ```
1. Supprimez la machine virtuelle SQL Server du cluster :
   ```azurecli-interactive
   # Remove the SQL VM from the cluster
   # example: az sql vm remove-from-group --name SQLVM3 --resource-group SQLVM-RG

   az sql vm remove-from-group --name <SQL VM name> --resource-group <RG name> 
   ```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Supprimez le réplica du groupe de disponibilité à l’aide de SQL Server Management Studio. 
1. Supprimez les métadonnées de la machine virtuelle SQL Server de l’écouteur :

   ```powershell-interactive
   # Update the listener metadata by removing the VM from the SQLVMs list
   # example: Update-AzAvailabilityGroupListener -Name AGListener -ResourceGroupName SQLVM-RG `
   #  -SqlVMGroupName Cluster -SqlVirtualMachineId SQLVM3

   Update-AzAvailabilityGroupListener -Name <Listener> -ResourceGroupName <Resource Group Name> `
      -SqlVMGroupName <cluster name> -SqlVirtualMachineId <SQL VMs that remain>

   ```
1. Supprimez la machine virtuelle SQL Server du cluster :

   ```powershell-interactive
   # Remove the SQL VM from the cluster
   # example: $sqlvm = Get-AzSqlVM -Name SQLVM3 -ResourceGroupName SQLVM-RG
   #  $sqlvm. SqlVirtualMachineGroup = ""
   #  Update-AzSqlVM -ResourceId $sqlvm -SqlVM $sqlvm

   $sqlvm = Get-AzSqlVM -Name <VM Name> -ResourceGroupName <Resource Group Name>
      $sqlvm. SqlVirtualMachineGroup = ""
    
    Update-AzSqlVM -ResourceId $sqlvm -SqlVM $sqlvm
   ```

---

## <a name="remove-listener"></a>Supprimer un écouteur
Si, par la suite, vous avez besoin de supprimer l’écouteur de groupe de disponibilité configuré avec Azure CLI, vous devez passer par l’extension SQL IaaS Agent. Comme l’écouteur est inscrit par l’intermédiaire de l’extension SQL IaaS Agent, le supprimer via SQL Server Management Studio ne suffit pas. 

La meilleure méthode consiste à le supprimer par l’intermédiaire de l’extension SQL IaaS Agent en utilisant l’extrait de code suivant dans l’interface de ligne de commande Azure. Cela a pour effet de supprimer les métadonnées de l’écouteur de groupe de disponibilité au niveau de l’extension SQL IaaS Agent. De même, l’écouteur de groupe de disponibilité est supprimé physiquement du groupe de disponibilité. 

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
# Remove the availability group listener
# example: az sql vm group ag-listener delete --group-name Cluster --name AGListener --resource-group SQLVM-RG

az sql vm group ag-listener delete --group-name <cluster name> --name <listener name > --resource-group <resource group name>
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell-interactive
# Remove the availability group listener
# example: Remove-AzAvailabilityGroupListener -Name AGListener `
#   -ResourceGroupName SQLVM-RG -SqlVMGroupName Cluster

Remove-AzAvailabilityGroupListener -Name <Listener> `
   -ResourceGroupName <Resource Group Name> -SqlVMGroupName <cluster name>
```

---

## <a name="remove-cluster"></a>Supprimer un cluster

Supprimez tous les nœuds du cluster pour le détruire, puis supprimez les métadonnées relatives au cluster de l’extension SQL IaaS Agent. Pour ce faire, vous pouvez utiliser Azure CLI ou PowerShell. 


# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Commencez par supprimer toutes les machines virtuelles SQL Server du cluster : 

```azurecli-interactive
# Remove the VM from the cluster metadata
# example: az sql vm remove-from-group --name SQLVM2 --resource-group SQLVM-RG

az sql vm remove-from-group --name <VM1 name>  --resource-group <resource group name>
az sql vm remove-from-group --name <VM2 name>  --resource-group <resource group name>
```

S’il s’agit des seules machines virtuelles du cluster, le cluster sera détruit. S’il existe dans le cluster d’autres machines que les machines virtuelles SQL Server qui ont été supprimées, ces autres machines virtuelles ne seront pas supprimées et le cluster ne sera pas détruit. 

Ensuite, supprimez les métadonnées relatives au cluster de l’extension SQL IaaS Agent : 

```azurecli-interactive
# Remove the cluster from the SQL VM RP metadata
# example: az sql vm group delete --name Cluster --resource-group SQLVM-RG

az sql vm group delete --name <cluster name> Cluster --resource-group <resource group name>
```



# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Tout d’abord, supprimez toutes les machines virtuelles SQL Server du cluster. Cette opération supprime physiquement les nœuds du cluster et détruit le cluster : 

```powershell-interactive
# Remove the SQL VM from the cluster
# example: $sqlvm = Get-AzSqlVM -Name SQLVM3 -ResourceGroupName SQLVM-RG
#  $sqlvm. SqlVirtualMachineGroup = ""
#  Update-AzSqlVM -ResourceId $sqlvm -SqlVM $sqlvm

$sqlvm = Get-AzSqlVM -Name <VM Name> -ResourceGroupName <Resource Group Name>
   $sqlvm. SqlVirtualMachineGroup = ""
   
   Update-AzSqlVM -ResourceId $sqlvm -SqlVM $sqlvm
```

S’il s’agit des seules machines virtuelles du cluster, le cluster est détruit. S’il existe dans le cluster d’autres machines que les machines virtuelles SQL Server qui ont été supprimées, ces autres machines virtuelles ne seront pas supprimées et le cluster ne sera pas détruit. 

Ensuite, supprimez les métadonnées relatives au cluster de l’extension SQL IaaS Agent : 

```powershell-interactive
# Remove the cluster metadata
# example: Remove-AzSqlVMGroup -ResourceGroupName "SQLVM-RG" -Name "Cluster"

Remove-AzSqlVMGroup -ResourceGroupName "<resource group name>" -Name "<cluster name> "
```

---

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants : 

* [Vue d’ensemble des machines virtuelles SQL Server](sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Questions fréquentes (FAQ) sur les machines virtuelles SQL Server](frequently-asked-questions-faq.md)
* [Notes de publication pour les machines virtuelles SQL Server](../../database/doc-changes-updates-release-notes.md)
* [Changement de modèles de licence pour une machine virtuelle SQL Server](licensing-model-azure-hybrid-benefit-ahb-change.md)
* [Vue d’ensemble des groupes de disponibilité Always On &#40;SQL Server&#41;](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server)   
* [Configuration d’une instance de serveur pour des groupes de disponibilité Always On &#40;SQL Server&#41;](/sql/database-engine/availability-groups/windows/configuration-of-a-server-instance-for-always-on-availability-groups-sql-server)   
* [Administration d’un groupe de disponibilité &#40;SQL Server&#41;](/sql/database-engine/availability-groups/windows/administration-of-an-availability-group-sql-server)   
* [Supervision de groupes de disponibilité &#40;SQL Server&#41;](/sql/database-engine/availability-groups/windows/monitoring-of-availability-groups-sql-server)
* [Vue d’ensemble des instructions Transact-SQL pour les groupes de disponibilité Always On &#40;SQL Server&#41;](/sql/database-engine/availability-groups/windows/transact-sql-statements-for-always-on-availability-groups)   
* [Vue d’ensemble des applets de commande PowerShell pour les groupes de disponibilité Always On &#40;SQL Server&#41;](/sql/database-engine/availability-groups/windows/overview-of-powershell-cmdlets-for-always-on-availability-groups-sql-server)
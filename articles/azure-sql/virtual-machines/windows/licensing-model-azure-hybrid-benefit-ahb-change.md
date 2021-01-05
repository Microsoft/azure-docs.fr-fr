---
title: Changer le modèle de licence d’une machine virtuelle SQL dans Azure
description: Découvrez comment faire passer la machine virtuelle SQL Server dans Azure de la licence avec paiement à l’utilisation à la licence BYOL (apportez votre propre licence) à l’aide d’Azure Hybrid Benefit.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.subservice: management
ms.workload: iaas-sql-server
ms.date: 11/13/2019
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: da136c1f3ce36bf85592c3c73c9e8f92c41c7442
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97357906"
---
# <a name="change-the-license-model-for-a-sql-virtual-machine-in-azure"></a>Changer le modèle de licence d’une machine virtuelle SQL dans Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]


Cet article décrit comment changer le modèle de licence d’une machine virtuelle SQL Server dans Azure à l’aide de [l’extension SQL IaaS Agent](./sql-server-iaas-agent-extension-automate-management.md).

## <a name="overview"></a>Vue d’ensemble

Il existe trois modèles de licence pour une machine virtuelle Azure qui héberge SQL Server : Paiement à l’utilisation, Azure Hybrid Benefit (AHB) et Haute disponibilité/reprise d’activité (HA/DR). En utilisant le Portail Azure, Azure CLI ou PowerShell, vous pouvez changer le modèle de licence de votre machine virtuelle SQL Server. 

- Le modèle de **paiement à l’utilisation** signifie que le coût par seconde de l’exécution de la machine virtuelle Azure comprend le coût de la licence SQL Server.
- [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/) vous permet d’utiliser votre propre licence SQL Server avec une machine virtuelle qui exécute SQL Server. 
- Le type de licence **HA/DR** est utilisé pour le [réplica HA/DR gratuit](business-continuity-high-availability-disaster-recovery-hadr-overview.md#free-dr-replica-in-azure) dans Azure. 

Microsoft Azure Hybrid Benefit permet d’utiliser des licences SQL Server avec Software Assurance (« Licence qualifiée ») sur des machines virtuelles Azure. Avec Azure Hybrid Benefit, les clients ne sont pas facturés pour l’utilisation d’une licence SQL Server sur une machine virtuelle. Toutefois, ils doivent toujours payer le coût du calcul cloud sous-jacent (c’est-à-dire, le taux de base), le stockage et les sauvegardes. Ils doivent également payer pour les E/S associées à leur utilisation des services (le cas échéant).

Selon les conditions d’utilisation du produit Microsoft : « Les clients déclarent utiliser Azure SQL Database (Managed Instance, Pool élastique et base de données unique), Azure Data Factory, SQL Server Integration Services ou SQL Server Virtual Machines sous Azure Hybrid Benefit pour SQL Server lors de la configuration des charges de travail sur Azure. »

Pour déclarer l’utilisation d’Azure Hybrid Benefit pour SQL Server sur une machine virtuelle Azure et être conforme, vous disposez de trois options :

- Approvisionnez une machine virtuelle à l’aide d’une image BYOL (apportez votre propre licence) SQL Server à partir de la Place de marché Azure. Cette option est disponible uniquement pour les clients qui disposent d’un Accord Entreprise.
- Provisionnez une machine virtuelle à l’aide d’une image BYOL (apportez votre propre licence) SQL Server à partir de la Place de marché Azure, puis activez Azure Hybrid Benefit.
- Installez automatiquement SQL Server sur une machine virtuelle Azure, [l’inscrire manuellement auprès de l’extension SQL IaaS Agent](sql-agent-extension-manually-register-single-vm.md) et activez Azure Hybrid Benefit.

Le type de licence de SQL Server peut être configuré lors de l’approvisionnement de la machine virtuelle ou à tout moment par la suite. Le passage d’un modèle à un autre n’entraîne aucun temps d’arrêt, aucun redémarrage de la machine virtuelle ou du service SQL Server, aucun coût supplémentaire et prend effet immédiatement. En fait, l’activation d'*Azure Hybrid Benefit* réduit les coûts.

## <a name="prerequisites"></a>Prérequis

Le changement de modèle de licence de votre machine virtuelle SQL Server présente les exigences suivantes : 

- Un [abonnement Azure](https://azure.microsoft.com/free/).
- Une [machine virtuelle SQL Server](./create-sql-vm-portal.md) inscrite auprès de [l’extension SQL IaaS Agent](./sql-server-iaas-agent-extension-automate-management.md).
- Vous devez disposer de [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default) pour utiliser [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/). 


## <a name="change-license-model"></a>Changer de modèle de licence

# <a name="azure-portal"></a>[Azure portal](#tab/azure-portal)

[!INCLUDE [windows-virtual-machines-sql-use-new-management-blade](../../../../includes/windows-virtual-machines-sql-new-resource.md)]

Vous pouvez modifier le modèle de licence directement à partir du portail : 

1. Ouvrez le [portail Azure](https://portal.azure.com), puis la [ressource des machines virtuelles SQL](manage-sql-vm-portal.md#access-the-sql-virtual-machines-resource) pour votre machine virtuelle SQL Server. 
1. Sélectionnez **Configurer** sous **Paramètres**. 
1. Sélectionnez l’option **Azure Hybrid Benefit** et cochez la case pour confirmer que vous disposez d’une licence SQL Server avec Software Assurance. 
1. Sélectionnez **Appliquer** en bas de la page **Configurer**. 

![Azure Hybrid Benefit dans le portail](./media/licensing-model-azure-hybrid-benefit-ahb-change/ahb-in-portal.png)


# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Vous pouvez utiliser l’interface de ligne de commande Azure pour changer votre modèle de licence.  

Indiquez les valeurs suivantes pour **license-type** :
- `AHUB` pour Azure Hybrid Benefit
- `PAYG` pour le paiement à l’utilisation
- `DR` pour activer le réplica HA/DR gratuit


```azurecli-interactive
# example: az sql vm update -n AHBTest -g AHBTest --license-type AHUB

az sql vm update -n <VMName> -g <ResourceGroupName> --license-type <license-type>
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Vous pouvez utiliser PowerShell pour changer votre modèle de licence.

Indiquez les valeurs suivantes pour **license-type** :
- `AHUB` pour Azure Hybrid Benefit
- `PAYG` pour le paiement à l’utilisation
- `DR` pour activer le réplica HA/DR gratuit


```powershell-interactive
Update-AzSqlVM -ResourceGroupName <resource_group_name> -Name <VM_name> -LicenseType <license-type>
```

---

## <a name="remarks"></a>Notes

- Les clients Azure Cloud Solution Provider (CSP) peuvent utiliser Azure Hybrid Benefit en commençant par déployer une machine virtuelle avec paiement à l’utilisation, puis en la convertissant en licence BYOL s’ils disposent d’un programme Software Assurance actif.
- Si vous supprimez votre ressource de machine virtuelle SQL, vous revenez au paramètre de licence codé en dur de l’image. 
- La possibilité de changer le modèle de licence est une fonctionnalité de l’extension SQL IaaS Agent. Pendant le déploiement d’une image de Place de marché Azure via le portail Azure, une machine virtuelle SQL Server est inscrite automatiquement auprès de l’extension. Mais les clients qui installent automatiquement SQL Server doivent [inscrire manuellement leur machine virtuelle SQL Server](sql-agent-extension-manually-register-single-vm.md). 
- L’ajout d’une machine virtuelle SQL Server à un groupe à haute disponibilité implique la recréation de la machine virtuelle. Dès lors, toute machine virtuelle ajoutée à un groupe à haute disponibilité revient au type de licence paiement par défaut à l'utilisation. Azure Hybrid Benefit doit être réactivé. 


## <a name="limitations"></a>Limites

Le changement de modèle de licence est :
   - Disponible uniquement pour les clients disposant de [Software Assurance](https://www.microsoft.com/en-us/licensing/licensing-programs/software-assurance-overview).
   - Pris en charge uniquement pour les éditions Standard et Entreprise de SQL Server. Les changements de licence ne sont pas pris en charge pour les versions Express, Web et Développeur. 
   - Pris en charge uniquement pour les machines virtuelles déployées à l’aide du modèle Azure Resource Manager. Les machines virtuelles déployées via le modèle classique ne sont pas prises en charge. 
   - Disponible uniquement pour les clouds publics ou Azure Government. 
   - Pris en charge uniquement sur les machines virtuelles dotées d’une seule interface réseau (NIC). 

> [!Note]
> Seule une licence par cœur SQL Server avec Software Assurance ou des licences d’abonnement sont éligibles pour Azure Hybrid Benefit. Si vous utilisez une licence Serveur + licence d’accès client pour SQL Server et que vous disposez de Software Assurance, vous pouvez utiliser BYOL (apportez votre propre licence) sur une image de machine virtuelle Azure SQL Server pour tirer parti de la mobilité des licences pour ces serveurs, mais vous ne pouvez pas tirer parti des autres fonctionnalités d’Azure Hybrid Benefit. 

## <a name="known-errors"></a>Erreurs connues

Passez en revue les erreurs courantes connues et leurs résolutions. 

**La ressource ’Microsoft.SqlVirtualMachine/SqlVirtualMachines/\<resource-group>’ sous le groupe de ressources ’\<resource-group>’ est introuvable.**

Cette erreur se produit lorsque vous essayez de changer le modèle de licence sur une machine virtuelle SQL Server qui n’a pas été inscrite auprès de l’extension SQL Server IaaS Agent :

`The Resource 'Microsoft.SqlVirtualMachine/SqlVirtualMachines/\<resource-group>' under resource group '\<resource-group>' was not found. The property 'sqlServerLicenseType' cannot be found on this object. Verify that the property exists and can be set.`

Vous devez inscrire votre abonnement auprès du fournisseur de ressources, puis [inscrire votre machine virtuelle SQL Server auprès de l’extension SQL IaaS Agent](sql-agent-extension-manually-register-single-vm.md). 


**La machine virtuelle ’\<vmname\>’ a plus d’une carte réseau associée**

Cette erreur se produit sur les machines virtuelles dotées de plusieurs cartes d’interface réseau. Supprimez l’une des cartes d’interface réseau avant de changer de modèle de licence. Bien que vous puissiez rajouter la carte d’interface réseau à la machine virtuelle après avoir changé de modèle de licence, les opérations dans le portail Azure telles que la sauvegarde et la mise à jour corrective automatiques ne sont plus prises en charge. 


## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants : 

* [Vue d’ensemble de SQL Server sur une machine virtuelle Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Questions fréquentes (FAQ) pour SQL Server sur une machine virtuelle Windows](frequently-asked-questions-faq.md)
* [Guide des tarifs pour SQL Server sur une machine virtuelle Windows](pricing-guidance.md)
* [Notes de publication pour SQL Server sur une machine virtuelle Windows](../../database/doc-changes-updates-release-notes.md)
* [Vue d’ensemble de l’extension SQL IaaS Agent](./sql-server-iaas-agent-extension-automate-management.md)

---
title: Qu’est-ce que l’extension SQL Server IaaS Agent ?
description: Cet article explique comment l’extension SQL Server IaaS Agent permet d’automatiser la gestion et des tâches d’administration spécifiques de SQL Server sur les machines virtuelles Azure. Celles-ci incluent des fonctionnalités, telles que la sauvegarde automatisée, la mise à jour corrective automatisée, l’intégration d’Azure Key Vault, la gestion des licences, la configuration du stockage et la gestion centralisée de toutes les instances de machine virtuelle SQL Server.
services: virtual-machines-windows
documentationcenter: ''
author: MashaMSFT
editor: ''
tags: azure-resource-manager
ms.assetid: effe4e2f-35b5-490a-b5ef-b06746083da4
ms.service: virtual-machines-sql
ms.subservice: management
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 11/07/2020
ms.author: mathoma
ms.reviewer: jroth
ms.custom: seo-lt-2019
ms.openlocfilehash: e8268630b2c108dc95ded059ce41866a14fadd0e
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359249"
---
# <a name="automate-management-with-the-sql-server-iaas-agent-extension"></a>Automatiser la gestion avec l’extension SQL Server IaaS Agent
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]


L’extension SQL Server IaaS Agent (SqlIaasExtension) s’exécute dans SQL Server sur Machines virtuelles Azure pour automatiser les tâches d’administration et de gestion. 

Cet article fournit une vue d’ensemble de l’extension. Pour installer l’extension SQL Server IaaS dans SQL Server sur des machines virtuelles Azure, consultez les articles se rapportant à l’[Installation automatique](sql-agent-extension-automatic-registration-all-vms.md), aux [Machines virtuelles uniques](sql-agent-extension-manually-register-single-vm.md) ou aux [Machines virtuelles en bloc](sql-agent-extension-manually-register-vms-bulk.md). 

## <a name="overview"></a>Vue d’ensemble

L’extension SQL Server IaaS Agent offre un certain nombre d’avantages pour SQL Server sur les machines virtuelles Azure : 

- **Avantages en termes de fonctionnalités** : L’extension déverrouille un certain nombre d’avantages en termes de fonctionnalités d’automatisation, telles que l’administration du portail, la flexibilité des licences, la sauvegarde automatisée, la mise à jour corrective automatisée et bien plus encore. Pour plus d’informations, consultez [Avantages en termes de fonctionnalités](#feature-benefits) plus loin dans cet article. 

- **Conformité** : l’extension offre une méthode simplifiée satisfaisant à l’exigence de notification à Microsoft qu’Azure Hybrid Benefit a été activé, tel que spécifié dans les conditions d’utilisation du produit. Ce processus élimine la nécessité de gérer les formulaires d’inscription de licence pour chaque ressource.  

- **Gratuit** : l’extension dans les trois modes d’administration est entièrement gratuite. Aucun coût supplémentaire n’est associé à l’extension ou au changement de modes de gestion. 

- **Gestion des licences simplifiée**  : l’extension simplifie la gestion des licences SQL Server et vous permet d’identifier rapidement les machines virtuelles SQL Server avec Azure Hybrid Benefit activé, à l’aide du portail Azure, de l’interface [Azure CLI](manage-sql-vm-portal.md) ou de PowerShell : 

   # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

   ```azurecli-interactive
   $vms = az sql vm list | ConvertFrom-Json
   $vms | Where-Object {$_.sqlServerLicenseType -eq "AHUB"}
   ```

   # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

   ```powershell-interactive
   Get-AzSqlVM | Where-Object {$_.LicenseType -eq 'AHUB'}
   ```

   ---


> [!IMPORTANT]
> L’extension SQL IaaS Agent collecte des données dans le seul but de fournir d’autres avantages aux clients lors de l’utilisation de SQL Server dans Machines virtuelles Azure. Microsoft n’utilisera pas ces données pour les audits de gestion des licences sans le consentement préalable du client. Pour plus d’informations, consultez l’[Avenant à la déclaration de confidentialité de SQL Server](/sql/sql-server/sql-server-privacy#non-personal-data).


## <a name="feature-benefits"></a>Avantages en termes de fonctionnalités 

L’extension SQL Server IaaS Agent déverrouille un certain nombre d’avantages en matière de fonctionnalités pour la gestion de votre machine virtuelle SQL Server. 

Le tableau suivant détaille ces avantages : 


| Fonctionnalité | Description |
| --- | --- |
| **Gestion du portail** | Déverrouille la [gestion dans le portail](manage-sql-vm-portal.md), ce qui vous permet de visualiser toutes vos machines virtuelles SQL Server dans un seul emplacement, et d’activer ou de désactiver les fonctionnalités spécifiques à SQL, directement à partir du portail. 
| **Sauvegarde automatisée** |Automatise la planification des sauvegardes pour toutes les bases de données pour l’instance par défaut ou une instance nommée [correctement installée](frequently-asked-questions-faq.md#administration) de SQL Server sur la machine virtuelle. Pour plus d’informations, consultez [Sauvegarde automatisée pour SQL Server dans les machines virtuelles Azure (Resource Manager)](automated-backup-sql-2014.md). |
| **Mise à jour corrective automatisée** |Configure une fenêtre de maintenance pendant laquelle les mises à jour de sécurité SQL Server et Windows importantes de votre machine virtuelle peuvent avoir lieu ; vous évitez ainsi les mises à jour pendant les heures de pointe de votre charge de travail. Pour plus d’informations, consultez [Mise à jour corrective automatisée pour SQL Server dans les machines virtuelles Azure (Resource Manager)](automated-patching.md). |
| **Intégration d’Azure Key Vault** |Permet d’installer et de configurer automatiquement Azure Key Vault sur votre machine virtuelle SQL Server. Pour plus d’informations, consultez [Configurer l’intégration du coffre de clés Azure SQL Server sur des machines virtuelles (Resource Manager)](azure-key-vault-integration-configure.md). |
| **Gestion des licences flexible** | Économisez sur le coût en [passant en toute transparence](licensing-model-azure-hybrid-benefit-ahb-change.md) de votre propre licence (également appelée Azure Hybrid Benefit) au modèle de licence avec paiement à l’utilisation, et inversement. | 
| **Version/édition flexible** | Si vous décidez de modifier la [version](change-sql-server-version.md) ou l’[édition](change-sql-server-edition.md) de SQL Server, vous pouvez mettre à jour les métadonnées dans le portail Azure sans avoir à redéployer toute la machine virtuelle SQL Server.  | 


## <a name="management-modes"></a>Modes de gestion

Vous pouvez choisir d’inscrire votre extension SQL IaaS à trois modes de gestion différents : 

- Le mode **léger** copie les fichiers binaires d’extension dans la machine virtuelle, mais n’installe pas l’agent et ne redémarre pas le service SQL Server. Le mode léger prend uniquement en charge le changement du type de licence et de l’édition de SQL Server et offre une gestion limitée du portail. Utilisez cette option pour les machines virtuelles SQL Server avec plusieurs instances, ou participant à une instance de cluster de basculement. Le mode léger est le mode d’administration par défaut lors de l’utilisation de la fonctionnalité d’[inscription automatique](sql-agent-extension-automatic-registration-all-vms.md), ou lorsqu’un type de gestion n’est pas spécifié au moment de l’inscription manuelle. Le mode léger n’a aucun impact sur la mémoire ou le processeur et il n’y a aucun coût associé. Nous vous recommandons d’abord d’inscrire votre machine virtuelle SQL Server en mode léger, puis de procéder à une mise à niveau vers le mode complet pendant une fenêtre de maintenance planifiée. 

- Le mode **complet** installe SQL IaaS Agent sur la machine virtuelle et fournit toutes les fonctionnalités, mais nécessite un redémarrage des autorisations SQL Server et administrateur système. Utilisez-la pour gérer une machine virtuelle SQL Server avec une seule instance. Le mode Full installe deux services Windows qui ont un impact minimal sur la mémoire et le processeur. L’activité de ces services est visible dans le gestionnaire des tâches. L’utilisation du mode de gestion Full est gratuite. 

- Le mode **sans agent** est dédié à SQL Server 2008 et à SQL Server 2008 R2 sur Windows Server 2008. Le mode NoAgent n’a aucun impact sur la mémoire ou l’UC. Il n’existe aucun coût associé à l’utilisation du mode de gestion sans agent, le SQL Server n’est pas redémarré et un agent n’est pas installé sur la machine virtuelle. 

Vous pouvez afficher le mode actuel de votre agent SQL Server IaaS en utilisant Azure PowerShell : 

  ```powershell-interactive
  # Get the SqlVirtualMachine
  $sqlvm = Get-AzSqlVM -Name $vm.Name  -ResourceGroupName $vm.ResourceGroupName
  $sqlvm.SqlManagementType
  ```

## <a name="installation"></a>Installation

Inscrivez votre machine virtuelle SQL Server avec l’extension SQL Server IaaS Agent pour créer la _ressource_ de **machine virtuelle SQL** dans votre abonnement ; il s’agit d’une ressource _distincte_ de la ressource de machine virtuelle. Le fait de désinscrire votre machine virtuelle SQL Server de l’extension va supprimer la _ressource_ de **machine virtuelle SQL**, mais pas la machine virtuelle elle-même.

Pendant le déploiement d’une image de machine virtuelle SQL Server de la Place de marché Azure via le portail Azure, la machine virtuelle SQL Server est inscrite automatiquement à l’extension. Toutefois, si vous choisissez d’installer SQL Server vous-même sur une machine virtuelle Azure, ou de provisionner une machine virtuelle Azure à partir d’un disque dur virtuel personnalisé, vous devez inscrire votre machine virtuelle SQL Server à de l’extension SQL IaaS pour débloquer les avantages en termes de fonctionnalités. 

L’inscription de l’extension en mode léger copie les fichiers binaires, mais n’installe pas l’agent sur la machine virtuelle. L’agent est installé sur la machine virtuelle lorsque l’extension est mise à niveau vers le mode de gestion complet. 

Il existe trois possibilités d’inscription à l’extension : 
- [Automatiquement pour toutes les machines virtuelles actuelles et futures dans un abonnement](sql-agent-extension-automatic-registration-all-vms.md)
- [Manuellement pour une seule machine virtuelle](sql-agent-extension-manually-register-single-vm.md)
- [Manuellement pour plusieurs machines virtuelles en bloc](sql-agent-extension-manually-register-vms-bulk.md)

### <a name="named-instance-support"></a>Prise en charge d’une instance nommée

L’extension SQL Server IaaS Agent fonctionne avec une instance nommée de SQL Server s’il s’agit de la seule instance SQL Server disponible sur la machine virtuelle. L’installation de l’extension échoue sur les machines virtuelles comportant plusieurs instances de SQL Server. 

Pour utiliser une instance nommée de SQL Server, déployez une machine virtuelle Azure, installez-y une instance SQL Server nommée unique, puis inscrivez-la à l’[extension SQL IaaS](sql-agent-extension-manually-register-single-vm.md).

Sinon, pour utiliser une instance nommée avec une image SQL Server de la Place de marché Azure, suivez ces étapes : 

   1. Déployez une machine virtuelle SQL Server à partir de la Place de marché Azure. 
   1. [Désinscrivez](sql-agent-extension-manually-register-single-vm.md#unregister-from-extension) la machine virtuelle SQL Server de l’extension SQL IaaS Agent. 
   1. Désinstallez complètement SQL Server sur la machine virtuelle SQL Server.
   1. Installez SQL Server avec une instance nommée sur la machine virtuelle SQL Server. 
   1. [Inscrivez la machine virtuelle à l’extension SQL IaaS Agent](sql-agent-extension-manually-register-single-vm.md#register-with-extension). 

## <a name="verify-status-of-extension"></a>Vérifier l’état de l’extension

Utilisez le portail Azure ou Azure PowerShell pour vérifier l’état de l’extension. 

### <a name="azure-portal"></a>Portail Azure

Vérifiez que l’extension est installée dans le portail Azure. 

Sélectionnez **Tous les paramètres** dans le volet de la machine virtuelle, puis sélectionnez **Extensions**. L’extension **SslIaaSExtension** doit s’afficher.

![État de l’extension SQL IaaS Agent dans le portail Azure](./media/sql-server-iaas-agent-extension-automate-management/azure-rm-sql-server-iaas-agent-portal.png)


### <a name="azure-powershell"></a>Azure PowerShell

Vous pouvez également utiliser la cmdlet Azure PowerShell **Get-AzVMSqlServerExtension** :

   ```powershell-interactive
   Get-AzVMSqlServerExtension -VMName "vmname" -ResourceGroupName "resourcegroupname"
   ```

La commande précédente confirme que l’agent est installé et fournit des informations générales sur son état. Vous pouvez obtenir des informations d’état sur la sauvegarde automatisée et la mise à jour corrective automatisée à l’aide des commandes suivantes :

   ```powershell-interactive
    $sqlext = Get-AzVMSqlServerExtension -VMName "vmname" -ResourceGroupName "resourcegroupname"
    $sqlext.AutoPatchingSettings
    $sqlext.AutoBackupSettings
   ```


## <a name="limitations"></a>Limites

L’extension SQL IaaS Agent prend uniquement en charge les éléments suivants : 

- Les machines virtuelles SQL Server déployées par le biais d’Azure Resource Manager. Les machines virtuelles SQL Server déployées via le modèle classique ne sont pas prises en charge. 
- Les machines virtuelles SQL Server déployées sur le cloud public ou le cloud Azure Government. Les déploiements sur d’autres clouds privés ou du secteur public ne sont pas pris en charge. 


## <a name="frequently-asked-questions"></a>Forum aux questions 

**Dois-je inscrire ma machine virtuelle SQL Server provisionnée à partir d’une image SQL Server de la Place de marché Azure ?**

Non. Microsoft inscrit automatiquement les machines virtuelles provisionnées à partir des images SQL Server de la Place de marché Azure. L’inscription à l’extension n’est nécessaire que si la machine virtuelle n’a *pas* été provisionnée à partir des images SQL Server de la Place de marché Azure et que SQL Server a été installé automatiquement.

**L’extension SQL IaaS Agent est-elle disponible pour tous les clients ?** 

Oui. Les clients doivent inscrire leurs machines virtuelles SQL Server à l’extension s’ils n’ont pas utilisé d’image SQL Server de la Place de marché Azure et qu’à la place, ils ont installé SQL Server automatiquement ou apporté leur disque dur virtuel personnalisé. Les machines virtuelles relevant de tous les types d’abonnement (Direct, Contrat Entreprise et Fournisseur de solutions Cloud) peuvent s’inscrire à l’extension SQL IaaS Agent.

**Quel est le mode de gestion par défaut au moment de l’inscription de l’extension SQL IaaS Agent ?**

Le mode d’administration par défaut au moment de l’inscription à l’extension SQL IaaS Agent est *léger*. Si la propriété de gestion SQL Server n’est pas définie au moment de l’inscription à de l’extension, le mode d’administration léger est défini et votre service SQL Server ne redémarre pas. Nous vous recommandons d’abord d’effectuer l’inscription à l’extension SQL IaaS Agent en mode léger, puis de procéder à une mise à niveau vers le mode complet pendant une fenêtre de maintenance. De même, l’administration par défaut est également en mode léger quand vous utilisez la [fonctionnalité d’inscription automatique](sql-agent-extension-automatic-registration-all-vms.md).

**Quels sont les prérequis pour s’inscrire à l’extension SQL IaaS Agent ?**

Il n’existe aucun prérequis à l’inscription à l’extension SQL IaaS Agent, à part l’installation de SQL Server sur la machine virtuelle. Notez que si l’extension SQL IaaS Agent est installée en mode complet, le service SQL Server redémarre ; l’exécution de cette opération est donc recommandée pendant une fenêtre de maintenance.

**L’inscription à l’extension SQL IaaS Agent a-t-elle pour effet d’installer un agent sur ma machine virtuelle ?**

Oui, l’inscription à l’extension SQL IaaS Agent en mode d’administration complet installe un agent sur la machine virtuelle. L’inscription en mode léger ou sans agent ne le fait pas. 

L’inscription à l’extension SQL IaaS Agent en mode léger copie uniquement les *fichiers binaires* de l’extension SQL IaaS Agent sur la machine virtuelle, elle n’installe pas l’agent. Ces fichiers binaires sont ensuite utilisés pour installer l’agent lorsque le mode d’administration est mis à niveau vers complet.


**L’inscription à l’extension SQL IaaS Agent a-t-elle pour effet le redémarrage de SQL Server sur ma machine virtuelle ?**

Cela dépend du mode spécifié lors de l’inscription. Si le mode léger ou sans agent est spécifié, le service SQL Server ne redémarre pas. En revanche, si vous spécifiez le mode d’administration complet, le service SQL Server redémarre. La fonctionnalité d’inscription automatique inscrit vos machines virtuelles SQL Server en mode léger, sauf si la version de Windows Server est 2008, auquel cas la machine virtuelle SQL Server sera inscrite en mode sans agent. 

**Quelle est la différence entre les modes d’administration léger et sans agent au moment de l’inscription à l’extension SQL IaaS Agent ?** 

Le mode d’administration sans agent est le seul mode d’administration disponible pour les instances SQL Server 2008 et SQL Server 2008 R2 sur Windows Server 2008. Pour toutes les versions ultérieures de Windows Server, les deux modes d’administration disponibles sont léger et complet. 

Pour le mode sans agent, les propriétés de version et d’édition de SQL Server doivent être définies par le client. Le mode léger interroge la machine virtuelle pour déterminer la version et l’édition de l’instance SQL Server.

**Puis-je m’inscrire à l’extension SQL IaaS Agent sans spécifier le type de licence SQL Server ?**

Non. Le type de licence SQL Server n’est pas une propriété facultative au moment de vous inscrire à l’extension SQL IaaS Agent. Vous devez définir le type de licence « paiement à l’utilisation » ou Azure Hybrid Benefit au moment de vous inscrire à l’extension SQL IaaS Agent dans tous les modes d’administration (sans agent, léger et complet). Si vous disposez d’une version gratuite de SQL Server installée, comme l’édition Developer ou Evaluation, vous devez vous inscrire avec une licence de paiement à l’utilisation. Azure Hybrid Benefit n’est disponible que pour les versions payantes de SQL Server, telles que les éditions Enterprise et Standard.

**Puis-je mettre à niveau l’extension SQL Server IaaS du mode sans agent vers le mode complet ?**

Non. La mise à niveau vers le mode d’administration complet ou léger n’est pas disponible pour le mode sans agent. Il s’agit d’une limitation technique de Windows Server 2008. Vous devez d’abord mettre à niveau le système d’exploitation vers Windows Server 2008 R2 ou une version ultérieure, puis effectuer une mise à niveau vers le mode de gestion complet. 

**Puis-je mettre à niveau l’extension IaaS SQL Server du mode léger vers le mode complet ?**

Oui. La mise à niveau du mode d’administration léger à complet est prise en charge via Azure PowerShell ou le portail Azure. Un redémarrage du service SQL Server sera déclenché.

**Puis-je rétrograder l’extension SQL Server IaaS du mode complet au mode d’administration sans agent ou léger ?**

Non. La rétrogradation du mode de gestion de l’extension IaaS SQL Server n’est pas prise en charge. Le mode d’administration ne peut pas être rétrogradé du mode complet vers le mode léger ou sans agent, ni du mode léger vers le mode sans agent. 

Pour passer d’un mode d’administration complet à un autre mode d’administration, [désinscrivez](sql-agent-extension-manually-register-single-vm.md#unregister-from-extension) la machine virtuelle SQL Server de l’extension SQL IaaS Agent en supprimant la _ressource_ de machine virtuelle SQL, puis en réinscrivant la machine virtuelle SQL Server à l’extension SQL IaaS Agent avec un mode d’administration différent.

**Puis-je m’inscrire à l’extension SQL IaaS Agent à partir du portail Azure ?**

Non. L’inscription à l’extension SQL IaaS Agent n’est pas disponible sur le portail Azure. L’inscription à l’extension SQL IaaS Agent est uniquement prise en charge avec l’interface Azure CLI ou Azure PowerShell. 

**Puis-je inscrire une machine virtuelle à l’extension SQL IaaS Agent avant d’installer SQL Server ?**

Non. Une machine virtuelle doit disposer d’au moins une instance SQL Server (moteur de base de données) pour pouvoir être inscrite à l’extension SQL IaaS Agent. S’il n’existe aucune instance SQL Server sur la machine virtuelle, la nouvelle ressource Microsoft.SqlVirtualMachine sera en état d’échec.

**Puis-je inscrire une machine virtuelle à l’extension SQL IaaS Agent s’il existe plusieurs instances SQL Server ?**

Oui. L’extension SQL IaaS Agent n'inscrira qu'une seule instance de SQL Server (moteur de base de données). L’extension SQL IaaS Agent inscrira l’instance SQL Server par défaut en présence de plusieurs instances. En l’absence d’instance par défaut, l’inscription uniquement en mode léger est prise en charge. Pour effectuer une mise à niveau du mode de gestion léger à complet, l’instance SQL Server par défaut doit exister ou la machine virtuelle ne doit avoir qu’une seule instance SQL Server nommée.

**Puis-je inscrire une instance de cluster de basculement SQL Server à l’extension SQL IaaS Agent ?**

Oui. Les instances de cluster de basculement SQL Server sur une machine virtuelle Azure peuvent être inscrites à l’extension SQL IaaS Agent en mode léger. Cependant, les instances de cluster de basculement SQL Server ne peuvent pas être mises à niveau vers le mode de gestion complet.

**Puis-je inscrire ma machine virtuelle à l’extension SQL IaaS Agent si le groupe de disponibilité Always On est configuré ?**

Oui. Il n’existe aucune restriction quant à l’inscription d’une instance SQL Server sur une machine virtuelle Azure à l’extension SQL IaaS Agent si vous faites partie d’une configuration de groupe de disponibilité Always On.

**Quel est le coût de l’inscription à l’extension SQL IaaS Agent ou de la mise à niveau vers le mode de gestion complet ?**

Aucun. L’inscription à l’extension ou l’utilisation de l’un des trois modes de gestion n’est pas facturée. La gestion de votre machine virtuelle SQL Server avec l’extension est entièrement gratuite. 

**Quel est l’impact de l’utilisation des différents modes de gestion sur les performances ?**

Les modes de gestion *NoAgent* et *Lightweight* n’ont pas d’impact sur la mémoire et l’UC. L’utilisation du mode de gestion *Full* à partir de deux services installés sur le système d’exploitation a un impact minime. Ces derniers peuvent être supervisés par le biais du gestionnaire des tâches et observés dans la console des services Windows intégrée. 

Les deux noms de service sont les suivants :
- `SqlIaaSExtensionQuery` (nom d’affichage : `Microsoft SQL Server IaaS Query Service`)
- `SQLIaaSExtension` (nom d’affichage : `Microsoft SQL Server IaaS Agent`)

**Comment supprimer l’extension ?**

Supprimez l’extension en [désinscrivant](sql-agent-extension-manually-register-single-vm.md#unregister-from-extension) la machine Virtuelle SQL Server à l’extension SQL IaaS Agent. 

## <a name="next-steps"></a>Étapes suivantes

Pour installer l’extension SQL Server IaaS dans SQL Server sur des machines virtuelles Azure, consultez les articles se rapportant à l’[Installation automatique](sql-agent-extension-automatic-registration-all-vms.md), aux [Machines virtuelles uniques](sql-agent-extension-manually-register-single-vm.md) ou aux [Machines virtuelles en bloc](sql-agent-extension-manually-register-vms-bulk.md).

Pour plus d’informations sur l’exécution de SQL Server sur des machines virtuelles Azure, voir [Qu’est-ce que SQL Server sur les machines virtuelles Azure ?](sql-server-on-azure-vm-iaas-what-is-overview.md).

---
title: Exécuter une machine virtuelle SQL Server sur un hôte Azure Dedicated Host
description: Découvrez comment exécuter votre machine virtuelle SQL Server sur un hôte Azure Dedicated Host.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.subservice: management
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 08/12/2019
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 957e458bf21eb4a95b1faba1daf0459271648310
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97358688"
---
# <a name="run-sql-server-vm-on-an-azure-dedicated-host"></a>Exécuter une machine virtuelle SQL Server sur un hôte Azure Dedicated Host 
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Cet article détaille les spécificités de l’utilisation d’une machine virtuelle SQL Server avec [Azure Dedicated Host](../../../virtual-machines/dedicated-hosts.md). Des informations complémentaires sur Azure Dedicated Host sont disponibles dans le billet de blog [Présentation des hôtes dédiés Azure](https://azure.microsoft.com/blog/introducing-azure-dedicated-host/). 

## <a name="overview"></a>Vue d’ensemble
Un [hôte dédié Azure](../../../virtual-machines/dedicated-hosts.md) est un service qui fournit des serveurs physiques (capables d’héberger une ou plusieurs machines virtuelles) dédiés à un abonnement Azure. Les hôtes dédiés sont les mêmes serveurs physiques que ceux utilisés dans les centres de données de Microsoft, fournis en tant que ressources. Vous pouvez approvisionner des hôtes dédiés au sein d’une région, d’une zone de disponibilité et d’un domaine d’erreur. Ensuite, vous pouvez placer des machines virtuelles directement dans vos hôtes approvisionnés, dans la configuration qui répond le mieux à vos besoins.

## <a name="limitations"></a>Limites

- Toutes les séries de machines virtuelles ne sont pas prises en charge sur des hôtes dédiés, et la disponibilité des séries de machines virtuelles varie selon les régions. Pour plus d’informations, consultez la [présentation détaillée des hôtes dédiés Azure](../../../virtual-machines/dedicated-hosts.md).

## <a name="licensing"></a>Licence

Vous avez le choix entre deux options de licence quand vous installez votre machine virtuelle SQL Server sur un hôte Azure Dedicated Host. 

  - **Gestion des licences de machines virtuelles SQL** : Il s’agit de l’option existante de gestion des licences, où vous payez pour chaque licence de machine virtuelle SQL Server. 
  - **Gestion des licences d’hôtes dédiés** : Nouveau modèle de licence disponible pour l’hôte dédié Azure, où les licences SQL Server sont regroupées en bundle et payées au niveau de l’hôte. 


Options au niveau de l’hôte pour l’utilisation des licences SQL Server existantes : 
  - Microsoft SQL Server Édition Entreprise – Azure Hybrid Benefit (AHB)
    - Accessible aux clients disposant d’un Contrat de service ou d’un abonnement.
    - Affectez une licence à tous les cœurs physiques disponibles, et tirez parti d’une virtualisation illimitée (en fonction du nombre maximal de processeurs virtuels pris en charge par l’hôte).
        - Pour plus d’informations sur l’application de Microsoft Azure Hybrid Benefit à Azure Dedicated Host, consultez les [FAQ sur Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/faq/). 
  - Licences SQL Server acquises avant le 1er octobre
      - SQL Server Édition Entreprise dispose à la fois des options de licence au niveau de l’hôte et des options de licence par machine virtuelle. 
      - Microsoft SQL Server Édition Standard dispose uniquement d’une option de licence par machine virtuelle. 
          - Pour plus d’informations, consultez les [conditions relatives au produit de Microsoft](https://www.microsoft.com/licensing/product-licensing/products). 
  - Si aucune option au niveau de l’hôte dédié SQL Server n’est sélectionnée, vous pouvez choisir SQL Server AHB au niveau de machines virtuelles individuelles, comme c’est le cas pour des machines virtuelles multilocataires.



## <a name="provisioning"></a>Approvisionnement  
L’approvisionnement d’une machine virtuelle SQL Server sur l’hôte dédié n’est pas différent de celui d’une autre machine virtuelle Azure. Vous pouvez utiliser [Azure PowerShell](../../../virtual-machines/windows/dedicated-hosts-powershell.md), le [Portail Azure](../../../virtual-machines/dedicated-hosts-portal.md) et [Azure CLI](../../../virtual-machines/linux/dedicated-hosts-cli.md).

Le processus d’ajout d’une machine virtuelle SQL Server existante à l’hôte dédié nécessite un temps d’arrêt. Toutefois, cela n’a pas d’impact sur les données et n’entraîne aucune perte de ces dernières. Néanmoins, toutes les bases de données, notamment les bases de données système, doivent être sauvegardées avant le déplacement.

## <a name="virtualization"></a>Virtualisation 

La virtualisation illimitée est l’un des avantages d’un hôte dédié. Par exemple, vous pouvez avoir des licences pour 64 vCores, mais il est possible de configurer l’hôte pour en avoir 128. Ainsi, vous disposez du double de vCores, mais vous ne payez que la moitié du montant attendu pour les licences SQL Server. 

Comme il s’agit de votre hôte, vous pouvez configurer la virtualisation avec un ratio de 1:2. 

## <a name="faq"></a>Questions fréquentes (FAQ)

**Q : Comment Azure Hybrid Benefit fonctionne-t-il avec les licences Windows Server/SQL Server sur l’hôte dédié Azure ?**

A : Les clients peuvent utiliser leurs licences Windows Server et SQL Server existantes avec le programme Software Assurance, ou des licences d’abonnement éligibles, pour bénéficier d’un tarif réduit sur l’hôte dédié Azure via Azure Hybrid Benefit. Les clients Windows Server Datacenter et SQL Server Édition Entreprise bénéficient d’une virtualisation illimitée (ils peuvent déployer autant de machines virtuelles Windows Server que possible sur l’hôte, en fonction de la capacité physique du serveur sous-jacent) quand ils affectent une licence à l’ensemble de l’hôte et qu’ils utilisent Azure Hybrid Benefit.  Toutes les charges de travail Windows Server et SQL Server de l’hôte dédié Azure sont également éligibles aux mises à jour de sécurité étendues pour Windows Server et SQL Server 2008/R2, sans frais supplémentaires. 

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants : 

* [Vue d’ensemble de SQL Server sur une machine virtuelle Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Questions fréquentes (FAQ) pour SQL Server sur une machine virtuelle Windows](frequently-asked-questions-faq.md)
* [Guide des tarifs pour SQL Server sur une machine virtuelle Windows](pricing-guidance.md)
* [Notes de publication pour SQL Server sur une machine virtuelle Windows](doc-changes-updates-release-notes.md)
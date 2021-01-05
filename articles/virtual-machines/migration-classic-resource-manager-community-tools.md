---
title: Outils de la communauté - Déplacer des ressources classiques vers Azure Resource Manager
description: Cet article répertorie les outils qui ont été fournis par la communauté afin de faciliter la migration de ressources IaaS à partir de déploiements classiques vers le modèle de déploiement Azure Resource Manager.
author: tanmaygore
manager: vashan
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: conceptual
ms.date: 02/06/2020
ms.author: tagore
ms.openlocfilehash: 75ca87836a429a81f28b133e329291143c03ea68
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94904631"
---
# <a name="community-tools-to-migrate-iaas-resources-from-classic-to-azure-resource-manager"></a>Outils de la communauté permettant de faire migrer des ressources IaaS à partir de déploiements classiques vers Azure Resource Manager

> [!IMPORTANT]
> Aujourd'hui, environ 90 % des machines virtuelles IaaS utilisent [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/). Depuis le 28 février 2020, les machines virtuelles classiques sont dépréciées. Elles seront entièrement mises hors service le 1er mars 2023. [Apprenez-en davantage]( https://aka.ms/classicvmretirement) sur cette désapprobation et [son impact sur vous](classic-vm-deprecation.md#how-does-this-affect-me).

Cet article répertorie les outils qui ont été fournis par la communauté afin de faciliter la migration de ressources IaaS à partir de déploiements classiques vers le modèle de déploiement Azure Resource Manager.

> [!NOTE]
> Ces outils ne sont pas pris en charge officiellement par le Support Microsoft. Ils sont donc disponibles en open source sur GitHub, et nous acceptons volontiers les demandes de publication de correctifs ou de scénarios supplémentaires. Pour signaler un problème, utilisez la fonctionnalité de problèmes GitHub.
> 
> Une migration avec ces outils entraînera un temps d’arrêt pour votre machine virtuelle Classic. Si vous souhaitez effectuer une migration prise en charge par la plateforme, consultez les articles suivants : 
> 
>   * [Migration prise en charge par la plateforme de ressources IaaS Classic vers Azure Resource Manager](migration-classic-resource-manager-overview.md)
>   * [Étude technique approfondie de la migration prise en charge par la plateforme de ressources Classic vers Azure Resource Manager](migration-classic-resource-manager-deep-dive.md)
>   * [Faire migrer des ressources IaaS Classic vers Azure Resource Manager à l’aide d’Azure PowerShell](migration-classic-resource-manager-ps.md)
> 
> 

## <a name="asmmetadataparser"></a>AsmMetadataParser
Il s’agit d’un ensemble d’outils d’assistance créé dans le cadre de migrations d’entreprise depuis la gestion des services Azure vers Azure Resource Manager. Cet outil vous permet de répliquer votre infrastructure dans un autre abonnement, qui peut être utilisé pour tester la migration et mettre à plat tous les problèmes avant d’exécuter la migration sur votre abonnement de production.

[Lien vers la documentation des outils](https://github.com/Azure/classic-iaas-resourcemanager-migration/tree/master/AsmToArmMigrationApiToolset)

## <a name="migaz"></a>migAz
migAz est une option supplémentaire pour migrer un ensemble complet de ressources IaaS classiques vers des ressources Azure Resource Manager IaaS. La migration peut être exécutée dans le même abonnement ou entre différents abonnements et types d’abonnements (par ex. : les abonnements CSP).

[Lien vers la documentation des outils](https://github.com/Azure/migAz)

## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble de la migration prise en charge par la plateforme de ressources IaaS Classic vers Azure Resource Manager](migration-classic-resource-manager-overview.md)
* [Étude technique approfondie de la migration prise en charge par la plateforme de ressources Classic vers Azure Resource Manager](migration-classic-resource-manager-deep-dive.md)
* [Planification de la migration des ressources IaaS d’Azure Classic vers Azure Resource Manager](migration-classic-resource-manager-plan.md)
* [Faire migrer des ressources IaaS Classic vers Azure Resource Manager à l’aide d’Azure PowerShell](migration-classic-resource-manager-ps.md)
* [Faire migrer des ressources IaaS Classic vers Azure Resource Manager à l’aide de l’interface de ligne de commande Azure](migration-classic-resource-manager-cli.md)
* [Passer en revue les erreurs courantes de migration](migration-classic-resource-manager-errors.md)
* [Passez en revue les questions fréquemment posées sur la migration des ressources IaaS de Classic vers Azure Resource Manager](migration-classic-resource-manager-faq.md)

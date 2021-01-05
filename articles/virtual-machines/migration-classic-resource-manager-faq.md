---
title: Questions fréquemment posées sur la migration du modèle classique vers le modèle Azure Resource Manager
description: Questions fréquemment posées sur la migration du modèle classique vers le modèle Azure Resource Manager
author: tanmaygore
manager: vashan
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: conceptual
ms.date: 02/06/2020
ms.author: tagore
ms.openlocfilehash: 24e52f517f46de06fef8aa52e889185826c20d44
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498393"
---
# <a name="frequently-asked-questions-about-classic-to-azure-resource-manager-migration"></a>Questions fréquemment posées sur la migration du modèle classique vers le modèle Azure Resource Manager

> [!IMPORTANT]
> Aujourd’hui, environ 90 % des machines virtuelles IaaS utilisent [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/). Depuis le 28 février 2020, les machines virtuelles classiques sont dépréciées. Elles seront entièrement mises hors service le 1er mars 2023. [Apprenez-en davantage]( https://aka.ms/classicvmretirement) sur cette dépréciation et [son impact sur vous](./classic-vm-deprecation.md#how-does-this-affect-me).

## <a name="what-is-azure-service-manager-and-what-does-it-mean-by-classic"></a>Qu’est-ce qu’Azure Service Manager et qu’est-ce que signifie « classic » ?

Le mot « Classic » dans la machine virtuelle IaaS (Classic) fait référence aux machines virtuelles gérées par Azure Service Manager (ASM). Azure Service Manager (ASM) est l’ancien plan de contrôle d’Azure chargé de la création, de la gestion, de la suppression des machines virtuelles et de l’exécution d’autres opérations de plan de contrôle. 

## <a name="what-is-azure-resource-manager"></a>Qu’est-ce qu’Azure Resource Manager ?

[Azure Resource Manager](../azure-resource-manager/management/overview.md) est le plan de contrôle d’Azure le plus récent chargé de la création, de la gestion, de la suppression des machines virtuelles et de l’exécution d’autres opérations de plan de contrôle. 

## <a name="what-is-the-time-required-for-migration"></a>Quel est le temps nécessaire pour une migration ?

La planification et l’exécution de la migration dépend fortement de la complexité de l’architecture et peut prendre deux ou trois mois.  

## <a name="what-is-the-definition-of-a-new-customer-on-iaas-vms-classic"></a>Quelle est la définition d’un nouveau client sur des machines virtuelles IaaS (Classic) ?

Les clients qui n’avaient pas de machines virtuelles IaaS (Classic) dans leurs abonnements en février 2020 (un mois avant le début de la dépréciation) sont considérés comme de nouveaux clients. 

## <a name="what-is-the-definition-of-an-existing-customer-on-iaas-virtual-machines-classic"></a>Quelle est la définition d’un client existant sur des machines virtuelles IaaS (Classic) ?

Les clients qui avaient des machines virtuelles IaaS actives ou arrêtées mais allouées (Classic) dans leurs abonnements au mois de février 2020, sont considérés comme des clients existants. Seuls ces clients ont jusqu’au 1er mars 2023 pour migrer leurs machines virtuelles d’Azure Service Manager vers Azure Resource Manager. 

## <a name="why-am-i-getting-an-error-stating-newclassicvmcreationnotallowedforsubscription"></a>Pourquoi est-ce que j’obtiens une erreur indiquant « NewClassicVMCreationNotAllowedForSubscription » ?

Dans le cadre du processus de mise hors service, les machines virtuelles IaaS (Classic) ne sont plus disponibles pour de nouveaux clients. Étant donné que nous vous avons identifié comme nouveau client, votre opération n’a pas été autorisée. Nous vous recommandons vivement d'utiliser Azure Resource Manager. Si vous ne pouvez pas utiliser de machines virtuelles Azure à l'aide d'Azure Resource Manager, contactez le support pour ajouter votre abonnement à la liste autorisée.

## <a name="does-this-migration-plan-affect-any-of-my-existing-services-or-applications-that-run-on-azure-virtual-machines"></a>Ce plan de migration affecte-t-il l’un de mes services ou applications existants qui s’exécutent sur des machines virtuelles Azure ? 

Pas jusqu’au 1er mars 2023 pour les machines virtuelles IaaS (Classic). Les machines virtuelles IaaS (Classic) sont des services entièrement pris en charge en disponibilité générale. Vous pouvez continuer à utiliser ces ressources pour développer votre empreinte sur Microsoft Azure. Le 1er mars 2023, ces machines virtuelles seront toutes mises hors service et toutes les machines virtuelles actives ou allouées seront arrêtées et libérées.

Il n’y aura aucun impact sur d’autres ressources classiques comme les services cloud (Classic), les comptes de stockage (Classic), etc.

## <a name="what-happens-to-my-vms-if-i-dont-plan-on-migrating-in-the-near-future"></a>Que se passera-t-il pour mes machines virtuelles si je n’envisage pas de procéder à cette migration dans un avenir proche ? 

Le 1er mars 2023, les machines virtuelles IaaS (Classic) seront toutes mises hors service et toutes les machines virtuelles actives ou allouées seront arrêtées et libérées. Pour éviter tout impact sur l’activité, nous vous recommandons vivement de commencer à planifier votre migration aujourd’hui et à la terminer avant le 1er mars 2023. Nous ne déconseillons pas l’utilisation des API, des services cloud ni du modèle de ressource Classic existants. Compte tenu des fonctionnalités avancées offertes par le modèle de déploiement Resource Manager, notre objectif est de faciliter la migration. Nous vous recommandons de commencer à planifier la migration de ces ressources vers Azure Resource Manager. 

## <a name="what-does-this-migration-plan-mean-for-my-existing-tooling"></a>Quelles sont les implications de ce plan de migration pour mes outils existants ? 

La mise à jour de vos outils vers le modèle de déploiement Resource Manager constitue l’un des plus importants changements à prendre en compte dans vos plans de migration.

## <a name="how-long-will-the-management-plane-downtime-be"></a>Quelle sera la durée de l’arrêt du plan de gestion ? 

Cette durée dépend du nombre de ressources dont vous effectuez la migration. Pour les déploiements de plus petite taille (quelques dizaines de machines virtuelles), le processus de migration complet devrait prendre moins d’une heure. Pour les déploiements à grande échelle (plusieurs centaines de machines virtuelles), la migration peut prendre quelques heures.

## <a name="can-i-roll-back-after-my-migrating-resources-are-committed-in-resource-manager"></a>Puis-je procéder à une restauration après avoir validé la migration de mes ressources dans Resource Manager ? 

Vous pouvez abandonner la migration tant que vos ressources se trouvent à l’état Préparé. La restauration n’est plus prise en charge une fois que les ressources ont fait l’objet d’une migration par le biais de l’opération de validation.

## <a name="can-i-roll-back-my-migration-if-the-commit-operation-fails"></a>Puis-je restaurer ma migration en cas d’échec de l’opération de validation ? 

Vous ne pouvez pas abandonner la migration si l’opération de validation échoue. Toutes les opérations de migration, notamment l’opération de validation, sont idempotentes. Nous vous recommandons donc de retenter l’opération après une courte période. Si vous rencontrez toujours une erreur, créez un ticket de support.

## <a name="do-i-have-to-buy-another-express-route-circuit-if-i-have-to-use-iaas-under-resource-manager"></a>Dois-je acheter un autre circuit ExpressRoute si je dois utiliser la ressource IaaS sous Resource Manager ? 

Non. Nous avons récemment activé la [migration de circuits ExpressRoute du modèle de déploiement classique vers le modèle de déploiement Resource Manager](../expressroute/expressroute-move.md). Vous n’avez pas besoin d’acheter de nouveau circuit ExpressRoute si vous en possédez déjà un.

## <a name="what-if-i-had-configured-azure-role-based-access-control-policies-for-my-classic-iaas-resources"></a>Que se passera-t-il si j'ai configuré des stratégies de contrôle d'accès en fonction du rôle Azure pour mes ressources IaaS Classic ? 

Pendant la migration, les ressources Classic sont converties en ressources Resource Manager. Nous vous recommandons donc de planifier les mises à jour de stratégies Azure RBAC qui seront nécessaires après la migration.

## <a name="i-backed-up-my-classic-vms-in-a-vault-can-i-migrate-my-vms-from-classic-mode-to-resource-manager-mode-and-protect-them-in-a-recovery-services-vault"></a>J’ai sauvegardé mes machines virtuelles Classic dans un coffre. Puis-je migrer mes machines virtuelles du mode Classic vers le mode Ressource Manager et les protéger dans un coffre Recovery Services ?

Lorsque vous faites évoluer une machine virtuelle du mode Classic vers le mode Resource Manager, les sauvegardes effectuées avant cette migration ne sont pas transférées vers la nouvelle machine virtuelle Resource Manager. Toutefois, si vous souhaitez conserver vos sauvegardes des machines virtuelles Classic, procédez comme suit avant la migration. 

1. Dans le coffre Recovery Services, accédez à l’onglet **Éléments protégés**, puis sélectionnez la machine virtuelle. 
2. Cliquez sur Arrêter la protection. Laissez l’option *Supprimer les données de sauvegarde associées* **non cochée**.

> [!NOTE]
> Les frais de sauvegarde vous seront facturés tant que vous conserverez les données. Les copies de sauvegarde seront supprimées en fonction de la durée de rétention définie. Toutefois, la dernière copie de sauvegarde est systématiquement conservée jusqu’à ce que vous supprimiez explicitement les données de sauvegarde. Il est recommandé de vérifier la durée de rétention de la machine virtuelle et de déclencher l’option « Supprimer les données de sauvegarde » sur l’élément protégé dans le coffre à l’issue de la durée de rétention. 
>
>

Pour effectuer la migration de la machine virtuelle en mode Resource Manager : 

1. Supprimez l’extension de sauvegarde/capture instantanée de la machine virtuelle.
2. Migrez la machine virtuelle du mode Classic vers le mode Resource Manager. Vérifiez que les informations relatives au stockage et au réseau correspondant à la machine virtuelle sont également migrées vers le mode Resource Manager.

En outre, si vous souhaitez sauvegarder la machine virtuelle migrée, accédez au panneau de gestion de la machine virtuelle pour [activer la sauvegarde](../backup/quick-backup-vm-portal.md#enable-backup-on-a-vm).

## <a name="can-i-validate-my-subscription-or-resources-to-see-if-theyre-capable-of-migration"></a>Puis-je vérifier si mon abonnement ou mes ressources peuvent faire l’objet d’une migration ? 

Oui. Dans l’option de migration prise en charge par la plateforme, la première étape de préparation de la migration consiste à s’assurer que les ressources peuvent faire l’objet d’une migration. En cas d’échec de l’opération de validation, vous recevez des messages avec toutes les raisons pour lesquelles la migration ne peut pas aboutir.

## <a name="what-happens-if-i-run-into-a-quota-error-while-preparing-the-iaas-resources-for-migration"></a>Que se passe-t-il si je rencontre une erreur de quota lors de la préparation des ressources IaaS pour la migration ? 

Nous vous recommandons d’abandonner votre migration et de créer ensuite une demande de support en vue d’augmenter les quotas pour la région dans laquelle vous effectuez la migration des machines virtuelles. Une fois votre demande de quota approuvée, vous pouvez réexécuter la procédure de migration.

## <a name="how-do-i-report-an-issue"></a>Comment signaler un problème ? 

Publiez vos problèmes et questions sur la migration dans notre [page de questions Microsoft Q&R sur les machines virtuelles](/answers/topics/azure-virtual-machines.html), avec le mot clé ClassicIaaSMigration. Nous vous recommandons de poster toutes vos questions sur ce forum. Si vous disposez d’un contrat de support, vous pouvez également créer un ticket de support.

## <a name="what-if-i-dont-like-the-names-of-the-resources-that-the-platform-chose-during-migration"></a>Comment procéder si je n’apprécie pas les noms que la plateforme a choisis pour mes ressources lors de la migration ? 

Les noms de toutes les ressources que vous avez explicitement fournis dans le modèle de déploiement sont conservés pendant la migration. Dans certains cas, de nouvelles ressources seront créées. Par exemple, une interface réseau est créée pour chaque machine virtuelle. Actuellement, nous ne prenons pas en charge la possibilité de contrôler les noms de ces ressources créées pendant la migration. Vous pouvez voter en facteur de cette fonctionnalité via le [forum des commentaires sur Azure](https://feedback.azure.com).

## <a name="can-i-migrate-expressroute-circuits-used-across-subscriptions-with-authorization-links"></a>Puis-je migrer des circuits ExpressRoute utilisés dans les abonnements avec des liens d’autorisation ? 

Les circuits ExpressRoute utilisant des liens d’autorisation entre abonnements ne peuvent pas être migrés automatiquement sans temps d’arrêt. Il existe des étapes manuelles permettant de migrer ces circuits. Consultez [Migrer des circuits ExpressRoute et les réseaux virtuels associés du modèle de déploiement classique au modèle de déploiement Resource Manager](../expressroute/expressroute-migration-classic-resource-manager.md) pour voir la procédure et obtenir plus d’informations.

## <a name="i-got-the-message-vm-is-reporting-the-overall-agent-status-as-not-ready-hence-the-vm-cannot-be-migrated-ensure-that-the-vm-agent-is-reporting-overall-agent-status-as-ready-or-vm-contains-extension-whose-status-is-not-being-reported-from-the-vm-hence-this-vm-cannot-be-migrated"></a>J’ai reçu le message *« La machine virtuelle signale que l’agent est dans l’état général Pas prêt. Par conséquent, la machine virtuelle ne peut pas être migrée. Assurez-vous que l’état général de l’agent signalé par l’agent de machine virtuelle est « Prêt »* ou *« La machine virtuelle contient une extension dont l’état n’est pas signalé par la machine virtuelle. Par conséquent, elle ne peut pas faire l’objet d’une migration. »*

Ce message est affiché lorsque la machine virtuelle n’a pas de connectivité sortante à Internet. L’agent de machine virtuelle utilise la connectivité sortante pour atteindre le compte de stockage Azure afin de mettre à jour l’état de l’agent toutes les 5 minutes.


## <a name="next-steps"></a>Étapes suivantes

Pour Linux :

* [Vue d’ensemble de la migration prise en charge par la plateforme de ressources IaaS Classic vers Azure Resource Manager](./migration-classic-resource-manager-overview.md)
* [Étude technique approfondie de la migration prise en charge par la plateforme de ressources Classic vers Azure Resource Manager](migration-classic-resource-manager-deep-dive.md)
* [Planification de la migration des ressources IaaS d’Azure Classic vers Azure Resource Manager](migration-classic-resource-manager-plan.md)
* [Faire migrer des ressources IaaS Classic vers Azure Resource Manager à l’aide d’Azure PowerShell](migration-classic-resource-manager-ps.md)
* [Faire migrer des ressources IaaS Classic vers Azure Resource Manager à l’aide de l’interface de ligne de commande Azure](migration-classic-resource-manager-cli.md)
* [Outils de la communauté pour aider à la migration de ressources IaaS de Classic vers Azure Resource Manager](migration-classic-resource-manager-community-tools.md)
* [Passer en revue les erreurs courantes de migration](migration-classic-resource-manager-errors.md)

Pour Windows :

* [Vue d’ensemble de la migration prise en charge par la plateforme de ressources IaaS Classic vers Azure Resource Manager](migration-classic-resource-manager-overview.md)
* [Étude technique approfondie de la migration prise en charge par la plateforme de ressources Classic vers Azure Resource Manager](migration-classic-resource-manager-deep-dive.md)
* [Planification de la migration des ressources IaaS d’Azure Classic vers Azure Resource Manager](migration-classic-resource-manager-plan.md)
* [Faire migrer des ressources IaaS Classic vers Azure Resource Manager à l’aide d’Azure PowerShell](migration-classic-resource-manager-ps.md)
* [Faire migrer des ressources IaaS Classic vers Azure Resource Manager à l’aide de l’interface de ligne de commande Azure](migration-classic-resource-manager-cli.md)
* [Outils de la communauté pour aider à la migration de ressources IaaS de Classic vers Azure Resource Manager](migration-classic-resource-manager-community-tools.md)
* [Passer en revue les erreurs courantes de migration](migration-classic-resource-manager-errors.md)
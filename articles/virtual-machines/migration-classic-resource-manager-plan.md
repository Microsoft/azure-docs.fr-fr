---
title: Planification de migration de ressources Classic vers Azure Resource Manager
description: Dans cet article, apprenez à planifier la migration des ressources IaaS d’Azure Classic vers Azure Resource Manager dans Linux.
services: virtual-machines-linux
author: tanmaygore
manager: vashan
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.topic: conceptual
ms.date: 02/06/2020
ms.author: tagore
ms.openlocfilehash: 271a20c5c9b6e157343246afb79f40894fe8a097
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94904621"
---
# <a name="planning-for-migration-of-iaas-resources-from-classic-to-azure-resource-manager-in-linux"></a>Planification de la migration des ressources IaaS d’Azure Classic vers Azure Resource Manager dans Linux

> [!IMPORTANT]
> Aujourd’hui, environ 90 % des machines virtuelles IaaS utilisent [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/). Depuis le 28 février 2020, les machines virtuelles classiques sont dépréciées. Elles seront entièrement mises hors service le 1er mars 2023. [En savoir plus]( https://aka.ms/classicvmretirement) sur cette dépréciation et [la façon dont elle vous concerne](classic-vm-deprecation.md#how-does-this-affect-me).

Si Azure Resource Manager offre de nombreuses fonctionnalités exceptionnelles, il est essentiel de planifier son parcours de migration pour que tout se passe sans heurts. Il est nécessaire de consacrer du temps à la planification pour être sûr de ne pas rencontrer de problèmes lors de l’exécution des activités de migration.

> [!NOTE]
> L’aide suivante a majoritairement été rédigée par l’équipe Azure Customer Advisory et les architectes de solutions cloud qui travaillent avec les clients sur la migration d’environnements de grande taille. De ce fait, ce document sera mis à jour au fil de l’émergence de nouveaux modèles de réussite : revenez régulièrement voir s’il comporte de nouvelles recommandations.

Le parcours de migration comprend quatre étapes générales :

![Phases de migration](./media/virtual-machines-windows-migration-classic-resource-manager/plan-labtest-migrate-beyond.png)

## <a name="plan"></a>Plan

### <a name="technical-considerations-and-tradeoffs"></a>Considérations et concessions techniques

Selon la dimension de vos spécifications techniques, vos zones géographiques et vos pratiques opérationnelles, il peut être intéressant de prendre en compte les aspects suivants :

1. Pourquoi votre organisation souhaite-t-elle passer à Azure Resource Manager ?  Quelles sont les raisons métier de la migration ?
2. Quelles sont les justifications techniques d’Azure Resource Manager ?  Quels services Azure supplémentaires aimeriez-vous utiliser, le cas échéant ?
3. Quelles sont les applications (ou les groupes de machines virtuelles) incluses dans la migration ?
4. Quels sont les scénarios pris en charge par l’API de migration ?  Consultez les [fonctionnalités et configurations non prises en charge](migration-classic-resource-manager-overview.md).
5. Vos équipes opérationnelles prennent-elles maintenant en charge les applications/machines virtuelles dans les modes Classic et Azure Resource Manager ?
6. En quoi Azure Resource Manager modifie-t-il les processus de déploiement, de gestion, de surveillance et de création de rapports concernant vos machines virtuelles ?  Est-il nécessaire de mettre à jour vos scripts de déploiement ?
7. Quel est le plan de communication pour avertir les parties prenantes (propriétaires d’applications, utilisateurs finaux et propriétaires d’infrastructure) ?
8. Selon la complexité de l’environnement, faut-il mettre en place une période de maintenance pendant laquelle l’application n’est pas accessible aux utilisateurs finaux et aux propriétaires d’applications ?  Si oui, pendant combien de temps ?
9. Quel est le plan de formation permettant aux parties prenantes d’être compétentes et expertes sur Azure Resource Manager ?
10. Quel est le plan de gestion des programmes ou des projets pour la migration ?
11. Quelle est la chronologie de la migration d’Azure Resource Manager et d’autres feuilles de route technologiques associées ?  Sont-elles parfaitement coordonnées ?

### <a name="patterns-of-success"></a>Modèles de réussite

Les clients les plus efficaces disposent de plans détaillés qui abordent, documentent et régissent les questions précédentes.  Veillez à diffuser largement les plans de migration auprès des commanditaires et des parties prenantes.  Informez-vous sur les options de migration à votre disposition ; il est fortement recommandé de lire cet ensemble de documents sur la migration.

* [Vue d’ensemble de la migration prise en charge par la plateforme de ressources IaaS Classic vers Azure Resource Manager](migration-classic-resource-manager-overview.md)
* [Étude technique approfondie de la migration prise en charge par la plateforme de ressources Classic vers Azure Resource Manager](migration-classic-resource-manager-deep-dive.md)
* [Planification de la migration des ressources IaaS d’Azure Classic vers Azure Resource Manager](migration-classic-resource-manager-plan.md)
* [Faire migrer des ressources IaaS Classic vers Azure Resource Manager à l’aide d’Azure PowerShell](migration-classic-resource-manager-ps.md)
* [Faire migrer des ressources IaaS Classic vers Azure Resource Manager à l’aide de l’interface de ligne de commande Azure](migration-classic-resource-manager-cli.md)
* [Outils de la communauté pour aider à la migration de ressources IaaS de Classic vers Azure Resource Manager](migration-classic-resource-manager-community-tools.md)
* [Passer en revue les erreurs courantes de migration](migration-classic-resource-manager-errors.md)
* [Passez en revue les questions fréquemment posées sur la migration des ressources IaaS de Classic vers Azure Resource Manager](migration-classic-resource-manager-faq.md)

### <a name="pitfalls-to-avoid"></a>Pièges à éviter

- Absence de planification.  Les étapes technologiques de cette migration sont éprouvées et le résultat est prévisible.
- Présupposition selon laquelle l’API de migration prise en charge par la plateforme gérera tous les scénarios. Lisez les [fonctionnalités et configurations non prises en charge](migration-classic-resource-manager-overview.md) pour connaître les scénarios pris en charge.
- Absence de planification d’une interruption potentielle de l’application pour les utilisateurs finaux.  Prévoyez suffisamment de tampon pour avertir convenablement les utilisateurs finaux que l’application sera potentiellement indisponible pendant un certain temps.


## <a name="lab-test"></a>Test en labo

**Répliquer un environnement et effectuer un test de migration**
  > [!NOTE]
  > La réplication à l’exact de l’environnement existant est réalisée à l’aide d’un outil développé par la communauté, qui n’est pas pris en charge officiellement par le Support Microsoft. Il s’agit par conséquent d’une étape **facultative** ; cependant, c’est le meilleur moyen d’identifier des problèmes sans toucher aux environnements de production. S’il ne vous est pas possible d’utiliser un outil développé par la communauté, lisez le passage sur la recommandation d’essai à blanc Valider/Préparer/Abandonner ci-dessous.
  >

  La meilleure solution pour garantir une migration sans heurts est de mener un test en labo de votre scénario en particulier (calcul, mise en réseau et stockage), afin de garantir :

- Un laboratoire totalement distinct ou un environnement hors production existant pour les tests. Nous recommandons un laboratoire totalement distinct qui peut faire l’objet de plusieurs migrations et de modifications destructrices.  Les scripts pour collecter/sérialiser les métadonnées à partir des abonnements réels sont listés ci-dessous.
- Il est judicieux de créer le laboratoire dans un abonnement distinct. En effet, le labo sera détruit à plusieurs reprises, et le fait de disposer d’un abonnement distinct et isolé réduit le risque de supprimer accidentellement des données réelles.

  Pour cela, vous pouvez utiliser l’outil AsmMetadataParser. [En savoir plus sur cet outil](https://github.com/Azure/classic-iaas-resourcemanager-migration/tree/master/AsmToArmMigrationApiToolset)

### <a name="patterns-of-success"></a>Modèles de réussite

Voici quelques-uns des problèmes rencontrés dans la majorité des migrations importantes. Cette liste n’est pas exhaustive. Reportez-vous aux [fonctionnalités et configurations non prises en charge](migration-classic-resource-manager-overview.md) pour plus de détails. Il n’est pas certain que vous rencontriez ces problèmes techniques ; si c’est le cas, vous garantirez une meilleure expérience en résolvant ces points avant de tenter la migration.

- **Effectuer un essai à blanc Valider/Préparer/Abandonner** : il s’agit sans doute de l’étape la plus importante pour garantir la réussite de la migration de Classic vers Azure Resource Manager. L’API de migration comporte trois étapes principales : valider, préparer et soumettre. L’étape Valider lit l’état de votre environnement Classic et retourne un résultat comportant tous les problèmes. Toutefois, dans la mesure où il peut exister des problèmes dans la pile Azure Resource Manager, elle n’intercepte pas tout. L’étape suivante du processus de migration, Préparer, aide à mettre en évidence ces problèmes. Elle déplace les métadonnées de Classic vers Azure Resource Manager, mais ne valide pas le déplacement et ne supprime ni ne modifie rien du côté de Classic. L’essai à blanc implique de préparer la migration, puis d’abandonner (**ne pas soumettre**) la préparation de la migration. L’essai à blanc Valider/Préparer/Abandonner a pour objectif de montrer toutes les métadonnées de la pile Azure Resource Manager, de les examiner (*par programme ou dans le portail*) et de vérifier que tous les éléments migrent correctement et viennent à bout des problèmes techniques.  Il donne également une idée de la durée de la migration de façon à permettre de planifier un temps d’arrêt adapté.  Une opération Valider/Préparer/Abandonner ne provoque pas de temps d’arrêt pour les utilisateurs ; par conséquent, elle ne perturbe pas l’utilisation des applications.
  - Les éléments ci-dessous devront être résolus avant l’essai à blanc, mais un test d’essai à blanc éliminera également sans risque ces étapes de préparation si elles ne sont pas suivies. Nous avons conclu que l’essai à blanc représente un moyen sûr et incomparable d’assurer la préparation à la migration lors de la migration d’entreprise.
  - Au cours de la préparation, le plan de contrôle (opérations de gestion Azure) sera verrouillé pour l’ensemble du réseau virtuel ; ainsi, aucune modification ne pourra être apportée aux métadonnées de la machine virtuelle au cours de l’opération Valider/Préparer/Abandonner.  Par ailleurs, aucune fonction de l’application (bureau à distance, utilisation de la machine virtuelle, etc.) ne sera affectée.  Les utilisateurs des machines virtuelles ne sauront pas que l’essai à blanc est en cours d’exécution.

- **Circuits ExpressRoute et VPN**. Actuellement, il n’est pas possible de migrer les passerelles ExpressRoute avec liens d’autorisation sans temps mort. Vous trouverez une solution de contournement à la page [Migrer des circuits ExpressRoute et les réseaux virtuels associés du modèle de déploiement Classic au modèle de déploiement Resource Manager](../expressroute/expressroute-migration-classic-resource-manager.md).

- **Extensions de machines virtuelles** : les extensions de machines virtuelles sont potentiellement l’un des principaux obstacles à la migration de machines virtuelles en cours d’exécution. La correction des extensions de machines virtuelles peut prendre plus d’un ou deux jours : adaptez votre planification en conséquence.  Un agent Azure fonctionnel est nécessaire pour rapporter l’état des extensions de machines virtuelles en cours d’exécution. Si l’état renvoyé pour une machine virtuelle en cours d’exécution est mauvais, la migration sera interrompue. L’agent lui-même n’a pas besoin d’être en état de fonctionnement pour permettre la migration, mais, si des extensions existent sur la machine virtuelle, un agent fonctionnel ET une connectivité Internet sortante (avec DNS) seront nécessaires pour que la migration se poursuive.
  - Si la connectivité à un serveur DNS est perdue pendant la migration, toutes les extensions de machines virtuelles, à l’exception de BGInfo v1.\* doivent être supprimées de chacune des machines virtuelles avant la préparation de la migration, puis rajoutées aux machines virtuelles après la migration d’Azure Resource Manager.  **Cela concerne uniquement les machines virtuelles en cours d’exécution.**  Si les machines virtuelles sont arrêtées et libérées, il n’est pas nécessaire de supprimer les extensions de machines virtuelles. **Remarque :** De nombreuses extensions, par exemple Diagnostics Azure et la surveillance Security Center, se réinstallent après la migration. Les supprimer n’est donc pas un problème.
  - Par ailleurs, assurez-vous que les groupes de sécurité réseau ne restreignent pas l’accès à Internet sortant. Cela peut se produire avec certaines configurations de groupes de sécurité réseau. Un accès à Internet sortant (et un DNS) est nécessaire pour migrer les extensions de machines virtuelles vers Azure Resource Manager.
  - Il existe deux versions de l’extension BGInfo : v1 et v2.  Si la machine virtuelle a été créée avec le portail Azure ou PowerShell, elle comporte probablement l’extension v1. Il n’est pas nécessaire de supprimer cette extension : elle sera ignorée (non migrée) par l’API de migration. En revanche, si la machine virtuelle Classic a été créée avec le nouveau Portail Azure, elle comportera probablement la version v2 JSON de BGInfo, qui peut être migrée vers Azure Resource Manager à condition que l’agent fonctionne et dispose d’un accès à Internet sortant (et d’un DNS).
  - **Option de correction 1**. Si vous savez que vos machines virtuelles ne disposeront pas d’un accès à Internet sortant, d’un service DNS fonctionnel et d’agents Azure fonctionnels, désinstallez toutes les extensions de machines virtuelles dans le cadre de la migration avant l’opération Préparer, puis réinstallez-les après la migration.
  - **Option de correction 2**. Si les extensions de machines virtuelles représentent un trop gros obstacle, il est également possible d’arrêter/libérer toutes les machines virtuelles avant la migration. Migrez les machines virtuelles libérées, puis redémarrez-les du côté d’Azure Resource Manager. L’avantage est que les extensions de machines virtuelles seront migrées. L’inconvénient est que toutes les adresses IP virtuelles publiques seront perdues (ce qui peut être inenvisageable) et, bien évidemment, que les machines virtuelles seront arrêtées, ce qui aura un impact bien plus important sur les applications en cours d’exécution.

    > [!NOTE]
    > S’il existe une stratégie de sécurité Azure Security Center configurée sur les machines virtuelles en cours d’exécution et de migration, elle doit être arrêtée avant de supprimer les extensions ; sinon, l’extension de contrôle de la sécurité sera réinstallée automatiquement sur la machine virtuelle après sa suppression.

- **Groupes à haute disponibilité** : pour qu’un réseau virtuel migre vers Azure Resource Manager, toutes les machines virtuelles contenues dans le déploiement Classic (c’est-à-dire le service cloud) doivent se trouver dans un même groupe à haute disponibilité ou bien n’être dans aucun groupe à haute disponibilité. Il n’est pas possible d’avoir plusieurs groupes à haute disponibilité dans le service cloud avec Azure Resource Manager ; cela a pour effet de stopper la migration.  En outre, il ne peut pas y avoir des machines virtuelles dans un groupe à haute disponibilité et d’autres ailleurs que dans un groupe à haute disponibilité. Pour résoudre ce problème, vous devrez corriger ou remanier votre service cloud.  Adaptez votre planification en conséquence, car cette opération peut prendre du temps.

- **Déploiements de rôles Web/de travail** : il n’est pas possible de migrer les services cloud contenant des rôles Web et de travail vers Azure Resource Manager. Les rôles Web/de travail doivent être retirés du réseau virtuel pour que la migration puisse démarrer.  Une solution classique consiste tout simplement à déplacer les instances de rôles Web/de travail vers un réseau virtuel Classic distinct, également lié à un circuit ExpressRoute, ou à migrer le code vers des App Services PaaS plus récents (cet aspect n’est pas abordé dans ce document). Dans le premier cas de redéploiement, créez un réseau virtuel Classic, déplacez/redéployez les rôles Web/de travail sur ce réseau virtuel, puis supprimez les déploiements du réseau virtuel déplacé. Aucune modification de code requise : La nouvelle fonctionnalité de [peering de réseau virtuel](../virtual-network/virtual-network-peering-overview.md) peut être utilisée pour effectuer le peering du réseau virtuel Classic contenant les rôles Web/de travail et d’autres réseaux virtuels de la même région Azure, par exemple le réseau virtuel en cours de migration (**après la fin de la migration du réseau virtuel, car il n’est pas possible de migrer des réseaux virtuels appairés**), ce qui fournit les mêmes fonctionnalités sans perte de performances et sans pénalités du point de vue de la latence et de la bande passante. Grâce à l’ajout du [peering de réseau virtuel](../virtual-network/virtual-network-peering-overview.md), il est maintenant facile de corriger les déploiements de rôles Web/de travail, de sorte qu’ils ne bloquent pas la migration vers Azure Resource Manager.

- **Quotas d’Azure Resource Manager** : les régions Azure disposent de quotas/limites distincts pour Classic et Azure Resource Manager. Bien qu’il n’y ait pas de nouveau matériel utilisé dans un scénario de migration *(on permute les machines virtuelles existantes de Classic vers Azure Resource Manager)* , les quotas d’Azure Resource Manager devront être mis en place avec une capacité suffisante pour que la migration puisse démarrer. Vous trouverez ci-dessous les principales limites dont nous avons constaté qu’elles posaient problème.  Ouvrez un ticket de support concernant les quotas pour relever les limites.

    > [!NOTE]
    > Ces limites doivent être relevées dans la même région que votre environnement actuel à migrer.
    >

  - Interfaces réseau
  - Équilibreurs de charge
  - Adresses IP publiques
  - Adresses IP publiques statiques
  - Cœurs
  - Network Security Group
  - Tables de routage

    Vous pouvez vérifier vos quotas Azure Resource Manager actuels en utilisant les commandes suivantes avec la dernière version d’Azure CLI.

    **Compute** *(cœurs, groupes à haute disponibilité)*

    ```azurecli
    az vm list-usage -l <azure-region> -o jsonc
    ```

    **Réseau** *(réseaux virtuels, adresses IP publiques statiques, adresses IP publiques, groupes de sécurité réseau, interfaces réseau, équilibreurs de charge, tables de route)*

    ```azurecli
    az network list-usages -l <azure-region> -o jsonc
    ```

    **Stockage** *(compte de stockage)*

    ```azurecli
    az storage account show-usage
    ```

- **Limitations de l’API Azure Resource Manager** : si vous avez un environnement de grande taille (par exemple, Plus de 400 machines virtuelles dans un réseau virtuel), vous risquez d’atteindre les limitations de l’API par défaut pour les écritures (actuellement, **1200 écritures/heure**) dans Azure Resource Manager. Avant de commencer la migration, il vous faut déclencher un ticket de support afin d’augmenter cette limite pour votre abonnement.

- **État Provisioning Timed Out pour une machine virtuelle** : si une machine virtuelle a l’état **provisioning timed out**, la situation doit être résolue avant la migration. La seule façon de procéder consiste à déprovisionner/réapprovisionner la machine virtuelle (la supprimer, conserver le disque et recréer la machine virtuelle) avec temps d’arrêt.

- **État RoleStateUnknown pour une machine virtuelle** : si la migration s’arrête à cause d’un message d’erreur **role state unknown**, inspectez la machine virtuelle à l’aide du portail et vérifiez qu’elle est en cours d’exécution. Cette erreur disparaîtra normalement toute seule (aucune correction n’est requise) après quelques minutes ; souvent de type temporaire, elle est généralement constatée au cours des opérations **start**, **stop**, **restart** de machine virtuelle. **Pratique recommandée :** effectuez une nouvelle tentative de migration après quelques minutes.

- **La structure du cluster n’existe pas** : il arrive qu’il ne soit pas possible de migrer une machine virtuelle pour d’obscures raisons. Parmi les cas connus figure celui où la machine virtuelle a été créée récemment (aux alentours de la semaine précédente) et s’est retrouvée sur un cluster Azure qui n’était pas encore équipé pour les charges de travail Azure Resource Manager.  Vous obtiendrez une erreur **fabric cluster does not exist** et la machine virtuelle ne pourra pas être migrée. Il suffit en général de patienter quelques jours pour résoudre ce problème en particulier, car le cluster sera bientôt compatible avec Azure Resource Manager. Toutefois, il existe une solution de contournement immédiat, qui consiste à `stop-deallocate` la machine virtuelle, puis à poursuivre la migration et à lancer la sauvegarde de la machine virtuelle dans Azure Resource Manager après la migration.

### <a name="pitfalls-to-avoid"></a>Pièges à éviter

- Ne prenez pas de raccourcis et ne négligez pas l’essai à blanc Valider/Préparer/Abandonner des migrations.
- La plupart, voire la totalité, des problèmes potentiels apparaîtront au cours des étapes Valider/Préparer/Abandonner.

## <a name="migration"></a>Migration

### <a name="technical-considerations-and-tradeoffs"></a>Considérations et concessions techniques

Maintenant que vous êtes venu à bout des problèmes connus de votre environnement, vous êtes prêt.

Voici les aspects à prendre en compte pour les migrations réelles :

1. Planifiez et programmez le réseau virtuel (la plus petite unité de migration) avec une priorité supérieure.  Commencez par les réseaux virtuels simples, puis continuez avec les réseaux virtuels plus complexes.
2. La plupart des clients ont des environnements de production et hors production.  Planifiez la production en dernier.
3. **(FACULTATIF)** Planifiez un temps d’arrêt pour maintenance avec suffisamment de mémoire tampon en cas de problème inattendu.
4. Communiquez et coordonnez-vous avec vos équipes du support technique au cas où des problèmes surviendraient.

### <a name="patterns-of-success"></a>Modèles de réussite

Il est conseillé d’étudier et d’appliquer l’aide technique de la section Test en labo ci-dessus avant toute migration réelle.  Avec les tests appropriés, la migration se déroule sans aucun problème.  Pour les environnements de production, il peut être utile de disposer d’un support supplémentaire, par exemple un partenaire de confiance Microsoft ou des services Microsoft Premier.

### <a name="pitfalls-to-avoid"></a>Pièges à éviter

Des tests incomplets peuvent provoquer des problèmes et des retards de migration.  

## <a name="beyond-migration"></a>Au-delà de la migration

### <a name="technical-considerations-and-tradeoffs"></a>Considérations et concessions techniques

Maintenant que vous êtes en mode Azure Resource Manager, tirez le meilleur parti de la plateforme.  Lisez la [vue d’ensemble d’Azure Resource Manager](../azure-resource-manager/management/overview.md) pour découvrir les avantages supplémentaires.

Points importants à prendre en compte :

- Groupez la migration avec d’autres activités.  La plupart des clients choisissent une fenêtre de maintenance des applications.  Dans ce cas, vous pouvez utiliser ce temps d’arrêt pour activer d’autres fonctionnalités d’Azure Resource Manager, comme le chiffrement et la migration vers la fonctionnalité Disques managés.
- Repassez en revue les raisons techniques et commerciales d’Azure Resource Manager ; activez les services supplémentaires disponibles uniquement sur Azure Resource Manager qui s’appliquent à votre environnement.
- Modernisez votre environnement avec les services PaaS.

### <a name="patterns-of-success"></a>Modèles de réussite

Ciblez les services que vous voulez activer dans Azure Resource Manager.  De nombreux clients trouvent les services ci-dessous incontournables pour leur environnement Azure :

- [Contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md).
- [Modèles Azure Resource Manager, pour un déploiement plus facile et mieux contrôlé](../azure-resource-manager/templates/overview.md).
- [Tags](../azure-resource-manager/management/tag-resources.md) (balises).
- [Contrôle d’activité](../azure-resource-manager/management/view-activity-logs.md)
- [Stratégies Azure](../governance/policy/overview.md)

### <a name="pitfalls-to-avoid"></a>Pièges à éviter

Gardez à l’esprit les raisons pour lesquelles vous avez démarré ce parcours de migration de Classic vers Azure Resource Manager.  Quelles étaient les raisons commerciales initiales ? Avez-vous atteint l’objectif métier ?


## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble de la migration prise en charge par la plateforme de ressources IaaS Classic vers Azure Resource Manager](migration-classic-resource-manager-overview.md)
* [Étude technique approfondie de la migration prise en charge par la plateforme de ressources Classic vers Azure Resource Manager](migration-classic-resource-manager-deep-dive.md)
* [Planification de la migration des ressources IaaS d’Azure Classic vers Azure Resource Manager](migration-classic-resource-manager-plan.md)
* [Faire migrer des ressources IaaS Classic vers Azure Resource Manager à l’aide d’Azure PowerShell](migration-classic-resource-manager-ps.md)
* [Outils de la communauté pour aider à la migration de ressources IaaS de Classic vers Azure Resource Manager](migration-classic-resource-manager-community-tools.md)
* [Passer en revue les erreurs courantes de migration](migration-classic-resource-manager-errors.md)
* [Passez en revue les questions fréquemment posées sur la migration des ressources IaaS de Classic vers Azure Resource Manager](migration-classic-resource-manager-faq.md)

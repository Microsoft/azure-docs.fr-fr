---
title: Étude technique approfondie de la migration de Classic à Azure Resource Manager
description: Étude technique approfondie de la migration prise en charge par la plateforme de ressources du modèle de déploiement Classic vers Azure Resource Manager.
author: tanmaygore
manager: vashan
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: conceptual
ms.date: 02/06/2020
ms.author: tagore
ms.openlocfilehash: d73ad3235e5ff2c9dbf0cca546308469ef6b5ac0
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94887037"
---
# <a name="technical-deep-dive-on-platform-supported-migration-from-classic-to-azure-resource-manager"></a>Étude technique approfondie de la migration prise en charge par la plateforme de ressources Classic vers Azure Resource Manager

> [!IMPORTANT]
> Aujourd’hui, environ 90 % des machines virtuelles IaaS utilisent [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/). Depuis le 28 février 2020, les machines virtuelles classiques sont dépréciées. Elles seront entièrement mises hors service le 1er mars 2023. [Apprenez-en davantage]( https://aka.ms/classicvmretirement) sur cette dépréciation et [son impact sur vous](./classic-vm-deprecation.md#how-does-this-affect-me).

Examinons en détail la migration à partir du modèle de déploiement Azure classique vers le modèle de déploiement Azure Resource Manager. Nous examinons les ressources au niveau des fonctionnalités et des ressources pour vous aider à comprendre comment la plateforme Azure migre les ressources entre les deux modèles de déploiement. Pour plus d’informations, lisez l’article annonçant le service : [Migration prise en charge par la plateforme de ressources IaaS Classic vers Azure Resource Manager](migration-classic-resource-manager-overview.md).


## <a name="migrate-iaas-resources-from-the-classic-deployment-model-to-azure-resource-manager"></a>Migrer des ressources IaaS depuis le modèle de déploiement classique vers Azure Resource Manager
Tout d’abord, il est important de comprendre la différence entre les opérations de plan de données et les opérations de plan de gestion sur les ressources IaaS (infrastructure as a service).

* *Plan de gestion/contrôle* décrit l’ensemble des appels destinés au plan de gestion/contrôle ou à l’API en vue de modifier les ressources. Par exemple, les opérations telles que la création d’une machine virtuelle, le redémarrage d’une machine virtuelle et la mise à jour d’un réseau virtuel avec un nouveau sous-réseau permettent de gérer les ressources en cours d’exécution. Elles n’affectent pas directement la connexion aux machines virtuelles.
* *Plan de données* (application) décrit le runtime de l’application proprement dite et implique une interaction avec les instances qui ne passent pas par l’API Azure. Par exemple, l’accès à votre site web ou l’extraction de données depuis une instance SQL Server ou depuis un serveur MongoDB en cours d’exécution sont considérés comme des interactions d’application ou de plan de données. La copie d’un objet blob depuis un compte de stockage et l’accès à une adresse IP publique pour utiliser le protocole RDP (Remote Desktop Protocol) ou Secure Shell (SSH) sur l’ordinateur virtuel constituent d’autres exemples. Ces opérations garantissent le fonctionnement continu de l’application dans l’ensemble des services de calcul, de mise en réseau et de stockage.

Le plan de données est le même entre le modèle de déploiement classique et les piles Resource Manager. La différence tient au fait que durant le processus de migration, Microsoft convertit la représentation des ressources depuis un modèle de déploiement classique vers celui de la pile Resource Manager. Par conséquent, vous devez utiliser de nouveaux outils, API et Kits de développement logiciel (SDK) pour gérer vos ressources dans la pile Resource Manager.

![Diagramme illustrant la différence entre un plan de contrôle/gestion et le plan de données](./media/virtual-machines-windows-migration-classic-resource-manager/data-control-plane.png)


> [!NOTE]
> Dans certains scénarios de migration, la plateforme Azure arrête, désalloue, puis redémarre vos machines virtuelles. Cela provoque un bref temps d’arrêt du plan de données.
>

## <a name="the-migration-experience"></a>Expérience de migration
Avant de commencer la migration :

* Assurez-vous que les ressources que vous souhaitez migrer n’utilisent pas de fonctionnalités ou de configurations non prises en charge. Généralement, la plateforme détecte ces problèmes et génère une erreur.
* Si certaines de vos machines virtuelles n’appartiennent à aucun réseau virtuel, elles seront arrêtées et désallouées dans le cadre de l’opération de préparation. Si vous souhaitez conserver l’adresse IP publique, envisagez de réserver l’adresse IP avant de déclencher l’opération de préparation. Si les machines virtuelles se trouvent dans un réseau virtuel, elles ne sont ni arrêtées, ni désallouées.
* Planifiez votre migration en dehors des heures d’ouverture afin de tenir compte des erreurs inattendues susceptibles de survenir au cours du processus.
* Téléchargez la configuration actuelle de vos machines virtuelles à l’aide de PowerShell, de commandes de l’interface de ligne de commande ou d’API REST, afin d’en faciliter la vérification après l’étape de préparation.
* Mettez à jour vos scripts d’automatisation/d’opérationnalisation pour gérer le modèle de déploiement Resource Manager avant de commencer la migration. Vous pouvez également effectuer des opérations GET lorsque les ressources se trouvent à l’état Préparé.
* Évaluez les stratégies de contrôle d'accès en fonction du rôle Azure (Azure RBAC) qui sont configurées sur les ressources IaaS dans le modèle de déploiement classique, et planifiez ce qui se passera une fois la migration terminée.

Le workflow de migration se présente comme suit :

![Diagramme illustrant le workflow de migration](./media/migration-classic-resource-manager/migration-workflow.png)

> [!NOTE]
> Toutes les opérations décrites dans les sections suivantes sont idempotentes. Si vous rencontrez un problème autre qu’une fonctionnalité non prise en charge ou qu’une erreur de configuration, ré-exécutez la procédure de préparation, d’abandon ou de validation. Azure tente à nouveau l’action.
>
>

### <a name="validate"></a>Valider
L’opération de validation constitue la première étape du processus de migration. L’objectif de cette étape est d’analyser l’état des ressources que vous souhaitez migrer dans le modèle de déploiement classique. L’opération évalue si les ressources sont en mesure de migrer (succès ou échec).

Vous sélectionnez le réseau virtuel ou un service cloud (s’il ne s’agit pas d’un réseau virtuel) que vous souhaitez valider pour la migration. Si la ressource n’est pas en mesure de migrer, Azure en répertorie les raisons.

#### <a name="checks-not-done-in-the-validate-operation"></a>Vérifications non effectuées lors de l’opération de validation

L’opération de validation ne fait qu’analyser l’état des ressources dans le modèle de déploiement classique. Cette opération permet de vérifier tous les échecs et les scénarios non pris en charge en raison de diverses configurations dans le modèle de déploiement classique. Il est impossible de vérifier tous les problèmes que pourrait causer la pile d’Azure Resource Manager aux ressources pendant la migration. Ces problèmes ne sont vérifiés que lorsque les ressources subissent une transformation à l’étape suivante de la migration (l’opération de préparation). Le tableau suivant répertorie tous les problèmes non archivés dans l’opération de validation :


|Vérifications réseau non effectuées dans l’opération de validation|
|-|
|Un réseau virtuel ayant des passerelles ER et VPN.|
|Une connexion de passerelle de réseau virtuel dans un état déconnecté.|
|Tous les circuits ER sont pré-migrés vers la pile d’Azure Resource Manager.|
|Vérifications de quotas Azure Resource Manager pour les ressources réseau. Exemple : adresse IP publique statique, adresses IP publiques dynamiques, équilibreur de charge, groupes de sécurité réseau, tables d’itinéraires et interfaces réseau. |
| Toutes les règles d’équilibrage de charge sont valides sur le déploiement et le réseau virtuel. |
| Adresses IP privées en conflit entre des machines virtuelles arrêtées-désallouées dans le même réseau virtuel. |

### <a name="prepare"></a>Préparation
L’opération de préparation constitue la deuxième étape du processus de migration. L’objectif de cette étape est de simuler la transformation des ressources IaaS depuis le modèle de déploiement classique vers les ressources Resource Manager. En outre, l’opération de préparation présente cette simulation côte à côte pour vous permettre de visualiser.

> [!NOTE] 
> Vos ressources dans le modèle de déploiement classique ne sont pas modifiées au cours de cette étape. Il s’agit d’une étape sécurisée à exécuter si vous testez la migration. 

Vous sélectionnez le réseau virtuel ou le service cloud (s’il ne s’agit pas d’un réseau virtuel) que vous souhaitez préparer pour la migration.

* Si la ressource n’est pas en mesure de migrer, Azure arrête le processus de migration et indique la raison de l’échec de l’opération de préparation.
* Si la ressource est en mesure de migrer, Azure verrouille les opérations de plan de gestion pour les ressources en cours de migration. Par exemple, vous ne pourrez pas ajouter de disque de données à une machine virtuelle en cours de migration.

Azure démarre ensuite la migration des métadonnées depuis le modèle de déploiement classique vers Resource Manager pour les ressources faisant l’objet de la migration.

Une fois l’opération de préparation terminée, vous avez la possibilité de visualiser les ressources dans le modèle de déploiement classique et dans Resource Manager. Pour chaque service cloud du modèle de déploiement classique, nous créons un nom de groupe de ressources au format `cloud-service-name>-Migrated`.

> [!NOTE]
> Il est impossible de sélectionner le nom d’un groupe de ressources créé pour les ressources migrées (autrement dit, à l’état « -Migré »). Toutefois, une fois la migration terminée, vous pouvez utiliser la fonctionnalité de déplacement d’Azure Resource Manager pour déplacer des ressources vers le groupe de ressources souhaité. Pour plus d’informations, consultez la page [Déplacement de ressources vers un nouveau groupe de ressources ou un abonnement](../azure-resource-manager/management/move-resource-group-and-subscription.md).

Les deux captures d’écran suivantes affichent le résultat après la réussite d’une opération de préparation. La première affiche un groupe de ressources contenant le service cloud d’origine. La deuxième affiche le nouveau groupe de ressources « -Migré » contenant les ressources Azure Resource Manager équivalentes.

![Capture d’écran affichant le service cloud d’origine](./media/migration-classic-resource-manager/portal-classic.png)

![Capture d’écran affichant les ressources Azure Resource Manager lors de l’opération de préparation](./media/migration-classic-resource-manager/portal-arm.png)

Voici un aperçu des coulisses de vos ressources à l’issue de la phase de préparation. Notez que la ressource dans le plan de données est la même. Elle est représentée dans le plan de gestion (modèle de déploiement classique) et dans le plan de contrôle (Resource Manager).

![Diagramme de la phase de préparation](./media/migration-classic-resource-manager/behind-the-scenes-prepare.png)

> [!NOTE]
> Les machines virtuelles ne faisant pas partie d’un réseau virtuel dans le modèle de déploiement classique sont arrêtées et désallouées dans cette phase de migration.
>

### <a name="check-manual-or-scripted"></a>Vérification (manuelle ou sur la base d’un script)
Lors de l’étape de vérification, vous avez la possibilité d’utiliser la configuration que vous avez téléchargée précédemment pour valider le bon déroulement de la migration. Une autre possibilité consiste à se connecter au portail et à procéder à une vérification ponctuelle des propriétés et des ressources pour valider le déroulement adéquat de la migration des métadonnées.

Si vous effectuez la migration d’un réseau virtuel, la majeure partie de la configuration des machines virtuelles n’est pas redémarrée. En ce qui concerne les applications installées sur ces machines virtuelles, vous pouvez valider le bon fonctionnement de ces applications.

Vous pouvez tester vos scripts opérationnels et de surveillance pour voir si les machines virtuelles fonctionnent comme prévu, et si vos scripts mis à jour fonctionnent correctement. Seules les opérations GET sont prises en charge lorsque les ressources sont en état Préparé.

Il n’existe pas de fenêtre de temps défini imposant un délai dans lequel vous devez valider la migration. Vous pouvez conserver les ressources en l’état pour la durée de votre choix. Toutefois, le plan de gestion est verrouillé pour ces ressources jusqu’à ce que vous choisissiez d’abandonner ou de valider la migration.

Si vous constatez certains problèmes, vous pouvez toujours abandonner la migration et revenir au modèle de déploiement Classic. Une fois que vous revenez en arrière, Azure ouvre les opérations de plan de gestion sur les ressources, de manière à ce que vous puissiez reprendre les opérations normales sur ces machines virtuelles dans le modèle de déploiement classique.

### <a name="abort"></a>Abandon
Il s’agit d’une étape facultative si vous souhaitez annuler vos modifications, rétablir le modèle de déploiement classique et arrêter la migration. Cette opération supprime les métadonnées de Resource Manager (créées à l’étape de préparation) pour vos ressources. 

![Diagramme de l’étape d’abandon](media/migration-classic-resource-manager/behind-the-scenes-abort.png)


> [!NOTE]
> Vous ne pouvez pas effectuer cette opération après le déclenchement de l’opération de validation.     
>

### <a name="commit"></a>Commit
Après avoir terminé la validation, vous pouvez valider la migration. Les ressources n’apparaissent plus dans le modèle de déploiement classique et ne sont disponibles que dans le modèle de déploiement Resource Manager. Les ressources migrées peuvent être gérées uniquement dans le nouveau portail.

> [!NOTE]
> Il s’agit d’une opération idempotente. En cas d’échec, recommencez l’opération. Si l’échec persiste, créez un ticket de support ou créez un forum Internet sur [Microsoft Q&A](/answers/index.html).
>
>

![Diagramme de l’étape de validation](media/migration-classic-resource-manager/behind-the-scenes-commit.png)

## <a name="migration-flowchart"></a>Organigramme de la migration

Cet organigramme montre comment procéder à la migration :

![Screenshot that shows the migration steps](media/migration-classic-resource-manager/migration-flow.png)

## <a name="translation-of-the-classic-deployment-model-to-resource-manager-resources"></a>Traduction du modèle de déploiement classique vers les ressources Resource Manager
Les représentations des ressources du modèle de déploiement classique et de Resource Manager sont décrites dans la table suivante. D’autres fonctionnalités et ressources ne sont pas prises en charge actuellement.

| Représentation Classic | Représentation Resource Manager | Notes |
| --- | --- | --- |
| Nom du service cloud |Nom DNS |Pendant la migration, un groupe de ressources est créé pour chaque service cloud correspondant au modèle de dénomination `<cloudservicename>-migrated`. Ce groupe de ressources contient toutes vos ressources. Le nom du service cloud devient un nom DNS associé à l’adresse IP publique. |
| Machine virtuelle |Machine virtuelle |Les propriétés spécifiques à la machine virtuelle sont inchangées après la migration. Certaines informations osProfile, comme le nom de l’ordinateur, ne sont pas stockées dans le modèle de déploiement classique et restent vides après la migration. |
| Ressources de disque attachées à une machine virtuelle |Disques implicites attachés à une machine virtuelle |Les disques ne sont pas modélisés comme ressources de niveau supérieur dans le modèle de déploiement Resource Manager. Ils font l’objet d’une migration sous forme de disques implicites sous la machine virtuelle. Seuls des disques attachés à une machine virtuelle sont actuellement pris en charge. Les machines virtuelles Resource Manager peuvent désormais utiliser des comptes de stockage dans le modèle de déploiement classique, ce qui permet une migration facile des disques sans aucune mise à jour. |
| Extensions de machine virtuelle |Extensions de machine virtuelle |Toutes les extensions de ressource font l’objet d’une migration depuis le déploiement Classic, à l’exception des extensions XML. |
| Certificats de machine virtuelle |Certificats dans Azure Key Vault |Si un service cloud contient des certificats de service, la migration crée un nouveau coffre de clés Azure par service cloud et déplace les certificats dans le coffre de clés. Les machines virtuelles sont mises à jour pour référencer les certificats du coffre de clés. <br><br> Ne supprimez pas le coffre de clés. Cela peut provoquer un état d’échec de la machine virtuelle. |
| Configuration WinRM |Configuration WinRM sous osProfile |La configuration de Windows Remote Management est déplacée sans être modifiée dans le cadre de la migration. |
| Propriété de groupe à haute disponibilité |Ressource de groupe à haute disponibilité | La spécification des groupes à haute disponibilité est une propriété de la machine virtuelle dans le modèle de déploiement classique. Les groupes à haute disponibilité deviennent une ressource de niveau supérieur dans le cadre de la migration. Les configurations suivantes ne sont pas prises en charge : plusieurs groupes à haute disponibilité par service cloud, ou un ou plusieurs groupes à haute disponibilité en même temps que des machines virtuelles ne figurant dans aucun groupe à haute disponibilité d’un service cloud. |
| Configuration réseau sur une machine virtuelle |Interface réseau principale |La configuration réseau sur une machine virtuelle est représentée sous forme de ressource d’interface réseau principale après la migration. Dans le cas des machines virtuelles qui ne figurent pas dans un réseau virtuel, l’adresse IP interne change lors de la migration. |
| Plusieurs interfaces réseau sur une machine virtuelle |Interfaces réseau |Si une machine virtuelle possède plusieurs interfaces réseau associées, chaque interface réseau devient une ressource de niveau supérieur dans le cadre de la migration, ainsi que toutes les propriétés. |
| Jeux de points de terminaison soumis à l’équilibrage de charge |Équilibrage de charge |Dans le modèle de déploiement Classic, la plateforme a affecté un équilibreur de charge implicite à chaque service cloud. Pendant la migration, une ressource d’équilibreur de charge est créée, et le jeu de points de terminaison d’équilibrage de charge constitue les règles de l’équilibreur de charge. |
| Règles NAT entrantes |Règles NAT entrantes |Les points de terminaison d’entrée définis sur la machine virtuelle sont convertis en règles de traduction NAT (Network Access Translation) du trafic entrant sous l’équilibreur de charge lors de la migration. |
| Adresse IP virtuelle |Adresse IP publique avec nom DNS |L’adresse IP virtuelle devient une adresse IP publique et est associée à l’équilibreur de charge. Une adresse IP virtuelle ne peut faire l’objet d’une migration que si un point de terminaison d’entrée lui a été attribué. |
| Réseau virtuel |Réseau virtuel |Le réseau virtuel fait l’objet d’une migration vers le modèle de déploiement Resource Manager avec toutes ses propriétés. Un nouveau groupe de ressources est créé avec le nom `-migrated`. |
| IP réservées |Adresse IP publique avec méthode d’allocation statique |La migration des IP réservées associées à l’équilibreur de charge est effectuée en même temps que la migration du service cloud ou de la machine virtuelle. Vous pouvez migrer des adresses IP réservées non associées à l’aide de la commande [Move-AzureReservedIP](/powershell/module/servicemanagement/azure.service/move-azurereservedip?view=azuresmps-4.0.0).  |
| Adresse IP publique par machine virtuelle |Adresse IP publique avec méthode d’allocation dynamique |L’adresse IP publique associée à la machine virtuelle est convertie en ressource d’adresse IP publique avec la méthode d’allocation statique. |
| Groupes de sécurité réseau |Groupes de sécurité réseau |Les groupes de sécurité réseau (NSG) associés à un sous-réseau sont clonés dans le cadre de la migration vers le modèle de déploiement Resource Manager. Le groupe de sécurité réseau (NSG) du modèle de déploiement classique n’est pas supprimé durant la migration. Toutefois, les opérations du plan de gestion relatives au NSG sont bloquées pendant le processus de migration. Vous pouvez migrer des groupes de sécurité réseau non associés à l’aide de la commande [Move-AzureNetworkSecurityGroup](/powershell/module/servicemanagement/azure.service/move-azurenetworksecuritygroup?view=azuresmps-4.0.0).|
| Serveurs DNS |Serveurs DNS |La migration des serveurs DNS associés à un réseau virtuel ou à la machine virtuelle est effectuée dans le cadre de la migration des ressources correspondantes avec toutes les propriétés. |
| Itinéraires définis par l’utilisateur (UDR) |Itinéraires définis par l’utilisateur (UDR) |Les Itinéraires définis par l’utilisateur associés à un sous-réseau sont clonés dans le cadre de la migration vers le modèle de déploiement Resource Manager. L’itinéraire défini par l’utilisateur dans le modèle de déploiement classique n’est pas supprimé durant la migration. Les opérations du plan de gestion relatives pour l’itinéraire défini par l’utilisateur sont bloquées pendant le processus de migration. Vous pouvez migrer des Itinéraires définis par l’utilisateur (UDR) non associés à l’aide de la commande [Move-AzureRouteTable](/powershell/module/servicemanagement/azure.service/Move-AzureRouteTable?view=azuresmps-4.0.0). |
| Propriété de transfert IP sur la configuration réseau d’une machine virtuelle |Propriété de transfert IP sur la carte d’interface réseau (NIC) |La propriété de transfert IP sur une machine virtuelle est convertie en propriété sur l’interface réseau pendant la migration. |
| Équilibreur de charge avec plusieurs adresses IP |Équilibreur de charge avec plusieurs ressources d’adresses IP publiques |Chaque adresse IP publique associée à l’équilibreur de charge est convertie en une ressource d’adresse IP publique et est associée à l’équilibreur de charge après la migration. |
| Noms DNS internes sur la machine virtuelle |Noms DNS internes sur la carte réseau |Pendant la migration, les suffixes DNS internes pour les machines virtuelles sont migrés vers une propriété en lecture seule nommée « InternalDomainNameSuffix » sur la carte réseau. Le suffixe reste inchangé après la migration et la résolution de la machine virtuelle doit continuer à fonctionner comme auparavant. |
| Passerelle de réseau virtuel |Passerelle de réseau virtuel |Les propriétés de la passerelle de réseau virtuel sont inchangées après la migration. L’adresse IP virtuelle associée à la passerelle reste inchangée également. |
| Site de réseau local |Passerelle de réseau local |Les propriétés du site de réseau local sont migrées sans modification vers une nouvelle ressource appelée Passerelle de réseau local. Ceci représente les préfixes d’adresse locale et l’adresse IP de la passerelle distante. |
| Références de connexions |Connexion |Les références de connectivité entre la passerelle et le site réseau local dans la configuration du réseau sont représentées par une nouvelle ressource appelée Connexion. Toutes les propriétés de référence de connectivité dans les fichiers de configuration réseau sont copiées sans modification vers la ressource Connexion. La connectivité entre les réseaux virtuels dans le modèle de déploiement classique est obtenue en créant deux tunnels IPsec pour les sites de réseau local représentant les réseaux virtuels. Elle est transformée en type de connexion réseau virtuel à réseau virtuel dans le modèle Resource Manager sans qu’il soit nécessaire de faire appel à des passerelles de réseau local. |

## <a name="changes-to-your-automation-and-tooling-after-migration"></a>Modifications de vos services d’automatisation et de vos outils après la migration
Dans le cadre de la migration de vos ressources depuis le modèle de déploiement classique vers le modèle de déploiement Resource Manager, vous devez mettre à jour votre automatisation ou vos outils existants pour vous assurer qu’il continue de fonctionner après la migration.


## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble de la migration prise en charge par la plateforme de ressources IaaS Classic vers Azure Resource Manager](migration-classic-resource-manager-overview.md)
* [Planification de la migration des ressources IaaS d’Azure Classic vers Azure Resource Manager](migration-classic-resource-manager-plan.md)
* [Faire migrer des ressources IaaS Classic vers Azure Resource Manager à l’aide d’Azure PowerShell](migration-classic-resource-manager-ps.md)
* [Faire migrer des ressources IaaS Classic vers Azure Resource Manager à l’aide de l’interface de ligne de commande Azure](migration-classic-resource-manager-cli.md)
* [Migration de la passerelle VPN du modèle Classic vers le modèle Resource Manager](../vpn-gateway/vpn-gateway-classic-resource-manager-migration.md)
* [Migrer des circuits ExpressRoute et les réseaux virtuels associés du modèle de déploiement classique au modèle de déploiement Resource Manager](../expressroute/expressroute-migration-classic-resource-manager.md)
* [Outils de la communauté pour aider à la migration de ressources IaaS de Classic vers Azure Resource Manager](migration-classic-resource-manager-community-tools.md)
* [Passer en revue les erreurs courantes de migration](migration-classic-resource-manager-errors.md)
* [Passez en revue les questions fréquemment posées sur la migration des ressources IaaS de Classic vers Azure Resource Manager](migration-classic-resource-manager-faq.md)
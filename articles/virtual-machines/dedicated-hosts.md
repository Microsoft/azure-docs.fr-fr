---
title: Vue d’ensemble des Azure Dedicated Hosts pour machines virtuelles
description: Découvrez-en plus sur la manière dont les hôtes dédiés Azure peuvent être utilisés pour déployer des machines virtuelles.
author: cynthn
ms.service: virtual-machines
ms.topic: conceptual
ms.workload: infrastructure
ms.date: 12/07/2020
ms.author: cynthn
ms.reviewer: zivr
ms.openlocfilehash: 4e29bb0fee496af6a8c0fd30d5559bf865123c39
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97007889"
---
# <a name="azure-dedicated-hosts"></a>Hôtes dédiés Azure

Un hôte dédié Azure est un service qui fournit des serveurs physiques capables d’héberger une ou plusieurs machines virtuelles, dédiés à un abonnement Azure. Les hôtes dédiés sont les mêmes serveurs physiques que ceux utilisés dans nos centres de données, fournis en tant que ressource. Vous pouvez approvisionner des hôtes dédiés au sein d’une région, d’une zone de disponibilité et d’un domaine d’erreur. Ensuite, vous pouvez placer des machines virtuelles directement dans vos hôtes approvisionnés, dans la configuration qui répond le mieux à vos besoins.


## <a name="benefits"></a>Avantages 

La réservation de l’intégralité de l’hôte offre les avantages suivants:

-   Isolation matérielle au niveau du serveur physique. Aucune autre machine virtuelle ne sera placée sur vos hôtes. Les hôtes dédiés sont déployés dans les mêmes centres de données et partagent le même réseau et la même infrastructure de stockage sous-jacente que les autres hôtes non isolés.
-   Contrôle des événements de maintenance initiés par la plateforme Azure. Alors que la majorité des événements de maintenance n’ont que peu d’impact sur vos machines virtuelles, il existe des charges de travail sensibles où chaque seconde de pause peut avoir un impact. Avec les hôtes dédiés, vous pouvez vous abonner à une fenêtre de maintenance pour réduire l’impact sur votre service.
-   Avec l’offre Azure Hybrid Benefit, vous pouvez apporter vos propres licences pour Windows et SQL à Azure. L’utilisation des avantages Hybrid Benefit vous offre des avantages supplémentaires. Pour plus d’informations, consultez [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/).


## <a name="groups-hosts-and-vms"></a>Groupes, hôtes et machines virtuelles  

![Vue des nouvelles ressources pour les hôtes dédiés.](./media/virtual-machines-common-dedicated-hosts/dedicated-hosts2.png)

Un **groupe hôte** est une ressource qui représente une collection d’hôtes dédiés. Vous créez un groupe hôte dans une région et une zone de disponibilité, et lui ajoutez des hôtes.

Un **hôte** est une ressource, mappée à un serveur physique dans un centre de données Azure. Le serveur physique est alloué lors de la création de l’hôte. Un hôte est créé dans un groupe hôte. Un hôte dispose d’une référence SKU décrivant les tailles de machine virtuelle qui peuvent être créées. Chaque hôte peut héberger plusieurs machines virtuelles de différentes tailles, à condition qu’elles proviennent de la même série de tailles.


## <a name="high-availability-considerations"></a>Considérations relatives à la haute disponibilité 

Pour une haute disponibilité, vous devez déployer plusieurs machines virtuelles, réparties sur plusieurs hôtes (au minimum 2). Avec les hôtes dédiés Azure, vous disposez de plusieurs options pour configurer votre infrastructure afin de mettre en forme vos limites d’isolation des erreurs.

### <a name="use-availability-zones-for-fault-isolation"></a>Utiliser des zones de disponibilité pour l’isolation des erreurs

Les Zones de disponibilité sont des emplacements physiques uniques au sein d’une région Azure. Chaque zone de disponibilité est composée d’un ou de plusieurs centres de données équipés d’une alimentation, d’un système de refroidissement et d’un réseau indépendants. Un groupe hôte est créé dans une seule zone de disponibilité. Une fois créés, tous les hôtes sont placés dans cette zone. Pour obtenir une haute disponibilité entre les zones, vous devez créer plusieurs groupes hôtes (un par zone) et répartir vos hôtes en conséquence.

Si vous affectez un groupe hôte à une zone de disponibilité, toutes les machines virtuelles créées sur cet hôte doivent être créées dans la même zone.

### <a name="use-fault-domains-for-fault-isolation"></a>Utiliser des domaines d’erreur pour l’isolation des erreurs

Un hôte peut être créé dans un domaine d’erreur spécifique. À l’instar d’une machine virtuelle dans un groupe identique ou un groupe à haute disponibilité, les hôtes situés dans différents domaines d’erreur sont placés sur des racks physiques différents dans le centre de données. Lorsque vous créez un groupe hôte, vous devez spécifier le nombre de domaines d’erreur. Lorsque vous créez des hôtes dans le groupe hôte, vous affectez un domaine d’erreur pour chaque hôte. Les machines virtuelles ne nécessitent aucune attribution de domaine d’erreur.

Des domaines d’erreur n’équivalent pas à une colocation. Le fait d’avoir le même domaine d’erreur pour deux hôtes ne signifie pas qu’ils sont à proximité les uns des autres.

Les domaines d’erreur sont étendus au groupe hôte. Vous ne devez pas faire d’hypothèses sur l’anti-affinité entre deux groupes hôtes (sauf s’ils se trouvent dans des zones de disponibilité différentes).

Les machines virtuelles déployées sur des hôtes avec des domaines d’erreur différents auront leurs services sous-jacents sur plusieurs tampons de stockage, afin d’augmenter la protection contre l’isolement des erreurs.

### <a name="using-availability-zones-and-fault-domains"></a>Utilisation de zones de disponibilité et de domaines d’erreur

Vous pouvez utiliser ces deux fonctionnalités ensemble pour obtenir une isolation des erreurs encore plus étendue. Dans ce cas, vous allez spécifier la zone de disponibilité et le nombre de domaines d’erreur pour chaque groupe hôte, attribuer un domaine d’erreur à chacun de vos hôtes dans le groupe et affecter une zone de disponibilité à chacune de vos machines virtuelles.

L’exemple de modèle Resource Manager, trouvé [ici](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-dedicated-hosts/README.md), utilise les zones et les domaines d’erreur pour diffuser des hôtes et obtenir une résilience maximale dans une région.


## <a name="manual-vs-automatic-placement"></a>Sélection élective manuelle ou automatique 

Lorsque vous créez une machine virtuelle dans Azure, vous pouvez sélectionner l’hôte dédié à utiliser. Vous pouvez également utiliser l’option pour placer automatiquement vos machines virtuelles sur des hôtes existants, au sein d’un groupe hôte. 

Lors de la création d’un groupe hôte, vérifiez que le paramètre de sélection élective automatique est sélectionné. Lorsque vous créez votre machine virtuelle, sélectionnez le groupe hôte et laissez Azure sélectionner le meilleur hôte pour votre machine virtuelle. 

Les groupes hôtes activés pour la sélection élective automatique n’exigent pas que toutes les machines virtuelles soient placées automatiquement. Vous pourrez toujours choisir explicitement un hôte, même si la sélection élective automatique est sélectionnée pour le groupe hôte. 

### <a name="limitations"></a>Limites

Problèmes connus et limitations lors de l’utilisation de la sélection élective automatique :

- Vous ne pourrez pas appliquer Azure Hybrid Benefit sur vos hôtes dédiés.
- Vous ne pourrez pas redéployer votre machine virtuelle. 
- Vous ne pourrez pas utiliser de machines virtuelles Lsv2, NVasv4, NVsv3, Msv2 ou de série M avec des hôtes dédiés. 


## <a name="virtual-machine-scale-set-support"></a>Prise en charge de groupes de machines virtuelles identiques

Les groupes de machines virtuelles identiques vous permettent de traiter un groupe de machines virtuelles en tant que ressource unique, et d’appliquer des stratégies de disponibilité, de gestion, de mise à l’échelle et d’orchestration en tant que groupe. Vos hôtes dédiés existants peuvent également être utilisés pour les groupes de machines virtuelles identiques. 

Lorsque vous créez un groupe de machines virtuelles identiques, vous pouvez spécifier un groupe hôte existant afin que toutes les instances de machine virtuelle soient créées sur des hôtes dédiés.

Les conditions suivantes s’appliquent lors de la création d’un groupe de machines virtuelles identiques dans un groupe hôte dédié :

- La sélection élective automatique doit être activée.
- Le paramètre de disponibilité de votre groupe hôte doit correspondre à votre groupe identique. 
    - Un groupe hôte régional (créé sans spécifier de zone de disponibilité) doit être utilisé pour les groupes identiques régionaux.
    - Le groupe hôte et le groupe identique doivent utiliser la même zone de disponibilité. 
    - Le nombre de domaines d’erreur pour le niveau du groupe hôte doit correspondre au nombre de domaines d’erreur pour votre groupe identique. Le portail Azure vous permet de spécifier la *diffusion maximale* pour votre groupe identique, qui définit le nombre de domaines d’erreur sur 1.
- Les hôtes dédiés doivent être créés en premier, avec une capacité suffisante, et avec les mêmes paramètres pour les zones de groupe identique et les domaines d’erreur.
- Les tailles de machines virtuelles prises en charge pour vos hôtes dédiés doivent correspondre à celle utilisée pour votre groupe identique.

Les paramètres d’optimisation et d’orchestration de groupe identique ne sont pas tous pris en charge par les hôtes dédiés. Appliquez les paramètres suivants à votre groupe identique : 
- Un surprovisionnement n’est pas recommandé et il est désactivé par défaut. Vous pouvez activer le surprovisionnement, mais l’allocation du groupe identique échoue si le groupe d’hôtes n’a pas la capacité nécessaire pour toutes les machines virtuelles, y compris les instances surprovisionnées. 
- Utilisez le mode d’orchestration ScaleSetVM. 
- N’utilisez pas de groupes de placement de proximité pour la colocalisation.



## <a name="maintenance-control"></a>Contrôle de la maintenance

L’infrastructure qui prend en charge vos machines virtuelles peut parfois être mise à jour pour améliorer la fiabilité, les performances, la sécurité et pour lancer de nouvelles fonctionnalités. La plateforme Azure tente de réduire l’impact de la maintenance de la plateforme chaque fois que cela est possible, mais les clients avec des charges de travail *sensibles à la maintenance* ne peuvent tolérer ( même pendant quelques secondes) que la machine virtuelle doive être gelée ou déconnectée pour la maintenance.

**Le contrôle de maintenance** offre aux clients la possibilité d’ignorer les mises à jour régulières de la plateforme planifiées sur leurs hôtes dédiés, puis de les appliquer au moment de leur choix dans une fenêtre de 35 jours. Dans la fenêtre de maintenance, vous pouvez appliquer la maintenance directement au niveau de l’hôte, dans n’importe quel ordre. Une fois la fenêtre de maintenance dépassée, Microsoft passe à l’étape suivante et applique la maintenance en attente aux hôtes dans un ordre qui peut ne pas suivre les domaines d’erreur définis par l’utilisateur.

Pour plus d’informations, consultez [Gestion des mises à jour des plateformes avec le contrôle de maintenance](./maintenance-control.md).

## <a name="capacity-considerations"></a>Considérations relatives à la capacité

Une fois qu’un hôte dédié est approvisionné, Azure l’attribue au serveur physique. Cela garantit la disponibilité de la capacité lorsque vous avez besoin d’approvisionner votre machine virtuelle. Azure utilise la totalité de la capacité dans la région (ou zone) pour choisir un serveur physique pour votre hôte. Cela signifie également que les clients peuvent s’attendre à pouvoir augmenter l’encombrement de leur hôte dédié sans avoir à manquer d’espace dans le cluster.

## <a name="quotas"></a>Quotas

Deux types de quotas sont utilisés lorsque vous déployez un hôte dédié.

1. Quota de processeurs virtuels hôtes dédiés. La limite de quota par défaut est de 3000 processeurs virtuels par région.
1. Quota de famille de tailles de machine virtuelle. Par exemple, un abonnement avec **paiement à l’utilisation** peut uniquement avoir un quota de 10 processeurs virtuels disponibles pour la série de tailles Dsv3, dans la région USA Est. Pour déployer un hôte dédié Dsv3, vous devez demander une augmentation de quota à au moins 64 processeurs virtuels avant de pouvoir déployer l’hôte dédié. 

Pour demander une augmentation du quota, créez une demande de support dans le [portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

L’approvisionnement d’un hôte dédié consomme le quota de processeurs virtuels hôtes dédiés et le quota de famille de tailles de machine virtuelle, mais il ne consomme pas les processeurs virtuels régionaux.


![Capture d’écran de la page d’utilisation et des quotas dans le portail](./media/virtual-machines-common-dedicated-hosts/quotas.png)

Pour plus d’informations, consultez [Quotas de processeurs virtuels pour les machines virtuelles](./windows/quotas.md).

L’évaluation gratuite et les abonnements MSDN n’ont pas de quota pour les hôtes dédiés Azure.

## <a name="pricing"></a>Tarifs

Les utilisateurs sont facturés par hôte dédié, quel que soit le nombre de machines virtuelles déployées. Dans votre relevé mensuel, vous verrez un nouveau type de ressource facturable d’hôtes. Les machines virtuelles sur un hôte dédié seront toujours affichées dans votre instruction, mais le prix sera de 0.

Le prix de l’hôte est défini en fonction de la série de machines virtuelles, du type (taille matérielle) et de la région. Un prix d’hôte est relatif à la taille de machine virtuelle la plus grande prise en charge sur l’hôte.

Les licences logicielles, le stockage et l’utilisation du réseau sont facturés séparément de l’hôte et des machines virtuelles. Il n’y aucune modification pour ces éléments facturables.

Pour plus d’informations, consultez la page [Tarification pour hôte dédié Azure](https://aka.ms/ADHPricing).

Vous pouvez également économiser sur les coûts grâce à une [instance réservée d’ordinateurs Azure Dedicated Host](prepay-dedicated-hosts-reserved-instances.md).
 
## <a name="sizes-and-hardware-generations"></a>Tailles et générations de matériel

Une référence SKU est définie pour un hôte et représente la série et le type de la taille de machine virtuelle. Vous pouvez combiner plusieurs machines virtuelles de différentes tailles au sein d’un même hôte, à condition qu’elles soient de la même série de tailles. 

Le *type* indique la génération du matériel. Les différents types de matériel pour les mêmes séries de machines virtuelles proviennent de différents fournisseurs de processeurs, et ont des générations d’UC et des nombres de cœurs différents. 

Les tailles et les types de matériel varient en fonction de la région. Pour en savoir plus, consultez la [page de tarification de l’hôte](https://aka.ms/ADHPricing).

> [!NOTE]
> Une fois qu’un hôte dédié est provisionné, vous ne pouvez pas modifier la taille ou le type. Si vous avez besoin d’une taille de type différente, vous devez créer un hôte.  

## <a name="host-life-cycle"></a>Cycle de vie de l’hôte


Azure surveille et gère l’état d’intégrité de vos hôtes. Les états suivants sont retournés lorsque vous interrogez votre hôte :

| État d’intégrité   | Description       |
|----------|----------------|
| Hôte disponible     | Il n’y a aucun problème connu avec votre hôte.   |
| Examen en cours de l’hôte  | Nous enquêtons sur des problèmes que nous rencontrons avec l’hôte. Il s’agit d’un état transitoire requis pour qu’Azure tente d’identifier l’étendue et la cause racine du problème identifié. Les machines virtuelles en cours d’exécution sur l’hôte peuvent être affectées. |
| Hôte en attente de désallocation   | Azure ne peut pas restaurer l’hôte dans un état sain et vous demande de redéployer vos machines virtuelles hors de cet hôte. Si `autoReplaceOnFailure` est activé, vos machines virtuelles *sont réparées* sur du matériel sain. Dans le cas contraire, votre machine virtuelle est peut-être en cours d’exécution sur un hôte qui va échouer.|
| Hôte désalloué  | Toutes les machines virtuelles ont été supprimées de l’hôte. Vous n’êtes plus facturé pour cet hôte, car le matériel a été retiré de la rotation.   |


## <a name="next-steps"></a>Étapes suivantes

- Vous pouvez déployer un hôte dédié à l’aide d’[Azure PowerShell](./windows/dedicated-hosts-powershell.md), du [portail](./dedicated-hosts-portal.md) et d’[Azure CLI](./linux/dedicated-hosts-cli.md).

- Un exemple de modèle, disponible [ici](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-dedicated-hosts/README.md), utilise les zones et les domaines d’erreur pour offrir une résilience maximale dans une région.

- Vous pouvez également économiser sur les coûts grâce à une [instance réservée d’ordinateurs Azure Dedicated Host](prepay-dedicated-hosts-reserved-instances.md).

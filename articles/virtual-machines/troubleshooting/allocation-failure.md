---
title: Résolution des échecs d’allocation des machines virtuelles Azure | Microsoft Docs
description: Résoudre les problèmes d’allocation pendant la création, le redémarrage ou le redimensionnement d’une machine virtuelle dans Azure
services: virtual-machines
documentationcenter: ''
author: DavidCBerry13
manager: felixwu
editor: ''
tags: top-support-issue,azure-resource-manager,azure-service-management
ms.assetid: 1ef41144-6dd6-4a56-b180-9d8b3d05eae7
ms.service: virtual-machines
ms.topic: troubleshooting
ms.date: 11/06/2020
ms.author: daberry
ms.openlocfilehash: 79bc043a991404a3ee9da954b9639bf1a41f2c51
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94365871"
---
# <a name="troubleshoot-allocation-failures-when-you-create-restart-or-resize-vms-in-azure"></a>Résoudre les problèmes d’allocation pendant la création, le redémarrage ou le redimensionnement de machines virtuelles dans Azure

Quand vous créez une machine virtuelle, redémarrez des machines virtuelles ayant été arrêtées (libérées) ou redimensionnez une machine virtuelle, Microsoft Azure alloue des ressources de calcul à votre abonnement. Nous investissons constamment dans l’infrastructure et dans l’ajout de fonctionnalités afin de garantir la disponibilité de tous les types de machines virtuelles et de répondre aux besoins des clients. Toutefois, il est possible que vous rencontriez des échecs d’allocation de ressources en raison d’une augmentation sans précédent de la demande de services Azure dans certaines régions. Ce problème peut se produire lorsque vous essayez de créer ou de démarrer des machines virtuelles dans une région, et que celles-ci affichent le code d’erreur et le message suivants :

**Code d’erreur** : AllocationFailed ou ZonalAllocationFailed

**Message d’erreur** : « Allocation failed. We do not have sufficient capacity for the requested VM size in this region. Pour en savoir plus sur l’amélioration de la probabilité de réussite de l’allocation, voir https :\//aka.ms/allocation-guidance

> [!NOTE]
> Si vous résolvez des problèmes de groupe de machines virtuelles identiques (VMSS), le processus est le même que pour une machine virtuelle standard. Pour résoudre le problème, suivez les instructions de cet article.
> 
>**Message d’erreur** : « L'allocation a échoué. Si vous essayez d’ajouter une nouvelle machine virtuelle à un groupe de machines virtuelles identiques avec un seul groupe de placements ou de mettre à jour/redimensionner une machine virtuelle existante dans un groupe de machines virtuelles identiques avec un seul groupe de placements, sachez qu’une telle allocation est limitée à un seul cluster dont la capacité peut être insuffisante. Pour en savoir plus sur l’amélioration de la probabilité de réussite de l’allocation, consultez la page http:\//aka.ms/allocation-guidance. »

Cet article explique les causes de certains échecs d’allocation courants et propose des solutions possibles.

Si votre problème avec Azure n’est pas traité dans cet article, parcourez les [forums Azure sur MSDN et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez publier votre problème sur ces forums ou sur @AzureSupport sur Twitter. Vous pouvez également créer une demande de support Azure en sélectionnant Obtenir de l’aide sur le site du [support Azure](https://azure.microsoft.com/support/options/).

En attendant que votre type de machine virtuelle préféré soit disponible dans votre région par défaut, il est conseillé aux clients qui rencontrent des problèmes de déploiement de suivre les instructions du tableau suivant en guise de solution de contournement temporaire. 

Identifiez le scénario qui correspond le mieux à votre cas, puis renvoyez votre demande d’allocation à l’aide de la solution de contournement suggérée pour accroître les chances de réussite de l’allocation. Vous pouvez également réessayer à un autre moment. Cela signifie que suffisamment de ressources ont été libérées dans le cluster, la région ou la zone pour répondre à votre demande. 


## <a name="resize-a-vm-or-add-vms-to-an-existing-availability-set"></a>Redimensionner une machine virtuelle ou ajouter des machines virtuelles à un groupe à haute disponibilité existant

### <a name="cause"></a>Cause

Une demande pour redimensionner une machine virtuelle ou ajouter une machine virtuelle à un groupe à haute disponibilité existant doit être effectuée sur le cluster d’origine qui héberge le groupe à haute disponibilité existant. La taille de machine virtuelle demandée est prise en charge par le cluster, mais le cluster peut ne pas avoir la capacité suffisante pour le moment. 

### <a name="workaround"></a>Solution de contournement

Si la machine virtuelle peut faire partie d’un autre groupe à haute disponibilité, créez une machine virtuelle dans un autre groupe à haute disponibilité (dans la même région). Cette nouvelle machine virtuelle peut ensuite être ajoutée au même réseau virtuel.

Arrêtez (libérez) toutes les machines virtuelles dans le même groupe à haute disponibilité, puis redémarrez chacune d’elles.
Procédure d’arrêt : Cliquez sur Groupes de ressources > [votre groupe de ressources] > Ressources > [votre groupe à haute disponibilité] > Machines virtuelles > [votre machine virtuelle] > Arrêter.
Une fois toutes les machines virtuelles arrêtées, sélectionnez la première, puis cliquez sur Démarrer.
Cette étape permet de retenter une allocation et de sélectionner un autre cluster disposant d’une capacité suffisante.

## <a name="restart-partially-stopped-deallocated-vms"></a>Redémarrer des machines virtuelles partiellement arrêtées (désallouées)

### <a name="cause"></a>Cause

Une désallocation partielle signifie que vous avez arrêté (désalloué) une ou plusieurs machines virtuelles, mais pas toutes, dans un groupe à haute disponibilité. Quand vous libérez une machine virtuelle, les ressources associées sont elles aussi libérées. Redémarrer des machines virtuelles dans un groupe à haute disponibilité partiellement libéré équivaut à ajouter des machines virtuelles à un groupe à haute disponibilité existant. Par conséquent, la demande d’allocation doit être tentée sur le cluster d’origine qui héberge le groupe à haute disponibilité existant, lequel peut ne pas disposer d’une capacité suffisante.

### <a name="workaround"></a>Solution de contournement

Arrêtez (libérez) toutes les machines virtuelles dans le même groupe à haute disponibilité, puis redémarrez chacune d’elles.
Procédure d’arrêt : Cliquez sur Groupes de ressources > [votre groupe de ressources] > Ressources > [votre groupe à haute disponibilité] > Machines virtuelles > [votre machine virtuelle] > Arrêter.
Une fois toutes les machines virtuelles arrêtées, sélectionnez la première, puis cliquez sur Démarrer.
Ainsi, vous pouvez effectuer une nouvelle tentative d’allocation et sélectionner un autre cluster disposant d’une capacité suffisante.

## <a name="restart-fully-stopped-deallocated-vms"></a>Redémarrer des machines virtuelles complètement arrêtées (désallouées)

### <a name="cause"></a>Cause

Une désallocation complète signifie que vous avez arrêté (désalloué) toutes les machines virtuelles d’un groupe à haute disponibilité. La demande d’allocation pour redémarrer ces machines virtuelles cible tous les clusters qui prennent en charge la taille souhaitée dans la région ou la zone. Modifiez votre demande d’allocation en suivant les suggestions de cet article, puis renvoyez-la pour accroître les chances de réussite de l’allocation. 

### <a name="workaround"></a>Solution de contournement

Si vous utilisez d’anciennes séries ou tailles de machines virtuelles (telles que Dv1, DSv1, Av1, D15v2 ou DS15v2), nous vous conseillons de passer à des versions plus récentes. Consultez les recommandations applicables à chaque taille de machine virtuelle.
Si vous ne pouvez pas utiliser une autre taille de machine virtuelle, essayez de la déployer dans une autre région de la même zone géographique. Pour plus d’informations sur les tailles de machine virtuelle disponibles dans chaque région, consultez https://aka.ms/azure-regions.

Si vous utilisez des zones de disponibilité, essayez une autre zone de la même la région susceptible d’avoir une capacité disponible pour la taille de machine virtuelle demandée.

Si votre demande d’allocation est importante (plus de 500 cœurs), suivez les instructions des sections suivantes pour fractionner la demande en plusieurs petits déploiements.

Essayez de [redéployer la machine virtuelle](./redeploy-to-new-node-windows.md). Le redéploiement de la machine virtuelle alloue la machine virtuelle à un nouveau cluster dans la région.

## <a name="allocation-failures-for-older-vm-sizes-av1-dv1-dsv1-d15v2-ds15v2-etc"></a>Échecs d’allocation pour les anciennes tailles de machine virtuelle (Av1, Dv1, DSv1, D15v2, DS15v2, etc.)

À mesure que nous étendons l’infrastructure Azure, nous déployons du matériel de nouvelle génération conçu pour prendre en charge les types de machines virtuelles les plus récents. Certaines anciennes machines virtuelles ne peuvent pas être exécutées dans notre infrastructure de dernière génération. Les clients peuvent donc rencontrer des échecs d’allocation avec ces références SKU héritées. Pour éviter ce problème, nous encourageons les clients qui utilisent des machines virtuelles appartenant à d’anciennes séries de passer à l’équivalent récent de leurs machines virtuelles en suivant les recommandations ci-dessous. Ces machines virtuelles sont optimisées pour la dernière génération de matériel et vous permettent de profiter de meilleurs tarifs et de meilleures performances. 

|Taille/Série de machine virtuelle héritée|Taille/Série de machine virtuelle recommandée|Plus d’informations|
|----------------------|----------------------------|--------------------|
|Série Av1|[Série Av2](../av2-series.md)|https://azure.microsoft.com/blog/new-av2-series-vm-sizes/
|Série Dv1 ou DSv1 (D1 à D5)|[Série Dv3 ou DSv3](../dv3-dsv3-series.md)|https://azure.microsoft.com/blog/introducing-the-new-dv3-and-ev3-vm-sizes/
|Série Dv1 ou DSv1 (D11 à D14)|[Série Ev3 ou ESv3](../ev3-esv3-series.md)|
|D15v2 ou DS15v2|Si vous utilisez le modèle de déploiement Resource Manager pour obtenir de plus grandes machines virtuelles, vous pouvez passer aux séries D16v3/DS16v3 ou D32v3/DS32v3. Celles-ci sont conçues pour s’exécuter sur le matériel de dernière génération. Si vous utilisez le modèle de déploiement Resource Manager pour isoler votre instance de machine virtuelle du matériel dédié à un seul client, vous pouvez passer aux nouvelles tailles de machines virtuelles isolées, comme E64i_v3 ou E64is_v3, qui sont conçues pour s’exécuter sur le matériel de dernière génération. |https://azure.microsoft.com/blog/new-isolated-vm-sizes-now-available/

## <a name="allocation-failures-for-large-deployments-more-than-500-cores"></a>Échecs d’allocation pour les déploiements de grande échelle (plus de 500 cœurs)

Réduisez le nombre d’instances de la taille de machine virtuelle demandée, puis recommencez le déploiement. De plus, pour les déploiements plus importants, vous pouvez évaluer les [groupes de machines virtuelles identiques Azure](../../virtual-machine-scale-sets/index.yml). Le nombre d’instances de machines virtuelles peut augmenter ou diminuer automatiquement en réponse à la demande ou au calendrier défini. L’allocation a donc plus de chances de réussir, puisque les déploiements peuvent être répartis sur plusieurs clusters. 

## <a name="background-information"></a>Informations contextuelles
### <a name="how-allocation-works"></a>Fonctionnement de l’allocation
Les serveurs des centres de données Azure sont partitionnés en clusters. En règle générale, les tentatives de demande d’allocation sont exécutées dans plusieurs clusters, mais il est possible que certaines contraintes de la demande d’allocation forcent la plateforme Azure à exécuter la tentative dans un seul cluster. Dans cet article, nous parlerons « d’épinglage à un cluster ». La Figure 1 ci-dessous illustre le cas d’une allocation normale dans laquelle la tentative est exécutée dans plusieurs clusters. La Figure 2 illustre le cas d’une allocation épinglée au Cluster 2, là où est hébergé le service cloud CS_1 ou le groupe à haute disponibilité.
![Schéma d’allocation](./media/virtual-machines-common-allocation-failure/Allocation1.png)

### <a name="why-allocation-failures-happen"></a>Raisons des échecs d’une allocation
Lorsqu’une demande d’allocation est épinglée à un cluster, il y a plus de risque de ne pas trouver les ressources disponibles puisque le pool de ressources disponibles est réduit. En outre, si votre demande d’allocation est épinglée à un cluster alors que le type de ressource que vous avez demandé n’est pas pris en charge par ce cluster, votre demande échouera, même si le cluster comporte des ressources disponibles. Le diagramme 3 ci-dessous illustre le cas d’une allocation épinglée qui échoue, car le seul cluster candidat ne comporte pas de ressources disponibles. La Figure 4 illustre le cas de figure où une allocation épinglée échoue parce que le seul cluster candidat ne prend pas en charge la taille de machine virtuelle demandée, bien qu'il puisse libérer des ressources.

![Échec d’allocation épinglée](./media/virtual-machines-common-allocation-failure/Allocation2.png)

---
title: Groupes de placement de proximité Azure pour les applications SAP | Microsoft Docs
description: Décrit des scénarios de déploiement SAP avec des groupes de placement de proximité Azure
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/29/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: aae822665702300064e82e80d74b5c2256423ea1
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94957280"
---
# <a name="azure-proximity-placement-groups-for-optimal-network-latency-with-sap-applications"></a>Groupes de placement de proximité Azure pour une latence réseau optimale avec les applications SAP
Les applications SAP basées sur l’architecture SAP NetWeaver ou SAP S/4HANA sont sensibles à la latence réseau entre la couche Application SAP et la couche Base de données SAP. Cette sensibilité est due au fait que la plupart de la logique métier s’exécute dans la couche Application. Étant donné que la couche Application SAP exécute la logique métier, elle émet des requêtes à la base de données à une fréquence élevée (plusieurs milliers ou dizaines de milliers par seconde). Dans la plupart des cas, la nature de ces requêtes est simple. Elles peuvent souvent être exécutées sur la couche Base de données en 500 microsecondes ou moins.

Le temps passé sur le réseau à envoyer une telle requête de la couche Application à la couche Base de données et à recevoir le jeu de résultats a un impact majeur sur le temps nécessaire à l’exécution des processus métier. Cette sensibilité à la latence réseau est la raison pour laquelle vous souhaiterez peut-être atteindre une latence réseau maximale donnée dans les projets de déploiement SAP. Voir la [Note de support SAP 1100926 - FAQ : Performances réseau](https://launchpad.support.sap.com/#/notes/1100926/E) pour obtenir des instructions sur la façon de classifier la latence réseau.

Dans de nombreuses régions Azure, le nombre de centres de données a augmenté. En même temps, les clients, en particulier pour les systèmes SAP haut de gamme, utilisent des références SKU de machines virtuelles spéciales de la série M ou Mv2, ou de grandes instances HANA. Ces types de machines virtuelles Azure ne sont pas toujours disponibles dans tous les centres de données qui complètent une région Azure spécifique. Ces faits peuvent créer des opportunités d’optimisation de la latence du réseau entre la couche application SAP et la couche SGBD SAP.

Pour vous permettre d’optimiser la latence du réseau, Azure propose des [groupes de placement de proximité](../../linux/co-location.md). Les groupes de placement de proximité peuvent être utilisés pour forcer le regroupement de différents types de machines virtuelles dans un même centre de donnée Azure afin d’optimiser au maximum la latence du réseau entre ces différents types de machines virtuelles. Lors du déploiement de la première machine virtuelle dans un groupe de placement de proximité, la machine virtuelle est liée à un centre de donnée spécifique. Comme c’est le cas pour ce prospect, l’utilisation de la construction introduit également des restrictions :

- Vous ne pouvez pas supposer que tous les types de machines virtuelles Azure sont disponibles dans tous les centres de données Azure. Par conséquent, la combinaison de différents types de machines virtuelles dans un groupe de placement de proximité peut être restreinte. Ces restrictions sont dues au fait que le matériel hôte nécessaire pour exécuter un certain type de machine virtuelle peut ne pas être présent dans le centre de donnés où le groupe de placement a été déployé
- Lorsque vous redimensionnez des parties de machines virtuelles qui se trouvent dans un groupe de placement de proximité, vous ne pouvez pas supposer automatiquement que, dans tous les cas, le nouveau type de machine virtuelle est disponible dans le même centre de centres que les autres machines virtuelles qui font partie du groupe de placement de proximité
- Comme Azure désactive le matériel, il peut forcer certaines machines virtuelles d’un groupe de placement de proximité dans un autre centre de donnés Azure. Pour plus d’informations sur ce cas, lisez le document [Colocaliser des ressources pour améliorer la latence](../../linux/co-location.md#planned-maintenance-and-proximity-placement-groups)  

> [!IMPORTANT]
> En raison de restrictions potentielles, les groupes de placement de proximité doivent être utilisés :
>
> - Uniquement lorsque cela est nécessaire
> - Uniquement sur la granularité d’un seul système SAP et non pour l’ensemble du paysage système ou SAP complet
> - Pour conserver les différents types de machines virtuelles et un nombre minimum de machines virtuelles dans un groupe de placement de proximité


## <a name="what-are-proximity-placement-groups"></a>Définition des groupes de placement de proximité 
Un groupe de placement de proximité Azure est une construction logique. Lorsqu’un groupe de placement de proximité est défini, il est lié à une région Azure et à un groupe de ressources Azure. Quand des machines virtuelles sont déployées, un groupe de placement de proximité est référencé par :

- La première machine virtuelle Azure déployée dans le centre de données. La première machine virtuelle peut être considérée comme une « machine virtuelle d’étendue » qui est déployée dans un centre de données en fonction d’algorithmes d’allocation Azure qui peuvent être associés à des définitions d’utilisateurs pour une zone de disponibilité spécifique.
- Toutes les machines virtuelles suivantes déployées qui référencent le groupe de placement de proximité, afin de placer toutes les machines virtuelles Azure déployées ultérieurement dans le même centre de données que celui de la première machine virtuelle.

> [!NOTE]
> Si aucun matériel hôte pouvant exécuter un type de machine virtuelle spécifique dans le même centre de données que celui où la première machine virtuelle a été placée n’est déployé, le déploiement du type de machine virtuelle demandé échoue. Vous recevez alors un message d’échec.

Plusieurs groupes de placement de proximité peuvent être affectés à un seul et même [groupe de ressources Azure](../../../azure-resource-manager/management/manage-resources-portal.md). En revanche, un groupe de placement de proximité ne peut être affecté qu’à un seul groupe de ressources Azure.


## <a name="proximity-placement-groups-with-sap-systems-that-use-only-azure-vms"></a>Groupes de placement de proximité avec des systèmes SAP qui utilisent uniquement des machines virtuelles Azure
La plupart des déploiements de systèmes SAP NetWeaver et S/4HANA sur Azure n’utilisent pas de [grandes instances HANA](./hana-overview-architecture.md). Pour les déploiements qui n’utilisent pas de grandes instances HANA, il est important de fournir des performances optimales entre la couche Application SAP et la couche SGBD. Pour ce faire, définissez un groupe de placement de proximité Azure uniquement pour le système.

Dans la plupart des déploiements clients, les clients créent un seul [groupe de ressources Azure](../../../azure-resource-manager/management/manage-resources-portal.md) pour les systèmes SAP. Dans ce cas, il existe une relation un-à-un entre, par exemple, les groupes de ressources du système ERP de production et son groupe de placement de proximité. Dans d’autres cas, les clients organisent leurs groupes de ressources horizontalement et collectent tous les systèmes de production dans un seul groupe de ressources. Dans ce cas, vous avez une relation un-à-plusieurs entre votre groupe de ressources pour les systèmes SAP de production et plusieurs groupes de placement de proximité pour votre système ERP SAP de production, SAP BW, et ainsi de suite.

Évitez de regrouper plusieurs systèmes de non-production ou de production SAP dans un même groupe de placement de proximité. Quand un petit nombre de systèmes SAP ou un système SAP et certaines applications environnantes ont besoin d’une communication réseau à faible latence, vous pouvez envisager de déplacer ces systèmes vers un groupe de placement de proximité unique. Évitez les regroupements de systèmes, car plus vous regroupez de systèmes dans un groupe de placement de proximité, plus la probabilité est élevée :

- Que vous ayez besoin d’un type de machine virtuelle qui ne peut pas être exécuté dans le centre de données spécifique constituant l’étendue du groupe de placement de proximité.
- Que les ressources des machines virtuelles non courantes, telles que les machines virtuelles de la série M, soient finalement non satisfaites si vous avez besoin de davantage, car vous ajoutez des logiciels à un groupe de placement de proximité au fil du temps.

La configuration idéale, telle qu’elle est décrite ici, ressemble à ceci :

![Groupes de placement de proximité avec uniquement des machines virtuelles Azure](./media/sap-proximity-placement-scenarios/ppg-for-all-azure-vms.png)

Dans ce cas, les systèmes SAP uniques sont regroupés dans un groupe de ressources, chacun avec un groupe de placement de proximité. Il n’existe aucune dépendance quant à l’utilisation de configurations de scale-out HANA ou de scale-up SGBD.

## <a name="proximity-placement-groups-and-hana-large-instances"></a>Groupes de placement de proximité et grandes instances HANA
Si certains de vos systèmes SAP s’appuient sur de [grandes instances HANA](./hana-overview-architecture.md) pour la couche Application, vous pouvez bénéficier d’améliorations importantes de la latence réseau entre l’unité de grandes instances HANA et les machines virtuelles Azure quand vous utilisez des unités de grandes instances HANA déployées dans des [tampons ou lignes de la révision 4](./hana-network-architecture.md#networking-architecture-for-hana-large-instance). L’une des améliorations est que les unités de grandes instances HANA, une fois déployées, sont déployées avec un groupe de placement de proximité. Vous pouvez utiliser ce groupe de placements de proximité pour déployer vos machines virtuelles de couche Application. Ainsi, ces machines virtuelles seront déployées dans le même centre de données que celui qui héberge votre unité de grandes instances HANA.

Pour déterminer si votre unité de grandes instances HANA est déployée dans un tampon ou une ligne de la révision 4, consultez l’article [Contrôle des grandes instances Azure HANA à l’aide du portail Azure](./hana-li-portal.md#look-at-attributes-of-single-hli-unit). Dans la vue d’ensemble des attributs de votre unité de grandes instances HANA, vous pouvez également déterminer le nom du groupe de placement de proximité car il a été créé au moment du déploiement de votre unité de grandes instances HANA. Le nom affiché dans la vue d’ensemble des attributs est le nom du groupe de placement de proximité dans lequel vous devez déployer vos machines virtuelles de couche Application.

Par rapport aux systèmes SAP qui utilisent uniquement des machines virtuelles Azure, quand vous utilisez de grandes instances HANA, vous disposez de moins de souplesse quant au nombre de [groupes de ressources Azure](../../../azure-resource-manager/management/manage-resources-portal.md) à utiliser. Toutes les unités de grandes instances HANA d’un [locataire de grandes instances HANA](./hana-know-terms.md) sont regroupées dans un seul groupe de ressources, comme décrit dans [cet article](./hana-li-portal.md#display-of-hana-large-instance-units-in-the-azure-portal). À moins d’effectuer un déploiement dans plusieurs locataires, par exemple pour séparer les systèmes de production et de non-production ou d’autres systèmes, toutes vos unités de grandes instances HANA seront déployées dans un locataire de grandes instances HANA. Ce locataire a une relation un-à-un avec un groupe de ressources. Toutefois, un groupe de placement de proximité distinct sera défini pour chacune des unités.

Ainsi, les relations entre les groupes de ressources Azure et les groupes de placement de proximité pour un même locataire seront les suivantes :

![Groupes de placement de proximité et grandes instances HANA](./media/sap-proximity-placement-scenarios/ppg-for-hana-large-instance-units.png)

## <a name="example-of-deployment-with-proximity-placement-groups"></a>Exemple de déploiement avec des groupes de placement de proximité
Voici quelques commandes PowerShell que vous pouvez utiliser pour déployer vos machines virtuelles avec des groupes de placement de proximité Azure.

La première étape, une fois connecté à [Azure Cloud Shell](https://azure.microsoft.com/features/cloud-shell/), consiste à vérifier si vous êtes dans l’abonnement Azure que vous souhaitez utiliser pour le déploiement :

<pre><code>
Get-AzureRmContext
</code></pre>

Si vous devez changer d’abonnement, vous pouvez le faire en exécutant la commande suivante :

<pre><code>
Set-AzureRmContext -Subscription "my PPG test subscription"
</code></pre>

Créez un groupe de ressources Azure en exécutant la commande suivante :

<pre><code>
New-AzResourceGroup -Name "myfirstppgexercise" -Location "westus2"
</code></pre>

Créez le groupe de placement de proximité en exécutant la commande suivante :

<pre><code>
New-AzProximityPlacementGroup -ResourceGroupName "myfirstppgexercise" -Name "letsgetclose" -Location "westus2"
</code></pre>

Déployez votre première machine virtuelle dans le groupe de placement de proximité à l’aide d’une commande telle que celle-ci :

<pre><code>
New-AzVm -ResourceGroupName "myfirstppgexercise" -Name "myppganchorvm" -Location "westus2" -OpenPorts 80,3389 -ProximityPlacementGroup "letsgetclose" -Size "Standard_DS11_v2"
</code></pre>

La commande précédente déploie une machine virtuelle Windows. Une fois le déploiement de la machine virtuelle effectué, l’étendue du centre de données du groupe de placement de proximité est définie dans la région Azure. Tous les déploiements de machines virtuelles ultérieurs qui référencent le groupe de placement de proximité, comme indiqué dans la commande précédente, seront effectués dans le même centre de données Azure tant que le type de machine virtuelle peut être hébergé sur le matériel placé dans ce centre de données et que de la capacité est disponible pour ce type de machine virtuelle.

## <a name="combine-availability-sets-and-availability-zones-with-proximity-placement-groups"></a>Combiner des groupes à haute disponibilité et des zones de disponibilité avec des groupes de placement de proximité
L’un des inconvénients que présente l’utilisation de zones de disponibilité pour des déploiements de systèmes SAP est que vous ne pouvez pas déployer la couche Application SAP au moyen de groupes à haute disponibilité dans la zone spécifique. Vous souhaitez que la couche Application SAP soit déployée dans les mêmes zones que la couche SGBD. Le référencement d’une zone de disponibilité et d’un groupe à haute disponibilité lors du déploiement d’une machine virtuelle unique n’est pas pris en charge. Ainsi, avant vous deviez déployer votre couche Application en référençant une zone. Vous perdiez la possibilité de vous assurer que les machines virtuelles de la couche Application étaient réparties dans différents domaines de mise à jour et d’échec.

En utilisant des groupes de placement de proximité, vous pouvez contourner cette restriction. Voici la séquence de déploiement :

- Créez un groupe de placement de proximité.
- Déployez votre machine virtuelle d’ancrage, généralement le serveur SGBD, en référençant une zone de disponibilité.
- Créez un groupe à haute disponibilité qui référence le groupe de proximité Azure. (Voir la commande plus loin dans cet article.)
- Déployez les machines virtuelles de la couche Application en référençant le groupe à haute disponibilité et le groupe de placement de proximité.

Au lieu de déployer la première machine virtuelle comme illustré dans la section précédente, vous référencez une zone de disponibilité et le groupe de placement de proximité lors du déploiement de la machine virtuelle :

<pre><code>
New-AzVm -ResourceGroupName "myfirstppgexercise" -Name "myppganchorvm" -Location "westus2" -OpenPorts 80,3389 -Zone "1" -ProximityPlacementGroup "letsgetclose" -Size "Standard_E16_v3"
</code></pre>

Un déploiement réussi de cette machine virtuelle hébergerait l’instance de base de données du système SAP dans une zone de disponibilité. L’étendue du groupe de placement de proximité est fixée à l’un des centres de données représentant la zone de disponibilité que vous avez définie.

Supposez que vous déployez les machines virtuelles des services centraux de la même façon que les machines virtuelles SGBD, en référençant la ou les mêmes zones et les mêmes groupes de placement de proximité. À l’étape suivante, vous devez créer les groupes à haute disponibilité que vous voulez utiliser pour la couche Application de votre système SAP.

Définissez et créez le groupe de placement de proximité. La commande permettant de créer le groupe à haute disponibilité nécessite une référence supplémentaire à l’ID de groupe de placements de proximité (et non pas à son nom). Vous pouvez obtenir l’ID du groupe de placement de proximité à l’aide de cette commande :

<pre><code>
Get-AzProximityPlacementGroup -ResourceGroupName "myfirstppgexercise" -Name "letsgetclose"
</code></pre>

Quand vous créez le groupe à haute disponibilité, vous devez prendre en compte des paramètres supplémentaires lors de l’utilisation de disques managés (par défaut, sauf indication contraire) et des groupes de placement de proximité :

<pre><code>
New-AzAvailabilitySet -ResourceGroupName "myfirstppgexercise" -Name "myppgavset" -Location "westus2" -ProximityPlacementGroupId "/subscriptions/my very long ppg id string" -sku "aligned" -PlatformUpdateDomainCount 3 -PlatformFaultDomainCount 2 
</code></pre>

Dans l’idéal, vous devez utiliser trois domaines d’erreur. Toutefois, le nombre de domaines d’erreur pris en charge peut varier d’une région à l’autre. Dans ce cas, le nombre maximal de domaines d’erreur possibles pour les régions spécifiques est de deux. Pour déployer vos machines virtuelles de couche Application, vous devez ajouter une référence au nom de votre groupe à haute disponibilité et au nom du groupe de placement de proximité, comme indiqué ici :

<pre><code>
New-AzVm -ResourceGroupName "myfirstppgexercise" -Name "myppgavsetappvm" -Location "westus2" -OpenPorts 80,3389 -AvailabilitySetName "myppgavset" -ProximityPlacementGroup "letsgetclose" -Size "Standard_DS11_v2"
</code></pre>

Le résultat de ce déploiement est le suivant :
- Une couche SGBD et des services centraux pour votre système SAP situé dans une ou plusieurs zones de disponibilité spécifiques.
- Une couche Application SAP qui se trouve dans des groupes à haute disponibilité dans les mêmes centres de données Azure que les machines virtuelles SGBD.

> [!NOTE]
> Étant donné que vous déployez une machine virtuelle SGBD dans une zone et la seconde dans une autre zone pour créer une configuration de haute disponibilité, vous aurez besoin d’un groupe de placement de proximité différent pour chacune des zones. Il en va de même pour tout groupe à haute disponibilité que vous utilisez.

## <a name="move-an-existing-system-into-proximity-placement-groups"></a>Déplacer un système existant dans des groupes de placement de proximité
Si vous avez déjà des systèmes SAP déployés, vous souhaiterez peut-être optimiser la latence réseau de certains de vos systèmes critiques et localiser la couche Application et la couche SGBD dans le même centre de données. Pour déplacer les machines virtuelles d’un groupe à haute disponibilité Azure complet vers un groupe de placements de proximité existant dont l’étendue est déjà définie, vous devez arrêter toutes les machines virtuelles du groupe et affecter celui-ci au groupe de placements de proximité existant via le portail Azure, PowerShell ou CLI. Si vous souhaitez déplacer une machine virtuelle qui ne fait pas partie d’un groupe à haute disponibilité vers un groupe de placements de proximité existant, il vous suffit d’arrêter la machine virtuelle et de l’affecter au groupe de placements de proximité existant souhaité. 


## <a name="next-steps"></a>Étapes suivantes
Consultez la documentation :

- [Check-list relative à la planification et au déploiement de la charge de travail SAP sur Azure](./sap-deployment-checklist.md)
- [Préversion : Déployer des machines virtuelles dans des groupes de placements de proximité avec Azure CLI](../../linux/proximity-placement-groups.md)
- [Préversion : Déployer des machines virtuelles dans des groupes de placements de proximité avec PowerShell](../../windows/proximity-placement-groups.md)
- [Facteurs à prendre en compte pour le déploiement SGBD des machines virtuelles Azure pour la charge de travail SAP](./dbms_guide_general.md)
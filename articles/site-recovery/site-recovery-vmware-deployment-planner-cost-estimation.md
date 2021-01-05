---
title: Examiner les estimations de coût dans le planificateur de déploiement Azure Site Recovery
description: Cet article explique comment examiner les estimations de coût dans le planificateur de déploiement Azure Site Recovery pour la reprise d’activité après sinistre de VMware.
author: rajeswari-mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 7/29/2019
ms.author: ramamill
ms.openlocfilehash: dba585fbadca479c146ad42ac1bc5aa9d5349f49
ms.sourcegitcommit: 7a7b6c7ac0aa9dac678c3dfd4b5bcbc45dc030ca
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/02/2020
ms.locfileid: "93186703"
---
# <a name="review-cost-estimations-in-the-vmware-deployment-planner"></a>Examiner les estimations de coût dans le planificateur de déploiement VMware 

Le rapport du planificateur de déploiement fournit le résumé de l’estimation des coûts dans les feuilles [Recommandations](site-recovery-vmware-deployment-planner-analyze-report.md#recommendations) et l’analyse des coûts détaillée dans la feuille Estimation du coût. Une analyse des coûts détaillée par machine virtuelle est proposée. 

>[!Note]
>La version actuelle de l’outil Planificateur de déploiement est v2.5. Elle fournit des estimations de coûts pour les machines virtuelles dupliqués sur les disques managés.

### <a name="cost-estimation-summary"></a>Résumé de l’estimation des coûts 
Le graphique affiche la vue Résumé du coût estimé de la récupération d’urgence totale (DR) pour Azure de votre région cible choisie et avec la devise que vous avez spécifiée pour la génération du rapport.
Résumé de l’estimation des coûts

![Résumé de l’estimation des coûts](media/site-recovery-vmware-deployment-planner-analyze-report/cost-estimation-summary-v2a.png)

Le résumé vous aide à comprendre le coût que vous devez payer pour le stockage, le calcul, le réseau et la licence lorsque vous protégez toutes vos machines virtuelles compatibles avec Azure à l’aide de Azure Site Recovery. Le coût est calculé à partir de machines virtuelles compatibles et non sur toutes les machines virtuelles profilées.  
 
Vous pouvez afficher le coût mensuel ou annuel. En savoir plus sur les [régions cibles prises en charge](./site-recovery-vmware-deployment-planner-cost-estimation.md#supported-target-regions) et les [devises prises en charge](./site-recovery-vmware-deployment-planner-cost-estimation.md#supported-currencies).

**Coût par composant**  : le coût total de la reprise d’activité est divisé en quatre composants, à savoir le coût de la licence Azure Site Recovery, du stockage, du réseau et de la capacité de calcul. Le coût est calculé en fonction de la consommation facturée pendant la réplication et au moment de la récupération d’urgence pour le calcul, le stockage (premium et standard), le ExpressRoute/VPN configuré entre le site local et Azure, et la licence de Azure Site Recovery.

**Coût par état** Le coût total de la récupération d’urgence (DR) est catégorisé selon deux états différents, la réplication et l’extraction de la récupération d’urgence. 

**Coût de la réplication**  :  coût engendré par la réplication. Il couvre le coût du stockage, du réseau et de la licence d’Azure Site Recovery. 

**Coût de simulation de reprise d’activité**  : coût engendré par les basculements de test. Azure Site Recovery prépare des machines virtuelles pendant le basculement de test. Le coût d’extraction de la récupération d’urgence couvre les coûts de calcul et de stockage des machines virtuelles en cours d’exécution. 

**Coût de stockage Azure par mois/année** Il montre le coût de stockage total qui sera engagé pour le stockage standard et premium pour la réplication et l’extraction de récupération d’urgence.

## <a name="detailed-cost-analysis"></a>Analyse du coût détaillée
Les prix Azure de calcul, de stockage, de réseau, etc., varient entre les régions Azure. Vous pouvez générer un rapport d’estimation des coûts avec les prix Azure les plus récents selon votre abonnement, l’offre associée à votre abonnement et pour la région cible Azure dans la devise indiquée. Par défaut, l’outil utilise la région Azure USA Ouest 2 et la devise dollar américain (USD). Si vous avez utilisé une autre région et devise, la prochaine fois que vous générez un rapport sans ID d’abonnement, ID d’offre, région cible et devise, il utilisera les prix de la dernière région cible utilisée et la dernière devise utilisée pour l’estimation des coûts.
Cette section présente l’ID d’abonnement et l’ID d’offre que vous avez utilisés pour la génération de rapports.  Si aucun n’a été utilisé, il est vide.

Dans l’ensemble du rapport, les cellules marquées en gris sont en lecture seule. Les cellules en blanc peuvent être modifiées selon vos besoins.

![Estimation des coûts details1](media/site-recovery-hyper-v-deployment-planner-cost-estimation/cost-estimation1-h2a.png)

### <a name="overall-dr-cost-by-components"></a>Coût global de récupération d’urgence par composants
La première section indique le coût global de récupération d’urgence par composants et le coût de récupération d’urgence par états. 

**Calcul**  : coût des machines virtuelles IaaS exécutées sur Azure pour les besoins de la reprise d’activité. Il inclut les machines virtuelles créées par Azure Site Recovery pendant les extractions de la récupération d’urgence (basculements de test) et les machines virtuelles exécutées sur Azure telles que SQL Server avec groupes de disponibilité AlwaysOn et contrôleurs de domaine/serveurs de noms de domaine.

**Stockage** : coût de consommation du stockage Azure pour les besoins de la reprise d’activité. Il inclut la consommation du stockage pour la réplication et au cours des exercices de récupération d’urgence.
Réseau : coût d’ExpressRoute et du VPN site à site pour les besoins de la reprise d’activité. 

**Licence ASR**  : coût de la licence Azure Site Recovery pour toutes les machines virtuelles compatibles. Si vous avez saisi manuellement une machine virtuelle dans la table d’analyse des coûts détaillée, le coût de la licence Azure Site Recovery est également inclus pour cette machine virtuelle.

### <a name="overall-dr-cost-by-states"></a>Coût global de récupération d’urgence par états
Le coût total de la récupération d’urgence est catégorisé selon deux états différents, la réplication et l’extraction de la récupération d’urgence.

**Coût de la réplication**  : coût engagé au moment de la réplication. Il couvre le coût du stockage, du réseau et de la licence d’Azure Site Recovery. 

**Coût de simulation de reprise d’activité**  : coût engagé au moment des simulations de reprise d’activité. Azure Site Recovery prépare des machines virtuelles pendant les extractions de la récupération d’urgence. Le coût d’extraction de la récupération d’urgence couvre les coûts de calcul et de stockage des machines virtuelles en cours d’exécution.
Extraction de la récupération d’urgence totale par an = Nombre d’extractions de la récupération d’urgence x Durée de chaque extraction de la récupération d’urgence (jours) Coût moyen d’extraction de la récupération d’urgence - Coût total de l’extraction de la récupération d’urgence / 12

### <a name="storage-cost-table"></a>Table des coûts de stockage :
Ce tableau montre les coûts de stockage standard et premium liés à la réplication et aux extractions de la récupération d’urgence avec et sans remise.

### <a name="site-to-azure-network"></a>Du site vers le réseau Azure
Sélectionnez le paramètre approprié en fonction de vos besoins. 

**ExpressRoute**  : Par défaut, l’outil sélectionne le plan ExpressRoute le plus proche qui correspond à la bande passante réseau requise pour la réplication delta. Vous pouvez modifier le plan en fonction de vos besoins.

**Passerelle VPN**  : sélectionnez la passerelle VPN si vous en avez une dans votre environnement. Par défaut, N/A.

**Région cible**  : région Azure spécifiée pour la reprise d’activité. Le prix utilisé dans le rapport pour le calcul, le stockage, le réseau et la licence est basé sur la tarification Azure pour cette région. 

### <a name="vm-running-on-azure"></a>Machine virtuelle s’exécutant sur Azure
Si vous disposez d’un contrôleur de domaine ou d’une machine virtuelle DNS ou SQL Server avec groupes de disponibilité AlwaysOn s’exécutant sur Azure pour la récupération d’urgence, vous pouvez indiquer le nombre de machines virtuelles et la taille à prendre en compte de leur coût de calcul dans le coût total de la récupération d’urgence. 

### <a name="apply-overall-discount-if-applicable"></a>Appliquer la remise globale le cas échéant
Si vous êtes client ou partenaire Azure et si vous bénéficiez d’une remise sur la tarification Azure globale, vous pouvez utiliser ce champ. L’outil applique la remise (en %) sur tous les composants.

### <a name="number-of-virtual-machines-type-and-compute-cost-per-year"></a>Nombre de types de machines virtuelles et coût du calcul (par an)
Ce tableau montre le nombre de machines virtuelles Windows et non Windows et le coût du calcul de l’extraction de la récupération d’urgence associé.

### <a name="settings"></a>Paramètres 

**Devise**  : Devise dans laquelle le rapport est généré. Période de coût :  vous pouvez afficher tous les coûts pour le mois ou l’ensemble de l’année. 

## <a name="detailed-cost-analysis-table"></a>Tableau d’analyse du coût détaillée
![Analyse du coût détaillée](media/site-recovery-hyper-v-deployment-planner-cost-estimation/detailed-cost-analysis-h2a.png) Le tableau répertorie la répartition des coûts pour chaque machine virtuelle compatible. Vous pouvez également utiliser cette table pour obtenir le coût estimé de récupération d’urgence Azure des machines virtuelles non profilées en ajoutant manuellement des machines virtuelles. Elle est utile lorsque vous devez estimer les coûts Azure d’un nouveau déploiement de récupération d’urgence sans profilage détaillé.
Pour ajouter manuellement des machines virtuelles : 
1.  Cliquez sur le bouton « Ligne d’insertion » pour insérer une nouvelle ligne entre les lignes de début et de fin.

2.  Remplissez les colonnes suivantes en fonction de la taille approximative de la machine virtuelle et le nombre de machines virtuelles correspondant à cette configuration : 

* Nombre de machines virtuelles, taille IaaS (votre sélection)
* Type de stockage (Standard/Premium)
* Taille totale de stockage de la machine virtuelle (en Go) de la machine source
* Nombre d’extractions de la récupération d’urgence par an 
* Durée de chaque extraction de la récupération d’urgence (en jours) 
* Type de système d'exploitation
* Redondance des données 
* Azure Hybrid Benefit

1. Vous pouvez appliquer la même valeur à toutes les machines virtuelles du tableau en cliquant sur le bouton « Appliquer à tous » pour le nombre d’extraction de la récupération d’urgence par an, la durée de chaque extraction de la récupération d’urgence (en jours), la redondance des données et d’Azure Hybrid Use Benefit.

1. Cliquez sur « Recalculer le coût » pour mettre à jour le coût.

**Nom de la machine virtuelle**  : nom de la machine virtuelle.

**Nombre de machines virtuelles**  : nombre de machines virtuelles correspondant à la configuration. Vous pouvez mettre à jour le nombre de machines virtuelles existantes si des machines virtuelles de configuration similaire ne sont pas profilées mais sont protégées.

**Taille d’IaaS (recommandation)**  : taille du rôle de machine virtuelle de la machine virtuelle compatible recommandée par l’outil. 

**Taille d’IaaS (votre sélection)**  : par défaut, elle est identique à la taille de rôle de machine virtuelle recommandée. Vous pouvez changer le rôle selon vos besoins. Le coût du calcul est basé sur la taille du rôle de machine virtuelle sélectionnée.

**Type de stockage**  : type de stockage utilisé par la machine virtuelle. Il s’agit du stockage Standard ou Premium.

**Taille de stockage totale de la machine virtuelle (en Go)**  : Stockage total de la machine virtuelle source.

**Nombre de simulations de reprise d’activité en un an**  : nombre de fois où vous réalisez des simulations de reprise d’activité en une année. Par défaut, il est de 4 fois par an. Vous pouvez modifier la période pour des machines virtuelles spécifiques ou appliquer la nouvelle valeur à toutes les machines virtuelles en saisissant la nouvelle valeur sur la ligne du haut et en cliquant sur le bouton « Appliquer à tous ». Le coût total de l’extraction de la récupération d’urgence est calculé en fonction du nombre d’extractions de la récupération d’urgence et la période de cette dernière.  

**Durée de chaque simulation de reprise d’activité (en jours)**  : durée de chaque simulation de reprise d’activité. Par défaut, elle est de 7 jours tous les 90 jours selon l’[avantage Récupération d’urgence de la Software Assurance](https://azure.microsoft.com/pricing/details/site-recovery). Vous pouvez modifier la période pour des machines virtuelles spécifiques ou appliquer la nouvelle valeur à toutes les machines virtuelles en saisissant la nouvelle valeur sur la ligne du haut et en cliquant sur le bouton « Appliquer à tous ». Le coût total de l’extraction de la récupération d’urgence est calculé en fonction du nombre d’extractions de la récupération d’urgence et la période de cette dernière.
  
**Type de système d’exploitation**  : type de système d’exploitation de la machine virtuelle. Il peut s’agir de Windows ou de Linux. Si le type de système d’exploitation est Windows, Azure Hybrid Use Benefit peut être appliqué à cette machine virtuelle. 

**Redondance des données**  : vous pouvez sélectionner le stockage localement redondant (LRS), le stockage géoredondant (GRS) ou le stockage géoredondant avec accès en lecture (RA-GRS). La valeur par défaut est LRS. Vous pouvez modifier le type en fonction de votre compte de stockage pour les machines virtuelles spécifiques, ou appliquer le nouveau type à toutes les machines virtuelles en modifiant le type de la ligne du haut et en cliquant sur « Appliquer à tous ».  Le coût du stockage pour la réplication est calculé en fonction du prix de la redondance des données sélectionné. 

**Azure Hybrid Benefit**  : vous pouvez l’appliquer aux machines virtuelles Windows, le cas échéant.  La valeur par défaut est Oui. Vous pouvez modifier le paramètre pour les machines virtuelles spécifiques, ou mettre à jour toutes les machines virtuelles en cliquant sur le bouton « Appliquer à tous ».

**Consommation Azure totale**  : inclut le coût de la capacité de calcul, du stockage et de la licence Azure Site Recovery pour votre reprise d’activité. Selon votre sélection, cela indique le coût mensuel ou annuel.

**Coût de réplication d’état stable**  : inclut le coût de stockage pour la réplication.

**Coût total de la simulation de reprise d’activité (moyenne)**  : inclut le coût de la capacité de calcul et du stockage pour la simulation de reprise d’activité.

**Coût de licence ASR**  : coût de la licence Azure Site Recovery.

## <a name="supported-target-regions"></a>Régions cibles prises en charge
Le planificateur de déploiement Azure Site Recovery fournit une estimation de coût pour les régions Azure suivantes. Si votre région n’est pas répertoriée ci-dessous, vous pouvez utiliser une des régions suivantes dont la tarification se rapproche le plus de votre région.

eastus, eastus2, westus, centralus, northcentralus, southcentralus, northeurope, westeurope, eastasia, southeastasia, japaneast, japanwest, australiaeast, australiasoutheast, brazilsouth, southindia, centralindia, westindia, canadacentral, canadaeast, westus2, westcentralus, uksouth, ukwest, koreacentral, koreasouth 

## <a name="supported-currencies"></a>Devises prises en charge
Le Planificateur de déploiement Azure Site Recovery peut générer le rapport de coût avec une des devises suivantes.

|Devise|Nom|Devise|Nom|Devise|Nom|
|---|---|---|---|---|---|---|---|
|ARS|Peso argentin ($)|AUD|Dollar australien ($)|BRL|Real brésilien (R$)|
|CAD|Dollar canadien ($)|CHF|Franc suisse (chf)|DKK|Couronne danoise (kr)|
|EUR|Euro (€)|GBP|Livre britannique (£)|HKD|Dollar de Hong Kong (R.A.S.) (HK$)|
|IDR|Roupie indonésienne (Rp)|INR|Roupie indienne (₹)|JPY|Yen japonais (¥)|
|KRW|Won coréen (₩)|MXN|Peso mexicain (MXN$)|MYR|Ringgit malais (RM$)|
|NOK|Couronne norvégienne (kr)|NZD|Dollar néo-zélandais ($)|RUB|Rouble russe (руб)|
|SAR|Riyal saoudien (SR)|SEK|Couronne suédoise (kr)|TWD|Dollar taiwanais (NT$)|
|TRY|Lire turque (TL)|USD| Dollar américain ($)|ZAR|Rand sud-africain (R)|

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur la protection des [machines virtuelles de VMware vers Azure à l’aide d’Azure Site Recovery](./vmware-azure-tutorial.md).

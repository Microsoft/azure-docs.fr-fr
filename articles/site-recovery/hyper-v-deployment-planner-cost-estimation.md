---
title: Examiner le rapport d’estimation des coûts du Planificateur de déploiement Azure Site Recovery pour la récupération d’urgence de machines virtuelles Hyper-V sur Azure | Microsoft Docs
description: Cet article explique comment examiner le rapport d’estimation des coûts généré par le Planificateur de déploiement Azure Site Recovery pour la récupération d’urgence d’Hyper-V sur Azure.
services: site-recovery
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 4/9/2019
ms.author: mayg
ms.openlocfilehash: 29457f2f5021fed9d8785f5764c4119de4be1fa9
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95999210"
---
# <a name="cost-estimation-report-by-azure-site-recovery-deployment-planner"></a>Rapport d’estimation de coût du planificateur de déploiement Azure Site Recovery 

Le rapport du planificateur de déploiement Azure Site Recovery fournit le résumé de l’estimation des coûts dans les feuilles [Recommandations](hyper-v-deployment-planner-analyze-report.md#recommendations) et l’analyse détaillée des coûts dans la feuille Estimation du coût. Une analyse des coûts détaillée par machine virtuelle est proposée. 

### <a name="cost-estimation-summary"></a>Résumé de l’estimation des coûts 
Le graphique affiche la vue Résumé du coût estimé de la récupération d’urgence totale pour Azure dans la région cible que vous avez choisie et avec la devise que vous avez spécifiée pour la génération de rapport.

![Résumé de l’estimation des coûts](media/hyper-v-azure-deployment-planner-cost-estimation/cost-estimation-summary-h2a.png)

Le résumé vous aide à comprendre le coût que vous devez payer pour le stockage, le calcul, le réseau et la licence lorsque vous protégez vos machines virtuelles compatibles avec Azure Site Recovery. Le coût est calculé pour les machines virtuelles compatibles et non pour toutes les machines virtuelles profilées. 
 
Vous pouvez afficher le coût mensuel ou annuel. En savoir plus sur les [régions cibles prises en charge](./hyper-v-deployment-planner-cost-estimation.md#supported-target-regions) et les [devises prises en charge](./hyper-v-deployment-planner-cost-estimation.md#supported-currencies).

**Coût par composant** : le coût total de la reprise d’activité est divisé en quatre composants, à savoir le coût de la licence Site Recovery, du stockage, du réseau et de la capacité de calcul. Le coût est calculé sur la base de la consommation générée lors de la réplication et du test de récupération d’urgence. Le coût du calcul, du stockage (Premium et standard), du VPN/ExpressRoute configuré entre le site local et Azure, mais aussi de la licence Site Recovery est utilisé pour les calculs.

**Coût par état** : le coût total de la reprise d’activité est catégorisé selon deux états différents, à savoir la réplication et la simulation de reprise d’activité. 

**Coût de la réplication** : coût engendré pendant la réplication. Il couvre le coût du stockage, du réseau et de la licence Site Recovery. 

**Coût de simulation de reprise d’activité** : coût engendré par les basculements de test. Site Recovery prépare des machines virtuelles pendant le test de basculement. Le coût du test de récupération d’urgence couvre les coûts de calcul et de stockage des machines virtuelles en cours d’exécution. 

**Coût de stockage Azure par mois/année** : coût de stockage total engagé pour le stockage Standard et Premium, pour la réplication et la simulation de reprise d’activité.

## <a name="detailed-cost-analysis"></a>Analyse du coût détaillée
Les prix Azure pour le calcul, le stockage et le réseau varient selon les régions Azure. Vous pouvez générer un rapport d’estimation des coûts avec les prix Azure les plus récents selon votre abonnement, l’offre associée à votre abonnement et la région cible Azure dans la devise indiquée. Par défaut, l’outil utilise la région Azure USA Ouest 2 et la devise dollar américain (USD). Si vous avez utilisé une autre région et une autre devise, la prochaine fois que vous générerez un rapport sans ID d’abonnement, ID d’offre, région cible et devise, l’outil utilisera les prix de la dernière région cible utilisée et la dernière devise utilisée pour l’estimation des coûts.

Cette section montre l’ID d’abonnement et l’ID d’offre que vous avez utilisés pour la génération de rapports. Si vous n’en avez pas utilisé, elle est vide.

Dans l’ensemble du rapport, les cellules marquées en gris sont en lecture seule. Les cellules en blanc peuvent être modifiées selon vos besoins.

![Détail de l’estimation des coûts](media/hyper-v-azure-deployment-planner-cost-estimation/cost-estimation1-h2a.png)

### <a name="overall-dr-costs-by-components"></a>Coûts de récupération d’urgence globaux par composant
La première section indique le coût global de récupération d’urgence par composants et le coût de récupération d’urgence par états. 

**Calcul** : coût des machines virtuelles IaaS exécutées sur Azure pour les besoins de la reprise d’activité. Il inclut les machines virtuelles créées par Site Recovery au cours des tests de récupération d’urgence (tests de basculement). Il comprend aussi les machines virtuelles exécutées sur Azure, comme SQL Server avec les groupes de disponibilité Always On et les contrôleurs de domaine ou serveurs de noms de domaine.

**Stockage** : coût de consommation de stockage Azure pour les besoins de la reprise d’activité. Il inclut la consommation du stockage pour la réplication et au cours des exercices de récupération d’urgence.

**Réseau** : coût d’ExpressRoute et du VPN site à site pour les besoins de la reprise d’activité. 

**Licence Azure Site Recovery** : coût de la licence Site Recovery pour toutes les machines virtuelles compatibles. Si vous avez saisi manuellement une machine virtuelle dans la table d’analyse des coûts détaillée, le coût de la licence Site Recovery est également inclus pour cette machine virtuelle.

### <a name="overall-dr-costs-by-states"></a>Coûts de récupération d’urgence globaux par état
Le coût total de la récupération d’urgence est catégorisé selon deux états différents, la réplication et le test de récupération d’urgence.

**Réplication** : coût engagé au moment de la réplication. Il couvre le coût du stockage, du réseau et de la licence Site Recovery. 

**Simulation de reprise d’activité** : coût engagé au moment des simulations de reprise d’activité. Site Recovery prépare des machines virtuelles pendant les tests de récupération d’urgence. Le coût du test de récupération d’urgence couvre les coûts de calcul et de stockage des machines virtuelles en cours d’exécution.

* Durée totale des tests de récupération d’urgence sur un an = Nombre de tests de récupération d’urgence x Durée de chaque test de récupération d’urgence (en jours)
* Coût moyen des tests de récupération d’urgence (par mois) = Coût total des tests de récupération d’urgence / 12

### <a name="storage-cost-table"></a>Tableau des coûts de stockage
Ce tableau montre les coûts de stockage Standard et Premium liés à la réplication et aux tests de récupération d’urgence avec et sans remise.

### <a name="site-to-azure-network"></a>Du site vers le réseau Azure
Sélectionnez le paramètre approprié en fonction de vos besoins. 

**ExpressRoute** : Par défaut, l’outil sélectionne le plan ExpressRoute le plus proche qui correspond à la bande passante réseau requise pour la réplication delta. Vous pouvez modifier le plan en fonction de vos besoins.

**Type de passerelle VPN** : sélectionnez la passerelle VPN Azure si vous en avez une dans votre environnement. Par défaut, N/A.

**Région cible** : région Azure spécifiée pour la reprise d’activité. Le prix utilisé dans le rapport pour le calcul, le stockage, le réseau et la licence est basé sur la tarification Azure pour cette région. 

### <a name="vm-running-on-azure"></a>Machine virtuelle s’exécutant sur Azure
Vous disposez peut-être d’un contrôleur de domaine ou d’une machine virtuelle DNS ou SQL Server avec des groupes de disponibilité Always On qui s’exécutent sur Azure pour la récupération d’urgence. Vous pouvez indiquer le nombre de machines virtuelles et leur taille pour évaluer le coût de calcul par rapport au coût total de récupération d’urgence. 

### <a name="apply-overall-discount-if-applicable"></a>Appliquer la remise globale le cas échéant
Si vous êtes client ou partenaire Azure et si vous bénéficiez d’une remise sur la tarification Azure globale, vous pouvez utiliser ce champ. L’outil applique la remise (en pourcentage) sur tous les composants.

### <a name="number-of-virtual-machines-type-and-compute-cost-per-year"></a>Nombre de types de machines virtuelles et coût du calcul (par an)
Ce tableau montre le nombre de machines virtuelles Windows et non Windows et le coût du calcul pour le test de récupération d’urgence associé.

### <a name="settings"></a>Paramètres 
**Utilisation de disques managés** : ce paramètre indique si le disque managé est utilisé au moment des simulations de reprise d’activité. La valeur par défaut de ce paramètre est **Oui**. Si vous avez défini **-UseManagedDisks** sur **Non**, le prix du disque non managé est utilisé pour le calcul du coût.

**Devise** : Devise dans laquelle le rapport est généré.

**Période de coût** : vous pouvez afficher tous les coûts pour le mois ou l’ensemble de l’année. 

## <a name="detailed-cost-analysis-table"></a>Tableau d’analyse du coût détaillée
![Analyse du coût détaillée](media/hyper-v-azure-deployment-planner-cost-estimation/detailed-cost-analysis-h2a.png)

Le tableau répertorie la répartition des coûts pour chaque machine virtuelle compatible. Vous pouvez également utiliser ce tableau pour obtenir le coût estimé de la récupération d’urgence Azure pour les machines virtuelles non profilées en ajoutant manuellement des machines virtuelles. Cette solution est utile lorsque vous devez estimer les coûts Azure d’un nouveau déploiement de récupération d’urgence sans profilage détaillé.

Pour ajouter manuellement des machines virtuelles :

1. Sélectionnez **Insérer une ligne** pour insérer une nouvelle ligne entre les lignes **Début** et **Fin**.

1. Remplissez les colonnes suivantes en fonction de la taille approximative de la machine virtuelle et du nombre de machines virtuelles correspondant à cette configuration : 

    a. **Nombre de machines virtuelles**

    b. **Taille de l’IaaS (votre sélection)**

    c. **Type de stockage Standard/Premium**

    d. **Taille de stockage totale de la machine virtuelle (en Go)**

    e. **Nombre de tests de récupération d’urgence en un an**

    f. **Durée de chaque test de récupération d’urgence (en jours)**

    g. **OS Type**

    h. **Redondance des données**

    i. **Azure Hybrid Use Benefit**

1. Vous pouvez appliquer la même valeur à toutes les machines virtuelles du tableau en cliquant sur le bouton **Apply to all** (Appliquer à tous) pour le **nombre de tests de récupération d’urgence en un an**, la **durée de chaque test de récupération d’urgence (en jours)** , la **redondance des données** et **Azure Hybrid Use Benefit**.

1. Cliquez sur **Re-calculate cost** (Recalculer le coût) pour mettre à jour le coût.

**Nom de la machine virtuelle** : nom de la machine virtuelle.

**Nombre de machines virtuelles** : nombre de machines virtuelles correspondant à la configuration. Vous pouvez mettre à jour le nombre de machines virtuelles existantes si une configuration de machines virtuelles similaire n’est pas profilée mais protégée.

**Taille d’IaaS (recommandation)**  : taille du rôle de machine virtuelle de la machine virtuelle compatible recommandée par l’outil. 

**Taille d’IaaS (votre sélection)**  : par défaut, la taille est identique à la taille de rôle de machine virtuelle recommandée. Vous pouvez changer le rôle selon vos besoins. Le coût du calcul est basé sur la taille du rôle de machine virtuelle sélectionnée.

**Type de stockage** : type de stockage utilisé par la machine virtuelle. Il s’agit du stockage Standard ou Premium.

**Taille de stockage totale de la machine virtuelle (en Go)**  : stockage total de la machine virtuelle.

**Nombre de simulations de reprise d’activité en un an** : nombre de fois où vous réalisez des simulations de reprise d’activité en une année. Par défaut, ces tests sont effectués 4 fois par an. Vous pouvez modifier la période pour les machines virtuelles spécifiques ou appliquer la nouvelle valeur à toutes les machines virtuelles. Entrez la nouvelle valeur dans la ligne du haut, puis sélectionnez **Apply to all** (Appliquer à tous). Le coût total des tests de récupération d’urgence est calculé en fonction du nombre de tests de récupération d’urgence par an et de la durée de chaque test. 

**Durée de chaque simulation de reprise d’activité (en jours)**  : durée de chaque simulation de reprise d’activité. Par défaut, elle est de 7 jours tous les 90 jours selon l’[avantage Récupération d’urgence de la Software Assurance](https://azure.microsoft.com/pricing/details/site-recovery). Vous pouvez modifier la période pour certaines machines virtuelles spécifiques ou appliquer une nouvelle valeur à toutes les machines virtuelles. Entrez une nouvelle valeur dans la ligne du haut, puis sélectionnez **Apply to all** (Appliquer à tous). Le coût total des tests de récupération d’urgence est calculé en fonction du nombre de tests de récupération d’urgence par an et de la durée de chaque test.
 
**Type de système d’exploitation** : type de système d’exploitation exécuté sur la machine virtuelle. Il peut s’agir de Windows ou de Linux. Si le système d’exploitation est de type Windows, Azure Hybrid Use Benefit peut être appliqué à cette machine virtuelle. 

**Redondance des données** : stockage localement redondant, stockage géoredondant ou stockage géoredondant avec accès en lecture. Par défaut, le stockage localement redondant est sélectionné. Vous pouvez modifier le type en fonction de votre compte de stockage pour certaines machines virtuelles spécifiques, ou appliquer le nouveau type à toutes les machines virtuelles. Changez le type de la ligne du haut, puis sélectionnez **Apply to all** (Appliquer à tous). Le coût du stockage pour la réplication est calculé en fonction du prix de la redondance des données sélectionné. 

**Azure Hybrid Use Benefit** : vous pouvez l’appliquer aux machines virtuelles Windows, le cas échéant. La valeur par défaut de ce paramètre est **Oui**. Vous pouvez modifier le paramètre de certaines machines virtuelles spécifiques, ou vous pouvez mettre à jour toutes les machines virtuelles. Sélectionnez **Apply to all** (Appliquer à tous).

**Consommation Azure totale** : coût de la capacité de calcul, du stockage et de la licence Site Recovery pour votre reprise d’activité. Selon votre sélection, ce champ indique le coût mensuel ou annuel.

**Coût de réplication d’état stable** : coût de stockage pour la réplication.

**Coût total de la simulation de reprise d’activité (moyenne)**  : coût de la capacité de calcul et du stockage pour les simulations de reprise d’activité.

**Coût de la licence Azure Site Recovery** : coût de la licence Site Recovery.

## <a name="supported-target-regions"></a>Régions cibles prises en charge
Le planificateur de déploiement Site Recovery fournit une estimation de coût pour les régions Azure suivantes. Si votre région n’est pas répertoriée ici, vous pouvez utiliser la région, parmi les suivantes, dont la tarification se rapproche le plus de celle de votre région :

eastus, eastus2, westus, centralus, northcentralus, southcentralus, northeurope, westeurope, eastasia, southeastasia, japaneast, japanwest, australiaeast, australiasoutheast, brazilsouth, southindia, centralindia, westindia, canadacentral, canadaeast, westus2, westcentralus, uksouth, ukwest, koreacentral, koreasouth 

## <a name="supported-currencies"></a>Devises prises en charge
Le planificateur de déploiement Site Recovery peut générer le rapport de coût avec une des devises suivantes.

|Devise|Nom|Devise|Nom|Devise|Nom|
|---|---|---|---|---|---|---|---|
|ARS|Peso argentin ($)|AUD|Dollar australien ($)|BRL|Réal brésilien (R$)|
|CAD|Dollar canadien ($)|CHF|Franc suisse (CHF)|DKK|Couronne danoise (kr)|
|EUR|Euro (€)|GBP|Livre britannique (£)|HKD|Dollar de Hong Kong (R.A.S.) (HK$)|
|IDR|Roupie indonésienne (Rp)|INR|Roupie indienne (₹)|JPY|Yen japonais (¥)|
|KRW|Won coréen (₩)|MXN|Peso mexicain (MXN$)|MYR|Ringgit malais (RM$)|
|NOK|Couronne norvégienne (kr)||NZD|Dollar néo-zélandais ($)||RUB|Rouble russe (руб)|
|SAR|Riyal saoudien (SR)|SEK|Couronne suédoise (kr)|TWD|Dollar taiwanais (NT$)|
|TRY|Lire turque (TL)|USD| Dollar américain ($)|ZAR|Rand sud-africain (R)|

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur la protection des [machines virtuelles Hyper-V vers Azure à l’aide de Site Recovery](hyper-v-azure-tutorial.md).

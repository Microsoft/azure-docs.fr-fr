---
title: Analyser le rapport du planificateur de déploiement Hyper-V dans Azure Site Recovery
description: Cet article décrit comment analyser un rapport généré par le planificateur de déploiement Azure Site Recovery pour la reprise d’activité de machines virtuelles Hyper-V sur Azure.
services: site-recovery
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/21/2019
ms.author: mayg
ms.openlocfilehash: f230445ecdb046c2b631e89567df71e1d09c3234
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95999227"
---
# <a name="analyze-the-azure-site-recovery-deployment-planner-report"></a>Analyser le rapport du planificateur de déploiement Azure Site Recovery
Cet article décrit les feuilles de calcul du rapport Excel généré par le Planificateur de déploiement Azure Site Recovery pour le déploiement d’Hyper-V vers Azure.

## <a name="on-premises-summary"></a>Résumé local
La feuille de calcul du résumé local fournit une vue d’ensemble de l’environnement Hyper-V profilé.

![Résumé local](media/hyper-v-deployment-planner-analyze-report/on-premises-summary-h2a.png)

**Start Date** et **End Date** : dates de début et de fin du profilage des données prises en compte pour la génération de rapport. Par défaut, la date de début est la date à laquelle le profilage démarre, et la date de fin est la date à laquelle le profilage s’arrête. Il peut s’agir des valeurs StartDate et EndDate si le rapport est généré avec ces paramètres.

**Total number of profiling days** : le nombre total de jours de profilage compris entre les dates de début et de fin pendant lesquelles le rapport est généré.

**Number of compatible virtual machines** : nombre total de machines virtuelles compatibles pour lesquelles la bande passante réseau requise, ainsi que le nombre requis de comptes de stockage et de cœurs Azure sont calculés.

**Total number of disks across all compatible virtual machines** : le nombre total de disques sur toutes les machines virtuelles compatibles.

**Average number of disks per compatible virtual machine** : le nombre moyen de disques calculé sur toutes les machines virtuelles compatibles.

**Average disk size (GB)**  : la taille de disque moyenne calculée sur toutes les machines virtuelles compatibles.

**Desired RPO (minutes)** : objectif de point de récupération par défaut ou valeur transmise pour le paramètre DesiredRPO au moment de la génération de rapport afin d’estimer la bande passante requise.

**Desired bandwidth (Mbps)** : valeur que vous avez transmise pour le paramètre Bandwidth au moment de la génération de rapport afin d’estimer l’objectif de point de récupération (RPO) réalisable.

**Observed typical data churn per day (GB)**  : l’activité moyenne des données observée tous les jours de profilage.

## <a name="recommendations"></a>Recommandations 
La feuille Recommandations d’Hyper-V pour le rapport Azure présente les détails suivants selon le RPO souhaité que vous avez sélectionné :

![Recommandations d’Hyper-V pour le rapport Azure](media/hyper-v-deployment-planner-analyze-report/Recommendations-h2a.png)

### <a name="profile-data"></a>Données de profil
![Données de profil](media/hyper-v-deployment-planner-analyze-report/profile-data-h2a.png)

**Profiled data period**: la période pendant laquelle le profilage a été exécuté. Par défaut, l’outil inclut dans le calcul toutes les données profilées. Si vous avez utilisé les options StartDate et EndDate dans la génération de rapport, il génère le rapport correspondant à la période concernée. 

**Number of Hyper-V servers profiled** : nombre de serveurs Hyper-V pour lesquels un rapport de machines virtuelles est généré. Sélectionnez le nombre pour afficher le nom des serveurs Hyper-V. La feuille Exigences de stockage en local s’ouvre : elle présente tous les serveurs ainsi que leurs besoins en stockage. 

**Desired RPO** :l’objectif de point de récupération (RPO) de votre déploiement. Par défaut, la bande passante réseau requise est calculée pour les valeurs RPO de 15, 30 et 60 minutes. En fonction de la sélection, les valeurs concernées sont mises à jour sur la feuille. Si vous avez utilisé le paramètre DesiredRPOinMin lors de la génération de rapport, cette valeur est affichée dans le résultat Desired RPO.

### <a name="profiling-overview"></a>Vue d’ensemble du profilage
![Vue d’ensemble du profilage](media/hyper-v-deployment-planner-analyze-report/profiling-overview-h2a.png)

**Total Profiled Virtual Machines** : le nombre total de machines virtuelles dont les données profilées sont disponibles. Si le fichier VMListFile contient des noms de machines virtuelles qui n’ont pas été profilées, ces dernières ne sont pas prises en compte dans la génération de rapport et sont exclues du nombre total de machines virtuelles profilées.

**Compatible Virtual Machines** : le nombre de machines virtuelles pouvant être protégées dans Azure à l’aide d’Azure Site Recovery. Il s’agit du nombre total de machines virtuelles compatibles pour lesquelles la bande passante réseau requise, le nombre de comptes de stockage et le nombre de cœurs Azure sont calculés. Les détails de chaque machine virtuelle compatible sont disponibles dans la section « Machines virtuelles compatibles ».

**Incompatible Virtual Machines** : le nombre de machines virtuelles qui sont incompatibles pour la protection assurée avec Site Recovery. Les raisons de l’incompatibilité sont indiquées dans la section Machines virtuelles incompatibles. Si VMListFile contient des noms de machines virtuelles qui n’ont pas été profilées, ces machines virtuelles sont exclues du nombre de machines virtuelles incompatibles. Ces machines virtuelles sont répertoriées comme « Data not found » à la fin de la section Machines virtuelles incompatibles.

**Desired RPO** : votre objectif de point de récupération souhaité, en minutes. Le rapport est généré pour les trois valeurs RPO : 15 (valeur par défaut), 30 et 60 minutes. La recommandation de la bande passante dans le rapport est modifiée en fonction de votre sélection dans la liste déroulante **Desired RPO** dans l’angle supérieur droit de la feuille. Si vous avez généré le rapport à l’aide du paramètre -DesiredRPO avec une valeur personnalisée, cette valeur personnalisée s’affiche par défaut dans la liste déroulante **Desired RPO**.

### <a name="required-network-bandwidth-mbps"></a>Bande passante réseau requise (Mbits/s)
![Bande passante réseau requise](media/hyper-v-deployment-planner-analyze-report/required-network-bandwidth-h2a.png)

**To meet RPO 100% of the time :** bande passante recommandée en Mbits/s à allouer pour atteindre votre RPO souhaité 100 % du temps. Cette quantité de bande passante doit être dédiée à la réplication différentielle à l’état stable de toutes vos machines virtuelles compatibles afin d’éviter toute violation de RPO.

**To meet RPO 90% of the time** : en raison de la tarification haut débit ou pour toute autre raison, vous ne pouvez pas configurer la bande passante nécessaire pour atteindre votre RPO souhaité 100 % du temps. Dans ce cas, vous pouvez choisir de configurer une quantité de bande passante inférieure permettant d’atteindre le RPO souhaité 90 % du temps. Pour comprendre les implications de la configuration de cette bande passante réduite, le rapport fournit une analyse par simulation du nombre et de la durée de violations de RPO à attendre.

**Achieved Throughput :** débit du serveur sur lequel vous avez exécuté la commande GetThroughput pour la région Azure où se trouve le compte de stockage. Ce débit indique le niveau estimé que vous pouvez obtenir lorsque vous protégez les machines virtuelles compatibles à l’aide de Site Recovery. Les caractéristiques de réseau et de stockage du serveur Hyper-V doivent rester les mêmes que celles du serveur à partir duquel vous exécutez l’outil.

Pour tous les déploiements Site Recovery d’entreprise, nous vous recommandons d’utiliser [ExpressRoute](https://aka.ms/expressroute).

### <a name="required-storage-accounts"></a>Comptes de stockage requis
Le graphique suivant indique le nombre total de comptes de stockage (standard et premium) nécessaires pour protéger toutes les machines virtuelles compatibles. Pour savoir quel compte de stockage utiliser pour chaque machine virtuelle, consultez la section « VM-storage placement ».

![Comptes de stockage Azure requis](media/hyper-v-deployment-planner-analyze-report/required-storage-accounts-h2a.png)

### <a name="required-number-of-azure-cores"></a>Required number of Azure cores
Il s’agit du nombre total de cœurs à configurer avant le basculement ou le test de basculement de l’ensemble des machines virtuelles compatibles. Si un nombre insuffisant de cœurs est disponible dans l’abonnement, Site Recovery ne peut pas créer de machines virtuelles au moment du basculement ou du test de basculement.

![Required number of Azure cores](media/hyper-v-deployment-planner-analyze-report/required-number-of-azure-cores-h2a.png)


### <a name="additional-on-premises-storage-requirement"></a>Exigence de stockage local supplémentaire

Espace total de stockage disponible requis sur les serveurs Hyper-V pour la réussite de la réplication initiale et de la réplication delta afin de s’assurer que la réplication de machine virtuelle n’entraîne pas de temps d’arrêt indésirable pour vos applications de production. Des informations complémentaires sur les exigences relatives à chaque volume sont disponibles dans la section [Exigence de stockage local](#on-premises-storage-requirement). 

Si vous souhaitez comprendre pourquoi un espace libre est nécessaire pour la réplication, reportez-vous à la section[Exigence de stockage local](#why-do-i-need-free-space-on-the-hyper-v-server-for-the-replication).

![Exigence de stockage local et fréquence de copie](media/hyper-v-deployment-planner-analyze-report/on-premises-storage-and-copy-frequency-h2a.png)

### <a name="maximum-copy-frequency"></a>Fréquence de copie maximale
La fréquence de copie maximale recommandée doit être définie pour la réplication afin d’atteindre le RPO souhaité. Par défaut, la valeur est fixée à cinq minutes. Vous pouvez définir la fréquence de copie à 30 secondes pour obtenir un meilleur RPO.

### <a name="what-if-analysis"></a>Analyse de scénarios
![Analyse de scénarios](media/hyper-v-deployment-planner-analyze-report/what-if-analysis-h2a.png) Cette analyse indique le nombre de violations susceptibles de se produire pendant la période de profilage lorsque vous configurez une bande passante plus faible pour que le RPO souhaité soit atteint seulement 90 % du temps. Une ou plusieurs violations de RPO peuvent se produire sur un jour donné. Le graphique affiche le pic RPO de la journée. En vous appuyant sur cette analyse, vous pouvez décider si le nombre de violations de RPO tous les jours et d’accès RPO maximal par jour est acceptable avec la bande passante inférieure spécifiée. S’il est acceptable, vous pouvez allouer la bande passante inférieure pour la réplication. S’il est inacceptable, allouez une bande passante supérieure comme suggéré pour atteindre le RPO souhaité 100 % du temps. 

### <a name="recommendation-for-successful-initial-replication"></a>Recommandation pour la réussite de la réplication initiale
Cette section traite du nombre de lots dans lesquels les machines virtuelles doivent être protégées ainsi que la bande passante minimale nécessaire pour réussir la réplication initiale (IR). 

![Traitement par lot IR](media/hyper-v-deployment-planner-analyze-report/ir-batching-h2a.png)

Les machines virtuelles doivent être protégées dans l’ordre des lots donné. Chaque lot comporte une liste spécifique de machines virtuelles. Les machines virtuelles du lot 1 doivent être protégées avant celles du lot 2. Les machines virtuelles du lot 2 doivent être protégées avant celles du lot 3, et ainsi de suite. Une fois la réplication initiale des machines virtuelles du lot 1 terminée, vous pouvez activer la réplication pour les machines virtuelles du lot 2. De même, une fois la réplication initiale des machines virtuelles du lot 2 terminée, vous pouvez activer la réplication pour les machines virtuelles du lot 3, et ainsi de suite. 

Si l’ordre des lots n’est pas respecté, les machines virtuelles qui devront être protégées ultérieurement ne disposeront peut-être pas d’une bande passante suffisante pour effectuer la réplication initiale. Par conséquent, soit les machines virtuelles ne terminent jamais la réplication initiale, soit quelques machines virtuelles protégées peuvent entrer en mode de resynchronisation. Le traitement par lot IR pour la feuille de RPO sélectionnée contient des informations détaillées sur les machines virtuelles qui doivent être incluses dans chaque lot.

Le graphique présenté ici montre la répartition de la bande passante pour la réplication initiale et la réplication delta sur l’ensemble des lots selon l’ordre indiqué. Lorsque vous protégez le premier lot de machines virtuelles, toute la bande passante est disponible pour la réplication initiale. Une fois la réplication initiale terminée pour le premier lot, une partie de la bande passante est nécessaire pour la réplication delta. La bande passante restante est disponible pour la réplication initiale du deuxième lot de machines virtuelles. 

La barre du lot 2 montre la bande passante nécessaire pour la réplication delta des machines virtuelles du lot 1 et la bande passante disponible pour la réplication initiale des machines virtuelles du lot 2. De même, la barre du lot 3 montre la bande passante nécessaire pour la réplication delta des lots précédents (machines virtuelles du lot 1 et du lot 2) et la bande passante disponible pour la réplication initiale pour le lot 3, et ainsi de suite. Une fois la réplication initiale de tous les lots terminée, la dernière barre indique la bande passante nécessaire pour la réplication delta de toutes les machines virtuelles protégées.

**Pourquoi faut-il effectuer un traitement par lot pour la réplication initiale ?**
La durée d’exécution de la réplication initiale dépend de la taille de disque des machines virtuelles, de l’espace disque utilisé et du débit réseau disponible. Les détails sont disponibles dans le traitement par lot IR pour une feuille RPO sélectionnée.

### <a name="cost-estimation"></a>Estimation des coûts
Le graphique affiche la vue Résumé du coût estimé de la récupération d’urgence totale pour Azure dans la région cible que vous avez choisie et avec la devise que vous avez spécifiée pour la génération de rapport.

![Résumé de l’estimation des coûts](media/hyper-v-deployment-planner-analyze-report/cost-estimation-summary-h2a.png)

Le résumé vous aide à comprendre le coût que vous devez payer pour le stockage, le calcul, le réseau et la licence lorsque vous protégez toutes vos machines virtuelles compatibles avec Azure à l’aide de Site Recovery. Le coût est calculé pour les machines virtuelles compatibles et non pour toutes les machines virtuelles profilées. 
 
Vous pouvez afficher le coût mensuel ou annuel. En savoir plus sur les [régions cibles prises en charge](./hyper-v-deployment-planner-cost-estimation.md#supported-target-regions) et les [devises prises en charge](./hyper-v-deployment-planner-cost-estimation.md#supported-currencies).

**Coût par composant** : le coût total de la récupération d’urgence est divisé en quatre composants, à savoir, le coût de la licence Site Recovery, du stockage, du réseau et du calcul. Le coût est calculé sur la base de la consommation générée lors de la réplication et de l’extraction de la récupération d’urgence. Le coût du calcul, du stockage (Premium et standard), du VPN/ExpressRoute configuré entre le site local et Azure, mais aussi de la licence Site Recovery est utilisé pour les calculs.

**Coût par état** : le coût total de la récupération d’urgence est catégorisé selon deux états différents (la réplication et l’extraction de la récupération d’urgence). 

**Coût de la réplication** : le coût engendré pendant la réplication. Il couvre le coût du stockage, du réseau et de la licence Site Recovery. 

**Coût d’extraction de la récupération d’urgence** : coût engendré par les tests de basculement. Site Recovery prépare des machines virtuelles pendant le test de basculement. Le coût d’extraction de la récupération d’urgence couvre les coûts de calcul et de stockage des machines virtuelles en cours d’exécution. 

**Coût de stockage Azure par mois/année** : le graphique à barres montre le coût de stockage total engagé dans le stockage Standard et Premium pour la réplication et l’extraction de la récupération d’urgence. Vous pouvez afficher une analyse détaillée des coûts par machine virtuelle dans la feuille [Estimation des coûts](hyper-v-deployment-planner-cost-estimation.md).

### <a name="growth-factor-and-percentile-values-used"></a>Growth factor and percentile values used
Cette section figurant en bas de la feuille indique la valeur de centile utilisée pour tous les compteurs de performances des machines virtuelles profilées (par défaut : 95e centile). Il montre également le facteur de croissance (par défaut : 30 %) qui est utilisé dans tous les calculs.

![Growth factor and percentile values used](media/hyper-v-deployment-planner-run/growth-factor-max-percentile-value.png)

## <a name="recommendations-with-available-bandwidth-as-input"></a>Recommandations relatives à la bande passante disponible comme entrée
![Vue d’ensemble du profilage avec entrée de la bande passante](media/hyper-v-deployment-planner-analyze-report/profiling-overview-bandwidth-input-h2a.png)

Vous pouvez vous trouver dans une situation dans laquelle vous ne pouvez pas configurer une bande passante supérieure à x Mbits/s pour la réplication Site Recovery. Cet outil vous permet d’entrer la bande passante disponible (à l’aide du paramètre -Bandwidth lors de la génération de rapport) et d’obtenir le RPO réalisable en quelques minutes. Avec cette valeur de RPO réalisable, vous pouvez choisir d’approvisionner une bande passante supplémentaire ou vous contenter d’une solution de récupération d’urgence avec ce RPO.

![RPO réalisable](media/hyper-v-deployment-planner-analyze-report/achivable-rpo-h2a.PNG)

## <a name="vm-storage-placement-recommendation"></a>Recommandation relative au placement de stockage des machines virtuelles 
![VM-storage placement](media/hyper-v-deployment-planner-analyze-report/vm-storage-placement-h2a.png)

**Disk Storage Type** : le compte de stockage standard ou premium utilisé pour répliquer toutes les machines virtuelles correspondantes, mentionnées dans la colonne **VMs to Place**.

**Suggested Prefix** : le préfixe suggéré à trois caractères qui permet de nommer le compte de stockage. Vous pouvez utiliser votre propre préfixe, mais la suggestion de l’outil suit la [convention d’affectation de noms aux partitions pour les comptes de stockage](/en-in/azure/storage/blobs/storage-performance-checklist).

**Suggested Account Name** : le nom du compte de stockage après lequel vous incluez le préfixe suggéré. Remplacez le nom entre crochets pointus(< and >) avec votre entrée personnalisée.

**Log Storage Account :** tous les journaux d’activité de réplication sont stockés dans un compte de stockage standard. Pour les machines virtuelles qui répliquent vers un compte de stockage premium, configurez un compte de stockage standard supplémentaire pour le stockage des journaux. Un seul et même compte de stockage standard des journaux peut être utilisé par plusieurs comptes de stockage de réplication premium. Les machines virtuelles qui sont répliquées vers les comptes de stockage standard utilisent le même compte de stockage pour les journaux d’activité.

**Suggested Log Account Name** : le nom de votre compte de stockage des journaux après lequel vous incluez le préfixe suggéré. Remplacez le nom entre crochets pointus(< and >) avec votre entrée personnalisée.

**Placement Summary** : un résumé de la charge totale des machines virtuelles sur le compte de stockage au moment de la réplication et du test de basculement/basculement. Le résumé inclut les données suivantes :

* Nombre total de machines virtuelles mappées au compte de stockage. 
* Total des IOPS en lecture/écriture sur toutes les machines virtuelles placées dans ce compte de stockage.
* Total des IOPS (réplication) en écriture.
* Taille totale d’installation sur tous les disques.
* Nombre total de disques.

**VMs to Place** : liste de toutes les machines virtuelles qui doivent être placées dans le compte de stockage donné pour optimiser les performances et l’utilisation.

## <a name="compatible-vms"></a>Machines virtuelles compatibles
Le rapport Excel généré par le Planificateur de déploiement Azure Site Recovery fournit tous les détails relatifs aux machines virtuelles compatibles dans la feuille « Machines virtuelles compatibles ».

![Machines virtuelles compatibles](media/hyper-v-deployment-planner-analyze-report/compatible-vms-h2a.png)

**VM Name** : nom de la machine virtuelle utilisé dans VMListFile lorsqu’un rapport est généré. Cette colonne répertorie également les disques (VHD) qui sont attachés aux machines virtuelles. Les noms incluent les noms d’hôte Hyper-V sur lesquels les machines virtuelles ont été placées lorsqu’elles ont été découvertes par l’outil pendant le profilage.

**VM Compatibility** : les valeurs sont **Oui** et **Oui**\*. **Oui**\* : pour les instances dans lesquelles la machine virtuelle est adaptée aux [disques SSD Premium](../virtual-machines/disks-types.md). Ici, le disque profilé à forte activité ou à IOPS élevé s’ajuste dans une taille de disque premium supérieure à la taille mappée au disque. Le compte de stockage décide du type de disque de stockage Premium sur lequel mapper un disque, en fonction de sa taille : 
* < 128 Go : disque P10.
* 128 Go à 256 Go : disque P15.
* 256 Go à 512 Go : disque P20.
* 512 Go à 1 024 Go : disque P30.
* 1 025 Go à 2 048 Go : disque P40.
* 2 049 Go à 4 095 Go : disque P50.

Par exemple, si les caractéristiques de charge de travail d’un disque le placent dans la catégorie P20 ou P30, mais que la taille le mappe à un niveau ou à un type de disque de stockage premium inférieur, l’outil marque cette machine virtuelle comme **Oui**\*. L’outil recommande également que vous modifiiez la taille du disque source pour que celui s’adapte au type de disque de stockage premium recommandé, ou que vous modifiiez le type de disque cible après le basculement.

**Storage Type** : standard ou premium.

**Suggested Prefix** : le préfixe de compte de stockage à trois caractères.

**Storage Account** : le nom utilisé par le préfixe du compte de stockage suggéré.

**Peak R/W IOPS (with Growth Factor)** : pic d’IOPS en lecture/écriture de la charge de travail sur le disque (95e centile par défaut), y compris le facteur de croissance futur (30 % par défaut). Le total d’IOPS en lecture/écriture d’une machine virtuelle n’est pas toujours la somme des IOPS en lecture/écriture des disques individuels de la machine virtuelle. Le pic d’IOPS en lecture/écriture de la machine virtuelle désigne le pic de la somme des IOPS en lecture/écriture de ses différents disques à chaque minute de la période de profilage.

**Peak Data Churn in MB/s (with Growth Factor)** : pic du taux d’activité sur le disque (95e centile par défaut), ainsi que le facteur de croissance futur (30 % par défaut). L’activité totale des données de la machine virtuelle ne correspond pas toujours à la somme de l’activité des données des disques individuels de la machine virtuelle. Le pic d’activité des données de la machine virtuelle désigne le pic de la somme de l’activité de ses différents disques à chaque minute de la période de profilage.

**Azure VM Size** : taille de machine virtuelle Azure Cloud Services mappée idéale pour cette machine virtuelle locale. Le mappage est basé sur la mémoire de la machine virtuelle locale, le nombre de disques/cœurs/cartes réseau et les IOPS en lecture/écriture. La recommandation est toujours la plus petite taille de machine virtuelle Azure qui correspond à toutes les caractéristiques des machines virtuelles locales.

**Number of Disks** : le nombre total de disques de machines virtuelles (VHD) sur la machine virtuelle.

**Disk size (GB)** : taille totale de tous les disques de la machine virtuelle. L’outil affiche également la taille des disques individuels de la machine virtuelle.

**Cores** : le nombre de cœurs de processeur de la machine virtuelle.

**Memory (MB)**  : la mémoire RAM de la machine virtuelle.

**NICs** : le nombre de cartes réseau de la machine virtuelle.

**Boot Type** : type de démarrage de la machine virtuelle. Le type de démarrage peut prendre la valeur BIOS ou EFI.

## <a name="incompatible-vms"></a>Machines virtuelles incompatibles
Le rapport Excel généré par le Planificateur de déploiement Azure Site Recovery fournit tous les détails relatifs aux machines virtuelles incompatibles dans la feuille « Machines virtuelles incompatibles ».

![Machines virtuelles incompatibles](media/hyper-v-deployment-planner-analyze-report/incompatible-vms-h2a.png)

**VM Name** : nom de la machine virtuelle utilisé dans VMListFile lorsqu’un rapport est généré. Cette colonne répertorie également les disques (VHD) qui sont attachés aux machines virtuelles. Les noms incluent les noms d’hôte Hyper-V sur lesquels les machines virtuelles ont été placées lorsqu’elles ont été découvertes par l’outil pendant le profilage.

**VM Compatibility** : indique pourquoi la machine virtuelle spécifiée est incompatible avec une utilisation avec Site Recovery. Les raisons sont décrites pour chaque disque incompatible de la machine virtuelle et, en fonction des [limites de stockage](/en-in/azure/storage/common/scalability-targets-standard-account), peuvent figurer parmi les suivantes :

* La taille du disque est supérieure à 4 095 Go. Actuellement, le stockage Azure ne prend pas en charge les tailles de disques de données supérieures à 4 095 Go.

* Le disque de système d’exploitation est supérieur à 2 047 Go pour les machines virtuelles de génération 1 (type de démarrage BIOS). Site Recovery ne prend pas en charge une taille de disque du système d’exploitation supérieure à 2 047 Go pour les machines virtuelles de génération 1.

* Le disque de système d’exploitation est supérieur à 300 Go pour les machines virtuelles de génération 2 (type de démarrage EFI). Site Recovery ne prend pas en charge une taille de disque du système d’exploitation supérieure à 300 Go pour les machines virtuelles de génération 2.

* Les noms de machine virtuelle contenant l’un des caractères suivants “” [] ` ne sont pas pris en charge. L’outil ne peut pas obtenir de données profilées de machines virtuelles dont le nom contient l’un de ces caractères. 

* Un disque dur virtuel est partagé par au moins deux machines virtuelles. Azure ne prend pas en charge les machines virtuelles contenant un disque dur virtuel partagé.

* Les machines virtuelles avec Virtual Fibre Channel ne sont pas prises en charge. Site Recovery ne prend pas en charge les machines virtuelles avec Virtual Fibre Channel.

* Le cluster Hyper-V ne contient pas de répartiteur de réplication. Site Recovery ne prend en charge les machines virtuelles dans un cluster Hyper-V si le répartiteur de réplication Hyper-V n’est pas configuré pour le cluster.

* La machine virtuelle n’est pas hautement disponible. Site Recovery ne prend pas en charge les machines virtuelles d’un nœud de cluster Hyper-V dont les disques durs virtuels sont stockés sur le disque local et non sur le disque du cluster. 

* La taille totale de machine virtuelle (réplication + test de basculement) dépasse la limite de taille du compte de stockage Premium prise en charge (35 To). Cette incompatibilité se produit généralement lorsqu’un seul disque de la machine virtuelle présente une caractéristique de performances dépassant les limites maximales prises en charge Azure ou Site Recovery pour le stockage standard. Une telle instance envoie la machine virtuelle dans la zone de stockage premium en mode Push. Toutefois, la taille maximale prise en charge pour un compte de stockage Premium est de 35 To. Une seule machine virtuelle protégée ne peut pas être protégée sur plusieurs comptes de stockage. 

    Lorsqu’un test de basculement est exécuté sur une machine virtuelle protégée, si un disque non managé est configuré pour le test de basculement, il s’exécute dans le compte de stockage où la réplication est en cours. Dans ce cas, une quantité d’espace de stockage supplémentaire identique à celle de la réplication est nécessaire. Cela permet à la réplication de progresser tout en permettant la bonne exécution du basculement de test en parallèle. Lorsqu’un disque managé est configuré pour le test de basculement, aucun espace supplémentaire ne doit être pris en compte pour la machine virtuelle du test de basculement.

* Les IOPS sources dépassent la limite des IOPS prises en charge par le stockage qui s’élève à 7 500 par disque.

* Les IOPS sources dépassent la limite des IOPS prises en charge par le stockage qui s’élève à 80 000 par disque.

* L’activité moyenne des données de la machine virtuelle source dépasse la limite d’activité des données prise en charge par Site Recovery, qui est de 20 Mo/s pour la taille d’E/S moyenne.

* Les IOPS d’écriture moyennes effectives de la machine virtuelle source dépassent la limite d’IOPS prise en charge par Site Recovery, qui est de 840.

* Le stockage calculé des captures instantanées dépasse la limite de stockage des captures instantanées prise en charge, qui est de 10 To.

**Peak R/W IOPS (with Growth Factor)** : pic des IOPS de la charge de travail sur le disque (95e centile par défaut), y compris le facteur de croissance futur (30 % par défaut). Le total d’IOPS en lecture/écriture d’une machine virtuelle n’est pas toujours la somme des IOPS en lecture/écriture des disques individuels de la machine virtuelle. Le pic d’IOPS en lecture/écriture de la machine virtuelle désigne le pic de la somme des IOPS en lecture/écriture de ses différents disques à chaque minute de la période de profilage.

**Peak Data Churn MB/s (with Growth Factor)** : pic du taux d’activité sur le disque (95e centile par défaut), ainsi que le facteur de croissance futur (30 % par défaut). L’activité totale des données de la machine virtuelle ne correspond pas toujours à la somme de l’activité des données des disques individuels de la machine virtuelle. Le pic d’activité des données de la machine virtuelle désigne le pic de la somme de l’activité de ses différents disques à chaque minute de la période de profilage.

**Number of Disks** : le nombre total de disques de machines virtuelles (VHD) sur la machine virtuelle.

**Disk Size (GB)** : taille totale d’installation de tous les disques de la machine virtuelle. L’outil affiche également la taille des disques individuels de la machine virtuelle.

**Cores** : le nombre de cœurs de processeur de la machine virtuelle.

**Memory (MB)**  : la quantité de RAM sur la machine virtuelle.

**NICs** : le nombre de cartes réseau de la machine virtuelle.

**Boot Type** : type de démarrage de la machine virtuelle. Le type de démarrage peut prendre la valeur BIOS ou EFI.

## <a name="azure-site-recovery-limits"></a>Limites Azure Site Recovery
Le tableau suivant présente les limites de Site Recovery. Ces limites sont basées sur des tests, mais elles ne peuvent pas couvrir toutes les combinaisons d’E/S d’application possibles. Les résultats réels varient en fonction de la combinaison d’E/S de votre application. Pour de meilleurs résultats, même après la planification du déploiement, effectuez des tests d’application approfondis à l’aide d’un test de basculement pour obtenir une image réelle des performances de l’application.
 
**Stockage de réplication cible** | **Taille des E/S moyenne de la machine virtuelle source** |**Activité moyenne des données de la machine virtuelle source** | **Total de l’activité des données de la machine virtuelle source par jour**
---|---|---|---
Stockage Standard | 8 Ko | 2 Mo/s par machine virtuelle | 168 Go par machine virtuelle
Stockage Premium | 8 Ko  | 5 Mo/s par machine virtuelle | 421 Go par machine virtuelle
Stockage Premium | 16 Ko ou plus| 20 Mo/s par machine virtuelle | 1 684 Go par machine virtuelle

Ces limites représentent des moyennes en partant sur un chevauchement d’E/S de 30 pour cent. Site Recovery est capable de gérer un débit plus élevé en fonction du ratio de chevauchement, de tailles d’écriture plus grandes et du comportement d’E/S des charges de travail réelles. Les valeurs précédentes supposent un retard de traitement typique de cinq minutes. Autrement dit, une fois que les données sont chargées, elles sont traitées, et un point de récupération est créé dans un délai de cinq minutes.

## <a name="on-premises-storage-requirement"></a>Exigence de stockage local

La feuille de calcul indique l’espace de stockage total disponible pour chaque volume des serveurs Hyper-V (où résident les disques durs virtuels) nécessaire pour que réussissent la réplication initiale et la réplication delta. Avant d’activer la réplication, ajoutez l’espace de stockage requis sur les volumes pour vous assurer que la réplication n’entraîne aucun temps d’arrêt indésirable pour vos applications de production. 

  Le Planificateur de déploiement Site Recovery identifie l’espace de stockage optimal requis en fonction de la taille des disques durs virtuels et de la bande passante réseau utilisée pour la réplication.

![Exigence de stockage local](media/hyper-v-deployment-planner-analyze-report/on-premises-storage-requirement-h2a.png)

### <a name="why-do-i-need-free-space-on-the-hyper-v-server-for-the-replication"></a>Pourquoi ai-je besoin d’espace libre sur le serveur Hyper-V pour la réplication ?
* Lorsque vous activez la réplication d’une machine virtuelle, Site Recovery prend un instantané de chaque disque dur virtuel de la machine virtuelle pour la réplication initiale. Pendant la réplication initiale, l’application écrit sur les disques les nouvelles modifications. Site Recovery assure le suivi de ces modifications delta dans les fichiers journaux, qui nécessitent un espace de stockage supplémentaire. Tant que la réplication initiale n’est pas terminée, les fichiers journaux sont stockés en local. 

    Faute d’espace disponible suffisant pour le stockage des fichiers journaux et de l’instantané (AVHDX), la réplication passe en mode de resynchronisation et ne pourra jamais aboutir. Dans le pire des cas, vous aurez besoin d’un espace disque supplémentaire correspondant à 100 % de la taille du disque dur virtuel pour la réplication initiale.
* Une fois la réplication initiale terminée, la réplication delta démarre. Site Recovery assure le suivi de ces modifications delta dans les fichiers journaux, qui sont stockés sur le volume où se trouvent les disques durs virtuels de la machine virtuelle. Ces fichiers journaux sont répliqués vers Azure à une fréquence de copie configurée. Selon la bande passante réseau disponible, la réplication des fichiers journaux dans Azure peut prendre un certain temps. 

    Si l’espace disponible n’est pas suffisant pour stocker les fichiers journaux, la réplication est suspendue. L’état de réplication de la machine virtuelle passe ensuite à « Resynchronisation nécessaire ».
* Si la bande passante réseau n’est pas suffisante pour transmettre les fichiers journaux dans Azure, les fichiers journaux s’accumulent sur le volume. Dans le pire des cas, lorsque la taille des fichiers journaux atteint 50 % de la taille du disque dur virtuel, la réplication de la machine virtuelle bascule en mode « Resynchronisation nécessaire ». Dans le pire des cas, vous aurez besoin d’un espace disponible supplémentaire correspondant à 50 % de la taille du disque dur virtuel pour la réplication delta.

**Hyper-V host** : la liste des serveurs Hyper-V profilés. Si un serveur fait partie d’un cluster Hyper-V, tous les nœuds de cluster sont regroupés ensemble.

**Volume (VHD path)** : chaque volume d’un hôte Hyper-V contenant des VHD/VHDX. 

**Free space available (GB)** : l’espace libre disponible sur le volume.

**Total storage space required on the volume (GB)** : espace de stockage disponible total requis sur le volume pour que réussissent la réplication initiale et la réplication delta. 

**Total additional storage to be provisioned on the volume for successful replication (GB)** : espace total supplémentaire recommandé qui doit être configuré sur le volume pour que réussissent la réplication initiale et la réplication delta.

## <a name="initial-replication-batching"></a>Traitement par lot de la réplication initiale 

### <a name="why-do-i-need-initial-replication-batching"></a>Pourquoi faut-il effectuer un traitement par lot pour la réplication initiale ?
Si toutes les machines virtuelles sont protégées en même temps, l’exigence en matière d’espace de stockage disponible est beaucoup plus élevée. Si l’espace de stockage disponible n’est pas suffisant, la réplication des machines virtuelles passe en mode de resynchronisation. En outre, les besoins en bande passante réseau sont bien plus importants pour que la réplication initiale combinée de toutes les machines virtuelles aboutisse. 

### <a name="initial-replication-batching-for-a-selected-rpo"></a>Traitement par lot d’une réplication initiale pour un RPO sélectionné
Cette feuille de calcul fournit une vue détaillée de chaque lot pour la réplication initiale. Pour chaque RPO, une feuille de traitement par lot IR distincte est créée. 

Dès lors que vous respectez l’exigence de stockage local recommandée pour chaque volume, vous n’avez besoin que de la liste des machines virtuelles pouvant être protégées en parallèle pour effectuer la réplication. Ces machines virtuelles sont regroupées dans un lot, et il peut y avoir plusieurs lots. Protégez les machines virtuelles dans l’ordre des lots indiqué. Protégez tout d’abord les machines virtuelles du lot 1. Une fois que la réplication initiale est terminée, protégez les machines virtuelles du lot 2, et ainsi de suite. Vous pouvez obtenir la liste des lots et des machines virtuelles correspondantes à partir de cette feuille. 

![Détails du traitement par lot de la réplication initiale](media/hyper-v-deployment-planner-analyze-report/ir-batching-for-rpo-h2a.png)

![Détails complémentaires du traitement par lot de la réplication initiale](media/hyper-v-deployment-planner-analyze-report/ir-batching-for-rpo2-h2a.png)

### <a name="each-batch-provides-the-following-information"></a>Chaque lot fournit les informations suivantes : 
**Hyper-V host**: l’hôte Hyper-V de la machine virtuelle à protéger.

**Virtual Machine** : machine virtuelle à protéger. 

**Comments** : commentaire indiqué ici si une action est requise pour un volume spécifique d’une machine virtuelle. Par exemple, si l’espace disponible n’est pas suffisant sur un volume, un commentaire du type « Ajouter un stockage supplémentaire pour protéger cette machine virtuelle » est ajouté.

**Volume (VHD path)** : nom du volume sur lequel résident les disques durs virtuels de la machine virtuelle. 

**Free space available on the volume (GB)** : espace disque libre disponible sur le volume de la machine virtuelle. Lors du calcul de l’espace libre disponible sur les volumes, il tient compte de l’espace disque utilisé pour la réplication delta par les machines virtuelles des lots précédents dont les disques durs virtuels se trouvent sur le même volume. 

Par exemple, VM1, VM2 et VM3 résident sur un volume E:\VHDpath. Avant la réplication, l’espace libre sur le volume est de 500 Go. VM1 fait partie du lot 1, VM2 fait partie du lot 2 et VM3 fait partie du lot 3. Pour la VM1, l’espace libre disponible est de 500 Go. Pour VM2, l’espace disponible est de 500 Go, soit l’espace disque requis pour la réplication delta de VM1. Si VM1 nécessite 300 Go d’espace pour la réplication delta, l’espace disponible pour VM2 est de 500 Go - 300 Go = 200 Go. De la même manière, la VM2 a besoin de 300 Go pour la réplication delta. L’espace disponible pour VM3 est donc de 200 - 300 Go =-100 Go.

**Storage required on the volume for initial replication (GB)** : l’espace de stockage libre requis sur le volume pour la réplication initiale de la machine virtuelle.

**Storage required on the volume for delta replication (GB)** : espace de stockage libre requis sur le volume pour la réplication delta de la machine virtuelle.

**Additional storage required based on deficit to avoid replication failure (GB)** : l’espace de stockage supplémentaire requis sur le volume pour la machine virtuelle. Il s’agit de l’espace maximal de stockage supplémentaire requis pour la réplication initiale et la réplication delta, moins l’espace disponible sur le volume.

**Minimum bandwidth required for initial replication (Mbps)** : la bande passante minimale requise pour la réplication initiale de la machine virtuelle.

**Minimum bandwidth required for delta replication (Mbps)** : bande passante minimale requise pour la réplication delta de la machine virtuelle.

### <a name="network-utilization-details-for-each-batch"></a>Détails relatifs à l’utilisation réseau pour chaque lot 
Chaque table de lot fournit un résumé de la consommation réseau du lot.

**Bandwidth available for batch** : bande passante disponible pour le lot après prise en compte de la bande passante utilisée pour la réplication delta du lot précédent.

**Approximate bandwidth available for initial replication of batch** : la bande passante disponible pour la réplication initiale des machines virtuelles du lot. 

**Approximate bandwidth consumed for delta replication of batch** : la bande passante nécessaire pour la réplication delta des machines virtuelles du lot. 

**Estimated initial replication time for batch (HH:MM)** : durée estimée de la réplication initiale au format heures:minutes.



## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur [l’estimation des coûts](hyper-v-deployment-planner-cost-estimation.md).
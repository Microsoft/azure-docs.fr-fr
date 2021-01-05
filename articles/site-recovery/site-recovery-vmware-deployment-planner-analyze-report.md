---
title: Analyser le rapport du planificateur de déploiement pour la récupération d’urgence VMware avec Azure Site Recovery
description: Cet article décrit comment analyser un rapport généré par le planificateur de déploiement de récupération pour la récupération d’urgence VMware sur Azure à l’aide d’Azure Site Recovery.
author: rajeswari-mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/4/2019
ms.author: ramamill
ms.openlocfilehash: 7e2db720bb37a25b8511bd1c42c0c18e139aa216
ms.sourcegitcommit: 7a7b6c7ac0aa9dac678c3dfd4b5bcbc45dc030ca
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/02/2020
ms.locfileid: "93186601"
---
# <a name="analyze-the-deployment-planner-report-for-vmware-disaster-recovery-to-azure"></a>Analyser le rapport du planificateur de déploiement pour la récupération d’urgence VMware sur Azure

Le rapport Microsoft Excel généré contient les feuilles suivantes :
## <a name="on-premises-summary"></a>Résumé local
La feuille de calcul du résumé local fournit une vue d’ensemble de l’environnement VMware profilé.

![Résumé local de l’environnement VMware](media/site-recovery-vmware-deployment-planner-analyze-report/on-premises-summary-v2a.png)

**Date de début** et **Date de fin**  : dates de début et de fin du profilage des données prises en compte pour la génération de rapport. Par défaut, la date de début est la date à laquelle le profilage démarre, et la date de fin est la date à laquelle le profilage s’arrête. Il peut s’agir des valeurs StartDate et EndDate si le rapport est généré avec ces paramètres.

**Nombre total de jours de profilage**  : nombre total de jours de profilage compris entre les dates de début et de fin pendant lesquelles le rapport est généré.

**Nombre total de machines virtuelles compatibles**  : nombre total de machines virtuelles compatibles pour lesquelles la bande passante réseau requise, ainsi que le nombre requis de comptes de stockage, de cœurs Microsoft Azure et de serveurs de traitement supplémentaires et de serveurs de configuration sont calculés.

**Nombre total de disques sur toutes les machines virtuelles compatibles**  : nombre utilisé comme nombre d’entrées pour déterminer le nombre de serveurs de traitement supplémentaires et de serveurs de configuration à utiliser dans le déploiement.

**Nombre moyen de disques par machine virtuelle compatible**  : nombre moyen de disques calculé sur toutes les machines virtuelles compatibles.

**Taille de disque moyenne (Go)**  : taille de disque moyenne calculée sur toutes les machines virtuelles compatibles.

**Objectif de point de récupération désiré (minutes)**  : objectif de point de récupération par défaut ou valeur passée pour le paramètre « DesiredRPO » au moment de la génération de rapport afin d’estimer la bande passante requise.

**Bande passante désirée (Mbits/s)**  : valeur que vous avez transmise pour le paramètre « Bandwidth » au moment de la génération de rapport pour estimer le RPO réalisable.

**Activité type des donnée observée par jour (Go)**  : activité moyenne des données observée tous les jours de profilage. Ce nombre est utilisé comme nombre d’entrées pour déterminer le nombre de serveurs de processus supplémentaires et de serveurs de configuration à utiliser dans le déploiement.

## <a name="recommendations"></a>Recommandations

La feuille de recommandations de VMware pour le rapport Azure présente les détails suivants selon le RPO sélectionné :

![Recommandations de VMware pour le rapport Azure](media/site-recovery-vmware-deployment-planner-analyze-report/Recommendations-v2a.png)

### <a name="profiled-data"></a>Données profilées
![La vue des données profilées dans le Deployment planner](media/site-recovery-vmware-deployment-planner-analyze-report/profiled-data-v2a.png)

**Période de données profilées**  : période pendant laquelle le profilage a été exécuté. Par défaut, l’outil inclut toutes les données profilées pour le calcul, sauf s’il génère le rapport pour une période spécifique à l’aide des options StartDate et EndDate pendant la génération du rapport.

**Nom du serveur**  : nom ou adresse IP de VMware vCenter ou de l’hôte ESXi dont le rapport des machines virtuelles est généré.

**Objectif de point de récupération désiré**  : objectif de point de récupération de votre déploiement. Par défaut, la bande passante réseau requise est calculée pour les valeurs RPO de 15, 30 et 60 minutes. En fonction de la sélection, les valeurs concernées sont mises à jour sur la feuille. Si vous avez utilisé le paramètre *DesiredRPOinMin* lors de la génération du rapport, cette valeur est affichée dans le résultat Desired RPO.

### <a name="profiling-overview"></a>Vue d’ensemble du profilage

![Résultats de profilage dans le Deployment planner](media/site-recovery-vmware-deployment-planner-analyze-report/profiling-overview-v2a.png)

**Nombre total de machines virtuelles profilées**  : nombre total de machines virtuelles dont les données profilées sont disponibles. Si le fichier VMListFile contient des noms de machines virtuelles qui n’ont pas été profilées, celles-ci ne sont pas prises en compte dans la génération de rapport et sont exclues du nombre total de machines virtuelles profilées.

**Machines virtuelles compatibles**  : nombre de machines virtuelles pouvant être protégées dans Azure à l’aide de Site Recovery. Il s’agit du nombre total de machines virtuelles compatibles pour lesquelles la bande passante réseau requise, le nombre de comptes de stockage, le nombre de cœurs Azure et le nombre de serveurs de configuration et de serveurs de processus supplémentaires sont calculés. Les détails de chaque machine virtuelle compatible sont disponibles dans la section « Machines virtuelles compatibles ».

**Machines virtuelles incompatibles**  : nombre de machines virtuelles profilées qui sont incompatibles pour la protection avec Site Recovery. Les raisons de l’incompatibilité sont indiquées dans la section Machines virtuelles incompatibles. Si VMListFile contient des noms de machines virtuelles qui n’ont pas été profilées, ces machines virtuelles sont exclues du nombre de machines virtuelles incompatibles. Ces machines virtuelles sont répertoriées comme « Data not found » à la fin de la section Machines virtuelles incompatibles.

**Objectif de point de récupération désiré**  : votre objectif de point de récupération (RPO) souhaité, en minutes. Le rapport est généré pour les trois valeurs RPO : 15 (valeur par défaut), 30 et 60 minutes. La recommandation de la bande passante dans le rapport est modifiée en fonction de votre sélection dans la liste déroulante « Desired RPO » dans l’angle supérieur droit de la feuille. Si vous avez généré le rapport à l’aide du paramètre *-DesiredRPO* avec une valeur personnalisée, cette valeur personnalisée s’affiche par défaut dans la liste déroulante « Desired RPO ».

### <a name="required-network-bandwidth-mbps"></a>Bande passante réseau requise (Mbits/s)

![Bande passante réseau requise dans le Deployment planner](media/site-recovery-vmware-deployment-planner-analyze-report/required-network-bandwidth-v2a.png)

**Pour atteindre le RPO 100 % du temps :** bande passante recommandée en Mbits/s à allouer pour atteindre votre RPO souhaité 100 % du temps. Cette quantité de bande passante doit être dédiée à la réplication différentielle à l’état stable de toutes vos machines virtuelles compatibles afin d’éviter toute violation de RPO.

**Pour atteindre le RPO 90 % du temps :** en raison des tarifs du haut débit ou pour toute autre raison, si vous ne pouvez pas configurer la bande passante nécessaire pour atteindre votre RPO souhaité 100 % du temps, vous pouvez choisir un paramètre de bande passante inférieure pouvant satisfaire votre RPO souhaité 90 % du temps. Pour comprendre les implications de la configuration de cette bande passante réduite, le rapport fournit une analyse par simulation du nombre et de la durée de violations de RPO à attendre.

**Débit atteint :** débit du serveur sur lequel vous avez exécuté la commande GetThroughput pour la région Microsoft Azure où se trouve le compte de stockage. Cette valeur de débit indique le niveau estimé que vous pouvez obtenir lorsque vous protégez les machines virtuelles compatibles à l’aide de Site Recovery, sous réserve que les caractéristiques de réseau et de stockage du serveur de configuration/serveur de processus demeurent identiques à celles du serveur à partir duquel vous avez exécuté l’outil.

Pour la réplication, vous devez définir la bande passante recommandée pour atteindre le RPO souhaité en permanence. Après avoir défini la bande passante, si vous ne constatez pas d’augmentation du débit atteint signalé par l’outil, vérifiez les points suivants :

1. Vérifiez si un réseau Qualité de Service (QoS) limite le débit Site Recovery.

2. Vérifiez si votre coffre Site Recovery se trouve dans la région Microsoft Azure physique prise en charge la plus proche pour réduire la latence du réseau.

3. Vérifiez les caractéristiques de stockage local pour déterminer si vous pouvez améliorer le matériel (par exemple, passer d’un disque dur à un disque SSD).

4. Modifiez les paramètres Site Recovery sur le serveur de processus pour [augmenter la quantité de bande passante réseau utilisée pour la réplication](./site-recovery-plan-capacity-vmware.md#control-network-bandwidth).

Si vous exécutez l’outil sur un serveur de configuration ou un serveur de processus qui a déjà protégé les machines virtuelles, exécutez l’outil plusieurs fois. La valeur de débit atteint change selon la quantité des données d’activité en cours de traitement à ce moment précis.

Pour tous les déploiements Site Recovery d’entreprise, nous vous recommandons d’utiliser [ExpressRoute](https://aka.ms/expressroute).

### <a name="required-storage-accounts"></a>Comptes de stockage requis
Le graphique suivant indique le nombre total de comptes de stockage (standard et premium) nécessaires pour protéger toutes les machines virtuelles compatibles. Pour savoir quel compte de stockage utiliser pour chaque machine virtuelle, consultez la section « VM-storage placement ». Si vous utilisez la version 2.5 du Planificateur de déploiement, cette recommandation indique uniquement le nombre de comptes de stockage du cache standard nécessaires à la réplication, car les données sont écrites directement dans des disques managés.

![Comptes de stockage requis dans Deployment planner](media/site-recovery-vmware-deployment-planner-analyze-report/required-storage-accounts-v2a.png)

### <a name="required-number-of-azure-cores"></a>Required number of Azure cores
Il s’agit du nombre total de cœurs à configurer avant le basculement ou le test de basculement de l’ensemble des machines virtuelles compatibles. Si un nombre insuffisant de cœurs est disponible dans l’abonnement, Site Recovery ne peut pas créer de machines virtuelles au moment du basculement ou du test de basculement.

![Nombre de cœurs Azure dans Deployment planner](media/site-recovery-vmware-deployment-planner-analyze-report/required-cores-v2a.png)

### <a name="required-on-premises-infrastructure"></a>Required on-premises infrastructure
Il s’agit du nombre total de serveurs de processus supplémentaires et de serveurs de configuration à configurer qui suffirait pour protéger toutes les machines virtuelles compatibles. En fonction des [recommandations de taille pour le serveur de configuration](/en-in/azure/site-recovery/site-recovery-plan-capacity-vmware#size-recommendations-for-the-configuration-server) prises en charge, l’outil peut recommander des serveurs supplémentaires. La recommandation est basée sur la valeur la plus élevée entre l’activité quotidienne et le nombre maximal de machines virtuelles protégées (en partant d’une moyenne de trois disques par machine virtuelle), peu importe que le serveur de configuration ou le serveur de processus supplémentaire soit atteint en premier. Vous trouverez tous les détails de l’activité totale quotidienne et du nombre total de disques protégés dans la section « Résumé local ».

![Infrastructure locale requise dans Deployment planner](media/site-recovery-vmware-deployment-planner-analyze-report/required-on-premises-components-v2a.png)

### <a name="what-if-analysis"></a>Analyse de scénarios
Cette analyse indique le nombre de violations susceptibles de se produire pendant la période de profilage lorsque vous configurez une bande passante plus faible pour que le RPO souhaité ne soit rempli que 90 pour cent du temps. Une ou plusieurs violations de RPO peuvent se produire sur un jour donné. Le graphique affiche le pic RPO de la journée.
En vous appuyant sur cette analyse, vous pouvez décider si le nombre de violations de RPO tous les jours et d’accès RPO maximal par jour est acceptable avec la bande passante inférieure spécifiée. S’il est acceptable, vous pouvez allouer la bande passante inférieure pour la réplication. Sinon, allouez la bande passante supérieure, comme suggéré pour atteindre le RPO souhaité 100 pour cent du temps.

![Analyse de scénarios dans Deployment planner](media/site-recovery-vmware-deployment-planner-analyze-report/what-if-analysis-v2a.png)

### <a name="recommended-vm-batch-size-for-initial-replication"></a>Recommended VM batch size for initial replication
Dans cette section, nous recommandons le nombre de machines virtuelles qui peuvent être protégées en parallèle pour effectuer la réplication initiale dans les 72 heures avec la bande passante suggérée pour respecter le RPO souhaité en permanence. Cette valeur est configurable. Pour modifier cette valeur au moment de la génération de rapports, utilisez le paramètre *GoalToCompleteIR*.

Ce graphique affiche une plage de valeurs de bande passante et un nombre de tailles de lot de machines virtuelles pour effectuer une réplication initiale en 72 heures en fonction de la taille moyenne de machine virtuelle détectée sur toutes les machines virtuelles compatibles.

Dans la préversion publique, le rapport ne spécifie pas les machines virtuelles à inclure dans un lot. Vous pouvez utiliser la taille de disque indiquée dans la section « Machines virtuelles compatibles » pour déterminer la taille de chaque machine virtuelle, et sélectionner des machines virtuelles pour un lot, ou les sélectionner en fonction des caractéristiques de charge de travail connues. La durée d’exécution de la réplication initiale change proportionnellement en fonction de la taille de disque réelle des machines virtuelles, de l’espace disque utilisé et du débit réseau disponible.

![Taille de lot recommandée de machines virtuelles](media/site-recovery-vmware-deployment-planner-analyze-report/ir-batching-v2a.png)

### <a name="cost-estimation"></a>Estimation des coûts
Le graphique affiche la vue Résumé du coût estimé de la récupération d’urgence totale (DR) pour Azure de votre région cible choisie et avec la devise que vous avez spécifiée pour la génération du rapport.

![Résumé de l’estimation des coûts](media/site-recovery-vmware-deployment-planner-analyze-report/cost-estimation-summary-v2a.png)

Le résumé vous aide à comprendre le coût que vous devez payer pour le stockage, le calcul, le réseau et la licence lorsque vous protégez toutes vos machines virtuelles compatibles avec Azure à l’aide de Azure Site Recovery. Le coût est calculé à partir de machines virtuelles compatibles et non sur toutes les machines virtuelles profilées.  

Vous pouvez afficher le coût mensuel ou annuel. En savoir plus sur les [régions cibles prises en charge](./site-recovery-vmware-deployment-planner-cost-estimation.md#supported-target-regions) et les [devises prises en charge](./site-recovery-vmware-deployment-planner-cost-estimation.md#supported-currencies).

**Coût par composant**  : le coût total de la reprise d’activité est divisé en quatre composants, à savoir le coût de la licence Azure Site Recovery, du stockage, du réseau et de la capacité de calcul. Le coût est calculé en fonction de la consommation facturée pendant la réplication et au moment de la récupération d’urgence pour le calcul, le stockage (premium et standard), le ExpressRoute/VPN configuré entre le site local et Azure, et la licence de Azure Site Recovery.

**Coût par état** Le coût total de la récupération d’urgence (DR) est catégorisé selon deux états différents, la réplication et l’extraction de la récupération d’urgence.

**Coût de la réplication**  :  coût engendré par la réplication. Il couvre le coût du stockage, du réseau et de la licence d’Azure Site Recovery.

**Coût de simulation de reprise d’activité**  : coût engendré par les basculements de test. Azure Site Recovery prépare des machines virtuelles pendant le basculement de test. Le coût d’extraction de la récupération d’urgence couvre les coûts de calcul et de stockage des machines virtuelles en cours d’exécution.

**Coût de stockage Azure par mois/année** Il montre le coût de stockage total qui sera engagé pour le stockage standard et premium pour la réplication et l’extraction de récupération d’urgence.
Vous pouvez afficher une analyse détaillée des coûts par machine virtuelle dans la feuille [Estimation des coûts](site-recovery-vmware-deployment-planner-cost-estimation.md).

### <a name="growth-factor-and-percentile-values-used"></a>Growth factor and percentile values used
Cette section figurant en bas de la feuille indique la valeur de centile utilisé pour tous les compteurs de performances des machines virtuelles profilées (par défaut : 95e centile) et le facteur de croissance (par défaut : 30 %) qui est utilisé dans tous les calculs.

![Growth factor and percentile values used](media/site-recovery-vmware-deployment-planner-analyze-report/growth-factor-v2a.png)

## <a name="recommendations-with-available-bandwidth-as-input"></a>Recommandations relatives à la bande passante disponible comme entrée

![Recommandations relatives à la bande passante disponible comme entrée](media/site-recovery-vmware-deployment-planner-analyze-report/profiling-overview-bandwidth-input-v2a.png)

Vous pouvez vous trouver dans une situation dans laquelle vous ne pouvez pas configurer une bande passante supérieure à x Mbits/s pour la réplication Site Recovery. L’outil vous permet d’entrer la bande passante disponible (à l’aide du paramètre -Bandwidth lors de la génération de rapport) et d’obtenir le RPO réalisable en minutes. Avec cette valeur de RPO réalisable, vous pouvez choisir de configurer une bande passante supplémentaire ou vous contenter d’une solution de récupération d’urgence avec ce RPO.

![RPO possible pour une bande passante de 500 Mbits/s](media/site-recovery-vmware-deployment-planner-analyze-report/achievable-rpo-v2a.png)

## <a name="vm-storage-placement"></a>VM-storage placement

>[!Note]
>À partir de la version 2.5, le Planificateur de déploiement recommande l’emplacement de stockage pour les machines qui seront répliquées directement sur les disques managés.

![VM-storage placement](media/site-recovery-vmware-deployment-planner-analyze-report/vm-storage-placement-v2a.png)

**Type de stockage de réplication**  : disque managé standard ou premium utilisé pour répliquer toutes les machines virtuelles correspondantes, mentionnées dans la colonne **Machines virtuelles à placer**.

**Enregistrer le type de compte de stockage**  : tous les journaux d’activité de réplication sont stockés dans un compte de stockage standard.

**Préfixe suggéré pour le compte de stockage**  : préfixe suggéré à trois caractères qui permet de nommer le compte de stockage de cache. Vous pouvez utiliser votre propre préfixe, mais la suggestion de l’outil suit la [convention d’affectation de noms aux partitions pour les comptes de stockage](/en-in/azure/storage/blobs/storage-performance-checklist).

**Nom de compte de journal suggéré**  : nom du compte de stockage après l’ajout du préfixe suggéré. Remplacez le nom entre crochets pointus(< and >) avec votre entrée personnalisée.

**Récapitulatif du placement**  : un résumé des disques nécessaires à la protection des machines virtuelles par type de stockage. Il inclut le nombre total de machines virtuelles, la taille totale approvisionnée sur tous les disques et le nombre total de disques.

**Machines virtuelles à placer**  : liste de toutes les machines virtuelles qui doivent être placées dans le compte de stockage donné pour optimiser les performances et l’utilisation.

## <a name="compatible-vms"></a>Machines virtuelles compatibles
![Feuille de calcul Excel des machines virtuelles compatibles](media/site-recovery-vmware-deployment-planner-analyze-report/compatible-vms-v2a.png)

**Nom de la machine virtuelle**  : nom de la machine virtuelle ou adresse IP utilisés dans VMListFile quand un rapport est généré. Cette colonne répertorie également les disques (VMDK) qui sont attachés aux machines virtuelles. Pour distinguer les machines virtuelles vCenter avec des noms ou des adresses IP en double, les noms incluent le nom de l’hôte ESXi. L’hôte ESXi répertorié est celui dans lequel la machine virtuelle a été placée lors de la détection de l’outil pendant le profilage.

**Compatibilité de la machine virtuelle**  : Les valeurs sont **Oui** et **Oui\* *_. _* Oui**\* : pour les instances dans lesquelles la machine virtuelle est adaptée aux [disques SSD Premium](../virtual-machines/disks-types.md). Ici l’activité élevée profilée ou le disque d’E/S par seconde se classe dans la catégorie P20 ou P30, mais la taille du disque entraîne classification inférieure à P10 ou P20. Le compte de stockage décide du type de disque de stockage Premium sur lequel mapper un disque, en fonction de sa taille. Par exemple :
* < 128 Go : disque P10.
* 128 Go à 256 Go : disque P15
* 256 Go à 512 Go : disque P20.
* 512 Go à 1024 Go : disque P30.
* 1025 Go à 2048 Go : disque P40.
* 2049 Go à 4095 Go : disque P50.

Par exemple, si les caractéristiques de charge de travail d’un disque le placent dans la catégorie P20 ou P30, mais que la taille le mappe à un niveau ou à un type de disque de stockage premium inférieur, l’outil marque cette machine virtuelle comme **Oui**\*. L’outil recommande également que vous modifiiez la taille du disque source pour que celui s’adapte au type de disque de stockage premium recommandé, ou que vous modifiiez le type de disque cible après le basculement.

**Storage Type**  : standard ou premium.

**Asrseeddisk (disque managé) créé pour la réplication**  : le nom du disque qui est créé lorsque vous activez la réplication. Il stocke les données et ses captures instantanées dans Azure.

**Peak R/W IOPS (with Growth Factor)** : opérations d’E/S par seconde de lecture/écriture de la charge de travail de pointe sur le disque (95e centile par défaut), y compris le facteur de croissance futur (30 % par défaut). Notez que les E/S par seconde en lecture/écriture d’une machine virtuelle ne sont pas toujours la somme des E/S par seconde en lecture/écriture des disques individuels de la machine virtuelle, car les E/S par seconde en lecture/écriture de pointe de la machine virtuelle représentent le pic de la somme des E/S par seconde de ses disques individuels pendant chaque minute de la période de profilage.

**Peak Data Churn in Mbps (with Growth Factor)** : taux d’activité de pointe sur le disque (95e centile par défaut), y compris le facteur de croissance futur (30 % par défaut). Notez que le taux total d’activité des données de la machine virtuelle n’est pas toujours la somme des taux d’activité des données des disques individuels de la machine virtuelle, car le taux d’activité de pointe de la machine virtuelle représente le pic de la somme du taux d’activité de ses disques individuels pendant chaque minute de la période de profilage.

**Azure VM Size**  : la taille de machine virtuelle d’Azure Cloud Services mappée idéale pour cette machine virtuelle locale. Le mappage est basé sur la mémoire, le nombre de disques/cœurs/cartes réseau de la machine virtuelle locale, et les E/S par seconde en lecture/écriture. La recommandation est toujours la plus petite taille de machine virtuelle Azure qui correspond à toutes les caractéristiques des machines virtuelles locales.

**Number of Disks**  : le nombre total de disques de machines virtuelles (VMDK) sur la machine virtuelle.

**Disk size (GB)**  : la taille totale d’installation de tous les disques de la machine virtuelle. L’outil affiche également la taille des disques individuels de la machine virtuelle.

**Cores**  : le nombre de cœurs de processeur de la machine virtuelle.

**Memory (MB)**  : la mémoire RAM de la machine virtuelle.

**NICs**  : le nombre de cartes réseau de la machine virtuelle.

**Boot Type** : type de démarrage de la machine virtuelle. Le type de démarrage peut prendre la valeur BIOS ou EFI.  Azure Site Recovery prend en charge les machines virtuelles EFI Windows Server (Windows Server 2012, 2012 R2 et 2016) si le nombre de partitions dans le disque de démarrage est inférieure à 4 et la taille du secteur de démarrage est de 512 octets. Pour protéger les machines virtuelles EFI, le service mobilité de Azure Site Recovery doit être la version 9.13 ou ultérieure. Seul le basculement est pris en charge pour les machines virtuelles EFI. La restauration automatique n’est pas prise en charge.  

**OS Type** : le type de système d’exploitation de la machine virtuelle. Cela peut être Windows, Linux ou un autre basé sur le modèle choisi à partir de VMware vSphere lors de la création de la machine virtuelle.  

## <a name="incompatible-vms"></a>Machines virtuelles incompatibles

![Feuille de calcul Excel des machines virtuelles incompatibles
](media/site-recovery-vmware-deployment-planner-analyze-report/incompatible-vms-v2a.png)

**VM Name**  : nom de la machine virtuelle ou adresse IP utilisés dans VMListFile lorsqu’un rapport est généré. Cette colonne répertorie également les disques de machines virtuelles qui sont attachés aux machines virtuelles. Pour distinguer les machines virtuelles vCenter avec des noms ou des adresses IP en double, les noms incluent le nom de l’hôte ESXi. L’hôte ESXi répertorié est celui dans lequel la machine virtuelle a été placée lors de la détection de l’outil pendant le profilage.

**VM Compatibility**  : indique pourquoi la machine virtuelle spécifiée est incompatible avec une utilisation avec Site Recovery. Les raisons sont décrites pour chaque disque incompatible de la machine virtuelle et, en fonction des [limites de stockage](/en-in/azure/storage/common/scalability-targets-standard-account), peuvent figurer parmi les suivantes :

* La taille du disque de données ou la taille du disque du système d’exploitation est incorrecte. [Évaluation](vmware-physical-azure-support-matrix.md#azure-vm-requirements) des limites de prise en charge. 
* La taille totale de machine virtuelle (réplication + TFO) dépasse la limite de taille du compte de stockage prise en charge (35 To). Cette incompatibilité se produit généralement lorsqu’un seul disque de la machine virtuelle présente une caractéristique de performances dépassant les limites maximales prises en charge Azure ou Site Recovery pour le stockage standard. Une telle instance envoie la machine virtuelle dans la zone de stockage premium en mode Push. Néanmoins, la taille maximale prise en charge d’un compte de stockage premium est de 35 To, et une seule et même machine virtuelle protégée ne peut pas être protégée sur plusieurs comptes de stockage. Notez également que, lorsqu’un test de basculement est exécuté sur une machine virtuelle protégée, elle s’exécute dans le compte de stockage où la réplication est en cours. Dans ce cas, configurez 2 fois la taille du disque pour que la progression de la réplication et le test de basculement réussissent en parallèle.

* Les E/S par seconde source excèdent la limite des E/S par seconde prise en charge par le stockage qui est de 7 500 par disque.

* Les E/S par seconde source excèdent la limite des E/S par seconde prise en charge par le stockage qui est de 80 000 par machine virtuelle.

* L’activité moyenne des données dépasse la limite d’activité des données prise en charge par Site Recovery, qui est de 20 Mo/s pour la taille d’E/S moyenne du disque.

* L’activité de pointe des données sur tous les disques de la machine virtuelle dépasse la limite d’activité des données maximale de pointe prise en charge par Site Recovery, qui est de 54 Mo/s par machine virtuelle.

* Les E/S par seconde d’écriture moyennes effectives dépassent la limite des E/S par seconde prise en charge par Site Recovery, qui est de 840 par disque.

* Le stockage calculé des captures instantanées dépasse la limite de stockage des captures instantanées prise en charge, qui est de 10 To.

* L’activité totale des données par jour dépasse la limite d’activité des données prise en charge de 2 To par serveur de processus.


**Peak R/W IOPS (with Growth Factor)**  : les opérations d’E/S par seconde de la charge de travail de pointe sur le disque (95e centile par défaut), y compris le facteur de croissance futur (30 pour cent par défaut). Notez que les E/S par seconde en lecture/écriture de la machine virtuelle ne sont pas toujours la somme des E/S par seconde en lecture/écriture des disques individuels de la machine virtuelle, car les E/S par seconde en lecture/écriture de pointe de la machine virtuelle représentent le pic de la somme des E/S par seconde de ses disques individuels pendant chaque minute de la période de profilage.

**Peak Data Churn in Mbps (with Growth Factor)**  : le taux d’activité de pointe sur le disque (95e centile par défaut), y compris le facteur de croissance futur (par défaut : 30 pour cent). Notez que le taux total d’activité des données de la machine virtuelle n’est pas toujours la somme des taux d’activité des données des disques individuels de la machine virtuelle, car le taux d’activité de pointe de la machine virtuelle représente le pic de la somme du taux d’activité de ses disques individuels pendant chaque minute de la période de profilage.

**Number of Disks**  : le nombre total de disques de machines virtuelles (VMDK) sur la machine virtuelle.

**Disk size (GB)**  : la taille totale d’installation de tous les disques de la machine virtuelle. L’outil affiche également la taille des disques individuels de la machine virtuelle.

**Cores**  : le nombre de cœurs de processeur de la machine virtuelle.

**Memory (MB)**  : la quantité de RAM sur la machine virtuelle.

**NICs**  : le nombre de cartes réseau de la machine virtuelle.

**Boot Type** : type de démarrage de la machine virtuelle. Le type de démarrage peut prendre la valeur BIOS ou EFI.  Azure Site Recovery prend en charge les machines virtuelles EFI Windows Server (Windows Server 2012, 2012 R2 et 2016) si le nombre de partitions dans le disque de démarrage est inférieure à 4 et la taille du secteur de démarrage est de 512 octets. Pour protéger les machines virtuelles EFI, le service mobilité de Azure Site Recovery doit être la version 9.13 ou ultérieure. Seul le basculement est pris en charge pour les machines virtuelles EFI. La restauration automatique n’est pas prise en charge.

**Type de système d’exploitation**  :  type de système d’exploitation de la machine virtuelle. Cela peut être Windows, Linux ou un autre basé sur le modèle choisi à partir de VMware vSphere lors de la création de la machine virtuelle.

## <a name="azure-site-recovery-limits"></a>Limites Azure Site Recovery
Le tableau suivant présente les limites d’Azure Site Recovery. Les limites sont basées sur nos tests, mais ne peuvent pas couvrir toutes les combinaisons d’E/S d’application possibles. Les résultats réels varient en fonction de la combinaison d’E/S de votre application. Pour de meilleurs résultats, même après la planification du déploiement, nous vous recommandons toujours d’effectuer des tests d’application approfondis à l’aide d’un test de basculement pour obtenir une image réelle des performances de l’application.

**Stockage de réplication cible** | **Taille d’E/S moyenne de disque source** |**Activité des données moyenne de disque source** | **Total de l’activité des données de disque source par jour**
---|---|---|---
Stockage Standard | 8 Ko | 2 Mo/s | 168 Go par disque
Disque Premium P10 ou P15 | 8 Ko  | 2 Mo/s | 168 Go par disque
Disque Premium P10 ou P15 | 16 Ko | 4 Mo/s |  336 Go par disque
Disque Premium P10 ou P15 | 32 Ko ou plus | 8 Mo/s | 672 Go par disque
Disque Premium P20 ou P30 ou P40 ou P50 | 8 Ko    | 5 Mo/s | 421 Go par disque
Disque Premium P20 ou P30 ou P40 ou P50 | 16 Ko ou plus | 20 Mo/s | 1 684 Go par disque

**Activité de données sources** | **Limite maximale**
---|---
Activité des données de pointe sur tous les disques d’une machine virtuelle | 54 Mo/s
Activité des données maximale par jour prise en charge par un serveur de processus | 2 To

Il s’agit de moyennes en partant sur un chevauchement d’E/S de 30 pour cent. Site Recovery est capable de gérer un débit plus élevé en fonction du ratio de chevauchement, de tailles d’écriture plus grandes et du comportement d’E/S des charges de travail réelles. Les valeurs précédentes supposent un retard de traitement typique de cinq minutes. Autrement dit, une fois que les données sont chargées, elles sont traitées, et un point de récupération est créé dans un délai de cinq minutes.


## <a name="cost-estimation"></a>Estimation des coûts
En savoir plus sur [l’estimation des coûts](site-recovery-vmware-deployment-planner-cost-estimation.md).


## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur [l’estimation des coûts](site-recovery-vmware-deployment-planner-cost-estimation.md).
---
title: Résoudre les problèmes de performances des machines virtuelles Azure sur Linux ou Windows
description: Cet article décrit comment résoudre les problèmes de performances génériques des machines virtuelles en supervisant et en observant les goulots d’étranglement. Il offre des solutions possibles aux problèmes pouvant survenir.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: top-support-issue, azure-resource-manager
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 09/18/2019
ms.author: v-miegge
ms.openlocfilehash: 53fd2332224d903c5a4b33563470cf3569f82b13
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86526654"
---
# <a name="troubleshoot-azure-virtual-machine-performance-on-linux-or-windows"></a>Résoudre les problèmes de performances des machines virtuelles Azure sur Linux ou Windows

Cet article décrit comment résoudre les problèmes de performances génériques des machines virtuelles en supervisant et en observant les goulots d’étranglement. Il offre des solutions possibles aux problèmes pouvant survenir. Outre la supervision, vous pouvez utiliser Perfinsights qui peut fournir un rapport avec les bonnes pratiques suggérées et les goulots d’étranglement clés concernant les E/S, le processeur et la mémoire. PerfInsights est disponible à la fois pour les machines virtuelles [Windows](./how-to-use-perfinsights.md) et [Linux](./how-to-use-perfinsights-linux.md) dans Azure.

Cet article vous guide dans l’utilisation de la supervision pour diagnostiquer les goulots d’étranglement de performances.

## <a name="enabling-monitoring"></a>Activation de la surveillance

### <a name="azure-iaas-virtual-machine-monitoring"></a>Supervision d’une machine virtuelle IaaS Azure

Pour superviser la machine virtuelle invitée, utilisez la fonctionnalité de supervision de machine virtuelle Azure, qui vous informe de certaines conditions de ressources générales. Pour vérifier si les diagnostics de machine virtuelle sont activés, consultez [Vue d’ensemble des journaux de ressource Azure](../../azure-monitor/learn/tutorial-resource-logs.md). Si vous voyez le message suivant, cela signifie probablement que les diagnostics ne sont pas activés :

![La supervision n’est pas activée.](media/troubleshoot-performance-virtual-machine-linux-windows/1-virtual-machines-monitoring-not-enabled.png)
 
### <a name="enable-vm-diagnostics-through-microsoft-azure-portal"></a>Activer les diagnostics de machine virtuelle à l’aide du portail Microsoft Azure

Pour activer les diagnostics de machine virtuelle :

1. Accédez à la machine virtuelle.
2. Cliquez sur **Paramètres de diagnostic**.
3. Sélectionnez le compte de stockage et cliquez sur **Activer la supervision d’invités**.

   ![Cliquer sur Paramètres, puis sur Diagnostics](media/troubleshoot-performance-virtual-machine-linux-windows/2-virtual-machines-diagnostics.png)

Vous pouvez cocher le compte de stockage utilisé pour l’installation des diagnostics sous l’onglet **Agent**, dans **Paramètres de diagnostic**.

![Cocher le compte de stockage](media/troubleshoot-performance-virtual-machine-linux-windows/3-check-storage-account.png)

### <a name="enable-storage-account-diagnostics-through-azure-portal"></a>Activer les diagnostics de compte de stockage à l’aide du portail Azure

Le stockage est un niveau très important lorsque nous avons l’intention d’analyser les performances d’E/S d’une machine virtuelle dans Azure. Pour obtenir des métriques liées au stockage, nous devons activer les diagnostics en tant qu’étape supplémentaire. Cela peut également être activé si nous souhaitons uniquement analyser les compteurs liés au stockage.

1. Identifiez le ou les comptes de stockage que votre machine virtuelle utilise en sélectionnant la machine virtuelle. Cliquez sur **Paramètres**, puis sur **Disques** :

   ![Cliquer sur Paramètres, puis sur Disques](media/troubleshoot-performance-virtual-machine-linux-windows/4-storage-disks-disks-selection.png)

2. Dans le portail, accédez au compte de stockage (ou aux comptes de stockage) de la machine virtuelle et suivez les étapes ci-dessous :

   1. Cliquez sur Vue d’ensemble pour le compte de stockage que vous avez trouvé à l’étape précédente.
   2. Les métriques par défaut sont affichées. 

    ![Mesures par défaut](media/troubleshoot-performance-virtual-machine-linux-windows/5-default-metrics.png)

3. Cliquez sur l’une des métriques pour afficher un autre panneau avec davantage d’options de configuration et d’ajout de métriques.

   ![Ajouter des métriques](media/troubleshoot-performance-virtual-machine-linux-windows/6-add-metrics.png)

Pour configurer ces options :

1.  Sélectionnez **Métriques**.
2.  Sélectionnez la **ressource** (compte de stockage).
3.  Sélectionnez l’**espace de noms**.
4.  Sélectionnez la **métrique**.
5.  Sélectionnez le type d’**agrégation**.
6.  Vous pouvez épingler cette vue dans le tableau de bord.

## <a name="observing-bottlenecks"></a>Observation des goulots d’étranglement

Après avoir terminé le processus d’installation initial pour les métriques nécessaires et activé les diagnostics pour la machine virtuelle et le compte de stockage associé, nous pouvons passer à la phase d’analyse.

### <a name="accessing-the-monitoring"></a>Accès à la supervision

Sélectionnez la machine virtuelle Azure que vous souhaitez examiner, puis sélectionnez **Supervision**.

![Sélectionnez Surveillance.](media/troubleshoot-performance-virtual-machine-linux-windows/7-select-monitoring.png)
 
### <a name="timelines-of-observation"></a>Intervalle de temps d’observation

Passez en revue vos données pour détecter la présence de goulots d’étranglement. Si votre ordinateur semble fonctionner correctement, mais qu’une baisse de performances récente a été signalée, examinez les données sur une plage de temps qui englobe les données de métriques de performance avant la modification signalée, au moment où le problème est survenu et après.

### <a name="check-for-cpu-bottleneck"></a>Vérifier la présence d’un goulot d’étranglement au niveau du processeur

![Vérifier la présence d’un goulot d’étranglement au niveau du processeur](media/troubleshoot-performance-virtual-machine-linux-windows/8-cpu-bottleneck-time-range.png)

1. Modifiez le graphe.
2. Définissez l’intervalle de temps.
3. Vous devez ensuite ajouter le compteur : Pourcentage d’utilisation du processeur par le système d’exploitation invité
4. Enregistrez.

### <a name="cpu-observe-trends"></a>Processeur : observation des tendances

Quand vous examinez les problèmes de performances, tenez compte des tendances et déterminez si elles vous concernent. Dans les sections suivantes, nous allons utiliser les graphes de supervision du portail pour visualiser les tendances. Ils peuvent également être utiles pour faire des recoupements entre différents comportements de ressources sur la même période. Pour personnaliser les graphes, cliquez sur [Plateforme de données Azure Monitor](../../azure-monitor/platform/data-platform.md).

Pic : un pic peut être associé à une tâche planifiée ou un événement connu. Si vous pouvez identifier la tâche, déterminez si elle s’exécute au niveau de performance requis. Si les performances sont acceptables, vous n’aurez peut-être pas besoin d’augmenter les ressources.

Pic et niveau constant : ceci indique souvent une nouvelle charge de travail. S’il ne s’agit pas d’une charge de travail reconnue, activez la supervision sur la machine virtuelle pour déterminer le ou les processus à l’origine de ce comportement. Une fois le processus reconnu, déterminez si l’augmentation de la consommation est due à un code inefficace ou à une consommation normale. Si la consommation est normale, déterminez si le processus fonctionne au niveau de performance requis.

Niveau constant : déterminez si la machine virtuelle s’exécute toujours à ce niveau ou si cela se produit seulement depuis l’activation du diagnostic. Si c’est le cas, identifiez le ou les processus à l’origine du problème et envisagez d’ajouter d’autres ressources de ce type.

Augmentation régulière : une augmentation régulière de la consommation est souvent liée à un code inefficace ou à un processus utilisant la charge de travail utilisateur de façon plus intensive.

### <a name="high-cpu-utilization-remediation"></a>Solution en cas d’utilisation élevée du processeur

Si votre application ou votre processus ne s’exécute pas au niveau de performances approprié et que vous constatez une utilisation du processeur constante de 95 % ou plus, vous pouvez effectuer l’une des opérations suivantes :

* Pour une résolution immédiate, augmentez la taille de la machine virtuelle en choisissant un nombre de cœurs supérieur.
* Essayez de comprendre le problème : localisez l’application/le processus concerné et procédez à la résolution du problème en conséquence.

Si vous avez augmenté la taille de la machine virtuelle et que le processeur fonctionne toujours à 95 %, déterminez si ce paramètre améliore les performances ou le débit de l’application de façon à offrir un niveau acceptable. Si ce n’est pas le cas, procédez à la résolution du problème au niveau de l’application/du processus concerné.

Vous pouvez utiliser Perfinsights pour [Windows](./how-to-use-perfinsights.md) ou [Linux](./how-to-use-perfinsights-linux.md) afin d’analyser le processus qui pilote la consommation du processeur. 

## <a name="check-for-memory-bottleneck"></a>Vérifier la présence d’un goulot d’étranglement au niveau de la mémoire

Pour voir les métriques :

1. Ajoutez une section.
2. Ajoutez une vignette.
3. Ouvrez la galerie.
4. Sélectionnez la vignette Utilisation de la mémoire et faites-la glisser. Une fois la vignette ancrée, cliquez avec le bouton droit et sélectionnez **6x4**.

### <a name="memory-observe-trends"></a>Mémoire : observation des tendances

La mesure d’utilisation de la mémoire indique la quantité de mémoire consommée avec la machine virtuelle. Essayez de comprendre la tendance et de déterminer si elle correspond à la période à laquelle vous rencontrez des problèmes. La mémoire disponible doit toujours être supérieure à 100 Mo.

Pic et niveau constant/consommation régulière et constante : une utilisation élevée de la mémoire n’est pas nécessairement à l’origine de mauvaises performances. En effet, certaines applications telles que les moteurs de bases de données relationnelles allouent une grande quantité de mémoire, et cette utilisation n’est pas toujours significative. Toutefois, la présence de plusieurs applications gourmandes en mémoire peut entraîner des pertes de performances. Ceci est dû à la contention de mémoire, qui provoque des insuffisances et des opérations de pagination et d’échange sur le disque. Ces pertes de performances représentent souvent une cause majeure de baisse des performances d’une application.

Augmentation régulière de la consommation : ceci peut être dû à la mise en route d’une application et se produit souvent lors du démarrage de moteurs de base de données. Toutefois, cela peut également être le signe d’une fuite de mémoire dans une application. Identifiez l’application et déterminez si le comportement est attendu.

Utilisation d’un fichier de pagination ou d’échange : vérifiez si vous utilisez le fichier de pagination Windows (sur D:\)) ou le fichier d’échange Linux (à l’emplacement `/dev/sdb`) de manière intensive. Si ces volumes ne contiennent rien à l’exception de ces fichiers, vérifiez si ces disques font l’objet d’un nombre élevé de lectures/écritures. Ce problème indique des conditions de mémoire insuffisante.

### <a name="high-memory-utilization-remediation"></a>Solution en cas d’utilisation élevée de la mémoire

Pour résoudre les problèmes d’utilisation élevée de la mémoire, effectuez l’une des opérations suivantes :

* Pour une résolution immédiate ou en cas d’utilisation d’un fichier de pagination ou d’échange, augmentez la taille de la machine virtuelle en choisissant une mémoire supérieure, puis supervisez la machine virtuelle.
* Essayez de comprendre le problème : localisez les applications/processus concernés et identifiez les applications gourmandes en mémoire.
* Si vous savez de quelle application il s’agit, vérifiez si l’allocation de mémoire peut être limitée.

Si, après une mise à niveau vers une machine virtuelle plus grande, vous constatez toujours une augmentation régulière et constante jusqu’à 100 %, identifiez l’application/le processus concerné et procédez à la résolution du problème.

Vous pouvez utiliser Perfinsights pour [Windows](./how-to-use-perfinsights.md) ou [Linux](./how-to-use-perfinsights-linux.md) afin d’analyser le processus qui pilote la consommation de mémoire. 

## <a name="check-for-disk-bottleneck"></a>Vérifier la présence d’un goulot d’étranglement au niveau du disque

Pour vérifier le sous-système de stockage de la machine virtuelle, vérifiez les diagnostics au niveau de la machine virtuelle Azure. Utilisez pour cela les compteurs fournis dans les diagnostics de machine virtuelle et les diagnostics de compte de stockage.

Dans le cas d’une résolution des problèmes spécifique aux machines virtuelles, vous pouvez utiliser Perfinsights pour [Windows](./how-to-use-perfinsights.md) ou [Linux](./how-to-use-perfinsights-linux.md), ce qui peut aider à analyser quel processus pilote les E/S. 

Notez que nous ne proposons pas de compteurs pour les comptes de stockage redondants interzones et Premium. En cas de problème lié à ces compteurs, ouvrez un cas de support.

### <a name="viewing-storage-account-diagnostics-in-monitoring"></a>Visualisation des diagnostics de compte de stockage dans la fonctionnalité de supervision

Pour utiliser les éléments ci-dessous, accédez au compte de stockage associé à la machine virtuelle dans le portail :

![Visualisation des diagnostics de compte de stockage dans la fonctionnalité de supervision](media/troubleshoot-performance-virtual-machine-linux-windows/9-virtual-machine-storage-account.png)

1. Modifiez le graphe de supervision.
2. Définissez l’intervalle de temps.
3. Ajoutez les compteurs décrits dans les étapes ci-dessous.
4. Enregistrez les modifications.

### <a name="disk-observe-trends-standard-storage-only"></a>Disque : observation des tendances (stockage standard uniquement)

Pour identifier les problèmes de stockage, examinez les métriques de performances dans les diagnostics de compte de stockage et les diagnostics de machine virtuelle.

Pour chaque vérification ci-dessous, recherchez les principales tendances quand les problèmes surviennent dans l’intervalle de temps concerné.

#### <a name="check-azure-storage-availability--add-the-storage-account-metric-availability"></a>Vérifier la disponibilité du stockage Azure – Ajouter la métrique de compte de stockage : disponibilité

Si vous constatez une baisse de la disponibilité, cela peut indiquer un problème avec la plateforme. Vérifiez l’[état Azure](https://azure.microsoft.com/status/). Si aucun problème n’y est indiqué, ouvrez une nouvelle demande de support.

#### <a name="check-for-azure-storage-timeout---add-the-storage-account-metrics"></a>Vérifier le délai d’expiration de stockage Azure – Ajouter les métriques de compte de stockage :

* ClientTimeoutError
* ServerTimeOutError
* AverageE2ELatency
* AverageServerLatency
* TotalRequests

Les valeurs des métriques *TimeOutError indiquent qu’une opération d’E/S a duré trop longtemps et qu’elle a expiré. Les étapes suivantes vous aideront à identifier les causes potentielles.

L’augmentation de la valeur AverageServerLatency en même temps que la valeur TimeOutErrors peut être liée à un problème de plateforme. Dans ce cas, ouvrez une nouvelle demande de support.

La valeur AverageE2ELatency représente la latence du client. Vérifiez le nombre d’IOPS effectuées par l’application. Vérifiez si la métrique TotalRequests présente une augmentation ou un niveau élevé constant. Cette métrique représente les IOPS. Si vous commencez à atteindre les limites du compte de stockage ou d’un disque dur virtuel unique, la latence peut être liée à la limitation.

#### <a name="check-for-azure-storage-throttling---add-the-storage-account-metrics-throttlingerror"></a>Vérifier la limitation de stockage Azure - Ajouter les métriques de compte de stockage : ThrottlingError

Les valeurs de limitation indiquent que vous êtes limité au niveau du compte de stockage et que vous atteignez la limite d’IOPS du compte. Vous pouvez déterminer si vous atteignez le seuil d’IOPS en vérifiant la métrique **TotalRequests**.

Notez que chaque disque dur virtuel a une limite de 500 IOPS ou 60 Mbits/s, mais est restreint par la limite cumulée de 20 000 IOPS par compte de stockage.

Cette métrique ne vous permet pas d’identifier le blob à l’origine de la limitation ni ceux qui s’en trouvent affectés. Toutefois, vous atteignez les limites d’IOPS ou d’entrée/sortie du compte de stockage.

Pour déterminer si vous atteignez la limite d’IOPS, accédez aux diagnostics de compte de stockage et vérifiez la métrique TotalRequests pour voir si sa valeur approche 20 000. Déterminez si le modèle a subi un changement, si vous voyez la limite pour la première fois ou si cette limite se présente à un moment spécifique.

Avec les nouvelles offres de disque sous le stockage standard, les limites de débit et IOPS peuvent différer, mais la limite cumulée du compte de stockage standard est de 20 000 IOPS (le stockage Premium a des limites différentes au niveau du compte ou du disque). Apprenez-en plus sur les différentes offres de disque de stockage standard et les limites par disque :

* [Cibles de scalabilité et de performance des disques de machine virtuelle sur Windows](../windows/disk-scalability-targets.md).

#### <a name="references"></a>References

* [Objectifs d’extensibilité et de performances pour les comptes de stockage d’objets blob de pages Premium](../../storage/blobs/scalability-targets-premium-page-blobs.md)

La bande passante du compte de stockage est mesurée à l’aide des métriques de compte de stockage : TotalIngress et TotalEgress. Les seuils de bande passante sont différents selon le type de redondance et les régions.

* [Objectifs d’extensibilité et de performances pour les comptes de stockage standard](../../storage/common/scalability-targets-standard-account.md)

Comparez les valeurs TotalIngress et TotalEgress aux limites d’entrée et de sortie pour le type de redondance et la région du compte de stockage.

Vérifiez les limites de débit des disques durs virtuels attachés à la machine virtuelle. Ajoutez les métriques de machine virtuelle de lecture et écriture sur disque.

Les nouvelles offres de disque sous le stockage standard ont des limites de débit et IOPS différentes (les IOPS ne sont pas exposées par disque dur virtuel). Examinez les données pour voir si vous atteignez les limites du débit combiné (en Mbits/s) du ou des disques durs virtuels au niveau de la machine virtuelle à l’aide des métriques de lecture et écriture sur disque, puis optimisez la configuration du stockage de la machine virtuelle pour porter la capacité au-delà des limites d’un seul disque dur virtuel. Apprenez-en plus sur les différentes offres de disque de stockage standard et les limites par disque :

* [Cibles de scalabilité et de performance des disques de machine virtuelle sur Windows](../windows/disk-scalability-targets.md).

### <a name="high-disk-utilizationlatency-remediation"></a>Solution en cas d’utilisation/latence élevée du disque

Réduire la latence du client et optimiser les E/S de machine virtuelle pour porter la capacité au-delà des limites d’un disque dur virtuel

* [Optimisation des E/S pour Windows dans Azure](../../azure-sql/virtual-machines/windows/performance-guidelines-best-practices.md?toc=/azure/virtual-machines/windows/toc.json)

* [Optimisation des E/S pour Linux dans Azure](/archive/blogs/igorpag/azure-storage-secrets-and-linux-io-optimizations)

#### <a name="reduce-throttling"></a>Réduire la limitation

Si vous atteignez les limites supérieures des comptes de stockage, rééquilibrez les disques durs virtuels entre les comptes de stockage. Consultez [Objectifs de performance et de scalabilité du stockage Azure](../../storage/common/scalability-targets-standard-account.md).

### <a name="increase-throughput-and-reduce-latency"></a>Augmenter le débit et réduire la latence

En présence d’une application sensible à la latence, si vous avez besoin d’un débit élevé, migrez vos disques durs virtuels vers le stockage Premium Azure à l’aide des machines virtuelles de séries DS et GS.

Ces articles décrivent les scénarios spécifiques :

* [Migration vers le stockage Premium Azure](../windows/migrate-to-managed-disks.md)

* [Utilisation du stockage Premium Azure avec SQL Server](/previous-versions/azure/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-sql-server-premium-storage)

## <a name="next-steps"></a>Étapes suivantes

Si vous avez besoin d’une aide supplémentaire à quelque étape que ce soit dans cet article, contactez les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/).

Vous pouvez également signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur **Obtenir un support**.

---
title: Comprendre un redémarrage du système pour une machine virtuelle Azure | Microsoft Docs
description: Répertorie les événements pouvant entraîner le redémarrage d’une machine virtuelle
services: virtual-machines
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: ''
ms.service: virtual-machines
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/31/2018
ms.author: genli
ms.openlocfilehash: 4694b6ac829c42f20c6783810c248ee18d220433
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91965758"
---
# <a name="understand-a-system-reboot-for-azure-vm"></a>Comprendre un redémarrage du système pour Azure

Les machines virtuelles (VM) Azure peuvent parfois redémarrer sans raison apparente, même si vous n’êtes pas à l’origine d’une opération de redémarrage. Cet article répertorie les actions et les événements qui peuvent entraîner le redémarrage des machines virtuelles et présente comment éviter les problèmes de redémarrage inattendus ou réduire leur impact.

## <a name="configure-the-vms-for-high-availability"></a>Configurer les machines virtuelles de sorte qu’elles soient compatibles avec la haute disponibilité

La meilleure façon de protéger une application en cours d’exécution sur Azure contre un redémarrage ou un arrêt de la machine virtuelle consiste à configurer une haute disponibilité sur ces machines virtuelles.

Pour assurer ce niveau de redondance de votre application, nous vous recommandons de regrouper au moins deux machines virtuelles dans un groupe à haute disponibilité. Cette configuration assure la disponibilité d’au moins une des machines virtuelles pendant un événement de maintenance planifié ou non, avec le niveau de 99,95 % stipulé dans le [contrat de niveau de service (SLA) Azure](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_5/).

Pour plus d’informations sur les groupes à haute disponibilité, consultez [Gérer la disponibilité des machines virtuelles](../manage-availability.md).

## <a name="resource-health-information"></a>Informations sur Resource Health

Azure Resource Health est un service qui expose l’intégrité des ressources Azure individuelles et fournit des conseils applicables permettant de résoudre les problèmes. Dans un environnement cloud où il est impossible d’accéder directement aux serveurs ou aux éléments d’infrastructure, Resource Health vise à réduire le temps passé à résoudre des problèmes. L’objectif est notamment de réduire le temps passé à déterminer si la cause du problème est liée à l’application ou à un événement interne à la plateforme Azure. Pour plus d’informations, consultez la section [Understand and use Resource Health](../../service-health/resource-health-overview.md) (Comprendre et utiliser Resource Health).

## <a name="actions-and-events-that-can-cause-the-vm-to-reboot"></a>Actions et événements pouvant entraîner le redémarrage de la machine virtuelle

### <a name="planned-maintenance"></a>Maintenance planifiée

Microsoft Azure exécute régulièrement des mises à jour afin d’améliorer la fiabilité, les performances et la sécurité de l’infrastructure hôte qui supporte les machines virtuelles. Nombre de ces mises à jour sont exécutées sans impact sur les machines virtuelles ou les services cloud, y compris les mises à jour de préservation de la mémoire.

Toutefois, certaines mises à jour nécessitent un redémarrage. Dans ce cas, les machines virtuelles s’arrêtent pendant la mise à jour de l’infrastructure, puis redémarrent une fois cette dernière terminée.

Découvrez en quoi consiste la maintenance planifiée Azure et son incidence possible sur la disponibilité de vos machines virtuelles Linux en consultant les articles ci-dessous. Ces articles fournissent des informations sur le processus de maintenance planifiée Azure et sa planification afin de réduire davantage l’impact.

- [Maintenance planifiée des machines virtuelles dans Azure](../maintenance-and-updates.md?bc=/azure/virtual-machines/windows/breadcrumb/toc.json&toc=/azure/virtual-machines/windows/toc.json)
- [Planification d’une maintenance planifiée sur des machines virtuelles Azure](../maintenance-and-updates.md?bc=/azure/virtual-machines/windows/breadcrumb/toc.json&toc=/azure/virtual-machines/windows/toc.json)

### <a name="memory-preserving-updates"></a>Mises à jour de préservation de la mémoire

Pour cette classe de mises à jour dans Microsoft Azure, les utilisateurs ne constatent aucun impact sur leurs machines virtuelles en cours d’exécution. La plupart de ces mises à jour sont des composants ou des services qui peuvent être mis à jour sans interférer avec l'instance en cours d'exécution. Certaines sont des mises à jour d’infrastructure de la plateforme sur le système d’exploitation hôte qui peuvent être appliquées sans requérir un redémarrage des machines virtuelles.

Ces mises à jour de préservation de la mémoire sont réalisées avec la technologie qui permet la migration en direct. Lors de sa mise à jour, la machine virtuelle est mise *en pause*. Cette mise en pause permet de préserver la mémoire RAM, pendant que le système d’exploitation hôte sous-jacent reçoit les correctifs et mises à jour nécessaires. La machine virtuelle redémarre après 30 secondes d’arrêt. Une fois la machine virtuelle redémarrée, son horloge est synchronisée automatiquement.

En raison de la courte durée de la pause, le déploiement de mises à jour via ce mécanisme permet de réduire considérablement l’impact sur les machines virtuelles. Toutefois, toutes les mises à jour ne peuvent être déployées de cette manière. 

Les mises à jour multi-instances (pour les machines virtuelles d’un groupe à haute disponibilité) se voient appliquer un seul domaine de mise à jour à la fois.

> [!NOTE]
> Les machines Linux dotées de versions anciennes de noyau sont affectées par une alerte sur le noyau au cours de cette méthode de mise à jour. Pour éviter ce problème, mettez à niveau le noyau vers la version 3.10.0-327.10.1 ou une version ultérieure. Pour plus d’informations, consultez [Une machine virtuelle Linux Azure sur un noyau 3.10 panique après une mise à niveau du nœud hôte](https://support.microsoft.com/help/3212236).

### <a name="user-initiated-reboot-or-shutdown-actions"></a>Actions d’arrêt ou de redémarrage initiées par l’utilisateur

Si vous effectuez un redémarrage à partir du portail Azure, d’Azure PowerShell, d’une interface de ligne de commande ou d’une API REST, l’événement est consigné dans le [journal d’activité Azure](../../azure-monitor/platform/platform-logs-overview.md).

Si vous effectuez un redémarrage à partir du système d’exploitation de la machine virtuelle, l’événement est consigné dans le journal système.

Plusieurs actions de modification de la configuration peuvent également entraîner le redémarrage de la machine virtuelle. En règle générale, un message d’avertissement s’affiche, vous indiquant que l’exécution d’une action particulière entraîne un redémarrage de la machine virtuelle. Il peut s’agir d’opérations de redimensionnement de machines virtuelles, de modification du mot de passe du compte d’administration et de la définition d’une adresse IP statique.

### <a name="azure-security-center-and-windows-update"></a>Azure Security Center et Windows Update

Azure Security Center recherche quotidiennement les mises à jour manquantes du système d’exploitation sur les machines virtuelles Windows et Linux. Security Center récupère une liste des mises à jour de sécurité et critiques disponibles dans Windows Update ou WSUS (Windows Server Update Services), selon le service configuré sur les machines virtuelles Windows. Security Center recherche également les dernières mises à jour pour les systèmes Linux. S’il manque une mise à jour système sur votre machine virtuelle, Security Center vous recommande de l’appliquer. L’application de ces mises à jour du système est contrôlée par Security Center sur le portail Azure. L’application de certaines mises à jour peut nécessiter un redémarrage des machines virtuelles. Pour en savoir plus, consultez [Appliquer les mises à jour système dans Azure Security Center](../../security-center/asset-inventory.md).

Comme pour les serveurs locaux, Azure n’envoie pas les mises à jour de Windows Update à des machines virtuelles Windows, car ces machines doivent être gérées par l’utilisateur. Cependant, vous êtes invité à maintenir la configuration automatique de Windows Update activée. L’installation automatique des mises à jour de Windows Update peut également entraîner des redémarrages une fois les mises à jour appliquées. Pour plus d’informations, consultez [Windows Update : Forum Aux Questions](https://support.microsoft.com/help/12373/windows-update-faq).

### <a name="other-situations-affecting-the-availability-of-your-vm"></a>Autres situations affectant la disponibilité de votre machine virtuelle

Il existe des autres cas dans lesquels Azure peut interrompre activement l’utilisation d’une machine virtuelle. Vous recevrez des e-mails de notification avant d’effectuer cette action pour vous donner la possibilité de résoudre les problèmes sous-jacents. Les violations de sécurité et l’expiration de modes de paiement sont des exemples de problèmes qui affectent la disponibilité d’une machine virtuelle.

### <a name="host-server-faults"></a>Erreurs du serveur hôte

La machine virtuelle est hébergée sur un serveur physique en cours d’exécution à l’intérieur d’un centre de données Azure. Le serveur physique exécute un agent appelé Agent hôte et quelques autres composants Azure. Lorsque ces composants logiciels Azure du serveur physique ne répondent plus, le système de surveillance déclenche un redémarrage du serveur hôte pour tenter une récupération. La machine virtuelle est généralement de nouveau disponible dans les cinq minutes et continue à résider sur le même hôte que précédemment.

Les erreurs de serveur sont généralement dues à une défaillance matérielle telles que la défaillance d’un disque dur ou un disque SSD. Azure surveille en permanence ces occurrences, identifie les bogues sous-jacents et déploie les mises à jour après que l’atténuation a été implémentée et testée.

Étant donné que certaines erreurs du serveur hôte peuvent être spécifiques à ce serveur, une situation de redémarrage de machine virtuelle répétée pourrait être améliorée par un redéploiement manuel de celle-ci sur un autre serveur hôte. Cette opération peut être déclenchée à l’aide de l’option de **redéploiement** sur la page Détails de la machine virtuelle, ou par l’arrêt et le redémarrage de la machine virtuelle sur le portail Azure.

### <a name="auto-recovery"></a>Récupération automatique

Si, pour une raison quelconque, le serveur hôte ne peut pas redémarrer, la plateforme Azure lance une action de récupération automatique pour isoler le serveur hôte défaillant en vue d’un examen approfondi. 

Toutes les machines virtuelles sont automatiquement déplacées vers un autre serveur hôte en état de fonctionnement. Ce processus est généralement terminé dans les 15 minutes. Pour en savoir plus sur le processus de récupération automatique, consultez [Récupération automatique des machines virtuelles](https://azure.microsoft.com/blog/service-healing-auto-recovery-of-virtual-machines).

### <a name="unplanned-maintenance"></a>Maintenance non planifiée

Il est rare que l’équipe d’exploitation Azure ait besoin d’effectuer des activités de maintenance pour garantir l’intégrité globale de la plateforme Azure. Ce comportement peut affecter la disponibilité de la machine virtuelle et aboutit généralement à la même action de récupération automatique que celle décrite précédemment.  

La maintenance non planifiée comprend les actions suivantes :

- Défragmentation urgente de nœud
- Mises à jour du commutateur réseau urgente

### <a name="vm-crashes"></a>Incidents de machine virtuelle

Des problèmes au sein d’une machine virtuelle pourraient causer son redémarrage. La charge de travail ou le rôle en cours d’exécution sur la machine virtuelle pourrait déclencher une vérification des bogues au sein du système d’exploitation invité. Pour identifier la raison de l’incident, consultez les journaux des applications et du système pour les machines virtuelles Windows et les journaux d’activité de série pour les machines virtuelles Linux.

### <a name="storage-related-forced-shutdowns"></a>Arrêt forcés relatifs au stockage

Les machines virtuelles d’Azure reposent sur des disques virtuels pour le système d’exploitation et sur le stockage de données hébergé sur l’infrastructure de stockage Azure. Chaque fois que la disponibilité ou la connectivité entre la machine virtuelle et les disques virtuels associés est affectée pendant plus de 120 secondes, la plateforme Azure effectue un arrêt forcé des machines virtuelles afin d’éviter une altération des données. Les machines virtuelles sont automatiquement remises sous tension une fois la connectivité de stockage restaurée. 

La durée de l’arrêt peut être de cinq minutes ou beaucoup plus longue. Voici un des cas spécifiques associé aux arrêts forcés relatifs au stockage : 

**Dépassement des limites d’E/S**

Il se peut que les machines virtuelles soient temporairement arrêtées lorsque les demandes d’E/S sont constamment limitées en raison d’un volume d’opérations d’E/S par seconde dépassant les limites d’E/S du disque. (Stockage de disque standard limité à 500 E/S par seconde.) Pour atténuer ce problème, utilisez l’entrelacement de disques ou configurez l’espace de stockage au sein de la machine virtuelle invitée en fonction de la charge de travail. 

### <a name="other-incidents"></a>Autres incidents

Dans de rares circonstances, un problème étendu peut affecter plusieurs serveurs dans un centre de données Azure. Si cela se produit, l’équipe Azure envoie des notifications par e-mail aux abonnements concernés. Vous pouvez consulter le [Tableau de bord d’intégrité des services Azure](https://azure.microsoft.com/status/) et le portail Azure pour connaître l’état des pannes en cours et des incidents passés.